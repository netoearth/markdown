自VLESS协议和XTLS黑科技横空出世后，我曾经自以为这就是V2Ray的终极协议，不需要再折腾了。然而戏剧性的是，自2020年11月份因V2fly社区成员对LISENCE许可问题与[@rprx](https://github.com/rprx)大佬产生分歧，  最终导致 rprx大佬及其支持者自立门户，并创建 Xray 和 Project X 项目，跟 V2Ray 和 Project V 项目分家了。

咱先暂且不管这些事，今天[一灯不是和尚](https://iyideng.net/)带小伙伴们一键搭建Xray服务器并配置Xray客户端实现科学上网以及详细的Xray使用教程。

-   [1、准备工具](https://iyideng.net/black-technology/cgfw/xray-xtls-one-click-script-building-and-using-tutorial.html#1%E3%80%81%E5%87%86%E5%A4%87%E5%B7%A5%E5%85%B7 "1、准备工具")
-   [2、注册域名和购买非中国大陆地区的VPS](https://iyideng.net/black-technology/cgfw/xray-xtls-one-click-script-building-and-using-tutorial.html#2%E3%80%81%E6%B3%A8%E5%86%8C%E5%9F%9F%E5%90%8D%E5%92%8C%E8%B4%AD%E4%B9%B0%E9%9D%9E%E4%B8%AD%E5%9B%BD%E5%A4%A7%E9%99%86%E5%9C%B0%E5%8C%BA%E7%9A%84VPS "2、注册域名和购买非中国大陆地区的VPS")
    -   [（1）注册域名](https://iyideng.net/black-technology/cgfw/xray-xtls-one-click-script-building-and-using-tutorial.html#%EF%BC%881%EF%BC%89%E6%B3%A8%E5%86%8C%E5%9F%9F%E5%90%8D "（1）注册域名")
    -   [（2）购买非中国大陆地区的VPS](https://iyideng.net/black-technology/cgfw/xray-xtls-one-click-script-building-and-using-tutorial.html#%EF%BC%882%EF%BC%89%E8%B4%AD%E4%B9%B0%E9%9D%9E%E4%B8%AD%E5%9B%BD%E5%A4%A7%E9%99%86%E5%9C%B0%E5%8C%BA%E7%9A%84VPS "（2）购买非中国大陆地区的VPS")
-   [3、SSH远程连接并管理 Vultr VPS](https://iyideng.net/black-technology/cgfw/xray-xtls-one-click-script-building-and-using-tutorial.html#3%E3%80%81SSH%E8%BF%9C%E7%A8%8B%E8%BF%9E%E6%8E%A5%E5%B9%B6%E7%AE%A1%E7%90%86_Vultr_VPS "3、SSH远程连接并管理 Vultr VPS")
-   [4、执行Xray服务器一键搭建脚本](https://iyideng.net/black-technology/cgfw/xray-xtls-one-click-script-building-and-using-tutorial.html#4%E3%80%81%E6%89%A7%E8%A1%8CXray%E6%9C%8D%E5%8A%A1%E5%99%A8%E4%B8%80%E9%94%AE%E6%90%AD%E5%BB%BA%E8%84%9A%E6%9C%AC "4、执行Xray服务器一键搭建脚本")
    -   [（1）Xray一键脚本升级版（Atrandys作品）](https://iyideng.net/black-technology/cgfw/xray-xtls-one-click-script-building-and-using-tutorial.html#%EF%BC%881%EF%BC%89Xray%E4%B8%80%E9%94%AE%E8%84%9A%E6%9C%AC%E5%8D%87%E7%BA%A7%E7%89%88%EF%BC%88Atrandys%E4%BD%9C%E5%93%81%EF%BC%89 "（1）Xray一键脚本升级版（Atrandys作品）")
    -   [（2）Xray七合一共存脚本（mack-a作品，推荐）](https://iyideng.net/black-technology/cgfw/xray-xtls-one-click-script-building-and-using-tutorial.html#%EF%BC%882%EF%BC%89Xray%E4%B8%83%E5%90%88%E4%B8%80%E5%85%B1%E5%AD%98%E8%84%9A%E6%9C%AC%EF%BC%88mack-a%E4%BD%9C%E5%93%81%EF%BC%8C%E6%8E%A8%E8%8D%90%EF%BC%89 "（2）Xray七合一共存脚本（mack-a作品，推荐）")
    -   [（3）multi-v2ray 多用户管理脚本（Jrohy作品）](https://iyideng.net/black-technology/cgfw/xray-xtls-one-click-script-building-and-using-tutorial.html#%EF%BC%883%EF%BC%89multi-v2ray_%E5%A4%9A%E7%94%A8%E6%88%B7%E7%AE%A1%E7%90%86%E8%84%9A%E6%9C%AC%EF%BC%88Jrohy%E4%BD%9C%E5%93%81%EF%BC%89 "（3）multi-v2ray 多用户管理脚本（Jrohy作品）")
-   [5、一键安装并开启BBR加速](https://iyideng.net/black-technology/cgfw/xray-xtls-one-click-script-building-and-using-tutorial.html#5%E3%80%81%E4%B8%80%E9%94%AE%E5%AE%89%E8%A3%85%E5%B9%B6%E5%BC%80%E5%90%AFBBR%E5%8A%A0%E9%80%9F "5、一键安装并开启BBR加速")
-   [6、Xray客户端下载与配置使用教程](https://iyideng.net/black-technology/cgfw/xray-xtls-one-click-script-building-and-using-tutorial.html#6%E3%80%81Xray%E5%AE%A2%E6%88%B7%E7%AB%AF%E4%B8%8B%E8%BD%BD%E4%B8%8E%E9%85%8D%E7%BD%AE%E4%BD%BF%E7%94%A8%E6%95%99%E7%A8%8B "6、Xray客户端下载与配置使用教程")
-   [7、自建vps和买机场哪个好](https://iyideng.net/black-technology/cgfw/xray-xtls-one-click-script-building-and-using-tutorial.html#7%E3%80%81%E8%87%AA%E5%BB%BAvps%E5%92%8C%E4%B9%B0%E6%9C%BA%E5%9C%BA%E5%93%AA%E4%B8%AA%E5%A5%BD "7、自建vps和买机场哪个好")
-   [8、优质Xray机场推荐](https://iyideng.net/black-technology/cgfw/xray-xtls-one-click-script-building-and-using-tutorial.html#8%E3%80%81%E4%BC%98%E8%B4%A8Xray%E6%9C%BA%E5%9C%BA%E6%8E%A8%E8%8D%90 "8、优质Xray机场推荐")

## 1、准备工具

俗话说得好，“工欲善其事，必先利其器。”在一键搭建Xray服务器之前，咱们需要先准备一些工具：

-   一台非中国大陆地区的VPS，且IP可以在中国大陆正常访问；
-   一个域名，并成功解析到准备好的VPS的IP；
-   SSH远程连接软件，如 Xshell、Putty 等；
-   所用平台的 Xray 客户端。

### （1）注册域名

如果你使用“Vmess+TLS+Web+CDN”模式的话，就需要用域名伪装成网站，否则不需要域名。注册域名有两种途径：注册国外免费域名和付费购买域名。

1）注册免费域名

注册免费域名请参考文章 [2023年最新的国外免费顶级域名网站Freenom注册免费域名教程与Cloudflare托管解析的方法](https://iyideng.net/welfare/freenom-free-domain-register.html)

2）购买付费域名

购买付费域名推荐 [NameSilo](https://ad.bgfw.cc/NameSilo)(推荐) 或 [Namecheap](https://ad.bgfw.cc/Namecheap)，大部分域名一般不到4美元/第1年，像 .xray 和 .top 这样顶级域名还不到1美元/年，非常划算，第2年重新注册一个新的就可以了。

如果你打算在 NameSilo 注册域名的话，请参考文章 [NameSilo – 美国知名域名注册商 | 仅年付0.99/1.99美元 | 域名购买与账户注册图文教程](https://iyideng.net/note/namesilo-domain-registrar.html)

3）解析域名

注册好域名之后，请把域名解析到你要搭建V2Ray服务器所用VPS对应的IP地址，一般5分钟之内就可以生效，最迟72小时生效。我一直推荐大家使用 Cloudflare 管理域名，解析速度快，基本秒生效，非常方便快捷，而且 Cloudflare 的免费 CDN 很好用，而且搭建自用SS/SSR/V2Ray/Trojan机场经常会用到。

关于互联网域名注册、购买与添加DNS域名解析记录的详细操作教程，请参考文章 [互联网域名注册、服务购买与添加域名解析记录及更改DNS服务器的详细图文教程](https://iyideng.net/note/domain-name-registration-buy-resolution-and-change-dns.html)

### （2）购买非中国大陆地区的VPS

1）为什么要购买非中国大陆地区的VPS？

因为中国大陆经营的正规VPS提供商都在国家正规备案的，不允许你用VPS搭建V2Ray节点服务器，一旦发现你违规使用会停掉你的VPS，还可能不退款。另外，即使你可以搭建科学上网服务器，而且没有被发现，那么也可能被监控，并有隐私泄露的风险，因为像国内大厂的云产品上面都有监控代码，你未必能清理干净。鉴于以上原因，我建议你选择非天朝公司且不在大陆备案经营的VPS提供商。

2）使用香港或者澳门公司的VPS怎么样？

当然是可以的，但是他们都很贵，而且仍是天朝下辖地区，还是不碰为好。我推荐你选择美国、台湾、日本、新加坡、韩国等地区机房的VPS，甚至欧洲公司的产品，毕竟中国台湾还没有被统一，天朝直接管不了。

通过以上分析和介绍，我相信你也不愿意选择阿里云、腾讯云、百度云和华为云等这样的国内大公司的VPS，最好选择美国西海岸、日本东京或新加坡机房的VPS。

3）用什么VPS搭建Xray服务器比较好？

国外VPS哪家好？一键搭建Xray服务器推荐你使用 **[Vultr](https://iyideng.net/special/vps/latest-vultr-vps-registration-purchase-and-installation-system-tutorial.html)**（**推荐**）、**[搬瓦工(BandwagonHOST)](https://iyideng.net/special/vps/bandwagonhost-vps-all.html)** 或 **[Hostwinds](https://ad.bgfw.cc/Hostwinds)** 等大公司的VPS。其中，Vultr在全球拥有23个数据中心，虽然在不同国家和地区的机房数据中心对中国大陆不同地域网络的访问速度和延迟有些差别，但你只需要测试好最适合自己当地网络环境的机房位置即可，因为你可以方便且无限制地免费更换IP，直至找到最适合你的那个数据中心。**毋庸置疑，在优质国外VPS服务商中，Vultr是性价比最高、最值得推荐的一家。**另外，BandwagonHOST(搬瓦工)在中国大陆的知名度非常高，它的速度和稳定性都很不错，尤其是 CN2 GIA 线路的套餐，但价格非常贵，是同类产品中相对较贵的，性价比一般，而且现在换IP也非常贵。最后，鉴于Hostwinds在国外口碑非常好，服务器安全稳定，还支持免费更换IP，网站不仅支持中文操作界面，而且有中文客服实时在线，所以也是值得一试的。

如果你是老鸟的话，你应该懂得任何一家VPS都有值得推荐的优势。如果你追求速度和稳定性的话，我推荐您使用有中国电信 CN2 GIA、中国联通 CUVIP(AS9929) 或移动CMI，甚至日本软银等高端线路的VPS。其中，由于中国香港、中国台湾、日本和韩国的数据中心更靠近中国大陆，网络延迟相对更低，连接响应速度会更快，但峰值网络带宽并不一定高。虽然这些国家和地区的物理优势明显，但价格也是比较贵，尤其 CN2 GIA 和CU2(AS9929) 线路非常昂贵，不太适合普通用户，所以我们一般选择较多选择美国或欧洲机房的特殊优化或高端线路，性价比相对较高。**如果你追求超高性价比的话，我相信Vultr肯定是你最好的选择。**

关于 Vultr 的账户注册、套餐购买和VPS服务器系统安装与远程管理的详细使用教程，请参考 [最新Vultr账户注册、VPS套餐购买与服务器系统安装以及SSH远程管理的详细图文教程](https://iyideng.net/special/vps/latest-vultr-vps-registration-purchase-and-installation-system-tutorial.html)，鉴于图文教程已经非常详尽，我这里就不再赘述，我后面的图文教程均以 Vultr VPS 为例进行演示。

## 3、SSH远程连接并管理 Vultr VPS

关于 Xshell/Putty 远程连接并管理 Vultr VPS 服务器的详细使用教程，请参考 [最新Vultr账户注册、VPS套餐购买与服务器系统安装以及SSH远程管理的详细图文教程](https://iyideng.net/special/vps/latest-vultr-vps-registration-purchase-and-installation-system-tutorial.html#5SSH_lian_jie_yuan_cheng_Vultr_VPS)，鉴于图文教程已经非常详尽，我这里就不再赘述。

当完成 Vultr 账户注册、套餐购买并成功连接到远程 VPS 之后，我们就可以继续下面的步骤了。

## 4、执行Xray服务器一键搭建脚本

### （1）Xray一键脚本升级版（Atrandys作品）

1）支持环境与特点

-   支持 CentOS 7 / Debian 9+ / Ubuntu 16.04+ 系统；
-   VLESS自动配置文件：VLESS+TCP+XTLS / VLESS+TCP+TLS / VLESS+WS+TLS
-   支持切换配置文件
-   安装时不要开启CDN，安装完成后vless+ws+tls模式可以开启CDN，切换tcp模式记得关闭CDN

2）安装 Curl 依赖包

```
yum update -y && yum install curl -y #CentOS/Fedora/RHEL
apt-get update -y && apt-get install curl -y #Debian/Ubuntu
```

3）执行一键安装命令

```
bash <(curl -Ls https://raw.githubusercontent.com/atrandys/xray/main/install_triple_config.sh)
```

4）选择安装模式

执行Xray一键脚本安装命令后，有以下3种模式可供选择，如下图所示：

![](https://iyideng.net/wp-content/uploads/2020/12/Xray%E4%B8%80%E9%94%AE%E5%AE%89%E8%A3%85%E8%84%9A%E6%9C%AC%E5%8D%87%E7%BA%A7%E7%89%88.png)

5）安装完成后自动显示“Xray配置参数”，然后请到Xray客户端手动添加配置即可。如下图所示：

![](https://iyideng.net/wp-content/uploads/2020/12/Xray%E9%85%8D%E7%BD%AE%E5%8F%82%E6%95%B0.png)

6）再次执行一键脚本，既可在菜单项选择“5.切换配置”，切换至其它安装模式。

### （2）Xray七合一共存脚本（mack-a作品，推荐）

Xray七合一共存脚本(v2ray-agent)具有（VLESS+TCP+TLS/VLESS+TCP+XTLS/VLESS+WS+TLS/VMess+TCP+TLS/VMess+WS+TLS/Trojan/Trojan-Go WS）等多种安装模式，还能自动生成伪装博客，且支持多内核安装。

v2ray-agent项目官网：[https://github.com/mack-a/v2ray-agent](https://github.com/mack-a/v2ray-agent)

**1）脚本特性**

-   支持Xray-core\[XTLS\]、v2ray-core \[XTLS\]、v2ray-core
-   支持不同核心之间的配置文件互相读取。
-   支持 VLESS/VMess/trojan/trojan-go–>ws的协议
-   支持Debian、Ubuntu、Centos，支持主流的cpu架构。**不建议用Centos、以及低版本的系统**
-   支持个性化安装。
-   不需要卸载就可以重装任何组合。卸载脚本时，是完全卸载无残留。
-   支持纯ipv6，ipv6注意事项
-   支持ipv6人机验证 **需自己申请IPv6隧道，不建议使用自带的IPv6**

**2）支持组合方式**

-   VLESS+TCP+TLS
-   VLESS+TCP+xtls-rprx-direct【推荐】
-   VLESS+WS+TLS
-   VMess+TCP+TLS
-   VMess+WS+TLS
-   Trojan【推荐】
-   Trojan-Go+WS

**3）脚本目录**

-   v2ray-core 【/etc/v2ray-agent/v2ray】
-   Xray-core 【/etc/v2ray-agent/xray】
-   Trojan 【/etc/v2ray-agent/trojan】
-   TLS证书 【/etc/v2ray-agent/tls】
-   Nginx配置文件 【/etc/nginx/conf.d/alone.conf】、Nginx伪装博客目录 【/usr/share/nginx/html】

**4）安装 wget 命令**

```
yum -y install wget #CentOS/Fedora/RHEL
apt-get install wget #Debian/Ubuntu
```

**5）执行一键安装Xray服务器脚本**

```
wget -P /root -N --no-check-certificate "https://raw.githubusercontent.com/mack-a/v2ray-agent/master/install.sh" && chmod 700 /root/install.sh && /root/install.sh
```

执行Xray七合一共存脚本（mack-a作品）后，如下图所示：

![](https://iyideng.net/wp-content/uploads/2020/12/Xray%E4%B8%83%E5%90%88%E4%B8%80%E5%85%B1%E5%AD%98%E8%84%9A%E6%9C%AC%EF%BC%88make-a%E4%BD%9C%E5%93%81%EF%BC%89.png)

请在上图选择“1.安装”或“2.任意组合安装”（选择“1”只可安装一种模式，自定义项较多；“2”可以多种模式共存；无论何种模式我都建议安装“Xray-core”），然后进行个性化安装过程。如下图所示：

![](https://iyideng.net/wp-content/uploads/2020/12/Xray%E4%B8%83%E5%90%88%E4%B8%80%E5%85%B1%E5%AD%98%E8%84%9A%E6%9C%AC%EF%BC%88make-a%E4%BD%9C%E5%93%81%EF%BC%89%E4%B8%AA%E6%80%A7%E5%8C%96%E5%AE%89%E8%A3%85.png)

我们这里选择“0.VLESS+TLS/XTLS+TCP”，输入“0”回车后，稍等片刻会出现提示，请按照提示输入你要配置的域名，然后回车；稍等片刻会出现提示“当前域名的IP为 \[11.22.33.44\]，是否正确\[y/n\]？”（其中，“11.22.33.44”为教程演示用IP，请勿直接照搬），如果域名解析的IP地址正确，则输入字母“y”，然后回车即可进入整个安装过程。安装完成后如下图所示：

![](https://iyideng.net/wp-content/uploads/2020/12/Xray%E4%B8%83%E5%90%88%E4%B8%80%E5%85%B1%E5%AD%98%E8%84%9A%E6%9C%AC%EF%BC%88make-a%E4%BD%9C%E5%93%81%EF%BC%89VLESS-TCP-TLSXTLS-directXTLS-splice.png)

请复制以上生成的通用json(VLESS+TCP+TLS)文件的配置代码，然后到Xray客户端修改配置文件即可。与此同时，Xray七合一共存脚本（mack-a作品）会生成伪装网站，如下图所示：

![](https://iyideng.net/wp-content/uploads/2020/12/Xray%E4%B8%83%E5%90%88%E4%B8%80%E5%85%B1%E5%AD%98%E8%84%9A%E6%9C%AC%EF%BC%88make-a%E4%BD%9C%E5%93%81%EF%BC%89%E7%94%9F%E6%88%90%E7%9A%84%E4%BC%AA%E8%A3%85%E7%BD%91%E7%AB%99.png)

**6）脚本管理命令**

vasma #启动脚本，唤出管理菜单

```
systemctl restart xray #重启xray
systemctl restart v2ray #重启v2ray
systemctl restart trojan-go #重启trojan-go

systemctl start xray #启动xray
systemctl start v2ray #启动v2ray
systemctl start trojan-go #启动trojan-go

systemctl stop xray #关闭xray
systemctl stop v2ray #关闭v2ray
systemctl stop trojan-go #关闭trojan-go

nginx -s reload #重启Nginx
nginx #启动Nginx
nginx -s stop #关闭Nginx
```

**7）注意事项**

-   修改Cloudflare->SSL/TLS->Overview->Full
-   Cloudflare —> A记录解析的云朵必须为灰色
-   wget: command not found \[**这里需要自己手动安装下wget**\]，如未使用过Linux，[点击查看](https://github.com/mack-a/v2ray-agent/tree/master/documents/install_tools.md)安装教程
-   脚本安装路径\[/etc/v2ray-agent\]
-   不支持非root账户
-   现在脚本进入相对稳定的时期，如果有功能不完善的地方，请提issues。
-   **脚本默认屏蔽BT**
-   **中间的版本号升级意味可能不兼容之前安装的内容，如果不是追新用户或者必须升级的版本请谨慎升级。 例如 2.2.\*，不兼容2.1.\***
-   **建议纯净系统**
-   **如发现Nginx启动问题，请先卸载掉自编译的nginx或者重新build系统**

### （3）multi-v2ray 多用户管理脚本（Jrohy作品）

现在，multi-v2ray 多用户管理脚本已增加支持一键搭建VLESS/VLESS+WS/VLESS+XTLS/Trojan/Xray服务器，功能更强大了。Jrohy大神出品必属精品，功能强大，操作简单，可定制性也非常强。

**温馨提醒：**现在，multi-v2ray 多用户管理脚本已增加支持一键搭建VLESS/VLESS+WS/VLESS+XTLS/Trojan/Xray服务器，功能更强大了。在这里，我推荐您使用 Debian 9/11 或 CentOS 7 系统搭建环境（推荐使用 Debian 11 系统），如果您使用其他系统环境搭建可能会遇到莫名其妙的问题，浪费时间和精力。经测试，使用 Debian 10 搭建，curl组件可能无法正常使用，导致一键脚本安装命令执行不成功，请认真斟酌！！！

**安装 Curl 依赖包**

```
yum update -y && yum install curl -y #CentOS/Fedora
apt-get update -y && apt-get install curl -y #Debian/Ubuntu
```

**执行一键安装脚本命令**

```
source <(curl -sL https://multi.netlify.app/v2ray.sh) --zh
```

回车执行上述代码，稍等片刻即可安装完成，期间不需要你任何操作。安装完成后，如下图所示：

![](https://iyideng.net/wp-content/uploads/2020/07/multi-v2ray-%E5%A4%9A%E7%94%A8%E6%88%B7%E7%AE%A1%E7%90%86%E8%84%9A%E6%9C%AC%E5%AE%89%E8%A3%85%E5%AE%8C%E6%88%90.png)

此时，Xray服务并没有被安装，你需要输入管理命令“Xray”，这时候系统会提示你“检测到本机没安装xray, 正在自动安装..”，请稍等片刻，即可完成。如下图所示：

![](https://iyideng.net/wp-content/uploads/2020/12/multi-v2ray-%E5%A4%9A%E7%94%A8%E6%88%B7%E7%AE%A1%E7%90%86%E8%84%9A%E6%9C%AC%E5%AE%89%E8%A3%85Xray%E6%9C%8D%E5%8A%A1.png)

**一键安装脚本升级命令(保留配置文件更新)**

```
source <(curl -sL https://multi.netlify.app/v2ray.sh) -k
```

**一键安装脚本卸载命令**

```
source <(curl -sL https://multi.netlify.app/v2ray.sh) --remove
```

**一键安装脚本的V2Ray管理命令行参数**

```
v2ray [-h|--help] [options]
    -h, --help           查看帮助
    -v, --version        查看版本号
    start                启动 V2Ray
    stop                 停止 V2Ray
    restart              重启 V2Ray
    status               查看 V2Ray 运行状态
    new                  重建新的v2ray json配置文件
    update               更新 V2Ray 到最新Release版本
    update.sh            更新 multi-v2ray 到最新版本
    add                  新增mkcp + 随机一种 (srtp|wechat-video|utp|dtls|wireguard) header伪装的端口(Group)
    add [wechat|utp|srtp|dtls|wireguard|socks|mtproto|ss]     新增一种协议的组，端口随机,如 v2ray add utp 为新增utp协议
    del                  删除端口组
    info                 查看配置
    port                 修改端口
    tls                  修改tls
    tfo                  修改tcpFastOpen
    stream               修改传输协议
    cdn                  走cdn
    stats                v2ray流量统计
    iptables             iptables流量统计
    clean                清理日志
    log                  查看日志
```

xray管理程序使用方法与v2ray管理程序基本一致，请根据提示操作即可。你可以参考文章 [\[一键搭建V2Ray服务器教程\]使用 Vultr VPS 自建V2Ray节点机场及客户端配置多用户实现科学上网](https://iyideng.net/black-technology/cgfw/vmess-v2ray-server-building-and-using-tutorial.html)

## 5、一键安装并开启BBR加速

**一键安装BBR加速脚本，执行如下命令：**

```
cd /usr/src && wget -N --no-check-certificate "https://raw.githubusercontent.com/chiakge/Linux-NetSpeed/master/tcp.sh" && chmod +x tcp.sh && ./tcp.sh
```

执行以上安装命令后，如下图所示：

![](https://iyideng.net/wp-content/uploads/2020/06/BBR%E5%8A%A0%E9%80%9FTCP%E5%8A%A0%E9%80%9F-%E4%B8%80%E9%94%AE%E5%AE%89%E8%A3%85%E7%AE%A1%E7%90%86%E8%84%9A%E6%9C%AC.png)

我这里选择“2”，“安装 BBRplus版内核”加速。在安装过程中，可能会出现如下提示，用右方向键选“<No>”，然后回车。如下图所示：

![](https://iyideng.net/wp-content/uploads/2020/04/TCP%E5%8A%A0%E9%80%9F-%E4%B8%80%E9%94%AE%E5%AE%89%E8%A3%85%E7%AE%A1%E7%90%86%E8%84%9A%E6%9C%AC-%E7%A7%BB%E9%99%A4%E5%8E%9F%E5%86%85%E6%A0%B8%E8%AD%A6%E7%A4%BA.png)

安装完成后会提示重启服务器，这时候输入字母“y”，回车后，重启服务器。当服务器启动后，我们再次执行安装命令，选择“7”启用“使用BBRplus版加速”。

至此，BBR Plus 加速模块安装并启用完成。

## 6、Xray客户端下载与配置使用教程

关于Xray客户端下载与配置使用教程，请参考文章 [Xray – 一款原生支持XTLS黑科技且源自V2Ray却超越V2Ray的科学上网工具及Xray服务器搭建与客户端配置使用教程](https://iyideng.net/black-technology/cgfw/xray-xtls-tutorial.html)

关于Xray客户端winXray使用教程，请参考文章 [winXray – 又一款全能型的科学上网代理客户端 | 支持SS/V2Ray/Trojan/Trojan-Go/Xray等代理 | 仅支持Windows平台](https://iyideng.net/tools/winxray.html)

## 7、自建vps和买机场哪个好

如果你有一定的计算机网络技术基础，还懂得那么一点英文的话，一键搭建Xray服务器是一件非常简单的事情；否则，对你来说还真是有点麻烦，因为从购买域名和选择VPS、远程登录和执行安装代码等，都是一件很闹心的事情，尤其是有问题的时候，一定会让你崩溃，甚至想放弃。如果你已经决定自建Xray服务器（即一键Xray搭建梯子），那么搭梯子VPS推荐你用 [Vultr](https://iyideng.net/special/vps/latest-vultr-vps-registration-purchase-and-installation-system-tutorial.html)，一旦被墙，能很方便地免费更换IP，搭建时请尽可能使用较新的加密方式；否则，请购买Xray机场节点服务。

## 8、优质Xray机场推荐

不管你是萌新，还是老鸟，使用一键安装脚本搭建Xray服务器真的很简单，喜欢折腾的用户可以尝试更多高级的玩法。如果你不喜欢折腾，只想要一个稳定可靠的科学上网工具，那么我推荐你选择一家。有没有比较好的VLESS订阅节点购买地址？很遗憾，由于VLESS技术还不够成熟稳定，目前还没有使用VLESS协议的Xray机场。

**【温馨提醒】**如果您是新手小白，或不能成功搭建科学上网代理服务器，或对线路节点的速度和稳定性均有更高需求，那么[一灯不是和尚](https://iyideng.net/)推荐您参考文章 [优质高速稳定SS/SSR/Xray/Trojan/V2Ray机场推荐 | 网络加速器梯子推荐](https://iyideng.net/special/bgfw/best-ss-ssr-v2ray-trojan.html)，它能帮助您挑选一家最适合您的优质SS/SSR/Trojan/Xray/V2Ray机场梯子。

本文由更新于2023年1月18日；如果您有什么意见或建议，请在文章下面评论区留言反馈。