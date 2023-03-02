-   [前言](https://v2rayssr.com/xrayall.html#%E5%89%8D%E8%A8%80 "前言")
-   [准备工作](https://v2rayssr.com/xrayall.html#%E5%87%86%E5%A4%87%E5%B7%A5%E4%BD%9C "准备工作")
-   [开始搭建](https://v2rayssr.com/xrayall.html#%E5%BC%80%E5%A7%8B%E6%90%AD%E5%BB%BA "开始搭建")
    -   [更新系统](https://v2rayssr.com/xrayall.html#%E6%9B%B4%E6%96%B0%E7%B3%BB%E7%BB%9F "更新系统")
    -   [安装 Curl Nginx Tar](https://v2rayssr.com/xrayall.html#%E5%AE%89%E8%A3%85_Curl_Nginx_Tar "安装 Curl Nginx Tar")
    -   [安装官方Xray服务](https://v2rayssr.com/xrayall.html#%E5%AE%89%E8%A3%85%E5%AE%98%E6%96%B9Xray%E6%9C%8D%E5%8A%A1 "安装官方Xray服务")
    -   [为域名申请证书](https://v2rayssr.com/xrayall.html#%E4%B8%BA%E5%9F%9F%E5%90%8D%E7%94%B3%E8%AF%B7%E8%AF%81%E4%B9%A6 "为域名申请证书")
    -   [设置 Nginx 开机启动](https://v2rayssr.com/xrayall.html#%E8%AE%BE%E7%BD%AE_Nginx_%E5%BC%80%E6%9C%BA%E5%90%AF%E5%8A%A8 "设置 Nginx 开机启动")
    -   [检验Xray配置文件](https://v2rayssr.com/xrayall.html#%E6%A3%80%E9%AA%8CXray%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6 "检验Xray配置文件")
    -   [下载伪装网站及部署](https://v2rayssr.com/xrayall.html#%E4%B8%8B%E8%BD%BD%E4%BC%AA%E8%A3%85%E7%BD%91%E7%AB%99%E5%8F%8A%E9%83%A8%E7%BD%B2 "下载伪装网站及部署")
-   [后记](https://v2rayssr.com/xrayall.html#%E5%90%8E%E8%AE%B0 "后记")

## 前言

很多小伙伴在咨询 Xray 和 Trojan 共存等一些方面的教程，其实吧，Xray 它是支持 Trojan 协议的。

我们前几期视频提到了 Xray 强大的回落功能，其实 Xray 的分流功能也是很强大，大家看下面一些我们经常搭建的协议

-   VLESS over TCP with XTLS（数倍性能，首选方式）
-   VLESS over TCP with TLS
-   VLESS over WS with TLS
-   VMess over TCP with TLS
-   VMess over WS with TLS
-   Trojan over TCP with TLS

在 Xray 中，我们是支持上面的 **所有协议** **同时** 连接我们的服务器的。

不相信？那我们继续！

点击下图观看本期视频教程：

[![](https://v2rayssr.com/wp-content/uploads/2021/03/%E6%88%AA%E5%B1%8F2021-03-21-09.24.19.png)](https://v2rayssr.com/go?url=https://www.youtube.com/watch?v=Jss-hiPg0Sg)

## 准备工作

1、VPS 一台，重置好主流的操作系统。

2、域名一个，并解析到 VPS，若是需要开启 CDN，请自行托管域名到 CloudFlare。

（若是域名申请、解析以及托管到 CloudFlare 不会的话，请 [点击这里看教程](https://v2rayssr.com/yumingreg.html)）

3、自行安装 BBR 加速之类的软件，脚本 [请点击](https://v2rayssr.com/bbr.html)

4、实在不会请点击 [观看视频教程](https://v2rayssr.com/go?url=https://youtu.be/Jss-hiPg0Sg)

## 开始搭建

### 更新系统

CentOS 需要安装开源发行软件包版本库，命令如下

```
yum install epel-release -yyum update -y
```

Debian更新系统较为简单

```
apt update -y
```

### 安装 Curl Nginx Tar

```
yum install curl tar nginx -y   #CentOS命令apt install curl tar nginx -y   #Debian命令
```

**CentOS 为了避免FireWalld，请务必直接执行下面代码，Debian跳过**

```
firewall-cmd --zone=public --add-port=80/tcp --permanentfirewall-cmd --zone=public --add-port=443/tcp --permanentfirewall-cmd --reload
```

启动 Nginx服务

（PS：在执行下面命令之前，一定记得重启一下 VPS，以免上述的更新命令没有完成导致 Nginx 无法启动，有的搬瓦工机器需要人工重启——因为我的就是如此 ）

```
systemctl start nginx
```

现在可以在浏览器中输入你的域名，看看是否可以访问到 Nginx 的欢迎页面

### ![](https://v2rayssr.com/wp-content/uploads/2021/03/%E6%88%AA%E5%B1%8F2021-03-05-20.32.00.png)

### 安装官方Xray服务

以下一键安装程序来源于官方

```
bash -c "$(curl -L https://github.com/XTLS/Xray-install/raw/main/install-release.sh)" @ install -u root
```

UUID 随机生成代码

```
cat /proc/sys/kernel/random/uuid # 粘贴到VPS运行即可生成 UUID
```

安装完毕以后，在VPS目录 /usr/local/etc/xray 找到 config,json 文件，贴入下面的配置文件

更多范例的 Xray 配置文件，请点击 [官方的 Xray 配置模板库](https://v2rayssr.com/go?url=https://github.com/XTLS/Xray-examples)

```
{    "log": {        "loglevel": "warning"    },    "inbounds": [        {            "port": 443,            "protocol": "vless",            "settings": {                "clients": [                    {                        "id": "c4616432-97cd-4514-8979-f3a426fccdfd", // 填写你的 UUID                        "flow": "xtls-rprx-direct",                        "level": 0,                        "email": "love@example.com"                    }                ],                "decryption": "none",                "fallbacks": [                    {                        "dest": 1310, // 默认回落到 Xray 的 Trojan 协议                        "xver": 1                    },                    {                        "path": "/websocket", // 必须换成自定义的 PATH                        "dest": 1234,                        "xver": 1                    },                    {                        "path": "/vmesstcp", // 必须换成自定义的 PATH                        "dest": 2345,                        "xver": 1                    },                    {                        "path": "/vmessws", // 必须换成自定义的 PATH                        "dest": 3456,                        "xver": 1                    }                ]            },            "streamSettings": {                "network": "tcp",                "security": "xtls",                "xtlsSettings": {                    "alpn": [                        "http/1.1"                    ],                    "certificates": [                        {                            "certificateFile": "/usr/local/etc/xray/cert/cert.crt", // 换成你的证书，绝对路径                            "keyFile": "/usr/local/etc/xray/cert/private.key" // 换成你的私钥，绝对路径                        }                    ]                }            }        },        {            "port": 1310,            "listen": "127.0.0.1",            "protocol": "trojan",            "settings": {                "clients": [                    {                        "password": "00000000", // 填写你的密码                        "level": 0,                        "email": "love@example.com"                    }                ],                "fallbacks": [                    {                        "dest": 80 // 或者回落到其它也防探测的代理                    }                ]            },            "streamSettings": {                "network": "tcp",                "security": "none",                "tcpSettings": {                    "acceptProxyProtocol": true                }            }        },        {            "port": 1234,            "listen": "127.0.0.1",            "protocol": "vless",            "settings": {                "clients": [                    {                        "id": "c4616432-97cd-4514-8979-f3a426fccdfd", // 填写你的 UUID                        "level": 0,                        "email": "love@example.com"                    }                ],                "decryption": "none"            },            "streamSettings": {                "network": "ws",                "security": "none",                "wsSettings": {                    "acceptProxyProtocol": true, // 提醒：若你用 Nginx/Caddy 等反代 WS，需要删掉这行                    "path": "/websocket" // 必须换成自定义的 PATH，需要和分流的一致                }            }        },        {            "port": 2345,            "listen": "127.0.0.1",            "protocol": "vmess",            "settings": {                "clients": [                    {                        "id": "c4616432-97cd-4514-8979-f3a426fccdfd", // 填写你的 UUID                        "level": 0,                        "email": "love@example.com"                    }                ]            },            "streamSettings": {                "network": "tcp",                "security": "none",                "tcpSettings": {                    "acceptProxyProtocol": true,                    "header": {                        "type": "http",                        "request": {                            "path": [                                "/vmesstcp" // 必须换成自定义的 PATH，需要和分流的一致                            ]                        }                    }                }            }        },        {            "port": 3456,            "listen": "127.0.0.1",            "protocol": "vmess",            "settings": {                "clients": [                    {                        "id": "c4616432-97cd-4514-8979-f3a426fccdfd", // 填写你的 UUID                        "level": 0,                        "email": "love@example.com"                    }                ]            },            "streamSettings": {                "network": "ws",                "security": "none",                "wsSettings": {                    "acceptProxyProtocol": true, // 提醒：若你用 Nginx/Caddy 等反代 WS，需要删掉这行                    "path": "/vmessws" // 必须换成自定义的 PATH，需要和分流的一致                }            }        }    ],    "outbounds": [        {            "protocol": "freedom"        }    ]}
```

### 为域名申请证书

若你是用下面的 acme 脚本无法申请的话，请 [点击这里用其他方式申请](https://v2rayssr.com/go?url=https://freessl.cn/)

（记得申请完毕以后，证书文件命名为 cert.crt ，密钥文件命名为 private.key，存放到 /usr/local/etc/xray/cert/ 文件夹之下，若是没有该文件夹，请自行新建！）

以下六行脚本请逐行复制粘贴到 VPS 运行

（**PS：务必修改下面的域名和邮箱为你自己的域名、邮箱**）

```
curl https://get.acme.sh | sh~/.acme.sh/acme.sh --register-account -m xxxx@xxxx.com~/.acme.sh/acme.sh  --issue  -d 1.bozai.us  --webroot /usr/share/nginx/html/mkdir /usr/local/etc/xray/cert~/.acme.sh/acme.sh --installcert -d 1.bozai.us --key-file /usr/local/etc/xray/cert/private.key --fullchain-file /usr/local/etc/xray/cert/cert.crt~/.acme.sh/acme.sh --upgrade --auto-upgradechmod -R 755 /usr/local/etc/xray/cert
```

证书默认的更新周期为60天（自动），若是出问题，请再次自行框内的代码即可

### 设置 Nginx 开机启动

并重新启动 Nginx

```
systemctl enable nginxsystemctl restart nginx
```

### 检验Xray配置文件

```
systemctl restart xray  #重启xray服务systemctl status xray  #查看xray运行状态
```

### 下载伪装网站及部署

默认的网站主程序文件夹在 /usr/share/nginx/html/ ，大家可以自行的替换里面的任何东西（整站程序）

```
rm -rf /usr/share/nginx/html/*cd /usr/share/nginx/html/wget https://github.com/V2RaySSR/Trojan/raw/master/web.zipunzip web.zipsystemctl restart nginx
```

到此，就部署完毕了。相对来说也是很简单。现在你就可以打开 QV2ray 和 V2ray 连接你的节点了

具体的填入参数可以参考下图：

![](https://v2rayssr.com/wp-content/uploads/2021/03/1.png) ![](https://v2rayssr.com/wp-content/uploads/2021/03/2.png) ![](https://v2rayssr.com/wp-content/uploads/2021/03/3.png)

## 后记

大家可以随心所欲的去互联网看小姐姐了。

**若是还不会，请看视频教程！** [点击观看视频教程](https://v2rayssr.com/go?url=https://youtu.be/Jss-hiPg0Sg)