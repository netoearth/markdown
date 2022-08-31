## ![](https://oss-emcsprod-public.modb.pro/wechatSpider/modb_20220505_8a4304c8-cc69-11ec-8739-fa163eb4f6be.png)

image-20210530081851911

## 目录

-   实战：使用harbor搭建Docker私有仓库-2022.2.28
    
-   目录
    
-   实验环境
    
-   实验软件
    
-   前言
    

-   1.harbor介绍
    
-   2.docker-compose项目介绍
    

-   先决条件
    
-   0、基础环境配置(Linux基础配置+docker安装+主机名配置)
    
-   1、安装 docker-compose
    

-   1.方法1：在线安装
    
-   2.方法2：离线安装(本次安装方式)
    
-   3.方法3：使用python 的pip安装docker-compose
    

-   2、安装 Harbor 私有仓库
    

-   1.下载 Harbor 安装文件
    
-   2.配置 Harbor
    

-   3、启动Harbor
    

-   1.确认上述配置没有问题后，进行启动harbor
    
-   2.开始安装 harbor
    
-   3.查看 Harbor 依赖的镜像及启动服务如下
    

-   4、使用 harbor 管理镜像
    
-   5、新建一个仓库项目
    
-   6、修改本地 docker 服务使用 http 协议和私有仓库通信
    
-   7、重启harbor服务，登录harbor仓库
    
-   8、上传镜像到私有仓库中
    

-   1.查看当前本地镜像
    
-   2.admin 登录harbor
    
-   3.给本地镜像打 tag
    
-   4.将打好tag的本地镜像push到仓库
    
-   5.在 web 界面查看刚才上传到仓库中的镜像
    

-   9、从私有仓库 pull 镜像
    

-   1.首先删除本地镜像
    
-   2.pull 私有仓库中的镜像并验证
    

-   QA
    

-   📍 docker-compose命令使用
    
-   📍 不同harbor软件包的方式有点区别
    
-   📍 关于如何修改harmor密码，感觉很繁琐的样子。。。(修改了好几次都没成功。。。)
    
-   📍 还有个问题，如何卸载harbor服务呢？
    
-   📍 还有个问题：这个harbor的版本号怎么奇奇怪怪的。。。
    

-   总结
    

## 实验环境

```
1台centos7.x虚机。docker version：20.10.6(附近版本都可以)harbor版本： v1.5.0-d59c257e登录后复制
```

| 用途 | 主机名 | ip | 系统版本 | 备注 |
| --- | --- | --- | --- | --- |
| 作为harbor使用 | harbor | 172.29.9.10 | centos7.7 1908 | 使用高配版虚拟机；  
注意：安装 harbor，**系统根分区的可用空间需要大于 6G，否则安装时会报空间不足**。内存 2G 以上。 |

## 实验软件

链接：https://pan.baidu.com/s/1RHOBb\_BnO4RLXFNRJ6R47w?pwd=k0r0 提取码：k0r0`2022.2.28-使用harbor搭建Docker私有仓库-实验软件`  

![](https://oss-emcsprod-public.modb.pro/wechatSpider/modb_20220505_8a757318-cc69-11ec-8739-fa163eb4f6be.png)

image-20220228231234799

## 前言

### 1.harbor介绍

Docker容器应用的开发和运行离不开可靠的镜像管理，虽然 Docker 官方也提供了公共的镜像仓库，但是从安全和效率等方面考虑，部署我们私有环境内的 Registry 也是非常必要的。

Harbor是由**VMWare公司**开源的容器镜像仓库。事实上，**Harbor是在Docker Registry上进行了相应的企业级扩展**，从而获得了更加广泛的应用，这些新的企业级特性包括：管理用户界面，基于角色的访问控制 ，AD/LDAP集成以及审计日志等，足以满足基本企业需求。

官方：https://goharbor.io/

Github：https://github.com/goharbor/harbor

harbor \['hɑ:bə\] 海湾 n.

![](https://oss-emcsprod-public.modb.pro/wechatSpider/modb_20220505_8a9a9ad0-cc69-11ec-8739-fa163eb4f6be.png)

image-20210529165339643

### 2.docker-compose项目介绍

**docker-compose 项目是 Docker 官方的开源项目，负责实现对 Docker 容器集群的快速编排。Docker-Compose 的工程配置文件默认为 docker-compose.yml，Docker-Compose 运行目录下的必要有一个 docker-compose.yml。docker-compose 可以管理多个 docker 实例。**

## 先决条件

**服务器硬件配置：**

• 最低要求：CPU2核/内存4G/硬盘40GB

• 推荐：CPU4核/内存8G/硬盘160GB

**软件：**

• Docker CE 17.06版本+

• Docker Compose 1.18版本+ (**harbor就是采用这种方式去管理里面的一些服务的**)

**Harbor安装有2种方式：**

• 在线安装：从Docker Hub下载Harbor相关镜像，因此安装软件包非常小

• 离线安装：安装包包含部署的相关镜像，因此安装包比较大

![](https://oss-emcsprod-public.modb.pro/wechatSpider/modb_20220505_8abe3a62-cc69-11ec-8739-fa163eb4f6be.png)

image-20220228193447683

## 0、基础环境配置(Linux基础配置+docker安装+主机名配置)

和之前配置一样，这里快速用脚本跑一下即可。

1、基础环境配置脚本

```
(1)关闭且禁用如下服务：firewalld、NetworkManager、selinux systemctl stop firewalld && systemctl disable  firewalld && systemctl stop NetworkManager && systemctl disable  NetworkManagersetenforce 0sed -i s/SELINUX=enforcing/SELINUX=disabled/ etc/selinux/config(2)配置网络yum源：cd etc/yum.repos.d/mkdir backup-`date +%F`mv * !$wget -O etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repoyum clean allyum makecacheyum install -y net-toolsyum install -y vimyum install -y wgetyum install -y lrzsz(3)配置主机名(对相应的机器配置主机名)hostnamectl --static set-hostname harborexec bash登录后复制
```

2、安装docker脚本

```
#安装必要的一些系统工具yum install -y yum-utils device-mapper-persistent-data lvm2#配置国内 docker 的 yum 源yum-config-manager --add-repo https://mirrors.tuna.tsinghua.edu.cn/docker-ce/linux/Centos/docker-ce.repoyum makecache#安装 docker-ceyum install docker-ce docker-ce-cli containerd.io -y#启动并开机自启docker服务systemctl start docker && systemctl enable docker#查看docker版本docker versiondocker info#配置docker镜像地址mkdir -p etc/dockertee etc/docker/daemon.json <<-'EOF'{ "registry-mirrors":["https://dockerhub.azk8s.cn","http://hub-mirror.c.163.com","http://qtid6917.mirror.aliyuncs.com"]}EOFsystemctl daemon-reloadsystemctl restart docker#开启网络转发功能echo "net.ipv4.ip_forward = 1" >> etc/sysctl.conf sysctl -pcat proc/sys/net/ipv4/ip_forward登录后复制
```

基础环境配置完成后，对上述配置进行验证，确保符合要求。

验证全部符合要求后，对虚机做一个init-success快照。

## 1、安装 docker-compose

### 1.方法1：在线安装

```
[root@harbor ~]# curl -L https://github.com/docker/compose/releases/download/1.26.2/docker-compose-`uname -s`-`uname -m` > usr/local/bin/docker-compose#添加执行权限[root@harbor ~]# chmod +x usr/local/bin/docker-compose登录后复制
```

### 2.方法2：离线安装(本次安装方式)

github 地址：https://github.com/docker/compose/releases/

![](https://oss-emcsprod-public.modb.pro/wechatSpider/modb_20220505_8af0edae-cc69-11ec-8739-fa163eb4f6be.png)

image-20210529191431346

下载二进制文件上传至 linux（课程资料已提供 docker-compose 二进制文件可直接上传）

![](https://oss-emcsprod-public.modb.pro/wechatSpider/modb_20220505_8b2cb1d6-cc69-11ec-8739-fa163eb4f6be.png)

image-20210529192243531

```
[root@harbor ~]# rz[root@harbor ~]# ll -h docker-compose-Linux-x86_64 -r-------- 1 root root 12M 7月  24 2020 docker-compose-Linux-x86_64[root@harbor ~]# mv docker-compose-Linux-x86_64 usr/local/bin/docker-compose[root@harbor ~]# chmod +x usr/local/bin/docker-compose #添加执行权限登录后复制
```

### 3.方法3：使用python 的pip安装docker-compose

-   安装epel源
    

```
[root@harbor ~]# yum install -y epel-release登录后复制
```

-   安装并升级pip
    

```
[root@harbor ~]# yum install -y python-pip[root@harbor ~]# pip install --upgrade pip登录后复制
```

-   使用pip安装docker-compose
    

```
[root@harbor ~]# pip install -U -i https://pypi.tuna.tsinghua.edu.cn/simple docker-compose登录后复制
```

## 2、安装 Harbor 私有仓库

### 1.下载 Harbor 安装文件

wget http://harbor.orientsoft.cn/harbor-v1.5.0/harbor-offline-installer-v1.5.0.tgz 注：此文件有 800M 左右大小，建议大家提前下载好后，上传到 linux 系统上。

```
[root@harbor ~]# rz[root@harbor ~]# ll -h harbor-offline-installer-v1.5.0.tgz -r-------- 1 root root 824M 2月  11 2019 harbor-offline-installer-v1.5.0.tgz[root@harbor ~]# tar xf harbor-offline-installer-v1.5.0.tgz -C opt/登录后复制
```

![](https://oss-emcsprod-public.modb.pro/wechatSpider/modb_20220505_8b4aa63c-cc69-11ec-8739-fa163eb4f6be.png)

image-20210529192440592

### 2.配置 Harbor

```
[root@harbor ~]# vim opt/harbor/harbor.cfg #查看配置文件中的基本信息# 修改点1：hostname 设置访问地址，可以使用 ip、域名，不可以设置为 127.0.0.1 或 localhost改： 7 hostname = reg.mydomain.com 为： 7 hostname = 172.29.9.10# 修改点2：启动 Harbor 后，管理员 UI 登录的密码，默认是 Harbor12345，改为 123456改：68 harbor_admin_password = Harbor12345为：68 harbor_admin_password = 123456# 修改点3：注释https相关信息#https:    # https port for harbor, default is 443   # port: 443   # The path of cert and key files for nginx  # certificate: your/certificate/path  # private_key: your/private/key/path登录后复制
```

修改完成后，保存退出。

## 3、启动Harbor

### 1.确认上述配置没有问题后，进行启动harbor

```
[root@harbor ~]# cd opt/harbor/[root@harbor harbor]# ./prepare #初始化安装环境登录后复制
```

![](https://oss-emcsprod-public.modb.pro/wechatSpider/modb_20220505_8b7cecf0-cc69-11ec-8739-fa163eb4f6be.png)

image-20210529194623465

### 2.开始安装 harbor

```
[root@harbor harbor]# ./install.sh #执行./install.sh，Harbor 服务就会根据当期目录下的/opt/harbor/docker-compose.yml 开始下载依赖的镜像，检测并按照顺序依次启动各个服务。最终弹出以下画面，说明安装成功了。#这里耐心等待一会儿就好！登录后复制
```

![](https://oss-emcsprod-public.modb.pro/wechatSpider/modb_20220505_8bacca92-cc69-11ec-8739-fa163eb4f6be.png)

image-20210529194938665

![](https://oss-emcsprod-public.modb.pro/wechatSpider/modb_20220505_8bd3d7e0-cc69-11ec-8739-fa163eb4f6be.png)

image-20210529195106008

![](https://oss-emcsprod-public.modb.pro/wechatSpider/modb_20220505_8c2ec01a-cc69-11ec-8739-fa163eb4f6be.png)

image-20210529195202456

### 3.查看 Harbor 依赖的镜像及启动服务如下

查看镜像：

![](https://oss-emcsprod-public.modb.pro/wechatSpider/modb_20220505_8ca4094c-cc69-11ec-8739-fa163eb4f6be.png)

image-20210529195337891

```
[root@harbor harbor]#docker-compose ps #用这个命令查看启动的服务       Name                     Command                       State                                    Ports                          --------------------------------------------------------------------------------------------------------------------------------------harbor-adminserver   harbor/start.sh                 Up (health: starting)                                                           harbor-db            usr/local/bin/docker-entr ...   Up (health: starting)   3306/tcp                                                harbor-jobservice    harbor/start.sh                 Up                                                                              harbor-log           /bin/sh -c /usr/local/bin/ ...   Up (healthy)            127.0.0.1:1514->10514/tcp                               harbor-ui            /harbor/start.sh                 Up (health: starting)                                                           nginx                nginx -g daemon off;             Up (health: starting)   0.0.0.0:443->443/tcp,:::443->443/tcp,                                                                                                 0.0.0.0:4443->4443/tcp,:::4443->4443/tcp,                                                                                             0.0.0.0:80->80/tcp,:::80->80/tcp                        redis                docker-entrypoint.sh redis ...   Up                      6379/tcp                                                registry             /entrypoint.sh serve /etc/ ...   Up (health: starting)   5000/tcp                                                登录后复制
```

## 4、使用 harbor 管理镜像

登录 http://172.29.9.10/harbor/sign-in 用户：admin 密码：Harbor12345

![](https://oss-emcsprod-public.modb.pro/wechatSpider/modb_20220505_8cfce350-cc69-11ec-8739-fa163eb4f6be.png)

image-20210529195513648

![](https://oss-emcsprod-public.modb.pro/wechatSpider/modb_20220505_8d2be114-cc69-11ec-8739-fa163eb4f6be.png)

image-20210529195519143

## 5、新建一个仓库项目

我们新建一个名称为 xuegod-web 的项目，设置公开。 注意：**当项目设为公开后，任何人都有此项目下镜像的读权限**。命令行用户不需要“docker login”就可以拉取此项目下的镜像。

![](https://oss-emcsprod-public.modb.pro/wechatSpider/modb_20220505_8d5c359e-cc69-11ec-8739-fa163eb4f6be.png)

image-20210529204902290

![](https://oss-emcsprod-public.modb.pro/wechatSpider/modb_20220505_8d782146-cc69-11ec-8739-fa163eb4f6be.png)

image-20210529204915186

## 6、修改本地 docker 服务使用 http 协议和私有仓库通信

:warning: **这个是在docker客户端进行配置的，而不是在harbor机器上配置。**

```
#配置可信任（如果仓库是HTTPS访问不用配置）#在 daemon.json 中添加以下参数[root@harbor ~]# vim /etc/docker/daemon.json #创建此文件，并写入以下内容{"insecure-registries": [ "172.29.9.10" ] }#重启docker 服务[root@harbor ~]# systemctl daemon-reload && systemctl restart docker登录后复制
```

注：出现这问题的原因是：**Docker 自从 1.3.X 之后 docker registry 交互默认使用的是 HTTPS**，**但是搭建私有镜像默认使用的是 HTTP 服务**，所以需要加上这一行。 **不然 docker 没有办法往私有仓库中上传镜像，并且docker也没办法从是私有仓库拉取镜像，即docker login 私有仓库地址是会报错的。**

## 7、重启harbor服务，登录harbor仓库

> 注意：这里是把harbor机器当docker主机来进行测试使用的；

**之前重启 docker 服务时，把 harhor 服务也关了，所以需要再启动一下 harbor**。

**你可以使用 docker-compose 来启动或关闭 Harbor 服务。但必须在与 docker-compose.yml 相****同的目录中运行**。 compose \[kəmˈpəʊz\] 组成

```
[root@harbor harbor]# docker-compose stop[root@harbor harbor]# docker-compose start登录后复制
```

-   登录私有仓库
    

```
[root@harbor harbor]# docker login 172.29.9.10登录后复制
```

![](https://oss-emcsprod-public.modb.pro/wechatSpider/modb_20220505_8dadd598-cc69-11ec-8739-fa163eb4f6be.png)

image-20210529210357556

## 8、上传镜像到私有仓库中

### 1.查看当前本地镜像

![](https://oss-emcsprod-public.modb.pro/wechatSpider/modb_20220505_8dfd3430-cc69-11ec-8739-fa163eb4f6be.png)

image-20210529210547322

### 2.admin 登录harbor

```
[root@harbor harbor]# docker login 172.29.9.10登录后复制
```

![](https://oss-emcsprod-public.modb.pro/wechatSpider/modb_20220505_8e2b4348-cc69-11ec-8739-fa163eb4f6be.png)

image-20210529210622952

### 3.给本地镜像打 tag

tag 的名称，可以在 web 界面这里查看到。

![](https://oss-emcsprod-public.modb.pro/wechatSpider/modb_20220505_8e754858-cc69-11ec-8739-fa163eb4f6be.png)

image-20210529210659362

![](https://oss-emcsprod-public.modb.pro/wechatSpider/modb_20220505_8e99fb6c-cc69-11ec-8739-fa163eb4f6be.png)

image-20210529210722547

```
#举例docker tag SOURCE_IMAGE[:TAG] 172.29.9.10/xuegod-web/IMAGE[:TAG]docker push 172.29.9.10/xuegod-web/IMAGE[:TAG]登录后复制
```

```
#实际打的tag[root@harbor harbor]# docker tag vmware/redis-photon:v1.5.0 172.29.9.10/xuegod-web/vmware/redis-photon:v1.5.0登录后复制
```

### 4.将打好tag的本地镜像push到仓库

```
[root@harbor harbor]# docker push 172.29.9.10/xuegod-web/vmware/redis-photon:v1.5.0登录后复制
```

![](https://oss-emcsprod-public.modb.pro/wechatSpider/modb_20220505_8ec3a52a-cc69-11ec-8739-fa163eb4f6be.png)

image-20210529211050985

### 5.在 web 界面查看刚才上传到仓库中的镜像

![](https://oss-emcsprod-public.modb.pro/wechatSpider/modb_20220505_8ee73940-cc69-11ec-8739-fa163eb4f6be.png)

image-20210529211130801

![](https://oss-emcsprod-public.modb.pro/wechatSpider/modb_20220505_8f250fe0-cc69-11ec-8739-fa163eb4f6be.png)

image-20210529211146209

![](https://oss-emcsprod-public.modb.pro/wechatSpider/modb_20220505_8f43870e-cc69-11ec-8739-fa163eb4f6be.png)

image-20210529211203436

## 9、从私有仓库 pull 镜像

### 1.首先删除本地镜像

```
[root@harbor harbor]# docker rmi 172.29.9.10/xuegod-web/vmware/redis-photon:v1.5.0登录后复制
```

### 2.pull 私有仓库中的镜像并验证

![](https://oss-emcsprod-public.modb.pro/wechatSpider/modb_20220505_8f658886-cc69-11ec-8739-fa163eb4f6be.png)

image-20210529211417689

```
[root@harbor harbor]# docker pull 172.29.9.10/xuegod-web/vmware/redis-photon:v1.5.0登录后复制
```

![](https://oss-emcsprod-public.modb.pro/wechatSpider/modb_20220505_8f957f96-cc69-11ec-8739-fa163eb4f6be.png)

image-20210529211500278

拉取成功！

以上就是如何使用harbor搭建Docker私有仓库的全部过程了。

## QA

### 📍 docker-compose命令使用

```
docker-compose ps #查看all的容器docker-compose up -d #启动all的容器 == docker-compose startdocker-compose stop #关闭all的容器登录后复制
```

> :warning: 特别注意
> 
> **你可以使用 docker-compose 来启动或关闭 Harbor 服务。但必须在与 docker-compose.yml 相****同的目录中运行**。
> 
> 即使之前把docker-compose二进制文件移动到PATH路径里了，但也需要在合适的地方执行这个命令才行；

![](https://oss-emcsprod-public.modb.pro/wechatSpider/modb_20220505_8fc1b872-cc69-11ec-8739-fa163eb4f6be.png)

image-20220228222152938

### 📍 不同harbor软件包的方式有点区别

> 注意：不同harbor软件包的方式有点区别，但是大体上是差不多的；

-   harbor-offline-installer-v1.5.0.tgz
    

这里配置的是`harbor.cfg`  
文件。

![](https://oss-emcsprod-public.modb.pro/wechatSpider/modb_20220505_9006f126-cc69-11ec-8739-fa163eb4f6be.png)

image-20220228222440303

-   harbor-offline-installer-v1.10.10.tgz
    

![](https://oss-emcsprod-public.modb.pro/wechatSpider/modb_20220505_904cef78-cc69-11ec-8739-fa163eb4f6be.png)

image-20220228222614706

![](https://oss-emcsprod-public.modb.pro/wechatSpider/modb_20220505_908fe238-cc69-11ec-8739-fa163eb4f6be.png)

image-20220228222547143

### 📍 关于如何修改harmor密码，感觉很繁琐的样子。。。(修改了好几次都没成功。。。)

![](https://oss-emcsprod-public.modb.pro/wechatSpider/modb_20220505_90b824aa-cc69-11ec-8739-fa163eb4f6be.png)

image-20220228225303456

-   自己按网上这个修改了，但是没生效，很奇怪。。。
    

http://www.javashuo.com/article/p-mwfuezme-gm.html

![](https://oss-emcsprod-public.modb.pro/wechatSpider/modb_20220505_90d3e24e-cc69-11ec-8739-fa163eb4f6be.png)

image-20220228230654307

### 📍 还有个问题，如何卸载harbor服务呢？

![](https://oss-emcsprod-public.modb.pro/wechatSpider/modb_20220505_90faf852-cc69-11ec-8739-fa163eb4f6be.png)

image-20220228230554885

![](https://oss-emcsprod-public.modb.pro/wechatSpider/modb_20220505_91327db8-cc69-11ec-8739-fa163eb4f6be.png)

image-20220228230617891

### 📍 还有个问题：这个harbor的版本号怎么奇奇怪怪的。。。

怎么最新的版本号是v1.10开头的呢，难道是1版本和2版本同时迭代的吗？？？。。。。

https://github.com/goharbor/harbor/tags

![](https://oss-emcsprod-public.modb.pro/wechatSpider/modb_20220505_916232f6-cc69-11ec-8739-fa163eb4f6be.png)

image-20220228230422475

## 总结

关于使用harbor搭建Docker私有仓库实验过程没有什么难度，但能否最终测试出实验效果，需要读者具有一定的细心能力。

好了，今天实战到此结束，我们下期见。

![](https://oss-emcsprod-public.modb.pro/wechatSpider/modb_20220505_9191f400-cc69-11ec-8739-fa163eb4f6be.png)

image-20210530081930963