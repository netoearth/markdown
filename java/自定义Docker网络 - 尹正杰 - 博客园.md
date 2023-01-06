　　　　　　　　　　　　　　　**自定义Docker网络**

　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　**作者：尹正杰**

**版权声明：原创作品，谢绝转载！否则将追究法律责任。**

**一.docker管理网络的命令**

**1>.查看docker network的帮助信息**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@docker102.yinzhengjie.org.cn ~]# docker network --help

Usage:    docker network COMMAND

Manage networks

Commands:
  connect     Connect a container to a network
  create      Create a network
  disconnect  Disconnect a container from a network
  inspect     Display detailed information on one or more networks
  ls          List networks
  prune       Remove all unused networks
  rm          Remove one or more networks

Run 'docker network COMMAND --help' for more information on a command.
[root@docker102.yinzhengjie.org.cn ~]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

**2>.查看已经存在的网卡信息**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@docker102.yinzhengjie.org.cn ~]# docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
34d7483acce3        bridge              bridge              local
37f2cd930d53        host                host                local
4089feeb2359        none                null                local
[root@docker102.yinzhengjie.org.cn ~]# 
[root@docker102.yinzhengjie.org.cn ~]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

**3>.查看创建网卡命令的帮助信息**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@docker102.yinzhengjie.org.cn ~]# docker network create --help

Usage:    docker network create [OPTIONS] NETWORK

Create a network

Options:
      --attachable           Enable manual container attachment
      --aux-address map      Auxiliary IPv4 or IPv6 addresses used by Network driver (default map[])
      --config-from string   The network from which copying the configuration
      --config-only          Create a configuration only network
  -d, --driver string        Driver to manage the Network (default "bridge")
      --gateway strings      IPv4 or IPv6 Gateway for the master subnet
      --ingress              Create swarm routing-mesh network
      --internal             Restrict external access to the network
      --ip-range strings     Allocate container ip from a sub-range
      --ipam-driver string   IP Address Management Driver (default "default")
      --ipam-opt map         Set IPAM driver specific options (default map[])
      --ipv6                 Enable IPv6 networking
      --label list           Set metadata on a network
  -o, --opt map              Set driver specific options (default map[])
      --scope string         Control the network's scope
      --subnet strings       Subnet in CIDR format that represents a network segment
[root@docker102.yinzhengjie.org.cn ~]# 
[root@docker102.yinzhengjie.org.cn ~]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

**二.自定义docker案例**

**1>.创建一个bridge模式的网络**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@docker102.yinzhengjie.org.cn ~]# docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
34d7483acce3        bridge              bridge              local
37f2cd930d53        host                host                local
4089feeb2359        none                null                local
[root@docker102.yinzhengjie.org.cn ~]# 
[root@docker102.yinzhengjie.org.cn ~]# 
[root@docker102.yinzhengjie.org.cn ~]# docker network create -d bridge --subnet 10.30.1.0/24 --gateway 10.30.1.254 yinzhengjie-net
ec32b69e252b7d84a87436f1bb6ae33e2711b98f952df0ad2dd7289a34645827
[root@docker102.yinzhengjie.org.cn ~]# 
[root@docker102.yinzhengjie.org.cn ~]# docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
34d7483acce3        bridge              bridge              local
37f2cd930d53        host                host                local
4089feeb2359        none                null                local
ec32b69e252b        yinzhengjie-net     bridge              local
[root@docker102.yinzhengjie.org.cn ~]# 
[root@docker102.yinzhengjie.org.cn ~]#  
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

**![](https://img2018.cnblogs.com/i-beta/795254/202002/795254-20200201171337974-1899024558.png)**

**2>.基于咱们上一步自定义的网络启动一个容器并验证是否可以正常访问互联网**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@docker102.yinzhengjie.org.cn ~]# docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
34d7483acce3        bridge              bridge              local
37f2cd930d53        host                host                local
4089feeb2359        none                null                local
ec32b69e252b        yinzhengjie-net     bridge              local
[root@docker102.yinzhengjie.org.cn ~]# 
[root@docker102.yinzhengjie.org.cn ~]# 
[root@docker102.yinzhengjie.org.cn ~]# docker container run -it --network yinzhengjie-net --name myCentOS centos:centos7.6.1810 bash
[root@e56d37aa51a9 /]# 
[root@e56d37aa51a9 /]# yum -y install net-tools
[root@e56d37aa51a9 /]# 
[root@e56d37aa51a9 /]# ifconfig 
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 10.30.1.1  netmask 255.255.255.0  broadcast 10.30.1.255
        ether 02:42:0a:1e:01:01  txqueuelen 0  (Ethernet)
        RX packets 2665  bytes 13320309 (12.7 MiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 2425  bytes 133997 (130.8 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 66  bytes 5790 (5.6 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 66  bytes 5790 (5.6 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

[root@e56d37aa51a9 /]# 
[root@e56d37aa51a9 /]# route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         10.30.1.254     0.0.0.0         UG    0      0        0 eth0
10.30.1.0       0.0.0.0         255.255.255.0   U     0      0        0 eth0
[root@e56d37aa51a9 /]# 
[root@e56d37aa51a9 /]# ping www.baidu.com
PING www.a.shifen.com (111.206.223.173) 56(84) bytes of data.
64 bytes from 111.206.223.173 (111.206.223.173): icmp_seq=1 ttl=127 time=7.03 ms
64 bytes from 111.206.223.173 (111.206.223.173): icmp_seq=2 ttl=127 time=6.14 ms
64 bytes from 111.206.223.173 (111.206.223.173): icmp_seq=3 ttl=127 time=6.83 ms
^C
--- www.a.shifen.com ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2004ms
rtt min/avg/max/mdev = 6.140/6.670/7.034/0.383 ms
[root@e56d37aa51a9 /]# 
[root@e56d37aa51a9 /]#  
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

**3>.再基于咱们**自定义的网络启动一个容器并验证它们是否可以相互通信****

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@docker102.yinzhengjie.org.cn ~]# docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
34d7483acce3        bridge              bridge              local
37f2cd930d53        host                host                local
4089feeb2359        none                null                local
ec32b69e252b        yinzhengjie-net     bridge              local
[root@docker102.yinzhengjie.org.cn ~]# 
[root@docker102.yinzhengjie.org.cn ~]# docker container run -it --network yinzhengjie-net --name myCentOS2 centos:centos7.6.1810 bash 
[root@83a18f56cc14 /]# 
[root@83a18f56cc14 /]# hostname -i
10.30.1.2
[root@83a18f56cc14 /]# 
[root@83a18f56cc14 /]# ping 10.30.1.1
PING 10.30.1.1 (10.30.1.1) 56(84) bytes of data.
64 bytes from 10.30.1.1: icmp_seq=1 ttl=64 time=0.071 ms
64 bytes from 10.30.1.1: icmp_seq=2 ttl=64 time=0.097 ms
64 bytes from 10.30.1.1: icmp_seq=3 ttl=64 time=0.041 ms
64 bytes from 10.30.1.1: icmp_seq=4 ttl=64 time=0.037 ms
64 bytes from 10.30.1.1: icmp_seq=5 ttl=64 time=0.039 ms
64 bytes from 10.30.1.1: icmp_seq=6 ttl=64 time=0.052 ms
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

![](https://img2018.cnblogs.com/i-beta/795254/202002/795254-20200201172948996-894247880.png)

**三.使自定义网络和默认的bridge网络互通**

**1>.默认情况下自定义的网络和默认的bridge网络是不互通的(如果想要它们相互通信就得修改iptables规则)**

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@docker102.yinzhengjie.org.cn ~]# docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
34d7483acce3        bridge              bridge              local
37f2cd930d53        host                host                local
4089feeb2359        none                null                local
ec32b69e252b        yinzhengjie-net     bridge              local
[root@docker102.yinzhengjie.org.cn ~]# 
[root@docker102.yinzhengjie.org.cn ~]# docker container run -it -p 8080:80 --network bridge --rm centos:centos7.6.1810 bash
[root@87b5dc93b45d /]# 
[root@87b5dc93b45d /]# hostname -i
10.200.0.1
[root@87b5dc93b45d /]# 
[root@87b5dc93b45d /]# ping 10.30.1.1
PING 10.30.1.1 (10.30.1.1) 56(84) bytes of data.
```

\[root@docker102.yinzhengjie.org.cn ~\]# docker container run -it -p 8080:80 --network bridge --rm centos:centos7.6.1810 bash

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@docker102.yinzhengjie.org.cn ~]# docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
34d7483acce3        bridge              bridge              local
37f2cd930d53        host                host                local
4089feeb2359        none                null                local
ec32b69e252b        yinzhengjie-net     bridge              local
[root@docker102.yinzhengjie.org.cn ~]# 
[root@docker102.yinzhengjie.org.cn ~]# docker container run -it -p 80:80 --network yinzhengjie-net --rm centos:centos7.6.1810 bash
[root@ad24d400de75 /]# 
[root@ad24d400de75 /]# hostname -i
10.30.1.1
[root@ad24d400de75 /]# 
[root@ad24d400de75 /]# ping 10.200.0.1
PING 10.200.0.1 (10.200.0.1) 56(84) bytes of data.
```

\[root@docker102.yinzhengjie.org.cn ~\]# docker container run -it -p 80:80 --network yinzhengjie-net --rm centos:centos7.6.1810 bash

![](https://img2018.cnblogs.com/i-beta/795254/202002/795254-20200201174443354-74381634.png)

**2>.修改iptables的规则**

![](https://img2018.cnblogs.com/i-beta/795254/202002/795254-20200201175848629-1490879763.png)

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@docker102.yinzhengjie.org.cn ~]# iptables-save > iptables-rule.txt　　　　　　　　　　#先将iptables的规则导出
[root@docker102.yinzhengjie.org.cn ~]# 
[root@docker102.yinzhengjie.org.cn ~]# vim iptables-rule.txt 　　　　　　　　　　　　　　　　　#编辑防火墙的规则，为了保险起见，将上图所示的两行代码注释掉,不推荐直接删除。
[root@docker102.yinzhengjie.org.cn ~]# 
[root@docker102.yinzhengjie.org.cn ~]# cat iptables-rule.txt 
# Generated by iptables-save v1.4.21 on Sun Feb  2 01:59:26 2020
*filter
:INPUT ACCEPT [390:27397]
:FORWARD DROP [0:0]
:OUTPUT ACCEPT [229:44673]
:DOCKER - [0:0]
:DOCKER-ISOLATION-STAGE-1 - [0:0]
:DOCKER-ISOLATION-STAGE-2 - [0:0]
:DOCKER-USER - [0:0]
-A FORWARD -j DOCKER-USER
-A FORWARD -j DOCKER-ISOLATION-STAGE-1
-A FORWARD -o br-ec32b69e252b -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT
-A FORWARD -o br-ec32b69e252b -j DOCKER
-A FORWARD -i br-ec32b69e252b ! -o br-ec32b69e252b -j ACCEPT
-A FORWARD -i br-ec32b69e252b -o br-ec32b69e252b -j ACCEPT
-A FORWARD -o docker0 -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT
-A FORWARD -o docker0 -j DOCKER
-A FORWARD -i docker0 ! -o docker0 -j ACCEPT
-A FORWARD -i docker0 -o docker0 -j ACCEPT
-A FORWARD -s 172.200.0.0/21 -j ACCEPT
-A DOCKER-ISOLATION-STAGE-1 -i br-ec32b69e252b ! -o br-ec32b69e252b -j DOCKER-ISOLATION-STAGE-2
-A DOCKER-ISOLATION-STAGE-1 -i docker0 ! -o docker0 -j DOCKER-ISOLATION-STAGE-2
-A DOCKER-ISOLATION-STAGE-1 -j RETURN
#-A DOCKER-ISOLATION-STAGE-2 -o br-ec32b69e252b -j DROP
#-A DOCKER-ISOLATION-STAGE-2 -o docker0 -j DROP
-A DOCKER-ISOLATION-STAGE-2 -j RETURN
-A DOCKER-USER -j RETURN
COMMIT
# Completed on Sun Feb  2 01:59:26 2020
# Generated by iptables-save v1.4.21 on Sun Feb  2 01:59:26 2020
*nat
:PREROUTING ACCEPT [3:208]
:INPUT ACCEPT [3:208]
:OUTPUT ACCEPT [0:0]
:POSTROUTING ACCEPT [0:0]
:DOCKER - [0:0]
-A PREROUTING -m addrtype --dst-type LOCAL -j DOCKER
-A OUTPUT ! -d 127.0.0.0/8 -m addrtype --dst-type LOCAL -j DOCKER
-A POSTROUTING -s 10.30.1.0/24 ! -o br-ec32b69e252b -j MASQUERADE
-A POSTROUTING -s 10.200.0.0/16 ! -o docker0 -j MASQUERADE
-A DOCKER -i br-ec32b69e252b -j RETURN
-A DOCKER -i docker0 -j RETURN
COMMIT
# Completed on Sun Feb  2 01:59:26 2020
[root@docker102.yinzhengjie.org.cn ~]# 
[root@docker102.yinzhengjie.org.cn ~]# iptables-restore < iptables-rule.txt 　　　　　　　　#再将修改后的iptables规则导入
[root@docker102.yinzhengjie.org.cn ~]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

![](https://img2018.cnblogs.com/i-beta/795254/202002/795254-20200201180406945-956484986.png)

**3>.再次通过**自定义的网络和默认的bridge网络启动容器验证容器是否互通****

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@docker102.yinzhengjie.org.cn ~]# docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
34d7483acce3        bridge              bridge              local
37f2cd930d53        host                host                local
4089feeb2359        none                null                local
ec32b69e252b        yinzhengjie-net     bridge              local
[root@docker102.yinzhengjie.org.cn ~]# 
[root@docker102.yinzhengjie.org.cn ~]# docker container run -it -p 8080:80 --network bridge --rm centos:centos7.6.1810 bash
[root@70da6a2cad6b /]# 
[root@70da6a2cad6b /]# hostname -i
10.200.0.1
[root@70da6a2cad6b /]# 
[root@70da6a2cad6b /]# ping 10.30.1.1
PING 10.30.1.1 (10.30.1.1) 56(84) bytes of data.
64 bytes from 10.30.1.1: icmp_seq=1 ttl=63 time=0.104 ms
64 bytes from 10.30.1.1: icmp_seq=2 ttl=63 time=0.057 ms
64 bytes from 10.30.1.1: icmp_seq=3 ttl=63 time=0.047 ms
64 bytes from 10.30.1.1: icmp_seq=4 ttl=63 time=0.052 ms
64 bytes from 10.30.1.1: icmp_seq=5 ttl=63 time=0.047 ms
64 bytes from 10.30.1.1: icmp_seq=6 ttl=63 time=0.052 ms
64 bytes from 10.30.1.1: icmp_seq=7 ttl=63 time=0.049 ms
64 bytes from 10.30.1.1: icmp_seq=8 ttl=63 time=0.053 ms
64 bytes from 10.30.1.1: icmp_seq=9 ttl=63 time=0.051 ms
```

\[root@docker102.yinzhengjie.org.cn ~\]# docker container run -it -p 8080:80 --network bridge --rm centos:centos7.6.1810 bash

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@docker102.yinzhengjie.org.cn ~]# docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
34d7483acce3        bridge              bridge              local
37f2cd930d53        host                host                local
4089feeb2359        none                null                local
ec32b69e252b        yinzhengjie-net     bridge              local
[root@docker102.yinzhengjie.org.cn ~]# 
[root@docker102.yinzhengjie.org.cn ~]# docker container run -it -p 80:80 --network yinzhengjie-net --rm centos:centos7.6.1810 bash
[root@ec704cf0295c /]# 
[root@ec704cf0295c /]# hostname -i
10.30.1.1
[root@ec704cf0295c /]# 
[root@ec704cf0295c /]# ping 10.200.0.1
PING 10.200.0.1 (10.200.0.1) 56(84) bytes of data.
64 bytes from 10.200.0.1: icmp_seq=1 ttl=63 time=0.047 ms
64 bytes from 10.200.0.1: icmp_seq=2 ttl=63 time=0.048 ms
64 bytes from 10.200.0.1: icmp_seq=3 ttl=63 time=0.050 ms
64 bytes from 10.200.0.1: icmp_seq=4 ttl=63 time=0.049 ms
64 bytes from 10.200.0.1: icmp_seq=5 ttl=63 time=0.124 ms
64 bytes from 10.200.0.1: icmp_seq=6 ttl=63 time=0.051 ms
64 bytes from 10.200.0.1: icmp_seq=7 ttl=63 time=0.050 ms
64 bytes from 10.200.0.1: icmp_seq=8 ttl=63 time=0.072 ms
64 bytes from 10.200.0.1: icmp_seq=9 ttl=63 time=0.058 ms
```

\[root@docker102.yinzhengjie.org.cn ~\]# docker container run -it -p 80:80 --network yinzhengjie-net --rm centos:centos7.6.1810 bash

![](https://img2018.cnblogs.com/i-beta/795254/202002/795254-20200201180716599-1029383807.png)

当你的才华还撑不起你的野心的时候，你就应该静下心来学习。当你的能力还驾驭不了你的目标的时候，你就应该沉下心来历练。问问自己，想要怎样的人生。 欢迎加入基础架构自动化运维：598432640，大数据SRE进阶之路：959042252，DevOps进阶之路：526991186