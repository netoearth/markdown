## 1\. Gitlab概述

### 1.1 GitLab介绍

GitLab是利用Ruby on Rails一个开源的版本管理系统，实现一个自托管的Git项目仓库，可通过Web界面进行访问公开的或者私人项目。

GitLab能够浏览源代码，管理缺陷和注释。可以管理团队对仓库的访问，它非常易于浏览提交过的版本并提供一个文件历史库。团队成员可以利用内置的简单聊天程序(Wall)进行交流。

它还提供一个代码片段收集功能可以轻松实现代码复用，便于日后有需要的时候进行查找

### 1.2 Gitlab服务构成

Nginx：静态web服务器。

gitlab-shell：用于处理Git命令和修改authorized keys列表。

gitlab-workhorse: 轻量级的反向代理服务器。

logrotate：日志文件管理工具。

postgresql：数据库。

redis：缓存数据库。

sidekiq：用于在后台执行队列任务（异步执行）。

unicorn：An HTTP server for Rack applications，GitLab Rails应用是托管在这个服务器上面的。

### 1.3 Gitlab工作流程

![](https://img2020.cnblogs.com/blog/1278240/202012/1278240-20201205202848509-1838984252.png)

### 1.4 GitLab Shell

GitLab Shell有两个作用：为GitLab处理Git命令、修改authorized keys列表

当通过SSH访问GitLab Server时，GitLab Shell会：

-   限制执行预定义好的Git命令（git push，git pull，git annex）
-   调用GitLab Rails API检查权限
-   执行pre-receive钩子（在企业版中叫做Git钩子）
-   执行用户请求的动作，处理GitLab的post-receive动作
-   处理自定义的post-receive动作

当通过http(s)访问GitLab Server时，工作流程取决于你是从Git仓库拉取(pull)代码还是向git仓库推送(push)代码：

如果是从Git仓库拉取(pull)代码，GitLab Rails应用会全权负责处理用户鉴权和执行Git命令的工作

如果是向Git仓库推送(push)代码，GitLab Rails应用既不会进行用户鉴权也不会执行Git命令，它会把以下工作交由GitLab Shell进行处理：

-   调用GitLab Rails API 检查权限
-   执行pre-receive钩子（在GitLab企业版中叫做Git钩子）
-   执行你请求的动作
-   处理GitLab的post-receive动作
-   处理自定义的post-receive动作

### 1.5 GitLab Workhorse 

GitLab Workhorse是一个敏捷的反向代理。它会处理一些大的HTTP请求，比如文件上传、文件下载、Git push/pull和Git包下载。其它请求会反向代理到GitLab Rails应用，即反向代理给后端的unicorn。

## 2\. Gitlab的安装部署

-   Gitlab要求服务器内存2G以上

### 2.1 方式一:下载gitlab-ce的rpm包

-   [gitlab官方rpm包下载](https://packages.gitlab.com/gitlab/gitlab-ce)
-   [清华的源](https://mirrors.tuna.tsinghua.edu.cn/gitlab-ce/yum/el7/)

将对应版本的gitlab-ce下载到本地后，直接yum安装即可

copy

```
# 要先将这个rpm包下载到本地
yum install -y gitlab-ce-13.6.1-ce.0.el7.x86_64.rpm
```

### 2.2 方式二:配置yum源

在 /etc/yum.repos.d/ 下新建 gitlab-ce.repo，写入如下内容：

![复制代码](https://common.cnblogs.com/images/copycode.gif)

copy

```
[gitlab-ce]
name=gitlab-ce
baseurl=https://mirrors.tuna.tsinghua.edu.cn/gitlab-ce/yum/el7/
Repo_gpgcheck=0
Enabled=1
Gpgkey=https://packages.gitlab.com/gpg.key
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

然后创建cache，再直接安装gitlab-ce

![复制代码](https://common.cnblogs.com/images/copycode.gif)

copy

```
yum makecache  # 这一步会创建大量的数据

# 直接安装最新版
yum install -y gitlab-ce 

# 如果要安装指定的版本，在后面填上版本号即可
yum install -y  gitlab-ce-13.6.1

# 如果安装时出现gpgkey验证错误，只需在安装时明确指明不进行gpgkey验证
yum install gitlab-ce -y --nogpgcheck
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

### 2.3 gitlab的配置

配置文件位置  /etc/gitlab/gitlab.rb

![复制代码](https://common.cnblogs.com/images/copycode.gif)

copy

```
[root@centos7 test]# vim /etc/gitlab/gitlab.rb

[root@centos7 test]# grep "^[a-Z]" /etc/gitlab/gitlab.rb

external_url 'http://10.0.0.51'  # 这里一定要加上http://

# 配置邮件服务
gitlab_rails['smtp_enable'] = true
gitlab_rails['smtp_address'] = "smtp.qq.com"
gitlab_rails['smtp_port'] = 25
gitlab_rails['smtp_user_name'] = "hgzerowzh@qq.com"  # 自己的qq邮箱账号
gitlab_rails['smtp_password'] = "xxx"  # 开通smtp时返回的授权码
gitlab_rails['smtp_domain'] = "qq.com"
gitlab_rails['smtp_authentication'] = "login"   
gitlab_rails['smtp_enable_starttls_auto'] = true
gitlab_rails['smtp_tls'] = false
gitlab_rails['gitlab_email_from'] = "hgzerowzh@qq.com"  # 指定发送邮件的邮箱地址
user["git_user_email"] = "shit@qq.com"   # 指定接收邮件的邮箱地址
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

**修改好配置文件后，要使用 gitlab-ctl reconfigure 命令重载一下配置文件，否则不生效。**

copy

```
gitlab-ctl reconfigure # 重载配置文件
```

### 2.4 Gitlab常用命令

![复制代码](https://common.cnblogs.com/images/copycode.gif)

copy

```
gitlab-ctl start         # 启动所有 gitlab 组件
gitlab-ctl stop          # 停止所有 gitlab 组件
gitlab-ctl restart       # 重启所有 gitlab 组件
gitlab-ctl status        # 查看服务状态

gitlab-ctl reconfigure   # 启动服务
gitlab-ctl show-config   # 验证配置文件

gitlab-ctl tail          # 查看日志

gitlab-rake gitlab:check SANITIZE=true --trace    # 检查gitlab

 vim /etc/gitlab/gitlab.rb # 修改默认的配置文件
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

## 3\. Gitlab的使用

-   Gitlab安装好后，设置密码，管理账户为root

### **3.1 创建Group**

-   填上组名即可，这里组名为java

![](https://img2020.cnblogs.com/blog/1278240/202012/1278240-20201207220740204-310622731.png)

![](https://img2020.cnblogs.com/blog/1278240/202012/1278240-20201207221101705-120553704.png)

### **3.2 创建User**

-   创建四个User：pm、dev1、dev2、dev3

![](https://img2020.cnblogs.com/blog/1278240/202012/1278240-20201207221212377-1797394268.png)

 ![](https://img2020.cnblogs.com/blog/1278240/202012/1278240-20201207221312324-784634515.png)

### **3.3 添加User到Group中并授权**

![](https://img2020.cnblogs.com/blog/1278240/202012/1278240-20201207221900054-1772460677.png)

### **3.4 创建Project并配置SSH**

![](https://img2020.cnblogs.com/blog/1278240/202012/1278240-20201207222010732-1602502839.png)

 ![](https://img2020.cnblogs.com/blog/1278240/202012/1278240-20201207222201775-2072305192.png)

![](https://img2020.cnblogs.com/blog/1278240/202012/1278240-20201207222429638-1919267323.png)

![](https://img2020.cnblogs.com/blog/1278240/202012/1278240-20201207222540571-554043328.png)

### **3.5 在项目中添加成员**

![](https://img2020.cnblogs.com/blog/1278240/202012/1278240-20201207222837961-1011491961.png)

![](https://img2020.cnblogs.com/blog/1278240/202012/1278240-20201207223023029-1523481399.png)

### 3.6 将本地文件推送到Gitlab

![复制代码](https://common.cnblogs.com/images/copycode.gif)

copy

```
# 将app01项目克隆下来
git clone git@10.0.0.51:java/app01.git

# 初始化配置
git config --global user.name "hgzero"
git config --global user.email "hgzero@qq.com"

# 在app01目录下新建一些文件

# 推送到gitlab
git add .
git commit -m "first edition"
git push origin master
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

![](https://img2020.cnblogs.com/blog/1278240/202012/1278240-20201207224200805-1615948610.png)

## 4\. 制定开发计划

### **4.1 创建开发计划**

-   项目：app01
-   版本：v1.0

![](https://img2020.cnblogs.com/blog/1278240/202012/1278240-20201208220630422-901707409.png)

### **4.2 创建里程碑Milestones**

-   用pm账号登录gitlab后操作（先要在admin中设置pm账号的密码）
-   要根据开发计划来创建Milestones

![](https://img2020.cnblogs.com/blog/1278240/202012/1278240-20201208190209635-1190789761.png)

![](https://img2020.cnblogs.com/blog/1278240/202012/1278240-20201208190238204-1073215692.png)

 ![](https://img2020.cnblogs.com/blog/1278240/202012/1278240-20201208190433835-2097506142.png)

![](https://img2020.cnblogs.com/blog/1278240/202012/1278240-20201208191155236-973927486.png)

###  4.3 根据开发计划创建issue

-   创建4个issue，分派给dev1和dev2这两个开发人员

 ![](https://img2020.cnblogs.com/blog/1278240/202012/1278240-20201208191330962-1807380887.png)

![](https://img2020.cnblogs.com/blog/1278240/202012/1278240-20201208191703312-1665224640.png)

![](https://img2020.cnblogs.com/blog/1278240/202012/1278240-20201208191904226-1716909827.png)

![](https://img2020.cnblogs.com/blog/1278240/202012/1278240-20201208193044770-2124330934.png)

![](https://img2020.cnblogs.com/blog/1278240/202012/1278240-20201208192323913-1809326796.png)

![](https://img2020.cnblogs.com/blog/1278240/202012/1278240-20201208193325808-1501559347.png)

### **4.4 开发者登录账号查看分派的任务** 

-   然后开发dev1登录gitlab，就能看到任务已经分配过来了

![](https://img2020.cnblogs.com/blog/1278240/202012/1278240-20201208193706705-598380841.png)

### **4.5 开发流程**

-   公司里的开发开始任务

![复制代码](https://common.cnblogs.com/images/copycode.gif)

copy

```
# 1. 先从仓库把项目拉下来
git clone git@10.0.0.51:java/app01.git
cd app01/

# 2.先创建一个自己的分支，然后进行开发
git checkout -b index   # 创建一个叫index的分支，并切换到这个分支
git status

# 3. 开始开发首页
echo "<h1>welcome to this app</h1>" > index.html  # 假设就开发了一个index页面

# 4. 开发完成后，把项目传到仓库
git add .
git commit -m "index"
# 如果写成 git commit -m "close #2" ，则表示merge请求允许且merge成功之后，自动删除编号为#2的issue

# 传到index分支
git push origin index 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

### **4.6 合并分支**

**1）开发dev1发送合并分支请求给pm**

![](https://img2020.cnblogs.com/blog/1278240/202012/1278240-20201208205656272-1853593312.png)

![](https://img2020.cnblogs.com/blog/1278240/202012/1278240-20201208205854771-78130642.png)

![](https://img2020.cnblogs.com/blog/1278240/202012/1278240-20201208210018623-1203282499.png)

**2）pm收到开发的Merge请求后进行处理**

-    使用pm登录，就可以看到pm已经收到了合并请求merge request

![](https://img2020.cnblogs.com/blog/1278240/202012/1278240-20201208210155378-2083747745.png)

![](https://img2020.cnblogs.com/blog/1278240/202012/1278240-20201208210437684-78308682.png)

![](https://img2020.cnblogs.com/blog/1278240/202012/1278240-20201208211131823-1594918617.png)

![](https://img2020.cnblogs.com/blog/1278240/202012/1278240-20201208211239027-1226601369.png)

**3）开发dev1确认任务完成**

-   退出pm账户，登入dev1账户：

![](https://img2020.cnblogs.com/blog/1278240/202012/1278240-20201208211509075-1076355560.png)

-    或者点进去后，在侧边栏进行标识Done，然后已经完成的issue，可以将其Close

![](https://img2020.cnblogs.com/blog/1278240/202012/1278240-20201208211746173-54963268.png)

-   这个时候Milestones的进度已经往前进了一些了：

![](https://img2020.cnblogs.com/blog/1278240/202012/1278240-20201208211908829-790499125.png)

### 4.7 开发其他功能

-   然后其他开发者或者自己再次进行开发时，先要把刚刚更新后的内容（master主干）拉回来，然后再进行开发

copy

```
git checkout master  # 切换到master
git pull             # 从远端仓库拉取数据# 然后再进行其他操作
```

## 5\. Gitlab备份恢复

### 5.1 备份gitlab

**1）修改配置文件**

-   **/etc/gitlab/gitlab.rb**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

copy

```
# 备份保存的位置，这里是默认位置，可修改成指定的位置
gitlab_rails['backup_path'] = "/var/opt/gitlab/backups"

# 设置备份保存的时间，超过此时间的日志将会被新覆盖
gitlab_rails['backup_keep_time'] = 604800  # 这里是默认设置，保存7天

# 特别注意：
#     如果自定义了备份保存位置，则要修改备份目录的权限，比如：
#     chown -R git.git /data/backup/gitlab
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

-   配置完成后要重启以使配置生效

![复制代码](https://common.cnblogs.com/images/copycode.gif)

copy

```
# 重读配置文件
gitlab-ctl reconfigure  

# 重启gitlab
gitlab-ctl restart
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

**2）设置定时任务**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

copy

```
# 每天凌晨2点定时创建备份
# 将一下内容写入到定时任务中 crontab -e
0 2 * * * /usr/bin/gitlab-rake gitlab:backup:create

# 备份策略建议：
#     本地保留3到7天，在异地备份永久保存
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

**3）备份时间的识别**

copy

```
# 备份后的文件类似这样的形式：1494170842_gitlab_backup.tar，可以根据前面的时间戳确认备份生成的时间

data  -d  @1494170842
```

### 5.2 恢复gitlab

**1）停止数据写入服务**

copy

```
# 停止数据写入服务
gitlab-ctl stop unicorn
gitlab-ctl stop sidekiq
```

**2）进行数据恢复并重启**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

copy

```
# 进行恢复
gitlab-rake gitlab:backup:restore BACKUP=1494170842  # 这个时间戳就是刚刚备份的文件前面的时间戳

# 重启
gitlab-ctl restart
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

## 6\. gitlab邮件通知配置

-   vim  /etc/gitlab/gitlab.rb

![复制代码](https://common.cnblogs.com/images/copycode.gif)

copy

```
gitlab_rails['time_zone'] = 'Asia/Shanghai'

gitlab_rails['gitlab_email_enabled'] = true
gitlab_rails['gitlab_email_from'] = 'example@163.com' # 填写发件人的邮箱地址
gitlab_rails['gitlab_email_display_name'] = 'gitlab'

gitlab_rails['smtp_enable'] = true
gitlab_rails['smtp_address'] = "smtp.163.com"  # smtp服务器的地址,如网易的地址
gitlab_rails['smtp_port'] = 25                 # 要注意如果使用了SSL/TLS的话,端口可能不是25
gitlab_rails['smtp_user_name'] = "smtp用户名"
gitlab_rails['smtp_password'] = "smtp用户密码"
gitlab_rails['smtp_domain'] = "163.com"
gitlab_rails['smtp_authentication'] = "login"
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

## 7. 使用SourceTree进行项目开发

### 7.1 项目拉取

-   先把项目克隆下来

![](https://img2020.cnblogs.com/blog/1278240/202012/1278240-20201209091012859-1719164035.png)

 ![](https://img2020.cnblogs.com/blog/1278240/202012/1278240-20201209091130338-1210401710.png)

-    如果ssh的方式克隆失败，可能是因为SSH Key没找到，可以在这里添加

![](https://img2020.cnblogs.com/blog/1278240/202012/1278240-20201209091357473-1313656566.png)

![](https://img2020.cnblogs.com/blog/1278240/202012/1278240-20201209091314794-1268697141.png)

### 7.2 创建分支进行功能开发

**1）新建立一个叫“pay”的分支**

![](https://img2020.cnblogs.com/blog/1278240/202012/1278240-20201209091543272-2078639172.png)

 ![](https://img2020.cnblogs.com/blog/1278240/202012/1278240-20201209091641355-1433800575.png)

**2）进行功能开发**

![](https://img2020.cnblogs.com/blog/1278240/202012/1278240-20201209091958461-216662907.png)

### 7.3 提交项目  

**1）开发pay功能完成后进行提交**

-   可以看到SourceTree中已经有“未提交的更改”

![](https://img2020.cnblogs.com/blog/1278240/202012/1278240-20201209092255166-518657240.png)

![](https://img2020.cnblogs.com/blog/1278240/202012/1278240-20201209092514163-932531269.png)

**2）添加“用户信息”**

![](https://img2020.cnblogs.com/blog/1278240/202012/1278240-20201209092935266-1648599730.png)

 **3）进行提交**

![](https://img2020.cnblogs.com/blog/1278240/202012/1278240-20201209092730322-11026778.png)

-   注释也可以写成  close #3    ，作用是提交完成后关闭3号issue 

### **7.4 推送到仓库**

![](https://img2020.cnblogs.com/blog/1278240/202012/1278240-20201209093416282-715006719.png)

 ![](https://img2020.cnblogs.com/blog/1278240/202012/1278240-20201209093458097-94709496.png)

-    然后就可以在gitlab上进行发送merge请求了，后面就可以进行其他操作了

### 7.5 项目上线

**1）当所有工作完成之后，就可以进行上线了**

![](https://img2020.cnblogs.com/blog/1278240/202012/1278240-20201209094359502-1671181213.png)

**2）打标签**

-    上线先打个标签

![](https://img2020.cnblogs.com/blog/1278240/202012/1278240-20201209094528156-1512355107.png)

![](https://img2020.cnblogs.com/blog/1278240/202012/1278240-20201209094642241-360548164.png)

![](https://img2020.cnblogs.com/blog/1278240/202012/1278240-20201209094721052-1835892074.png)

 **3）删除无用分支**

-   然后删除已经合并到主干中的不必要的分支，如index、pay等
-   最后一定要注意时间一定要同步，不然会错乱

本文作者：Praywu

本文链接：https://www.cnblogs.com/hgzero/p/14088215.html

版权声明：本作品采用知识共享署名-非商业性使用-禁止演绎 2.5 中国大陆许可协议进行许可。