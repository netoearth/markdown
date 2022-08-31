I am a new Ubuntu Linux user. My laptop has Nvidia GPU. How do I install Nvidia drivers on Ubuntu Linux 16.04, 18.04, 18.10, 20.04/22.04 LTS?  
  
**Introduction**: Nvidia graphics processing units (GPUs) used for gaming and professional use in offices. Nvidia GPUs used in data centers, visualization, automobile industry, and artificial intelligence. By default X.org X server use nouveau free/libre software drivers for nVidia cards. The nouveau driver generally provides the inferior performance to Nvidia’s proprietary graphics device drivers for gaming and other professional use. **This page shows how to install Nvidia GPU drivers on an Ubuntu Linux desktop.**

| Tutorial details |
| --- |
| Difficulty level | [Easy](http://www.cyberciti.biz/faq/tag/easy/ "See all Easy Linux / Unix System Administrator Tutorials") |
| Root privileges | [Yes](http://www.cyberciti.biz/faq/how-can-i-log-in-as-root/ "See how to login as root user") |
| Requirements | Linux terminal |
| Category | [Driver Management](https://www.cyberciti.biz/faq/ubuntu-linux-install-nvidia-driver-latest-proprietary-driver/#Driver_Management "See ALL other tutorials in Driver Management category") |
| Prerequisites | Ubuntu Linux |
| OS compatibility | Mint • Pop!\_OS • [Linux](https://www.cyberciti.biz/faq/category/linux/ "See all Linux distributions tutorials") • [Ubuntu](https://www.cyberciti.biz/faq/category/ubuntu-linux/ "See all Ubuntu Linux tutorials") |
| Est. reading time | 6 minutes |

## Ubuntu Linux Install Nvidia Driver

The procedure to install proprietary Nvidia GPU Drivers on Ubuntu 16.04 / 17.10 / 18.04 / 18.10 / 20.04 / 22.04 LTS is as follows:

1.  Update your system running apt-get command
2.  You can install Nvidia drivers either using GUI or CLI method
3.  Open “**Software and Updates**” app to install install Nvidia driver using GUI
4.  OR type “sudo apt install nvidia-driver-510 nvidia-dkms-510” at the CLI
5.  Reboot the computer/laptop to load the drivers
6.  Verify drivers are working

Let us see all commands and step-by-step instructions in details.

## Finding out information about your GPU

Naturally, you can only install Nvidia driver if you have Nvidia GPU in your system. Type the hwinfo command/lshw command to find out info about your GPU  
`$ hwinfo --gfxcard --short`  
OR  
`$ sudo lshw -C display`  
![Ubuntu Linux Find Nvidia GPU info](https://www.cyberciti.biz/media/new/faq/2018/10/Ubuntu-Linux-Find-Nvidia-GPU-info.png)![Ubuntu Linux Find Nvidia GPU info](https://www.cyberciti.biz/media/new/faq/2018/10/Ubuntu-Linux-Find-Nvidia-GPU-info.png)  
You must have Nvidia GPU card detected by your system. See “[Linux Find Out Graphics Card Installed In My System](https://www.cyberciti.biz/faq/linux-tell-which-graphics-vga-card-installed/)” for more info.

  

Patreon supporters only guides 🤓

**Warning**: You can use only one method. If you use a GUI or apt CLI method, do not try other commands. I have observed broken systems if you mixed various methods on the same system. Stick with one method. If you followed the tutorial from any other sources, make sure you remove NVIDIA driver and undo whatever you did. Then reboot the system. Then proceed as follows. The following instructions might fail if you are using a custom build Linux kernel on Ubuntu Linux, as it heavily depends upon a specific version of your distro.

## Installing Nvidia driver using GUI method # 1 on Ubuntu Linux

Press the Super key (Windows key) and type the following in search box:  
`update manager`  
![Ubuntu software manager to update packages](https://www.cyberciti.biz/media/new/faq/2018/10/Ubuntu-software-manager-to-update-packages.png)![Ubuntu software manager to update packages](https://www.cyberciti.biz/media/new/faq/2018/10/Ubuntu-software-manager-to-update-packages.png)  
Click on the **Settings** button:  
![Ubuntu Linux Software Updater](https://www.cyberciti.biz/media/new/faq/2018/10/Ubuntu-Linux-Software-Updater.png)  
Click on the **Additional drivers** tab:  
![Ubuntu Linux Install Nvidia Driver using Sofware Manager GUI tool](https://www.cyberciti.biz/media/new/faq/2018/10/Ubuntu-Linux-Install-Nvidia-Driver-using-Sofware-Manager-GUI-tool.png)  
Choose **nvidia-driver-460 (proprietary, tested)** and click on the **Apply Changes** button. You must authenticate yourself to install Nvidia driver on Ubuntu Linux. You must wait for some time as Nvidia driver downloaded and installed from the internet:  
![Installing Nvdia drivers on Ubuntu Linux](https://www.cyberciti.biz/media/new/faq/2018/10/Installing-Nvdia-drivers-on-Ubuntu-Linux.png)  
Click on the Close button when done. You must [reboot Ubuntu Linux computer to use the driver](https://www.cyberciti.biz/faq/howto-reboot-linux/) by typing the following [shutdown command](https://www.cyberciti.biz/faq/howto-shutdown-linux/ "How To Shutdown Linux Using Command Line"):  
`$ sudo shutdown -r now`  
OR  
`$ sudo reboot`  
Now skip to the driver [verification](https://www.cyberciti.biz/faq/ubuntu-linux-install-nvidia-driver-latest-proprietary-driver/#verification) step.

## Ubuntu Install Nvidia driver using the CLI method # 2

Open the termial application and search for nvidia drivers using the [apt command](https://www.cyberciti.biz/faq/ubuntu-lts-debian-linux-apt-command-examples/ "apt Command Examples for Ubuntu/Debian for new users") or [apt-get command](https://www.cyberciti.biz/tips/linux-debian-package-management-cheat-sheet.html "Ubuntu/Debian Linux apt-get package management cheat sheet"):  
`$ apt search nvidia-driver`  
OR use the apt-cache command to search package:  
`$ apt-cache search nvidia-driver`  
![apt-get search for Nvidia driver package](https://www.cyberciti.biz/media/new/faq/2018/10/apt-get-search-for-Nvidia-driver-package.png)  
Here is how to list all nvidia driver version on your Ubuntu machine using the combination of the apt-cache command and [egrep command](https://www.cyberciti.biz/faq/grep-regular-expressions/ "Regular expressions in grep ( regex ) with examples")/[grep command](https://www.cyberciti.biz/faq/howto-use-grep-command-in-linux-unix/ "How to use grep command in Linux/ Unix with examples"):

```
apt-cache search 'nvidia-driver-' | grep '^nvidia-driver-[[:digit:]]*'
## search for DKMS package too ##
apt-cache search 'nvidia-dkms-' | grep '^nvidia-dkms-[[:digit:]]*'
```

```
nvidia-driver-390 - NVIDIA driver metapackage
nvidia-driver-418 - Transitional package for nvidia-driver-430
nvidia-driver-430 - Transitional package for nvidia-driver-440
nvidia-driver-435 - Transitional package for nvidia-driver-455
nvidia-driver-440 - Transitional package for nvidia-driver-450
nvidia-driver-450 - Transitional package for nvidia-driver-460
nvidia-driver-450-server - NVIDIA Server Driver metapackage
nvidia-driver-455 - Transitional package for nvidia-driver-460
nvidia-driver-460 - Transitional package for nvidia-driver-470
nvidia-driver-465 - Transitional package for nvidia-driver-470
nvidia-driver-470 - NVIDIA driver metapackage
nvidia-driver-470-server - NVIDIA Server Driver metapackage
nvidia-driver-495 - Transitional package for nvidia-driver-510
nvidia-driver-510 - NVIDIA driver metapackage
nvidia-driver-418-server - NVIDIA Server Driver metapackage
nvidia-driver-440-server - Transitional package for nvidia-driver-450-server
nvidia-driver-460-server - Transitional package for nvidia-driver-470-server
nvidia-driver-510-server - NVIDIA Server Driver metapackage
```

### [Updating and installing all security and important updates on Ubuntu](https://www.cyberciti.biz/faq/ubuntu-20-04-update-installed-packages-for-security/)

Do not skip the following two commands as you must apply all pending security updates:  
`$ sudo apt update   $ sudo apt upgrade`

### Installing NVIDIA drivers on Ubuntu

Let us install the nvidia-driver-390 package:  
`$ sudo apt install nvidia-driver-390 nvidia-dkms-390`  
At this time, the latest tested proprietary drive version is 510. We can install that one as follows on Ubuntu Linux 20.04 LTS:  
`$ sudo apt install nvidia-driver-510 nvidia-dkms-510`  
[Reboot the Linux with help of either reboot or shutdown](https://www.cyberciti.biz/faq/linux-reboot-command/) command:  
`$ sudo reboot`  
Now go to the driver [verification](https://www.cyberciti.biz/faq/ubuntu-linux-install-nvidia-driver-latest-proprietary-driver/#verification) step.

## Verification

Open the terminal application and type nvidia-smi to see GPU info and process that are using Nvidia GPU:  
`$ nvidia-smi`  
![NVIDIA System Management Interface program on Ubuntu](https://www.cyberciti.biz/media/new/faq/2018/10/NVIDIA-System-Management-Interface-program-on-Ubuntu.png)  
The nvidia-smi command line utility provides monitoring and management capabilities for each of NVIDIA’s Tesla, Quadro, GRID and GeForce devices from Fermi and higher architecture families. You can see running apps on your GPU and GPU temperature.

### How do I configure the NVIDIA graphics driver?

The nvidia-settings command start a GUI tool for configuring the NVIDIA graphics driver. This is useful to see all GPU info or configure multiple external screen/monitors connected to your system. Open the terminal and type the following commands:  
`$ nvidia-settings`  
If you wish to save settings start it as follows:  
`$ sudo nvidia-settings`  
![Ubuntu nvidia-settings tool to configure NVIDIA graphics driver](https://www.cyberciti.biz/media/new/faq/2018/10/Ubuntu-nvidia-settings-tool-to-configure-NVIDIA-graphics-driver.png)

**Related:** [Top 7 Linux GPU Monitoring and Diagnostic Commands Line Tools](https://www.cyberciti.biz/open-source/command-line-hacks/linux-gpu-monitoring-and-diagnostic-commands/)

## A note about ubuntu-drivers command-line method # 3

We can also change drivers without the use of the X GUI/Windows desktop. For these purposes, Ubuntu comes with a unique command called ubuntu-drivers to manage binary drivers for NVidia and other devices. This is an alternative to the [apt command](https://www.cyberciti.biz/faq/ubuntu-lts-debian-linux-apt-command-examples/ "apt Command Examples for Ubuntu/Debian for new users")/[apt-get command](https://www.cyberciti.biz/tips/linux-debian-package-management-cheat-sheet.html "Ubuntu/Debian Linux apt-get package management cheat sheet") we used earlier.

### Show all driver packages which apply to the current system

`$ sudo ubuntu-drivers list`  
Outputs from my Thinkpad X1E laptop:

### Display all OEM enablement packages which apply to this system

`$ sudo ubuntu-drivers list-oem`

### View all hardware NVidia devices which need drivers, and which packages

`$ sudo ubuntu-drivers devices`  
Here is what we see:

```
== /sys/devices/pci0000:00/0000:00:01.0/0000:01:00.0 ==
modalias : pci:v000010DEd00001F91sv000017AAsd0000229Fbc03sc00i00
vendor   : NVIDIA Corporation
model    : TU117M [GeForce GTX 1650 Mobile / Max-Q]
driver   : nvidia-driver-440-server - distro non-free
driver   : nvidia-driver-455 - distro non-free recommended
driver   : nvidia-driver-418-server - distro non-free
driver   : nvidia-driver-450 - distro non-free
driver   : nvidia-driver-450-server - distro non-free
driver   : xserver-xorg-video-nouveau - distro free builtin
```

### Ubuntu command to install a driver NVidia version 455

Let us install recommended driver automatically:  
`sudo ubuntu-drivers install`

## Conclusion

You learned three different ways to install Nvidia Driver on Ubuntu 18.04 and 20.04 LTS Linux desktop or server. Unmistakably, the GUI method is simple and easy to use. The CLI method is useful for server installation, and the ubuntu-drivers command-line method is recommended for a fresh installation or OEM system like Dell XPS 13 and others. The For more info, see the official Nvidia Unix/Linux driver [page online](https://www.nvidia.com/object/unix.html).  

  

  

  
  

| Category | List of Unix and Linux commands |
| --- | --- |
| Download managers | [wget](https://www.cyberciti.biz/tips/linux-wget-your-ultimate-command-line-downloader.html?utm_source=Cmd_Table&utm_medium=nixCraft&utm_campaign=Apr_22_2022_EOP "Wget Command in Linux with Examples") |
| Driver Management | [Linux Nvidia driver](https://www.cyberciti.biz/faq/tag/linux-nvidia-driver/?utm_source=Cmd_Table&utm_medium=nixCraft&utm_campaign=Apr_22_2022_EOP "Linux Nvidia Driver How To - Linux / Unix Q & A from nixCraft") • [lsmod](https://www.cyberciti.biz/faq/linux-show-the-status-of-modules-driver-lsmod-command/?utm_source=Cmd_Table&utm_medium=nixCraft&utm_campaign=Apr_22_2022_EOP "Linux find out what kernel drivers loaded with lsmod command") |
| Documentation | [help](https://bash.cyberciti.biz/guide/Help_command?utm_source=Cmd_Table&utm_medium=nixCraft&utm_campaign=Apr_22_2022_EOP "Help command - Linux Bash Shell Scripting Tutorial") • [mandb](https://bash.cyberciti.biz/guide/Mandb_command?utm_source=Cmd_Table&utm_medium=nixCraft&utm_campaign=Apr_22_2022_EOP "Mandb command - Linux Bash Shell Scripting Tutorial") • [man](https://bash.cyberciti.biz/guide/Man_command?utm_source=Cmd_Table&utm_medium=nixCraft&utm_campaign=Apr_22_2022_EOP "Man command - Linux Bash Shell Scripting Tutorial") • [pinfo](https://www.cyberciti.biz/open-source/command-line-hacks/linux-command-pinfo-for-colorful-info-pages/?utm_source=Cmd_Table&utm_medium=nixCraft&utm_campaign=Apr_22_2022_EOP "pinfo - Read Linux Info Documentation in Colors") |
| Disk Management | [df](https://www.cyberciti.biz/faq/df-command-examples-in-linux-unix/?utm_source=Cmd_Table&utm_medium=nixCraft&utm_campaign=Apr_22_2022_EOP "How to use df command in Linux / Unix {with examples}") • [duf](https://www.cyberciti.biz/open-source/command-line-hacks/duf-disk-usage-free-utility-for-linux-bsd-macos-windows/?utm_source=Cmd_Table&utm_medium=nixCraft&utm_campaign=Apr_22_2022_EOP "duf - Disk Usage/Free Utility for Linux, BSD, macOS & Windows") • [ncdu](https://www.cyberciti.biz/open-source/install-ncdu-on-linux-unix-ncurses-disk-usage/?utm_source=Cmd_Table&utm_medium=nixCraft&utm_campaign=Apr_22_2022_EOP "How to install ncdu on Linux / Unix to see disk usage") • [pydf](https://www.cyberciti.biz/tips/unix-linux-bsd-pydf-command-in-colours.html?utm_source=Cmd_Table&utm_medium=nixCraft&utm_campaign=Apr_22_2022_EOP "Unix / Linux: See Colourised Filesystem Disk Space Usage with pydf") |
| File Management | [cat](https://www.cyberciti.biz/faq/linux-unix-appleosx-bsd-cat-command-examples/?utm_source=Cmd_Table&utm_medium=nixCraft&utm_campaign=Apr_22_2022_EOP "cat Command in Linux / Unix with examples") • [cp](https://www.cyberciti.biz/faq/copy-command/?utm_source=Cmd_Table&utm_medium=nixCraft&utm_campaign=Apr_22_2022_EOP "Linux Copy File Command [ cp Command Examples ]") • [less](https://bash.cyberciti.biz/guide/Less_command?utm_source=Cmd_Table&utm_medium=nixCraft&utm_campaign=Apr_22_2022_EOP "Less command - Linux Bash Shell Scripting Tutorial") • [mkdir](https://www.cyberciti.biz/faq/linux-make-directory-command/?utm_source=Cmd_Table&utm_medium=nixCraft&utm_campaign=Apr_22_2022_EOP "Linux: How to Make a Directory Command") • [more](https://bash.cyberciti.biz/guide/More_command?utm_source=Cmd_Table&utm_medium=nixCraft&utm_campaign=Apr_22_2022_EOP "More command - Linux Bash Shell Scripting Tutorial") • [tree](https://www.cyberciti.biz/faq/linux-show-directory-structure-command-line/?utm_source=Cmd_Table&utm_medium=nixCraft&utm_campaign=Apr_22_2022_EOP "Linux see directory tree structure using tree command") |
| Firewall | [Alpine Awall](https://www.cyberciti.biz/faq/how-to-set-up-a-firewall-with-awall-on-alpine-linux/?utm_source=Cmd_Table&utm_medium=nixCraft&utm_campaign=Apr_22_2022_EOP "How To Set Up a Firewall with Awall on Alpine Linux") • [CentOS 8](https://www.cyberciti.biz/faq/how-to-set-up-a-firewall-using-firewalld-on-centos-8/?utm_source=Cmd_Table&utm_medium=nixCraft&utm_campaign=Apr_22_2022_EOP "How to set up a firewall using FirewallD on CentOS 8") • [OpenSUSE](https://www.cyberciti.biz/faq/set-up-a-firewall-using-firewalld-on-opensuse-linux/?utm_source=Cmd_Table&utm_medium=nixCraft&utm_campaign=Apr_22_2022_EOP "How to set up a firewall using FirewallD on OpenSUSE Linux") • [RHEL 8](https://www.cyberciti.biz/faq/configure-set-up-a-firewall-using-firewalld-on-rhel-8/?utm_source=Cmd_Table&utm_medium=nixCraft&utm_campaign=Apr_22_2022_EOP "How to set up a firewall using FirewallD on RHEL 8") • [Ubuntu 16.04](https://www.cyberciti.biz/faq/howto-configure-setup-firewall-with-ufw-on-ubuntu-linux/?utm_source=Cmd_Table&utm_medium=nixCraft&utm_campaign=Apr_22_2022_EOP "How to set up a UFW firewall on Ubuntu 16.04 LTS server") • [Ubuntu 18.04](https://www.cyberciti.biz/faq/how-to-setup-a-ufw-firewall-on-ubuntu-18-04-lts-server/?utm_source=Cmd_Table&utm_medium=nixCraft&utm_campaign=Apr_22_2022_EOP "How to setup a UFW firewall on Ubuntu 18.04 LTS server") • [Ubuntu 20.04](https://www.cyberciti.biz/faq/how-to-configure-firewall-with-ufw-on-ubuntu-20-04-lts/?utm_source=Cmd_Table&utm_medium=nixCraft&utm_campaign=Apr_22_2022_EOP "How To Configure Firewall with UFW on Ubuntu 20.04 LTS") |
| KVM Virtualization | [CentOS/RHEL 7](https://www.cyberciti.biz/faq/how-to-install-kvm-on-centos-7-rhel-7-headless-server/?utm_source=Cmd_Table&utm_medium=nixCraft&utm_campaign=Apr_22_2022_EOP "How to install KVM on CentOS 7 / RHEL 7 Headless Server") • [CentOS/RHEL 8](https://www.cyberciti.biz/faq/how-to-install-kvm-on-centos-8-headless-server/?utm_source=Cmd_Table&utm_medium=nixCraft&utm_campaign=Apr_22_2022_EOP "How to install KVM on CentOS 8 Headless Server") • [Debian 9/10/11](https://www.cyberciti.biz/faq/install-kvm-server-debian-linux-9-headless-server/?utm_source=Cmd_Table&utm_medium=nixCraft&utm_campaign=Apr_22_2022_EOP "How to install KVM server on Debian 9/10 Headless Server") • [Ubuntu 20.04](https://www.cyberciti.biz/faq/how-to-install-kvm-on-ubuntu-20-04-lts-headless-server/?utm_source=Cmd_Table&utm_medium=nixCraft&utm_campaign=Apr_22_2022_EOP "How to install KVM on Ubuntu 20.04 LTS Headless Server") |
| Linux Desktop apps | [Chrome](https://www.cyberciti.biz/faq/how-to-install-google-chrome-in-ubuntu-linux-12-xx-13-xx/?utm_source=Cmd_Table&utm_medium=nixCraft&utm_campaign=Apr_22_2022_EOP "Ubuntu Linux: Install Google Chrome Browser Command") • [Chromium](https://www.cyberciti.biz/faq/install-chromium-browser-on-ubuntu-linux/?utm_source=Cmd_Table&utm_medium=nixCraft&utm_campaign=Apr_22_2022_EOP "How to install Chromium browser on Ubuntu Linux") • [GIMP](https://www.cyberciti.biz/faq/how-to-install-gimp-on-ubuntu-debian-linux/?utm_source=Cmd_Table&utm_medium=nixCraft&utm_campaign=Apr_22_2022_EOP "How to install GIMP 2.10 on Ubuntu or Debian Linux") • [Skype](https://www.cyberciti.biz/faq/how-to-install-skype-application-on-linux/?utm_source=Cmd_Table&utm_medium=nixCraft&utm_campaign=Apr_22_2022_EOP "How to install Skype application on Linux") • [Spotify](https://www.cyberciti.biz/faq/how-to-install-spotify-application-on-linux/?utm_source=Cmd_Table&utm_medium=nixCraft&utm_campaign=Apr_22_2022_EOP "How to install Spotify application on Linux") • [VLC 3](https://www.cyberciti.biz/faq/how-to-install-vlc-3-application-vetinari-on-linux/?utm_source=Cmd_Table&utm_medium=nixCraft&utm_campaign=Apr_22_2022_EOP "How to install VLC 3 application (Vetinari) on Linux") |
| Modern utilities | [bat](https://www.cyberciti.biz/open-source/bat-linux-command-a-cat-clone-with-written-in-rust/?utm_source=Cmd_Table&utm_medium=nixCraft&utm_campaign=Apr_22_2022_EOP "bat Linux command - A cat clone with written in Rust") • [exa](https://www.cyberciti.biz/open-source/command-line-hacks/exa-a-modern-replacement-for-ls-written-in-rust-for-linuxunix/?utm_source=Cmd_Table&utm_medium=nixCraft&utm_campaign=Apr_22_2022_EOP "exa a modern replacement for ls command in rust for Linux/Unix") |
| Network Utilities | [NetHogs](https://www.cyberciti.biz/faq/linux-find-out-what-process-is-using-bandwidth/?utm_source=Cmd_Table&utm_medium=nixCraft&utm_campaign=Apr_22_2022_EOP "Linux See Bandwidth Usage Per Process With Nethogs Tool") • [dig](https://www.cyberciti.biz/faq/linux-unix-dig-command-examples-usage-syntax/?utm_source=Cmd_Table&utm_medium=nixCraft&utm_campaign=Apr_22_2022_EOP "Linux and Unix dig Command Examples") • [host](https://www.cyberciti.biz/faq/linux-unix-host-command-examples-usage-syntax/?utm_source=Cmd_Table&utm_medium=nixCraft&utm_campaign=Apr_22_2022_EOP "Linux and Unix host Command Examples") • [ip](https://www.cyberciti.biz/faq/linux-ip-command-examples-usage-syntax/?utm_source=Cmd_Table&utm_medium=nixCraft&utm_campaign=Apr_22_2022_EOP "Linux ip Command Examples") • [nmap](https://www.cyberciti.biz/security/nmap-command-examples-tutorials/?utm_source=Cmd_Table&utm_medium=nixCraft&utm_campaign=Apr_22_2022_EOP "Nmap Command Examples For Linux Sys/Network Admins") • [ping](https://www.cyberciti.biz/faq/unix-ping-command-examples/?utm_source=Cmd_Table&utm_medium=nixCraft&utm_campaign=Apr_22_2022_EOP "UNIX ping Command Examples") |
| OpenVPN | [CentOS 7](https://www.cyberciti.biz/faq/centos-7-0-set-up-openvpn-server-in-5-minutes/?utm_source=Cmd_Table&utm_medium=nixCraft&utm_campaign=Apr_22_2022_EOP "CentOS 7 Set Up OpenVPN Server In 5 Minutes") • [CentOS 8](https://www.cyberciti.biz/faq/centos-8-set-up-openvpn-server-in-5-minutes/?utm_source=Cmd_Table&utm_medium=nixCraft&utm_campaign=Apr_22_2022_EOP "CentOS 8 Set Up OpenVPN Server In 5 Minutes") • [Debian 10](https://www.cyberciti.biz/faq/debian-10-set-up-openvpn-server-in-5-minutes/?utm_source=Cmd_Table&utm_medium=nixCraft&utm_campaign=Apr_22_2022_EOP "Debian 10 Set Up OpenVPN Server In 5 Minutes") • [Debian 11](https://www.cyberciti.biz/faq/debian-11-set-up-openvpn-server-in-5-minutes/?utm_source=Cmd_Table&utm_medium=nixCraft&utm_campaign=Apr_22_2022_EOP "Debian 11 Set Up OpenVPN Server In 5 Minutes") • [Debian 8/9](https://www.cyberciti.biz/faq/install-configure-openvpn-server-on-debian-9-linux/?utm_source=Cmd_Table&utm_medium=nixCraft&utm_campaign=Apr_22_2022_EOP "Install and Configure an OpenVPN on Debian 9 In 5 Minutes") • [Ubuntu 18.04](https://www.cyberciti.biz/faq/ubuntu-18-04-lts-set-up-openvpn-server-in-5-minutes/?utm_source=Cmd_Table&utm_medium=nixCraft&utm_campaign=Apr_22_2022_EOP "Ubuntu 18.04 LTS Set Up OpenVPN Server In 5 Minutes") • [Ubuntu 20.04](https://www.cyberciti.biz/faq/ubuntu-20-04-lts-set-up-openvpn-server-in-5-minutes/?utm_source=Cmd_Table&utm_medium=nixCraft&utm_campaign=Apr_22_2022_EOP "Ubuntu 20.04 LTS Set Up OpenVPN Server In 5 Minutes") |
| Power Management | [upower](https://www.cyberciti.biz/faq/linux-upower-command-examples-and-syntax/?utm_source=Cmd_Table&utm_medium=nixCraft&utm_campaign=Apr_22_2022_EOP "upower command in Linux with examples") |
| Package Manager | [apk](https://www.cyberciti.biz/faq/10-alpine-linux-apk-command-examples/?utm_source=Cmd_Table&utm_medium=nixCraft&utm_campaign=Apr_22_2022_EOP "10 Alpine Linux apk Command Examples") • [apt-get](https://www.cyberciti.biz/tips/linux-debian-package-management-cheat-sheet.html?utm_source=Cmd_Table&utm_medium=nixCraft&utm_campaign=Apr_22_2022_EOP "Ubuntu/Debian Linux apt-get package management cheat sheet") • [apt](https://www.cyberciti.biz/faq/ubuntu-lts-debian-linux-apt-command-examples/?utm_source=Cmd_Table&utm_medium=nixCraft&utm_campaign=Apr_22_2022_EOP "apt Command Examples for Ubuntu/Debian for new users") • [yum](https://www.cyberciti.biz/faq/rhel-centos-fedora-linux-yum-command-howto/?utm_source=Cmd_Table&utm_medium=nixCraft&utm_campaign=Apr_22_2022_EOP "How to use yum command on CentOS/RHEL") |
| Processes Management | [bg](https://www.cyberciti.biz/faq/unix-linux-bg-command-examples-usage-syntax/?utm_source=Cmd_Table&utm_medium=nixCraft&utm_campaign=Apr_22_2022_EOP "Linux / Unix: bg Command Examples") • [chroot](https://www.cyberciti.biz/faq/unix-linux-chroot-command-examples-usage-syntax/?utm_source=Cmd_Table&utm_medium=nixCraft&utm_campaign=Apr_22_2022_EOP "Linux / Unix: chroot Command Examples") • [cron](https://www.cyberciti.biz/faq/how-do-i-add-jobs-to-cron-under-linux-or-unix-oses/?utm_source=Cmd_Table&utm_medium=nixCraft&utm_campaign=Apr_22_2022_EOP "How To Add Jobs To cron Under Linux or UNIX") • [disown](https://www.cyberciti.biz/faq/unix-linux-disown-command-examples-usage-syntax/?utm_source=Cmd_Table&utm_medium=nixCraft&utm_campaign=Apr_22_2022_EOP "Linux / Unix: disown Command Examples") • [fg](https://www.cyberciti.biz/faq/unix-linux-fg-command-examples-usage-syntax/?utm_source=Cmd_Table&utm_medium=nixCraft&utm_campaign=Apr_22_2022_EOP "Linux / Unix: fg Command Examples") • [glances](https://www.cyberciti.biz/faq/linux-install-glances-monitoring-tool/?utm_source=Cmd_Table&utm_medium=nixCraft&utm_campaign=Apr_22_2022_EOP "Linux: Keep An Eye On Your System With Glances Monitor") • [gtop](https://www.cyberciti.biz/howto/gtop-awesome-system-monitoring-dashboard-for-terminal/?utm_source=Cmd_Table&utm_medium=nixCraft&utm_campaign=Apr_22_2022_EOP "gtop: Awesome system monitoring dashboard for Linux/macOS Unix terminal") • [iotop](https://www.cyberciti.biz/hardware/linux-iotop-simple-top-like-io-monitor/?utm_source=Cmd_Table&utm_medium=nixCraft&utm_campaign=Apr_22_2022_EOP "Linux iotop Check Whats Stressing & Increasing Load On Hard Disks") • [jobs](https://www.cyberciti.biz/faq/unix-linux-jobs-command-examples-usage-syntax/?utm_source=Cmd_Table&utm_medium=nixCraft&utm_campaign=Apr_22_2022_EOP "Linux / Unix: jobs Command Examples") • [killall](https://www.cyberciti.biz/faq/unix-linux-killall-command-examples-usage-syntax/?utm_source=Cmd_Table&utm_medium=nixCraft&utm_campaign=Apr_22_2022_EOP "Linux / Unix: killall Command Examples") • [kill](https://www.cyberciti.biz/faq/unix-kill-command-examples/?utm_source=Cmd_Table&utm_medium=nixCraft&utm_campaign=Apr_22_2022_EOP "Linux / UNIX: Kill Command Examples") • [pidof](https://www.cyberciti.biz/faq/linux-pidof-command-examples-find-pid-of-program/?utm_source=Cmd_Table&utm_medium=nixCraft&utm_campaign=Apr_22_2022_EOP "Linux pidof Command Examples To Find PID of A Program/Command") • [pstree](https://www.cyberciti.biz/faq/unix-linux-pstree-command-examples-shows-running-processestree/?utm_source=Cmd_Table&utm_medium=nixCraft&utm_campaign=Apr_22_2022_EOP "Linux/Unix: pstree Command Examples: See A Tree Of Processes") • [pwdx](https://www.cyberciti.biz/faq/unix-linux-pwdx-command-examples-usage-syntax/?utm_source=Cmd_Table&utm_medium=nixCraft&utm_campaign=Apr_22_2022_EOP "Linux / Unix: pwdx Command Examples") • [time](https://www.cyberciti.biz/faq/unix-linux-time-command-examples-usage-syntax/?utm_source=Cmd_Table&utm_medium=nixCraft&utm_campaign=Apr_22_2022_EOP "Linux / Unix: time Command Examples") • [vtop](https://www.cyberciti.biz/faq/how-to-install-and-use-vtop-graphical-terminal-activity-monitor-on-linux/?utm_source=Cmd_Table&utm_medium=nixCraft&utm_campaign=Apr_22_2022_EOP "How to install and use vtop graphical terminal monitor on Linux") |
| Searching | [ag](https://www.cyberciti.biz/open-source/command-line-hacks/ag-supercharge-string-search-through-directory-hierarchy/?utm_source=Cmd_Table&utm_medium=nixCraft&utm_campaign=Apr_22_2022_EOP "ag - supercharge string search through a directory on a Linux") • [egrep](https://www.cyberciti.biz/faq/grep-regular-expressions/?utm_source=Cmd_Table&utm_medium=nixCraft&utm_campaign=Apr_22_2022_EOP "Regular expressions in grep ( regex ) with examples") • [grep](https://www.cyberciti.biz/faq/howto-use-grep-command-in-linux-unix/?utm_source=Cmd_Table&utm_medium=nixCraft&utm_campaign=Apr_22_2022_EOP "How to use grep command in Linux/ Unix with examples") • [whereis](https://www.cyberciti.biz/faq/unix-linux-whereis-command-examples-to-locate-binary/?utm_source=Cmd_Table&utm_medium=nixCraft&utm_campaign=Apr_22_2022_EOP "Linux / Unix whereis Command Examples") • [which](https://www.cyberciti.biz/faq/unix-linux-which-command-examples-syntax-to-locate-programs/?utm_source=Cmd_Table&utm_medium=nixCraft&utm_campaign=Apr_22_2022_EOP "Linux / Unix which Command Examples To Locate a Command") |
| Shell builtins | [compgen](https://www.cyberciti.biz/open-source/command-line-hacks/compgen-linux-command/?utm_source=Cmd_Table&utm_medium=nixCraft&utm_campaign=Apr_22_2022_EOP "compgen: An Awesome Command To List All Linux Commands") • [echo](https://bash.cyberciti.biz/guide/Echo_Command?utm_source=Cmd_Table&utm_medium=nixCraft&utm_campaign=Apr_22_2022_EOP "Echo Command - Linux Bash Shell Scripting Tutorial") • [printf](https://bash.cyberciti.biz/guide/Printf_command?utm_source=Cmd_Table&utm_medium=nixCraft&utm_campaign=Apr_22_2022_EOP "Printf command - Linux Bash Shell Scripting Tutorial") |
| System Management | [reboot](https://www.cyberciti.biz/faq/howto-reboot-linux/?utm_source=Cmd_Table&utm_medium=nixCraft&utm_campaign=Apr_22_2022_EOP "Reboot Linux System Command") • [shutdown](https://www.cyberciti.biz/faq/howto-shutdown-linux/?utm_source=Cmd_Table&utm_medium=nixCraft&utm_campaign=Apr_22_2022_EOP "How To Shutdown Linux Using Command Line") |
| Terminal/ssh | [tty](https://www.cyberciti.biz/faq/linux-unix-appleosx-bsd-what-tty-command/?utm_source=Cmd_Table&utm_medium=nixCraft&utm_campaign=Apr_22_2022_EOP "Find Out What tty Im Using tty command under Linux / Unix") |
| Text processing | [cut](https://bash.cyberciti.biz/guide/Cut_command?utm_source=Cmd_Table&utm_medium=nixCraft&utm_campaign=Apr_22_2022_EOP "Cut command - Linux Bash Shell Scripting Tutorial") • [rev](https://bash.cyberciti.biz/guide/Rev_command?utm_source=Cmd_Table&utm_medium=nixCraft&utm_campaign=Apr_22_2022_EOP "Rev command - Linux Bash Shell Scripting Tutorial") |
| User Environment | [exit](https://bash.cyberciti.biz/guide/Exit_command?utm_source=Cmd_Table&utm_medium=nixCraft&utm_campaign=Apr_22_2022_EOP "Exit command - Linux Bash Shell Scripting Tutorial") • [who](https://www.cyberciti.biz/faq/unix-linux-who-command-examples-syntax-usage/?utm_source=Cmd_Table&utm_medium=nixCraft&utm_campaign=Apr_22_2022_EOP "Linux / Unix who Command Examples To List Users on The Systems") |
| User Information | [groups](https://www.cyberciti.biz/faq/unix-linux-groups-command-examples-syntax-usage/?utm_source=Cmd_Table&utm_medium=nixCraft&utm_campaign=Apr_22_2022_EOP "Linux / Unix: groups Command Examples") • [id](https://www.cyberciti.biz/faq/unix-linux-id-command-examples-usage-syntax/?utm_source=Cmd_Table&utm_medium=nixCraft&utm_campaign=Apr_22_2022_EOP "Linux / Unix id Command Examples") • [lastcomm](https://www.cyberciti.biz/faq/linux-unix-lastcomm-command-examples-usage-syntax/?utm_source=Cmd_Table&utm_medium=nixCraft&utm_campaign=Apr_22_2022_EOP "Linux / Unix: lastcomm Command Examples") • [last](https://www.cyberciti.biz/faq/linux-unix-last-command-examples/?utm_source=Cmd_Table&utm_medium=nixCraft&utm_campaign=Apr_22_2022_EOP "Linux / Unix last Command Examples") • [lid/libuser-lid](https://www.cyberciti.biz/faq/linux-lid-command-examples-syntax-usage/?utm_source=Cmd_Table&utm_medium=nixCraft&utm_campaign=Apr_22_2022_EOP "Linux lid (libuser-lid) Command Examples") • [logname](https://www.cyberciti.biz/faq/unix-linux-logname-command-examples-syntax-usage/?utm_source=Cmd_Table&utm_medium=nixCraft&utm_campaign=Apr_22_2022_EOP "Linux / Unix: logname Command Examples To Display Loginname") • [members](https://www.cyberciti.biz/faq/linux-members-command-examples-usage-syntax/?utm_source=Cmd_Table&utm_medium=nixCraft&utm_campaign=Apr_22_2022_EOP "Linux members Command Examples") • [users](https://www.cyberciti.biz/faq/unix-linux-users-command-examples-syntax-usage/?utm_source=Cmd_Table&utm_medium=nixCraft&utm_campaign=Apr_22_2022_EOP "Linux / Unix: users Command Examples") • [whoami](https://www.cyberciti.biz/faq/unix-linux-whoami-command-examples-syntax-usage/?utm_source=Cmd_Table&utm_medium=nixCraft&utm_campaign=Apr_22_2022_EOP "Linux / Unix: whoami Command Examples") • [w](https://www.cyberciti.biz/faq/unix-linux-w-command-examples-syntax-usage-2/?utm_source=Cmd_Table&utm_medium=nixCraft&utm_campaign=Apr_22_2022_EOP "Linux / Unix: w Command Examples ") |
| User Management | [/etc/group](https://www.cyberciti.biz/faq/understanding-etcgroup-file/?utm_source=Cmd_Table&utm_medium=nixCraft&utm_campaign=Apr_22_2022_EOP "Understanding /etc/group File") • [/etc/passwd](https://www.cyberciti.biz/faq/understanding-etcpasswd-file-format/?utm_source=Cmd_Table&utm_medium=nixCraft&utm_campaign=Apr_22_2022_EOP "Understanding /etc/passwd File Format") • [/etc/shadow](https://www.cyberciti.biz/faq/understanding-etcshadow-file/?utm_source=Cmd_Table&utm_medium=nixCraft&utm_campaign=Apr_22_2022_EOP "Understanding /etc/shadow file format on Linux") |
| WireGuard VPN | [Alpine](https://www.cyberciti.biz/faq/how-to-set-up-wireguard-vpn-server-on-alpine-linux/?utm_source=Cmd_Table&utm_medium=nixCraft&utm_campaign=Apr_22_2022_EOP "Alpine Linux set up WireGuard VPN server") • [CentOS 8](https://www.cyberciti.biz/faq/centos-8-set-up-wireguard-vpn-server/?utm_source=Cmd_Table&utm_medium=nixCraft&utm_campaign=Apr_22_2022_EOP "CentOS 8 set up WireGuard VPN server") • [Debian 10](https://www.cyberciti.biz/faq/debian-10-set-up-wireguard-vpn-server/?utm_source=Cmd_Table&utm_medium=nixCraft&utm_campaign=Apr_22_2022_EOP "Debian 10 set up WireGuard VPN server") • [Firewall](https://www.cyberciti.biz/faq/how-to-set-up-wireguard-firewall-rules-in-linux/?utm_source=Cmd_Table&utm_medium=nixCraft&utm_campaign=Apr_22_2022_EOP "How To Set Up WireGuard Firewall Rules in Linux") • [Ubuntu 20.04](https://www.cyberciti.biz/faq/ubuntu-20-04-set-up-wireguard-vpn-server/?utm_source=Cmd_Table&utm_medium=nixCraft&utm_campaign=Apr_22_2022_EOP "How to set up WireGuard VPN server on Ubuntu 20.04") • [qrencode](https://www.cyberciti.biz/faq/how-to-generate-wireguard-qr-code-on-linux-for-mobile/?utm_source=Cmd_Table&utm_medium=nixCraft&utm_campaign=Apr_22_2022_EOP "How to generate WireGuard QR code on Linux for mobile") |