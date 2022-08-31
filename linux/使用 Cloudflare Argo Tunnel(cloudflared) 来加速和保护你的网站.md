是的，这是一篇记录 cloudflared 的使用历程和一些思路的文章，关于 Cloudflare 的 Argo Tunnel 的用法，我们在 「[Cloudflare Argo Tunnel 小试：我终于可以用树莓派做网站啦](https://dmesg.app/argo-tunnel.html)」 文章中可以看到：Cloudflare Argo Tunnel 提供一个轻量级的 daemon 程序，被称为 cloudflared，用于安装在你自己的机器上并主动和 Cloudflare 保持连接，并可以提供 Web 服务，用 Cloudflare 的话来说，就是：

> With Tunnel, users can create a private link from their origin server directly to Cloudflare without a publicly routable IP address. Instead, this private connection is established by running a lightweight daemon, cloudflared, on your origin, which creates a secure, outbound-only connection. This means that only traffic that routes through Cloudflare can reach your origin.

我们之前要建立一个网站并加入 Cloudflare 一般会有如下典型的步骤：

1.  租用服务器并在上面运行我们的 App（比如这里监听了 `127.0.0.1:8080`）
2.  配置 Caddy/Nginx 监听某个域名，并反向代理到 App 的端口
3.  在 Cloudflare 上配置一个域名指向我们的服务器 IP，并开启 CDN（保护源站 IP）
4.  （可选）在服务器上/Nginx 上设置一堆复杂的规则防止被一些工具通过扫 IP/TLS 证书发现我们的源站 IP（导致被 DDoS）

这其中有许多的麻烦不说，可能你设置好了许多防火墙规则，最后发现由于 Docker 在你的机器上开个洞（如 「[我的 ufw 怎么又不好用了？使用 docker 时如何拒绝非 cloudflare 访问](https://dmesg.app/ufw-vs-docker.html)」 一文所记录），导致你的应用其实直接裸奔在 `<服务器公网 IP>:8080` 上，这就很尴尬了。

对于以上的情况，使用 Cloudflare Argo Tunnel 就可以解决这个问题，它的工作模式如下：

1.  你的服务器甚至不需要拥有公网 IP
2.  你的 App 继续运行在本地某个端口（比如这里监听了 `127.0.0.1:8080`）
3.  你可以在任何地方设置防火墙，直接阻断服务器除了 SSH 以外的所有 Inbound 流量
4.  运行 Argo Tunnel 的程序（cloudflared）并设置本地应用的端口，这里 Argo Tunnel 会根据你的要求自动生成一个域名给你使用
5.  直接访问 Argo Tunnel 提供的域名

![](https://nova-moe-blog-assets.webp.se/pics/cloudflare-argo/argo.png)

> 是不是听上去和 Tor 的 Hidden Service 很像？

## Practical cloudflared

作为一篇记录，我们来看看上述操作需要怎么做吧。（假设你已经有了一个 Cloudflare 帐号并且已经添加好了域名）

首先我们需要下载 `cloudflared` ，由于就是一个 binary，我们直接下载下来跑就好：

```
wget https://github.com/cloudflare/cloudflared/releases/download/2022.3.4/cloudflared-linux-amd64
chmod +x cloudflared-linux-amd64
mv cloudflared-linux-amd64 /usr/bin/cloudflared
```

本地登录一下 `cloudflared` ，这个时候会给出一个 URL，用浏览器访问后选择域名即可：

```
cloudflared tunnel login
```

此时 Cloudflare 会创建一个 `cert.pem` 文件放在你的 `~/.cloudflared` 目录下。

然后创建一个隧道，比如我这里叫 `knat-tunnel`：

```
cloudflared tunnel create knat-tunnel
```

此时会输出一些隧道 ID 之类的信息（比如我这里是 `xxxxxxx-5b0e-xxxx-8034-xxxxxxx`），需要记录一下，接下来需要用到。

给隧道创建一个域名，比如我这里用了 `tunnel.knat.network`：

```
cloudflared tunnel route dns knat-tunnel tunnel.knat.network
```

最后我们需要创建一个配置文件，比如我打算放在 `~/.cloudflared/knat.yml`，文件内容如下：

```
url: http://localhost:8080
tunnel: xxxxxxx-5b0e-xxxx-8034-xxxxxxx
credentials-file: /root/.cloudflared/xxxxxxx-5b0e-xxxx-8034-xxxxxxx.json
```

启动隧道：

```
cloudflared tunnel --config ~/.cloudflared/knat.yml run
```

此时会有一些调试信息，比如它告诉你连接到了哪些 Cloudflare 节点之类的：

```
022-03-26T06:52:31Z INF Starting tunnel tunnelID=xxxxxxx-5b0e-xxxx-8034-xxxxxxx
2022-03-26T06:52:31Z INF Version 2022.3.4
...
2022-03-26T06:52:31Z INF Generated Connector ID: 624aa020-a90a-4bef-91da-330c74edb02f
2022-03-26T06:52:31Z INF Initial protocol http2
2022-03-26T06:52:31Z INF Starting metrics server on 127.0.0.1:44143/metrics
2022-03-26T06:52:33Z INF Connection 34504363-646c-46a2-973d-bd112943c58f registered connIndex=0 location=KIX
2022-03-26T06:52:34Z INF Connection 7a3ec8f7-482c-4fe5-93c4-69d1177ca457 registered connIndex=1 location=NRT
2022-03-26T06:52:35Z INF Connection 7d571bdb-96d2-49d3-b8bf-14754aa6cf8b registered connIndex=2 location=KIX
2022-03-26T06:52:36Z INF Connection 473e30ae-e98b-4da1-8768-12bf5304c7ab registered connIndex=3 location=NRT
```

这个时候本地启动一个监听在 `127.0.0.1:8080` 的服务之后就可以直接通过这个域名访问了。

> 如果你希望在别的机器上创建隧道的话，只需要把 `~/.cloudflared/` 目录一并复制走即可，无需重新登录。

## Speed test

创建隧道的过程相信很多人都已经做过测试了，但是本文写作的动机在于这里的速度，我们知道，Cloudflare 对于网站而言是默认就近回源，在我之前的文章：「[搭建 Cloudflare 背后的 IPv6 AnyCast 网络](https://nova.moe/simulate-argo-using-anycast-network-behind-cloudflare/)」中就有如下描述：

-   一般的，Cloudflare 回源的方式是由访客命中的数据中心进行回源
-   开启 Argo 之后，Cloudflare 由它认为距离你源站 IP 最近最快的 Cloudflare 数据中心回源

如果上述两个结论不好理解的话，我们带入两个条件来方便大家理解，假设我的博客在法国（假设解析为 `origin.nova.moe`），你是一个大陆访客，假设 Cloudflare 使用的 Nginx，在目前网络环境下：

-   一般的，你会访问到 Cloudflare 位于美西的 San Jose 节点，然后 San Jose 节点上的 Nginx 就 `proxy_pass https://origin.nova.moe;` 了，很简单是不是？但是这样就是「公网回源」
-   如果你有 Argo，你还是会访问到 Cloudflare 位于美西的 San Jose 节点，但是这个时候 Cloudflare 会发现我们的源站和 Paris 的节点距离很近，就会将请求转发到 Cloudflare 位于 Paris 的机器上 `proxy_pass https://origin.nova.moe;`

开了 Argo 之后由于回源流量会有很大一段不经过公网（而是 Cloudflare 的各种公网隧道），所以理论上说路由路径更多地受到 Cloudflare 管控，在某些时候可以减少绕路，简单来说，速度应该会快一些，Cloudflare 官方宣传图如下：

![](https://nova-moe-blog-assets.webp.se/pics/anycast-network/cf-argo.png)

对于这一点而言，既然我们是用了 Argo Tunnel 了，那么这里其实也是适用的，为了说明这一点，我们来个测试，测试的情况如下：

1.  一台赫尔辛基的服务器，300Mbps 的标称带宽
2.  一台日本的服务器，200Mbps 的标称带宽
3.  两台服务器之前延迟为：233 ms ![](https://nova-moe-blog-assets.webp.se/pics/cloudflare-argo/ping.png)
4.  两台机器之间直接用 iperf3 拉的话，速度为 60Mbps 左右 ![](https://nova-moe-blog-assets.webp.se/pics/cloudflare-argo/iperf.png)
5.  Cloudflare 上缓存策略为 By Pass（不缓存）
6.  `without-tunnel.knat.network` 直接解析到赫尔辛基的服务器上并开启 Cloudflare CDN
7.  `tunnel.knat.network` 为在赫尔辛基的服务器使用 cloudflared 创建的隧道
8.  生成一个垃圾文件： `fallocate -l 1G CoronaVac.img`
9.  在日本的机器上 wget 拉取

我们直接看结论，如果直接连接的话，速度是这样的（平均速度：4.97 MB/s）：

```
○ wget https://without-tunnel.knat.network/CoronaVac.img
--2022-03-26 15:07:39--  https://without-tunnel.knat.network/CoronaVac.img
Resolving without-tunnel.knat.network (without-tunnel.knat.network)... 2606:4700:3037::6815:2403, 2606:4700:3033::ac43:b694, 172.67.182.148, ...
Connecting to without-tunnel.knat.network (without-tunnel.knat.network)|2606:4700:3037::6815:2403|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 1073741824 (1.0G) [application/octet-stream]
Saving to: ‘CoronaVac.img’

CoronaVac.img                           100%[==============================================================================>]   1.00G  5.37MB/s    in 3m 26s  

2022-03-26 15:11:06 (4.97 MB/s) - ‘CoronaVac.img’ saved [1073741824/1073741824]

```

如果走 Argo Tunnel 的话，速度是这样的（平均速度 13.6 MB/s）：

```
○ wget https://tunnel.knat.network/CoronaVac.img
--2022-03-26 15:12:39--  https://tunnel.knat.network/CoronaVac.img
Resolving tunnel.knat.network (tunnel.knat.network)... 2606:4700:3037::6815:2403, 2606:4700:3033::ac43:b694, 172.67.182.148, ...
Connecting to tunnel.knat.network (tunnel.knat.network)|2606:4700:3037::6815:2403|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 1073741824 (1.0G) [application/octet-stream]
Saving to: ‘CoronaVac.img’

CoronaVac.img                           100%[==============================================================================>]   1.00G  14.8MB/s    in 75s     

2022-03-26 15:13:56 (13.6 MB/s) - ‘CoronaVac.img’ saved [1073741824/1073741824]
```

速度提升了 4 倍。

![](https://nova-moe-blog-assets.webp.se/pics/cloudflare-argo/route.jpg)

emmm，而且这个服务甚至是免费的：

> In the past, Argo Tunnel has been priced based on bandwidth consumption as part of Argo Smart Routing, Cloudflare’s traffic acceleration feature. Starting today, we’re excited to announce that any organization can use the secure, outbound-only connection feature of the product at no cost.
> 
> ——A Boring Announcement: Free Tunnels for Everyone
> 
> [https://blog.cloudflare.com/tunnel-for-everyone/](https://blog.cloudflare.com/tunnel-for-everyone/)

## 一点额外的想法

我们的许多基础设施要么是美国公司提供的，要么就是大量使用了美国公司的产品，我们耳熟能详的（至少自称）开发者友好的云（Vultr/Digital Ocean/Linode），大型公有云（AWS/GCP/Azure），大型 CDN 服务商（Cloudflare，Akamai，Fastly），平时用的 Docker/Kubernetes，两大前端框架（React，Augular） 都是美国公司的产品。

以至于如果希望有服务能完全 Hosted in EU，Owned totally by EU company 都是一件比较困难的事情。

很多时候我会想：当这些公司推出了一个个令人兴奋的且难以被替代产品（比如 S3，比如 Lambda）的时候，我们又在做些什么？我们自己有什么产品是**真正在推动互联网某些领域**进步吗？哪怕只是一点很小的进步…

（或者说，我们还在忙着打造 xx 场景，为 xx 行业赋能，给 xx 提供抓手，助力出海东南亚？）

不禁又一次想到了 Cloudflare 博客上的一句话：

> At Cloudflare, our mission is to help build a better Internet.

希望这个记录和测试对部分人的部分业务有所启发，以上。