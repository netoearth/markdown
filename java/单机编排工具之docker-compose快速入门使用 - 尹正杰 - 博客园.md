　　　　　　　　　　**单机编排工具之docker-compose快速入门使用**

　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　**作者：尹正杰**

**版权声明：原创作品，谢绝转载！否则将追究法律责任。**

**一.实验镜像准备**

**1>.自行安装harbor服务及制作web服务镜像**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
　　配置Harbor支持https功能实战篇:
　　　　https://www.cnblogs.com/yinzhengjie/p/12237263.html

　　自定义haproxy镜像:
　　　　https://www.cnblogs.com/yinzhengjie/p/12231702.html

　　自定义tomcat业务镜像:
　　　　https://www.cnblogs.com/yinzhengjie/p/12230043.html

　　基于DockerFile制作yum版nginx镜像:
　　　　https://www.cnblogs.com/yinzhengjie/p/12194460.html
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

**2>.在harbor服务端新建镜像仓库**

**![](https://img2018.cnblogs.com/i-beta/795254/202002/795254-20200202104815979-872772077.png)**

**3>.将haproxy镜像推送到harbor服务器端**

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@docker101.yinzhengjie.org.cn ~]# docker image ls
REPOSITORY               TAG                 IMAGE ID            CREATED             SIZE
nginx                    v0.1-20200201       1a8b4f68e96a        27 hours ago        449MB
centos-haproxy           v1.8.20             1858fe05d96f        9 days ago          606MB
registry                 latest              708bc6af7e5e        9 days ago          25.8MB
tomcat-app01             v0.1                bf45c22f2d5b        10 days ago         983MB
tomcat-base              8.5.50              9ff79f369094        11 days ago         968MB
jdk-base                 1.8.0_231           0f63a97ddc85        11 days ago         953MB
centos-base              7.6.1810            b4931fd9ace2        11 days ago         551MB
centos                   centos7.6.1810      f1cb7c7d58b7        10 months ago       202MB
lorel/docker-stress-ng   latest              1ae56ccafe55        3 years ago         8.1MB
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# docker tag centos-haproxy:v1.8.20 docker103.yinzhengjie.org.cn/yinzhengjie/centos-haproxy:v1.8.20
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# docker image ls
REPOSITORY                                                TAG                 IMAGE ID            CREATED             SIZE
nginx                                                     v0.1-20200201       1a8b4f68e96a        27 hours ago        449MB
centos-haproxy                                            v1.8.20             1858fe05d96f        9 days ago          606MB
docker103.yinzhengjie.org.cn/yinzhengjie/centos-haproxy   v1.8.20             1858fe05d96f        9 days ago          606MB
registry                                                  latest              708bc6af7e5e        9 days ago          25.8MB
tomcat-app01                                              v0.1                bf45c22f2d5b        10 days ago         983MB
tomcat-base                                               8.5.50              9ff79f369094        11 days ago         968MB
jdk-base                                                  1.8.0_231           0f63a97ddc85        11 days ago         953MB
centos-base                                               7.6.1810            b4931fd9ace2        11 days ago         551MB
centos                                                    centos7.6.1810      f1cb7c7d58b7        10 months ago       202MB
lorel/docker-stress-ng                                    latest              1ae56ccafe55        3 years ago         8.1MB
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# 
```

\[root@docker101.yinzhengjie.org.cn ~\]# docker tag centos-haproxy:v1.8.20 docker103.yinzhengjie.org.cn/yinzhengjie/centos-haproxy:v1.8.20

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@docker101.yinzhengjie.org.cn ~]# docker image ls
REPOSITORY                                                TAG                 IMAGE ID            CREATED             SIZE
nginx                                                     v0.1-20200201       1a8b4f68e96a        27 hours ago        449MB
centos-haproxy                                            v1.8.20             1858fe05d96f        9 days ago          606MB
docker103.yinzhengjie.org.cn/yinzhengjie/centos-haproxy   v1.8.20             1858fe05d96f        9 days ago          606MB
registry                                                  latest              708bc6af7e5e        9 days ago          25.8MB
tomcat-app01                                              v0.1                bf45c22f2d5b        10 days ago         983MB
tomcat-base                                               8.5.50              9ff79f369094        11 days ago         968MB
jdk-base                                                  1.8.0_231           0f63a97ddc85        11 days ago         953MB
centos-base                                               7.6.1810            b4931fd9ace2        11 days ago         551MB
centos                                                    centos7.6.1810      f1cb7c7d58b7        10 months ago       202MB
lorel/docker-stress-ng                                    latest              1ae56ccafe55        3 years ago         8.1MB
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# docker image push docker103.yinzhengjie.org.cn/yinzhengjie/centos-haproxy
The push refers to repository [docker103.yinzhengjie.org.cn/yinzhengjie/centos-haproxy]
4aeb0c6e6b59: Pushed 
019912d545c2: Pushed 
47eb3689b39c: Pushed 
c8ff3249af9a: Pushed 
0f448859d86e: Mounted from base_images/centos-base 
89169d87dbe2: Mounted from base_images/centos-base 
v1.8.20: digest: sha256:ba408aaaf5c50c57981444835464acc0ab1b5118c3b7aab54fad907813903eb0 size: 1579
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# 
```

\[root@docker101.yinzhengjie.org.cn ~\]# docker image push docker103.yinzhengjie.org.cn/yinzhengjie/centos-haproxy

 ![](https://img2018.cnblogs.com/i-beta/795254/202002/795254-20200202111433475-551406271.png)

**4>.将nginx**镜像推送到harbor服务器端****

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@docker101.yinzhengjie.org.cn ~]# docker image ls
REPOSITORY                                                TAG                 IMAGE ID            CREATED             SIZE
nginx                                                     v0.1-20200201       1a8b4f68e96a        28 hours ago        449MB
centos-haproxy                                            v1.8.20             1858fe05d96f        9 days ago          606MB
docker103.yinzhengjie.org.cn/yinzhengjie/centos-haproxy   v1.8.20             1858fe05d96f        9 days ago          606MB
registry                                                  latest              708bc6af7e5e        9 days ago          25.8MB
tomcat-app01                                              v0.1                bf45c22f2d5b        10 days ago         983MB
tomcat-base                                               8.5.50              9ff79f369094        11 days ago         968MB
jdk-base                                                  1.8.0_231           0f63a97ddc85        11 days ago         953MB
centos-base                                               7.6.1810            b4931fd9ace2        11 days ago         551MB
centos                                                    centos7.6.1810      f1cb7c7d58b7        10 months ago       202MB
lorel/docker-stress-ng                                    latest              1ae56ccafe55        3 years ago         8.1MB
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# docker image tag nginx:v0.1-20200201 docker103.yinzhengjie.org.cn/yinzhengjie/centos-nginx:v1.14.2
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# docker image ls
REPOSITORY                                                TAG                 IMAGE ID            CREATED             SIZE
nginx                                                     v0.1-20200201       1a8b4f68e96a        28 hours ago        449MB
docker103.yinzhengjie.org.cn/yinzhengjie/centos-nginx     v1.14.2             1a8b4f68e96a        28 hours ago        449MB
docker103.yinzhengjie.org.cn/yinzhengjie/centos-haproxy   v1.8.20             1858fe05d96f        9 days ago          606MB
centos-haproxy                                            v1.8.20             1858fe05d96f        9 days ago          606MB
registry                                                  latest              708bc6af7e5e        9 days ago          25.8MB
tomcat-app01                                              v0.1                bf45c22f2d5b        10 days ago         983MB
tomcat-base                                               8.5.50              9ff79f369094        11 days ago         968MB
jdk-base                                                  1.8.0_231           0f63a97ddc85        11 days ago         953MB
centos-base                                               7.6.1810            b4931fd9ace2        11 days ago         551MB
centos                                                    centos7.6.1810      f1cb7c7d58b7        10 months ago       202MB
lorel/docker-stress-ng                                    latest              1ae56ccafe55        3 years ago         8.1MB
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# 
```

\[root@docker101.yinzhengjie.org.cn ~\]# docker image tag nginx:v0.1-20200201 docker103.yinzhengjie.org.cn/yinzhengjie/centos-nginx:v1.14.2

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@docker101.yinzhengjie.org.cn ~]# docker image ls
REPOSITORY                                                TAG                 IMAGE ID            CREATED             SIZE
nginx                                                     v0.1-20200201       1a8b4f68e96a        28 hours ago        449MB
docker103.yinzhengjie.org.cn/yinzhengjie/centos-nginx     v1.14.2             1a8b4f68e96a        28 hours ago        449MB
centos-haproxy                                            v1.8.20             1858fe05d96f        9 days ago          606MB
docker103.yinzhengjie.org.cn/yinzhengjie/centos-haproxy   v1.8.20             1858fe05d96f        9 days ago          606MB
registry                                                  latest              708bc6af7e5e        9 days ago          25.8MB
tomcat-app01                                              v0.1                bf45c22f2d5b        10 days ago         983MB
tomcat-base                                               8.5.50              9ff79f369094        11 days ago         968MB
jdk-base                                                  1.8.0_231           0f63a97ddc85        11 days ago         953MB
centos-base                                               7.6.1810            b4931fd9ace2        11 days ago         551MB
centos                                                    centos7.6.1810      f1cb7c7d58b7        10 months ago       202MB
lorel/docker-stress-ng                                    latest              1ae56ccafe55        3 years ago         8.1MB
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# docker image push docker103.yinzhengjie.org.cn/yinzhengjie/centos-nginx
The push refers to repository [docker103.yinzhengjie.org.cn/yinzhengjie/centos-nginx]
bcba843846df: Pushed 
54d99d2f8875: Pushed 
265c695d22e3: Pushed 
89169d87dbe2: Mounted from yinzhengjie/centos-haproxy 
v1.14.2: digest: sha256:5bcaca2d82b6894253d2afee30571221e1132e9a8611f70ca632f4fe8658b6fe size: 1156
[root@docker101.yinzhengjie.org.cn ~]# 
```

\[root@docker101.yinzhengjie.org.cn ~\]# docker image push docker103.yinzhengjie.org.cn/yinzhengjie/centos-nginx

![](https://img2018.cnblogs.com/i-beta/795254/202002/795254-20200202115816443-1269622443.png)

**5>.**将tomcat**镜像推送到harbor服务器端******

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@docker101.yinzhengjie.org.cn ~]# docker image ls
REPOSITORY                                              TAG                 IMAGE ID            CREATED             SIZE
docker103.yinzhengjie.org.cn/yinzhengjie/centos-nginx   v1.14.2             1a8b4f68e96a        29 hours ago        449MB
nginx                                                   v0.1-20200201       1a8b4f68e96a        29 hours ago        449MB
registry                                                latest              708bc6af7e5e        9 days ago          25.8MB
tomcat-app01                                            v0.1                bf45c22f2d5b        10 days ago         983MB
tomcat-base                                             8.5.50              9ff79f369094        11 days ago         968MB
jdk-base                                                1.8.0_231           0f63a97ddc85        11 days ago         953MB
centos-base                                             7.6.1810            b4931fd9ace2        11 days ago         551MB
centos                                                  centos7.6.1810      f1cb7c7d58b7        10 months ago       202MB
lorel/docker-stress-ng                                  latest              1ae56ccafe55        3 years ago         8.1MB
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# docker image tag tomcat-app01:v0.1 docker103.yinzhengjie.org.cn/yinzhengjie/tomcat-app01:8.5.50
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# docker image ls
REPOSITORY                                              TAG                 IMAGE ID            CREATED             SIZE
docker103.yinzhengjie.org.cn/yinzhengjie/centos-nginx   v1.14.2             1a8b4f68e96a        29 hours ago        449MB
nginx                                                   v0.1-20200201       1a8b4f68e96a        29 hours ago        449MB
registry                                                latest              708bc6af7e5e        9 days ago          25.8MB
tomcat-app01                                            v0.1                bf45c22f2d5b        10 days ago         983MB
docker103.yinzhengjie.org.cn/yinzhengjie/tomcat-app01   8.5.50              bf45c22f2d5b        10 days ago         983MB
tomcat-base                                             8.5.50              9ff79f369094        11 days ago         968MB
jdk-base                                                1.8.0_231           0f63a97ddc85        11 days ago         953MB
centos-base                                             7.6.1810            b4931fd9ace2        11 days ago         551MB
centos                                                  centos7.6.1810      f1cb7c7d58b7        10 months ago       202MB
lorel/docker-stress-ng                                  latest              1ae56ccafe55        3 years ago         8.1MB
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# 
```

\[root@docker101.yinzhengjie.org.cn ~\]# docker image tag tomcat-app01:v0.1 docker103.yinzhengjie.org.cn/yinzhengjie/tomcat-app01:8.5.50

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@docker101.yinzhengjie.org.cn ~]# docker image ls
REPOSITORY                                              TAG                 IMAGE ID            CREATED             SIZE
nginx                                                   v0.1-20200201       1a8b4f68e96a        29 hours ago        449MB
docker103.yinzhengjie.org.cn/yinzhengjie/centos-nginx   v1.14.2             1a8b4f68e96a        29 hours ago        449MB
registry                                                latest              708bc6af7e5e        9 days ago          25.8MB
tomcat-app01                                            v0.1                bf45c22f2d5b        10 days ago         983MB
docker103.yinzhengjie.org.cn/yinzhengjie/tomcat-app01   8.5.50              bf45c22f2d5b        10 days ago         983MB
tomcat-base                                             8.5.50              9ff79f369094        11 days ago         968MB
jdk-base                                                1.8.0_231           0f63a97ddc85        11 days ago         953MB
centos-base                                             7.6.1810            b4931fd9ace2        11 days ago         551MB
centos                                                  centos7.6.1810      f1cb7c7d58b7        10 months ago       202MB
lorel/docker-stress-ng                                  latest              1ae56ccafe55        3 years ago         8.1MB
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# docker image push docker103.yinzhengjie.org.cn/yinzhengjie/tomcat-app01:8.5.50 
The push refers to repository [docker103.yinzhengjie.org.cn/yinzhengjie/tomcat-app01]
5a4703440aa0: Pushed 
6cb4b904056a: Pushed 
8470f758a98b: Pushed 
926483cbcbb4: Pushed 
84bb4f431a8f: Pushed 
22ac492b2c15: Pushed 
fc1a47f2a301: Pushed 
9f0513d2c943: Pushed 
a9a8bd89bd66: Pushed 
0f448859d86e: Mounted from yinzhengjie/centos-haproxy 
89169d87dbe2: Mounted from yinzhengjie/centos-nginx 
8.5.50: digest: sha256:184fb625634163294e0f2fb68c40657176fff69a8bed4b0329c742cab5e7e088 size: 2623
[root@docker101.yinzhengjie.org.cn ~]# 
```

\[root@docker101.yinzhengjie.org.cn ~\]# docker image push docker103.yinzhengjie.org.cn/yinzhengjie/tomcat-app01:8.5.50

![](https://img2018.cnblogs.com/i-beta/795254/202002/795254-20200202125805706-989235553.png)

**二.安装docker-compose工具**

**1>.****下载新repo到/etc/yum.repos.d/**

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@docker102.yinzhengjie.org.cn ~]# wget -O /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-7.repo
--2020-02-02 19:39:24--  http://mirrors.aliyun.com/repo/epel-7.repo
Resolving mirrors.aliyun.com (mirrors.aliyun.com)... 119.167.173.238, 119.167.173.244, 27.221.56.244, ...
Connecting to mirrors.aliyun.com (mirrors.aliyun.com)|119.167.173.238|:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 664 [application/octet-stream]
Saving to: ‘/etc/yum.repos.d/epel.repo’

100%[============================================================================================================================================================>] 664         --.-K/s   in 0s      

2020-02-02 19:39:24 (1.47 MB/s) - ‘/etc/yum.repos.d/epel.repo’ saved [664/664]

[root@docker102.yinzhengjie.org.cn ~]# 
[root@docker102.yinzhengjie.org.cn ~]# ll /etc/yum.repos.d/
total 40
-rw-r--r--. 1 root root 1664 Nov 23  2018 CentOS-Base.repo
-rw-r--r--. 1 root root 1309 Nov 23  2018 CentOS-CR.repo
-rw-r--r--. 1 root root  649 Nov 23  2018 CentOS-Debuginfo.repo
-rw-r--r--. 1 root root  314 Nov 23  2018 CentOS-fasttrack.repo
-rw-r--r--. 1 root root  630 Nov 23  2018 CentOS-Media.repo
-rw-r--r--. 1 root root 1331 Nov 23  2018 CentOS-Sources.repo
-rw-r--r--. 1 root root 5701 Nov 23  2018 CentOS-Vault.repo
-rw-r--r--  1 root root 2640 Jan 23 16:12 docker-ce.repo
-rw-r--r--  1 root root  664 May 11  2018 epel.repo
[root@docker102.yinzhengjie.org.cn ~]# 
```

\[root@docker102.yinzhengjie.org.cn ~\]# wget -O /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-7.repo

**2>.**安装python环境及pip命令****

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@docker102.yinzhengjie.org.cn ~]# yum -y install python-pip
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
 * base: mirror.bit.edu.cn
 * extras: mirrors.neusoft.edu.cn
 * updates: mirror.bit.edu.cn
epel                                                                                                                                                                           | 5.3 kB  00:00:00     
(1/3): epel/x86_64/group_gz                                                                                                                                                    |  90 kB  00:00:00     
(2/3): epel/x86_64/updateinfo                                                                                                                                                  | 1.0 MB  00:00:00     
(3/3): epel/x86_64/primary_db                                                                                                                                                  | 6.9 MB  00:00:00     
Resolving Dependencies
--> Running transaction check
---> Package python2-pip.noarch 0:8.1.2-10.el7 will be installed
--> Processing Dependency: python-setuptools for package: python2-pip-8.1.2-10.el7.noarch
--> Running transaction check
---> Package python-setuptools.noarch 0:0.9.8-7.el7 will be installed
--> Processing Dependency: python-backports-ssl_match_hostname for package: python-setuptools-0.9.8-7.el7.noarch
--> Running transaction check
---> Package python-backports-ssl_match_hostname.noarch 0:3.5.0.1-1.el7 will be installed
--> Processing Dependency: python-ipaddress for package: python-backports-ssl_match_hostname-3.5.0.1-1.el7.noarch
--> Processing Dependency: python-backports for package: python-backports-ssl_match_hostname-3.5.0.1-1.el7.noarch
--> Running transaction check
---> Package python-backports.x86_64 0:1.0-8.el7 will be installed
---> Package python-ipaddress.noarch 0:1.0.16-2.el7 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

======================================================================================================================================================================================================
 Package                                                             Arch                                   Version                                        Repository                            Size
======================================================================================================================================================================================================
Installing:
 python2-pip                                                         noarch                                 8.1.2-10.el7                                   epel                                 1.7 M
Installing for dependencies:
 python-backports                                                    x86_64                                 1.0-8.el7                                      base                                 5.8 k
 python-backports-ssl_match_hostname                                 noarch                                 3.5.0.1-1.el7                                  base                                  13 k
 python-ipaddress                                                    noarch                                 1.0.16-2.el7                                   base                                  34 k
 python-setuptools                                                   noarch                                 0.9.8-7.el7                                    base                                 397 k

Transaction Summary
======================================================================================================================================================================================================
Install  1 Package (+4 Dependent packages)

Total size: 2.1 M
Total download size: 1.7 M
Installed size: 9.4 M
Downloading packages:
python2-pip-8.1.2-10.el7.noarch.rpm                                                                                                                                            | 1.7 MB  00:00:00     
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : python-backports-1.0-8.el7.x86_64                                                                                                                                                  1/5 
  Installing : python-ipaddress-1.0.16-2.el7.noarch                                                                                                                                               2/5 
  Installing : python-backports-ssl_match_hostname-3.5.0.1-1.el7.noarch                                                                                                                           3/5 
  Installing : python-setuptools-0.9.8-7.el7.noarch                                                                                                                                               4/5 
  Installing : python2-pip-8.1.2-10.el7.noarch                                                                                                                                                    5/5 
  Verifying  : python-ipaddress-1.0.16-2.el7.noarch                                                                                                                                               1/5 
  Verifying  : python-setuptools-0.9.8-7.el7.noarch                                                                                                                                               2/5 
  Verifying  : python-backports-ssl_match_hostname-3.5.0.1-1.el7.noarch                                                                                                                           3/5 
  Verifying  : python-backports-1.0-8.el7.x86_64                                                                                                                                                  4/5 
  Verifying  : python2-pip-8.1.2-10.el7.noarch                                                                                                                                                    5/5 

Installed:
  python2-pip.noarch 0:8.1.2-10.el7                                                                                                                                                                   

Dependency Installed:
  python-backports.x86_64 0:1.0-8.el7       python-backports-ssl_match_hostname.noarch 0:3.5.0.1-1.el7       python-ipaddress.noarch 0:1.0.16-2.el7       python-setuptools.noarch 0:0.9.8-7.el7      

Complete!
[root@docker102.yinzhengjie.org.cn ~]#  
```

\[root@docker102.yinzhengjie.org.cn ~\]# yum -y install python-pip

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@docker102.yinzhengjie.org.cn ~]# pip install --upgrade pip
Collecting pip
  Downloading https://files.pythonhosted.org/packages/54/0c/d01aa759fdc501a58f431eb594a17495f15b88da142ce14b5845662c13f3/pip-20.0.2-py2.py3-none-any.whl (1.4MB)
    100% |████████████████████████████████| 1.4MB 1.1MB/s 
Installing collected packages: pip
  Found existing installation: pip 8.1.2
    Uninstalling pip-8.1.2:
      Successfully uninstalled pip-8.1.2
Successfully installed pip-20.0.2
[root@docker102.yinzhengjie.org.cn ~]# 
[root@docker102.yinzhengjie.org.cn ~]# 
```

\[root@docker102.yinzhengjie.org.cn ~\]# pip install --upgrade pip

**3>.安装docker-compose工具(ubantu系统只需要执行"apt-get install docker-compose"即可)**

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@docker102.yinzhengjie.org.cn ~]# yum -y install python-devel
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
 * base: mirror.bit.edu.cn
 * extras: mirrors.neusoft.edu.cn
 * updates: mirror.bit.edu.cn
Resolving Dependencies
--> Running transaction check
---> Package python-devel.x86_64 0:2.7.5-86.el7 will be installed
--> Processing Dependency: python(x86-64) = 2.7.5-86.el7 for package: python-devel-2.7.5-86.el7.x86_64
--> Processing Dependency: python2-rpm-macros > 3-30 for package: python-devel-2.7.5-86.el7.x86_64
--> Processing Dependency: python-rpm-macros > 3-30 for package: python-devel-2.7.5-86.el7.x86_64
--> Running transaction check
---> Package python.x86_64 0:2.7.5-76.el7 will be updated
---> Package python.x86_64 0:2.7.5-86.el7 will be an update
--> Processing Dependency: python-libs(x86-64) = 2.7.5-86.el7 for package: python-2.7.5-86.el7.x86_64
---> Package python-rpm-macros.noarch 0:3-32.el7 will be installed
--> Processing Dependency: python-srpm-macros for package: python-rpm-macros-3-32.el7.noarch
---> Package python2-rpm-macros.noarch 0:3-32.el7 will be installed
--> Running transaction check
---> Package python-libs.x86_64 0:2.7.5-76.el7 will be updated
---> Package python-libs.x86_64 0:2.7.5-86.el7 will be an update
---> Package python-srpm-macros.noarch 0:3-32.el7 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

======================================================================================================================================================================================================
 Package                                                Arch                                       Version                                             Repository                                Size
======================================================================================================================================================================================================
Installing:
 python-devel                                           x86_64                                     2.7.5-86.el7                                        base                                     398 k
Installing for dependencies:
 python-rpm-macros                                      noarch                                     3-32.el7                                            base                                     8.8 k
 python-srpm-macros                                     noarch                                     3-32.el7                                            base                                     8.4 k
 python2-rpm-macros                                     noarch                                     3-32.el7                                            base                                     7.7 k
Updating for dependencies:
 python                                                 x86_64                                     2.7.5-86.el7                                        base                                      95 k
 python-libs                                            x86_64                                     2.7.5-86.el7                                        base                                     5.6 M

Transaction Summary
======================================================================================================================================================================================================
Install  1 Package  (+3 Dependent packages)
Upgrade             ( 2 Dependent packages)

Total download size: 6.2 M
Downloading packages:
Delta RPMs disabled because /usr/bin/applydeltarpm not installed.
(1/6): python-2.7.5-86.el7.x86_64.rpm                                                                                                                                          |  95 kB  00:00:00     
(2/6): python-srpm-macros-3-32.el7.noarch.rpm                                                                                                                                  | 8.4 kB  00:00:00     
(3/6): python2-rpm-macros-3-32.el7.noarch.rpm                                                                                                                                  | 7.7 kB  00:00:00     
(4/6): python-rpm-macros-3-32.el7.noarch.rpm                                                                                                                                   | 8.8 kB  00:00:00     
(5/6): python-devel-2.7.5-86.el7.x86_64.rpm                                                                                                                                    | 398 kB  00:00:00     
(6/6): python-libs-2.7.5-86.el7.x86_64.rpm                                                                                                                                     | 5.6 MB  00:00:01     
------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
Total                                                                                                                                                                 5.1 MB/s | 6.2 MB  00:00:01     
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Updating   : python-libs-2.7.5-86.el7.x86_64                                                                                                                                                    1/8 
  Updating   : python-2.7.5-86.el7.x86_64                                                                                                                                                         2/8 
  Installing : python2-rpm-macros-3-32.el7.noarch                                                                                                                                                 3/8 
  Installing : python-srpm-macros-3-32.el7.noarch                                                                                                                                                 4/8 
  Installing : python-rpm-macros-3-32.el7.noarch                                                                                                                                                  5/8 
  Installing : python-devel-2.7.5-86.el7.x86_64                                                                                                                                                   6/8 
  Cleanup    : python-2.7.5-76.el7.x86_64                                                                                                                                                         7/8 
  Cleanup    : python-libs-2.7.5-76.el7.x86_64                                                                                                                                                    8/8 
  Verifying  : python-libs-2.7.5-86.el7.x86_64                                                                                                                                                    1/8 
  Verifying  : python-devel-2.7.5-86.el7.x86_64                                                                                                                                                   2/8 
  Verifying  : python-2.7.5-86.el7.x86_64                                                                                                                                                         3/8 
  Verifying  : python-srpm-macros-3-32.el7.noarch                                                                                                                                                 4/8 
  Verifying  : python2-rpm-macros-3-32.el7.noarch                                                                                                                                                 5/8 
  Verifying  : python-rpm-macros-3-32.el7.noarch                                                                                                                                                  6/8 
  Verifying  : python-2.7.5-76.el7.x86_64                                                                                                                                                         7/8 
  Verifying  : python-libs-2.7.5-76.el7.x86_64                                                                                                                                                    8/8 

Installed:
  python-devel.x86_64 0:2.7.5-86.el7                                                                                                                                                                  

Dependency Installed:
  python-rpm-macros.noarch 0:3-32.el7                              python-srpm-macros.noarch 0:3-32.el7                              python2-rpm-macros.noarch 0:3-32.el7                             

Dependency Updated:
  python.x86_64 0:2.7.5-86.el7                                                                    python-libs.x86_64 0:2.7.5-86.el7                                                                   

Complete!
[root@docker102.yinzhengjie.org.cn ~]# 
```

\[root@docker102.yinzhengjie.org.cn ~\]# yum -y install python-devel　　　　　　　　　　#如果这个依赖包不安装可能在安装docker compose会失败哟~

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@docker102.yinzhengjie.org.cn ~]# pip install docker-compose
DEPRECATION: Python 2.7 reached the end of its life on January 1st, 2020. Please upgrade your Python as Python 2.7 is no longer maintained. A future version of pip will drop support for Python 2.7. 
More details about Python 2 support in pip, can be found at https://pip.pypa.io/en/latest/development/release-process/#python-2-supportCollecting docker-compose
  Using cached docker_compose-1.25.3-py2.py3-none-any.whl (136 kB)
Requirement already satisfied: backports.shutil-get-terminal-size==1.0.0; python_version < "3.3" in /usr/lib/python2.7/site-packages (from docker-compose) (1.0.0)
Requirement already satisfied: six<2,>=1.3.0 in /usr/lib/python2.7/site-packages (from docker-compose) (1.14.0)
Requirement already satisfied: PyYAML<6,>=3.10 in /usr/lib64/python2.7/site-packages (from docker-compose) (5.3)
Requirement already satisfied: docker[ssh]<5,>=3.7.0 in /usr/lib/python2.7/site-packages (from docker-compose) (4.1.0)
Requirement already satisfied: dockerpty<1,>=0.4.1 in /usr/lib/python2.7/site-packages (from docker-compose) (0.4.1)
Requirement already satisfied: jsonschema<4,>=2.5.1 in /usr/lib/python2.7/site-packages (from docker-compose) (3.2.0)
Requirement already satisfied: requests<3,>=2.20.0 in /usr/lib/python2.7/site-packages (from docker-compose) (2.22.0)
Requirement already satisfied: websocket-client<1,>=0.32.0 in /usr/lib/python2.7/site-packages (from docker-compose) (0.57.0)
Requirement already satisfied: cached-property<2,>=1.2.0 in /usr/lib/python2.7/site-packages (from docker-compose) (1.5.1)
Requirement already satisfied: ipaddress<2,>=1.0.16; python_version < "3.3" in /usr/lib/python2.7/site-packages (from docker-compose) (1.0.16)
Requirement already satisfied: docopt<1,>=0.6.1 in /usr/lib/python2.7/site-packages (from docker-compose) (0.6.2)
Collecting subprocess32<4,>=3.5.4; python_version < "3.2"
  Using cached subprocess32-3.5.4.tar.gz (97 kB)
Requirement already satisfied: enum34<2,>=1.0.4; python_version < "3.4" in /usr/lib/python2.7/site-packages (from docker-compose) (1.1.6)
Collecting texttable<2,>=0.9.0
  Using cached texttable-1.6.2-py2.py3-none-any.whl (10 kB)
Requirement already satisfied: backports.ssl-match-hostname<4,>=3.5; python_version < "3.5" in /usr/lib/python2.7/site-packages (from docker-compose) (3.5.0.1)
Requirement already satisfied: paramiko>=2.4.2; extra == "ssh" in /usr/lib/python2.7/site-packages (from docker[ssh]<5,>=3.7.0->docker-compose) (2.7.1)
Requirement already satisfied: setuptools in /usr/lib/python2.7/site-packages (from jsonschema<4,>=2.5.1->docker-compose) (0.9.8)
Requirement already satisfied: pyrsistent>=0.14.0 in /usr/lib64/python2.7/site-packages (from jsonschema<4,>=2.5.1->docker-compose) (0.15.7)
Requirement already satisfied: attrs>=17.4.0 in /usr/lib/python2.7/site-packages (from jsonschema<4,>=2.5.1->docker-compose) (19.3.0)
Requirement already satisfied: importlib-metadata; python_version < "3.8" in /usr/lib/python2.7/site-packages (from jsonschema<4,>=2.5.1->docker-compose) (1.5.0)
Requirement already satisfied: functools32; python_version < "3" in /usr/lib/python2.7/site-packages (from jsonschema<4,>=2.5.1->docker-compose) (3.2.3.post2)
Requirement already satisfied: chardet<3.1.0,>=3.0.2 in /usr/lib/python2.7/site-packages (from requests<3,>=2.20.0->docker-compose) (3.0.4)
Requirement already satisfied: idna<2.9,>=2.5 in /usr/lib/python2.7/site-packages (from requests<3,>=2.20.0->docker-compose) (2.8)
Requirement already satisfied: urllib3!=1.25.0,!=1.25.1,<1.26,>=1.21.1 in /usr/lib/python2.7/site-packages (from requests<3,>=2.20.0->docker-compose) (1.25.8)
Requirement already satisfied: certifi>=2017.4.17 in /usr/lib/python2.7/site-packages (from requests<3,>=2.20.0->docker-compose) (2019.11.28)
Requirement already satisfied: bcrypt>=3.1.3 in /usr/lib64/python2.7/site-packages (from paramiko>=2.4.2; extra == "ssh"->docker[ssh]<5,>=3.7.0->docker-compose) (3.1.7)
Requirement already satisfied: pynacl>=1.0.1 in /usr/lib64/python2.7/site-packages (from paramiko>=2.4.2; extra == "ssh"->docker[ssh]<5,>=3.7.0->docker-compose) (1.3.0)
Requirement already satisfied: cryptography>=2.5 in /usr/lib64/python2.7/site-packages (from paramiko>=2.4.2; extra == "ssh"->docker[ssh]<5,>=3.7.0->docker-compose) (2.8)
Requirement already satisfied: pathlib2; python_version < "3" in /usr/lib/python2.7/site-packages (from importlib-metadata; python_version < "3.8"->jsonschema<4,>=2.5.1->docker-compose) (2.3.5)
Requirement already satisfied: contextlib2; python_version < "3" in /usr/lib/python2.7/site-packages (from importlib-metadata; python_version < "3.8"->jsonschema<4,>=2.5.1->docker-compose) (0.6.0.po
st1)Requirement already satisfied: zipp>=0.5 in /usr/lib/python2.7/site-packages (from importlib-metadata; python_version < "3.8"->jsonschema<4,>=2.5.1->docker-compose) (1.1.0)
Requirement already satisfied: configparser>=3.5; python_version < "3" in /usr/lib/python2.7/site-packages (from importlib-metadata; python_version < "3.8"->jsonschema<4,>=2.5.1->docker-compose) (4.
0.2)Requirement already satisfied: cffi>=1.1 in /usr/lib64/python2.7/site-packages (from bcrypt>=3.1.3->paramiko>=2.4.2; extra == "ssh"->docker[ssh]<5,>=3.7.0->docker-compose) (1.13.2)
Requirement already satisfied: scandir; python_version < "3.5" in /usr/lib64/python2.7/site-packages (from pathlib2; python_version < "3"->importlib-metadata; python_version < "3.8"->jsonschema<4,>=
2.5.1->docker-compose) (1.10.0)Requirement already satisfied: pycparser in /usr/lib/python2.7/site-packages (from cffi>=1.1->bcrypt>=3.1.3->paramiko>=2.4.2; extra == "ssh"->docker[ssh]<5,>=3.7.0->docker-compose) (2.19)
Installing collected packages: subprocess32, texttable, docker-compose
    Running setup.py install for subprocess32 ... done
Successfully installed docker-compose-1.25.3 subprocess32-3.5.4 texttable-1.6.2
[root@docker102.yinzhengjie.org.cn ~]# 
```

\[root@docker102.yinzhengjie.org.cn ~\]# pip install docker-compose

![](https://img2018.cnblogs.com/i-beta/795254/202002/795254-20200202114826365-506832095.png)

**4>.验证docker-compose版本**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@docker102.yinzhengjie.org.cn ~]# docker-compose version
docker-compose version 1.25.3, build unknown
docker-py version: 4.1.0
CPython version: 2.7.5
OpenSSL version: OpenSSL 1.0.2k-fips  26 Jan 2017
[root@docker102.yinzhengjie.org.cn ~]# 
[root@docker102.yinzhengjie.org.cn ~]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

**5>.查看**docker-compose**的帮助信息**

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@docker102.yinzhengjie.org.cn ~]# docker-compose --help
Define and run multi-container applications with Docker.

Usage:
  docker-compose [-f <arg>...] [options] [COMMAND] [ARGS...]
  docker-compose -h|--help

Options:
  -f, --file FILE             Specify an alternate compose file
                              (default: docker-compose.yml)
  -p, --project-name NAME     Specify an alternate project name
                              (default: directory name)
  --verbose                   Show more output
  --log-level LEVEL           Set log level (DEBUG, INFO, WARNING, ERROR, CRITICAL)
  --no-ansi                   Do not print ANSI control characters
  -v, --version               Print version and exit
  -H, --host HOST             Daemon socket to connect to

  --tls                       Use TLS; implied by --tlsverify
  --tlscacert CA_PATH         Trust certs signed only by this CA
  --tlscert CLIENT_CERT_PATH  Path to TLS certificate file
  --tlskey TLS_KEY_PATH       Path to TLS key file
  --tlsverify                 Use TLS and verify the remote
  --skip-hostname-check       Don't check the daemon's hostname against the
                              name specified in the client certificate
  --project-directory PATH    Specify an alternate working directory
                              (default: the path of the Compose file)
  --compatibility             If set, Compose will attempt to convert keys
                              in v3 files to their non-Swarm equivalent
  --env-file PATH             Specify an alternate environment file

Commands:
  build              Build or rebuild services
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
[root@docker102.yinzhengjie.org.cn ~]# 
```

\[root@docker102.yinzhengjie.org.cn ~\]# docker-compose --help

 ![](https://img2018.cnblogs.com/i-beta/795254/202002/795254-20200202115110870-988365984.png)

**三.使用****docker-compose****工具**

**1>.操作平台及试验架构说明**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
　　使用docker-compose工具自动生成三个容器应用，即harpoxy,nginx-web,tomcat-app01。　　harpory应用功能:　　　　负载均衡服务器，主要将访问80端口的转发给后台的nginx-web应用，暴漏8888端口为harpoxy的状态页。　　nginx-web应用功能：　　　　主要讲访问"/app01"的path调度到后端的tomcat-app01应用服务上。　　tomcat-app01:　　　　主要提供tomcat应用的访问页面。　　以上三种镜像我在之前的笔记也分享过，制作起来也非常简单，可参考我第一部分提供的连接自行制作即可。
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

**![](https://img2018.cnblogs.com/i-beta/795254/202002/795254-20200202164934328-1579159609.png)**

**2>.为了实验方便,建议找一台干净的宿主机(只需要安装docker环境即可) ，或者将该节点的所有容器和镜像都删除掉**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@docker102.yinzhengjie.org.cn ~]# docker container rm -f `docker container ps -a -q`
70da6a2cad6b
[root@docker102.yinzhengjie.org.cn ~]# 
[root@docker102.yinzhengjie.org.cn ~]# docker image ls
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
redis               latest              9b188f5fb1e6        4 weeks ago         98.2MB
centos              centos7.6.1810      f1cb7c7d58b7        10 months ago       202MB
[root@docker102.yinzhengjie.org.cn ~]# 
[root@docker102.yinzhengjie.org.cn ~]# 
[root@docker102.yinzhengjie.org.cn ~]# docker image rm -f 9b188f5fb1e6 f1cb7c7d58b7
Untagged: redis:latest
Untagged: redis@sha256:90d44d431229683cadd75274e6fcb22c3e0396d149a8f8b7da9925021ee75c30
Deleted: sha256:9b188f5fb1e6e1c7b10045585cb386892b2b4e1d31d62e3688c6fa8bf9fd32b5
Deleted: sha256:fe7afb618c11b8be098a10564a9a1682f83915bfdbaaa5af48791950d418b2d5
Deleted: sha256:3a284ce371b3431ba30071057478e2db8fc096232b1a84f092c4df9e06a4a3e4
Deleted: sha256:4396548b331d1b748c8ba1542f8da54e0a8b84102d8205440aac61e3941bdf71
Deleted: sha256:c80de70938af062d3c273f9925641ec672fe182a796bb4a096a37963c92e071a
Deleted: sha256:e807dfe0532b9dae274911841bab81588e9e34591a5b809b8da39471fb75fdbd
Deleted: sha256:556c5fb0d91b726083a8ce42e2faaed99f11bc68d3f70e2c7bbce87e7e0b3e10
Untagged: centos:centos7.6.1810
Untagged: centos@sha256:62d9e1c2daa91166139b51577fe4f4f6b4cc41a3a2c7fc36bd895e2a17a3e4e6
Deleted: sha256:f1cb7c7d58b73eac859c395882eec49d50651244e342cd6c68a5c7809785f427
Deleted: sha256:89169d87dbe2b72ba42bfbb3579c957322baca28e03a1e558076542a1c1b2b4a
[root@docker102.yinzhengjie.org.cn ~]# 
[root@docker102.yinzhengjie.org.cn ~]# 
[root@docker102.yinzhengjie.org.cn ~]# docker image ls
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
[root@docker102.yinzhengjie.org.cn ~]# 
[root@docker102.yinzhengjie.org.cn ~]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

![](https://img2018.cnblogs.com/i-beta/795254/202002/795254-20200202112011269-1321948230.png)

**3>.编写docke-compose的配置文件**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@docker102.yinzhengjie.org.cn ~]# 
[root@docker102.yinzhengjie.org.cn ~]# cd /yinzhengjie/data/
[root@docker102.yinzhengjie.org.cn /yinzhengjie/data]# 
[root@docker102.yinzhengjie.org.cn /yinzhengjie/data]# ll
total 4
-rw-r--r-- 1 root root 984 Feb  3 00:15 docker-compose.yml
[root@docker102.yinzhengjie.org.cn /yinzhengjie/data]# 
[root@docker102.yinzhengjie.org.cn /yinzhengjie/data]# vim docker-compose.yml 
[root@docker102.yinzhengjie.org.cn /yinzhengjie/data]# 
[root@docker102.yinzhengjie.org.cn /yinzhengjie/data]# cat docker-compose.yml 
haproxy:
    image: docker103.yinzhengjie.org.cn/yinzhengjie/centos-haproxy:v1.8.20
    container_name: myHaproxy
    expose:
        - 80
        - 8888
    volumes:
        - /yinzhengjie/softwares/dockerfile/web/haproxy/haproxy.cfg:/etc/haproxy/haproxy.cfg
    ports:
        - "80:80"
        - "8888:8888"
    links:
        - nginx-web


nginx-web:
    image: docker103.yinzhengjie.org.cn/yinzhengjie/centos-nginx:v1.14.2
    container_name: myNginx
    ports:
        - "8080:80"
        - "443:443"
    volumes:
        - /yinzhengjie/softwares/dockerfile/web/nginx/nginx.conf:/etc/nginx/nginx.conf
        - /yinzhengjie/softwares/dockerfile/web/nginx/log:/var/log/nginx/
        #- /yinzhengjie/softwares/dockerfile/web/nginx/webpage:/usr/share/nginx/html/webpage
    links:
        - tomcat-app01

tomcat-app01:
    image: docker103.yinzhengjie.org.cn/yinzhengjie/tomcat-app01:8.5.50
    container_name: myTomcat
    expose:
        - 8080
    ports:
        - "8081:8080"
[root@docker102.yinzhengjie.org.cn /yinzhengjie/data]# 
[root@docker102.yinzhengjie.org.cn /yinzhengjie/data]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

![](https://img2018.cnblogs.com/i-beta/795254/202002/795254-20200202162305095-1480153643.png)

**4>.准备haproxy和nginx的配置文件**

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@docker102.yinzhengjie.org.cn ~]# cat /yinzhengjie/softwares/dockerfile/web/haproxy/haproxy.cfg
global
    maxconn 100000
    chroot /yinzhengjie/softwares/haproxy
    stats socket /yinzhengjie/softwares/haproxy/haproxy.sock mode 600 level admin
    uid 99
    gid 99
    daemon
    pidfile /yinzhengjie/softwares/haproxy/run/haproxy.pid

defaults
    option http-keep-alive
    option  forwardfor
    option redispatch
    option abortonclose
    maxconn 100000
    mode http
    timeout connect 300000ms
    timeout client  300000ms
    timeout server  300000ms

listen status_page
    bind 0.0.0.0:8888
    stats enable
    stats uri /haproxy-status
    stats auth    admin:yinzhengjie
    stats realm "Welcome to the haproxy load balancer status page of YinZhengjie"

listen WEB_PORT_80
    bind 0.0.0.0:80
    mode http
    server web01 myNginx:80 check inter 3000 fall 3 rise 5
[root@docker102.yinzhengjie.org.cn ~]# 
```

\[root@docker102.yinzhengjie.org.cn ~\]# cat /yinzhengjie/softwares/dockerfile/web/haproxy/haproxy.cfg

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@docker102.yinzhengjie.org.cn ~]# cat /yinzhengjie/softwares/dockerfile/web/nginx/nginx.conf 
user nginx;

worker_processes auto;

error_log /var/log/nginx/error.log;

pid /run/nginx.pid;

daemon off;

include /usr/share/nginx/modules/*.conf;

events {
    worker_connections 1024;
}
   
http {
    log_format my_access_json '{"@timestamp":"$time_iso8601",' 
                              '"host":"$server_addr",' 
                              '"clientip":"$remote_addr",' 
                  '"size":$body_bytes_sent,' 
                             '"responsetime":$request_time,' 
                             '"upstreamtime":"$upstream_response_time",' 
                             '"upstreamhost":"$upstream_addr",' 
                              '"http_host":"$host",' 
                            '"uri":"$uri",' 
                             '"domain":"$host",' 
                             '"xff":"$http_x_forwarded_for",' 
                  '"referer":"$http_referer",' 
                             '"tcp_xff":"$proxy_protocol_addr",' 
                             '"http_user_agent":"$http_user_agent",' 
                             '"status":"$status"}';
              
    access_log /var/log/nginx/access_json.log my_access_json;

    sendfile            on;
    keepalive_timeout   65;
    include       mime.types;
    default_type  text/html;
    charset utf-8;

    upstream tomcat {
        server myTomcat:8080;
    }

    server {
        listen       80 default_server;
        listen       [::]:80 default_server;
        server_name  _;
        root         /usr/share/nginx/html;
        include /etc/nginx/default.d/*.conf;

        location / {   
    }

    location /app01 {
        proxy_pass http://tomcat;
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Real-IP $remote_addr;    
    }

        error_page 404 /404.html;             
        location = /40x.html {
        }
        error_page 500 502 503 504 /50x.html;
        location = /50x.html {
        }
    }
}
[root@docker102.yinzhengjie.org.cn ~]# 
[root@docker102.yinzhengjie.org.cn ~]# 
```

\[root@docker102.yinzhengjie.org.cn ~\]# cat /yinzhengjie/softwares/dockerfile/web/nginx/nginx.conf

![](https://img2018.cnblogs.com/i-beta/795254/202002/795254-20200202162736226-944353083.png)

**5>.使用docker-compose启动容器**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@docker102.yinzhengjie.org.cn ~]# docker image ls
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
[root@docker102.yinzhengjie.org.cn ~]# 
[root@docker102.yinzhengjie.org.cn ~]# docker container ls
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
[root@docker102.yinzhengjie.org.cn ~]# 
[root@docker102.yinzhengjie.org.cn ~]# 
[root@docker102.yinzhengjie.org.cn ~]# cd /yinzhengjie/data/
[root@docker102.yinzhengjie.org.cn /yinzhengjie/data]# 
[root@docker102.yinzhengjie.org.cn /yinzhengjie/data]# ll
total 4
-rw-r--r-- 1 root root 984 Feb  3 00:16 docker-compose.yml
[root@docker102.yinzhengjie.org.cn /yinzhengjie/data]# 
[root@docker102.yinzhengjie.org.cn /yinzhengjie/data]# docker-compose up -d
Pulling tomcat-app01 (docker103.yinzhengjie.org.cn/yinzhengjie/tomcat-app01:8.5.50)...
8.5.50: Pulling from yinzhengjie/tomcat-app01
ac9208207ada: Pull complete
1a93113d354a: Pull complete
f108858f6f61: Pull complete
9a2b3813e78f: Pull complete
22846f1b35ad: Pull complete
4974dcf47ca5: Pull complete
cccf3b0fa894: Pull complete
f521958217e9: Pull complete
6a72405885f1: Pull complete
3459f1899556: Pull complete
67b34fd0aea6: Pull complete
Digest: sha256:184fb625634163294e0f2fb68c40657176fff69a8bed4b0329c742cab5e7e088
Status: Downloaded newer image for docker103.yinzhengjie.org.cn/yinzhengjie/tomcat-app01:8.5.50
Pulling nginx-web (docker103.yinzhengjie.org.cn/yinzhengjie/centos-nginx:v1.14.2)...
v1.14.2: Pulling from yinzhengjie/centos-nginx
ac9208207ada: Already exists
e7b74095699d: Pull complete
36fb77d3c4ec: Pull complete
1d762a9127ad: Pull complete
Digest: sha256:5bcaca2d82b6894253d2afee30571221e1132e9a8611f70ca632f4fe8658b6fe
Status: Downloaded newer image for docker103.yinzhengjie.org.cn/yinzhengjie/centos-nginx:v1.14.2
Pulling haproxy (docker103.yinzhengjie.org.cn/yinzhengjie/centos-haproxy:v1.8.20)...
v1.8.20: Pulling from yinzhengjie/centos-haproxy
ac9208207ada: Already exists
1a93113d354a: Already exists
944626ba00dc: Pull complete
efac8e5b786f: Pull complete
44a575eed86d: Pull complete
5068d27f2e99: Pull complete
Digest: sha256:25501644249322b2b5fadf473b3aad5dafe90b5a0ac2c01e7fd815c7c2c66433
Status: Downloaded newer image for docker103.yinzhengjie.org.cn/yinzhengjie/centos-haproxy:v1.8.20
Creating myTomcat ... done
Creating myNginx  ... done
Creating myHaproxy ... done
[root@docker102.yinzhengjie.org.cn /yinzhengjie/data]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

**![](https://img2018.cnblogs.com/i-beta/795254/202002/795254-20200202163451516-375802057.png)**

**6>.验证镜像是否下载及容器是否启动成功**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@docker102.yinzhengjie.org.cn /yinzhengjie/data]# 
[root@docker102.yinzhengjie.org.cn /yinzhengjie/data]# docker container ls
CONTAINER ID        IMAGE                                                             COMMAND                  CREATED             STATUS              PORTS                                        NAMES
d57ee2d3ba5e        docker103.yinzhengjie.org.cn/yinzhengjie/centos-haproxy:v1.8.20   "/usr/bin/run_haprox…"   4 minutes ago       Up 4 minutes        0.0.0.0:80->80/tcp, 0.0.0.0:8888->8888/tcp   myHaproxy
064e1c3eafb9        docker103.yinzhengjie.org.cn/yinzhengjie/centos-nginx:v1.14.2     "nginx"                  4 minutes ago       Up 4 minutes        0.0.0.0:443->443/tcp, 0.0.0.0:8080->80/tcp   myNginx
46b233a84aa5        docker103.yinzhengjie.org.cn/yinzhengjie/tomcat-app01:8.5.50      "/yinzhengjie/softwa…"   4 minutes ago       Up 4 minutes        8443/tcp, 0.0.0.0:8081->8080/tcp             myTomcat
[root@docker102.yinzhengjie.org.cn /yinzhengjie/data]# 
[root@docker102.yinzhengjie.org.cn /yinzhengjie/data]# docker image ls
REPOSITORY                                                TAG                 IMAGE ID            CREATED             SIZE
docker103.yinzhengjie.org.cn/yinzhengjie/centos-haproxy   v1.8.20             a4825da6c2fe        4 hours ago         606MB
docker103.yinzhengjie.org.cn/yinzhengjie/centos-nginx     v1.14.2             1a8b4f68e96a        33 hours ago        449MB
docker103.yinzhengjie.org.cn/yinzhengjie/tomcat-app01     8.5.50              bf45c22f2d5b        10 days ago         983MB
[root@docker102.yinzhengjie.org.cn /yinzhengjie/data]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

**![](https://img2018.cnblogs.com/i-beta/795254/202002/795254-20200202163742305-251516242.png)**

**7>.验证服务是否生效**

**![](https://img2018.cnblogs.com/i-beta/795254/202002/795254-20200202164204394-766912150.png)**

**8>.访问haproxy的状态页**

**![](https://img2018.cnblogs.com/i-beta/795254/202002/795254-20200202165727476-1409491408.png)**

```
　　浏览器访问"http://docker102.yinzhengjie.org.cn:8888/haproxy-status"会弹出如上图所示的对话框。

　　输入咱们自定义的用户名和密码登录成功后，就可以看到haproxy状态页面,如下图所示。
```

 ![](https://img2018.cnblogs.com/i-beta/795254/202002/795254-20200202165921013-533526704.png)

当你的才华还撑不起你的野心的时候，你就应该静下心来学习。当你的能力还驾驭不了你的目标的时候，你就应该沉下心来历练。问问自己，想要怎样的人生。 欢迎加入基础架构自动化运维：598432640，大数据SRE进阶之路：959042252，DevOps进阶之路：526991186