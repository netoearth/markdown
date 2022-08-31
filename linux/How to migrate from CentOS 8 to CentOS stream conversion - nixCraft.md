As you know, [CentOS 8 is ending soon](https://www.cyberciti.biz/linux-news/centos-linux-8-will-end-in-2021-and-shifts-focus-to-centos-stream/). Red Hat is making the shift from CentOS 8 to CentOS Stream. CentOS stream places itself between Fedora Linux and RHEL. It is not 100% RHEL clone but ahead of RHEL development. Think of it as a midstream distro. Of course, if you need 100% RHEL compatibility, then you need Rocky Linux or AlmaLinux. However, the CentOS stream is more than sufficient for me as I only need Apache, Perl, and Python for my use case. This page explains how to migrate the existing installation of CentOS 8 stable to CentOS Stream without reinstalling a new operating system.  
  
CentOS Stream is an open-source operating system and one of the replacement candidates for CentOS 8. However, I decided to stick with CentOS Stream because I didn’t have time or energy to install a new replacement such as Rocky Linux or AlmaLinux. Then restore data. It is too much work for my side project. Eventually, I will convert my legacy app to Docker format, but for now, I am going to upgrade my VM and save some time. So without further ado, let us see how to migrate from CentOS 8 to CentOS Stream using the ssh command.

## Step 1 – Backup

Like every seasoned developer and sysadmin, I backed up all my project files, MySQL database and config files. So if something goes wrong, I should be able to go back quickly. That is all. Hence, keep verified backups. I have following software installed:

-   [ELEP repo enabled for CentOS 8](https://www.cyberciti.biz/faq/how-to-enable-and-install-epel-repo-on-centos-8-x/)
-   SELinux and [FirewallD on CentOS 8](https://www.cyberciti.biz/faq/how-to-set-up-a-firewall-using-firewalld-on-centos-8/) enabled
-   Python 3.6.8.
-   python3-mod\_wsgi.
-   Perl 5.26.3.
-   mod\_perl 2.0.11-4.
-   Apache (httpd) 2.4.37 with mod\_ssl, mod\_security, mod\_session, mod\_speedycgi and other modules.

Do note down and list enabled repos:  
`# yum repolist   # yum repolist enabled > /root/pre.update.dnf.repo.txt`

## Step 2 – Installing all updates on CentOS 8

Login using the ssh command:  
`ssh {userName}@{your-Server-Name-IP-Here}   # for example:   ssh vivek@nixcraft-centos-httpd`  
Next, simply run the dnf command:  
`sudo dnf update`  
Then [reboot the Linux box](https://www.cyberciti.biz/faq/howto-reboot-linux/) using the [shutdown command](https://www.cyberciti.biz/faq/howto-shutdown-linux/ "How To Shutdown Linux Using Command Line") or [reboot command](https://www.cyberciti.biz/faq/linux-reboot-command/ "Linux Reboot Command Example"):  
`sudo shutdown -r now`  
Let us verify CentOS Linux 8 version using the [cat command](https://www.cyberciti.biz/faq/linux-unix-appleosx-bsd-cat-command-examples/ "cat Command in Linux / Unix with examples")/[more command](https://bash.cyberciti.biz/guide/More_command "More command - Linux Shell Scripting Wiki") or [less command](https://bash.cyberciti.biz/guide/Less_command "Less command - Linux Shell Scripting Wiki"):  
`more /etc/centos-release`  

![Migrate from CentOS 8 to CentOS Stream Verification](https://www.cyberciti.biz/media/new/cms/2021/12/Migrate-from-CentOS-8-to-CentOS-Stream-Verification.png)![Migrate from CentOS 8 to CentOS Stream Verification](https://www.cyberciti.biz/media/new/cms/2021/12/Migrate-from-CentOS-8-to-CentOS-Stream-Verification.png)

My fully patched CentOS 8 stable ready for migration to CentOS Stream

## Step 3 – Installing CentOS Stream package

Let us install the CentOS-Stream release file by typing the following dnf command:  
`sudo dnf in centos-release-stream`  

![How to Migrate CentOS 8 Installation to CentOS Stream Using DNF](https://www.cyberciti.biz/media/new/cms/2021/12/How-to-Migrate-CentOS-8-Installation-to-CentOS-Stream-Using-DNF.png)![How to Migrate CentOS 8 Installation to CentOS Stream Using DNF](https://www.cyberciti.biz/media/new/cms/2021/12/How-to-Migrate-CentOS-8-Installation-to-CentOS-Stream-Using-DNF.png)

Installing CentOS-Stream release files

## Step 4 – Migrating From CentOS 8 to CentOS Stream

Now we have the required package is in place. In other words, the dnf is ready to provide a simple way for us to start migrating from CentOS 8 to CentOS Stream. But, first, we must swap repos. The following command will remove CentOS 8 repo and replace it with the CentOS Stream repo. We will get all updates and libs/apps from CentOS Stream. Execute the following command:  
`sudo dnf swap centos-linux-repos centos-stream-repos`

```
Last metadata expiration check: 0:07:46 ago on Friday 03 December 2021 05:14:03 PM UTC.
Dependencies resolved.
========================================================================================
 Package                      Architecture  Version          Repository            Size
========================================================================================
Installing:
 centos-stream-release        noarch        8.6-1.el8        Stream-BaseOS         22 k
     replacing  centos-linux-release.noarch 8.5-1.2111.el8
     replacing  centos-release-stream.x86_64 8.1-1.1911.0.7.el8
 centos-stream-repos          noarch        8-3.el8          extras                19 k
Removing:
 centos-linux-repos           noarch        8-3.el8          @baseos               26 k
 
Transaction Summary
========================================================================================
Install  2 Packages
Remove   1 Package
 
Total download size: 42 k
Is this ok [y/N]: y
Downloading Packages:
(1/2): centos-stream-repos-8-3.el8.noarch.rpm           588 kB/s |  19 kB     00:00    
(2/2): centos-stream-release-8.6-1.el8.noarch.rpm        61 kB/s |  22 kB     00:00    
----------------------------------------------------------------------------------------
Total                                                    91 kB/s |  42 kB     00:00     
Running transaction check
Transaction check succeeded.
Running transaction test
Transaction test succeeded.
Running transaction
  Preparing        :                                                                1/1 
  Running scriptlet: centos-stream-release-8.6-1.el8.noarch                         1/1 
  Installing       : centos-stream-release-8.6-1.el8.noarch                         1/5 
  Installing       : centos-stream-repos-8-3.el8.noarch                             2/5 
  Obsoleting       : centos-release-stream-8.1-1.1911.0.7.el8.x86_64                3/5 
  Obsoleting       : centos-linux-release-8.5-1.2111.el8.noarch                     4/5 
  Erasing          : centos-linux-repos-8-3.el8.noarch                              5/5 
warning: /etc/yum.repos.d/CentOS-Linux-PowerTools.repo saved as /etc/yum.repos.d/CentOS-Linux-PowerTools.repo.rpmsave
warning: /etc/yum.repos.d/CentOS-Linux-Plus.repo saved as /etc/yum.repos.d/CentOS-Linux-Plus.repo.rpmsave
warning: /etc/yum.repos.d/CentOS-Linux-HighAvailability.repo saved as /etc/yum.repos.d/CentOS-Linux-HighAvailability.repo.rpmsave
warning: /etc/yum.repos.d/CentOS-Linux-FastTrack.repo saved as /etc/yum.repos.d/CentOS-Linux-FastTrack.repo.rpmsave
warning: /etc/yum.repos.d/CentOS-Linux-Extras.repo saved as /etc/yum.repos.d/CentOS-Linux-Extras.repo.rpmsave
warning: /etc/yum.repos.d/CentOS-Linux-Devel.repo saved as /etc/yum.repos.d/CentOS-Linux-Devel.repo.rpmsave
warning: /etc/yum.repos.d/CentOS-Linux-ContinuousRelease.repo saved as /etc/yum.repos.d/CentOS-Linux-ContinuousRelease.repo.rpmsave
warning: /etc/yum.repos.d/CentOS-Linux-BaseOS.repo saved as /etc/yum.repos.d/CentOS-Linux-BaseOS.repo.rpmsave
warning: /etc/yum.repos.d/CentOS-Linux-AppStream.repo saved as /etc/yum.repos.d/CentOS-Linux-AppStream.repo.rpmsave
 
  Running scriptlet: centos-linux-repos-8-3.el8.noarch                              5/5 
  Verifying        : centos-stream-repos-8-3.el8.noarch                             1/5 
  Verifying        : centos-stream-release-8.6-1.el8.noarch                         2/5 
  Verifying        : centos-linux-release-8.5-1.2111.el8.noarch                     3/5 
  Verifying        : centos-release-stream-8.1-1.1911.0.7.el8.x86_64                4/5 
  Verifying        : centos-linux-repos-8-3.el8.noarch                              5/5 
 
Installed:
  centos-stream-release-8.6-1.el8.noarch       centos-stream-repos-8-3.el8.noarch      
Removed:
  centos-linux-repos-8-3.el8.noarch                                                     
 
Complete!
```

  

Patreon supporters only guides 🤓

### Finally migrate CentOS 8 installation to CentOS stream

First list repos, run:  
`sudo dnf repolist`

#### Upgrading CentOS 8 to CentOS Stream

We are now close to our target. Next, we need to get packages for CentOS Stream [ABI/API](https://en.wikipedia.org/wiki/Application_binary_interface). This will upgrade or downgrade packages to match the new ABI/API and break 100% RHEL compatibility due to ABI/API change. I am OK with that. So let us do it:  
`sudo dnf distro-sync`

```
Last metadata expiration check: 0:00:19 ago on Friday 03 December 2021 05:28:32 PM UTC.
Dependencies resolved.
========================================================================================
 Package                     Arch   Version                             Repo       Size
========================================================================================
Upgrading:
 NetworkManager              x86_64 1:1.36.0-0.1.el8                    baseos    2.3 M
 NetworkManager-libnm        x86_64 1:1.36.0-0.1.el8                    baseos    1.8 M
 NetworkManager-team         x86_64 1:1.36.0-0.1.el8                    baseos    149 k
 NetworkManager-tui          x86_64 1:1.36.0-0.1.el8                    baseos    341 k
 bash                        x86_64 4.4.20-3.el8                        baseos    1.5 M
 c-ares                      x86_64 1.13.0-6.el8                        baseos     93 k
 ca-certificates             noarch 2021.2.50-82.el8                    baseos    390 k
 cloud-init                  noarch 21.1-9.el8                          appstream 1.0 M
 cpio                        x86_64 2.12-11.el8                         baseos    266 k
 cronie                      x86_64 1.5.2-6.el8                         baseos    118 k
 cronie-anacron              x86_64 1.5.2-6.el8                         baseos     42 k
 crypto-policies             noarch 20211116-1.gitae470d6.el8           baseos     64 k
 crypto-policies-scripts     noarch 20211116-1.gitae470d6.el8           baseos     83 k
 device-mapper               x86_64 8:1.02.181-1.el8                    baseos    377 k
 device-mapper-libs          x86_64 8:1.02.181-1.el8                    baseos    409 k
 dnf                         noarch 4.7.0-5.el8                         baseos    543 k
 dnf-automatic               noarch 4.7.0-5.el8                         baseo
.....
..
 vim-enhanced                x86_64 2:8.0.1763-16.el8_5.2               appstream 1.4 M
 vim-filesystem              noarch 2:8.0.1763-16.el8_5.2               appstream  49 k
 vim-minimal                 x86_64 2:8.0.1763-16.el8_5.2               baseos    573 k
 yum                         noarch 4.7.0-5.el8                         baseos    206 k
Installing dependencies:
 glibc-gconv-extra           x86_64 2.28-170.el8                        baseos    1.4 M
Installing weak dependencies:
 sqlite                      x86_64 3.26.0-15.el8                       baseos    668 k
Downgrading:
 unzip                       x86_64 6.0-45.el8                          baseos    195 k
 
Transaction Summary
========================================================================================
Install     2 Packages
Upgrade    86 Packages
Downgrade   1 Package
 
Total download size: 95 M
Downloading Packages:
(1/89): unzip-6.0-45.el8.x86_64.rpm                     571 kB/s | 195 kB     00:00    
(2/89): sqlite-3.26.0-15.el8.x86_64.rpm                 1.4 MB/s | 668 kB     00:00    
 
.....
..
  vim-filesystem-2:8.0.1763-16.el8_5.2.noarch                                           
  vim-minimal-2:8.0.1763-16.el8_5.2.x86_64                                              
  yum-4.7.0-5.el8.noarch                                                                
Downgraded:
  unzip-6.0-45.el8.x86_64                                                               
Installed:
  glibc-gconv-extra-2.28-170.el8.x86_64           sqlite-3.26.0-15.el8.x86_64          
 
Complete!
```

## Step 5 – Reboot the system

Run:  
`sudo systemctl reboot`

## Step 6 – Verification

See [CentOS stream version](https://www.cyberciti.biz/faq/how-to-check-os-version-in-linux-command-line/) for verification:  
`cat /etc/centos-release`  
![Upgrading CentOS 8 to CentOS Stream Linux using DNF](https://www.cyberciti.biz/media/new/cms/2021/12/Upgrading-CentOS-8-to-CentOS-Stream-Linux-using-DNF.png)![Upgrading CentOS 8 to CentOS Stream Linux using DNF](https://www.cyberciti.biz/media/new/cms/2021/12/Upgrading-CentOS-8-to-CentOS-Stream-Linux-using-DNF.png)  
Make sure there are no errors. Let us [see or view errors log on Linux](https://www.cyberciti.biz/faq/linux-log-files-location-and-how-do-i-view-logs-files/):  
`sudo tail -f /var/log/messages   sudo grep -i -E 'err|wan|cri' /var/log/messages   sudo grep -i -E 'err|wan|cri' /var/log/nginx/error.log`  
[List open ports on Linux](https://www.cyberciti.biz/faq/how-to-check-open-ports-in-linux-using-the-cli/) and make sure services such as httpd, mysqld and stuff is running using the systemctl command or netstat command/ss command:  
`sudo ss -tulpn   systemctl status httpd   systemctl status mysqld`

### A note about 3rd party dnf repos

Some dnf repos may be renamed or disabled to prevent upgrade issues. It would be best if you validated those manually. For example:  
\# yum repolist  
\# yum repolist enabled > /root/post.update.dnf.repo.txt  
You can compare those two files:  
`# cat /root/post.update.dnf.repo.txt   # cat /root/pre.update.dnf.repo.txt   # diff /root/pre.update.dnf.repo.txt /root/post.update.dnf.repo.txt`  
Then enable missing repos again.

## Video demo

Here is a quick video demo:  
[](https://www.youtube.com/watch?v=O2Eexp9SSSE)

## Summing up

I am pleasantly surprised by how smoothly the process went. This saved me time. I am also aware that one can take a similar approach to switch to AlmaLinux or RocyLinux, or Oracle Linux by swapping repos. For now, my side project working smoothly on AWS EC2 VM. But, one can use self-hosted or other cloud service providers too. I also deployed a local Rocky Linux VM on my Laptop, where I will containerize my app, so I no longer have to depend on the host OS. That is why I didn’t waste much time with a completely new OS. Do check out the CentOS Stream [website for documentation](https://www.centos.org/centos-stream/).  

  

ADVERTISEMENT  

  
  

| Category | List of Unix and Linux commands |
| --- | --- |
| Download managers | [wget](https://www.cyberciti.biz/tips/linux-wget-your-ultimate-command-line-downloader.html "Wget Command in Linux with Examples") |
| Driver Management | [Linux Nvidia driver](https://www.cyberciti.biz/faq/tag/linux-nvidia-driver/ "Linux Nvidia Driver How To - Linux / Unix Q & A from nixCraft") • [lsmod](https://www.cyberciti.biz/faq/linux-show-the-status-of-modules-driver-lsmod-command/ "Linux find out what kernel drivers loaded with lsmod command") |
| Documentation | [help](https://bash.cyberciti.biz/guide/Help_command "Help command - Linux Shell Scripting Wiki") • [mandb](https://bash.cyberciti.biz/guide/Mandb_command "Mandb command - Linux Shell Scripting Wiki") • [man](https://bash.cyberciti.biz/guide/Man_command "Man command - Linux Shell Scripting Wiki") • [pinfo](https://www.cyberciti.biz/open-source/command-line-hacks/linux-command-pinfo-for-colorful-info-pages/ "pinfo - Read Linux Info Documentation in Colors") |
| Disk Management | [df](https://www.cyberciti.biz/faq/df-command-examples-in-linux-unix/ "How to use df command in Linux / Unix {with examples}") • [duf](https://www.cyberciti.biz/open-source/command-line-hacks/duf-disk-usage-free-utility-for-linux-bsd-macos-windows/ "duf - Disk Usage/Free Utility for Linux, BSD, macOS & Windows") • [ncdu](https://www.cyberciti.biz/open-source/install-ncdu-on-linux-unix-ncurses-disk-usage/ "How to install ncdu on Linux / Unix to see disk usage") • [pydf](https://www.cyberciti.biz/tips/unix-linux-bsd-pydf-command-in-colours.html "Unix / Linux: See Colourised Filesystem Disk Space Usage with pydf") |
| File Management | [cat](https://www.cyberciti.biz/faq/linux-unix-appleosx-bsd-cat-command-examples/ "cat Command in Linux / Unix with examples") • [cp](https://www.cyberciti.biz/faq/copy-command/ "Linux Copy File Command [ cp Command Examples ]") • [less](https://bash.cyberciti.biz/guide/Less_command "Less command - Linux Shell Scripting Wiki") • [mkdir](https://www.cyberciti.biz/faq/linux-make-directory-command/ "Linux: How to Make a Directory Command") • [more](https://bash.cyberciti.biz/guide/More_command "More command - Linux Shell Scripting Wiki") • [tree](https://www.cyberciti.biz/faq/linux-show-directory-structure-command-line/ "Linux see directory tree structure using tree command") |
| Firewall | [Alpine Awall](https://www.cyberciti.biz/faq/how-to-set-up-a-firewall-with-awall-on-alpine-linux/ "How To Set Up a Firewall with Awall on Alpine Linux") • [CentOS 8](https://www.cyberciti.biz/faq/how-to-set-up-a-firewall-using-firewalld-on-centos-8/ "How to set up a firewall using FirewallD on CentOS 8") • [OpenSUSE](https://www.cyberciti.biz/faq/set-up-a-firewall-using-firewalld-on-opensuse-linux/ "How to set up a firewall using FirewallD on OpenSUSE Linux") • [RHEL 8](https://www.cyberciti.biz/faq/configure-set-up-a-firewall-using-firewalld-on-rhel-8/ "How to set up a firewall using FirewallD on RHEL 8") • [Ubuntu 16.04](https://www.cyberciti.biz/faq/howto-configure-setup-firewall-with-ufw-on-ubuntu-linux/ "How to set up a UFW firewall on Ubuntu 16.04 LTS server") • [Ubuntu 18.04](https://www.cyberciti.biz/faq/how-to-setup-a-ufw-firewall-on-ubuntu-18-04-lts-server/ "How to setup a UFW firewall on Ubuntu 18.04 LTS server") • [Ubuntu 20.04](https://www.cyberciti.biz/faq/how-to-configure-firewall-with-ufw-on-ubuntu-20-04-lts/ "How To Configure Firewall with UFW on Ubuntu 20.04 LTS") |
| Linux Desktop apps | [GIMP](https://www.cyberciti.biz/faq/how-to-install-gimp-on-ubuntu-debian-linux/ "How to install GIMP 2.10 on Ubuntu or Debian Linux") • [Skype](https://www.cyberciti.biz/faq/how-to-install-skype-application-on-linux/ "How to install Skype application on Linux") • [Spotify](https://www.cyberciti.biz/faq/how-to-install-spotify-application-on-linux/ "How to install Spotify application on Linux") • [VLC 3](https://www.cyberciti.biz/faq/how-to-install-vlc-3-application-vetinari-on-linux/ "How to install VLC 3 application (Vetinari) on Linux") |
| Modern utilities | [bat](https://www.cyberciti.biz/open-source/bat-linux-command-a-cat-clone-with-written-in-rust/ "bat Linux command - A cat clone with written in Rust") • [exa](https://www.cyberciti.biz/open-source/command-line-hacks/exa-a-modern-replacement-for-ls-written-in-rust-for-linuxunix/ "exa a modern replacement for ls command in rust for Linux/Unix") |
| Network Utilities | [NetHogs](https://www.cyberciti.biz/faq/linux-find-out-what-process-is-using-bandwidth/ "Linux See Bandwidth Usage Per Process With Nethogs Tool") • [dig](https://www.cyberciti.biz/faq/linux-unix-dig-command-examples-usage-syntax/ "Linux and Unix dig Command Examples") • [host](https://www.cyberciti.biz/faq/linux-unix-host-command-examples-usage-syntax/ "Linux and Unix host Command Examples") • [ip](https://www.cyberciti.biz/faq/linux-ip-command-examples-usage-syntax/ "Linux ip Command Examples") • [nmap](https://www.cyberciti.biz/security/nmap-command-examples-tutorials/ "Nmap Command Examples For Linux Sys/Network Admins") • [ping](https://www.cyberciti.biz/faq/unix-ping-command-examples/ "UNIX ping Command Examples") |
| OpenVPN | [CentOS 7](https://www.cyberciti.biz/faq/centos-7-0-set-up-openvpn-server-in-5-minutes/ "CentOS 7 Set Up OpenVPN Server In 5 Minutes") • [CentOS 8](https://www.cyberciti.biz/faq/centos-8-set-up-openvpn-server-in-5-minutes/ "CentOS 8 Set Up OpenVPN Server In 5 Minutes") • [Debian 10](https://www.cyberciti.biz/faq/debian-10-set-up-openvpn-server-in-5-minutes/ "Debian 10 Set Up OpenVPN Server In 5 Minutes") • [Debian 11](https://www.cyberciti.biz/faq/debian-11-set-up-openvpn-server-in-5-minutes/ "Debian 11 Set Up OpenVPN Server In 5 Minutes") • [Debian 8/9](https://www.cyberciti.biz/faq/install-configure-openvpn-server-on-debian-9-linux/ "Install and Configure an OpenVPN on Debian 9 In 5 Minutes") • [Ubuntu 18.04](https://www.cyberciti.biz/faq/ubuntu-18-04-lts-set-up-openvpn-server-in-5-minutes/ "Ubuntu 18.04 LTS Set Up OpenVPN Server In 5 Minutes") • [Ubuntu 20.04](https://www.cyberciti.biz/faq/ubuntu-20-04-lts-set-up-openvpn-server-in-5-minutes/ "Ubuntu 20.04 LTS Set Up OpenVPN Server In 5 Minutes") |
| Power Management | [upower](https://www.cyberciti.biz/faq/linux-upower-command-examples-and-syntax/ "upower command in Linux with examples") |
| Package Manager | [apk](https://www.cyberciti.biz/faq/10-alpine-linux-apk-command-examples/ "10 Alpine Linux apk Command Examples") • [apt-get](https://www.cyberciti.biz/tips/linux-debian-package-management-cheat-sheet.html "Ubuntu/Debian Linux apt-get package management cheat sheet") • [apt](https://www.cyberciti.biz/faq/ubuntu-lts-debian-linux-apt-command-examples/ "apt Command Examples for Ubuntu/Debian for new users") • [yum](https://www.cyberciti.biz/faq/rhel-centos-fedora-linux-yum-command-howto/ "How to use yum command on CentOS/RHEL") |
| Processes Management | [bg](https://www.cyberciti.biz/faq/unix-linux-bg-command-examples-usage-syntax/ "Linux / Unix: bg Command Examples") • [chroot](https://www.cyberciti.biz/faq/unix-linux-chroot-command-examples-usage-syntax/ "Linux / Unix: chroot Command Examples") • [cron](https://www.cyberciti.biz/faq/how-do-i-add-jobs-to-cron-under-linux-or-unix-oses/ "How To Add Jobs To cron Under Linux or UNIX") • [disown](https://www.cyberciti.biz/faq/unix-linux-disown-command-examples-usage-syntax/ "Linux / Unix: disown Command Examples") • [fg](https://www.cyberciti.biz/faq/unix-linux-fg-command-examples-usage-syntax/ "Linux / Unix: fg Command Examples") • [glances](https://www.cyberciti.biz/faq/linux-install-glances-monitoring-tool/ "Linux: Keep An Eye On Your System With Glances Monitor") • [gtop](https://www.cyberciti.biz/howto/gtop-awesome-system-monitoring-dashboard-for-terminal/ "gtop: Awesome system monitoring dashboard for Linux/macOS Unix terminal") • [iotop](https://www.cyberciti.biz/hardware/linux-iotop-simple-top-like-io-monitor/ "Linux iotop Check Whats Stressing & Increasing Load On Hard Disks") • [jobs](https://www.cyberciti.biz/faq/unix-linux-jobs-command-examples-usage-syntax/ "Linux / Unix: jobs Command Examples") • [killall](https://www.cyberciti.biz/faq/unix-linux-killall-command-examples-usage-syntax/ "Linux / Unix: killall Command Examples") • [kill](https://www.cyberciti.biz/faq/unix-kill-command-examples/ "Linux / UNIX: Kill Command Examples") • [pidof](https://www.cyberciti.biz/faq/linux-pidof-command-examples-find-pid-of-program/ "Linux pidof Command Examples To Find PID of A Program/Command") • [pstree](https://www.cyberciti.biz/faq/unix-linux-pstree-command-examples-shows-running-processestree/ "Linux/Unix: pstree Command Examples: See A Tree Of Processes") • [pwdx](https://www.cyberciti.biz/faq/unix-linux-pwdx-command-examples-usage-syntax/ "Linux / Unix: pwdx Command Examples") • [time](https://www.cyberciti.biz/faq/unix-linux-time-command-examples-usage-syntax/ "Linux / Unix: time Command Examples") • [vtop](https://www.cyberciti.biz/faq/how-to-install-and-use-vtop-graphical-terminal-activity-monitor-on-linux/ "How to install and use vtop graphical terminal monitor on Linux") |
| Searching | [ag](https://www.cyberciti.biz/open-source/command-line-hacks/ag-supercharge-string-search-through-directory-hierarchy/ "ag - supercharge string search through a directory on a Linux") • [egrep](https://www.cyberciti.biz/faq/grep-regular-expressions/ "Regular expressions in grep ( regex ) with examples") • [grep](https://www.cyberciti.biz/faq/howto-use-grep-command-in-linux-unix/ "How to use grep command in Linux/ Unix with examples") • [whereis](https://www.cyberciti.biz/faq/unix-linux-whereis-command-examples-to-locate-binary/ "Linux / Unix whereis Command Examples") • [which](https://www.cyberciti.biz/faq/unix-linux-which-command-examples-syntax-to-locate-programs/ "Linux / Unix which Command Examples To Locate a Command") |
| Shell builtins | [compgen](https://www.cyberciti.biz/open-source/command-line-hacks/compgen-linux-command/ "compgen: An Awesome Command To List All Linux Commands") • [echo](https://bash.cyberciti.biz/guide/Echo_Command "Echo Command - Linux Shell Scripting Wiki") • [printf](https://bash.cyberciti.biz/guide/Printf_command "Printf command - Linux Shell Scripting Wiki") |
| System Management | [reboot](https://www.cyberciti.biz/faq/howto-reboot-linux/ "Reboot Linux System Command") • [shutdown](https://www.cyberciti.biz/faq/howto-shutdown-linux/ "How To Shutdown Linux Using Command Line") |
| Terminal/ssh | [tty](https://www.cyberciti.biz/faq/linux-unix-appleosx-bsd-what-tty-command/ "Find Out What tty Im Using tty command under Linux / Unix") |
| Text processing | [cut](https://bash.cyberciti.biz/guide/Cut_command "Cut command - Linux Shell Scripting Wiki") • [rev](https://bash.cyberciti.biz/guide/Rev_command "Rev command - Linux Shell Scripting Wiki") |
| User Environment | [exit](https://bash.cyberciti.biz/guide/Exit_command "Exit command - Linux Shell Scripting Wiki") • [who](https://www.cyberciti.biz/faq/unix-linux-who-command-examples-syntax-usage/ "Linux / Unix who Command Examples To List Users on The Systems") |
| User Information | [groups](https://www.cyberciti.biz/faq/unix-linux-groups-command-examples-syntax-usage/ "Linux / Unix: groups Command Examples") • [id](https://www.cyberciti.biz/faq/unix-linux-id-command-examples-usage-syntax/ "Linux / Unix id Command Examples") • [lastcomm](https://www.cyberciti.biz/faq/linux-unix-lastcomm-command-examples-usage-syntax/ "Linux / Unix: lastcomm Command Examples") • [last](https://www.cyberciti.biz/faq/linux-unix-last-command-examples/ "Linux / Unix last Command Examples") • [lid/libuser-lid](https://www.cyberciti.biz/faq/linux-lid-command-examples-syntax-usage/ "Linux lid (libuser-lid) Command Examples") • [logname](https://www.cyberciti.biz/faq/unix-linux-logname-command-examples-syntax-usage/ "Linux / Unix: logname Command Examples To Display Loginname") • [members](https://www.cyberciti.biz/faq/linux-members-command-examples-usage-syntax/ "Linux members Command Examples") • [users](https://www.cyberciti.biz/faq/unix-linux-users-command-examples-syntax-usage/ "Linux / Unix: users Command Examples") • [whoami](https://www.cyberciti.biz/faq/unix-linux-whoami-command-examples-syntax-usage/ "Linux / Unix: whoami Command Examples") • [w](https://www.cyberciti.biz/faq/unix-linux-w-command-examples-syntax-usage-2/ "Linux / Unix: w Command Examples ") |
| User Management | [/etc/group](https://www.cyberciti.biz/faq/understanding-etcgroup-file/ "Understanding /etc/group File") • [/etc/passwd](https://www.cyberciti.biz/faq/understanding-etcpasswd-file-format/ "Understanding /etc/passwd File Format") • [/etc/shadow](https://www.cyberciti.biz/faq/understanding-etcshadow-file/ "Understanding /etc/shadow file format on Linux") |
| WireGuard VPN | [Alpine](https://www.cyberciti.biz/faq/how-to-set-up-wireguard-vpn-server-on-alpine-linux/ "Alpine Linux set up WireGuard VPN server") • [CentOS 8](https://www.cyberciti.biz/faq/centos-8-set-up-wireguard-vpn-server/ "CentOS 8 set up WireGuard VPN server") • [Debian 10](https://www.cyberciti.biz/faq/debian-10-set-up-wireguard-vpn-server/ "Debian 10 set up WireGuard VPN server") • [Firewall](https://www.cyberciti.biz/faq/how-to-set-up-wireguard-firewall-rules-in-linux/ "How To Set Up WireGuard Firewall Rules in Linux") • [Ubuntu 20.04](https://www.cyberciti.biz/faq/ubuntu-20-04-set-up-wireguard-vpn-server/ "How to set up WireGuard VPN server on Ubuntu 20.04") • [qrencode](https://www.cyberciti.biz/faq/how-to-generate-wireguard-qr-code-on-linux-for-mobile/ "How to generate WireGuard QR code on Linux for mobile") |