之前为了搭建个人博客，在django-cms 和 wordpress之间犹豫了很久，最后决定选择相对较成熟且部署较为简单的wordpress。  
wordpress是目前使用范围最广泛的建站cms，可以让所以人都能很轻松地建立网站、博客或应用程序。  
目前最新版本5.7，官方建议服务器具备php7.4以上版本，最好搭配apache或者nginx来使用。

**写在前面： 若嫌本文太长懒得看，也可以直接跳到文末，下载demo版简单修改后直接部署**

## 1    部署方式简介

一般情况下，部署lnmp时可以选择使用普通的编译安装、二进制部署、yum/apt 包管理器部署，或者3者相结合的方式，部署教程网上很多，此处不再赘述。

当然也可以采用docker容器的方式来部署，这样更灵活一些。不论是日常管理还是服务器到期后迁移等，都很方便。

docker部署时有2种常见方式:  
① 将nginx、php-fpm、mysql全部放置到同一个容器内，宿主机对外只暴露nginx的443和80端口  
② 将nginx、php-fpm、mysql分别构建3个容器，最后使用docker的bridge网络将3者互联，宿主机对外同样只暴露443和80端口

上述2种方式实际操作起来都可以, 但是docker官方更建议 "**一个容器只运行一个进程服务**" 所以本文采用方法②

## 2    容器集群编排的3种主要技术

接上文，既然决定分开部署，就又出现了一个问题：  
每当lnmp服务要启动、 停止、重启时，都需要手动执行docker命令，管理略有些混乱。  
题外话：本文中lnmp只包含3个容器，若遇到其他一些稍复杂的业务，例如微服务，每次都需要启动好几十个容器，并且容器启动顺序也有要求，这时候光想想就让人头大

互联网各大佬们也考虑到这种情况，并且提出了解决方案：**容器编排技术**  
顾名思义，容器编排就是通过一些手段，对容器进行批量管理，定义其生命周期内的所有行为

其中docker官方提出了docker-compose(单节点) 和 docker swarm(集群化)  
当然还有谷歌提出的更重量级的kubernetes(集群化)，也就是k8s

其中docker swarm没有干过k8s，基本已经凉凉了，毕竟docker官方都已经官宣支持k8s了。  
(似曾相识的感觉：微软官宣自家的edge浏览器采用谷歌内核，说真的，自从edge更换内核后，出奇的好用，我都从谷歌换到这儿了)

目前容器编排基本有2种常见操作：  
① 如果在**单台**服务器上部署多个容器，一般都采用docker-compose，更简单方便  
② 如果在**多台**服务器上部署多个容器，一般都会搭建k8s平台，更强大  
本文仅仅搭建个lnmp，docker-compose都绰绰有余，暂且不聊k8s了

## 3    docker-compose 部署

## 3.1    目录结构

其实简单来说，相比于docker本身，docker-compose仅仅是将docker容器的创建、部署等过程所使用的命令全部整合到一个yml配置文件中（docker-compose.yml）

话不多说，直接上图

```
docker-compose_lnmp_wp   # 本次docker-compose的总目录
├── conf                 # 配置文件
│   ├── nginx            # nginx配置文件目录,预先从docker镜像中复制出来,并修改了配置
│   └── ssl              # 放置https 证书的目录,推荐1个免费的ssl证书申请网站( freessl.cn )
├── docker-compose.yml   # docker-compose的主体, 需要构建的所有信息都在其中定义
├── dockerfile            
│   └── php-fpm          # 需要先重构php-fpm镜像后再使用，重构所需dockerfile文件在此目录下
├── html                 # 前端页面目录 需要分别映射到nginx容器和php-fpm容器中
│   └── wordpress        # wp官网下载的包,解压到这里就可以了,注意修改属主属组
└── mariadb_data         # 数据库目录,需要映射到数据库容器中, 数据库的所有数据将全部存放在此
```

这是本文中docker-compose目录的结构，我在服务器上新建了一个docker-compose\_lnmp\_wp目录，用于放置本文用到的所有东西  
注意设置文件目录权限归属，html 和conf目录的属主是www-data

文末会将整个目录全部打包上传，懒人解压后，做一些简单修改即可直接部署，不用再重复造轮子  

## 3.2    准备

### 3.2.1    配置文件

配置文件按理来说需要有3个，分别是nginx、php-fpm以及mariadb。

nginx的配置文件的话，可以先docker run 新建一个nginx容器，将其中的配置文件（默认是容器内/etc/nginx目录）使用docker cp命令全部复制出来，根据自身需要做一些修改即可。

php-fpm容器和mariadb容器的话，默认的配置就可以了。毕竟只用来搭建博客，实在懒得再详细优化。  
若有需求也可使用类似nginx的方法将php-fpm和mariadb容器的配置文件也复制出来，修改优化之后在docker-compose.yml的相应的容器配置中添加映射即可

### 3.2.2    容器镜像

共需要3个容器镜像：nginx、php-fpm、mariadb（mysql也行）

**若不提前准备也不会有影响，docker引擎会自动拉取所需镜像；**  
**但若是国内服务器，且没有配置镜像加速，拉取会很慢很慢。。。**  
（阿里云有提供免费的docker容器镜像加速，具体可以百度下，很多教程）

本文中nginx镜像使用目前最新的 1.19.8 版本； mariadb使用最新的 10.5.9 版本；  
php-fpm使用wordpress官方的推荐的版本 php:7.4.16-fpm  
但是由于默认没有启用mysqli等模块，所以连接数据库会报错，需要进入容器启用，这很麻烦  
所以我的解决方法是，在官方镜像的基础上使用docker build 重新构建镜像，一劳永逸

**docker-compose.yml中可以使用build关键字来根据指定的dockerfile文件直接构建镜像，不需要事先专门去构建**

附注：docker-compose.yml中，build和image 不能同时存在于一个容器。否则docker引擎会迷茫：“我究竟该听哪个的？”

### 3.2.3    ssl证书

基于安全考虑，我需要我搭建的博客必须使用https协议访问  
毕竟http很容易被反代，谁也不想自己辛辛苦苦搭的站点，被别人一个nginx proxy直接反代替换成他自己的内容，还宣称是他自己站点

具体ssl证书申请方法也很简单，完全免费。  
首先需要有自己的域名（必须是域名，不是xxx.f3322.net之类的主机记录）  
随后去 [https://freessl.cn/](https://freessl.cn/) 按照网站流程走就可以，这样生成的证书有1年的有限期

附注：上边儿这个申请时按照国家法律，需要报备手机号的，若你不想报备，也可以去它的国外版  
[https://freessl.org/](https://freessl.org/) 但是这里申请的免费版只有3个月有限期，到期需要再次申请。

总之，申请成功之后，可以下载一个zip 包，大概是下边这个结构

```
└── caojie-blog
    ├── apache
    ├── iis
    ├── nginx
    └── tomcat
```

freessl网站很贴心的直接准备了4种常用的格式，因为本文使用nginx，所以直接使用nginx目录下的各证书映射到对应的nginx容器即可

### 3.2.4    wordpress 前端代码

一般到wordpress官网下载.tag.gz压缩包到服务器上，解压到前端目录下即可。  
本文需要解压到 docker-compose\_lnmp\_wp/html 下， 并命名为wordpress

最新版本 [https://cn.wordpress.org/latest-zh\_CN.tar.gz](https://cn.wordpress.org/latest-zh_CN.tar.gz)

## 3.3    主控代码

### 3.3.1    编写php-fpm的dockerfile文件

```
FROM php:7.4.16-fpm
LABEL maintainer="https://caojie.blog"
ADD https://github.com/mlocati/docker-php-extension-installer/releases/latest/download/install-php-extensions /usr/local/bin/
RUN chmod +x /usr/local/bin/install-php-extensions && sync 
RUN install-php-extensions mysqli pdo_mysql gd exif imagick zip
```

install-php-extensions 是个类似apt、yum的包管理工具，只不过其对象是php的各个模块，专为dcoker设计，在github上有25k的star，具体详情可以参考我的另一篇文章  
[docker build 重构php-fpm镜像（增加新扩展）](https://caojie.blog/?p=1615 "docker build 重构php-fpm镜像（增加新扩展）")  
或者直接访问github上 install-php-extensions 项目，地址如下：  
[https://github.com/mlocati/docker-php-extension-installer](https://github.com/mlocati/docker-php-extension-installer "https://github.com/mlocati/docker-php-extension-installer")

### 3.3.2    编写docker-compose.yml

话不多说，直接上成品。

```
version: "3"
services:

  nginx_wp:
    image: nginx:1.19.8
    ports:
      - "80:80"
      - "443:443"
    networks:                   # 将此容器接入网络lnmp_wp
      - lnmp_wp
    volumes:      
      - ./conf/nginx:/etc/nginx
      - ./conf/ssl:/etc/ssl
      - ./html:/var/www/html
    restart: always
    depends_on:
      - mariadb_wp
      - php-fpm_wp

  php-fpm_wp:
    build:
      context: ./dockerfile/php-fpm
      dockerfile: Dockerfile
    volumes:
      - ./html:/var/www/html
    networks:
      - lnmp_wp
    restart: always
    depends_on:
      - mariadb_wp

  mariadb_wp:
    image: mariadb:10.5.9
    volumes:
      - ./mariadb_data:/var/lib/mysql
    environment:
      MYSQL_ROOT_PASSWORD: '123456'
      MYSQL_DATABASE: 'wordpress'
      MYSQL_USER: 'wordpress'
      MYSQL_PASSWORD: '123456'
    command: mysqld  --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci
    networks:
      - lnmp_wp
    restart: always
    privileged: true 
networks:
  lnmp_wp:                        # 创建一个新的网络，名为 lnmp_wp 
    driver: bridge                # 网络类型为桥接
    ipam:
      config:
        - subnet: '172.31.0.0/16'  # 定义网段
```

一些参数介绍：

① version: "3"  
docker-compose版本  
docker-compose yaml版本选择  
[https://docs.docker.com/compose/compose-file/compose-versioning/](https://docs.docker.com/compose/compose-file/compose-versioning/)

② depends\_on:  
指定依赖关系，表面本容器要在哪些容器成功启动后才能启动

③ restart: always  
容器有问题终止后会自动尝试重启

④ privileged: true  
授予该容器宿主机的真实root权限，否则容器root用户只是宿主机上的普通用户；  
在这里配置此参数只是为了防止有意外的权限问题，可以尝试去掉此参数。

⑤  volumes:   
挂载卷，既将宿主机的文件或目录与容器做映射  
一般都这样使用 **宿主机路径:容器内路径** ； 也可以 **直接指明自定义的挂载卷。**  
需要注意的是，指明宿主机路径来映射时，即便使用相对路径，也需要用 ./ 来开头。  
否则docker无法分辨你输入的究竟是挂载卷的名字还是相对路径

## 3.4    启动并配置wordpress

### 3.4.1    启动docker-compose

首先进入总目录

```
cd docker-compose_lnmp_wp ;
```

然后一次启动所有容器

```
docker-compose up -d
```

![image.png](https://i0.wp.com/caojie.blog/wp-content/uploads/2021/04/post-1581-607c46917a1a0.png?w=760&ssl=1)  
![image.png](https://i0.wp.com/caojie.blog/wp-content/uploads/2021/04/post-1581-607c469376dc2.png?w=760&ssl=1)  
可以看到，首先按要求创建了网络，随后构建镜像，再按照配置的顺序，依次启动所有容器  
启动后通过以下命令查看本个docker-compose的运行状态，可以看到，只有443和80被映射到了宿主机，数据库和php的端口都仅仅在其所处的虚拟网络 lnmp-wp中![image.png](https://i0.wp.com/caojie.blog/wp-content/uploads/2021/04/post-1581-607c46947b90d.png?w=760&ssl=1)  
虚拟网络查看  
![image.png](https://i0.wp.com/caojie.blog/wp-content/uploads/2021/04/post-1581-607c4695cd739.png?w=760&ssl=1)  
由于我并没有事先拉取镜像，所以docker引擎自动拉取了所需镜像，并且根据dockerfile重新构建了php-fpm  
![image.png](https://i0.wp.com/caojie.blog/wp-content/uploads/2021/04/post-1581-607c4696cf05a.png?w=760&ssl=1)  
若要停止所有容器，则使用命令

```
docker-compose stop
```

若要停止并删除所有容器、网络、卷、镜像等，则使用命令

```
docker-compose down
```

### 3.4.2    安装wordpress

浏览器访问nginx中定义的站点，即可开始初始化  
![image.png](https://i0.wp.com/caojie.blog/wp-content/uploads/2021/04/post-1581-607c46985071d.png?w=760&ssl=1)  
![image.png](https://i0.wp.com/caojie.blog/wp-content/uploads/2021/04/post-1581-607c46993a0bc.png?w=760&ssl=1)  
![image.png](https://i0.wp.com/caojie.blog/wp-content/uploads/2021/04/post-1581-607c469a59a31.png?w=760&ssl=1)  
此时要检查目录权限  
![image.png](https://i0.wp.com/caojie.blog/wp-content/uploads/2021/04/post-1581-607c469b944a2.png?w=760&ssl=1)

至此，最基本的wp站点已搭建完成

## 4    日常维护及备份

使用docker部署的一大好处就是可以很方便的迁移到其他服务器  
迁移的3个步骤  
① 新服务器上部署docker-compose环境  
② 将整个项目文件全部复制到新服务器上  
③ cd进入文件，执行docker-compose up -d 即可

docker-compose构建完成后会自动加载数据库和前端页面，不需要额外的操作。  
整个站点的数据也不会有损失

备份其实可以搭配crontab 通过简单的脚本完成，此处不再赘述

## 5    太长不看版

为了方便懒人（主要是我自己），我搞了个demo版本，若有兴趣可以直接下载，简单更改即可使用

[docker-compose\_lnmp\_wp\_demo.tar](https://caojie.blog/wp-content/uploads/2021/03/docker-compose_lnmp_wp_demo.tar "docker-compose_lnmp_wp_demo")