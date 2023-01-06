If you are using `iptables`, it’s very likely that you wish to make it persistent, and restore your firewall rules after a reboot.  
I’ll present here 3 ways to make your iptables persistent:

1\. using `systemd` my personal favorite, since it works for all Linux _distributions_ and without requiring 3rd party software.

2\. using `iptables-persistent` mostly for DEB-based Linux _distributions_, required 3rd party software

3\. using `iptables-services` for RPM-based Linux _distributions_, required 3rd party software

## Systemd

Systemd is a system and service manager for Linux operating systems. Using `systemd`we can run a script file after boot, that will restore our firewall rules and make it persistent without installing a 3rd party software.

first let’s create the script that we wish to run to restore our firewall:  
`sudo vi /etc/iptables-persistent/restore.sh`

```
#!/bin/sh/usr/bin/flock /run/.iptables-restore /sbin/iptables-restore < {{your ip tables dump file}}
```

next we will need to create an host file for our `systemd` service  
`sudo vi /etc/systemd/system/iptables-persistent.service`

```
[Unit] Description=runs iptables restore on bootConditionFileIsExecutable=/etc/iptables/restore-iptables.shAfter=network.target[Service]Type=forkingExecStart=/etc/iptables/restore-iptables.shstart TimeoutSec=0RemainAfterExit=yesGuessMainPID=no[Install]WantedBy=multi-user.target
```

all left to do is to enable our service by running  
`sudo systemctl enable iptables-persistent.service`

## iptables-persistent (DEB)

`iptables-persistent`automatically loads your saved ip-tables rules after a reboot.

First step will be to install `iptables-persistent` using `sudo apt-get install iptables-persistent`

![](https://miro.medium.com/max/623/1*tEFBQ62qPACcA5QohNc92Q.png)

since iptables-persistant will look for two dump files:

1.  `/etc/iptables/rules.v4`for ipv4 rules
2.  `/etc/iptables/rules.v6` for, wait for it, ipv6 rules

which you can easily create running the following commands:  
`sudo iptables-save > /etc/iptables/rules.v4   ``sudo ip6tables-save > /etc/iptables/rules.v6`

Depends on your OS version, behind the scenes `iptables-persistent` works with `netfilter-persistent.service` you can verify that your service up and running using `sudo systemctl status netfilter-persistent.service`

> netfilter-persistent.service — netfilter persistent configuration  
> Loaded: loaded (/lib/systemd/system/netfilter-persistent.service; enabled; ve  
> Active: active (exited) since Sat 2022–04–09 18:14:42 IDT; 29min ago

## iptables-`services` (RPM)

`iptables-services` contains a persistent utility that loads your saved `ip-tables` rules after a reboot.

Let’s start with installing `iptables-services` using `sudo dnf install iptables-services`

after installing `iptables-services` we will need to make sure that our service is up and that `firewalld` is disabled and won’t interfere with our `iptables` configuration, using the following commands:  
`sudo systemctl stop firewalld   sudo systemctl disable firewalld   sudo systemctl start iptables   sudo systemctl enable iptables`

since `iptables-services` will look for two dump files:

1.  `/etc/sysconfig/iptables` for ipv4 rules
2.  `/etc/sysconfig/ip6tables` for, wait for it, ipv6 rules

which you can easily create running the following commands:  
`sudo iptables-save > /etc/iptables/rules.v4   ``sudo ip6tables-save > /etc/iptables/rules.v6`

and that’s it, you can feel free to reboot your machine without losing your changes :)