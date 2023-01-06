　　　　　　　　　　　　**Docker Private Registry 常用组件**

　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　**作者：尹正杰**

**版权声明：原创作品，谢绝转载！否则将追究法律责任。**

**一.Docker Registry概述**

**1>.什么是registry**

```
　　Registry用于保存docker镜像，包括镜像的层次结构和元数据。　　用户可自建Registry，也可使用官方的Docker Hub。
```

**2>.docker registry 分类**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
　　Sponsor Registry:　　　　第三方的registry，供客户和Docker社区使用。　　Mirror Registry:　　　　第三方的registry，只让客户使用。　　Vendor Registry:　　　　由发布Docker镜像的供应商提供的registry。　　Private Registry:　　　　通过设有防火墙和额外的安全层的私有实体提供的registry。
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

**二.使用docker官方提供的安装包(**docker-registry**)快速构建registry仓库**

```
博主推荐阅读:Docker 官方手册（完整的可用参数列表）：
　　https://docs.docker.com/engine/reference/commandline/dockerd/#run-multiple-daemons
```

**1>.查看docker-registry软件包信息**

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node102.yinzhengjie.org.cn ~]# yum info docker-registry　　　　　　#查看官方提供的软件包相关信息
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
 * base: mirrors.huaweicloud.com
 * extras: mirrors.huaweicloud.com
 * updates: mirrors.huaweicloud.com
Available Packages
Name        : docker-registry
Arch        : x86_64
Version     : 0.9.1
Release     : 7.el7
Size        : 123 k
Repo        : extras/7/x86_64
Summary     : Registry server for Docker
URL         : https://github.com/docker/docker-registry
License     : ASL 2.0
Description : Registry server for Docker (hosting/delivering of repositories and images).

[root@node102.yinzhengjie.org.cn ~]# 
```

\[root@node102.yinzhengjie.org.cn ~\]# yum info docker-registry　　　　　　　　　　#查看官方提供的软件包相关信息

**2>.安装"**docker-registry**"软件**

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node102.yinzhengjie.org.cn ~]# yum -y install docker-registry　　　　　　#安装该软件实际上会安装一个叫"docker-distribution"的软件
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
 * base: mirrors.huaweicloud.com
 * extras: mirrors.huaweicloud.com
 * updates: mirrors.huaweicloud.com
base                                                                                                                      | 3.6 kB  00:00:00     
docker-ce-stable                                                                                                          | 3.5 kB  00:00:00     
extras                                                                                                                    | 2.9 kB  00:00:00     
updates                                                                                                                   | 2.9 kB  00:00:00     
(1/2): updates/7/x86_64/primary_db                                                                                        | 2.8 MB  00:00:00     
(2/2): docker-ce-stable/x86_64/primary_db                                                                                 |  37 kB  00:00:06     
Package docker-registry is obsoleted by docker-distribution, trying to install docker-distribution-2.6.2-2.git48294d9.el7.x86_64 instead
Resolving Dependencies
--> Running transaction check
---> Package docker-distribution.x86_64 0:2.6.2-2.git48294d9.el7 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

=================================================================================================================================================
 Package                                 Arch                       Version                                     Repository                  Size
=================================================================================================================================================
Installing:
 docker-distribution                     x86_64                     2.6.2-2.git48294d9.el7                      extras                     3.5 M

Transaction Summary
=================================================================================================================================================
Install  1 Package

Total download size: 3.5 M
Installed size: 12 M
Downloading packages:
docker-distribution-2.6.2-2.git48294d9.el7.x86_64.rpm                                                                     | 3.5 MB  00:00:00     
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : docker-distribution-2.6.2-2.git48294d9.el7.x86_64                                                                             1/1 
  Verifying  : docker-distribution-2.6.2-2.git48294d9.el7.x86_64                                                                             1/1 

Installed:
  docker-distribution.x86_64 0:2.6.2-2.git48294d9.el7                                                                                            

Complete!
[root@node102.yinzhengjie.org.cn ~]# 
```

\[root@node102.yinzhengjie.org.cn ~\]# yum -y install docker-registry　　　　　　#安装该软件实际上会安装一个叫"docker-distribution"的软件

**3>.查看"docker-distribution"软件包安装信息**

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node102.yinzhengjie.org.cn ~]# rpm -ql docker-distribution
/etc/docker-distribution/registry/config.yml　　　　　　　　　　　　#该文件为配置文件
/usr/bin/registry　　　　　　　　　　　　　　　　　　　　　　　　　　
/usr/lib/systemd/system/docker-distribution.service
/usr/share/doc/docker-distribution-2.6.2
/usr/share/doc/docker-distribution-2.6.2/AUTHORS
/usr/share/doc/docker-distribution-2.6.2/CONTRIBUTING.md
/usr/share/doc/docker-distribution-2.6.2/LICENSE
/usr/share/doc/docker-distribution-2.6.2/MAINTAINERS
/usr/share/doc/docker-distribution-2.6.2/README.md
/var/lib/registry　　　　　　　　　　　　　　　　　　　　　　　　　　　#默认的镜像存储路径，建议生产环境使用一个较大的目录，可通过上面的配置文件进行修改。　　　
[root@node102.yinzhengjie.org.cn ~]# 
```

\[root@node102.yinzhengjie.org.cn ~\]# rpm -ql docker-distribution

**4>.启动"docker-distribution"服务**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@node102.yinzhengjie.org.cn ~]# systemctl start docker-distribution
[root@node102.yinzhengjie.org.cn ~]# 
[root@node102.yinzhengjie.org.cn ~]# ss -ntl
State       Recv-Q Send-Q                           Local Address:Port                                          Peer Address:Port              
LISTEN      0      128                                          *:22                                                       *:*                  
LISTEN      0      128                                         :::5000                                                    :::*                  
LISTEN      0      128                                         :::22                                                      :::*                  
[root@node102.yinzhengjie.org.cn ~]# 
[root@node102.yinzhengjie.org.cn ~]# ll /var/lib/registry/　　　　　　#由于我们还没有上传任何镜像，因此默认该目录是空的
total 0
[root@node102.yinzhengjie.org.cn ~]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

**5>.客户端上传镜像到自建的registry仓库**

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node101.yinzhengjie.org.cn ~]# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
test                v0.1-1              d2003d6841b7        4 hours ago         16.1MB
myweb               v0.3-11             c0a33ce27a6b        4 hours ago         16MB
myweb               v0.3-10             fb0ef0b3aea3        4 hours ago         16MB
myweb               v0.3-9              6f9999d46b69        4 hours ago         16MB
myweb               v0.3-8              0110cfd06ce1        4 hours ago         16MB
myweb               v0.3-7              cb08b09cd959        5 hours ago         16MB
myweb               v0.3-6              0082820cf7f0        9 hours ago         16MB
myweb               v0.3-5              fb4f5a5ac1ba        9 hours ago         16MB
myweb               v0.3-4              7697fbbe8233        9 hours ago         16MB
myweb               v0.3-3              b512ab335e80        9 hours ago         16MB
myweb               v0.3-2              6162cd671ca9        9 hours ago         16MB
myweb               v0.3-1              f14829ada08d        10 hours ago        16MB
tinyhttpd           v0.2-3              c06eb08ea6d6        37 hours ago        1.22MB
tinyhttpd           v0.2-2              c7447ee49e2b        37 hours ago        1.22MB
tinyhttpd           v0.2-1              807230671730        38 hours ago        1.22MB
tinyhttpd           v0.1-9              24c887b252ad        2 days ago          8.49MB
tinyhttpd           v0.1-8              42c71beb05d5        2 days ago          7.45MB
tinyhttpd           v0.1-7              9a5668f5f3a3        2 days ago          7.45MB
tinyhttpd           v0.1-6              2368b7546664        2 days ago          7.45MB
tinyhttpd           v0.1-5              f6f58ddb1520        2 days ago          7.45MB
tinyhttpd           v0.1-4              5086cc9cdc4a        2 days ago          7.45MB
tinyhttpd           v0.1-3              aa780396b2a3        2 days ago          2.27MB
tinyhttpd           v0.1-2              de390df4c4c3        2 days ago          1.23MB
tinyhttpd           v0.1-1              794f641ae85e        2 days ago          1.22MB
redis               latest              de25a81a5a0b        5 days ago          98.2MB
busybox             latest              19485c79a9bb        6 weeks ago         1.22MB
nginx               1.14-alpine         8a2fb25a19f5        6 months ago        16MB
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# docker tag myweb:v0.3-11 node102.yinzhengjie.org.cn:5000/myweb:v0.3-11　　　　　　　　#为客户端已有镜像打tag，便于上传到自建仓库
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# docker images
REPOSITORY                              TAG                 IMAGE ID            CREATED             SIZE
test                                    v0.1-1              d2003d6841b7        4 hours ago         16.1MB
myweb                                   v0.3-11             c0a33ce27a6b        4 hours ago         16MB
node102.yinzhengjie.org.cn:5000/myweb   v0.3-11             c0a33ce27a6b        4 hours ago         16MB
myweb                                   v0.3-10             fb0ef0b3aea3        4 hours ago         16MB
myweb                                   v0.3-9              6f9999d46b69        4 hours ago         16MB
myweb                                   v0.3-8              0110cfd06ce1        4 hours ago         16MB
myweb                                   v0.3-7              cb08b09cd959        5 hours ago         16MB
myweb                                   v0.3-6              0082820cf7f0        9 hours ago         16MB
myweb                                   v0.3-5              fb4f5a5ac1ba        9 hours ago         16MB
myweb                                   v0.3-4              7697fbbe8233        9 hours ago         16MB
myweb                                   v0.3-3              b512ab335e80        9 hours ago         16MB
myweb                                   v0.3-2              6162cd671ca9        10 hours ago        16MB
myweb                                   v0.3-1              f14829ada08d        10 hours ago        16MB
tinyhttpd                               v0.2-3              c06eb08ea6d6        37 hours ago        1.22MB
tinyhttpd                               v0.2-2              c7447ee49e2b        37 hours ago        1.22MB
tinyhttpd                               v0.2-1              807230671730        38 hours ago        1.22MB
tinyhttpd                               v0.1-9              24c887b252ad        2 days ago          8.49MB
tinyhttpd                               v0.1-8              42c71beb05d5        2 days ago          7.45MB
tinyhttpd                               v0.1-7              9a5668f5f3a3        2 days ago          7.45MB
tinyhttpd                               v0.1-6              2368b7546664        2 days ago          7.45MB
tinyhttpd                               v0.1-5              f6f58ddb1520        2 days ago          7.45MB
tinyhttpd                               v0.1-4              5086cc9cdc4a        2 days ago          7.45MB
tinyhttpd                               v0.1-3              aa780396b2a3        2 days ago          2.27MB
tinyhttpd                               v0.1-2              de390df4c4c3        2 days ago          1.23MB
tinyhttpd                               v0.1-1              794f641ae85e        2 days ago          1.22MB
redis                                   latest              de25a81a5a0b        5 days ago          98.2MB
busybox                                 latest              19485c79a9bb        6 weeks ago         1.22MB
nginx                                   1.14-alpine         8a2fb25a19f5        6 months ago        16MB
[root@node101.yinzhengjie.org.cn ~]# 
```

\[root@node101.yinzhengjie.org.cn ~\]# docker tag myweb:v0.3-11 node102.yinzhengjie.org.cn:5000/myweb:v0.3-11　　　　　　　　#为客户端已有镜像打tag，便于上传到自建仓库

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node101.yinzhengjie.org.cn ~]# docker push node102.yinzhengjie.org.cn:5000/myweb:v0.3-11　　　　　　　　　　#将已经打tag的镜像上传到自建仓库中失败，原因是默认使用HTTPS协议，而我们使用的是HTTP协议。
The push refers to repository [node102.yinzhengjie.org.cn:5000/myweb]
Get https://node102.yinzhengjie.org.cn:5000/v2/: http: server gave HTTP response to HTTPS client
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# vi /etc/docker/daemon.json 
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# cat /etc/docker/daemon.json 　　　　　　　　　　#我们在客户端配置"insecure-registries"参数。让该节点允许基于HTTP协议进行上传。
{
  "registry-mirrors": ["https://tuv7rqqq.mirror.aliyuncs.com"],
  "bip":"192.168.100.254/24",
  "dns":["219.141.139.10","219.141.140.10"],
  "hosts":["tcp://0.0.0.0:8888","unix:///var/run/docker.sock"],
  "insecure-registries":["node102.yinzhengjie.org.cn:5000"]
}
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# systemctl restart docker
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# docker push node102.yinzhengjie.org.cn:5000/myweb:v0.3-11　　　　　　#将自定义镜像上传到自建的私有Registry中。
The push refers to repository [node102.yinzhengjie.org.cn:5000/myweb]
a7487ac35bc0: Pushed 
edfefde38548: Pushed 
076c58d2644f: Pushed 
b2cbae4b8c15: Pushed 
5ac9a5170bf2: Pushed 
a464c54f93a9: Pushed 
v0.3-11: digest: sha256:aafa903875c7f59f3f3f615097d292994ac5581a404bfe2f3cb77bf9389a9855 size: 1568
[root@node101.yinzhengjie.org.cn ~]# 
```

\[root@node101.yinzhengjie.org.cn ~\]# docker push node102.yinzhengjie.org.cn:5000/myweb:v0.3-11　　　　　　#将自定义镜像上传到自建的私有Registry中。

**6>.查看服务端的镜像存储目录，发现的确由新目录生成**

```
[root@node102.yinzhengjie.org.cn ~]# ll /var/lib/registry
total 0
drwxr-xr-x 3 root root 22 Oct 22 20:51 docker
[root@node102.yinzhengjie.org.cn ~]# 
```

**7>.使用客户端从自建的镜像仓库拉取镜像**

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node103.yinzhengjie.org.cn ~]# docker images　　　　　　　　#当前节点本地还未有镜像
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
[root@node103.yinzhengjie.org.cn ~]# 
[root@node103.yinzhengjie.org.cn ~]# docker pull node102.yinzhengjie.org.cn:5000/myweb:v0.3-11　　　　　　#下载私有仓库镜像失败，原因相比大家也知道了，继续参考上面的解决办法。
Error response from daemon: Get https://node102.yinzhengjie.org.cn:5000/v2/: http: server gave HTTP response to HTTPS client
[root@node103.yinzhengjie.org.cn ~]# 
[root@node103.yinzhengjie.org.cn ~]# 
[root@node103.yinzhengjie.org.cn ~]# vi /etc/docker/daemon.json 
[root@node103.yinzhengjie.org.cn ~]# 
[root@node103.yinzhengjie.org.cn ~]# cat /etc/docker/daemon.json 　　　　#我们可以理解"insecure-registries"将node102.yinzhengjie.org.cn:5000这个节点关闭安全检查
{
  "registry-mirrors": ["https://tuv7rqqq.mirror.aliyuncs.com"],
  "insecure-registries":["node102.yinzhengjie.org.cn:5000"]
}
[root@node103.yinzhengjie.org.cn ~]# 
[root@node103.yinzhengjie.org.cn ~]# systemctl restart docker
[root@node103.yinzhengjie.org.cn ~]# 
[root@node103.yinzhengjie.org.cn ~]# docker pull node102.yinzhengjie.org.cn:5000/myweb:v0.3-11　　　　　　#下载自建私有仓库的镜像到本地
v0.3-11: Pulling from myweb
bdf0201b3a05: Pull complete 
3d0a573c81ed: Pull complete 
8129faeb2eb6: Pull complete 
3dc99f571daf: Pull complete 
41addc55f022: Pull complete 
ea60c368e85c: Pull complete 
Digest: sha256:aafa903875c7f59f3f3f615097d292994ac5581a404bfe2f3cb77bf9389a9855
Status: Downloaded newer image for node102.yinzhengjie.org.cn:5000/myweb:v0.3-11
node102.yinzhengjie.org.cn:5000/myweb:v0.3-11
[root@node103.yinzhengjie.org.cn ~]# 
[root@node103.yinzhengjie.org.cn ~]# docker image ls
REPOSITORY                              TAG                 IMAGE ID            CREATED             SIZE
node102.yinzhengjie.org.cn:5000/myweb   v0.3-11             c0a33ce27a6b        4 hours ago         16MB
[root@node103.yinzhengjie.org.cn ~]# 
[root@node103.yinzhengjie.org.cn ~]# docker images
REPOSITORY                              TAG                 IMAGE ID            CREATED             SIZE
node102.yinzhengjie.org.cn:5000/myweb   v0.3-11             c0a33ce27a6b        4 hours ago         16MB
[root@node103.yinzhengjie.org.cn ~]# 
[root@node103.yinzhengjie.org.cn ~]# 
```

\[root@node103.yinzhengjie.org.cn ~\]# docker pull node102.yinzhengjie.org.cn:5000/myweb:v0.3-11　　　　　　#下载自建私有仓库的镜像到本地

**三.Harbor私有仓库**

**1>.什么是Harbor**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
　　Project Harbor是一个开源的云原生registry项目，它支持存储、签名和扫描(判定镜像是否有安全风险)内容;

　　Harbor通过添加用户通常需要的功能（如安全、身份和管理）扩展了开源Docker发行版;

　　Harbor支持高级功能，如用户管理、访问控制、活动监视和实例之间的复制。

　　官方地址:https://goharbor.io
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

**2>.Harbor特性**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
　　Multi-tenant content signing and validation(支持多租户内容签名和验证);　　Security and vulnerability analysis(支持安全及风险分析);　　Audit logging(支持审计日志);　　Identity integration and role-based access contron(支持身份整合与基于角色的访问控制);　　Image replication between instances(支持实例之间的映像复制);　　Extensible API and graphical UI(支持可扩展API和图形UI);　　Internationalization(currently English and Chinese,支持国际化，目前仅支持英文和中文)。
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

**3>.Harbor安装指南**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
　　在物理机上Harbor的安装和部署是相当困难的，因此为了简化Harbor的应用，Harbor官方直接将Harbor做成了直接在容器中运行的应用;

　　由于Harbor容器依赖于redis，MySQL/PgSQL等很多存储系统，所以需要编排多个容器来协同工作，因此Vmware Harbor在部署和使用时需要借助于docker的单机编排工具,即Docker Compose(compose是一个定义和运行多容器docker的工具，官方地址:https://docs.docker.com/compose/)。
　　
　　Harbor的GitHub地址:https://github.com/goharbor/harbor

　　Harhor的安装指南:https://github.com/goharbor/harbor/blob/master/docs/installation_guide.md
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node103.yinzhengjie.org.cn ~]# yum info docker-compose　　　　　　　　　　#compose时一个定义和运行多容器docker的工具。
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
epel/x86_64/metalink                                                                                                      | 8.0 kB  00:00:00     
 * base: mirrors.nju.edu.cn
 * epel: hkg.mirror.rackspace.com
 * extras: mirrors.huaweicloud.com
 * updates: mirrors.nju.edu.cn
epel                                                                                                                      | 5.3 kB  00:00:00     
(1/3): epel/x86_64/group_gz                                                                                               |  90 kB  00:00:06     
(2/3): epel/x86_64/updateinfo                                                                                             | 1.0 MB  00:00:06     
(3/3): epel/x86_64/primary_db                                                                                             | 6.9 MB  00:00:08     
Available Packages
Name        : docker-compose
Arch        : noarch
Version     : 1.18.0
Release     : 4.el7
Size        : 222 k
Repo        : epel/x86_64
Summary     : Multi-container orchestration for Docker
URL         : https://github.com/docker/compose
License     : ASL 2.0
Description : Compose is a tool for defining and running multi-container Docker
            : applications. With Compose, you use a Compose file to configure your
            : application's services. Then, using a single command, you create and
            : start all the services from your configuration.
            : 
            : Compose is great for development, testing, and staging environments,
            : as well as CI workflows.
            : 
            : Using Compose is basically a three-step process.
            : 
            : 1. Define your app's environment with a Dockerfile so it can be
            :    reproduced anywhere.
            : 2. Define the services that make up your app in docker-compose.yml so
            :    they can be run together in an isolated environment:
            : 3. Lastly, run docker-compose up and Compose will start and run your
            :    entire app.

[root@node103.yinzhengjie.org.cn ~]# 
```

\[root@node103.yinzhengjie.org.cn ~\]# yum info docker-compose　　　　　　　　　　#compose时一个定义和运行多容器docker的工具。

**4>.下载Harbor软件包（[https://github.com/goharbor/harbor/releases](https://github.com/goharbor/harbor/releases)）**

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node103.yinzhengjie.org.cn ~]# wget https://storage.googleapis.com/harbor-releases/release-1.9.0/harbor-offline-installer-v1.9.1.tgz
--2019-10-22 23:30:23--  https://storage.googleapis.com/harbor-releases/release-1.9.0/harbor-offline-installer-v1.9.1.tgz
Resolving storage.googleapis.com (storage.googleapis.com)... 172.217.160.80, 2404:6800:4012::2010
Connecting to storage.googleapis.com (storage.googleapis.com)|172.217.160.80|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 619113214 (590M) [application/x-tar]
Saving to: ‘harbor-offline-installer-v1.9.1.tgz’

100%[=======================================================================================================>] 619,113,214 9.49MB/s   in 70s    

2019-10-22 23:31:39 (8.44 MB/s) - ‘harbor-offline-installer-v1.9.1.tgz’ saved [619113214/619113214]

[root@node103.yinzhengjie.org.cn ~]# 
```

\[root@node103.yinzhengjie.org.cn ~\]# wget https://storage.googleapis.com/harbor-releases/release-1.9.0/harbor-offline-installer-v1.9.1.tgz

![](https://img2018.cnblogs.com/blog/795254/201910/795254-20191022235856947-1767388079.png)

**5>.安装Harbor软件**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@node103.yinzhengjie.org.cn ~]# ll
total 604604
-rw-r--r-- 1 root root 619113214 Oct 15 15:47 harbor-offline-installer-v1.9.1.tgz
[root@node103.yinzhengjie.org.cn ~]# 
[root@node103.yinzhengjie.org.cn ~]# tar -xf harbor-offline-installer-v1.9.1.tgz -C /usr/local/
[root@node103.yinzhengjie.org.cn ~]# 
[root@node103.yinzhengjie.org.cn ~]# cd /usr/local/harbor/
[root@node103.yinzhengjie.org.cn /usr/local/harbor]# 
[root@node103.yinzhengjie.org.cn /usr/local/harbor]# ll
total 607872
-rw-r--r-- 1 root root 622428100 Sep 27 14:52 harbor.v1.9.1.tar.gz
-rw-r--r-- 1 root root      5805 Sep 27 14:52 harbor.yml　　　　　　　　#配置文件
-rwxr-xr-x 1 root root      5088 Sep 27 14:52 install.sh　　　　　　　　#安装脚本 
-rw-r--r-- 1 root root     11347 Sep 27 14:52 LICENSE
-rwxr-xr-x 1 root root      1748 Sep 27 14:52 prepare
[root@node103.yinzhengjie.org.cn /usr/local/harbor]# 
[root@node103.yinzhengjie.org.cn /usr/local/harbor]#
[root@node103.yinzhengjie.org.cn /usr/local/harbor]# grep hostname harbor.yml  | grep -v ^#　　　　　　#此处我们可以修改一下主机名，打开该配置文件里面有相应的英文注释
hostname: node103.yinzhengjie.org.cn
[root@node103.yinzhengjie.org.cn /usr/local/harbor]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node103.yinzhengjie.org.cn ~]# yum -y install docker-compose　　　　　　　　　　　　#由于docker-compose是由python 3开发的，因此在安装该软件时我们会发现它会自动安装一些python模块。
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
 * base: mirrors.nju.edu.cn
 * epel: hkg.mirror.rackspace.com
 * extras: mirrors.huaweicloud.com
 * updates: mirrors.nju.edu.cn
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
---> Package python36-requests.noarch 0:2.12.5-3.el7 will be installed
--> Processing Dependency: python36-urllib3 = 1.19.1 for package: python36-requests-2.12.5-3.el7.noarch
--> Processing Dependency: python36-chardet for package: python36-requests-2.12.5-3.el7.noarch
--> Processing Dependency: python36-idna for package: python36-requests-2.12.5-3.el7.noarch
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
---> Package python36-urllib3.noarch 0:1.19.1-5.el7 will be installed
--> Running transaction check
---> Package libtirpc.x86_64 0:0.2.4-0.16.el7 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

=================================================================================================================================================
 Package                                        Arch                        Version                              Repository                 Size
=================================================================================================================================================
Installing:
 docker-compose                                 noarch                      1.18.0-4.el7                         epel                      222 k
Installing for dependencies:
 libtirpc                                       x86_64                      0.2.4-0.16.el7                       base                       89 k
 libyaml                                        x86_64                      0.1.4-11.el7_0                       base                       55 k
 python3                                        x86_64                      3.6.8-10.el7                         base                       69 k
 python3-libs                                   x86_64                      3.6.8-10.el7                         base                      7.0 M
 python3-pip                                    noarch                      9.0.3-5.el7                          base                      1.8 M
 python3-setuptools                             noarch                      39.2.0-10.el7                        base                      629 k
 python36-PyYAML                                x86_64                      3.12-1.el7                           epel                      149 k
 python36-cached_property                       noarch                      1.5.1-2.el7                          epel                       18 k
 python36-chardet                               noarch                      3.0.4-1.el7                          epel                      190 k
 python36-docker                                noarch                      2.6.1-3.el7                          epel                      180 k
 python36-docker-pycreds                        noarch                      0.2.1-2.el7                          epel                       15 k
 python36-dockerpty                             noarch                      0.4.1-10.el7                         epel                       29 k
 python36-docopt                                noarch                      0.6.2-8.el7                          epel                       29 k
 python36-idna                                  noarch                      2.7-2.el7                            epel                       98 k
 python36-jsonschema                            noarch                      2.5.1-4.el7                          epel                       76 k
 python36-pysocks                               noarch                      1.6.8-6.el7                          epel                       30 k
 python36-requests                              noarch                      2.12.5-3.el7                         epel                      109 k
 python36-six                                   noarch                      1.11.0-3.el7                         epel                       33 k
 python36-texttable                             noarch                      1.6.2-1.el7                          epel                       23 k
 python36-urllib3                               noarch                      1.19.1-5.el7                         epel                      134 k
 python36-websocket-client                      noarch                      0.47.0-2.el7                         epel                       59 k

Transaction Summary
=================================================================================================================================================
Install  1 Package (+21 Dependent packages)

Total download size: 11 M
Installed size: 55 M
Downloading packages:
(1/22): libtirpc-0.2.4-0.16.el7.x86_64.rpm                                                                                |  89 kB  00:00:00     
(2/22): python3-libs-3.6.8-10.el7.x86_64.rpm                                                                              | 7.0 MB  00:00:01     
(3/22): python3-setuptools-39.2.0-10.el7.noarch.rpm                                                                       | 629 kB  00:00:00     
(4/22): python3-pip-9.0.3-5.el7.noarch.rpm                                                                                | 1.8 MB  00:00:01     
(5/22): python36-PyYAML-3.12-1.el7.x86_64.rpm                                                                             | 149 kB  00:00:02     
(6/22): libyaml-0.1.4-11.el7_0.x86_64.rpm                                                                                 |  55 kB  00:00:05     
(7/22): docker-compose-1.18.0-4.el7.noarch.rpm                                                                            | 222 kB  00:00:07     
(8/22): python36-chardet-3.0.4-1.el7.noarch.rpm                                                                           | 190 kB  00:00:00     
(9/22): python36-docker-2.6.1-3.el7.noarch.rpm                                                                            | 180 kB  00:00:00     
(10/22): python3-3.6.8-10.el7.x86_64.rpm                                                                                  |  69 kB  00:00:12     
(11/22): python36-cached_property-1.5.1-2.el7.noarch.rpm                                                                  |  18 kB  00:00:05     
(12/22): python36-docker-pycreds-0.2.1-2.el7.noarch.rpm                                                                   |  15 kB  00:00:00     
(13/22): python36-dockerpty-0.4.1-10.el7.noarch.rpm                                                                       |  29 kB  00:00:00     
(14/22): python36-docopt-0.6.2-8.el7.noarch.rpm                                                                           |  29 kB  00:00:00     
python36-idna-2.7-2.el7.noarch FAILED                                          
http://my.mirrors.thegigabit.com/epel/7/x86_64/Packages/p/python36-idna-2.7-2.el7.noarch.rpm: [Errno 12] Timeout on http://103.238.48.2/my.mirror
s.thegigabit.com/epel/7/x86_64/Packages/p/python36-idna-2.7-2.el7.noarch.rpm: (28, 'Connection timed out after 30997 milliseconds')Trying other mirror.
(15/22): python36-pysocks-1.6.8-6.el7.noarch.rpm                                                                          |  30 kB  00:00:01     
(16/22): python36-requests-2.12.5-3.el7.noarch.rpm                                                                        | 109 kB  00:00:00     
(17/22): python36-six-1.11.0-3.el7.noarch.rpm                                                                             |  33 kB  00:00:21     
(18/22): python36-texttable-1.6.2-1.el7.noarch.rpm                                                                        |  23 kB  00:00:05     
python36-jsonschema-2.5.1-4.el FAILED                                          
http://my.mirrors.thegigabit.com/epel/7/x86_64/Packages/p/python36-jsonschema-2.5.1-4.el7.noarch.rpm: [Errno 12] Timeout on http://103.238.48.2/m
y.mirrors.thegigabit.com/epel/7/x86_64/Packages/p/python36-jsonschema-2.5.1-4.el7.noarch.rpm: (28, 'Connection timed out after 30184 milliseconds')Trying other mirror.
(19/22): python36-urllib3-1.19.1-5.el7.noarch.rpm                                                                         | 134 kB  00:00:03     
(20/22): python36-websocket-client-0.47.0-2.el7.noarch.rpm                                                                |  59 kB  00:00:00     
(21/22): python36-idna-2.7-2.el7.noarch.rpm                                                                               |  98 kB  00:00:08     
(22/22): python36-jsonschema-2.5.1-4.el7.noarch.rpm                                                                       |  76 kB  00:00:11     
-------------------------------------------------------------------------------------------------------------------------------------------------
Total                                                                                                            114 kB/s |  11 MB  00:01:37     
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : libyaml-0.1.4-11.el7_0.x86_64                                                                                                1/22 
  Installing : libtirpc-0.2.4-0.16.el7.x86_64                                                                                               2/22 
  Installing : python3-pip-9.0.3-5.el7.noarch                                                                                               3/22 
  Installing : python3-setuptools-39.2.0-10.el7.noarch                                                                                      4/22 
  Installing : python3-3.6.8-10.el7.x86_64                                                                                                  5/22 
  Installing : python3-libs-3.6.8-10.el7.x86_64                                                                                             6/22 
  Installing : python36-six-1.11.0-3.el7.noarch                                                                                             7/22 
  Installing : python36-websocket-client-0.47.0-2.el7.noarch                                                                                8/22 
  Installing : python36-pysocks-1.6.8-6.el7.noarch                                                                                          9/22 
  Installing : python36-urllib3-1.19.1-5.el7.noarch                                                                                        10/22 
  Installing : python36-dockerpty-0.4.1-10.el7.noarch                                                                                      11/22 
  Installing : python36-docker-pycreds-0.2.1-2.el7.noarch                                                                                  12/22 
  Installing : python36-PyYAML-3.12-1.el7.x86_64                                                                                           13/22 
  Installing : python36-texttable-1.6.2-1.el7.noarch                                                                                       14/22 
  Installing : python36-jsonschema-2.5.1-4.el7.noarch                                                                                      15/22 
  Installing : python36-idna-2.7-2.el7.noarch                                                                                              16/22 
  Installing : python36-docopt-0.6.2-8.el7.noarch                                                                                          17/22 
  Installing : python36-cached_property-1.5.1-2.el7.noarch                                                                                 18/22 
  Installing : python36-chardet-3.0.4-1.el7.noarch                                                                                         19/22 
  Installing : python36-requests-2.12.5-3.el7.noarch                                                                                       20/22 
  Installing : python36-docker-2.6.1-3.el7.noarch                                                                                          21/22 
  Installing : docker-compose-1.18.0-4.el7.noarch                                                                                          22/22 
  Verifying  : libtirpc-0.2.4-0.16.el7.x86_64                                                                                               1/22 
  Verifying  : python36-pysocks-1.6.8-6.el7.noarch                                                                                          2/22 
  Verifying  : python3-libs-3.6.8-10.el7.x86_64                                                                                             3/22 
  Verifying  : docker-compose-1.18.0-4.el7.noarch                                                                                           4/22 
  Verifying  : python36-urllib3-1.19.1-5.el7.noarch                                                                                         5/22 
  Verifying  : python3-pip-9.0.3-5.el7.noarch                                                                                               6/22 
  Verifying  : python36-texttable-1.6.2-1.el7.noarch                                                                                        7/22 
  Verifying  : python36-jsonschema-2.5.1-4.el7.noarch                                                                                       8/22 
  Verifying  : python36-idna-2.7-2.el7.noarch                                                                                               9/22 
  Verifying  : python36-websocket-client-0.47.0-2.el7.noarch                                                                               10/22 
  Verifying  : python36-PyYAML-3.12-1.el7.x86_64                                                                                           11/22 
  Verifying  : python36-dockerpty-0.4.1-10.el7.noarch                                                                                      12/22 
  Verifying  : python36-docker-2.6.1-3.el7.noarch                                                                                          13/22 
  Verifying  : python36-six-1.11.0-3.el7.noarch                                                                                            14/22 
  Verifying  : python3-setuptools-39.2.0-10.el7.noarch                                                                                     15/22 
  Verifying  : python36-docopt-0.6.2-8.el7.noarch                                                                                          16/22 
  Verifying  : python36-cached_property-1.5.1-2.el7.noarch                                                                                 17/22 
  Verifying  : python3-3.6.8-10.el7.x86_64                                                                                                 18/22 
  Verifying  : libyaml-0.1.4-11.el7_0.x86_64                                                                                               19/22 
  Verifying  : python36-chardet-3.0.4-1.el7.noarch                                                                                         20/22 
  Verifying  : python36-docker-pycreds-0.2.1-2.el7.noarch                                                                                  21/22 
  Verifying  : python36-requests-2.12.5-3.el7.noarch                                                                                       22/22 

Installed:
  docker-compose.noarch 0:1.18.0-4.el7                                                                                                           

Dependency Installed:
  libtirpc.x86_64 0:0.2.4-0.16.el7           libyaml.x86_64 0:0.1.4-11.el7_0                  python3.x86_64 0:3.6.8-10.el7                     
  python3-libs.x86_64 0:3.6.8-10.el7         python3-pip.noarch 0:9.0.3-5.el7                 python3-setuptools.noarch 0:39.2.0-10.el7         
  python36-PyYAML.x86_64 0:3.12-1.el7        python36-cached_property.noarch 0:1.5.1-2.el7    python36-chardet.noarch 0:3.0.4-1.el7             
  python36-docker.noarch 0:2.6.1-3.el7       python36-docker-pycreds.noarch 0:0.2.1-2.el7     python36-dockerpty.noarch 0:0.4.1-10.el7          
  python36-docopt.noarch 0:0.6.2-8.el7       python36-idna.noarch 0:2.7-2.el7                 python36-jsonschema.noarch 0:2.5.1-4.el7          
  python36-pysocks.noarch 0:1.6.8-6.el7      python36-requests.noarch 0:2.12.5-3.el7          python36-six.noarch 0:1.11.0-3.el7                
  python36-texttable.noarch 0:1.6.2-1.el7    python36-urllib3.noarch 0:1.19.1-5.el7           python36-websocket-client.noarch 0:0.47.0-2.el7   

Complete!
[root@node103.yinzhengjie.org.cn ~]# 
```

\[root@node103.yinzhengjie.org.cn ~\]# yum -y install docker-compose　　　　　　　#由于docker-compose是由python 3开发的，因此在安装该软件时我们会发现它会自动安装一些python模块。

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node103.yinzhengjie.org.cn /usr/local/harbor]# ll
total 607872
-rw-r--r-- 1 root root 622428100 Sep 27 14:52 harbor.v1.9.1.tar.gz
-rw-r--r-- 1 root root      5815 Oct 22 23:40 harbor.yml
-rwxr-xr-x 1 root root      5088 Sep 27 14:52 install.sh
-rw-r--r-- 1 root root     11347 Sep 27 14:52 LICENSE
-rwxr-xr-x 1 root root      1748 Sep 27 14:52 prepare
[root@node103.yinzhengjie.org.cn /usr/local/harbor]# 
[root@node103.yinzhengjie.org.cn /usr/local/harbor]# ./install.sh 　　　　　　　　#安装Harbor软件，安装完成后它会自动启动相关web服务

[Step 0]: checking installation environment ...

Note: docker version: 19.03.3

Note: docker-compose version: 1.18.0

[Step 1]: loading Harbor images ...
b80136ee24a4: Loading layer [==================================================>]  34.25MB/34.25MB
cad87ea2da29: Loading layer [==================================================>]  77.02MB/77.02MB
034ded39ed39: Loading layer [==================================================>]  3.072kB/3.072kB
f6ca716ef169: Loading layer [==================================================>]   59.9kB/59.9kB
baf21a4a14d3: Loading layer [==================================================>]  61.95kB/61.95kB
Loaded image: goharbor/redis-photon:v1.9.1
38d0cc9d1ffd: Loading layer [==================================================>]   8.98MB/8.98MB
f68a1f0c31fc: Loading layer [==================================================>]  3.072kB/3.072kB
72515108750d: Loading layer [==================================================>]   2.56kB/2.56kB
a2cda355c3ef: Loading layer [==================================================>]   20.1MB/20.1MB
ab2865eb128e: Loading layer [==================================================>]   20.1MB/20.1MB
Loaded image: goharbor/registry-photon:v2.7.1-patch-2819-2553-v1.9.1
527ef66806e1: Loading layer [==================================================>]   8.98MB/8.98MB
0f11d90dd35b: Loading layer [==================================================>]  3.072kB/3.072kB
3425f8898e4f: Loading layer [==================================================>]   20.1MB/20.1MB
cfa38640b856: Loading layer [==================================================>]  3.072kB/3.072kB
e7794afc289f: Loading layer [==================================================>]  8.661MB/8.661MB
76012da7ad6a: Loading layer [==================================================>]  28.76MB/28.76MB
Loaded image: goharbor/harbor-registryctl:v1.9.1
e83beb288a0d: Loading layer [==================================================>]    113MB/113MB
93e08bdb2f3c: Loading layer [==================================================>]  11.17MB/11.17MB
7a823857fc40: Loading layer [==================================================>]  2.048kB/2.048kB
fc24f20df72d: Loading layer [==================================================>]  48.13kB/48.13kB
bd925688a6c6: Loading layer [==================================================>]  3.072kB/3.072kB
d0812a8a6aa1: Loading layer [==================================================>]  11.22MB/11.22MB
Loaded image: goharbor/clair-photon:v2.0.9-v1.9.1
4c42b997c1d3: Loading layer [==================================================>]  337.7MB/337.7MB
b921c11a7cce: Loading layer [==================================================>]  119.8kB/119.8kB
Loaded image: goharbor/harbor-migrator:v1.9.1
50d9ef917fd0: Loading layer [==================================================>]  8.985MB/8.985MB
49be4811c210: Loading layer [==================================================>]  44.39MB/44.39MB
e3e7d0ecbd56: Loading layer [==================================================>]  2.048kB/2.048kB
bd6ae8ad3688: Loading layer [==================================================>]  3.072kB/3.072kB
f8daafb6452b: Loading layer [==================================================>]   44.4MB/44.4MB
Loaded image: goharbor/chartmuseum-photon:v0.9.0-v1.9.1
f8be5d65a497: Loading layer [==================================================>]   2.56kB/2.56kB
ec2558a18995: Loading layer [==================================================>]  1.536kB/1.536kB
9253b305f3e0: Loading layer [==================================================>]  72.14MB/72.14MB
f5bc3b95773b: Loading layer [==================================================>]  42.56MB/42.56MB
1cb18728da6a: Loading layer [==================================================>]  156.7kB/156.7kB
10d8c3845ea7: Loading layer [==================================================>]  3.006MB/3.006MB
Loaded image: goharbor/prepare:v1.9.1
63f870b51f41: Loading layer [==================================================>]  63.49MB/63.49MB
7306099794e1: Loading layer [==================================================>]  53.21MB/53.21MB
8a9f36b3fb5f: Loading layer [==================================================>]  5.632kB/5.632kB
75daaad9bfa8: Loading layer [==================================================>]  2.048kB/2.048kB
d8f94d1114c6: Loading layer [==================================================>]   2.56kB/2.56kB
7ecefe35a143: Loading layer [==================================================>]   2.56kB/2.56kB
a8fab8d5af16: Loading layer [==================================================>]   2.56kB/2.56kB
94cf351a3c19: Loading layer [==================================================>]  10.24kB/10.24kB
Loaded image: goharbor/harbor-db:v1.9.1
9837bf8ee24e: Loading layer [==================================================>]  8.979MB/8.979MB
ba364bbbdbdd: Loading layer [==================================================>]  6.239MB/6.239MB
3fd94bb9f201: Loading layer [==================================================>]  15.13MB/15.13MB
a71f5b58dc99: Loading layer [==================================================>]  26.47MB/26.47MB
80997da865e8: Loading layer [==================================================>]  22.02kB/22.02kB
376d829dd4a8: Loading layer [==================================================>]  47.84MB/47.84MB
Loaded image: goharbor/notary-server-photon:v0.6.1-v1.9.1
7310144c80b8: Loading layer [==================================================>]  12.75MB/12.75MB
57822721a26e: Loading layer [==================================================>]  48.13MB/48.13MB
Loaded image: goharbor/harbor-jobservice:v1.9.1
b9172b8bd1c0: Loading layer [==================================================>]  10.82MB/10.82MB
Loaded image: goharbor/nginx-photon:v1.9.1
3999c34ab3d1: Loading layer [==================================================>]  13.72MB/13.72MB
55d2a47566d6: Loading layer [==================================================>]  26.47MB/26.47MB
605a3b90e10b: Loading layer [==================================================>]  22.02kB/22.02kB
753afe3849f7: Loading layer [==================================================>]  46.43MB/46.43MB
Loaded image: goharbor/notary-signer-photon:v0.6.1-v1.9.1
f6d400c78205: Loading layer [==================================================>]  7.012MB/7.012MB
da6e265c346e: Loading layer [==================================================>]  196.6kB/196.6kB
11f6a9c90cd1: Loading layer [==================================================>]    172kB/172kB
99996a16f7b1: Loading layer [==================================================>]  15.36kB/15.36kB
d94e53aaf9b8: Loading layer [==================================================>]  3.584kB/3.584kB
b84ee50e8238: Loading layer [==================================================>]  10.82MB/10.82MB
Loaded image: goharbor/harbor-portal:v1.9.1
e755a945ae2a: Loading layer [==================================================>]  12.75MB/12.75MB
dc5d22c53956: Loading layer [==================================================>]  55.39MB/55.39MB
15f93df7269e: Loading layer [==================================================>]  5.632kB/5.632kB
ee595495168a: Loading layer [==================================================>]  36.35kB/36.35kB
8fb7538d9d82: Loading layer [==================================================>]  55.39MB/55.39MB
Loaded image: goharbor/harbor-core:v1.9.1
3e458fabaeef: Loading layer [==================================================>]  50.61MB/50.61MB
2a88a9994014: Loading layer [==================================================>]  3.584kB/3.584kB
19d8eee966bd: Loading layer [==================================================>]  3.072kB/3.072kB
bd017a9bef7c: Loading layer [==================================================>]   2.56kB/2.56kB
186ca97fbd0d: Loading layer [==================================================>]  3.072kB/3.072kB
8b15fc511dbf: Loading layer [==================================================>]  3.584kB/3.584kB
cc84783073de: Loading layer [==================================================>]  12.29kB/12.29kB
Loaded image: goharbor/harbor-log:v1.9.1


[Step 2]: preparing environment ...
prepare base dir is set to /usr/local/harbor
Generated configuration file: /config/log/logrotate.conf
Generated configuration file: /config/log/rsyslog_docker.conf
Generated configuration file: /config/nginx/nginx.conf
Generated configuration file: /config/core/env
Generated configuration file: /config/core/app.conf
Generated configuration file: /config/registry/config.yml
Generated configuration file: /config/registryctl/env
Generated configuration file: /config/db/env
Generated configuration file: /config/jobservice/env
Generated configuration file: /config/jobservice/config.yml
Generated and saved secret to file: /secret/keys/secretkey
Creating harbor-log ... done
Generated configuration file: /compose_location/docker-compose.yml
Clean up the input dir


Creating registry ... done
Creating harbor-core ... done
Creating network "harbor_harbor" with the default driver
Creating nginx ... done
Creating harbor-db ... 
Creating registry ... 
Creating redis ... 
Creating harbor-portal ... 
Creating registryctl ... 
Creating harbor-core ... 
Creating nginx ... 
Creating harbor-jobservice ... 

✔ ----Harbor has been installed and started successfully.----

Now you should be able to visit the admin portal at http://node103.yinzhengjie.org.cn. 
For more details, please visit https://github.com/goharbor/harbor .

[root@node103.yinzhengjie.org.cn /usr/local/harbor]# 
[root@node103.yinzhengjie.org.cn /usr/local/harbor]# echo $?
0
[root@node103.yinzhengjie.org.cn /usr/local/harbor]# 
[root@node103.yinzhengjie.org.cn /usr/local/harbor]# ll
total 607880
drwxr-xr-x 3 root root        20 Oct 22 23:48 common
-rw-r--r-- 1 root root      5290 Oct 22 23:48 docker-compose.yml
-rw-r--r-- 1 root root 622428100 Sep 27 14:52 harbor.v1.9.1.tar.gz
-rw-r--r-- 1 root root      5815 Oct 22 23:40 harbor.yml
-rwxr-xr-x 1 root root      5088 Sep 27 14:52 install.sh
-rw-r--r-- 1 root root     11347 Sep 27 14:52 LICENSE
-rwxr-xr-x 1 root root      1748 Sep 27 14:52 prepare
[root@node103.yinzhengjie.org.cn /usr/local/harbor]# 
```

\[root@node103.yinzhengjie.org.cn /usr/local/harbor\]# ./install.sh 　　　　　　　　#安装Harbor软件，安装完成后它会自动启动相关web服务

**6>.访问Harbor的Web UI**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@node103.yinzhengjie.org.cn /usr/local/harbor]# netstat -ntl
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State      
tcp        0      0 127.0.0.1:1514          0.0.0.0:*               LISTEN     
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN     
tcp6       0      0 :::80                   :::*                    LISTEN     
tcp6       0      0 :::22                   :::*                    LISTEN     
[root@node103.yinzhengjie.org.cn /usr/local/harbor]#
[root@node103.yinzhengjie.org.cn /usr/local/harbor]# grep harbor_admin_password harbor.yml 　　　　　　#这是Harbor服务的默认密码，如果有需要可以在安装服务前自行修改哟~
harbor_admin_password: Harbor12345
[root@node103.yinzhengjie.org.cn /usr/local/harbor]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

![](https://img2018.cnblogs.com/blog/795254/201910/795254-20191022235459645-440995829.png)

**7>.Harbor的Web UI登录成功**

**![](https://img2018.cnblogs.com/blog/795254/201910/795254-20191022235753531-793704621.png)**

**四.基于自建Harbor上传镜像案例演示(日志存放路径: "/var/log/harbor")**

**1>.使用普通用户登录**

**![](https://img2018.cnblogs.com/blog/795254/201910/795254-20191023071950721-455112555.png)**

**2>.登录成功**

![](https://img2018.cnblogs.com/blog/795254/201910/795254-20191023072252161-1276139080.png)

**3>.点击"新建项目",自定义项目名称**

![](https://img2018.cnblogs.com/blog/795254/201910/795254-20191023072401291-981160136.png)

**4>.进入新创建的项目**

![](https://img2018.cnblogs.com/blog/795254/201910/795254-20191023072441586-1768625626.png)

**5>.查看推送镜像的方式**

**![](https://img2018.cnblogs.com/blog/795254/201910/795254-20191023072624247-1336245345.png)**

**6>.将"node101.yinzhengjie.org.cn"节点的自建镜像推送到"node103.yinzhengjie.org.cn"的Harbor仓库**

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node101.yinzhengjie.org.cn ~]# cat /etc/docker/daemon.json 　　　　　　　　#关闭node103.yinzhengjie.org.cn的安全检查并重启服务
{
  "registry-mirrors": ["https://tuv7rqqq.mirror.aliyuncs.com"],
  "bip":"192.168.100.254/24",
  "dns":["219.141.139.10","219.141.140.10"],
  "hosts":["tcp://0.0.0.0:8888","unix:///var/run/docker.sock"],
  "insecure-registries":["node102.yinzhengjie.org.cn:5000"],
  "insecure-registries":["node103.yinzhengjie.org.cn:80"]
}
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# systemctl restart docker
[root@node101.yinzhengjie.org.cn ~]# 
```

\[root@node101.yinzhengjie.org.cn ~\]# cat /etc/docker/daemon.json 　　　　　　　　#关闭node103.yinzhengjie.org.cn的安全检查并重启服务

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node101.yinzhengjie.org.cn ~]# docker images
REPOSITORY                              TAG                 IMAGE ID            CREATED             SIZE
test                                    v0.1-1              d2003d6841b7        15 hours ago        16.1MB
myweb                                   v0.3-11             c0a33ce27a6b        15 hours ago        16MB
node102.yinzhengjie.org.cn:5000/myweb   v0.3-11             c0a33ce27a6b        15 hours ago        16MB
myweb                                   v0.3-10             fb0ef0b3aea3        15 hours ago        16MB
myweb                                   v0.3-9              6f9999d46b69        15 hours ago        16MB
myweb                                   v0.3-8              0110cfd06ce1        15 hours ago        16MB
myweb                                   v0.3-7              cb08b09cd959        16 hours ago        16MB
myweb                                   v0.3-6              0082820cf7f0        20 hours ago        16MB
myweb                                   v0.3-5              fb4f5a5ac1ba        20 hours ago        16MB
myweb                                   v0.3-4              7697fbbe8233        20 hours ago        16MB
myweb                                   v0.3-3              b512ab335e80        20 hours ago        16MB
myweb                                   v0.3-2              6162cd671ca9        20 hours ago        16MB
myweb                                   v0.3-1              f14829ada08d        20 hours ago        16MB
tinyhttpd                               v0.2-3              c06eb08ea6d6        47 hours ago        1.22MB
tinyhttpd                               v0.2-2              c7447ee49e2b        2 days ago          1.22MB
tinyhttpd                               v0.2-1              807230671730        2 days ago          1.22MB
tinyhttpd                               v0.1-9              24c887b252ad        2 days ago          8.49MB
tinyhttpd                               v0.1-8              42c71beb05d5        2 days ago          7.45MB
tinyhttpd                               v0.1-7              9a5668f5f3a3        2 days ago          7.45MB
tinyhttpd                               v0.1-6              2368b7546664        2 days ago          7.45MB
tinyhttpd                               v0.1-5              f6f58ddb1520        2 days ago          7.45MB
tinyhttpd                               v0.1-4              5086cc9cdc4a        2 days ago          7.45MB
tinyhttpd                               v0.1-3              aa780396b2a3        2 days ago          2.27MB
tinyhttpd                               v0.1-2              de390df4c4c3        2 days ago          1.23MB
tinyhttpd                               v0.1-1              794f641ae85e        2 days ago          1.22MB
redis                                   latest              de25a81a5a0b        5 days ago          98.2MB
busybox                                 latest              19485c79a9bb        6 weeks ago         1.22MB
nginx                                   1.14-alpine         8a2fb25a19f5        6 months ago        16MB
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# docker tag myweb:v0.3-1 node103.yinzhengjie.org.cn:80/devops/myweb:v0.3-1　　　　　　#将已经有的镜像进行打tag操作(咱们为了试验效果，共tag了5个镜像)
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# docker tag myweb:v0.3-2 node103.yinzhengjie.org.cn:80/devops/myweb:v0.3-2
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# docker tag myweb:v0.3-3 node103.yinzhengjie.org.cn:80/devops/myweb:v0.3-3
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# docker tag myweb:v0.3-4 node103.yinzhengjie.org.cn:80/devops/myweb:v0.3-4
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# docker tag myweb:v0.3-5 node103.yinzhengjie.org.cn:80/devops/myweb:v0.3-5
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# docker images
REPOSITORY                                   TAG                 IMAGE ID            CREATED             SIZE
test                                         v0.1-1              d2003d6841b7        15 hours ago        16.1MB
myweb                                        v0.3-11             c0a33ce27a6b        15 hours ago        16MB
node102.yinzhengjie.org.cn:5000/myweb        v0.3-11             c0a33ce27a6b        15 hours ago        16MB
myweb                                        v0.3-10             fb0ef0b3aea3        15 hours ago        16MB
myweb                                        v0.3-9              6f9999d46b69        15 hours ago        16MB
myweb                                        v0.3-8              0110cfd06ce1        15 hours ago        16MB
myweb                                        v0.3-7              cb08b09cd959        16 hours ago        16MB
myweb                                        v0.3-6              0082820cf7f0        20 hours ago        16MB
myweb                                        v0.3-5              fb4f5a5ac1ba        20 hours ago        16MB
node103.yinzhengjie.org.cn:80/devops/myweb   v0.3-5              fb4f5a5ac1ba        20 hours ago        16MB
myweb                                        v0.3-4              7697fbbe8233        20 hours ago        16MB
node103.yinzhengjie.org.cn:80/devops/myweb   v0.3-4              7697fbbe8233        20 hours ago        16MB
myweb                                        v0.3-3              b512ab335e80        20 hours ago        16MB
node103.yinzhengjie.org.cn:80/devops/myweb   v0.3-3              b512ab335e80        20 hours ago        16MB
myweb                                        v0.3-2              6162cd671ca9        20 hours ago        16MB
node103.yinzhengjie.org.cn:80/devops/myweb   v0.3-2              6162cd671ca9        20 hours ago        16MB
myweb                                        v0.3-1              f14829ada08d        20 hours ago        16MB
node103.yinzhengjie.org.cn:80/devops/myweb   v0.3-1              f14829ada08d        20 hours ago        16MB
tinyhttpd                                    v0.2-3              c06eb08ea6d6        47 hours ago        1.22MB
tinyhttpd                                    v0.2-2              c7447ee49e2b        2 days ago          1.22MB
tinyhttpd                                    v0.2-1              807230671730        2 days ago          1.22MB
tinyhttpd                                    v0.1-9              24c887b252ad        2 days ago          8.49MB
tinyhttpd                                    v0.1-8              42c71beb05d5        2 days ago          7.45MB
tinyhttpd                                    v0.1-7              9a5668f5f3a3        2 days ago          7.45MB
tinyhttpd                                    v0.1-6              2368b7546664        2 days ago          7.45MB
tinyhttpd                                    v0.1-5              f6f58ddb1520        2 days ago          7.45MB
tinyhttpd                                    v0.1-4              5086cc9cdc4a        2 days ago          7.45MB
tinyhttpd                                    v0.1-3              aa780396b2a3        2 days ago          2.27MB
tinyhttpd                                    v0.1-2              de390df4c4c3        2 days ago          1.23MB
tinyhttpd                                    v0.1-1              794f641ae85e        2 days ago          1.22MB
redis                                        latest              de25a81a5a0b        5 days ago          98.2MB
busybox                                      latest              19485c79a9bb        6 weeks ago         1.22MB
nginx                                        1.14-alpine         8a2fb25a19f5        6 months ago        16MB
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# 
```

\[root@node101.yinzhengjie.org.cn ~\]# docker tag myweb:v0.3-1 node103.yinzhengjie.org.cn:80/devops/myweb:v0.3-1　　#将已经有的镜像进行打tag操作(咱们为了试验效果，共tag了5个镜像)

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node101.yinzhengjie.org.cn ~]# docker login node103.yinzhengjie.org.cn:80　　　　#此处我们基于80端口登录，默认时基于443端口登录的
Username: yinzhengjie
Password: 
WARNING! Your password will be stored unencrypted in /root/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# 
```

\[root@node101.yinzhengjie.org.cn ~\]# docker login node103.yinzhengjie.org.cn:80　　　　#此处我们基于80端口登录，默认时基于443端口登录的

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node101.yinzhengjie.org.cn ~]# docker image ls
REPOSITORY                                   TAG                 IMAGE ID            CREATED             SIZE
test                                         v0.1-1              d2003d6841b7        15 hours ago        16.1MB
node102.yinzhengjie.org.cn:5000/myweb        v0.3-11             c0a33ce27a6b        15 hours ago        16MB
myweb                                        v0.3-11             c0a33ce27a6b        15 hours ago        16MB
myweb                                        v0.3-10             fb0ef0b3aea3        15 hours ago        16MB
myweb                                        v0.3-9              6f9999d46b69        15 hours ago        16MB
myweb                                        v0.3-8              0110cfd06ce1        15 hours ago        16MB
myweb                                        v0.3-7              cb08b09cd959        16 hours ago        16MB
myweb                                        v0.3-6              0082820cf7f0        20 hours ago        16MB
myweb                                        v0.3-5              fb4f5a5ac1ba        20 hours ago        16MB
node103.yinzhengjie.org.cn:80/devops/myweb   v0.3-5              fb4f5a5ac1ba        20 hours ago        16MB
myweb                                        v0.3-4              7697fbbe8233        20 hours ago        16MB
node103.yinzhengjie.org.cn:80/devops/myweb   v0.3-4              7697fbbe8233        20 hours ago        16MB
myweb                                        v0.3-3              b512ab335e80        20 hours ago        16MB
node103.yinzhengjie.org.cn:80/devops/myweb   v0.3-3              b512ab335e80        20 hours ago        16MB
node103.yinzhengjie.org.cn:80/devops/myweb   v0.3-2              6162cd671ca9        20 hours ago        16MB
myweb                                        v0.3-2              6162cd671ca9        20 hours ago        16MB
myweb                                        v0.3-1              f14829ada08d        20 hours ago        16MB
node103.yinzhengjie.org.cn:80/devops/myweb   v0.3-1              f14829ada08d        20 hours ago        16MB
tinyhttpd                                    v0.2-3              c06eb08ea6d6        47 hours ago        1.22MB
tinyhttpd                                    v0.2-2              c7447ee49e2b        2 days ago          1.22MB
tinyhttpd                                    v0.2-1              807230671730        2 days ago          1.22MB
tinyhttpd                                    v0.1-9              24c887b252ad        2 days ago          8.49MB
tinyhttpd                                    v0.1-8              42c71beb05d5        2 days ago          7.45MB
tinyhttpd                                    v0.1-7              9a5668f5f3a3        2 days ago          7.45MB
tinyhttpd                                    v0.1-6              2368b7546664        2 days ago          7.45MB
tinyhttpd                                    v0.1-5              f6f58ddb1520        2 days ago          7.45MB
tinyhttpd                                    v0.1-4              5086cc9cdc4a        2 days ago          7.45MB
tinyhttpd                                    v0.1-3              aa780396b2a3        2 days ago          2.27MB
tinyhttpd                                    v0.1-2              de390df4c4c3        2 days ago          1.23MB
tinyhttpd                                    v0.1-1              794f641ae85e        2 days ago          1.22MB
redis                                        latest              de25a81a5a0b        5 days ago          98.2MB
busybox                                      latest              19485c79a9bb        6 weeks ago         1.22MB
nginx                                        1.14-alpine         8a2fb25a19f5        6 months ago        16MB
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# docker push node103.yinzhengjie.org.cn:80/devops/myweb　　　　　　#将已经打好tag的镜像全部推送到已经登录的Harbor服务器上
The push refers to repository [node103.yinzhengjie.org.cn:80/devops/myweb]
55514dc3bafb: Pushed 
076c58d2644f: Pushed 
b2cbae4b8c15: Pushed 
5ac9a5170bf2: Pushed 
a464c54f93a9: Pushed 
v0.3-1: digest: sha256:9c7535800e955be11ffa7ab5adb0d758b11ddee3c11ea408cc6eab7755890710 size: 1360
55514dc3bafb: Layer already exists 
076c58d2644f: Layer already exists 
b2cbae4b8c15: Layer already exists 
5ac9a5170bf2: Layer already exists 
a464c54f93a9: Layer already exists 
v0.3-2: digest: sha256:f79e1abd048ce9e4e5ee2ebfe4a1a3454772b14b0818fcf5fe117418e6038280 size: 1360
55514dc3bafb: Layer already exists 
076c58d2644f: Layer already exists 
b2cbae4b8c15: Layer already exists 
5ac9a5170bf2: Layer already exists 
a464c54f93a9: Layer already exists 
v0.3-3: digest: sha256:4ad45d226ea605f1625967a1cfe86fe22fff535becb722d20a2abd0b5ca88e97 size: 1360
0dac9c03dd8a: Pushed 
076c58d2644f: Layer already exists 
b2cbae4b8c15: Layer already exists 
5ac9a5170bf2: Layer already exists 
a464c54f93a9: Layer already exists 
v0.3-4: digest: sha256:894dea926a0d07248ddbd80ca83ce7149c0d4f9250933a681981f418fe0e98b2 size: 1360
77040215254b: Pushed 
076c58d2644f: Layer already exists 
b2cbae4b8c15: Layer already exists 
5ac9a5170bf2: Layer already exists 
a464c54f93a9: Layer already exists 
v0.3-5: digest: sha256:b436ff5ab1f8e626015a351588c74cad0a006ae99b62565f5aea6721050dec27 size: 1360
[root@node101.yinzhengjie.org.cn ~]# 
```

\[root@node101.yinzhengjie.org.cn ~\]# docker push node103.yinzhengjie.org.cn:80/devops/myweb　　　　　　#将已经打好tag的镜像全部推送到已经登录的Harbor服务器上

![](https://img2018.cnblogs.com/blog/795254/201910/795254-20191023074046393-1712860820.png)

**7>.在Harbor的Web UI查看上传的镜像详细信息**

**![](https://img2018.cnblogs.com/blog/795254/201910/795254-20191023074138438-297692902.png)**

**8>.docker-compose脚本文件概述（建议熟悉Harbor服务的配置文件:"/usr/local/harbor/harbor.yml"）**

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node103.yinzhengjie.org.cn ~]# docker-compose  --help　　　　　　#查看该命令的帮助信息
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
[root@node103.yinzhengjie.org.cn ~]# 
```

\[root@node103.yinzhengjie.org.cn ~\]# docker-compose --help　　　　　　#查看该命令的帮助信息

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node103.yinzhengjie.org.cn ~]# cd /usr/local/harbor/
[root@node103.yinzhengjie.org.cn /usr/local/harbor]# 
[root@node103.yinzhengjie.org.cn /usr/local/harbor]# ll
total 607880
drwxr-xr-x 3 root root        20 Oct 22 23:48 common
-rw-r--r-- 1 root root      5290 Oct 22 23:48 docker-compose.yml
-rw-r--r-- 1 root root 622428100 Sep 27 14:52 harbor.v1.9.1.tar.gz
-rw-r--r-- 1 root root      5815 Oct 22 23:40 harbor.yml
-rwxr-xr-x 1 root root      5088 Sep 27 14:52 install.sh
-rw-r--r-- 1 root root     11347 Sep 27 14:52 LICENSE
-rwxr-xr-x 1 root root      1748 Sep 27 14:52 prepare
[root@node103.yinzhengjie.org.cn /usr/local/harbor]# 
[root@node103.yinzhengjie.org.cn /usr/local/harbor]# docker-compose  pause　　　　#执行暂停服务命令时要进入存放"harbor.yml"的目录
Pausing harbor-log        ... done
Pausing harbor-db         ... done
Pausing registry          ... done
Pausing redis             ... done
Pausing harbor-portal     ... done
Pausing registryctl       ... done
Pausing harbor-core       ... done
Pausing harbor-jobservice ... done
Pausing nginx             ... done
[root@node103.yinzhengjie.org.cn /usr/local/harbor]# 
```

\[root@node103.yinzhengjie.org.cn /usr/local/harbor\]# docker-compose pause　　　　#执行暂停服务命令时要进入存放"harbor.yml"的目录

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node103.yinzhengjie.org.cn /usr/local/harbor]# docker-compose  unpause　　 #不暂停harbor服务
Unpausing nginx             ... done
Unpausing harbor-jobservice ... done
Unpausing harbor-core       ... done
Unpausing registryctl       ... done
Unpausing harbor-portal     ... done
Unpausing redis             ... done
Unpausing registry          ... done
Unpausing harbor-db         ... done
Unpausing harbor-log        ... done
[root@node103.yinzhengjie.org.cn /usr/local/harbor]# 
```

\[root@node103.yinzhengjie.org.cn /usr/local/harbor\]# docker-compose unpause　　 #不暂停harbor服务

当你的才华还撑不起你的野心的时候，你就应该静下心来学习。当你的能力还驾驭不了你的目标的时候，你就应该沉下心来历练。问问自己，想要怎样的人生。 欢迎加入基础架构自动化运维：598432640，大数据SRE进阶之路：959042252，DevOps进阶之路：526991186