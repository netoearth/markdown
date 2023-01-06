　　　　　　　　　　　　　　**Docker存储卷篇**

　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　**作者：尹正杰**

**版权声明：原创作品，谢绝转载！否则将追究法律责任。**

**一.写时复制(COW)机制**

**![](https://img2018.cnblogs.com/blog/795254/201910/795254-20191019064029838-1729152255.png)**

```
所谓写时复制的效果如上图所示：　　Docker镜像由多个只读层叠加而成，启动容器时，Docker会加载只读镜像层并在镜像栈顶部添加一个读写层。　　如果运行中的容器修改了现有的一个已经存在的文件，那该文件将会从读写层下面的只读层复制到读写层，该文件的只读版本依然存在，只是已经被读写层中该文件的副本所隐藏，此即"读写复制(COW)"机制。
```

**二.数据卷(Data Volume)概述**

 **1>.什么是存储卷(volume)**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
　　关闭并重启容器，其数据不受影响；但删除Docker容器，则其更改将会全部丢失。原因就是我们上面提到的写时复制机制，这个写时复制机制存在于容器，数据默认都保存在容器内。　　Docker设计存在的问题：　　　　1>.存储于联合文件系统中，不易于宿主机访问;　　　　2>.容器间数据共享不便;　　　　3>.删除容器其数据会丢失;　　解决方案:"存储卷(volume)"　　　　如下图所示:"卷"是容器上的一个或多个"目录",此类目录可绕过联合文件系统，于宿主机上的某个目录"绑定(关联)"
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

 ![](https://img2018.cnblogs.com/blog/795254/201910/795254-20191019070620174-1012454442.png)

 **2>.存储卷(volume)的作用**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
　　1.数据卷为持久或共享数据提供了几个有用的特性：　　　　1>.Volume于容器(container)初始化之时即会创建，由base image提供的卷中的数据于此期间完成复制；　　　　2>.数据卷可以在容器之间共享和重用;　　　　3>.更新镜像(image)时不会包括对数据卷的更改;　　　　4>.即使容器(container)自身被删除，数据卷仍保持不变。　　2.Volume的初衷是独立于容器的生命周期实现数据持久化：　　　　1>.默认情况下删除容器之时既不会删除卷，也不会对哪怕未被引用的卷做垃圾回收操作;　　　　2>.如果我们删除容器时使用了特殊的选项(类似于linux的"userdel -r"选项)也可以一并删除容器的存储卷，但是生成环境我们不会这样做。　　3.卷为docker提供了独立于容器的数据管理机制(如下图所示):　　　　1>.可以把"镜像"想像成静态文件，例如"程序",把卷类比为动态内容，例如"数据"；于是，镜像可以重用，而卷可以共享;　　　　2>.卷实现了"程序(镜像)"和"数据(卷)"分离，以及"程序(镜像)"和"制作镜像的主机"分离，用户制作镜像时无须再考虑镜像运行的容器所在的主机的环境;
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

![](https://img2018.cnblogs.com/blog/795254/201910/795254-20191019072926023-1910207688.png)

 **3>.docker存储卷类型(Volume types)**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
　　如下图所示，Docker有两种类型的卷，每种类型都在容器中存在一个挂载点，但其在宿主机上的位置有所不同：　　　　1>.绑定挂载卷(Bind mount volume)　　　　　　指向主机文件系统上用户指定位置的卷。换句话说，在宿主机上的路径你需要人工指定一个特定路径，再容器上的路径也需要指定一个特定路径，让这两个已知路径建立关联关系。随着容器删除，存储卷数据并不会丢失。　　　　2>.docker管理的卷(Docker managed volume)　　　　　　Docker守护进程在Docker拥有的主机文件系统的一部分中创建管理卷。换句话说，我们只需要在容器内指定挂载点是什么，而绑定的是宿主机哪个路径下的目录我们不需要管，由容器引擎的docker daemon自行创建一个空目录或者使用一个已存在目录与你的存储卷路径建立关联关系。随着容器删除，存储卷数据也随着丢失了。
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

 ![](https://img2018.cnblogs.com/blog/795254/201910/795254-20191019080325895-1514077843.png)

**三.在容器这种使用Volumes**

**1>.Docker managed volume案例**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@node101.yinzhengjie.org.cn ~]# docker run --name c1 -it --rm busybox
/ # ls /
bin   dev   etc   home  proc  root  sys   tmp   usr   var
/ # 
/ # exit
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# docker run --name c1 -it -v /data --rm busybox　　　　　　#注意，我们使用"-v"选项来指定c1容器内docker管理卷存放为位置，而具体对应宿主机的哪个路径我们无需关心，而是由docker daemon进程自行创建，我们可以通过相关命令查看对应的具体路径。
/ # 
/ # ls /　　　　　　　　　　#很上面的容器相比，很显然，咱们这个容器多出了一个"/data"目录，该目录即为docker managed volume实例。
bin   data  dev   etc   home  proc  root  sys   tmp   usr   var
/ # 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

![](https://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif)

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@node101.yinzhengjie.org.cn ~]# docker container ls
CONTAINER ID IMAGE COMMAND CREATED STATUS PORTS NAMES
d574ca3e42a6 busybox "sh" 11 minutes ago Up 11 minutes c1
[root@node101.yinzhengjie.org.cn ~]#
[root@node101.yinzhengjie.org.cn ~]# docker inspect c1                           #查看c1容器的详细信息，注意观察keys为"Mounts"的"Source"属性对应的值，它所指向的路径就是c1容器"/data"目录所在宿主机的存储路径。
[
    {
        "Id": "d574ca3e42a6cc08e009ba44b3c7bcd467362ff10c71d9461211a7e9687e2c09",
        "Created": "2019-10-19T01:12:06.848411027Z",
        "Path": "sh",
        "Args": [],
        "State": {
            "Status": "running",
            "Running": true,
            "Paused": false,
            "Restarting": false,
            "OOMKilled": false,
            "Dead": false,
            "Pid": 7124,
            "ExitCode": 0,
            "Error": "",
            "StartedAt": "2019-10-19T01:12:07.402652338Z",
            "FinishedAt": "0001-01-01T00:00:00Z"
        },
        "Image": "sha256:19485c79a9bbdca205fce4f791efeaa2a103e23431434696cc54fdd939e9198d",
        "ResolvConfPath": "/var/lib/docker/containers/d574ca3e42a6cc08e009ba44b3c7bcd467362ff10c71d9461211a7e9687e2c09/resolv.conf",
        "HostnamePath": "/var/lib/docker/containers/d574ca3e42a6cc08e009ba44b3c7bcd467362ff10c71d9461211a7e9687e2c09/hostname",
        "HostsPath": "/var/lib/docker/containers/d574ca3e42a6cc08e009ba44b3c7bcd467362ff10c71d9461211a7e9687e2c09/hosts",
        "LogPath": "/var/lib/docker/containers/d574ca3e42a6cc08e009ba44b3c7bcd467362ff10c71d9461211a7e9687e2c09/d574ca3e42a6cc08e009ba4
4b3c7bcd467362ff10c71d9461211a7e9687e2c09-json.log",        "Name": "/c1",
        "RestartCount": 0,
        "Driver": "overlay2",
        "Platform": "linux",
        "MountLabel": "",
        "ProcessLabel": "",
        "AppArmorProfile": "",
        "ExecIDs": null,
        "HostConfig": {
            "Binds": null,
            "ContainerIDFile": "",
            "LogConfig": {
                "Type": "json-file",
                "Config": {}
            },
            "NetworkMode": "default",
            "PortBindings": {},
            "RestartPolicy": {
                "Name": "no",
                "MaximumRetryCount": 0
            },
            "AutoRemove": true,
            "VolumeDriver": "",
            "VolumesFrom": null,
            "CapAdd": null,
            "CapDrop": null,
            "Capabilities": null,
            "Dns": [],
            "DnsOptions": [],
            "DnsSearch": [],
            "ExtraHosts": null,
            "GroupAdd": null,
            "IpcMode": "private",
            "Cgroup": "",
            "Links": null,
            "OomScoreAdj": 0,
            "PidMode": "",
            "Privileged": false,
            "PublishAllPorts": false,
            "ReadonlyRootfs": false,
            "SecurityOpt": null,
            "UTSMode": "",
            "UsernsMode": "",
            "ShmSize": 67108864,
            "Runtime": "runc",
            "ConsoleSize": [
                0,
                0
            ],
            "Isolation": "",
            "CpuShares": 0,
            "Memory": 0,
            "NanoCpus": 0,
            "CgroupParent": "",
            "BlkioWeight": 0,
            "BlkioWeightDevice": [],
            "BlkioDeviceReadBps": null,
            "BlkioDeviceWriteBps": null,
            "BlkioDeviceReadIOps": null,
            "BlkioDeviceWriteIOps": null,
            "CpuPeriod": 0,
            "CpuQuota": 0,
            "CpuRealtimePeriod": 0,
            "CpuRealtimeRuntime": 0,
            "CpusetCpus": "",
            "CpusetMems": "",
            "Devices": [],
            "DeviceCgroupRules": null,
            "DeviceRequests": null,
            "KernelMemory": 0,
            "KernelMemoryTCP": 0,
            "MemoryReservation": 0,
            "MemorySwap": 0,
            "MemorySwappiness": null,
            "OomKillDisable": false,
            "PidsLimit": null,
            "Ulimits": null,
            "CpuCount": 0,
            "CpuPercent": 0,
            "IOMaximumIOps": 0,
            "IOMaximumBandwidth": 0,
            "MaskedPaths": [
                "/proc/asound",
                "/proc/acpi",
                "/proc/kcore",
                "/proc/keys",
                "/proc/latency_stats",
                "/proc/timer_list",
                "/proc/timer_stats",
                "/proc/sched_debug",
                "/proc/scsi",
                "/sys/firmware"
            ],
            "ReadonlyPaths": [
                "/proc/bus",
                "/proc/fs",
                "/proc/irq",
                "/proc/sys",
                "/proc/sysrq-trigger"
            ]
        },
        "GraphDriver": {
            "Data": {
                "LowerDir": "/var/lib/docker/overlay2/b91e241fd4070485ceb246e18c2d258df9666797453448b287e4253e6a5bc437-init/diff:/var/l
ib/docker/overlay2/ead7a29001855b8898300b67763a050eeb50e5c7988076f4777b56e85365169e/diff",                "MergedDir": "/var/lib/docker/overlay2/b91e241fd4070485ceb246e18c2d258df9666797453448b287e4253e6a5bc437/merged",
                "UpperDir": "/var/lib/docker/overlay2/b91e241fd4070485ceb246e18c2d258df9666797453448b287e4253e6a5bc437/diff",
                "WorkDir": "/var/lib/docker/overlay2/b91e241fd4070485ceb246e18c2d258df9666797453448b287e4253e6a5bc437/work"
            },
            "Name": "overlay2"
        },
        "Mounts": [
            {
                "Type": "volume",
                "Name": "3fc67ccb38ce00fb1257950e77ad71b5b01fc0d53f6959eda1e810e970153914",
                "Source": "/var/lib/docker/volumes/3fc67ccb38ce00fb1257950e77ad71b5b01fc0d53f6959eda1e810e970153914/_data",　　　　　　　　#注意观察这里，这就是我们c1容器"/data"存储卷对应的宿主机路径。
                "Destination": "/data",
                "Driver": "local",
                "Mode": "",
                "RW": true,
                "Propagation": ""
            }
        ],
        "Config": {
            "Hostname": "d574ca3e42a6",
            "Domainname": "",
            "User": "",
            "AttachStdin": true,
            "AttachStdout": true,
            "AttachStderr": true,
            "Tty": true,
            "OpenStdin": true,
            "StdinOnce": true,
            "Env": [
                "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
            ],
            "Cmd": [
                "sh"
            ],
            "Image": "busybox",
            "Volumes": {
                "/data": {}
            },
            "WorkingDir": "",
            "Entrypoint": null,
            "OnBuild": null,
            "Labels": {}
        },
        "NetworkSettings": {
            "Bridge": "",
            "SandboxID": "7fcfdb3cbc2d42503fbd9f44d4c9606ee484cedc26d88f7c5c7a7934efb82be5",
            "HairpinMode": false,
            "LinkLocalIPv6Address": "",
            "LinkLocalIPv6PrefixLen": 0,
            "Ports": {},
            "SandboxKey": "/var/run/docker/netns/7fcfdb3cbc2d",
            "SecondaryIPAddresses": null,
            "SecondaryIPv6Addresses": null,
            "EndpointID": "d81e297a392c3f6c73422ddb016b9c126a8f50ace798920947d331cea3c9baa0",
            "Gateway": "192.168.100.254",
            "GlobalIPv6Address": "",
            "GlobalIPv6PrefixLen": 0,
            "IPAddress": "192.168.100.1",
            "IPPrefixLen": 24,
            "IPv6Gateway": "",
            "MacAddress": "02:42:c0:a8:64:01",
            "Networks": {
                "bridge": {
                    "IPAMConfig": null,
                    "Links": null,
                    "Aliases": null,
                    "NetworkID": "7ad23e4ff4b3bbaa8cea6b89507921beb407568e021f3dad666a829ec817be7e",
                    "EndpointID": "d81e297a392c3f6c73422ddb016b9c126a8f50ace798920947d331cea3c9baa0",
                    "Gateway": "192.168.100.254",
                    "IPAddress": "192.168.100.1",
                    "IPPrefixLen": 24,
                    "IPv6Gateway": "",
                    "GlobalIPv6Address": "",
                    "GlobalIPv6PrefixLen": 0,
                    "MacAddress": "02:42:c0:a8:64:01",
                    "DriverOpts": null
                }
            }
        }
    }
]
[root@node101.yinzhengjie.org.cn ~]# cd /var/lib/docker/volumes/3fc67ccb38ce00fb1257950e77ad71b5b01fc0d53f6959eda1e810e970153914/_data                #我们进入到c1容器对应"/data"目录在宿主机的路径，就可以实现c1容器和宿主机数据共享。
[root@node101.yinzhengjie.org.cn /var/lib/docker/volumes/3fc67ccb38ce00fb1257950e77ad71b5b01fc0d53f6959eda1e810e970153914/_data]# 
[root@node101.yinzhengjie.org.cn /var/lib/docker/volumes/3fc67ccb38ce00fb1257950e77ad71b5b01fc0d53f6959eda1e810e970153914/_data]# ls
[root@node101.yinzhengjie.org.cn /var/lib/docker/volumes/3fc67ccb38ce00fb1257950e77ad71b5b01fc0d53f6959eda1e810e970153914/_data]# 
[root@node101.yinzhengjie.org.cn /var/lib/docker/volumes/3fc67ccb38ce00fb1257950e77ad71b5b01fc0d53f6959eda1e810e970153914/_data]# echo "hello container" >> test.html
[root@node101.yinzhengjie.org.cn /var/lib/docker/volumes/3fc67ccb38ce00fb1257950e77ad71b5b01fc0d53f6959eda1e810e970153914/_data]# 
[root@node101.yinzhengjie.org.cn /var/lib/docker/volumes/3fc67ccb38ce00fb1257950e77ad71b5b01fc0d53f6959eda1e810e970153914/_data]# ls
test.html
[root@node101.yinzhengjie.org.cn /var/lib/docker/volumes/3fc67ccb38ce00fb1257950e77ad71b5b01fc0d53f6959eda1e810e970153914/_data]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

**2>.Bind mount volume案例**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@node101.yinzhengjie.org.cn ~]# docker run --name c1 -it  --rm busybox
/ # 
/ # ls /
bin   dev   etc   home  proc  root  sys   tmp   usr   var
/ # 
/ # exit
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# docker run --name c1 -it -v /data/volumes/c1:/data2 --rm busybox　　　　　　#和上面有所不同，我们手动指定宿主机"/data/volumes/c1"(该路径会自动创建)路径和c1容器的"/data2"路径建立关联关系
/ # 
/ # ls /　　　　#注意，和上面的容器相比，当前容器多出来了一个"/data2"目录，该目录是咱们手动指定的"bind mount volume"卷。
bin   data2  dev   etc   home  proc  root  sys   tmp   usr   var
/ # 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node101.yinzhengjie.org.cn ~]# docker container ls
CONTAINER ID        IMAGE               COMMAND             CREATED              STATUS              PORTS               NAMES
8d6992f8321d        busybox             "sh"                About a minute ago   Up About a minute                       c1
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# docker inspect c1 　　　　　　　　　　#查看c1容器的详细信息，注意观察keys为"Mounts"的"Source"属性对应的值
[
    {
        "Id": "8d6992f8321d04bbab860550f9d48e9b37c8989052a6f671ebfbe66543f549ad",
        "Created": "2019-10-19T01:30:45.806381621Z",
        "Path": "sh",
        "Args": [],
        "State": {
            "Status": "running",
            "Running": true,
            "Paused": false,
            "Restarting": false,
            "OOMKilled": false,
            "Dead": false,
            "Pid": 7738,
            "ExitCode": 0,
            "Error": "",
            "StartedAt": "2019-10-19T01:30:46.253664572Z",
            "FinishedAt": "0001-01-01T00:00:00Z"
        },
        "Image": "sha256:19485c79a9bbdca205fce4f791efeaa2a103e23431434696cc54fdd939e9198d",
        "ResolvConfPath": "/var/lib/docker/containers/8d6992f8321d04bbab860550f9d48e9b37c8989052a6f671ebfbe66543f549ad/resolv.conf",
        "HostnamePath": "/var/lib/docker/containers/8d6992f8321d04bbab860550f9d48e9b37c8989052a6f671ebfbe66543f549ad/hostname",
        "HostsPath": "/var/lib/docker/containers/8d6992f8321d04bbab860550f9d48e9b37c8989052a6f671ebfbe66543f549ad/hosts",
        "LogPath": "/var/lib/docker/containers/8d6992f8321d04bbab860550f9d48e9b37c8989052a6f671ebfbe66543f549ad/8d6992f8321d04bbab86055
0f9d48e9b37c8989052a6f671ebfbe66543f549ad-json.log",        "Name": "/c1",
        "RestartCount": 0,
        "Driver": "overlay2",
        "Platform": "linux",
        "MountLabel": "",
        "ProcessLabel": "",
        "AppArmorProfile": "",
        "ExecIDs": null,
        "HostConfig": {
            "Binds": [
                "/data/volumes/c1:/data2"
            ],
            "ContainerIDFile": "",
            "LogConfig": {
                "Type": "json-file",
                "Config": {}
            },
            "NetworkMode": "default",
            "PortBindings": {},
            "RestartPolicy": {
                "Name": "no",
                "MaximumRetryCount": 0
            },
            "AutoRemove": true,
            "VolumeDriver": "",
            "VolumesFrom": null,
            "CapAdd": null,
            "CapDrop": null,
            "Capabilities": null,
            "Dns": [],
            "DnsOptions": [],
            "DnsSearch": [],
            "ExtraHosts": null,
            "GroupAdd": null,
            "IpcMode": "private",
            "Cgroup": "",
            "Links": null,
            "OomScoreAdj": 0,
            "PidMode": "",
            "Privileged": false,
            "PublishAllPorts": false,
            "ReadonlyRootfs": false,
            "SecurityOpt": null,
            "UTSMode": "",
            "UsernsMode": "",
            "ShmSize": 67108864,
            "Runtime": "runc",
            "ConsoleSize": [
                0,
                0
            ],
            "Isolation": "",
            "CpuShares": 0,
            "Memory": 0,
            "NanoCpus": 0,
            "CgroupParent": "",
            "BlkioWeight": 0,
            "BlkioWeightDevice": [],
            "BlkioDeviceReadBps": null,
            "BlkioDeviceWriteBps": null,
            "BlkioDeviceReadIOps": null,
            "BlkioDeviceWriteIOps": null,
            "CpuPeriod": 0,
            "CpuQuota": 0,
            "CpuRealtimePeriod": 0,
            "CpuRealtimeRuntime": 0,
            "CpusetCpus": "",
            "CpusetMems": "",
            "Devices": [],
            "DeviceCgroupRules": null,
            "DeviceRequests": null,
            "KernelMemory": 0,
            "KernelMemoryTCP": 0,
            "MemoryReservation": 0,
            "MemorySwap": 0,
            "MemorySwappiness": null,
            "OomKillDisable": false,
            "PidsLimit": null,
            "Ulimits": null,
            "CpuCount": 0,
            "CpuPercent": 0,
            "IOMaximumIOps": 0,
            "IOMaximumBandwidth": 0,
            "MaskedPaths": [
                "/proc/asound",
                "/proc/acpi",
                "/proc/kcore",
                "/proc/keys",
                "/proc/latency_stats",
                "/proc/timer_list",
                "/proc/timer_stats",
                "/proc/sched_debug",
                "/proc/scsi",
                "/sys/firmware"
            ],
            "ReadonlyPaths": [
                "/proc/bus",
                "/proc/fs",
                "/proc/irq",
                "/proc/sys",
                "/proc/sysrq-trigger"
            ]
        },
        "GraphDriver": {
            "Data": {
                "LowerDir": "/var/lib/docker/overlay2/3b5c1e2681b83025782d55c671c56e94028d17febc0058e621fdb3d2816c3de8-init/diff:/var/l
ib/docker/overlay2/ead7a29001855b8898300b67763a050eeb50e5c7988076f4777b56e85365169e/diff",                "MergedDir": "/var/lib/docker/overlay2/3b5c1e2681b83025782d55c671c56e94028d17febc0058e621fdb3d2816c3de8/merged",
                "UpperDir": "/var/lib/docker/overlay2/3b5c1e2681b83025782d55c671c56e94028d17febc0058e621fdb3d2816c3de8/diff",
                "WorkDir": "/var/lib/docker/overlay2/3b5c1e2681b83025782d55c671c56e94028d17febc0058e621fdb3d2816c3de8/work"
            },
            "Name": "overlay2"
        },
        "Mounts": [
            {
                "Type": "bind",
                "Source": "/data/volumes/c1",
                "Destination": "/data2",
                "Mode": "",
                "RW": true,
                "Propagation": "rprivate"
            }
        ],
        "Config": {
            "Hostname": "8d6992f8321d",
            "Domainname": "",
            "User": "",
            "AttachStdin": true,
            "AttachStdout": true,
            "AttachStderr": true,
            "Tty": true,
            "OpenStdin": true,
            "StdinOnce": true,
            "Env": [
                "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
            ],
            "Cmd": [
                "sh"
            ],
            "Image": "busybox",
            "Volumes": null,
            "WorkingDir": "",
            "Entrypoint": null,
            "OnBuild": null,
            "Labels": {}
        },
        "NetworkSettings": {
            "Bridge": "",
            "SandboxID": "5a5cc75df84d9544f2bcb17d7d155caac3bb70a850b44dd7a7dbafbe9affda69",
            "HairpinMode": false,
            "LinkLocalIPv6Address": "",
            "LinkLocalIPv6PrefixLen": 0,
            "Ports": {},
            "SandboxKey": "/var/run/docker/netns/5a5cc75df84d",
            "SecondaryIPAddresses": null,
            "SecondaryIPv6Addresses": null,
            "EndpointID": "b67ad92c10b5d239071a18f43b17a14a06f15ac4e39bd82f6c0eb8285f1b56f2",
            "Gateway": "192.168.100.254",
            "GlobalIPv6Address": "",
            "GlobalIPv6PrefixLen": 0,
            "IPAddress": "192.168.100.1",
            "IPPrefixLen": 24,
            "IPv6Gateway": "",
            "MacAddress": "02:42:c0:a8:64:01",
            "Networks": {
                "bridge": {
                    "IPAMConfig": null,
                    "Links": null,
                    "Aliases": null,
                    "NetworkID": "7ad23e4ff4b3bbaa8cea6b89507921beb407568e021f3dad666a829ec817be7e",
                    "EndpointID": "b67ad92c10b5d239071a18f43b17a14a06f15ac4e39bd82f6c0eb8285f1b56f2",
                    "Gateway": "192.168.100.254",
                    "IPAddress": "192.168.100.1",
                    "IPPrefixLen": 24,
                    "IPv6Gateway": "",
                    "GlobalIPv6Address": "",
                    "GlobalIPv6PrefixLen": 0,
                    "MacAddress": "02:42:c0:a8:64:01",
                    "DriverOpts": null
                }
            }
        }
    }
]
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# ls /data/volumes/c1/
[root@node101.yinzhengjie.org.cn ~]# 
```

\[root@node101.yinzhengjie.org.cn ~\]# docker inspect c1 　　　　　　　　　　#查看c1容器的详细信息，注意观察keys为"Mounts"的"Source"属性对应的值

```
[root@node101.yinzhengjie.org.cn ~]# docker inspect -f {{.Mounts}} c1　　　　　　　　　　　　　　　　#我们不用获取c1的全部信息，比如我们只需要获取"Mounts"的概要属性。
[{bind  /data/volumes/c1 /data2   true rprivate}]
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# docker inspect -f {{.NetworkSettings.IPAddress}} c1　　　　 #同理，我们也可以按照这种方式取出其它咱们想要的信息
192.168.100.1
[root@node101.yinzhengjie.org.cn ~]# 
```

**四.共享卷(Sharing volumes)**

**1>.多个容器使用同一个存储卷案例(咱们试验只是使用两个容器，实际上只要资源充足的情况下你可以启动任意个容器共享存储卷)**

![](https://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif)

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@node101.yinzhengjie.org.cn ~]# docker run --name c1 -it -v /data/volumes/c1:/data --rm busybox　　　　　　#我们创建一个C1容器，并在/data目录中写入测试数据，该容器不要停止它，在重新开启一个终端运行另外一个容器，来和c1容器共享存储卷数据。
/ # 
/ # ls /
bin   data  dev   etc   home  proc  root  sys   tmp   usr   var
/ # 
/ # ls /data/
/ # 
/ # echo "<h1>docker container test</h1>" > /data/index.html
/ # 
/ # ls /data/
index.html
/ # 
/ # 
/ # cat /data/index.html 
<h1>docker container test</h1>
/ # 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@node101.yinzhengjie.org.cn ~]# docker run --name c2 -it -v /data/volumes/c1:/data --rm busybox　　　　　　#通过测试发现，c2和c1虽然不是同一个容器且各自由独立的用户空间，但它们却可以通过存储卷来实现数据共享。
/ # 
/ # ls /
bin   data  dev   etc   home  proc  root  sys   tmp   usr   var
/ # 
/ # ls /data/
index.html
/ # 
/ # cat /data/index.html 
<h1>docker container test</h1>
/ # 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

**2>.复制使用其它容器的卷**

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node101.yinzhengjie.org.cn ~]# docker run --name InfrastructureImage -it -v /data/InfrastructureImage/volumes:/data/web/html busybox　　　　#此处启动一个基础镜像
/ # 
/ # ls /
bin   data  dev   etc   home  proc  root  sys   tmp   usr   var
/ # / # ls /data/ -R
/data/:
web

/data/web:
html

/data/web/html:
/ # 
/ # 
/ # ifconfig 
eth0 Link encap:Ethernet HWaddr 02:42:C0:A8:64:01 
inet addr:192.168.100.1 Bcast:192.168.100.255 Mask:255.255.255.0
UP BROADCAST RUNNING MULTICAST MTU:1500 Metric:1
RX packets:0 errors:0 dropped:0 overruns:0 frame:0
TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
collisions:0 txqueuelen:0 
RX bytes:0 (0.0 B) TX bytes:0 (0.0 B)

lo Link encap:Local Loopback 
inet addr:127.0.0.1 Mask:255.0.0.0
UP LOOPBACK RUNNING MTU:65536 Metric:1
RX packets:0 errors:0 dropped:0 overruns:0 frame:0
TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
collisions:0 txqueuelen:1000 
RX bytes:0 (0.0 B) TX bytes:0 (0.0 B)

/ #
```

\[root@node101.yinzhengjie.org.cn ~\]# docker run --name InfrastructureImage -it -v /data/InfrastructureImage/volumes:/data/web/html busybox　　　　#此处启动一个基础镜像

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@node101.yinzhengjie.org.cn ~]# docker run --name nginx --network container:InfrastructureImage --volumes-from InfrastructureImage -it busybox　　#此处使用"InfrastructureImage"的容器存储卷配置信息。/ # 
/ # ifconfig 
eth0      Link encap:Ethernet  HWaddr 02:42:C0:A8:64:01  
          inet addr:192.168.100.1  Bcast:192.168.100.255  Mask:255.255.255.0
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0 
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)

lo        Link encap:Local Loopback  
          inet addr:127.0.0.1  Mask:255.0.0.0
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)

/ # 
/ # ls /data/ -R
/data/:
web

/data/web:
html

/data/web/html:
/ # 
/ # 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

当你的才华还撑不起你的野心的时候，你就应该静下心来学习。当你的能力还驾驭不了你的目标的时候，你就应该沉下心来历练。问问自己，想要怎样的人生。 欢迎加入基础架构自动化运维：598432640，大数据SRE进阶之路：959042252，DevOps进阶之路：526991186