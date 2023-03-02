> **本文适合阅读对象****：**对网站架构，及互联网应用有一定的基础技术背景的外贸运营人员。  
> 如果你是外贸业务员，对网站架构，SSL，及[域名解析](https://www.mrsmoke.net/tags-domain-resolution.html)都不太了解的话，可以就此跳过。（因为会让你比较痛苦）

做[外贸网站](https://www.mrsmoke.net/tags-foreign-trade-website.html)，如果你的网站访问速度慢，会在一定程度上影响到你的成交转化率，因为在不同地区访问你网站的速度是不一样的。比如你的服务器在美国，你在中国访问挺快的，但是在俄罗斯，或者是其他一些地区访问速度就不那么快了。 因为在美国有些机房是专门针对中国地区做过速度优化的，所以千万不能说，我访问速度很快，客户就访问也很快。这不是一定的。

对于外贸网站设置[CDN](https://www.mrsmoke.net/tags-cdn.html)加速，[什么是CDN？](https://www.mrsmoke.net/website-operation/aws-cloudfront-cdn-configuration-method-in-detail.html#) （CDN的全称是Content Delivery Network，即内容分发网络）市面上有机种接入方式，用的比较多的就是 [Cloudflare](https://www.mrsmoke.net/tags-cloudflare.html) CDN，AWS CDN 以及[阿里云](https://www.mrsmoke.net/tags-ali-cloud.html)全球加速，其中Cloudflare是免费的，但是他有收费更高需求的配置。 AWS 和 阿里云则都是提供一定的免费额度，然后超过用量就收费。在我另外一篇文章里，有这3家CDN的测试对比效果，[你可以点击此链接去阅读](https://www.mrsmoke.net/website-operation/aws-cloudfront-cdn-configuration-method-in-detail.html#)。  

## AWS  CloudFront  详细配置目录

一，[宝塔](https://www.mrsmoke.net/tags-%E5%AE%9D%E5%A1%94.html)面板用户，采用CMS系统网站配置方法  
二，[采用wordpress CMS的网站配置方法](https://www.mrsmoke.net/website-operation/aws-cloudfront-cdn-configuration-method-in-detail.html#)  
三，服务器本身就在亚马逊的网站配置方法  （这个不详细讲解，因为大部分人服务器都不在AWS）

## 宝塔面板用户，采用CMS系统网站配置方法  

市面上有很多种AWS对于各项功能均有详细的官方文档可供参考，但官方文档过于繁杂，本文将就CloudFront服务的使用流程作简单说明。  
本文主要讲解宝塔用户的CMS网站的配置，如果您采用的是[Wordpress](https://www.mrsmoke.net/tags-wordpress.html) CMS 系统的话，可以点文章最下面的扩展阅读链接转到相关页面。  

### Amazon CloudFront 分布节点

> 其在欧洲、亚洲、北美、澳洲、南美、美国多个主要大城市多地拥有自己的数据中心，共 107 个网络边际服务点（Edge Servers，即边缘服务器）提供服务。它可以加快将静态和动态 Web 内容（如 .html、.css、.js 和图像文件）分发到用户的速度，即当用户请求您用 CloudFront 提供的内容时，用户被路由到提供最低延迟 (时间延迟) 的边缘服务器，从而以尽可能最佳的性能传送内容。

### 资费

[官方国际版资费](https://aws.amazon.com/cn/cloudfront/pricing/)  
[![](https://www.mrsmoke.net/zb_users/upload/2021/04/202104271619453650527092.png)](https://www.mrsmoke.net/zb_users/upload/2021/04/202104271619453650527092.png "外贸网站海外加速 AWS  CloudFront  CDN详细配置方法")  
按流量均分来算，开最低配 EC2（$43.56），大约每月可使用传入中国大陆地区流量 33G（$4）。

### 部署

本次操作主要目的是，用我们在宝塔上的网站来配置AWS的 CDN 加速

### 证书  

AWS CDN 需要配置证书，无论你的网站之前是有证书，还是在AWS上新申请证书都可以，本文讲的是用宝塔网站配置，宝塔原先就带有Let's Encrypt SSL免费证书，AWS 也可以创建 免费SSL证书，但是需要申请验证等一系列操作，所以我们为了方便就使用宝塔自带的Let's Encrypt SSL免费证书。

1\. 打开ACM服务，点击导入证书。 [点这里直达](https://console.aws.amazon.com/acm/home?region=us-east-1#/?id=arn:aws:acm:us-east-1:429928425236:certificate%2F057c6e13-c339-45dd-bcac-32e254d6dfc3)  
[![](https://www.mrsmoke.net/zb_users/upload/2021/04/202104271619453662652171.png)](https://www.mrsmoke.net/zb_users/upload/2021/04/202104271619453662652171.png "外贸网站海外加速 AWS  CloudFront  CDN详细配置方法")  
2\. 填写证书内容。  
首页找到我们在宝塔网站里面的SSL 设置页面COPY里面的密匙（KEY）和证书（PEM格式）里面的内容到 AWS  
[![](https://www.mrsmoke.net/zb_users/upload/2021/04/202104271619453684721441.png)](https://www.mrsmoke.net/zb_users/upload/2021/04/202104271619453684721441.png "外贸网站海外加速 AWS  CloudFront  CDN详细配置方法")  
3，宝塔复制过来的粘贴到AWS相应的地方，点击审核并导入即可。  

> **特别要注意的是：**宝塔上面的证书（PEM格式）里面是分为2段内容，COPY第一段到AWS的正文里面。COPY第二段到AWS里面的证书链里面。

[![](https://www.mrsmoke.net/zb_users/upload/2021/04/202104271619453700331379.png)](https://www.mrsmoke.net/zb_users/upload/2021/04/202104271619453700331379.png "外贸网站海外加速 AWS  CloudFront  CDN详细配置方法")  
提交后大约等待几分钟就会显示状态为已颁发，就代表你的证书已经导入到AWS里面了。  
[![](https://www.mrsmoke.net/zb_users/upload/2021/04/202104271619453712955973.png)](https://www.mrsmoke.net/zb_users/upload/2021/04/202104271619453712955973.png "外贸网站海外加速 AWS  CloudFront  CDN详细配置方法")

### **分配**

证书导入好以后，我们就可以楷书部署分配 CloudFront了。

打开 [CloudFront管理界面](https://console.aws.amazon.com/cloudfront/home?region=us-west-1#distributions:)，点击创建分配  

[![](https://www.mrsmoke.net/zb_users/upload/2021/04/202104271619453723192572.png)](https://www.mrsmoke.net/zb_users/upload/2021/04/202104271619453723192572.png "外贸网站海外加速 AWS  CloudFront  CDN详细配置方法")

  
分发方式选择Web:  
[![](https://www.mrsmoke.net/zb_users/upload/2021/04/202104271619453734583987.png)](https://www.mrsmoke.net/zb_users/upload/2021/04/202104271619453734583987.png "外贸网站海外加速 AWS  CloudFront  CDN详细配置方法")  

源设置这里，源域名是指专门分配给 CloudFront 来加速真正网站的域名，不是实际要开放给大家访问的真正网站。什么意思呢，就是说如果开放访问的真正做站的域名是 www.你的域名.com，那么在这里设置的源域名就不可以是 www.你的域名.com，否则会无限循环；我们可以增加一个子域名作为源，可以设置为 cdn.你的域名.com,  然后用这个二级域名解析A记录到你的IP。下面就设置红框里面的就可以，其他默认。  

[![](https://www.mrsmoke.net/zb_users/upload/2021/04/202104271619453747140993.png)](https://www.mrsmoke.net/zb_users/upload/2021/04/202104271619453747140993.png "外贸网站海外加速 AWS  CloudFront  CDN详细配置方法")  
默认缓存行为设置：可以根据实际情况选择  **查看器协议策略**，**源请求策略**，每个策略具体作用可以点下面的 “**查看策略详细信息**”  
根据红框里面的信息设置好，其他按默认  
[![](https://www.mrsmoke.net/zb_users/upload/2021/04/202104271619453758357428.png)](https://www.mrsmoke.net/zb_users/upload/2021/04/202104271619453758357428.png "外贸网站海外加速 AWS  CloudFront  CDN详细配置方法")  

分配设置这里，备用域名（CNA\[原\]MES）需要填写你现在网站的访\[创\]问域名  www.你的域名.com&nb\[文\]sp;  你的域名.com,\[章\]如果没有的话就只能使用Clou\[来\]dFront分配的域名；

SSL证书选择前面导入的证书；其他配置保持默认，然后点击创建分配即可  

[![](https://www.mrsmoke.net/zb_users/upload/2021/04/202104271619453770597319.png)](https://www.mrsmoke.net/zb_users/upload/2021/04/202104271619453770597319.png "外贸网站海外加速 AWS  CloudFront  CDN详细配置方法")  
等待一会儿后，待分配状态变成已部署后，CloudFront CDN 就部署成功可以使用了。  
[![](https://www.mrsmoke.net/zb_users/upload/2021/04/202104271619453781489998.png)](https://www.mrsmoke.net/zb_users/upload/2021/04/202104271619453781489998.png "外贸网站海外加速 AWS  CloudFront  CDN详细配置方法")  
解析域名到源域名

CloudFront CDN 部署成功后会分配一个\*.cloudfront.net 的域名：   
[![](https://www.mrsmoke.net/zb_users/upload/2021/04/202104271619453793261890.png)](https://www.mrsmoke.net/zb_users/upload/2021/04/202104271619453793261890.png "外贸网站海外加速 AWS  CloudFront  CDN详细配置方法")  
1,把cdn的二级域名做A记录解析到你的IP，这个就相当于源站

2,把WWW做CNAME指向到 AWS分配给你的域名xxxxxx.cloudfront.net   
[![](https://www.mrsmoke.net/zb_users/upload/2021/04/202104271619453803619139.png)](https://www.mrsmoke.net/zb_users/upload/2021/04/202104271619453803619139.png "外贸网站海外加速 AWS  CloudFront  CDN详细配置方法")  
解析生效后，我们就可以访问使用 CloudFront CDN加速的网站了。

### 测试

最后去ping一下自己的域名看到IP已经变成不同的国家地区都会由AWS当地的服务器来进行分发加速了。

[![](https://www.mrsmoke.net/zb_users/upload/2021/04/202104271619453816130902.png)](https://www.mrsmoke.net/zb_users/upload/2021/04/202104271619453816130902.png "外贸网站海外加速 AWS  CloudFront  CDN详细配置方法")

## 扩展阅读 其他CDN详细配置方法

一，[外贸网站阿里云全球加速设置](https://www.mrsmoke.net/website-operation/ali-cloud-cdn-configuration-method.html)  
二，[外贸网站Cloudflare免费全球CDN配置](https://www.mrsmoke.net/website-operation/cloudflare-configuration.html)