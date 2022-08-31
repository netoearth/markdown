众所周知，DNS 的作用与电话簿类似，将人类可读的域名映射到机器可读 IP 地址、使人更方便地访问互联网。DNS 是非常重要的互联网基础设施，对于改善上网冲浪的体验中的重要程度不容小觑。

## 基础知识储备

如果没有相关的知识储备，你在阅读本文时可能会难以理解其中的一些观点。你可以阅读我之前写过的 [数篇关于 DNS 的文章](https://blog.skk.moe/tags/DNS/)，本文不会再做详细赘述。

-   [权威 DNS 自身的解析记录和 Glue Record 的关系](https://blog.skk.moe/post/authoritative-dns-record-glue/)。
-   [浅谈在代理环境中的 DNS 解析行为](https://blog.skk.moe/post/what-happend-to-dns-in-proxy/)。
-   [简析代理客户端导致的远端服务器流媒体 DNS 解锁失效的可能原因](https://blog.skk.moe/post/how-proxy-client-cause-netflix-unlock-to-fail/)。

## 避免不必要的 DNS 解析

如果想要通过优化 DNS 来改善自己的网络体验，第一步其实是避免不必要的 DNS 解析。

### 使用 Fake IP 避免本机 DNS 解析

正如我在 [浅谈在代理环境中的 DNS 解析行为](https://blog.skk.moe/post/what-happend-to-dns-in-proxy/) 一文中所提到的，对于不支持设置 SOCKS/HTTP(S) 代理的软件，Surge/Clash 等软件一般选择通过 TUN/TAP 或 转发 redir 透明网关接管网络请求，从而拿到原始的 TCP/IP 连接。

在 POSIX 规范下，执行网络请求需要先通过 `gethostbyname`、`getremoteaddr` 等操作系统提供的方法进行 DNS 解析，获取到 IP 地址以后发起连接；如果 DNS 解析不成功，网络请求就无从谈起了。因此绝大部分依赖 TUN 和 TAP 的某些软件都会接管系统 DNS 解析。接管 DNS 解析后随之而来的便是一系列问题：

-   DNS 污染：由于特殊的网络环境，通过你本机直接进行 DNS 解析得到的结果可能不可靠。
-   CDN 优化：如果要访问的目标网站使用了 CDN，最理想的结果是 `距离代理服务器最近的 CDN 节点 - 代理服务器 - 你`。如果你通过本机直接进行 DNS 解析，获取到的 IP 地址可能并不是距离你 远端代理服务器 最近的 CDN 节点。

由于常见的某些协议都运行在 Layer 4 上、支持封装域名；因此 Surge/Clash 等软件在转发流量时，都是封装目标域名，而不是目标域名在本机解析到的 IP 地址，从而规避 DNS 污染和实现 CDN 优化。  
如果软件一旦决定将某个域名转发给远端代理服务器，远端代理服务器也需要对拿到的域名进行一次解析，在本机解析的 IP 地址其实没有起任何作用，白白浪费一个 RTT。于是 2001 年四月，IETF 通过了 [RFC3089](https://www.rfc-editor.org/rfc/rfc3089)，描述了一种网关通过接管 DNS、返回 Fake IP 来建立 TCP/IP 链接的方法。简单的流程如下：

-   代理网关接管本机的 DNS 解析
-   一个软件意图对一个域名发起网络请求，于是先通过 DNS 解析获取域名对应的 IP
-   代理网关收到 DNS 解析请求后，不做任何 DNS 解析，而是直接返回一个保留 IP 地址（Fake IP）
-   发起网络请求的软件获取到 Fake IP 后，试图以 Fake IP 为目标发起网络请求
-   代理网关截获网络请求，通过目标的 Fake IP 反推出目标域名
-   代理网关将流量和目标域名使用某种协议重新封装后、转发给远端代理服务器

不过在日常使用中，即使有了 Fake IP 也不能完全避免本机进行 DNS 解析。Surge/Clash 使用 Fake IP 后，当且只当以下两种情况时 会在本机进行 DNS 解析：

-   目标域名需要使用 DIRECT 策略（即直连）、此时 Surge/Clash 需要得到真实的目标 IP、不通过代理服务器直接发起连接
-   Surge/Clash 遇到了基于 IP 分流的策略（如 IP-CIDR、IP-ASN、GEOIP、LAN 等），此时 Surge/Clash 需要得到一个 IP 用于匹配分流

> 关于上述两种情况的细节、需要本机 DNS 解析的原因，我在 [浅谈在代理环境中的 DNS 解析行为](https://blog.skk.moe/post/what-happend-to-dns-in-proxy/) 一文中已经介绍过，此处不再赘述。

也就是说，如果 Surge 和 Clash 能够匹配到了一条域名规则、指示网络请求需要被转发给远端代理服务器，Surge 和 Clash 便不会在本地进行 DNS 解析。因此在编写 Surge 和 Clash（以及同类软件 Shadowrocket、Quantumult (X)、Surfboard 等）的规则时，将 IP 相关规则（IP-CIDR、IP-ASN、GEOIP 等）放在其余的规则（DOMAIN、DST-PORT、SRC-PORT、PROTOCOL、URL-REGEX）的后面；除此以外，需要代理的域名的规则组越完善、Surge / Clash 匹配到 IP 类规则的概率也就越低，需要本机 DNS 解析的次数也就越少。

### 使用域名分流规则直接拦截广告

不论是去广告 Hosts 还是 AdGuardHome 等解决方案，本质上都是在 DNS 解析环节拦截域名；由于 DNS 的局限性，这类解决方案只能拦截完整的广告域名，不能拦截某一个域名下的具体 URL，因此要么不够强力、要么误杀严重，不能取代专业的浏览器去广告插件（如 ADBlock Plus、AdGuard for Chrome）和去广告软件（如 AdGuard for Android）。

除此以外，由于 Surge/Clash 等支持使用域名规则进行分流的软件也都提供了对 REJECT 策略的支持（Surge 还额外支持两种特殊的 REJECT 策略：不回复任何数据包、任由链接自行超时的 REJECT-DROP，和返回空白 1 像素 GIF 图像文件的 TINY-GIF）。因此，我们可以编写域名规则拦截广告请求、彻底杜绝被拦截域名的 DNS 解析、加快阻断。

> 目前 Surge 支持 `DOMAIN-SET` 格式，可以在一个配置文件里记录数十万条域名，而不会内存泄漏或崩溃；Clash、Quantumult (X) 的域名规则可能不支持上万级别数量的域名、强行导入可能导致内存泄漏或 Panic；Surfboard 虽然完全兼容 Surge 的 `DOMAIN-SET` 格式，但是可能存在优化程度不够、达不到 Surge 同等速度和效率。  
> 建议先对有关软件进行测试，如果不能很好的处理大量域名规则的，仍然可以使用 AdGuardHome 作为去广告的替代。

我使用的去广告域名规则可以在我 [个人自用 Surge 规则](https://github.com/SukkaW/Surge) GitHub 仓库中查看，我在其中也开源了我编写的自动抓取、解析 uBlock Origin、AdGuard、EasyList 等多个数据来源、剔除包含 URL 规则（仅保留纯域名规则）、汇总去重合并的工具 [`Build/build-reject-domainset.js`](https://github.com/SukkaW/Surge/blob/master/Build/build-reject-domainset.js) 。

> 我的去广告规则基本可以保证没有误杀；即使规则存在误杀，所有 AdGuard、uBlock Origin 的用户也一定都会受到影响，遇到这种情况只需要直接向 AdGuard、uBlock Origin 的规则维护者进行反馈。不过为了避免不必要的麻烦，我开源的 Surge 规则组都属于「个人自用」、**不接受任何反馈意见**。

## 中场休息：递归 DNS 是怎么知道哪个 CDN 节点距离我最近的？

先暂时抛开对我的 DNS 配置的介绍，简单谈谈递归 DNS（Local DNS）是如何实现「CDN 优化」的。

假设你的宽带 IP 是 `114.5.1.4`，你现在试图访问一个使用了阿里云 CDN 的网站 `alicdn.example.skk.moe`、接入 CDN 的方式是 CNAME 到 `alicdn.example.skk.moe.w.alikunlun.com`。

> 注意，为了节省篇幅，在接下来的描述中，我刻意省去了 Local DNS 向 Root DNS 查询 `moe` 和 `skk.moe` 的权威 DNS 的过程。如果想要了解完整的 DNS 查询过程，可以阅读由 Cloudflare 编写的 [What is DNS? | How DNS works](https://www.cloudflare.com/learning/dns/what-is-dns/)。

### 运营商 DNS

一开始，你使用的是由运营商下发给你的运营商 DNS，假设运营商的递归 DNS 的 IP 是 `1.2.3.4`，于是：

-   你向 `1.2.3.4` 发起 DNS 查询：请问 `alicdn.example.skk.moe` 的解析结果是什么？
-   `1.2.3.4` 问 `skk.moe` 的权威 DNS 查询：`alicdn.example.skk.moe` 的解析结果是什么？
-   `skk.moe` 的权威 DNS 告诉 `1.2.3.4`：`alicdn.example.skk.moe` 用 CNAME 指向了 `alicdn.example.skk.moe.w.alikunlun.com`。
-   于是 `1.2.3.4` 把结果返回给你：`alicdn.example.skk.moe` 用 CNAME 指向了 `alicdn.example.skk.moe.w.alikunlun.com`
-   你问 `1.2.3.4`：请问 `alicdn.example.skk.moe.w.alikunlun.com` 的解析结果是什么？
-   `1.2.3.4` 问 `alikunlun.com` 的权威 DNS：`alicdn.example.skk.moe.w.alikunlun.com` 的解析结果是什么？
-   `alikunlun.com` 是阿里云 CDN 的域名，阿里的权威 DNS 开始找：地理位置最接近 `1.2.3.4` 的 CDN 节点是哪些？有 `19.19.8.10`。
-   `alikunlun.com` 告诉 `1.2.3.4`：`alicdn.example.skk.moe.w.alikunlun.com` 解析到了 `19.19.8.10`。
-   `1.2.3.4` 把结果返回给你：`alicdn.example.skk.moe.w.alikunlun.com` 解析到了 `19.19.8.10`。

![how-isp-dns-work-with-cdn.png](https://img10.360buyimg.com/ddimg/jfs/t1/146488/34/28315/17213/62e2c4baEbc752c5b/30a1f5ab369dccf9.png)

因此，阿里云 CDN 并不是返回「最适合你」的 CDN 节点 IP，而是返回「最适合这个运营商 DNS」的 CDN 节点 IP。只不过由于运营商 DNS 一般都距离你很近，所以也可以把这个结果当作是「最适合你」的 CDN 节点 IP。

### 公共 DNS

再后来，你听说南京信风提供的公共递归 DNS `114.114.114.114` 很有名，于是你手动将你的 DNS 设置为 `114.114.114.114`。

`114.114.114.114` 是一个 Anycast IP，即一个 IP 能够对应不同区域、不同 ISP 的多个数据中心、多台服务器。截止到本文写就，南京信风仅在江苏南京的 电信、联通、移动 和 美国伊利诺伊州芝加哥的 Cogent 广播了 `114.114.114.114`，也就是说 `114.114.114.114` 仅对应到了这两地的服务器节点。一般的，国内的用户连接 `114.114.114.114`，都是访问位于江苏南京的节点。

你通过 `114.114.114.114` 和通过运营商 DNS 获取 `alicdn.example.skk.moe` 的解析结果的过程大同小异，区别在于这一次，`alikunlun.com` 并不会试图返回最适合 `1.2.3.4`（你的运营商 DNS）的 CDN 节点，而是试图返回最适合江苏南京的 CDN 节点。

由于中国复杂的互联网环境和封闭的网络基础设施建设，很难将 Anycast 覆盖到中国 30 余省市的十数个主流运营商、如上文所说，南京信风的 `114.114.114.114` 在国内甚至只有一个节点、只能覆盖三个运营商；即使是腾讯云 DNSPod 和阿里的公共 DNS，在国内也只有不到 10 个节点。

![how-isp-dns-work-with-public-dns.png](https://img10.360buyimg.com/ddimg/jfs/t1/83233/4/21065/31514/62e2cdbbE119b1be9/a5c60eed5f99b2ab.png)

因此，在国内使用公共 DNS，不仅不能够优化 CDN，反而还会劣化 CDN。也是因为同一个原因，运营商要劫持你的 DNS 查询，避免因为你的 DNS 设置不当、反而投诉运营商的网络速度慢、差。

与此同时，公共 DNS 为了解决少数 Anycast 节点无法对应到全国 30 余省市数十个运营商的问题，另辟蹊径想出了一个新的方案：

### 多出口 IP 的公共 DNS

有的公共 DNS 除了在全国设立 Anycast 节点、负责接收 DNS 查询以外，还在全国 30 余省市均部署了额外的服务器（称作「DNS 出口服务器」）。这些 DNS 出口服务器不会直接接收来自终端用户的 DNS 查询，而是 Anycast 节点接收到终端用户的 DNS 查询后，转交给 DNS 出口服务器再进行解析：

-   你（`114.5.1.4`）向 `233.5.5.5` 发起查询：请问 `alicdn.example.skk.moe` 的解析结果是什么？
-   `223.5.5.5` 的众多 Anycast 节点中的一个收到了你的查询、开始寻找：我在全国部署的上百个 DNS 出口服务器中，哪一个是距离 `114.5.1.4` 最近的？
-   `223.5.5.5` 将 DNS 查询转交给距离你最近的 DNS 出口服务器（称作「DNS 出口 A」）。
-   DNS 出口 A 问 `alikunlun.com` 的权威 DNS：`alicdn.example.skk.moe.w.alikunlun.com` 的解析结果是什么？
-   阿里云 CDN 开始找：地理位置最接近 A 的 CDN 节点都是哪些？
-   `alikunlun.com` 告诉 A：`alicdn.example.skk.moe.w.alikunlun.com` 解析到了这些 IP。
-   A 告诉 `223.5.5.5`：`alicdn.example.skk.moe.w.alikunlun.com` 解析到了这些 IP。
-   `223.5.5.5` 告诉你：`alicdn.example.skk.moe.w.alikunlun.com` 解析到了这些 IP。

![i-have-my-unique-dns-setup/how-multi-exit-pulic-dns-work.png](https://img10.360buyimg.com/ddimg/jfs/t1/153096/20/25296/50645/62e2d9d1Ecd4952d1/04b7653191b3064f.png)

这样，虽然接收到你查询的公共 DNS 的 Anycast 节点不一定距离你非常非常近，但是最终向权威 DNS 发起查询的，却是距离你尽可能近的「DNS 出口服务器」，因此得到的「最适合 DNS 出口服务器」的 CDN 节点 IP、也是最适合你的。

### 支持 EDNS Client Subnet 的公共 DNS

为了解决权威 DNS 难以根据终端用户的真实 IP 返回最适合用户的 CDN 节点的问题，IETF 通过了 [RFC7871](https://datatracker.ietf.org/doc/html/rfc7871)，即 EDNS Client Subnet（ECS）。RFC7871 定义了在 DNS 查询时，用户可以指定一个 IP 网段，权威 DNS 可以据此返回最适合这个 IP 网段的 CDN 节点：

-   你（`114.5.1.4`）问支持 ECS 的公共 DNS `119.29.29.29`：我是 `114.5.1.0/24`，请问 `alicdn.example.skk.moe.w.alikunlun.com` 的解析结果是什么？
-   `119.29.29.29` 问 `alikunlun.com` 的权威 DNS：`114.5.1.0/24` 在问 `alicdn.example.skk.moe.w.alikunlun.com` 的解析结果是什么？
-   阿里云 CDN 开始找：地理位置最接近 `114.5.1.0/24` 的 CDN 节点都是哪些？
-   `alikunlun.com` 告诉 `119.29.29.29`：`alicdn.example.skk.moe.w.alikunlun.com` 解析到了这些 IP。
-   `1.2.3.4` 把结果返回给你：`alicdn.example.skk.moe.w.alikunlun.com` 解析到了这些 IP。

![how-isp-dns-work-with-ecs.png](https://img10.360buyimg.com/ddimg/jfs/t1/133439/36/24835/36650/62e2cfcbE677efac1/ae1aafe37d118d01.png)

虽然 ECS 解决了权威 DNS 无法获取用户真实 IP 的问题，但是在实践中仍然存在一些困难：

-   使用 ECS 有可能泄漏用户的隐私信息，一些公共 DNS（如 Cloudflare 的 `1.1.1.1` 和 `1.0.0.1`）因此拒绝提供 ECS 支持。
-   在 [RFC7871 的 11.2 章节](https://datatracker.ietf.org/doc/html/rfc7871#section-11.2) 中提到了一种针对 ECS 的攻击（即 Birthday Attack），因此当 DNS 请求/响应中的 ECS 信息不完整时、需要彻底忽略 ECS，降级回传统 DNS 查询。
-   使用 ECS 会降低递归 DNS 的性能、甚至可以被用于发动针对递归 DNS 的攻击：以前一个递归 DNS 可以为所有人缓存同一个 CDN 节点 IP，现在却需要为每个人缓存不同的 CDN 节点 IP。[RFC7871 的 11.3 章节](https://datatracker.ietf.org/doc/html/rfc7871#section-11.3) 也因此指出，并非所有的递归 DNS 都需要支持 ECS。
-   ECS 需要从用户、到递归 DNS、到权威 DNS 全链路均提供支持才能生效。虽然在 [DNSFlagDay](https://dnsflagday.net/) 的大力推动下，绝大部分权威 DNS 已经支持 ECS，但在递归 DNS 中 ECS 的普及率仍然不容乐观。

___

在补充介绍了递归 DNS 是如何优化 CDN 结果后，不难得出结论：

-   需要被代理的域名、**必须在远端代理服务器上进行解析**、才能得到最合适的解析结果。
-   在本地对需要代理的域名进行 DNS 解析，只不过是为了让 Surge/Clash 等软件能够基于 IP 分流（Surge/Clash 的 TUN/TAP 会直接返回 Fake IP、本地 DNS 解析的结果根本不会暴露给外部）罢了。本地 DNS 解析的结果不需要很精确，**建议牺牲准确度换更快的速度**。
-   为了能够让被代理的域名在远端服务器上解析，**在通过某种协议将代理请求发送给远端代理服务器时，必须直接封装该网络请求的域名**。
-   使用 Surge/Clash 等软件后，**完全无需使用 dnsproxy 或 dns2socks 转发本地 DNS 查询。代理此类 DNS 查询不仅没有必要，反而会导致延迟升高、影响上网体验**。

## 正确配置 SmartDNS

> SmartDNS 是一个运行在本地的 DNS 服务器，它接受来自本地客户端的 DNS 查询请求，然后从多个上游 DNS 服务器获取 DNS 查询结果，并将访问速度最快的结果返回给客户端，以此提高网络访问速度。 SmartDNS 同时支持指定特定域名 IP 地址，能够以极高的性能进行匹配，可用于过滤广告或分流。

由于 SmartDNS 的极致性能和强大特性，以及「提高网络访问速度」的功能，许多 YouTube 视频和教程文档都将其奉若神明、称为一切的解药。然而事实上，如果不经过仔细的配置，SmartDNS 不仅不能起到预期的效果，反而还会导致负优化。

正如我在前文与 [浅谈在代理环境中的 DNS 解析行为](https://blog.skk.moe/post/what-happend-to-dns-in-proxy/) 一文中所说，如果一个网络请求将会被封装转发给远端代理服务器、会在远端代理服务器进行 DNS 解析，因此在本机进行的 DNS 解析得到的结果是没有任何意义的。因此，需要被代理的域名，并不在乎能否得到延时最低的 IP，**只需要不干扰 Surge/Clash 等软件的 IP 分流规则即可，无需非常精确**。

> 因此，不需要对 SmartDNS 产生的 DNS 查询请求和测速握手进行代理。将 DNS 查询转发给远端代理服务器会大幅增加 DNS 解析用时、影响上网体验！

因此，需要被代理的域名不需要进行测速——测速不仅浪费一个 RTT，而且 ISP 和其它中间人可能会记录你的 ICMP 或 TCP 握手行为。除此以外，考虑到绝大部分递归 DNS（Local DNS）都可能存留日志用作各种用途（如下图所示的「中国科学技术大学 USTC 校园网 DNS 最近 10 分钟查询统计」），需要被代理的域名也最好不要选用位于国内的递归 DNS 作为上游。

![中国科学技术大学 USTC 校园网 DNS 最近 10 分钟查询统计](https://img10.360buyimg.com/ddimg/jfs/t1/59587/8/20713/160126/62dbd1a7Ee226718b/d2dbd163866997b3.png)

### 中场休息：SmartDNS 是如何避免因测速导致 DNS 解析过慢的

第二个中场休息环节，这次简单讲讲 SmartDNS 的工作原理。

有一些人认为，SmartDNS 配置了数十个上游，需要对上游返回的每一个 IP 都进行测速，反而严重影响 DNS 解析速度。但是实际使用 SmartDNS 后，并没有出现 DNS 解析过慢的情况。这是因为 SmartDNS 早就考虑到了测速与延时的问题、并进行了相关的优化。SmartDNS 在首次 DNS 解析请求时，会同时向所有上游发起并发查询；一旦有一个上游返回了结果，SmartDNS 就会对这第一个返回的结果进行测速，得到其中延时最低的 IP，将其返回给用户、设置 TTL 为 10；与此同时，SmartDNS 仍然会等待剩余上游返回结果、异步进行测速，直到所有上游都返回了结果（或超时）、SmartDNS 将所有的 IP 都进行测速以后，才会得到最优 IP：

-   假设我们向 SmartDNS 解析一个域名 `example.skk.moe`，由于没有命中 SmartDNS 的缓存，因此不得不向上游获取结果。
-   SmartDNS 同时向上游 A、B、C 发起查询请求
-   假设上游 B 最先返回了查询结果，查询结果包含了三个 IP：`114.5.1.4`、`11.45.1.4` 和 `19.19.8.10`。
-   SmartDNS 立刻开始对这三个 IP 进行测速。假设测得延时最低的 IP 是 `11.45.1.4`
-   SmartDNS 会立刻返回 `11.45.1.4` 给客户端，同时设置 TTL 为 10（即指示 `11.45.1.4` 只应该在客户端被缓存 10 秒中）
-   在接下来 10 秒内，客户端都会使用都会使用 `11.45.1.4` 来处理发往 `example.skk.moe` 的网络连接；与此同时 SmartDNS 仍然在等待上游 A 和 C 的结果。
-   一旦上游 A 和 C 的查询结果也都返回，SmartDNS 会把上游 A、B、C 的结果进行汇总去重、重新测速，最终得到最快的那个 IP。
-   由于 SmartDNS 有着非常严格的超时设置，因此上述「等待剩余上游结果并分别进行测速」步骤不会超过 10 秒。
-   等到 10 秒过去、客户端再次向 SmartDNS 查询 `example.skk.moe` 时，SmartDNS 才会返回最快的 IP、并设置一个「正确」的 TTL。

总而言之，SmartDNS 首先会尽快返回一个「次优」的 IP、要求客户端仅在接下来 10 秒钟内使用「次优」的 IP，之后 SmartDNS 就能返回「最优」的 IP。

> 不过正如我在前文所说，Surge 支持针对 DNS 返回的多个 IP 同时进行握手、并使用最先完成握手的 TCP 进行后续请求（丢弃其余的 TCP 握手），因此 Surge 使用的 IP 一定是延时最低的、而且能够跳过出现故障的 IP；而且 Surge 复用了并发握手时的 TCP 连接进行后续请求，因此延时比 SmartDNS「先测速、后返回 IP」更低。  
> 因此在搭配 Surge 使用时，上游 DNS 不需要自带测速；最好是能合并多个上游返回的结果，将多个上游返回的一大堆 IP 全部喂给 Surge、让其并发握手。我给 SmartDNS 开了 [对应的 Feature Request](https://github.com/pymumu/smartdns/issues/994)，感兴趣的可以关注一下。

### 在 SmartDNS 中使用 dnsmasq-china-list 进行分流

[felixonmars/dnsmasq-china-list](https://github.com/felixonmars/dnsmasq-china-list) 是一组开源的，覆盖了绝大部分中国大陆的域名的 dnsmasq 配置文件，也可以通过预定义的 `Makefile` 生成供 unbound、bind9、dnscrypt-proxy、SmartDNS、AdGuardHome、coredns 使用的配置文件。截至本文写就，`dnsmasq-china-list` 已经收录了 65743 个域名。满足以下任意两条规则之一的域名即会被收录到列表中：

-   是 `.cn` 后缀的域名（包括 `.edu.cn`、`.gov.cn`、`.org.cn`、`.ac.cn` 等）
-   同时满足以下两条规则的非 `.cn` 后缀的域名：
    -   域名使用的权威 DNS（Authoritative DNS）拥有位于中国大陆境内的节点
    -   通过位于中国大陆境内的递归 DNS 解析时，解析得到的 IP 亦位于中国大陆境内

如果需要设置 SmartDNS 针对指定域名使用与默认配置不同的上游进行解析，可以使用 `nameserver`，如下所示：

```
nameserver /example.cn/domestic
```

如果需要设置 SmartDNS 针对指定域名不使用默认配置的上游、且采用与默认不同的测速方式，需要使用 `domain-rules`，如下所示：

```
domain-rules /example.cn/ -speed-check-mode tcp:80 -nameserver domestic
```

`dnsmasq-china-list` 预定义的 `Makefile` 同时支持生成上述两种配置格式的文件。使用下述命令可以生成使用 `nameserver` 的配置条目：

```
make SERVER=domestic smartdns
```

我给 `dnsmasq-china-list` 开了 PR（[felixonmars/dnsmasq-china-list#381](https://github.com/felixonmars/dnsmasq-china-list/pull/381)）且已经被合并，现在已经可以生成使用 `domain-rules` 的配置条目：

```
make SERVER=domestic SMARTDNS_SPEEDTEST_MODE=tcp:80 smartdns-domain-rules
```

> 注意，你可能需要安装 `make` 才可以使用上述命令。在 macOS 上，`make` 包含在 Xcode Command Line Tools 之中。

### 配置 SmartDNS 上游和仅测速国内域名

如前文所说，在本地解析需要被代理的域名时，不需要测速、也不一定要绝对准确，只需要解析得到的 IP 不会干扰 Surge/Clash 分流即可；只将国内的递归 DNS 作为上游解析国内直连域名，也只对其进行测速。因此，我们使用「白名单」策略，默认解析不测速、不使用国内递归 DNS 作为上游。

首先需要禁用 SmartDNS 全局的测速设置：

```
speed-check-mode none
```

然后设置两组上游 DNS：默认的一组不位于中国大陆境内的递归 DNS；另一组位于中国大陆境内的递归 DNS，仅用于解析 `dnsmasq-china-list` 列表中的域名：

```
# ----- Default Group -----
# 默认使用的上游 DNS 组
# OpenDNS 非常规 443 端口、支持 TCP 查询
server-tcp 208.67.220.220:443
# OpenDNS 的 IP DoH
server-https https://146.112.41.2/dns-query
# TWNIC 的 IP DoH
server-https https://101.101.101.101/dns-query
# 你也可以配置其它 DNS 作为上游

# ----- Domestic Group: domestic -----
# 仅用于解析 dnsmasq-china-list 列表中的域名
# 腾讯 DNSPod IP DoT
server-tls 1.12.12.12:853 -group domestic -exclude-default-group
server-tls 120.53.53.53:853 -group domestic -exclude-default-group
# 阿里 IP DoT
server-tls 223.5.5.5:853 -group domestic -exclude-default-group
server-tls 223.6.6.6:853 -group domestic -exclude-default-group
# 114 DNS、使用 TCP 查询
server-tcp 114.114.114.114 -group domestic -exclude-default-group
server-tcp 114.114.115.115 -group domestic -exclude-default-group
# CNNIC 公共 DNS、仅支持 UDP 查询
server 1.2.4.8 -group domestic -exclude-default-group
server 210.2.4.8 -group domestic -exclude-default-group
```

其中，设置有 `-exclude-default-group` 的上游 DNS 默认不会被使用，仅当 `domain-rules` 或 `nameserver` 配置明确指定时使用。

> 你可能注意到，我的配置中添加了 CNNIC（中国互联网络信息中心）的公共 DNS。这是考虑到 CNNIC 的公共 DNS 节点质量较差、出口 IP 位置稀少、基本没有针对 CDN 做任何优化（截至本文写就，`1.2.4.8` 的 Anycast 仅在 浙江杭州阿里云 和 香港 Zenlayer 广播路由，`210.2.4.8` 的 Anycast 仅在 北京联通 广播路由）。所以，一般情况下，只有 CNNIC 的公共 DNS **一定不能** 返回距离我位置最近的 CDN 节点（其余的公共 DNS 基本都会返回给我最优的 CDN 节点）。  
> 设想一下，除 CNNIC 外、大部分公共 DNS 都会尽可能返回距离我本地运营商最近的、最优的 CDN 节点；由于 SmartDNS 的测速和优选，CNNIC 返回的非最优 CDN 节点一般会被忽略。然而，假如距离我最近的 CDN 节点出现故障，只有 CNNIC 能够给我返回不一样的 CDN 节点、能够响应 SmartDNS 测速，因此我能够使用并非最优、但是可用的 CDN 节点「救急」、不至于直接「断网」。  
> 简单来说，就是因为 CNNIC 公共 DNS **能够非常稳定地提供质量最差的递归 DNS 服务、不会间歇发生解析质量好转**，才得以入选。

最后，引入前文由 `dnsmasq-china-list` 生成的 `domain-rules` 配置文件：

```
conf-file /path/to/dnsmasq-china-list/accelerated-domains.china.domain.smartdns.conf
conf-file /path/to/dnsmasq-china-list/apple.china.domain.smartdns.conf
```

## 参考资料

-   [Surge 对网络体验的优化](https://community.nssurge.com/d/4-surge)
-   [RFC3809 - A SOCKS-based IPv6/IPv4 Gateway Mechanism](https://www.rfc-editor.org/rfc/rfc3089)
-   [使用公共 DNS 上网的弊端（一）](https://ephen.me/2017/PublicDns_1/)
-   [使用公共 DNS 上网的弊端（二）](https://ephen.me/2017/PublicDns_2/)