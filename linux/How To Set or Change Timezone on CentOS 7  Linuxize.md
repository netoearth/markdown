On CentOS, the system’s timezone is set during the install, but it can be easily changed at a later time.

Using the correct timezone is important for many systems related tasks and processes. For example, the cron daemon uses the system’s timezone for executing cron jobs, and the timestamps in the log files are based on the same system’s timezone.

This tutorial explains how to set or change the timezone on CentOS 7.

## Prerequisites

To be able change the system’s timezone you’ll need to be logged in as root or [user with sudo privileges](https://linuxize.com/post/create-a-sudo-user-on-centos/) .

## Checking the Current Timezone

In CentOS and other modern Linux distros, you can use the `timedatectl` command to display and set the current system’s time and timezone.

```
timedatectl
```

The output below shows that the system’s timezone is set to UTC:

```
      Local time: Wed 2019-02-06 22:43:42 UTC
  Universal time: Wed 2019-02-06 22:43:42 UTC
        RTC time: Wed 2019-02-06 22:43:42
       Time zone: Etc/UTC (UTC, +0000)
     NTP enabled: no
NTP synchronized: yes
 RTC in local TZ: no
      DST active: n/a
```

The system timezone is configured by symlinking `/etc/localtime` to a binary timezone identifier in the `/usr/share/zoneinfo` directory. So, another option to check the timezone is to show the path the symlink points to using the [ls command](https://linuxize.com/post/how-to-list-files-in-linux-using-the-ls-command/) :

```
ls -l /etc/localtime
```

```
lrwxrwxrwx. 1 root root 29 Dec 11 09:25 /etc/localtime -> ../usr/share/zoneinfo/Etc/UTC
```

## Changing Timezone in CentOS

Before changing the timezone, you’ll need to find out the long name for the timezone you want to use. The timezone naming convention usually uses a “Region/City” format.

To list all available time zones, you can either list the files in the `/usr/share/zoneinfo` directory or use the `timedatectl` command.

```
timedatectl list-timezones
```

```
...
America/Tijuana
America/Toronto
America/Tortola
America/Vancouver
America/Whitehorse
America/Winnipeg
...
```

Once you identify which time zone is accurate to your location, run the following command as sudo user:

```
sudo timedatectl set-timezone your_time_zone
```

For example, to change the system’s timezone to `America/Toronto`:

```
sudo timedatectl set-timezone America/Toronto
```

Run the `timedatectl` command to verify the changes:

```
timedatectl
```

```
      Local time: Wed 2019-02-06 17:47:10 EST
  Universal time: Wed 2019-02-06 22:47:10 UTC
        RTC time: Wed 2019-02-06 22:47:10
       Time zone: America/Toronto (EST, -0500)
     NTP enabled: no
NTP synchronized: yes
 RTC in local TZ: no
      DST active: no
 Last DST change: DST ended at
                  Sun 2018-11-04 01:59:59 EDT
                  Sun 2018-11-04 01:00:00 EST
 Next DST change: DST begins (the clock jumps one hour forward) at
                  Sun 2019-03-10 01:59:59 EST
                  Sun 2019-03-10 03:00:00 EDT
```

## Changing the Timezone by Creating a Symlink

If you are running an older [version of CentOS](https://linuxize.com/post/how-to-check-your-centos-version/) and the `timedatectl` command is not present on your system, you can change the timezone by symlinking `/etc/localtime` to the timezone file in the `/usr/share/zoneinfo` directory.

Delete the current `/etc/localtime` file or symlink:

```
sudo rm -rf /etc/localtime
```

Identify the timezone you want to configure and [create a symlink](https://linuxize.com/post/how-to-create-symbolic-links-in-linux-using-the-ln-command/) :

```
sudo ln -s /usr/share/zoneinfo/America/Toronto /etc/localtime
```

You can verify it either by listing the `/etc/localtime` file or issuing the `date` command:

```
date
```

```
Wed Feb  6 17:52:58 EST 2019
```

## Conclusion

In this guide, we have shown you how change your CentOS system’s timezone.

Feel free to leave a comment if you have any questions.