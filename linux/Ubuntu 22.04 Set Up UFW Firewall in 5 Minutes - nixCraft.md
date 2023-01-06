[![See all Ubuntu Linux related FAQ](https://www.cyberciti.biz/media/new/category/old/ubuntu-logo.jpg)](https://www.cyberciti.biz/faq/category/ubuntu-linux/ "See all Ubuntu Linux related FAQ")

A Ubuntu 22.04 LTS comes with UFW (uncomplicated firewall) that protects the desktop or server against unauthorized access. UFW is easy to use frontend app for a Linux packet filtering system called Netfilter. Traditionally Netfilter rules are set up or configured using the [iptables command](https://www.cyberciti.biz/tips/linux-iptables-examples.html?utm_source=Linux_Unix_Command&utm_medium=faq&utm_campaign=nixcmd "Linux 25 Iptables Netfilter Firewall Examples") by developers and sysadmins. However, new Ubuntu Linux users and developers unfamiliar with firewall concepts find Netfilter syntax confusing. Hence, the ufw project provides easy to use frontend for Ubuntu 22.04 LTS Linux server and desktop. It is so super easy to set up. You can configure UFW in under 5 minutes and secure your host.  
  
This page explains how to set up a firewall with UFW on Ubuntu 22.04 LTS server or desktop.

| Tutorial details |
| --- |
| Difficulty level | [Easy](https://www.cyberciti.biz/faq/tag/easy/ "See all Easy Linux / Unix System Administrator Tutorials") |
| Root privileges | [Yes](https://www.cyberciti.biz/faq/how-can-i-log-in-as-root/ "See how to login as root user") |
| Requirements | Linux terminal |
| Category | [Firewall](https://www.cyberciti.biz/faq/ubuntu-22-04-lts-set-up-ufw-firewall-in-5-minutes/#Firewall "See ALL other tutorials in Firewall category") |
| OS compatibility | [Debian](https://www.cyberciti.biz/faq/category/debian-ubuntu/ "See all Debian Linux tutorials") • [Linux](https://www.cyberciti.biz/faq/category/linux/ "See all Linux distributions tutorials") • Mint • [Ubuntu](https://www.cyberciti.biz/faq/category/ubuntu-linux/ "See all Ubuntu Linux tutorials") |
| Est. reading time | 6 minutes |

## Ubuntu 22.04 LTS Set Up UFW Firewall in 5 Minutes

The steps are as follows:

### Step 1 – Set Up default UFW policies

Let us view the current status:  
`$ sudo ufw status`  
The default policy firewall works excellent for servers and the desktop. It is always a good policy to close all ports on the server and open only the required TCP or UDP ports. Let us block all incoming connections and only allow outgoing connections from the Ubuntu 22.04 LTS cloud server:  
`$ sudo ufw default allow outgoing   $ sudo ufw default deny incoming`  
Make sure IPv6 support enabled too. Run the [grep command](https://www.cyberciti.biz/faq/howto-use-grep-command-in-linux-unix/?utm_source=Linux_Unix_Command&utm_medium=faq&utm_campaign=nixcmd "How to use grep command in Linux/ Unix with examples"):  
`$ grep IPV6 /etc/default/ufw`  
Otherwise, edit the /etc/default/ufw:  
`$ sudo nano /etc/default/ufw`  
Set it as follows:

```
IPV6=yes
```

### Step 2 – Open SSH TCP port 22 using the ufw

The next rational step is to allow incoming SSH connections on the default TCP port 22 as follows:  
`$ sudo ufw allow ssh`  
Say you are running the OpenSSH server on TCP port 4242, then:  
`$ sudo ufw allow 4242/tcp`  
You can limit ssh port access as follows too:  
`$ sudo ufw limit ssh`  
See “[How to limit SSH (TCP port 22) connections with ufw on Ubuntu Linux](https://www.cyberciti.biz/faq/howto-limiting-ssh-connections-with-ufw-on-ubuntu-debian/ "How to limit SSH (TCP port 22) connections with ufw on Ubuntu Linux")” for more information.

### Step 3 – Turning on the firewall

That is all needed. Now turn on the firewall protection for your Ubuntu Linux 22.04 LTS machine. For example:  
`$ sudo ufw enable`  
You need to confirm the operation by typing the y and followed by the \[Enter\] key:  

[![How To Set Up a Firewall with UFW on Ubuntu 22.04 LTS](https://www.cyberciti.biz/media/new/faq/2022/08/How-To-Set-Up-a-Firewall-with-UFW-on-Ubuntu-22.04-LTS-599x439.png)](https://www.cyberciti.biz/media/new/faq/2022/08/How-To-Set-Up-a-Firewall-with-UFW-on-Ubuntu-22.04-LTS.png)

Click to enlarge

To view the current firewall status, type the systemctl command:  
`$ sudo ufw status`  
Please note that once UFW is enabled, it runs across system reboots. You can verify that easily using the systemctl command:  
`$ sudo systemctl status ufw.service`

```
● ufw.service - Uncomplicated firewall
     Loaded: loaded (/lib/systemd/system/ufw.service; enabled; vendor preset: enabled)
     Active: active (exited) since Fri 2022-08-26 01:02:24 UTC; 20min ago
       Docs: man:ufw(8)
    Process: 433 ExecStart=/lib/ufw/ufw-init start quiet (code=exited, status=0/SUCCESS)
   Main PID: 433 (code=exited, status=0/SUCCESS)
        CPU: 2ms

Aug 26 01:02:24 localhost systemd[1]: Starting Uncomplicated firewall...
Aug 26 01:02:24 localhost systemd[1]: Finished Uncomplicated firewall.

```

### Step 4 – Opening (allow) TCP or UDP ports

Now that you set up a firewall policy and opened TCP port 22 for ssh purposes, it is time to open other service ports as per the needs of your application. For example, open TCP port 80 and 443 for Nginx or Apache web server as follows:  
`$ sudo ufw allow 80/tcp comment 'Allow Apache HTTP'   $ sudo ufw allow 443/tcp comment 'Allow Nginx HTTPS'`  
Here is how to open the WireGuard VPN UDP port 41194, type:  
`$ sudo ufw allow 41194/udp comment 'Allow WireGuard VPN'`  
The ufw comment keywords adds [comments, which act as an instrumental in understanding](https://www.cyberciti.biz/faq/howto-adding-comments-to-ufw-firewall-rule/ "How do you add comments on UFW firewall rule?") firewall rules.

#### Opening TCP and UDP port ranges

`$ sudo ufw allow 4000:4200/tcp   $ sudo ufw allow 6000:7000/udp`

#### Allowing connection from a single IP or CIDR

In this example, you want to allow ALL connections from an IP address called 1.2.3.4, enter:  
`$ sudo ufw allow from 1.2.3.4`  
Let us allow connections from an IP address called 1.2.3.4 to our port 25, enter:  
`$ sudo ufw allow from 104.22.11.213 to any port 25 proto tcp`  
And you can set destination IP 222.222.222.222 for port 25 too:  
sudo ufw allow from 1.2.3.4 to 222.222.222.222 port 25 proto tcp

#### How to allow connection on specific interface

Open TCP port 22 for wg0 interface only:  
`$ sudo ufw allow in on wg0 to any port 22`  
Say you want to allow connection for TCP port 3306 on lxdbr0 interface from 10.105.28.22, then add:  
`$ sudo ufw allow in on lxdbr0 from 10.105.28.22 to any port 3306 proto tcp`

### Step 5 – Blocking TCP or UDP ports and connections

Do you want to close ports and block certain IP addresses? The syntax is as follows to deny access. In other words, simply ignoring access to port 23:  
`$ sudo ufw deny 23/tcp comment 'Block telnet'`  
Here is how to deny all connections from an IP address called 1.2.3.4, enter:  
`$ sudo ufw deny from 1.2.3.4`  
How about clock IP/subnet (CIDR) called 103.13.42.42/28, enter:  
`$ sudo ufw deny from 103.13.42.42/28`  
Finally, deny access to 1.1.1.2 (say bad guys or hacker IP address) on port 22? Try:  
`$ sudo ufw deny from 1.1.1.2 to any port 22 proto tcp`

### Step 6 – Viewing firewall rules

You can see firewall status as numbered list of RULES:  
`$ sudo ufw status numbered`  

[![How to view ufw firewall rules on Ubuntu Linux 22.04 LTS](https://www.cyberciti.biz/media/new/faq/2022/08/How-to-view-ufw-firewall-rules-on-Ubuntu-Linux-22.04-LTS-599x240.png)](https://www.cyberciti.biz/media/new/faq/2022/08/How-to-view-ufw-firewall-rules-on-Ubuntu-Linux-22.04-LTS.png)

Click to enlarge

### Step 7 – Deleting ufw firewall rules

Get list all of the current rules in a numbered list format:  
`$ sudo ufw status numbered`  
Outputs:

```
Status: active

     To                         Action      From
     --                         ------      ----
[ 1] 22/tcp                     ALLOW IN    Anywhere                  
[ 2] 80/tcp                     ALLOW IN    Anywhere                   # Allow Apache HTTP
[ 3] 443/tcp                    ALLOW IN    Anywhere                   # Allow Nginx HTTPS
[ 4] 41194/udp                  ALLOW IN    Anywhere                   # Allow WireGuard VPN
[ 5] 23/tcp                     DENY IN     Anywhere                   # Block telnet
[ 6] Anywhere                   DENY IN     103.13.42.32/28           
[ 7] 22/tcp (v6)                ALLOW IN    Anywhere (v6)             
[ 8] 80/tcp (v6)                ALLOW IN    Anywhere (v6)              # Allow Apache HTTP
[ 9] 443/tcp (v6)               ALLOW IN    Anywhere (v6)              # Allow Nginx HTTPS
[10] 41194/udp (v6)             ALLOW IN    Anywhere (v6)              # Allow WireGuard VPN
[11] 23/tcp (v6)                DENY IN     Anywhere (v6)              # Block telnet


```

To remove firewall rule # 6 type the command:  
`$ sudo ufw delete 6   $ sudo ufw status numbered`  
See [how to delete a UFW firewall rule on Ubuntu / Debian Linux tutorial](https://www.cyberciti.biz/faq/how-to-delete-a-ufw-firewall-rule-on-ubuntu-debian-linux/ "How to delete a UFW firewall rule on Ubuntu / Debian Linux") for further information.

### Step 8 – Stopping and removing UFW

If you no longer need ufw, here is how to disable it:  
`$ sudo ufw disable   $ sudo ufw reset`

### Step 9 – View the firewall logs

By default all UFW entries are logged into /var/log/ufw.log file. Use the [grep](https://www.cyberciti.biz/faq/howto-use-grep-command-in-linux-unix/?utm_source=Linux_Unix_Command&utm_medium=faq&utm_campaign=nixcmd "How to use grep command in Linux/ Unix with examples")/[less](https://www.cyberciti.biz/faq/howto-use-grep-command-in-linux-unix/?utm_source=Linux_Unix_Command&utm_medium=faq&utm_campaign=nixcmd "How to use grep command in Linux/ Unix with examples")/[more](https://www.cyberciti.biz/faq/howto-use-grep-command-in-linux-unix/?utm_source=Linux_Unix_Command&utm_medium=faq&utm_campaign=nixcmd "How to use grep command in Linux/ Unix with examples") and other commands to view the ufw logs. For examples:  
`$ sudo more /var/log/ufw.log   $ sudo tail -f /var/log/ufw.log`  
Let us print a list of all IP address trying to log in via SSH port but dropped by the UFW:  
`$ grep 'DPT=22' /var/log/ufw.log |\   [egrep](https://www.cyberciti.biz/faq/grep-regular-expressions/?utm_source=Linux_Unix_Command&utm_medium=faq&utm_campaign=nixcmd "Regular expressions in grep ( regex ) with examples") -o 'SRC=([0-9]{1,3}[\.]){3}[0-9]{1,3}' |\   awk -F'=' '{ print $2 }' | sort -u`  
Finally, here is how to display the list of rules:  
`$ sudo ufw show listening   $ sudo ufw show added`

## Summing up

Wasn’t that easy? Now you know how to protect your Ubuntu 22.04 LTS Linux server. Please read the ufw command [docs online or using](https://help.ubuntu.com/community/UFW) the [man command](https://bash.cyberciti.biz/guide/Man_command?utm_source=Linux_Unix_Command&utm_medium=faq&utm_campaign=nixcmd "Man command - Linux Bash Shell Scripting Tutorial Wiki") ([ufw help](https://bash.cyberciti.biz/guide/Help_command?utm_source=Linux_Unix_Command&utm_medium=faq&utm_campaign=nixcmd "Help command - Linux Bash Shell Scripting Tutorial Wiki") command) as follows:  
`$ man ufw`