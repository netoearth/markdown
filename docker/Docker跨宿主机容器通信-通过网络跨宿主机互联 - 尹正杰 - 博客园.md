　　　　　　　　　　　**Docker跨宿主机容器通信-通过网络跨宿主机互联**

　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　**作者：尹正杰**

**版权声明：原创作品，谢绝转载！否则将追究法律责任。**

**一.docker网络类型**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
　　Docker的网络有四种类型,分别为host模式,none模式,container模式,bridge模式。　　host模式:　　　　创建容器时,可以使用"-net=host"指定。　　　　启动的容器如果指定了网络类型为host模式，那么新创建的容器不会创建自己的虚拟网卡,而是直接使用宿主机的网卡和IP地址,因此容器里面查看到的IP信息就是宿主机的信息，访问容器的时候直接使用"宿主机IP:容器端口"即可,不过容器的其它资源们比如文件系统，系统进程等还是和宿主机保持隔离。　　　　此模式的网络性能最高,但是各容器之间端口不能相同(因为该模式直接使用的是宿主机的网络)，适用于运行容器端口比较固定的业务，比如Mysql,Redis等。　　none模式:　　　　创建容器时，可以使用"-net=none"指定。　　　　在使用none模式后,docker容器不会进行任何网络配置，其没有网卡，没有IP也没有路由，因此默认无法与外界通信，需要手动添加配置IP等，所以极少使用。　　container模式:　　　　创建容器时,可以使用"-net=container:容器名称或容器ID"指定。　　　　使用此模式创建的容器需指定和一个已经存在的容器共享一个网络,而不是和宿主机共享网络，换句话说，就是两个容器公用同一个IP地址。　　　　新创建的容器不会创建自己的网卡也不会配置自己的IP,而是和一个已经存在的被指定的容器IP和端口范围,因此这个容器的端口不能和被指定的端口冲突,除了网络之外的文件系统，进程信息等仍然保持相互隔离，两个容器的进程可以通过lo网卡设备通信。　　bridge模式:　　　　docker的默认模式即不指定任何模式就是bridge模式，也是使用比较多的模式，此模式创建的容器会为每一个容器分配自己的网络IP等信息，并将容器连接到一个虚拟网桥与外界通信。　　　　　博主推荐阅读:　　　　https://www.cnblogs.com/yinzhengjie/p/11517865.html
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

![](https://img2018.cnblogs.com/i-beta/795254/202002/795254-20200201100100567-865780974.png)

**1>.不指定容器的网络模式默认就是bridge模式**

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@docker101.yinzhengjie.org.cn ~]# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,SLAVE,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast master bond0 state UP group default qlen 1000
    link/ether 00:0c:29:fa:bd:9a brd ff:ff:ff:ff:ff:ff
3: eth1: <BROADCAST,MULTICAST,SLAVE,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast master bond1 state UP group default qlen 1000
    link/ether 00:0c:29:fa:bd:a4 brd ff:ff:ff:ff:ff:ff
4: eth2: <BROADCAST,MULTICAST,SLAVE,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast master bond0 state UP group default qlen 1000
    link/ether 00:0c:29:fa:bd:9a brd ff:ff:ff:ff:ff:ff
5: eth3: <BROADCAST,MULTICAST,SLAVE,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast master bond1 state UP group default qlen 1000
    link/ether 00:0c:29:fa:bd:a4 brd ff:ff:ff:ff:ff:ff
6: bond0: <BROADCAST,MULTICAST,MASTER,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether 00:0c:29:fa:bd:9a brd ff:ff:ff:ff:ff:ff
    inet 172.200.3.101/21 brd 172.200.7.255 scope global bond0
       valid_lft forever preferred_lft forever
7: bond1: <BROADCAST,MULTICAST,MASTER,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether 00:0c:29:fa:bd:a4 brd ff:ff:ff:ff:ff:ff
    inet 192.200.3.101/21 brd 192.200.7.255 scope global bond1
       valid_lft forever preferred_lft forever
8: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default 
    link/ether 02:42:81:7d:84:58 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# docker container run -it --name nginx-web01-net-bridge -d nginx:v0.1-20200201 
0b55d9badcb01e113b19fa4b3b1fa399d1705ce467d19dc87d1a3ce9b76d814a
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# docker container ps
CONTAINER ID        IMAGE                 COMMAND             CREATED             STATUS              PORTS               NAMES
0b55d9badcb0        nginx:v0.1-20200201   "nginx"             3 seconds ago       Up 3 seconds        80/tcp, 443/tcp     nginx-web01-net-bridge
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,SLAVE,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast master bond0 state UP group default qlen 1000
    link/ether 00:0c:29:fa:bd:9a brd ff:ff:ff:ff:ff:ff
3: eth1: <BROADCAST,MULTICAST,SLAVE,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast master bond1 state UP group default qlen 1000
    link/ether 00:0c:29:fa:bd:a4 brd ff:ff:ff:ff:ff:ff
4: eth2: <BROADCAST,MULTICAST,SLAVE,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast master bond0 state UP group default qlen 1000
    link/ether 00:0c:29:fa:bd:9a brd ff:ff:ff:ff:ff:ff
5: eth3: <BROADCAST,MULTICAST,SLAVE,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast master bond1 state UP group default qlen 1000
    link/ether 00:0c:29:fa:bd:a4 brd ff:ff:ff:ff:ff:ff
6: bond0: <BROADCAST,MULTICAST,MASTER,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether 00:0c:29:fa:bd:9a brd ff:ff:ff:ff:ff:ff
    inet 172.200.3.101/21 brd 172.200.7.255 scope global bond0
       valid_lft forever preferred_lft forever
7: bond1: <BROADCAST,MULTICAST,MASTER,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether 00:0c:29:fa:bd:a4 brd ff:ff:ff:ff:ff:ff
    inet 192.200.3.101/21 brd 192.200.7.255 scope global bond1
       valid_lft forever preferred_lft forever
8: docker0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether 02:42:81:7d:84:58 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever
34: veth72e6ea8@if33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master docker0 state UP group default 
    link/ether 82:5a:ca:c9:1f:59 brd ff:ff:ff:ff:ff:ff link-netnsid 0
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# 
```

\[root@docker101.yinzhengjie.org.cn ~\]# docker container run -it --name nginx-web01-net-bridge -d nginx:v0.1-20200201　　　　#默认使用bridge模式，该模式会自动创建一块虚拟网卡哟~

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@docker101.yinzhengjie.org.cn ~]# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,SLAVE,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast master bond0 state UP group default qlen 1000
    link/ether 00:0c:29:fa:bd:9a brd ff:ff:ff:ff:ff:ff
3: eth1: <BROADCAST,MULTICAST,SLAVE,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast master bond1 state UP group default qlen 1000
    link/ether 00:0c:29:fa:bd:a4 brd ff:ff:ff:ff:ff:ff
4: eth2: <BROADCAST,MULTICAST,SLAVE,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast master bond0 state UP group default qlen 1000
    link/ether 00:0c:29:fa:bd:9a brd ff:ff:ff:ff:ff:ff
5: eth3: <BROADCAST,MULTICAST,SLAVE,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast master bond1 state UP group default qlen 1000
    link/ether 00:0c:29:fa:bd:a4 brd ff:ff:ff:ff:ff:ff
6: bond0: <BROADCAST,MULTICAST,MASTER,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether 00:0c:29:fa:bd:9a brd ff:ff:ff:ff:ff:ff
    inet 172.200.3.101/21 brd 172.200.7.255 scope global bond0
       valid_lft forever preferred_lft forever
7: bond1: <BROADCAST,MULTICAST,MASTER,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether 00:0c:29:fa:bd:a4 brd ff:ff:ff:ff:ff:ff
    inet 192.200.3.101/21 brd 192.200.7.255 scope global bond1
       valid_lft forever preferred_lft forever
8: docker0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether 02:42:81:7d:84:58 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever
34: veth72e6ea8@if33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master docker0 state UP group default 
    link/ether 82:5a:ca:c9:1f:59 brd ff:ff:ff:ff:ff:ff link-netnsid 0
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# docker container ps -a
CONTAINER ID        IMAGE                 COMMAND             CREATED             STATUS              PORTS               NAMES
0b55d9badcb0        nginx:v0.1-20200201   "nginx"             3 seconds ago       Up 3 seconds        80/tcp, 443/tcp     nginx-web01-net-bridge
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# docker container rm -fv `docker container ps -a -q`
0b55d9badcb0
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,SLAVE,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast master bond0 state UP group default qlen 1000
    link/ether 00:0c:29:fa:bd:9a brd ff:ff:ff:ff:ff:ff
3: eth1: <BROADCAST,MULTICAST,SLAVE,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast master bond1 state UP group default qlen 1000
    link/ether 00:0c:29:fa:bd:a4 brd ff:ff:ff:ff:ff:ff
4: eth2: <BROADCAST,MULTICAST,SLAVE,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast master bond0 state UP group default qlen 1000
    link/ether 00:0c:29:fa:bd:9a brd ff:ff:ff:ff:ff:ff
5: eth3: <BROADCAST,MULTICAST,SLAVE,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast master bond1 state UP group default qlen 1000
    link/ether 00:0c:29:fa:bd:a4 brd ff:ff:ff:ff:ff:ff
6: bond0: <BROADCAST,MULTICAST,MASTER,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether 00:0c:29:fa:bd:9a brd ff:ff:ff:ff:ff:ff
    inet 172.200.3.101/21 brd 172.200.7.255 scope global bond0
       valid_lft forever preferred_lft forever
7: bond1: <BROADCAST,MULTICAST,MASTER,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether 00:0c:29:fa:bd:a4 brd ff:ff:ff:ff:ff:ff
    inet 192.200.3.101/21 brd 192.200.7.255 scope global bond1
       valid_lft forever preferred_lft forever
8: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default 
    link/ether 02:42:81:7d:84:58 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft fore
```

\[root@docker101.yinzhengjie.org.cn ~\]# docker container rm -fv \`docker container ps -a -q\`　　　　　　　　　　　　　　　　　　　　#删除容器时虚拟网卡也随之删除啦

![](https://img2018.cnblogs.com/i-beta/795254/202002/795254-20200201101523329-1867065541.png)

**2>.创建容器时显式指定网络模式为host**

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@docker102.yinzhengjie.org.cn ~]# docker container rm -fv `docker container ps -a -q`　　　　　　　　　　　　　　　　　　　　#为了试验方便，我们将其它容器删除掉
11bc2df8a985
9c4086710d57
[root@docker102.yinzhengjie.org.cn ~]# 
```

\[root@docker102.yinzhengjie.org.cn ~\]# docker container rm -fv \`docker container ps -a -q\`　　　　　　　　　　　　　　　　　　　　#为了试验方便，我们将其它容器删除掉

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@docker102.yinzhengjie.org.cn ~]# ss -ntl
State      Recv-Q Send-Q                                                                                          Local Address:Port                                                                                                         Peer Address:Port              
LISTEN     0      128                                                                                                         *:22                                                                                                                      *:*                  
LISTEN     0      128                                                                                                        :::22                                                                                                                     :::*                  
[root@docker102.yinzhengjie.org.cn ~]# 
[root@docker102.yinzhengjie.org.cn ~]# docker container run -it --name redis-net-host1 --net=host -d redis:latest
f53da097d378134762858c8663bd7263df91e63a01f26e5585f45348e93f3ef6
[root@docker102.yinzhengjie.org.cn ~]# 
[root@docker102.yinzhengjie.org.cn ~]# ss -ntl
State      Recv-Q Send-Q                                                                                          Local Address:Port                                                                                                         Peer Address:Port              
LISTEN     0      511                                                                                                         *:6379                                                                                                                    *:*                  
LISTEN     0      128                                                                                                         *:22                                                                                                                      *:*                  
LISTEN     0      511                                                                                                        :::6379                                                                                                                   :::*                  
LISTEN     0      128                                                                                                        :::22                                                                                                                     :::*                  
[root@docker102.yinzhengjie.org.cn ~]# 
[root@docker102.yinzhengjie.org.cn ~]# docker container ps -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
f53da097d378        redis:latest        "docker-entrypoint.s…"   16 seconds ago      Up 15 seconds                           redis-net-host1
[root@docker102.yinzhengjie.org.cn ~]# 
```

\[root@docker102.yinzhengjie.org.cn ~\]# docker container run -it --name redis-net-host1 --net=host -d redis:latest　　　　　　#第一个容器创建成功  

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@docker102.yinzhengjie.org.cn ~]# ss -ntl
State      Recv-Q Send-Q                                                                                          Local Address:Port                                                                                                         Peer Address:Port              
LISTEN     0      511                                                                                                         *:6379                                                                                                                    *:*                  
LISTEN     0      128                                                                                                         *:22                                                                                                                      *:*                  
LISTEN     0      511                                                                                                        :::6379                                                                                                                   :::*                  
LISTEN     0      128                                                                                                        :::22                                                                                                                     :::*                  
[root@docker102.yinzhengjie.org.cn ~]# 
[root@docker102.yinzhengjie.org.cn ~]# docker container ps -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
f53da097d378        redis:latest        "docker-entrypoint.s…"   16 seconds ago      Up 15 seconds                           redis-net-host1
[root@docker102.yinzhengjie.org.cn ~]# 
[root@docker102.yinzhengjie.org.cn ~]# docker container run -it --name redis-net-host2 --net=host -d redis:latest
78b42aaa9b8520680d243de85f8efc2d8c4a85ec0947a952223958d2c6886aab
[root@docker102.yinzhengjie.org.cn ~]# 
[root@docker102.yinzhengjie.org.cn ~]# docker container ps -a
CONTAINER ID        IMAGE               COMMAND                  CREATED              STATUS                     PORTS               NAMES
78b42aaa9b85        redis:latest        "docker-entrypoint.s…"   3 seconds ago        Exited (1) 3 seconds ago                       redis-net-host2
f53da097d378        redis:latest        "docker-entrypoint.s…"   About a minute ago   Up About a minute                              redis-net-host1
[root@docker102.yinzhengjie.org.cn ~]# 
[root@docker102.yinzhengjie.org.cn ~]# ss -ntl
State      Recv-Q Send-Q                                                                                          Local Address:Port                                                                                                         Peer Address:Port              
LISTEN     0      511                                                                                                         *:6379                                                                                                                    *:*                  
LISTEN     0      128                                                                                                         *:22                                                                                                                      *:*                  
LISTEN     0      511                                                                                                        :::6379                                                                                                                   :::*                  
LISTEN     0      128                                                                                                        :::22                                                                                                                     :::*                  
[root@docker102.yinzhengjie.org.cn ~]# 
```

\[root@docker102.yinzhengjie.org.cn ~\]# docker container run -it --name redis-net-host2 --net=host -d redis:latest　　　　　　#第二次启动容器时却失败啦~

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@docker102.yinzhengjie.org.cn ~]# docker container ps -a
CONTAINER ID        IMAGE               COMMAND                  CREATED              STATUS                     PORTS               NAMES
78b42aaa9b85        redis:latest        "docker-entrypoint.s…"   3 seconds ago        Exited (1) 3 seconds ago                       redis-net-host2
f53da097d378        redis:latest        "docker-entrypoint.s…"   About a minute ago   Up About a minute                              redis-net-host1
[root@docker102.yinzhengjie.org.cn ~]# 
[root@docker102.yinzhengjie.org.cn ~]# ss -ntl
State      Recv-Q Send-Q                                                                                          Local Address:Port                                                                                                         Peer Address:Port              
LISTEN     0      511                                                                                                         *:6379                                                                                                                    *:*                  
LISTEN     0      128                                                                                                         *:22                                                                                                                      *:*                  
LISTEN     0      511                                                                                                        :::6379                                                                                                                   :::*                  
LISTEN     0      128                                                                                                        :::22                                                                                                                     :::*                  
[root@docker102.yinzhengjie.org.cn ~]# 
[root@docker102.yinzhengjie.org.cn ~]# docker container logs redis-net-host2
1:C 01 Feb 2020 10:29:26.160 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
1:C 01 Feb 2020 10:29:26.160 # Redis version=5.0.7, bits=64, commit=00000000, modified=0, pid=1, just started
1:C 01 Feb 2020 10:29:26.160 # Warning: no config file specified, using the default config. In order to specify a config file use redis-server /path/to/redis.conf
1:M 01 Feb 2020 10:29:26.160 # Could not create server TCP listening socket *:6379: bind: Address already in use
[root@docker102.yinzhengjie.org.cn ~]# 
```

\[root@docker102.yinzhengjie.org.cn ~\]# docker container logs redis-net-host2　　　　　　　　　　　　　　　　　　　　　　　　　　　　#可以查看启动失败的日志信息

![](https://img2018.cnblogs.com/i-beta/795254/202002/795254-20200201104304884-1734330763.png)

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@docker101.yinzhengjie.org.cn ~]# yum -y install epel-release
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
 * base: mirrors.tuna.tsinghua.edu.cn
 * extras: mirrors.tuna.tsinghua.edu.cn
 * updates: mirror.bit.edu.cn
Resolving Dependencies
--> Running transaction check
---> Package epel-release.noarch 0:7-11 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

==============================================================================================================================================================================================================================================================================
 Package                                                               Arch                                                            Version                                                          Repository                                                       Size
==============================================================================================================================================================================================================================================================================
Installing:
 epel-release                                                          noarch                                                          7-11                                                             extras                                                           15 k

Transaction Summary
==============================================================================================================================================================================================================================================================================
Install  1 Package

Total download size: 15 k
Installed size: 24 k
Downloading packages:
epel-release-7-11.noarch.rpm                                                                                                                                                                                                                           |  15 kB  00:00:00     
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : epel-release-7-11.noarch                                                                                                                                                                                                                                   1/1 
  Verifying  : epel-release-7-11.noarch                                                                                                                                                                                                                                   1/1 

Installed:
  epel-release.noarch 0:7-11                                                                                                                                                                                                                                                  

Complete!
[root@docker101.yinzhengjie.org.cn ~]# 
```

\[root@docker101.yinzhengjie.org.cn ~\]# yum -y install epel-release

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@docker101.yinzhengjie.org.cn ~]# yum -y install redis
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
epel/x86_64/metalink                                                                                                                                                                                                                                   | 8.7 kB  00:00:00     
 * base: mirrors.tuna.tsinghua.edu.cn
 * epel: mirrors.tuna.tsinghua.edu.cn
 * extras: mirrors.tuna.tsinghua.edu.cn
 * updates: mirror.bit.edu.cn
epel                                                                                                                                                                                                                                                   | 5.4 kB  00:00:00     
(1/3): epel/x86_64/group_gz                                                                                                                                                                                                                            |  90 kB  00:00:00     
(2/3): epel/x86_64/updateinfo                                                                                                                                                                                                                          | 1.0 MB  00:00:00     
(3/3): epel/x86_64/primary_db                                                                                                                                                                                                                          | 6.9 MB  00:00:11     
Resolving Dependencies
--> Running transaction check
---> Package redis.x86_64 0:3.2.12-2.el7 will be installed
--> Processing Dependency: libjemalloc.so.1()(64bit) for package: redis-3.2.12-2.el7.x86_64
--> Running transaction check
---> Package jemalloc.x86_64 0:3.6.0-1.el7 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

==============================================================================================================================================================================================================================================================================
 Package                                                           Arch                                                            Version                                                                Repository                                                     Size
==============================================================================================================================================================================================================================================================================
Installing:
 redis                                                             x86_64                                                          3.2.12-2.el7                                                           epel                                                          544 k
Installing for dependencies:
 jemalloc                                                          x86_64                                                          3.6.0-1.el7                                                            epel                                                          105 k

Transaction Summary
==============================================================================================================================================================================================================================================================================
Install  1 Package (+1 Dependent package)

Total download size: 648 k
Installed size: 1.7 M
Downloading packages:
warning: /var/cache/yum/x86_64/7/epel/packages/jemalloc-3.6.0-1.el7.x86_64.rpm: Header V3 RSA/SHA256 Signature, key ID 352c64e5: NOKEY                                                                                                      ]  0.0 B/s |    0 B  --:--:-- ETA 
Public key for jemalloc-3.6.0-1.el7.x86_64.rpm is not installed
(1/2): jemalloc-3.6.0-1.el7.x86_64.rpm                                                                                                                                                                                                                 | 105 kB  00:00:00     
(2/2): redis-3.2.12-2.el7.x86_64.rpm                                                                                                                                                                                                                   | 544 kB  00:00:00     
------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
Total                                                                                                                                                                                                                                         936 kB/s | 648 kB  00:00:00     
Retrieving key from file:///etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-7
Importing GPG key 0x352C64E5:
 Userid     : "Fedora EPEL (7) <epel@fedoraproject.org>"
 Fingerprint: 91e9 7d7c 4a5e 96f1 7f3e 888f 6a2f aea2 352c 64e5
 Package    : epel-release-7-11.noarch (@extras)
 From       : /etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-7
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : jemalloc-3.6.0-1.el7.x86_64                                                                                                                                                                                                                                1/2 
  Installing : redis-3.2.12-2.el7.x86_64                                                                                                                                                                                                                                  2/2 
  Verifying  : redis-3.2.12-2.el7.x86_64                                                                                                                                                                                                                                  1/2 
  Verifying  : jemalloc-3.6.0-1.el7.x86_64                                                                                                                                                                                                                                2/2 

Installed:
  redis.x86_64 0:3.2.12-2.el7                                                                                                                                                                                                                                                 

Dependency Installed:
  jemalloc.x86_64 0:3.6.0-1.el7                                                                                                                                                                                                                                               

Complete!
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# 
```

\[root@docker101.yinzhengjie.org.cn ~\]# yum -y install redis

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@docker101.yinzhengjie.org.cn ~]# redis-cli -h docker102.yinzhengjie.org.cn
docker102.yinzhengjie.org.cn:6379> info
# Server
redis_version:5.0.7
redis_git_sha1:00000000
redis_git_dirty:0
redis_build_id:b671f408463590b9
redis_mode:standalone
os:Linux 3.10.0-957.el7.x86_64 x86_64
arch_bits:64
multiplexing_api:epoll
atomicvar_api:atomic-builtin
gcc_version:8.3.0
process_id:1
run_id:77cfc6e3c18987d1752a9dfcb48cecd537a979b0
tcp_port:6379
uptime_in_seconds:1170
uptime_in_days:0
hz:10
configured_hz:10
lru_clock:3495764
executable:/data/redis-server
config_file:

# Clients
connected_clients:1
client_recent_max_input_buffer:2
client_recent_max_output_buffer:0
blocked_clients:0

# Memory
used_memory:854160
used_memory_human:834.14K
used_memory_rss:11182080
used_memory_rss_human:10.66M
used_memory_peak:854160
used_memory_peak_human:834.14K
used_memory_peak_perc:100.12%
used_memory_overhead:841982
used_memory_startup:792288
used_memory_dataset:12178
used_memory_dataset_perc:19.68%
allocator_allocated:1355032
allocator_active:1609728
allocator_resident:8474624
total_system_memory:4123009024
total_system_memory_human:3.84G
used_memory_lua:37888
used_memory_lua_human:37.00K
used_memory_scripts:0
used_memory_scripts_human:0B
number_of_cached_scripts:0
maxmemory:0
maxmemory_human:0B
maxmemory_policy:noeviction
allocator_frag_ratio:1.19
allocator_frag_bytes:254696
allocator_rss_ratio:5.26
allocator_rss_bytes:6864896
rss_overhead_ratio:1.32
rss_overhead_bytes:2707456
mem_fragmentation_ratio:13.77
mem_fragmentation_bytes:10369920
mem_not_counted_for_evict:0
mem_replication_backlog:0
mem_clients_slaves:0
mem_clients_normal:49694
mem_aof_buffer:0
mem_allocator:jemalloc-5.1.0
active_defrag_running:0
lazyfree_pending_objects:0

# Persistence
loading:0
rdb_changes_since_last_save:0
rdb_bgsave_in_progress:0
rdb_last_save_time:1580552898
rdb_last_bgsave_status:ok
rdb_last_bgsave_time_sec:-1
rdb_current_bgsave_time_sec:-1
rdb_last_cow_size:0
aof_enabled:0
aof_rewrite_in_progress:0
aof_rewrite_scheduled:0
aof_last_rewrite_time_sec:-1
aof_current_rewrite_time_sec:-1
aof_last_bgrewrite_status:ok
aof_last_write_status:ok
aof_last_cow_size:0

# Stats
total_connections_received:1
total_commands_processed:1
instantaneous_ops_per_sec:0
total_net_input_bytes:31
total_net_output_bytes:11468
instantaneous_input_kbps:0.00
instantaneous_output_kbps:0.00
rejected_connections:0
sync_full:0
sync_partial_ok:0
sync_partial_err:0
expired_keys:0
expired_stale_perc:0.00
expired_time_cap_reached_count:0
evicted_keys:0
keyspace_hits:0
keyspace_misses:0
pubsub_channels:0
pubsub_patterns:0
latest_fork_usec:0
migrate_cached_sockets:0
slave_expires_tracked_keys:0
active_defrag_hits:0
active_defrag_misses:0
active_defrag_key_hits:0
active_defrag_key_misses:0

# Replication
role:master
connected_slaves:0
master_replid:9d2adf0cdd5cad67d6173cf06ca8bed2b9dc4237
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:0
second_repl_offset:-1
repl_backlog_active:0
repl_backlog_size:1048576
repl_backlog_first_byte_offset:0
repl_backlog_histlen:0

# CPU
used_cpu_sys:0.748182
used_cpu_user:0.957966
used_cpu_sys_children:0.001870
used_cpu_user_children:0.000882

# Cluster
cluster_enabled:0

# Keyspace
docker102.yinzhengjie.org.cn:6379> 
docker102.yinzhengjie.org.cn:6379> quit
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# 
```

\[root@docker101.yinzhengjie.org.cn ~\]# redis-cli -h docker102.yinzhengjie.org.cn

**3>.**创建容器时显式指定网络模式为none(使用场景相对较少)****

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@docker101.yinzhengjie.org.cn ~]# docker image ls
REPOSITORY               TAG                 IMAGE ID            CREATED             SIZE
nginx                    v0.1-20200201       1a8b4f68e96a        3 hours ago         449MB
centos-haproxy           v1.8.20             1858fe05d96f        8 days ago          606MB
registry                 latest              708bc6af7e5e        8 days ago          25.8MB
tomcat-app01             v0.1                bf45c22f2d5b        9 days ago          983MB
tomcat-base              8.5.50              9ff79f369094        10 days ago         968MB
jdk-base                 1.8.0_231           0f63a97ddc85        10 days ago         953MB
centos-base              7.6.1810            b4931fd9ace2        10 days ago         551MB
centos                   centos7.6.1810      f1cb7c7d58b7        10 months ago       202MB
lorel/docker-stress-ng   latest              1ae56ccafe55        3 years ago         8.1MB
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# docker container run --name nginx-net-none --net=none -v /yinzhengjie:/yinzhengjie -d nginx:v0.1-20200201 
3efb710ed1868dd514758bee0b2864957f914741f445ba47102845897535fe3c
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# docker container ps -a
CONTAINER ID        IMAGE                 COMMAND             CREATED             STATUS              PORTS               NAMES
3efb710ed186        nginx:v0.1-20200201   "nginx"             3 seconds ago       Up 3 seconds                            nginx-net-none
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# docker container exec -it nginx-net-none bash
[root@3efb710ed186 /]# 
[root@3efb710ed186 /]# ifconfig -a
lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

[root@3efb710ed186 /]# 
[root@3efb710ed186 /]# ll /yinzhengjie/
total 0
drwxr-xr-x 4 root root 34 Jan 29 16:48 data
drwxr-xr-x 3 root root 24 Jan 21 23:27 softwares
[root@3efb710ed186 /]# 
[root@3efb710ed186 /]# exit 
exit
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# ll /yinzhengjie/
total 0
drwxr-xr-x 4 root root 34 Jan 30 00:48 data
drwxr-xr-x 3 root root 24 Jan 22 07:27 softwares
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# 
```

\[root@docker101.yinzhengjie.org.cn ~\]# docker container run --name nginx-net-none --net=none -v /yinzhengjie:/yinzhengjie -d nginx:v0.1-20200201

![](https://img2018.cnblogs.com/i-beta/795254/202002/795254-20200201110453730-1592732461.png)

**4>.****创建容器时显式指定网络模式为一个已经存在的容器的网络空间(即两个容器共用同一块网卡设备)******

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@docker101.yinzhengjie.org.cn ~]# docker container rm -fv `docker container ps -a -q`
90ddc49aa5e8
972600e88309
[root@docker101.yinzhengjie.org.cn ~]# 
```

\[root@docker101.yinzhengjie.org.cn ~\]# docker container rm -fv \`docker container ps -a -q\`　　　　　　　　　　#删除无效的容器

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@docker101.yinzhengjie.org.cn ~]# docker container ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# docker container run -it --name nginx-net-bridge --net=bridge -v /yinzhengjie:/yinzhengjie -d tomcat-base:8.5.50 
232919619cc7eaf19bc6ed064087ca0d6933816c345a2b47c1f8ebcc0aa88814
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# docker container ps -a
CONTAINER ID        IMAGE                COMMAND             CREATED             STATUS              PORTS               NAMES
232919619cc7        tomcat-base:8.5.50   "/bin/bash"         3 seconds ago       Up 2 seconds                            nginx-net-bridge
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# docker container exec -it nginx-net-bridge bash
[root@232919619cc7 /]# 
[root@232919619cc7 /]# ifconfig 
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 172.17.0.2  netmask 255.255.0.0  broadcast 172.17.255.255
        ether 02:42:ac:11:00:02  txqueuelen 0  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

[root@232919619cc7 /]# 
[root@232919619cc7 /]# ll /yinzhengjie/
total 0
drwxr-xr-x 4 root root 34 Jan 30 00:48 data
drwxr-xr-x 3 root root 24 Jan 22 07:27 softwares
[root@232919619cc7 /]# 
[root@232919619cc7 /]# exit 
exit
[root@docker101.yinzhengjie.org.cn ~]# 
```

\[root@docker101.yinzhengjie.org.cn ~\]# docker container run -it --name nginx-net-bridge --net=bridge -v /yinzhengjie:/yinzhengjie -d tomcat-base:8.5.50　　　　　　　　#创建一个容器

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@docker101.yinzhengjie.org.cn ~]# docker container run -it --name nginx-net-bridge2 --net=container:nginx-net-bridge -d nginx:v0.1-20200201 
d0d41584cd60a5990c59380d0013db80f6fea202e449035646ee3c65906c8a05
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# docker container ps -a
CONTAINER ID        IMAGE                 COMMAND             CREATED             STATUS              PORTS               NAMES
d0d41584cd60        nginx:v0.1-20200201   "nginx"             3 seconds ago       Up 2 seconds                            nginx-net-bridge2
232919619cc7        tomcat-base:8.5.50    "/bin/bash"         31 seconds ago      Up 30 seconds                           nginx-net-bridge
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# docker container exec -it nginx-net-bridge2 bash
[root@232919619cc7 /]# 
[root@232919619cc7 /]# ifconfig 
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 172.17.0.2  netmask 255.255.0.0  broadcast 172.17.255.255
        ether 02:42:ac:11:00:02  txqueuelen 0  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

[root@232919619cc7 /]# 
[root@232919619cc7 /]# ll /yinzhengjie
ls: cannot access /yinzhengjie: No such file or directory
[root@232919619cc7 /]# 
[root@232919619cc7 /]# exit 
exit
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# 
```

\[root@docker101.yinzhengjie.org.cn ~\]# docker container run -it --name nginx-net-bridge2 --net=container:nginx-net-bridge -d nginx:v0.1-20200201　　　　　　　　#在创建一个容器，网络空间和上面创建的"nginx-net-bridge"容器共享

![](https://img2018.cnblogs.com/i-beta/795254/202002/795254-20200201120923803-2025025703.png)

**5>.查看宿主机默认网络模式的信息概要**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@docker101.yinzhengjie.org.cn ~]# docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
34054a7dc2e6        bridge              bridge              local
e537f0ed209d        host                host                local
408c4e463c01        none                null                local
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@docker101.yinzhengjie.org.cn ~]# docker network inspect bridge
[
    {
        "Name": "bridge",
        "Id": "34054a7dc2e6fd90f7b3038a93130db1e629398659ea84ed9b1dd2d566ba212e",
        "Created": "2020-02-01T15:21:36.488094467+08:00",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    "Subnet": "172.17.0.0/16",
                    "Gateway": "172.17.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {
            "232919619cc7eaf19bc6ed064087ca0d6933816c345a2b47c1f8ebcc0aa88814": {
                "Name": "nginx-net-bridge",
                "EndpointID": "f12d868d0dc6b4277bccb19dd1c44a33690d9f738eea6bc3d5dcf482c8ebf3da",
                "MacAddress": "02:42:ac:11:00:02",
                "IPv4Address": "172.17.0.2/16",
                "IPv6Address": ""
            }
        },
        "Options": {
            "com.docker.network.bridge.default_bridge": "true",
            "com.docker.network.bridge.enable_icc": "true",
            "com.docker.network.bridge.enable_ip_masquerade": "true",
            "com.docker.network.bridge.host_binding_ipv4": "0.0.0.0",
            "com.docker.network.bridge.name": "docker0",
            "com.docker.network.driver.mtu": "1500"
        },
        "Labels": {}
    }
]
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# 
```

\[root@docker101.yinzhengjie.org.cn ~\]# docker network inspect bridge

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@docker101.yinzhengjie.org.cn ~]# docker network inspect host
[
    {
        "Name": "host",
        "Id": "e537f0ed209d7594743c7d35a221bd09bbb6999f35fee80b44f3fa8188e0ffda",
        "Created": "2020-01-21T16:25:36.207042152+08:00",
        "Scope": "local",
        "Driver": "host",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": []
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {},
        "Options": {},
        "Labels": {}
    }
]
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# 
```

\[root@docker101.yinzhengjie.org.cn ~\]# docker network inspect host

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@docker101.yinzhengjie.org.cn ~]# docker network inspect none
[
    {
        "Name": "none",
        "Id": "408c4e463c01f8662a1f82ffde60b2a97b2aaf317779ce661197a1b10362da00",
        "Created": "2020-01-21T16:25:36.199531528+08:00",
        "Scope": "local",
        "Driver": "null",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": []
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {},
        "Options": {},
        "Labels": {}
    }
]
[root@docker101.yinzhengjie.org.cn ~]# 
```

\[root@docker101.yinzhengjie.org.cn ~\]# docker network inspect none

**二.docker跨主机互联简单实现**

```
　　跨主机互联是说A宿主机的容器可以访问B宿主机上的容器,但是前提是保证各宿主机之间的网络是可以相互通信的，然后各容器才可以通过各自的宿主机访问到对方的容器。

　　实现原理是在各宿主机上做相应的网络路由就可以实现A宿主机和容器访问B主机的目的，复杂的网络或者大型的网络可以通过Google开源的K8S进行互联。　　Docker的默认网段都是172.17.0.0/24，而且每个宿主机都是一样的,因此要做路由的前提就是个主机网络不能一致。为了避免对实验的影响，建议大家选两个"干净"的docker环境节点，或者删除之前在试验节点创建的所有容器，在个节点上执行"docker container  rm -f `docker container ps -a -q`"
```

**1>.修改docker101.yinzhengjie.org.cn节点的默认网段**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@docker101.yinzhengjie.org.cn ~]# ifconfig docker0
docker0: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
        inet 172.17.0.1  netmask 255.255.0.0  broadcast 172.17.255.255
        ether 02:42:81:7d:84:58  txqueuelen 0  (Ethernet)
        RX packets 13994  bytes 609179 (594.9 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 16608  bytes 78555306 (74.9 MiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# grep ExecStart /lib/systemd/system/docker.service 
ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# vim /lib/systemd/system/docker.service 
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# grep ExecStart /lib/systemd/system/docker.service 
ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock --bip=10.100.0.254/16
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# systemctl daemon-reload 
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# systemctl restart docker.service 
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# ifconfig docker0
docker0: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
        inet 10.100.0.254  netmask 255.255.0.0  broadcast 10.100.255.255
        ether 02:42:81:7d:84:58  txqueuelen 0  (Ethernet)
        RX packets 13994  bytes 609179 (594.9 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 16608  bytes 78555306 (74.9 MiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

[root@docker101.yinzhengjie.org.cn ~]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

**2>.**修改docker102.yinzhengjie.org.cn节点的默认网段****

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@docker102.yinzhengjie.org.cn ~]# ifconfig docker0
docker0: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
        inet 172.17.0.1  netmask 255.255.0.0  broadcast 172.17.255.255
        ether 02:42:6b:e0:f6:2e  txqueuelen 0  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

[root@docker102.yinzhengjie.org.cn ~]# 
[root@docker102.yinzhengjie.org.cn ~]# grep ExecStart /lib/systemd/system/docker.service
ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock --insecure-registry docker103.yinzhengjie.org.cn --insecure-registry docker104.yinzhengjie.org.cn
[root@docker102.yinzhengjie.org.cn ~]# 
[root@docker102.yinzhengjie.org.cn ~]# vim /lib/systemd/system/docker.service
[root@docker102.yinzhengjie.org.cn ~]# 
[root@docker102.yinzhengjie.org.cn ~]# grep ExecStart /lib/systemd/system/docker.service
ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock --insecure-registry docker103.yinzhengjie.org.cn --insecure-registry docker104.yinzhengjie.org.cn --bip=10.200.0.254/16
[root@docker102.yinzhengjie.org.cn ~]# 
[root@docker102.yinzhengjie.org.cn ~]# systemctl daemon-reload
[root@docker102.yinzhengjie.org.cn ~]# 
[root@docker102.yinzhengjie.org.cn ~]# systemctl restart docker.service
[root@docker102.yinzhengjie.org.cn ~]# 
[root@docker102.yinzhengjie.org.cn ~]# ifconfig docker0
docker0: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
        inet 10.200.0.254  netmask 255.255.0.0  broadcast 10.200.255.255
        ether 02:42:6b:e0:f6:2e  txqueuelen 0  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

[root@docker102.yinzhengjie.org.cn ~]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

**3>.**docker101.yinzhengjie.org.cn节点上运行一个容器并查看网关信息****

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@docker101.yinzhengjie.org.cn ~]# docker image pull centos:centos7.6.1810
centos7.6.1810: Pulling from library/centos
ac9208207ada: Pull complete 
Digest: sha256:62d9e1c2daa91166139b51577fe4f4f6b4cc41a3a2c7fc36bd895e2a17a3e4e6
Status: Downloaded newer image for centos:centos7.6.1810
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# docker container run -it --name centos01 -d centos:centos7.6.1810
6f147e263a544757b0c443ca7f801edb2de968c808a30d612adbfe09260ea2bf
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# docker container exec -it 6f147e263a544757b0c443ca7f801edb2de968c808a30d612adbfe09260ea2bf bash
[root@6f147e263a54 /]# 
[root@6f147e263a54 /]# yum -y install net-tools
[root@6f147e263a54 /]# 
[root@6f147e263a54 /]# ifconfig 
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 10.100.0.1  netmask 255.255.0.0  broadcast 10.100.255.255
        ether 02:42:0a:64:00:01  txqueuelen 0  (Ethernet)
        RX packets 2779  bytes 13327375 (12.7 MiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 2545  bytes 140646 (137.3 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

[root@6f147e263a54 /]# 
[root@6f147e263a54 /]# route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         10.100.0.254    0.0.0.0         UG    0      0        0 eth0
10.100.0.0      0.0.0.0         255.255.0.0     U     0      0        0 eth0
[root@6f147e263a54 /]# 
[root@6f147e263a54 /]# exit 
exit
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

**4>.****docker102.yinzhengjie.org.cn节点上运行一个容器并查看网关信息******

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@docker102.yinzhengjie.org.cn ~]# docker image pull centos:centos7.6.1810
centos7.6.1810: Pulling from library/centos
ac9208207ada: Pull complete 
Digest: sha256:62d9e1c2daa91166139b51577fe4f4f6b4cc41a3a2c7fc36bd895e2a17a3e4e6
Status: Downloaded newer image for centos:centos7.6.1810
docker.io/library/centos:centos7.6.1810
[root@docker102.yinzhengjie.org.cn ~]# 
[root@docker102.yinzhengjie.org.cn ~]# docker container run -it --name centos02 -d centos:centos7.6.1810
f7329ef9419289c4d9599c557f04518f383ced60c01e3a3749d6a18ef32244bb
[root@docker102.yinzhengjie.org.cn ~]# 
[root@docker102.yinzhengjie.org.cn ~]# docker container exec -it f7329ef9419289c4d9599c557f04518f383ced60c01e3a3749d6a18ef32244bb bash
[root@f7329ef94192 /]# 
[root@f7329ef94192 /]# yum -y install net-tools
[root@f7329ef94192 /]# 
[root@f7329ef94192 /]# ifconfig 
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 10.200.0.1  netmask 255.255.0.0  broadcast 10.200.255.255
        ether 02:42:0a:c8:00:01  txqueuelen 0  (Ethernet)
        RX packets 2429  bytes 13308486 (12.6 MiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 2345  bytes 129858 (126.8 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

[root@f7329ef94192 /]# 
[root@f7329ef94192 /]# route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         10.200.0.254    0.0.0.0         UG    0      0        0 eth0
10.200.0.0      0.0.0.0         255.255.0.0     U     0      0        0 eth0
[root@f7329ef94192 /]# 
[root@f7329ef94192 /]# exit 
exit
[root@docker102.yinzhengjie.org.cn ~]# 
[root@docker102.yinzhengjie.org.cn ~]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

**5>.******docker101.yinzhengjie.org.cn节点上添加静态路径********

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@docker101.yinzhengjie.org.cn ~]# iptables -vnL
Chain INPUT (policy ACCEPT 61 packets, 5225 bytes)
 pkts bytes target     prot opt in     out     source               destination         

Chain FORWARD (policy DROP 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination         
    0     0 DOCKER-USER  all  --  *      *       0.0.0.0/0            0.0.0.0/0           
    0     0 DOCKER-ISOLATION-STAGE-1  all  --  *      *       0.0.0.0/0            0.0.0.0/0           
    0     0 ACCEPT     all  --  *      docker0  0.0.0.0/0            0.0.0.0/0            ctstate RELATED,ESTABLISHED
    0     0 DOCKER     all  --  *      docker0  0.0.0.0/0            0.0.0.0/0           
    0     0 ACCEPT     all  --  docker0 !docker0  0.0.0.0/0            0.0.0.0/0           
    0     0 ACCEPT     all  --  docker0 docker0  0.0.0.0/0            0.0.0.0/0           

Chain OUTPUT (policy ACCEPT 42 packets, 6673 bytes)
 pkts bytes target     prot opt in     out     source               destination         

Chain DOCKER (1 references)
 pkts bytes target     prot opt in     out     source               destination         

Chain DOCKER-ISOLATION-STAGE-1 (1 references)
 pkts bytes target     prot opt in     out     source               destination         
    0     0 DOCKER-ISOLATION-STAGE-2  all  --  docker0 !docker0  0.0.0.0/0            0.0.0.0/0           
    0     0 RETURN     all  --  *      *       0.0.0.0/0            0.0.0.0/0           

Chain DOCKER-ISOLATION-STAGE-2 (1 references)
 pkts bytes target     prot opt in     out     source               destination         
    0     0 DROP       all  --  *      docker0  0.0.0.0/0            0.0.0.0/0           
    0     0 RETURN     all  --  *      *       0.0.0.0/0            0.0.0.0/0           

Chain DOCKER-USER (1 references)
 pkts bytes target     prot opt in     out     source               destination         
    0     0 RETURN     all  --  *      *       0.0.0.0/0            0.0.0.0/0           
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# iptables -A FORWARD -s 172.200.0.0/21 -j ACCEPT
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# iptables -vnL
Chain INPUT (policy ACCEPT 17 packets, 1132 bytes)
 pkts bytes target     prot opt in     out     source               destination         

Chain FORWARD (policy DROP 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination         
    0     0 DOCKER-USER  all  --  *      *       0.0.0.0/0            0.0.0.0/0           
    0     0 DOCKER-ISOLATION-STAGE-1  all  --  *      *       0.0.0.0/0            0.0.0.0/0           
    0     0 ACCEPT     all  --  *      docker0  0.0.0.0/0            0.0.0.0/0            ctstate RELATED,ESTABLISHED
    0     0 DOCKER     all  --  *      docker0  0.0.0.0/0            0.0.0.0/0           
    0     0 ACCEPT     all  --  docker0 !docker0  0.0.0.0/0            0.0.0.0/0           
    0     0 ACCEPT     all  --  docker0 docker0  0.0.0.0/0            0.0.0.0/0           
    0     0 ACCEPT     all  --  *      *       172.200.0.0/21       0.0.0.0/0           

Chain OUTPUT (policy ACCEPT 11 packets, 1308 bytes)
 pkts bytes target     prot opt in     out     source               destination         

Chain DOCKER (1 references)
 pkts bytes target     prot opt in     out     source               destination         

Chain DOCKER-ISOLATION-STAGE-1 (1 references)
 pkts bytes target     prot opt in     out     source               destination         
    0     0 DOCKER-ISOLATION-STAGE-2  all  --  docker0 !docker0  0.0.0.0/0            0.0.0.0/0           
    0     0 RETURN     all  --  *      *       0.0.0.0/0            0.0.0.0/0           

Chain DOCKER-ISOLATION-STAGE-2 (1 references)
 pkts bytes target     prot opt in     out     source               destination         
    0     0 DROP       all  --  *      docker0  0.0.0.0/0            0.0.0.0/0           
    0     0 RETURN     all  --  *      *       0.0.0.0/0            0.0.0.0/0           

Chain DOCKER-USER (1 references)
 pkts bytes target     prot opt in     out     source               destination         
    0     0 RETURN     all  --  *      *       0.0.0.0/0            0.0.0.0/0           
[root@docker101.yinzhengjie.org.cn ~]# 
```

\[root@docker101.yinzhengjie.org.cn ~\]# iptables -A FORWARD -s 172.200.0.0/21 -j ACCEPT

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@docker101.yinzhengjie.org.cn ~]# hostname -i
172.200.3.101
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         172.200.7.254   0.0.0.0         UG    0      0        0 bond0
10.100.0.0      0.0.0.0         255.255.0.0     U     0      0        0 docker0
169.254.0.0     0.0.0.0         255.255.0.0     U     1006   0        0 bond0
169.254.0.0     0.0.0.0         255.255.0.0     U     1007   0        0 bond1
172.200.0.0     0.0.0.0         255.255.248.0   U     0      0        0 bond0
192.200.0.0     0.0.0.0         255.255.248.0   U     0      0        0 bond1
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# route add -net 10.200.0.0/16 gw 172.200.3.102
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         172.200.7.254   0.0.0.0         UG    0      0        0 bond0
10.100.0.0      0.0.0.0         255.255.0.0     U     0      0        0 docker0
10.200.0.0      172.200.3.102   255.255.0.0     UG    0      0        0 bond0
169.254.0.0     0.0.0.0         255.255.0.0     U     1006   0        0 bond0
169.254.0.0     0.0.0.0         255.255.0.0     U     1007   0        0 bond1
172.200.0.0     0.0.0.0         255.255.248.0   U     0      0        0 bond0
192.200.0.0     0.0.0.0         255.255.248.0   U     0      0        0 bond1
[root@docker101.yinzhengjie.org.cn ~]#
```

\[root@docker101.yinzhengjie.org.cn ~\]# route add -net 10.200.0.0/16 gw 172.200.3.102

********6>.******docker102.yinzhengjie.org.cn节点上添加静态路径**************

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@docker102.yinzhengjie.org.cn ~]# iptables -vnL
Chain INPUT (policy ACCEPT 60 packets, 5185 bytes)
 pkts bytes target     prot opt in     out     source               destination         

Chain FORWARD (policy DROP 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination         
    0     0 DOCKER-USER  all  --  *      *       0.0.0.0/0            0.0.0.0/0           
    0     0 DOCKER-ISOLATION-STAGE-1  all  --  *      *       0.0.0.0/0            0.0.0.0/0           
    0     0 ACCEPT     all  --  *      docker0  0.0.0.0/0            0.0.0.0/0            ctstate RELATED,ESTABLISHED
    0     0 DOCKER     all  --  *      docker0  0.0.0.0/0            0.0.0.0/0           
    0     0 ACCEPT     all  --  docker0 !docker0  0.0.0.0/0            0.0.0.0/0           
    0     0 ACCEPT     all  --  docker0 docker0  0.0.0.0/0            0.0.0.0/0           

Chain OUTPUT (policy ACCEPT 42 packets, 6481 bytes)
 pkts bytes target     prot opt in     out     source               destination         

Chain DOCKER (1 references)
 pkts bytes target     prot opt in     out     source               destination         

Chain DOCKER-ISOLATION-STAGE-1 (1 references)
 pkts bytes target     prot opt in     out     source               destination         
    0     0 DOCKER-ISOLATION-STAGE-2  all  --  docker0 !docker0  0.0.0.0/0            0.0.0.0/0           
    0     0 RETURN     all  --  *      *       0.0.0.0/0            0.0.0.0/0           

Chain DOCKER-ISOLATION-STAGE-2 (1 references)
 pkts bytes target     prot opt in     out     source               destination         
    0     0 DROP       all  --  *      docker0  0.0.0.0/0            0.0.0.0/0           
    0     0 RETURN     all  --  *      *       0.0.0.0/0            0.0.0.0/0           

Chain DOCKER-USER (1 references)
 pkts bytes target     prot opt in     out     source               destination         
    0     0 RETURN     all  --  *      *       0.0.0.0/0            0.0.0.0/0           
[root@docker102.yinzhengjie.org.cn ~]# 
[root@docker102.yinzhengjie.org.cn ~]# iptables -A FORWARD -s 172.200.0.0/21 -j ACCEPT
[root@docker102.yinzhengjie.org.cn ~]# 
[root@docker102.yinzhengjie.org.cn ~]# iptables -vnL
Chain INPUT (policy ACCEPT 8 packets, 528 bytes)
 pkts bytes target     prot opt in     out     source               destination         

Chain FORWARD (policy DROP 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination         
    0     0 DOCKER-USER  all  --  *      *       0.0.0.0/0            0.0.0.0/0           
    0     0 DOCKER-ISOLATION-STAGE-1  all  --  *      *       0.0.0.0/0            0.0.0.0/0           
    0     0 ACCEPT     all  --  *      docker0  0.0.0.0/0            0.0.0.0/0            ctstate RELATED,ESTABLISHED
    0     0 DOCKER     all  --  *      docker0  0.0.0.0/0            0.0.0.0/0           
    0     0 ACCEPT     all  --  docker0 !docker0  0.0.0.0/0            0.0.0.0/0           
    0     0 ACCEPT     all  --  docker0 docker0  0.0.0.0/0            0.0.0.0/0           
    0     0 ACCEPT     all  --  *      *       172.200.0.0/21       0.0.0.0/0           

Chain OUTPUT (policy ACCEPT 5 packets, 716 bytes)
 pkts bytes target     prot opt in     out     source               destination         

Chain DOCKER (1 references)
 pkts bytes target     prot opt in     out     source               destination         

Chain DOCKER-ISOLATION-STAGE-1 (1 references)
 pkts bytes target     prot opt in     out     source               destination         
    0     0 DOCKER-ISOLATION-STAGE-2  all  --  docker0 !docker0  0.0.0.0/0            0.0.0.0/0           
    0     0 RETURN     all  --  *      *       0.0.0.0/0            0.0.0.0/0           

Chain DOCKER-ISOLATION-STAGE-2 (1 references)
 pkts bytes target     prot opt in     out     source               destination         
    0     0 DROP       all  --  *      docker0  0.0.0.0/0            0.0.0.0/0           
    0     0 RETURN     all  --  *      *       0.0.0.0/0            0.0.0.0/0           

Chain DOCKER-USER (1 references)
 pkts bytes target     prot opt in     out     source               destination         
    0     0 RETURN     all  --  *      *       0.0.0.0/0            0.0.0.0/0           
[root@docker102.yinzhengjie.org.cn ~]# 
```

\[root@docker102.yinzhengjie.org.cn ~\]# iptables -A FORWARD -s 172.200.0.0/21 -j ACCEPT

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@docker102.yinzhengjie.org.cn ~]# hostname -i
172.200.3.102
[root@docker102.yinzhengjie.org.cn ~]# 
[root@docker102.yinzhengjie.org.cn ~]# route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         172.200.7.254   0.0.0.0         UG    0      0        0 bond0
10.200.0.0      0.0.0.0         255.255.0.0     U     0      0        0 docker0
169.254.0.0     0.0.0.0         255.255.0.0     U     1006   0        0 bond0
169.254.0.0     0.0.0.0         255.255.0.0     U     1007   0        0 bond1
172.200.0.0     0.0.0.0         255.255.248.0   U     0      0        0 bond0
192.200.0.0     0.0.0.0         255.255.248.0   U     0      0        0 bond1
[root@docker102.yinzhengjie.org.cn ~]# 
[root@docker102.yinzhengjie.org.cn ~]# route add -net 10.100.0.0/16 gw 172.200.3.101
[root@docker102.yinzhengjie.org.cn ~]# 
[root@docker102.yinzhengjie.org.cn ~]# route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         172.200.7.254   0.0.0.0         UG    0      0        0 bond0
10.100.0.0      172.200.3.101   255.255.0.0     UG    0      0        0 bond0
10.200.0.0      0.0.0.0         255.255.0.0     U     0      0        0 docker0
169.254.0.0     0.0.0.0         255.255.0.0     U     1006   0        0 bond0
169.254.0.0     0.0.0.0         255.255.0.0     U     1007   0        0 bond1
172.200.0.0     0.0.0.0         255.255.248.0   U     0      0        0 bond0
192.200.0.0     0.0.0.0         255.255.248.0   U     0      0        0 bond1
[root@docker102.yinzhengjie.org.cn ~]# 
```

\[root@docker102.yinzhengjie.org.cn ~\]# route add -net 10.100.0.0/16 gw 172.200.3.101

********7>.两个节点相互ping测试******** 

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@docker101.yinzhengjie.org.cn ~]# docker container ps
CONTAINER ID        IMAGE                   COMMAND             CREATED             STATUS              PORTS               NAMES
6f147e263a54        centos:centos7.6.1810   "/bin/bash"         About an hour ago   Up 8 minutes                            centos01
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# docker container exec -it centos01 bash
[root@6f147e263a54 /]# 
[root@6f147e263a54 /]# ifconfig 
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 10.100.0.1  netmask 255.255.0.0  broadcast 10.100.255.255
        ether 02:42:0a:64:00:01  txqueuelen 0  (Ethernet)
        RX packets 11  bytes 1050 (1.0 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 11  bytes 910 (910.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

[root@6f147e263a54 /]# 
[root@6f147e263a54 /]# ping 10.200.0.1
PING 10.200.0.1 (10.200.0.1) 56(84) bytes of data.
64 bytes from 10.200.0.1: icmp_seq=1 ttl=62 time=0.481 ms
64 bytes from 10.200.0.1: icmp_seq=2 ttl=62 time=0.729 ms
64 bytes from 10.200.0.1: icmp_seq=3 ttl=62 time=1.71 ms
64 bytes from 10.200.0.1: icmp_seq=4 ttl=62 time=0.583 ms
64 bytes from 10.200.0.1: icmp_seq=5 ttl=62 time=1.04 ms
64 bytes from 10.200.0.1: icmp_seq=6 ttl=62 time=1.66 ms
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

********![](https://img2018.cnblogs.com/i-beta/795254/202002/795254-20200201164925525-1247223216.png)********

**8>.在任意一个容器中使用tcpdump工具进行抓包**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@docker101.yinzhengjie.org.cn ~]# docker container ps 
CONTAINER ID        IMAGE                   COMMAND             CREATED             STATUS              PORTS               NAMES
6f147e263a54        centos:centos7.6.1810   "/bin/bash"         About an hour ago   Up 19 minutes                           centos01
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# docker container exec -it centos01  bash
[root@6f147e263a54 /]# 
[root@6f147e263a54 /]# yum -y install tcpdump
[root@6f147e263a54 /]# 
[root@6f147e263a54 /]# ifconfig 
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 10.100.0.1  netmask 255.255.0.0  broadcast 10.100.255.255
        ether 02:42:0a:64:00:01  txqueuelen 0  (Ethernet)
        RX packets 288  bytes 27244 (26.6 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 288  bytes 27104 (26.4 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

[root@6f147e263a54 /]# 
[root@6f147e263a54 /]# tcpdump -i eth0 -vnn icmp
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

![](https://img2018.cnblogs.com/i-beta/795254/202002/795254-20200201165603834-1779113121.png)

当你的才华还撑不起你的野心的时候，你就应该静下心来学习。当你的能力还驾驭不了你的目标的时候，你就应该沉下心来历练。问问自己，想要怎样的人生。 欢迎加入基础架构自动化运维：598432640，大数据SRE进阶之路：959042252，DevOps进阶之路：526991186