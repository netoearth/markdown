[When you install Ubuntu](https://itsfoss.com/install-ubuntu/), it asks you to set timezone. If you chose a wrong timezone or if you have moved to some other part of the world, you can easily change it later.

## How to change Timezone in Ubuntu and other Linux distributions

There are two ways to change the timezone in Ubuntu. You can use the graphical settings or use the timedatectl command in the terminal. You may also change the /etc/timezone file directly but I won’t advise that.

I’ll show you both graphical and terminal way in this beginner’s tutorial:

-   [Change timezone in Ubuntu via GUI](https://itsfoss.com/change-timezone-ubuntu/#change-timezone-gui) (suitable for desktop users)
-   [Change timezone in Ubuntu via command line](https://itsfoss.com/change-timezone-ubuntu/#change-timezone-command-line) (works for both desktop and servers)

![How to  Change Time Zone in Ubuntu](https://itsfoss.com/wp-content/uploads/2020/01/Ubuntu_Change-_Time_Zone.png)

### Method 1: Change Ubuntu timezone via terminal

[Ubuntu](https://ubuntu.com/) or any other distributions using systemd can use the timedatectl command to set timezone in Linux terminal.

You can check the current date and timezone setting using timedatectl command without any option:

```
abhishek@nuc:~$ timedatectl 
                      Local time: Sat 2020-01-18 17:39:52 IST
                  Universal time: Sat 2020-01-18 12:09:52 UTC
                        RTC time: Sat 2020-01-18 12:09:52
                       Time zone: Asia/Kolkata (IST, +0530)
       System clock synchronized: yes
systemd-timesyncd.service active: yes
                 RTC in local TZ: no
```

As you can see in the output above, my system uses Asia/Kolkata. It also tells me that it is 5:30 hours ahead of GMT.

To set a timezone in Linux, you need to know the exact timezone. You must use the correct format of the timezone (which is Continent/City).

To get the timezone list, use the _list-timezones_ option of _timedatectl_ command:

```
timedatectl list-timezones
```

It will show you a huge list of the available time zones.

![Timezones In Ubuntu](https://itsfoss.com/wp-content/uploads/2020/01/timezones_in_ubuntu.jpg)

Timezones List

You can use the up and down arrow or PgUp and PgDown key to move between the pages.

You may also grep the output and search for your timezone. For example, if you are looking for time zones in Europe, you may use:

```
timedatectl list-timezones | grep -i europe
```

Let’s say you want to set the timezone to Paris. The timezone value to be used here is Europe/Paris:

```
timedatectl set-timezone Europe/Paris
```

It won’t show any success message but the timezone is changed instantly. You don’t need to restart or log out.

Keep in mind that though you don’t need to become root user and use sudo with the command but your account still need to have admin rights in order to change the timezone.

You can verify the changed time and timezone by using the [date command](https://linuxhandbook.com/date-command/):

```
abhishek@nuc:~$ date
Sat Jan 18 13:56:26 CET 2020
```

### Method 2: Change Ubuntu timezone via GUI

Press the super key (Windows key) and search for Settings:

![Applications Menu Settings](https://itsfoss.com/wp-content/uploads/2019/08/applications_menu_settings.jpg)

Applications Menu Settings

Scroll down a little and look for Details in the left sidebar:

![Settings Detail Ubuntu](https://itsfoss.com/wp-content/uploads/2020/01/settings_detail_ubuntu.jpg)

Go to Settings -> Details

In Details, you’ll fine Date & Time in the left sidebar. Here, you should turn off Automatic Time Zone option (if it is enabled) and then click on the Time Zone:

![Change Timezone In Ubuntu](https://itsfoss.com/wp-content/uploads/2020/01/change_timezone_in_ubuntu.jpg)

In Details -> Date & Time, turn off the Automatic Time Zone

When you click the Time Zone, it will open an interactive map and you can click on the geographical location of your choice or type the city name. Once you have selected the correct timezone, close the window.

![Set Timezone In Ubuntu](https://itsfoss.com/wp-content/uploads/2020/01/change_timezone_in_ubuntu_2.jpg)

Select a timezone

You don’t have to do anything other than closing this map after selecting the new timezone. No need to logout or [shutdown Ubuntu](https://itsfoss.com/schedule-shutdown-ubuntu/).

I hope this quick tutorial helped you to change timezone in Ubuntu and other Linux distributions. If you have questions or suggestions, please let me know.

Skip![arrow](https://sdk.apester.com/assets/iconRightArrow.svg)

Skip

Skip ad in

![arrow](https://sdk.apester.com/assets/iconRightArrow.svg)

![](https://itsfoss.com/wp-content/gravatars/2003563996b123e9460848e5ea27f4d6)

Creator of It's FOSS. An ardent Linux user & open source promoter. Huge fan of classic detective mysteries ranging from Agatha Christie and Sherlock Holmes to Detective Columbo & Ellery Queen. Also a movie buff with a soft corner for film noir.