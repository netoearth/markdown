One of the biggest problems with universal packages like [Snap](https://snapcraft.io/), [Flatpak](https://flatpak.org/) and [AppImage](https://appimage.org/) is managing them. Most built-in package managers do not support all of these new formats.

Thankfully, I stumbled across an application that supports several universal package formats.

## Bauh – a Manager for Your Multi-Package Needs

Originally named fpakman, [bauh](https://github.com/vinifmor/bauh) is designed to handle Flatpak, Snap, [AppImage](https://itsfoss.com/use-appimage-linux/), and [AUR](https://itsfoss.com/best-aur-helpers/) packages. Creator [vinifmor](https://github.com/vinifmor) started the project in June’19 with the [intention](https://forum.manjaro.org/t/bauh-formerly-known-as-fpakman-a-gui-for-flatpak-and-snap-management/96180) of “giving a graphical interface to manage Flatpaks for Manjaro users.” Since then, he has expanded the application to add support for Debian-based systems.

![Bauh About](https://itsfoss.com/wp-content/uploads/2019/11/bauh-about.jpg)

Bauh About

When you first open bauh, it will scan your installed applications and check for updates. If there are any that need to be updated, they will be listed front and center. Once all the packages are updated, you will see a list of packages you have installed. You can deselect a package with updates to prevent it from being updated. You can also choose to install a previous version of the application.

![Bauh](https://itsfoss.com/wp-content/uploads/2019/11/bauh.jpg)

With Bauh you can manage various types of packages from one application

You can also search for applications. Bauh has detailed information for both installed and searched packages. If you are not interested in one (or more) of the packaging types, you can deselect them in settings.

## Installing bauh on your Linux distribution

Let’s see how to install bauh.

### Arch-based distributions

If you have a recent install of [Manjaro](https://manjaro.org/), you should be all set. Bauh comes installed by default. If you have an older install of Manjaro (like I do) or a different Arch-based distro, you can install it from the [AUR](https://aur.archlinux.org/packages/bauh) by typing this in terminal:

```
sudo pacman -S bauh
```

![Bauh Package Info](https://itsfoss.com/wp-content/uploads/2019/11/bauh-package-info.jpg)

Bauh Package Info

### Debian/Ubuntu based distributions

If you have a Debianor Ubuntubased Linux distribution, you can install bauh with pip. First, make sure to [install pip on Ubuntu](https://itsfoss.com/install-pip-ubuntu/).

```
sudo apt install python3-pip
```

And then use it to install bauh:

```
pip3 install bauh
```

However, the creator recommends installing it [manually](https://github.com/vinifmor/bauh#manual-installation) to avoid messing up your system’s libraries.

To install bauh manually, you have to first download the [latest release](https://github.com/vinifmor/bauh/releases). Once you download it, you can [unzip using a graphical tool](https://itsfoss.com/unzip-linux/) or the [unzip command](https://linuxhandbook.com/unzip-command/). Next, open up the folder in your terminal. You will need to use the following steps to complete the install.

First, create a virtualenv in a folder called env:

```
python3 -m venv env
```

Now install the application code inside the env:

```
env/bin/pip install .
```

And launch the application:

```
env/bin/bauh
```

![Bauh Updating](https://itsfoss.com/wp-content/uploads/2019/11/bauh-updating.jpg)

Bauh Updating

Once you finish installing bauh, you can [fine-tune](https://github.com/vinifmor/bauh#general-settings) it by changing the environment setting and arguments.

## The road ahead for bauh

Bauh has grown quite a bit in a few short months. It plans to continue to grow. The current [road map](https://github.com/vinifmor/bauh#roadmap) includes:

-   Support for other packaging technologies
-   Separate modules for each packaging technology
-   Memory and performance improvements
-   Improve the user experience

![Bauh Search](https://itsfoss.com/wp-content/uploads/2019/11/bauh-search-800x319.png)

Bauh Search

## Final thoughts

When I tried out bauh, I ran into a couple of issues. When I opened it up for the first time, it told me that Snap was not installed and that I would have to install it if I wanted to use Snaps. I know that Snap is installed because I ran `snap list` in the terminal and it worked. I restarted the system and snaps worked.

The other issue I ran into was that one of my AUR packages failed to update. I was able to update the package without any issue with `yay`. There might be an issue with my install of Manjaro, I’ve had it going for 3 or 4 years.

Overall, bauh worked. It did what was printed on the tin. I can’t ask for more than that.

Have you ever used bauh? What is your favorite tool to manage different package formats if there is one? Let us know in the comments below.

If you found this article interesting, please take a minute to share it on social media, Hacker News or [Reddit](https://reddit.com/r/linuxusersgroup).

Skip![arrow](https://sdk.apester.com/assets/iconRightArrow.svg)

Skip

Skip ad in

![arrow](https://sdk.apester.com/assets/iconRightArrow.svg)

![](https://itsfoss.com/wp-content/gravatars/4294f7c8ccf41abdd950c3566d5371cc)

My name is John Paul Wohlscheid. I'm an aspiring mystery writer who loves to play with technology, especially Linux. You can catch up with me at [my personal website](http://johnpaulwohlscheid.work/). You can check out my ebooks [here](https://books2read.com/ap/RW7mNR/John-Paul-Wohlscheid).

I also write a [newsletter about the stuff that most history books miss](https://historicaltidbits.substack.com/) and [old computer ads](https://computeradsfromthepast.substack.com/). Check them out.