 [科学上网](https://itlanyan.com/category/for-free-internet/)2022年12月9日

本文目录

-   [为何需要流量伪装](https://itlanyan.com/v2ray-traffic-mask/#bnp_i_1)
-   [V2ray流量伪装前提条件](https://itlanyan.com/v2ray-traffic-mask/#bnp_i_2)
-   [V2ray流量伪装教程](https://itlanyan.com/v2ray-traffic-mask/#bnp_i_3)
-   [V2ray流量伪装总结](https://itlanyan.com/v2ray-traffic-mask/#bnp_i_4)
-   [参考](https://itlanyan.com/v2ray-traffic-mask/#bnp_i_5)

> 欢迎加入tg群交流：[@tlanyantg](https://t.me/tlanyantg)

前文 “[V2Ray教程](https://itlanyan.com/v2ray-tutorial)” 介绍了`V2Ray`基本用法，本文介绍`V2Ray`高级使用技巧：流量伪装。客户端下载请访问：[V2ray客户端](https://itlanyan.com/v2ray-clients-download)，拯救被墙的服务器请参考：[拯救被墙的服务器](https://itlanyan.com/recovery-blocked-ip/)。

## 为何需要流量伪装

自设立墙以来，我国的网络封锁技术一直是全球领先的。有方校长等人持续做贡献，封锁和干扰技术也在不断演化和进步。传统的VPN、[ssh隧道](https://itlanyan.com/configure-ssh-to-visit-extenal-websites/)等科学上网方式渐渐被墙识别和干扰，访问境外网站越来越难，倒逼网民不断推动穿墙技术的发展。

科学上网这些年来，见证了诸多技术的兴起和消沉，自架服务器被绊者更是数不胜数。经历过谈笑风生的岁月，才知闷声发大财是人生真理。翻墙技术同样如此，一是不要高调和张扬，否则大概率被打击和屏蔽；其次流量尽量与常见流量接近，不要特立独行，例如使用非常用端口、自定义奇怪的协议。本文介绍的V2ray伪装便是将穿墙流量以常见的HTTPS/TLS包装，大大降低vps被墙或被干扰的可能性，在敏感时期提供稳如狗的上外网体验。

> V2ray发布了新版的VLESS协议，强烈推荐使用，详情请参考 [V2ray的VLESS协议介绍和使用教程](https://itlanyan.com/introduce-v2ray-vless-protocol/)
> 
> 推荐使用XTLS+Splice技术的Xray，详情参考：[Xray教程](https://itlanyan.com/xray-tutorial/)

## V2ray流量伪装前提条件

本文假设读者已经具备以下条件：

1.  一台境外的vps，购买可参考：[一些VPS商家整理](https://itlanyan.com/vps-merchant-collection/);
2.  一个域名，无备案要求（备案可使用国内CDN加速，但意味着万一有事，被喝茶更容易）。域名购买可参考：[Namesilo域名注册和使用教程](https://itlanyan.com/namesilo-domain-tutorial/) 或从 [适合国人的域名注册商推荐](https://itlanyan.com/domain-register-for-mainland/) 选购；
3.  为域名申请一个证书，请参考：[使用acme.sh签发证书](https://itlanyan.com/use-acme-sh-get-free-cert/) 或 [从阿里云获取免费SSL证书](https://itlanyan.com/get-free-ssl-certificates-from-aliyun/)(伪装域名用来做站推荐使用)；
4.  有基本linux技巧，能使用vim/nano等编辑器。

理论上来说，证书不是必须的。但没有tls加持或不做加密，防火墙直接能看出来流量真实意图从而进行干扰，这也是为什么不建议伪装http流量的原因。本文给出的方法采用合法机构签发的证书对流量进行加密，不是做特征混淆得到的TLS流量，从而更难被检测和干扰。

关于伪装技术的选择，**V2ray web+websocket+tls** 和 **V2ray web+http2+tls** 常用来做对比。理论上http2省去了upgrade的请求，性能更好。但实际使用中两者没有明显区别，加之某些web服务器（例如Nginx）不支持后端服务器为http2，所以websocket的方式更流行。如果你要上http2，记得web服务器不能用Nginx，要用支持反代http2的Caddy等软件。

下文介绍流量伪装的配置步骤，演示域名为`tlanyan.pp.ua`，服务器为Linux(CentOS)，web服务器软件用Nginx，web+websocket+tls组合，最终效果为：http/https方式打开域名，显示正常的网页；`V2Ray`客户端请求特定的路径，例如`https://itlanyan.com/awesomepath`，能科学上网；浏览器直接请求`https://itlanyan.com/awesomepath`，返回”400 bad request”。即外部看起来完全是一个人畜无害的正规网站，特定手段请求特定网址才是科学上网的通道。

**注意：**如果使用伪装后网速慢，可尝试更轻量级的trojan，请参考：[trojan教程](https://itlanyan.com/trojan-tutorial/)。

## V2ray流量伪装教程

服务端涉及到了Nginx和v2ray，分别介绍其配置。

### 1\. 配置dns

先设置dns将域名解析到vps的ip，例如`tlanyan.pp.ua`解析到xxx.xxx.xx.xx。

如果你上了CDN，则dns要解析到cdn给的ip或者别名网址（cname）。使用cdn能隐藏真实vps的ip，避免vps被墙或能拯救被封锁ip的vps。上cdn有好处，但国内cdn要求域名备案，国外cdn基本上会降低网速，而且配置起来稍显麻烦。建议新手先摸透https流量伪装，有特殊需求再考虑上cdn。

**注意：**如果你要用Let’s Encrypt签发的证书，并且希望上cdn，那么应该先解析到vps的ip，获取到证书后再解析到cdn。

**更新**：上cdn可参考：[v2ray使用cloudflare中转流量，拯救被墙ip](https://itlanyan.com/go.php?key=cloudflare-forward)，请务必记得先让伪装正常工作再上cdn！

### 2\. 配置Nginx

如果你的域名并正确配置了SSL证书，可忽略这一步。

Nginx是市面上占有率最高的网站服务器软件，Centos 7系统安装nginx命令：`yum install -y epel-release && yum install -y nginx`。

Linux系统上Nginx默认站点配置文件是`/etc/nginx/conf.d/`目录下的`default.conf`（debian/Ubuntu系统路径是 `/etc/nginx/site-enabled/default.conf`），我们对伪装网站进行全站https配置，示例内容如下：

```
server {
    listen 80;
    server_name xxxxx;  # 改成你的域名
    rewrite ^(.*) https://$server_name$1 permanent;
}

server {
    listen       443 ssl http2;
    server_name xxxxx;
    charset utf-8;

    # ssl配置
    ssl_protocols TLSv1.2 TLSv1.3; # tls 1.3要求nginx 1.13.0及以上版本
    ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384;
    ssl_prefer_server_ciphers off;
    ssl_session_cache shared:SSL:10m;
    ssl_session_timeout 1d;
    ssl_session_tickets off;
    ssl_certificate xxxxx; # 改成你的证书地址
    ssl_certificate_key xxxx; # 改成证书密钥文件地址

    access_log  /var/log/nginx/xxxx.access.log;
    error_log /var/log/nginx/xxx.error.log;

    root /usr/share/nginx/html;
    location / {
        index  index.html;
    }
}
```

配置好用`nginx -t`命令查看有无错误，没问题的话`systemctl restart nginx`启动Nginx。打开浏览器在地址栏输入域名，应该能看到https访问的Nginx欢迎页。

新域名如何快速做一个像模像样的网站？最简单的办法是从网上下载网站模板，上传到web服务器的根目录（默认是`/usr/share/nginx/html`）。对于伪装站来说，静态站足够。如果你的境外流量比较大，建议用爬虫或者其他手段做一个看起来受欢迎、流量大的站点，例如美食博客，图片站等。

### 3\. 安装配置V2Ray

详细过程可参考上篇：[V2Ray教程](https://itlanyan.com/v2ray-tutorial)。为了让V2ray能正常运行，请确认防火墙已放行对应端口，并且关闭了selinux：`setenforce 0`。

到此为止，Nginx和V2ray应该都能各自独立正常工作。如果有一个出现问题，应该先解决再继续下面的操作。

### 4\. Nginx配置websocket

接下来我们让Nginx和v2ray结合，完成服务端的配置。

首先我们选择一个**伪装路径**，建议为二级或者较长的一级路径，例如`/abc/def`或`/awesomepath`。

配置Nginx将伪装路径的访问都转发到v2ray。编辑`/etc/nginx/conf.d/default.conf`的第二个`server`段，增加以下转发配置：

```
location /awesomepath { # 与 V2Ray 配置中的 path 保持一致
      proxy_redirect off;
      proxy_pass http://127.0.0.1:12345; # 假设v2ray的监听地址是12345
      proxy_http_version 1.1;
      proxy_set_header Upgrade $http_upgrade;
      proxy_set_header Connection "upgrade";
      proxy_set_header Host $host;
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
}
```

配置好后重启Nginx：`systemctl restart nginx`。

配置v2ray接受Nginx传来的数据。编辑 `/etc/v2ray/config.json` 文件，在“inbounds”中新增“streamSetting”配置，设置传输协议为“websocket”。配置好后`config.json`文件看起来是：

```
{
  "log": {
    "loglevel": "warning",
    "access": "/var/log/v2ray/access.log",
    "error": "/var/log/v2ray/error.log"
   },
  "inbounds": [{
    "port": 12345,
    "protocol": "vmess",
    "settings": {
      "clients": [
        {
          "id": "xxxxx", # 可以使用/usr/bin/v2ray/v2ctl uuid生成
          "level": 1,
          "alterId": 0
        }
      ]
    },
    "streamSettings": {     # 载体配置段，设置为websocket
        "network": "ws",
        "wsSettings": {
          "path": "/awesomepath"  # 与nginx中的路径保持一致
        }
      },
    "listen": "127.0.0.1" # 出于安全考虑，建议只接受本地链接
  }],
  "outbounds": [{
    "protocol": "freedom",
    "settings": {}
  },{
    "protocol": "blackhole",
    "settings": {},
    "tag": "blocked"
  }],
  "routing": {
    "rules": [
      {
        "type": "field",
        "ip": ["geoip:private"],
        "outboundTag": "blocked"
      }
    ]
  }
}
```

> **注意：**
> 
> 1\. json文件不支持注释，上述配置中”#”号及后续内容都要删掉；
> 
> 2\. 因为使用tls加密，**强烈建议alterId改成0**，以节省cpu加快v2ray的速度

配置无误后，重启v2ray服务：`systemctl restart v2ray`。

如何测试nginx与v2ray结合没有问题？打开浏览器，输入域名及其他路径，应该显示正常网页或者页面不存在，说明Nginx正常工作；输入域名加v2ray路径，例如`https://itlanyan.com/awesomepath`，应该出现”Bad Request”，说明Nginx将流量转发给了v2ray，并且v2ray收到了请求。

### 客户端设置

最后是配置客户端，本文以Windows平台的`V2RayW`软件为例说明使用方法。

打开`V2RayW`，右键托盘图标，点击“配置”。在弹框中新建或修改已有的服务器，输入服务器ip，端口写443，把用户id、额外id信息填上，网络类型选择”ws”。接着点“传输设置”，找到“websocket”，路径一栏输入nginx和v2ray中的路径，例如“/awesomepath”；http头部输入：

```
{
"Host":"你的域名，例如tlanyan.pp.ua"
}
```

截图如下：

[![v2ray客户端配置websocket](https://itlanyan.com/wp-content/uploads/2019/09/V2RAY1.png "v2ray客户端配置websocket")](https://itlanyan.com/v2ray-traffic-mask/v2ray1/)

接着点击“tls”，勾选“启用传输层加密tls”（同时建议勾选“允许不安全的加密方式”和“允许不安全连接”），在“服务器域名”的输入框中输入域名，截图如下：

[![v2ray客户端配置tls](https://itlanyan.com/wp-content/uploads/2019/09/v2ray2.png "v2ray客户端配置tls")](https://itlanyan.com/v2ray-traffic-mask/v2ray2/)

信息填写正确后，点击“保存”。打开浏览器访问google.com，youtube.com等网站，配置无误的话应该都能正常打开。

> 因为启用了TLS，因此VMESS配置界面的加密方式(security)建议配置为none，以节省cpu，提高设备续航能力

如果对nginx/v2ray以及客户端的配置不熟悉，建议使用这个v2ray配置生成工具：[v2ray配置生成](https://www.veekxt.com/utils/v2ray_gen)。

## V2ray流量伪装总结

https/tls会加密路径信息，仅靠中间环节捕捉到的流量包极难区分是正常请求还是夹带私货的流量。这也显示了v2ray的强大之处：通过配置不同的协议和载体，就能对进出的流量做定制。从流量伪装、反向代理的功能上看，v2ray毫无疑问的是一个强大的网络框架/工具，科学上网功能只是其一个成功应用。

比较让人遗憾的是v2ray的ios客户端均收费，客户端下载请访问：[V2ray客户端下载](https://itlanyan.com/v2ray-clients-download)。

## 参考

1.  [新 V2Ray 白话文指南](https://guide.v2fly.org/advanced/advanced.html)
2.  [Use HTTP/2.0 between nginx reverse-proxy and backend webserver](https://serverfault.com/questions/765258/use-http-2-0-between-nginx-reverse-proxy-and-backend-webserver)

赞(441)