**psacct** or **acct** both are open source utilities for monitoring users’ activities on the Linux system. These utilities run in the background and keep track of each user’s activity on your system as well as what resources are being consumed.

I personally used these tools in our company, we have a development team where our developers continuously work on servers. So, these are the best utilities to keep an eye on them.

These programs provide an excellent way to monitor what users are doing, what commands are they executing, how many resources are being consumed by them, and how long users are active on the system. Another useful feature is, that it gives total resources consumed by services like Apache, MySQL, FTP, SSH, etc.

**\[ You might also like: [How to Monitor Linux Commands Executed by System Users in Real-time](https://www.tecmint.com/monitor-linux-commands-executed-by-system-users-in-real-time/ "Monitor Linux Commands Executed by Users") \]**

I think this is one of the great and most needed utilities for every Linux/Unix System Administrator, who wanted to keep a track of user activities on their servers/systems.

The **psacct** or **acct** package provides several features for monitoring process activities.

-   **ac** command prints the statistics of user logins/logouts (connect time) in hours.
-   **lastcomm** command prints the information of previously executed commands of the user.
-   **accton** commands is used to turn on/off process for accounting.
-   **sa** command summarizes information of previously executed commands.
-   **last** and **lastb** commands show a listing of [last logged-in users](https://www.tecmint.com/find-user-account-info-and-login-details-in-linux/ "Find User Account Info and Login Details in Linux").

### Installing psacct or acct Packages in Linux

**psacct** and **acct** both are similar packages and there is not much difference between them, but the **psacct** package is only available for [rpm-based distributions](https://www.tecmint.com/redhat-based-linux-distributions/ "The Best RedHat-based Linux Distributions") such as **RHEL**, **CentOS,** and **Fedora**, whereas the **acct** package is available for distributions like **Ubuntu**, **Debian,** and **Linux Mint**.

To install the **psacct** package under rpm-based distributions issue the following [yum command](https://www.tecmint.com/20-linux-yum-yellowdog-updater-modified-commands-for-package-mangement/ "Linux Yum Command Examples").

```
# yum install psacct

```

To install the **acct** package using the [apt command](https://www.tecmint.com/apt-advanced-package-command-examples-in-ubuntu/ "Ubuntu APT Command Examples") under **Ubuntu** / **Debian** / **Linux Mint**.

```
$ sudo apt install acct

```

On other Linux distributions, you can install it as shown.

```
$ sudo apk add psacct          [On Alpine Linux]
$ sudo pacman -S acct          [On Arch Linux]
$ sudo zypper install acct     [On OpenSUSE]    

```

##### Starting psacct or acct service

By default, the **psacct** service is in disabled mode and you need to start it manually under RHEL-based distributions. Use the following command to check the status of the service.

```
$ sudo systemctl status psacct

```

You see the status showing as disabled, so let’s start it manually using the following commands, which will create a **/var/account/pacct** file.

```
$ sudo systemctl start psacct
$ sudo systemctl enable psacct
$ sudo systemctl status psacct

```

[![Start psacct Service](https://www.tecmint.com/wp-content/uploads/2013/02/Start-psacct-Service.png)](https://www.tecmint.com/wp-content/uploads/2013/02/Start-psacct-Service.png)

Start psacct Service

Under **Ubuntu**, **Debian,** and **Mint** service is started automatically, you don’t need to start it again.

### Display Statistics of Users Connect Time

**ac** command without specifying any argument will display total statistics of connect time in hours based on the user logins/logouts from the current **wtmp** file.

```
# ac

total     11299.15

```

[![Print Total Connect Time of Linux User](https://www.tecmint.com/wp-content/uploads/2013/02/Print-User-Total-Connect-Time.png)](https://www.tecmint.com/wp-content/uploads/2013/02/Print-User-Total-Connect-Time.png)

Print Total Connect Time of Linux User

### Display Statistics of Linux Users Day-wise

Using the command “**ac -d**” will print out the total login time in hours by day-wise.

```
# ac -d

Jun 25total        0.19
Oct 13total       14.45
Oct 27total      672.00
Oct 28total       15.82
Nov  3total        4.29
Nov  5total       10.13
Dec  7total       14.04
Dec 10total       23.60
Dec 27total      808.93
Jan  3total       12.31
Mar  3total     1438.67
Jul 22total     6767.81
Todaytotal     1517.09

```

[![Print Linux User Total Login Time](https://www.tecmint.com/wp-content/uploads/2013/02/Print-User-Total-Login-Time.png)](https://www.tecmint.com/wp-content/uploads/2013/02/Print-User-Total-Login-Time.png)

Print Linux User Total Login Time

### Display Total Login Time of All Linux Users

Using the command “**ac -p**” will print the total login time of each Linux user in hours.

```
# ac -p

rockylinux                         425.61
tecmint                            702.29
root                             10171.54
total    11299.44

```

[![Print Total Login Time of Users](https://www.tecmint.com/wp-content/uploads/2013/02/Print-Total-Login-Time-of-Users.png)](https://www.tecmint.com/wp-content/uploads/2013/02/Print-Total-Login-Time-of-Users.png)

Print Total Login Time of Users

### Display Linux User Login Time

To get the total login statistics time of user “**tecmint**” in hours, use the command as.

```
# ac tecmint
```

```
 total      702.29
```

### Display Day-Wise Login Time of User

The following command will print the day-wise total login time of user “**tecmint**” in hours.

```
# ac -d tecmint
```

```
Oct 11  total        8.01
Oct 12  total       24.00
Oct 15  total       70.50
Oct 16  total       23.57
Oct 17  total       24.00
Oct 18  total       18.70
Nov 20  total        0.18

```

### Print All Linux Commands Executed by Users

The “**sa**” command is used to print the summary of commands that were executed by users.

```
# sa
```

```
       2       9.86re       0.00cp     2466k   sshd*
       8       1.05re       0.00cp     1064k   man
       2      10.08re       0.00cp     2562k   sshd
      12       0.00re       0.00cp     1298k   psacct
       2       0.00re       0.00cp     1575k   troff
      14       0.00re       0.00cp      503k   ac
      10       0.00re       0.00cp     1264k   psacct*
      10       0.00re       0.00cp      466k   consoletype
       9       0.00re       0.00cp      509k   sa
       8       0.02re       0.00cp      769k   udisks-helper-a
       6       0.00re       0.00cp     1057k   touch
       6       0.00re       0.00cp      592k   gzip
       6       0.00re       0.00cp      465k   accton
       4       1.05re       0.00cp     1264k   sh*
       4       0.00re       0.00cp     1264k   nroff*
       2       1.05re       0.00cp     1264k   sh
       2       1.05re       0.00cp     1120k   less
       2       0.00re       0.00cp     1346k   groff
       2       0.00re       0.00cp     1383k   grotty
       2       0.00re       0.00cp     1053k   mktemp
       2       0.00re       0.00cp     1030k   iconv
       2       0.00re       0.00cp     1023k   rm
       2       0.00re       0.00cp     1020k   cat
       2       0.00re       0.00cp     1018k   locale
       2       0.00re       0.00cp      802k   gtbl

```

Explanation of the above command output:

-   **9.86re** is a “**real-time**” as per wall clock minutes
-   **0.01cp** is a sum of system/user time in cpu minutes
-   **2466k** is a cpu-time averaged core usage, i.e. **1k** units
-   **sshd** command name

### Print Linux User Information

To get the information of an individual user, use the options **\-u**.

```
# sa -u
```

```
root       0.00 cpu      465k mem accton
root       0.00 cpu     1057k mem touch
root       0.00 cpu     1298k mem psacct
root       0.00 cpu      466k mem consoletype
root       0.00 cpu     1264k mem psacct           *
root       0.00 cpu     1298k mem psacct
root       0.00 cpu      466k mem consoletype
root       0.00 cpu     1264k mem psacct           *
root       0.00 cpu     1298k mem psacct
root       0.00 cpu      466k mem consoletype
root       0.00 cpu     1264k mem psacct           *
root       0.00 cpu      465k mem accton
root       0.00 cpu     1057k mem touch
```

### Print Number of Linux Processes

This command prints the [total number of processes](https://www.tecmint.com/find-processes-by-memory-usage-top-batch-mode/ "Find Top Processes by Memory Usage") and CPU minutes. If you see a continued increase in these numbers, then it’s time to look into the system about what is happening.

```
# sa -m
```

```
sshd                                    2       9.86re       0.00cp     2466k
root                                  127      14.29re       0.00cp      909k
```

### Print and Sort Usage by Percentage

The command “**sa -c**” displays the highest percentage of users.

```
# sa -c
```

```
 132  100.00%      24.16re  100.00%       0.01cp  100.00%      923k
       2    1.52%       9.86re   40.83%       0.00cp   53.33%     2466k   sshd*
       8    6.06%       1.05re    4.34%       0.00cp   20.00%     1064k   man
       2    1.52%      10.08re   41.73%       0.00cp   13.33%     2562k   sshd
      12    9.09%       0.00re    0.01%       0.00cp    6.67%     1298k   psacct
       2    1.52%       0.00re    0.00%       0.00cp    6.67%     1575k   troff
      18   13.64%       0.00re    0.00%       0.00cp    0.00%      509k   sa
      14   10.61%       0.00re    0.00%       0.00cp    0.00%      503k   ac
      10    7.58%       0.00re    0.00%       0.00cp    0.00%     1264k   psacct*
      10    7.58%       0.00re    0.00%       0.00cp    0.00%      466k   consoletype
       8    6.06%       0.02re    0.07%       0.00cp    0.00%      769k   udisks-helper-a
       6    4.55%       0.00re    0.00%       0.00cp    0.00%     1057k   touch
       6    4.55%       0.00re    0.00%       0.00cp    0.00%      592k   gzip
       6    4.55%       0.00re    0.00%       0.00cp    0.00%      465k   accton
       4    3.03%       1.05re    4.34%       0.00cp    0.00%     1264k   sh*
       4    3.03%       0.00re    0.00%       0.00cp    0.00%     1264k   nroff*
       2    1.52%       1.05re    4.34%       0.00cp    0.00%     1264k   sh
       2    1.52%       1.05re    4.34%       0.00cp    0.00%     1120k   less
       2    1.52%       0.00re    0.00%       0.00cp    0.00%     1346k   groff
       2    1.52%       0.00re    0.00%       0.00cp    0.00%     1383k   grotty
       2    1.52%       0.00re    0.00%       0.00cp    0.00%     1053k   mktemp
```

### List Last Executed Commands of User

The ‘**latcomm**‘ command is used to search and display previously executed user command information. You can also search commands of individual usernames. For example, we see commands of the user (**tecmint**).

```
# lastcomm tecmint
```

```
su                      tecmint  pts/0      0.00 secs Wed Feb 13 15:56
ls                      tecmint  pts/0      0.00 secs Wed Feb 13 15:56
ls                      tecmint  pts/0      0.00 secs Wed Feb 13 15:56
ls                      tecmint  pts/0      0.00 secs Wed Feb 13 15:56
bash               F    tecmint  pts/0      0.00 secs Wed Feb 13 15:56
id                      tecmint  pts/0      0.00 secs Wed Feb 13 15:56
grep                    tecmint  pts/0      0.00 secs Wed Feb 13 15:56
grep                    tecmint  pts/0      0.00 secs Wed Feb 13 15:56
bash               F    tecmint  pts/0      0.00 secs Wed Feb 13 15:56
dircolors               tecmint  pts/0      0.00 secs Wed Feb 13 15:56
bash               F    tecmint  pts/0      0.00 secs Wed Feb 13 15:56
tput                    tecmint  pts/0      0.00 secs Wed Feb 13 15:56
tty                     tecmint  pts/0      0.00 secs Wed Feb 13 15:56
bash               F    tecmint  pts/0      0.00 secs Wed Feb 13 15:56
id                      tecmint  pts/0      0.00 secs Wed Feb 13 15:56
bash               F    tecmint  pts/0      0.00 secs Wed Feb 13 15:56
id                      tecmint  pts/0      0.00 secs Wed Feb 13 15:56
```

### Search Logs for Commands

With the help of the **lastcomm** command, you will be able to view the individual use of each command.

```
# lastcomm ls
```

```
ls                      tecmint  pts/0      0.00 secs Wed Feb 13 15:56
ls                      tecmint  pts/0      0.00 secs Wed Feb 13 15:56
ls                      tecmint  pts/0      0.00 secs Wed Feb 13 15:56

```

For more information and usage, check out the manual pages of these tools.

## If You Appreciate What We Do Here On TecMint, You Should Consider:

TecMint is the fastest growing and most trusted community site for any kind of Linux Articles, Guides and Books on the web. Millions of people visit TecMint! to search or browse the thousands of published articles available FREELY to all.

If you like what you are reading, please consider buying us a coffee ( or 2 ) as a token of appreciation.

[![Support Us](https://www.tecmint.com/wp-content/uploads/2015/01/coffee.png)](https://www.buymeacoffee.com/tecmint)

**We are thankful for your never ending support.**