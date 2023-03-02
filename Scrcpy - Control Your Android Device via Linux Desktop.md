_**Brief: This guide shows how to install scrcpy an application that helps you connect, display and control an android device from your Linux desktop computer.**_

**Scrcpy** (pronounced “**screen copy**“) is a free, open-source, and cross-platform application used to display and control an Android device from your Linux desktop computer. It works on Linux, Windows, and macOS, and allows you to control a device connected via a USB or wirelessly (over TCP/IP).

It features mirroring with Android device screen off, configurable screen display quality, recording, copy and paste in both directions, using an Android device as a webcam (Linux only), physical keyboard and mouse simulation, OTG mode, and much more.

To use **scrcpy**, your **Android** device must have at least API 21 (Android 5.0), and it must also have adb (Android Debug Bridge) debugging (USB debugging) enabled. But, it does not require root user access on Linux.

### Install Scrcpy on Linux Systems

On [Debian-based distributions](https://www.tecmint.com/debian-based-linux-distributions/ "Best Debian-based Linux Distributions") such as **Ubuntu** and **Linux Mint** systems, you can install **scrcpy** from the default repository as shown.

```
$ sudo apt install scrcpy

```

On **Fedora**, you can install it from the **Cool Other Packages Repository** (**COPR**), as follows:

```
$ sudo dnf copr enable zeno/scrcpy
$ sudo dnf install scrcpy

```

On **Arch Linux**, issue the following command:

```
# pacman -S scrcpy

```

**Scrcpy** is also available as a [snap](https://www.tecmint.com/install-snap-in-linux/ "A Beginners Guide to Snaps in Linux"), for example, to install it on [RHEL-based distributions](https://www.tecmint.com/redhat-based-linux-distributions/ "Best RedHat-based Linux Distributions"), run the following commands:

```
$ sudo yum install snapd
$ sudo systemctl enable --now snapd.socket
$ sudo ln -s /var/lib/snapd/snap /snap
$ sudo snap install scrcpy

```

### Connect to An Android Device via USB in Linux

Once the installation is complete, remember to enable **USB** debugging in your Android device (go to **Settings=>Developer** –> Options=>USB debugging) as mentioned before, then connect your device to your Linux desktop computer via a USB cable.

Next, a popup should open on the device to request authorization to allow USB debugging from the computer, and select **Allow** to proceed.

[![Allow USB Debugging](https://www.tecmint.com/wp-content/uploads/2023/01/Allow-USB-Debugging.jpg)](https://www.tecmint.com/wp-content/uploads/2023/01/Allow-USB-Debugging.jpg)

Allow USB Debugging

Then run the following command from the terminal to launch **scrcpy**:

```
$ scrcpy

```

If the command has run successfully, a window should open up displaying your device’s active screen as shown in the following screenshot.

[![Connect Android Device from Linux Desktop](https://www.tecmint.com/wp-content/uploads/2023/01/Connect-Android-Device-from-Linux-Desktop.jpg)](https://www.tecmint.com/wp-content/uploads/2023/01/Connect-Android-Device-from-Linux-Desktop.jpg)

Connect Android Device from Linux Desktop

### Connect to Android Device via Wifi in Linux Desktop

First, install the **adb** command-line tool on your computer as follows. If you already have the **adb** tool installed, skip the installation steps:

```
$ sudo apt install adb         [On Debian, Ubuntu and Mint]
$ sudo yum install adb         [On RHEL/CentOS/Fedora and Rocky Linux/AlmaLinux]
$ sudo pacman -S adb           [On Arch Linux]

```

Once the **adb** tool has been installed on your computer, connect your Android device and computer to a common Wi-Fi network. Then connect the Android device to the computer with a USB cable.

Next, disconnect the **USB** cable from the target device and find the IP address of the **Android** device (go to Settings –> Connections –> Wi-Fi –> Wi-Fi name –> tap on its settings) or run the following command to view the device IP address:

```
$ adb shell ip route

```

[![Find Android Device IP Address](https://www.tecmint.com/wp-content/uploads/2023/01/Find-Android-Device-IP-Address.png)](https://www.tecmint.com/wp-content/uploads/2023/01/Find-Android-Device-IP-Address.png)

Find Android Device IP Address

Then set the target **Android** device to listen for a **TCP/IP** connection on port **5555** by running the following command (check any prompt on the device):

```
$ adb tcpip 5555

```

Next, disconnect the USB cable and connect the target device using its IP address as shown:

```
$ adb connect 192.168.1.4:5555

```

Last but not least, run the **scrcpy** command to mirror the **Android** device’s screen on a Linux desktop:

```
$ scrcpy

```

[![Mirror Android Device in Linux Desktop](https://www.tecmint.com/wp-content/uploads/2023/01/Mirror-Android-Device-in-Linux-Desktop.png)](https://www.tecmint.com/wp-content/uploads/2023/01/Mirror-Android-Device-in-Linux-Desktop.png)

Mirror Android Device in Linux Desktop

### Scrcpy Command Examples with Options

From the previous screenshots, you can see that by default, **scrcpy** displays the device model as the window title. You can set a custom window title using the `--window-title` command-line option as shown (remember to replace ‘**My device**‘ with the title you prefer):

```
$ scrcpy  --window-title='My device'

```

To control the width and height of the mirrored Android screen, use the `--max-size` or `-m` switch as shown:

```
$ scrcpy -m 1024
OR
$ scrcpy --max-size=1024

```

**Scrcpy** also allows for screen recording while mirroring, using the `--record` or `-r` flag as shown:

```
$ scrcpy -r filename.mp4

```

If you wish to disable mirroring while recording, use the `--no-display` or `-N` flag as follows. Note that to stop the recording process, simply press `Ctrl+C`:

```
$ scrcpy -Nr  filename.mp4

```

To change the bit rate from the default of 8 Mbps, use the `--bit-rate` or `-b` option as shown:

```
$ scrcpy -b 4M

```

There are several other command-line options to control the behavior of **scrcpy**, run the following command to view a list of all of them:

```
$ scrcpy   --help

```

Last but not least, note that to control some Android devices using a keyboard and mouse, need to enable additional options. For more information, go to the [scrcpy Github repository](https://github.com/Genymobile/scrcpy "Scrcpy - Control Android Device from Linux").

## If You Appreciate What We Do Here On TecMint, You Should Consider:

TecMint is the fastest growing and most trusted community site for any kind of Linux Articles, Guides and Books on the web. Millions of people visit TecMint! to search or browse the thousands of published articles available FREELY to all.

If you like what you are reading, please consider buying us a coffee ( or 2 ) as a token of appreciation.

[![Support Us](https://www.tecmint.com/wp-content/uploads/2015/01/coffee.png)](https://www.buymeacoffee.com/tecmint)

**We are thankful for your never ending support.**