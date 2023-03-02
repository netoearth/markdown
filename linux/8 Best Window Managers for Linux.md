A window manager should be helpful if you dabble with multiple active windows on your system and want to make the most out of the available screen space.

Sure, you can re-size and reposition your windows to organize them to some extent. However, with a window manager, you can **step up your multitasking potential** by organizing the windows using keyboard shortcuts, mouse and even automating some of it.

With a window manager, you can **improve not only the productivity but the look and feel of your desktop** if you decide to put in the effort.

Here's an example of a user's customized desktop using a window manager to organize the active windows:

[![bspwm desktop screenshot](https://itsfoss.com/content/images/2023/01/window-manager-example.jpg)](https://itsfoss.com/content/images/2023/01/window-manager-example.jpg)

Image Credits: Customized Desktop with bspwm by [u/emanuelep57](https://www.reddit.com/r/unixporn/comments/mnbx1o/bpswm_my_first_rice_bspwm_spaceblue_with_optional/)

📋

Most of the options mentioned work for the **Xorg display server**, considering window managers on Wayland are called compositors. You can explore [Arch's documentation](https://wiki.archlinux.org/title/window_manager) to learn more.

For multi-monitor setups, you might want to check for **RandR** and **Xinerama** protocol support with the window managers.

## Is it Easy to Use a Window Manager?

Yes and no.

If you decide to use a window manager, you must be willing to research/read documentation to get things right. It may not be feasible to cover everything in this article.

There are some points to note that include:

-   Some window managers provide you with room for endless customizability. If you do not know how to configure it to your liking, you may not be able to use it.
-   Some window managers might need configuration even before using it. Unless you do not set it up, you cannot utilize the window manager.
-   Most users prefer **Arch Linux** to use window managers. So, you can expect better community support for configurations/setup if you pick Arch Linux as your distro of choice. But, it is not mandatory, you can try other distributions.

Here, I provide the **links to its documentation, and the installation commands** to give you a head start.

To make things easy, you can also utilize other **users' dotfiles (configuration)** to quickly modify the look and behavior of your desktop. However, you still need to know what you're doing before you use them.

🚧

Some window managers/compositors for Wayland may not support proprietary graphics drivers, including NVIDIA. So, before installing and configuring any of the window managers, you might want to check what it supports.

## 1\. bspwm

[![bspwm desktop screenshot](https://itsfoss.com/content/images/2023/01/window-manager-example.jpg)](https://itsfoss.com/content/images/2023/01/window-manager-example.jpg)

Image Credits: Customized Desktop with bspwm by [u/emanuelep57](https://www.reddit.com/r/unixporn/comments/mnbx1o/bpswm_my_first_rice_bspwm_spaceblue_with_optional/)

[bspwm](https://github.com/baskerville/bspwm) is a **lightweight tiling window manager**. The first screenshot you see in this article was made possible using bspwm.

You must install the window manager and a separate package to use the keyboard/pointer bindings. bspwm lets you manually choose how new windows get inserted or positioned or set it to automatic mode, where it follows a particular scheme to arrange the windows.

You need to configure it properly before you get to use it. Unfortunately, the [documentation](https://github.com/baskerville/bspwm) for it may not be easy to follow for new users trying a window manager. [Arch Linux's documentation page](https://wiki.archlinux.org/title/bspwm) on bspwm should help you get started.

**Install bspwm**

You can easily find the package for it on the official repositories of Fedora, Ubuntu, and Arch.

For Ubuntu, type in the following command to get it installed:

```
sudo apt install bspwm sxhkd
```

📋

The list is in no particular order of ranking.

## 2\. Qtile

[![qtile screenshot](https://itsfoss.com/content/images/2023/01/qtile-screenshot.jpg)](https://itsfoss.com/content/images/2023/01/qtile-screenshot.jpg)

Customized Desktop with Qtile by [u/lzmkalos](https://www.reddit.com/r/unixporn/comments/104a08p/qtile_yeah_i_love_red_palette/)

[Qtile](http://qtile.org/) is a customizable tiling window manager that works on **X11 and Wayland**.

It packs in various features and yet a simple implementation. You get a command shell to inspect and manage all aspects of the window manager.

One of the highlights of Qtile is **complete remote scriptability**.

**Install Qtlie**

You can install Qtile using pip. Once you have [pip installed on Ubuntu](https://itsfoss.com/install-pip-ubuntu/), run these commands:

```
pip install xcffib
pip install qtile
```

Refer to the [official documentation](https://docs.qtile.org/en/latest/manual/install/ubuntu.html) or its [GitHub page](https://github.com/qtile/qtile) for other Linux distributions.

## 3\. herbstluftwm

[![herbstluftwm screenshot](https://itsfoss.com/content/images/2023/01/herbstluftwm-screenshot.jpg)](https://itsfoss.com/content/images/2023/01/herbstluftwm-screenshot.jpg)

Customized Desktop with herbstluftwm by [u/CIMPBIBAI](https://www.reddit.com/r/unixporn/comments/z4hrw8/herbstluftwm_powerline_config_for_polybar/)

herbstluftwm (I know, a mouthful) is a **manual tiling window manager**. Not as popular as other options, but a promising option for Linux users.

The key highlight of the window manager is that the **configuration for the tool happens at runtime**. So, you do not need to restart the window manager and yet manage to make changes live.

The [documentation](https://herbstluftwm.org/news.html) may not be beginner-friendly, but you can choose to explore parts of it to better understand its functioning.

**Install herbstluftwm**

You can find it in the official repository. To install it, run the following command:

```
sudo apt install herbstluftwm
```

In either case, feel free to explore its official website and [GitHub page](https://github.com/herbstluftwm/herbstluftwm) for more info.

## 4\. awesome

[![awesome wm desktop screenshot](https://itsfoss.com/content/images/2023/01/awesome-wm.jpg)](https://itsfoss.com/content/images/2023/01/awesome-wm.jpg)

[awesome](https://awesomewm.org/index.html) is a **fast and configurable window manager**. It does require a few dependencies along with the installation process to get things working, but it should not be a problem for most.

If you want to access a window manager without needing to configure much from the start, awesomewm should be a good option. It may not look pleasing if you just install and use it without configuration, but you can access most of its functions easily.

The [documentation](https://awesomewm.org/doc/api/documentation/07-my-first-awesome.md.html) for awesome window manager is valuable enough to make the most out of it.

**Install awesome**

The package should be available in the repositories of all major distributions. For Ubuntu, you can type in the following command:

```
sudo apt install awesome
```

## 5\. IceWM

[![icewm desktop screenshot](https://itsfoss.com/content/images/2023/01/icewm-arch.jpg)](https://itsfoss.com/content/images/2023/01/icewm-arch.jpg)

Customized Desktop with icewm by [u/Wolandark](https://www.reddit.com/r/unixporn/comments/zipj6w/icewm_first_hour_with_icewm_a_very_pleasant/)

IceWM is one of the oldest tiling window managers out there. You can find it as the default window manager with some distributions like [antiX](https://antixlinux.com/blog/).

You may not get an extensive list of functionalities with IceWM, but it has a simple approach that lets you easily use it. By default, it features an app launcher and a taskbar to keep things familiar and accessible.

Head to its [official website](https://ice-wm.org/) for documentation and get started.

**Install IceWM**

IceWM is available in official repositories of all major distros. You can install it on Ubuntu using the following command:

```
sudo apt install icewm
```

## 6\. i3

[![customized desktop i3 screenshot](https://itsfoss.com/content/images/2023/01/i3-screenshot.jpg)](https://itsfoss.com/content/images/2023/01/i3-screenshot.jpg)

Customized Desktop with i3 by [u/Ramin-Yousefpour](https://www.reddit.com/user/Ramin-Yousefpour/)

[i3](https://i3wm.org/) is the most popular option if you are in for an **insane amount of customization**. Yes, it is aimed at advanced users and developers, but with its well-documented [instructions](https://i3wm.org/docs/), anyone can try to use it.

You can expect numerous abilities with i3 as long as you can configure them. Whether you have dual-monitor setup, or a multi-monitor setup with horizontal displays, configuration is the key here.

**Install i3**

i3 is available in repositories for every major distribution. For Ubuntu, you can use the command below to get it installed:

```
sudo apt install i3
```

To explore technical details, head to its [GitHub page](https://github.com/i3/i3).

## 7\. Sway

[![sway desktop screenshot](https://itsfoss.com/content/images/2023/01/sway-screenshot.jpg)](https://itsfoss.com/content/images/2023/01/sway-screenshot.jpg)

Customized Desktop with i3 by [u/J\_o\_a\_n](https://www.reddit.com/r/unixporn/comments/10bwwzw/sway_love_wayland_and_catppuccin_mocha_blue/)

[Sway](https://swaywm.org/) is **designed for Wayland sessions** while offering compatibility with i3. In other words, the same commands are supported with Sway.

If you are using i3 and want to move to Sway on a Wayland desktop, the transition should be easy by copying the configuration to the correct file.

You should have the essential features here to organize app windows and make efficient use of desktop space.

**Install Sway**

Most of the popular distributions should already have the package available. For Debian-based systems, you can use the terminal to get it installed:

```
sudo apt install sway
```

To explore more, check out its [GitHub page](https://github.com/swaywm/sway).

## 8\. xmonad

[![xmonad desktop screenshot](https://itsfoss.com/content/images/2023/01/xmonad-screenshot.png)](https://itsfoss.com/content/images/2023/01/xmonad-screenshot.png)

Customized Desktop with xmonad by [u/Walker0712](https://www.reddit.com/r/unixporn/comments/z0w7ai/xmonad_arch_first_rice/)

[xmonad](https://xmonad.org/) is a tiling window manager for X11 written and configured in [Haskell](https://www.haskell.org/) language.

It aims to provide all kinds of functionalities while **making it easier to automate things** all the way.

You get a decent [documentation](https://xmonad.org/) to get started and start taking charge of your windows.

**Install xmonad**

Unlike others, it is not as simple as installing a single package (especially for Debian/Ubuntu users).

So, you may want to follow the [official installation instructions](https://xmonad.org/INSTALL.html) and its [GitHub page](https://github.com/xmonad/xmonad) to proceed.

## Honorable Mentions

There are various other compositors (Wayland) and window managers like [ratpoison](https://www.nongnu.org/ratpoison/) that may not be feature-rich or popular enough but can be interesting to try.

Some of those options are:

-   [Cagebreak](https://github.com/project-repo/cagebreak) (Wayland)
-   [river](https://github.com/riverwm/river) (Wayland)
-   [JWM](https://github.com/joewing/jwm)
-   [Spectrwm](https://github.com/conformal/spectrwm)
-   [dwl](https://github.com/djpohly/dwl) (Wayland)

💬 _What are your favorite window managers for Linux? Did we miss any of your favorites? Let us know in the comments section below._

Skip![arrow](https://sdk.apester.com/assets/iconRightArrow.svg)

Skip

Skip ad in

![arrow](https://sdk.apester.com/assets/iconRightArrow.svg)