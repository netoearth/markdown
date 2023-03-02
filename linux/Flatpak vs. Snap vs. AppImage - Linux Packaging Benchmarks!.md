hat we’re going to be doing is checking out some of the more popular Linux packaging formats, and running some benchmarks to see if there are any clear winners between them. But before we get into the actual benchmarking, we must first talk about what these different packaging formats are, and what are some of the key differences between them.

We’re going to be focusing on [flatpaks](https://flatpak.org/), [snaps](https://snapcraft.io/), and [appimages](https://appimage.org/). All of these have the benefit of being independent from your Linux distribution, meaning that no matter what version or distribution of Linux we’re running, you can run all of these packaging formats. And this differs from something like APT, pacman, DNF, whatever it may be, because those are distributions specific package management systems.

So now to cover some of the differences. First up, I’m going to talk about flatpaks. This is a decentralized system, where you can pull applications from a variety of sources with [flathub](https://flathub.org/home) being the most popular at the moment, the actual flatpak applications run in a sandboxed environment. So it will be completely separate from your systems runtime. And these sandbox applications include everything that you’re going to need to run the application including all of the dependencies. And generally, this also makes flatpaks secure, because it won’t actually interfere with your system at all unless if you give it explicit permission to do so.

A negative of flatpaks (and other sandboxed packages) compared to your distributions native package management system is the file sizes are going to generally be larger because of all those preincluded dependencies.

## Snaps

Now this is definitely the most controversial of the bunch. Snap packages are a packaging system that is developed by [Canonical](https://canonical.com/) the parent company of Ubuntu. One of the big issues that a lot of people have with the snap packages is the snap store itself is proprietary and controlled by Canonical. Recently, they’ve started replacing some of the packages on the Ubuntu system with snaps by default. And another “feature” of snaps is their auto updating nature now like flatpaks snaps are also going to contain all of the dependencies needed to run the actual application. And generally, it’s going to be isolated from the rest of your system.

## AppImages

Now of these Appimages are definitely the most interesting and unique as they are a individual executable file that will contain everything you need to run an entire application. So there’s simply just one file per appimage. And with this that makes appimages incredibly portable. And with that very easy to manage various versions of one application on your system. For those of you who are running Windows, a really good way to kind of compare this is these are very similar to portable .exe files that you may open and run on Windows.

Appimages are really easy to run. Within permissions make sure it is actually executable, and then click on the file to open it up or you can launch it through the terminal. [AppImageHub](https://www.appimagehub.com/) is a huge repository and a great resource to find various app images and applications. And many developers who opt to actually provide this format for their applications usually have direct downloads on their websites.

Now the main issue with the app image format is these applications are not just directly installed on your system. So this will require more manual intervention to create desktop entries that will show up in your menus. And of course, updating them, you’re just gonna have to download the new version, and replace it depending on how you have your system set up.

<iframe loading="lazy" src="https://www.youtube.com/embed/OftD86RgAcc?feature=oembed" allowfullscreen="allowfullscreen" width="200" height="113" frameborder="0"></iframe>

_This video is sponsored by SurfShark. If you go to_ [_SurfShark.deals/TECHHUT_](https://surfshark.deals/TECHHUT) _you can get 83% off your subscription and a extra three months for free!_

## Browser Benchmarks

So I was browsing around on our r/Linux on Reddit and I stumbled across [this posting](https://www.reddit.com/r/linux/comments/ttpix9/snap_firefox_vs_official_firefox_from_their_site/) showing a substantial difference in internet speeds between the actual snap packages and a direct Firefox download. So that got me thinking what else can we do to see the performance differences of the same application but on different packaging formats?

![](https://miro.medium.com/max/700/1*bV71Exwd-HlBDcUJNMyT3g.png)

So first, let’s take a closer look at Firefox. Now. I wasn’t able to do this same internet speed test that I saw on Reddit, mostly because there is wild variations in my internet speed. But I was able to run a suite of browser benchmarks, and the very first one we ran was BaseMark Web 3.0.

## Basemark Web 3.0

This benchmark includes various system and graphics tests that use web recommended features and gives us a comprehensive performance benchmark as a score so we can see the differences. Now for all of these tests, I did it on both Ubuntu and Fedora using in addition to these various packaging formats, the APT and DNF installations.

![](https://miro.medium.com/max/700/1*Zq-V1O_rAl_gZTXIb5qwrg.png)

We can see some substantial differences between the packaging formats, and in between Fedora and Ubuntu. So it’s kind of a distribution comparison to if you’re interested in that. And for this, this was done on Firefox version 99, across all platforms and distributions. We can see app images and flatpaks are neck and neck and appimages performing slightly better on Fedora and flatpaks performing slightly better on Ubuntu.

## MotionMark

And then from there, we went ahead and ran a MotionMark which this specifically focuses on graphic rendering. Again, this is scores. So the higher the better. And overall, we saw very similar results.

![](https://miro.medium.com/max/700/1*Td6obo0pajUStt0Ta7ccUQ.png)

In this case, Ubuntu actually had an edge in these scores with appimages generally performing better than flatpaks. And then the native package installation through APT or DNF, was slightly behind the snap package performance.

## Speedometer

Next, we went ahead and ran Speedometer, which just as a very general test for browser responsiveness, this one was a lot tighter again, we’re seeing a lot of the similar results across these browser benchmarks, which generally I think is really good that these are consistent between various tests, basically telling me that these results are fairly accurate.

![](https://miro.medium.com/max/700/1*XIAcfANX0ztmhwYac_oy9Q.png)

Again, appimages and flatpaks are neck to neck. And with this variation of 2% it’s basically a tie between appimages and flatpaks. And with general browser responsiveness, the DNF installation on Fedora did perform substantially better than the snap package. And then with Ubuntu, the snap had a very slight advantage in comparison to the APT installation.

## Rendering Benchmarks

Now going forward from Firefox, there aren’t too many benchmarking tools that we could use to actually measure this. So I had to kind of come up with some of my own benchmarking techniques for some of these.

## Kdenlive Render

First I ran a simple Kdenlive render, what I did was added about an eight minute video added one effect to the entire thing. So it did have to do a little bit of extra work with the rendering.

![](https://miro.medium.com/max/700/1*pQZUb4E-5RCbNmxc0CjZPQ.png)

Now this is going to be kind of flipped from what we saw with Firefox. In this case, this is in time (seconds) so the lower the score, the better. APT and DNF did substantially better than all of the cross platform package systems. And this was by a wide margin. If you’re going to be rendering things in Kdenlive, by the looks of this anyways, you should probably just install it with your system package management software. Appimage was behind by about a 30m second difference, but then flatpaks and snaps, lagged behind everything by 20–50 seconds.

## GIMP Lava Render

For the next step we ran one of my favorite little benchmarks, which is the GIMP at Lava render. I opened up GIMP created a 5000 by 5000 canvas, rendered out the lava texture, which is a slightly intensive process. Now I will note the appimage version of gimp isn’t kept as up to date as all the other versions as it’s behind about six months.

![](https://miro.medium.com/max/700/1*lTDMVXtIA0Yc1Rpvb4TgYA.png)

Even with this appimage did perform the best and actually did beat out the native installation by just about eight seconds. Both flat packs and snap packages fell behind the native installation by a considerable margin. So just considering this graph, personally, I’m going to be switching my GIMP installation to simply using an app image.

## Blender Image Render

Now there is actually a b[lender benchmarking utility](https://www.blender.org/news/introducing-blender-benchmark/). But unfortunately, there’s not a flatpak or any other sandboxed package. So I had to download some project files directly from Blender and then render those out as a single image. My computer’s not very good. So rendering out the entire animation would have taken hours.

![](https://miro.medium.com/max/700/1*flIIFAXOLHMP3YCsAhESlA.png)

As we can see here rendering out a single image took over 400 seconds no matter what I ran. For some reason the native installation on both Ubuntu and Fedora was just not cooperating with the files that I downloaded. So we only ran this benchmark with flatpak and the snap package. This is one of the very few cases that snap package is performing slightly better than the flatpak.

This is arguable, but part of the reason for this might be that our snap package are one of the download options directly from Blender. So it’s one of their main supported formats.

## Thoughts

Running something where you’re rendering out files doing creative work, it’s probably better to use the native application from your distributions package manager. But for everything else, flatpaks, appimages, might be better. Snaps, in general, kind of suck.

But this is just speaking in generals, every application can have major differences. If you are looking for very specific benchmarks, these were incredibly easy to run, you could run them for yourself. Just get your latest project for whatever application you want to know runs a better time the render process or whatever it may be, and figure out what works best for you.

![](https://miro.medium.com/max/700/1*SPhudUlQYjFtQJPLSakSdg.jpeg)