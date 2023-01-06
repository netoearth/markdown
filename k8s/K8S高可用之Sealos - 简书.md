## K8S高可用之Sealos

[![](https://cdn2.jianshu.io/assets/default_avatar/6-fd30f34c8641f6f32f5494df5d6b8f3c.jpg)](https://www.jianshu.com/u/1d296c64c64a)

2022.03.25 08:35:18字数 1,107阅读 1,945

### Sealos基础

Sealos 是一个 Go 语言开发的简单干净且轻量的 Kubernetes 集群部署工具， Sealos 能很好的支持在生产环境中部署高可用的 Kubernetes 集群。

##### Sealos 特性与优势

1.  支持离线安装，工具与部署资源包分离，方便不同版本间快速升级。
2.  证书有效期默认延期至 99 年。
3.  工具使用非常简单。
4.  支持使用自定义配置文件，可灵活完成集群环境定制。
5.  使用内核进行本地负载，稳定性极高，故障排查也极其简单。
6.  最主要的优势是不需要翻墙出去！！！

##### 为什么不用 KeepAlived 和 HAProxy 实现集群高可用

无论是通过 KeepAlived 还是 HAProxy 进行高可用集群调度都会存在以下一些劣势。

1.  软件源不一致可能导致容器中安装的软件版本也不一致，进而会引起相应检查脚本不生效等故障。
2.  可能因为系统依赖库问题，在某些特定环境下就直接无法完成安装。
3.  只依靠检测 HAProxy 进程是否存活是无法保证集群高可用的，正确的检测方式应该是判断ApiServer 是否 healthz 状态。
4.  Keepalived 可能存在 Cpu 占满的情况。

##### 本地负载为什么不使用 Envoy 或者 Nginx 实现

Sealos 高可用实现是通过本地负载方式完成的。本地负载实现方式有多种，比如： IPVS 、 Envoy 、 Nginx 等，而 Sealos 采用的是通过内核 IPVS 来实现的。  
本地负载：在每个 Node 节点上都启动一个负载均衡，同时监听集群中的多个 Master 节点。  
Sealos 选择通过内核 IPVS 来实现主要有以下几个原因：  
1）如果使用 Envoy 等需要在每个节点上都跑一个进程，消耗更多资源。虽然 IPVS 实际上也会多跑一个 lvscare 进程 ，但是 lvscare 只是负责管理 IPVS 规则，原理和 Kube-Proxy 类似。真正的流量直接从内核层面走，不需要把数据包先走到用户态中去处理。  
2）使用 Envoy 存在启动优先级的问题，比如：Join 集群时，如果负载均衡没有建立，Kubelet 就会启动失败。使用 IPVS 则不会存在这样的问题，因为我们可以在 Join 集群前先建立好转发规则。

##### 为什么要定制 Kubeadm

1）解决默认证书有效期只有一年的问题。  
2）更方便的实现本地负载。  
3）核心的功能均集成到 Kubeadm 中了，Sealos 只管分发和执行上层命令，相对就更轻量了。

##### Sealos 执行流程

1.  通过 SFTP 或者 Wget 命令把离线安装包拷贝到目标机器上，包括所有 Master 和 Node 节点。
2.  在 Master 0 节点上执行 kubeadm init 命令。
3.  在其它 Master 节点上执行 kubeadm join 命令并设置控制面。这个过程中多个 Master 节点上的 Etcd 会自动组成一个 Etcd 集群，并启动相应控制组件。
4.  所有 Node 节点都加入到集群中，这个过程中会在 Node 节点上进行 IPVS 转发规则和/etc/hosts 配置。  
    Node 节点对 ApiServer 的访问均是通过域名进行的。因为 Node 节点需要通过 虚拟 IP 连接到多个 Master 上，但是每个 Node 节点的 Kubelet 与 Kube-Proxy 访问 ApiServer 的地址是不同的，所以这里使用域名来解析每个节点上 ApiServer 不同的 IP 地址。

### 部署准备

##### sealos官网

```
官网地址: 
https://sealyun.com/ 

企业级应用的集群离线包需要付费，我们只是学习，使用作者提供的免费离线包：
http://store.lameleg.com/
```

##### 安装文档

```
官网中文安装手册： https://github.com/fanux/sealos
```

##### sealos下载

```
可以下载sealos不同版本的二进制文件 
https://github.com/fanux/sealos/releases
```

##### K8S离线包

离线包包含所有二进制文件配置文件和镜像

```
非免费V1.16.0版本下载地址 
https://sealyun.oss-cn- beijing.aliyuncs.com/cf6bece970f6dab3d8dc8bc5b588cc18- 1.16.0/kube1.16.0.tar.gz 

免费V1.17.0版本下载地址 
https://sealyun.oss-cn- beijing.aliyuncs.com/413bd3624b2fb9e466601594b4f72072- 1.17.0/kube1.17.0.tar.gz 

非免费V1.18.0版本下载地址： 
https://sealyun.oss-cn- beijing.aliyuncs.com/7b6af025d4884fdd5cd51a674994359c- 1.18.0/kube1.18.0.tar.gz 

免费V1.18.1版本下载地址： 
https://sealyun.oss-cn- beijing.aliyuncs.com/7b6af025d4884fdd5cd51a674994359c- 1.18.0/kube1.18.0.tar.gz 

非免费V1.18.2版本下载地址： 
https://sealyun.oss-cn- beijing.aliyuncs.com/9a8299ea8016abe32e1564a44d5162e4- 1.18.2/kube1.18.2.tar.gz
```

##### Sealos 常用参数说明

```
--master Master 节点服务器地址列表 
--node Node 节点服务器地址列表 
--user 服务器 SSH 用户名 
--passwd 服务器 SSH 用户密码 
--pkg-url 离线包所在位置，可以是本地目录，也可以是一个 HTTP 地址 
--version 指定需要部署的 Kubernetes 版本 
--pk 指定 SSH 私钥所在位置，默认为 /root/.ssh/id_rsa 

Other flags: 

  --kubeadm-config string kubeadm-config.yaml 用于指定自定义 kubeadm 配置文件 
  --vip string virtual ip (default "10.103.97.2") 本地负载时虚拟 IP ，不推荐修改，集群外不可访问
```

### 部署集群

##### 节点信息

服务器用户名：root，服务器密码：123456

  

![](https://upload-images.jianshu.io/upload_images/23703037-d273889a2997dff2.png)

image.png

##### 安装相关环境依赖

```
通过 Sealos 进行 Kubernetes 集群部署，你需要先准备好以下环境。 
1.在所有要部署的机器上，先完成 Docker 的安装和启动。 
2.下载 Kubernetes 离线安装包。 
3.下载最新版本 Sealos。 
4.对所有服务器进行时间同步。 
5.系统支持：centos7.2以上，ubuntu16.04以上。内核推荐4.14以上。推荐配置：centos7.8 
6.主机名不可重复 
7.master节点CPU必须2C以上 
8.请使用sealos 3.2.0以上版本
```

**升级系统内核**

```
rpm -Uvh http://www.elrepo.org/elrepo-release-7.0-3.el7.elrepo.noarch.rpm 
yum --enablerepo=elrepo-kernel install -y kernel-lt 
grep initrd16 /boot/grub2/grub.cfg 
grub2-set-default 0 

reboot
```

**修改Cgroup Driver**

```
修改/etc/docker/daemon.json，新增：

"exec-opts": ["native.cgroupdriver=systemd"] 

重启docker服务：
systemctl daemon-reload 
systemctl restart docker 

查看修改后状态： 
docker info | grep Cgroup
```

**设置hostname**

```
cat <<EOF >>/etc/hosts 
192.168.198.121 sealos-master01 
192.168.198.122 sealos-master02 
192.168.198.123 sealos-master03 
192.168.198.124 sealos-node01 
192.168.198.125 sealos-node02 
192.168.198.126 sealos-node03 
EOF
```

##### 初始化安装

**多master**

```
将sealos二进制文件上传sealos-master01:/data 
将kubernates离线安装包上传sealos-master01:/data 
cd /data 

授权并移动到/usr/bin目录中 
chmod +x sealos && mv sealos /usr/bin 

多master HA: 
sealos init \ 
    --master 192.168.198.121 \ 
    --master 192.168.198.122 \ 
    --master 192.168.198.123 \ 
    --node 192.168.198.124 \
    --node 192.168.198.125 \ 
    --node 192.168.198.126 \ 
    --user root \ 
    --passwd 123456 \ 
    --version v1.17.0 \ 
    --pkg-url /data/kube1.17.0.tar.gz

单master多node: 
sealos init --master 192.168.198.121 \ 
    --node 192.168.198.124 \ 
    --node 192.168.198.125 \ 
    --node 192.168.198.126 \ 
    --user root \ 
    --passwd 123456 \ 
    --version v1.17.0 \ 
    --pkg-url /data/kube1.17.0.tar.gz
```

**kubectl命令自动补全**  
sealos默认已经帮我们安装命令补全功能。

```
echo "source <(kubectl completion bash)" >> ~/.bash_profile 
source ~/.bash_profile
```

### master节点操作

##### 增加master

```
sealos join --master 192.168.198.127 --master 192.168.198.128 

或者多个连续IP 
sealos join --master 192.168.198.127-192.168.198.128
```

##### 删除指定master节点

```
sealos clean --master 192.168.198.122 --master 192.168.198.123 

或者多个连续IP 
sealos clean --master 192.168.198.122-192.168.198.123
```

### node节点操作

##### 增加node

```
sealos join --node 192.168.198.127 --node 192.168.198.128 

或者多个连续IP 
sealos join --node 192.168.198.127-192.168.198.128
```

##### 删除指定node节点

```
sealos clean --node 192.168.198.125 --node 192.168.198.126 

或者多个连续IP
sealos clean --node 192.168.198.125-192.168.198.126
```

### 清理集群

更多精彩内容，就在简书APP

"小礼物走一走，来简书关注我"

还没有人赞赏，支持一下

[![  ](https://cdn2.jianshu.io/assets/default_avatar/6-fd30f34c8641f6f32f5494df5d6b8f3c.jpg)](https://www.jianshu.com/u/1d296c64c64a)

总资产1共写了54.0W字获得33个赞共19个粉丝

### 推荐阅读[更多精彩内容](https://www.jianshu.com/)

-   欢迎转载，转载请标明原文地址：https://www.jianshu.com/p/3de558d8b57a 一、环...
    
    [![](https://upload-images.jianshu.io/upload_images/4191539-92acc219dec18832.png?imageMogr2/auto-orient/strip|imageView2/1/w/300/h/240/format/webp)](https://www.jianshu.com/p/3de558d8b57a)
-   install-kubeadm/create-cluster-kubeadm/ control-planehttp...
    
    [![](https://upload.jianshu.io/users/upload_avatars/18908711/52a49abf-fc65-4475-9ff7-6edd29d67dc6?imageMogr2/auto-orient/strip|imageView2/1/w/48/h/48/format/webp)偷油考拉](https://www.jianshu.com/u/c3a29284ef10)阅读 5,511评论 0赞 1
    

-   引子 目前，kubeadm 已经支持了搭建高可用的 Kubernetes 集群，大大降低了搭建的难度，官方的文档也...
    
    [![](https://upload.jianshu.io/users/upload_avatars/5903030/b5ea8241-46eb-4f80-bab0-062f5a095368.jpg?imageMogr2/auto-orient/strip|imageView2/1/w/48/h/48/format/webp)猴子精h](https://www.jianshu.com/u/7fd36fa45e51)阅读 13,256评论 2赞 2
    
    [![](https://upload-images.jianshu.io/upload_images/5903030-6710f46745dd3bd9.png?imageMogr2/auto-orient/strip|imageView2/1/w/300/h/240/format/webp)](https://www.jianshu.com/p/8eb81d1674dc)
-   操作系统：CentOS Linux release 7.7.1908 (Core)docker版本：18.09.1...
    
-   部署一套完整的企业级K8s集群 一、准备环境 服务器要求： • 建议最小硬件配置：4核CPU、4G内存、50G硬盘...
    
    [![](https://upload-images.jianshu.io/upload_images/21199738-73a841a26a79ff56.png?imageMogr2/auto-orient/strip|imageView2/1/w/300/h/240/format/webp)](https://www.jianshu.com/p/b0138cf76882)
-   机器准备 系统初始配置 添加国内镜像源 软件安装、配置 使用kubeadm部署kubernetes 安装flann...
    
    [![](https://upload.jianshu.io/users/upload_avatars/24586457/1281b28b-200f-4ceb-acdd-fcd738a2801a?imageMogr2/auto-orient/strip|imageView2/1/w/48/h/48/format/webp)运维搬砖工](https://www.jianshu.com/u/80ddfe7d06de)阅读 1,875评论 0赞 11
    
    [![](https://upload-images.jianshu.io/upload_images/24586457-c8926ba8f652344f.png?imageMogr2/auto-orient/strip|imageView2/1/w/300/h/240/format/webp)](https://www.jianshu.com/p/dacfda2289eb)
-   介绍 在上一期HA k8s搭建介绍有说到，目前有两种比较火的HA集群，那么今天我们来说说第二种，也是目前比较主流的...
    
    [![](https://upload-images.jianshu.io/upload_images/6791900-49be694dc017e03a.png?imageMogr2/auto-orient/strip|imageView2/1/w/300/h/240/format/webp)](https://www.jianshu.com/p/a2701f02cfd3)
-   教程 然后就没有然后了。。 概述与设计原则 sealos旨在做一个简单干净轻量级稳定的kubernetes安装工具...
    
-   1.k8s集群的安装(kubeadm安装) 1.1 k8s的架构 从系统架构来看，k8s分为2个节点 Master...
    
    [![](https://upload.jianshu.io/users/upload_avatars/19901269/13e5f0c3-5642-491c-b3ff-379a74b5f401.jpg?imageMogr2/auto-orient/strip|imageView2/1/w/48/h/48/format/webp)唯爱熊](https://www.jianshu.com/u/5436d3cc4cc8)阅读 1,034评论 0赞 1
    
    [![](https://upload-images.jianshu.io/upload_images/19901269-16ea9218cdff5c5d.png?imageMogr2/auto-orient/strip|imageView2/1/w/300/h/240/format/webp)](https://www.jianshu.com/p/204985b799ee)
-   k8s的搭建难度很大，这里的教程是结合是视频，博客，官网各种折腾，耗时一个五一假期折腾出来的，遇到问题不要灰心，因...
    
    [![](https://upload.jianshu.io/users/upload_avatars/22844171/d37e8622-15fc-4d04-97d5-35bf296e1aac.jpg?imageMogr2/auto-orient/strip|imageView2/1/w/48/h/48/format/webp)笔记本一号](https://www.jianshu.com/u/5596294774f3)阅读 2,284评论 1赞 12
    
    [![](https://upload-images.jianshu.io/upload_images/22844171-1312a184e7a4c400.png?imageMogr2/auto-orient/strip|imageView2/1/w/300/h/240/format/webp)](https://www.jianshu.com/p/8e077225e4f7)
-   本文教你如何用一条命令构建 k8s 高可用集群且不依赖 haproxy 和 keepalived，也无需 ansi...
    
    [![](https://upload-images.jianshu.io/upload_images/1255061-2d0a6b33454b029b.jpeg?imageMogr2/auto-orient/strip|imageView2/1/w/300/h/240/format/webp)](https://www.jianshu.com/p/cfb88db213fe)
-   安装前准备 1. 一台或多台主机，这里准备三台机器 主节点 192.168.0.10 master ...
    
-   一、K8S 集群架构方案 Kubernetes 集群组件： etcd 一个高可用的 K/V 键值对存储和服务发现系...
    
    [![](https://upload.jianshu.io/users/upload_avatars/2865141/8914b1a2-7073-4d0a-a05c-0cb1a3070e10.jpg?imageMogr2/auto-orient/strip|imageView2/1/w/48/h/48/format/webp)小胡\_鸭](https://www.jianshu.com/u/702819649703)阅读 1,209评论 0赞 0
    
    [![](https://upload-images.jianshu.io/upload_images/2865141-b788bd6dd5541bd3.png?imageMogr2/auto-orient/strip|imageView2/1/w/300/h/240/format/webp)](https://www.jianshu.com/p/518d993b2163)
-   概述 Kubernetes(K8s) 作为当前最知名的容器编排工具，称得上是云原生(Cloud Native)时代...
    
    [![](https://upload-images.jianshu.io/upload_images/6925317-1cac2fd305462bb6.png?imageMogr2/auto-orient/strip|imageView2/1/w/300/h/240/format/webp)](https://www.jianshu.com/p/1db2613f5865)
-   \[TOC\] 1. 前置知识点 1.1 生产环境可部署Kubernetes集群的两种方式 1. Kubeadm 2....
    
    [![](https://upload-images.jianshu.io/upload_images/20968074-997ced08157d078e.png?imageMogr2/auto-orient/strip|imageView2/1/w/300/h/240/format/webp)](https://www.jianshu.com/p/e59a5ac15d82)
-   今天青石的票圈出镜率最高的，莫过于张艺谋的新片终于定档了。 一张满溢着水墨风的海报一次次的出现在票圈里，也就是老谋...
    
    [![](https://cdn2.jianshu.io/assets/default_avatar/2-9636b13945b9ccf345bc98d0d81074eb.jpg)青石电影](https://www.jianshu.com/u/aa52975c0a31)阅读 8,556评论 1赞 3
    
    [![](https://upload-images.jianshu.io/upload_images/680837-5490380964d6549d?imageMogr2/auto-orient/strip|imageView2/1/w/300/h/240/format/webp)](https://www.jianshu.com/p/aed76156f092)
-   一、jQuery简介 JQ是JS的一个优秀的库，大型开发必备。在此，我想说的是，JQ里面很多函数使用和JS类似，所...
    
-   字符串 1.什么是字符串 使用单引号或者双引号括起来的字符集就是字符串。 引号中单独的符号、数字、字母等叫字符。 ...
    
-   《闭上眼睛才能看清楚自己》这本书是香海禅寺主持贤宗法师的人生体悟，修行心得及讲学录，此书从六个章节讲述了禅修是什么...
    
    [![](https://upload.jianshu.io/users/upload_avatars/12400153/7ae71049-9f82-4d1f-afe8-2f1aca8f0e31.jpg?imageMogr2/auto-orient/strip|imageView2/1/w/48/h/48/format/webp)宜均](https://www.jianshu.com/u/7412fb0d06c8)阅读 7,576评论 1赞 25
    
-   偶然间从公众号里看见了小白训练营的课。就点进去看了看。刚开始的时候我觉得就是骗人的。后来一想，学费那么少。干嘛...