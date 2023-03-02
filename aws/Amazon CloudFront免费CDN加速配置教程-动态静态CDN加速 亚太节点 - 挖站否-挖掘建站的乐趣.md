[Amazon CloudFront](https://wzfou.com/tag/amazon-cloudfront/)是由亚马逊网络服务系统提供基础服务的一个内容分发网络（CDN）。其在欧洲、亚洲、北美、澳洲、南美、美国多个主要大城市多地拥有自己的数据中心，共107个网络边际服务点提供服务。Amazon旗下的CloudFront功能强大，用户众多。

[CloudFront](https://wzfou.com/tag/cloudfront/)提供的CDN加速有亚太加速节点，相对于我们来说，比较好的节点有：中国香港、吉隆坡、马来西亚、日本大阪、韩国首尔、新加坡、中国台湾台北、日本东京等等，这些CDN加速节点可以有效加快我们的网站访问速度，是我们理想的CDN加速效果。

CloudFront一直以来都有免费的额度，不过前一段时间，CloudFront免费层级从前 12 个月每月 50GB 流出 + 2,000,000 次 HTTP/HTTPS 请求改为永久 1TB 流出 + 10,000,000 次HTTP/HTTPS 请求+2,000,000 次 CloudFront 函数调用。这个免费额度对一般的网站来说足够了。

[![Amazon CloudFront免费CDN加速配置教程-动态静态CDN加速 亚太节点](https://wzfou.cdn.bcebos.com/wp-content/uploads/2022/12/CloudFront_00-680x366.png)](https://wzfou.cdn.bcebos.com/wp-content/uploads/2022/12/CloudFront_00.png)

这篇文章就来分享一下Amazon CloudFront设置网站[CDN加速](https://wzfou.com/cdn/)的方法，更多的[CDN加速](https://wzfou.com/cdn/)可以参考：

1.  [最新CloudFlare免费CNAME和IP接入教程-无需修改NS直接接入CloudFlare](https://wzfou.com/cloudflare-cname-cdn/)
2.  [国外十大CDN加速服务-适合网站全球CDN加速,防DDos攻击,企业个人建站使用](https://wzfou.com/guowai-cdn/)
3.  [CloudFlare自定义IP地址-优选本地高速IP地址 提升CloudFlare CDN速度](https://wzfou.com/cloudflare-y-ip/)

## 一、CloudFront申请开通

网站：

1.  https://aws.amazon.com/cn/cloudfront/

Amazon CloudFront也是Amazon AWS免费套餐的一部分，免费你想要找免费VPS主机，可以查看：[AWS免费VPS主机申请使用-Amazon EC2韩国日本香港机房VPS主机评测](https://wzfou.com/aws-vps-pingce/)。

进入到[Amazon CloudFront](https://wzfou.com/tag/amazon-cloudfront/)，然后添加你想要加速的网站域名，端口那里如果你同时有Https和Http访问，可以选择匹配。另外，选择了Https需要保证你的证书是有效的。

[![Amazon CloudFront添加域名](https://wzfou.cdn.bcebos.com/wp-content/uploads/2022/12/CloudFront_01-680x366.png)](https://wzfou.cdn.bcebos.com/wp-content/uploads/2022/12/CloudFront_01.png)

下面就是选择协议类型和Http了。第一条是访问设置`Viewer`。`Viewer protocol policy`根据你对外访问需要决定是否将HTTP跳转到HTTPS；`Allowed HTTP methods`为允许的请求头，可以根据实际需要选择，简单概括文件下载场景可选第一条、静态网站可选第二条、动态网站必须选择第三条，如果无法明确判定自己需要直接选择第三条全部支持即可。

[![Amazon CloudFront连接方式](https://wzfou.cdn.bcebos.com/wp-content/uploads/2022/12/CloudFront_02-680x366.png)](https://wzfou.cdn.bcebos.com/wp-content/uploads/2022/12/CloudFront_02.png)

对于CDN优化策略，默认的就行。

[![Amazon CloudFront优化策略](https://wzfou.cdn.bcebos.com/wp-content/uploads/2022/12/CloudFront_03-680x366.png)](https://wzfou.cdn.bcebos.com/wp-content/uploads/2022/12/CloudFront_03.png)

其他也保持默认即可，然后点击创建。

[![Amazon CloudFront加速设置](https://wzfou.cdn.bcebos.com/wp-content/uploads/2022/12/CloudFront_04-680x366.png)](https://wzfou.cdn.bcebos.com/wp-content/uploads/2022/12/CloudFront_04.png)

[![Amazon CloudFront默认设置](https://wzfou.cdn.bcebos.com/wp-content/uploads/2022/12/CloudFront_05-680x366.png)](https://wzfou.cdn.bcebos.com/wp-content/uploads/2022/12/CloudFront_05.png)

## 二、CloudFront CDN开启

### 2.1 CDN加速设置

CloudFront开通成功后，你就可以看到CloudFront为你生成的二级域名了，以下就是CloudFront的CDN管理中心。

[![Amazon CloudFront设置域名](https://wzfou.cdn.bcebos.com/wp-content/uploads/2022/12/CloudFront_06-680x366.png)](https://wzfou.cdn.bcebos.com/wp-content/uploads/2022/12/CloudFront_06.png)

点击编辑，可以对CloudFront CDN的设置参数进行重新调整。

[![Amazon CloudFront重新设置](https://wzfou.cdn.bcebos.com/wp-content/uploads/2022/12/CloudFront_07-680x366.png)](https://wzfou.cdn.bcebos.com/wp-content/uploads/2022/12/CloudFront_07.png)

打开CloudFront的域名域名，你就可以看到你的网站的图片等静态文件了。

[![Amazon CloudFront静态文件](https://wzfou.cdn.bcebos.com/wp-content/uploads/2022/12/CloudFront_18-680x366.png)](https://wzfou.cdn.bcebos.com/wp-content/uploads/2022/12/CloudFront_18.png)

### 2.2 申请SSL证书

Amazon CloudFront提供免费的SSL证书，如果你想让Amazon CloudFront使用你自己的域名，就需要申请Amazon CloudFront免费SSL证书了。

[![Amazon CloudFront申请SSL](https://wzfou.cdn.bcebos.com/wp-content/uploads/2022/12/CloudFront_08-680x366.png)](https://wzfou.cdn.bcebos.com/wp-content/uploads/2022/12/CloudFront_08.png)

点击请求SSL证书，下一步。

[![Amazon CloudFront请求SSL](https://wzfou.cdn.bcebos.com/wp-content/uploads/2022/12/CloudFront_09-680x366.png)](https://wzfou.cdn.bcebos.com/wp-content/uploads/2022/12/CloudFront_09.png)

然后选择DNS验证。

[![Amazon CloudFront验证DNS](https://wzfou.cdn.bcebos.com/wp-content/uploads/2022/12/CloudFront_10-680x366.png)](https://wzfou.cdn.bcebos.com/wp-content/uploads/2022/12/CloudFront_10.png)

接着，Amazon CloudFront就会给出一个CNAME记录，这个记录是专门用来验证你的域名所有权的。

[![Amazon CloudFront验证域名](https://wzfou.cdn.bcebos.com/wp-content/uploads/2022/12/CloudFront_11-680x366.png)](https://wzfou.cdn.bcebos.com/wp-content/uploads/2022/12/CloudFront_11.png)

到DNS解析商那里修改一下CNAME记录，待域名验证成功后，你的SSL证书就签发完成了。

[![Amazon CloudFront申请成功](https://wzfou.cdn.bcebos.com/wp-content/uploads/2022/12/CloudFront_12-680x366.png)](https://wzfou.cdn.bcebos.com/wp-content/uploads/2022/12/CloudFront_12.png)

**接下来就是到你的DNS解析商那里将你的域名添加CNAME记录，记录值就是Amazon CloudFront最先为你生成的二级域名。**

### 2.3 添加CDN域名

**注意：上面我们用了根域名通过CNAME的方式接入到Amazon CloudFront，**不过有些DNS解析商不支持根域名做CNAME记录。目前，已知国内的DNSPOD是可以的：[五个国内云主机DNS云解析服务对比-国内免费和付费DNS云解析服务](https://wzfou.com/gn-dns-jiexi/)。

对于不支持根域名CNAME的，或者仅仅想用自己的二级域名做为CDN加速域名，那么我们可以在CDN设置处额外添加一个域名。在备用域名CNAME处添加你的二级域名。

[![Amazon CloudFront二级域名加速](https://wzfou.cdn.bcebos.com/wp-content/uploads/2022/12/CloudFront_19-680x366.png)](https://wzfou.cdn.bcebos.com/wp-content/uploads/2022/12/CloudFront_19.png)

然后申请SSL证书。

[![Amazon CloudFront自定义SSL](https://wzfou.cdn.bcebos.com/wp-content/uploads/2022/12/CloudFront_16-680x366.png)](https://wzfou.cdn.bcebos.com/wp-content/uploads/2022/12/CloudFront_16.png)

最后，到你的域名DNS解析商处添加CNAME记录。

[![Amazon CloudFront设置CNAME记录](https://wzfou.cdn.bcebos.com/wp-content/uploads/2022/12/CloudFront_17-680x366.png)](https://wzfou.cdn.bcebos.com/wp-content/uploads/2022/12/CloudFront_17.png)

到源站配置（Origins）那里将你添加的二级域名绑定到源站中。

[![Amazon CloudFront修改源站](https://wzfou.cdn.bcebos.com/wp-content/uploads/2022/12/CloudFront_06-1-680x366.png)](https://wzfou.cdn.bcebos.com/wp-content/uploads/2022/12/CloudFront_06-1.png)

现在打开你的二级域名，就可以看到二级域名已经通过Amazon CloudFront接入CDN了。

[![Amazon CloudFront接入CDN](https://wzfou.cdn.bcebos.com/wp-content/uploads/2022/12/CloudFront_18-1-680x366.png)](https://wzfou.cdn.bcebos.com/wp-content/uploads/2022/12/CloudFront_18-1.png)

## 三、CloudFront CDN设置

### 3.1 缓存路径

我们在创建[CloudFront CDN](https://wzfou.com/tag/cloudfront-cdn/)时默认是开启全站路径缓存的，如果你想单独对某一些网站URL路径设置缓存，那么可以在CDN路径中进行设置。

[![Amazon CloudFront模式](https://wzfou.cdn.bcebos.com/wp-content/uploads/2022/12/CloudFront_14-680x366.png)](https://wzfou.cdn.bcebos.com/wp-content/uploads/2022/12/CloudFront_14.png)

我们需要缓存或者排除的内容依赖的是`Path pattern`（区分大小写），主要是通配符`*`和`??`的应用，规则如下：

> Path pattern  匹配的文件范围
> 
> /files/\* 指定/files/路径下所有文件
> 
> /\*.jpg 指定CDN资源内所有jpg后缀文件
> 
> /\*.css\* 指定CDN资源内所有css后缀及包含参数的访问
> 
> /files/\*.gif 指定/files/路径下所有gif后缀文件
> 
> /a??.mp3 指定CDN资源内以a开头的mp3后缀文件

### 3.2  黑白名单

Amazon CloudFront可以对访问地区进行限制。

[![Amazon CloudFront位置](https://wzfou.cdn.bcebos.com/wp-content/uploads/2022/12/CloudFront_15-680x366.png)](https://wzfou.cdn.bcebos.com/wp-content/uploads/2022/12/CloudFront_15.png)

## 四、CloudFront CDN效果

CloudFront CDN提供了亚太节点，以下为CloudFront CDN使用后访问到节点，效果还是不错的。

[![Amazon CloudFront加速效果](https://wzfou.cdn.bcebos.com/wp-content/uploads/2022/12/CloudFront_22-680x366.png)](https://wzfou.cdn.bcebos.com/wp-content/uploads/2022/12/CloudFront_22.png)

[![Amazon CloudFront亚太节点](https://wzfou.cdn.bcebos.com/wp-content/uploads/2022/12/CloudFront_21-680x366.png)](https://wzfou.cdn.bcebos.com/wp-content/uploads/2022/12/CloudFront_21.png)

## 五、CloudFront CDN费用

虽然[Amazon CloudFront](https://wzfou.com/tag/amazon-cloudfront/)是免费的，但是Amazon CloudFront有一定的免费额度，使用CloudFront需要注意自己是否超出免费额度，超出后的价格是相当贵的。

[![Amazon CloudFront费用](https://wzfou.cdn.bcebos.com/wp-content/uploads/2022/12/CloudFront_23-680x366.png)](https://wzfou.cdn.bcebos.com/wp-content/uploads/2022/12/CloudFront_23.png)

请注意站点请求流量开销，AWS提供的免费额度不能完全覆盖正常使用，同时经过测试选用EC2作为源站也是无法避免此项开销的。账单是每天出一次，记得及时关注自己的费用。

## 六、总结

[CloudFront CDN](https://wzfou.com/tag/cloudfront-cdn/)作为优秀的CDN服务商，提供的免费额度基本上足够一个小博客使用的，但是大家在使用的过程中一定要超出免费额度会产生费用，没有开启安全规则的时候对CloudFront恶意攻击是非常容易的，一旦超标很容易被扣款。

另外，在使用 CloudFront 出现以下错误时，请直接发工单提交客服解决。

> Your account must be verified before you can add new CloudFront resources. To verify your account, please contact AWS Support (https://console.aws.amazon.com/support/home#/) and include this error message.

文章出自：[挖站否](https://wzfou.com/) [https://wzfou.com/cloudfront/](https://wzfou.com/cloudfront/)，版权所有。本站文章除注明出处外，皆为作者原创文章，可自由引用，但请注明来源。