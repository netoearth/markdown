　　　　　　　　　　　　　　　　**私有仓库GitLab快速入门篇**

　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　**作者：尹正杰**

**版权声明：原创作品，谢绝转载！否则将追究法律责任。**

　　**安装文档请参考官网：https://about.gitlab.com/installation/#centos-7。当然根据本篇博客的步骤走也是可以成功部署GitLab的哟。**

**一.GitLab简介**

**1>.什么是GitLab**

　　GitLab 是一个用于仓库管理系统的开源项目。使用Git作为代码管理工具，并在此基础上搭建起来的web服务。可通过Web界面进行访问公开的或者私人项目。它拥有与Github类似的功能，能够浏览源代码，管理缺陷和注释。可以管理团队对仓库的访问，它非常易于浏览提交过的版本并提供一个文件历史库。团队成员可以利用内置的简单聊天程序(Wall)进行交流。它还提供一个代码片段收集功能可以轻松实现代码复用。

 **2>.常用的网站**

　　官网：[https://about.gitlab.com/](https://about.gitlab.com/)

　　国内镜像：[https://mirrors.tuna.tsinghua.edu.cn/gitlab-ce/yum/](https://mirrors.tuna.tsinghua.edu.cn/gitlab-ce/yum/)

 **3>.安装GitLab的操纵系统环境**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@yinzhengjie ~]# cat /etc/redhat-release 
CentOS Linux release 7.2.1511 (Core) 
[root@yinzhengjie ~]# 
[root@yinzhengjie ~]# free -m
              total        used        free      shared  buff/cache   available
Mem:           7808         258        7338           8         211        7342
Swap:          2047           0        2047
[root@yinzhengjie ~]# 
[root@yinzhengjie ~]# uname -r
3.10.0-327.el7.x86_64
[root@yinzhengjie ~]# 
[root@yinzhengjie ~]# uname -m
x86_64
[root@yinzhengjie ~]# 
[root@yinzhengjie ~]# ifconfig  | grep broadcast | awk '{print $2}'
172.30.1.101
[root@yinzhengjie ~]# 
[root@yinzhengjie ~]# 
[root@yinzhengjie ~]# cat /etc/hosts | grep 172.30.1.101
172.30.1.101 www.yinzhengjie.org.cn
[root@yinzhengjie ~]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

**二.GitLab部署**

**1>.安装GitLab依赖包**

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@yinzhengjie download]# yum install -y curl policycoreutils-python openssh-server openssh-clients
Loaded plugins: fastestmirror
base                                                                                                                  | 3.6 kB  00:00:00     
elasticsearch-2.x                                                                                                     | 2.9 kB  00:00:00     
extras                                                                                                                | 3.4 kB  00:00:00     
mysql-connectors-community                                                                                            | 2.5 kB  00:00:00     
mysql-tools-community                                                                                                 | 2.5 kB  00:00:00     
mysql56-community                                                                                                     | 2.5 kB  00:00:00     
updates                                                                                                               | 3.4 kB  00:00:00     
Loading mirror speeds from cached hostfile
 * base: mirrors.huaweicloud.com
 * extras: mirrors.shu.edu.cn
 * updates: mirrors.huaweicloud.com
Resolving Dependencies
--> Running transaction check
---> Package curl.x86_64 0:7.29.0-25.el7.centos will be updated
---> Package curl.x86_64 0:7.29.0-46.el7 will be an update
--> Processing Dependency: libcurl = 7.29.0-46.el7 for package: curl-7.29.0-46.el7.x86_64
---> Package openssh-server.x86_64 0:6.6.1p1-22.el7 will be updated
---> Package openssh-server.x86_64 0:7.4p1-16.el7 will be an update
--> Processing Dependency: openssh = 7.4p1-16.el7 for package: openssh-server-7.4p1-16.el7.x86_64
---> Package policycoreutils-python.x86_64 0:2.5-22.el7 will be installed
--> Processing Dependency: policycoreutils = 2.5-22.el7 for package: policycoreutils-python-2.5-22.el7.x86_64
--> Processing Dependency: setools-libs >= 3.3.8-2 for package: policycoreutils-python-2.5-22.el7.x86_64
--> Processing Dependency: libsemanage-python >= 2.5-9 for package: policycoreutils-python-2.5-22.el7.x86_64
--> Processing Dependency: audit-libs-python >= 2.1.3-4 for package: policycoreutils-python-2.5-22.el7.x86_64
--> Processing Dependency: python-IPy for package: policycoreutils-python-2.5-22.el7.x86_64
--> Processing Dependency: libselinux-python for package: policycoreutils-python-2.5-22.el7.x86_64
--> Processing Dependency: libqpol.so.1(VERS_1.4)(64bit) for package: policycoreutils-python-2.5-22.el7.x86_64
--> Processing Dependency: libqpol.so.1(VERS_1.2)(64bit) for package: policycoreutils-python-2.5-22.el7.x86_64
--> Processing Dependency: libcgroup for package: policycoreutils-python-2.5-22.el7.x86_64
--> Processing Dependency: libapol.so.4(VERS_4.0)(64bit) for package: policycoreutils-python-2.5-22.el7.x86_64
--> Processing Dependency: checkpolicy for package: policycoreutils-python-2.5-22.el7.x86_64
--> Processing Dependency: libqpol.so.1()(64bit) for package: policycoreutils-python-2.5-22.el7.x86_64
--> Processing Dependency: libapol.so.4()(64bit) for package: policycoreutils-python-2.5-22.el7.x86_64
--> Running transaction check
---> Package audit-libs-python.x86_64 0:2.8.1-3.el7_5.1 will be installed
--> Processing Dependency: audit-libs(x86-64) = 2.8.1-3.el7_5.1 for package: audit-libs-python-2.8.1-3.el7_5.1.x86_64
---> Package checkpolicy.x86_64 0:2.5-6.el7 will be installed
---> Package libcgroup.x86_64 0:0.41-15.el7 will be installed
---> Package libcurl.x86_64 0:7.29.0-25.el7.centos will be updated
---> Package libcurl.x86_64 0:7.29.0-46.el7 will be an update
---> Package libselinux-python.x86_64 0:2.5-12.el7 will be installed
---> Package libsemanage-python.x86_64 0:2.5-11.el7 will be installed
--> Processing Dependency: libsemanage = 2.5-11.el7 for package: libsemanage-python-2.5-11.el7.x86_64
--> Processing Dependency: libsemanage.so.1(LIBSEMANAGE_1.1)(64bit) for package: libsemanage-python-2.5-11.el7.x86_64
---> Package openssh.x86_64 0:6.6.1p1-22.el7 will be updated
--> Processing Dependency: openssh = 6.6.1p1-22.el7 for package: openssh-clients-6.6.1p1-22.el7.x86_64
---> Package openssh.x86_64 0:7.4p1-16.el7 will be an update
---> Package policycoreutils.x86_64 0:2.2.5-20.el7 will be updated
---> Package policycoreutils.x86_64 0:2.5-22.el7 will be an update
---> Package python-IPy.noarch 0:0.75-6.el7 will be installed
---> Package setools-libs.x86_64 0:3.3.8-2.el7 will be installed
--> Running transaction check
---> Package audit-libs.x86_64 0:2.4.1-5.el7 will be updated
--> Processing Dependency: audit-libs = 2.4.1-5.el7 for package: audit-2.4.1-5.el7.x86_64
---> Package audit-libs.x86_64 0:2.8.1-3.el7_5.1 will be an update
---> Package libsemanage.x86_64 0:2.1.10-18.el7 will be updated
---> Package libsemanage.x86_64 0:2.5-11.el7 will be an update
---> Package openssh-clients.x86_64 0:6.6.1p1-22.el7 will be updated
---> Package openssh-clients.x86_64 0:7.4p1-16.el7 will be an update
--> Running transaction check
---> Package audit.x86_64 0:2.4.1-5.el7 will be updated
---> Package audit.x86_64 0:2.8.1-3.el7_5.1 will be an update
--> Processing Conflict: libsemanage-2.5-11.el7.x86_64 conflicts selinux-policy-base < 3.13.1-66
--> Restarting Dependency Resolution with new changes.
--> Running transaction check
---> Package selinux-policy-targeted.noarch 0:3.13.1-60.el7 will be updated
---> Package selinux-policy-targeted.noarch 0:3.13.1-192.el7_5.6 will be an update
--> Processing Dependency: selinux-policy = 3.13.1-192.el7_5.6 for package: selinux-policy-targeted-3.13.1-192.el7_5.6.noarch
--> Processing Dependency: selinux-policy = 3.13.1-192.el7_5.6 for package: selinux-policy-targeted-3.13.1-192.el7_5.6.noarch
--> Running transaction check
---> Package selinux-policy.noarch 0:3.13.1-60.el7 will be updated
---> Package selinux-policy.noarch 0:3.13.1-192.el7_5.6 will be an update
--> Finished Dependency Resolution

Dependencies Resolved

=============================================================================================================================================
 Package                                    Arch                      Version                               Repository                  Size
=============================================================================================================================================
Installing:
 policycoreutils-python                     x86_64                    2.5-22.el7                            base                       454 k
Updating:
 curl                                       x86_64                    7.29.0-46.el7                         base                       268 k
 openssh-server                             x86_64                    7.4p1-16.el7                          base                       458 k
 selinux-policy-targeted                    noarch                    3.13.1-192.el7_5.6                    updates                    6.6 M
Installing for dependencies:
 audit-libs-python                          x86_64                    2.8.1-3.el7_5.1                       updates                     75 k
 checkpolicy                                x86_64                    2.5-6.el7                             base                       294 k
 libcgroup                                  x86_64                    0.41-15.el7                           base                        65 k
 libselinux-python                          x86_64                    2.5-12.el7                            base                       235 k
 libsemanage-python                         x86_64                    2.5-11.el7                            base                       112 k
 python-IPy                                 noarch                    0.75-6.el7                            base                        32 k
 setools-libs                               x86_64                    3.3.8-2.el7                           base                       619 k
Updating for dependencies:
 audit                                      x86_64                    2.8.1-3.el7_5.1                       updates                    247 k
 audit-libs                                 x86_64                    2.8.1-3.el7_5.1                       updates                     99 k
 libcurl                                    x86_64                    7.29.0-46.el7                         base                       220 k
 libsemanage                                x86_64                    2.5-11.el7                            base                       150 k
 openssh                                    x86_64                    7.4p1-16.el7                          base                       510 k
 openssh-clients                            x86_64                    7.4p1-16.el7                          base                       655 k
 policycoreutils                            x86_64                    2.5-22.el7                            base                       867 k
 selinux-policy                             noarch                    3.13.1-192.el7_5.6                    updates                    453 k

Transaction Summary
=============================================================================================================================================
Install  1 Package  (+7 Dependent packages)
Upgrade  3 Packages (+8 Dependent packages)

Total download size: 12 M
Downloading packages:
Delta RPMs disabled because /usr/bin/applydeltarpm not installed.
(1/19): audit-2.8.1-3.el7_5.1.x86_64.rpm                                                                              | 247 kB  00:00:00     
(2/19): curl-7.29.0-46.el7.x86_64.rpm                                                                                 | 268 kB  00:00:00     
(3/19): libcurl-7.29.0-46.el7.x86_64.rpm                                                                              | 220 kB  00:00:00     
(4/19): audit-libs-2.8.1-3.el7_5.1.x86_64.rpm                                                                         |  99 kB  00:00:00     
(5/19): libselinux-python-2.5-12.el7.x86_64.rpm                                                                       | 235 kB  00:00:00     
(6/19): libsemanage-python-2.5-11.el7.x86_64.rpm                                                                      | 112 kB  00:00:00     
(7/19): libcgroup-0.41-15.el7.x86_64.rpm                                                                              |  65 kB  00:00:00     
(8/19): openssh-7.4p1-16.el7.x86_64.rpm                                                                               | 510 kB  00:00:00     
(9/19): checkpolicy-2.5-6.el7.x86_64.rpm                                                                              | 294 kB  00:00:00     
(10/19): openssh-server-7.4p1-16.el7.x86_64.rpm                                                                       | 458 kB  00:00:00     
(11/19): policycoreutils-2.5-22.el7.x86_64.rpm                                                                        | 867 kB  00:00:00     
(12/19): python-IPy-0.75-6.el7.noarch.rpm                                                                             |  32 kB  00:00:00     
(13/19): policycoreutils-python-2.5-22.el7.x86_64.rpm                                                                 | 454 kB  00:00:00     
(14/19): libsemanage-2.5-11.el7.x86_64.rpm                                                                            | 150 kB  00:00:00     
(15/19): selinux-policy-3.13.1-192.el7_5.6.noarch.rpm                                                                 | 453 kB  00:00:00     
(16/19): setools-libs-3.3.8-2.el7.x86_64.rpm                                                                          | 619 kB  00:00:00     
(17/19): openssh-clients-7.4p1-16.el7.x86_64.rpm                                                                      | 655 kB  00:00:00     
(18/19): selinux-policy-targeted-3.13.1-192.el7_5.6.noarch.rpm                                                        | 6.6 MB  00:00:03     
audit-libs-python-2.8.1-3.el7_ FAILED                                          
http://centos.ustc.edu.cn/centos/7.5.1804/updates/x86_64/Packages/audit-libs-python-2.8.1-3.el7_5.1.x86_64.rpm: [Errno 14] curl#7 - "Failed to connect to 2001:da8:d800:95::110: Network is unreachable"
Trying other mirror.
(19/19): audit-libs-python-2.8.1-3.el7_5.1.x86_64.rpm                                                                 |  75 kB  00:00:00     
---------------------------------------------------------------------------------------------------------------------------------------------
Total                                                                                                        824 kB/s |  12 MB  00:00:15     
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Updating   : audit-libs-2.8.1-3.el7_5.1.x86_64                                                                                        1/30 
  Updating   : libsemanage-2.5-11.el7.x86_64                                                                                            2/30 
  Updating   : policycoreutils-2.5-22.el7.x86_64                                                                                        3/30 
  Updating   : openssh-7.4p1-16.el7.x86_64                                                                                              4/30 
  Updating   : selinux-policy-3.13.1-192.el7_5.6.noarch                                                                                 5/30 
  Installing : libsemanage-python-2.5-11.el7.x86_64                                                                                     6/30 
  Installing : audit-libs-python-2.8.1-3.el7_5.1.x86_64                                                                                 7/30 
  Installing : checkpolicy-2.5-6.el7.x86_64                                                                                             8/30 
  Installing : libcgroup-0.41-15.el7.x86_64                                                                                             9/30 
  Updating   : libcurl-7.29.0-46.el7.x86_64                                                                                            10/30 
  Installing : python-IPy-0.75-6.el7.noarch                                                                                            11/30 
  Installing : libselinux-python-2.5-12.el7.x86_64                                                                                     12/30 
  Installing : setools-libs-3.3.8-2.el7.x86_64                                                                                         13/30 
  Installing : policycoreutils-python-2.5-22.el7.x86_64                                                                                14/30 
  Updating   : curl-7.29.0-46.el7.x86_64                                                                                               15/30 
  Updating   : selinux-policy-targeted-3.13.1-192.el7_5.6.noarch                                                                       16/30 
  Updating   : openssh-server-7.4p1-16.el7.x86_64                                                                                      17/30 
  Updating   : openssh-clients-7.4p1-16.el7.x86_64                                                                                     18/30 
  Updating   : audit-2.8.1-3.el7_5.1.x86_64                                                                                            19/30 
  Cleanup    : selinux-policy-targeted-3.13.1-60.el7.noarch                                                                            20/30 
  Cleanup    : openssh-server-6.6.1p1-22.el7.x86_64                                                                                    21/30 
  Cleanup    : selinux-policy-3.13.1-60.el7.noarch                                                                                     22/30 
  Cleanup    : policycoreutils-2.2.5-20.el7.x86_64                                                                                     23/30 
  Cleanup    : libsemanage-2.1.10-18.el7.x86_64                                                                                        24/30 
  Cleanup    : openssh-clients-6.6.1p1-22.el7.x86_64                                                                                   25/30 
  Cleanup    : openssh-6.6.1p1-22.el7.x86_64                                                                                           26/30 
  Cleanup    : audit-2.4.1-5.el7.x86_64                                                                                                27/30 
  Cleanup    : curl-7.29.0-25.el7.centos.x86_64                                                                                        28/30 
  Cleanup    : libcurl-7.29.0-25.el7.centos.x86_64                                                                                     29/30 
  Cleanup    : audit-libs-2.4.1-5.el7.x86_64                                                                                           30/30 
  Verifying  : libsemanage-python-2.5-11.el7.x86_64                                                                                     1/30 
  Verifying  : libsemanage-2.5-11.el7.x86_64                                                                                            2/30 
  Verifying  : setools-libs-3.3.8-2.el7.x86_64                                                                                          3/30 
  Verifying  : audit-libs-python-2.8.1-3.el7_5.1.x86_64                                                                                 4/30 
  Verifying  : curl-7.29.0-46.el7.x86_64                                                                                                5/30 
  Verifying  : policycoreutils-2.5-22.el7.x86_64                                                                                        6/30 
  Verifying  : audit-2.8.1-3.el7_5.1.x86_64                                                                                             7/30 
  Verifying  : openssh-server-7.4p1-16.el7.x86_64                                                                                       8/30 
  Verifying  : libselinux-python-2.5-12.el7.x86_64                                                                                      9/30 
  Verifying  : policycoreutils-python-2.5-22.el7.x86_64                                                                                10/30 
  Verifying  : python-IPy-0.75-6.el7.noarch                                                                                            11/30 
  Verifying  : selinux-policy-targeted-3.13.1-192.el7_5.6.noarch                                                                       12/30 
  Verifying  : libcurl-7.29.0-46.el7.x86_64                                                                                            13/30 
  Verifying  : libcgroup-0.41-15.el7.x86_64                                                                                            14/30 
  Verifying  : audit-libs-2.8.1-3.el7_5.1.x86_64                                                                                       15/30 
  Verifying  : openssh-7.4p1-16.el7.x86_64                                                                                             16/30 
  Verifying  : openssh-clients-7.4p1-16.el7.x86_64                                                                                     17/30 
  Verifying  : selinux-policy-3.13.1-192.el7_5.6.noarch                                                                                18/30 
  Verifying  : checkpolicy-2.5-6.el7.x86_64                                                                                            19/30 
  Verifying  : curl-7.29.0-25.el7.centos.x86_64                                                                                        20/30 
  Verifying  : openssh-6.6.1p1-22.el7.x86_64                                                                                           21/30 
  Verifying  : selinux-policy-targeted-3.13.1-60.el7.noarch                                                                            22/30 
  Verifying  : openssh-clients-6.6.1p1-22.el7.x86_64                                                                                   23/30 
  Verifying  : libsemanage-2.1.10-18.el7.x86_64                                                                                        24/30 
  Verifying  : selinux-policy-3.13.1-60.el7.noarch                                                                                     25/30 
  Verifying  : libcurl-7.29.0-25.el7.centos.x86_64                                                                                     26/30 
  Verifying  : openssh-server-6.6.1p1-22.el7.x86_64                                                                                    27/30 
  Verifying  : audit-libs-2.4.1-5.el7.x86_64                                                                                           28/30 
  Verifying  : policycoreutils-2.2.5-20.el7.x86_64                                                                                     29/30 
  Verifying  : audit-2.4.1-5.el7.x86_64                                                                                                30/30 

Installed:
  policycoreutils-python.x86_64 0:2.5-22.el7                                                                                                 

Dependency Installed:
  audit-libs-python.x86_64 0:2.8.1-3.el7_5.1          checkpolicy.x86_64 0:2.5-6.el7                  libcgroup.x86_64 0:0.41-15.el7         
  libselinux-python.x86_64 0:2.5-12.el7               libsemanage-python.x86_64 0:2.5-11.el7          python-IPy.noarch 0:0.75-6.el7         
  setools-libs.x86_64 0:3.3.8-2.el7                  

Updated:
  curl.x86_64 0:7.29.0-46.el7        openssh-server.x86_64 0:7.4p1-16.el7        selinux-policy-targeted.noarch 0:3.13.1-192.el7_5.6       

Dependency Updated:
  audit.x86_64 0:2.8.1-3.el7_5.1             audit-libs.x86_64 0:2.8.1-3.el7_5.1               libcurl.x86_64 0:7.29.0-46.el7              
  libsemanage.x86_64 0:2.5-11.el7            openssh.x86_64 0:7.4p1-16.el7                     openssh-clients.x86_64 0:7.4p1-16.el7       
  policycoreutils.x86_64 0:2.5-22.el7        selinux-policy.noarch 0:3.13.1-192.el7_5.6       

Complete!
[root@yinzhengjie download]# 
```

\[root@yinzhengjie download\]# yum install -y curl policycoreutils-python openssh-server openssh-clients

**2>.下载GitLab的rpm包**

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@yinzhengjie ~]# cd download/
[root@yinzhengjie download]# ll
total 9676
-rw-r--r-- 1 root root 9904915 Sep  3 18:10 apache-tomcat-9.0.11.tar.gz
[root@yinzhengjie download]# 
[root@yinzhengjie download]# 
[root@yinzhengjie download]# wget https://mirrors.tuna.tsinghua.edu.cn/gitlab-ce/yum/el7/gitlab-ce-10.0.6-ce.0.el7.x86_64.rpm
--2018-09-09 01:50:27--  https://mirrors.tuna.tsinghua.edu.cn/gitlab-ce/yum/el7/gitlab-ce-10.0.6-ce.0.el7.x86_64.rpm
Resolving mirrors.tuna.tsinghua.edu.cn (mirrors.tuna.tsinghua.edu.cn)... 101.6.8.193, 2402:f000:1:408:8100::1
Connecting to mirrors.tuna.tsinghua.edu.cn (mirrors.tuna.tsinghua.edu.cn)|101.6.8.193|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 359552899 (343M) [application/x-redhat-package-manager]
Saving to: ‘gitlab-ce-10.0.6-ce.0.el7.x86_64.rpm’

100%[===================================================================================================>] 359,552,899 1.39MB/s   in 4m 6s  

2018-09-09 01:54:33 (1.39 MB/s) - ‘gitlab-ce-10.0.6-ce.0.el7.x86_64.rpm’ saved [359552899/359552899]

[root@yinzhengjie download]# ll
total 360804
-rw-r--r-- 1 root root   9904915 Sep  3 18:10 apache-tomcat-9.0.11.tar.gz
-rw-r--r-- 1 root root 359552899 Nov  9  2017 gitlab-ce-10.0.6-ce.0.el7.x86_64.rpm
[root@yinzhengjie download]# 
```

\[root@yinzhengjie download\]# wget https://mirrors.tuna.tsinghua.edu.cn/gitlab-ce/yum/el7/gitlab-ce-10.0.6-ce.0.el7.x86\_64.rpm

**3>.通过yum本地安装GitLab**

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@yinzhengjie download]# yum -y localinstall gitlab-ce-10.0.6-ce.0.el7.x86_64.rpm 
Loaded plugins: fastestmirror
Examining gitlab-ce-10.0.6-ce.0.el7.x86_64.rpm: gitlab-ce-10.0.6-ce.0.el7.x86_64
Marking gitlab-ce-10.0.6-ce.0.el7.x86_64.rpm to be installed
Resolving Dependencies
--> Running transaction check
---> Package gitlab-ce.x86_64 0:10.0.6-ce.0.el7 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

=============================================================================================================================================
 Package                   Arch                   Version                            Repository                                         Size
=============================================================================================================================================
Installing:
 gitlab-ce                 x86_64                 10.0.6-ce.0.el7                    /gitlab-ce-10.0.6-ce.0.el7.x86_64                 1.0 G

Transaction Summary
=============================================================================================================================================
Install  1 Package

Total size: 1.0 G
Installed size: 1.0 G
Downloading packages:
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : gitlab-ce-10.0.6-ce.0.el7.x86_64                                                                                          1/1 
It looks like GitLab has not been configured yet; skipping the upgrade script.

       *.                  *.
      ***                 ***
     *****               *****
    .******             *******
    ********            ********
   ,,,,,,,,,***********,,,,,,,,,
  ,,,,,,,,,,,*********,,,,,,,,,,,
  .,,,,,,,,,,,*******,,,,,,,,,,,,
      ,,,,,,,,,*****,,,,,,,,,.
         ,,,,,,,****,,,,,,
            .,,,***,,,,
                ,*,.
  


     _______ __  __          __
    / ____(_) /_/ /   ____ _/ /_
   / / __/ / __/ /   / __ \`/ __ \
  / /_/ / / /_/ /___/ /_/ / /_/ /
  \____/_/\__/_____/\__,_/_.___/
  

Thank you for installing GitLab!
GitLab was unable to detect a valid hostname for your instance.
Please configure a URL for your GitLab instance by setting `external_url`
configuration in /etc/gitlab/gitlab.rb file.
Then, you can start your GitLab instance by running the following command:
  sudo gitlab-ctl reconfigure

For a comprehensive list of configuration options please see the Omnibus GitLab readme
https://gitlab.com/gitlab-org/omnibus-gitlab/blob/master/README.md

  Verifying  : gitlab-ce-10.0.6-ce.0.el7.x86_64                                                                                          1/1 

Installed:
  gitlab-ce.x86_64 0:10.0.6-ce.0.el7                                                                                                         

Complete!
[root@yinzhengjie download]# 
```

\[root@yinzhengjie download\]# yum -y localinstall gitlab-ce-10.0.6-ce.0.el7.x86\_64.rpm

**4>.修改GitLab的主配置文件**

**![](https://images2018.cnblogs.com/blog/795254/201809/795254-20180908153803074-990523007.png)**

**5>.初始化GitLab，就执行一次**

```
[root@yinzhengjie download]# gitlab-ctl reconfigure
```

**![](https://images2018.cnblogs.com/blog/795254/201809/795254-20180908141558373-995931162.png)**

**6>.管理GitLab服务的常用命令**

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@yinzhengjie download]# gitlab-ctl start
ok: run: gitaly: (pid 4100) 119s
ok: run: gitlab-monitor: (pid 4171) 118s
ok: run: gitlab-workhorse: (pid 4124) 119s
ok: run: logrotate: (pid 3855) 148s
ok: run: nginx: (pid 3837) 150s
ok: run: node-exporter: (pid 3912) 142s
ok: run: postgres-exporter: (pid 4196) 118s
ok: run: postgresql: (pid 3575) 188s
ok: run: prometheus: (pid 4180) 118s
ok: run: redis: (pid 3515) 194s
ok: run: redis-exporter: (pid 3949) 138s
ok: run: sidekiq: (pid 3735) 160s
ok: run: unicorn: (pid 3697) 162s
[root@yinzhengjie download]# 
```

启动GitLab服务（\[root@yinzhengjie download\]# gitlab-ctl start）

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@yinzhengjie download]# gitlab-ctl status
run: gitaly: (pid 4100) 123s; run: log: (pid 3751) 162s
run: gitlab-monitor: (pid 4171) 122s; run: log: (pid 3933) 144s
run: gitlab-workhorse: (pid 4124) 123s; run: log: (pid 3818) 156s
run: logrotate: (pid 3855) 152s; run: log: (pid 3854) 152s
run: nginx: (pid 3837) 154s; run: log: (pid 3836) 154s
run: node-exporter: (pid 3912) 146s; run: log: (pid 3911) 146s
run: postgres-exporter: (pid 4196) 122s; run: log: (pid 4021) 134s
run: postgresql: (pid 3575) 192s; run: log: (pid 3574) 192s
run: prometheus: (pid 4180) 122s; run: log: (pid 3999) 136s
run: redis: (pid 3515) 198s; run: log: (pid 3514) 198s
run: redis-exporter: (pid 3949) 142s; run: log: (pid 3948) 142s
run: sidekiq: (pid 3735) 164s; run: log: (pid 3734) 164s
run: unicorn: (pid 3697) 166s; run: log: (pid 3696) 166s
[root@yinzhengjie download]#
```

查看GitLab的状态（\[root@yinzhengjie download\]# gitlab-ctl status）

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@yinzhengjie download]# gitlab-ctl stop
ok: down: gitaly: 0s, normally up
ok: down: gitlab-monitor: 1s, normally up
ok: down: gitlab-workhorse: 0s, normally up
ok: down: logrotate: 0s, normally up
ok: down: nginx: 1s, normally up
ok: down: node-exporter: 0s, normally up
ok: down: postgres-exporter: 1s, normally up
ok: down: postgresql: 0s, normally up
ok: down: prometheus: 1s, normally up
ok: down: redis: 0s, normally up
ok: down: redis-exporter: 0s, normally up
ok: down: sidekiq: 0s, normally up
ok: down: unicorn: 1s, normally up
[root@yinzhengjie download]# 
[root@yinzhengjie download]# 
[root@yinzhengjie download]# 
[root@yinzhengjie download]# gitlab-ctl status
down: gitaly: 8s, normally up; run: log: (pid 3751) 690s
down: gitlab-monitor: 8s, normally up; run: log: (pid 3933) 672s
down: gitlab-workhorse: 7s, normally up; run: log: (pid 3818) 684s
down: logrotate: 7s, normally up; run: log: (pid 3854) 680s
down: nginx: 7s, normally up; run: log: (pid 3836) 682s
down: node-exporter: 6s, normally up; run: log: (pid 3911) 674s
down: postgres-exporter: 6s, normally up; run: log: (pid 4021) 662s
down: postgresql: 5s, normally up; run: log: (pid 3574) 720s
down: prometheus: 5s, normally up; run: log: (pid 3999) 664s
down: redis: 4s, normally up; run: log: (pid 3514) 726s
down: redis-exporter: 4s, normally up; run: log: (pid 3948) 670s
down: sidekiq: 3s, normally up; run: log: (pid 3734) 692s
down: unicorn: 3s, normally up; run: log: (pid 3696) 694s
[root@yinzhengjie download]# 
[root@yinzhengjie download]# 
```

停止GitLab服务（\[root@yinzhengjie download\]# gitlab-ctl stop）

**7>.通过webUI访问GitLab,设置初始密码**

![](https://images2018.cnblogs.com/blog/795254/201809/795254-20180908142225218-2097527724.png)

**8>.登录GitLab**

![](https://images2018.cnblogs.com/blog/795254/201809/795254-20180908142356591-40828964.png)

**9>.登录成功后会有以下界面**

![](https://images2018.cnblogs.com/blog/795254/201809/795254-20180908142444608-907370439.png)

**10>.GitLab的安装和日志存放位置**

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@yinzhengjie download]# ll /opt/gitlab/
total 1672
drwxr-xr-x  2 root root     103 Sep  9 02:02 bin
-rw-r--r--  1 root root  144144 Nov  8  2017 dependency_licenses.json
drwxr-xr-x 20 root root    4096 Sep  9 02:13 embedded
drwxr-xr-x  6 root root    4096 Sep  9 02:14 etc
drwxr-xr-x  2 root root    4096 Sep  9 02:14 init
-rw-r--r--  1 root root 1499217 Nov  8  2017 LICENSE
drwxr-xr-x  2 root root    4096 Sep  9 02:02 LICENSES
drwxr-xr-x  2 root root    4096 Sep  9 02:14 service
drwxr-xr-x 15 root root    4096 Sep  9 02:14 sv
drwxr-xr-x  3 root root      20 Sep  9 02:14 var
-rw-r--r--  1 root root   19883 Nov  8  2017 version-manifest.json
-rw-r--r--  1 root root    8696 Nov  8  2017 version-manifest.txt
[root@yinzhengjie download]# 
```

GitLab的安装位置（\[root@yinzhengjie download\]# ll /opt/gitlab/）

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@yinzhengjie download]# ll /var/log/gitlab/
total 4
drwx------ 2 git               root         44 Sep  9 02:14 gitaly
drwx------ 2 git               root         44 Sep  9 02:14 gitlab-monitor
drwx------ 2 git               root       4096 Sep  9 02:14 gitlab-rails
drwx------ 2 git               root         29 Sep  9 02:13 gitlab-shell
drwx------ 2 git               root         44 Sep  9 02:14 gitlab-workhorse
drwx------ 2 root              root         44 Sep  9 02:14 logrotate
drwxr-x--- 2 root              gitlab-www  124 Sep  9 02:14 nginx
drwx------ 2 gitlab-prometheus root         44 Sep  9 02:14 node-exporter
drwx------ 2 gitlab-psql       root         44 Sep  9 02:14 postgres-exporter
drwx------ 2 gitlab-psql       root         44 Sep  9 02:13 postgresql
drwx------ 2 gitlab-prometheus root         44 Sep  9 02:14 prometheus
drwxr-xr-x 2 root              root         27 Sep  9 02:12 reconfigure
drwx------ 2 gitlab-redis      root         44 Sep  9 02:13 redis
drwx------ 2 gitlab-redis      root         44 Sep  9 02:14 redis-exporter
drwx------ 2 git               root         44 Sep  9 02:14 sidekiq
drwx------ 2 git               root         94 Sep  9 02:14 unicorn
[root@yinzhengjie download]# 
```

GitLab的日志存放位置（\[root@yinzhengjie download\]# ll /var/log/gitlab/）

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@yinzhengjie ~]# ll /etc/gitlab/
total 84
-rw------- 1 root root 70929 Sep  9 02:03 gitlab.rb
-rw------- 1 root root  9648 Sep  9 02:13 gitlab-secrets.json
drwxr-xr-x 2 root root     6 Sep  9 02:13 trusted-certs
[root@yinzhengjie ~]# 
```

GitLab配置文件存放目录（\[root@yinzhengjie ~\]# ll /etc/gitlab/）

**三.使用GitLab步骤详解**

**1>.点击新建项目**

**![](https://images2018.cnblogs.com/blog/795254/201809/795254-20180908150046432-725056266.png)**

**2>.编辑新建项目信息**

![](https://images2018.cnblogs.com/blog/795254/201809/795254-20180908150830662-988714918.png)

**3>创建项目成功后的界面**

**![](https://images2018.cnblogs.com/blog/795254/201809/795254-20180908151328716-1043185266.png)**

**4>配置秘钥登录-****点击“add an SSH key”**

**![](https://images2018.cnblogs.com/blog/795254/201809/795254-20180908151610375-1076092278.png)**

**5>配置秘钥登录-将服务器端的秘钥id拷贝到GitLab的webUI界面上**

　　****关于配置SSH免密码登录大家可以参考我以前的笔记：https://www.cnblogs.com/yinzhengjie/p/9065191.html。****

**![](https://images2018.cnblogs.com/blog/795254/201809/795254-20180908152130283-1282672206.png)**

**6>.配置秘钥登录-配置成功的界面**

**![](https://images2018.cnblogs.com/blog/795254/201809/795254-20180908152327498-681889706.png)**

**7>.创建新仓库-查看帮助信息**

![](https://images2018.cnblogs.com/blog/795254/201809/795254-20180908160821400-2122600944.png)

**8>.**创建新仓库-克隆gitlab的项目到服务器本地****

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@yinzhengjie ~]# ll
total 0
drwxr-xr-x 2 root root 83 Sep  9 01:50 download
drwxr-xr-x 3 root root 45 Sep  8 23:13 git_data
[root@yinzhengjie ~]# 
[root@yinzhengjie ~]# git clone git@www.yinzhengjie.org.cn:root/yinzhengjieCode.git
Cloning into 'yinzhengjieCode'...
The authenticity of host 'www.yinzhengjie.org.cn (172.30.1.101)' can't be established.
ECDSA key fingerprint is SHA256:1MkICaFrw0jl80J9+gRJBa4W1QjDRafGqrFzRzae81E.
ECDSA key fingerprint is MD5:b6:44:e8:e7:76:d4:c2:4c:e0:02:7e:9c:d8:59:d8:13.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added 'www.yinzhengjie.org.cn' (ECDSA) to the list of known hosts.
warning: You appear to have cloned an empty repository.
[root@yinzhengjie ~]# 
[root@yinzhengjie ~]# ll
total 0
drwxr-xr-x 2 root root 83 Sep  9 01:50 download
drwxr-xr-x 3 root root 45 Sep  8 23:13 git_data
drwxr-xr-x 3 root root 17 Sep  9 04:08 yinzhengjieCode
[root@yinzhengjie ~]# 
[root@yinzhengjie ~]# cd yinzhengjieCode/
[root@yinzhengjie yinzhengjieCode]# 
[root@yinzhengjie yinzhengjieCode]# ll
total 0
[root@yinzhengjie yinzhengjieCode]# 
[root@yinzhengjie yinzhengjieCode]# 
```

\[root@yinzhengjie ~\]# git clone git@www.yinzhengjie.org.cn:root/yinzhengjieCode.git　　　#将gitlab的数据下载到本地服务器中  

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@yinzhengjie ~]# cd yinzhengjieCode/
[root@yinzhengjie yinzhengjieCode]# 
[root@yinzhengjie yinzhengjieCode]# ll
total 0
[root@yinzhengjie yinzhengjieCode]# 
[root@yinzhengjie yinzhengjieCode]# touch README.md
[root@yinzhengjie yinzhengjieCode]# echo http://www.cnblogs.com/yinzhengjie >> README.md 
[root@yinzhengjie yinzhengjieCode]# git add README.md
[root@yinzhengjie yinzhengjieCode]# git commit -m "add README"
[master (root-commit) 64b2f56] add README
 1 file changed, 0 insertions(+), 0 deletions(-)
 create mode 100644 README.md
[root@yinzhengjie yinzhengjieCode]# git push -u origin master    #将数据推送到gitlab中
Counting objects: 3, done.
Writing objects: 100% (3/3), 217 bytes | 0 bytes/s, done.
Total 3 (delta 0), reused 0 (delta 0)
To git@www.yinzhengjie.org.cn:root/yinzhengjieCode.git
 * [new branch]      master -> master
Branch master set up to track remote branch master from origin.
[root@yinzhengjie yinzhengjieCode]# 
[root@yinzhengjie yinzhengjieCode]# 
```

\[root@yinzhengjie yinzhengjieCode\]# git push -u origin master 　　　　　　　　　　　　　　#将数据推送到gitlab中

![](https://images2018.cnblogs.com/blog/795254/201809/795254-20180908162136380-471609667.png)

**9>.在网页上编辑文件**

![](https://images2018.cnblogs.com/blog/795254/201809/795254-20180908162401225-1805816709.png)

**10>.在网页上编辑完成后点击提交并查看修改后的内容**

**![](https://images2018.cnblogs.com/blog/795254/201809/795254-20180908162600751-2065964268.png)**

**11>.在服务器端查看编译后的文件内容**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@yinzhengjie yinzhengjieCode]# ll
total 4
-rw-r--r-- 1 root root 70 Sep  9 04:15 README.md
[root@yinzhengjie yinzhengjieCode]# 
[root@yinzhengjie yinzhengjieCode]# cat README.md 
http://www.cnblogs.com/yinzhengjie
http://www.cnblogs.com/yinzhengjie
[root@yinzhengjie yinzhengjieCode]# 
[root@yinzhengjie yinzhengjieCode]# git pull
remote: Counting objects: 3, done.
remote: Compressing objects: 100% (2/2), done.
remote: Total 3 (delta 0), reused 0 (delta 0)
Unpacking objects: 100% (3/3), done.
From www.yinzhengjie.org.cn:root/yinzhengjieCode
   abe070d..a697c69  master     -> origin/master
Auto-merging README.md
CONFLICT (content): Merge conflict in README.md
Automatic merge failed; fix conflicts and then commit the result.
[root@yinzhengjie yinzhengjieCode]# 
[root@yinzhengjie yinzhengjieCode]# ll
total 4
-rw-r--r-- 1 root root 236 Sep  9 04:27 README.md
[root@yinzhengjie yinzhengjieCode]# 
[root@yinzhengjie yinzhengjieCode]# cat README.md 
http://www.cnblogs.com/yinzhengjie
<<<<<<< HEAD
http://www.cnblogs.com/yinzhengjie
=======

尹正杰到此一游！！！！


大数据


自动化运维


人工智能


深度学习

>>>>>>> a697c6969dff86c6743060f0d829090508eb1243
[root@yinzhengjie yinzhengjieCode]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

当你的才华还撑不起你的野心的时候，你就应该静下心来学习。当你的能力还驾驭不了你的目标的时候，你就应该沉下心来历练。问问自己，想要怎样的人生。 欢迎加入基础架构自动化运维：598432640，大数据SRE进阶之路：959042252，DevOps进阶之路：526991186