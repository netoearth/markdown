The most commonly used command line [tools for process monitoring on Linux](https://itsfoss.com/linux-system-monitoring-tools/) are top and its colorful, feature rich cousin [htop](https://hisham.hm/htop/) .

To [monitor temperature on Linux](https://itsfoss.com/monitor-cpu-gpu-temp-linux/), you can use [lm-sensors](https://github.com/lm-sensors/lm-sensors). Similarly, there are many utilities to monitor other real-time metrics such as Disk I/O, Network Stats and others.

[Glances](https://nicolargo.github.io/glances/) is a system monitoring tool which ties them all together and offers a lot more features. What I like the most is that you can use run Glances on a remote Linux server and monitor the system resources on your local system or monitor it through your web browser.

Here’s what it looks like. The terminal in the below screenshot has been [beautified with Pywal tool that automatically changes the color based on wallpaper](https://itsfoss.com/pywal/).

![Glances Linux](https://itsfoss.com/wp-content/uploads/2020/08/glances-linux-800x510.png)

You can also integrate it to a tool like [Grafana](https://grafana.com/) to monitor the stats in an intuitive dashboard.

It is written in Python which means that it can be used on most platforms with almost all features.

## Features of Glances

![Glances Data In Grafana Dashboard](https://itsfoss.com/wp-content/uploads/2020/08/glances-data-in-grafana-dashboard.jpg)

Glances Data In Grafana Dashboard

Let’s have a quick glance over the main feature Glances provides (pun intended).

-   Can monitor 15+ metrics on a system (including docker containers).
-   Flexible usage modes: standalone, client-server, over SSH and web mode.
-   Versatile REST API and XML-RPC available for integrations.
-   Readily supports exporting data to different services and databases.
-   Highly configurable and adaptable to different needs.
-   Very comprehensive Documentation.

## Installing Glances on Ubuntu and other Linux distributions

Glances is available in the official repositories of many Linux distributions. This means you can use your distribution’s package manager to install it easily.

On Debian/Ubuntu based distributions, you can use the following command:

```
sudo apt install glances
```

You can also install the latest Glances using snap package:

```
sudo snap install glances 
```

Since Glances is based on Python, you can also use PIP to install it on most Linux distributions. [Install PIP](https://itsfoss.com/install-pip-ubuntu/) first and then use it to install Glances:

```
sudo pip3 install glances
```

If nothing else, you can always use the auto install script provided by Glances developer. Though we at It’s FOSS don’t recommend running random scripts directly on your system. It’s entirely your decision.

```
curl -L https://bit.ly/glances | /bin/bash
```

You can check out other ways to install Glances from their [documentation](https://github.com/nicolargo/glances/blob/master/README.rst#installation), you can also install it as a docker container.

## Using Glances to monitor Linux system resources on local system (standalone mode)

You can easily launch Glances to monitor your local machine by running this command n the terminal.

```
glances
```

You can instantly observe that it integrates a lot of different information in one single screen. I like that it shows the public and private IP’s of the computer right at the top.

![Screenshot From 2020 08 13 11 54 18](https://itsfoss.com/wp-content/uploads/2020/08/Screenshot-from-2020-08-13-11-54-18-800x503.png)

Glances is also interactive, meaning you can use commands to interact with it while it is running. You can press **“s”** to bring up the sensors onto the screen; **“k”** to bring up the TCP connections list; **“1”** to expand the CPU stats to show individual threads.

You can also use the arrow keys to move around the list of processes and sort the table by different metrics.

You can launch Glances with various command line options. There are plenty more interactive commands as well. You can find the complete list in their [comprehensive documentation](https://glances.readthedocs.io/en/latest/cmds.html).

Press Ctrl+C to exit Glances.

## Using Glances to monitor remote Linux systems (Client – Server mode)

To monitor a remote computer you can use Glances in client-server mode. You need to have Glances installed on both systems.

On the remote Linux system, you’ll have to launch glances in server mode using the -s option.

```
glances -s
```

On the client system, use this command to launch Glances in client mode and connect to the server.

```
glances -c server_ip_address
```

You can also SSH into any computer and launch Glances, which works flawlessly. More information about client-server mode [here](https://glances.readthedocs.io/en/latest/quickstart.html#central-client).

## Monitor Linux system resources in web browser using Glances (Web Mode)

Glances can also run in web mode. This means that you can use a web browser to access Glances. Unlike the previous client-server mode, you don’t need to install Glances on the client system.

To start Glances in web mode, use the -w option.

```
glances -w
```

Keep in mind that it may display “Glances Web User Interface started on http://0.0.0.0:61208” even on a Linux server when it is actually using the server’s IP address.

The main part is that it uses 61208 port number and you can use it to access Glances via web browser. Just type the server’s IP address followed by the port number like http://123.123.123.123:61208.

You can also use it on the local system using http://0.0.0.0:61208/ or [https://localhost:61208/](https://localhost:61208/).

![Glances in web mode](https://itsfoss.com/wp-content/uploads/2020/08/Screenshot-from-2020-08-13-16-49-11-800x471.png)

The web mode also emulates the terminal look. The web version is built with responsive design principles in mind and looks good even on phones.

You may want to protect the web mode with a password so that only authorized people could use it. The default username is glances.

```
root@localhost:~# glances -w --password
Define the Glances webserver password (glances username): 
Password (confirm): 
Do you want to save the password? [Yes/No]: n
Glances Web User Interface started on http://0.0.0.0:61208/
```

You can find more information on configuring password in the [quickstart guide](https://glances.readthedocs.io/en/stable/quickstart.html).

## Export Glances data to different services

One of the biggest advantage of using Glances is the out of the box support to export data to various databases, services and integration into various data pipelines seamlessly.

You can export to CSV while monitoring with this command.

```
glances --export csv --export-csv-file /tmp/glances.csv
```

‘/tmp/glances.csv’ is the location of the file. The data is filled in neatly as time series.

![export Glances data](https://itsfoss.com/wp-content/uploads/2020/08/Screenshot-from-2020-08-13-12-25-40-800x448.png)

You can export to powerful applications like [Prometheus](https://prometheus.io/) to enable conditional triggers and notifications.

It can directly plug into messaging services like RabbitMQ, MQTT, streaming platforms like Kafka and export time series data to databases like InfluxDB and visualize using Grafana.

You can check out the whole list of services and export options [here](https://glances.readthedocs.io/en/latest/gw/index.html).

## Use REST API to integrate Glances with other services

This is my favorite feature in the whole stack. Glances not only brings together various metrics together, it also exposes them via APIs.

This simple yet powerful feature makes it very easy to build custom applications, services and middleware applications for any specific use cases.

REST API server stars automatically when you launch Glances in web mode. To launch it in API server only mode you can use the following command.

```
glances -w --disable-webui
```

The [REST API](https://github.com/nicolargo/glances/wiki/The-Glances-RESTFULL-JSON-API) documentation is comprehensive and the responses are easy to integrate with a web application. This makes it easy to build a unified dashboard to monitor multiple servers with a tool like [Node-RED](https://nodered.org/).

![Glances running with NODE-RED using REST API](https://itsfoss.com/wp-content/uploads/2020/08/Screenshot-from-2020-08-13-17-49-41-800x468.png)

Glances also provides an XML-RPC server, you can check out the documentation [here](https://github.com/nicolargo/glances/wiki).

## Closing thoughts on Glances

Glances uses the [psutil](https://pypi.org/project/psutil/) python library to access different system stats. I had built a simple API server using the same library to retrieve CPU usage back in 2017. I was able to monitor all the Raspberry Pi’s in a cluster using a dashboard built with Node-RED.

Glances could have saved me some time while offering way more features, unfortunately I didn’t know about it back then.

I did try to install Glances on my Raspberry Pi’s while writing this article, unfortunately it failed with some errors with all the installation methods. I’ll update the article when I succeed or maybe write another article with steps to install on a Raspberry Pi.

I wish Glances offered a way to kill processes like top or htop. Let’s hope that we get it in upcoming releases.

I hope this gave you a good deal of information on Glances. What system monitoring tools do you guys use? let me know in the comments.

Skip![arrow](https://sdk.apester.com/assets/iconRightArrow.svg)

Skip

Skip ad in

![arrow](https://sdk.apester.com/assets/iconRightArrow.svg)

![](https://itsfoss.com/wp-content/gravatars/2ec07c731e03d4fe42624d770a41cbe0)

I'm a Maker, love to tinker with electronics. PCMR fanatic and I occasionally make YouTube videos about tech.  
Philosophy and Psychology are my interests on the other side of the isle.