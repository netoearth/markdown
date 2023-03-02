Docker 私有仓库搭建（docker registry/nexus/harbor）

___

## 一、仓库介绍

Docker比较流行使用的三种私有仓库

1、 Docker-registry 这是docker hub 提供的一直私有仓库解决方案。没有图形化界面

2、 Nexus 这个更多用于存放jar包的一个仓库，也能存储docker image，使用过，由于一个版本一个版本的保存和叠加，清理起来比较麻烦。

3、 Harbor 这是个带有图形化界面的工具，用户管理，及查看更加方便

```
系统环境说明：
 Centos7.9 
 Docker Version: 20.10.9
```

官方下载地址： [https://github.com/goharbor/harbor/releases](https://link.zhihu.com/?target=https%3A//github.com/goharbor/harbor/releases)

![](https://pic3.zhimg.com/v2-d2690e5f7bb40f14bba72bb752ba22c6_b.jpg)

Harbor官方分别提供了在线版（不含组件镜像，相对较小）和离线版（包含组件镜像，相对较大），这里我们下载离线版的。

## 三、生产HTTPS证书

官方https生成文档：[https://goharbor.io/docs/1.10/install-config/configure-https/](https://link.zhihu.com/?target=https%3A//goharbor.io/docs/1.10/install-config/configure-https/)

## 四、docker-compose安装

官方docker-compose安装教程：[https://docs.docker.com/compose/install/](https://link.zhihu.com/?target=https%3A//docs.docker.com/compose/install/)

由于网络问题，多运行两遍：

```
 curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```

赋权：

```
chmod +x /usr/local/bin/docker-compose
```

可定义全局变量或者软链接，方便执行：

```
ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
```

验证：

## 五、安装Harbor

### 1、解压并配置

```
tar xvf harbor-offline-installer-v1.10.9.tgz
cd harbor/
```

编辑harbor.yml，修改hostname、https证书路径、admin密码（可选）

![](https://pic1.zhimg.com/v2-d36d799a21aa9171a1c2a2e39de2aa64_b.jpg)

### 2、运行install.sh

___

看到下面表示成功安装

```
✔ ----Harbor has been installed and started successfully.----
```

### 3、访问harbor 默认https

地址：[https://10.2.10.69/](https://link.zhihu.com/?target=https%3A//10.2.10.69/)

账号密码按配置文件： admin/Harbor12345

![](https://pic4.zhimg.com/v2-c73611539725a96dd51e3af5e7643f8f_b.jpg)

至此、算安装成功。

### 4、一些操作说明

重启：

```
docker-compose -f docker-compose.yml down
docker-compose -f docker-compose.yml up -d
```

默认volume bind 在 /data 下面

查看运行

```
# docker-compose ps
 Name Command State Ports 
-----------------------------------------------------------------------------------------------------------------------------------------
harbor-core /harbor/harbor_core  Up (healthy) 
harbor-db /docker-entrypoint.sh Up (healthy) 5432/tcp 
harbor-jobservice /harbor/harbor_jobservice ... Up (healthy) 
harbor-log /bin/sh -c /usr/local/bin/ ... Up (healthy) 127.0.0.1:1514->10514/tcp  
harbor-portal nginx -g daemon off; Up (healthy) 8080/tcp 
nginx nginx -g daemon off; Up (healthy) 0.0.0.0:80->8080/tcp,:::80->8080/tcp, 
 0.0.0.0:443->8443/tcp,:::443->8443/tcp 
redis redis-server /etc/redis.conf Up (healthy) 6379/tcp  
registry /home/harbor/entrypoint.sh Up (healthy) 5000/tcp 
registryctl /home/harbor/start.sh Up (healthy)   
```