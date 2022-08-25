## Jenkins+Gogs搭建自动化部署平台

[![](https://upload.jianshu.io/users/upload_avatars/1774338/d442691d-3606-4688-bc4c-b69844777c48.jpeg?imageMogr2/auto-orient/strip|imageView2/1/w/96/h/96/format/webp)](https://www.jianshu.com/u/d3b9df1ce6d9)

0.5892018.12.27 15:41:30字数 1,002阅读 12,891

## 1、安装Gogs

> （1）、配置数据库（以mysql为例）

```
  #创建gogs数据库
  create database gogs;
  
  #创建gogs数据库用户
  create user 'gogs'@'localhost' identified by 'your-password';
  grant all privileges on gogs.* to 'gogs'@'localhost';
  flush privileges;
```

> （2）、配置Git

```
#git版本需 >= 1.7.1
git version
git version 2.15.1

#创建git用户
sudo  adduser  git
#切换至git用户
su  git
```

> （3）、下载安装Gogs

-   返回根目录

-   下载安装包

```
wget  https://dl.gogs.io/0.11.79/gogs_0.11.79_linux_amd64.tar.gz
```

-   解压安装包

```
tar  -xzvf  gogs_0.11.79_linux_amd64.tar.gz
```

-   进入解压后的目录

-   编辑配置文件

```
vim ./scripts/init/debian/gogs

PATH=/sbin:/usr/sbin:/bin:/usr/bin
DESC="Go Git Service"
NAME=gogs
SERVICEVERBOSE=yes
PIDFILE=/var/run/$NAME.pid
SCRIPTNAME=/etc/init.d/$NAME
WORKINGDIR=/home/git/gogs #这个根据自己的目录修改
DAEMON=$WORKINGDIR/$NAME
DAEMON_ARGS="web"
USER=git  #如果运行gogs不是用的这个用户，修改对应用户
```

-   切换到root账户/复制命令/增加命令权限

```
sudo cp /home/git/gogs/scripts/init/debian/gogs /etc/init.d/

sudo chmod +x /etc/init.d/gogs
```

-   配置service命令

```
cp /home/git/gogs/scripts/systemd/gogs.service /etc/systemd/system/
```

-   启动Gogs

```
sudo service gogs start
或
sudo /home/git/gogs/gogs web
```

-   浏览器访问完成安装

```
http://localhost:3000/install

# Gogs配置文件在 /home/git/gogs/custom/conf/app.ini
# 进入安装页后按照提示填写完成最终安装~
```

![](https://upload-images.jianshu.io/upload_images/1774338-38ce2d2346ae2045.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

Gogs

## 2、安装jenkins

> （1）、安装JDK

> （2）、安装jenkins

**添加Jenkins库到yum库，Jenkins将从这里下载安装**

```
wget -O /etc/yum.repos.d/jenkins.repo http://pkg.jenkins-ci.org/redhat/jenkins.repo

rpm --import https://jenkins-ci.org/redhat/jenkins-ci.org.key

yum install -y jenkins
```

**如果不能安装就到官网下载jenkis的rmp包，官网地址（[http://pkg.jenkins-ci.org/redhat-stable/](http://pkg.jenkins-ci.org/redhat-stable/)）**

```
wget http://pkg.jenkins-ci.org/redhat-stable/jenkins-2.7.3-1.1.noarch.rpm

rpm -ivh jenkins-2.7.3-1.1.noarch.rpm
```

**配置jenkins端口号**

```
 vi /etc/sysconfig/jenkins

# 找到修改端口号：
JENKINS_PORT="8088"  此端口不冲突可以不修改
```

> （3）、启动jenkins

```
service jenkins start/stop/restart
```

-   安装成功后Jenkins将作为一个守护进程随系统启动
-   系统会创建一个“jenkins”用户来允许这个服务，如果改变服务所有者，同时-  
    需要修改/var/log/jenkins, /var/lib/jenkins, 和/var/cache/jenkins的所有者
-   启动的时候将从/etc/sysconfig/jenkins获取配置参数  
    默认情况下，Jenkins运行在8080端口，在浏览器中直接访问该端进行服务配置
-   Jenkins的RPM仓库配置被加到/etc/yum.repos.d/jenkins.repo

> （4）、打开jenkins

```
在浏览器中访问 http://IP:端口
首次进入会要求输入初始密码如下图， 
初始密码在：/var/lib/jenkins/secrets/initialAdminPassword
```

![](https://upload-images.jianshu.io/upload_images/1774338-26bd26abe727d838.png?imageMogr2/auto-orient/strip|imageView2/2/w/558/format/webp)

**选择“Install suggested plugins”安装默认的插件，下面Jenkins就会自己去下载相关的插件进行安装**

![](https://upload-images.jianshu.io/upload_images/1774338-e90832364d5a08d6.png?imageMogr2/auto-orient/strip|imageView2/2/w/558/format/webp)

![](https://upload-images.jianshu.io/upload_images/1774338-41ed02f96230268e.png?imageMogr2/auto-orient/strip|imageView2/2/w/558/format/webp)

**创建超级管理员账号**

![](https://upload-images.jianshu.io/upload_images/1774338-56ada88d9ffde0b1.png?imageMogr2/auto-orient/strip|imageView2/2/w/558/format/webp)

![](https://upload-images.jianshu.io/upload_images/1774338-7c7966a96132d401.png?imageMogr2/auto-orient/strip|imageView2/2/w/558/format/webp)

> （5）、创建项目

![](https://upload-images.jianshu.io/upload_images/1774338-4b91e5c5f7984961.png?imageMogr2/auto-orient/strip|imageView2/2/w/1132/format/webp)

**其它配置项先默认即可**

## 3、Jenkins配置Gogs webhook插件

> （1）、进入jenkins平台打开 系统管理 -> 管理插件 -> 可选插件，在右上角输入框中输入"gogs"来筛选插件：

![](https://upload-images.jianshu.io/upload_images/1774338-c31234611c96d3f8.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

> （2）、gogs中仓库配置

-   1、进入对应的仓库，点击仓库设置

![](https://upload-images.jianshu.io/upload_images/1774338-20def0c434ca2d1e.png?imageMogr2/auto-orient/strip|imageView2/2/w/1123/format/webp)

-   2、添加webhook

**点击 管理Web钩子 -> 添加Web钩子 -> 选择Gogs**

![](https://upload-images.jianshu.io/upload_images/1774338-3307b869943b91a2.png?imageMogr2/auto-orient/strip|imageView2/2/w/1078/format/webp)

-   3、添加如下配置：

![](https://upload-images.jianshu.io/upload_images/1774338-8b12403b6ed0ca49.png?imageMogr2/auto-orient/strip|imageView2/2/w/1005/format/webp)

**推送地址的格式为：**

```
http(s)://<你的Jenkins地址>/gogs-webhook/?job=<你的Jenkins任务名>
```

-   4、点击 推送测试，如 成功 会看到下推送记录

![](https://upload-images.jianshu.io/upload_images/1774338-a4bb30884ad26863.png?imageMogr2/auto-orient/strip|imageView2/2/w/792/format/webp)

## 4、Jenkins配置构建远程服务器

> （1）、进入jenkins平台 -> 安装插件 -> 搜索 Publish Over SSH 并安装

![](https://upload-images.jianshu.io/upload_images/1774338-8cdfe48a1befb97e.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

> （2）、重启Jenkins服务

> （3）、配置服务器密钥认证  
> 注：  
> A服务器：jenkins所在服务器 (192.168.100.101)  
> B服务器：目标线上服务器 (192.168.100.100)

-   1、服务器A上生成密钥

-   2、查看生成的公钥

-   3、登录服务器B将服务器A生成的公钥加入authorized\_keys

```
vim  ~/.ssh/authorized_keys
```

-   4、jenkins配置 Publish Over SSH

**进入Jenkins平台，点击 系统管理 -> 系统设置 -> 找到 Publish Over SSH 配置项**

```
# 查看并拷贝服务器A的私钥

cat ~/.ssh/jenkins
```

![](https://upload-images.jianshu.io/upload_images/1774338-aba1b6f70f5e3f36.png?imageMogr2/auto-orient/strip|imageView2/2/w/1066/format/webp)

**注：Jenkins SSH Key 这一栏默认会使用Jenkins管理员admin账户的密码，可以不填则设置为空密码**

-   5、项目Git项相关配置

**打开项目（test）配置页**

![](https://upload-images.jianshu.io/upload_images/1774338-a519c5abb2296f9a.png?imageMogr2/auto-orient/strip|imageView2/2/w/985/format/webp)

-   6、建构触发器配置

**构建触发器，以及构建环境都不需要配置，因为我们发布的是php代码！**

![](https://upload-images.jianshu.io/upload_images/1774338-5a101b6ffba5e0b7.png?imageMogr2/auto-orient/strip|imageView2/2/w/1126/format/webp)

-   7、构建项配置

**最核心的一步，选择"Send files or execute commands over SSH"**

![](https://upload-images.jianshu.io/upload_images/1774338-0dd619f111284136.png?imageMogr2/auto-orient/strip|imageView2/2/w/1162/format/webp)

**简单说明**

-   SSH Server，Name 选择对应的服务器，
-   Transfers, Source files填写**/**，表示全部文件
-   Remove prefix可以指定截掉的前缀目录，这里留空即可，
-   Remote directory指定远程服务器上代码存放路径，比如/data/wwwroot/[www.aaa.com](http://www.aaa.com/)
-   Exec command为文件传输完成后要执行的命令，比如可以是更改文件权限的命令，设置完成后点击 “Add Transfer Set”，如果还有另外的机器，可以点击 “Add Server”重复以上操作

## 5、推送自动构建测试

**Gogs上关联的仓库（test）master分支下push一条修改记录后，会发现jenkins上自动完成本地push的远程构建**

![](https://upload-images.jianshu.io/upload_images/1774338-1b6766bf6f9cb9dc.png?imageMogr2/auto-orient/strip|imageView2/2/w/1156/format/webp)

![](https://upload-images.jianshu.io/upload_images/1774338-45742be5bb1e5757.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

**查看远程服务器（B）目录情况**

![](https://upload-images.jianshu.io/upload_images/1774338-c44d105c61e75fbe.png?imageMogr2/auto-orient/strip|imageView2/2/w/466/format/webp)

线上服务器B

___

**OK~~ 整个Jenkins+Gogs自动化部署平台完成 ~~**

___

**注：**

-   1、私有库项目Jenkins中Git项授权配置

![](https://upload-images.jianshu.io/upload_images/1774338-962bf0ba63963e38.png?imageMogr2/auto-orient/strip|imageView2/2/w/1000/format/webp)

添加授权凭证

![](https://upload-images.jianshu.io/upload_images/1774338-661bed2ef2f47c16.png?imageMogr2/auto-orient/strip|imageView2/2/w/1055/format/webp)

添加授权凭证

![](https://upload-images.jianshu.io/upload_images/1774338-c969fb173b981d35.png?imageMogr2/auto-orient/strip|imageView2/2/w/976/format/webp)

授权成功

更多精彩内容，就在简书APP

"小礼物走一走，来简书关注我"

还没有人赞赏，支持一下

[![  ](https://upload.jianshu.io/users/upload_avatars/1774338/d442691d-3606-4688-bc4c-b69844777c48.jpeg?imageMogr2/auto-orient/strip|imageView2/1/w/100/h/100/format/webp)](https://www.jianshu.com/u/d3b9df1ce6d9)

总资产43共写了8134字获得80个赞共45个粉丝

### 推荐阅读[更多精彩内容](https://www.jianshu.com/)

-   原文：Jenkins Gitlab持续集成打包平台搭建 相关概念 Jenkins Jenkins，一个用Java编...
    
    [![](https://upload-images.jianshu.io/upload_images/1760830-4c8e848c67f9f20c.png?imageMogr2/auto-orient/strip|imageView2/1/w/300/h/240/format/webp)](https://www.jianshu.com/p/fee3503e9b3e)
-   废话不多说，开始实施... 一些基本知识需要自己实践完，进行自我补充和提高。在下一节会介绍原理。 文中所涉及到的l...
    
    [![](https://upload-images.jianshu.io/upload_images/4997925-bf0bd55d873df5e9.png?imageMogr2/auto-orient/strip|imageView2/1/w/300/h/240/format/webp)](https://www.jianshu.com/p/e6e88bc099e4)

-   上一篇大概介绍了JWT的用法，实现了一个简单的登录注册以及邮箱验证。而这一篇呢就负责把我们的项目部署到自己的服务器...
    
    [![](https://upload-images.jianshu.io/upload_images/2300716-9a42542f5fb58525.png?imageMogr2/auto-orient/strip|imageView2/1/w/300/h/240/format/webp)](https://www.jianshu.com/p/47ef444c74da)
-   1.Jenkins简介 Jenkins是一个可扩展的持续集成引擎，是一个CI系统。主要用于： 持续、自动构建/测试...
    
    [![](https://upload.jianshu.io/users/upload_avatars/1874274/33f619d29396.jpg?imageMogr2/auto-orient/strip|imageView2/1/w/48/h/48/format/webp)郭之源](https://www.jianshu.com/u/eaf33e662631)阅读 1,924评论 1赞 6
    
    [![](https://upload-images.jianshu.io/upload_images/1874274-9ed2ad74dbe9cce1.png?imageMogr2/auto-orient/strip|imageView2/1/w/300/h/240/format/webp)](https://www.jianshu.com/p/e3e8e692a1be)
-   去年夏天，云姐放弃了《神偷奶爸》，陪我去看新上映的《悟空传》，影片结束，云姐觉得没什么，倒是我后悔的不得了:早知道...
    
    [![](https://cdn2.jianshu.io/assets/default_avatar/2-9636b13945b9ccf345bc98d0d81074eb.jpg)淇畔芷影](https://www.jianshu.com/u/271177ac5a8d)阅读 176评论 0赞 2
    
-   目录 1、出现 2、成长 3、快乐 4、选择 5、迷失 6、寻踪 7、幸福 前言：我一直想写一些关于成长，关于选...
    
-   上星期我学金话筒，老师讲完课时，对我们说了一句话：“我们要举行大赛，想报名的都在微信上报名，初赛还是免费的。”我...
    
    [![](https://upload-images.jianshu.io/upload_images/6737965-a6a1bf8d15d94707.jpg?imageMogr2/auto-orient/strip|imageView2/1/w/300/h/240/format/webp)](https://www.jianshu.com/p/1fcad311dcec)