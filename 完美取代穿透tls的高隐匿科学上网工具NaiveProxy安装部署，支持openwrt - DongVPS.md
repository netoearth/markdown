## 发生了什么

这段时间gfw防火墙又搞事情，我前几天发了个视频，收到了不少评论，~基~本可以确定，高风险的主要是部分ip段、tls与xtls伪装。

不管怎么样，通过cdn进一步伪装能够有效的复活vps，但是cdn使用的公共ip也并不是万无一失的，部分公共ip也被封了，这就导致有时候443能够正常访问，有时候却不能。

## **NaïveProxy**吼不吼

这就非常尴尬了，经过网友的推荐，了解到一款隐匿性更高的自建梯子，他就是**NaïveProxy**

**工作原理架构：**\[浏览器 → NaiveProxy客户端\] ⟶ GFW ⟶ \[常用前端 → NaiveProxy服务端\] ⟶ 互联网

由于 NaiveProxy 使用Chrome的网络堆栈，GFW审查截获的流量行为与Chrome和标准前端（如 Caddy、HAProxy）之间的常规 HTTP/2 流量完全相同。前端还会将未经身份验证的用户和活动探测器重新路由到后端HTTP服务器，从而使得无法检测到代理的存在，比如像这样：探查⟶常用前端⟶网站页面。

大家知道现在主流的tls都使用http/1.1进行伪装，这可能正是最近gfw能够识别并封杀tls类梯子的原因，而np使用的传输协议是http2与http3。

## http2/3介绍

http1.1使用的是文本的报文进行传输，而http2把报文压缩成了二进制格式，传输和解析速度都大幅提高，而且报文会在客户端与服务端进行互相验证，这就阻止了梯子被探测流量探测到，安全性大幅提高

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5ba24e56c7c84bfda6b382cb08090961~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

不仅如此，http2还支持多路复用，你可以简单理解成多线程

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b2f7d14013604bcd8d12a3139f39e091~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/42967e5085aa431d959ab80abd8a5a5a~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

但是现在有一个更信的http3，基于udp的quic协议

UDP是“无连接”的，根本就不需要“握手”和“挥手”，所以就比TCP来得快

同时因为naiveproxy通过重用了chrome的网络堆栈，就实现了流量的彻底隐藏，同时又获得了http3的高速传输

## 搭建**NaïveProxy**

而我们今天搭建的naiveproxy会直接启用http3的quic协议。因为udp与tcp的443端口不是同一个端口，所以即使你的vps被封掉了，你依然可以启用443端口来通过vps的梯子访问外网。

### 安装golang

```
wget https://go.dev/dl/go1.19.linux-amd64.tar.gz
tar -zxvf go1.19.linux-amd64.tar.gz -C /usr/local/
```

`vi /etc/profile`

```
#  /etc/profile 中添加 GO语言的 环境变量
cp /etc/profile /etc/profile.bak
echo export GOROOT=/usr/local/go >> /etc/profile
echo export PATH=$GOROOT/bin:$PATH  >> /etc/profile
```

```
source /etc/profile
go version
```

### **安装NaïveProxy和Caddy**

需要安装NaïveProxy，且不是单独安装Caddy，务必按照命令执行。

NaiveProxy项目主页：https://github.com/klzgrad/forwardproxy

项目主页：https://github.com/caddyserver/caddy

Caddy官方文档：https://caddyserver.com/docs/

提前抱怨几句，Caddy这个工具，我也说不上来为什么，配置起来真的是非常反直觉，为了实现无配置的傻瓜式上手可用，发明了一套奇怪的使用逻辑，类似docker-compose的Caddyfile配置启动参数，caddy会在启动时帮你完成配置

不好评价这个事情，但是作为nginx老用户真的是非常反感这种产品逻辑。

不吐槽了，开始搞起。

以下命令，在服务器上执行，需要保证服务器到github的网络通畅。编译build需要一定的时间，看你服务器的CPU性能，耐心等待。

```
mkdir src
cd src
go install github.com/caddyserver/xcaddy/cmd/xcaddy@latest
~/go/bin/xcaddy build --with github.com/caddyserver/forwardproxy@caddy2=github.com/klzgrad/forwardproxy@naive
cp caddy /usr/bin/
/usr/bin/caddy version        # 2022-4-8 23:09
#v2.4.6 h1:HGkGICFGvyrodcqOOclHKfvJC0qTU7vny/7FhYp9hNw=  
setcap cap_net_bind_service=+ep /usr/bin/caddy  # 设置bind权限，可443
```

**Caddy的配置文件**

通常大家都用的IPv4地址，故而，这里需要配置一个域名的A记录，指向你的服务器公网IPv4地址，且需要一套可信的证书文件，不要用自签的证书。

```
# 存放证书的目录
mkdir -p /etc/ssl/caddy
# 存放Caddyfile的目录
mkdir /etc/caddy/
# 
```

官方给到的配置示例

vim /etc/caddy/Caddyfile

```
:443, example.com
tls me@example.com
route {
  forward_proxy {
    basic_auth user pass
    hide_ip
    hide_via
    probe_resistance
  }
  file_server {
    root /var/www/html 
  }
}
```

语法解释，官方地址 [https://caddyserver.com/docs/json/](https://caddyserver.com/docs/json/)

```
:443, example.com   # example.com为服务器的A或者AAAA记录，域名
tls me@example.com   # 邮箱地址
route {
  forward_proxy {
    basic_auth user pass   # 自定义用户名和密码 #多用户就按照这个格式新增一行
    hide_ip
    hide_via
    probe_resistance  # 抗探测
  }
  reverse_proxy {another.website.domain} # 要反代的网站，二选一
  file_server { root /var/www/html } # 自检的网站，二选一
}
```

```
# 启动
caddy run --config "/etc/caddy/Caddyfile"
```

_**目前来说，国内的网络环境有Qos限流，会导致传输大文件比如观看youtube时发生quic断流，如果你那边经常发生quic断流的情况，就是用https而不是quic即可**_

我这边测试如果长时间超过20Mb/s会被强制断流，过段时间会自行恢复。

**配置服务**

```
# /etc/systemd/system/naive.service
[Unit]
Description=Caddy
Documentation=https://caddyserver.com/docs/
After=network.target network-online.target
Requires=network-online.target

[Service]
Type=notify
User=root
Group=root
ExecStart=/usr/bin/caddy run --environ --config /etc/caddy/Caddyfile
ExecReload=/usr/bin/caddy reload --config /etc/caddy/Caddyfile
TimeoutStopSec=5s
LimitNOFILE=1048576
LimitNPROC=512
PrivateTmp=true
ProtectSystem=full
AmbientCapabilities=CAP_NET_BIND_SERVICE

[Install]
WantedBy=multi-user.target
```

加载服务

```
systemctl daemon-reload
systemctl start naive
systemctl status naive
systemctl enable naive
```

json文件即是手动配置caddy的方式，好处是可以指定端口

`给到一个服务器的示范配置信息` 

```
#  /etc/caddy/caddy_server.json
{
  "admin": {
    "disabled": true
  },
  "apps": {
    "http": {
      "servers": {
        "srv0": {
          "listen": [
            ":443"
          ],
          "routes": [
            {
              "handle": [
                {
                  "handler": "subroute",
                  "routes": [
                    {
                      "handle": [
                        {
                          "auth_pass_deprecated": "User",
                          "auth_user_deprecated": "P_Bsa2022",
                          "handler": "forward_proxy",
                          "hide_ip": true,
                          "hide_via": true,
                          "probe_resistance": {}
                        }
                      ]
                    },
                    {
                      "match": [
                        {
                          "host": [
                            "域名"
                          ]
                        }
                      ],
                      "handle": [
                        {
                          "handler": "file_server",
                          "root": "/var/www/html",
                          "index_names": [
                            "index.html"
                          ]
                        }
                      ],
                      "terminal": true
                    }
                  ]
                }
              ]
            }
          ],
          "tls_connection_policies": [
            {
              "match": {
                "sni": [
                  "域名"
                ]
              }
            }
          ],
          "automatic_https": {
            "disable": true
          }
        }
      }
    },
    "tls": {
      "certificates": {
        "load_files": [
          {
            "certificate": "/etc/letsencrypt/live/域名/fullchain.pem",
            "key": "/etc/letsencrypt/live/域名/privkey.pem"
          }
        ]
      }
    }
  }
}
```

运行服务端

```
/usr/bin/caddy run --config /etc/caddy/caddy_server.jsonpc
```

客户端配置

到[Release](https://github.com/klzgrad/naiveproxy/releases)页面，下载 NaiveProxy 对应的客户端，解压执行，尽量用新版本的，修改config.json配置文件，编辑客户端配置文件，/etc/naive/config.json

```
{  
    "listen": "socks://127.0.0.1:1080",  
    "proxy": "https://用户名：密码@域名:18443" 
}
# 客户端执行 naive config.json
```

NaiveProxy 客户端配置（如果使用 HTTP3 则将 https:// 改为 quic://）

安卓客户端 

https://github.com/SagerNet/SagerNet/releases

x先下载sn客户端，然后下载naiveproxy-plugin

openwrt客户端

现在已知支持naiveproxy的客户端有passwall2，支持quic，其他的客户端虽然也支持naive，但是并没有quic支持