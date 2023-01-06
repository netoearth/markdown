   

[![](https://p26-passport.byteacctimg.com/img/user-avatar/e3b82b14bcbe9861a2b50f64dcd89de1~300x300.image)](https://juejin.cn/user/3611262085773719)

2022年08月25日 17:31 ·  阅读 31689

![《‘狂’人日记》---Docker从入门到进阶之基础操作（三）](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e02049bfbef8458d83b82ae5acb78777~tplv-k3u1fbpfcp-zoom-crop-mark:3024:3024:3024:1702.awebp?)  

携手创作，共同成长！这是我参与「掘金日新计划 · 8 月更文挑战」的第25天，[点击查看活动详情](https://juejin.cn/post/7123120819437322247 "https://juejin.cn/post/7123120819437322247")

## 前两章介绍了Docker的容器和镜像部分，那么这一章就开始介绍仓库

## 1.仓库

**仓库（Repository）是集中存放镜像的地方。 一个容易混淆的概念是注册服务器（Registry）。实际上注册服务器是管理仓库的具体服务器，每个服务器上可以有多个仓 库，而每个仓库下面有多个镜像。从这方面来说，仓库可以被认为是一个具体的项目或目录。例如对于仓库地址 dl.dockerpool.com/ubuntu 来说， dl.dockerpool.com 是注册服务器地址， ubuntu 是仓库名。 大部分时候，并不需要严格区分这两者的概念。**

## 1.1 DockerHub

## 1.1.1 什么是Docker Hub

**目前 Docker 官方维护了一个公共仓库 Docker Hub，其中已经包括了超过 15,000 的镜像。大部分需求，都可以通过在 Docker Hub 中直接下载镜像来实现。 登录 可以通过执行 docker login 命令来输入用户名、密码和邮箱来完成注册和登录。 注册成功后，本地用户目录的 .dockercfg 中将保存用户的认证信息。 基本操作 用户无需登录即可通过 docker search 命令来查找官方仓库中的镜像，并利用 docker pull 命令来将它下载到本 地。 例如以 centos 为关键词进行搜索：**

```
[root@base ~]# docker search centos
INDEX       NAME                                                   DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
docker.io   docker.io/centos                                       The official build of CentOS.                   7292      [OK]
docker.io   docker.io/kasmweb/centos-7-desktop                     CentOS 7 desktop for Kasm Workspaces            24
docker.io   docker.io/couchbase/centos7-systemd                    centos7-systemd images with additional deb...   4                    [OK]
docker.io   docker.io/continuumio/centos5_gcc5_base                                                                3
docker.io   docker.io/dokken/centos-7                              CentOS 7 image for kitchen-dokken               3
docker.io   docker.io/dokken/centos-stream-8                                                                       2
docker.io   docker.io/dokken/centos-stream-9                                                                       2
docker.io   docker.io/spack/centos6                                CentOS 6 with Spack preinstalled                1
docker.io   docker.io/spack/centos7                                CentOS 7 with Spack preinstalled                1
docker.io   docker.io/bitnami/centos-base-buildpack                Centos base compilation image                   0                    [OK]
docker.io   docker.io/bitnami/centos-extras-base                                                                   0
docker.io   docker.io/corpusops/centos                             centos corpusops baseimage                      0
docker.io   docker.io/corpusops/centos-bare                        https://github.com/corpusops/docker-images/     0
docker.io   docker.io/couchbase/centos-69-sdk-build                                                                0
docker.io   docker.io/couchbase/centos-69-sdk-nodevtoolset-build                                                   0
docker.io   docker.io/couchbase/centos-70-sdk-build                                                                0
docker.io   docker.io/couchbase/centos-72-java-sdk                                                                 0
docker.io   docker.io/couchbase/centos-72-jenkins-core                                                             0
docker.io   docker.io/datadog/centos-i386                                                                          0
docker.io   docker.io/dokken/centos-5                              EOL DISTRO: For use with kitchen-dokken, B...   0
docker.io   docker.io/dokken/centos-6                              CentOS 6 image for kitchen-dokken               0
docker.io   docker.io/dokken/centos-8                              CentOS 8 image for kitchen-dokken               0
docker.io   docker.io/fnndsc/centos-python3                        Source for a slim Centos-based Python3 ima...   0                    [OK]
docker.io   docker.io/spack/centos-stream                                                                          0
docker.io   docker.io/ustclug/centos                               Official CentOS Image with USTC Mirror          0

复制代码
```

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/55b65289c87c4947bb0dfda5f288d288~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

## 1.1.2 pull---从dockerhub拉取镜像(ubuntu)

```
[root@base ~]# docker pull ubuntu
Using default tag: latest
Trying to pull repository docker.io/library/ubuntu ...
latest: Pulling from docker.io/library/ubuntu
d19f32bd9e41: Pull complete
Digest: sha256:34fea4f31bf187bc915536831fd0afc9d214755bf700b5cdb1336c82516d154e
Status: Downloaded newer image for docker.io/ubuntu:latest
[root@base ~]# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
ssh-server          latest              4e28f6e4fc6a        19 hours ago        390 MB
docker.io/ubuntu    latest              df5de72bdb3b        3 weeks ago         77.8 MB
docker.io/centos    7                   eeb6ee3f44bd        11 months ago       204 MB

复制代码
```

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/375fd371fc0f4cb087e4c5d9c6f10ac4~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

## 1.2 设置镜像加速

\[点击此处设置阿里云镜像加速服务\]([cr.console.aliyun.com/cn-](https://link.juejin.cn/?target=https%3A%2F%2Fcr.console.aliyun.com%2Fcn- "https://link.juejin.cn/?target=https%3A%2F%2Fcr.console.aliyun.com%2Fcn-")

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a303c74fbded4a0e885ede37b5a9698a~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)  
修改为阿里云镜像加速\\

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6eb6e48112294ccabb304739762d0788~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

## 1.3 私有仓库

**因为某些安全、隐私、网络、法规等等问题，使用公共仓库并不方便，用户可以使用docker提供的私有仓库镜像搭建自己的私有仓库**

## 1.3.1 docker-registry

**docker-registry 是官方提供的工具，可以用于构建私有的镜像仓库**

**搭建方法**

```
# 运行本地仓库容器
docker run -dit --name registry --restart=always -p 5000:5000 registry

# 修改docker配置文件
vi /etc/sysconfig/docker
# 添加
# 172.31.36.113 为宿主机局域网IP地址，请自行更改为自己的IP地址
ADD_REGISTRY='--add-registry 192.168.200.201:5000'
INSECURE_REGISTRY='--insecure-registry 192.168.200.201:5000'

vi /etc/docker/daemon.json
# 添加
# 注意上一行后面需增加 , 
"insecure-registries":["192.168.200.201:5000"]

vi /usr/lib/systemd/system/docker.service
删除  $INSECURE_REGISTRY \ 这一行

# 重新加载
systemctl deamon-reload
systemctl restart docker
复制代码
```

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/017314396bb74661bed451f3d17e949c~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

![](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/00ba359ecd0075e59ffbc3d810af551d.svg) 2

![](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/3d482c7a948bac826e155953b2a28a9e.svg) 收藏