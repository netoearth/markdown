本文适合希望使用 CloudFront 对网站、视频等进行加速的用户。

## 背景

随着移动互联网的发展，越来越多的图片分享、短视频、在线直播等应用越来越火，如何安全、经济高效、低延迟的将内容分发到终端用户受到开发者越来越多的关注。内容分发网络（CDN）是为解决互联网访问拥塞的问题而生，通过使用大量更靠近终端用户的边缘节点向终端用户提供服务以加速内容的分发，因此在应用系统前端部署 CDN 成为内容加速分发的不二选择。

### 什么是 Amazon CloudFront

Amazon CloudFront 是一种全球内容分发网络服务，可以安全地以低延迟和高传输速度向浏览者分发数据、视频、应用程序和 API。CloudFront 与 亚马逊云科技的多种产品集成，如用于 DDoS 缓解的 Amazon Shield、应用程序防火墙 Amazon WAF、 Amazon S3、 Elastic Load Balancing、 Amazon EC2 以及 Amazon Route 53等，还可以在亚马逊云科技全球基础设施运行用户的自定义代码 Lambda@Edge以及在边缘节点直接进行计算的CloudFront Functions。

截止目前（2022年4月），Amazon CloudFront 有超过 310 个节点（超过 300 个边缘站点和 13 个区域性边缘缓存）的全球网络，该网络覆盖 47 个国家/地区的 90 余个城市。

举例说明，当客户端发起对 www.customer.com 的访问时，首先需要通过 DNS系统解析出该域名对应的主机 IP，通过本地 ISP DNS 递归查询到 customer.com 的 DNS 域名服务器并了解到该域名是指向了 xxx.cloudfront.net，进一步解析 xxx.cloudfront.net，CloudFront 的 DNS 域名服务器会根据请求来源的 IP 等信息，返回最适合当前该客户端访问的边缘节点的主机 IP 如1.1.1.1，最终该客户端向1.1.1.1发出请求。如果该边缘节点已经缓存了该客户端请求的内容（图片、视频等静态文件），则直接返回给客户端，如果未缓存，则首先回到源站取回该内容，并存储在边缘节点，以便下次客户端对该内容请求时可以直接返回该内容。

更好的 CDN 性能表现能带来更多的访问流量，以及更好的用户体验，下面我们将介绍如何配置以及优化建议。

## 如何配置 CloudFront Distribution

下面介绍下如何为自己的应用配置 CloudFront，实现网站访问加速。

在开始之前，首先简单介绍下 CloudFront的常见术语。

Distribution：分配，是 CloudFront 的基本单元，每个分配有唯一的 ID 以及CloudFront 为其分配的域名（类似 abcdefg13456789.cloudfront.net）。

Origin： 源站，顾名思义，是需要被加速的站点，可以是 S3 存储桶，可以是 ELB/EC2，可以是 Elemental MediaStore/MediaPackage，或者是用户自定义的站点（如第三方 IDC 中的 HTTP Web 服务器）。一个分配中可以有多个源站。

Behaviors：行为， CloudFront 通过路径匹配和优先级决定执行哪一个缓存行为，一个分配中可以有多个 行为，并且每个 行为 对应一个源站。在 行为 中可以设置缓存 TTL 时间，允许的 HTTP 行为（GET，PUT，POST 等），与 Lambda@edge函数关联等等。

本节将涉及如下内容：

-   创建一个分配；
-   该分配有两个源站，其中一个是创建时添加，一个是后添加。一个源站为ELB，一个源站为 S3 存储桶；
-   两个 行为，一个是默认 行为 并对应回源 ELB，一个是新加的 行为 并对应源站 S3 存储桶；
-   ELB，S3 创建过程不做介绍；

### 创建 分配

进入 CloudFront console，并选择新建 Distribution。设置如下：

#### 源

[![](https://s3.cn-north-1.amazonaws.com.cn/awschinablog/configure-amazon-cloudfront-to-accelerate-the-whol1.png)](https://s3.cn-north-1.amazonaws.com.cn/awschinablog/configure-amazon-cloudfront-to-accelerate-the-whol1.jpg)

参数解释如下：

<table><tbody><tr><td colspan="2">源 设置</td></tr><tr><td>源域</td><td><p>源站地址，仅支持域名方式。也可以通过下拉列表直接选择帐号里已经创建过的 ELB，S3，MediaStore，MediaPackage等。</p><p>如果是自定义站点，需先给该站点配置一个域名，不能填写 IP。</p></td></tr><tr><td>协议</td><td><p>CloudFront 回源协议，可以是 HTTP 或 HTTPS，或者与实际客户请求时一样的协议。注意这儿是 CloudFront 回源站时用的协议，而不是CloudFront对外服务的协议。</p><p>如果选用了 HTTPS，一定要注意源站已经配置对应回源域名（Origin Domain Name）的 SSL 证书。</p></td></tr><tr><td>源路径</td><td>（可选）如果源站内容有多层目录，而又希望回源的时候路径上不体现这些目录，可以在此设置要隐藏的目录层级。例如：配置源路径 /version1 后，客户访问www.customer.com/page.html 相当于访问源站 origin.customer.com/version1/page.html</td></tr><tr><td>名称</td><td>可以设置一个容易记忆的名字</td></tr><tr><td>添加自定义标头</td><td>可以在请求源站时，带上特殊的Http header头，例如可以带上自定义的验证头。</td></tr><tr><td>启用源护盾</td><td>源护盾是一个附加的缓存层，可以减少源站回源的压力，如果源站不在亚马逊云科技上部署，通过源护盾，还可以改善回源的稳定性和速度。源护盾会产生额外的费用。</td></tr><tr><td>其他设置</td><td>可以设置回源时的失败重试次数，连接超时，响应超时，以及连接复用的保持时间。</td></tr></tbody></table>

#### 行为

[![](https://s3.cn-north-1.amazonaws.com.cn/awschinablog/configure-amazon-cloudfront-to-accelerate-the-whol2.png)](https://s3.cn-north-1.amazonaws.com.cn/awschinablog/configure-amazon-cloudfront-to-accelerate-the-whol2.jpg)

参数解释如下：

<table><tbody><tr><td colspan="2">默认缓存行为</td></tr><tr><td>路径模式</td><td>默认是 * ，就是全部匹配。支持通配符 * 代表0或多个字符，? 代表完全匹配一个字符。注意路径模式是区分大小写的。例如 image/*.jpg 代表image目录下的所有jpg文件都遵循这个缓存行为。</td></tr><tr><td>自动压缩对象</td><td>是否在客户端支持的时候，返回源站文件的压缩版本，以优化体验。注意还需要在下面的缓存策略中，勾选使用 GZip 或 Brotli 压缩算法。</td></tr><tr><td>查看器协议策略</td><td>允许客户端访问CloudFront时使用的协议，支持 HTTP 和 HTTPS，并且提供重定向 HTTP 到 HTTPS。</td></tr><tr><td>允许的 HTTP 方法</td><td>允许的 HTTP 动作，不同的 行为 可以配置不同的选项。</td></tr><tr><td>限制查看器访问</td><td>是否使用签名的 URL 或签名的 Cookie 才能访问，也就是常说的防盗链技术。</td></tr><tr><td>缓存键和源请求</td><td>如何进行源的缓存，以及如何区分不同的缓存，通过这两个设置进行。这里有两种方式，一种是通过缓存策略和源请求策略共同确定，另一种旧的遗留策略，不推荐使用。下面将会详细介绍如何通过缓存策略和源请求策略确定缓存的情况。</td></tr><tr><td>响应标头策略</td><td>通过响应标头策略，可以实现安全相关需求和获取更多的统计信息，下面将会详细介绍。</td></tr><tr><td>其他设置</td><td>可以设置IIS的平滑流，字段级加密，以及实时访问日志流配置。</td></tr></tbody></table>

#### 策略配置

下面介绍上面提到的三个关键的策略配置：

-   缓存策略

缓存策略用于决定内容是否进行缓存，以及缓存的时间。CloudFront默认提供了多种缓存托管策略，可以直接选择使用，也可以根据需要自定义缓存策略来使用，默认提供的托管策略如下：

<table><tbody><tr><td colspan="2">托管 缓存策略</td></tr><tr><td><code>CachingOptimized</code></td><td>适用于静态网站加速的场景。源站不会因为不同用户、不同终端等返回不同的内容，内容默认进行了压缩。</td></tr><tr><td><code>CachingOptimizedForUncompressedObjects</code></td><td>和上面策略相同，但不进行压缩。</td></tr><tr><td><code>CachingDisabled</code></td><td>适用于动态内容，或不可缓存的内容。</td></tr><tr><td><code>Elemental-MediaPackage</code></td><td>为Amazon Elemental MediaPackage服务配置的策略。</td></tr><tr><td><code>Amplify</code></td><td>为Amazon Amplify Web应用程序配置的策略。</td></tr></tbody></table>

如果需要自定义缓存策略，各个设置项如下：

<table><tbody><tr><td colspan="2">自定义 缓存策略</td></tr><tr><td>TTL设置</td><td><p>最短TTL：CloudFront在到达这个时间后，会向源请求以确认缓存是否最新。</p><p>默认TTL：如果源的内容没有指定Cache-Control或Expires标头时，CloudFront会按这个值设置缓存过期时间。</p><p>最长TTL：当源的内容指定了Cache-Control或Expires标头时，设置缓存过期时间为标头和最长TTL中的最小值。</p><p>注意，这三个值只影响CloudFront回源和缓存的行为，向客户端返回的缓存标头以源站返回的标头为准。</p></td></tr><tr><td>缓存键设置</td><td><p>该设置指定在缓存源站内容时，应该用哪些关键键值进行区分不同内容，分别进行缓存。</p><p>标题：无 表示不需要区分不同的Http Header标头。可以通过 包括以下标题 选择需要区分的Http Header标头。</p><p>查询字符串：无 表示不需要区分不同查询字符串的回源。全部 表示需要带上所有查询字符串进行缓存区分。包括/排除指定字符串 表示包括或排除对应的字符串来区分缓存。</p><p>Cookie：和查询字符串类似，但是是处理Cookie里包含的值。</p></td></tr><tr><td>压缩支持</td><td>可以开启GZip和Brotli压缩，注意选择了 自动压缩对象 后才会生效。</td></tr></tbody></table>

-   源请求策略

源请求策略用于设置回源请求时的行为，CloudFront默认提供了多种源请求托管策略，可以直接选择使用，也可以根据需要自定义源请求策略来使用，默认提供的托管策略如下：

<table><tbody><tr><td colspan="2">托管 源请求策略</td></tr><tr><td><code>UserAgentRefererHeaders</code></td><td>仅包含User-Agent和Referer标头，可以统计客户来源。</td></tr><tr><td><code>CORS-CustomOrigin</code></td><td>包含Origin标头，适用于自定义源启用跨源资源共享 CORS。</td></tr><tr><td><code>CORS-S3Origin</code></td><td>适用于S3源启用跨源资源共享 CORS。</td></tr><tr><td><code>AllViewer</code></td><td>适用于动态请求的源站，源站可以获取查询字符串和Cookie等信息。</td></tr><tr><td><code>Elemental-MediaTailor-PersonalizedManifests</code></td><td>适用于Amazon Elemental MediaTailor 终端节点的源</td></tr></tbody></table>

如果需要自定义源请求策略，各个设置项如下：

<table><tbody><tr><td colspan="2">自定义 源请求策略</td></tr><tr><td>标题</td><td><p>定义回源时，怎样设置Http Header。</p><p>无：不使用Http Header标头进行回源请求。</p><p>包括以下标题：选择指定的Http Header标头进行回源。</p><p>所有查看器标题和以下CloudFront标题：CloudFront可以判断一些常用的情况，并进行标识，例如客户端是否是手机，客户端访问的国家标识等。</p><p>所有查看器标题：传递客户端的所有标题。</p><p>默认情况下，源站收到 CloudFront的请求中的 Host 字段值为 源站 设置中的源站域名，如果用户的源站需要拿到客户端发来的 Host 字段的值（即用户 CNAME 到该分配的域名），在此处就需要将 Host 添加到白名单，此时源站将收到客户端发出请求时的域名。</p></td></tr><tr><td>查询字符串</td><td><p>定义回源时，怎样设置Url上问号后的参数。</p><p>无：表示不带任何字符串回源。</p><p>全部：表示带上所有查询字符串回源。</p><p>包括指定字符串：表示包括对应的字符串回源。</p></td></tr><tr><td>Cookie</td><td><p>定义回源时，怎样设置Cookie值。</p><p>无：表示不带任何Cookie回源。</p><p>全部：表示带上所有Cookie回源。</p><p>包括指定Cookie：表示包括对应的Cookie回源。</p></td></tr></tbody></table>

-   响应标头策略

响应标头策略用于返回给客户端内容时，增加特殊的Http Header标头，实现对应功能。CloudFront默认提供了多种响应标头托管策略，可以直接选择使用，也可以根据需要自定义响应标头策略来使用，默认提供的托管策略如下：

<table><tbody><tr><td colspan="2">托管 响应标头策略</td></tr><tr><td><code>SimpleCORS</code></td><td>允许来自任何源的简单 CORS 请求（添加标头：Access-Control-Allow-Origin: *）。</td></tr><tr><td><code>CORS-With-Preflight</code></td><td>支持CORS的预检请求（Http的OPTIONS方法）。</td></tr><tr><td><code>SecurityHeadersPolicy</code></td><td>添加一组安全标头。</td></tr><tr><td><code>CORS-and-SecurityHeadersPolicy</code></td><td>添加CORS标头和安全标头。</td></tr><tr><td><code>CORS-with-preflight-and-SecurityHeadersPolicy</code></td><td>添加CORS，预检和安全标头。</td></tr></tbody></table>

如果需要自定义响应标头策略，各个设置项如下：

<table><tbody><tr><td colspan="2">自定义 响应标头策略</td></tr><tr><td>跨源资源共享 (CORS)</td><td>配置CORS相关设置。如果源提供了CORS相关设置，且没有选择 源覆盖，则会把源的设置返回到客户端。</td></tr><tr><td>安全标头</td><td>多种安全相关标头，可以进行自定义设置。</td></tr><tr><td>自定义标头</td><td>可以设置自定义的标头。</td></tr><tr><td>服务计时标头</td><td><p>可以提供CloudFront是否命中缓存，回源点，各级回源所用时间等信息。</p><p>可以设置采样的百分比。</p></td></tr></tbody></table>

### 函数

[![](https://s3.cn-north-1.amazonaws.com.cn/awschinablog/configure-amazon-cloudfront-to-accelerate-the-whol3.png)](https://s3.cn-north-1.amazonaws.com.cn/awschinablog/configure-amazon-cloudfront-to-accelerate-the-whol3.jpg)

函数关联设置，可以在CloudFront处理请求的不同阶段，嵌入Lambda@edge和 CloudFront Functions代码，进行自定义的逻辑处理，这里不展开。

### 设置

[![](https://s3.cn-north-1.amazonaws.com.cn/awschinablog/configure-amazon-cloudfront-to-accelerate-the-whol4.png)](https://s3.cn-north-1.amazonaws.com.cn/awschinablog/configure-amazon-cloudfront-to-accelerate-the-whol4.jpg)

参数解释如下：

<table><tbody><tr><td colspan="2">设置</td></tr><tr><td>价格级别</td><td>有三类，北美与欧洲、北美欧洲加亚太非、全球加速，不同类别应用到的 PoP 点不同，对应价格也不一样。</td></tr><tr><td>AWS WAF Web ACL</td><td>如果配置了 WAF，可以在此关联，当然 CloudFront Distribution 创建完成后也可以在WAF Console关联。</td></tr><tr><td>备用域名（CNAMEs）</td><td>（若使用自己域名，该项是必须项）CloudFront Distribution 创建完成后，CloudFront 会提供一个以 cloudfront.net 结尾的域名，如果需要使用自己的域名的话，需要在此处填写待使用的域名。</td></tr><tr><td>自定义SSL证书</td><td><p>支持用户使用自己域名的证书，需要与上一栏域名匹配。可以使用 Amazon ACM 申请证书，需要注意的是，需要在 us-east-1 区域下的 ACM 申请才能应用到 CloudFront。</p><p>支持 dedicate IP 和 SNI 两种模式。</p></td></tr><tr><td>支持的 HTTP 版本</td><td>可以启用HTTP/2的支持。</td></tr><tr><td>默认根对象</td><td>可以指定默认指向的文件名，例如：index.html</td></tr><tr><td>标准日志记录</td><td>标准访问日志记录，建议开启。日志开启还需要选择存放的S3桶。</td></tr><tr><td>IPv6</td><td>是否支持IPv6。</td></tr></tbody></table>

点击 创建分配 后，来到 分发 的列表页面，可以看到状态是“In Process”，大概10分钟左右看到状态是“已启用”时，则表示该分配创建完成，可以使用，并且可见到 CloudFront 为其分配的域名，例如xxxxxxx.cloudfront.net，如果需要使用自己的域名，需要在自己域名的DNS管理中，添加DNS解释，把域名 CNAME 到 CloudFront分配的域名，例如此处的xxxxxxx.cloudfront.net，添加后就可以使用自己域名进行访问了。

### 添加源站

接下来，我们可以再添加一个源站，点击分配的 ID，可见当前分配的相关设置，点击 “源”，可以看到当前的源站设置，点击 “创建源”，可以添加一个源站。

[![](https://s3.cn-north-1.amazonaws.com.cn/awschinablog/configure-amazon-cloudfront-to-accelerate-the-whol5.png)](https://s3.cn-north-1.amazonaws.com.cn/awschinablog/configure-amazon-cloudfront-to-accelerate-the-whol5.jpg)

此时我们添加一个 S3 桶为源站，可以在源域上选择需要的S3桶。

[![](https://s3.cn-north-1.amazonaws.com.cn/awschinablog/configure-amazon-cloudfront-to-accelerate-the-whol6.png)](https://s3.cn-north-1.amazonaws.com.cn/awschinablog/configure-amazon-cloudfront-to-accelerate-the-whol6.jpg)

注意选择 S3 桶为源站时，会出现S3存储桶访问的“使用OAI”选项，我们知道 S3 通过 ACL 和 Bucket Policy 控制存储桶的对象是否被公开访问，因此该 S3 存储桶需要允许 CloudFront 能够从 S3 存储桶拉取对象，因此这里有两种方式，一个是该桶设置为公开访问桶，任何人可以直接从该桶下载，而另一种方法是使用 OAI（Origin Access Identity），即该 分配 获取一个 OAI，并且在 S3 bucket policy 中的 principle 部分填写该 OAI，这样该 S3 存储桶将仅向该分配开放了相应的权限，而其他人无法直接从该存储桶下载资源，建议选择“是的，使用OAI”以避免 Bucket Policy 配置错误。

### 添加 行为

同样进入分配后，在 行为 栏选择 创建行为。我们假定访问 jpg图片的请求，回源到前面创建的 S3 存储桶。

[![](https://s3.cn-north-1.amazonaws.com.cn/awschinablog/configure-amazon-cloudfront-to-accelerate-the-whol7.png)](https://s3.cn-north-1.amazonaws.com.cn/awschinablog/configure-amazon-cloudfront-to-accelerate-the-whol7.jpg)

当一个请求到达CloudFront POP 点，CloudFront 检查该请求的 URL 路径，并与 行为 中设置的进行匹配，并按优先级匹配到对应的 行为 配置，执行对应的行为。

可以调整 行为 的优先级调整策略。

[![](https://s3.cn-north-1.amazonaws.com.cn/awschinablog/configure-amazon-cloudfront-to-accelerate-the-whol8.png)](https://s3.cn-north-1.amazonaws.com.cn/awschinablog/configure-amazon-cloudfront-to-accelerate-the-whol8.jpg)

## 缓存优化

本节将介绍常见的 TTL 设置建议、动态加速，设置样例以及错误处理。

### TTL 设置建议

CloudFront 在计算 Cache key 时会将请求的 URL 以及当前分配对应的 行为 配置（如是否转发 Header、Cookie、查询字符串）考虑在内，计算出唯一值。因此即使两次请求都是相同的 URL，如果两次请求的个别 Header 不一样，且该 Header 配置为需要区分缓存，则计算出的 Cache key 也不同，返回给客户端的内容自然也不同。因此配置转发的查询字符串、Cookie 及 Header 越少，Cache key 也将越少，缓存命中率就越高，带来的性能也越好。

对于用户内容在 PoP 点的缓存 TTL，可以使用源站设置的 Cache Control: max-age 的值，或者在 Behavior 中使用 Customized 设置 Minimum TTL，Maximum TTL、Default TTL（[https://docs.aws.amazon.com/zh\_cn/AmazonCloudFront/latest/DeveloperGuide/Expiration.html](https://docs.aws.amazon.com/zh_cn/AmazonCloudFront/latest/DeveloperGuide/Expiration.html) ）。

<table><tbody><tr><td>分类</td><td>静态内容，长期</td><td>静态内容，短期</td><td>动态内容</td></tr><tr><td>举例</td><td><p>*.css, *.js, *.jpg, *.png</p><p>软件下载，媒体文件，媒体分片文件（HLS *.ts，*.m3u8）。</p></td><td>登录页，index.jsp，新闻，天气信息，HLS 直播 m3u8 文件。</td><td>变化的内容，不可缓存的内容。</td></tr><tr><td>建议</td><td><p>不变的内容可以设置较大的 TTL 值，使用版本号更新内容，内容放置在S3桶中。</p><p>选择缓存策略：CachingOptimized</p></td><td>定期更新的内容设置低 TTL 值。TTL 到期后，CloudFront 回源校验源站内容是否发生变化。</td><td><p>经常变化的内容；按请求不同内容不同；设置很低甚至0 TTL。</p><p>选择缓存策略：CachingDisabled</p></td></tr></tbody></table>

对于动态内容，动态加速场景，可以在源站设置：

`Cache-Control: no-cache; max-age=0; No-store; private`  或  `Cache-Control: public; max-age=0;`

对于 Cache-Control Header 设置样例：

<table><tbody><tr><td>静态资源</td><td>登录页</td><td>媒体分片</td><td>动态内容</td><td>HLS 直播</td></tr><tr><td>*.css, *.js, 软件下载，更新包等</td><td><code>Index.html</code></td><td><code>/*.ts</code></td><td></td><td><code>/*.m3u8</code></td></tr><tr><td><code>Cache-Control:</code><br><code>public;</code><br><code>max-age=31536000</code></td><td><code>Cache-Control:</code><br><code>no-cache=Set-Cookie;</code><br><code>max-age=30</code></td><td><code>Cache-Control:</code><br><code>public;</code><br><code>max-age=31536000</code></td><td><code>Cache-Control:</code><br><code>no-cache;</code><br><code>max-age=0;No-store;private</code></td><td><code>Cache-Control:</code><br><code>public;</code><br><code>max-age=2</code></td></tr></tbody></table>

### 错误处理

当源站不可用时，可以在 CloudFront 配置针对400，403，404，405，414，500，501，502，503，504等错误码的自定义响应页并修改返回给客户端的响应码。CloudFront 将周期性验证源站的可用性，并可在源站恢复前将当前缓存中的内容作为响应返回给客户端。

[![](https://s3.cn-north-1.amazonaws.com.cn/awschinablog/configure-amazon-cloudfront-to-accelerate-the-whol9.png)](https://s3.cn-north-1.amazonaws.com.cn/awschinablog/configure-amazon-cloudfront-to-accelerate-the-whol9.jpg)

设置方式可见

[https://docs.aws.amazon.com/zh\_cn/AmazonCloudFront/latest/DeveloperGuide/GeneratingCustomErrorResponses.html](https://docs.aws.amazon.com/zh_cn/AmazonCloudFront/latest/DeveloperGuide/GeneratingCustomErrorResponses.html)

## 内容更新

当源站静态内容更新时，如何让客户端在 TTL 失效时间前即可拿到最新版本的内容是每一位开发者关心的问题，在 CloudFront 有两种方式。

### 版本号控制

当已发布的内容更新时，可在文件名或者路径中使用版本号进行区分，比如从 v1 到 v2，或者时间戳等可以区别同一对象两种版本的其他方法，同时应用测对资源链接指向新版本号的资源。

### 使用 失效(Invalidation) 功能

失效 功能可以使文件在 TTL 失效时间到达前将文件从 PoP 点删除，使用该功能可以快速的在文件名不变的前提下将文件更新。

失效 支持指定单个文件或者以\*通配符结尾的路径。限制及费用见

[https://docs.aws.amazon.com/zh\_cn/AmazonCloudFront/latest/DeveloperGuide/Invalidation.html](https://docs.aws.amazon.com/zh_cn/AmazonCloudFront/latest/DeveloperGuide/Invalidation.html)

[![](https://s3.cn-north-1.amazonaws.com.cn/awschinablog/configure-amazon-cloudfront-to-accelerate-the-whol10.png)](https://s3.cn-north-1.amazonaws.com.cn/awschinablog/configure-amazon-cloudfront-to-accelerate-the-whol10.jpg)

## 监控与日志

CloudFront 提供多种报告供用户了解自己分配的使用率等情况，您可以使用 CloudWatch API 获取分配的相应监控指标，需要注意的是，使用 CloudWatch API 时 region 应设置为 us-east-1。

CloudFront 报告提供按天或按小时的请求数、数据传输、HTTP Status Code、Top50 访问的对象，使用率以及按设备、浏览器、操作系统、客户端位置等指标的报告，同时可以配置 CloudWatch 警报，对关键指标数据进行监控（请求数、已下载字节、已上传字节、总错误率、4xx错误率、5xx错误率），一旦超过正常阈值，运维人员可第一时间得到告警。

[![](https://s3.cn-north-1.amazonaws.com.cn/awschinablog/configure-amazon-cloudfront-to-accelerate-the-whol11.png)](https://s3.cn-north-1.amazonaws.com.cn/awschinablog/configure-amazon-cloudfront-to-accelerate-the-whol11.jpg)

尽管 CloudFront 提供了多维度的报告，但可能仍不能满足用户的多维度分析的需求。因此推荐这类用户对 CloudFront 日志进行分析，当用户开启 CloudFront 分配日志后，日志将会上传到指定的 S3 存储桶，通过分析日志可以了解有关该分配的更多的客户端请求行为，有助于做运营分析或者 debug。关于日志字段解释见

[https://docs.aws.amazon.com/zh\_cn/AmazonCloudFront/latest/DeveloperGuide/AccessLogs.html](https://docs.aws.amazon.com/zh_cn/AmazonCloudFront/latest/DeveloperGuide/AccessLogs.html)

用户可以使用无服务器交互式查询服务Amazon Athena轻松的使用标准 SQL 直接在 S3 中分析数据，并与 Amazon QuickSight  集成，轻松实现数据可视化。使用 Amazon Athena 查询  Amazon CloudFront 日志方式见

[https://docs.aws.amazon.com/zh\_cn/athena/latest/ug/cloudfront-logs.html](https://docs.aws.amazon.com/zh_cn/athena/latest/ug/cloudfront-logs.html)

对于需要实时获取CloudFront日志，可以使用CloudFront的实时日志功能，这里介绍一个实时日志的一键部署解决方案，帮助客户快速分析日志：[https://aws.amazon.com/cn/blogs/china/quickly-build-custom-cdn-monitoring-through-amazon-cloudfront-real-time-log/](https://aws.amazon.com/cn/blogs/china/quickly-build-custom-cdn-monitoring-through-amazon-cloudfront-real-time-log/)

## 合规与安全

CloudFront 目前已经获得 PCI DSS 合规，GDPR（[https://aws.amazon.com/cn/compliance/privacy-features/](https://aws.amazon.com/cn/compliance/privacy-features/)）、HIPPA、SOC 以及 ISO 9001, 27001, 27017, 27018 等合规认证。

同时开启 CloudTrail，用户帐号下所有的 CloudFront API 调用记录将均被CloudTrail 记录，便于用户后期审计。下面我们将从4个方面进行介绍 CloudFront 如何保证您的内容安全。

### 回源保护

我们可以将源站设置为仅允许 CloudFront 访问，而拒绝客户端的直接回源。

#### 源站为 ELB/EC2

-   ELB/EC2 的安全组仅对 CloudFront 节点 IP 开放对应的端口（80/443），CloudFront 节点服务器的 IP 地址范围见：

[https://docs.aws.amazon.com/zh\_cn/AmazonCloudFront/latest/DeveloperGuide/LocationsOfEdgeServers.html](https://docs.aws.amazon.com/zh_cn/AmazonCloudFront/latest/DeveloperGuide/LocationsOfEdgeServers.html)

-   在 CloudFront Origin 设置时，添加自定义 Header 字段及值，源站对请求中的字段检查，若不含有该 Header 字段及值，则返回错误码。
-   回源 HTTPS，回源支持1、TLSv1.2 安全协议以及 RSA，ECDSA 两类密码算法。
-   使用 源护盾 保障源的安全

[https://docs.aws.amazon.com/zh\_cn/AmazonCloudFront/latest/DeveloperGuide/origin-shield.html](https://docs.aws.amazon.com/zh_cn/AmazonCloudFront/latest/DeveloperGuide/origin-shield.html)

### 源站为 S3 存储桶

在 S3 存储桶策略中应用 OAI，仅允许带该 OAI 的 CloudFront 分配从存储桶中获取内容。[https://docs.aws.amazon.com/zh\_cn/AmazonCloudFront/latest/DeveloperGuide/private-content-restricting-access-to-s3.html](https://docs.aws.amazon.com/zh_cn/AmazonCloudFront/latest/DeveloperGuide/private-content-restricting-access-to-s3.html)

### 传输安全

CloudFront 支持客户端到 PoP 点使用 HTTPS 请求，CloudFront 为每个分配生成的域名支持 HTTPS 请求，并支持用户上传自己从可信 CA 购买的证书。用户可以在 Amazon ACM 服务免费申请自己域名的证书，需要注意的是，需要在 us-east-1 区的 ACM 服务申请，才能应用到 CloudFront 的分配。对于用户域名证书，CloudFront 支持专属 IP 和 SNI 两种方式。

### 内容保护

#### 签名 URL、签名 Cookie

针对有些内容限制访问的情景，如仅允许付费用户或者授权用户访问的内容，我们可以通过签名 URL 或签名 Cookie 的方式提供内容的私有化访问。文档中已有非常详细的说明，在此不再展开，具体方式见：

[https://docs.aws.amazon.com/zh\_cn/AmazonCloudFront/latest/DeveloperGuide/PrivateContent.html](https://docs.aws.amazon.com/zh_cn/AmazonCloudFront/latest/DeveloperGuide/PrivateContent.html)

#### Field-Level Encryption 字段级加密

2017年12月份，CloudFront 推出了该功能，使用该功能，可以进一步增强敏感数据 (如信用卡号码或个人身份信息) 的安全性。在将  POST  请求转发到您的源站之前，CloudFront 的字段级加密使用特定于字段的加密密钥 (由用户提供) 对  HTTPS  表单中的敏感数据进行进一步加密。这可确保敏感数据只能被应用程序堆栈中的某些组件或服务解密和查看。配置方式见：

[https://docs.aws.amazon.com/zh\_cn/AmazonCloudFront/latest/DeveloperGuide/field-level-encryption.html](https://docs.aws.amazon.com/zh_cn/AmazonCloudFront/latest/DeveloperGuide/field-level-encryption.html)

#### Geo 限制

CloudFront 提供了基于地理位置访问限制的功能，用户可以通过设置白名单或者黑名单的方式，允许或禁止某个国家或地区对自己分配的访问。

[![](https://s3.cn-north-1.amazonaws.com.cn/awschinablog/configure-amazon-cloudfront-to-accelerate-the-whol12.png)](https://s3.cn-north-1.amazonaws.com.cn/awschinablog/configure-amazon-cloudfront-to-accelerate-the-whol12.jpg)

### 缓解攻击

Amazon WAF 是一款 Web 应用程序防火墙，帮助保护您的 Web 应用程序免受常见 Web 漏洞的攻击。AWS Shield 是一种托管式分布式拒绝服务(DDoS)防护服务，可以保护在 AWS 上运行的防护应用程序。

CloudFront可以与 Amazon WAF、Amazon Shield 进行集成，用户可以在创建 CloudFront 分配时关联已创建的 WAF，或者在 WAF Console 创建完 WAF 规则后与 CloudFront 分配关联。

## 可编程 CDN

亚马逊云科技推出了Lambda@edge 和 CloudFront Functions，用户可以通过CloudFront事件触发，例如源服务器和浏览者之间的内容请求或响应。通过函数代码快速实现自定义功能，感兴趣的读者可以查看以下内容：

[https://docs.aws.amazon.com/zh\_cn/AmazonCloudFront/latest/DeveloperGuide/lambda-examples.html](https://docs.aws.amazon.com/zh_cn/AmazonCloudFront/latest/DeveloperGuide/lambda-examples.html)

[https://aws.amazon.com/cn/blogs/aws/introducing-cloudfront-functions-run-your-code-at-the-edge-with-low-latency-at-any-scale/](https://aws.amazon.com/cn/blogs/aws/introducing-cloudfront-functions-run-your-code-at-the-edge-with-low-latency-at-any-scale/)

## 参考资料

\[1\] 开发人员指南：[https://docs.aws.amazon.com/zh\_cn/AmazonCloudFront/latest/DeveloperGuide/Introduction.html](https://docs.aws.amazon.com/zh_cn/AmazonCloudFront/latest/DeveloperGuide/Introduction.html)

\[2\] Amazon WAF [https://aws.amazon.com/cn/waf/](https://aws.amazon.com/cn/waf/)

\[3\] Amazon Certificate Manager [https://aws.amazon.com/cn/certificate-manager/](https://aws.amazon.com/cn/certificate-manager/)

\[4\] Amazon Shield [https://aws.amazon.com/cn/shield/](https://aws.amazon.com/cn/shield/)

## 本篇作者