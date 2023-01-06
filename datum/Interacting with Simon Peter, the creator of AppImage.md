As a Linux user, you might have come across [AppImages](https://appimage.org/). This is a portable packaging format that allows you to run an application on any Linux distribution.

[Using AppImage](https://itsfoss.com/use-appimage-linux/) is really simple. You just need to give it execute permission and double click to run it, like the .exe files in Windows. This solves a major problem in Linux as different kind of distributions have different kind of packaging formats. You cannot [install .deb files](https://itsfoss.com/install-deb-files-ubuntu/) (of Debian/Ubuntu) on [Fedora](https://getfedora.org/) and vice versa.

We talked to Simon, the developer of AppImage, about how and why he created this project. Read some of the interesting background story and insights Simon shares about AppImage.

![AppImage Simon Interview](https://itsfoss.com/wp-content/uploads/2020/02/appimage_simon_interview.jpg)

**_It’s FOSS: Few people know about the person behind AppImage. How about sharing a little background information about yourself?_**

**Simon:** Hi, I’m Simon Peter, based near Frankfurt in Germany. My background is in Economics and Business Administration, but I’ve always been a tinkerer and hacker in my free time, and been working in tech ever since I graduated.

AppImage, though, is strictly a hobby which I enjoy working on in my spare time. I do a lot of my AppImage work while I’m on a train going from here to there. Somehow I seem to be on the move all the time. Professionally, I work in the product management of a large telecommunications company.

**_It’s FOSS: Why did you create AppImage?_**

**Simon:** The first computer I could get my hands on was a [Macintosh](https://en.wikipedia.org/wiki/Macintosh) in the late 80s. For me, this is the benchmark when it comes to simplicity and usability. When I started to experiment with Linux on the desktop, I always wished it was as elegant and simple to operate and gave me as much flexibility as the early Macs.

When I tried Linux for the first time in the late 90s, I had to go through a cumbersome process formatting and partitioning hard disks, installing stuff – it took a lot of time and was really cumbersome. A couple of years later, I tried out a Linux Live CD-ROM. It was a complete game changer. You popped in the CD, booted the computer, and everything just worked, right out of the box. No installation, no configuration. The system was always in factory-new state whenever you rebooted the machine. Exactly how I liked it.

There was only one downside: You could not install additional applications on a read-only CD. Packages always insisted on writing in /usr, where the Live CD was not writeable. Thus, I asked myself: Why can’t I just put applications wherever I want, like on a USB drive or a network share, as I am used from the Mac? How cool would it be if every application was just one single file that I could put wherever I want? And thus the idea for AppImage was born (back then under the name of “klik”).

Turns out that over time Live systems have become more capable, but I still like the simplicity and freedom that comes with the “one app = one file” idea. For example, I want to be in control of where stuff resides on my hard disks. I want to decide what to update or not to update and when. For most tasks I need a stable, rarely-changing operating system with the latest applications. To this day all I ever run are Live systems, because the operating system “just works” out of the box without any installation or configuration on my side, and every time I reboot the machine I have a “factory new”, known-good state.

**_It’s FOSS: What challenges did you face in the past and what challenges are you facing right now?_**

**Simon:** People told me that the idea was nuts, and I had no clue how “things are done on Linux”. Just about when I was beginning to give in, I came across a video of [Linus Torvalds](https://itsfoss.com/linus-torvalds-facts/) of all people who I noticed was complaining about many of the same things that I always had felt were too complicated when it came to distributing applications for Linux. While I was watching his rant, I also noticed, hey, AppImage actually solves many of those issues. Some time later, Linus came across AppImage, and he apparently liked the idea. That made me think, maybe it’s not that stupid an idea as people had made me believe all the time up to that point.

Today, people tend to mention AppImage as “one of the new package formats” together with [Snap](https://itsfoss.com/install-snap-linux/) and [Flatpak](https://flatpak.org/). I think that’s comparing apples to oranges. Not only is AppImage not “new” (it’s been around since well over a decade by now), but also it has very different objectives and design principles than the other systems. AppImage is all about single-file application bundles that can be “managed” by nothing else than a web browser and a file manager. It’s meant for “mere morals”, end users, not system administrators. It needs no package manager, it needs no root rights, it needs nothing to be installed on the system. It gives complete freedom to application developers and users.

**_It’s FOSS: AppImage is a “universal packaging system” and there you compete with Snap (backed by Ubuntu) and Flatpak (backed by Fedora). How do you plan to ‘fight’ against these big corporates?_**

**Simon:** See? That’s what I mean. AppImage plays in an entirely different playing field.

AppImage wants to be what exe files or PortableApps are for Windows and what apps inside dmg files are on the Mac – but better.

Besides, Snap (backed by [Canonical](https://canonical.com/)) does not work out-of-the-box on Fedora, and Flatpak (backed by [Red Hat](https://www.redhat.com/en)) does not work out-of-the-box on Ubuntu. AppImages can run on either system, and many more, without the need to install anything.

**_It’s FOSS: How do you see the adoption of AppImage? Are you happy with its growth?_**

**Simon:** As of early 2020, there are now around 1,000 official AppImages made by the respective application authors that are passing my compatibility tests and can run on the oldest still-supported Ubuntu LTS release, and hundreds more are being worked on as we speak. “Household name” applications like Inkscape, Kdenlive, KDevelop, LibreOffice, PrusaSlicer, Scribus, Slic3r, Ultimaker Cura (too many to name them all) are being distributed in AppImage format. This makes me very happy and I am always excited when I read about a new version being released on Twitter, and then am able to download and run the AppImage instantly, without having to wait for my Linux distribution to carry that new version, and without having to throw away the old (known-good) version just because I want to try out the new (bleeding edge) one.

The adoption of AppImage is especially strong for nightly and continuous builds. This is because the “one app = one file” concept of AppImage lends itself especially well to try-out software, where you keep multiple versions around for testing purposes, and never have to install anything into the running system. Worst thing that can happen with AppImage is that an application does not launch. In that case, file a bug, delete the file, done. Worst thing that can happen with distribution packages: complete system breakage…

**_It’s FOSS: One major issue with AppImage is that not all the developers provide an easy way of updating the AppImage versions. Any suggestions for handling it?_**

**Simon:** AppImage has this concept of “binary delta updates”. Think of it as “diff for applications”. A new version of an application comes out, you download only the parts that have changed, and apply them to the old version. As a result, you get both the old and the new version and can keep them in parallel until you have determined that you don’t need the old version any longer, and throw it away.

In general, I don’t want to enforce anything with AppImage. Application authors are at liberty to control the whole experience. Up to now, application authors have to do some setup work to make AppImages with this update capability. That being said, I am convinced that if we make it easy enough for developers to get working binary delta updates “for free”, then many will offer them. To this end, I am currently working on a new set of tools written in Go that will set up updates almost automatically, and I hope this will significantly increase the percentage of AppImages that come with this capability.

**_It’s FOSS: [Nitrux](https://itsfoss.com/nitrux-linux-overview/) is one of the rare distributions that relies heavily on AppImage. Or there any other such distributions? What can be done make AppImage more popular?_**

**Simon:** Linux distributions traditionally have thought of themselves as more than just the base operating system itself – they also wanted to control application distribution. Now, as Apple and Microsoft are trying to get more control over application distribution on their desktop platforms, the trend is slowly reversing in Linux land where people are slowly beginning to understand that distributions could be much more polished if they focused on the base operating system and left the packaging of applications to the application authors.

To make AppImage more popular, I think users and application authors should continue to spread the word that upstream-provided AppImages are in many cases working better than distribution packages. With AppImage, you get a software stack where the application author had a chance to cherry-pick which versions of libraries work together, test and tune both functionality and performance. Who is surprised that the result tends to work better than a “random” combination of whatever versions happened to be in a Linux distribution at a certain, random point in time when a distribution release was put together?

[Desktop environments](https://itsfoss.com/best-linux-desktop-environments/) could greatly increase usability, not only for AppImages, but also for any other kind of “side-loaded” applications that are not being installed. Just see how a desktop environments handles double-clicking on an executable file that is missing the executable bit. Some are doing a great job in this regard, like [Deepin Linux](https://www.deepin.org/en/). Stuff tends to “just work” there as it should.

Finally, I am currently working on a new set of tools written in Go which I hope will greatly simplify, and make yet more enjoyable, the production and consumption of AppImages. My goal here is to make things less complex for users, remove the need for configuration, make things “just work”, like on the early Macintoshes. Are there any Go developers out here interested to join the effort?

**_It’s FOSS: I can see there is a website that lists available AppImage applications. Do you have plans to integrate it with other software managers on Linux or create a software manager for AppImage?_**

**Simon:** [appimage.github.io](https://appimage.github.io/) lists AppImages that have passed my compatibility tests on the oldest still-supported Ubuntu LTS release. Projects creating app stores or software managers are free to use this data. Myself, I am not much interested in those things as I always download AppImages right from the respective project’s download pages. My typical AppImage discovery goes like this:

1.  Read on Twitter that PrusaSlicer has this cool new feature
2.  Go to the PrusaSlicer GitHub project and read the release notes there
3.  While there, download the AppImage and have it running a few seconds later

So for me personally, I have no need for app centers and app stores, but if people like them, they are free to put AppImages in there. I just never felt the need…

**_It’s FOSS: What plans do you have for AppImage in future (new features that you plan to add)?_**

**Simon:** Simplify things even more, remove configuration options, make things “just work”. Reduce the number of GitHub projects needed to get the core AppImage experience for producing and consuming AppImages, including aspects like binary delta updates, sandboxing, etc. Improve usability.

**_It’s FOSS: Does AppImage project makes money? What kind of support (if any) do you seek from the end users?_**

**Simon:** No, AppImage makes no money whatsoever.

I’ll just request the readers to spread the word. Tell your favorite application’s authors that you’d like to see an AppImage, and why.

___

Team It’s FOSS congratulates Simon for his hard work. Please feel free to convey any message and queries to him in the comment section.

Skip![arrow](https://sdk.apester.com/assets/iconRightArrow.svg)

Skip

Skip ad in

![arrow](https://sdk.apester.com/assets/iconRightArrow.svg)

![](https://itsfoss.com/wp-content/gravatars/2003563996b123e9460848e5ea27f4d6)

Creator of It's FOSS. An ardent Linux user & open source promoter. Huge fan of classic detective mysteries ranging from Agatha Christie and Sherlock Holmes to Detective Columbo & Ellery Queen. Also a movie buff with a soft corner for film noir.