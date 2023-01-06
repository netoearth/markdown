## Objective

Add DXVK to an existing Wine prefix, improving performance.

## Distributions

This guide focuses on Ubuntu, but the procedure will work on any distribution.

## Requirements

A working Linux install with root privileges.

## Conventions

-   **#** – requires given [linux commands](chrome-extension://hajanaajapkhaabfcofdjgjnlgkdkknm/linux-commands) to be executed with root privileges either directly as a root user or by use of `sudo` command
-   **$** – requires given [linux commands](chrome-extension://hajanaajapkhaabfcofdjgjnlgkdkknm/linux-commands) to be executed as a regular non-privileged user

## Introduction

Wine gaming is sort of a moving target. It always has been. Just when you think you know the lay of the land, something new pops up and disrupts everything. The latest curveball comes in the form of DXVK.

DXVK is a set of replacement `dll` files that translate from DirectX 11 to Vulkan. While DXVK is still very new, and it hasn’t even seen its 1.0 release yet, Wine gamers are jumping on board with both feet.

Because it moves DX11 code to Vulkan, DXVK directly addresses the biggest problem with Wine gaming today, incompatibility with newer forms of DirectX. Most games are moving as far from DirextX 9 as possible, and breaking Wine comparability in the process. DXVK has very real potential as a solution.

## Install Vulkan

Before you can make use of DXVK, you need Vulkan support. That means different things, depending on your graphics card and drivers, but there are some universal parts. Install them first.

```
$ sudo apt install libvulkan1 libvulkan-dev vulkan-utils
```

___

___

### Mesa

If you’re using Mesa, ether with AMD or Intel, it’s a very good idea to get the absolute latest version of Mesa possible. There’s a great PPA that continually updates Mesa from Git for Ubuntu.

```
$ sudo add-apt-repository ppa:oibaf/graphics-drivers
$ sudo apt updat
```

Upgrade everything.

```
$ sudo apt upgrade
```

Now, install the Mesa Vulkan drivers.

```
$ sudo apt install mesa-vulkan-drivers
```

It’s a good idea to restart your computer here to make sure that you’re using the new version of Mesa with Vulkan.

### NVIDIA

The NVIDIA proprietary drivers already come with Vulkan support, so there isn’t anything extra that you need to do. Just be sure that you have the latest ones on your system. If you’re still running drivers from the default repositories, consider adding the graphics PPA.

```
$ sudo add-apt-repository ppa:graphics-drivers/ppa
$ sudo apt update
$ sudo apt upgrade
```

There is also a Vulkan package provided by the PPA. Install it too.

```
$ sudo apt install vulkan
```

___

___

## Install Lutris

You can absolutely run DXVK without Lutris, but it makes everything with Wine so much easier. Consider running Lutris rather than wrangling independent Wine configurations yourself.

DXVK also works on a per-prefix basis, so the compartmentalization that Lutris brings also makes it much better suited for this sort of thing.

If you need help getting Lutris set up, check out our [Lutris guide](https://linuxconfig.org/install-lutris-on-ubuntu-18-04-bionic-beaver-linux).

## Install A Game

Pick a game to install. Anything that runs on DirectX 11 is a good candidate to test out. Keep in mind that not every game runs better with DXVK. It’s still a very young project, and it’s not optimized for every situation yet. This guide is going to follow Overwatch. It’s a fairly popular DX11-only game, and it works well with Lutris.

Go to the [game page](https://lutris.net/games/overwatch/), and click the “Install” button below the image slideshow. That will begin the Lutris install.

Let the install go as normal, and follow the instructions given by Lutris. Don’t worry about DXVK just yet.

When the install is done, exit the game, or don’t launch it at all when prompted.

### Update Wine

If you are following along with Overwatch, you might want to update the version of Wine that Lutris is using. The Overwatch script hasn’t been updated in a while, and still uses Wine 2.21.

Click the “Runners” icon. It’s the second one from the left. Scroll down to Wine in the resulting Window. Click on “Manage versions” button. Select the latest version of Wine Staging, and wait for it to install. When it’s done, close both windows.

[![Lutris Change Wine Version](https://linuxconfig.org/wp-content/uploads/2018/05/dxvk-ow-wine-version.jpg)](https://linuxconfig.org/wp-content/uploads/2018/05/dxvk-ow-wine-version.jpg "Lutris Change Wine Version")

Lutris Change Wine Version

Right click on the game’s banner image, and select `Configure`. Choose the `Runner options` tab. Change the version of Wine to the version of Staging that you just downloaded.

___

___

## Download DXVK

You’re finally ready to bring DXVK into the equation. Head over to the project’s [release page](https://github.com/doitsujin/dxvk/releases), and download the latest tarball.

Unpack the tarball someplace convenient. DXVK installs itself via symlinks, so you can leave the single folder in one central location.

## Run The Installer Scripts

Inside the DXVK folder, you’ll find two additional folders, one for x32 and one for x64. You need both. Change into the x32 one first.

```
$ cd ~/Downloads/dxvk-0.50/x32
```

There are a couple of things in the folder. It has the two replacement `dll` files, and an installer script. The script places symlinks of the `dll`s into `system32` of your Wine prefix and creates an override for each one to be used natively.

[![DXVK Run Install Script](https://linuxconfig.org/wp-content/uploads/2018/05/dxvk-run-script.jpg)](https://linuxconfig.org/wp-content/uploads/2018/05/dxvk-run-script.jpg "DXVK Run Install Script")

DXVK Run Install Script

To run the script, set the Wine prefix, and run it.

```
$ WINEPREFIX=~/Games/overwatch ./setup_dxvk.sh
```

Do the same thing in the x64 folder. It’ll create links in `syswow64`.

## Test It Out

[![DXVK DLL Overrides](https://linuxconfig.org/wp-content/uploads/2018/05/dxvk-wine-overrides.jpg)](https://linuxconfig.org/wp-content/uploads/2018/05/dxvk-wine-overrides.jpg "DXVK DLL Overrides")

DXVK DLL Overrides

To make sure that the script ran, right click on your game again, and select `Wine Configuration`. This will bring up a a typical `winecfg` window. Check under the `Libraries` tab. You should see overrides for `d3d11` and `dxgi`.

Open up and run your game like you normally would. Everything should still work, but now, you should notice a performance bump. Again, results aren’t exactly guaranteed here, but it’s always worth testing.

## Closing Thoughts

You now have a game running DXVK with Wine. Expect rapid progress and advancements with DXVK in the coming months. This young project has a bright future and may just end up in mainline Wine some day.