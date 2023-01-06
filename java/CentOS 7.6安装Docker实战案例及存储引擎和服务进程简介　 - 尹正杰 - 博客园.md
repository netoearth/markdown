　　　　　　　　**CentOS 7.6安装Docker实战案例及存储引擎和服务进程简介**　

　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　**作者：尹正杰**

**版权声明：原创作品，谢绝转载！否则将追究法律责任。**

****一.通过CentOS安装docker****

**1>.**查看操作平台****

![](https://img2018.cnblogs.com/i-beta/795254/202001/795254-20200113114044200-624029731.png)

**2>.安装必要的一些系统工具** 

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@docker202.yinzhengjie.org.cn ~]#  yum -y install yum-utils device-mapper-persistent-data lvm2
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
 * base: mirror.bit.edu.cn
 * extras: mirror.bit.edu.cn
 * updates: mirrors.tuna.tsinghua.edu.cn
Resolving Dependencies
--> Running transaction check
---> Package device-mapper-persistent-data.x86_64 0:0.7.3-3.el7 will be updated
---> Package device-mapper-persistent-data.x86_64 0:0.8.5-1.el7 will be an update
---> Package lvm2.x86_64 7:2.02.180-8.el7 will be updated
---> Package lvm2.x86_64 7:2.02.185-2.el7_7.2 will be an update
--> Processing Dependency: lvm2-libs = 7:2.02.185-2.el7_7.2 for package: 7:lvm2-2.02.185-2.el7_7.2.x86_64
---> Package yum-utils.noarch 0:1.1.31-52.el7 will be installed
--> Processing Dependency: python-kitchen for package: yum-utils-1.1.31-52.el7.noarch
--> Processing Dependency: libxml2-python for package: yum-utils-1.1.31-52.el7.noarch
--> Running transaction check
---> Package libxml2-python.x86_64 0:2.9.1-6.el7_2.3 will be installed
---> Package lvm2-libs.x86_64 7:2.02.180-8.el7 will be updated
---> Package lvm2-libs.x86_64 7:2.02.185-2.el7_7.2 will be an update
--> Processing Dependency: device-mapper-event = 7:1.02.158-2.el7_7.2 for package: 7:lvm2-libs-2.02.185-2.el7_7.2.x86_64
---> Package python-kitchen.noarch 0:1.1.1-5.el7 will be installed
--> Processing Dependency: python-chardet for package: python-kitchen-1.1.1-5.el7.noarch
--> Running transaction check
---> Package device-mapper-event.x86_64 7:1.02.149-8.el7 will be updated
---> Package device-mapper-event.x86_64 7:1.02.158-2.el7_7.2 will be an update
--> Processing Dependency: device-mapper-event-libs = 7:1.02.158-2.el7_7.2 for package: 7:device-mapper-event-1.02.158-2.el7_7.2.x86_64
--> Processing Dependency: device-mapper = 7:1.02.158-2.el7_7.2 for package: 7:device-mapper-event-1.02.158-2.el7_7.2.x86_64
---> Package python-chardet.noarch 0:2.2.1-3.el7 will be installed
--> Running transaction check
---> Package device-mapper.x86_64 7:1.02.149-8.el7 will be updated
--> Processing Dependency: device-mapper = 7:1.02.149-8.el7 for package: 7:device-mapper-libs-1.02.149-8.el7.x86_64
---> Package device-mapper.x86_64 7:1.02.158-2.el7_7.2 will be an update
---> Package device-mapper-event-libs.x86_64 7:1.02.149-8.el7 will be updated
---> Package device-mapper-event-libs.x86_64 7:1.02.158-2.el7_7.2 will be an update
--> Running transaction check
---> Package device-mapper-libs.x86_64 7:1.02.149-8.el7 will be updated
---> Package device-mapper-libs.x86_64 7:1.02.158-2.el7_7.2 will be an update
--> Finished Dependency Resolution

Dependencies Resolved

=====================================================================================================================================================================================================================================
 Package                                                              Arch                                          Version                                                     Repository                                      Size
=====================================================================================================================================================================================================================================
Installing:
 yum-utils                                                            noarch                                        1.1.31-52.el7                                               base                                           121 k
Updating:
 device-mapper-persistent-data                                        x86_64                                        0.8.5-1.el7                                                 base                                           423 k
 lvm2                                                                 x86_64                                        7:2.02.185-2.el7_7.2                                        updates                                        1.3 M
Installing for dependencies:
 libxml2-python                                                       x86_64                                        2.9.1-6.el7_2.3                                             base                                           247 k
 python-chardet                                                       noarch                                        2.2.1-3.el7                                                 base                                           227 k
 python-kitchen                                                       noarch                                        1.1.1-5.el7                                                 base                                           267 k
Updating for dependencies:
 device-mapper                                                        x86_64                                        7:1.02.158-2.el7_7.2                                        updates                                        294 k
 device-mapper-event                                                  x86_64                                        7:1.02.158-2.el7_7.2                                        updates                                        190 k
 device-mapper-event-libs                                             x86_64                                        7:1.02.158-2.el7_7.2                                        updates                                        189 k
 device-mapper-libs                                                   x86_64                                        7:1.02.158-2.el7_7.2                                        updates                                        322 k
 lvm2-libs                                                            x86_64                                        7:2.02.185-2.el7_7.2                                        updates                                        1.1 M

Transaction Summary
=====================================================================================================================================================================================================================================
Install  1 Package  (+3 Dependent packages)
Upgrade  2 Packages (+5 Dependent packages)

Total download size: 4.6 M
Downloading packages:
Delta RPMs disabled because /usr/bin/applydeltarpm not installed.
(1/11): device-mapper-1.02.158-2.el7_7.2.x86_64.rpm                                                                                                                                                           | 294 kB  00:00:00     
(2/11): device-mapper-event-1.02.158-2.el7_7.2.x86_64.rpm                                                                                                                                                     | 190 kB  00:00:00     
(3/11): device-mapper-event-libs-1.02.158-2.el7_7.2.x86_64.rpm                                                                                                                                                | 189 kB  00:00:00     
(4/11): device-mapper-libs-1.02.158-2.el7_7.2.x86_64.rpm                                                                                                                                                      | 322 kB  00:00:00     
(5/11): lvm2-2.02.185-2.el7_7.2.x86_64.rpm                                                                                                                                                                    | 1.3 MB  00:00:00     
(6/11): lvm2-libs-2.02.185-2.el7_7.2.x86_64.rpm                                                                                                                                                               | 1.1 MB  00:00:00     
(7/11): libxml2-python-2.9.1-6.el7_2.3.x86_64.rpm                                                                                                                                                             | 247 kB  00:00:00     
(8/11): python-chardet-2.2.1-3.el7.noarch.rpm                                                                                                                                                                 | 227 kB  00:00:00     
(9/11): device-mapper-persistent-data-0.8.5-1.el7.x86_64.rpm                                                                                                                                                  | 423 kB  00:00:00     
(10/11): python-kitchen-1.1.1-5.el7.noarch.rpm                                                                                                                                                                | 267 kB  00:00:00     
(11/11): yum-utils-1.1.31-52.el7.noarch.rpm                                                                                                                                                                   | 121 kB  00:00:00     
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
Total                                                                                                                                                                                                7.1 MB/s | 4.6 MB  00:00:00     
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Updating   : 7:device-mapper-1.02.158-2.el7_7.2.x86_64                                                                                                                                                                        1/18 
  Updating   : 7:device-mapper-libs-1.02.158-2.el7_7.2.x86_64                                                                                                                                                                   2/18 
  Updating   : 7:device-mapper-event-libs-1.02.158-2.el7_7.2.x86_64                                                                                                                                                             3/18 
  Updating   : 7:device-mapper-event-1.02.158-2.el7_7.2.x86_64                                                                                                                                                                  4/18 
  Updating   : 7:lvm2-libs-2.02.185-2.el7_7.2.x86_64                                                                                                                                                                            5/18 
  Installing : python-chardet-2.2.1-3.el7.noarch                                                                                                                                                                                6/18 
  Installing : python-kitchen-1.1.1-5.el7.noarch                                                                                                                                                                                7/18 
  Installing : libxml2-python-2.9.1-6.el7_2.3.x86_64                                                                                                                                                                            8/18 
  Updating   : device-mapper-persistent-data-0.8.5-1.el7.x86_64                                                                                                                                                                 9/18 
  Updating   : 7:lvm2-2.02.185-2.el7_7.2.x86_64                                                                                                                                                                                10/18 
  Installing : yum-utils-1.1.31-52.el7.noarch                                                                                                                                                                                  11/18 
  Cleanup    : 7:lvm2-2.02.180-8.el7.x86_64                                                                                                                                                                                    12/18 
  Cleanup    : 7:lvm2-libs-2.02.180-8.el7.x86_64                                                                                                                                                                               13/18 
  Cleanup    : 7:device-mapper-event-1.02.149-8.el7.x86_64                                                                                                                                                                     14/18 
  Cleanup    : 7:device-mapper-event-libs-1.02.149-8.el7.x86_64                                                                                                                                                                15/18 
  Cleanup    : 7:device-mapper-1.02.149-8.el7.x86_64                                                                                                                                                                           16/18 
  Cleanup    : 7:device-mapper-libs-1.02.149-8.el7.x86_64                                                                                                                                                                      17/18 
  Cleanup    : device-mapper-persistent-data-0.7.3-3.el7.x86_64                                                                                                                                                                18/18 
  Verifying  : 7:device-mapper-libs-1.02.158-2.el7_7.2.x86_64                                                                                                                                                                   1/18 
  Verifying  : yum-utils-1.1.31-52.el7.noarch                                                                                                                                                                                   2/18 
  Verifying  : device-mapper-persistent-data-0.8.5-1.el7.x86_64                                                                                                                                                                 3/18 
  Verifying  : 7:lvm2-2.02.185-2.el7_7.2.x86_64                                                                                                                                                                                 4/18 
  Verifying  : libxml2-python-2.9.1-6.el7_2.3.x86_64                                                                                                                                                                            5/18 
  Verifying  : 7:lvm2-libs-2.02.185-2.el7_7.2.x86_64                                                                                                                                                                            6/18 
  Verifying  : python-kitchen-1.1.1-5.el7.noarch                                                                                                                                                                                7/18 
  Verifying  : 7:device-mapper-1.02.158-2.el7_7.2.x86_64                                                                                                                                                                        8/18 
  Verifying  : 7:device-mapper-event-1.02.158-2.el7_7.2.x86_64                                                                                                                                                                  9/18 
  Verifying  : 7:device-mapper-event-libs-1.02.158-2.el7_7.2.x86_64                                                                                                                                                            10/18 
  Verifying  : python-chardet-2.2.1-3.el7.noarch                                                                                                                                                                               11/18 
  Verifying  : device-mapper-persistent-data-0.7.3-3.el7.x86_64                                                                                                                                                                12/18 
  Verifying  : 7:lvm2-2.02.180-8.el7.x86_64                                                                                                                                                                                    13/18 
  Verifying  : 7:device-mapper-event-1.02.149-8.el7.x86_64                                                                                                                                                                     14/18 
  Verifying  : 7:lvm2-libs-2.02.180-8.el7.x86_64                                                                                                                                                                               15/18 
  Verifying  : 7:device-mapper-1.02.149-8.el7.x86_64                                                                                                                                                                           16/18 
  Verifying  : 7:device-mapper-libs-1.02.149-8.el7.x86_64                                                                                                                                                                      17/18 
  Verifying  : 7:device-mapper-event-libs-1.02.149-8.el7.x86_64                                                                                                                                                                18/18 

Installed:
  yum-utils.noarch 0:1.1.31-52.el7                                                                                                                                                                                                   

Dependency Installed:
  libxml2-python.x86_64 0:2.9.1-6.el7_2.3                                       python-chardet.noarch 0:2.2.1-3.el7                                       python-kitchen.noarch 0:1.1.1-5.el7                                      

Updated:
  device-mapper-persistent-data.x86_64 0:0.8.5-1.el7                                                                         lvm2.x86_64 7:2.02.185-2.el7_7.2                                                                        

Dependency Updated:
  device-mapper.x86_64 7:1.02.158-2.el7_7.2          device-mapper-event.x86_64 7:1.02.158-2.el7_7.2          device-mapper-event-libs.x86_64 7:1.02.158-2.el7_7.2          device-mapper-libs.x86_64 7:1.02.158-2.el7_7.2         
  lvm2-libs.x86_64 7:2.02.185-2.el7_7.2             

Complete!
[root@docker202.yinzhengjie.org.cn ~]# 
```

\[root@docker202.yinzhengjie.org.cn ~\]# yum -y install yum-utils device-mapper-persistent-data lvm2

**3>.添加软件源信息** 

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@docker202.yinzhengjie.org.cn ~]# ll /etc/yum.repos.d/
total 32
-rw-r--r--. 1 root root 1664 Nov 23  2018 CentOS-Base.repo
-rw-r--r--. 1 root root 1309 Nov 23  2018 CentOS-CR.repo
-rw-r--r--. 1 root root  649 Nov 23  2018 CentOS-Debuginfo.repo
-rw-r--r--. 1 root root  314 Nov 23  2018 CentOS-fasttrack.repo
-rw-r--r--. 1 root root  630 Nov 23  2018 CentOS-Media.repo
-rw-r--r--. 1 root root 1331 Nov 23  2018 CentOS-Sources.repo
-rw-r--r--. 1 root root 5701 Nov 23  2018 CentOS-Vault.repo
[root@docker202.yinzhengjie.org.cn ~]# 
[root@docker202.yinzhengjie.org.cn ~]# 
[root@docker202.yinzhengjie.org.cn ~]# yum-config-manager --add-repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
Loaded plugins: fastestmirror
adding repo from: https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
grabbing file https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo to /etc/yum.repos.d/docker-ce.repo
repo saved to /etc/yum.repos.d/docker-ce.repo
[root@docker202.yinzhengjie.org.cn ~]# 
[root@docker202.yinzhengjie.org.cn ~]# ll /etc/yum.repos.d/
total 36
-rw-r--r--. 1 root root 1664 Nov 23  2018 CentOS-Base.repo
-rw-r--r--. 1 root root 1309 Nov 23  2018 CentOS-CR.repo
-rw-r--r--. 1 root root  649 Nov 23  2018 CentOS-Debuginfo.repo
-rw-r--r--. 1 root root  314 Nov 23  2018 CentOS-fasttrack.repo
-rw-r--r--. 1 root root  630 Nov 23  2018 CentOS-Media.repo
-rw-r--r--. 1 root root 1331 Nov 23  2018 CentOS-Sources.repo
-rw-r--r--. 1 root root 5701 Nov 23  2018 CentOS-Vault.repo
-rw-r--r--. 1 root root 2640 Jan 12 16:10 docker-ce.repo
[root@docker202.yinzhengjie.org.cn ~]# 
[root@docker202.yinzhengjie.org.cn ~]# 
```

\[root@docker202.yinzhengjie.org.cn ~\]# yum-config-manager --add-repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@docker202.yinzhengjie.org.cn ~]# yum list docker-ce.x86_64 --showduplicates | sort -r　　　　　　　　　　　　　　#查看docker的版本
 * updates: mirrors.aliyun.com
Loading mirror speeds from cached hostfile
Loaded plugins: fastestmirror
Installed Packages
 * extras: mirror.bit.edu.cn
docker-ce.x86_64            3:19.03.5-3.el7                    docker-ce-stable 
docker-ce.x86_64            3:19.03.5-3.el7                    @docker-ce-stable
docker-ce.x86_64            3:19.03.4-3.el7                    docker-ce-stable 
docker-ce.x86_64            3:19.03.3-3.el7                    docker-ce-stable 
docker-ce.x86_64            3:19.03.2-3.el7                    docker-ce-stable 
docker-ce.x86_64            3:19.03.1-3.el7                    docker-ce-stable 
docker-ce.x86_64            3:19.03.0-3.el7                    docker-ce-stable 
docker-ce.x86_64            3:18.09.9-3.el7                    docker-ce-stable 
docker-ce.x86_64            3:18.09.8-3.el7                    docker-ce-stable 
docker-ce.x86_64            3:18.09.7-3.el7                    docker-ce-stable 
docker-ce.x86_64            3:18.09.6-3.el7                    docker-ce-stable 
docker-ce.x86_64            3:18.09.5-3.el7                    docker-ce-stable 
docker-ce.x86_64            3:18.09.4-3.el7                    docker-ce-stable 
docker-ce.x86_64            3:18.09.3-3.el7                    docker-ce-stable 
docker-ce.x86_64            3:18.09.2-3.el7                    docker-ce-stable 
docker-ce.x86_64            3:18.09.1-3.el7                    docker-ce-stable 
docker-ce.x86_64            3:18.09.0-3.el7                    docker-ce-stable 
docker-ce.x86_64            18.06.3.ce-3.el7                   docker-ce-stable 
docker-ce.x86_64            18.06.2.ce-3.el7                   docker-ce-stable 
docker-ce.x86_64            18.06.1.ce-3.el7                   docker-ce-stable 
docker-ce.x86_64            18.06.0.ce-3.el7                   docker-ce-stable 
docker-ce.x86_64            18.03.1.ce-1.el7.centos            docker-ce-stable 
docker-ce.x86_64            18.03.0.ce-1.el7.centos            docker-ce-stable 
docker-ce.x86_64            17.12.1.ce-1.el7.centos            docker-ce-stable 
docker-ce.x86_64            17.12.0.ce-1.el7.centos            docker-ce-stable 
docker-ce.x86_64            17.09.1.ce-1.el7.centos            docker-ce-stable 
docker-ce.x86_64            17.09.0.ce-1.el7.centos            docker-ce-stable 
docker-ce.x86_64            17.06.2.ce-1.el7.centos            docker-ce-stable 
docker-ce.x86_64            17.06.1.ce-1.el7.centos            docker-ce-stable 
docker-ce.x86_64            17.06.0.ce-1.el7.centos            docker-ce-stable 
docker-ce.x86_64            17.03.3.ce-1.el7                   docker-ce-stable 
docker-ce.x86_64            17.03.2.ce-1.el7.centos            docker-ce-stable 
docker-ce.x86_64            17.03.1.ce-1.el7.centos            docker-ce-stable 
docker-ce.x86_64            17.03.0.ce-1.el7.centos            docker-ce-stable 
 * base: mirrors.huaweicloud.com
Available Packages
[root@docker202.yinzhengjie.org.cn ~]# 
```

\[root@docker202.yinzhengjie.org.cn ~\]# yum list docker-ce.x86\_64 --showduplicates | sort -r　　　　　　　　　　　　　　#查看docker的版本

**4>.更新yum源并安装Docker-CE** 

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@docker202.yinzhengjie.org.cn ~]# yum makecache fast
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
 * base: mirrors.huaweicloud.com
 * extras: mirror.bit.edu.cn
 * updates: mirrors.huaweicloud.com
base                                                                                                                                                                                                          | 3.6 kB  00:00:00     
docker-ce-stable                                                                                                                                                                                              | 3.5 kB  00:00:00     
extras                                                                                                                                                                                                        | 2.9 kB  00:00:00     
updates                                                                                                                                                                                                       | 2.9 kB  00:00:00     
(1/2): docker-ce-stable/x86_64/primary_db                                                                                                                                                                     |  37 kB  00:00:00     
(2/2): docker-ce-stable/x86_64/updateinfo                                                                                                                                                                     |   55 B  00:00:00     
Metadata Cache Created
[root@docker202.yinzhengjie.org.cn ~]# 
```

\[root@docker202.yinzhengjie.org.cn ~\]# yum makecache fast

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@docker202.yinzhengjie.org.cn ~]# yum -y install docker-ce
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
 * base: mirror.bit.edu.cn
 * extras: mirror.bit.edu.cn
 * updates: mirrors.huaweicloud.com
Resolving Dependencies
--> Running transaction check
---> Package docker-ce.x86_64 3:19.03.5-3.el7 will be installed
--> Processing Dependency: container-selinux >= 2:2.74 for package: 3:docker-ce-19.03.5-3.el7.x86_64
--> Processing Dependency: containerd.io >= 1.2.2-3 for package: 3:docker-ce-19.03.5-3.el7.x86_64
--> Processing Dependency: libseccomp >= 2.3 for package: 3:docker-ce-19.03.5-3.el7.x86_64
--> Processing Dependency: docker-ce-cli for package: 3:docker-ce-19.03.5-3.el7.x86_64
--> Processing Dependency: libcgroup for package: 3:docker-ce-19.03.5-3.el7.x86_64
--> Processing Dependency: libseccomp.so.2()(64bit) for package: 3:docker-ce-19.03.5-3.el7.x86_64
--> Running transaction check
---> Package container-selinux.noarch 2:2.107-3.el7 will be installed
--> Processing Dependency: policycoreutils-python for package: 2:container-selinux-2.107-3.el7.noarch
---> Package containerd.io.x86_64 0:1.2.10-3.2.el7 will be installed
---> Package docker-ce-cli.x86_64 1:19.03.5-3.el7 will be installed
---> Package libcgroup.x86_64 0:0.41-21.el7 will be installed
---> Package libseccomp.x86_64 0:2.3.1-3.el7 will be installed
--> Running transaction check
---> Package policycoreutils-python.x86_64 0:2.5-33.el7 will be installed
--> Processing Dependency: policycoreutils = 2.5-33.el7 for package: policycoreutils-python-2.5-33.el7.x86_64
--> Processing Dependency: setools-libs >= 3.3.8-4 for package: policycoreutils-python-2.5-33.el7.x86_64
--> Processing Dependency: libsemanage-python >= 2.5-14 for package: policycoreutils-python-2.5-33.el7.x86_64
--> Processing Dependency: audit-libs-python >= 2.1.3-4 for package: policycoreutils-python-2.5-33.el7.x86_64
--> Processing Dependency: python-IPy for package: policycoreutils-python-2.5-33.el7.x86_64
--> Processing Dependency: libqpol.so.1(VERS_1.4)(64bit) for package: policycoreutils-python-2.5-33.el7.x86_64
--> Processing Dependency: libqpol.so.1(VERS_1.2)(64bit) for package: policycoreutils-python-2.5-33.el7.x86_64
--> Processing Dependency: libapol.so.4(VERS_4.0)(64bit) for package: policycoreutils-python-2.5-33.el7.x86_64
--> Processing Dependency: checkpolicy for package: policycoreutils-python-2.5-33.el7.x86_64
--> Processing Dependency: libqpol.so.1()(64bit) for package: policycoreutils-python-2.5-33.el7.x86_64
--> Processing Dependency: libapol.so.4()(64bit) for package: policycoreutils-python-2.5-33.el7.x86_64
--> Running transaction check
---> Package audit-libs-python.x86_64 0:2.8.5-4.el7 will be installed
--> Processing Dependency: audit-libs(x86-64) = 2.8.5-4.el7 for package: audit-libs-python-2.8.5-4.el7.x86_64
---> Package checkpolicy.x86_64 0:2.5-8.el7 will be installed
---> Package libsemanage-python.x86_64 0:2.5-14.el7 will be installed
---> Package policycoreutils.x86_64 0:2.5-29.el7 will be updated
---> Package policycoreutils.x86_64 0:2.5-33.el7 will be an update
---> Package python-IPy.noarch 0:0.75-6.el7 will be installed
---> Package setools-libs.x86_64 0:3.3.8-4.el7 will be installed
--> Running transaction check
---> Package audit-libs.x86_64 0:2.8.4-4.el7 will be updated
--> Processing Dependency: audit-libs(x86-64) = 2.8.4-4.el7 for package: audit-2.8.4-4.el7.x86_64
---> Package audit-libs.x86_64 0:2.8.5-4.el7 will be an update
--> Running transaction check
---> Package audit.x86_64 0:2.8.4-4.el7 will be updated
---> Package audit.x86_64 0:2.8.5-4.el7 will be an update
--> Finished Dependency Resolution

Dependencies Resolved

=====================================================================================================================================================================================================================================
 Package                                                       Arch                                          Version                                                   Repository                                               Size
=====================================================================================================================================================================================================================================
Installing:
 docker-ce                                                     x86_64                                        3:19.03.5-3.el7                                           docker-ce-stable                                         24 M
Installing for dependencies:
 audit-libs-python                                             x86_64                                        2.8.5-4.el7                                               base                                                     76 k
 checkpolicy                                                   x86_64                                        2.5-8.el7                                                 base                                                    295 k
 container-selinux                                             noarch                                        2:2.107-3.el7                                             extras                                                   39 k
 containerd.io                                                 x86_64                                        1.2.10-3.2.el7                                            docker-ce-stable                                         23 M
 docker-ce-cli                                                 x86_64                                        1:19.03.5-3.el7                                           docker-ce-stable                                         39 M
 libcgroup                                                     x86_64                                        0.41-21.el7                                               base                                                     66 k
 libseccomp                                                    x86_64                                        2.3.1-3.el7                                               base                                                     56 k
 libsemanage-python                                            x86_64                                        2.5-14.el7                                                base                                                    113 k
 policycoreutils-python                                        x86_64                                        2.5-33.el7                                                base                                                    457 k
 python-IPy                                                    noarch                                        0.75-6.el7                                                base                                                     32 k
 setools-libs                                                  x86_64                                        3.3.8-4.el7                                               base                                                    620 k
Updating for dependencies:
 audit                                                         x86_64                                        2.8.5-4.el7                                               base                                                    256 k
 audit-libs                                                    x86_64                                        2.8.5-4.el7                                               base                                                    102 k
 policycoreutils                                               x86_64                                        2.5-33.el7                                                base                                                    916 k

Transaction Summary
=====================================================================================================================================================================================================================================
Install  1 Package  (+11 Dependent packages)
Upgrade             (  3 Dependent packages)

Total download size: 90 M
Downloading packages:
Delta RPMs disabled because /usr/bin/applydeltarpm not installed.
(1/15): audit-libs-2.8.5-4.el7.x86_64.rpm                                                                                                                                                                     | 102 kB  00:00:00     
(2/15): audit-2.8.5-4.el7.x86_64.rpm                                                                                                                                                                          | 256 kB  00:00:00     
(3/15): audit-libs-python-2.8.5-4.el7.x86_64.rpm                                                                                                                                                              |  76 kB  00:00:00     
(4/15): checkpolicy-2.5-8.el7.x86_64.rpm                                                                                                                                                                      | 295 kB  00:00:00     
(5/15): container-selinux-2.107-3.el7.noarch.rpm                                                                                                                                                              |  39 kB  00:00:00     
warning: /var/cache/yum/x86_64/7/docker-ce-stable/packages/docker-ce-19.03.5-3.el7.x86_64.rpm: Header V4 RSA/SHA512 Signature, key ID 621e9f35: NOKEY                                              ] 6.9 MB/s |  43 MB  00:00:06 ETA 
Public key for docker-ce-19.03.5-3.el7.x86_64.rpm is not installed
(6/15): docker-ce-19.03.5-3.el7.x86_64.rpm                                                                                                                                                                    |  24 MB  00:00:04     
(7/15): containerd.io-1.2.10-3.2.el7.x86_64.rpm                                                                                                                                                               |  23 MB  00:00:04     
(8/15): libseccomp-2.3.1-3.el7.x86_64.rpm                                                                                                                                                                     |  56 kB  00:00:00     
(9/15): libcgroup-0.41-21.el7.x86_64.rpm                                                                                                                                                                      |  66 kB  00:00:00     
(10/15): libsemanage-python-2.5-14.el7.x86_64.rpm                                                                                                                                                             | 113 kB  00:00:00     
(11/15): policycoreutils-2.5-33.el7.x86_64.rpm                                                                                                                                                                | 916 kB  00:00:00     
(12/15): python-IPy-0.75-6.el7.noarch.rpm                                                                                                                                                                     |  32 kB  00:00:00     
(13/15): policycoreutils-python-2.5-33.el7.x86_64.rpm                                                                                                                                                         | 457 kB  00:00:01     
(14/15): docker-ce-cli-19.03.5-3.el7.x86_64.rpm                                                                                                                                                               |  39 MB  00:00:03     
(15/15): setools-libs-3.3.8-4.el7.x86_64.rpm                                                                                                                                                                  | 620 kB  00:00:02     
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
Total                                                                                                                                                                                                 11 MB/s |  90 MB  00:00:08     
Retrieving key from https://mirrors.aliyun.com/docker-ce/linux/centos/gpg
Importing GPG key 0x621E9F35:
 Userid     : "Docker Release (CE rpm) <docker@docker.com>"
 Fingerprint: 060a 61c5 1b55 8a7f 742b 77aa c52f eb6b 621e 9f35
 From       : https://mirrors.aliyun.com/docker-ce/linux/centos/gpg
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Updating   : audit-libs-2.8.5-4.el7.x86_64                                                                                                                                                                                    1/18 
  Updating   : policycoreutils-2.5-33.el7.x86_64                                                                                                                                                                                2/18 
  Installing : libcgroup-0.41-21.el7.x86_64                                                                                                                                                                                     3/18 
  Installing : audit-libs-python-2.8.5-4.el7.x86_64                                                                                                                                                                             4/18 
  Installing : setools-libs-3.3.8-4.el7.x86_64                                                                                                                                                                                  5/18 
  Installing : 1:docker-ce-cli-19.03.5-3.el7.x86_64                                                                                                                                                                             6/18 
  Installing : checkpolicy-2.5-8.el7.x86_64                                                                                                                                                                                     7/18 
  Installing : libseccomp-2.3.1-3.el7.x86_64                                                                                                                                                                                    8/18 
  Installing : libsemanage-python-2.5-14.el7.x86_64                                                                                                                                                                             9/18 
  Installing : python-IPy-0.75-6.el7.noarch                                                                                                                                                                                    10/18 
  Installing : policycoreutils-python-2.5-33.el7.x86_64                                                                                                                                                                        11/18 
  Installing : 2:container-selinux-2.107-3.el7.noarch                                                                                                                                                                          12/18 
  Installing : containerd.io-1.2.10-3.2.el7.x86_64                                                                                                                                                                             13/18 
  Installing : 3:docker-ce-19.03.5-3.el7.x86_64                                                                                                                                                                                14/18 
  Updating   : audit-2.8.5-4.el7.x86_64                                                                                                                                                                                        15/18 
  Cleanup    : policycoreutils-2.5-29.el7.x86_64                                                                                                                                                                               16/18 
  Cleanup    : audit-2.8.4-4.el7.x86_64                                                                                                                                                                                        17/18 
  Cleanup    : audit-libs-2.8.4-4.el7.x86_64                                                                                                                                                                                   18/18 
  Verifying  : audit-libs-2.8.5-4.el7.x86_64                                                                                                                                                                                    1/18 
  Verifying  : policycoreutils-python-2.5-33.el7.x86_64                                                                                                                                                                         2/18 
  Verifying  : audit-2.8.5-4.el7.x86_64                                                                                                                                                                                         3/18 
  Verifying  : 3:docker-ce-19.03.5-3.el7.x86_64                                                                                                                                                                                 4/18 
  Verifying  : audit-libs-python-2.8.5-4.el7.x86_64                                                                                                                                                                             5/18 
  Verifying  : python-IPy-0.75-6.el7.noarch                                                                                                                                                                                     6/18 
  Verifying  : libsemanage-python-2.5-14.el7.x86_64                                                                                                                                                                             7/18 
  Verifying  : 2:container-selinux-2.107-3.el7.noarch                                                                                                                                                                           8/18 
  Verifying  : libseccomp-2.3.1-3.el7.x86_64                                                                                                                                                                                    9/18 
  Verifying  : checkpolicy-2.5-8.el7.x86_64                                                                                                                                                                                    10/18 
  Verifying  : policycoreutils-2.5-33.el7.x86_64                                                                                                                                                                               11/18 
  Verifying  : containerd.io-1.2.10-3.2.el7.x86_64                                                                                                                                                                             12/18 
  Verifying  : 1:docker-ce-cli-19.03.5-3.el7.x86_64                                                                                                                                                                            13/18 
  Verifying  : setools-libs-3.3.8-4.el7.x86_64                                                                                                                                                                                 14/18 
  Verifying  : libcgroup-0.41-21.el7.x86_64                                                                                                                                                                                    15/18 
  Verifying  : policycoreutils-2.5-29.el7.x86_64                                                                                                                                                                               16/18 
  Verifying  : audit-2.8.4-4.el7.x86_64                                                                                                                                                                                        17/18 
  Verifying  : audit-libs-2.8.4-4.el7.x86_64                                                                                                                                                                                   18/18 

Installed:
  docker-ce.x86_64 3:19.03.5-3.el7                                                                                                                                                                                                   

Dependency Installed:
  audit-libs-python.x86_64 0:2.8.5-4.el7 checkpolicy.x86_64 0:2.5-8.el7         container-selinux.noarch 2:2.107-3.el7     containerd.io.x86_64 0:1.2.10-3.2.el7 docker-ce-cli.x86_64 1:19.03.5-3.el7 libcgroup.x86_64 0:0.41-21.el7
  libseccomp.x86_64 0:2.3.1-3.el7        libsemanage-python.x86_64 0:2.5-14.el7 policycoreutils-python.x86_64 0:2.5-33.el7 python-IPy.noarch 0:0.75-6.el7        setools-libs.x86_64 0:3.3.8-4.el7   

Dependency Updated:
  audit.x86_64 0:2.8.5-4.el7                                             audit-libs.x86_64 0:2.8.5-4.el7                                             policycoreutils.x86_64 0:2.5-33.el7                                            

Complete!
[root@docker202.yinzhengjie.org.cn ~]# 
```

\[root@docker202.yinzhengjie.org.cn ~\]# yum -y install docker-ce

**5>.开启docker服务** 

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@docker202.yinzhengjie.org.cn ~]# systemctl status docker
● docker.service - Docker Application Container Engine
   Loaded: loaded (/usr/lib/systemd/system/docker.service; disabled; vendor preset: disabled)
   Active: inactive (dead)
     Docs: https://docs.docker.com
[root@docker202.yinzhengjie.org.cn ~]# 
[root@docker202.yinzhengjie.org.cn ~]# 
[root@docker202.yinzhengjie.org.cn ~]# systemctl status docker
● docker.service - Docker Application Container Engine
   Loaded: loaded (/usr/lib/systemd/system/docker.service; disabled; vendor preset: disabled)
   Active: inactive (dead)
     Docs: https://docs.docker.com
[root@docker202.yinzhengjie.org.cn ~]# 
[root@docker202.yinzhengjie.org.cn ~]# 
[root@docker202.yinzhengjie.org.cn ~]# systemctl start docker
[root@docker202.yinzhengjie.org.cn ~]# 
[root@docker202.yinzhengjie.org.cn ~]# systemctl enable docker
Created symlink from /etc/systemd/system/multi-user.target.wants/docker.service to /usr/lib/systemd/system/docker.service.
[root@docker202.yinzhengjie.org.cn ~]# 
[root@docker202.yinzhengjie.org.cn ~]# 
[root@docker202.yinzhengjie.org.cn ~]# systemctl status docker
● docker.service - Docker Application Container Engine
   Loaded: loaded (/usr/lib/systemd/system/docker.service; enabled; vendor preset: disabled)
   Active: active (running) since Mon 2020-01-13 19:47:37 CST; 16s ago
     Docs: https://docs.docker.com
 Main PID: 18718 (dockerd)
   CGroup: /system.slice/docker.service
           └─18718 /usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock

Jan 13 19:47:37 docker202.yinzhengjie.org.cn dockerd[18718]: time="2020-01-13T19:47:37.291334215+08:00" level=info msg="scheme \"unix\" not registered, fallback to default scheme" module=grpc
Jan 13 19:47:37 docker202.yinzhengjie.org.cn dockerd[18718]: time="2020-01-13T19:47:37.291346994+08:00" level=info msg="ccResolverWrapper: sending update to cc: {[{unix:///run/containerd/containerd.sock 0  <nil>}...}" module=grpc
Jan 13 19:47:37 docker202.yinzhengjie.org.cn dockerd[18718]: time="2020-01-13T19:47:37.291356143+08:00" level=info msg="ClientConn switching balancer to \"pick_first\"" module=grpc
Jan 13 19:47:37 docker202.yinzhengjie.org.cn dockerd[18718]: time="2020-01-13T19:47:37.313238648+08:00" level=info msg="Loading containers: start."
Jan 13 19:47:37 docker202.yinzhengjie.org.cn dockerd[18718]: time="2020-01-13T19:47:37.592162634+08:00" level=info msg="Default bridge (docker0) is assigned with an IP address 172.17.0.0/16. Daemon option --bip c...ed IP address"
Jan 13 19:47:37 docker202.yinzhengjie.org.cn dockerd[18718]: time="2020-01-13T19:47:37.796559906+08:00" level=info msg="Loading containers: done."
Jan 13 19:47:37 docker202.yinzhengjie.org.cn dockerd[18718]: time="2020-01-13T19:47:37.810400452+08:00" level=info msg="Docker daemon" commit=633a0ea graphdriver(s)=overlay2 version=19.03.5
Jan 13 19:47:37 docker202.yinzhengjie.org.cn dockerd[18718]: time="2020-01-13T19:47:37.810723229+08:00" level=info msg="Daemon has completed initialization"
Jan 13 19:47:37 docker202.yinzhengjie.org.cn dockerd[18718]: time="2020-01-13T19:47:37.843366711+08:00" level=info msg="API listen on /var/run/docker.sock"
Jan 13 19:47:37 docker202.yinzhengjie.org.cn systemd[1]: Started Docker Application Container Engine.
Hint: Some lines were ellipsized, use -l to show in full.
[root@docker202.yinzhengjie.org.cn ~]# 
[root@docker202.yinzhengjie.org.cn ~]# 
```

\[root@docker202.yinzhengjie.org.cn ~\]# systemctl start docker

**6>.查看docker的版本信息**

 **![](https://img2018.cnblogs.com/i-beta/795254/202001/795254-20200113121006352-1848947585.png)**

**7>.安装指定的docker版本**

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@docker201.yinzhengjie.org.cn ~]# yum list docker-ce.x86_64 --showduplicates | sort -r                           #查看docker的版本
 * updates: mirrors.huaweicloud.com
Loading mirror speeds from cached hostfile
Loaded plugins: fastestmirror
 * extras: mirror.bit.edu.cn
docker-ce.x86_64            3:19.03.5-3.el7                     docker-ce-stable
docker-ce.x86_64            3:19.03.4-3.el7                     docker-ce-stable
docker-ce.x86_64            3:19.03.3-3.el7                     docker-ce-stable
docker-ce.x86_64            3:19.03.2-3.el7                     docker-ce-stable
docker-ce.x86_64            3:19.03.1-3.el7                     docker-ce-stable
docker-ce.x86_64            3:19.03.0-3.el7                     docker-ce-stable
docker-ce.x86_64            3:18.09.9-3.el7                     docker-ce-stable
docker-ce.x86_64            3:18.09.8-3.el7                     docker-ce-stable
docker-ce.x86_64            3:18.09.7-3.el7                     docker-ce-stable
docker-ce.x86_64            3:18.09.6-3.el7                     docker-ce-stable
docker-ce.x86_64            3:18.09.5-3.el7                     docker-ce-stable
docker-ce.x86_64            3:18.09.4-3.el7                     docker-ce-stable
docker-ce.x86_64            3:18.09.3-3.el7                     docker-ce-stable
docker-ce.x86_64            3:18.09.2-3.el7                     docker-ce-stable
docker-ce.x86_64            3:18.09.1-3.el7                     docker-ce-stable
docker-ce.x86_64            3:18.09.0-3.el7                     docker-ce-stable
docker-ce.x86_64            18.06.3.ce-3.el7                    docker-ce-stable
docker-ce.x86_64            18.06.2.ce-3.el7                    docker-ce-stable
docker-ce.x86_64            18.06.1.ce-3.el7                    docker-ce-stable
docker-ce.x86_64            18.06.0.ce-3.el7                    docker-ce-stable
docker-ce.x86_64            18.03.1.ce-1.el7.centos             docker-ce-stable
docker-ce.x86_64            18.03.0.ce-1.el7.centos             docker-ce-stable
docker-ce.x86_64            17.12.1.ce-1.el7.centos             docker-ce-stable
docker-ce.x86_64            17.12.0.ce-1.el7.centos             docker-ce-stable
docker-ce.x86_64            17.09.1.ce-1.el7.centos             docker-ce-stable
docker-ce.x86_64            17.09.0.ce-1.el7.centos             docker-ce-stable
docker-ce.x86_64            17.06.2.ce-1.el7.centos             docker-ce-stable
docker-ce.x86_64            17.06.1.ce-1.el7.centos             docker-ce-stable
docker-ce.x86_64            17.06.0.ce-1.el7.centos             docker-ce-stable
docker-ce.x86_64            17.03.3.ce-1.el7                    docker-ce-stable
docker-ce.x86_64            17.03.2.ce-1.el7.centos             docker-ce-stable
docker-ce.x86_64            17.03.1.ce-1.el7.centos             docker-ce-stable
docker-ce.x86_64            17.03.0.ce-1.el7.centos             docker-ce-stable
 * base: mirror.bit.edu.cn
Available Packages
[root@docker201.yinzhengjie.org.cn ~]# 
```

\[root@docker201.yinzhengjie.org.cn ~\]# yum list docker-ce.x86\_64 --showduplicates | sort -r 　　　　　　　　　　　　　　　　#查看docker的版本

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@docker201.yinzhengjie.org.cn ~]# yum -y install docker-ce-18.06.3.ce-3.el7
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
 * base: mirrors.huaweicloud.com
 * extras: mirror.bit.edu.cn
 * updates: mirrors.aliyun.com
Resolving Dependencies
--> Running transaction check
---> Package docker-ce.x86_64 0:18.06.3.ce-3.el7 will be installed
--> Processing Dependency: container-selinux >= 2.9 for package: docker-ce-18.06.3.ce-3.el7.x86_64
--> Processing Dependency: libseccomp >= 2.3 for package: docker-ce-18.06.3.ce-3.el7.x86_64
--> Processing Dependency: libcgroup for package: docker-ce-18.06.3.ce-3.el7.x86_64
--> Processing Dependency: libltdl.so.7()(64bit) for package: docker-ce-18.06.3.ce-3.el7.x86_64
--> Processing Dependency: libseccomp.so.2()(64bit) for package: docker-ce-18.06.3.ce-3.el7.x86_64
--> Running transaction check
---> Package container-selinux.noarch 2:2.107-3.el7 will be installed
--> Processing Dependency: policycoreutils-python for package: 2:container-selinux-2.107-3.el7.noarch
---> Package libcgroup.x86_64 0:0.41-21.el7 will be installed
---> Package libseccomp.x86_64 0:2.3.1-3.el7 will be installed
---> Package libtool-ltdl.x86_64 0:2.4.2-22.el7_3 will be installed
--> Running transaction check
---> Package policycoreutils-python.x86_64 0:2.5-33.el7 will be installed
--> Processing Dependency: policycoreutils = 2.5-33.el7 for package: policycoreutils-python-2.5-33.el7.x86_64
--> Processing Dependency: setools-libs >= 3.3.8-4 for package: policycoreutils-python-2.5-33.el7.x86_64
--> Processing Dependency: libsemanage-python >= 2.5-14 for package: policycoreutils-python-2.5-33.el7.x86_64
--> Processing Dependency: audit-libs-python >= 2.1.3-4 for package: policycoreutils-python-2.5-33.el7.x86_64
--> Processing Dependency: python-IPy for package: policycoreutils-python-2.5-33.el7.x86_64
--> Processing Dependency: libqpol.so.1(VERS_1.4)(64bit) for package: policycoreutils-python-2.5-33.el7.x86_64
--> Processing Dependency: libqpol.so.1(VERS_1.2)(64bit) for package: policycoreutils-python-2.5-33.el7.x86_64
--> Processing Dependency: libapol.so.4(VERS_4.0)(64bit) for package: policycoreutils-python-2.5-33.el7.x86_64
--> Processing Dependency: checkpolicy for package: policycoreutils-python-2.5-33.el7.x86_64
--> Processing Dependency: libqpol.so.1()(64bit) for package: policycoreutils-python-2.5-33.el7.x86_64
--> Processing Dependency: libapol.so.4()(64bit) for package: policycoreutils-python-2.5-33.el7.x86_64
--> Running transaction check
---> Package audit-libs-python.x86_64 0:2.8.5-4.el7 will be installed
--> Processing Dependency: audit-libs(x86-64) = 2.8.5-4.el7 for package: audit-libs-python-2.8.5-4.el7.x86_64
---> Package checkpolicy.x86_64 0:2.5-8.el7 will be installed
---> Package libsemanage-python.x86_64 0:2.5-14.el7 will be installed
---> Package policycoreutils.x86_64 0:2.5-29.el7 will be updated
---> Package policycoreutils.x86_64 0:2.5-33.el7 will be an update
---> Package python-IPy.noarch 0:0.75-6.el7 will be installed
---> Package setools-libs.x86_64 0:3.3.8-4.el7 will be installed
--> Running transaction check
---> Package audit-libs.x86_64 0:2.8.4-4.el7 will be updated
--> Processing Dependency: audit-libs(x86-64) = 2.8.4-4.el7 for package: audit-2.8.4-4.el7.x86_64
---> Package audit-libs.x86_64 0:2.8.5-4.el7 will be an update
--> Running transaction check
---> Package audit.x86_64 0:2.8.4-4.el7 will be updated
---> Package audit.x86_64 0:2.8.5-4.el7 will be an update
--> Finished Dependency Resolution

Dependencies Resolved

==============================================================================================================================================================================================================================================================================
 Package                                                                 Arch                                                    Version                                                              Repository                                                         Size
==============================================================================================================================================================================================================================================================================
Installing:
 docker-ce                                                               x86_64                                                  18.06.3.ce-3.el7                                                     docker-ce-stable                                                   41 M
Installing for dependencies:
 audit-libs-python                                                       x86_64                                                  2.8.5-4.el7                                                          base                                                               76 k
 checkpolicy                                                             x86_64                                                  2.5-8.el7                                                            base                                                              295 k
 container-selinux                                                       noarch                                                  2:2.107-3.el7                                                        extras                                                             39 k
 libcgroup                                                               x86_64                                                  0.41-21.el7                                                          base                                                               66 k
 libseccomp                                                              x86_64                                                  2.3.1-3.el7                                                          base                                                               56 k
 libsemanage-python                                                      x86_64                                                  2.5-14.el7                                                           base                                                              113 k
 libtool-ltdl                                                            x86_64                                                  2.4.2-22.el7_3                                                       base                                                               49 k
 policycoreutils-python                                                  x86_64                                                  2.5-33.el7                                                           base                                                              457 k
 python-IPy                                                              noarch                                                  0.75-6.el7                                                           base                                                               32 k
 setools-libs                                                            x86_64                                                  3.3.8-4.el7                                                          base                                                              620 k
Updating for dependencies:
 audit                                                                   x86_64                                                  2.8.5-4.el7                                                          base                                                              256 k
 audit-libs                                                              x86_64                                                  2.8.5-4.el7                                                          base                                                              102 k
 policycoreutils                                                         x86_64                                                  2.5-33.el7                                                           base                                                              916 k

Transaction Summary
==============================================================================================================================================================================================================================================================================
Install  1 Package  (+10 Dependent packages)
Upgrade             (  3 Dependent packages)

Total download size: 44 M
Downloading packages:
Delta RPMs disabled because /usr/bin/applydeltarpm not installed.
(1/14): audit-libs-2.8.5-4.el7.x86_64.rpm                                                                                                                                                                                                              | 102 kB  00:00:00     
(2/14): audit-2.8.5-4.el7.x86_64.rpm                                                                                                                                                                                                                   | 256 kB  00:00:00     
(3/14): audit-libs-python-2.8.5-4.el7.x86_64.rpm                                                                                                                                                                                                       |  76 kB  00:00:00     
(4/14): checkpolicy-2.5-8.el7.x86_64.rpm                                                                                                                                                                                                               | 295 kB  00:00:00     
(5/14): libcgroup-0.41-21.el7.x86_64.rpm                                                                                                                                                                                                               |  66 kB  00:00:00     
(6/14): libseccomp-2.3.1-3.el7.x86_64.rpm                                                                                                                                                                                                              |  56 kB  00:00:00     
(7/14): libsemanage-python-2.5-14.el7.x86_64.rpm                                                                                                                                                                                                       | 113 kB  00:00:00     
(8/14): container-selinux-2.107-3.el7.noarch.rpm                                                                                                                                                                                                       |  39 kB  00:00:00     
(9/14): policycoreutils-2.5-33.el7.x86_64.rpm                                                                                                                                                                                                          | 916 kB  00:00:00     
(10/14): libtool-ltdl-2.4.2-22.el7_3.x86_64.rpm                                                                                                                                                                                                        |  49 kB  00:00:00     
(11/14): python-IPy-0.75-6.el7.noarch.rpm                                                                                                                                                                                                              |  32 kB  00:00:00     
(12/14): policycoreutils-python-2.5-33.el7.x86_64.rpm                                                                                                                                                                                                  | 457 kB  00:00:00     
(13/14): setools-libs-3.3.8-4.el7.x86_64.rpm                                                                                                                                                                                                           | 620 kB  00:00:00     
warning: /var/cache/yum/x86_64/7/docker-ce-stable/packages/docker-ce-18.06.3.ce-3.el7.x86_64.rpm: Header V4 RSA/SHA512 Signature, key ID 621e9f35: NOKEY================================================================================-   ] 6.5 MB/s |  43 MB  00:00:00 ETA 
Public key for docker-ce-18.06.3.ce-3.el7.x86_64.rpm is not installed
(14/14): docker-ce-18.06.3.ce-3.el7.x86_64.rpm                                                                                                                                                                                                         |  41 MB  00:00:04     
------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
Total                                                                                                                                                                                                                                         8.7 MB/s |  44 MB  00:00:05     
Retrieving key from https://mirrors.aliyun.com/docker-ce/linux/centos/gpg
Importing GPG key 0x621E9F35:
 Userid     : "Docker Release (CE rpm) <docker@docker.com>"
 Fingerprint: 060a 61c5 1b55 8a7f 742b 77aa c52f eb6b 621e 9f35
 From       : https://mirrors.aliyun.com/docker-ce/linux/centos/gpg
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Updating   : audit-libs-2.8.5-4.el7.x86_64                                                                                                                                                                                                                             1/17 
/sbin/ldconfig: /lib64/libnvidia-container.so.1 is not a symbolic link

  Updating   : policycoreutils-2.5-33.el7.x86_64                                                                                                                                                                                                                         2/17 
  Installing : libcgroup-0.41-21.el7.x86_64                                                                                                                                                                                                                              3/17 
/sbin/ldconfig: /lib64/libnvidia-container.so.1 is not a symbolic link

  Installing : audit-libs-python-2.8.5-4.el7.x86_64                                                                                                                                                                                                                      4/17 
  Installing : setools-libs-3.3.8-4.el7.x86_64                                                                                                                                                                                                                           5/17 
/sbin/ldconfig: /lib64/libnvidia-container.so.1 is not a symbolic link

  Installing : checkpolicy-2.5-8.el7.x86_64                                                                                                                                                                                                                              6/17 
  Installing : libtool-ltdl-2.4.2-22.el7_3.x86_64                                                                                                                                                                                                                        7/17 
/sbin/ldconfig: /lib64/libnvidia-container.so.1 is not a symbolic link

  Installing : libseccomp-2.3.1-3.el7.x86_64                                                                                                                                                                                                                             8/17 
/sbin/ldconfig: /lib64/libnvidia-container.so.1 is not a symbolic link

  Installing : libsemanage-python-2.5-14.el7.x86_64                                                                                                                                                                                                                      9/17 
  Installing : python-IPy-0.75-6.el7.noarch                                                                                                                                                                                                                             10/17 
  Installing : policycoreutils-python-2.5-33.el7.x86_64                                                                                                                                                                                                                 11/17 
  Installing : 2:container-selinux-2.107-3.el7.noarch                                                                                                                                                                                                                   12/17 
  Installing : docker-ce-18.06.3.ce-3.el7.x86_64                                                                                                                                                                                                                        13/17 
  Updating   : audit-2.8.5-4.el7.x86_64                                                                                                                                                                                                                                 14/17 
  Cleanup    : policycoreutils-2.5-29.el7.x86_64                                                                                                                                                                                                                        15/17 
  Cleanup    : audit-2.8.4-4.el7.x86_64                                                                                                                                                                                                                                 16/17 
  Cleanup    : audit-libs-2.8.4-4.el7.x86_64                                                                                                                                                                                                                            17/17 
/sbin/ldconfig: /lib64/libnvidia-container.so.1 is not a symbolic link

  Verifying  : audit-libs-2.8.5-4.el7.x86_64                                                                                                                                                                                                                             1/17 
  Verifying  : policycoreutils-python-2.5-33.el7.x86_64                                                                                                                                                                                                                  2/17 
  Verifying  : audit-2.8.5-4.el7.x86_64                                                                                                                                                                                                                                  3/17 
  Verifying  : python-IPy-0.75-6.el7.noarch                                                                                                                                                                                                                              4/17 
  Verifying  : audit-libs-python-2.8.5-4.el7.x86_64                                                                                                                                                                                                                      5/17 
  Verifying  : libsemanage-python-2.5-14.el7.x86_64                                                                                                                                                                                                                      6/17 
  Verifying  : 2:container-selinux-2.107-3.el7.noarch                                                                                                                                                                                                                    7/17 
  Verifying  : libseccomp-2.3.1-3.el7.x86_64                                                                                                                                                                                                                             8/17 
  Verifying  : libtool-ltdl-2.4.2-22.el7_3.x86_64                                                                                                                                                                                                                        9/17 
  Verifying  : checkpolicy-2.5-8.el7.x86_64                                                                                                                                                                                                                             10/17 
  Verifying  : policycoreutils-2.5-33.el7.x86_64                                                                                                                                                                                                                        11/17 
  Verifying  : docker-ce-18.06.3.ce-3.el7.x86_64                                                                                                                                                                                                                        12/17 
  Verifying  : setools-libs-3.3.8-4.el7.x86_64                                                                                                                                                                                                                          13/17 
  Verifying  : libcgroup-0.41-21.el7.x86_64                                                                                                                                                                                                                             14/17 
  Verifying  : policycoreutils-2.5-29.el7.x86_64                                                                                                                                                                                                                        15/17 
  Verifying  : audit-2.8.4-4.el7.x86_64                                                                                                                                                                                                                                 16/17 
  Verifying  : audit-libs-2.8.4-4.el7.x86_64                                                                                                                                                                                                                            17/17 

Installed:
  docker-ce.x86_64 0:18.06.3.ce-3.el7                                                                                                                                                                                                                                         

Dependency Installed:
  audit-libs-python.x86_64 0:2.8.5-4.el7       checkpolicy.x86_64 0:2.5-8.el7   container-selinux.noarch 2:2.107-3.el7   libcgroup.x86_64 0:0.41-21.el7   libseccomp.x86_64 0:2.3.1-3.el7   libsemanage-python.x86_64 0:2.5-14.el7   libtool-ltdl.x86_64 0:2.4.2-22.el7_3  
  policycoreutils-python.x86_64 0:2.5-33.el7   python-IPy.noarch 0:0.75-6.el7   setools-libs.x86_64 0:3.3.8-4.el7       

Dependency Updated:
  audit.x86_64 0:2.8.5-4.el7                                                           audit-libs.x86_64 0:2.8.5-4.el7                                                           policycoreutils.x86_64 0:2.5-33.el7                                                          

Complete!
[root@docker201.yinzhengjie.org.cn ~]# 
```

\[root@docker201.yinzhengjie.org.cn ~\]# yum -y install docker-ce-18.06.3.ce-3.el7　　　　　　　　　　　　　　　　　　　　　　　　#安装docker时指明版本号

![](https://img2018.cnblogs.com/i-beta/795254/202001/795254-20200113120800026-2091179667.png)

**8>.docker镜像加速配置**

```
　　此过程相对较简单，和Ubuntu配置镜像加速几乎雷同，可参考我的笔记:"https://www.cnblogs.com/yinzhengjie/p/12182645.html"
```

**二.docker存储引擎**

**1>.官方推荐使用overlay2存储引擎**

```
　　目前docker默认的存储引擎为overlay2，需要磁盘分区支持d-type文件分层功能，因此需要系统磁盘的额外支持。

　　官方文档关于docker存储引擎的选择说明:https://docs.docker.com/storage/storagedriver/select-storage-driver/
```

![](https://img2018.cnblogs.com/i-beta/795254/202001/795254-20200113122323571-1340622746.png)

**2>.官网提示xfs文件系统格式需要添加"ftype=1"参数，否则在启动容器的时候会报错哟**

```
　　docker官方推荐首选存储引擎为overlay2其次为devicemapper，但是devicemapper存在空间方便的一些限制，虽然开源通过后配置解决但官方依然推荐使用overlay2。　　如果docker数据目录是一块单独的磁盘分区而且是xfs格式的，那么需要在格式化的时候加上"-n ftype=1",否则后期在启动容器的时候会报错不支持(" ... without d-type support."),这一点官网也有声明，如下图所示。
```

![](https://img2018.cnblogs.com/i-beta/795254/202001/795254-20200113123144520-1256732047.png)

**3>.centos 7.2安装docker默认的存储引擎是"devicemapper"(生产环境建议使用较新版本的CentOS 7.6发行版本)**

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
[root@computing121.yinzhengjie.org.cn ~]# docker info
Client:
 Debug Mode: false

Server:
 Containers: 0
  Running: 0
  Paused: 0
  Stopped: 0
 Images: 0
 Server Version: 19.03.5
 Storage Driver: devicemapper
  Pool Name: docker-253:0-350832-pool
  Pool Blocksize: 65.54kB
  Base Device Size: 10.74GB
  Backing Filesystem: xfs
  Udev Sync Supported: true
  Data file: /dev/loop0
  Metadata file: /dev/loop1
  Data loop file: /var/lib/docker/devicemapper/devicemapper/data
  Metadata loop file: /var/lib/docker/devicemapper/devicemapper/metadata
  Data Space Used: 11.8MB
  Data Space Total: 107.4GB
  Data Space Available: 107.4GB
  Metadata Space Used: 581.6kB
  Metadata Space Total: 2.147GB
  Metadata Space Available: 2.147GB
  Thin Pool Minimum Free Space: 10.74GB
  Deferred Removal Enabled: true
  Deferred Deletion Enabled: true
  Deferred Deleted Device Count: 0
  Library Version: 1.02.158-RHEL7 (2019-05-13)
 Logging Driver: json-file
 Cgroup Driver: cgroupfs
 Plugins:
  Volume: local
  Network: bridge host ipvlan macvlan null overlay
  Log: awslogs fluentd gcplogs gelf journald json-file local logentries splunk syslog
 Swarm: inactive
 Runtimes: runc
 Default Runtime: runc
 Init Binary: docker-init
 containerd version: b34a5c8af56e510852c35414db4c1f4fa6172339
 runc version: 3e425f80a8c931f88e6d94a8c831b9d5aa481657
 init version: fec3683
 Security Options:
  seccomp
   Profile: default
 Kernel Version: 3.10.0-327.el7.x86_64
 Operating System: CentOS Linux 7 (Core)
 OSType: linux
 Architecture: x86_64
 CPUs: 2
 Total Memory: 3.688GiB
 Name: computing121.yinzhengjie.org.cn
 ID: TFQX:FEPM:N6C5:SDU2:VJUR:6MHF:LLKV:VGOM:RYQQ:YAGZ:RETU:QWBL
 Docker Root Dir: /var/lib/docker
 Debug Mode: false
 Registry: https://index.docker.io/v1/
 Labels:
 Experimental: false
 Insecure Registries:
  127.0.0.0/8
 Live Restore Enabled: false

WARNING: bridge-nf-call-iptables is disabled
WARNING: bridge-nf-call-ip6tables is disabled
WARNING: the devicemapper storage-driver is deprecated, and will be removed in a future release.
WARNING: devicemapper: usage of loopback devices is strongly discouraged for production use.
         Use `--storage-opt dm.thinpooldev` to specify a custom block storage device.
[root@computing121.yinzhengjie.org.cn ~]# 
[root@computing121.yinzhengjie.org.cn ~]# 
```

使用CentOS7.2安装的默认存储引擎是devicemapper。(点击这里查看详情)

![](https://img2018.cnblogs.com/i-beta/795254/202001/795254-20200113124912683-1916812305.png)

****三.**docker服务进程关系******

****1>.containerd进程关系****

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@docker201.yinzhengjie.org.cn ~]# cat /etc/redhat-release 
CentOS Linux release 7.6.1810 (Core) 
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# uname -r
3.10.0-957.el7.x86_64
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# uname -m
x86_64
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# docker -v
Docker version 18.06.3-ce, build d7080c1
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# pstree -p 1 
systemd(1)─┬─NetworkManager(4588)─┬─{NetworkManager}(4597)
           │                      └─{NetworkManager}(4602)
           ├─agetty(4581)
           ├─auditd(18993)───{auditd}(18994)
           ├─crond(4574)
           ├─dbus-daemon(4564)───{dbus-daemon}(4565)
           ├─dockerd(19257)─┬─docker-containe(19264)─┬─docker-containe(19538)─┬─nginx(19555)───nginx(19590)
           │                │                        │                        ├─{docker-containe}(19539)
           │                │                        │                        ├─{docker-containe}(19540)
           │                │                        │                        ├─{docker-containe}(19541)
           │                │                        │                        ├─{docker-containe}(19542)
           │                │                        │                        ├─{docker-containe}(19543)
           │                │                        │                        ├─{docker-containe}(19544)
           │                │                        │                        ├─{docker-containe}(19546)
           │                │                        │                        ├─{docker-containe}(19547)
           │                │                        │                        └─{docker-containe}(19678)
           │                │                        ├─docker-containe(19616)─┬─nginx(19634)───nginx(19669)
           │                │                        │                        ├─{docker-containe}(19617)
           │                │                        │                        ├─{docker-containe}(19618)
           │                │                        │                        ├─{docker-containe}(19619)
           │                │                        │                        ├─{docker-containe}(19620)
           │                │                        │                        ├─{docker-containe}(19621)
           │                │                        │                        ├─{docker-containe}(19622)
           │                │                        │                        ├─{docker-containe}(19624)
           │                │                        │                        ├─{docker-containe}(19626)
           │                │                        │                        └─{docker-containe}(19679)
           │                │                        ├─{docker-containe}(19266)
           │                │                        ├─{docker-containe}(19267)
           │                │                        ├─{docker-containe}(19268)
           │                │                        ├─{docker-containe}(19269)
           │                │                        ├─{docker-containe}(19270)
           │                │                        ├─{docker-containe}(19271)
           │                │                        ├─{docker-containe}(19272)
           │                │                        ├─{docker-containe}(19279)
           │                │                        ├─{docker-containe}(19292)
           │                │                        ├─{docker-containe}(19335)
           │                │                        ├─{docker-containe}(19632)
           │                │                        └─{docker-containe}(19686)
           │                ├─docker-proxy(19532)─┬─{docker-proxy}(19533)
           │                │                     ├─{docker-proxy}(19534)
           │                │                     ├─{docker-proxy}(19535)
           │                │                     ├─{docker-proxy}(19536)
           │                │                     └─{docker-proxy}(19537)
           │                ├─docker-proxy(19610)─┬─{docker-proxy}(19611)
           │                │                     ├─{docker-proxy}(19612)
           │                │                     ├─{docker-proxy}(19613)
           │                │                     ├─{docker-proxy}(19614)
           │                │                     └─{docker-proxy}(19615)
           │                ├─{dockerd}(19258)
           │                ├─{dockerd}(19259)
           │                ├─{dockerd}(19260)
           │                ├─{dockerd}(19261)
           │                ├─{dockerd}(19262)
           │                ├─{dockerd}(19263)
           │                ├─{dockerd}(19265)
           │                ├─{dockerd}(19274)
           │                ├─{dockerd}(19275)
           │                ├─{dockerd}(19310)
           │                └─{dockerd}(19443)
           ├─firewalld(4584)───{firewalld}(4737)
           ├─irqbalance(4562)
           ├─lvmetad(18728)
           ├─master(5179)─┬─pickup(18610)
           │              └─qmgr(5186)
           ├─polkitd(4563)─┬─{polkitd}(4569)
           │               ├─{polkitd}(4570)
           │               ├─{polkitd}(4573)
           │               ├─{polkitd}(4577)
           │               ├─{polkitd}(4579)
           │               └─{polkitd}(4582)
           ├─pouchd(18422)─┬─containerd(18429)─┬─{containerd}(18430)
           │               │                   ├─{containerd}(18431)
           │               │                   ├─{containerd}(18432)
           │               │                   ├─{containerd}(18433)
           │               │                   ├─{containerd}(18434)
           │               │                   ├─{containerd}(18437)
           │               │                   ├─{containerd}(18438)
           │               │                   ├─{containerd}(18439)
           │               │                   └─{containerd}(18440)
           │               ├─{pouchd}(18423)
           │               ├─{pouchd}(18424)
           │               ├─{pouchd}(18425)
           │               ├─{pouchd}(18426)
           │               ├─{pouchd}(18427)
           │               ├─{pouchd}(18428)
           │               ├─{pouchd}(18436)
           │               ├─{pouchd}(18441)
           │               ├─{pouchd}(18442)
           │               └─{pouchd}(18498)
           ├─rsyslogd(4913)─┬─{rsyslogd}(4917)
           │                └─{rsyslogd}(4920)
           ├─sshd(4910)─┬─sshd(5423)───bash(5427)
           │            └─sshd(5481)───bash(5485)───pstree(19700)
           ├─systemd-journal(4323)
           ├─systemd-logind(4567)
           ├─systemd-udevd(4351)
           └─tuned(4911)─┬─{tuned}(5325)
                         ├─{tuned}(5326)
                         ├─{tuned}(5327)
                         └─{tuned}(5345)
[root@docker201.yinzhengjie.org.cn ~]# 
```

Docker version 18.06.3-ce进程关系

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
root@docker101:~# cat /etc/issue
Ubuntu 18.04.3 LTS \n \l

root@docker101:~# 
root@docker101:~# uname -r
4.15.0-74-generic
root@docker101:~# 
root@docker101:~# uname -m
x86_64
root@docker101:~# 
root@docker101:~# docker -v
Docker version 19.03.5, build 633a0ea838
root@docker101:~# 
root@docker101:~# pstree -p 1
systemd(1)─┬─VGAuthService(752)
           ├─accounts-daemon(1017)─┬─{accounts-daemon}(1037)
           │                       └─{accounts-daemon}(1088)
           ├─agetty(1108)
           ├─atd(1012)
           ├─containerd(4451)─┬─containerd-shim(6553)─┬─bash(6585)
           │                  │                       ├─{containerd-shim}(6555)
           │                  │                       ├─{containerd-shim}(6556)
           │                  │                       ├─{containerd-shim}(6557)
           │                  │                       ├─{containerd-shim}(6558)
           │                  │                       ├─{containerd-shim}(6559)
           │                  │                       ├─{containerd-shim}(6560)
           │                  │                       ├─{containerd-shim}(6561)
           │                  │                       ├─{containerd-shim}(6562)
           │                  │                       ├─{containerd-shim}(6616)
           │                  │                       └─{containerd-shim}(7481)
           │                  ├─containerd-shim(11521)─┬─nginx(11545)───nginx(11595)
           │                  │                        ├─{containerd-shim}(11522)
           │                  │                        ├─{containerd-shim}(11523)
           │                  │                        ├─{containerd-shim}(11524)
           │                  │                        ├─{containerd-shim}(11525)
           │                  │                        ├─{containerd-shim}(11526)
           │                  │                        ├─{containerd-shim}(11527)
           │                  │                        ├─{containerd-shim}(11528)
           │                  │                        ├─{containerd-shim}(11530)
           │                  │                        ├─{containerd-shim}(11582)
           │                  │                        └─{containerd-shim}(11872)
           │                  ├─{containerd}(4473)
           │                  ├─{containerd}(4474)
           │                  ├─{containerd}(4475)
           │                  ├─{containerd}(4476)
           │                  ├─{containerd}(4487)
           │                  ├─{containerd}(4490)
           │                  ├─{containerd}(4492)
           │                  ├─{containerd}(4511)
           │                  ├─{containerd}(4512)
           │                  ├─{containerd}(6569)
           │                  ├─{containerd}(6810)
           │                  └─{containerd}(8398)
           ├─cron(979)
           ├─dbus-daemon(985)
           ├─dirmngr(14085)───{dirmngr}(14086)
           ├─dnsmasq(13262)
           ├─dockerd(6171)─┬─{dockerd}(6184)
           │               ├─{dockerd}(6185)
           │               ├─{dockerd}(6186)
           │               ├─{dockerd}(6187)
           │               ├─{dockerd}(6203)
           │               ├─{dockerd}(6205)
           │               ├─{dockerd}(6206)
           │               ├─{dockerd}(6214)
           │               ├─{dockerd}(6269)
           │               ├─{dockerd}(6365)
           │               └─{dockerd}(6373)
           ├─irqbalance(1016)───{irqbalance}(1036)
           ├─lvmetad(520)
           ├─lxcfs(1032)─┬─{lxcfs}(1090)
           │             ├─{lxcfs}(1091)
           │             └─{lxcfs}(1733)
           ├─networkd-dispat(1010)───{networkd-dispat}(1175)
           ├─polkitd(1109)─┬─{polkitd}(1136)
           │               └─{polkitd}(1138)
           ├─rsyslogd(1024)─┬─{rsyslogd}(1042)
           │                ├─{rsyslogd}(1043)
           │                └─{rsyslogd}(1044)
           ├─snapd(14380)─┬─{snapd}(14404)
           │              ├─{snapd}(14405)
           │              ├─{snapd}(14406)
           │              ├─{snapd}(14407)
           │              ├─{snapd}(14410)
           │              ├─{snapd}(14412)
           │              ├─{snapd}(14449)
           │              ├─{snapd}(14450)
           │              ├─{snapd}(14452)
           │              ├─{snapd}(14453)
           │              ├─{snapd}(14454)
           │              └─{snapd}(14455)
           ├─sshd(1066)───sshd(14598)───bash(14754)───pstree(15040)
           ├─systemd(1338)───(sd-pam)(1352)
           ├─systemd-journal(511)
           ├─systemd-logind(1009)
           ├─systemd-network(874)
           ├─systemd-resolve(899)
           ├─systemd-timesyn(751)───{systemd-timesyn}(807)
           ├─systemd-udevd(536)
           ├─thermald(1031)───{thermald}(1097)
           ├─unattended-upgr(1087)───{unattended-upgr}(1180)
           └─vmtoolsd(827)
root@docker101:~# 
root@docker101:~# 
```

Docker version 19.03.5进程关系

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
dockerd:
　　被客户端直接访问，其父进程为宿主机的systemd守护进程。

docker-proxy:
　　实现容器通信,其父进程为dockerd。

containerd:
　　被dockerd进程调用以实现runc交互。

containerd-shim:　　真正运行容器的载体，其父进程为containerd。
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

****2>.容器的创建于管理过程****

```
创建容器通信过程如下:
　　1>.dockerd通过grpc和containerd模块通信，dockerd由libcontainerd负责和containerd进行交换，dockerd和containerd通信文件"/run/containerd/containerd.sock";　　2>.containerd在dockerd启动时被启动,然后containerd启动grpc请求监听，containerd处理grpc请求，根据请求做相应动作;　　3>.若是start或是exec容器,containerd拉起一个container-shim,并进行相应的操作。　　4>.container-shim被拉起后，start/exec/create拉起runc进程，通过exit/control文件和containerd通信,通过父子进程关系和SIGCHLD监控容器中进程状态;　　5>.在整个生命周期中，containerd通过epoll监控容器文件，监控容器事件。
```

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@docker201.yinzhengjie.org.cn ~]# containerd-shim -h
Usage of containerd-shim:
  -address string
        grpc address back to main containerd
  -containerd-binary containerd publish
        path to containerd binary (used for containerd publish) (default "containerd")
  -criu string
        path to criu binary
  -debug
        enable debug output in logs
  -namespace string
        namespace that owns the shim
  -runtime-root string
        root directory for the runtime (default "/run/containerd/runc")
  -socket string
        abstract socket path to serve
  -systemd-cgroup
        set runtime to use systemd-cgroup
  -workdir string
        path used to storge large temporary data
[root@docker201.yinzhengjie.org.cn ~]# 
```

\[root@docker201.yinzhengjie.org.cn ~\]# containerd-shim -h

![](https://img2018.cnblogs.com/i-beta/795254/202001/795254-20200113132543370-1517878606.png)

****3>.grpc简介****

```
　　gRPC是Google开发的一款高性能，开源和通信的RPC框架，支持众多语言客户端。
```

![](https://img2018.cnblogs.com/i-beta/795254/202001/795254-20200113132636881-103220649.png)

当你的才华还撑不起你的野心的时候，你就应该静下心来学习。当你的能力还驾驭不了你的目标的时候，你就应该沉下心来历练。问问自己，想要怎样的人生。 欢迎加入基础架构自动化运维：598432640，大数据SRE进阶之路：959042252，DevOps进阶之路：526991186