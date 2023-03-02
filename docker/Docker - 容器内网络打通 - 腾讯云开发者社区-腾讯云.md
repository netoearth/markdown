发布于2021-01-21 11:04:44阅读 8050

## 概述

通过Docker部署了，mysql还有mongodb。

发布服务端后发现不知道如何内网访问[数据库](https://cloud.tencent.com/solution/database?from=10680)，研究一下开搞。

## 为什么要打通容器目录

> 对于复杂的应用，不可避免需要多个服务部署在多个[容器](https://cloud.tencent.com/product/tke?from=10680)中，并且服务间存在相互间通信的情况。我不想在外网访问mysql，只在内网负责调用。

## 一、docker brctl

-   在安装好docker后，docker将创建一个linux网桥docker0，它在内核层连通了其他的物理或虚拟网卡，也就是所有容器和本地主机都放到同一个物理网络。我们可以通过 brctl 命令查看网桥的信息，brctl是需要自行安装的。

![](https://ask.qcloudimg.com/http-save/8128510/8j1j9rdwzt.png?imageView2/2/w/1620)

## 二、查看当前[宿主机](https://cloud.tencent.com/product/cdh?from=10680)中所有的docker网络

-   docker还会给我们创建三个网络：**bridge/host/none**。我们可以通过network ls命令查看当前宿主机中所有的docker网络。

![](https://ask.qcloudimg.com/http-save/8128510/nxdtsbn1kr.png?imageView2/2/w/1620)

## 三、配置docker网络

-   创建网络 docker network create \[name\]
-   其中，网桥bridge模式是在实际项目中常用的。接下来，以交互模式启动两个busybox容器。在没有指定相关网络的情况下，容器都会连接到默认的bridge网络。我们可以通过 **_\--network_ 参数**指定容器连接的网络。 docker run -p --name --network \[name\] -d images
-   启动容器后，检查当前默认网络情况。容器已经连接到了bridge网络，除此之外，还可以获取到指定容器的IP地址。

![](https://ask.qcloudimg.com/http-save/8128510/2be3fsy34u.png?imageView2/2/w/1620)

-   docker network inspect \[name\]

![](https://ask.qcloudimg.com/http-save/8128510/6fjiezzwt9.png?imageView2/2/w/1620)

![](https://ask.qcloudimg.com/http-save/8128510/48jst41r9q.png?imageView2/2/w/1620)

-   启动的时候可以加的参数
-   docker run -p --name --network \[name\] --network-alias \[name对应这个容器的别名随便起\]-d images

## 四、删除创建的network

-   docker network rm \[name\]

本文分享自作者个人站点/博客：https://blog.edlcloud.com/复制

如有侵权，请联系

cloudcommunity@tencent.com

删除。

### 相关文章

-   ### Docker - 容器内软件设置
    
-   ### Docker容器网络
    
    Docker在安装后自动提供3种网络，可以使用\`\`docker network ls\`命令查看
    
-   ### 如何拷贝Docker容器内的文件？
    
    某个项目容器需要添加 wkhtmltopdf 软件包用于处理html与pdf文件转换，由于默认的apt源服务器在国外，使用apt 安装 wkhtmltopdf ...
    
-   ### Docker容器内安装工具方式
    
-   ### Docker容器实战十：容器网络
    
    在最初的版本中，Docker的网络功能集成在Docker Daemon的代码中，这使得整体架构变得臃肿且缺乏灵活性，无法适应复杂的网络需求。为此，Docker公...
    
-   ### Docker 单容器网络
    
    docker network ls docker run -it --network=none busybox
    
-   ### Docker容器网络配置
    
    可以借助ip netns命令来完成对 Network Namespace 的各种操作。ip netns命令来自于iproute安装包，一般系统会默认安装，如果没...
    
-   ### Docker容器网络（七）
    
    在应用程序和网络之间是 Docker 网络，被亲切地称为容器网络模型 或 CNM（Container Network Model）。是 CNM 为您的 Dock...
    
-   ### Docker 容器的网络
    
    如果你通过 Docker 提供的用户指南，你应该已经完成了构建你的第一个 Docker 容器，并且运行了示例应用。
    
-   ### Docker容器内执行 jvm 分析工具命令
    
    目前我们公司使用的基本上都是java开发的后端，本文详细的介绍了公司java程序docker 包构建的演变过程，这里面不对java包本身的构建做过多的赘述。
    
-   ### Docker容器网络-基础篇
    
    Docker的技术依赖于Linux内核的虚拟化技术的发展，Docker使用到的网络技术有Network Namespace、Veth设备对、Iptables/N...
    
-   ### Docker容器内的监控命令数据修正思路
    
    思路概述：编写linux c代码，生成对应的动态链接库（so文件），通过LDPRELOAD实现对/proc文件系统访问的劫持。劫持之后，实现容器内正确的数据计算...
    
-   ### Docker系列教程15-Docker容器网络
    
    本文是篇翻译。原文：https://docs.docker.com/engine/userguide/networking/ 本节概述了Docker默认的网络行...
    
-   ### Docker网络模型以及容器通信
    
    Docker内置这三个网络，运行容器时，你可以使用该--network标志来指定容器应连接到哪些网络。
    
-   ### Docker容器网络连接配置
    
    Docker 允许通过外部访问容器或容器互联的方式来提供网络服务。由于容器的本质是一个进程，那么访问容器服务我们需要映射对应的端口。
    
-   ### 自定义docker容器网络
    
    1.通过bridge 驱动创建类似前面默认的 bridge 网络： docker network create --driver bridge my\_net 如...
    
-   ### Docker 容器之间网络的通信
    
    Docker在创建容器时有四种网络模式：bridge/host/container/none，bridge为默认不需要用–net去指定，其他三种模式需要在创建容...