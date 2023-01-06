　　　　　　　　　　**Linux的namespace和cgroups简介**

　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　**作者：尹正杰**

**版权声明：原创作品，谢绝转载！否则将追究法律责任。**

**一.Linux Namespace技术**

```
　　Namespace是Linux系统的底层概念,在内核层实现,即有一些不同类型的命名空间被部署在核内,各个docker容器运行在同一个docker主进程并且共用同一个宿主机系统内核。　　各docker容器运行在宿主机的用户空间,每个容器都要有类似于虚拟机一样的相互隔离的运行空间,但是容器技术是在一个进程内实现运行指定服务的运行环境,并且还可以保护宿主机内核不受其他进程的干扰和影响,如文件系统空间,网络空间,进程空间等，目前主要通过以下技术实现容器运行空间的相互隔离。
```

![](https://img2018.cnblogs.com/i-beta/795254/202001/795254-20200112142201943-166904609.png)

**1>.MNT Namespace** 

```
　　每个容器都要有独立的根文件系统用户空间,以实现在容器里面启动服务并且使用容器的运行环境，即一个宿主机是ubuntu的服务器，可以在里面启动一个centos运行环境的容器并且在里面启动一个Nginx服务,此Nginx运行时使用的运行环境就是centos系统目录的运行环境,但是在容器里面不能访问宿主机的资源,宿主机是使用了chroot技术把容器锁定到一个指的运行目录里面。
```

 ![](https://img2018.cnblogs.com/i-beta/795254/202001/795254-20200112163717524-1401754394.png)

**2>.IPC Namespace** 

```
　　一个容器内的进程间通信,允许一个容器内的不同进程的(内存,缓存等)数据访问，但是不能跨容器访问其他容器的数据 。
```

**3>.UTS Namespace** 

```
　　UTS namespace(UNIX Timesharing System 包含了运行内核的名称、版本底层体系结构类型等信息）用于系统标识,其中包含了hostname和域名domainname,它使得一个容器拥有属于自己hostname标识,这个主机名标识独立于宿主机系统和其上的他容器 。
```

 ![](https://img2018.cnblogs.com/i-beta/795254/202001/795254-20200112165057742-1057735359.png)

**4>.PID Namespace** 

```
　　Linux系统中，有一个PID为1的进程(init/systemd)是其他所有进程的父,那么 在每个容器内也要有一个父进程来管理其下属的子进程,那么多个容器的进程通的PID namespace进程隔离(比如PID编号重复、器内的主进程与回收子进程等)。
```

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
root@docker101:~# docker images                          #查看现有的镜像
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
nginx               latest              c7460dfcab50        2 days ago          126MB
centos              latest              0f3e07c0138f        3 months ago        220MB
root@docker101:~# 
root@docker101:~# docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
968e7ecc39f2        centos              "/bin/bash"         3 hours ago         Up 3 hours                              keen_meitner
root@docker101:~# 
root@docker101:~# docker run -d -it nginx                  #基于nginx镜像运行一个容器
44edd3477c0d7380ab23dc23f00055b7a17eecd483a666c47e11fac6786a2f3e
root@docker101:~# 
root@docker101:~# docker ps                                #查看正在运行的容器
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
44edd3477c0d        nginx               "nginx -g 'daemon of…"   4 seconds ago       Up 2 seconds        80/tcp              stupefied_driscoll
968e7ecc39f2        centos              "/bin/bash"              3 hours ago         Up 3 hours                              keen_meitner
root@docker101:~# 
root@docker101:~# ps -ef | grep docker                     #我们现在宿主机上查看所有docker相关的容器，我们通过目录前缀就可以判断出PID为"7183"的进程是咱们刚刚启动的"nginx"容器。
root       6171      1  0 08:18 ?        00:00:11 /usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock
root       6553   4451  0 08:24 ?        00:00:02 containerd-shim -namespace moby -workdir /var/lib/containerd/io.containerd.runtime.v1.linux/moby/968e7ecc39f277a3d3f98b658f8f496de622edccfa4ef45d8ec64c46f5012d4c -address /run/containerd/containerd.sock -containerd-binary /usr/bin/containerd -runtime-root /var/run/docker/runtime-runc
root       7831   4451  0 11:27 ?        00:00:00 containerd-shim -namespace moby -workdir /var/lib/containerd/io.containerd.runtime.v1.linux/moby/44edd3477c0d7380ab23dc23f00055b7a17eecd483a666c47e11fac6786a2f3e -address /run/containerd/containerd.sock -containerd-binary /usr/bin/containerd -runtime-root /var/run/docker/runtime-runc
root       7923   6955  0 11:27 pts/1    00:00:00 grep --color=auto docker
root@docker101:~# 
root@docker101:~# ps -ef | grep 4451
root       4451      1  0 08:03 ?        00:01:40 /usr/bin/containerd
root       6553   4451  0 08:24 ?        00:00:02 containerd-shim -namespace moby -workdir /var/lib/containerd/io.containerd.runtime.v1.linux/moby/968e7ecc39f277a3d3f98b658f8f496de622edccfa4ef45d8ec64c46f5012d4c -address /run/containerd/containerd.sock -containerd-binary /usr/bin/containerd -runtime-root /var/run/docker/runtime-runc
root       7831   4451  0 11:27 ?        00:00:00 containerd-shim -namespace moby -workdir /var/lib/containerd/io.containerd.runtime.v1.linux/moby/44edd3477c0d7380ab23dc23f00055b7a17eecd483a666c47e11fac6786a2f3e -address /run/containerd/containerd.sock -containerd-binary /usr/bin/containerd -runtime-root /var/run/docker/runtime-runc
root       8136   6955  0 11:43 pts/1    00:00:00 grep --color=auto 4451
root@docker101:~# 
root@docker101:~# ps -ef | grep 7831                       #查看nginx容器的进程信息。
root       7831   4451  0 11:27 ?        00:00:00 containerd-shim -namespace moby -workdir /var/lib/containerd/io.containerd.runtime.v1.linux/moby/44edd3477c0d7380ab23dc23f00055b7a17eecd483a666c47e11fac6786a2f3e -address /run/containerd/containerd.sock -containerd-binary /usr/bin/containerd -runtime-root /var/run/docker/runtime-runc
root       7862   7831  0 11:27 pts/0    00:00:00 nginx: master process nginx -g daemon off;
root       7925   6955  0 11:29 pts/1    00:00:00 grep --color=auto 7831
root@docker101:~# 
root@docker101:~# pstree -p 7831                           #很明显，在宿主机上"7831"运行的是"nginx"容器,而该容器中运行了nginx的主进程(pid为7862)和工作进程(pid为7904)
containerd-shim(7831)─┬─nginx(7862)───nginx(7904)
                      ├─{containerd-shim}(7832)
                      ├─{containerd-shim}(7833)
                      ├─{containerd-shim}(7834)
                      ├─{containerd-shim}(7835)
                      ├─{containerd-shim}(7836)
                      ├─{containerd-shim}(7837)
                      ├─{containerd-shim}(7838)
                      ├─{containerd-shim}(7840)
                      ├─{containerd-shim}(7892)
                      └─{containerd-shim}(8110)
root@docker101:~# 
```

宿主机上查看PID信息

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
root@docker101:~# docker ps　　　　　　　　　　　　　　　　　　　　　　#我们可以看到基于nginx镜像的容器处于正常运行状态。
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
44edd3477c0d        nginx               "nginx -g 'daemon of…"   28 minutes ago      Up 28 minutes       80/tcp              stupefied_driscoll
968e7ecc39f2        centos              "/bin/bash"              4 hours ago         Up 4 hours                              keen_meitner
root@docker101:~# 
root@docker101:~# 
root@docker101:~# docker exec -it 44edd3477c0d bash
root@44edd3477c0d:/# 
root@44edd3477c0d:/# cat /etc/issue　　　　　　　　　　　　　　　　　#查看当前nginx的镜像在哪个操作系统开发的，我们看到的信息是在Debian系统开发的，这很正常。
Debian GNU/Linux 10 \n \l

root@44edd3477c0d:/# 
root@44edd3477c0d:/# uname -a　　　　　　　　　　　　　　　　　　　　　#很显然，容器除了有自己的主机名，内核版本使用的是宿主机ubuntu的。
Linux 44edd3477c0d 4.15.0-74-generic #84-Ubuntu SMP Thu Dec 19 08:06:28 UTC 2019 x86_64 GNU/Linux
root@44edd3477c0d:/# 
root@44edd3477c0d:/# apt-get update                    　　　　　　#必须先更新软件源,否则无法执行下面的安装命令。
root@44edd3477c0d:/# 
root@44edd3477c0d:/# apt-get -y install net-tools   　　　　　　　　#Debian系统安装网络工具
root@44edd3477c0d:/# 
root@44edd3477c0d:/# apt-get -y install curl         　　　　　　   #Debian系统安装curl命令
root@44edd3477c0d:/# 
root@44edd3477c0d:/# apt-get -y install procps   　　　　　　　　　　#Debian系统安装top命令
root@44edd3477c0d:/# 
root@44edd3477c0d:/# apt-get -y install iputils-ping   　　　　　　#Debian系统安装ping命令
root@44edd3477c0d:/# 
root@44edd3477c0d:/# top　　　　　　　　　　　　　　　　　　　　　　　　 #不难发现，使用top命令我们可以看到PID为1的进程竟然是Nginx的主进程。
top - 12:09:29 up  6:27,  0 users,  load average: 0.00, 0.00, 0.00
Tasks:   4 total,   1 running,   3 sleeping,   0 stopped,   0 zombie
%Cpu(s):  0.0 us,  0.0 sy,  0.0 ni,100.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
MiB Mem :   3921.8 total,   2027.7 free,    359.8 used,   1534.2 buff/cache
MiB Swap:   8192.0 total,   8192.0 free,      0.0 used.   3312.7 avail Mem 

   PID USER      PR  NI    VIRT    RES    SHR S  %CPU  %MEM     TIME+ COMMAND                                                                                                                                                                                                
     1 root      20   0   10632   5468   4760 S   0.0   0.1   0:00.30 nginx                                                                                                                                                                                                  
     7 nginx     20   0   11120   2564   1436 S   0.0   0.1   0:00.00 nginx                                                                                                                                                                                                  
    37 root      20   0    3988   3284   2784 S   0.0   0.1   0:00.32 bash                                                                                                                                                                                                   
  2968 root      20   0    8024   3132   2664 R   0.0   0.1   0:00.00 top                                                                                                                                                                                                    
root@44edd3477c0d:/# 
root@44edd3477c0d:/# ps -ef | grep nginx
root          1      0  0 11:27 pts/0    00:00:00 nginx: master process nginx -g daemon off;
nginx         7      1  0 11:27 pts/0    00:00:00 nginx: worker process
root       2973     37  0 12:11 pts/1    00:00:00 grep nginx
root@44edd3477c0d:/# 
root@44edd3477c0d:/# netstat -untalp　　　　　　　　　　#可以看到nginx进程监听了80端口
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      1/nginx: master pro 
root@44edd3477c0d:/# 
root@44edd3477c0d:/# curl -I 127.0.0.1　　　　　　　　 #nginx的服务也是可以正常范围的
HTTP/1.1 200 OK
Server: nginx/1.17.7
Date: Sun, 12 Jan 2020 12:17:40 GMT
Content-Type: text/html
Content-Length: 612
Last-Modified: Tue, 24 Dec 2019 13:07:53 GMT
Connection: keep-alive
ETag: "5e020da9-264"
Accept-Ranges: bytes

root@44edd3477c0d:/# 
root@44edd3477c0d:/# pstree -p 1
nginx(1)---nginx(7)
root@44edd3477c0d:/# 
root@44edd3477c0d:/# nginx -s reload　　　　　　　　　　　　　　　　#我们可以对nginx进行重新加载，并不会让当前容器结束运行。
2020/01/12 12:12:22 [notice] 2975#2975: signal process started
root@44edd3477c0d:/# 
root@44edd3477c0d:/# ps -ef | grep nginx
root          1      0  0 11:27 pts/0    00:00:00 nginx: master process nginx -g daemon off;
nginx      2976      1  0 12:12 pts/0    00:00:00 nginx: worker process
root       2978     37  0 12:12 pts/1    00:00:00 grep nginx
root@44edd3477c0d:/# 
root@44edd3477c0d:/# 
root@44edd3477c0d:/# pstree -p 1
nginx(1)---nginx(2976)
root@44edd3477c0d:/# 
root@44edd3477c0d:/# nginx -s stop　　　　　　　　　　　　　　　　#如果我们将nginx容器中的nginx进程给停掉后，发现该容器也会跟着停止使用了。
2020/01/12 12:14:15 [notice] 2983#2983: signal process started
root@44edd3477c0d:/# root@docker101:~# 
root@docker101:~# 
root@docker101:~# docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
968e7ecc39f2        centos              "/bin/bash"         4 hours ago         Up 4 hours                              keen_meitner
root@docker101:~# 
root@docker101:~# docker ps -a　　　　　　　　　　　　　　　　　　#可以看到当前nginx容器是退出状态的。
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                      PORTS               NAMES
44edd3477c0d        nginx               "nginx -g 'daemon of…"   47 minutes ago      Exited (0) 11 seconds ago                       stupefied_driscoll
968e7ecc39f2        centos              "/bin/bash"              4 hours ago         Up 4 hours                                      keen_meitner
root@docker101:~# 
root@docker101:~# docker start 44edd3477c0d　　　　　　　　　　#当然，咱们也可以再次将该容器启动
44edd3477c0d
root@docker101:~# 
root@docker101:~# docker ps -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
44edd3477c0d        nginx               "nginx -g 'daemon of…"   48 minutes ago      Up 2 seconds        80/tcp              stupefied_driscoll
968e7ecc39f2        centos              "/bin/bash"              4 hours ago         Up 4 hours                              keen_meitner
root@docker101:~# 
root@docker101:~#
```

查看容器中的PID相关信息

**5>.Net Namespace** 

```
　　每一个容器都类似于虚拟机一样有自己的网卡,监听端口,TCP/IP协议栈等,Docker使用network namespace启动一个vethX接口,这样你的容器将拥有它自己的桥接ip地址,通常是docker0,而docker0实质就是Linux的虚拟网桥,网桥是在OSI七层模型的数据链路网络设备,通过mac地址对网络进行划分,并且在不同网络直接传递数据。
```

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
root@docker101:~# docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
44edd3477c0d        nginx               "nginx -g 'daemon of…"   About an hour ago   Up 16 minutes       80/tcp              stupefied_driscoll
968e7ecc39f2        centos              "/bin/bash"              4 hours ago         Up 4 hours                              keen_meitner
root@docker101:~# 
root@docker101:~# docker exec -it 44edd3477c0d bash
root@44edd3477c0d:/# 
root@44edd3477c0d:/# ifconfig 
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 172.17.0.3  netmask 255.255.0.0  broadcast 172.17.255.255
        ether 02:42:ac:11:00:03  txqueuelen 0  (Ethernet)
        RX packets 13  bytes 1006 (1006.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 44  bytes 4822 (4.7 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 44  bytes 4822 (4.7 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

root@44edd3477c0d:/# 
root@44edd3477c0d:/# exit 
exit
root@docker101:~# 
root@docker101:~# docker exec -it 968e7ecc39f2 bash
[root@968e7ecc39f2 /]# 
[root@968e7ecc39f2 /]# yum -y install net-tools
[root@968e7ecc39f2 /]# 
[root@968e7ecc39f2 /]# ifconfig 
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 172.17.0.2  netmask 255.255.0.0  broadcast 172.17.255.255
        ether 02:42:ac:11:00:02  txqueuelen 0  (Ethernet)
        RX packets 3708  bytes 15399326 (14.6 MiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 3356  bytes 185759 (181.4 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

[root@968e7ecc39f2 /]# 
[root@968e7ecc39f2 /]# ping 172.17.0.3　　　　　　　　　　　　#很显然，同一个宿主机的不同容器默认是可以相互通信的。
PING 172.17.0.3 (172.17.0.3) 56(84) bytes of data.
bytes from 172.17.0.3: icmp_seq=1 ttl=64 time=0.101 ms
bytes from 172.17.0.3: icmp_seq=2 ttl=64 time=0.042 ms
bytes from 172.17.0.3: icmp_seq=3 ttl=64 time=0.108 ms
bytes from 172.17.0.3: icmp_seq=4 ttl=64 time=0.052 ms
bytes from 172.17.0.3: icmp_seq=5 ttl=64 time=0.112 ms
^C
--- 172.17.0.3 ping statistics ---
packets transmitted, 5 received, 0% packet loss, time 83ms
rtt min/avg/max/mdev = 0.042/0.083/0.112/0.029 ms
[root@968e7ecc39f2 /]# 
[root@968e7ecc39f2 /]# route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         172.17.0.1      0.0.0.0         UG    0      0        0 eth0
172.17.0.0      0.0.0.0         255.255.0.0     U     0      0        0 eth0
[root@968e7ecc39f2 /]# 
[root@968e7ecc39f2 /]# 
```

同一个宿主机的不同容器默认是可以相互通信的

![](https://img2018.cnblogs.com/i-beta/795254/202001/795254-20200112204032019-1873669551.png)

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
root@docker101:~# ifconfig 
docker0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 172.17.0.1  netmask 255.255.0.0  broadcast 172.17.255.255
        inet6 fe80::42:87ff:febc:3cd8  prefixlen 64  scopeid 0x20<link>
        ether 02:42:87:bc:3c:d8  txqueuelen 0  (Ethernet)
        RX packets 5364  bytes 225742 (225.7 KB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 6098  bytes 28280444 (28.2 MB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

ens33: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.6.101  netmask 255.255.248.0  broadcast 192.168.7.255
        inet6 fe80::20c:29ff:fe57:8cb7  prefixlen 64  scopeid 0x20<link>
        ether 00:0c:29:57:8c:b7  txqueuelen 1000  (Ethernet)
        RX packets 179980  bytes 248819555 (248.8 MB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 37062  bytes 4513196 (4.5 MB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 314  bytes 31398 (31.3 KB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 314  bytes 31398 (31.3 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

veth47b028a: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet6 fe80::2ce5:b3ff:fe43:2cc4  prefixlen 64  scopeid 0x20<link>
        ether 2e:e5:b3:43:2c:c4  txqueuelen 0  (Ethernet)
        RX packets 7  bytes 574 (574.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 24  bytes 1832 (1.8 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

vethed7471a: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet6 fe80::9cef:51ff:fe7a:fd0c  prefixlen 64  scopeid 0x20<link>
        ether 9e:ef:51:7a:fd:0c  txqueuelen 0  (Ethernet)
        RX packets 3364  bytes 186375 (186.3 KB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 3719  bytes 15400120 (15.4 MB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

root@docker101:~# 
root@docker101:~# 
```

查看宿主机的网卡信息，如上图所示。

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
root@docker101:~# apt-get -y install bridge-utils                   　　　　　　#安装查看宿主机网桥的命令，安装后才能使用下面的"brctl"命令
root@docker101:~# 
root@docker101:~# brctl show
bridge name    bridge id        STP enabled    interfaces
docker0        8000.024287bc3cd8    no        veth47b028a
                            vethed7471a
root@docker101:~# 
```

查看宿主机的桥接设备

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
root@docker101:~# iptables -t nat -vnL　　　　　　　　　　　　　　　　　　　　　　　　#查看宿主机的iptables规则。
Chain PREROUTING (policy ACCEPT 60 packets, 3937 bytes)
 pkts bytes target     prot opt in     out     source               destination         
   52 DOCKER     all  --  *      *       0.0.0.0/0            0.0.0.0/0            ADDRTYPE match dst-type LOCAL

Chain INPUT (policy ACCEPT 9 packets, 676 bytes)
 pkts bytes target     prot opt in     out     source               destination         

Chain OUTPUT (policy ACCEPT 68 packets, 5251 bytes)
 pkts bytes target     prot opt in     out     source               destination         
    0 DOCKER     all  --  *      *       0.0.0.0/0           !127.0.0.0/8          ADDRTYPE match dst-type LOCAL

Chain POSTROUTING (policy ACCEPT 69 packets, 5335 bytes)
 pkts bytes target     prot opt in     out     source               destination         
 3177 MASQUERADE  all  --  *      !docker0  172.17.0.0/16        0.0.0.0/0           

Chain DOCKER (2 references)
 pkts bytes target     prot opt in     out     source               destination         
    0 RETURN     all  --  docker0 *       0.0.0.0/0            0.0.0.0/0           
root@docker101:~# 
root@docker101:~# iptables  -vnL
Chain INPUT (policy ACCEPT 25019 packets, 126M bytes)
 pkts bytes target     prot opt in     out     source               destination         

Chain FORWARD (policy DROP 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination         
11436   28M DOCKER-USER  all  --  *      *       0.0.0.0/0            0.0.0.0/0           
11436   28M DOCKER-ISOLATION-STAGE-1  all  --  *      *       0.0.0.0/0            0.0.0.0/0           
 6080   28M ACCEPT     all  --  *      docker0  0.0.0.0/0            0.0.0.0/0            ctstate RELATED,ESTABLISHED
    1    84 DOCKER     all  --  *      docker0  0.0.0.0/0            0.0.0.0/0           
 5355  225K ACCEPT     all  --  docker0 !docker0  0.0.0.0/0            0.0.0.0/0           
    1    84 ACCEPT     all  --  docker0 docker0  0.0.0.0/0            0.0.0.0/0           

Chain OUTPUT (policy ACCEPT 21954 packets, 3062K bytes)
 pkts bytes target     prot opt in     out     source               destination         

Chain DOCKER (1 references)
 pkts bytes target     prot opt in     out     source               destination         

Chain DOCKER-ISOLATION-STAGE-1 (1 references)
 pkts bytes target     prot opt in     out     source               destination         
 5355  225K DOCKER-ISOLATION-STAGE-2  all  --  docker0 !docker0  0.0.0.0/0            0.0.0.0/0           
11436   28M RETURN     all  --  *      *       0.0.0.0/0            0.0.0.0/0           

Chain DOCKER-ISOLATION-STAGE-2 (1 references)
 pkts bytes target     prot opt in     out     source               destination         
    0     0 DROP       all  --  *      docker0  0.0.0.0/0            0.0.0.0/0           
 5355  225K RETURN     all  --  *      *       0.0.0.0/0            0.0.0.0/0           

Chain DOCKER-USER (1 references)
 pkts bytes target     prot opt in     out     source               destination         
11436   28M RETURN     all  --  *      *       0.0.0.0/0            0.0.0.0/0           
root@docker101:~# 
root@docker101:~# 
```

查看宿主机的iptables规则，docker网络通信默认就是基于iptables规则实现的。docker的逻辑网络如下图所示。

![](https://img2018.cnblogs.com/i-beta/795254/202001/795254-20200112205054376-591178807.png)

**6>.User Namespace**

```
　　各个容器内可能会出现重名的用户和用户组名称,或重复的用户UID或者GID,那么怎隔离各个容器内的用户空间呢？
　　User Namespace允许在各个宿主机的各个容器空间内创建相同的用户名以及相同的用户UID和GID,只是会用户的作用范围限制在每个容器内，即A容器和B容器可以有相同的用户名称和ID的账户,但是此用户的有效范围仅是当前容器内，不能访问另外一个容器内的文件系统，即相互隔离,互不影响,永不相见 。
```

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
root@docker101:~# 
root@docker101:~# cat /etc/passwd
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
systemd-network:x:100:102:systemd Network Management,,,:/run/systemd/netif:/usr/sbin/nologin
systemd-resolve:x:101:103:systemd Resolver,,,:/run/systemd/resolve:/usr/sbin/nologin
syslog:x:102:106::/home/syslog:/usr/sbin/nologin
messagebus:x:103:107::/nonexistent:/usr/sbin/nologin
_apt:x:104:65534::/nonexistent:/usr/sbin/nologin
lxd:x:105:65534::/var/lib/lxd/:/bin/false
uuidd:x:106:110::/run/uuidd:/usr/sbin/nologin
dnsmasq:x:107:65534:dnsmasq,,,:/var/lib/misc:/usr/sbin/nologin
landscape:x:108:112::/var/lib/landscape:/usr/sbin/nologin
pollinate:x:109:1::/var/cache/pollinate:/bin/false
sshd:x:110:65534::/run/sshd:/usr/sbin/nologin
jason:x:1000:1000:jason:/home/jason:/bin/bash
root@docker101:~# 
root@docker101:~# 
root@docker101:~# docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
44edd3477c0d        nginx               "nginx -g 'daemon of…"   3 hours ago         Up 2 hours          80/tcp              stupefied_driscoll
968e7ecc39f2        centos              "/bin/bash"              6 hours ago         Up 6 hours                              keen_meitner
root@docker101:~# 
root@docker101:~# docker exec -it 968e7ecc39f2 bash
[root@968e7ecc39f2 /]# 
[root@968e7ecc39f2 /]# cat /etc/passwd
root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/bin:/sbin/nologin
daemon:x:2:2:daemon:/sbin:/sbin/nologin
adm:x:3:4:adm:/var/adm:/sbin/nologin
lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin
sync:x:5:0:sync:/sbin:/bin/sync
shutdown:x:6:0:shutdown:/sbin:/sbin/shutdown
halt:x:7:0:halt:/sbin:/sbin/halt
mail:x:8:12:mail:/var/spool/mail:/sbin/nologin
operator:x:11:0:operator:/root:/sbin/nologin
games:x:12:100:games:/usr/games:/sbin/nologin
ftp:x:14:50:FTP User:/var/ftp:/sbin/nologin
nobody:x:65534:65534:Kernel Overflow User:/:/sbin/nologin
dbus:x:81:81:System message bus:/:/sbin/nologin
systemd-coredump:x:999:997:systemd Core Dumper:/:/sbin/nologin
systemd-resolve:x:193:193:systemd Resolver:/:/sbin/nologin
[root@968e7ecc39f2 /]# 
[root@968e7ecc39f2 /]# exit 
exit
root@docker101:~# 
root@docker101:~# docker exec -it 44edd3477c0d bash
root@44edd3477c0d:/# 
root@44edd3477c0d:/# 
root@44edd3477c0d:/# cat /etc/passwd
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
_apt:x:100:65534::/nonexistent:/usr/sbin/nologin
nginx:x:101:101:nginx user,,,:/nonexistent:/bin/false
root@44edd3477c0d:/# 
root@44edd3477c0d:/# 
```

每个容器内部都有超级管理员root及其它普通用户，且与其它容器ID相同，但并不会相互影响，因为每个容器内部的用户只作用于其所在的容器,如下图所示。

![](https://img2018.cnblogs.com/i-beta/795254/202001/795254-20200112220946257-1926459684.png)

**二.Linux control groups**

**1>.什么是Linux Cgroups**

```
　　一个容器如果不对其做任何资源限制,则宿主机会允许其占用无限大的内存空间,有时候会因为代码bug程序会一直申请内存,直到把宿主机内存占完。　　为了避免此类的问题出现，宿主机有必要对容器进行资源分配限制，比如CPU，内存等，Linux Cgroups的全称是Linux Control Groups，它最主要的作用就是限制一个进程组能够使用的资源上限,包括CPU,内存,磁盘,网络带宽等等。　　此外，Linux Cgroups还能够对进程优先级设置，以及将进程挂起和恢复等操作。
```

**2>.验证系统**Linux Cgroups****

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@computing121.yinzhengjie.org.cn ~]# cat /etc/redhat-release 
CentOS Linux release 7.2.1511 (Core) 
[root@computing121.yinzhengjie.org.cn ~]# 
[root@computing121.yinzhengjie.org.cn ~]# uname -r
3.10.0-327.el7.x86_64
[root@computing121.yinzhengjie.org.cn ~]# 
[root@computing121.yinzhengjie.org.cn ~]# uname -m
x86_64
[root@computing121.yinzhengjie.org.cn ~]# 
[root@computing121.yinzhengjie.org.cn ~]# cat /boot/config-3.10.0-327.el7.x86_64 | grep -i cgroup
CONFIG_CGROUPS=y
# CONFIG_CGROUP_DEBUG is not set
CONFIG_CGROUP_FREEZER=y
CONFIG_CGROUP_DEVICE=y
CONFIG_CGROUP_CPUACCT=y
CONFIG_CGROUP_HUGETLB=y
CONFIG_CGROUP_PERF=y
CONFIG_CGROUP_SCHED=y
CONFIG_BLK_CGROUP=y
# CONFIG_DEBUG_BLK_CGROUP is not set
CONFIG_NETFILTER_XT_MATCH_CGROUP=m
CONFIG_NET_CLS_CGROUP=y
CONFIG_NETPRIO_CGROUP=m
[root@computing121.yinzhengjie.org.cn ~]# 
[root@computing121.yinzhengjie.org.cn ~]# 
[root@computing121.yinzhengjie.org.cn ~]# cat /boot/config-3.10.0-327.el7.x86_64 | grep -i cgroup | wc -l
13
[root@computing121.yinzhengjie.org.cn ~]# 
[root@computing121.yinzhengjie.org.cn ~]# 
```

CentOS7.2 Cgroups

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
root@docker101:~# hostname
docker101.yinzhengjie.org.cn
root@docker101:~# 
root@docker101:~# uname -r
4.15.0-74-generic
root@docker101:~# 
root@docker101:~# uname -m
x86_64
root@docker101:~# 
root@docker101:~# uname -a
Linux docker101.yinzhengjie.org.cn 4.15.0-74-generic #84-Ubuntu SMP Thu Dec 19 08:06:28 UTC 2019 x86_64 x86_64 x86_64 GNU/Linux
root@docker101:~# 
root@docker101:~# 
root@docker101:~# cat /boot/config-4.15.0-74-generic | grep -i cgroup
CONFIG_CGROUPS=y
CONFIG_BLK_CGROUP=y
# CONFIG_DEBUG_BLK_CGROUP is not set
CONFIG_CGROUP_WRITEBACK=y
CONFIG_CGROUP_SCHED=y
CONFIG_CGROUP_PIDS=y
CONFIG_CGROUP_RDMA=y
CONFIG_CGROUP_FREEZER=y
CONFIG_CGROUP_HUGETLB=y
CONFIG_CGROUP_DEVICE=y
CONFIG_CGROUP_CPUACCT=y
CONFIG_CGROUP_PERF=y
CONFIG_CGROUP_BPF=y
# CONFIG_CGROUP_DEBUG is not set
CONFIG_SOCK_CGROUP_DATA=y
CONFIG_NETFILTER_XT_MATCH_CGROUP=m
CONFIG_NET_CLS_CGROUP=m
CONFIG_CGROUP_NET_PRIO=y
CONFIG_CGROUP_NET_CLASSID=y
root@docker101:~# 
root@docker101:~# 
root@docker101:~# cat /boot/config-4.15.0-74-generic | grep -i cgroup | wc -l
19
root@docker101:~# 
```

Ubuntu18.04 Cgroups

 ![](https://img2018.cnblogs.com/i-beta/795254/202001/795254-20200112221256864-86451830.png)

```
　　Cgroups在内核层默认已经开启，从Centos(如上图所示)和Ubuntu(如下图所示)对比结果来看,显然内核较新的ubuntu支持的功能更多。
```

![](https://img2018.cnblogs.com/i-beta/795254/202001/795254-20200112221315482-834077453.png)

**3>.查看系统cgroups**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
root@docker101:~# ll /sys/fs/cgroup/
total 0
drwxr-xr-x 15 root root 380 Jan 12 05:41 ./
drwxr-xr-x 10 root root   0 Jan 12 05:41 ../
dr-xr-xr-x  5 root root   0 Jan 12 05:41 blkio/　　　　　　　　　　　　　　　　　　　　　　#块设备IO限制。
lrwxrwxrwx  1 root root  11 Jan 12 05:41 cpu -> cpu,cpuacct/　　　　　　　　　　　　　　#使用调度程序为cgroup任务提供cpu的访问。
lrwxrwxrwx  1 root root  11 Jan 12 05:41 cpuacct -> cpu,cpuacct/　　　　　　　　　　　 #产生cgroup任务的cpu资源报告。
dr-xr-xr-x  5 root root   0 Jan 12 05:41 cpu,cpuacct/
dr-xr-xr-x  3 root root   0 Jan 12 05:41 cpuset/　　　　　　　　　　　　　　　　　　　　　 #如果是多核心的cpu，这个子系统会为cgroup任务分配单独的cpu和内存。
dr-xr-xr-x  5 root root   0 Jan 12 05:41 devices/　　　　　　　　　　　　　　　　　　　　　#允许或拒绝cgroup任务对设备的访问。
dr-xr-xr-x  3 root root   0 Jan 12 05:41 freezer/　　　　　　　　　　　　　　　　　　　　　#暂停和恢复cgroup任务。
dr-xr-xr-x  3 root root   0 Jan 12 05:41 hugetlb/　　　　　　　　　　　　　　　　　　
dr-xr-xr-x  5 root root   0 Jan 12 05:41 memory/　　　　　　　　　　　　　　　　　　　　　 #设置每个cgroup的内存限制以及产生内存资源报告。
lrwxrwxrwx  1 root root  16 Jan 12 05:41 net_cls -> net_cls,net_prio/　　　　　　　　　#标记每个网络包以及提供cgroup方便是使用。
dr-xr-xr-x  3 root root   0 Jan 12 05:41 net_cls,net_prio/
lrwxrwxrwx  1 root root  16 Jan 12 05:41 net_prio -> net_cls,net_prio/
dr-xr-xr-x  3 root root   0 Jan 12 05:41 perf_event/　　　　　　　　　　　　　　　　　　　#增加了对每个group的监测跟踪的能力，可以监测属于某个特定的group的所有线程以及运行在特定CPU上的线程。
dr-xr-xr-x  5 root root   0 Jan 12 05:41 pids/
dr-xr-xr-x  2 root root   0 Jan 12 05:41 rdma/
dr-xr-xr-x  6 root root   0 Jan 12 05:41 systemd/
dr-xr-xr-x  5 root root   0 Jan 12 05:41 unified/
root@docker101:~# 
root@docker101:~# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

当你的才华还撑不起你的野心的时候，你就应该静下心来学习。当你的能力还驾驭不了你的目标的时候，你就应该沉下心来历练。问问自己，想要怎样的人生。 欢迎加入基础架构自动化运维：598432640，大数据SRE进阶之路：959042252，DevOps进阶之路：526991186