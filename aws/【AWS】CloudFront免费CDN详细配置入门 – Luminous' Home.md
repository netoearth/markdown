![](https://cdn.luotianyi.vc/wp-content/uploads/2021-12-27_09-14-09.jpg)

一转眼几个月没发文章了，考研的几个月里也不是很想动笔就一直咕咕咕了。话说回来，在2021年的11月，亚马逊AWS对他的Always Free策略做出了一些改变，这篇文章主要介绍如何利用其中的CloudFront资源，一起来看看吧~

___

## 零、序言

CloudFront是AWS推出的CDN功能，依托于AWS的全球基础设施，拥有海量的节点和带宽资源，过去一直以高昂的价格令个人站长望而却步。很多服务商如`腾讯云`、`QUIC.CLOUD`等，其全球布局都是租赁的CloudFront的边缘节点。

2021年对于云计算行业也是个不太平静的一年，经历了CloudFlare怒怼AWS流量暴利和AWS彻底砍掉自家Educate优惠码之后，AWS对自家免费产品额度做出了一些改变。

<table><tbody><tr><td><strong>CloudFront</strong></td><td><strong>提升前</strong></td><td><strong>提升后</strong></td></tr><tr><td><strong>流量</strong></td><td>50G/月 · 免费一年</td><td><span>&nbsp;1T/月 · 永久免费</span></td></tr><tr><td><strong>请求次数</strong></td><td>200万/月 · 免费一年</td><td><span>&nbsp;1000万/月 · 永久免费</span></td></tr></tbody></table>

除此之外，EC2的免费流量额度也从之前的免费一年15G或各区域1G提升到了永久每月100G，这么看起来新用户的免费EC2实例也不是那么的鸡肋了。经过博主测试，以上的免费额度已经在12月生效，可以放心使用。在配置过程中，AWS机翻的控制台与英文文档间理解存在诸多不便，因此全文以英文控制台为例进行演示。

> **注意：**本文为仅针对一般用途的非专业实践，并未完全利用CloudFront所具备的全部功能，更合理的高级功能应用请结合官方文档及个人实践探索，欢迎在评论区分享你的实践经验~

### 参考资料来源

> AWS Blogs：[点击前往](https://aws.amazon.com/cn/blogs/aws/aws-free-tier-data-transfer-expansion-100-gb-from-regions-and-1-tb-from-amazon-cloudfront-per-month/)  
> CloudFront Developer Guide：[点击前往](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/Introduction.html)

___

## 一、创建资源

开始使用CloudFront，来到CloudFront的控制台（[点击前往](https://console.aws.amazon.com/cloudfront/v3/home?region=us-east-1#)），要做的第一件事就是点击Create distribution通过引导新建一个CDN资源。整个引导其实是一个完整的单源站单规则配置流程，不过为了更加清晰的理解部分参数在此保持默认，后文第二节会对这些细节单独详细展开。

![](https://cdn.luotianyi.vc/wp-content/uploads/2021-12-28_04-24-19.jpg)

首先最开始的是源站配置，第一项`Origin domain`，只能填入一个解析到你源站的域名。如图如果你不想自己解析的话，可以利用`nip.io`项目在IP末尾添加`.nip.io`解析到源站IP。`Protcol`是取源协议，根据你的具体需求选择HTTP/HTTPS回源亦或者是匹配访问。

![](https://cdn.luotianyi.vc/wp-content/uploads/2021-12-28_04-34-04.jpg)

接下来是去设置默认的缓存行为，其中第一条是访问设置`Viewer`。`Viewer protocol policy`根据你对外访问需要决定是否将HTTP跳转到HTTPS；`Allowed HTTP methods`为允许的请求头，可以根据实际需要选择，简单概括文件下载场景可选第一条、静态网站可选第二条、动态网站必须选择第三条，如果无法明确判定自己需要直接选择第三条全部支持即可。

![](https://cdn.luotianyi.vc/wp-content/uploads/2021-12-28_04-42-59.jpg)

第二条是缓存设置和源请求设置`Cache key and origin requests`，作为默认规则建议分别选择`CachingDisabled`禁止缓存以及`AllViewer`将所有访客请求头转发到源站。

![](https://cdn.luotianyi.vc/wp-content/uploads/2021-12-28_04-47-59.jpg)

其他未提到的项目暂时保持默认（各个自定义名称的条目除外，看你要不要自己命名），然后拉到最下面点击`Create distribution`创建CDN资源。

![](https://cdn.luotianyi.vc/wp-content/uploads/2021-12-28_04-51-44.jpg)

这时候一个CDN资源就成功创建出来了，稍后你可在面板上看到为你分配的`*.cloudfront.net`默认域名，这也是你后续添加自定义域名所要指向的`CNAME`值。

___

## 二、规则配置

### 2.1 资源基础配置（General）

进入CDN控制面板，第一个页面便是资源概览的`General`选项卡，在下方`Settings`栏右侧点击`Edit`可以进入资源的基础配置。前两项价格层级和WAF如其名，建议都保持默认（全部节点、无防护），关于WAF的内容放到第三节再单独解释。

![](https://cdn.luotianyi.vc/wp-content/uploads/2021-12-28_05-01-44.jpg)

接下来就是比较重要的绑定自己的域名，在`Alternate domain name`可以添加一个或多个自己的域名到这个资源，随后可以前往DNS将自定义域名通过CNAME指向前文提到的CloudFront默认域名。

![](https://cdn.luotianyi.vc/wp-content/uploads/2021-12-28_05-12-49.jpg)

随后同样是比较重要的HTTPS配置，我们先要前往`us-east-1`可用区的ACM（[点击前往](https://console.aws.amazon.com/acm/home?region=us-east-1#/certificates/list)）导入自己的证书或者签发Amazon的免费证书，注意这里必须是Virginia区域的ACM否则CloudFront将无法读取到你的证书。点击`Import`导入自己证书，`Requst`签发免费证书。

![](https://cdn.luotianyi.vc/wp-content/uploads/2021-12-28_05-18-34.jpg)

这里推荐签发Amazon的证书，有效期395天且支持泛域名。如图填入自己的域名，选择合适的验证方式添加对应的记录即可，验证完成后很快就能够签发成功。

![](https://cdn.luotianyi.vc/wp-content/uploads/2021-12-28_05-21-50.jpg)

准备好证书后回到我们的CDN设置页面，在`Custom SSL certificate`选项卡下选择你刚刚签发的证书，安全策略建议选择`TLSv1.1_2016`以获得较为广泛的兼容性。

![](https://cdn.luotianyi.vc/wp-content/uploads/2021-12-28_10-51-25.jpg)

最后是IPv6选项，面向的访客位于中国大陆时建议关掉，面向海外的话可以保持默认的开启状态。其他未提及的选项保持默认即可，至此CDN资源配置完成。

### 2.2 源站配置（Origins）

随后第二个选项卡即为源站设置，包括源站设置和包含主备切换功能的源站组设置。

![](https://cdn.luotianyi.vc/wp-content/uploads/2021-12-28_11-03-48.jpg)

在这里可以添加多个源站，如图在右上角点击`Create origin`可以创建新的源站。

![](https://cdn.luotianyi.vc/wp-content/uploads/2021-12-28_04-24-19.jpg)

和之前的新建流程一样，`Origin domain`填入一个解析到你源站的域名，可以利用`nip.io`项目在IP末尾添加`.nip.io`解析到源站IP，`Protcol`取源协议根据你的具体需求选择HTTP/HTTPS回源亦或者是匹配访问。此处需要注意若以HTTPS方式回源，源站必须配置有效、可信任的证书，否则节点取源会返回502错误。

![](https://cdn.luotianyi.vc/wp-content/uploads/2021-12-28_11-20-24.jpg)

源站组则可以将两个配置好的源站配置载入其中，一个作为主源站另一个作为热备源站，在源站请求出现如图错误参数时实现主备切换，若不需要这样的功能忽略这一步即可。

### 2.3 访问规则及缓存配置（Behaviors）

接下来就是CDN配置中最为关键的一步，设置访问行为`Behaviors`规则，由此可以实现路径与源站的匹配以及节点缓存功能。

![](https://cdn.luotianyi.vc/wp-content/uploads/2021-12-28_11-31-17.jpg)

配置Path pattern是本节内容的核心，在一开始创建的引导中我们已将默认路径`Default (*)`设置为不缓存任何内容，因此针对每一种我们需要缓存的内容都要单独配置缓存规则。

![](https://cdn.luotianyi.vc/wp-content/uploads/2021-12-28_11-47-27.jpg)

选定我们需要缓存或者排除的内容依赖的是`Path pattern`（区分大小写），与CloudFlare的页面规则匹配类似，主要是通配符`*`和`??`的应用，常用的几种见如下表格。随后的设置内容即仅针对匹配的访问路径，首先就是选择`2.2`中配置的源站或源站组，以此也可实现不同路径的不同来源。

<table><tbody><tr><td><strong>Path pattern</strong></td><td><strong>匹配的文件范围</strong></td></tr><tr><td>/files/<code>*</code></td><td>指定<code>/files/</code>路径下所有文件</td></tr><tr><td>/<code>*</code>.jpg</td><td>指定CDN资源内所有<code>jpg</code>后缀文件</td></tr><tr><td>/<code>*</code>.css<code>*</code></td><td>指定CDN资源内所有<code>css</code>后缀及包含参数的访问</td></tr><tr><td>/files/<code>*</code>.gif</td><td>指定<code>/files/</code>路径下所有<code>gif</code>后缀文件</td></tr><tr><td>/a<code>??</code>.mp3</td><td>指定CDN资源内以a开头的<code>mp3</code>后缀文件</td></tr></tbody></table>

![](https://cdn.luotianyi.vc/wp-content/uploads/2021-12-28_04-34-04.jpg)

接下来`Viewer protocol policy`也跟第一步一样根据你对外访问需要决定是否将HTTP跳转到HTTPS；`Allowed HTTP methods`为允许的请求头，同样根据实际需要选择，无法明确判定自己需要直接选择第三条全部支持即可。

<table><tbody><tr><td><strong>HTTP请求头</strong></td><td><strong>适用场景</strong></td></tr><tr><td><code>GET</code>, <code>HEAD</code></td><td>提供文件下载等</td></tr><tr><td><code>GET</code>, <code>HEAD</code>, <code>OPTIONS</code></td><td>纯静态网站或用于引用的静态资源等</td></tr><tr><td><code>GET</code>, <code>HEAD</code>, <code>OPTIONS</code>, <code>PUT</code>, <code>POST</code>, <code>PATCH</code>, <code>DELETE</code></td><td>WordPress等动态网站</td></tr></tbody></table>

![](https://cdn.luotianyi.vc/wp-content/uploads/2021-12-28_04-42-59.jpg)

接下来缓存设置`Cache key and origin requests`，匹配之前选定的路径及文件选择不缓存的`CachingDisabled`或最优缓存`CachingOptimized`规则；最后源请求策略依然设置为`AllViewer`将所有访客请求头转发到源站。其他给出的预置缓存规则均有针对性，也可以自己配置，这些针对性的内容若有需要请参考官方文档进行配置。

至此，通过`Path pattern`圈定范围并指定缓存规则后，我们可以实现对路径的缓存和对特定后缀文件的缓存。同样的，若缓存多个指定的后缀则需要依次添加多个规则，通过在`Behavior`页面移动规则上下指定其优先级（靠上者优先级更高）。

> 看到这里恭喜你，灵活运用以上内容你已经可以使用CloudFront作为CDN绝大多数的使用需求了~

___

## 三、其他要点

### 3.1 CDN计费细则

CloudFront的免费额度是每月1T流量和1000万次请求，计费项目包括HTTP请求、HTTPS请求、节点发往源站流量、节点流出到用户四部分流量，对于流量计费的理解如图所示。

![](https://cdn.luotianyi.vc/wp-content/uploads/2021-12-28_12-39-46.jpg)

使用CloudFront需要注意自己是否超出免费额度，超出后的价格是很贵的。账单是每天出一次，没有实时统计提供（控制面板的流量统计也有几个小时延迟），因此请不要将其用于易受攻击的站点。

![](https://cdn.luotianyi.vc/wp-content/uploads/2021-12-28_12-18-16.jpg)

需要注意，博主2022年2月账单更新后Bandwidth中区分出`Traffic to Origin`上行到源站流量条目，并开始以0.06USD/GB（亚太地区）计费，与文档描述一致回源流量不在免费范围内。请注意站点请求流量开销，AWS提供的免费额度不能完全覆盖正常使用，同时经过测试选用EC2作为源站也是无法避免此项开销的。

### 3.2 502/503/504错误

初次配置访问出现的错误一般如下：

> `502 ERROR` The request could not be satisfied.

这个时候首先检查CDN域名是否已添加到源站，然后确认前文`2.3`中源请求策略设置为`AllViewer`将所有访客请求头转发到源站（主要是`HOST`未发往源站导致的），最后当使用HTTPS回源时确认源站是否安装了有效的SSL证书。

### 3.3 WAF/ACL防护

CloudFront提供有WAF防护，在前文未提到的原因是它是付费的而且价格并不便宜，因此并不推荐使用。以设置一条访问速率限制规则为例，你需要每月支付5美金的WAF规则费用以及1美元/条用于限流的ACL规则，此外还有请求次数的费用。

![](https://cdn.luotianyi.vc/wp-content/uploads/2021-12-28_12-46-27.jpg)

另外也需要补充一句，在没有开启安全规则的时候对CloudFront恶意攻击是非常容易的，请务必注意设置信用卡限额、权衡安全规则，避免使用在易受攻击的场景。下图博主的一个朋友就是被刷了6亿请求数导致产生了800多美金的账单。

![](https://cdn.luotianyi.vc/wp-content/uploads/2022-01-04_03-47-30.png)

### 3.4 WebSocket支持

CloudFront能够完全支持WebSocket功能，请求次数不会额外收费。

### 3.5 网络优化配置

曾经在Lightsail CDN的体验中博主有提到关于网络的内容，AWS并未使用Anycast技术且在这段时间并未对网络做出改变，可以参考该文章第【`三`】部分。

#### 【CDN】AWS Lightsail CDN测试（免费一年）

前几天在上管理Lightsail的时候偶然看到上面的广告写了个Lightsail CDN，AWS的一贯机翻没有让人失望，博主在这里愣了一小会才反应过来…… AWS自己家的CDN就 ...

[https://luotianyi.vc/4324.html](https://luotianyi.vc/4324.html)

___

## 四、结语

CloudFront作为全球CDN巨头之一，提供的服务质量也是数一数二的，只是作为云服务厂商相比CloudFlare配置流程显得非常不友好。整体而言与CloudFlare各有优劣，如何选择可以自己权衡。

如果你有其他的技巧分享，或这篇文章存在错误的地方，也欢迎在评论区留言。在文章的最后，还是要说一句免费资源来之不易，且用且珍惜啦~

___

\*原创文章，转载请注明出处