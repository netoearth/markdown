## The End Result

Let’s start with the visually-pleasing part, which is probably what brought you here. Here is what my dashboard looks like…

![](https://miro.medium.com/max/1100/1*uGGEtqbi9G_gFrfsqEIUJA.png)

Overview of the Grafana dashboard for my weather station

## The Layout

There are a lot of moving parts in my homelab to make this happen. Your version of this could be simpler, this is what worked for me. I’ll breakdown each component in later sections, but I figured it’s better to start with the 10,000 foot view before diving into the details.

![A component graph of how the physical hardware and software relate. A WS2032 weather station sends data on 433 MHz radio which is picked up by receiver connected to the Home Assistant controller via USB. A docker container runs rtl_433 to parse the data and send messages to an MQTT broker. Home Assistant picks up the MQTT message and sends it to InfluxDB which runs on a Synology NAS. Also on the NAS, Grafana run in Docker and displays the data from InfluxDB.](https://miro.medium.com/max/770/1*jf1ayPb9nP-OrY0tMDXY6w.png)

The layout of physical and digital components to make the weather dashboard a reality

> An extra bonus, I’ve been playing with PlantUML to generate diagrams like this one, so here is a [link to the source file that generated the above diagram](https://gist.github.com/jaxzin/8d71fa3097d7abe5820d6e13f4dddccb) which includes live links to generate the SVG and PNG file yourself.

## The Hardware

Here is the hardware I’m using, not including my home networking equipment. The only think I purchased explicitly for this project was the weather sensor, everything else I already owned.

-   **_The weather sensors_** — [WS2032 Weather Station](https://www.google.com/search?q=WS2032+Anemometer+Weather+Station)  
    _$40–85 USD, depending on the site._ I picked this because I found the [rtl\_433](https://github.com/merbanan/rtl_433) project and this unit was listed as supported.
-   **_A radio receiver_** — [RTL-SDR USB radio receiver](https://a.co/d/eFO6xWE)  
    _$30 USD on Amazon._ Used to receive the 433 Mhz radio signal from the weather station. I already had this laying around from an old project.
-   **_Something to run Home Assistant_** _—_ [Home Assistant Blue](https://www.home-assistant.io/blue/)  
    _No longer for sale, but consider the_ [_Home Assistant Yellow_](https://www.crowdsupply.com/nabu-casa/home-assistant-yellow) _or a_ [_Raspberry Pi_](https://www.raspberrypi.com/products/)_._
-   **_Something to run Grafana and InfluxDB_** _—_ [Synology NAS DSM 918+](https://www.google.com/search?q=synology+%22918%22)  
    Newer models exist, and the Docker containers could run on the same hardware as your Home Assistant. I ran these two on my NAS because it has more storage space.

## Setting up the Weather Station

![](https://miro.medium.com/max/704/1*BzJX0ldpzm-RtlqcEJO9KQ.gif)

The WS2032 in action, after being attached to a light post on my patio.

This step might have been the easiest. The WS2032’s assembly was pretty straightforward with two AA batteries and a few screws. For the price, it seems pretty well made with what appear to be Hall-effect sensors used to detect the position and rotational speed of the vane and anemometer components. The temperature and humidity sensors are built into the removable central core that holds the batteries and that assembly is shaded from direct sunlight and cooled by an open louvered design of layered fins. They also keeps the central core from getting wet while still being open to get ambient readings. The hardest part of this setup was align it to the cardinal direction listed near the vane on top. But after seeing the readings coming from the device, it’s precision is limited to 45º so it’s alignment to the compass is pretty forgiving. The WS2032 comes with an LCD display unit for indoors, and with it I was able to confirm the sensors were functioning and roughly providing accurate data.

## Setting up Home Assistant

So I won’t walkthrough a full setup tutorial on Home Assistant (HA). There are a lot of great resources out there to help you get started already. What this section will cover are the changes I needed to make to my already running instance of Home Assistant.

## But first, a little background on RTL-SDR

Now that the weather station was mounted and sending data, it was time to setup something to receive that data. `rtl_433` is an open-source project created for parsing signals from various sensors around your home sending data on radio frequencies (RF), the most common one being 433 MHz though other frequencies are supported.

Devices that uses RF — instead of Bluetooth, WiFi, ZigBee, Z-Wave, or Thread — are often cheaper or older and not originally intended to be integrated into a modern smart home. They may include gas or water meters, leak detectors, even the tire pressure sensors in your car can be detected and parsed with `rtl_433`. `rtl_433` is written to use many commonly available USB radio receivers called software-defined radios (SDR).

With the invention of digital TV, cheap chipsets sprung up to decode the HDTV signals and one of them, the RTL2832U from RealTek (commonly shortened to RTL), was found to be hackable to serve a wide-range of radio frequencies so amateur radio hackers could digitally listen in to things like police and airport bands from their computer…and so [the RTL-SDR movement](https://www.rtl-sdr.com/) was born.

That brings us back to `rtl_433`, a project to listen into RF signals and parse the data out of them, including the WS2032 weather station.

## **Setting up rtl\_433**

If you already have Home Assistant running — specifically a Home Assistant OS deployment that lets you run “add-ons” which are just another name for Docker containers in this ecosystem — the easiest way to get RF devices into Home Assistant is to run it as the following set of three add-ons.

-   [**Mosquitto**](https://github.com/home-assistant/addons/blob/master/mosquitto/DOCS.md) — the simplest setup will use the official add-on for this popular MQTT broker. If you use this add-on, the other two add-ons are already configured to use it. Otherwise you’ll need to customize the follow two add-ons to point to your existing MQTT broker. Explaining MQTT is outside the scope of this doc, but the TL;DR is its a messaging platform popular with Internet of Things (IoT) devices.
-   [**rtl\_433**](https://github.com/pbkhrv/rtl_433-hass-addons/tree/main/rtl_433) — This add-on is responsible for connecting to the RTL-SDR device over USB, detect and parse any RF data into MQTT messages that it then sends to a topic on the broker for anything that might be ready to consume that MQTT message, such as Home Assistant. This isn’t a standard add-on, so to add this addon to my instance I first needed to [add this rtl\_433 add-on repository](https://github.com/pbkhrv/rtl_433-hass-addons) from GitHub user `pbkhrv`. The README in that repository includes instructions how to installed it.
-   [**rtl\_433 Home Assistant MQTT Auto Discovery**](https://github.com/pbkhrv/rtl_433-hass-addons/tree/main/rtl_433_mqtt_autodiscovery) — By default if you’ve turned on MQTT integration with Home Assistant, it doesn’t automatically begin reading every MQTT message and turn it into a smart home device you can control. But Home Assistant does support the idea of special MQTT messages, meant only for Home Assistant by smart devices, to automatically add those devices to Home Assistant for control. This add-on is responsible for watch for the MQTT messages coming from the rtl\_433 addon and to then send the automatic discovery messages to Home Assistant over MQTT as well. This add-on comes as part of the overall rtl\_433 add-on repository I configured earlier. The only step remaining was to install the addon using this repository. Confused? Adding the addon repository to HA is like adding a new index of addons you could install. You’ll still need to install both add-ons now that they are available to Home Assistant.

One final step here, if you haven’t already, is to [enable the MQTT integration](https://www.home-assistant.io/integrations/mqtt/) with Home Assistant and point it at the Mosquitto addon. The Mosquitto addon’s README covers this setup, but I wanted to call it out explicitly.

At this point you should have data flowing into Home Assistant. Go into the Developer Tools panel, and search for `ws2032` in the States tab. You should see something like this (except the names will be verbose and the “\_mph” entity, I customized those myself).

![](https://miro.medium.com/max/770/1*DDrRcg3veyzvsB6H7DM75w.png)

## Customizing the Entities

So as you can see in the above example, the battery reading is probably the only one that closely matches anything you might be seeing as it is auto-discovered by the `rtl_433` containers. To make things look pretty when I add the entities to HA dashboards, I went to each entity and customized its Friendly Name and Icon. This allows me to quickly add the entities to cards with little fuss.

This is what a typical card on my HA dashboard looks like with these entities all prettied-up.

![](https://miro.medium.com/max/770/1*GyCJbR8__5wT-Fub5OLqfg.png)

Customizing the name and icon of the WS2032 entities makes them quicker to add to cards

Given that the wind and gust speed readings coming from the WS2032 were in metric as km/h and I’m American, I wanted to convert them to mph for display. So I created two template entities to convert their values to imperial units. I also created a third one to convert the Wind Direction readings from degrees to cardinal compass directions.

Template sensors for converting speed to Imperial units and direction to Cardinal identifiers.

## Custom HA Dashboards and Cards

Before tackling InfluxDB and Grafana, I setup Home Assistant to use this data it was now collecting. Here are some examples:

Well, now I was hooked. And I wanted to…

1.  Slice the data in ways beyond Home Assistant’s Lovelace frontend was capable.
2.  Keep the data forever for analyzing some long-term trends.

So I realized it was time to setup InfluxDB to record all this data for as long I can store it, and Grafana to graph it all.

## Setting up InfluxDB and Grafana

If you’re looking for the quickest start, you can [install InfluxDB as a Home Assistant add-on](https://github.com/hassio-addons/addon-influxdb) and [Grafana as a Home Assistant add-on](https://community.home-assistant.io/t/home-assistant-community-add-on-grafana/54674). I wanted to run both on my Synology NAS and as of this writing I’m running InfluxDB 2.5.1 and Grafana 9.2.6 in Docker containers from the official Docker images available on Docker Hub.

After I got InfluxDB running, I created an organization to represent my home, and a bucket to hold it’s data from Home Assistant. I also created two API tokens, one for Home Assistant to use and another that Grafana will use.

## Connect Home Assistant to InfluxDB

There is native support in Home Assistant to pump all it’s events into InfluxDB. So not only will I be able to create a weather dashboard, there is so much data I’ll be able to access from Grafana, this is just the start. To enable this event flow from HA to InfluxDB, [add a bit of YAML to your Home Assistant’s](https://www.home-assistant.io/integrations/influxdb/#configuration) `[configuration.yaml](https://www.home-assistant.io/integrations/influxdb/#configuration)` [file](https://www.home-assistant.io/integrations/influxdb/#configuration).

```
influxdb:  api_version: 2  ssl: false  host: nas.iot.jaxzin.com  port: 8086  token: !secret influxdb_token  organization: 8d2a3be3fdc94fa2  bucket: fallen-leaf  tags:    source: HA  tags_attributes:    - friendly_name  default_measurement: units
```

## Add InfluxDB to Grafana as a data source

I wanted to use the Flux query language, which I could see from my initial research is a powerful functional language. We’ll dive into Flux in the next section.

## Installing the Wind Rose plugin

The one visualization type, that I really wanted but wasn’t included in the base Grafana installation is a [wind rose](https://en.wikipedia.org/wiki/Wind_rose). There are a few wind rose plugins available for Grafana, but I landed on using [this windrose plugin from spectraphilic](https://github.com/spectraphilic/grafana-windrose). Installation directions are in the README and the [Grafana Docker instructions explains how to add custom plugins](https://grafana.com/docs/grafana/latest/setup-grafana/installation/docker/#install-plugins-in-the-docker-container).

![](https://miro.medium.com/max/770/1*rFR1h6IyTAshIz6onhvgsA.png)

An example of the wind rose plugin from spectraphilic.

## Crafting the Dashboard

Now for the fun and mildly overwhelming part, creating a custom Grafana dashboard which means [learning Flux](https://docs.influxdata.com/flux/v0.x/get-started/) and doing lots of Googling to figure out how to achieve the look I was going for.

## The Wind Rose

Thankfully spectraphilic included a Flux example in the README for that Grafana plugin, so I started out pretty fast and strong with the Flux query for the Wind Rose. This is an example of joining the wind direction `..._wd` and wind speed `..._ws` data from the WS2032.

```
from(bucket: "fallen-leaf")  |> range(start: v.timeRangeStart, stop: v.timeRangeStop)  |> filter(fn: (r) => r["entity_id"] == "ws2032_28817_wd" or r["entity_id"] == "ws2032_28817_ws")  |> filter(fn: (r) => r["_field"] == "value")  |> aggregateWindow(every: ${period}, fn: mean, createEmpty: false)  |> group()  |> pivot(rowKey:["_time"], columnKey: ["entity_id"], valueColumn: "_value")   |> filter(fn: (r) => exists r["ws2032_28817_wd"] and exists r["ws2032_28817_ws"])   |> rename(columns: {"ws2032_28817_wd": "direction", "ws2032_28817_ws": "speedKmps"})  |> map(fn: (r) => ({ r with  speed: r.speedKmps * 0.621371 }))   |> keep(columns: ["_time", "speed", "direction"])  |> aggregateWindow(every: ${period}, fn: first, column: "direction", createEmpty: false) 
```

The end result is a table of data with three columns: time, direction and speed. The wind rose visualization is responsible for taking that data and bucketing the speeds by direction. Given the resolution of the WS2032 is only 8 directions, I configured the rose to use only 8 “cake slices”.

![](https://miro.medium.com/max/770/1*JHuBj4zqINtRh02v0PWodw.png)

Inspecting the panel shows the raw table of data returned by the Flux query.

## Adding a dashboard variable for the stats period

Now before moving on to the other visualizations, I knew that I wanted more flexibility than just selecting different ranges of data. Wind data is really noisy and so I also wanted to easily change the sampling period within a selected time range. This sampling period would be used for calculating stats like mean, max, min, or percentiles. So I could see the wind speed or direction broken down hourly or daily. To accomplish this I needed to add a custom variable to the dashboard for the Window Period. You’ll see it referenced in later Flux queries as `${period}`.

## Wind Speed as a Heatmap

For a while I was playing with different visualizations of the wind speed history. I wanted to captured the gust and wind speed readings on one chart and landed on this visualization of both, using min, mean, and max lines to help demonstrate the variance in speeds.

![](https://miro.medium.com/max/770/1*36e8W_v8XdQNhhpBOHmh4w.png)

Wind & Gust Speed on one graph

To get this effect, you basically need 5 sets of data. And Grafana with Flux queries makes that as simple (but maybe not efficient) as running 5 queries.

```
from(bucket: "fallen-leaf")  |> range(start: v.timeRangeStart, stop: v.timeRangeStop)  |> filter(fn: (r) => r["entity_id"] == "ws2032_28817_ws_mph")  |> filter(fn: (r) => r["_measurement"] == "mph")  |> filter(fn: (r) => r["_field"] == "value")  |> keep(columns: ["_time","_value", "value", "_field"])  |> set(key:"_field", value:"max")  |> aggregateWindow(every: ${period}, fn: max, createEmpty: true)  |> yield(name: "max")from(bucket: "fallen-leaf")  |> range(start: v.timeRangeStart, stop: v.timeRangeStop)  |> filter(fn: (r) => r["entity_id"] == "ws2032_28817_ws_mph")  |> filter(fn: (r) => r["_measurement"] == "mph")  |> filter(fn: (r) => r["_field"] == "value")  |> keep(columns: ["_time","_value", "value", "_field"])  |> set(key:"_field", value:"mean")  |> aggregateWindow(every: ${period}, fn: mean, createEmpty: true)  |> yield(name: "mean")from(bucket: "fallen-leaf")  |> range(start: v.timeRangeStart, stop: v.timeRangeStop)  |> filter(fn: (r) => r["entity_id"] == "ws2032_28817_ws_mph")  |> filter(fn: (r) => r["_measurement"] == "mph")  |> filter(fn: (r) => r["_field"] == "value")  |> keep(columns: ["_time","_value", "value", "_field"])  |> set(key:"_field", value:"min")  |> aggregateWindow(every: ${period}, fn: min, createEmpty: true)  |> yield(name: "min")from(bucket: "fallen-leaf")  |> range(start: v.timeRangeStart, stop: v.timeRangeStop)  |> filter(fn: (r) => r["entity_id"] == "ws2032_28817_gs_mph")  |> filter(fn: (r) => r["_measurement"] == "mph")  |> filter(fn: (r) => r["_field"] == "value")  |> keep(columns: ["_time","_value", "value", "_field"])  |> set(key:"_field", value:"max (gust)")  |> aggregateWindow(every: ${period}, fn: max, createEmpty: true)  |> yield(name: "max_g")from(bucket: "fallen-leaf")  |> range(start: v.timeRangeStart, stop: v.timeRangeStop)  |> filter(fn: (r) => r["entity_id"] == "ws2032_28817_gs_mph")  |> filter(fn: (r) => r["_measurement"] == "mph")  |> filter(fn: (r) => r["_field"] == "value")  |> keep(columns: ["_time","_value", "value", "_field"])  |> set(key:"_field", value:"mean (gust)")  |> aggregateWindow(every: ${period}, fn: mean, createEmpty: true)  |> yield(name: "mean_g")
```

But it still wasn’t sitting right with me. I didn’t feel like I was visualizing the noisy data. So I tried the heatmap visualization, and I think I landed on something that shows the variance in wind speed while also making it easy to pick out where the bulk of the wind speeds are centered.

![](https://miro.medium.com/max/770/1*DZ-c5ex7VcbyP6EHw65PLw.png)

Wind Speed as a heatmap

```
from(bucket: "fallen-leaf")  |> range(start: v.timeRangeStart, stop: v.timeRangeStop)  |> filter(fn: (r) => r["entity_id"] == "ws2032_28817_ws")  |> filter(fn: (r) => r["_measurement"] == "km/h")  |> filter(fn: (r) => r["_field"] == "value")  |> map(fn: (r) => ({r with _value: r._value * 0.618}))  |> keep(columns: ["_time","_value", "value", "_field"])
```

I lost the ability to show wind and gust speed on one chart, so I’ve kept them both around. At least until I find a better solution.

## Wind Direction as a pseudo-topology chart, using percentiles

The wind direction data is really noisy, and so I wanted to use the power of statistics to try and create a visualization that helped find the signal in the noise. So I had the idea of “what if I drew each percentile as it’s own line? Would it show the direction of the data while also giving a better sense of the variance?” (Note, this was before I played with the heatmap above, which might also be a good solution here).

![](https://miro.medium.com/max/770/1*rXxXLzSNP1V4QPZewp9PWw.png)

Charting percentiles to create a flowing river of wind direction

But I didn’t want to type out 21 different queries, one for each percentile. Could I create a dynamic Flux query that calculated each percentile? For this query I had to turn to the Flux forums for some help and landed on this final query.

```
import "experimental/array"data = () => from(bucket: "fallen-leaf")  |> range(start: v.timeRangeStart, stop: v.timeRangeStop)  |> filter(fn: (r) => r["entity_id"] == "ws2032_28817_wd")  |> filter(fn: (r) => r["_field"] == "value")  |> filter(fn: (r) => r["_measurement"] == "°")  |> map(fn: (r) => ({r with _value: r._value * 400.0/360.0}))   |> aggregateWindow(     every: duration(v: int(v: ${period}) / 12),    fn: mean  )p = [1,5,10,15,20,25,30,35,40,45,50,55,60,65,70,75,80,85,90,95,99]q = (v) => {  d = data ()    |> aggregateWindow(        column: "_value",        every: ${period},        fn: (column, tables=<-) => tables |> quantile(q: float(v: v)/100.0, column: column),    )    |> set(key: "_field", value: "p${if v < 10 then "0" else ""}${v}")  return d}union(tables:  p   |> array.map(fn: (x) => (  q(v: x))))  |> keep(columns: ["_time", "_field", "_value"])
```

Wow, functional programming languages can be so powerful! I’m loving this syntax and how I was able to pull 21 datasets with a few extra lines of code. Big shout out to [Jay Clifford at influxData for the help](https://community.influxdata.com/t/trying-to-build-a-set-of-percentiles-or-quantiles-in-one-query-possible-bug-when-using-aggregatewindow-with-a-custom-function/27508) crafting this query.

## Customizing the colors and style

Now I had the data, but by default the Grafana Time Series visualization displays them as a rainbow of lines. I wanted to highlight the median (aka `p50`) as a brighter solid line with the rest of the percentiles displayed as dashed lines. I also wanted to use some background fill to highlight what was above the median versus what was below the median.

The answer to these customizations are Grafana overrides. To fill the background between two lines, on the higher series of data select a new property override of “Fill below to” and select the lower series of data. Now the visualization will include a dynamic background fill between the two series.

I also added different colors to the lines using a regex override so lines with a field name matching `p[0-4]\d` (aka `p01`\-`p49`) would be colored purple and lines matching `p[5-9]\d|p5[1-9]` (aka `p51`\-`p99`) would be colored green.

To highlight the median, I set the lines to default to dashed and then added an override for the `p50` line to customize it’s line style, color, and thickness.

## Thinking in compass directions instead of degrees

So at this point, I was close to done, but the readings and y-axis were still in degrees. But when it comes to wind direction I think in the cardinal (N,E,S,W) and ordinal (NE,SE,SW,NW) directions and I wanted the visualization to reflect that.

## Value Mappings

The time series visualization has the ability to map numeric values to other values, including simple text like the cardinal and ordinal directions. It can map specific values, or it can use more complex definitions of the mappings using ranges or regular expressions. This mapping will affect the value shown in the tooltip when you hover over data points, as well as the values displayed on the y-axis. So I went ahead and mapped the degree ranges to compass directions.

## Thresholds

The other thing I notices is it was hard to visually tell with an x/y graph which values were near a cardinal direction like N,S,E or W. So I wanted to added an indicator to the graph in the form of horizontal dashed lines at each of these four directions. Adding this type of horizontal indicator is called a threshold and it more powerful than just adding horizontal lines. It can color the lines based on their value vs the thresholds, and it can fill the background as well. So I added thresholds for the cardinal directions to display as lines and background fill.

## Fixing the labels on the y-axis by transforming the data

_(…to fit Grafana’s expectations)_

So everything was looking good, except when I added a Max value of 360º the y-axis didn’t clearly label the thresholds of N, E, S, and W. After a bunch of forum searching I was losing hope that it would be possible to get the axis to label these non-round numbers of 90º, 180º, 270º and 360º. It seems to insist on labeling 100º, 200º, 300º, etc. So inspiration struck. I was already mapping the values as displayed, could I adjust the original angle values to fit Grafana’s expectations and fix the issue via a bit of a hack?  
So that’s why this line exists.

```
...|> map(fn: (r) => ({r with _value: r._value * 400.0/360.0}))...
```

By re-mapping N, E, S, W to 0, 100, 200, 300 and 400 (N again), I was able to get the y-axis to label the cardinal thresholds and the fact that the data is no longer 0º to 360º doesn’t really matter for the display. But if you inspect the raw results, it may look odd to see values above 360º.