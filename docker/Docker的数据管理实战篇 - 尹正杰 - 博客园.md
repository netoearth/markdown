　　　　　　　　　　　　　**Docker的数据管理实战篇**

　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　**作者：尹正杰**

**版权声明：原创作品，谢绝转载！否则将追究法律责任。**

**一.Docker数据管理概述**

**1>.写时复制机制(如下图所示)**

```
　　Docker镜像由多个只读层叠加而成，启动容器时，Docker会加载只读镜像层并在镜像栈顶部添加一个读写层。
　　如果运行中的容器修改了现有的一个已经存在的文件，那该文件将会从读写层下面的只读层复制到读写层，该文件的只读版本依然存在，只是已经被读写层中该文件的副本所隐藏，此即"读写复制(COW)"机制。
```

![](https://img2018.cnblogs.com/i-beta/795254/202001/795254-20200129114837653-1556729639.png)

**2>.**什么是存储卷(volume)****

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
　　关闭并重启容器，其数据不受影响；但删除Docker容器，则其更改将会全部丢失。原因就是我们上面提到的写时复制机制，这个写时复制机制存在于容器，数据默认都保存在容器内。

　　Docker设计存在的问题：
　　　　1>.存储于联合文件系统中，不易于宿主机访问;
　　　　2>.容器间数据共享不便;
　　　　3>.删除容器其数据会丢失;

　　解决方案:"存储卷(volume)"
　　　　如下图所示:"卷"是容器上的一个或多个"目录",此类目录可绕过联合文件系统，于宿主机上的某个目录"绑定(关联)"
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

![](https://img2018.cnblogs.com/i-beta/795254/202001/795254-20200129155332718-373227870.png)

**3>.**存储卷(volume)的作用****

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
　　(1)数据卷为持久或共享数据提供了几个有用的特性：
　　　　1>.Volume于容器(container)初始化之时即会创建，由base image提供的卷中的数据于此期间完成复制；
　　　　2>.数据卷可以在容器之间共享和重用;
　　　　3>.更新镜像(image)时不会包括对数据卷的更改;
　　　　4>.即使容器(container)自身被删除，数据卷仍保持不变。

　　(2)Volume的初衷是独立于容器的生命周期实现数据持久化：
　　　　1>.默认情况下删除容器之时既不会删除卷，也不会对哪怕未被引用的卷做垃圾回收操作;
　　　　2>.如果我们删除容器时使用了特殊的选项(类似于linux的"userdel -r"选项)也可以一并删除容器的存储卷，但是生成环境我们不会这样做。

　　(3)卷为docker提供了独立于容器的数据管理机制(如下图所示):
　　　　1>.可以把"镜像"想像成静态文件，例如"程序",把卷类比为动态内容，例如"数据"；于是，镜像可以重用，而卷可以共享;
　　　　2>.卷实现了"程序(镜像)"和"数据(卷)"分离，以及"程序(镜像)"和"制作镜像的主机"分离，用户制作镜像时无须再考虑镜像运行的容器所在的主机的环境;
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

![](https://img2018.cnblogs.com/i-beta/795254/202001/795254-20200129155521650-455151482.png)

**4>.**docker存储卷类型(Volume types)****

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
　　如下图所示，Docker有两种类型的卷，每种类型都在容器中存在一个挂载点，但其在宿主机上的位置有所不同：
　　　　(1)绑定挂载卷(Bind mount volume)
　　　　　　也称为数据卷(类似于挂载的一块磁盘),指向主机文件系统上用户指定位置的卷。换句话说，在宿主机上的路径你需要人工指定一个特定路径，再容器上的路径也需要指定一个特定路径，让这两个已知路径建立关联关系。随着容器删除，存储卷数据并不会丢失。

　　　　(2)docker管理的卷(Docker managed volume)
　　　　　　也称为数据容器(将数据保存在一个容器上),Docker守护进程在Docker拥有的主机文件系统的一部分中创建管理卷。换句话说，我们只需要在容器内指定挂载点是什么，而绑定的是宿主机哪个路径下的目录我们不需要管，由容器引擎的docker daemon自行创建一个空目录或者使用一个已存在目录与你的存储卷路径建立关联关系。随着容器删除，存储卷数据也随着丢失了。
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

![](https://img2018.cnblogs.com/i-beta/795254/202001/795254-20200129155613138-98139159.png)

**二.验证容器默认存储数据路径**

**1>.创建容器(根据你自己已有的镜像随机启动一个容器,目的是为了验证默认存储数据路径)**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@docker101.yinzhengjie.org.cn ~]# docker image ls
REPOSITORY               TAG                 IMAGE ID            CREATED             SIZE
centos-haproxy           v1.8.20             1858fe05d96f        5 days ago          606MB
registry                 latest              708bc6af7e5e        5 days ago          25.8MB
tomcat-app01             v0.1                bf45c22f2d5b        6 days ago          983MB
tomcat-base              8.5.50              9ff79f369094        7 days ago          968MB
jdk-base                 1.8.0_231           0f63a97ddc85        7 days ago          953MB
centos-base              7.6.1810            b4931fd9ace2        7 days ago          551MB
centos                   centos7.6.1810      f1cb7c7d58b7        10 months ago       202MB
lorel/docker-stress-ng   latest              1ae56ccafe55        3 years ago         8.1MB
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# ss -ntl
State      Recv-Q Send-Q                                                                                          Local Address:Port                                                                                                         Peer Address:Port              
LISTEN     0      128                                                                                                         *:22                                                                                                                      *:*                  
LISTEN     0      128                                                                                                        :::22                                                                                                                     :::*                  
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# docker container run -it --rm --name myTomcat -d -p 80:8080 tomcat-app01:v0.1 
7990b03a5b1bc860d7618a1329c562778136fb598f377f995ae7e686d93c225b
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# docker container ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                            NAMES
7990b03a5b1b        tomcat-app01:v0.1   "/yinzhengjie/softwa…"   5 seconds ago       Up 4 seconds        8443/tcp, 0.0.0.0:80->8080/tcp   myTomcat
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# ss -ntl
State      Recv-Q Send-Q                                                                                          Local Address:Port                                                                                                         Peer Address:Port              
LISTEN     0      128                                                                                                         *:22                                                                                                                      *:*                  
LISTEN     0      20480                                                                                                      :::80                                                                                                                     :::*                  
LISTEN     0      128                                                                                                        :::22                                                                                                                     :::*                  
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

![](https://img2018.cnblogs.com/i-beta/795254/202001/795254-20200129161614079-701641738.png)

**2>.查看指定容器PID的容器信息存储数据路径**

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@docker101.yinzhengjie.org.cn ~]# docker container ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                            NAMES
7990b03a5b1b        tomcat-app01:v0.1   "/yinzhengjie/softwa…"   6 minutes ago       Up 6 minutes        8443/tcp, 0.0.0.0:80->8080/tcp   myTomcat
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# docker container inspect 7990b03a5b1b
[
    {
        "Id": "7990b03a5b1bc860d7618a1329c562778136fb598f377f995ae7e686d93c225b",
        "Created": "2020-01-29T16:10:43.131975737Z",
        "Path": "/yinzhengjie/softwares/web/tomcat/bin/run_tomcat.sh",
        "Args": [],
        "State": {
            "Status": "running",
            "Running": true,
            "Paused": false,
            "Restarting": false,
            "OOMKilled": false,
            "Dead": false,
            "Pid": 80905,
            "ExitCode": 0,
            "Error": "",
            "StartedAt": "2020-01-29T16:10:43.725782856Z",
            "FinishedAt": "0001-01-01T00:00:00Z"
        },
        "Image": "sha256:bf45c22f2d5b24610586fb9c356ff6e22727ef20cf156e6b5601c36492c5607e",
        "ResolvConfPath": "/var/lib/docker/containers/7990b03a5b1bc860d7618a1329c562778136fb598f377f995ae7e686d93c225b/resolv.conf",
        "HostnamePath": "/var/lib/docker/containers/7990b03a5b1bc860d7618a1329c562778136fb598f377f995ae7e686d93c225b/hostname",
        "HostsPath": "/var/lib/docker/containers/7990b03a5b1bc860d7618a1329c562778136fb598f377f995ae7e686d93c225b/hosts",
        "LogPath": "/var/lib/docker/containers/7990b03a5b1bc860d7618a1329c562778136fb598f377f995ae7e686d93c225b/7990b03a5b1bc860d7618a1329c562778136fb598f377f995ae7e686d93c225b-json.log",
        "Name": "/myTomcat",
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
            "PortBindings": {
                "8080/tcp": [
                    {
                        "HostIp": "",
                        "HostPort": "80"
                    }
                ]
            },
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
                "LowerDir": "/var/lib/docker/overlay2/e3564b6f6404fb0324d2830073c5286ad599fba0e34696144dc09e3cd7a8b845-init/diff:/var/lib/docker/overlay2/a04386e5282345b00c207b10afc3829b7efc1af0b0076e9b253154a996ac7154/diff:/var/lib/docker/overlay2/2364ce9759bebece9f6fa
48d8aaba766d600872a0b9590fb1ae5be573490efc7/diff:/var/lib/docker/overlay2/857599a00102223417b4c940a91c7d4d8d35a7acf4af92a01201a13c866a2957/diff:/var/lib/docker/overlay2/6b65fa21437fd9a2fa86455fda04c1bcec3671c336cc5acb47155a11d6244f18/diff:/var/lib/docker/overlay2/a2ab3cf4279e834fd3516707ad13994f5326f69ba382279c09fd22723617aa7d/diff:/var/lib/docker/overlay2/f68bd4ba604b3ac5ff3373178c374ff26c12acb7323953abf81b084dcad8cc48/diff:/var/lib/docker/overlay2/ec7dd4d489c66c261a2425e5dcd83a159d78c26e93e30d759ceaf895b4e9ffbd/diff:/var/lib/docker/overlay2/89b8bb19fd0d117c313180d49fdad20bff2efcf0c321cff1e95b17419a48a2d8/diff:/var/lib/docker/overlay2/67ff81b026e9127f46d299ed2ceccf7c34e066a7011eb5eee9c8f3a8b3a60d8f/diff:/var/lib/docker/overlay2/d175a2b15bf3678600e131c0aa991e78b45ce0007d59b000495f21f8703958d4/diff:/var/lib/docker/overlay2/dcc00f7ec83c156eff08f4335b344a2ee90d63896ba67749d1cfe957449080b7/diff",                "MergedDir": "/var/lib/docker/overlay2/e3564b6f6404fb0324d2830073c5286ad599fba0e34696144dc09e3cd7a8b845/merged",
                "UpperDir": "/var/lib/docker/overlay2/e3564b6f6404fb0324d2830073c5286ad599fba0e34696144dc09e3cd7a8b845/diff",
                "WorkDir": "/var/lib/docker/overlay2/e3564b6f6404fb0324d2830073c5286ad599fba0e34696144dc09e3cd7a8b845/work"
            },
            "Name": "overlay2"
        },
        "Mounts": [],
        "Config": {
            "Hostname": "7990b03a5b1b",
            "Domainname": "",
            "User": "",
            "AttachStdin": false,
            "AttachStdout": false,
            "AttachStderr": false,
            "ExposedPorts": {
                "8080/tcp": {},
                "8443/tcp": {}
            },
            "Tty": true,
            "OpenStdin": true,
            "StdinOnce": false,
            "Env": [
                "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/local/jdk/bin",
                "JAVA_HOME=/usr/local/jdk",
                "JRE_HOME=/usr/local/jdk/jre",
                "CLASSPATH=/usr/local/jdk/lib/:/usr/local/jdk/jre/lib:/usr/local/jdk/lib/tools.jar"
            ],
            "Cmd": [
                "/yinzhengjie/softwares/web/tomcat/bin/run_tomcat.sh"
            ],
            "Image": "tomcat-app01:v0.1",
            "Volumes": null,
            "WorkingDir": "",
            "Entrypoint": null,
            "OnBuild": null,
            "Labels": {
                "org.label-schema.build-date": "20181204",
                "org.label-schema.license": "GPLv2",
                "org.label-schema.name": "CentOS Base Image",
                "org.label-schema.schema-version": "1.0",
                "org.label-schema.vendor": "CentOS"
            }
        },
        "NetworkSettings": {
            "Bridge": "",
            "SandboxID": "9e72b96eb266011cda49687e96c92a5f044f13baeeba4aa00f2ba8217bebb76b",
            "HairpinMode": false,
            "LinkLocalIPv6Address": "",
            "LinkLocalIPv6PrefixLen": 0,
            "Ports": {
                "8080/tcp": [
                    {
                        "HostIp": "0.0.0.0",
                        "HostPort": "80"
                    }
                ],
                "8443/tcp": null
            },
            "SandboxKey": "/var/run/docker/netns/9e72b96eb266",
            "SecondaryIPAddresses": null,
            "SecondaryIPv6Addresses": null,
            "EndpointID": "399da9f4b4c41721a62d1931fbe670de62bfa0f054a594b67a98b1059b78334b",
            "Gateway": "172.17.0.1",
            "GlobalIPv6Address": "",
            "GlobalIPv6PrefixLen": 0,
            "IPAddress": "172.17.0.2",
            "IPPrefixLen": 16,
            "IPv6Gateway": "",
            "MacAddress": "02:42:ac:11:00:02",
            "Networks": {
                "bridge": {
                    "IPAMConfig": null,
                    "Links": null,
                    "Aliases": null,
                    "NetworkID": "6bfa389a9d1eb61996f432111d1b5d3fcdad140f2453d1dc70b705551236e80f",
                    "EndpointID": "399da9f4b4c41721a62d1931fbe670de62bfa0f054a594b67a98b1059b78334b",
                    "Gateway": "172.17.0.1",
                    "IPAddress": "172.17.0.2",
                    "IPPrefixLen": 16,
                    "IPv6Gateway": "",
                    "GlobalIPv6Address": "",
                    "GlobalIPv6PrefixLen": 0,
                    "MacAddress": "02:42:ac:11:00:02",
                    "DriverOpts": null
                }
            }
        }
    }
]
[root@docker101.yinzhengjie.org.cn ~]# 
```

\[root@docker101.yinzhengjie.org.cn ~\]# docker container inspect 7990b03a5b1b

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@docker101.yinzhengjie.org.cn ~]# docker container ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                            NAMES
7990b03a5b1b        tomcat-app01:v0.1   "/yinzhengjie/softwa…"   11 minutes ago      Up 11 minutes       8443/tcp, 0.0.0.0:80->8080/tcp   myTomcat
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# docker container inspect  -f "{{.GraphDriver.Data.UpperDir}}"  7990b03a5b1b 
/var/lib/docker/overlay2/e3564b6f6404fb0324d2830073c5286ad599fba0e34696144dc09e3cd7a8b845/diff
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# ll /var/lib/docker/overlay2/e3564b6f6404fb0324d2830073c5286ad599fba0e34696144dc09e3cd7a8b845/diff
total 0
drwxrwxrwt 3 root root 31 Jan 30 00:10 tmp
drwxr-xr-x 3 root root 17 Dec  4  2018 var
drwxr-xr-x 3 root root 23 Jan 22 12:49 yinzhengjie
[root@docker101.yinzhengjie.org.cn ~]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

**3>.在容器内部写入测试数据，观察宿主机目录是否有新文件生成(你也可以使用md5sum命令去检测容器内部文件和宿主机存储的同名文件的校验值是否一致)**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@docker101.yinzhengjie.org.cn ~]# docker container ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                            NAMES
7990b03a5b1b        tomcat-app01:v0.1   "/yinzhengjie/softwa…"   11 minutes ago      Up 11 minutes       8443/tcp, 0.0.0.0:80->8080/tcp   myTomcat
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# docker container inspect  -f "{{.GraphDriver.Data.UpperDir}}"  7990b03a5b1b 
/var/lib/docker/overlay2/e3564b6f6404fb0324d2830073c5286ad599fba0e34696144dc09e3cd7a8b845/diff
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# ll /var/lib/docker/overlay2/e3564b6f6404fb0324d2830073c5286ad599fba0e34696144dc09e3cd7a8b845/diff
total 0
drwxrwxrwt 3 root root 31 Jan 30 00:10 tmp
drwxr-xr-x 3 root root 17 Dec  4  2018 var
drwxr-xr-x 3 root root 23 Jan 22 12:49 yinzhengjie
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# docker container exec -it 7990b03a5b1b bash
[root@7990b03a5b1b /]# 
[root@7990b03a5b1b /]# dd if=/dev/zero of=test.txt bs=1M count=1024
1024+0 records in
1024+0 records out
1073741824 bytes (1.1 GB) copied, 2.21405 s, 485 MB/s
[root@7990b03a5b1b /]# 
[root@7990b03a5b1b /]# ll
total 1048588
-rw-r--r--   1 root root      12053 Dec  4  2018 anaconda-post.log
lrwxrwxrwx   1 root root          7 Dec  4  2018 bin -> usr/bin
drwxr-xr-x   5 root root        360 Jan 30 00:10 dev
drwxr-xr-x   1 root root         66 Jan 30 00:10 etc
drwxr-xr-x   1 root root         33 Jan 22 08:29 home
lrwxrwxrwx   1 root root          7 Dec  4  2018 lib -> usr/lib
lrwxrwxrwx   1 root root          9 Dec  4  2018 lib64 -> usr/lib64
drwxr-xr-x   2 root root          6 Apr 11  2018 media
drwxr-xr-x   2 root root          6 Apr 11  2018 mnt
drwxr-xr-x   2 root root          6 Apr 11  2018 opt
dr-xr-xr-x 177 root root          0 Jan 30 00:10 proc
dr-xr-x---   1 root root         18 Jan 22 08:27 root
drwxr-xr-x   1 root root          6 Jan 22 08:29 run
lrwxrwxrwx   1 root root          8 Dec  4  2018 sbin -> usr/sbin
drwxr-xr-x   2 root root          6 Apr 11  2018 srv
dr-xr-xr-x  13 root root          0 Jan 30 00:10 sys
-rw-r--r--   1 root root 1073741824 Jan 30 00:26 test.txt
drwxrwxrwt   1 root root         31 Jan 30 00:10 tmp
drwxr-xr-x   1 root root         19 Dec  4  2018 usr
drwxr-xr-x   1 root root         17 Dec  4  2018 var
drwxr-xr-x   1 root root         23 Jan 22 12:49 yinzhengjie
[root@7990b03a5b1b /]#  
[root@7990b03a5b1b /]# exit 
exit
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# ll -h /var/lib/docker/overlay2/e3564b6f6404fb0324d2830073c5286ad599fba0e34696144dc09e3cd7a8b845/diff
total 1.0G
dr-xr-x--- 2 root root   27 Jan 30 00:27 root
-rw-r--r-- 1 root root 1.0G Jan 30 00:26 test.txt
drwxrwxrwt 3 root root   31 Jan 30 00:10 tmp
drwxr-xr-x 3 root root   17 Dec  4  2018 var
drwxr-xr-x 3 root root   23 Jan 22 12:49 yinzhengjie
[root@docker101.yinzhengjie.org.cn ~]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

![](https://img2018.cnblogs.com/i-beta/795254/202001/795254-20200129163121196-410848052.png)

**4>.删除容器后，默认情况下，该容器生成的数据也会随之删除**

![](https://img2018.cnblogs.com/i-beta/795254/202001/795254-20200129163403582-1800554435.png)

**5>.如何在删除容器的情况下不删除数据呢？**

```
　　目前有两种解决方案，就是我们上面所提到的数据卷(Bind mount volume)和数据卷容器(Docker managed volume)技术解决。
```

**三.容器使用数据卷案例**

**1>.自定义tomcat镜像**

```
　　博主推荐阅读:
　　　　https://www.cnblogs.com/yinzhengjie/p/12230043.html
```

**2>.**创建测试网页代码数据****

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@docker101.yinzhengjie.org.cn ~]# mkdir -pv /yinzhengjie/data/tomcat/app01
mkdir: created directory ‘/yinzhengjie/data/tomcat’
mkdir: created directory ‘/yinzhengjie/data/tomcat/app01’
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# echo "<h1>YinZhengjie's Tomcat Server 2020,Come to China, come to Wuhan. </h1>" > /yinzhengjie/data/tomcat/app01/index.html
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# cat /yinzhengjie/data/tomcat/app01/index.html
<h1>YinZhengjie's Tomcat Server 2020,Come to China, come to Wuhan. </h1>
[root@docker101.yinzhengjie.org.cn ~]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

**3>.查看镜像默认的数据**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@docker101.yinzhengjie.org.cn ~]# docker image ls 
REPOSITORY               TAG                 IMAGE ID            CREATED             SIZE
centos-haproxy           v1.8.20             1858fe05d96f        5 days ago          606MB
registry                 latest              708bc6af7e5e        5 days ago          25.8MB
tomcat-app01             v0.1                bf45c22f2d5b        6 days ago          983MB
tomcat-base              8.5.50              9ff79f369094        7 days ago          968MB
jdk-base                 1.8.0_231           0f63a97ddc85        7 days ago          953MB
centos-base              7.6.1810            b4931fd9ace2        7 days ago          551MB
centos                   centos7.6.1810      f1cb7c7d58b7        10 months ago       202MB
lorel/docker-stress-ng   latest              1ae56ccafe55        3 years ago         8.1MB
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]#  docker container run -it --rm --name myTomcat  -p 80:8080 tomcat-app01:v0.1 bash
[root@e98341416050 /]# 
[root@e98341416050 /]# ll /yinzhengjie/data/tomcat/webapps/app01/
total 8
-rw-r--r-- 1 tomcat tomcat 32 Jan 23 16:59 index.html
-rw-r--r-- 1 tomcat tomcat 37 Jan 23 16:59 index2020.html
[root@e98341416050 /]# 
[root@e98341416050 /]# cat /yinzhengjie/data/tomcat/webapps/app01/index.html 
YinZhengjie's Tomcat Web Server
[root@e98341416050 /]# 
[root@e98341416050 /]# cat /yinzhengjie/data/tomcat/webapps/app01/index2020.html 
YinZhengjie's Tomcat Web Server 2020
[root@e98341416050 /]# 
[root@e98341416050 /]# exit 
exit
[root@docker101.yinzhengjie.org.cn ~]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

**4>.使用宿主机的路径替换镜像文件的路径**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@docker101.yinzhengjie.org.cn ~]#  docker container run -it --rm --name myTomcat  -p 80:8080 tomcat-app01:v0.1 bash
[root@e98341416050 /]# 
[root@e98341416050 /]# ll /yinzhengjie/data/tomcat/webapps/app01/
total 8
-rw-r--r-- 1 tomcat tomcat 32 Jan 23 16:59 index.html
-rw-r--r-- 1 tomcat tomcat 37 Jan 23 16:59 index2020.html
[root@e98341416050 /]# 
[root@e98341416050 /]# cat /yinzhengjie/data/tomcat/webapps/app01/index.html 
YinZhengjie's Tomcat Web Server
[root@e98341416050 /]# 
[root@e98341416050 /]# cat /yinzhengjie/data/tomcat/webapps/app01/index2020.html 
YinZhengjie's Tomcat Web Server 2020
[root@e98341416050 /]# 
[root@e98341416050 /]# exit 
exit
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# docker container run -it --rm --name myTomcat -v /yinzhengjie/data/tomcat/app01:/yinzhengjie/data/tomcat/webapps/app01 -p 80:8080 -d tomcat-app01:v0.1 
632e9bdd3d133b0f162a45b1c7c74e52e2e5d8ce2584621c013d6f59a9be8690
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# docker container ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                            NAMES
632e9bdd3d13        tomcat-app01:v0.1   "/yinzhengjie/softwa…"   8 seconds ago       Up 7 seconds        8443/tcp, 0.0.0.0:80->8080/tcp   myTomcat
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# docker container exec -it 632e9bdd3d13 bash
[root@632e9bdd3d13 /]# 
[root@632e9bdd3d13 /]# netstat -untalp
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 0.0.0.0:8080            0.0.0.0:*               LISTEN      -                   
tcp        0      0 127.0.0.1:8005          0.0.0.0:*               LISTEN      -                   
tcp        0      0 0.0.0.0:8009            0.0.0.0:*               LISTEN      -                   
[root@632e9bdd3d13 /]# 
[root@632e9bdd3d13 /]# ll /yinzhengjie/data/tomcat/webapps/app01/
total 4
-rw-r--r-- 1 root root 73 Jan 30 00:54 index.html
[root@632e9bdd3d13 /]# 
[root@632e9bdd3d13 /]# cat /yinzhengjie/data/tomcat/webapps/app01/index.html 
<h1>YinZhengjie's Tomcat Server 2020,Come to China, come to Wuhan. </h1>
[root@632e9bdd3d13 /]# 
[root@632e9bdd3d13 /]# exit 
exit
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# ll /yinzhengjie/data/tomcat/app01/
total 4
-rw-r--r-- 1 root root 73 Jan 30 00:54 index.html
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# cat /yinzhengjie/data/tomcat/app01/index.html 
<h1>YinZhengjie's Tomcat Server 2020,Come to China, come to Wuhan. </h1>
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

![](https://img2018.cnblogs.com/i-beta/795254/202001/795254-20200129171349002-1601596543.png)

**5>.浏览器访问宿主机的80端口**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@docker101.yinzhengjie.org.cn ~]# docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                            NAMES
632e9bdd3d13        tomcat-app01:v0.1   "/yinzhengjie/softwa…"   7 minutes ago       Up 7 minutes        8443/tcp, 0.0.0.0:80->8080/tcp   myTomcat
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# ss -ntl
State      Recv-Q Send-Q                                                                                          Local Address:Port                                                                                                         Peer Address:Port              
LISTEN     0      128                                                                                                         *:22                                                                                                                      *:*                  
LISTEN     0      20480                                                                                                      :::80                                                                                                                     :::*                  
LISTEN     0      128                                                                                                        :::22                                                                                                                     :::*                  
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# ll /yinzhengjie/data/tomcat/app01/
total 4
-rw-r--r-- 1 root root 73 Jan 30 00:54 index.html
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# cat /yinzhengjie/data/tomcat/app01/index.html 
<h1>YinZhengjie's Tomcat Server 2020,Come to China, come to Wuhan. </h1>
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# curl http://docker101.yinzhengjie.org.cn/app01/index.html
<h1>YinZhengjie's Tomcat Server 2020,Come to China, come to Wuhan. </h1>
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# curl -I  http://docker101.yinzhengjie.org.cn/app01/index.html
HTTP/1.1 200 
Accept-Ranges: bytes
ETag: W/"73-1580316881000"
Last-Modified: Wed, 29 Jan 2020 16:54:41 GMT
Content-Type: text/html
Content-Length: 73
Date: Wed, 29 Jan 2020 17:15:31 GMT

[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

![](https://img2018.cnblogs.com/i-beta/795254/202001/795254-20200129171744836-95342828.png)

**6>.验证容器对挂载宿主机目录的权限(默认是读写权限)**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@docker101.yinzhengjie.org.cn ~]# ll /yinzhengjie/data/tomcat/app01/
total 4
-rw-r--r-- 1 root root 73 Jan 30 00:54 index.html
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                            NAMES
632e9bdd3d13        tomcat-app01:v0.1   "/yinzhengjie/softwa…"   22 minutes ago      Up 22 minutes       8443/tcp, 0.0.0.0:80->8080/tcp   myTomcat
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# docker container exec -it 632e9bdd3d13 bash
[root@632e9bdd3d13 /]# 
[root@632e9bdd3d13 /]# ll /yinzhengjie/data/tomcat/webapps/app01/
total 4
-rw-r--r-- 1 root root 73 Jan 30 00:54 index.html
[root@632e9bdd3d13 /]# 
[root@632e9bdd3d13 /]# echo `date +%F` > /yinzhengjie/data/tomcat/webapps/app01/data.html
[root@632e9bdd3d13 /]# 
[root@632e9bdd3d13 /]# mkdir /yinzhengjie/data/tomcat/webapps/app01/testDir
[root@632e9bdd3d13 /]# 
[root@632e9bdd3d13 /]# 
[root@632e9bdd3d13 /]# ll /yinzhengjie/data/tomcat/webapps/app01/
total 8
-rw-r--r-- 1 root root 11 Jan 30 01:29 data.html
-rw-r--r-- 1 root root 73 Jan 30 00:54 index.html
drwxr-xr-x 2 root root  6 Jan 30 01:29 testDir
[root@632e9bdd3d13 /]# 
[root@632e9bdd3d13 /]# cat /yinzhengjie/data/tomcat/webapps/app01/data.html 
2020-01-30
[root@632e9bdd3d13 /]# 
[root@632e9bdd3d13 /]# exit 
exit
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# ll /yinzhengjie/data/tomcat/app01/
total 8
-rw-r--r-- 1 root root 11 Jan 30 01:29 data.html
-rw-r--r-- 1 root root 73 Jan 30 00:54 index.html
drwxr-xr-x 2 root root  6 Jan 30 01:29 testDir
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# cat /yinzhengjie/data/tomcat/app01/data.html 
2020-01-30
[root@docker101.yinzhengjie.org.cn ~]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

![](https://img2018.cnblogs.com/i-beta/795254/202001/795254-20200129173317149-1220823643.png)

**7>.删除容器后，挂载到容器的数据卷并不会随之删除哟~**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@docker101.yinzhengjie.org.cn ~]# ll /yinzhengjie/data/tomcat/app01/
total 8
-rw-r--r-- 1 root root 11 Jan 30 01:29 data.html
-rw-r--r-- 1 root root 73 Jan 30 00:54 index.html
drwxr-xr-x 2 root root  6 Jan 30 01:29 testDir
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# docker container ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                            NAMES
632e9bdd3d13        tomcat-app01:v0.1   "/yinzhengjie/softwa…"   27 minutes ago      Up 27 minutes       8443/tcp, 0.0.0.0:80->8080/tcp   myTomcat
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# docker container rm -f 632e9bdd3d13
632e9bdd3d13
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# docker container ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# ll /yinzhengjie/data/tomcat/app01/
total 8
-rw-r--r-- 1 root root 11 Jan 30 01:29 data.html
-rw-r--r-- 1 root root 73 Jan 30 00:54 index.html
drwxr-xr-x 2 root root  6 Jan 30 01:29 testDir
[root@docker101.yinzhengjie.org.cn ~]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

![](https://img2018.cnblogs.com/i-beta/795254/202001/795254-20200129173641833-2009754576.png)

**8>.挂载数据卷时指定权限为只读**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@docker101.yinzhengjie.org.cn ~]# ll /yinzhengjie/data/tomcat/app01/index.html 
-rw-r--r-- 1 root root 73 Jan 30 00:54 /yinzhengjie/data/tomcat/app01/index.html
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# cat /yinzhengjie/data/tomcat/app01/index.html 
<h1>YinZhengjie's Tomcat Server 2020,Come to China, come to Wuhan. </h1>
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# docker container run -it --rm --name myTomcat -v /yinzhengjie/data/tomcat/app01:/yinzhengjie/data/tomcat/webapps/app01:ro -p 80:8080 -d tomcat-app01:v0.1 
79ae89f051b298903197dcdf8913c9e02845cc6633f6ed0bda024cc2d51980e4
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# docker container ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                            NAMES
79ae89f051b2        tomcat-app01:v0.1   "/yinzhengjie/softwa…"   6 seconds ago       Up 5 seconds        8443/tcp, 0.0.0.0:80->8080/tcp   myTomcat
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# docker container exec -it 79ae89f051b2 bash
[root@79ae89f051b2 /]# 
[root@79ae89f051b2 /]# cd /yinzhengjie/data/tomcat/webapps/app01/
[root@79ae89f051b2 app01]# 
[root@79ae89f051b2 app01]# ll
total 4
-rw-r--r-- 1 root root 73 Jan 30 00:54 index.html
[root@79ae89f051b2 app01]# 
[root@79ae89f051b2 app01]# echo `date +%F` > test.html
bash: test.html: Read-only file system
[root@79ae89f051b2 app01]# 
[root@79ae89f051b2 app01]# netstat -untalp
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 127.0.0.1:8005          0.0.0.0:*               LISTEN      -                   
tcp        0      0 0.0.0.0:8009            0.0.0.0:*               LISTEN      -                   
tcp        0      0 0.0.0.0:8080            0.0.0.0:*               LISTEN      -                   
[root@79ae89f051b2 app01]# 
[root@79ae89f051b2 app01]# exit 
exit
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# ss -ntl
State      Recv-Q Send-Q                                                                                          Local Address:Port                                                                                                         Peer Address:Port              
LISTEN     0      128                                                                                                         *:22                                                                                                                      *:*                  
LISTEN     0      20480                                                                                                      :::80                                                                                                                     :::*                  
LISTEN     0      128                                                                                                        :::22                                                                                                                     :::*                  
[root@docker101.yinzhengjie.org.cn ~]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

![](https://img2018.cnblogs.com/i-beta/795254/202001/795254-20200129175325064-1413498625.png)

**9>.一个容器挂载多个数据卷案例**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@docker101.yinzhengjie.org.cn ~]# ll /yinzhengjie/data/tomcat/app01/
total 4
-rw-r--r-- 1 root root 73 Jan 30 00:54 index.html
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# md5sum /etc/hosts
9ee1454b4aa700591887afea9f3ddd13  /etc/hosts
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# docker container run -it --rm --name myTomcat -v /yinzhengjie/data/tomcat/app01:/yinzhengjie/data/tomcat/webapps/app01:rw -v /etc/hosts:/etc/hosts:ro -p 80:8080 -d tomcat-app01:v0.1 
39020b57791e2996bd35df1f4827256899d52fafd1879f06a8bc80f0fdb6eac5
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# docker container ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                            NAMES
39020b57791e        tomcat-app01:v0.1   "/yinzhengjie/softwa…"   9 seconds ago       Up 8 seconds        8443/tcp, 0.0.0.0:80->8080/tcp   myTomcat
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# docker container exec -it 39020b57791e bash
[root@39020b57791e /]# 
[root@39020b57791e /]# md5sum /etc/hosts
9ee1454b4aa700591887afea9f3ddd13  /etc/hosts
[root@39020b57791e /]# 
[root@39020b57791e /]# echo "172.200.6.105 kafka105.yinzhengjie.org.cn" > /etc/hosts
bash: /etc/hosts: Read-only file system
[root@39020b57791e /]# 
[root@39020b57791e /]# echo `date +%F` > /yinzhengjie/data/tomcat/webapps/app01/date.html
[root@39020b57791e /]# 
[root@39020b57791e /]# ll /yinzhengjie/data/tomcat/webapps/app01/
total 8
-rw-r--r-- 1 root root 11 Jan 30 02:16 date.html
-rw-r--r-- 1 root root 73 Jan 30 00:54 index.html
[root@39020b57791e /]# 
[root@39020b57791e /]# exit 
exit
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# ll /yinzhengjie/data/tomcat/app01/
total 8
-rw-r--r-- 1 root root 11 Jan 30 02:16 date.html
-rw-r--r-- 1 root root 73 Jan 30 00:54 index.html
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

![](https://img2018.cnblogs.com/i-beta/795254/202001/795254-20200129182149858-1965941701.png)

****10>.多个容器使用同一个宿主机数据卷案例****

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@docker101.yinzhengjie.org.cn ~]# ll /yinzhengjie/data/tomcat/app01/
total 4
-rw-r--r-- 1 root root 73 Jan 30 00:54 index.html
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# docker container run -it --rm --name myTomcat01 -v /yinzhengjie/data/tomcat/app01:/yinzhengjie/data/tomcat/webapps/app01:rw -v /etc/hosts:/etc/hosts:ro -p 80:8080 -d tomcat-app01:v0.1 
80d4a2f2874ce807848aebf41f2e0d1d1f6affe1b796946b5e07e8076fef14df
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# docker container run -it --rm --name myTomcat02 -v /yinzhengjie/data/tomcat/app01:/yinzhengjie/data/tomcat/webapps/app01:rw -v /etc/hosts:/etc/hosts:ro -p 81:8080 -d tomcat-app01:v0.1 
2eb6690aa422f7072a2655762fed657432307887d74fa54769ce689ff53d75b9
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# docker container ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                            NAMES
2eb6690aa422        tomcat-app01:v0.1   "/yinzhengjie/softwa…"   8 seconds ago       Up 7 seconds        8443/tcp, 0.0.0.0:81->8080/tcp   myTomcat02
80d4a2f2874c        tomcat-app01:v0.1   "/yinzhengjie/softwa…"   19 seconds ago      Up 18 seconds       8443/tcp, 0.0.0.0:80->8080/tcp   myTomcat01
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# docker container exec -it myTomcat01 bash
[root@80d4a2f2874c /]# 
[root@80d4a2f2874c /]# ll /yinzhengjie/data/tomcat/webapps/app01/
total 4
-rw-r--r-- 1 root root 73 Jan 30 00:54 index.html
[root@80d4a2f2874c /]# 
[root@80d4a2f2874c /]# echo `date +%F` > /yinzhengjie/data/tomcat/webapps/app01/date.html
[root@80d4a2f2874c /]# 
[root@80d4a2f2874c /]# ll /yinzhengjie/data/tomcat/webapps/app01/
total 8
-rw-r--r-- 1 root root 11 Jan 30 02:24 date.html
-rw-r--r-- 1 root root 73 Jan 30 00:54 index.html
[root@80d4a2f2874c /]# 
[root@80d4a2f2874c /]# exit 
exit
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# docker container exec -it myTomcat02 bash
[root@2eb6690aa422 /]# 
[root@2eb6690aa422 /]# ll /yinzhengjie/
data/      softwares/ 
[root@2eb6690aa422 /]# ll /yinzhengjie/data/tomcat/webapps/app01/
total 8
-rw-r--r-- 1 root root 11 Jan 30 02:24 date.html
-rw-r--r-- 1 root root 73 Jan 30 00:54 index.html
[root@2eb6690aa422 /]# 
[root@2eb6690aa422 /]# cat /yinzhengjie/data/tomcat/webapps/app01/date.html 
2020-01-30
[root@2eb6690aa422 /]# 
[root@2eb6690aa422 /]# exit 
exit
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# ll /yinzhengjie/data/tomcat/app01/
total 8
-rw-r--r-- 1 root root 11 Jan 30 02:24 date.html
-rw-r--r-- 1 root root 73 Jan 30 00:54 index.html
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# cat /yinzhengjie/data/tomcat/app01/date.html 
2020-01-30
[root@docker101.yinzhengjie.org.cn ~]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

![](https://img2018.cnblogs.com/i-beta/795254/202001/795254-20200129182936427-1990424889.png)

**四.容器使用数据卷容器案例**

**1>.启动一个容器，并挂载到主机的宿主机目录**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@docker101.yinzhengjie.org.cn ~]# ll /yinzhengjie/data/tomcat/app01/
total 4
-rw-r--r-- 1 root root 73 Jan 30 00:54 index.html
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# docker container run -it --rm --name volume-server -v /yinzhengjie/data/tomcat/app01:/yinzhengjie/data/tomcat/webapps/app01:rw  -d tomcat-app01:v0.1 
bcb27f67b70683140351924de515ecab2c4b0e10de4c945a32f81f441ddfd88a
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# docker container ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                NAMES
bcb27f67b706        tomcat-app01:v0.1   "/yinzhengjie/softwa…"   12 seconds ago      Up 12 seconds       8080/tcp, 8443/tcp   volume-server
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

**2>.基于上一步的数据卷容器来创建新的客户端容器**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@docker101.yinzhengjie.org.cn ~]# docker container ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                NAMES
bcb27f67b706        tomcat-app01:v0.1   "/yinzhengjie/softwa…"   12 minutes ago      Up 12 minutes       8080/tcp, 8443/tcp   volume-server
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# docker container run -it --rm --name volume-client01 -p 80:8080 --volumes-from volume-server -d tomcat-app01:v0.1 
3c34e7e977ac92c1849b032c81992ed9567b6b85b23aae8ae480f03bd03b4222
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# docker container run -it --rm --name volume-client02 -p 81:8080 --volumes-from volume-server -d tomcat-app01:v0.1 
2aee6944d5a17abc73a3dd4bc8eb8a8690fe9dbee999c6e8be1543e13e58ac57
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# docker container ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                            NAMES
2aee6944d5a1        tomcat-app01:v0.1   "/yinzhengjie/softwa…"   3 seconds ago       Up 2 seconds        8443/tcp, 0.0.0.0:81->8080/tcp   volume-client02
3c34e7e977ac        tomcat-app01:v0.1   "/yinzhengjie/softwa…"   20 seconds ago      Up 19 seconds       8443/tcp, 0.0.0.0:80->8080/tcp   volume-client01
bcb27f67b706        tomcat-app01:v0.1   "/yinzhengjie/softwa…"   12 minutes ago      Up 12 minutes       8080/tcp, 8443/tcp               volume-server
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# ss -ntl
State      Recv-Q Send-Q                                                                                          Local Address:Port                                                                                                         Peer Address:Port              
LISTEN     0      128                                                                                                         *:22                                                                                                                      *:*                  
LISTEN     0      20480                                                                                                      :::80                                                                                                                     :::*                  
LISTEN     0      20480                                                                                                      :::81                                                                                                                     :::*                  
LISTEN     0      128                                                                                                        :::22                                                                                                                     :::*                  
[root@docker101.yinzhengjie.org.cn ~]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

**3>.浏览器访问新创建的客户端容器**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@docker101.yinzhengjie.org.cn ~]# docker container ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                            NAMES
2aee6944d5a1        tomcat-app01:v0.1   "/yinzhengjie/softwa…"   3 minutes ago       Up 3 minutes        8443/tcp, 0.0.0.0:81->8080/tcp   volume-client02
3c34e7e977ac        tomcat-app01:v0.1   "/yinzhengjie/softwa…"   3 minutes ago       Up 3 minutes        8443/tcp, 0.0.0.0:80->8080/tcp   volume-client01
bcb27f67b706        tomcat-app01:v0.1   "/yinzhengjie/softwa…"   15 minutes ago      Up 15 minutes       8080/tcp, 8443/tcp               volume-server
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# docker container exec -it volume-client01 bash
[root@3c34e7e977ac /]# 
[root@3c34e7e977ac /]# ll /yinzhengjie/data/tomcat/webapps/app01/
total 4
-rw-r--r-- 1 root root 73 Jan 30 00:54 index.html
[root@3c34e7e977ac /]# 
[root@3c34e7e977ac /]# cat /yinzhengjie/data/tomcat/webapps/app01/index.html 
<h1>YinZhengjie's Tomcat Server 2020,Come to China, come to Wuhan. </h1>
[root@3c34e7e977ac /]# 
[root@3c34e7e977ac /]# exit 
exit
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# docker container exec -it volume-client02 bash
[root@2aee6944d5a1 /]# 
[root@2aee6944d5a1 /]# ll /yinzhengjie/data/tomcat/webapps/app01/
total 4
-rw-r--r-- 1 root root 73 Jan 30 00:54 index.html
[root@2aee6944d5a1 /]# 
[root@2aee6944d5a1 /]# cat /yinzhengjie/data/tomcat/webapps/app01/index.html 
<h1>YinZhengjie's Tomcat Server 2020,Come to China, come to Wuhan. </h1>
[root@2aee6944d5a1 /]# 
[root@2aee6944d5a1 /]# exit 
exit
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# ss -ntl
State      Recv-Q Send-Q                                                                                          Local Address:Port                                                                                                         Peer Address:Port              
LISTEN     0      128                                                                                                         *:22                                                                                                                      *:*                  
LISTEN     0      20480                                                                                                      :::80                                                                                                                     :::*                  
LISTEN     0      20480                                                                                                      :::81                                                                                                                     :::*                  
LISTEN     0      128                                                                                                        :::22                                                                                                                     :::*                  
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# curl http://docker101.yinzhengjie.org.cn/app01/
<h1>YinZhengjie's Tomcat Server 2020,Come to China, come to Wuhan. </h1>
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# curl http://docker101.yinzhengjie.org.cn:81/app01/
<h1>YinZhengjie's Tomcat Server 2020,Come to China, come to Wuhan. </h1>
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

![](https://img2018.cnblogs.com/i-beta/795254/202001/795254-20200129192300145-1669322330.png)

**4>.在任意一个客户端容器修改内容，通过浏览器验证其它容器的页面内容是否也发生改变**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@docker101.yinzhengjie.org.cn ~]# docker container ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                            NAMES
2aee6944d5a1        tomcat-app01:v0.1   "/yinzhengjie/softwa…"   7 minutes ago       Up 7 minutes        8443/tcp, 0.0.0.0:81->8080/tcp   volume-client02
3c34e7e977ac        tomcat-app01:v0.1   "/yinzhengjie/softwa…"   7 minutes ago       Up 7 minutes        8443/tcp, 0.0.0.0:80->8080/tcp   volume-client01
bcb27f67b706        tomcat-app01:v0.1   "/yinzhengjie/softwa…"   19 minutes ago      Up 19 minutes       8080/tcp, 8443/tcp               volume-server
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# docker container exec -it volume-client01 bash
[root@3c34e7e977ac /]# 
[root@3c34e7e977ac /]# ll /yinzhengjie/data/tomcat/webapps/app01/
total 4
-rw-r--r-- 1 root root 73 Jan 30 00:54 index.html
[root@3c34e7e977ac /]# 
[root@3c34e7e977ac /]# echo "<h1>`date +%F`</h1>" > /yinzhengjie/data/tomcat/webapps/app01/data.html
[root@3c34e7e977ac /]# 
[root@3c34e7e977ac /]# exit 
exit
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# docker container exec -it volume-client02 bash
[root@2aee6944d5a1 /]# 
[root@2aee6944d5a1 /]# ll /yinzhengjie/data/tomcat/webapps/app01/
total 8
-rw-r--r-- 1 root root 20 Jan 30 03:25 data.html
-rw-r--r-- 1 root root 73 Jan 30 00:54 index.html
[root@2aee6944d5a1 /]# 
[root@2aee6944d5a1 /]# cat /yinzhengjie/data/tomcat/webapps/app01/data.html  
<h1>2020-01-30</h1>
[root@2aee6944d5a1 /]# 
[root@2aee6944d5a1 /]# exit 
exit
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# curl http://docker101.yinzhengjie.org.cn/app01/data.html
<h1>2020-01-30</h1>
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# curl http://docker101.yinzhengjie.org.cn:81/app01/data.html
<h1>2020-01-30</h1>
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

![](https://img2018.cnblogs.com/i-beta/795254/202001/795254-20200129192906403-2085766688.png)

**5>.删除服务端容器(volume-server),客户端容器(volume-client01和volume-client02)并不会受到影响(因为容器时通过挂载访问数据的),但无法再基于服务端容器创建新的客户端容器**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@docker101.yinzhengjie.org.cn ~]# docker container ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                            NAMES
2aee6944d5a1        tomcat-app01:v0.1   "/yinzhengjie/softwa…"   14 minutes ago      Up 14 minutes       8443/tcp, 0.0.0.0:81->8080/tcp   volume-client02
3c34e7e977ac        tomcat-app01:v0.1   "/yinzhengjie/softwa…"   15 minutes ago      Up 14 minutes       8443/tcp, 0.0.0.0:80->8080/tcp   volume-client01
bcb27f67b706        tomcat-app01:v0.1   "/yinzhengjie/softwa…"   27 minutes ago      Up 27 minutes       8080/tcp, 8443/tcp               volume-server
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# docker container rm -f volume-server
volume-server
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# docker container ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                            NAMES
2aee6944d5a1        tomcat-app01:v0.1   "/yinzhengjie/softwa…"   14 minutes ago      Up 14 minutes       8443/tcp, 0.0.0.0:81->8080/tcp   volume-client02
3c34e7e977ac        tomcat-app01:v0.1   "/yinzhengjie/softwa…"   15 minutes ago      Up 15 minutes       8443/tcp, 0.0.0.0:80->8080/tcp   volume-client01
[root@docker101.yinzhengjie.org.cn ~]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

![](https://img2018.cnblogs.com/i-beta/795254/202001/795254-20200129193905987-1839650668.png)

**五.数据卷与数据卷容器特点及使用场景**

**1>.数据卷的特点及使用场景**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
　　数据卷的特点:
　　　　(1)数据卷是目录或文件，并且可以在多个容器之间共同使用;　　　　(2)对数据卷更改数据容器里面会立即更新;　　　　(3)数据卷的数据可以持久保存,即使删除使用该数据卷的容器也不影响使用,即并不会删除数据卷(它是宿主机的目录);　　　　(4)在容器里面写入数据不会影响到镜像本身;　　数据卷的使用场景:　　　　(1)日志输出;　　　　(2)静态web页面;　　　　(3)应用配置文件;　　　　(4)多容器间目录或文件共享;
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

**2>.数据卷容器特点及使用场景**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
　　数据卷容器的特点:
　　　　数据卷容器最大的功能是可以让数据在多个docker容器之间共享,即可以让B容器访问A容器的内容，而其C容器也可以访问A容器的内容，即先要创建一个后台运行的容器作为Server,用于卷提供,这个卷可以为其它容器提供数据存储服务,其它使用此卷的容器作为client端。　　数据卷容器的使用场景:　　　　可以用于线上数据库，共享数据目录等环境，因为即使数据卷容器被删除了，其它已经基于该数据卷容器创建的容器依然可以挂载使用。　　　　数据卷容器可以作为共享的方式为其它容器提供文件共享，类似于NFS共享，可以在生产中启用一个实例挂载本地的目录，然后其它的容器分别挂载此容器的目录，即可保证个容器之间的数据一致性。　　　　这只是docker官方提供的一种数据挂载的解决方案，生产环境很少见人使用这种方式，大多数使用数据卷挂载的方式较多，而且数据卷容器的功能咱们使用数据卷挂载也能实现相同的功能。
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

当你的才华还撑不起你的野心的时候，你就应该静下心来学习。当你的能力还驾驭不了你的目标的时候，你就应该沉下心来历练。问问自己，想要怎样的人生。 欢迎加入基础架构自动化运维：598432640，大数据SRE进阶之路：959042252，DevOps进阶之路：526991186