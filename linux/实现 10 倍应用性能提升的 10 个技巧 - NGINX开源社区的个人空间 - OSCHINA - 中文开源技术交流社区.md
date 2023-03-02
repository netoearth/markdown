> 原文作者：Floyd Smith of F5
> 
> 原文链接：[实现 10 倍应用性能提升的 10 个技巧](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fwww.nginx-cn.net%2Fblog%2F10-tips-for-10x-application-performance%2F)
> 
> 转载来源：NGINX 官方网站

___

Web 应用性能优化迫在眉睫。线上经济活动份额不断增长，发达世界的互联网经济已占经济总量的 5% 以上（请参见下文的[互联网统计数据来源](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fwww.nginx-cn.net%2Fblog%2F10-tips-for-10x-application-performance%2F%23resources)）。在这个始终在线、超级互联的现代世界，用户的期望已经今非昔比。如果您的网站没有立即做出响应，或者如果您的应用无法毫不延迟地运行，用户转身就会投向您的竞争对手。

例如，Amazon 近 10 年前的一项研究证明，页面加载时间每减少 100 毫秒，收入就会增加 1% —— 10 年前就已如此，更何况是现在。最近的另一项研究也强调指出，超过一半的受访网站所有者表示他们由于应用性能不佳失去了收入或客户。

一个网站到底多快才行？页面加载每延长 1 秒钟，就有大约 4% 的用户走掉。主要电商网站的首次交互时间从 1 秒到 3 秒不等，这个区间的转化率最高。Web 应用性能的重要性由此可见，并且可能还会与日俱增。

想要提高性能很容易，难的是怎么看到结果。为此，本文提出了帮您实现 10 倍网站性能提升的 10 个技巧。本文是一系列博文的开帖之作，详细介绍了如何借助一些久经考验的优化技术以及 NGINX 的一些支持提高应用性能。该系列还概括介绍了您可能因此而获得的安全性方面的改进。

## 技巧 1 —— 使用反向代理服务器加速并保护应用

如果您的 Web 应用在一台机器上运行，那要提升其性能非常简单：换一台更快的机器、多加几颗处理器、增加 RAM、换成高速磁盘阵列即可，这样新机器就可以更快地运行 WordPress 服务器、Node.js 应用和 Java 应用了。（如果您的应用需要访问数据库服务器，解决办法仍然很简单：找两台更快的机器，用更快的网络连起来就行了。）

问题是，机器的速度可能并不是问题所在。Web 应用通常运行很慢，因为计算机要在不同类型的任务之间切换：通过数千个连接与用户交互、访问磁盘文件、运行应用代码等等。应用服务器可能会因此而崩溃 —— 内存耗尽、将内存块交换到磁盘、让许多请求等待一个任务（比如磁盘 I/O）等。

除了升级硬件之外，您还可以采用一种完全不同的方法：添加反向代理服务器来卸载一些任务。[反向代理服务器](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fwww.nginx-cn.net%2Fresources%2Fglossary%2Freverse-proxy-server)位于运行应用的机器的前面，负责处理互联网流量。只有反向代理服务器直接连接到互联网，与应用服务器的通信是通过一个高速内网进行的。

反向代理服务器可以让应用服务器不必再等待用户与 Web 应用的交互，而是专注于构建页面，剩下的发送任务则交由反向代理来执行。由于无需等待客户端响应，应用服务器可以以接近优化基准测试所达到的速度运行。

添加反向代理服务器还可以增加 Web 服务器设置的灵活性。例如，如果执行既定任务的服务器过载了，您可以轻松添加一台同类型的服务器；如果一台服务器宕机，那么替换它也不难。

鉴于这种灵活性，反向代理服务器往往也是许多其他性能优化手段的先决条件，例如：

-   **负载均衡**（见[技巧 2](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fwww.nginx-cn.net%2Fblog%2F10-tips-for-10x-application-performance%2F%23tip2)） —— 在反向代理服务器上运行负载均衡器，以便在多个应用服务器之间均匀地共享流量。借助负载均衡器，您无需更改应用即可添加应用服务器。
-   **缓存静态文件**（见[技巧 3](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fwww.nginx-cn.net%2Fblog%2F10-tips-for-10x-application-performance%2F%23tip3)） —— 直接请求的文件（如图像文件或代码文件）可以存储在反向代理服务器上，并直接发送到客户端，这样不仅可以更快地提供资产，还能减轻应用服务器的负担，进而加快应用运行速度。
-   **保护站点安全** —— 反向代理服务器可以配置为高安全性并进行监控，以便快速识别和响应攻击，保护应用服务器的安全。

NGINX 软件一开始的设计定位就是具有上述附加功能的反向代理服务器。NGINX 采用了事件驱动型处理方法，因此比传统服务器效率更高。NGINX Plus 增添了更多高级反向代理功能，比如应用[健康检查](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fwww.nginx-cn.net%2Fproducts%2Fnginx%2Fload-balancing%2F%23health-checks)、特定请求的路由、高级缓存，并提供技术支持。

![](https://www.nginx-cn.net/wp-content/uploads/2015/10/traditional-server-vs-NGINX-worker-1.png)

## 技巧 2 —— 添加负载均衡器

添加[负载均衡器](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fwww.nginx-cn.net%2Fsolutions%2Fadc%2F)相对简单，但却能显著提升站点性能和安全性。您不需要升级核心 Web 服务器的硬件性能或规模，而是直接使用负载均衡器将流量分发到多台服务器。即使应用写得不好或者难以扩展，负载均衡器也可以在应用不做出任何更改的情况下改善用户体验。

负载均衡器首先是一个反向代理服务器（见[技巧 1](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fwww.nginx-cn.net%2Fblog%2F10-tips-for-10x-application-performance%2F%23tip1)），负责接收互联网流量并将请求转发给其他服务器。这里的关键在于负载均衡服务器可以支持两台或更多应用服务器，使用[您选择的算法](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fdocs.nginx.com%2Fnginx%2Fadmin-guide%2Fload-balancer%2Fhttp-load-balancer%2F%23method)在不同的服务器间分配请求。最简单的负载均衡方法是轮询，其中每个新请求都会被发送到列表中的下一台服务器。将请求发送到活动连接数最少的服务器也是常用的方法之一。NGINX Plus 还具有会话保持[功能](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fwww.nginx-cn.net%2Fblog%2Fload-balancing-with-nginx-plus%2F)，即将当前用户的多个会话保持在同一台服务器上。

负载均衡器可以避免出现一台服务器过载而其他服务器过闲的情况，从而显著提性能。此外，Web 服务器扩容也会变得简单，您可以添加相对便宜的服务器，并确保物尽其用。

可以进行负载均衡的协议包括 HTTP、HTTPS、SPDY、HTTP/2、WebSocket、[FastCGI](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fwww.digitalocean.com%2Fcommunity%2Ftutorials%2Funderstanding-and-implementing-fastcgi-proxying-in-nginx)、SCGI、uwsgi、memcached 以及其他应用类型，包括了基于 TCP 的应用和其他四层协议。您应该分析您的 Web 应用，确定它的类型以及性能短板是什么。

用于负载均衡的一台或若干台服务器还可以同时处理其他任务，例如 SSL 终止、按客户端来选择 HTTP/1.\_x\_和 HTTP/2 的使用、静态文件缓存等。

负载均衡是 NGINX 常用的功能之一。有关更多信息，请下载我们的电子书：[《选择软件负载均衡器的五个理由》](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fwww.nginx-cn.net%2Fresources%2Flibrary%2Ffive-reasons-choose-software-load-balancer%2F)；有关基本配置说明，请参见[《NGINX 和 NGINX Plus 负载均衡：第一部分》](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fwww.nginx-cn.net%2Fblog%2Fload-balancing-with-nginx-plus%2F)；有关完整文档，请参见[《NGINX Plus 管理员指南》](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fdocs.nginx.com%2Fnginx%2Fadmin-guide%2Fload-balancer%2F)。[NGINX Plus](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fwww.nginx-cn.net%2Fproducts%2Fnginx%2F) 是我们的商业产品，支持更多专业化的负载均衡功能，例如基于服务器响应时间的负载路由以及 Microsoft NTLM 协议负载均衡。

## 技巧 3 —— 缓存静态和动态内容

缓存可以通过更快地向客户端交付内容来改善 Web 应用的性能。缓存可以运用多种策略：预处理内容以便在需要时快速交付内容、将内容存储在更快的设备上、将内容存储在靠近客户端的地方等等，这些策略也可以组合运用。

缓存有两种类型：

-   **静态内容缓存** —— 图像文件 (JPEG、PNG) 和代码文件 (CSS、JavaScript) 等不常更改的文件可以存储在边缘服务器上，以便快速从内存或磁盘中检索。
-   **动态内容缓存** —— 许多 Web 应用都会为每个页面请求生成新的 HTML。通过短暂缓存一份生成的 HTML 副本，您可以大大减少必须要生成的页面总数，同时仍然提供足够满足需求的新鲜内容。

例如，假设一个页面每秒被查看 10 次，那么缓存 1 秒就代表这个页面有 90% 的请求来自缓存。如果您单独缓存各个静态内容，那么即使是全新生成的页面，也可能大部分来自缓存的内容。

缓存 Web 应用生成的内容主要会用到三种技术：

-   **将内容放到更靠近用户的地方** —— 离用户越近，传输时间越少。
-   **将内容放到更快的机器上** —— 机器越快，检索越快。
-   **将内容从过度使用的机器上移除** —— 在执行某个特定任务时，机器的运行速度有时会比基准性能慢得多，因为它们忙于其他任务。在不同的机器上缓存可以提高缓存资源和非缓存资源的性能，因为主机过载较少。

Web 应用缓存可以由内（Web 应用服务器）而外的实施。首先，缓存动态内容，以减少应用服务器的负载。其次，缓存静态内容（包括动态内容的临时副本），以进一步减少应用服务器的负载。然后，将缓存从应用服务器转移到更快和 / 或更靠近用户的机器上，从而减轻应用服务器的负担，缩短检索和传输时间。

优化的缓存可以极大地加快应用速度。对于很多网页来说，静态数据（例如大图像文件）往往占据一半以上的内容。如果不缓存，检索和传输此类数据可能会花好几秒钟，但是如果将数据缓存在本地，可能只花几分之一秒。

对于缓存的实践应用，我们可以看这样一个例子：NGINX 和 NGINX Plus 使用两个指令来[设置缓存](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fwww.nginx-cn.net%2Fblog%2Fnginx-caching-guide%2F)：`proxy_cache_path` 和 `proxy_cache`。您可以指定缓存的位置和大小、文件的最长缓存时间以及其他参数。您甚至还可以使用指令 `proxy_cache_use_stale`（相当受欢迎），指示缓存在本应提供新鲜内容的服务器太忙或停机时提供旧文件，毕竟对客户端来说，有总比没有强。这可能会极大地改善您的网站或应用的正常运行时间，给用户树立一种十分稳定的形象。

NGINX Plus 具有[高级缓存功能](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fwww.nginx-cn.net%2Fproducts%2Fnginx%2Fcaching%2F)，包括支持[缓存清除](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fnginx.org%2Fen%2Fdocs%2Fhttp%2Fngx_http_proxy_module.html%23proxy_cache_purge)、在[仪表盘](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fwww.nginx-cn.net%2Fproducts%2Fnginx%2Flive-activity-monitoring%2F)上以可视化的形式显示缓存状态（以监控实时活动）等。

有关 NGINX 缓存的更多信息，请参见[参考文档](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fnginx.org%2Fen%2Fdocs%2Fhttp%2Fngx_http_proxy_module.html%23proxy_cache)和[《NGINX Plus 管理员指南》](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fdocs.nginx.com%2Fnginx%2Fadmin-guide%2Fcontent-cache%2Fcontent-caching%2F)。

**注意**：缓存跨越了应用开发人员、资本投资决策人员和实时网络运营人员之间的组织界线。复杂的缓存策略（比如本文提到的这些）很好地体现了 [DevOps](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fwww.nginx-cn.net%2Fblog%2Fnetwork-vs-devops-how-to-manage-your-control-issues%2F) 的价值，其中应用开发人员、架构师和运营人员携手合作，朝着保障网站功能、响应时间、安全和业务成效（比如完成的交易或销售）的目标共同发力。

## 技巧 4 —— 压缩数据

压缩是一个重要的、潜在的性能加速器。图片（JPEG 和 PNG）、视频 (MPEG-4）和音乐 (MP3) 等文件都有着精心打造和非常高效的压缩标准，其中任何一个标准都可以将文件缩小一个数量级甚至更多。

HTML（包括纯文本和 HTML 标签）、CSS 和代码 (如 JavaScript）等文本数据经常在不压缩的情况下进行传输。压缩这些数据可能会对感知到的 Web 应用性能产生特别明显的影响，尤其是对速度缓慢或移动连接受限的客户端来说。

这是因为文本数据通常足以让用户与页面进行交互，而多媒体数据可能更多的体现支持性或锦上添花的作用。智能内容压缩通常可以将 HTML、Javascript、CSS 和其他文本内容的带宽需求减少 30% 及以上，并相应地减少加载时间。

如果使用 SSL，压缩可以减少必须进行 SSL 编码的数据量，从而抵消压缩数据所需的一些 CPU 时间。

压缩文本数据的方法有很多。例如，[技巧 6](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fwww.nginx-cn.net%2Fblog%2F10-tips-for-10x-application-performance%2F%23tip6) 中就提到了一种在 SPDY 和 HTTP/2 中压缩文本的新颖机制，该机制专门针对请求头数据进行了调整。还有一个文本压缩的例子，那就是您可以在 NGINX 中启用 [GZIP](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fdocs.nginx.com%2Fnginx%2Fadmin-guide%2Fweb-server%2Fcompression%2F) 压缩功能。在服务中[预压缩文本数据](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fnginx.org%2Fen%2Fdocs%2Fhttp%2Fngx_http_gzip_static_module.html)之后，您可以使用 `gzip_static` 指令直接提供压缩的 \*\*.gz\*\* 文件。

## 技巧 5 —— 优化 SSL/TLS

安全套接字层 ([SSL](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fwww.digicert.com%2Fssl.htm)) 协议及其升级版传输层安全 (TLS) 协议在网站中的应用越来越广泛。SSL/TLS 通过加密从源服务器传输给用户的数据来提高站点安全性。Google 现在将 SSL/TLS 的使用作为搜索引擎排名的加分项，这在某种程度上加速了这一技术的普及。

虽然 SSL/TLS 变得越来越受欢迎，但它们带来的相关性能问题也困扰着许多网站。SSL/TLS 拖慢网站的原因有两个：

1.  每当打开新连接时，建立加密密钥都需要进行初始握手。浏览器使用 HTTP/1.\_x\_时对每台服务器建立多个连接的方式加剧了这个问题。
2.  服务器端加密数据和客户端解密数据也会带来持续的开销。

为了鼓励人们使用 SSL/TLS，HTTP/2 和 SPDY 协议（见[技巧 6](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fwww.nginx-cn.net%2Fblog%2F10-tips-for-10x-application-performance%2F%23tip6)）应运而生，这样浏览器只需为每次会话建立一个连接便可，同时也搞定了 SSL 的一大开销来源。但是，通过 SSL/TLS 交付的应用还有很大的性能提升空间。

SSL/TLS 的优化机制因 Web 服务器而异。例如，NGINX 使用在标准商用硬件上运行的 [OpenSSL](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fwww.openssl.org%2F) 来提供与专用硬件解决方案相媲美的性能。NGINX 的 [SSL 性能优化](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fwww.nginx-cn.net%2Fblog%2Fnginx-ssl-performance%2F)是有据可查的，可以将执行 SSL/TLS 加密和解密的时间及 CPU 消耗降到最低。

此外，[本博文](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fwww.nginx-cn.net%2Fblog%2Fimprove-seo-https-nginx%2F)还详细介绍了提升 SSL/TLS 性能的其他方式，简而言之，这些技术是：

-   **会话缓存** —— 使用 [`ssl_session_cache`](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fnginx.org%2Fen%2Fdocs%2Fhttp%2Fngx_http_ssl_module.html%23ssl_session_cache)指令缓存保护每个 SSL/STL 新连接时使用的参数。
-   **Session Ticket 或 ID**—— 将特定 SSL/TLS 会话的信息存储在 ticket 或 ID 中，这样无需再次握手即可顺利重用连接。
-   **OCSP Stapling**—— 通过缓存 SSL/TLS 证书信息缩短握手时间。

NGINX 和 NGINX PLUS 支持 SSL/TLS 终止，即在处理客户端流量加密和解密的同时，与其他服务器进行明文通信。如欲设置 NGINX 或 NGINX Plus 的 SSL/TLS 终止功能，请参见 [HTTPS 连接](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fdocs.nginx.com%2Fnginx%2Fadmin-guide%2Fsecurity-controls%2Fterminating-ssl-http%2F)和 [TCP 加密连接](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fdocs.nginx.com%2Fnginx%2Fadmin-guide%2Fsecurity-controls%2Fterminating-ssl-tcp%2F)的说明。

## 技巧 6 —— 实施 HTTP/2 或 SPDY

对于已经使用 SSL/TLS 的站点，HTTP/2 和 SPDY 可能会让性能更上一层楼，因为一个连接仅需要一次握手。对于尚未使用 SSL/TLS 的站点，HTTP/2 和 SPDY 从响应性的角度来看，转向 SSL/TLS（通常会降低性能）是一种洗礼。

Google 在 2012 年推出了 SPDY，致力于在 HTTP/1.\_x\_之上实现更快的速度。HTTP/2 是 IETF 最近批准的基于 SPDY 的标准。SPDY 得到了广泛的支持，但很快就会被 HTTP/2 取代。

SPDY 和 HTTP/2 的关键功能是使用单个连接而非多个连接。单个连接是多路复用的，因此它可以同时承载多个请求和响应。

通过充分利用一个连接，这些协议避免了设置和管理多个连接（浏览器实施 HTTP/1.\_x\_的必要条件）的开销。使用单个连接对 SSL 尤为有用，因为这可以将 SSL/TLS 建立安全连接所需的握手时间降到最少。

SPDY 协议要求使用 SSL/TLS；HTTP/2 没有对此作出正式要求，但目前所有支持 HTTP/2 的浏览器都只会在启用 SSL/TLS 的情况下使用它。也就是说，只有当网站使用 SSL 且其服务器接受 HTTP/2 流量时，支持 HTTP/2 的浏览器才能使用 HTTP/2，否则浏览器将通过 HTTP/1.\_x\_通信。

实施 SPDY 或 HTTP/2 后，就用不到域分片 (domain sharding)、资源合并、图像拼合 (image spriting) 等典型的 HTTP 性能优化技术了。这些变动也可以简化代码和部署的管理。如欲详细了解 HTTP/2 带来的变化，请参阅我们的白皮书：[《面向 Web 应用开发人员的 HTTP/2》](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fwww.nginx-cn.net%2Fresources%2Fdatasheet%2Fdatasheet-nginx-http2-whitepaper%2F)。

![NGINX Supports SPDY and HTTP/2 for increased web application performance](https://www.nginx-cn.net/wp-content/uploads/2015/10/http2-gateway-1.png)

在对这些协议的支持方面，NGINX 从很早就开始支持 SPDY 了，并且现在使用 SPDY 的[大多数站点](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fw3techs.com%2Fblog%2Fentry%2F25_percent_of_the_web_runs_nginx_including_46_6_percent_of_the_top_10000_sites)都部署了 NGINX。NGINX 也是[率先支持](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fwww.nginx-cn.net%2Fblog%2Fhow-nginx-plans-to-support-http2%2F) HTTP/2 的开拓者之一，从 2015 年 9 月起，NGINX 开源版和 NGINX Plus 就引入了对 HTTP/2 的[支持](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fwww.nginx-cn.net%2Fblog%2Fnginx-plus-r7-released%2F)。

NGINX 希望随着时间的推移，大多数站点都能完全启用 SSL 并迁移到 HTTP/2。这不仅有助于提高安全性，而且随着新型优化技术的发现和实施，性能更高的代码也将得到进一步简化。

## 技巧 7 —— 更新软件版本

提高应用性能的一种简单方法是根据稳定性和性能为您的软件栈选择组件。此外，由于高质量组件的开发人员可能会不断增强性能和修复 bug，使用最新的稳定版软件是非常划算的。新版本会受到开发人员和用户社区的更多关注，同时也会利用新的编译器优化技术，包括对新硬件的调优。

从兼容性和性能角度来看，新发布的稳定版本通常也比旧版本更胜一筹。坚持软件更新还可以让您在调优、bug 修复和安全警报方面保持优势。

使用旧版本会妨碍新功能使用。例如，上文提到的 HTTP/2 目前要求使用 OpenSSL 1.0.1.。从 2016 年年中开始，HTTP/2 会要求使用 OpenSSL 1.0.2（于 2015 年 1 月发布）。

NGINX 用户可以先从升级到最新版的 [NGINX](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fnginx.org%2Fen%2Fdownload.html) 或 [NGINX Plus](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fwww.nginx-cn.net%2Fproducts%2Fnginx%2F) 入手，它们具备套接字分片 (socket sharding) 和线程池（见[技巧 9](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fwww.nginx-cn.net%2Fblog%2F10-tips-for-10x-application-performance%2F%23web-server-tuning)）等新功能，并且均会持续进行性能调优。然后再深入检查一下堆栈中的软件，尽量迁移到最新版本。

## 技巧 8 —— 优化 Linux 性能

Linux 是当今大多数 Web 服务器的底层操作系统。作为基础架构的基石，Linux 蕴藏着重大的调优潜能。默认情况下，许多 Linux 系统的优化都比较保守，以占用少量资源、满足常规桌面工作负载为目标。这意味着在 Web 应用用例中，要想达到最高性能，调优是必不可少的。

Linux 优化因 Web 服务器而异。以 NGINX 为例，您可以重点考虑从以下几个方面给 Linux 提速：

-   **积压队列 (Backlog Queue)**—— 如果有一些连接得不到处理，可以考虑增大 `net.core.somaxconn` 的值，即等待 NGINX 处理的最大连接数。如果当前连接限制数太小，就会收到错误消息，您可以逐渐增加此参数，直到错误消息不再出现。
-   **文件描述符** —— NGINX 最多对每个连接使用两个文件描述符。如果系统服务于很多连接，则可能需要增大 `sys.fs.file_max`（整个系统范围内对文件描述符的限制）以及 `nofile`（用户文件描述符限制），以支持增加的负载。
-   **临时端口** —— 当用作代理时，NGINX 会为每个上游服务器创建临时端口。您可以使用 `net.ipv4.ip_local_port_range` 拉宽端口的值阈，以增加可用端口的数量。您还可以使用 `net.ipv4.tcp_fin_timeout` 减少重用非活动端口之前的超时，从而加快周转。

对于 NGINX，请参见[《NGINX 性能调优指南》](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fwww.nginx-cn.net%2Fblog%2Ftuning-nginx%2F)，了解如何优化您的 Linux 系统，使其轻松处理大量网络流量！

## 技巧 9 —— 优化 Web 服务器的性能

无论您使用哪种 Web 服务器，都要对其进行调优才能提升 Web 应用的性能。以下建议通常适用于任何 Web 服务器，其中包含一些针对 NGINX 的特定设置。主要的优化技术有：

-   **访问日志** —— 不要立即将每个请求的日志写到磁盘，可以先将条目放到内存缓冲区，然后再批量写入。对于 NGINX，将 `buffer=_size_`参数添加到 `access_log` 指令中，等内存缓冲区写满后再把日志写入磁盘。如果添加 `flush=_time_`参数，缓冲区的内容也会在指定的时间过后写入磁盘。
-   **缓冲** —— 缓冲用于在内存中保存一部分响应，直到缓冲区填满为止，可以实现与客户端更高效的通信。不适合写入内存的响应会被写入磁盘，导致性能降低。[启用](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fnginx.org%2Fen%2Fdocs%2Fhttp%2Fngx_http_proxy_module.html%23proxy_buffering) NGINX 缓冲功能后，可以使用 `proxy_buffer_size` 和 `proxy_buffers` 指令进行管理。
-   **客户端 keepalive 连接** —— Keepalive 连接可以减少开销，尤其是在使用 SSL/TLS 时。对于 NGINX，您可以增加客户端通过现有连接发出的 `keepalive_requests` 的最大数量（默认值为 100），同时增大 `keepalive_timeout` 的值，让 keepalive 连接保持更长时间的打开状态，从而加快后续请求的速度。
-   **Upstream keepalive 连接** —— Upstream 连接，即与应用服务器、数据库服务器等服务器的连接，同样可以从 keepalive 连接的设置中获得好处。对于 Upstream 连接，您可以增加 `keepalive`，即为每个 worker 进程保持打开状态的空闲 keepalive 连接的数量。这可以增加对连接的重复使用，减少打开全新连接的需求。有关更多信息，请参阅我们的博文：[《HTTP Keepalive 连接和 Web 性能》](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fwww.nginx-cn.net%2Fblog%2Fhttp-keepalives-and-web-performance%2F)。
-   **限流** —— 限制客户端使用的资源可以提升性能和安全性。对于 NGINX，`limit_conn` 和 `limit_conn_zone` 指令可以限制来自既定源的连接数，`limit_rate` 则可以限制带宽。这些设置可以防止合法用户 “侵吞” 资源，也可以帮助防止攻击。`limit_req` 和 `limit_req_zone` 指令能够限制客户端请求。对于到上游服务器的连接，您可以在 upstream 配置块中使用 `server` 指令的 `max_conns` 参数，以限制到上游服务器的连接，防止过载。关联的 `queue` 指令会创建一个队列，在达到 `max_conns` 限制后在指定的时间长度内保存指定数量的请求。
-   **Worker 进程** —— Worker 进程负责处理请求。NGINX 采用基于事件的模型和依赖操作系统的机制在 worker 进程之间高效地分发请求。建议将 `worker_processes` 的值设置为每个 CPU 一个。如果需要，大多数系统都支持提高 `worker_connections` 的最大数值（默认为 512），可以通过试验找出最适合您的系统的值。
-   **套接字分片 (Socket Sharding)**—— 套接字监听器通常会向所有 worker 进程分发新连接。套接字分片 (Socket Sharding) 为每个 worker 进程创建一个套接字监听器，由内核在套接字监听器可用时为其指定连接。这可以减少锁争用并提高多核系统的性能。要启用[套接字分片](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fwww.nginx-cn.net%2Fblog%2Fsocket-sharding-nginx-release-1-9-1%2F)功能，请在 `listen` 指令中添加 `reuseport` 参数。
-   **线程池** —— 任何计算机进程都可能会被一个缓慢的操作所拖累。对于 Web 服务器软件，磁盘访问可能阻碍很多较快的操作，例如内存中的计算或信息复制。使用线程池时，慢操作会被分配给一组单独的任务，而主处理循环仍然继续运行较快的操作。磁盘操作完成后，结果会返回到主处理循环。NGINX 将 `read()` 系统调用和 `sendfile()` 这两个操作卸载到了[线程池](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fwww.nginx-cn.net%2Fblog%2Fthread-pools-boost-performance-9x%2F)。

![Thread pools help increase application performance by assigning a slow operation to a separate set of tasks](https://www.nginx-cn.net/wp-content/uploads/2015/10/thread-pools-worker-process-event-cycle-1.png)

**小贴士**：如果要更改任何操作系统或支持服务的设置，请一次先更改一项设置，然后测试性能。如果更改引发了问题或者不会加快网站运行速度，那就再改回去。

有关 NGINX Web 服务器调优的更多信息，请参见我们的博文：[《优化 NGINX 的性能》](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fwww.nginx-cn.net%2Fblog%2Ftuning-nginx%2F)。

## 技巧 10 —— 监控实时活动，以解决问题和发现瓶颈

高效的应用开发和交付方法的关键在于实时密切监控应用的真实性能。您必须要监控特定设备和 Web 基础架构中的活动。

监控站点活动多数时候是被动的 —— 它只告诉您发生了什么，至于如何发现和解决问题，则是我们自己的事情。

监控可以捕获以下几种不同的问题：

-   服务器停机。
-   服务器运行不稳定，导致连接中断。
-   服务器出现大量缓存未命中问题。
-   服务器没有发送正确的内容。

全局应用性能监控工具（例如 New Relic 或 Dynatrace）可帮助您远程监控页面加载时间，NGINX 则可帮助您监控应用交付这一端。应用性能数据可以显示，您的优化在什么情况下对用户产生了真正的影响，以及什么情况下要为基础架构扩容以满足流量需求。

为了帮助快速发现和解决问题，NGINX Plus 增加了[应用感知型健康检查](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fwww.nginx-cn.net%2Fproducts%2Fnginx%2Fload-balancing%2F%23health-checks%2F)功能，这类综合事务（synthetic transaction）会定期重复，在发现问题后向您发出告警。NGINX Plus 还具有[会话耗尽](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fnginx.org%2Fen%2Fdocs%2Fhttp%2Fngx_http_upstream_module.html%23drain)功能，能够在现有任务完成时停止新连接，其慢启动特性还允许恢复的服务器在负载均衡组内逐渐升到正常水平。只要使用得当，健康检查就会帮助您及时发现问题，以免对用户体验造成重大影响，会话耗尽和慢启动则允许您在不降低感知到的性能或正常运行时间的情况下替换服务器。下图展示了 NGINX Plus 内置了 Web 基础架构的[实时活动监控](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fwww.nginx-cn.net%2Fproducts%2Fnginx%2Flive-activity-monitoring%2F)仪表盘，涵盖了服务器、TCP 连接和缓存。

![The NGINX Plus dashboard provides detailed statistics for monitoring and managing your infrastructure](https://www.nginx-cn.net/wp-content/uploads/2015/10/dashboard-nginx-plus-r9-1.png)

## 结论 —— 实现 10 倍的性能提升

不同的 Web 应用有着不同的性能提升空间，实际的效果取决于您的预算、投入的时间以及当前实施的差距。那么，如何才能让您的应用实现 10 倍的性能提升呢？

为了帮助您理解每种优化技术的潜在影响，下面分条列出了文中所述的各项技巧可能带来的一些改进，希望大家各取所需：

-   **反向代理服务器及负载均衡** —— 没有负载均衡或者负载均衡不好可能会导致性能极低。添加反向代理服务器（例如 NGINX）可以防止 Web 应用在内存和磁盘之间来回奔波。负载均衡可以将处理任务从过载的服务器转移到可用的服务器，也可以简化扩展。这些改变能够显著提升性能，与原有部署方式最差的时候相比，10 倍性能提升是很轻松的事，即使不到 10 倍也在总体上有了质的飞跃。
-   **缓存动态和静态内容** —— 如果您的 Web 服务器同时又超负荷地充当应用服务器，那么仅缓存动态内容就可以在高峰期达到 10 倍的性能提升，缓存静态文件也可以实现几倍的性能提升。
-   **压缩数据** —— 使用 JPEG（照片）、PNG（图形）、MPEG-4（电影）和 MP3（音乐文件）等媒体文件压缩格式可以显著提升性能。如果这些方式都用上了，那么压缩文本数据（代码和 HTML）可以将初始页面加载速度提高两倍。
-   **优化 SSL/TLS**—— 安全握手对性能的影响很大，因此对其进行优化可能会让初次响应加快两倍，尤其是对文本内容较多的网站来说。如果是在 SSL/TLS 下优化媒体文件传输，性能优化的效果可能不是很大。
-   **实施 HTTP/2 和 SPDY**—— 当与 SSL/TLS 一起使用时，这些协议可能会对网站整体性能带来增量改进。
-   **Linux 和 Web 服务器软件（如 NGINX）调优** —— 优化缓冲、使用 keepalive 连接、将耗时的任务卸载到独立的线程池等修复措施可以显著提升性能，例如线程池可以将磁盘操作密集型任务的速度提高[大约一个数量级](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fwww.nginx-cn.net%2Fblog%2Fthread-pools-boost-performance-9x%2F)。

我们希望大家多多尝试这些技术，也期待听到大家在改进应用性能方面的心得。

___

## **更多资源**

想要更及时全面地获取 NGINX 相关的技术干货、互动问答、系列课程、活动资源？

请前往 NGINX 开源社区：

-   官网：[https://www.nginx.org.cn/](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fwww.nginx.org.cn%2F)
-   微信公众号：[https://mp.weixin.qq.com/s/XVE5yvDbmJtpV2alsIFwJg](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fmp.weixin.qq.com%2Fs%2FXVE5yvDbmJtpV2alsIFwJg)
-   微信群：[https://www.nginx.org.cn/static/pc/images/homePage/QR-code.png?v=1621313354](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fwww.nginx.org.cn%2Fstatic%2Fpc%2Fimages%2FhomePage%2FQR-code.png%3Fv%3D1621313354)
-   B 站：[https://space.bilibili.com/6283](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fspace.bilibili.com%2F6283)