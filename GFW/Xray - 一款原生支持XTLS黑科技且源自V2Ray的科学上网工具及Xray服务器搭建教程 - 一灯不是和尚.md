Xray 和 Project X 诞生了，它天生不凡！通过本篇文章，[一灯不是和尚](https://iyideng.win/)主要介绍 Xray 和 Project X 项目的由来，以及与 V2ray 和 Project V 项目的关系，还有手动部署安装 Xray 服务端和配置 Xray 客户端的详细Xray使用教程。

-   [1、Xray 和 Project X 项目的由来](https://iyideng.win/black-technology/cgfw/xray-xtls-tutorial.html#1%E3%80%81Xray_%E5%92%8C_Project_X_%E9%A1%B9%E7%9B%AE%E7%9A%84%E7%94%B1%E6%9D%A5 "1、Xray 和 Project X 项目的由来")
-   [2、不得不说的Xray项目大佬@rprx](https://iyideng.win/black-technology/cgfw/xray-xtls-tutorial.html#2%E3%80%81%E4%B8%8D%E5%BE%97%E4%B8%8D%E8%AF%B4%E7%9A%84Xray%E9%A1%B9%E7%9B%AE%E5%A4%A7%E4%BD%ACrprx "2、不得不说的Xray项目大佬@rprx")
-   [3、Xray-Core 的版本发布历史](https://iyideng.win/black-technology/cgfw/xray-xtls-tutorial.html#3%E3%80%81Xray-Core_%E7%9A%84%E7%89%88%E6%9C%AC%E5%8F%91%E5%B8%83%E5%8E%86%E5%8F%B2 "3、Xray-Core 的版本发布历史")
-   [4、Xray和V2Ray的区别和关系](https://iyideng.win/black-technology/cgfw/xray-xtls-tutorial.html#4%E3%80%81Xray%E5%92%8CV2Ray%E7%9A%84%E5%8C%BA%E5%88%AB%E5%92%8C%E5%85%B3%E7%B3%BB "4、Xray和V2Ray的区别和关系")
-   [5、Xray服务器搭建和客户端配置使用教程](https://iyideng.win/black-technology/cgfw/xray-xtls-tutorial.html#5%E3%80%81Xray%E6%9C%8D%E5%8A%A1%E5%99%A8%E6%90%AD%E5%BB%BA%E5%92%8C%E5%AE%A2%E6%88%B7%E7%AB%AF%E9%85%8D%E7%BD%AE%E4%BD%BF%E7%94%A8%E6%95%99%E7%A8%8B "5、Xray服务器搭建和客户端配置使用教程")
    -   [（1）注册域名](https://iyideng.win/black-technology/cgfw/xray-xtls-tutorial.html#%EF%BC%881%EF%BC%89%E6%B3%A8%E5%86%8C%E5%9F%9F%E5%90%8D "（1）注册域名")
    -   [（2）安装Xray服务端](https://iyideng.win/black-technology/cgfw/xray-xtls-tutorial.html#%EF%BC%882%EF%BC%89%E5%AE%89%E8%A3%85Xray%E6%9C%8D%E5%8A%A1%E7%AB%AF "（2）安装Xray服务端")
    -   [（3）一键搭建Xray服务器（推荐）](https://iyideng.win/black-technology/cgfw/xray-xtls-tutorial.html#%EF%BC%883%EF%BC%89%E4%B8%80%E9%94%AE%E6%90%AD%E5%BB%BAXray%E6%9C%8D%E5%8A%A1%E5%99%A8%EF%BC%88%E6%8E%A8%E8%8D%90%EF%BC%89 "（3）一键搭建Xray服务器（推荐）")
    -   [（4）配置Xray客户端](https://iyideng.win/black-technology/cgfw/xray-xtls-tutorial.html#%EF%BC%884%EF%BC%89%E9%85%8D%E7%BD%AEXray%E5%AE%A2%E6%88%B7%E7%AB%AF "（4）配置Xray客户端")
-   [6、“Xray+XTLS”黑科技推荐](https://iyideng.win/black-technology/cgfw/xray-xtls-tutorial.html#6%E3%80%81XrayXTLS%E9%BB%91%E7%A7%91%E6%8A%80%E6%8E%A8%E8%8D%90 "6、“Xray+XTLS”黑科技推荐")

## **1、Xray 和 Project X 项目的由来**

我自2020年11月份就开始关注Xray了！Xray诞生事件的大致经过：由于Debian包的维护人员发现[XTLS库](https://github.com/XTLS/Go)不是BSD许可后，希望作者 [@rprx](https://github.com/rprx) 能修改LISENCE许可，这样方便打包。这个问题在社区引发了大家非常广泛的讨论，但rprx坚持认为目前LISENCE不是什么问题。但由于V2Ray一直遵循MIT协议，Project V 项目社区(V2fly社区)的维护团队发起投票，并最终认定XTLS不符合协议要求，于是在V2ray-core 4.33.0版移除了XTLS黑科技。随后， rprx和其支持者另起炉灶，很快就创建了 [Project X 项目](https://github.com/XTLS) 和其核心 [Xray](https://github.com/XTLS/Xray-core)，并以XTLS为核心协议陆续发布了 Xray-core 的多个版本，于是Xray诞生了。

也许你已经发现，Xray 和 Project X 项目与 V2Ray 和 Project V 项目几乎是一一对应的。

没错，就是如此。其实，Xray是V2Ray的超集，就是包含V2Ray所有的功能，并完全兼容V2Ray；而 Project X 就是 Project V 的超集，也是完全兼容。所以，我们像称呼V2Ray一样，以后也用Xray代指 Project X 项目及其相关技术。因此，我们可以把Xray看成V2Ray的增强版，以XTLS为核心协议。也许你跟我有一样的感觉，没有XTLS的VLESS好像缺少点什么，这个需要V2fly社区那些大佬们努力创造了，毕竟现在VMess协议已显颓势。我绝对相信XTLS不是唯一一个科学上网黑科技，也不会是最后一个。在科学上网技术领域，百花齐放总归是一件太大的好事，所以我更看好Xray独立发展的前景。

-   Xray官网：[https://xtls.github.io](https://xtls.github.io/)
-   Xray Github项目主页：[https://github.com/XTLS](https://github.com/XTLS)
-   Xray TG讨论组：[@projectXray](https://t.me/projectXray)；
-   Xray/Projcet X 官方文档：[https://xtls.github.io](https://xtls.github.io/)
-   Project X 项目官方频道：[@Project X Channel](https://t.me/projectXtls)

## **2、不得不说的Xray项目大佬@rprx**

为什么我这里还要重点介绍一下 [@rprx](https://github.com/rprx) 大佬呢？你先看大佬的这些简历就略知一二，只能顶礼膜拜，仰慕大神的技术实力。

-   rprx 是 VLESS 协议的设计者，而VLESS协议是rprx大神作为终极协议来打造的，并以达成“性能至上、可扩展性空前，目标是全场景终极协议”为最终目的；
-   rprx 也是 XTLS 的作者，是第一个将内外两条TLS连接并变成现实的第一人。@rprx 曾经说过，XTLS是划时代的革命性概念与技术，后来XTLS被众多粉丝追捧为科学上网第一黑科技，也是实至名归；
-   rprx 还是 Xray 和 Project X 项目的创始人。自创建 Xray 起，Xray-core 一直保持着超高的更新频率，这一定是令 @rprx 及其粉丝都非常兴奋的一件事情。

## **3、Xray-Core 的版本发布历史**

Xray-Core 自2020年11月份创立以来，仅历时一个多月就发布了七个版本。截至目前，Xray-Core已发布版本简介：

-   Xray-core 1.0.0版本：项目创建，提供完整的VLESS和XTLS支持，对V2ray-core性能优化，并将v2ray和v2ctl可执行文件合并为xray，功能得到增强，功能上成为V2ray-core的超集。
-   Xray-core 1.1.0和1.1.1版本：测试过渡版本，未提供详细发行说明；
-   Xray-core 1.1.2版本：引入了Linux Kernel Splice技术，适用于Android安卓、路由器等Linux环境。Splice技术减少了数据拷贝次数和内存占用，拥有更强的性能。需要说明的是，只适用于类Linux环境的客户端出栈，服务端入栈仍然是direct；
-   Xray-core 1.1.3版本：重构了透明代理的 REDIRECT 模式，使之同时支持 IPv4 和 IPv6，解决了V2ray和trojan遗留的问题；
-   Xray-core 1.1.4版本：优化内存占用，TLS更多选项配置，使服务端伪装站的TLS设置在[SSL Labs](https://www.ssllabs.com/ssltest/)能达到A+评分；
-   Xray-core 1.1.5版本：测试过渡版本，支持YAML等配置文件，支持OCSP Stapling等功能，官方安装脚本大更新。
-   Xray-core 1.2.0版本：Xray 是有史以来第一个不受限制的多协议平台，只需 Xray 即可解决问题，无需借力其它实现，支持各大主流协议！
-   Xray-core 1.2.1版本：[小小白白话文](https://xtls.github.io/document/level-0/)连载上线啦,老师呕心沥血之作, 手把手教你从什么都不会到熟练配置 Xray！大量的 UDP 相关修复，甚至可以在育碧的土豆服务器上玩彩虹六号！
-   Xray-core 1.2.2版本：回落分流又解锁了奇怪的新姿势! 回落中可以根据 SNI 分流啦！之前预告的 UUID 修改正式上线，日志现在看起来比上一次顺眼又更顺眼了一丢丢，远程 DOH 和其他的 DNS 模式一样学会了走路由分流。
-   Xray-core 1.2.3版本：对 SS 协议的支持又变强了, 支持单端口多用户！对 trojan 协议的支持又变强了，trojan 的回落也解锁 SNI 分流的新姿势啦！_(VLESS: 嘤嘤嘤)_UDP 奇奇怪怪的 BUG 被干掉了，一个字，“稳定”。嗅探可以排除你不想嗅探的域名，可以开启一些新玩法。
-   Xray-core 1.2.4版本：解决两个“连接至标准 Socks 服务端时可能出错”的历史遗留问题。
-   Xray-core 1.4.0版本：这次是个大更新！为链式代理引入了传输层支持；为 Dialer 引入了 Domain Strategy，解决奇妙的 DNS 问题；添加了 gRPC 传输方式，与更快一点的 Multi Mode；添加了 WebSocket Early-Data 功能，减少了 WebSocket 的延迟；添加了 FakeDNS；还修复了系列的问题，添加了各类功能，详情请见更新日志。
-   Xray-core 1.4.2版本：加入 Browser Dialer，用与改变 TLS 指纹与行为；加入 uTLS，用与改变 TLS Client Hello 的指纹；顺便修复了一大堆奇妙的问题，具体的内容见更新日志。
-   Xray-core 1.4.3版本：这是一个 xray 的阶段性维护版本。
-   Xray-core 1.4.5版本：功能增强和bug修复。
-   Xray-core 1.5.0版本：重构和功能增强，以及bug更新。
-   Xray-core 1.5.1版本：这是一个 xray 过渡时期的阶段性的维护版本。
-   Xray-core 1.5.2版本：grpc 传输模式增加 `initial_windows_size` 选项 可以解决被 Cloudflare 切断连接的问题 [文档](https://github.com/XTLS/Xray-docs-next/blob/5ece68eec9d730047396b76c8463e961da37a236/docs/config/transports/grpc.md) [@hmol233](https://github.com/hmol233) [@xqzr](https://github.com/xqzr)；修复了 `xChaCha20-IETF-Poly1305` 加密的解析以及 nonce [@xqzr](https://github.com/xqzr)。
-   Xray-core 1.5.3版本：Quic 底层传输相关，Quic增强和bug修复，更新。

截止到目前，Xray-core一直在维持高频更新，社区氛围非常好。让我们拭目以待一个更加强大的Xray生态！

Xray完全模仿V2Ray的模式，所以只要我们搞清楚了V2Ray，那么Xray就很好理解了。一灯不是和尚先来跟大家说一下V2Ray、Projcet V 和 V2fly社区三者之间的关系。

V2Ray的由来：V2Ray 是 继 Shadowsocks 作者[@clowwindy](https://github.com/clowwindy)被请去喝茶之后，V2Ray 项目组为表示抗议而开发的，后破娃酱[@breakwa11](https://github.com/breakwa11)也被请喝茶，V2Ray项目原作者隐匿，项目长期停滞不前。于是，原V2Ray社区成员组建了V2fly社区，并继续V2Ray的维护开发，同时 Project V 项目由此诞生。

V2Ray 和 Project V 项目：V2Ray 是构建特定网络环境工具的项目 Project V 下的最核心的工具之一（因为其核心不只有V2Ray），而 Project V 其实是一个工具集，它可以帮助你打造专属的基础通信网络。V2Ray内核模块用于实际的网络交互、路由等针对网络数据的处理，而外围的用户界面程序提供了方便直接的操作流程，这从根本上解决了V2Ray搭建和使用对于新手小白特别不友好的问题。目前，Project V 项目支持 Socks5、Shadowsocks、VMess、VLESS 和 Trojan 等协议，其中只有VMess和VLESS协议为V2fly社区原创，其它均为整合的已经成熟的科学上网协议。

从时间上来说，先有 V2Ray 才有 Project V，在 V2Ray 得到普遍认可的时候才开发的 Project V 框架，然后才有了V2fly社区。简单地说，V2Ray 是 Projcet V 项目的一个核心模块，V2fly社区是维护 Project V 项目的核心团队。一直以来，由于 V2Ray 名气太大，所以现在大家都习惯称 Project V 为 V2Ray，也就是说我们通常所说的 V2Ray 实际上是指以 V2Ray 为核心的 Project V 项目。

-   V2ray官网：[https://v2ray.com/](https://v2ray.com/)
-   Github项目主页：[https://github.com/v2ray](https://github.com/v2ray)
-   TG讨论组：[@projectv2ray](https://t.me/projectv2ray)；

说到这里，我相信大多数小伙伴已经明白了。

Xray：与V2Ray完全类同，Xray 是 Project X 项目的核心模块。因为Xray和XTLS黑科技的作者rprx曾经是V2fly社区的重要成员，所以Xray直接Fork全部V2Ray的功能，然后进行性能优化，并增加了新功能，使Xray在功能上成为了V2Ray的超集，且完全兼容V2Ray。

简而言之，Xray是V2Ray的项目分支，Xray是V2Ray的超集，就跟Trojan-Go和Trojan-GFW的关系类似，而且Xray性能更好、速度更快，更新迭代也更频繁。由于自V2ray-core 4.33.0 版本起，删除了XTLS黑科技，但仍然支持VLESS，所以是否原生支持XTLS是Xray和V2Ray最大的区别之一。

最近，Xray真的是非常火，尤其XTLS黑科技让其成为在科学上网技术领域最耀眼的明星，星光熠熠，一时间簇拥者不可胜数。鉴于此，如果你非常喜欢XTLS黑科技的话，你只能使用 V2ray-Core 4.29.0 ～ 4.32.1版，或直接使用 Xray-core 就行了。当然，我是非常建议您直接使用 Xray-core 就OK了。

在Xray跟V2Ray分家之后，就一直有人说，“Xray和V2Ray有可能再次合并”。我认为可能性几乎为零，因为开弓没有回头箭，而且真没有重新回去的必要，自立门户对于支持Xray的社区开发者也是一件非常激动人心的事情，以后会在科学上网领域“青史留名”的，这是一种至高的荣誉。我相信任何一个社区开发者都喜欢这种成就感，因为这是自己参与并支持过的正确的道路，这也是一种持续创作的动力，是不可能再回头了。

## **5、Xray服务器搭建和客户端配置使用教程**

### **（1）注册域名**

如果你使用“VLESS+TCP+XTLS”模式的话，肯定需要域名伪装成网站。目前，注册域名有两种途径：注册国外免费域名和付费购买域名。

**1）注册免费域名**

注册免费域名请参考文章 [最新的国外免费顶级域名网站Freenom注册免费域名教程与Cloudflare托管解析的方法](https://iyideng.win/welfare/freenom-free-domain-register.html)

**2）购买付费域名**

购买付费域名推荐 [NameSilo](https://ad.bgfw.cc/NameSilo)(推荐) 或 [Namecheap](https://ad.bgfw.cc/Namecheap)，大部分域名一般不到4美元/第1年，像 .xray 和 .top 这样顶级域名还不到1美元/年，非常划算，第2年重新注册一个新的就可以了。

如果你打算在 NameSilo 注册域名的话，请参考文章 [NameSilo – 美国知名域名注册商 | 仅年付0.99/1.99美元 | 域名购买与账户注册图文教程](https://iyideng.win/note/namesilo-domain-registrar.html)

**3）解析域名**

注册好域名之后，请把域名解析到你要搭建V2Ray服务器所用VPS对应的IP地址，一般5分钟之内就可以生效，最迟72小时生效。我一直推荐大家使用 Cloudflare 管理域名，解析速度快，基本秒生效，非常方便快捷，而且 Cloudflare 的免费 CDN 很好用，而且搭建自用SS/SSR/V2Ray/Trojan机场经常会用到。

关于互联网域名注册、购买与添加DNS域名解析记录的详细操作教程，请参考文章 [互联网域名注册、服务购买与添加域名解析记录及更改DNS服务器的详细图文教程](https://iyideng.win/note/domain-name-registration-buy-resolution-and-change-dns.html)

通过以上对Xray的介绍，你应该已经知晓Xray和V2Ray服务器端部署和客户端配置操作基本一致，那就是Xray服务器搭建也分为安装Xray服务端和配置Xray客户端两部分。

### **（2）安装Xray服务端**

你需要有一定的Linux系统操作知识，至少能使用vim/nano等编辑器，否则我还是建议新手使用脚本[一键搭建Xray服务器](https://iyideng.win/black-technology/cgfw/xray-xtls-one-click-script-building-and-using-tutorial.html)更合适。Xray可执行文件位于 /usr/local/bin 目录，Xray配置文件在 /etc/local/etc/xray目录。由于官方脚本安装的配置文件内容为空，我们可参考 [Xray-examples](https://github.com/XTLS/Xray-examples) 中提供的模板编辑配置文件。如使用“VLESS+TCP+XTLS”模式的配置文件，代码如下：

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

因为XTLS需要域名启用SSL证书，所以我们必须先申请一个域名并成功开启HTTPS。

当以上配置完成后，我们可使用以下命令管理Xray服务，具体执行命令如下：

```
systemctl start xray #运行xray
systemctl stop xray #停止xray
systemctl restart xray #重启
journalctl -xe --no-pager -u xray #查看运行日志。
```

最后，请一定要在防火墙方形Xray服务的端口，否则是无法成功开启Xray服务的；另外，部分运营商需要你在网页端的安全组放行端口，如阿里云、腾讯云、微软云和亚马逊云等云服务商。

### **（3）一键搭建Xray服务器（推荐）**

[\[一键安装Xray服务器教程\]使用国外VPS搭建 Xray/VLESS+XTLS 模式服务器及配置Xray客户端实现科学上网](https://iyideng.win/black-technology/cgfw/xray-xtls-one-click-script-building-and-using-tutorial.html)

### **（4）配置Xray客户端**

鉴于星光夺目的Xray如此优秀，致使很多客户端都快速跟进对Xray的支持。目前，支持Xray的客户端有：

**1）Xray Windows客户端**

-   [V2rayN](https://github.com/2dust/v2rayN)：自 V2RayN 3.28 版本起支持Xray，你只需要下载[Xray-core](https://github.com/XTLS/Xray-core/releases)，然后将解压的文件放到V2rayN-Core文件夹下即可。另外，自 V2rayN 4.0版开启移除PAC列表，改用跟Qv2ray一样使用路由规则，会给习惯了使用PAC列表模式的用户带来不适应，完全适应需要一个过程。
-   [Qv2ray](https://iyideng.win/tools/qv2ray.html)：一个基于Qt框架开发的V2Ray客户端，可通过插件支持SS/SSR/VMESS/VLESS/Trojan/Trojan-Go/Xray等多种代理协议。
-   [winXray](https://iyideng.win/tools/winxray.html)：winXray 是Windows系统上同时支持 Shadowsocks/V2Ray/Trojan/Xray 等代理的通用客户端，可自动检测并连接访问速度最快的代理服务器。原作者已经删库后，同时Github上面可以搜索到的winXray客户端的安全性未知。

**2）Xray Mac客户端**

[Qv2ray](https://iyideng.win/tools/qv2ray.html)：一个基于Qt框架开发的跨平台v2ray客户端，因此支持Windows/MacOS系统。目前，在Mac系统上使用VLESS协议，你是没得选择的，且用且珍惜。

**3）Xray Android安卓客户端**

-   [V2rayNG](https://github.com/2dust/v2rayNG)：更新最及时的V2Ray客户端，一直跟随Xray版本快速更新，非常及时。
-   [Kitsunebi](https://github.com/rurirei/Kitsunebi/tree/release_xtls)：一款非常知名的Android安卓客户端，支持VMess/VLESS/Xray协议。

**4）Xray苹果iOS客户端**

小火箭(Shadowrocket)：Shadowrocket是iOS系统上更新最勤快的V2ray客户端，也是最知名的iOS平台科学上网代理客户端，而且价格也不贵，并支持非常多的协议，几乎主流的协议都支持，包括新出的知名科学上网代理协议都很友好。

**5）Xray OpenWrt路由器客户端**

-   [PassWall](https://github.com/xiaorouji/openwrt-passwall) – 一款功能非常强大的软路由科学上网客户端，支持SS/V2Ray/Trojan/Xray等代理。
-   [Hello World](https://github.com/jerrykuku/luci-app-vssr) – 是一个以用户最佳主观体验为导向的路由器插件，支持SS/SSR/VMess/VLESS/Trojan/Xray等多种主流代理协议和多种自定义视频分流服务，拥有精美的操作界面，并配上直观的节点信息。
-   [ShadowSocksR Plus+](https://github.com/fw876/helloworld) – 一款功能非常强大的软路由科学上网客户端，支持SS/V2Ray/Trojan/Xray等代理上网。

## **6、“Xray+XTLS”黑科技推荐**

俗话说得好，“一枝独放不是春，百花齐放春满园”，Xray和V2Ray分家拓宽了科学上网技术研究的方向，会激发更多的创造力，一定会让科学上网技术枝繁叶茂、硕果累累。我相信Xray和XTLS黑科技会带来更多的不可思议和无与伦比，这么牛逼的技术，你确定不来试一试？

本文由更新于2022年12月26日；如果您有什么意见或建议，请在文章下面评论区留言反馈。