此文比较入门详细

官网的介绍是：

> Gitea的首要目标是创建一个极易安装，运行非常快速，安装和使用体验良好的自建 Git 服务。我们采用Go作为后端语言，这使我们只要生成一个可执行程序即可。并且他还支持跨平台，支持 Linux, macOS 和 Windows 以及各种架构，除了x86，amd64，还包括 ARM 和 PowerPC

换句话说就是一个git管理工具，类似构建自己的github，但是github仓库需要公开（私有付费）,国内的gitee免费的也有一定的限制。我们可以通过gitea管理自己的项目代码。

官方文档地址（中文）：[文档 - Docs (gitea.io)](https://link.zhihu.com/?target=https%3A//docs.gitea.io/zh-cn/)

## **我为什么选择gitea**

1.  功能基本满足我的需求
2.  需要的机器性能不高，官方建议是2 核 CPU 及 1GB 内存，我自己跑1核1GB有点小慢（故而以下教程是买了一个2GB去跑的）

## **二进制安装**

我这边选择二进制安装，应该也是最简单的方式了。下面都是在centos系统操作的。

**注意⚠️：需要有MySQL、PostgreSQL、MSSQL 或 SQLite3其中一个用于gitea存储数据**

## **1、安装git**

## **2、创建一个目录用于存放gitea和git数据的目录**

我自己是放在/git里面，大家看着改。

## **3、添加Git用户**

用来运行gitea，后面创建的service也是通过这个用户运行，还有密钥那些东西不会和root之类的用户搞混。

```
groupadd git

useradd git -g git 
```

## **4、下载**

打开页面：[gitea | Gitea](https://link.zhihu.com/?target=https%3A//dl.gitea.io/gitea)，可以看到全部版本，最上面的就是最新的，选择你要的版本。

![](https://pic4.zhimg.com/v2-9422633664c656fdf82856f0eee20df3_b.jpg)

点击进入你要的版本找到后缀是linux-amd64的文件复制下链接，我下载的是[https://dl.gitea.io/gitea/1.13.1/gitea-1.13.1-linux-amd64](https://link.zhihu.com/?target=https%3A//dl.gitea.io/gitea/1.13.1/gitea-1.13.1-linux-amd64)，你们也可以把链接里面的1.13.1换成自己需要的版本。

在服务器上

```
# 去到你要安装的目录
cd /git

# 下载
wget -O gitea https://dl.gitea.io/gitea/1.13.1/gitea-1.13.1-linux-amd64

# 设置成可运行文件
chmod +x gitea

# 测试运行
./gitea web
```

运行成功的话，别退出测试下`[ip]:3000`是否可以访问，云服务器的话去设置下安全策略开放3000端口。

或者本身服务器开了防火墙的

```
# 查看是不是没有开放3000端口
firewall-cmd --zone=public --list-ports
# 如果没有就添加一个
firewall-cmd --zone=public --add-port=3000/tcp --permanent
# 重新载入配置文件
firewall-cmd --reload
```

没问题就直接退出gitea程序，继续下面的操作。

## **5、更换目录的用户**

## **6、配置service**

然后就可以很爽的用：systemctl控制了

```
vi /etc/systemd/system/gitea.service
```

官方有提供了一份services配置文件：[gitea/gitea.service at master · go-gitea/gitea (github.com)](https://link.zhihu.com/?target=https%3A//github.com/go-gitea/gitea/blob/master/contrib/systemd/gitea.service)

注意：

-   WorkingDirectory配置项是工作路径得是存在且是git有权限的目录，如果不想太麻烦，可以删除...
-   ExecStart是启动命令的意思，`/usr/local/bin/gitea web --config /etc/gitea/app.ini`中`/usr/local/bin/gitea`改成你自己的gitea文件的目录，我这里是`/git/gitea`，-- config带的参数是gitea的配置文件，如果你刚刚按照我的做法去做，在gitea的同级目录下存在`./custom/conf/app.ini`，把这个换到后面就行了，最后就是：`ExecStart=/git/gitea web --config /git/custom/conf/app.ini`。

**懒人版请直接复制下面的**（从头到尾和我一样的目录的）

```
[Unit]
Description=Gitea
After=syslog.target
After=network.target

[Service]
RestartSec=2s
Type=simple
User=git
Group=git
ExecStart=/git/gitea web --config /git/custom/conf/app.ini
Restart=always

[Install]
WantedBy=multi-user.target
```

启动服务和设置开机启动

```
# 运行
systemctl start gitea
# 查看是否成功运行
ps -aux | grep gitea
# 如果成功会看到一条git用户运行的gitea进程
git       1525  9.8 12.1 1375512 227352 ?      Ssl  17:17   0:00 /git/gitea web --config /git/custom/conf/app.ini
root      1525  0.0  0.0  12324  1040 pts/0    S+   17:17   0:00 grep --color=auto gitea
# 开机启动
systemctl enable gitea
```

然后在自己电脑浏览器打开\[ip\]:3000，点击登陆初始化设置，我这边给它设置了域名和端口，所以访问变成\[域名\]:\[端口号\]，完成。

![](https://pic3.zhimg.com/v2-1bb5be1fe69dde2c2b20cb57124d3eaa_b.jpg)

## **7、nginx代理**

因为我们80端口都是给nginx用了，3000端口不想报漏在外面（主要是看着域名后面跟着端口号好难受），我们给他做一下nginx代理 找到nginx的配置文件目录，添加一个网站配置文件,如果要ssl的自己添加就行了，改完修改下`app.ini`的`ROOT_URL`重启一下gitea，nginx重载下配置文件

```
server {
    listen       80;
    server_name  [域名]
}
location / {
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_pass    http://127.0.0.1:3000;
}
location ~ .*\.(js|css|png)$ {
    proxy_pass  http://127.0.0.1:3000;
}
```

## **8、配置SSH密钥**

> 作用：这些 SSH 公钥已经关联到你的账号。相应的私钥拥有完全操作你的仓库的权限。

github写的ssh密钥操作：

首先[检查现有 SSH 密钥 - GitHub Docs](https://link.zhihu.com/?target=https%3A//docs.github.com/cn/github/authenticating-to-github/checking-for-existing-ssh-keys)

如果没有ssh密钥[生成新 SSH 密钥并添加到 ssh-agent - GitHub Docs](https://link.zhihu.com/?target=https%3A//docs.github.com/cn/github/authenticating-to-github/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent)

然后：登陆->点击右上角的头像->设置->SSH/GPG密钥->增加密钥（就是刚刚生成文件中的`.pub`文件）

可以在你自己电脑输入命令可以查看

```
# 如果你刚刚 使用ssh-keygen生成密钥的时候-t参数有填写 那么下面命令的rsa换成你自己的参数
cat ~/.ssh/id_rsa.pub
```

然后在自己电脑激活一下完成

```
> ssh git@[你gitea域名或者ip]
# 显示下面文字就成功了
PTY allocation request failed on channel 0
Hi there, 你的gitea名字! You've successfully authenticated with the key named mac, but Gitea does not provide shell access.
If this is unexpected, please log in with password and setup Gitea under another user.
Connection to [你gitea域名或者ip] closed.
```