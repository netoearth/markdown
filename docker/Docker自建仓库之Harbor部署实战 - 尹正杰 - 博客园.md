　　　　　　　　　　　　**Docker自建仓库之Harbor部署实战**

　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　**作者：尹正杰**

**版权声明：原创作品，谢绝转载！否则将追究法律责任。**

**一.Harbor概述**

**1>.什么是Harbor**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
　　Harbor是一个用于存储和分发镜像的企业级Registry服务器,由Vmware开源,其通过添加一些企业必须的功能特性，例如安全,标识和管理等,扩展了开源Docker Distribution。

　　作为一个企业级私有Registry服务器,Harbor提供了更好的性能和安全。提升用户使用Registry构建和运行环境传输镜像的效率。

　　Harbor支持安装在多个Registry节点的镜像资源复制,镜像全部保存在私有Registry中，确保数据和知识产权在公司内部中管控,另外,Harbor也提供了高级的安全特性,诸如用户管理,访问控制和活动审计等。

　　官网地址:
　　　　https://vmware.github.io/

　　官方Github地址:
　　　　https://github.com/goharbor/harbor
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

**2>.Harbor功能介绍**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
　　基于角色的访问控制:
　　　　用户与Docker镜像仓库通过"项目"进行则指管理,一个用户可以对多个镜像仓库在同一个命名空间(project)里有不同的权限。

　　镜像复制:
　　　　镜像可以在多个Registry实例中复制(同步).尤其适合于负载均衡，高可用，混合云和多云的场景。

　　图形化用户界面:
　　　　用户可以通过浏览器来浏览，检索当前Docker镜像仓库，管理项目和命名空间。

　　AD/LDAP支持:
　　　　Harbor可以集成企业内部已有的AD/LDAP，用于鉴权认证管理。

　　审计管理:
　　　　所有针对镜像仓库的操作都可以被记录追溯,用于审计管理。
　　
　　国际化:
　　　　以拥有英文，中文，德文，日文和俄文的本地化版本。更多的语言将会添加进来。

　　RESTful API:
　　　　提供给管理员对于Harbor更多的操控，使得与其它管理软件集成变得更容易。

　　部署简单:
　　　　提供在线和离线两种安装工具，也可以安装到vSphere平台(OVA方式)虚拟设备。
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

**3>.下载Harbor(生产环境不建议大家直接用最新版，如果非要用最新版本建议在测试环境中做过足够的测试哟~)**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@docker102.yinzhengjie.org.cn ~]# cd /usr/local/src/
[root@docker102.yinzhengjie.org.cn /usr/local/src]# 
[root@docker102.yinzhengjie.org.cn /usr/local/src]# ll
total 0
[root@docker102.yinzhengjie.org.cn /usr/local/src]# 
[root@docker102.yinzhengjie.org.cn /usr/local/src]# wget https://storage.googleapis.com/harbor-releases/release-1.7.0/harbor-offline-installer-v1.7.5.tgz
--2020-01-28 01:14:30--  https://storage.googleapis.com/harbor-releases/release-1.7.0/harbor-offline-installer-v1.7.5.tgz
Resolving storage.googleapis.com (storage.googleapis.com)... 172.217.160.80, 2404:6800:4012::2010
Connecting to storage.googleapis.com (storage.googleapis.com)|172.217.160.80|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 580059210 (553M) [application/x-tar]
Saving to: ‘harbor-offline-installer-v1.7.5.tgz’

100%[==================================================================================================================================================================>] 580,059,210 8.86MB/s   in 2m 8s  

2020-01-28 01:16:45 (4.33 MB/s) - ‘harbor-offline-installer-v1.7.5.tgz’ saved [580059210/580059210]

[root@docker102.yinzhengjie.org.cn /usr/local/src]# 
[root@docker102.yinzhengjie.org.cn /usr/local/src]# ll
total 566468
-rw-r--r-- 1 root root 580059210 Apr  2  2019 harbor-offline-installer-v1.7.5.tgz
[root@docker102.yinzhengjie.org.cn /usr/local/src]# 
[root@docker102.yinzhengjie.org.cn /usr/local/src]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

![](https://img2018.cnblogs.com/i-beta/795254/202001/795254-20200127114603099-1311043733.png)

**二.Harbor节点准备依赖环境**

**1>.操作平台**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@docker103.yinzhengjie.org.cn ~]# uname -r
3.10.0-957.el7.x86_64
[root@docker103.yinzhengjie.org.cn ~]# 
[root@docker103.yinzhengjie.org.cn ~]# uname -m
x86_64
[root@docker103.yinzhengjie.org.cn ~]# 
[root@docker103.yinzhengjie.org.cn ~]# cat /etc/redhat-release 
CentOS Linux release 7.6.1810 (Core) 
[root@docker103.yinzhengjie.org.cn ~]# 
[root@docker103.yinzhengjie.org.cn ~]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

![](https://img2018.cnblogs.com/i-beta/795254/202001/795254-20200127182127056-402140071.png)

**2>.添加一块2.0T硬盘**

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@docker103.yinzhengjie.org.cn ~]# fdisk -l

Disk /dev/sdb: 2147.5 GB, 2147483648000 bytes, 4194304000 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes

WARNING: fdisk GPT support is currently new, and therefore in an experimental phase. Use at your own discretion.

Disk /dev/sda: 2199.0 GB, 2199023255552 bytes, 4294967296 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: gpt
Disk identifier: 60B0BC61-D7A6-4522-972F-E8B13E38A9C1


#         Start          End    Size  Type            Name
 1         2048         6143      2M  BIOS boot       
 2         6144      2103295      1G  Microsoft basic 
 3      2103296   4294723583      2T  Linux LVM       

Disk /dev/mapper/centos-root: 536.9 GB, 536870912000 bytes, 1048576000 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/mapper/centos-yinzhengjie: 1660.9 GB, 1660944384000 bytes, 3244032000 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes

[root@docker103.yinzhengjie.org.cn ~]# 
[root@docker103.yinzhengjie.org.cn ~]# 
```

\[root@docker103.yinzhengjie.org.cn ~\]# fdisk -l

**![](https://img2018.cnblogs.com/i-beta/795254/202001/795254-20200127174031993-1796985360.png)**

**3>.格式化新添加的硬盘**

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@docker103.yinzhengjie.org.cn ~]# mkfs.xfs -n ftype=1 /dev/sdb 
meta-data=/dev/sdb               isize=512    agcount=4, agsize=131072000 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=0, sparse=0
data     =                       bsize=4096   blocks=524288000, imaxpct=5
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
log      =internal log           bsize=4096   blocks=256000, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
[root@docker103.yinzhengjie.org.cn ~]# 
```

\[root@docker103.yinzhengjie.org.cn ~\]# mkfs.xfs -n ftype=1 /dev/sdb

![](https://img2018.cnblogs.com/i-beta/795254/202001/795254-20200127174255832-1850983459.png)

**4>.将格式化的硬盘挂载到docker默认的存储路径(由于此时还没有安装docker,因此需要咱们手动创建出"/var/lib/docker"目录)**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@docker103.yinzhengjie.org.cn ~]# mkdir -pv /var/lib/docker
mkdir: created directory ‘/var/lib/docker’
[root@docker103.yinzhengjie.org.cn ~]# 
[root@docker103.yinzhengjie.org.cn ~]# mount /dev/sdb /var/lib/docker/
[root@docker103.yinzhengjie.org.cn ~]# 
[root@docker103.yinzhengjie.org.cn ~]# xfs_info /var/lib/docker/
meta-data=/dev/sdb               isize=512    agcount=4, agsize=131072000 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=0 spinodes=0
data     =                       bsize=4096   blocks=524288000, imaxpct=5
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
log      =internal               bsize=4096   blocks=256000, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
[root@docker103.yinzhengjie.org.cn ~]# 
[root@docker103.yinzhengjie.org.cn ~]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

**![](https://img2018.cnblogs.com/i-beta/795254/202001/795254-20200127174629663-1819915231.png)**

**5>.设置硬盘开机自动挂载**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@docker103.yinzhengjie.org.cn ~]# blkid /dev/sdb
/dev/sdb: UUID="80802aac-ad94-465b-a42d-07f99b28ed6b" TYPE="xfs" 
[root@docker103.yinzhengjie.org.cn ~]# 
[root@docker103.yinzhengjie.org.cn ~]# vim /etc/fstab 
[root@docker103.yinzhengjie.org.cn ~]# 
[root@docker103.yinzhengjie.org.cn ~]# egrep -v "^$|^#" /etc/fstab 
/dev/mapper/centos-root /                       xfs     defaults        0 0
UUID=1865a93f-6113-4097-89dc-8c4ea5fdf68c /boot                   xfs     defaults        0 0
/dev/mapper/centos-yinzhengjie /yinzhengjie            xfs     defaults,noatime,nodiratime       0 0
UUID="80802aac-ad94-465b-a42d-07f99b28ed6b"    /var/lib/docker xfs    defaults    0 0
[root@docker103.yinzhengjie.org.cn ~]# 
[root@docker103.yinzhengjie.org.cn ~]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

**6>.安装docker**

```
　　博主推荐阅读:
　　　　https://www.cnblogs.com/yinzhengjie/p/12178843.html
```

**7>.启动docker服务**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@docker104.yinzhengjie.org.cn ~]# ll /var/lib/docker
total 0
[root@docker104.yinzhengjie.org.cn ~]# 
[root@docker104.yinzhengjie.org.cn ~]# systemctl start docker
[root@docker104.yinzhengjie.org.cn ~]# 
[root@docker104.yinzhengjie.org.cn ~]# systemctl enable docker
Created symlink from /etc/systemd/system/multi-user.target.wants/docker.service to /usr/lib/systemd/system/docker.service.
[root@docker104.yinzhengjie.org.cn ~]# 
[root@docker104.yinzhengjie.org.cn ~]# 
[root@docker104.yinzhengjie.org.cn ~]# ll /var/lib/docker
total 0
drwx------ 2 root root 24 Jan 28 02:04 builder
drwx--x--x 4 root root 92 Jan 28 02:04 buildkit
drwx------ 2 root root  6 Jan 28 02:04 containers
drwx------ 3 root root 22 Jan 28 02:04 image
drwxr-x--- 3 root root 19 Jan 28 02:04 network
drwx------ 3 root root 40 Jan 28 02:04 overlay2
drwx------ 4 root root 32 Jan 28 02:04 plugins
drwx------ 2 root root  6 Jan 28 02:04 runtimes
drwx------ 2 root root  6 Jan 28 02:04 swarm
drwx------ 2 root root  6 Jan 28 02:04 tmp
drwx------ 2 root root  6 Jan 28 02:04 trust
drwx------ 2 root root 25 Jan 28 02:04 volumes
[root@docker104.yinzhengjie.org.cn ~]# 
[root@docker104.yinzhengjie.org.cn ~]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

![](https://img2018.cnblogs.com/i-beta/795254/202001/795254-20200127180827738-830034047.png)

**8>.安装docker编排工具docker-compose(安装Harbor时需要依赖该服务)**

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@docker103.yinzhengjie.org.cn ~]# yum -y install epel-release
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
 * base: mirror.bit.edu.cn
 * extras: mirrors.tuna.tsinghua.edu.cn
 * updates: mirror.bit.edu.cn
Resolving Dependencies
--> Running transaction check
---> Package epel-release.noarch 0:7-11 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

=====================================================================================================================================================================================================================================
 Package                                                     Arch                                                  Version                                               Repository                                             Size
=====================================================================================================================================================================================================================================
Installing:
 epel-release                                                noarch                                                7-11                                                  extras                                                 15 k

Transaction Summary
=====================================================================================================================================================================================================================================
Install  1 Package

Total download size: 15 k
Installed size: 24 k
Downloading packages:
epel-release-7-11.noarch.rpm                                                                                                                                                                                  |  15 kB  00:00:00     
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : epel-release-7-11.noarch                                                                                                                                                                                          1/1 
  Verifying  : epel-release-7-11.noarch                                                                                                                                                                                          1/1 

Installed:
  epel-release.noarch 0:7-11                                                                                                                                                                                                         

Complete!
[root@docker103.yinzhengjie.org.cn ~]# 
```

\[root@docker103.yinzhengjie.org.cn ~\]# yum -y install epel-release

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@docker103.yinzhengjie.org.cn ~]# yum makecache fast
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
epel/x86_64/metalink                                                                                                                                                                                          | 8.9 kB  00:00:00     
 * base: mirror.bit.edu.cn
 * epel: mirrors.tuna.tsinghua.edu.cn
 * extras: mirror.bit.edu.cn
 * updates: mirrors.tuna.tsinghua.edu.cn
base                                                                                                                                                                                                          | 3.6 kB  00:00:00     
docker-ce-stable                                                                                                                                                                                              | 3.5 kB  00:00:00     
epel                                                                                                                                                                                                          | 5.3 kB  00:00:00     
extras                                                                                                                                                                                                        | 2.9 kB  00:00:00     
updates                                                                                                                                                                                                       | 2.9 kB  00:00:00     
(1/3): epel/x86_64/group_gz                                                                                                                                                                                   |  90 kB  00:00:00     
(2/3): epel/x86_64/updateinfo                                                                                                                                                                                 | 1.0 MB  00:00:00     
(3/3): epel/x86_64/primary_db                                                                                                                                                                                 | 6.9 MB  00:00:01     
Metadata Cache Created
[root@docker103.yinzhengjie.org.cn ~]# 
```

\[root@docker103.yinzhengjie.org.cn ~\]# yum makecache fast

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@docker103.yinzhengjie.org.cn ~]# yum -y install docker-compose
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
 * base: mirror.bit.edu.cn
 * epel: mirrors.tuna.tsinghua.edu.cn
 * extras: mirror.bit.edu.cn
 * updates: mirrors.tuna.tsinghua.edu.cn
Resolving Dependencies
--> Running transaction check
---> Package docker-compose.noarch 0:1.18.0-4.el7 will be installed
--> Processing Dependency: python(abi) = 3.6 for package: docker-compose-1.18.0-4.el7.noarch
--> Processing Dependency: python36-cached_property >= 1.2.0 for package: docker-compose-1.18.0-4.el7.noarch
--> Processing Dependency: python36-docker >= 2.6.1 for package: docker-compose-1.18.0-4.el7.noarch
--> Processing Dependency: python36-dockerpty >= 0.4.1 for package: docker-compose-1.18.0-4.el7.noarch
--> Processing Dependency: python36-docopt >= 0.6.1 for package: docker-compose-1.18.0-4.el7.noarch
--> Processing Dependency: python36-jsonschema >= 2.5.1 for package: docker-compose-1.18.0-4.el7.noarch
--> Processing Dependency: python36-pysocks >= 1.5.6 for package: docker-compose-1.18.0-4.el7.noarch
--> Processing Dependency: python36-requests >= 2.6.1 for package: docker-compose-1.18.0-4.el7.noarch
--> Processing Dependency: python36-six >= 1.3.0 for package: docker-compose-1.18.0-4.el7.noarch
--> Processing Dependency: python36-texttable >= 0.9.0 for package: docker-compose-1.18.0-4.el7.noarch
--> Processing Dependency: python36-websocket-client >= 0.32.0 for package: docker-compose-1.18.0-4.el7.noarch
--> Processing Dependency: python36-yaml >= 3.10 for package: docker-compose-1.18.0-4.el7.noarch
--> Processing Dependency: /usr/bin/python3.6 for package: docker-compose-1.18.0-4.el7.noarch
--> Processing Dependency: python36-setuptools for package: docker-compose-1.18.0-4.el7.noarch
--> Running transaction check
---> Package python3.x86_64 0:3.6.8-10.el7 will be installed
--> Processing Dependency: python3-libs(x86-64) = 3.6.8-10.el7 for package: python3-3.6.8-10.el7.x86_64
--> Processing Dependency: python3-pip for package: python3-3.6.8-10.el7.x86_64
--> Processing Dependency: libpython3.6m.so.1.0()(64bit) for package: python3-3.6.8-10.el7.x86_64
---> Package python3-setuptools.noarch 0:39.2.0-10.el7 will be installed
---> Package python36-PyYAML.x86_64 0:3.12-1.el7 will be installed
--> Processing Dependency: libyaml-0.so.2()(64bit) for package: python36-PyYAML-3.12-1.el7.x86_64
---> Package python36-cached_property.noarch 0:1.5.1-2.el7 will be installed
---> Package python36-docker.noarch 0:2.6.1-3.el7 will be installed
--> Processing Dependency: python36-docker-pycreds >= 0.2.1 for package: python36-docker-2.6.1-3.el7.noarch
---> Package python36-dockerpty.noarch 0:0.4.1-10.el7 will be installed
---> Package python36-docopt.noarch 0:0.6.2-8.el7 will be installed
---> Package python36-jsonschema.noarch 0:2.5.1-4.el7 will be installed
---> Package python36-pysocks.noarch 0:1.6.8-6.el7 will be installed
---> Package python36-requests.noarch 0:2.14.2-2.el7 will be installed
--> Processing Dependency: python36-chardet for package: python36-requests-2.14.2-2.el7.noarch
--> Processing Dependency: python36-idna for package: python36-requests-2.14.2-2.el7.noarch
--> Processing Dependency: python36-urllib3 for package: python36-requests-2.14.2-2.el7.noarch
---> Package python36-six.noarch 0:1.11.0-3.el7 will be installed
---> Package python36-texttable.noarch 0:1.6.2-1.el7 will be installed
---> Package python36-websocket-client.noarch 0:0.47.0-2.el7 will be installed
--> Running transaction check
---> Package libyaml.x86_64 0:0.1.4-11.el7_0 will be installed
---> Package python3-libs.x86_64 0:3.6.8-10.el7 will be installed
--> Processing Dependency: libtirpc.so.1()(64bit) for package: python3-libs-3.6.8-10.el7.x86_64
---> Package python3-pip.noarch 0:9.0.3-5.el7 will be installed
---> Package python36-chardet.noarch 0:3.0.4-1.el7 will be installed
---> Package python36-docker-pycreds.noarch 0:0.2.1-2.el7 will be installed
---> Package python36-idna.noarch 0:2.7-2.el7 will be installed
---> Package python36-urllib3.noarch 0:1.25.1-1.el7 will be installed
--> Processing Dependency: python36-rfc3986 for package: python36-urllib3-1.25.1-1.el7.noarch
--> Running transaction check
---> Package libtirpc.x86_64 0:0.2.4-0.16.el7 will be installed
---> Package python36-rfc3986.noarch 0:1.3.0-1.el7 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

=====================================================================================================================================================================================================================================
 Package                                                             Arch                                             Version                                                   Repository                                      Size
=====================================================================================================================================================================================================================================
Installing:
 docker-compose                                                      noarch                                           1.18.0-4.el7                                              epel                                           222 k
Installing for dependencies:
 libtirpc                                                            x86_64                                           0.2.4-0.16.el7                                            base                                            89 k
 libyaml                                                             x86_64                                           0.1.4-11.el7_0                                            base                                            55 k
 python3                                                             x86_64                                           3.6.8-10.el7                                              base                                            69 k
 python3-libs                                                        x86_64                                           3.6.8-10.el7                                              base                                           7.0 M
 python3-pip                                                         noarch                                           9.0.3-5.el7                                               base                                           1.8 M
 python3-setuptools                                                  noarch                                           39.2.0-10.el7                                             base                                           629 k
 python36-PyYAML                                                     x86_64                                           3.12-1.el7                                                epel                                           149 k
 python36-cached_property                                            noarch                                           1.5.1-2.el7                                               epel                                            18 k
 python36-chardet                                                    noarch                                           3.0.4-1.el7                                               epel                                           190 k
 python36-docker                                                     noarch                                           2.6.1-3.el7                                               epel                                           180 k
 python36-docker-pycreds                                             noarch                                           0.2.1-2.el7                                               epel                                            15 k
 python36-dockerpty                                                  noarch                                           0.4.1-10.el7                                              epel                                            29 k
 python36-docopt                                                     noarch                                           0.6.2-8.el7                                               epel                                            29 k
 python36-idna                                                       noarch                                           2.7-2.el7                                                 epel                                            98 k
 python36-jsonschema                                                 noarch                                           2.5.1-4.el7                                               epel                                            76 k
 python36-pysocks                                                    noarch                                           1.6.8-6.el7                                               epel                                            30 k
 python36-requests                                                   noarch                                           2.14.2-2.el7                                              epel                                           112 k
 python36-rfc3986                                                    noarch                                           1.3.0-1.el7                                               epel                                            49 k
 python36-six                                                        noarch                                           1.11.0-3.el7                                              epel                                            33 k
 python36-texttable                                                  noarch                                           1.6.2-1.el7                                               epel                                            23 k
 python36-urllib3                                                    noarch                                           1.25.1-1.el7                                              epel                                           173 k
 python36-websocket-client                                           noarch                                           0.47.0-2.el7                                              epel                                            59 k

Transaction Summary
=====================================================================================================================================================================================================================================
Install  1 Package (+22 Dependent packages)

Total download size: 11 M
Installed size: 56 M
Downloading packages:
(1/23): libyaml-0.1.4-11.el7_0.x86_64.rpm                                                                                                                                                                     |  55 kB  00:00:00     
(2/23): python3-3.6.8-10.el7.x86_64.rpm                                                                                                                                                                       |  69 kB  00:00:00     
(3/23): python3-pip-9.0.3-5.el7.noarch.rpm                                                                                                                                                                    | 1.8 MB  00:00:00     
(4/23): libtirpc-0.2.4-0.16.el7.x86_64.rpm                                                                                                                                                                    |  89 kB  00:00:03     
warning: /var/cache/yum/x86_64/7/epel/packages/docker-compose-1.18.0-4.el7.noarch.rpm: Header V3 RSA/SHA256 Signature, key ID 352c64e5: NOKEY====-                                                 ] 1.5 MB/s | 5.0 MB  00:00:04 ETA 
Public key for docker-compose-1.18.0-4.el7.noarch.rpm is not installed
(5/23): docker-compose-1.18.0-4.el7.noarch.rpm                                                                                                                                                                | 222 kB  00:00:04     
(6/23): python3-libs-3.6.8-10.el7.x86_64.rpm                                                                                                                                                                  | 7.0 MB  00:00:04     
(7/23): python3-setuptools-39.2.0-10.el7.noarch.rpm                                                                                                                                                           | 629 kB  00:00:01     
(8/23): python36-PyYAML-3.12-1.el7.x86_64.rpm                                                                                                                                                                 | 149 kB  00:00:00     
(9/23): python36-cached_property-1.5.1-2.el7.noarch.rpm                                                                                                                                                       |  18 kB  00:00:00     
(10/23): python36-chardet-3.0.4-1.el7.noarch.rpm                                                                                                                                                              | 190 kB  00:00:00     
(11/23): python36-docker-2.6.1-3.el7.noarch.rpm                                                                                                                                                               | 180 kB  00:00:00     
(12/23): python36-docker-pycreds-0.2.1-2.el7.noarch.rpm                                                                                                                                                       |  15 kB  00:00:00     
(13/23): python36-dockerpty-0.4.1-10.el7.noarch.rpm                                                                                                                                                           |  29 kB  00:00:00     
(14/23): python36-docopt-0.6.2-8.el7.noarch.rpm                                                                                                                                                               |  29 kB  00:00:00     
(15/23): python36-idna-2.7-2.el7.noarch.rpm                                                                                                                                                                   |  98 kB  00:00:00     
(16/23): python36-jsonschema-2.5.1-4.el7.noarch.rpm                                                                                                                                                           |  76 kB  00:00:00     
(17/23): python36-pysocks-1.6.8-6.el7.noarch.rpm                                                                                                                                                              |  30 kB  00:00:03     
(18/23): python36-requests-2.14.2-2.el7.noarch.rpm                                                                                                                                                            | 112 kB  00:00:00     
(19/23): python36-rfc3986-1.3.0-1.el7.noarch.rpm                                                                                                                                                              |  49 kB  00:00:00     
(20/23): python36-six-1.11.0-3.el7.noarch.rpm                                                                                                                                                                 |  33 kB  00:00:00     
(21/23): python36-texttable-1.6.2-1.el7.noarch.rpm                                                                                                                                                            |  23 kB  00:00:00     
(22/23): python36-urllib3-1.25.1-1.el7.noarch.rpm                                                                                                                                                             | 173 kB  00:00:00     
(23/23): python36-websocket-client-0.47.0-2.el7.noarch.rpm                                                                                                                                                    |  59 kB  00:00:00     
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
Total                                                                                                                                                                                                954 kB/s |  11 MB  00:00:11     
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
  Installing : libyaml-0.1.4-11.el7_0.x86_64                                                                                                                                                                                    1/23 
  Installing : libtirpc-0.2.4-0.16.el7.x86_64                                                                                                                                                                                   2/23 
  Installing : python3-pip-9.0.3-5.el7.noarch                                                                                                                                                                                   3/23 
  Installing : python3-setuptools-39.2.0-10.el7.noarch                                                                                                                                                                          4/23 
  Installing : python3-3.6.8-10.el7.x86_64                                                                                                                                                                                      5/23 
  Installing : python3-libs-3.6.8-10.el7.x86_64                                                                                                                                                                                 6/23 
  Installing : python36-six-1.11.0-3.el7.noarch                                                                                                                                                                                 7/23 
  Installing : python36-websocket-client-0.47.0-2.el7.noarch                                                                                                                                                                    8/23 
  Installing : python36-pysocks-1.6.8-6.el7.noarch                                                                                                                                                                              9/23 
  Installing : python36-dockerpty-0.4.1-10.el7.noarch                                                                                                                                                                          10/23 
  Installing : python36-docker-pycreds-0.2.1-2.el7.noarch                                                                                                                                                                      11/23 
  Installing : python36-PyYAML-3.12-1.el7.x86_64                                                                                                                                                                               12/23 
  Installing : python36-texttable-1.6.2-1.el7.noarch                                                                                                                                                                           13/23 
  Installing : python36-jsonschema-2.5.1-4.el7.noarch                                                                                                                                                                          14/23 
  Installing : python36-idna-2.7-2.el7.noarch                                                                                                                                                                                  15/23 
  Installing : python36-docopt-0.6.2-8.el7.noarch                                                                                                                                                                              16/23 
  Installing : python36-cached_property-1.5.1-2.el7.noarch                                                                                                                                                                     17/23 
  Installing : python36-chardet-3.0.4-1.el7.noarch                                                                                                                                                                             18/23 
  Installing : python36-rfc3986-1.3.0-1.el7.noarch                                                                                                                                                                             19/23 
  Installing : python36-urllib3-1.25.1-1.el7.noarch                                                                                                                                                                            20/23 
  Installing : python36-requests-2.14.2-2.el7.noarch                                                                                                                                                                           21/23 
  Installing : python36-docker-2.6.1-3.el7.noarch                                                                                                                                                                              22/23 
  Installing : docker-compose-1.18.0-4.el7.noarch                                                                                                                                                                              23/23 
  Verifying  : libtirpc-0.2.4-0.16.el7.x86_64                                                                                                                                                                                   1/23 
  Verifying  : python36-pysocks-1.6.8-6.el7.noarch                                                                                                                                                                              2/23 
  Verifying  : python3-libs-3.6.8-10.el7.x86_64                                                                                                                                                                                 3/23 
  Verifying  : docker-compose-1.18.0-4.el7.noarch                                                                                                                                                                               4/23 
  Verifying  : python3-pip-9.0.3-5.el7.noarch                                                                                                                                                                                   5/23 
  Verifying  : python36-urllib3-1.25.1-1.el7.noarch                                                                                                                                                                             6/23 
  Verifying  : python36-texttable-1.6.2-1.el7.noarch                                                                                                                                                                            7/23 
  Verifying  : python36-jsonschema-2.5.1-4.el7.noarch                                                                                                                                                                           8/23 
  Verifying  : python36-idna-2.7-2.el7.noarch                                                                                                                                                                                   9/23 
  Verifying  : python36-websocket-client-0.47.0-2.el7.noarch                                                                                                                                                                   10/23 
  Verifying  : python36-PyYAML-3.12-1.el7.x86_64                                                                                                                                                                               11/23 
  Verifying  : python36-requests-2.14.2-2.el7.noarch                                                                                                                                                                           12/23 
  Verifying  : python36-dockerpty-0.4.1-10.el7.noarch                                                                                                                                                                          13/23 
  Verifying  : python36-docker-2.6.1-3.el7.noarch                                                                                                                                                                              14/23 
  Verifying  : python36-six-1.11.0-3.el7.noarch                                                                                                                                                                                15/23 
  Verifying  : python3-setuptools-39.2.0-10.el7.noarch                                                                                                                                                                         16/23 
  Verifying  : python36-docopt-0.6.2-8.el7.noarch                                                                                                                                                                              17/23 
  Verifying  : python36-cached_property-1.5.1-2.el7.noarch                                                                                                                                                                     18/23 
  Verifying  : python3-3.6.8-10.el7.x86_64                                                                                                                                                                                     19/23 
  Verifying  : libyaml-0.1.4-11.el7_0.x86_64                                                                                                                                                                                   20/23 
  Verifying  : python36-chardet-3.0.4-1.el7.noarch                                                                                                                                                                             21/23 
  Verifying  : python36-docker-pycreds-0.2.1-2.el7.noarch                                                                                                                                                                      22/23 
  Verifying  : python36-rfc3986-1.3.0-1.el7.noarch                                                                                                                                                                             23/23 

Installed:
  docker-compose.noarch 0:1.18.0-4.el7                                                                                                                                                                                               

Dependency Installed:
  libtirpc.x86_64 0:0.2.4-0.16.el7               libyaml.x86_64 0:0.1.4-11.el7_0                   python3.x86_64 0:3.6.8-10.el7                   python3-libs.x86_64 0:3.6.8-10.el7      python3-pip.noarch 0:9.0.3-5.el7          
  python3-setuptools.noarch 0:39.2.0-10.el7      python36-PyYAML.x86_64 0:3.12-1.el7               python36-cached_property.noarch 0:1.5.1-2.el7   python36-chardet.noarch 0:3.0.4-1.el7   python36-docker.noarch 0:2.6.1-3.el7      
  python36-docker-pycreds.noarch 0:0.2.1-2.el7   python36-dockerpty.noarch 0:0.4.1-10.el7          python36-docopt.noarch 0:0.6.2-8.el7            python36-idna.noarch 0:2.7-2.el7        python36-jsonschema.noarch 0:2.5.1-4.el7  
  python36-pysocks.noarch 0:1.6.8-6.el7          python36-requests.noarch 0:2.14.2-2.el7           python36-rfc3986.noarch 0:1.3.0-1.el7           python36-six.noarch 0:1.11.0-3.el7      python36-texttable.noarch 0:1.6.2-1.el7   
  python36-urllib3.noarch 0:1.25.1-1.el7         python36-websocket-client.noarch 0:0.47.0-2.el7  

Complete!
[root@docker103.yinzhengjie.org.cn ~]# 
```

\[root@docker103.yinzhengjie.org.cn ~\]# yum -y install docker-compose

**三.Harbor部署实战**

**1>.解压harbor安装包**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@docker103.yinzhengjie.org.cn ~]# cd /usr/local/src/
[root@docker103.yinzhengjie.org.cn /usr/local/src]# 
[root@docker103.yinzhengjie.org.cn /usr/local/src]# ll
total 566468
-rw-r--r-- 1 root root 580059210 Jan 28 01:36 harbor-offline-installer-v1.7.5.tgz
[root@docker103.yinzhengjie.org.cn /usr/local/src]# 
[root@docker103.yinzhengjie.org.cn /usr/local/src]# tar xf harbor-offline-installer-v1.7.5.tgz 
[root@docker103.yinzhengjie.org.cn /usr/local/src]# 
[root@docker103.yinzhengjie.org.cn /usr/local/src]# ll
total 566468
drwxr-xr-x 3 root root       270 Jan 28 03:45 harbor
-rw-r--r-- 1 root root 580059210 Jan 28 01:36 harbor-offline-installer-v1.7.5.tgz
[root@docker103.yinzhengjie.org.cn /usr/local/src]# 
[root@docker103.yinzhengjie.org.cn /usr/local/src]# ll harbor
total 572840
drwxr-xr-x 3 root root        23 Jan 28 03:45 common
-rw-r--r-- 1 root root       939 Apr  1  2019 docker-compose.chartmuseum.yml
-rw-r--r-- 1 root root       975 Apr  1  2019 docker-compose.clair.yml
-rw-r--r-- 1 root root      1434 Apr  1  2019 docker-compose.notary.yml
-rw-r--r-- 1 root root      5608 Apr  1  2019 docker-compose.yml
-rw-r--r-- 1 root root      8045 Jan 28 03:52 harbor.cfg
-rw-r--r-- 1 root root 585234819 Apr  1  2019 harbor.v1.7.5.tar.gz
-rwxr-xr-x 1 root root      5739 Apr  1  2019 install.sh
-rw-r--r-- 1 root root     11347 Apr  1  2019 LICENSE
-rw-r--r-- 1 root root   1263409 Apr  1  2019 open_source_license
-rwxr-xr-x 1 root root     36337 Apr  1  2019 prepare
[root@docker103.yinzhengjie.org.cn /usr/local/src]# 
[root@docker103.yinzhengjie.org.cn /usr/local/src]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

**2>.修改Harbor的主机名(可以理解为外部访问Harbor的地址，当然也可以写IP地址哟~)**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@docker103.yinzhengjie.org.cn /usr/local/src]# egrep -v "^#|^$" harbor/harbor.cfg | grep hostname
hostname = reg.mydomain.com
[root@docker103.yinzhengjie.org.cn /usr/local/src]# 
[root@docker103.yinzhengjie.org.cn /usr/local/src]# sed -r -i 's#(hostname = )reg.mydomain.com#\1docker103.yinzhengjie.org.cn#' harbor/harbor.cfg 
[root@docker103.yinzhengjie.org.cn /usr/local/src]# 
[root@docker103.yinzhengjie.org.cn /usr/local/src]# egrep -v "^#|^$" harbor/harbor.cfg | grep hostname
hostname = docker103.yinzhengjie.org.cn
[root@docker103.yinzhengjie.org.cn /usr/local/src]#  
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

**3>.修改Harbor的默认密码**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@docker103.yinzhengjie.org.cn /usr/local/src]# egrep -v "^#|^$" harbor/harbor.cfg | grep harbor_admin_password
harbor_admin_password = Harbor12345
[root@docker103.yinzhengjie.org.cn /usr/local/src]# 
[root@docker103.yinzhengjie.org.cn /usr/local/src]# sed -r -i 's#(harbor_admin_password = )Harbor12345#\1yinzhengjie#' harbor/harbor.cfg 
[root@docker103.yinzhengjie.org.cn /usr/local/src]# 
[root@docker103.yinzhengjie.org.cn /usr/local/src]# egrep -v "^#|^$" harbor/harbor.cfg | grep harbor_admin_password
harbor_admin_password = yinzhengjie
[root@docker103.yinzhengjie.org.cn /usr/local/src]# 
[root@docker103.yinzhengjie.org.cn /usr/local/src]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

**4>.安装Harbor服务**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@docker103.yinzhengjie.org.cn ~]# cd /usr/local/src/harbor/
[root@docker103.yinzhengjie.org.cn /usr/local/src/harbor]# 
[root@docker103.yinzhengjie.org.cn /usr/local/src/harbor]# ll
total 572840
drwxr-xr-x 3 root root        23 Jan 28 03:45 common
-rw-r--r-- 1 root root       939 Apr  1  2019 docker-compose.chartmuseum.yml
-rw-r--r-- 1 root root       975 Apr  1  2019 docker-compose.clair.yml
-rw-r--r-- 1 root root      1434 Apr  1  2019 docker-compose.notary.yml
-rw-r--r-- 1 root root      5608 Apr  1  2019 docker-compose.yml
-rw-r--r-- 1 root root      8045 Jan 28 03:55 harbor.cfg
-rw-r--r-- 1 root root 585234819 Apr  1  2019 harbor.v1.7.5.tar.gz
-rwxr-xr-x 1 root root      5739 Apr  1  2019 install.sh
-rw-r--r-- 1 root root     11347 Apr  1  2019 LICENSE
-rw-r--r-- 1 root root   1263409 Apr  1  2019 open_source_license
-rwxr-xr-x 1 root root     36337 Apr  1  2019 prepare
[root@docker103.yinzhengjie.org.cn /usr/local/src/harbor]# 
[root@docker103.yinzhengjie.org.cn /usr/local/src/harbor]# ./install.sh 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

![](https://img2018.cnblogs.com/i-beta/795254/202001/795254-20200127211109022-1917755009.png)

**5>.访问Harbor的WebUI**

![](https://img2018.cnblogs.com/i-beta/795254/202001/795254-20200127215101029-2098281163.png)

**四.使用Harbor新建项目**

**1>.点击"新建项目"**

![](https://img2018.cnblogs.com/i-beta/795254/202001/795254-20200127215607353-1949159658.png)

**2>.项目创建成功**

![](https://img2018.cnblogs.com/i-beta/795254/202001/795254-20200127220218926-2106192051.png)

**3>.查看项目的"镜像仓库"**

![](https://img2018.cnblogs.com/i-beta/795254/202001/795254-20200127220309650-783050387.png)

**4>.查看项目的"配置管理"**

**![](https://img2018.cnblogs.com/i-beta/795254/202001/795254-20200127220443293-1111313943.png)**

**五.将本地镜像上传到自建的Harbor镜像仓库中**

**1>.登录自建的Harbor镜像仓库**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@docker101.yinzhengjie.org.cn ~]# vim /etc/docker/daemon.json 
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# cat /etc/docker/daemon.json 
{
  "registry-mirrors": ["https://tuv7rqqq.mirror.aliyuncs.com"],
  "insecure-registries":["docker103.yinzhengjie.org.cn"]
}
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# systemctl restart docker
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# docker login -u admin docker103.yinzhengjie.org.cn
Password: 
WARNING! Your password will be stored unencrypted in /root/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

**2>.为本地镜像打tag**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@docker101.yinzhengjie.org.cn ~]# docker image ls
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
centos-haproxy      v1.8.20             1858fe05d96f        3 days ago          606MB
registry            latest              708bc6af7e5e        3 days ago          25.8MB
tomcat-app01        v0.1                bf45c22f2d5b        4 days ago          983MB
tomcat-base         8.5.50              9ff79f369094        5 days ago          968MB
jdk-base            1.8.0_231           0f63a97ddc85        5 days ago          953MB
centos-base         7.6.1810            b4931fd9ace2        5 days ago          551MB
centos              centos7.6.1810      f1cb7c7d58b7        10 months ago       202MB
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# docker image tag centos-base:7.6.1810 docker103.yinzhengjie.org.cn/base_images/centos-base:v7.6.1810
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# docker image ls
REPOSITORY                                             TAG                 IMAGE ID            CREATED             SIZE
centos-haproxy                                         v1.8.20             1858fe05d96f        3 days ago          606MB
registry                                               latest              708bc6af7e5e        3 days ago          25.8MB
tomcat-app01                                           v0.1                bf45c22f2d5b        4 days ago          983MB
tomcat-base                                            8.5.50              9ff79f369094        5 days ago          968MB
jdk-base                                               1.8.0_231           0f63a97ddc85        5 days ago          953MB
docker103.yinzhengjie.org.cn/base_images/centos-base   v7.6.1810           b4931fd9ace2        5 days ago          551MB
centos-base                                            7.6.1810            b4931fd9ace2        5 days ago          551MB
centos                                                 centos7.6.1810      f1cb7c7d58b7        10 months ago       202MB
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

![](https://img2018.cnblogs.com/i-beta/795254/202001/795254-20200127221103762-783081126.png)

**3>.上传镜像成功**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@docker101.yinzhengjie.org.cn ~]# docker image ls
REPOSITORY                                             TAG                 IMAGE ID            CREATED             SIZE
centos-haproxy                                         v1.8.20             1858fe05d96f        3 days ago          606MB
registry                                               latest              708bc6af7e5e        4 days ago          25.8MB
tomcat-app01                                           v0.1                bf45c22f2d5b        4 days ago          983MB
tomcat-base                                            8.5.50              9ff79f369094        5 days ago          968MB
jdk-base                                               1.8.0_231           0f63a97ddc85        5 days ago          953MB
centos-base                                            7.6.1810            b4931fd9ace2        5 days ago          551MB
docker103.yinzhengjie.org.cn/base_images/centos-base   v7.6.1810           b4931fd9ace2        5 days ago          551MB
centos                                                 centos7.6.1810      f1cb7c7d58b7        10 months ago       202MB
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# docker image push docker103.yinzhengjie.org.cn/base_images/centos-base:v7.6.1810 
The push refers to repository [docker103.yinzhengjie.org.cn/base_images/centos-base]
0f448859d86e: Pushed 
89169d87dbe2: Pushed 
v7.6.1810: digest: sha256:62c5a70f2846bd7f8ecd65785e379d0e00acf33ae899f0ec96754a3731b2d425 size: 742
[root@docker101.yinzhengjie.org.cn ~]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

![](https://img2018.cnblogs.com/i-beta/795254/202001/795254-20200127222145825-1635414514.png)

**六.下载镜像**

**1>.**登录自建的Harbor镜像仓库****

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@docker102.yinzhengjie.org.cn ~]# vim /etc/docker/daemon.json 
[root@docker102.yinzhengjie.org.cn ~]# 
[root@docker102.yinzhengjie.org.cn ~]# 
[root@docker102.yinzhengjie.org.cn ~]# cat /etc/docker/daemon.json 
{
  "registry-mirrors": ["https://tuv7rqqq.mirror.aliyuncs.com"],
  "insecure-registries":["docker103.yinzhengjie.org.cn"]
}
[root@docker102.yinzhengjie.org.cn ~]# 
[root@docker102.yinzhengjie.org.cn ~]# 
[root@docker102.yinzhengjie.org.cn ~]# systemctl restart docker
[root@docker102.yinzhengjie.org.cn ~]# 
[root@docker102.yinzhengjie.org.cn ~]# docker login docker103.yinzhengjie.org.cn
Username: admin
Password: 
WARNING! Your password will be stored unencrypted in /root/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded
[root@docker102.yinzhengjie.org.cn ~]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

 ![](https://img2018.cnblogs.com/i-beta/795254/202001/795254-20200127222759964-434187424.png)

**2>.下载镜像到本地**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@docker102.yinzhengjie.org.cn ~]# docker image ls
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
[root@docker102.yinzhengjie.org.cn ~]# 
[root@docker102.yinzhengjie.org.cn ~]# docker image pull docker103.yinzhengjie.org.cn/base_images/centos-base:v7.6.1810 
v7.6.1810: Pulling from base_images/centos-base
ac9208207ada: Pull complete 
1a93113d354a: Pull complete 
Digest: sha256:62c5a70f2846bd7f8ecd65785e379d0e00acf33ae899f0ec96754a3731b2d425
Status: Downloaded newer image for docker103.yinzhengjie.org.cn/base_images/centos-base:v7.6.1810
docker103.yinzhengjie.org.cn/base_images/centos-base:v7.6.1810
[root@docker102.yinzhengjie.org.cn ~]# 
[root@docker102.yinzhengjie.org.cn ~]# docker image ls
REPOSITORY                                             TAG                 IMAGE ID            CREATED             SIZE
docker103.yinzhengjie.org.cn/base_images/centos-base   v7.6.1810           b4931fd9ace2        5 days ago          551MB
[root@docker102.yinzhengjie.org.cn ~]# 
[root@docker102.yinzhengjie.org.cn ~]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

![](https://img2018.cnblogs.com/i-beta/795254/202001/795254-20200127223221150-331189155.png)

**七.编写Harbor服务的启动脚本**

**1>.查看harbor的安装目录中关于docker-compose的配置文件(更多关于docker-compose工具的使用可参考:[https://www.cnblogs.com/yinzhengjie/p/12250356.html](https://www.cnblogs.com/yinzhengjie/p/12250356.html))**

![](https://img2018.cnblogs.com/i-beta/795254/202002/795254-20200202102301391-918931816.png)

**2>.使用docker-compose组件管理harbor服务**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@docker103.yinzhengjie.org.cn /usr/local/src/harbor]# docker-compose --help
Define and run multi-container applications with Docker.

Usage:
  docker-compose [-f <arg>...] [options] [COMMAND] [ARGS...]
  docker-compose -h|--help

Options:
  -f, --file FILE             Specify an alternate compose file (default: docker-compose.yml)
  -p, --project-name NAME     Specify an alternate project name (default: directory name)
  --verbose                   Show more output
  --no-ansi                   Do not print ANSI control characters
  -v, --version               Print version and exit
  -H, --host HOST             Daemon socket to connect to

  --tls                       Use TLS; implied by --tlsverify
  --tlscacert CA_PATH         Trust certs signed only by this CA
  --tlscert CLIENT_CERT_PATH  Path to TLS certificate file
  --tlskey TLS_KEY_PATH       Path to TLS key file
  --tlsverify                 Use TLS and verify the remote
  --skip-hostname-check       Don't check the daemon's hostname against the name specified
                              in the client certificate (for example if your docker host
                              is an IP address)
  --project-directory PATH    Specify an alternate working directory
                              (default: the path of the Compose file)

Commands:
  build              Build or rebuild services
  bundle             Generate a Docker bundle from the Compose file
  config             Validate and view the Compose file
  create             Create services
  down               Stop and remove containers, networks, images, and volumes
  events             Receive real time events from containers
  exec               Execute a command in a running container
  help               Get help on a command
  images             List images
  kill               Kill containers
  logs               View output from containers
  pause              Pause services
  port               Print the public port for a port binding
  ps                 List containers
  pull               Pull service images
  push               Push service images
  restart            Restart services
  rm                 Remove stopped containers
  run                Run a one-off command
  scale              Set number of containers for a service
  start              Start services
  stop               Stop services
  top                Display the running processes
  unpause            Unpause services
  up                 Create and start containers
  version            Show the Docker-Compose version information
[root@docker103.yinzhengjie.org.cn /usr/local/src/harbor]# 
[root@docker103.yinzhengjie.org.cn /usr/local/src/harbor]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@docker103.yinzhengjie.org.cn /usr/local/src/harbor]# ss -ntl
State       Recv-Q Send-Q                                                                                 Local Address:Port                                                                                                Peer Address:Port              
LISTEN      0      20480                                                                                      127.0.0.1:1514                                                                                                           *:*                  
LISTEN      0      128                                                                                                *:22                                                                                                             *:*                  
LISTEN      0      128                                                                                               :::22                                                                                                            :::*                  
[root@docker103.yinzhengjie.org.cn /usr/local/src/harbor]# 
[root@docker103.yinzhengjie.org.cn /usr/local/src/harbor]# docker-compose start
Starting log         ... done
Starting registry    ... done
Starting registryctl ... done
Starting postgresql  ... done
Starting adminserver ... done
Starting core        ... done
Starting portal      ... done
Starting redis       ... done
Starting jobservice  ... done
Starting proxy       ... done
[root@docker103.yinzhengjie.org.cn /usr/local/src/harbor]# 
[root@docker103.yinzhengjie.org.cn /usr/local/src/harbor]# 
[root@docker103.yinzhengjie.org.cn /usr/local/src/harbor]# ss -ntl
State       Recv-Q Send-Q                                                                                 Local Address:Port                                                                                                Peer Address:Port              
LISTEN      0      20480                                                                                      127.0.0.1:1514                                                                                                           *:*                  
LISTEN      0      128                                                                                                *:22                                                                                                             *:*                  
LISTEN      0      20480                                                                                             :::80                                                                                                            :::*                  
LISTEN      0      128                                                                                               :::22                                                                                                            :::*                  
LISTEN      0      20480                                                                                             :::443                                                                                                           :::*                  
LISTEN      0      20480                                                                                             :::4443                                                                                                          :::*                  
[root@docker103.yinzhengjie.org.cn /usr/local/src/harbor]# 
[root@docker103.yinzhengjie.org.cn /usr/local/src/harbor]# 
```

\[root@docker103.yinzhengjie.org.cn /usr/local/src/harbor\]# docker-compose start　　　　　　　　　　　　　　#启动harbor服务

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@docker103.yinzhengjie.org.cn /usr/local/src/harbor]# ss -ntl
State       Recv-Q Send-Q                                                                                 Local Address:Port                                                                                                Peer Address:Port              
LISTEN      0      20480                                                                                      127.0.0.1:1514                                                                                                           *:*                  
LISTEN      0      128                                                                                                *:22                                                                                                             *:*                  
LISTEN      0      20480                                                                                             :::80                                                                                                            :::*                  
LISTEN      0      128                                                                                               :::22                                                                                                            :::*                  
LISTEN      0      20480                                                                                             :::443                                                                                                           :::*                  
LISTEN      0      20480                                                                                             :::4443                                                                                                          :::*                  
[root@docker103.yinzhengjie.org.cn /usr/local/src/harbor]# 
[root@docker103.yinzhengjie.org.cn /usr/local/src/harbor]# docker-compose stop
Stopping nginx              ... done
Stopping harbor-jobservice  ... done
Stopping harbor-portal      ... done
Stopping harbor-core        ... done
Stopping registryctl        ... done
Stopping harbor-db          ... done
Stopping redis              ... done
Stopping registry           ... done
Stopping harbor-adminserver ... done
Stopping harbor-log         ... done
[root@docker103.yinzhengjie.org.cn /usr/local/src/harbor]# 
[root@docker103.yinzhengjie.org.cn /usr/local/src/harbor]# ss -ntl
State       Recv-Q Send-Q                                                                                 Local Address:Port                                                                                                Peer Address:Port              
LISTEN      0      128                                                                                                *:22                                                                                                             *:*                  
LISTEN      0      128                                                                                               :::22                                                                                                            :::*                  
[root@docker103.yinzhengjie.org.cn /usr/local/src/harbor]# 
[root@docker103.yinzhengjie.org.cn /usr/local/src/harbor]# 
```

\[root@docker103.yinzhengjie.org.cn /usr/local/src/harbor\]# docker-compose stop　　　　　　　　　　　　　　 #停止harbor服务

**3>.将harbor服务设置为开机自启动**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@docker103.yinzhengjie.org.cn ~]# vim /etc/rc.d/rc.local 
[root@docker103.yinzhengjie.org.cn ~]# 
[root@docker103.yinzhengjie.org.cn ~]# tail -2 /etc/rc.d/rc.local 
#Add by yinzhengjie
cd /usr/local/src/harbor && docker-compose start
[root@docker103.yinzhengjie.org.cn ~]# 
[root@docker103.yinzhengjie.org.cn ~]# ll /etc/rc.d/rc.local 
-rw-r--r-- 1 root root 543 Feb  2 18:28 /etc/rc.d/rc.local
[root@docker103.yinzhengjie.org.cn ~]# 
[root@docker103.yinzhengjie.org.cn ~]# chmod +x /etc/rc.d/rc.local 
[root@docker103.yinzhengjie.org.cn ~]# 
[root@docker103.yinzhengjie.org.cn ~]# ll /etc/rc.d/rc.local 
-rwxr-xr-x 1 root root 543 Feb  2 18:28 /etc/rc.d/rc.local
[root@docker103.yinzhengjie.org.cn ~]# 
[root@docker103.yinzhengjie.org.cn ~]# reboot 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

![](https://img2018.cnblogs.com/i-beta/795254/202002/795254-20200202103135401-745064838.png)

当你的才华还撑不起你的野心的时候，你就应该静下心来学习。当你的能力还驾驭不了你的目标的时候，你就应该沉下心来历练。问问自己，想要怎样的人生。 欢迎加入基础架构自动化运维：598432640，大数据SRE进阶之路：959042252，DevOps进阶之路：526991186