[![](https://v2rayssr.com/indexpic/mn-banner.png)](https://v2rayssr.com/dbwp)

-   [前言](https://v2rayssr.com/cf-auto-passwall.html#%E5%89%8D%E8%A8%80 "前言")
-   [准备工作](https://v2rayssr.com/cf-auto-passwall.html#%E5%87%86%E5%A4%87%E5%B7%A5%E4%BD%9C "准备工作")
-   [视频教程](https://v2rayssr.com/cf-auto-passwall.html#%E8%A7%86%E9%A2%91%E6%95%99%E7%A8%8B "视频教程")
-   [PacificRack 便宜套餐](https://v2rayssr.com/cf-auto-passwall.html#PacificRack_%E4%BE%BF%E5%AE%9C%E5%A5%97%E9%A4%90 "PacificRack 便宜套餐")
-   [更新系统开启BBR加速](https://v2rayssr.com/cf-auto-passwall.html#%E6%9B%B4%E6%96%B0%E7%B3%BB%E7%BB%9F%E5%BC%80%E5%90%AFBBR%E5%8A%A0%E9%80%9F "更新系统开启BBR加速")
-   [安装 Xray 可视化面板](https://v2rayssr.com/cf-auto-passwall.html#%E5%AE%89%E8%A3%85_Xray_%E5%8F%AF%E8%A7%86%E5%8C%96%E9%9D%A2%E6%9D%BF "安装 Xray 可视化面板")
    -   [面板安装代码](https://v2rayssr.com/cf-auto-passwall.html#%E9%9D%A2%E6%9D%BF%E5%AE%89%E8%A3%85%E4%BB%A3%E7%A0%81 "面板安装代码")
    -   [为域名申请证书](https://v2rayssr.com/cf-auto-passwall.html#%E4%B8%BA%E5%9F%9F%E5%90%8D%E7%94%B3%E8%AF%B7%E8%AF%81%E4%B9%A6 "为域名申请证书")
-   [CF 自动优选 IP 并部署](https://v2rayssr.com/cf-auto-passwall.html#CF_%E8%87%AA%E5%8A%A8%E4%BC%98%E9%80%89_IP_%E5%B9%B6%E9%83%A8%E7%BD%B2 "CF 自动优选 IP 并部署")
-   [设置软路由定时任务](https://v2rayssr.com/cf-auto-passwall.html#%E8%AE%BE%E7%BD%AE%E8%BD%AF%E8%B7%AF%E7%94%B1%E5%AE%9A%E6%97%B6%E4%BB%BB%E5%8A%A1 "设置软路由定时任务")
-   [后记](https://v2rayssr.com/cf-auto-passwall.html#%E5%90%8E%E8%AE%B0 "后记")

## 前言

既然我们有套用 CDN 的需求，那么难免就会遇到 IP 优选的问题。

每次进行 IP 优选，我们都需要等待一个结果，然后还需要把这个结果编辑到节点，那这个真的是很影响我们科学上网的心情。

IP 优选 = 耗时 + 耗力，那有没有更为简单的办法，让 IP 优选在我们的软路由上面定时运行，然后自动把运行的结果部署到我们的节点上面去呢？显然，这个是可以的。

## 准备工作

1、VPS 一台，重置好主流的操作系统（作者用的下面的 PacificRack VPS）

2、域名一个，托管到 CloudFlare 并解析域名到此台 VPS （[不会请点击](https://v2rayssr.com/yumingreg.html)）

## 视频教程

[![](https://v2rayssr.com/wp-content/uploads/2021/06/1-2.png)](https://v2rayssr.com/go?url=https://youtu.be/WzRHi9f9QKg)

## PacificRack 便宜套餐

洛杉矶 QN 机房：（ **下面链接均无 AFF** ）

<table><tbody><tr><td>内存</td><td>CPU</td><td>内存</td><td>SSD</td><td>流量</td><td>带宽</td><td>价格</td><td>购买</td></tr><tr><td>1G</td><td>1核</td><td>1G</td><td>20G</td><td>1T</td><td>100M</td><td>$9.99/年</td><td><a href="https://v2rayssr.com/go?url=https://pacificrack.com/portal/cart.php?a=confproduct&amp;i=0" target="_blank" rel="noopener"><strong>直达链接</strong></a></td></tr><tr><td>2G</td><td>2核</td><td>2G</td><td>30G</td><td>2T</td><td>100M</td><td>$19.99/年</td><td><a href="https://v2rayssr.com/go?url=https://pacificrack.com/portal/cart.php?a=confproduct&amp;i=1" target="_blank" rel="noopener"><strong>直达链接</strong></a></td></tr><tr><td>4G</td><td>4核</td><td>4G</td><td>40G</td><td>4T</td><td>100M</td><td>$29.99/年</td><td><a href="https://v2rayssr.com/go?url=https://pacificrack.com/portal/cart.php?a=confproduct&amp;i=2" target="_blank" rel="noopener"><strong>直达链接</strong></a></td></tr></tbody></table>

下图为该 VPS 的管理后台

![](https://v2rayssr.com/wp-content/uploads/2021/06/%E6%88%AA%E5%B1%8F2021-06-06-08.48.23.png)

## 更新系统开启BBR加速

> 有防火墙的 VPS ，请自行放行相关的防火墙端口，或是关闭防火墙

CentOS 防火墙相关命令

```
firewall-cmd --state                   # 查看防火墙状态systemctl stop firewalld.service       # 停止防火墙systemctl disable firewalld.service    # 禁止防火墙开机自启
```

Debian 更新命令

```
apt update -yapt install curl wget socat tar -y
```

CentOS 更新命令

```
yum update -yyum install curl wget socat tar -y
```

## 安装 Xray 可视化面板

### 面板安装代码

```
bash <(curl -Ls https://blog.sprov.xyz/v2-ui.sh)
```

### 为域名申请证书

（若是无法申请证书，请 [点击这里](https://v2rayssr.com/ssl.html) 查看更多证书申请的方式）

> **重要申明**
> 
> ___
> 
> **2021 年 6 月 17 日更新：**
> 
> **从 acme.sh v 3.0.0 开始，acme.sh 使用 Zerossl 作为默认 ca，您必须先注册帐户（一次），然后才能颁发新证书。**
> 
> **具体操作步骤如下：**
> 
> 1、安装 Acme 脚本之后，请先执行下面的命令（下面的邮箱为你的邮箱）  
> `~/.acme.sh/acme.sh --register-account -m xxxx@xxxx.com`
> 
> 2、其他的命令暂时没有变动

替换下面的 `mydomain.com` 为你解析的域名，`xxx@xxxx.com` 为你邮箱

```
curl https://get.acme.sh | sh~/.acme.sh/acme.sh --register-account -m xxxx@xxxx.com~/.acme.sh/acme.sh  --issue -d mydomain.com   --standalone~/.acme.sh/acme.sh --installcert -d mydomain.com --key-file /root/private.key --fullchain-file /root/cert.crt
```

默认证书会创建到 `/root` 文件夹 ，`/root/private.key` 和 `/root/cert.crt`

打开 V2-UI 面板，并创建一个 Xray 的 WS + TLS 节点

相关的创建过程请 [观看视频教程](https://v2rayssr.com/go?url=https://youtu.be/WzRHi9f9QKg)。

开启 CloudFlare 的 “小云朵”，开启我们的 CDN。

所用到的脚本，来自 GitHub：Lbingyi 以及 Paniy，作者仅仅只是把相关的功能合并了一下而已。

GitHub 脚本地址：[自动筛选 CF IP，并自动替换优选 IP 为 PassWall 的节点地址](https://v2rayssr.com/go?url=https://github.com/V2RaySSR/cf-auto-passwall)

以下简单带过，相关的命令我会写在下面，操作部分，请自行研究视频。

查看 软路由 PassWall 的自定义节点字符串。

> 关于 vi 的相关操作：  
> 按键盘上面的 `PgUp` 和 `PgDn` 进行翻页，按 `i` 键进行编辑，按 `esc` 退出编辑，按 `:wq` 保存并退出 vi 命令

```
vi /etc/config/passwall 
```

信息里面的 `config global` —— `option tcp_node` 后面的字符串，为你正在使用的自定义节点的绑定字符串

在软路由里面下载 [`cf-auto-passwall`](https://v2rayssr.com/go?url=https://raw.githubusercontent.com/V2RaySSR/cf-auto-passwall/main/cf-auto-passwall.sh) 脚本

```
wget https://raw.githubusercontent.com/V2RaySSR/cf-auto-passwall/main/cf-auto-passwall.sh
```

下载后，脚本的绝对地址为 `/root/cf-auto-passwall.sh`

编辑该脚本

```
vi cf-auto-passwall.sh
```

更改相关的参数（默认优选带宽大小、节点相对应的字符串），并保存

软路由运行下，看看出没出错。

```
chmod +x cf-auto-passwall.sh && bash cf-auto-passwall.sh
```

## 设置软路由定时任务

> 时程表的格式如下:
> 
> f1 f2 f3 f4 f5 Program
> 
> 其中 f1 是表示分钟，f2 表示小时，f3 表示一个月份中的第几日，f4 表示月份，f5 表示一个星期中的第几天。Program 表示要执行的命令。
> 
> 0 03 \* \* \* 表示每天的凌晨三点

```
0 04 * * * bash /root/cf-auto-passwall.sh > /dev/null
```

上面的意思是，每天的 凌晨 04：00 ，自动运行改脚本（自动优选IP并替换）

## 后记

经过 CDN 的加持，节点的速度也有了很大的提升，YouTube 数据直逼 9W。关键是优选 IP 的时候，再也不用我们人为进行等待了。9.9刀/年 的 VPS，大家随意就好，没有上 AFF 链接。

![](https://v2rayssr.com/wp-content/uploads/2021/06/1-1.png)

## 请输入验证码

请输入图片中的验证码  
点击发送按钮获取验证码

×

官网地址：点击访问 各类套餐以及价格 试用套餐 ¥3/星期 青铜套餐 ¥10/每月 白银套餐 ¥25/每月 25GB 使用流量 120GB 使用流量 300GB 使用流量 7个 在线客户端 7个 在线客户端 7个 在线客户端 150Mbps带宽上限 100Mbps带宽上限 200Mbps带宽上限 仅限个人使用 仅限个人使用 仅限个人使用 技术支持(仅文档中心) 技术支持(有限的客服帮助) 技术支持…

￥undefined

请打开手机使用微信扫码支付

「」

V2RaySSR综合网

×

打开微信扫一扫

扫码并「关注我们的公众号」安全快捷登录

×

**登录用户名**

请填写您的的登录用户名

**验证码** **发送验证码** **密码**

最少6位字符