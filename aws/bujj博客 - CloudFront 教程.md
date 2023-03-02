CloudFront 发布很久了，记得是和aws ec2 免费出栈流量变为100G 时候一起变的，作为 [AWS 免费使用套餐](https://aws.amazon.com/cn/free/)的一部分，每月可获得 1 TB 的数据传出、10,000,000 次 HTTP 和 HTTPS 请求以及 2000000 次 CloudFront 函数调用的使用配额

可以参见官方介绍：[https://aws.amazon.com/cn/cloudfront/](https://aws.amazon.com/cn/cloudfront/)

费用方面的情况：有一部分会收费（https附加费，cdn到用户，cdn到源站等）

具体参见：

[https://aws.amazon.com/cn/cloudfront/pricing/](https://aws.amazon.com/cn/cloudfront/pricing/)

[https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/CloudFrontPricing.html](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/CloudFrontPricing.html)

[![CloudFront](https://docs.aws.amazon.com/images/AmazonCloudFront/latest/DeveloperGuide/images/Charges.png)](https://docs.aws.amazon.com/images/AmazonCloudFront/latest/DeveloperGuide/images/Charges.png)

### 开始前的检查

首先谈一下账号配额，如果是32v及以上配额的账号是直接可以开的，

那言外之意就是，低于32v配额，不能直接开通 CloudFront，需要申请，但会验水电单什么的，所以，低v号要开通 CloudFront 的话要考虑好，自己是否能提供，提供的材料能否通过！

### 正式开始

一般大家都是https站点，那证书问题应当首先搞定

证书列表页：[https://us-east-1.console.aws.amazon.com/acm/home?region=us-east-1#/certificates/list](https://us-east-1.console.aws.amazon.com/acm/home?region=us-east-1#/certificates/list)

可以选择导入已有证书，也可选择请求证书，介绍下请求证书方面

#### 请求证书

打开：[https://us-east-1.console.aws.amazon.com/acm/home?region=us-east-1#/certificates/request/public](https://us-east-1.console.aws.amazon.com/acm/home?region=us-east-1#/certificates/request/public)

要配置泛域名的就像我这样填，具体域名的就具体写  
[![](https://www.bujj.org/wp-content/themes/CorePress/static/img/loading.gif)](https://img.bujj.org/202210262258153.png)

域名验证快的话，一两分钟，好了就可以开始配置cdn了

另外，AWS还提供私有证书： [https://us-east-1.console.aws.amazon.com/acm-pca/home?region=us-east-1#/](https://us-east-1.console.aws.amazon.com/acm-pca/home?region=us-east-1#/)，但是只有前30天免费，后面按月计费（不贵），这里不作过多介绍，有兴趣的自己去官网看： [https://aws.amazon.com/cn/private-ca/pricing/](https://aws.amazon.com/cn/private-ca/pricing/)

#### 配置cdn

打开：[https://us-east-1.console.aws.amazon.com/cloudfront/v3/home?region=us-east-1#/distributions/create](https://us-east-1.console.aws.amazon.com/cloudfront/v3/home?region=us-east-1#/distributions/create)

`源域`就是源站，不能用IP，只能填写域名，要想隐藏源站IP的，切记不可将源站解析到主域上

填好，往下滑，其他不用管，直到看到`查看器` ，按下图进行设置（强制https的动态站），自己可根据实际选  
[![](https://www.bujj.org/wp-content/themes/CorePress/static/img/loading.gif)](https://img.bujj.org/202210262307504.png)

`缓存键和源请求` 这里进行不缓存设置，如图  
[![](https://www.bujj.org/wp-content/themes/CorePress/static/img/loading.gif)](https://img.bujj.org/202210262310193.png)

再往下，看到粗体的`设置` ，主要是设置域名和证书  
[![](https://www.bujj.org/wp-content/themes/CorePress/static/img/loading.gif)](https://img.bujj.org/202210262322147.png)

别的没有什么了，直接默认即可，创建分配，好了就会给你一个`分配域名`  
[![](https://www.bujj.org/wp-content/themes/CorePress/static/img/loading.gif)](https://img.bujj.org/202210262332986.png)

按设置那里提交的域名，去进行CNAME解析到 xxxx.cloudfront.net

#### 配置SSL证书

这是最后一步了，就是给源站配上 SSL 证书，不是上一步申请的那个，而是要受信任的且有效的证书（Let's Encrypt，TrustAsia之类的都行），像 Cloudflare 那种自签证证书都是不行的（直接返回502）

成功的话，你的站点证书应该会显示为：`Amazon RSA 2048 M02`

#### 缓存和回源

上一步完成，站点已经可以正常访问了，基本结束了

但是若要针对特定的路径进行缓存和回源管理，可以设置点进`行为` 进行设置，不需要的这一步可以跳过去

[![](https://www.bujj.org/wp-content/themes/CorePress/static/img/loading.gif)](https://img.bujj.org/202210270011704.png)

创建一个行为。一般来说，我们希望某一个目录下的文件或者是图片等全缓存

`路径模式` 填写格式

比如希望站点目录下 wp-content 这个目录缓存一下，路径应为：`/wp-content/*`

缓存站点所有jpg图片：`*.jpg`

参考官方，写的非常详细：[https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/distribution-web-values-specify.html?icmpid=docs\_cf\_help\_panel#DownloadDistValuesPathPattern](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/distribution-web-values-specify.html?icmpid=docs_cf_help_panel#DownloadDistValuesPathPattern)

### 其他

其实到这里就结束了，但还想说一点，这样配置只有加速功能，是没有多少防护效果的

需要增强防护的，可以看下，这里不再赘述了

AWS WAF：[https://us-east-1.console.aws.amazon.com/wafv2/homev2](https://us-east-1.console.aws.amazon.com/wafv2/homev2)

文档：[https://docs.aws.amazon.com/waf/latest/developerguide/waf-chapter.html](https://docs.aws.amazon.com/waf/latest/developerguide/waf-chapter.html)

欢迎转载，转载务必加上[本页](https://www.bujj.org/index.php/2022/10/27/467/)链接，感激不尽