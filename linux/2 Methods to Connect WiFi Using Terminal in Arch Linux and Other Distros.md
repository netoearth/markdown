**This quick guide explains the steps you need to set up and connect to WiFi using a terminal in Arch Linux and other distros.**

This guide is ideal for those scenarios where you are stuck with a terminal without any GUI and no other wired internet connectivity is available. These steps help you to manually detect the wireless card and device and connect to the WiFi hotspot with password authentication via the terminal.

This guide uses [iwd](https://wiki.archlinux.org/index.php/Iwd) (Net Wireless Daemon) and nmcli to connect to WiFi via a terminal.

-   [Connect to WiFi Using Terminal in Arch Linux and Other Distros](https://www.debugpoint.com/connect-wifi-terminal-linux/#Connect_to_WiFi_Using_Terminal_in_Arch_Linux_and_Other_Distros "Connect to WiFi Using Terminal in Arch Linux and Other Distros")
-   [Method 1: Using nmcli](https://www.debugpoint.com/connect-wifi-terminal-linux/#Method_1_Using_nmcli "Method 1: Using nmcli")
-   [Method 2: Using iwd](https://www.debugpoint.com/connect-wifi-terminal-linux/#Method_2_Using_iwd "Method 2: Using iwd")
    -   [1\. Setup iwd](https://www.debugpoint.com/connect-wifi-terminal-linux/#1_Setup_iwd "1. Setup iwd")
    -   [2\. Configure](https://www.debugpoint.com/connect-wifi-terminal-linux/#2_Configure "2. Configure")
    -   [Connect](https://www.debugpoint.com/connect-wifi-terminal-linux/#Connect "Connect")
-   [Usage Guides](https://www.debugpoint.com/connect-wifi-terminal-linux/#Usage_Guides "Usage Guides")
-   [Wrapping Up](https://www.debugpoint.com/connect-wifi-terminal-linux/#Wrapping_Up "Wrapping Up")

## Method 1: Using nmcli

In the first method, try using connecting nmcli. The nmcli is a command-line tool used to create, display, edit, delete, activate, and deactivate network connections and control and display network device status.

It’s part of the `networkmanager` package. The following method will only work if you have NetworkManager installed. Make sure you have enabled any mobile hotspot or WiFi connection.

To connect to WiFi using the terminal, type the following:

```
nmcli device wifi list
```

It should give you the list of access points.

If it’s not showing your wifi access point, try to rescan again:

```
nmcli device wifi rescan
```

![nmcli device wifi list](https://www.debugpoint.com/wp-content/uploads/2022/12/nmcli-device-wifi-list.jpg)

nmcli device wifi list

And use the list command to see if your connection is visible. Once it shows, use the following command to connect with the user id and password.

```
nmcli device wifi connect access_point_name password your_password
```

Replace the **access\_point\_name with the SSID of your connection** in the above command and use your password. And you should be connected.

To verify the connection, run the following command:

```
nmcli connection show
```

[![nmcli connection show](https://www.debugpoint.com/wp-content/uploads/2022/12/nmcli-connection-show2.jpg)](https://www.debugpoint.com/wp-content/uploads/2022/12/nmcli-connection-show2.jpg)

nmcli connection show

You should see the connected access point in a different font/colour.

Now, you can carry on with your task.

## Method 2: Using iwd

### 1\. Setup iwd

The `iwd` package comes with three main modules:

**iwctl** : The wireless client  
**iwd**: The Daemon  
**iwmon** : Monitoring tool

On the terminal type –

```
iwctl
```

![iwctl Prompt](https://www.debugpoint.com/wp-content/uploads/2020/11/iwctl-Prompt.jpg)

iwctl Prompt

If you get a command not found, then you need to download the package from [here](https://www.archlinux.org/packages/?name=iwd).

So get help from any other system/laptop with an internet connection to download and install the package via mounting the USB.

Alternatively, if you have a USB dongle with the internet, then plugin that into your system. And install via the below commands.

The USB dongle should work out of the box in Arch and most Linux systems today to connect to the internet.

**Arch**

```
pacman -S iwd
```

**Debian, Ubuntu, and other similar distributions**

```
sudo apt-get install iwd
```

**Fedora**

```
sudo dnf install iwd
```

If you get an `iwctl` prompt (like below), then proceed to the next step.

### 2\. Configure

Run the below command to get your system’s **wireless device name**.

```
device list
```

[![iwctl - device list](https://www.debugpoint.com/wp-content/uploads/2020/11/iwctl-device-list-2.jpg)](https://www.debugpoint.com/wp-content/uploads/2020/11/iwctl-device-list-2.jpg)

iwctl – device list

To **get the list of WiFi networks**, run the below command. Replace `wlan0` with your device name on the below command and all the following commands.

```
station wlan0 get-networks
```

[![iwctl - available networks](https://www.debugpoint.com/wp-content/uploads/2020/11/iwctl-available-networks.jpg)](https://www.debugpoint.com/wp-content/uploads/2020/11/iwctl-available-networks.jpg)

iwctl – available networks

The command gives you the list of available WiFi network with security type and signal strength.

### Connect

To **connect to the WiFi networ**k, run the below command with the WiFi access point name from the above “get-networks” command.

```
station wlan0 connect
```

Enter your WiFi password when prompted.

[![connect to WiFi using iwctl](https://www.debugpoint.com/wp-content/uploads/2020/11/connect-to-WiFi-using-iwctl.jpg)](https://www.debugpoint.com/wp-content/uploads/2020/11/connect-to-WiFi-using-iwctl.jpg)

connect to WiFi using iwctl

If all, goes well you should be connected to the internet.

## Usage Guides

-   You can **check the connection** using a simple ping command as follows. The ping replies successful packet transfers for a stable connection.

```
ping -c 3 google.com
```

-   You can also check the **status of the connection** using the below command.

```
station wlan0 show
```

-   The iwd keeps the configuration file at `/var/lib/iwd` as a `.psk` file with your access point name.

-   This file contains a hash file that is generated using the password and SSID of your WiFi network.

-   Press `CTRL+D` to leave from the `iwctl` prompt.

## Wrapping Up

I hope this guide helps you to connect to the internet via the terminal. This helps when you have no other way to connect to WiFi. For example, if you are installing Arch Linux in a stand-alone system (not a VM), you need to connect to the internet to download packages via a terminal using `pacman`.

If you face any trouble, mention the error messages in the comment box below.