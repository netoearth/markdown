前文 使用Let’s Encrypt获取免费证书 介绍了使用 certbot 工具从Let’s Encrypt获取免费证书。但certbot需要自行设置定时任务更新证书、依赖于新版 Python（Debian 9等系统的Python是即将放弃支持的Python 3.5）、以及不少DNS验证插件需要自行安装。

鉴于上述缺点，考虑换成自动化程度更高、使用起来更简易的另一个ACME协议客户端：`acme.sh`。本教程简要介绍使用`acme.sh`从Let’s Encrypt获取免费证书。

## 使用acme.sh签发证书

### 安装acme.sh

`acme.sh` 官网是 [https://github.com/acmesh-official/acme.sh](https://github.com/acmesh-official/acme.sh)，里面已经有比较详细的中文使用说明。

首先是安装`acme.sh`：

```
curl https://get.acme.sh | sh
```

如果提示“curl: command not found”，CentOS/RHEL/Aliyun OS等衍生系统使用：`yum install -y curl`，Debian/Ubuntu及衍生系统使用：`apt install -y curl` 安装`curl`，然后再次运行上面的acme.sh安装命令。

`acme.sh` 依赖于`cron`执行定时任务，安装完成后，输入 `crontab -l` 命令，能看到如下输出：

```
41 0 * * * "/root/.acme.sh"/acme.sh --cron --home "/root/.acme.sh" > /dev/null
```

定时任务保证了证书在到期前能自动续期。由于ACME协议和Let’s Encrypt CA都在频繁的更新，因此建议开启acme.sh的自动升级：

```
～/.acme.sh/acme.sh  --upgrade  --auto-upgrade
```

### 签发证书

使用`acme.sh`签发证书非常简单：

1\. 如果你的服务器上已经运行了web软件，指定webroot即可签发证书：

```
～/.acme.sh/acme.sh --issue -d 域名 --webroot web目录
```

如果要为多个域名签发，直接加 `-d 域名` 就可以。

> 需要注意的是，非DNS验证方式签发证书的前提都是域名的IP需要指向当前服务器，并且防火墙需要

2\. 如果网站已经运行Nginx/Apache，指定对应插件就可以了：

```
～/.acme.sh/acme.sh --issue -d 域名 --nginx # 如果是apache，换成 --apache
```

3\. 如果没有运行web软件并且80端口空闲，可以使用acme.sh自己监听80端口进行验证：

```
～/.acme.sh/acme.sh --issue -d 域名 --standalone
```

4\. 也可以使用DNS方式，手动添加DNS记录进行验证：

```
～/.acme.sh/acme.sh --issue --dns -d 域名
# 命令结束后，acme.sh会显示解析记录，需要到DNS后台设置解析
# 设置好解析后，生成证书
～/.acme.sh/acme.sh --renew -d 域名
```

对于DNS方式，acme.sh内置了许多DNS服务商的插件，使用方法参考：[https://github.com/Neilpang/acme.sh/blob/master/dnsapi/README.md](https://github.com/Neilpang/acme.sh/blob/master/dnsapi/README.md)

> 如果有通配符证书要求，需要使用DNS验证方式，其他方式一个证书的域名数不能超过20

申请好证书的证书位于`~/.acme.sh`目录内，不建议直接使用，而是将其安装到指定目录：

```
～/.acme.sh/acme.sh --install-cert -d 域名 \
--key-file       密钥存放目录  \
--fullchain-file 证书存放路径 \
--reloadcmd     "service nginx force-reload"
```

其中`--reloadcmd`是可选的，用来安装好证书后重启web服务器。

接下来配置好网站的SSL配置，就可以愉快的用HTTPS方式访问你的站点了。

## 其他

上面提到的申请和安装命令，执行过一次后，`acme.sh`便会记下你的操作，在证书即将到期前自动帮你执行一遍，非常的好用和贴心。

## 文章导航