如果你是一个程序员，那么一定听说过 Git 或者已经在使用它。如果你还没有尝试过，推荐你去尝试一下，这是一个很不错的版本控制工具。

平时使用 Git，如果想把代码备份到云端或者跟别人合作开发，首先想到的可能是 Github，但是有时 Github 不是很方便，比如 Github 在国内的访问速度比较慢，虽然 Github 的私有存储库被微软爸爸收购以后变成了无数量限制，但是免费版每个私有存储库只允许最多 3 位合作者，如果是跟公司团队合作开发的话，3 个明显不够用。

面对这种情况，很多人都会选择[[自建 Git 服务]]器。搭建 Git 服务器很简单，用 Linux + Git 就可以做到，但是想要一些附加功能，比如：用户和存储库管理、Issues 提交，版本发布等，就需要依赖第三方服务或软件。

第三方软件还是有不少的，比如：Gitlab CE / EE、Gogs 和 Gitea 等。Gitlab 的功能是最丰富的，但是硬件需求实在是太高，没有 4G 内存根本跑不动。Gogs 是由国人开发的软件，而 Gitea 是由 lunny (国人) 和其他开源爱好者维护的 Gogs 的分支，两个都是开源且免费的，Gitea 在功能方面已经强于 Gogs，所以我最后选择了使用 Gitea 来搭建。

Gitea 在功能方面相比于 Gitlab CE 有一定的缺失，不过也有一些 Gitlab CE 没有的功能，当然不能跟 Gitlab EE 来比较，如果你想知道具体的功能差异，可以前往 [https://docs.gitea.io/en-us/comparison/](https://docs.gitea.io/en-us/comparison/) 查看，最重要的是 Gitea 的硬件需求很亲民，一台树莓派都可以用跑得动，而且对 Windows 也有着很好的支持。

如果你想尝试一下 Gitea，可以访问这个网址：[https://try.gitea.io/](https://try.gitea.io/)

扯了这么多，下面就教大家如何在 Linux 上使用 Gitea 搭建 Git 服务器。

![Gitea](https://cdn.mivm.cn/www.mivm.cn/archives/gitea/index.png)

## 准备工作

Gitea 使用 Golang 编写，采用独立二进制文件分发，所以系统不需要什么依赖库，但是 Git 还是需要的。

我推荐 Git 尽可能的使用最新版或者 2.3 +，这样对 Git LFS 有较好的支持。

另一个就是数据库，如果你正在运行 Debian 8 / CentOS 6 比较老的系统，运行 Gitea 可能会收到 glibc 版本过低的信息，这是因为 Gitea 集成的 SQLite 支持库导致的，这种情况下你可以选择从源代码编译安装，不过我更推荐大家使用新的 Linux 发行版。

Gitea 支持多种数据库：MySQL、PostgreSQL、SQL Server 和 SQLite。如果只是一两个人使用的话，数据库的选择无所谓，如果使用的人数过多，为了更好的性能，推荐使用除 SQLite3 之外的数据库。

CentOS 7 包管理器的 Git 版本很老了，这里给想升级新版本的小伙伴一种简便升级的方法，不需要从源码编译，执行以下命令即可。

`sudo yum install http://opensource.wandisco.com/centos/7/git/x86_64/wandisco-git-release-7-2.noarch.rpm`

`sudo yum clean all`

`sudo yum makecache`

`sudo yum install git`

检查 Git 版本：`git --version`

截止 2019年9月8日，这个软件源的 Git 版本是 2.22.0

## 安装

前往 Github 下载适用于自身系统和平台的二进制文件：[https://github.com/go-gitea/gitea/releases/latest](https://github.com/go-gitea/gitea/releases/latest)

下载完成后将文件重命名为`gitea`并移动到`/usr/local/bin/`，并添加执行权限`sudo chmod +x /usr/local/bin/gitea`

添加 gitea 用户用于执行程序：`sudo useradd -d /var/lib/gitea -m -r -s /bin/bash gitea`

创建所需目录：`sudo -u gitea mkdir -p /var/lib/gitea/{custom,data,log}`

设置目录权限：`sudo chmod -R 750 /var/lib/gitea/`

**如果你使用的系统（CentOS 6 / Debian 7）不支持 systemd，建议你升级到支持 systemd 的系统。**

下载 service 文件以使用 systemd 管理 Gitea：`sudo curl -L -o /etc/systemd/system/gitea.service https://cdn.mivm.cn/www.mivm.cn/archives/gitea/gitea.service`

**根据需要编辑 service 文件，如果要使用 SQLite3 之外的数据库，请去除 gitea.service 的对应注释。**

比如使用 MariaDB 作为数据库，编辑`gitea.service`，将`#Requires=mariadb.service`行的 # 删除即可。

如果需要让 Gitea 监听 1024 以下的端口，将`#CapabilityBoundingSet=CAP_NET_BIND_SERVICE`和`#AmbientCapabilities=CAP_NET_BIND_SERVICE`行前的 # 删除。

[小山](https://github.com/Hill-98)并不推荐直接让 Gitea 监听 80 之类的端口，而是让 Web 服务器反代 Gitea。

修改完成后，执行`sudo systemctl daemon-reload`重新加载 service 文件。

设置 Gitea 开机启动：`sudo systemctl enable gitea`

运行 Gitea：`sudo systemctl start gitea`

## 配置

在浏览器里访问 Gitea，如果你的系统没有开放 Gitea 的端口，建议你先看完下面的 Nginx 反向代理配置。

浏览器访问 Gitea 默认看到的是 Gitea 的首页，点击右上角的【注册】即可进入安装向导。

**除了数据库必须配置以外，还有几个选项需要注意一下，特别是配置了反向代理。**

SSH 服务域名：这个是存储库显示以 SSH 方式克隆时显示的域名，比如这个域名如果配置为 git.example .com，那么在存储库哪里的 SSH 方式克隆就会显示为`gitea@git.example.com:test/test.git`，域名的 IP 务必指向你的 Gitea 服务器。

SSH 服务端口：这个端口务必设置为你服务器的 SSH 端口，也就是你平时 SSH 登录服务器使用的端口，**如果你的服务器没有开启 SSH 服务，将它设置为 0 以禁用**。

Gitea 基本 URL：这个选项跟上面的 \[SSH 服务域名\] 同理，存储库显示以 HTTP(S) 方式克隆的时候显示的域名。

下面的 \[可选设置\] 部分根据自己的需要设置。

最后点击 【立即安装】 即可。

现在你的 Gitea 就安装好了，你可以创建存储库，并测试是否可以正常 clone 和 push 等。

如果你想修改 Gitea 的配置，可以参考官网的[文档](https://docs.gitea.io/en-us/config-cheat-sheet/)修改`/var/lib/gitea/custom/conf/app.ini`。

推荐在修改完成后，使用 chattr 命令来防止配置文件被修改，阻止修改`sudo chattr +i /var/lib/gitea/custom/conf/app.ini`，允许修改`sudo chattr -i /var/lib/gitea/custom/conf/app.ini`

[![Gitea](https://cdn.mivm.cn/www.mivm.cn/archives/gitea/01.jpg)](https://cdn.mivm.cn/www.mivm.cn/archives/gitea/01.jpg)

## Nginx 反向代理

大部分人的服务器肯定不止运行一个 Git 服务器，肯定还有别的 Web 服务，为了更好的使用 Gitea，最好让 Nginx 等前端 Web 服务器反向代理 Gitea。

以下是一个 Nginx 示例配置：

```
location / {
    client_max_body_size 0;
    proxy_pass http://localhost:3000;
}

```

___

以上就是如何使用 Gitea 搭建 Git 服务器的步骤，如果有什么不懂的地方可以加入 [QQ 群](https://jq.qq.com/?_wv=1027&k=46T5isq)与我探讨。

![微信公众号二维码](https://cdn.mivm.cn/www.mivm.cn/images/WeChat_Subscription.jpg)

微信扫描二维码关注我们

**如果觉得文章有帮助到你，可以点击下方的打赏按钮赞助下服务器费用。**