   

[![](https://p6-passport.byteacctimg.com/img/user-avatar/a5bdf441394bfed6d3926d2484d27e50~300x300.image)](https://juejin.cn/user/3474112472955837)

2022年08月26日 10:09 ·  阅读 373

> 转自[i.congm.in/ssl/](https://link.juejin.cn/?target=https%3A%2F%2Fi.congm.in%2Fssl%2F "https://i.congm.in/ssl/")

## Let's Encrypt介绍

Let's Encrypt: [letsencrypt.org](https://link.juejin.cn/?target=https%3A%2F%2Fletsencrypt.org "https://letsencrypt.org") , 是一个免费的、自动化的、开放的证书颁发机构。截至2018年9月，它的全球SSL证书市场份额已超过50%，得到主流浏览器和厂商的认可与支持。

Let's Encrypt证书提供免费的申请，但没有高额的安全保险，不具备点对点的技术支持，而且申请过程比较复杂，适合具有一定的专业知识的个人站长申请。并且每次申请到的SSL证书有效期只有90天，但是可以通过脚本实现提前自动续约达到自动化永久免费使用的目的。

在2018年5月，Let's Encrypt发布了免费泛域名通配符SSL证书：[community.letsencrypt.org/t/acme-v2-a…](https://link.juejin.cn/?target=https%3A%2F%2Fcommunity.letsencrypt.org%2Ft%2Facme-v2-and-wildcard-certificate-support-is-live%2F55579 "https://community.letsencrypt.org/t/acme-v2-and-wildcard-certificate-support-is-live/55579") ，在HTTPS全面普及的当下，越来越多的开发者选择了Let's Encrypt证书。

想要实现免费泛域名SSL证书申请及其自动续签，需要以下材料：

一枚域名 一台服务器（用以执行申请与自动续签脚本，且Web服务运行在此服务器上） 本文将以 \*.congm.in 为例进行泛域名证书申请演示

## [#](https://link.juejin.cn/?target=https%3A%2F%2Fwww.xkboke.com%2Fweb-inn%2Fblog%2Fblog-7.html%23%25E4%25B8%2580%25E3%2580%2581%25E8%25AF%2581%25E4%25B9%25A6%25E7%2594%25B3%25E8%25AF%25B7 "https://www.xkboke.com/web-inn/blog/blog-7.html#%E4%B8%80%E3%80%81%E8%AF%81%E4%B9%A6%E7%94%B3%E8%AF%B7")一、证书申请

Let's Encrypt官方提供了一系列的申请方法文档，但流程比较复杂，为了简化申请流程，我们可以在Github上找到这个仓库：[github.com/Neilpang/ac…](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2FNeilpang%2Facme.sh "https://github.com/Neilpang/acme.sh")

> acme.sh 的具体使用说明见：[github.com/Neilpang/ac…](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2FNeilpang%2Facme.sh%2Fwiki%2F%25E8%25AF%25B4%25E6%2598%258E "https://github.com/Neilpang/acme.sh/wiki/%E8%AF%B4%E6%98%8E")

接下来，我们就利用 acme.sh 对证书进行申请。

### [#](https://link.juejin.cn/?target=https%3A%2F%2Fwww.xkboke.com%2Fweb-inn%2Fblog%2Fblog-7.html%23_1%25E3%2580%2581%25E5%25AE%2589%25E8%25A3%2585-acme-sh "https://www.xkboke.com/web-inn/blog/blog-7.html#_1%E3%80%81%E5%AE%89%E8%A3%85-acme-sh")1、安装 acme.sh

进入服务器，执行命令:

```
$ curl  https://get.acme.sh | sh
复制代码
```

> 普通用户和root用户都可以安装使用，安装脚本其实是进行了如下操作:
> 
> 1）会把 `acme.sh` 安装到你所执行命令用户的用户目录下：`~/.acme.sh/`
> 
> 2）会创建 bash 的 alias，方便你的使用：`alias acme.sh=~/.acme.sh/acme.sh`
> 
> 3）会自动为你创建 `cronjob` 脚本，每天零点自动检测所有的证书，如果某证书快过期需要更新，则会自动更新该证书。

安装过程不会污染已有的系统任何功能和文件，所有后续的修改都将限制在安装目录中：`~/.acme.sh/`

### [#](https://link.juejin.cn/?target=https%3A%2F%2Fwww.xkboke.com%2Fweb-inn%2Fblog%2Fblog-7.html%23_2%25E3%2580%2581%25E9%25AA%258C%25E8%25AF%2581%25E5%259F%259F%25E5%2590%258D%25E5%25B9%25B6%25E7%2594%259F%25E6%2588%2590%25E8%25AF%2581%25E4%25B9%25A6 "https://www.xkboke.com/web-inn/blog/blog-7.html#_2%E3%80%81%E9%AA%8C%E8%AF%81%E5%9F%9F%E5%90%8D%E5%B9%B6%E7%94%9F%E6%88%90%E8%AF%81%E4%B9%A6")2、验证域名并生成证书

`acme.sh` 实现了 acme 协议支持的所有验证协议，通常一般有几种方式验证域名：HTTP文件验证 和 DNS验证 等，具体验证方式可查阅[脚本github文档](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2FNeilpang%2Facme.sh%2Fwiki%2FHow-to-issue-a-cert "https://github.com/Neilpang/acme.sh/wiki/How-to-issue-a-cert")。

#### [#](https://link.juejin.cn/?target=https%3A%2F%2Fwww.xkboke.com%2Fweb-inn%2Fblog%2Fblog-7.html%23%25E6%2596%25B9%25E5%25BC%258F1%25E3%2580%2581http%25E6%2596%2587%25E4%25BB%25B6%25E9%25AA%258C%25E8%25AF%2581-%25E4%25B8%258D%25E5%25BB%25BA%25E8%25AE%25AE "https://www.xkboke.com/web-inn/blog/blog-7.html#%E6%96%B9%E5%BC%8F1%E3%80%81http%E6%96%87%E4%BB%B6%E9%AA%8C%E8%AF%81-%E4%B8%8D%E5%BB%BA%E8%AE%AE")方式1、HTTP文件验证： (不建议)

```
$ acme.sh --issue -d congm.in -d *.congm.in -w /home/webroot
复制代码
```

> 注意：对于通配符证书需要加`-d 域名 -d *.域名`两个参数`-w`即webroot，为该`域名`通过http所访问到的本地目录上面这段过程将会在`/home/webroot`创建一个 `.well-known` 的文件夹，同时 Let’s Encrypt 将会通过你要注册的域名去访问那个文件来确定权限，它可能会去访问 `http://congm.in/.well-known/` 这个路径，验证成功会自动清理

#### [#](https://link.juejin.cn/?target=https%3A%2F%2Fwww.xkboke.com%2Fweb-inn%2Fblog%2Fblog-7.html%23%25E6%2596%25B9%25E5%25BC%258F2%25E3%2580%2581dns-api%25E9%25AA%258C%25E8%25AF%2581-%25E6%258E%25A8%25E8%258D%2590 "https://www.xkboke.com/web-inn/blog/blog-7.html#%E6%96%B9%E5%BC%8F2%E3%80%81dns-api%E9%AA%8C%E8%AF%81-%E6%8E%A8%E8%8D%90")方式2、DNS-API验证：（推荐）

通过DNS服务器提供 key 与 secret 实现自动验证，详情 [github.com/Neilpang/ac…](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2FNeilpang%2Facme.sh%2Ftree%2Fmaster%2Fdnsapi "https://github.com/Neilpang/acme.sh/tree/master/dnsapi")

例如，在阿里云解析的域名，请前往[www.alibabacloud.com/help/zh/doc…](https://link.juejin.cn/?target=https%3A%2F%2Fwww.alibabacloud.com%2Fhelp%2Fzh%2Fdoc-detail%2F62656.htm "https://www.alibabacloud.com/help/zh/doc-detail/62656.htm")查看具体创建方法。控制台中申请子账号 API Token 并执行命令：

```
$ export Ali_Key="sdfsdfsdfljlbjkljlkjsdfoiwje"
$ export Ali_Secret="jlsdflanljkljlfdsaklkjflsa"
复制代码
```

```
$ acme.sh --issue -d congm.in -d *.congm.in --dns dns_ali
$ acme.sh --issue --dns dns_ali -d example.com -d www.example.com
复制代码
```

其他域名解析平台具体执行命令参考[github.com/Neilpang/ac…](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2FNeilpang%2Facme.sh%2Fwiki%2Fdnsapi "https://github.com/Neilpang/acme.sh/wiki/dnsapi")

> 注意：对于通配符证书需要加 `-d 域名 -d *.域名` 两个参数`Ali_Key` 和 `Ali_Secret` 将会保存在 `~/.acme.sh/account.conf` 执行上述命令后，证书文件将会自动申请被存放在 `~/.acme.sh/`对应的域名文件夹中，如：`~/.acme.sh/congm.in`。后续 `acme.sh` 将会自动更新该文件夹内的证书。

### [#](https://link.juejin.cn/?target=https%3A%2F%2Fwww.xkboke.com%2Fweb-inn%2Fblog%2Fblog-7.html%23_3%25E3%2580%2581%25E9%2583%25A8%25E7%25BD%25B2nginx "https://www.xkboke.com/web-inn/blog/blog-7.html#_3%E3%80%81%E9%83%A8%E7%BD%B2nginx")3、部署nginx

参考：[github.com/Neilpang/ac…](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2FNeilpang%2Facme.sh%2Fwiki%2F%25E8%25AF%25B4%25E6%2598%258E "https://github.com/Neilpang/acme.sh/wiki/%E8%AF%B4%E6%98%8E")

1.  拷贝证书

```
$ acme.sh --install-cert -d congm.in \
--key-file       /etc/nginx/ssl/congm.in.key \
--fullchain-file /etc/nginx/ssl/congm.in.cer \
--reloadcmd      'service nginx force-reload'
复制代码
```

> 注意：通过该命令可将 `~/.acme.sh/congm.in` 内的证书copy到指定位置`/etc/nginx`为nginx服务器实际的地址(可修改为自己服务器对应的地址)`service nginx force-reload` 为nginx重启命令(可修改为自己服务器对应的命令)，`force-reload` 会重载证书，后续 `acme.sh` 签发了新证书后就自动完成该拷贝过程

2.  配置nginx

Nginx-Conf

```
$ vim /etc/nginx/conf.d/congm.in.conf
复制代码
```

```
# http(80) -> https(443/ssl)
server {
    listen 80;
    server_name congm.in;
    rewrite ^(.*)$ https://$host$request_uri;
}
# congm.in
server {
    listen 443;
    server_name congm.in;
    include ssl/congm.in.ssl.conf;

    location / {
        # todo
    }
}
复制代码
```

SSL-Conf

```
$ vim /etc/nginx/ssl/congm.in.ssl.conf
复制代码
```

```
ssl on;
ssl_certificate ssl/congm.in.cer;
ssl_certificate_key ssl/congm.in.key;
复制代码
```

![](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/00ba359ecd0075e59ffbc3d810af551d.svg) 1

![](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/3d482c7a948bac826e155953b2a28a9e.svg) 收藏