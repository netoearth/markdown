## Harbor私有镜像仓库搭建

[![](https://upload.jianshu.io/users/upload_avatars/8387919/a1cb75af-7a16-445c-b03c-49671eb851e4.jpg?imageMogr2/auto-orient/strip|imageView2/1/w/96/h/96/format/webp)](https://www.jianshu.com/u/d27f9b1a1a02)

2020.05.27 23:14:15字数 256阅读 909

![](https://upload-images.jianshu.io/upload_images/8387919-4cdb94cb9379f53f.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

> Harbor是VMware公司最近开源的企业级Docker Registry项目, 其目标是帮助用户迅速搭建一个企业级的Docker registry服务。

### 前置条件

-   Linux host: docker 17.06.0-ce+ and docker-compose 1.18.0+ .

> docker和docker-compse的安装自行百度，注意版本

-   下载离线安装包 [Harbor release](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Fvmware%2Fharbor%2Freleases)

### 下载并解压

```
wget https://github.com/goharbor/harbor/releases/download/v2.0.0/harbor-offline-installer-v2.0.0.tgz
github网速贼慢，耐心等待。。。
tar -zxvf harbor-offline-installer-v1.8.1.tgz -C /opt/app

```

### 配置

```
cd /opt/app/harbor
./prepare
发现生成了docker-compose.yml文件
vim vim harbor.yml
修改一下内容：

hostname: harbor.***.com
# http related config
http:
 port: 9292
 
harbor_admin_password: Harbor12345
database:
    password: root123

```

更多配置参考官方文档: [configure-yml-file](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Fgoharbor%2Fharbor%2Fblob%2Fmaster%2Fdocs%2Finstall-config%2Fconfigure-yml-file.md)

### 安装

```
./install.sh

查看docker-compose安装的docker镜像
root@huaheng:/opt/app/harbor# docker-compose ps | grep harbor
harbor-core         /harbor/start.sh                 Up                               
harbor-db           /entrypoint.sh postgres          Up      5432/tcp                 
harbor-jobservice   /harbor/start.sh                 Up                               
harbor-log          /bin/sh -c /usr/local/bin/ ...   Up      127.0.0.1:1514->10514/tcp
harbor-portal       nginx -g daemon off;             Up      80/tcp                   
registryctl         /harbor/start.sh    
```

### 访问

输入域名访问 [http://harbor.test.com](https://links.jianshu.com/go?to=http%3A%2F%2Fharbor.test.com)

![](https://upload-images.jianshu.io/upload_images/8387919-f8aa4582647979b9.png)

image.png

默认账号密码: `admin / Harbor12345`

![](https://upload-images.jianshu.io/upload_images/8387919-625419b8a2765d4a.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

### 创建项目 `mcgrady`

![](https://upload-images.jianshu.io/upload_images/8387919-0afda88f13829205.png)

这里不勾选公开，默认为私有项目

### 登录到docker私服

```
docker login harbor.test.com
输入用户名密码后，出现以下错误
Error response from daemon: Get https://harbor.huahengmxf.com:9200/v2/: http: server gave HTTP response to HTTPS client

是因为没有配置私服的地址

vim /etc/docker/daemon.json

{
  "registry-mirrors": ["https://n0st2wzy.mirror.aliyuncs.com"],
   "insecure-registries":["harbor.test.com:9292"]
}



[root@hadoop01 ~]#  docker login harbor.huahengmxf.com
Username: admin
Password: 
WARNING! Your password will be stored unencrypted in /root/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded

```

出现 sucess表示登录成功

### 推送镜像到`Harbor`仓库

这里找一个本地已有的`mysql`镜像作为测试进行推送  
看到新建的项目`mcgrady`中描述，推送镜像分为两步：  

![](https://upload-images.jianshu.io/upload_images/8387919-a8b3e675d64efcbe.png)

1.  tag

```
docker image ls
docker tag mysql:5.7.22  harbor.test.com:9292/mcgrady/mysql:5.7.22

```

2.  push

```
[root@hadoop01 ~]# docker push harbor.test.com:9292/mcgrady/mysql:5.7.22
The push refers to repository [harbor.test.com:9292/mcgrady/mysql]
a968f24d4187: Pushed 
f8cb294d5d80: Pushed 
489bddb9c55e: Pushed 
22b402e93939: Pushed 
8aeebb3964c1: Pushed 
94f8d8f5acbf: Pushed 
c0c26734fb83: Pushed 
4801a487d51a: Pushed 
aae63f31dee9: Pushed 
6f8d38b0e2b6: Pushed 
cdb3f9544e4c: Pushed 
5.7.22: digest: sha256:e744510d4d03fddd1162651312afd1e591cf33b051c6f29ed64b9a3e64b97aa7 size: 2621


```

![](https://upload-images.jianshu.io/upload_images/8387919-2b67f3eaa99901e5.png)

刷新`harbor`管控台可以看到刚刚推送的镜像  

![](https://upload-images.jianshu.io/upload_images/8387919-0a82b3d59f834a5f.png)

客户端重新拉取新推送的镜像

```
[root@hadoop01 ~]# docker pull harbor.test.com:9292/mcgrady/mysql:5.7.22
5.7.22: Pulling from mcgrady/mysql
be8881be8156: Pull complete 
c3995dabd1d7: Pull complete 
9931fdda3586: Pull complete 
bb1b6b6eff6a: Pull complete 
a65f125fa718: Pull complete 
2d9f8dd09be2: Pull complete 
37b912cb2afe: Pull complete 
79592d21cb7f: Pull complete 
00bfe968d82d: Pull complete 
79cf546d4770: Pull complete 
2b3c2e6bacee: Pull complete 
Digest: sha256:e744510d4d03fddd1162651312afd1e591cf33b051c6f29ed64b9a3e64b97aa7
Status: Downloaded newer image for harbor.test.com:9292/mcgrady/mysql:5.7.22
[root@hadoop01 ~]# docker images | grep mysql
harbor.huahengmxf.com:9292/mcgrady/mysql   5.7.22              6bb891430fb6        22 months ago       372MB

```

![](https://upload-images.jianshu.io/upload_images/8387919-bef4b8936ea49bae.png)

参考文档：

-   [https://github.com/goharbor/harbor](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Fgoharbor%2Fharbor)

更多精彩内容，就在简书APP

"小礼物走一走，来简书关注我"

还没有人赞赏，支持一下

[![  ](https://upload.jianshu.io/users/upload_avatars/8387919/a1cb75af-7a16-445c-b03c-49671eb851e4.jpg?imageMogr2/auto-orient/strip|imageView2/1/w/100/h/100/format/webp)](https://www.jianshu.com/u/d27f9b1a1a02)

总资产346共写了3.7W字获得405个赞共99个粉丝