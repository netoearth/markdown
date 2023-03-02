 [科学上网](https://itlanyan.com/category/for-free-internet/)2022年12月9日

本文目录

-   [VLESS协议介绍](https://itlanyan.com/introduce-v2ray-vless-protocol/#bnp_i_1)
-   [XTLS介绍](https://itlanyan.com/introduce-v2ray-vless-protocol/#bnp_i_2)
-   [VLESS配置组合](https://itlanyan.com/introduce-v2ray-vless-protocol/#bnp_i_3)
-   [VLESS + XTLS服务器配置教程](https://itlanyan.com/introduce-v2ray-vless-protocol/#bnp_i_4)
-   [VLESS客户端配置](https://itlanyan.com/introduce-v2ray-vless-protocol/#bnp_i_5)
-   [总结](https://itlanyan.com/introduce-v2ray-vless-protocol/#bnp_i_6)
-   [参考](https://itlanyan.com/introduce-v2ray-vless-protocol/#bnp_i_7)

> 欢迎加入tg群交流：[@tlanyantg](https://t.me/tlanyantg)

[V2ray](https://v2ray.com/) 近期因新增VLESS协议而备受关注。V2ray官方对VLESS协议的定义是“性能至上、可扩展性空前，目标是全场景终极协议”。本文是 [V2ray教程](https://itlanyan.com/v2ray-tutorial/) 和 [V2ray高级技巧：流量伪装](https://itlanyan.com/v2ray-traffic-mask/) 的续篇，给出V2ray的VLESS协议介绍和使用教程。下载V2ray客户端请到 [V2ray客户端](https://itlanyan.com/v2ray-clients-download/) 页面。

## VLESS协议介绍

VLESS是一种**无状态**的轻量级数据传输协议，被定义为下一代V2ray数据传输协议。作者对该协议的愿景是“**可扩展性空前，适合随意组合、全场景广泛使用，符合很多人的设想、几乎所有人的需求，足以成为 v2ray 的下一代主要协议，乃至整个 XX 界的终极协议。**”，由此可见VLESS协议的强大。

> VLESS命名源自“less is more”，写法与VMESS近似，个人认为命名非常有水准

与VMESS协议相同，VLESS使用UUID进行身份验证，配置分入栈和出栈两部分，可用在客户端和服务端。

VLESS和VMESS区别如下：

1.  VLESS协议不依赖于系统时间，不使用alterId。一些人的V2ray用不了，最后找出原因是电脑时间和服务器只相差两分钟，简直要让人抓狂；VLESS协议去掉了时间要求，双手举赞；
2.  VLESS协议不带加密，用于[科学上网](https://itlanyan.com/category/for-free-internet/)时要配合TLS等加密手段；
3.  VLESS协议支持分流和回落，比Nginx分流转发更简洁、高效和安全；
4.  使用TLS的情况下，VLESS协议比VMESS速度更快，性能更好，因为VLESS不会对数据进行加解密；
5.  V2ray官方对VLESS的期望更高，约束也更严格。例如要求客户端统一使用VLESS标识，而不是Vless、vless等名称；VLESS分享链接标准将由官方统一制定（尚未出炉）；
6.  VLESS协议的加密更灵活，不像VMESS一样高度耦合（仅对开发者有用）

对于普通用户来说，VLESS协议的主要优势是：1. 不需要客户端和服务器时间一致； 2. VLESS协议不自带加密，使用TLS的情况下性能比VMESS更好。

TLS将是接下来三到五年内躲避封锁的主流方式，例如[V2ray伪装](https://itlanyan.com/v2ray-traffic-mask/)、[trojan](https://itlanyan.com/trojan-tutorial/)、[SSRoT](https://github.com/ShadowsocksR-Live/shadowsocksr-native)都是基于TLS。V2ray之前因性能不好而被一些人唾弃，VLESS协议的出现，让V2ray的便利性和性能达到 [trojan](https://itlanyan.com/trojan-tutorial/) 的高度（使用TLS的情况下）。

仅有VLESS还不够，套了一层TLS，V2ray的性能依然不如[SS](https://itlanyan.com/shadowsock-clients/)、[SSR](https://itlanyan.com/shadowsockr-shadowsocksr-shadowsocksrr-clients/)。但随VLESS协议而来的另一个大惊喜：XTLS。（个人认为）这是划时代的概念和技术，将V2ray的性能提升到、甚至超越SS/SSR的水平。

## XTLS介绍

[XTLS官方库](https://github.com/XTLS/Go) 的介绍仅有一句话：THE FUTURE。这个十个字符足以透露出XTLS的牛逼和霸气。

[V2fly官网](https://www.v2fly.org/)（V2fly社区是V2ray技术的主要推动力量） 称 [XTLS为黑科技](https://www.v2fly.org/config/protocols/vless.html#xtls-%E9%BB%91%E7%A7%91%E6%8A%80)，VLESS协议作者的形容是 [划时代的革命性概念和技术：XTLS](https://github.com/rprx/v2ray-vless/releases/tag/xtls)。

the future、黑科技、划时代、革命性，无论哪个词，都足以形容XTLS的牛逼和独到之处。

**XTLS的原理是**：使用TLS代理时，https数据其实经过了两层TLS：外层是代理的TLS，内层是https的TLS。XTLS无缝拼接了内外两条货真价实的TLS，使得代理几乎无需再对https流量进行数据加解密，只起到流量中转的作用，极大的提高了性能。

VLESS + XTLS的组合可以理解为是增强版 ECH，即多支持身份认证、代理转发、明文加密、UDP over TCP 等。但从其原理可知，VLESS + XTLS对http流量是没有多大优势的。好消息是，目前超过90%的流量都是https的，因此VLESS + XTLS能极大的提升性能，无愧于上面的评价。

> 需要说明的是，XTLS是科学上网的future，不是TLS发展的future

所以，想 [科学上网](https://itlanyan.com/category/for-free-internet/) 速度更快、手机路由器耗电更少，请使用VLESS+XTLS。如果客户端或者服务端不支持XTLS，TLS的情况下简单将VMESS换成VLESS也能带来性能提升。

> 2020.11.20更新：因为license问题，XTLS已经在V2ray-core 4.33.0被移除，原因和吃瓜链接参考：[https://github.com/XTLS/Go/issues/9](https://github.com/XTLS/Go/issues/9)
> 
> XTLS作者创建了Xray项目，完整支持VLESS协议和XTLS，详情请参考：[Xray教程](https://itlanyan.com/xray-tutorial/)

## VLESS配置组合

VLESS协议本身不自带加密，用于翻墙时不能单独使用。由于XTLS的引入，目前VLESS协议有如下玩法：

-   VLESS + TCP + TLS
-   VLESS + TCP +TLS +  WS
-   VLESS + TCP + XTLS
-   VLESS + HTTP2 + h2c

> VLESS也可以配合mKCP使用

如果客户端和服务器支持，请优先使用VLESS+TCP+XTLS的配置，能极大的提升性能（有过CDN需求请选择WS版）。

## VLESS + XTLS服务器配置教程

本文以VLESS+XTLS为例，介绍VLESS+XTLS配置和使用教程。

> 新手建议使用 [V2ray多合一脚本，支持VMESS+websocket+TLS+Nginx、VLESS+TCP+XTLS、VLESS+TCP+TLS等组合](https://itlanyan.com/go.php?key=v2ray-new-script)

### 准备工作

1.  准备一台境外的VPS，购买可参考 [一些VPS商家整理](https://itlanyan.com/vps-merchant-collection/);
2.  准备一个域名，不需要备案，国内和国外买的都可以。域名购买可参考：[Namesilo域名注册和使用教程](https://itlanyan.com/namesilo-domain-tutorial/) 或从 [适合国人的域名注册商推荐](https://itlanyan.com/domain-register-for-mainland/) 选购；
3.  将域名的某个子域名DNS解析到境外VPS；
4.  SSH连接到境外服务器，windows请参考：[Bitvise连接Linux服务器教程](https://itlanyan.com/go.php?key=bitvise-tutorial)，mac系统请参考：[Mac电脑连接Linux教程](https://itlanyan.com/go.php?key=mac-ssh-tutorial)；
5.  为域名申请一个证书，请参考：[使用acme.sh签发证书](https://itlanyan.com/use-acme-sh-get-free-cert/) 或 [从阿里云获取免费SSL证书](https://itlanyan.com/get-free-ssl-certificates-from-aliyun/)；
6.  有基本linux技巧，能使用vim/nano等编辑器。

### 安装最新版V2ray-Core

目前V2fly分支的最新代码尚未合并到V2ray官方库中，因此我们需要V2fly提供的 [fhs-install-v2ray](https://github.com/v2fly/fhs-install-v2ray) 脚本安装最新版代码：

```
bash <(curl -L https://raw.githubusercontent.com/v2fly/fhs-install-v2ray/master/install-release.sh)
```

> **2020.11.23更新**：V2ray-Core最新版v4.33.0已经将XTLS移除，如果你希望用XTLS功能，安装命令请更改为：
> 
> `bash <(curl -L https://raw.githubusercontent.com/v2fly/fhs-install-v2ray/master/install-release.sh) --version 4.32.1`

### 服务器配置VLESS+XTLS协议

用vim、nano等编辑器编辑 `/usr/local/etc/v2ray/config.json`文件，填入以下信息：

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
                        "id": "", // 填写UUID，可以使用/usr/bin/v2ray/v2ctl uuid生成
                        "flow": "xtls-rprx-origin", # V2ray v4.32.1以上版本建议为xtls-rprx-direct，能极大提升性能
                        "level": 0
                    }
                ],
                "decryption": "none",
                "fallbacks": [
                    {
                        "dest": 80 // 回落配置，可以直接转到其他网站，例如"www.baidu.com:443"
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

然后设置开机自启并启动v2ray服务：`systemctl enable v2ray; systemctl start v2ray`，启动后使用 `ss -nltp | grep v2ray` 应该能看到类似如下输出：

```
LISTEN 0 128 *:443 *:* users:(("v2ray",pid=12345,fd=3))
```

> VLESS协议的fallback参数说明请参考：[VLESS协议的fallback参数介绍](https://itlanyan.com/vless-fallback-object/)
> 
> **2020.11.08更新**：V2ray v4.32.1中的[XTLS Direct 模式引入了ReadV增强](https://www.v2fly.org/config/protocols/vless.html#xtls-%E9%BB%91%E7%A7%91%E6%8A%80)(即配置文件中flow的值改为xtls-rprx-direct)，性能与VLESS无加密裸奔持平，是其他组合的数倍，能极大提高设备续航能力，推荐使用（目前客户端只有最新版V2rayNG支持，V2rayN可通过更新V2ray-Core支持）。

### 服务端注意事项

1.  VLESS + TLS、VLESS+TLS+WS及其他方案配置请参考 [v2ray-examples](https://github.com/v2fly/v2ray-examples)；
2.  json文件不支持注释，上面配置文件中的#号和后面内容都应该删去;
3.  如果使用了80回落端口，记得安装Nginx或其他web软件：`yum install -y epel-release; yum install -y nginx`;
4.  记得放行防火墙：`firewall-cmd --permanent --add-service=https; firewall-cmd --permanent --add-service=http; firewall-cmd --reload`;
5.  如果无法启动，请查看是否开启了SELINUX：`getenforce`，输出为Enforcing请禁用：`setenforce 0`，然后重启v2ray
6.  `journalctl -xe -u v2ray --no-pager` 可查看V2ray的系统日志
7.  建议添加V2ray运行日志输出，方便出问题时排查

## VLESS客户端配置

服务端配置好后，接下来介绍客户端配置VLESS。

### 支持VLESS协议的客户端

VLESS协议在V2ray-Core v4.27.0版本中引入，XTLS在V2ray-Core v4.29.0引入。因此如果使用VLESS协议，请确保客户端的内核版本至少是v4.27.0，使用XTLS功能则保证内核至少为4.29.0版本。

目前各平台有如下客户端支持VLESS：

-   **支持VLESS协议的Windows客户端**：
    -   V2rayN：3.21版本开始支持VLESS协议，3.24版本支持XTLS。目前最新版是3.26，完美支持VLESS + XTLS组合，并且支持trojan协议；
    -   Qv2ray：2.6.3版本开始支持VLESS协议，2.7.0 alpha1版本支持XTLS。目前最新版是2.7.0 alpha1，完美支持VLESS + XTLS组合；
-   **支持VLESS协议的安卓客户端**：
    -   V2rayNG：1.3.0版本支持VLESS协议，1.4.4版本支持XTLS。目前最新版1.4.8，完美支持VLESS + XTLS组合，还支持trojan协议。**注意**：V2rayNG自1.4.5版本开始不提供全架构的客户端，如果本站版本无法安装和使用，请从官网下载对应平台版本。
-   **支持VLESS协议的Mac客户端**：
    -   Qv2ray：2.6.3版本开始支持VLESS协议，2.7.0 alpha1版本支持XTLS。目前最新版是2.7.0 alpha1，完美支持VLESS + XTLS组合；
    -   V2rayU：3.0预览版支持VLESS+XTLS和trojan
-   **支持VLESS协议的iOS客户端**：
    -   Shadowrocket：2.1.60版本支持VLESS协议，~目前不支持XTLS~（**2020.11.25更新**：2.1.67版本已经支持XTLS，flow默认为xtls-rprx-direct，无法选择）
-   **支持VLESS协议的Linux客户端**：
    -   Qv2ray：2.6.3版本开始支持VLESS协议，2.7.0 alpha1版本支持XTLS。目前最新版是2.7.0 alpha1，完美支持VLESS + XTLS组合；
    -   V2rayA：V2rayA是一个依赖于V2ray的UI工具，因此需要自行安装V2ray。V2rayA自1.0.0版本支持VLESS，可通过自行编辑配置文件支持XTLS；

上面提到的V2ray客户端均可到 [V2ray客户端](https://itlanyan.com/v2ray-clients-download/) 页面下载。比较遗憾的是颜值非常高的Clash系列目前都不支持VLESS，iOS的小火箭尚未发布支持XTLS的版本。

### VLESS客户端配置教程

本文以Windows平台的V2rayN软件为例，介绍VLESS+XTLS的客户端配置教程。

1\. 从 [V2ray客户端](https://itlanyan.com/v2ray-clients-download/) 下载客户端，解压文件；

2\. 进入V2rayN-Core文件夹，双击V2rayN.exe启动；

3\. 屏幕右下角托盘找到V2rayN图标，双击打开面板菜单。点击上方的“服务器”-》“添加\[VLESS\]”服务器：

[![V2rayN添加VLESS服务器](https://itlanyan.com/wp-content/uploads/2020/10/V2rayN%E6%B7%BB%E5%8A%A0VLESS%E6%9C%8D%E5%8A%A1%E5%99%A8.jpg)](https://itlanyan.com/introduce-v2ray-vless-protocol/v2rayn%e6%b7%bb%e5%8a%a0vless%e6%9c%8d%e5%8a%a1%e5%99%a8/)

V2rayN添加VLESS服务器

4\. 配置VLESS+XTLS信息。流控与服务端配置一致(建议服务端和客户端都使用“xtls-rprx-direct”)，加密必须填“none”，别名可以自己随便填。协议传输选“tcp”，在伪装域名里输入你的域名，底层传输选“xtls”，跳过证书验证选true和false都可以（能保证证书总是有效就选false，否则选true）。配置好后点击“确定”保存；

[![V2rayN配置VLESS+XTLS](https://itlanyan.com/wp-content/uploads/2020/10/V2rayN%E9%85%8D%E7%BD%AEVLESSXTLS.jpg)](https://itlanyan.com/introduce-v2ray-vless-protocol/v2rayn%e9%85%8d%e7%bd%aevlessxtls/)

V2rayN配置VLESS+XTLS

5\. 点击“参数设置”，Http代理选择全局模式或者PAC模式，建议为PAC模式：

[![V2rayN开启PAC模式](https://itlanyan.com/wp-content/uploads/2020/10/V2rayN%E5%BC%80%E5%90%AFPAC%E6%A8%A1%E5%BC%8F.jpg)](https://itlanyan.com/introduce-v2ray-vless-protocol/v2rayn%e5%bc%80%e5%90%afpac%e6%a8%a1%e5%bc%8f/)

V2rayN开启PAC模式

接下来，就可以愉快的上Google、Youtube等网站了。

## 总结

VLESS+XTLS，可以说是目前最强悍的[科学上网](https://itlanyan.com/category/for-free-internet)组合，强烈建议使用。如果客户端不支持XTLS，在使用TLS的情况下，也建议切换到VLESS协议。

## 参考

1.  [V2ray教程](https://itlanyan.com/trojan-tutorial/)
2.  [V2ray高级技巧：流量伪装](https://itlanyan.com/v2ray-traffic-mask/)
3.  [VLESS协议](https://www.v2fly.org/config/protocols/vless.html)

赞(40)