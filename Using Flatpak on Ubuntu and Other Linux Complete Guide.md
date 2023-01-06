_**Brief: Flatpak is a new universal packaging format. Enabling Flatpak will give you access to the easy installation of many Linux applications. Here’s how to use Flatpak in Ubuntu and other Linux distributions.**_

Installing an application in Linux is as easy as opening the Software Center, searching and installing it. The applications which are not available in the App Stores can be installed via the DEB or RPM packages. Some of them are available via PPAs (for Debian based distributions) and if nothing, one can [build from the source code](https://itsfoss.com/install-software-from-source-code/).

There are some limitations though. The App Stores do not generally have the latest release of an application, handling dependencies can be tiresome and the PPAs may not always be safe! And, building from the source requires some terminal hands-on.

With several Linux distributions and the package management systems, there was a need of a Universal Packaging system that could run an application irrespective of what Linux distribution you are using. Canonical thought of it and created [Snaps](https://itsfoss.com/install-snap-linux/). There is also an independent universal software package called [AppImage](https://itsfoss.com/use-appimage-linux/) where you download an application and run it without actually installing the application.

Along with Snaps and [AppImage](https://appimage.org/), there is another universal package system called [Flatpak](https://www.flatpak.org/). We will see how to install and use Flatpak on most Linux distributions along with its advantages.

## What is Flatpak?

[Flatpak](https://flatpak.org/) is basically a framework for the applications on Linux. With the different distributions preferring their own package management, Flatpak aims at providing a cross-platform solution with other benefits. It makes the work for developers even easier. A single application build can be used in almost all the Linux distribution (that support Flatpak) without any modification to the bundle.

### Primary advantages of Flatpak

-   Apart from offering a single bundle for different Linux distributions, Flatpak offers integration to the Linux desktops making it easier to browse, install and use Flatpak applications, e.g. the Gnome Software Center can be used to install a Flatpak.
-   Flatpaks are forward compatible i.e. the same Flatpak app can run on the next releases of a distribution without changes.
-   Run-time dependencies are maintained which can be used by the application. Missing ones can be added as a part of the application.
-   Though Flatpak provides a centralized service for distribution of applications, it fully supports the decentralized distribution of applications.

## A. Enable Flatpak support for various Linux distributions

![How to use Flatpak in Ubuntu and other Linux distributions](https://itsfoss.com/wp-content/uploads/2018/05/use-flatpak-linux.jpeg)

Installing Flatpak is a two-step process. The first one is to install Flatpak and then we have to add a Flatpak repo (here, Flathub) from where we can install applications.

### Install Flatpak on Ubuntu and Linux Mint

Install Flatpak support on Ubuntu with the following command:

```
sudo apt install flatpak
```

### Install Flatpak on Red Hat and Fedora based Linux distributions

To install Flatpak on Red Hat and Fedora, you just have to type in the following the command below:

```
sudo yum install flatpak
```

### Install Flatpak on openSUSE

To enable Flatpak support on openSUSE based Linux distributions, use the command below:

```
sudo zypper install flatpak
```

### Install Flatpak on Arch Linux

To enable Flatpak support on Arch based Linux distributions, use the command below:

```
sudo pacman -S flatpak
```

## B. Enable Flatpak application support in Software Center

Flatpak applications can be completely managed via command line. But not everyone likes using command line for installing applications and this is where enabling Flatpak support in GNOME software center will be a lifesaver.

On some distrubutions like Pop!\_OS 20.04, you will find Flatpak integrated with the software center. So, you don’t need to separately do anything about it.

However, if you don’t have the Flatpak integration by default, you will need the GNOME software plugin to install flatpak via GUI. Use the below command to install it in Ubuntu based distributions:

```
sudo apt install gnome-software-plugin-flatpak
```

For other distributions, use the regular package installation command to install gnome-software-plugin-flatpak. Once installed, restart the Software Center or your machine.

Now you can download the **.flatpakref** file from the application developer’s website or from the official Flatpak application store, [Flathub](https://flathub.org/home).

Navigate to the download folder and double click on the downloaded .flatpakref file. It should open the Software Center and will provide the installation option as shown in the picture below:

![](https://itsfoss.com/wp-content/uploads/2018/05/Flatpak-800x434.jpg)

You can also right click on the file and **Open with Software Install (default)** if double click doesn’t work.

Once the installation completes, you can launch it from software center or from the application menu.

## C. Using Flatpak commands (for intermediate to experts)

Now that we have seen how to enable Flatpak support and how to install Flatpak applications, we can move forward to see Flatpak commands for complete control over package installation.

This part of the tutorial is optional and only intended for intermediate to expert users who prefer command line over GUI.

### Add repositories for installing Flatpak applications

Flatpak needs to have repository information from where you can find and download applications. It would be a good idea to add the Flathub repository so that you get access to a number of Flatpak applications.

It is worth noting that at the time of writing this — [Flathub](https://flathub.org/) is the most popular repository for installing Flatpak. So, we’ve used it for every command mentioned. If you’re using some other repository (remote source), feel free to replace Flathub with the one you’re using for every command.

To do that, use the following command:

```
flatpak remote-add --if-not-exists flathub https://flathub.org/repo/flathub.flatpakrepo
```

There could be other repositories available — check and add them as needed.

### Searching Flatpak through terminal

You can search for available Flatpak applications using the search option in Flatpak command in the following manner:

```
flatpak search applicationname
```

The application name need not be exact. It will show all possible results matching the search query.

For example, _flatpak search libreoffice_ returns LibreOffice stable release.

![Flatpak Search New](https://itsfoss.com/wp-content/uploads/2020/07/flatpak-search-new.jpg)

You should note two things in the above command output. The “**Application ID**” and “**Remotes**“. You’ll need these two for installing the application.

### Install Flatpak applications

The generic way to install a Flatpak application from a repository is:

```
flatpak install <remotes> <ApplicationID>
```

For example, in the previous search command, you got the Application ID and the repository name. You can use this info to install the application in the following manner:

```
flatpak install flathub org.libreoffice.LibreOffice
```

![Flatpak Flathub Installation](https://itsfoss.com/wp-content/uploads/2020/07/flatpak-flathub-installation.jpg)

Some developers provide their own repository. You can use the absolute path to the application’s flatpakref to install the application or through Flathub.

```
flatpak install --from https://flathub.org/repo/appstream/com.spotify.Client.flatpakref
```

### Install Flatpak applications from flatpakref file

If you have downloaded the .flatpakref file on your system, navigate to the directory and use the command to install it:

```
flatpak install <ApplicationID>.flatpakref
```

Suppose, you’ve downloaded **net.poedit.Poedit.flatpakref** file, the command will look like:

```
flatpak install net.poedit.Poedit.flatpakref
```

### Run a Flatpak

To run a Flatpak application, you can use the command below:

```
flatpak run <ApplicationID>
```

For instance, if you installed spotify, here’s how the command will look like:

```
flatpak run com.spotify.Client
```

### Display all Flatpak apps installed on your system

You can display all Flatpak applications installed on your system using the command below:

```
flatpak list
```

![Flatpak List](https://itsfoss.com/wp-content/uploads/2020/07/flatpak-list.png)

### Uninstall a Flatpak application

You can use the uninstall option with the application id to remove the installed Flatpak package.

```
flatpak uninstall <ApplicationID>
```

Here’s how it should look like:

```
flatpak uninstall com.spotify.Client
```

### Updating all Flatpak applications at once

```
flatpak update
```

### Free up space by removing unused Flatpak runtimes

It would be wise to clean your system and free up space from time to time. You can remove the unused Flatpak runtimes with this command:

```
flatpak uninstall --unused
```

The above command lists the unused runtimes and gives you the option to remove them all.

## D. Troubleshooting Flatpak

In this section, we’ll see some common issues you may face with Flatpak.

### Fix Flatpak Installation Error

If you encounter an error like this:

```
error: runtime/org.freedesktop.Platform/x86_64/1.6 not installed
```

You can easily fix it using this command:

```
flatpak update -v
```

You get the error if you had Flatpak installation incomplete because of poor internet connection or system shutdown. Updating Flatpak repositories usually fixes this problem.

### What do you think of Flatpak?

Enabling Flatpak support certainly provides access to more software. Flathub website provides an easy way of finding these Flatpak applications.

Not only Flatpak addresses the cross-platform application installation among Linux users, it saves efforts to develop separate bundles for different distribution. A single package can be used on various kinds of Linux distributions and the maintenance is super easy.

Though, in comparison to [Snap](https://itsfoss.com/install-snap-linux/), Flatpak is slightly complicated. Relying on the application id instead of application name is an annoyance in my opinion. I was also surprised that installation and removal of Flatpak application doesn’t require sudo rights.

What do you think about Flatpak and do you use them? Do you prefer it over AppImage or Snaps? Let us know if you face any issue in the comment section.

![](https://itsfoss.com/wp-content/gravatars/4ceb3b9f411b669e94670b4829cc685e)

DevOps Engineer by profession, believes in "Human Knowledge belongs to the world"!