 [科学上网](https://itlanyan.com/category/for-free-internet/)2022年12月9日

本以为 [V2ray的VLESS协议介绍和使用教程](https://itlanyan.com/introduce-v2ray-vless-protocol/) 是[V2ray系列教程](https://itlanyan.com/tag/v2ray%E6%95%99%E7%A8%8B/) 的最后一篇，万万没想到Xray横空出世，并且项目发展高歌猛进，风光无限。鉴于此，有必要为学不动的网友科普一下Xray项目。

本Xray教程重点介绍Xray项目由来、和V2ray的关系，至于服务端部署和客户端使用，目前基本与V2ray一致，因此仅做简要介绍。

## Xray介绍

一个[Debian](https://itlanyan.com/from-centos-to-debian/)包维护者发现[XTLS库](https://github.com/XTLS/Go)的LICENSE不是BSD许可，提了一个issue希望作者 [@rprx](https://github.com/rprx) 能修改方便打包，详见 [https://github.com/XTLS/Go/issues/9](https://github.com/XTLS/Go/issues/9)。由这个issue引发了广泛讨论，rprx认为目前许可不是问题，也有不少人认为协议是立场的体现，各执一词。

最终V2ray(V2fly社区)维护者经过投票确认XTLS不符合V2ray的MIT协议，并在[V2ray-core 4.33.0版本移除了XTLS](https://github.com/v2fly/v2ray-core/releases/tag/v4.33.0)。rprx和其拥护者行动起来，很快就创建了[Project X项目](https://github.com/XTLS)和[Xray子项目](https://github.com/XTLS/Xray-core)（Xray取名来自XTLS和V2ray的结合），并发布了Xray-core的多个版本。这便是Xray的大致由来。

[![xray项目](https://itlanyan.com/wp-content/uploads/2020/12/xray%E6%95%99%E7%A8%8B.jpg "Xray项目")](https://itlanyan.com/xray-tutorial/xray%e6%95%99%e7%a8%8b/)

xray项目

XTLS和Xray离不开作者 [@rprx](https://github.com/rprx) 的辛勤付出，因此也简要介绍一下[@rprx](https://github.com/rprx) ：

1\. [@rprx](https://github.com/rprx) 是[VLESS协议](https://itlanyan.com/introduce-v2ray-vless-protocol/)的设计者，在介绍VLESS协议时写下了 [性能至上、可扩展性空前，目标是全场景终极协议](https://github.com/v2ray/v2ray-core/issues/2636) 的宏壮愿景；  

2\. [@rprx](https://github.com/rprx) 是 XTLS 的作者，在 [XTLS库](https://github.com/XTLS/Go) 中写下了 “THE FUTURE” 的霸气描述。将内外两条TLS连接结合，rprx可能不是第一个有这想法的人，但却是第一个将其实现、并成熟应用到实际中的作者。从使用表现上看，XTLS无愧于rprx对其的评价：[划时代的革命性概念与技术：XTLS](https://github.com/rprx/v2ray-vless/releases/tag/xtls)，以及社区给出的“黑科技”称谓；

3\. [@rprx](https://github.com/rprx) 是Project X和Xray项目的创始人。由于LICENSE理念之争，rprx创建了对标Project V和V2ray-core的Project X和Xray-core项目，广受欢迎。

Xray-Core自上个月创立以来，短短一个月已经发布了七个版本，足见维护者的诚意。Xray-Core目前发布的各个版本主要介绍如下：

1\. **Xray-core 1.0.0版本**：项目创建，提供完整的VLESS和XTLS支持，功能上是V2ray-core的超集。主要变动是将v2ray和v2ctl可执行文件合并为xray，性能全面增强；

2\. **Xray-core 1.1.0和1.1.1版本**：测试过渡版本，未提供详细发行说明；

3\. **Xray-core 1.1.2版本**：引入了Linux Kernel Splice技术，适用于安卓、路由器等Linux环境。Splice技术减少了数据拷贝次数和内存占用，拥有更强的性能。需要说明的是，只适用于类Linux环境的客户端出栈，服务端入栈仍然是direct；

4\. **Xray-core 1.1.3版本**：重构了透明代理的 REDIRECT 模式，使之同时支持 IPv4 和 IPv6，解决了V2ray和trojan遗留的问题；

5\. **Xray-core 1.1.4版本**：优化内存占用，TLS更多选项配置，使服务端伪装站的TLS设置在[SSL Labs](https://www.ssllabs.com/ssltest/)能达到A+评分；

6\. **Xray-core 1.1.5版本**：测试过渡版本，支持YAML配置文件、OCSP Stapling等功能，官方安装脚本大更新；

7\. **Xray-core 1.2.0版本**：SS、trojan协议完美支持Fullcone，向游戏使用迈出了重要一步。

更多Xray-core新特性请参考官方说明：[https://xtls.github.io/about/new/](https://xtls.github.io/about/new/)，或者官方库发行说明：[https://github.com/XTLS/Xray-core/releases](https://github.com/XTLS/Xray-core/releases)

## Xray和V2ray的区别

在说明Xray和V2ray区别之前，先说一下三个相近但不同的概念：

-   **V2ray**：Project V 是用于构建基础通信网络的工具合集，其核心工具称为`V2Ray`。V2ray主要负责网络协议和功能的实现，既可以单独运行，也可以和其它工具配合。V2ray官网是：[https://v2ray.com/](https://v2ray.com/)，Github项目主页是：[https://github.com/v2ray](https://github.com/v2ray)，TG讨论组是：[@projectv2ray](https://t.me/projectv2ray)；
-   **V2fly**：出现一些科学上网作者被喝茶事件后，V2ray原开发者长期不上线，其他维护者没有完整权限，导致V2ray项目维护困难。因此社区在2019年组建了V2fly组织，继续维护V2ray，也是目前V2ray发展的主力。V2fly官网是：[https://www.v2fly.org](https://www.v2fly.org/)，Github项目主页是：[https://github.com/v2fly](https://github.com/v2fly)，TG通知频道：[@v2fly](https://t.me/v2fly)，TG交流群为：[@v2fly\_chat](https://t.me/v2fly_chat)；
-   **Xray**：因许可理念之争，VLESS和XTLS的作者单独创建了Xray项目，目前是V2ray的超集，后续可能有不同的发展路线。Xray文档官网（测试中）：[https://xtls.github.io/](https://xtls.github.io/)， Github项目主页：[https://github.com/XTLS](https://github.com/XTLS)，TG交流群：[@projectXray](https://t.me/projectXray)。

从上面可以看到，先有V2ray(Project V)，然后是V2fly，最后才出来Xray(Project X)。其中V2fly是V2ray的社区，可以认为两者是同一个组织。

详细一点说，Xray和V2ray区别如下：

1.  **Xray是V2ray的一个分支(Fork)**。Xray项目基于V2ray而来，其支持并且兼容V2ray的配置；
2.  **Xray是V2ray的超集**。虽然最新版V2ray删除了XTLS，但仍保留VLESS协议。Xray提供完整的VLESS和XTLS支持，目前是V2ray的超集，但后续Xray可能会有会有自己的发展方向；
3.  **如果使用XTLS，强烈推荐使用Xray**，或者安装V2ray-Core 4.29.0 ～ 4.32.1版本；不使用XTLS的情况下，使用V2ray和Xray均可。

一个小提示是，Xray项目创建以来，V2ray没再发布新版本，反而Xray热火朝天，不断出新版和新功能。此外Xray的TG群也非常热闹，每天至少七八K的消息。如果你喜欢尝试新东西和折腾，Xray适合你，否则V2ray也挺好用。

如今Xray和V2ray分家，后续有没有可能Xray再合并回V2ray呢？这个问题没有答案，也许会合并回去结束分裂，也可能就此分道扬镳。io.js从Node.js分出后来又合并回去，C++源自C但完全是一门新语言独立发展，因此一切皆有可能。

## Xray安装和使用教程

和其他自行部署的技术相同，使用Xray分为安装服务端和配置客户端两部分，接下来分别做介绍。

### 安装Xray

1\. 准备一台境外的VPS，购买可参考 [一些VPS商家整理](https://itlanyan.com/vps-merchant-collection/);

2\. SSH连接到境外服务器，windows请参考：[Bitvise连接Linux服务器教程](https://itlanyan.com/go.php?key=bitvise-tutorial)，mac系统请参考：[Mac电脑连接Linux教程](https://itlanyan.com/go.php?key=mac-ssh-tutorial)；

3\. 自行部署Xray服务端需要你有基本linux技巧，能使用vim/nano等编辑器。官方提供了[大多数Linux系统的一键脚本](https://github.com/XTLS/Xray-install)，可以直接使用：

```
bash <(curl -L https://github.com/XTLS/Xray-install/raw/main/install-release.sh) install
```

官方脚本安装的文件符合[FHS规范](https://en.wikipedia.org/wiki/Filesystem_Hierarchy_Standard)，可执行文件xray在 `/usr/local/bin` 目录下，配置文件位于 `/usr/local/etc/xray`目录内。

4\. 官方脚本安装的配置文件内容为空，可参考 [Xray-examples](https://github.com/XTLS/Xray-examples) 中提供的模板编辑配置文件。例如使用VLESS+TCP+XTLS的配置文件为：

```
{
    "log": {
        "loglevel": "info"
    },
    "inbounds": [
        {
            "port": 443, # 可以换成其他端口
            "protocol": "vless",
            "settings": {
                "clients": [
                    {
                        "id": "", // 填写UUID，可以使用xray uuid生成
                        "flow": "xtls-rprx-direct",
                        "level": 0
                    }
                ],
                "decryption": "none",
                "fallbacks": [
                    {
                        "dest": 80 // 回落配置，可以直接转到其他网站，例如"www.baidu.com:80"
                    }
                ]
            },
            "streamSettings": {
                "network": "tcp",
                "security": "xtls",
                "xtlsSettings": {
                    "alpn": [
                        "http/1.1"
                    ],
                    "certificates": [
                        {
                            "certificateFile": "/path/to/fullchain.crt", // 换成你的证书，绝对路径
                            "keyFile": "/path/to/private.key" // 换成你的私钥，绝对路径
                        }
                    ]
                }
            }
        }
    ],
    "outbounds": [
        {
            "protocol": "freedom"
        }
    ]
}
```

XTLS需要证书，因此需要一个域名并申请证书。域名不需要备案，国内和国外买的都可以。域名购买可参考：[Namesilo域名注册和使用教程](https://itlanyan.com/namesilo-domain-tutorial/) 或从 [适合国人的域名注册商推荐](https://itlanyan.com/domain-register-for-mainland/) 选购。域名申请证书可参考[使用acme.sh签发证书](https://itlanyan.com/use-acme-sh-get-free-cert/) 或 [从阿里云获取免费SSL证书](https://itlanyan.com/get-free-ssl-certificates-from-aliyun/)。

> fallback选项以及ALPN等设置请参考：[VLESS协议的fallback参数介绍](https://itlanyan.com/vless-fallback-object/)

5\. 配置完毕后，可通过 `systemctl start xray` 运行 xray，`systemctl stop xray` 停止xray，`systemctl restart xray` 重启，`journalctl -xe --no-pager -u xra`y 查看运行日志。

最后，记得放行防火墙。如果是阿里云、腾讯云、AWS/GCP等大厂的服务器，还需要到网页后台的安全组放行端口。

### 配置Xray客户端

服务端配置好后，接下来是配置客户端。目前有如下客户端支持Xray：

**Xray Windows客户端**：

-   V2rayN：3.28版本起支持xray，只需要下载[Xray-core](https://github.com/XTLS/Xray-core/releases)，将解压的文件放到V2rayN-Core文件夹下即可。需要注意的是V2rayN 4.0版本移除了PAC，改用路由规则，会给习惯了PAC的用户带来困扰。习惯Qv2ray的网友应该乐于接受这个改变；
-   winXray：winXray是Windows系统上简洁稳定的[Xray/V2Ray](https://itlanyan.com/v2ray-clients-download/)、[Shadowsocks](https://itlanyan.com/shadowsock-clients/)、[Trojan](https://itlanyan.com/trojan-go-clients-download/) 通用客户端，可自动检测并连接**访问速度最快的** 代理服务器。该项目原作者删库后出现了一些同名库，安全性未知，因此本站托管的依然是旧版；
-   Qv2ray：Qv2ray是一个基于Qt框架开发的v2ray客户端，可通过插件支持SS、SSR、VMESS、VLESS、trojan等多种协议。

**Xray安卓客户端**：

-   V2rayNG：V2rayNG可以说是最跟随Xray步伐的[V2ray客户端](https://itlanyan.com/v2ray-clients-download/)了，Xray发布新版本后会在第一时间更新，推荐使用。

**Xray Mac客户端**：

-   Qv2ray：Qv2ray是一个基于Qt框架开发的跨平台[v2ray客户端](https://itlanyan.com/v2ray-clients-download/)，因此支持MacOS系统。实际上，自V2rayU作者删库不更新后，Qv2ray算得上Mac系统上支持VLESS协议的独苗，但可能会出现设置系统代理无效的bug。

**Xray苹果客户端：**

-   Shadowrocket/小火箭：小火箭目前是ios系统上更新最频繁的[V2ray客户端](https://itlanyan.com/v2ray-clients-download/)，价格也不贵，支持多种协议，推荐使用。

以上客户端均可以在 [V2ray客户端](https://itlanyan.com/v2ray-clients-download/) 下载，请参考其中的配置教程配置，本文不再赘述。

## 总结

不管V2ray和Xray今后发展如何，本人都真心感谢为这两个项目付出时间和贡献的开发人员，他们为自由获取互联网信息作出了重要贡献。

本文到此结束，欢迎批评指正！

## 参考

1.  [V2ray教程](https://itlanyan.com/v2ray-tutorial/)
2.  [V2ray高级技巧：流量伪装](https://itlanyan.com/v2ray-traffic-mask/)
3.  [V2ray的VLESS协议介绍和使用教程](https://itlanyan.com/introduce-v2ray-vless-protocol/)
4.  [VLESS协议的fallback参数介绍](https://itlanyan.com/vless-fallback-object/)

赞(59)