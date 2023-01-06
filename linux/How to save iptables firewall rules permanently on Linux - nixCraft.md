[![See all Firewall related FAQ](https://www.cyberciti.biz/media/new/category/old/firewall.png)](https://www.cyberciti.biz/faq/category/iptables/ "See all Firewall related FAQ")

I am using Debian / Ubuntu Linux server. How do I save iptables rules permanently on Linux using the CLI added using the [iptables command](https://www.cyberciti.biz/tips/linux-iptables-examples.html?utm_source=Linux_Unix_Command&utm_medium=faq&utm_campaign=nixcmd "Linux 25 Iptables Netfilter Firewall Examples")? How can I store iptables IPv4 and IPv6 rules permanently on the Debian Linux cloud server?  
  
Linux system administrator and developers use iptables and ip6tables commands to set up, maintain, and inspect the firewall tables of IPv4 and IPv6 packet filter rules in the Linux kernel. Any modification made using these commands is lost when you reboot the Linux server. Hence, we need to store those rules across reboot permanently. This page examples how to save iptables firewall rules permanently either on Ubuntu or Debian Linux server.

| Tutorial details |
| --- |
| Difficulty level | [Advanced](https://www.cyberciti.biz/faq/tag/advanced/ "See all Expert Linux / Unix System Administrator Tutorials") |
| Root privileges | [Yes](https://www.cyberciti.biz/faq/how-can-i-log-in-as-root/ "See how to login as root user") |
| Requirements | Linux iptables |
| Category | [Firewall](https://www.cyberciti.biz/faq/how-to-save-iptables-firewall-rules-permanently-on-linux/#Firewall "See ALL other tutorials in Firewall category") |
| Est. reading time | 6 minutes |

Advertisement  

## Saving iptables firewall rules permanently on Linux

You need to use the following commands to save iptables firewall rules forever:

1.  **iptables-save command** or **ip6tables-save command** – Save or dump the contents of IPv4 or IPv6 Table in easily parseable format either to screen or to a specified file.
2.  **iptables-restore command** or **ip6tables-restore command** – Restore IPv4 or IPv6 firewall rules and tables from a given file under Linux.

![Saving iptables firewall rules permanently on Linux](https://www.cyberciti.biz/media/new/faq/2020/08/Saving-iptables-firewall-rules-permanently-on-Linux.png)

### Step 1 – Open the terminal

Open the terminal application and then type the following commands. For remote server login using the ssh command:  
`$ ssh vivek@server1.cyberciti.biz   $ ssh ec2-user@ec2-host-or-ip`  
You must type the following command as root user either using the sudo command or su command.

### Step 2 – Save IPv4 and IPv6 Linux firewall rules

Debian and Ubuntu Linux user type:  
`$ sudo /sbin/iptables-save > /etc/iptables/rules.v4   ## IPv6 ##   $ sudo /sbin/ip6tables-save > /etc/iptables/rules.v6`  
CentOS/RHEL/Fedora/Rocky and AlmaLinux users run:  
`$ sudo /sbin/iptables-save > /etc/sysconfig/iptables   ## IPv6 ##   $ sudo /sbin/ip6tables-save > /etc/sysconfig/ip6tables`

#### Displaying saved rules on Linux

We can display saved file using the [cat command](https://www.cyberciti.biz/faq/linux-unix-appleosx-bsd-cat-command-examples/?utm_source=Linux_Unix_Command&utm_medium=faq&utm_campaign=nixcmd "cat Command in Linux / Unix with examples") or search using the [grep command](https://www.cyberciti.biz/faq/howto-use-grep-command-in-linux-unix/?utm_source=Linux_Unix_Command&utm_medium=faq&utm_campaign=nixcmd "How to use grep command in Linux/ Unix with examples")/[egrep command](https://www.cyberciti.biz/faq/grep-regular-expressions/?utm_source=Linux_Unix_Command&utm_medium=faq&utm_campaign=nixcmd "Regular expressions in grep ( regex ) with examples"):  
`$ cat /etc/iptables/rules.v4`

```
*mangle
 
:PREROUTING ACCEPT [0:0]
:INPUT ACCEPT [0:0]
:FORWARD ACCEPT [0:0]
:OUTPUT ACCEPT [0:0]
:POSTROUTING ACCEPT [0:0]
 
 
COMMIT
 
 
*nat
 
:PREROUTING ACCEPT [0:0]
:POSTROUTING ACCEPT [0:0]
 
-A POSTROUTING -s 10.8.0.0/24 -m policy --pol none --dir out -j MASQUERADE
 
COMMIT
 
 
*filter
 
:INPUT DROP [0:0]
:FORWARD DROP [0:0]
:OUTPUT ACCEPT [0:0]
 
-A INPUT -i lo -j ACCEPT
-A INPUT -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT
-A INPUT -p esp -j ACCEPT
-A INPUT -p ah -j ACCEPT 
-A INPUT -p icmp --icmp-type echo-request -m hashlimit --hashlimit-upto 5/s --hashlimit-mode srcip --hashlimit-srcmask 32 --hashlimit-name icmp-echo-drop -j ACCEPT
-A INPUT -p udp -m multiport --dports 21194 -j ACCEPT
-A INPUT -p tcp -s 1xx.yy.zz.ttt,10.8.0.0/24,10.8.1.0/24 --dport 22 -m conntrack --ctstate NEW -j ACCEPT
-A INPUT -p tcp --dport 3128 -d 10.8.0.1 -s 10.8.0.0/24,10.8.1.0/24  -m conntrack --ctstate NEW -j ACCEPT
-A INPUT -d 172.xx.yyy.zzz -p udp --dport 53 -j ACCEPT
-A INPUT -d 172.xx.yyy.z -p udp --dport 53 -j ACCEPT
-A INPUT -i eth0 -m limit --limit 5/min -j LOG --log-prefix "[iptables denied] " --log-level 7
-A FORWARD -s 10.8.0.0/24 -d 10.8.0.0/24 -j DROP
-A OUTPUT -d 10.8.0.0/24 -m owner --gid-owner 15000 -j DROP
-A FORWARD -s 10.8.0.0/24 -d 169.254.0.0/16 -j DROP
-A OUTPUT -d 169.254.0.0/16 -m owner --gid-owner 15000 -j DROP
-A FORWARD -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT
-A FORWARD -p tcp --dport 445 -j DROP
-A FORWARD -p udp -m multiport --ports 137,138 -j DROP
-A FORWARD -p tcp -m multiport --ports 137,139 -j DROP
-A FORWARD -m conntrack --ctstate NEW -s 10.8.0.0/24 -m policy --pol none --dir in -j ACCEPT
-A FORWARD -m limit --limit 5/min -j LOG --log-prefix "[iptables forward denied] " --log-level 7
COMMIT
```

### Step 3 – Restore IPv4 and IPv6 Linux filewall rules

We just reverse above commands as follows per operating system:  
`## Debian or Ubuntu ##   $ sudo /sbin/iptables-restore < /etc/iptables/rules.v4   $ sudo /sbin/ip6tables-restore < /etc/iptables/rules.v6   ## CentOS/RHEL ##   $ sudo /sbin/iptables-save < /etc/sysconfig/iptables   $ sudo /sbin/ip6tables-save < /etc/sysconfig/ip6tables`

### Step 4 – Installing iptables-persistent package for Debian or Ubuntu Linux

Please note that the following command will conflict with iptables frontends such as [ufw command](https://www.cyberciti.biz/faq/ubuntu-22-04-lts-set-up-ufw-firewall-in-5-minutes/?utm_source=Linux_Unix_Command&utm_medium=faq&utm_campaign=nixcmd "Ubuntu 22.04 Set Up UFW Firewall in 5 Minutes") or [firewall-cmd command](https://www.cyberciti.biz/faq/configure-set-up-a-firewall-using-firewalld-on-rhel-8/?utm_source=Linux_Unix_Command&utm_medium=faq&utm_campaign=nixcmd "How to set up a firewall using FirewallD on RHEL 8"). Avoid using the following packages if you are using those tools.

We need to install iptables-persistent. It will act as a loader for Netfilter rules, iptables plugin netfilter-persistent, which is a loader for Netfilter configuration using a plugin-based architecture. In other words, the automatic loading of the saved iptables rules from the above files. Type the following [apt command](https://www.cyberciti.biz/faq/ubuntu-lts-debian-linux-apt-command-examples/?utm_source=Linux_Unix_Command&utm_medium=faq&utm_campaign=nixcmd "apt Command Examples for Ubuntu/Debian Linux") or [apt-get command](https://www.cyberciti.biz/tips/linux-debian-package-management-cheat-sheet.html?utm_source=Linux_Unix_Command&utm_medium=faq&utm_campaign=nixcmd "Ubuntu/Debian Linux apt-get package management cheat sheet"):  
`$ sudo apt install iptables-persistent   ## OR ##   $ sudo apt-get install iptables-persistent`  
![How to save iptables firewall rules permanently on Linux](https://www.cyberciti.biz/media/new/faq/2020/08/How-to-save-iptables-firewall-rules-permanently-on-Linux.jpg)  
Make sure services are enabled on Debian or Ubuntu using the systemctl command:  
`$ sudo systemctl is-enabled netfilter-persistent.service`  
If not enable it:  
`$ sudo systemctl enable netfilter-persistent.service`  
Get status:  
`$ sudo systemctl status netfilter-persistent.service`  
Outputs:

```
● netfilter-persistent.service - netfilter persistent configuration
     Loaded: loaded (/lib/systemd/system/netfilter-persistent.service; enabled; vendor preset: enabled)
    Drop-In: /etc/systemd/system/netfilter-persistent.service.d
             └─iptables.conf
     Active: active (exited) since Thu 2020-08-20 19:24:22 UTC; 3 days ago
       Docs: man:netfilter-persistent(8)
   Main PID: 577 (code=exited, status=0/SUCCESS)
      Tasks: 0 (limit: 4620)
     Memory: 0B
     CGroup: /system.slice/netfilter-persistent.service

Aug 20 19:24:21 nixcraft-vpn-1 systemd[1]: Starting netfilter persistent configuration...
Aug 20 19:24:21 nixcraft-vpn-1 netfilter-persistent[583]: run-parts: executing /usr/share/netfilter-persistent/plugins.d/15-ip4tables start
Aug 20 19:24:21 nixcraft-vpn-1 netfilter-persistent[583]: run-parts: executing /usr/share/netfilter-persistent/plugins.d/25-ip6tables start
Aug 20 19:24:22 nixcraft-vpn-1 systemd[1]: Finished netfilter persistent configuration.

```

### Step 5 – Install iptables-services package for RHEL/CentOS/Fedora/Rocky and AlmaLinux

By default, [RHEL/CentOS 7 or 8 comes with firewalld](https://www.cyberciti.biz/faq/how-to-set-up-a-firewall-using-firewalld-on-centos-8/). If you need old good file-based firewall then type the following commands:  
`# Disable firewalld if installed #   $ sudo systemctl stop firewalld.service   $ sudo systemctl disable firewalld.service   $ sudo systemctl mask firewalld.service   # install package on Linux to save iptables rules using the [yum command](https://www.cyberciti.biz/faq/rhel-centos-fedora-linux-yum-command-howto/?utm_source=Linux_Unix_Command&utm_medium=faq&utm_campaign=nixcmd "How to use yum command on CentOS/RHEL")/dnf command ##   $ sudo yum install iptables-services   $ sudo systemctl enable iptables   $ sudo systemctl enable ip6tables   $ sudo systemctl status iptables`  
Sample outputs:

```
● iptables.service - IPv4 firewall with iptables
   Loaded: loaded (/usr/lib/systemd/system/iptables.service; enabled; vendor preset: disabled)
   Active: active (exited) since Mon 2020-08-24 09:29:59 EDT; 3s ago
  Process: 8259 ExecStart=/usr/libexec/iptables/iptables.init start (code=exited, status=0/SUCCESS)
 Main PID: 8259 (code=exited, status=0/SUCCESS)

Aug 24 09:29:59 centos-8-cloud.sweet.home systemd[1]: Starting IPv4 firewall with iptables...
Aug 24 09:29:59 centos-8-cloud.sweet.home iptables.init[8259]: iptables: Applying firewall rules: [  OK  ]
Aug 24 09:29:59 centos-8-cloud.sweet.home systemd[1]: Started IPv4 firewall with iptables.

```

### Where are rules stored on RHEL/CentOS/Rocky/AlmaLinux?

Once the service is installed, you can configure the /etc/sysconfig/iptables for IPv4 and /etc/sysconfig/ip6tables for IPv6 files. Any rules added to these files make them persistent. Then restart the service to load those changes using the systemctl command:

```
# Step 1 - Edit/add/append/delete the Ipv4 and IPv6 rules:
sudo vim /etc/sysconfig/iptables
sudo vim /etc/sysconfig/ip6tables
 
# Step 2 - Load the changes:
sudo systemctl restart iptables
sudo systemctl restart ip6tables
```

## Conclusion

In this tutorial, you learned how to save and restore iptables rules permanently on Linux, especially on Debian/Ubuntu or CentOS/RHEL/Rocky/Alma Linux servers. Remember to use only one service to configure the Linux firewall. Do not mix ufw, firewalld, and iptables config files and frontends. Do read the following manual pages using the [man command](https://bash.cyberciti.biz/guide/Man_command?utm_source=Linux_Unix_Command&utm_medium=faq&utm_campaign=nixcmd "Man command - Linux Bash Shell Scripting Tutorial Wiki")/[help command](https://bash.cyberciti.biz/guide/Help_command?utm_source=Linux_Unix_Command&utm_medium=faq&utm_campaign=nixcmd "Help command - Linux Bash Shell Scripting Tutorial Wiki") and check out our [iptables command](https://www.cyberciti.biz/tips/linux-iptables-examples.html?utm_source=Linux_Unix_Command&utm_medium=faq&utm_campaign=nixcmd "Linux 25 Iptables Netfilter Firewall Examples") examples. For instance:  
`$ man iptables`