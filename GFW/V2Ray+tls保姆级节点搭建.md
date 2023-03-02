
## 前言

为了照顾新手，特地出了一篇小白可以入门的节点搭建教程。

对于新手来说，理论其实不是太过重要，首先能把属于自己的专属节点搭建起来，就算是成功了，熟能生巧，以后自然明白其中的一些道理。

## 视频教程

[![](https://v2rayssr.com/wp-content/uploads/2022/09/2.png)](https://v2rayssr.com/go?url=https://youtu.be/Stdj6U568S0)

## 准备篇

### 购买域名

#### 什么是域名

域名可以说是一个IP地址的代称，目的是为了便于记忆这个IP。例如，xxx.com 是一个域名。人们可以直接访问 xxx.com 来代替 IP 地址，然后域名系统（DNS）就会将它转化成便于机器识别的 IP 地址。这样，人们只需要记忆 xxx.com 这一串带有特殊含义的字符，而不需要记忆没有含义的数字

推荐在 Namesilo 进行购买，因为他的 WHOIS 隐私 是免费的，可以适当的进行一下隐私保护，而且域名还都挺便宜的

购买地址：[点击访问](https://v2rayssr.com/go?url=https://www.namesilo.com/?rid=6254266mw) （1.88刀/年 起）

![](https://v2rayssr.com/wp-content/uploads/2022/09/12-2.png)

### 购买 VPS

#### VPS 是什么

VPS(Virtual Private Server 虚拟专用服务器)技术，将一部服务器分割成多个虚拟专享服务器的优质服务。每个VPS都可分配独立公网IP地址、独立操作系统、独立超大空间、独立内存、独立CPU资源、独立执行程序和独立系统配置等。

Hosteons，这家不用多说了，老商家，价格便宜，对新手入门比较友好。

> **友情提示：教程演示的这个 OVZ 机型的 VPS 很垃圾**
> 
> **本视频仅仅只是讲解上网过程！若是有更好的选择，请选择横向最好的产品！搭建方法一模一样**

高端 VPS 推荐：[搬瓦工 CN2 – E 商务优化 VPS](https://v2rayssr.com/bwg.html) 、[搬瓦工测速视频观看](https://v2rayssr.com/go?url=https://youtu.be/qKdE9n802wI)

<table><tbody><tr><td><strong>CPU</strong></td><td><strong>内存</strong></td><td><strong>储存</strong></td><td><strong>流量</strong></td><td><strong>价格</strong></td><td><strong>链接</strong></td><td><strong>特点</strong></td></tr><tr><td>1核 OVZ</td><td>256 MB</td><td>5 GB SSD</td><td><strong>无限流量</strong></td><td>$12/年</td><td><a href="https://v2rayssr.com/go?url=https://my.hosteons.com/aff.php?aff=965&amp;pid=47" target="_blank" rel="noopener">购买</a></td><td>RDNSD、DOS 保护</td></tr></tbody></table>

### 注册 Cloudflare

注册地址：[点击访问](https://v2rayssr.com/go?url=https://dash.cloudflare.com/sign-up)

### 域名托管 Cloudflare

步骤比较多，推荐观看 演示视频

![](https://v2rayssr.com/wp-content/uploads/2022/09/13.png)

### 下载SSH连接工具

下载地址：[点击下载](https://v2rayssr.com/go?url=http://www.hostbuf.com/t/988.html)

## 实战篇

### 部署节点

测试 VPS 的 IP 是否可用，[点击进入测试](https://v2rayssr.com/go?url=https://ping.chinaz.com/)

用 SSH 工具连接 VPS，

输入相关代码

```
apt update -y          # Debian/Ubuntu 命令apt install -y curl socat    #Debian/Ubuntu 命令bash <(curl -Ls https://raw.githubusercontent.com/vaxilu/x-ui/master/install.sh) #X-ui面板安装
```

完成 X-ui 安装以后，我们可以输入 `VPSIP:端口`（如1.1.1.1:12345） 登录 X-ui 的管理面板（可以登录代表安装成功）

在 VPS 输入 `x-ui` 命令，进入 X-ui 的命令菜单

```
  x-ui 面板管理脚本  0. 退出脚本————————————————  1. 安装 x-ui  2. 更新 x-ui  3. 卸载 x-ui————————————————  4. 重置用户名密码  5. 重置面板设置  6. 设置面板端口  7. 查看当前面板设置————————————————  8. 启动 x-ui  9. 停止 x-ui  10. 重启 x-ui  11. 查看 x-ui 状态  12. 查看 x-ui 日志————————————————  13. 设置 x-ui 开机自启  14. 取消 x-ui 开机自启————————————————  15. 一键安装 bbr (最新内核)  16. 一键申请SSL证书(acme申请)
```

选择 16，申请 SSL 的证书。（申请需要有 Cloudflare API ，可以 [观看视频](https://v2rayssr.com/go?url=https://youtu.be/Stdj6U568S0) 获取 API）

![](https://v2rayssr.com/wp-content/uploads/2022/09/12-3.png)

申请的时候是申请的泛域名证书，所以，填写域名的时候，只填入 域 也就好了，例如 `xxx.com` 的格式。何为 泛域名？请 [观看视频](https://v2rayssr.com/go?url=https://youtu.be/Stdj6U568S0)

申请成功以后，证书和密钥文件在 VPS 目录的 `/root/cert` 文件夹里面

至于节点搭建，因为有可视化的管理面板，所以搭建起来，也是比较方便的，若还是不明白，请看 [演示视频](https://v2rayssr.com/go?url=https://youtu.be/Stdj6U568S0)

## 套用 CDN 拯救垃圾线路

当然，收费的 CDN 用来做代理，成本是巨大的，所以，Cloudflare 的免费内容分发系统，是我们偶尔用到的。

### 开启方法

1、打开 DNS 解析页面的的 小云朵。

2、找到 SSL/TLS 选项，把加密选项改为 `完全/完全（严格）`

3、在 X-ui 面板建立 `VLESS+WS+TLS` 的代理节点

### Cloudflare 优选 IP

Windows 用户下载本地压缩包，解压运行。[点击下载](https://v2rayssr.com/go?url=https://github.com/badafans/better-cloudflare-ip/releases/latest/download/batch.zip)

MacOS 用户直接在终端运行如下命令即可

```
curl https://raw.githubusercontent.com/badafans/better-cloudflare-ip/master/shell/cf.sh -o cf.sh && chmod +x cf.sh && ./cf.sh
```

以下是套用 CDN 之后的真实测速：测试时间 `09:28` 8K 视频

![](https://v2rayssr.com/wp-content/uploads/2022/09/122.png)

## 后记

新手经过上面的一些教程，就大致的了解了为什么要购买域名、VPS 是干什么的，如何简单的操控 VPS，以及代理节点的搭建等问题了。

