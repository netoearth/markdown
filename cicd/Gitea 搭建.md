Gitea 搭建

![1571386395115](https://ethanarea-1253140193.cos.ap-shanghai.myqcloud.com/blog_images/Gitea/1571386395115.png "1571386395115")

## 一、前言

最近团队里，需要进行项目代码的内部共享平台。所以打算利用公司电脑搭建一个基于git的代码平台。

经过一番考虑之后，选择了gitea作为搭建平台。

选择它的原因如下：

1.  由于公司台式机的机能所限，且还要应付其他的应用和工作。
2.  Gitlab固然很好很强大，但是我们小团队的访问量很低，轻量级的Gitea、Gogs完全满足使用
3.  部署方便快捷，在win10下即可快速部署。
4.  尝试部署Gogs失败（服务无法正常运行）

## 二、安装步骤

### 1\. git的部署

git部署教程请参考：[传送门](https://www.ethanarea.cn/2019/10/01/Win10%20%E4%B8%8Bgit%20%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA/)

### 2\. Gitea的下载

登陆官网下载： [传送门](https://dl.gitea.io/gitea/master/)，选择自己的所想搭建的平台，进行对应版本。

由于我选择win10系统作为部署平台，下载64位的win版：[gitea-master-windows-4.0-amd64.exe](https://dl.gitea.io/gitea/master/gitea-master-windows-4.0-amd64.exe)。

### 3\. Gitea的安装与部署

1.  将下好的exe文件放到你想安装的路径下，例如：E:DevEnvgitea
2.  修改文件名为gitea.exe
3.  用管理员权限打开cmd 终端，进入E:DevEnvgitea路径下
4.  输入一下指令（**注：记得替换成自己的X:XXXXXXgitea**）：

```
sc create gitea start= auto binPath= ""E:\DevEnv\gitea\gitea.exe" web --config"E:\DevEnv\gitea\custom\conf\app.ini"" 
```

![1571387470242](https://ethanarea-1253140193.cos.ap-shanghai.myqcloud.com/blog_images/Gitea/1571387470242.png)

1.  进入任务管理器，选择服务选项卡，找到gitea,右键选择开始，启动该服务
2.  此时可以通过访问 [http://localhost:3000](https://links.jianshu.com/go?to=http%3A%2F%2Flocalhost%3A3000) 进行访问，并配置数据库和管理员
3.  由于我们只是小组内部使用的平台，数据选的SQLite3.

## 三、配置与使用

### 1\. SSH访问设置

1.  修改’E:DevEnvgiteacustomconfapp.ini‘，在server版本修改以下两项：
    
    -   修改localhost 为相应的IP地址，否则页面上的SSH链接会包含localhost，导致复制后无法直接使用（内网访问建议修改）
    -   在最后加一条：
        
        ```
        START_SSH_SERVER = true
        ```
        
2.  保存之后，重启gitea服务。

![tu1](https://ethanarea-1253140193.cos.ap-shanghai.myqcloud.com/blog_images/Gitea/tu1.png)

### 2\. Git的配置

如果git本地没有生成公钥，请继续阅读本部分，已生成请看下一章节即可。

打开Git Bash，输入指令

```
 ssh-keygen -t rsa -C "xx@xxx.com"
 # "xx@xxx.com"为你的邮箱
```

狂按回车即可生成ssh私钥（id\_rsa）和公钥（id\_rsa.pub）。

2.git bash进入.ssh目录，一般为C:/User/你的电脑用户名/.ssh

输入命令:

```
cd C:/User/你的电脑用户名/.ssh
cat id_rsa.pub
```

即可显示的公钥

### 3\. Git的使用

#### 3.1 服务端配置

将本地的产生的公钥，添加至仓库端，如果需要写入权限请记得勾选，并提交至gitea平台。如果不是管理员，可以将公钥复制给管理员进行添加。

![1571392171704](https://ethanarea-1253140193.cos.ap-shanghai.myqcloud.com/blog_images/Gitea/1571392171704.png)

#### 3.2 本地端配置

（持续更新）

![1571392795416](https://ethanarea-1253140193.cos.ap-shanghai.myqcloud.com/blog_images/Gitea/1571392795416.png "1571392795416")