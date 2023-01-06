[![See all Firewall related FAQ](https://www.cyberciti.biz/media/new/category/old/firewall.png)](https://www.cyberciti.biz/faq/category/iptables/ "See all Firewall related FAQ")

I recently added NAT rules on my RHEL 6.x/7.x and Ubuntu/Debian v10/11 system. How do I see the rules including line numbers that I just added in Linux?  
  
Yes, you can easily **list all iptables rules using the following commands** on Linux:  
(1) [iptables command](https://www.cyberciti.biz/tips/linux-iptables-examples.html?utm_source=Linux_Unix_Command&utm_medium=faq&utm_campaign=nixcmd "Linux 25 Iptables Netfilter Firewall Examples") – IPv4 netfilter admin tool to display iptables firewall rules.  
(2) ip6tables command – IPv6 netfilter admin tool to show rules.

| Tutorial details |
| --- |
| Difficulty level | [Easy](https://www.cyberciti.biz/faq/tag/easy/ "See all Easy Linux / Unix System Administrator Tutorials") |
| Root privileges | [Yes](https://www.cyberciti.biz/faq/how-can-i-log-in-as-root/ "See how to login as root user") |
| Requirements | Linux terminal |
| Category | [nFirewall](https://www.cyberciti.biz/faq/how-to-list-all-iptables-rules-in-linux/#nFirewall "See ALL other tutorials in nFirewall category") |
| Prerequisites | iptables or ip6tables command |
| OS compatibility | Alma • [Alpine](https://www.cyberciti.biz/faq/category/alpine-linux/ "See all Alpine Linux tutorials") • [Arch](https://www.cyberciti.biz/faq/category/arch-linux/ "See all Arch Linux tutorials") • [Debian](https://www.cyberciti.biz/faq/category/debian-ubuntu/ "See all Debian Linux tutorials") • [Fedora](https://www.cyberciti.biz/faq/category/fedora-linux/ "See all Fedora Linux Enterprise tutorials") • Mint • [openSUSE](https://www.cyberciti.biz/faq/tag/opensuse/ "See all openSUSE Linux Enterprise tutorials") • Pop!\_OS • [RHEL](https://www.cyberciti.biz/faq/category/redhat-and-friends/ "See all RHEL (Red Hat Enterprise Linux) tutorials") • Rocky • [Stream](https://www.cyberciti.biz/faq/tag/centos-stream/ "See all CentOS Stream Linux tutorials") • [SUSE](https://www.cyberciti.biz/faq/category/suse/ "See all SUSE Linux Enterprise tutorials") • [Ubuntu](https://www.cyberciti.biz/faq/category/ubuntu-linux/ "See all Ubuntu Linux tutorials") |
| Est. reading time | 6 minutes |

Advertisement  

## How to list all iptables rules on Linux

The procedure to list all rules on Linux is as follows:

1.  Open the terminal app or login using ssh command:  
    `$ ssh user@server-name`
2.  To list all IPv4 rules:  
    `$ sudo iptables -S`
3.  Get list of all IPv6 rules:  
    `$ sudo ip6tables -S`
4.  To list all tables rules:  
    `$ sudo iptables -L -v -n | more`
5.  Just list all rules for INPUT tables:  
    `$ sudo iptables -L INPUT -v -n   $ sudo iptables -S INPUT`

Let us see all syntax and usage in details to show and list all iptables rules on Linux operating systems.

## Viewing all iptables rules in Linux

The syntax is:

```
iptables -S
iptables --list
iptables -L
iptables -S TABLE_NAME
iptables --table NameHere --list
iptables -t NameHere -L -n -v --line-numbers
```

## Print all rules in the selected chain

The command syntax is as follows for IPv4 rules:  
`$ sudo iptables -S   $ sudo iptables -S INPUT   $ iptables -S OUTPUT`  
[![How to list all iptables rules in Linux](https://www.cyberciti.biz/media/new/faq/2016/01/How-to-list-all-iptables-rules-in-Linux.jpg)](https://www.cyberciti.biz/media/new/faq/2016/01/How-to-list-all-iptables-rules-in-Linux.jpg)  
For IPv6 rules:  
`$ sudo ip6tables -S   $ sudo ip6tables -S INPUT   $ ip6tables -S OUTPUT`

### How to list rules for given tables

Type the following command as root user:  
`# iptables -L INPUT   # iptables -L FORWARD   # iptables -L OUTPUT   # iptables -L   # Listing IPv6 rules #   # ip6tables -L INPUT   # ip6tables -L FORWARD   # ip6tables -L OUTPUT   # ip6tables -L   `  
Sample outputs for IPv4:  

```
target     prot opt source               destination         
ACCEPT     tcp  --  anywhere             anywhere             tcp dpt:domain
ACCEPT     udp  --  anywhere             anywhere             udp dpt:domain
ACCEPT     tcp  --  anywhere             anywhere             tcp dpt:bootps
ACCEPT     udp  --  anywhere             anywhere             udp dpt:bootps
ufw-before-logging-input  all  --  anywhere             anywhere            
ufw-before-input  all  --  anywhere             anywhere            
ufw-after-input  all  --  anywhere             anywhere            
ufw-after-logging-input  all  --  anywhere             anywhere            
ufw-reject-input  all  --  anywhere             anywhere            
ufw-track-input  all  --  anywhere             anywhere            
 
Chain FORWARD (policy DROP)
target     prot opt source               destination         
ACCEPT     all  --  anywhere             anywhere            
ACCEPT     all  --  anywhere             anywhere            
ufw-before-logging-forward  all  --  anywhere             anywhere            
ufw-before-forward  all  --  anywhere             anywhere            
ufw-after-forward  all  --  anywhere             anywhere            
ufw-after-logging-forward  all  --  anywhere             anywhere            
ufw-reject-forward  all  --  anywhere             anywhere            
ufw-track-forward  all  --  anywhere             anywhere            
 
Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination         
ufw-before-logging-output  all  --  anywhere             anywhere            
ufw-before-output  all  --  anywhere             anywhere            
ufw-after-output  all  --  anywhere             anywhere            
ufw-after-logging-output  all  --  anywhere             anywhere            
ufw-reject-output  all  --  anywhere             anywhere            
ufw-track-output  all  --  anywhere             anywhere            
.....
..
..
Chain ufw-user-limit (0 references)
target     prot opt source               destination         
LOG        all  --  anywhere             anywhere             limit: avg 3/min burst 5 LOG level warning prefix "[UFW LIMIT BLOCK] "
REJECT     all  --  anywhere             anywhere             reject-with icmp-port-unreachable
 
Chain ufw-user-limit-accept (0 references)
target     prot opt source               destination         
ACCEPT     all  --  anywhere             anywhere            
 
Chain ufw-user-logging-forward (0 references)
target     prot opt source               destination         
 
Chain ufw-user-logging-input (0 references)
target     prot opt source               destination         
 
Chain ufw-user-logging-output (0 references)
target     prot opt source               destination         
 
Chain ufw-user-output (1 references)
target     prot opt source               destination
```

  
Let us try to understand rules output:

-   **target** – Tell what to do when a packet matches the rule. Typically, you ACCEPT or REJECT or DROP the packet. You can jump to another chain too.
-   **prot** – The protocol for rule.
-   **opt** – Additional options for rule.
-   **source** – The source IP address/subnet/domain name.
-   **destination** – The destination IP address/subnet/domain name.

#### **How to see nat rules:**

By default the filter table is used. To see NAT rules, enter:  
`# iptables -t nat -L`  
Other table options:  
`# iptables -t filter -L   # iptables -t raw -L   # iptables -t security -L   # iptables -t mangle -L   # iptables -t nat -L   # ip6tables -t filter -L`

#### **How to see nat rules with line numbers:**

Pass the \--line-numbers option:  
`# iptables -t nat -L --line-numbers -n   # for IPv4 rules #   # ip6tables -t nat -L --line-numbers -n`  
Sample outputs for IPv4:  

```
Chain PREROUTING (policy ACCEPT 28M packets, 1661M bytes)
num   pkts bytes target     prot opt in     out     source               destination         
1        0     0 DNAT       tcp  --  eth0   *       10.10.29.68          0.0.0.0/0            tcp dpt:3306 to:10.0.3.19:3306
2        0     0 DNAT       tcp  --  eth0   *       10.10.29.68          0.0.0.0/0            tcp dpt:11211 to:10.0.3.20:11211
3        0     0 DNAT       udp  --  eth0   *       10.10.29.68          0.0.0.0/0            udp dpt:11211 to:10.0.3.20:11211
 
Chain INPUT (policy ACCEPT 18M packets, 1030M bytes)
num   pkts bytes target     prot opt in     out     source               destination         
 
Chain OUTPUT (policy ACCEPT 23M packets, 1408M bytes)
num   pkts bytes target     prot opt in     out     source               destination         
 
Chain POSTROUTING (policy ACCEPT 33M packets, 1979M bytes)
num   pkts bytes target     prot opt in     out     source               destination         
1    38927 2336K MASQUERADE  all  --  *      *       10.0.3.0/24         !10.0.3.0/24         
2        0     0 MASQUERADE  all  --  *      *       10.0.3.0/24         !10.0.3.0/24
```

#### **How to see nat rules with counters (bytes and packets)**

Pass the \-v option to iptables command to view all iptables rules on Linux:  
`# iptables -t nat -L -n -v`  

![Fig.01: Linux viewing all iptables NAT, DNAT, MASQUERADE rules](https://www.cyberciti.biz/media/new/faq/2016/01/linux-view-nat-rules.jpg)

Fig.01: Linux viewing all iptables NAT, DNAT, MASQUERADE rules

## Say hello to ip6tables

ip6tables is administration tool for IPv6 packet filtering and NAT. To see IPv6 tables, enter:  
`# ip6tables -L -n -v`  

```
Chain INPUT (policy DROP 239 packets, 16202 bytes)
 pkts bytes target     prot opt in     out     source               destination         
 136K   30M ufw6-before-logging-input  all      *      *       ::/0                 ::/0                
 136K   30M ufw6-before-input  all      *      *       ::/0                 ::/0                
  241 16360 ufw6-after-input  all      *      *       ::/0                 ::/0                
  239 16202 ufw6-after-logging-input  all      *      *       ::/0                 ::/0                
  239 16202 ufw6-reject-input  all      *      *       ::/0                 ::/0                
  239 16202 ufw6-track-input  all      *      *       ::/0                 ::/0                

Chain FORWARD (policy DROP 483 packets, 32628 bytes)
 pkts bytes target     prot opt in     out     source               destination         
  483 32628 ufw6-before-logging-forward  all      *      *       ::/0                 ::/0                
  483 32628 ufw6-before-forward  all      *      *       ::/0                 ::/0                
  483 32628 ufw6-after-forward  all      *      *       ::/0                 ::/0                
  483 32628 ufw6-after-logging-forward  all      *      *       ::/0                 ::/0                
  483 32628 ufw6-reject-forward  all      *      *       ::/0                 ::/0                
  483 32628 ufw6-track-forward  all      *      *       ::/0                 ::/0                

Chain OUTPUT (policy ACCEPT 122 packets, 8555 bytes)
 pkts bytes target     prot opt in     out     source               destination         
 136K   30M ufw6-before-logging-output  all      *      *       ::/0                 ::/0                
 136K   30M ufw6-before-output  all      *      *       ::/0                 ::/0                
  183 14107 ufw6-after-output  all      *      *       ::/0                 ::/0                
  183 14107 ufw6-after-logging-output  all      *      *       ::/0                 ::/0                
  183 14107 ufw6-reject-output  all      *      *       ::/0                 ::/0                
  183 14107 ufw6-track-output  all      *      *       ::/0                 ::/0                

Chain ufw6-after-forward (1 references)
 pkts bytes target     prot opt in     out     source               destination         

...
....
..
 pkts bytes target     prot opt in     out     source               destination         
   19  1520 ACCEPT     tcp      *      *       ::/0                 ::/0                 ctstate NEW
   42  4032 ACCEPT     udp      *      *       ::/0                 ::/0                 ctstate NEW

Chain ufw6-user-forward (1 references)
 pkts bytes target     prot opt in     out     source               destination         

Chain ufw6-user-input (1 references)
 pkts bytes target     prot opt in     out     source               destination         

Chain ufw6-user-limit (0 references)
 pkts bytes target     prot opt in     out     source               destination         
    0     0 LOG        all      *      *       ::/0                 ::/0                 limit: avg 3/min burst 5 LOG flags 0 
level 4 prefix "[UFW LIMIT BLOCK] "
    0     0 REJECT     all      *      *       ::/0                 ::/0                 reject-with icmp6-port-unreachable

Chain ufw6-user-limit-accept (0 references)
 pkts bytes target     prot opt in     out     source               destination         
    0     0 ACCEPT     all      *      *       ::/0                 ::/0                

Chain ufw6-user-logging-forward (0 references)
 pkts bytes target     prot opt in     out     source               destination         

Chain ufw6-user-logging-input (0 references)
 pkts bytes target     prot opt in     out     source               destination         

Chain ufw6-user-logging-output (0 references)
 pkts bytes target     prot opt in     out     source               destination         

Chain ufw6-user-output (1 references)
 pkts bytes target     prot opt in     out     source               destination         
```

  
To see nat rules and line-numbers, enter:  
`# iptables -t nat -L --line-numbers -nip6tables -L -n -v -t nat --line-numbers`  

**Related**  
Also, check all our complete firewall tutorials for [Alpine Linux Awall](https://www.cyberciti.biz/faq/how-to-set-up-wireguard-vpn-server-on-alpine-linux/ "Alpine Linux set up WireGuard VPN server - nixCraft"), [CentOS 8](https://www.cyberciti.biz/faq/how-to-set-up-a-firewall-using-firewalld-on-centos-8/ "How to set up a firewall using FirewallD on CentOS 8 - nixCraft"), [OpenSUSE](https://www.cyberciti.biz/faq/set-up-a-firewall-using-firewalld-on-opensuse-linux/ "How to set up a firewall using FirewallD on OpenSUSE Linux - nixCraft"), [RHEL 8](https://www.cyberciti.biz/faq/configure-set-up-a-firewall-using-firewalld-on-rhel-8/ "How to set up a firewall using FirewallD on RHEL 8 - nixCraft"), Ubuntu Linux version [16.04 LTS](https://www.cyberciti.biz/faq/howto-configure-setup-firewall-with-ufw-on-ubuntu-linux/ "How to set up a UFW firewall on Ubuntu 16.04 LTS server - nixCraft")/[18.04 LTS](https://www.cyberciti.biz/faq/how-to-setup-a-ufw-firewall-on-ubuntu-18-04-lts-server/ "How to setup a UFW firewall on Ubuntu 18.04 LTS server - nixCraft")/[20.04 LTS](https://www.cyberciti.biz/faq/how-to-configure-firewall-with-ufw-on-ubuntu-20-04-lts/ "How To Configure Firewall with UFW on Ubuntu 20.04 LTS - nixCraft"), and [22.04 LTS](https://www.cyberciti.biz/faq/ubuntu-22-04-lts-set-up-ufw-firewall-in-5-minutes/ "Ubuntu 22.04 Set Up UFW Firewall in 5 Minutes - nixCraft").

## Conclusion

You learned how to display, filter and list all iptables rules on Linux system using the CLI. See iptables man pages by typing the following [man command](https://bash.cyberciti.biz/guide/Man_command?utm_source=Linux_Unix_Command&utm_medium=faq&utm_campaign=nixcmd "Man command - Linux Bash Shell Scripting Tutorial Wiki") or [help command](https://bash.cyberciti.biz/guide/Help_command?utm_source=Linux_Unix_Command&utm_medium=faq&utm_campaign=nixcmd "Help command - Linux Bash Shell Scripting Tutorial Wiki"):  
`$ man iptables   $ man ip6tables`