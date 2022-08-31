关于Harbor私有仓库的搭建及使用

[强烈推介IDEA2020.2破解激活，IntelliJ IDEA 注册码，2020.2 IDEA 激活码](http://javajgs.com/archives/6355)

![](https://ask.qcloudimg.com/http-save/yehe-1011815/0a6yd5v90y.png?imageView2/2/w/1620)

### 一、介绍

Harbor，是一个英文单词，意思是港湾，港湾是干什么的呢，就是停放货物的，而货物呢，是装在集装箱中的，说到集装箱，就不得不提到Docker[容器](https://cloud.tencent.com/product/tke?from=10680)，因为docker容器的技术正是借鉴了集装箱的原理。所以，Harbor正是一个用于存储Docker镜像的企业级Registry服务。 Registry是Dcoker官方的一个私有仓库镜像，可以将本地的镜像打标签进行标记然后push到以Registry起的容器的私有仓库中。企业可以根据自己的需求，使用Dokcerfile生成自己的镜像，并推到私有仓库中，这样可以大大提高拉取镜像的效率。

### 二、Harbor核心组件解释

-   Proxy：他是一个nginx的前端代理，代理Harbor的registry,UI, token等服务。
-   db：负责储存用户权限、审计日志、Dockerimage分组信息等数据。
-   UI：提供图形化界面，帮助用户管理registry上的镜像, 并对用户进行授权。
-   jobsevice：jobsevice是负责镜像复制工作的，他和registry通信，从一个registry pull镜像然后push到另一个registry，并记录job\_log。
-   Adminserver：是系统的配置管理中心附带检查存储用量，ui和jobserver启动时候回需要加载adminserver的配置。
-   Registry：[镜像仓库](https://cloud.tencent.com/product/tcr?from=10680)，负责存储镜像文件。
-   Log：为了帮助监控Harbor运行，负责收集其他组件的log，供日后进行分析。

### 三：Harbor和Registry的比较

Harbor和Registry都是Docker的镜像仓库，但是Harbor作为更多企业的选择，是因为相比较于Regisrty来说，它具有很多的优势。

1.提供分层传输机制，优化网络传输 Docker镜像是是分层的，而如果每次传输都使用全量文件(所以用FTP的方式并不适合)，显然不经济。必须提供识别分层传输的机制，以层的UUID为标识，确定传输的对象。 2.提供WEB界面，优化用户体验 只用镜像的名字来进行上传下载显然很不方便，需要有一个用户界面可以支持登陆、搜索功能，包括区分公有、私有镜像。 3.支持水平扩展集群 当有用户对镜像的上传下载操作集中在某[服务器](https://cloud.tencent.com/product/cvm?from=10680)，需要对相应的访问压力作分解。 4.良好的安全机制 企业中的开发团队有很多不同的职位，对于不同的职位人员，分配不同的权限，具有更好的安全性。 5.Harbor提供了基于角色的访问控制机制，并通过项目来对镜像进行组织和访问权限的控制。kubernetes中通过namespace来对资源进行隔离，在企业级应用场景中，通过将两者进行结合可以有效将kubernetes使用的镜像资源进行管理和访问控制，增强镜像使用的安全性。尤其是在多租户场景下，可以通过租户、namespace和项目相结合的方式来实现对多租户镜像资源的管理和访问控制。

## \=================== 安装步骤 =================

### 环境：

|  |  |  |
| --- | --- | --- |
| 
192.168.1.10



 | 

CentOS Linux release 7.6.1810 (Core)



 | 

docker、docker-compose、harbor



 |

```
#包含Harbor安装包和Docker-compose工具
链接：https://pan.baidu.com/s/18qnMOV50PxxpwXIDP7dimg 
提取码：rqz6 
```

### ①安装docker

配置Docker的Yum源

```
echo -e "[docker] \nname=docker \nbaseurl=https://mirrors.aliyun.com/docker-ce/linux/centos/7/x86_64/stable/ \nenabled=1 \ngpgcheck=1 \ngpgkey=https://mirrors.aliyun.com/docker-ce/linux/centos/gpg" >> /etc/yum.repos.d/docker.repo
```

安装docker 这个默认是最新版本

配置阿里云加速 拉取镜像的速度会快

```
echo {
   \"registry-mirrors\": [\"https://h78bv2ra.mirror.aliyuncs.com\"]} >> /etc/docker/daemon.json
```

启动docker

```
systemctl daemon-reload
systemctl start docker
```

版本信息

```
docker --version
Docker version 19.03.13, build 4484c46d9d
```

### ②安装docker-compose

```
下载docker-compose工具
cd /usr/bin/

导入下载好的docker-compose
chmod +x docker-compose

查看版本
docker-compose --version
docker-compose version 1.25.4, build 8d51620a
```

### ③安装Harbor私有仓库

[Harbor下载地址](https://javajgs.com/go?url=https://github.com/goharbor/harbor/releases)

```
#解压
tar -zxf harbor-offline-installer-v1.6.3.tgz
#修改配置文件
vi harbor/harbor.cfg
hostname = 192.168.1.10 修改为本机IP
harbor_admin_password = 12345 登录harbor仓库的密码
#其他配置根据自己需要进行修改
进行安装harbor
cd harbor && ./install.sh
查看安装情况
[root@localhost harbor]# docker ps
CONTAINER ID        IMAGE                                    COMMAND                  CREATED             STATUS                    PORTS                                                                NAMES
9fffb15a7822        goharbor/nginx-photon:v1.6.3             "nginx -g 'daemon of…"   54 minutes ago      Up 54 minutes (healthy)   0.0.0.0:443->443/tcp, 0.0.0.0:4443->4443/tcp, 0.0.0.0:80->80/tcp   nginx
6391b340e503        goharbor/harbor-jobservice:v1.6.3        "/harbor/start.sh"       54 minutes ago      Up 54 minutes                                                                                  harbor-jobservice
b2c17914ae7a        goharbor/harbor-ui:v1.6.3                "/harbor/start.sh"       55 minutes ago      Up 54 minutes (healthy)                                                                        harbor-ui
07856849920b        goharbor/registry-photon:v2.6.2-v1.6.3   "/entrypoint.sh /etc…"   55 minutes ago      Up 55 minutes (healthy)   5000/tcp                                                             registry
761546a6e0cb        goharbor/redis-photon:v1.6.3             "docker-entrypoint.s…"   55 minutes ago      Up 55 minutes             6379/tcp                                                             redis
794635b0d59c        goharbor/harbor-adminserver:v1.6.3       "/harbor/start.sh"       55 minutes ago      Up 54 minutes (healthy)                                                                        harbor-adminserver
12ea34cd31b8        goharbor/harbor-db:v1.6.3                "/entrypoint.sh post…"   55 minutes ago      Up 55 minutes (healthy)   5432/tcp                                                             harbor-db
a939a7d12646        goharbor/harbor-log:v1.6.3               "/bin/sh -c /usr/loc…"   55 minutes ago      Up 55 minutes (healthy)   127.0.0.1:1514->10514/tcp                                            harbor-log
#安装好后默认的端口是80
```

修改docker.service文件指定私有仓库地址

```
[root@localhost ~]# find / -name docker.service -type f
/usr/lib/systemd/system/docker.service
[root@localhost ~]# vi /usr/lib/systemd/system/docker.service
#ExecStart尾部添加地址
ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock --insecure-registry=http://192.168.1.10
#重启docker
systemctl daemon-reload  && systemctl restart docker
#查看一下harbor状态是否正常（如果正常跳过）
#不正常状态下操作如下：
cd harbor && docker-compose down && docker-compose up -d
```

访问web界面默认80端口 关闭防火墙或打开端口

![](https://ask.qcloudimg.com/http-save/yehe-1011815/610casxq6g.png?imageView2/2/w/1620)

![](https://ask.qcloudimg.com/http-save/yehe-1011815/woitbvifu8.png?imageView2/2/w/1620)

![](https://ask.qcloudimg.com/http-save/yehe-1011815/psxb4qlvlc.png?imageView2/2/w/1620)

### ④测试：

> 登录一下，防止后面上传镜像失败

```
[root@localhost harbor]# docker login 192.168.1.10
Username: admin
Password: 12345#文件设置的密码
Login Succeeded #出现则为成功
```

修改镜像标签测试上传

```
#随便pull一个镜像
[root@localhost ~]# docker pull mysql
[root@localhost ~]# docker tag mysql 192.168.1.10/hello/mysql #修改标签格式：ip地址/项目名称/自定义
#上传
[root@localhost ~]# docker push 192.168.1.10/hello/mysql
The push refers to repository [192.168.1.10/hello/mysql]
9b0377a95c0e: Pushed 
af6e790b8237: Pushed 
060fef62a228: Pushed 
7f893b7c04ac: Pushed 
7832ac00d41e: Pushed 
15b463db445c: Pushed 
c21e35e55228: Pushed 
36b89ee4c647: Pushed 
9dae2565e824: Pushed 
ec8c80284c72: Pushed 
329fe06a30f0: Pushed 
d0fe97fa8b8c: Mounted from test/nginx 
latest: digest: sha256:c7788fdc4c04a64bf02de3541656669b05884146cb3995aa64fa4111932bec0f size: 2828
#上传成功/web界面看一下
```

![](https://ask.qcloudimg.com/http-save/yehe-1011815/pcmf6ev7qz.png?imageView2/2/w/1620)

## 搭建成功

___

___

___

### 关于修改Harbor仓库默认端口

如果仓库已经搭建成功了先停止一下

```
cd harbor && docker-compose down #停止关闭
修改配置文件
[root@localhost ~]# vi harbor/docker-compose.yml
找到端口号
    ports:
      - 9090:80   #默认80:80 修改主机9090映射容器80
      - 443:443
      - 4443:4443
保存退出
还需要修改一个配置文件，否则上传文件会失败
[root@localhost ~]# vi harbor/common/templates/registry/config.yml
找到该字段，在$public_url:添加端口号
auth:
  token:
    issuer: harbor-token-issuer
    realm: $public_url:9090/service/token
    rootcertbundle: /etc/registry/root.crt
    service: harbor-registry
保存退出
cd harbor && ./install.sh #重新安装 会读取修改的配置
搭建成功 docker ps 可以看到端口映射成功
[root@localhost harbor]# docker ps
CONTAINER ID        IMAGE                                    COMMAND                  CREATED             STATUS                 PORTS                                                                NAMES
9fffb15a7822        goharbor/nginx-photon:v1.6.3             "nginx -g 'daemon of…"   2 hours ago         Up 2 hours (healthy)   0.0.0.0:443->443/tcp, 0.0.0.0:4443->4443/tcp, 0.0.0.0:9090->80/tcp   nginx
```

修改docker.service文件指定私有仓库地址

```
[root@localhost ~]# find / -name docker.service -type f
/usr/lib/systemd/system/docker.service
[root@localhost ~]# vi /usr/lib/systemd/system/docker.service
#ExecStart尾部添加地址
ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock --insecure-registry=http://192.168.1.10:9090
#重启docker
systemctl daemon-reload  && systemctl restart docker
#查看一下harbor状态是否正常（如果正常跳过）
#不正常状态下操作如下：
cd harbor && docker-compose down && docker-compose up -d
```

### 测试

web界面

![](https://ask.qcloudimg.com/http-save/yehe-1011815/up7x9uoud9.png?imageView2/2/w/1620)

> 登录一下，防止后面上传镜像失败

```
[root@localhost harbor]# docker login 192.168.1.10:9090
Username: admin
Password: 12345#文件设置的密码
Login Succeeded #出现则为成功
```

修改镜像标签测试上传

```
#随便pull一个镜像
[root@localhost ~]# docker pull nginx
[root@localhost ~]# docker tag nginx 192.168.1.10:9090/hello/nginx #修改标签格式：ip地址:端口/项目名称/自定义
#上传
[root@localhost harbor]# docker push 192.168.1.10:9090/hello/nginx
The push refers to repository [192.168.1.10:9090/hello/nginx]
7b5417cae114: Mounted from test/nginx 
aee208b6ccfb: Mounted from test/nginx 
2f57e21e4365: Mounted from test/nginx 
2baf69a23d7a: Mounted from test/nginx 
d0fe97fa8b8c: Mounted from hello/mysql 
latest: digest: sha256:34f3f875e745861ff8a37552ed7eb4b673544d2c56c7cc58f9a9bec5b4b3530e size: 1362
#上传成功/web界面看一下
```

### 成功

本文分享自作者个人站点/博客：http://javajgs.com复制

如有侵权，请联系

cloudcommunity@tencent.com

删除。