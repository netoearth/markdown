从2021年11月开始，CloudFlare禁用了Partner使用的`zone_set`API以避免滥用（因为该API接入不需要验证域名所有权），通过Partner实现CNAME接入的方式近乎落幕，仅剩Plesk空间存量的`ServerShield by Cloudflare`插件订阅能够正常接入。几个月过去了，官方也并没有对未来合作伙伴如何进行接入新域名给出任何解释。

![](https://cdn.luotianyi.vc/wp-content/uploads/2022-03-20_05-22-03.jpg)

错过了Partner的时代，官方的CloudFlare for SaaS也提供了一种更灵活的CNAME接入方式，一起来看看吧。

___

## 一、功能简介

CloudFlare for SaaS不是个新功能，这里单独拿出来讲，主要是几天前CF调整了一波免费额度。过去是每个域名收取2USD/月的费用，现在不仅提供100个域名免费额度，而且超额后每个域名仅按0.1USD/月收取费用，非常良心。

> **官方公告**：[https://blog.cloudflare.com/waf-for-saas/](https://blog.cloudflare.com/waf-for-saas/)

CloudFlare中一个完全接入的域名即为一个`zone`，点进去包括套餐、安全等等都是针对这一主域名配置的。官方SaaS功能针对的是你服务的客户，开放这项功能允许使用他们自己的域名直接附加在你的`zone`里，享受你`zone`包含的安全、加速等功能。

![](https://cdn.luotianyi.vc/wp-content/uploads/2022-03-20_04-11-39.jpg)

说起来可能不是很直白，这里举几个应用场景的例子：

> ★**应用场景1**：`a.com`通过NS接入了CF，`b.com`未接入CF；可以通过SaaS功能实现`1.b.com`/`2.b.com`等直接附加在`a.com`上，通过CNAME指向CF的节点。
> 
> ★**应用场景2**：`a.com`通过Plesk接入了CF，具有免费的Plesk Plus版本，`b.com`未接入CF或使用的免费版；可以通过SaaS功能实现`1.b.com`/`2.b.com`等直接附加在`a.com`上，享受`a.com`域名下的ECC+RSA双证书、页面规则、高级防火墙权益。

简而言之，可以通过这项功能，实现其他域名的CNAME接入以及对`zone`权益的共享，有兴趣的话，接着往下看吧~

___

## 二、配置接入

### 订阅CloudFlare for SaaS

打开一个域名，选择【SSL/TLS】下的【自定义主机名】，点击【启用CloudFlare for SaaS】后根据指示绑定外币卡或者PayPal，订阅CloudFlare for SaaS功能。

![](https://cdn.luotianyi.vc/wp-content/uploads/2022-03-20_04-16-44.jpg)

CloudFlare for SaaS订阅本身是针对整个计费账户的，所以通过Partner接入的域名出现【请联系客户成功经理以启用适用于SaaS的SSL】时，只需要选择个通过官方NS激活的域名启用订阅后即可使用。这里猜测可能是Partner接入的商务权限交给了合作伙伴，方便下放优惠和服务那些，我们绕过去就行了。![](https://cdn.luotianyi.vc/wp-content/uploads/2022-03-20_04-20-22.jpg)

激活页面中文翻译比较滞后，从英文的可以看到免费额度已经进行更新，可以放心使用。

![](https://cdn.luotianyi.vc/wp-content/uploads/2022-03-20_04-24-35.jpg)

### 设置源站

选择一个承载的域名`zone`点进去，依然是【SSL/TLS】下的【自定义主机名】，首先要设置附加上域名的源站。在这之前要在承载的域名`zone`中设置一个子域名作为源站的来源，比如`origin.a.com`，在Partner或者官方DNS设置好它的源站（注意是是在CF里添加，和正常添加网站的流程一样）。

![](https://cdn.luotianyi.vc/wp-content/uploads/2022-03-20_04-33-50.jpg)

SaaS这里的源站叫回退源（Fallback Origin），输入刚才设置的子域名并点击【Add Fallback Origin】，它会同步这个子域名设置的源站作为后续在此接入域名的源站。有些人就会问了，这样设置那不是后续SaaS添加的所有其他域名就只能用同一个源站了？答案确实是这样，为每个SaaS域名自定义源站需要Enterprise以上套餐，有多域名需求多开几个zone吧（苦笑）。

![](https://cdn.luotianyi.vc/wp-content/uploads/2022-03-20_04-36-57.jpg)

### 添加自定义主机名

后续的工作就很简单了，点击【添加自定义主机名】，输入你要添加的未在CF接入的子域名。建议直接选择TXT验证，因为除了证书还有另一条TXT记录要添加，一起加上去比较方便。

![](https://cdn.luotianyi.vc/wp-content/uploads/2022-03-20_04-42-43.jpg)

### 验证域名所有权

添加完成后，按要求解析证书和主机名两个TXT记录，解析生效后10分钟左右即可验证通过，到此这个SaaS域名就正确的添加到了你的`zone`中并接入了CF。

![](https://cdn.luotianyi.vc/wp-content/uploads/2022-03-20_04-46-28.jpg)

特别提醒，如图这里CF给出的验证TXT名称是应完整域名的解析记录，所以在自己的第三方DNS配置的时候，填入的主机名应当是`example`和`_cf-custom-hostname.example`，如果直接复制框内的内容把根域名`b.com`填进了主机名全域就变成了`example.b.com.b.com`了，是错误的。配置完成之后你可以通过直接复制的域名来检查TXT记录是否匹配，推荐MySSL的工具（[点击前往](https://myssl.com/dns_check.html#ssl_verify)）。

![](https://cdn.luotianyi.vc/wp-content/uploads/2022-11-03_08-24-53-e1667463970256.png)

经过测试，不论以何种方式验证并签发证书，只要保持SSL正确指向CloudFlare，系统就能够在到期前一个月自动为你续期证书，无需进行手动的操作。相应的，SaaS页面`Certificate validation method`也会变成`HTTP Validation`。

### SaaS域名解析

添加进去的SaaS域名，CF并不会给你提供明确的CNAME供指向。如果是官方接入的可以直接CNAME到你刚刚设置的源站域名比如`origin.a.com`，通过Partner接入的直接解析到源域名对应的CNAME比如`origin.a.com.cdn.cloudflare.net`即可。其他的配置比如分线路解析、自选IP就可以按照自己的喜好去设置了，在此不过多赘述。

此外，对于防火墙规则、页面规则，直接将添加进的域名输入其中即可圈定范围，完成对于其细则的设置。

___

## 三、结语

CloudFlare for SaaS是官方提供的一项非常方便的免费功能，弥补了早期未通过Partner接入只能强制NS接入的缺憾。有官方保障、灵活CNAME、免费的优点，也有源站不灵活等缺点，肯定还是不如已经通过Partner/Plesk接入的域名灵活。

最后要感谢CF提供这样的免费功能，希望未来能够下放自定义源站的功能（想桃子），也欢迎大家在评论区分享你们对这项功能的其他应用方式，一起学习交流吧~

___

\*原创文章，转载请注明出处