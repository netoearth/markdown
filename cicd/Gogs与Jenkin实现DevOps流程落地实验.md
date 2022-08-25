## 1\. docker-compose 安装jenkins

docker-compose.yml

```
version: "3"
services:
  jenkins:
    image: jenkins/jenkins:lts
    restart: always
    user: root
    privileged: true
    ports:
      - "8080:8080"  //这里端口默认内部为8080，最好不要改动
      - "50000:50000"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - /var/jenkins_home:/var/jenkins_home
      - /etc/localtime:/etc/localtime:ro
```

创建好对应文件后运行：

打开对应地址：

你会看见一张图叫你填写一个系统管理密码串，它在

```
/var/jenkins_home/secrets/initialAdminPassword
```

注意具体路径以你看见图片中的路径为准，并且最好是把路径映射到主机中，即加入volumes中。

请运行命令即可得到秘钥串：

```
cat /var/jenkins_home/secrets/initialAdminPassword
```

完了以后构建一个项目再进入插件管理中选择gogs：

![](https://tva1.sinaimg.cn/large/006y8mN6gy1g7jxhygmnzj327m0sw43y.jpg)

点击 直接安装 即可。

___

## 2\. docker-compose 安装 Gogs + Mysql

### 对应 docker-compose.yml

```
version: "3"
services:
  mysql:
    image: mysql:5.7
    restart: always
    environment:
      - MYSQL_ROOT_PASSWORD=gogs
      - MYSQL_USER=gogs
      - MYSQL_PASSWORD=gogs
      - MYSQL_DATABASE=gogs
    ports:
      - "10036:3306"
    volumes:
      - ./mysql:/var/lib/mysq
    networks:
      gogs_app:
        aliases:
          - mysql
  gogs:
    image: gogs/gogs:latest
    container_name: gogs
    restart: always
    ports:
      - "8082:3000"
      - "10022:22"
    depends_on:
      - mysql
    volumes:
      - ./data:/data
    networks:
      gogs_app:
        aliases:
          - gogs_main
networks:
  gogs_app:
```

创建好对应yml文件后运行：

成功后浏览器打开对应地址：

首次运行安装会让你进入路径 /install 配置安装页面填写相应数据：

![](https://tva1.sinaimg.cn/large/006y8mN6gy1g7jxl074j9j30yh0u00zr.jpg)

![](https://tva1.sinaimg.cn/large/006y8mN6gy1g7jxma004tj310p0u0dki.jpg)

启动成功后创建一个代码仓库即可，记住在设置中加入自己电脑的 ssh 秘钥不然你 push 不上来的：

![](https://tva1.sinaimg.cn/large/006y8mN6gy1g7jxnn6q3aj31d20u0dl3.jpg)

### 添加web钩子：

![](https://tva1.sinaimg.cn/large/006y8mN6gy1g7jy6eg4a4j31as0u046p.jpg)

推送地址的格式为：

```
http(s)://<你的Jenkins地址>/gogs-webhook/?job=<你的Jenkins任务名>
```

你可以点击测试推送，看见推送成功即可。

___

## 3\. Jenkins配置构建远程服务器

> （1）、进入jenkins平台 -> 安装插件 -> 搜索 Publish Over SSH 并安装

同上安装gogs插件过程一样，搜索 Publish Over SSH 点击直接安装即可。

你可以先选择安装完毕后自动重启jenkins（记得安装完后要重启哦）

安装好重启以后添加你的ssh登录秘钥，这里就算只有一台机器也是需要进行认证添加的，不然会报登录服务器授权错误问题

1.  ssh-keygen -f ssh 生成公钥和私钥
2.  cat ~/.ssh/ssh.pub 查看公钥
3.  vim ~/.ssh/authorized\_keys 登录服务器B将服务器A生成的公钥加入authorized\_keys （这里就是自己给自己添加即可）
4.  cat ~/.ssh/jenkins 查看私钥并填入下图中 首先进入系统设置：

![](https://tva1.sinaimg.cn/large/006y8mN6gy1g7kaq6wa0jj31mi0u0175.jpg)

![](https://tva1.sinaimg.cn/large/006y8mN6gy1g7kal9ddghj31qi0u0wld.jpg)

## 4\. 构建仓库代码相关配置

进入项目test配置管理：

![](https://tva1.sinaimg.cn/large/006y8mN6gy1g7jyx5yt8dj31tm0u00yb.jpg)

设置好你的项目git地址url和分支。

## 5\. 安装Go 编译工具

Golang Build-env

因为默认的Jenkins镜像是不带有Go的编译工具的，所以我们很有必要安装一个Go插件Go-Plugin-Jenkins 具体步骤如下：

-   插件管理中安装构建包 Go
-   在全局工具设置中配置启动go选定版本
-   再到test项目中勾选go即可

## 6\. 触发钩子安装插件 Generic Webhook Trigger Plugin

请参考下面这篇。

> 主要解决web钩子触发自动push构建无效的问题

但是创建gogs web钩子时请填写如下格式：

```
http://47.99.212.97:8080/generic-webhook-trigger/invoke?token=xxx
```

[Gogs webhook实现git push 远程分支后自动触发Jenkins构建](https://blog.csdn.net/cheungzz/article/details/80585454)

> 参考

[Jenkins+Gogs搭建自动化部署平台](https://www.jianshu.com/p/14e356cf8bb4)

[docker+jenkins+golang持续集成实践](https://segmentfault.com/a/1190000014788819)

[Jenkins使用SSH 脚本运行Dockerfile构建Go项目并执行](https://www.cnblogs.com/Survivalist/p/11339924.html)

[Jenkins配置Gogs webhook插件](https://www.cnblogs.com/stulzq/p/8629720.html)

___