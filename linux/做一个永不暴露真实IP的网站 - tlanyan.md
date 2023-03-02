 [安全](https://itlanyan.com/category/%e5%ae%89%e5%85%a8/)2022年11月30日

> 欢迎加入tg群交流：[@tlanyantg](https://t.me/tlanyantg)

防范[DDoS攻击](https://itlanyan.com/tag/DDos%E6%94%BB%E5%87%BB/)最主要的手段是加钱上高防，同时隐藏网站真实IP。前文 [隐藏网站的真实ip](https://itlanyan.com/hide-site-real-ip/) 简要介绍了网站隐藏真实ip的基本操作，但不够详细和深入。本文详细介绍几种网站隐藏真实ip的方法和优缺点，让你能真正做一个永不暴露真实IP的网站。

## 做一个永不暴露真实IP的网站

既然不想暴露网站的真实IP，那么真实服务器前面至少套一层代理。一般来说，位于最前线的反向代理主要有如下几种：

-   **CDN**：内容分发网络，就近为用户提供服务，加速访问；
-   **高防IP**：高防IP一般位于大带宽的骨干网节点上，用于清洗DDoS流量；
-   **SLB**：负载均衡器，用在大流量、繁忙的网站上，常见的SLB有LVS、F5等。

这三种反向代理主要作用不一样，配置好的情况下都能隐藏服务器真实IP。对于普通的网站，使用CDN或者高防就足够，业务量大的情况下才会用到SLB。

下面介绍使用了反向代理的情况下，隐藏网站真实IP的操作。

### 防火墙

使用防火墙是最简单的做法，即：将反向代理的回源IP加入白名单，屏蔽其他IP的任何请求。

例如使用[CloudFlare](https://www.cloudflare.com/)的免费CDN服务，其回源IP可从 [https://www.cloudflare.com/zh-cn/ips/](https://www.cloudflare.com/zh-cn/ips/) 获取，然后将其加入白名单，同时屏蔽其他IP：

```
# 将cf ip地址放在 cf_ips.txt
# 首先将cf的ip加入白名单
while read -r line
do
  firewall-cmd --zone=trusted --add-source=$line
done < cf_ips.txt
# 然后移除其他ip对http和https服务的访问
firewall-cmd --remove-service=http
firewall-cmd --remove-service=https
```

经过上述设置，[Cloudflare](https://itlanyan.com/tag/cloudflare/) 的IP能正常访问，其他IP完全无法访问真实ip的网站服务器，很好的隐藏了真实IP。

该方法设置简单，适用于服务器托管单站点的情形。当服务器上托管多个网站，并且某些站点需要直接暴露外网时，这种做法缺乏灵活性，无法实现。

> 也可以通过Nginx的allow/deny指令达到相同效果

### IPv6

对于防火墙和网络不熟悉的网友，可以考虑使用IPv6来隐藏网站的真实IP。具体操作为：

1.找一台有IPv6地址的服务器，只有IPv6的[NAT VPS](https://itlanyan.com/nat-vps-the-right-way/)更好。目前IPv6地址正在普及中，许多商家都免费提供IPv6地址，例如 [一些VPS商家整理](https://itlanyan.com/vps-merchant-collection/) 中的 [阿里云](https://www.aliyun.com/minisite/goods?userCode=yieb93ge)、[Vultr](https://www.vultr.com/?ref=8942545-8H)、[Linode](https://www.linode.com/?r=4459a4f56be13978ca0d156d0d0eaf6d9bc82853)、[CloudCone](https://app.cloudcone.com.cn/?ref=5033)，有的还提供不止一个IPv6地址；

2\. 设置网站只监听IPv6端口。以[Nginx](https://itlanyan.com/tag/nginx/)为例，网站配置文件形如：

```
server {
    listen [::]:80;
    server_name 主机名; # 请改成自己的主机名

    return 301 https://主机名$request_uri;
}
server {
    listen      [::]:443 ssl http2;
    server_name  主机名;
    ssl_certificate 证书路径;
    ssl_certificate_key ssl密钥路径;
    # 其他设置
}
```

3\. 找一家支持只有IPv6的CDN，例如 [Cloudflare](https://itlanyan.com/tag/cloudflare/)，设置IPv6解析：

[![CloudFlare设置ipv6解析](https://itlanyan.com/wp-content/uploads/2020/10/CloudFlare%E8%AE%BE%E7%BD%AEipv6%E8%A7%A3%E6%9E%90-1024x225.jpg)](https://itlanyan.com/do-hide-site-real-ip/cloudflare%e8%ae%be%e7%bd%aeipv6%e8%a7%a3%e6%9e%90-2/)

CloudFlare设置ipv6解析

经过上面三步设置，基本上可确保不会泄漏真实IP，原因如下：

1\. 绝大多数情况下，人们都会理所当然的找IPv4，不会想到你的网站根本不存在IPv4网络上；

2\. 相对于IPv4，IPv6的地址段实在太庞大。即使有zmap这种几小时扫描完全球ipv4段的神器，或者Shodan搜索引擎，也很难从海量地址中寻找单个地址。

如果不放心，可以同样加上防火墙，就万无一失了：

```
# 首先将cf的ip加入白名单
while read -r line
do
  firewall-cmd --zone=trusted --add-source=$line
done < cf_ips.txt
# 然后屏蔽其他地址对ipv6的访问权限
firewall-cmd --add-rich-rule="rule family='ipv6' source address='::0/0' drop"
```

该方法同样设置简单，以奇招胜出，单台服务器能托管多个网站，并且其他网站可直接暴露不受影响。

### CNAME

另一种常见隐藏真实IP方式是使用CNAME，同样无需设置防火墙。其操作如下：

1\. CDN回源时使用CNAME方式回源到另一个主机名上。例如itlanyan.com回源的www.abcdexfd.com。需要注意的是，前端域名和源站域名最好不是同一个，防止通过爆破二级域名泄漏真实IP；

2\. 在源站服务器上设置默认站点，防止通过host方式爆破。由于默认站点只是为了防止SNI方式泄漏真实IP，因此使用自签证书即可：

```
# 生成密钥
openssl genrsa -out example.key 2048
# 生成证书，期间需要填一些信息
openssl req -new -x509 -days 3650 -key example.key -out example.pem
```

接着以Nginx为例，设置默认站点：

```
server {
  listen 80 default_server;
  server_name example.com;
  return 301 https://example.com$request_uri;
}

server {
  listen 443 ssl http2;
  server_name example.com default_server;
  ssl_certificate example.pem;
  ssl_certificate_key example.key;
}
```

然后重启Nginx即可。

该方法无需设置防火墙，设置较为简单，但是需要额外一个域名。

## 注意事项

以上操作只能让他人在明面上无法直接访问真实服务器，但请仔细阅读 [隐藏网站的真实ip](https://itlanyan.com/hide-site-real-ip/)  中的建议，防止有发送邮件、WordPress pingback等隐式暴露IP的行为。

## 遇到DDoS怎么办?

如果域名之前从未用过，一出道就用上面提到的方法，基本上可以保证不会泄漏网站的真实IP。

但是不泄漏真实IP不代表不会被[DDoS](https://itlanyan.com/tag/DDos%E6%94%BB%E5%87%BB/)或者[CC攻击](https://itlanyan.com/tag/cc%E6%94%BB%E5%87%BB/)，遇到DDoS怎么办？解决办法主要有：

1\. 加钱上高防保平安；

2\. DNS解析域名到127.0.0.1保平安；

3\. 关机保平安。

请根据实际情况和业务需求采取相应措施。

## 参考

1.  [服务器简易防CC攻击设置](https://itlanyan.com/server-protect-cc-attack/)
2.  [网站添加ipv6访问](https://itlanyan.com/add-ipv6-for-website/)
3.  [DDOS 攻击的防范教程](http://www.ruanyifeng.com/blog/2018/06/ddos.html)

赞(11)