2023年2月17日 12:13:53394

[![abcproxy](https://jishubai.com/wp-content/ads/abcproxy.gif)](https://www.abcproxy.com/?utm-source=js&utm-keyword=?01)

CloudFront、由亚马逊网络服务公司提供的内容分发网络服务（CDN），跟著名的CloudFlare一样都简称CF，很容易让人搞混两者的区别。其中AWS CF跟Gcore CDN一样提供每月1T的网络加速流量+1千万次的请求、支持WebSocket协议，对于一般的网站来说已经足够使用，不过使用AWS CF需要有个AWS账户同时绑定信用卡，同时它的免费额度不能完全涵盖正常使用，使用不当的情况下会产生付费账单，开通后请注意免费额度的消耗和账单提示。

![AWS 最新永久免费CDN服务 — CloudFront配置CDN加速网站访问图文指南](https://jishubai.com/wp-content/uploads/2023/02/1676605951-image.png)

Amazon CloudFront地址：[https://aws.amazon.com/cn/cloudfront/](https://aws.amazon.com/cn/cloudfront/)文章源自技术白-https://jishubai.com/1210.html

Amazon CloudFront 定价：[https://aws.amazon.com/cn/cloudfront/pricing/](https://aws.amazon.com/cn/cloudfront/pricing/)文章源自技术白-https://jishubai.com/1210.html

#### CloudFront申请开通

1、打开CloudFront地址后登录自己的AWS账户，点击“创建CloudFront”，32v及以上配额的账号可以直接开通，新账号配额较低可能需要填写水电单验证申请；文章源自技术白-https://jishubai.com/1210.html

![AWS 最新永久免费CDN服务 — CloudFront配置CDN加速网站访问图文指南](https://jishubai.com/wp-content/uploads/2023/02/1676601054-image.png)

2、添加源域、也就是源站（这个域名需要已经解析到源服务器IP上面，不建议采用主域名或常见域名直接解析到源站），协议如果同时有http和https访问则选择匹配查看器；文章源自技术白-https://jishubai.com/1210.html

![AWS 最新永久免费CDN服务 — CloudFront配置CDN加速网站访问图文指南](https://jishubai.com/wp-content/uploads/2023/02/1676601244-image.png)

3、接下来配置缓存模式及协议类型，主要调整下允许的HTTP方法，文件下载场景可选第一条、静态网站可选第二条、动态网站必须选择第三条，如果无法明确判定自己需要直接选择第三条全部支持即可。文章源自技术白-https://jishubai.com/1210.html

![AWS 最新永久免费CDN服务 — CloudFront配置CDN加速网站访问图文指南](https://jishubai.com/wp-content/uploads/2023/02/1676601469-image.png)

4、然后其它选项可以保持默认，直接到设置选项里面编辑加速地点和SSL证书，默认使用全球加速节点，也可以单独对部分地区进行加速，如果网站使用了https访问，需要配置证书、这里后面说，先点击创建；文章源自技术白-https://jishubai.com/1210.html

![AWS 最新永久免费CDN服务 — CloudFront配置CDN加速网站访问图文指南](https://jishubai.com/wp-content/uploads/2023/02/1676601846-image.png)

#### 开启CDN

1、CloudFront创建成功后，就可以看到CloudFront生成分配的二级域名，同时也可以点击编辑重新修改各类参数；文章源自技术白-https://jishubai.com/1210.html

![AWS 最新永久免费CDN服务 — CloudFront配置CDN加速网站访问图文指南](https://jishubai.com/wp-content/uploads/2023/02/1676602674-image.png)

2、此时打开CloudFront分配的域名访问测试，正常情况下就能访问到你的网站了，当然这里还需要到DNS解析商修改自己的主域名也就是访问域名的CNAME记录以提供给网友访问，如果出现502错误那就是SSL证书问题；文章源自技术白-https://jishubai.com/1210.html

![AWS 最新永久免费CDN服务 — CloudFront配置CDN加速网站访问图文指南](https://jishubai.com/wp-content/uploads/2023/02/1676604523-image.png)

#### SSL证书申请

1、AWS CF提供免费的SSL证书，点击编辑然后请求证书，已有其它证书的情况下也可以导入证书使用；文章源自技术白-https://jishubai.com/1210.html

![AWS 最新永久免费CDN服务 — CloudFront配置CDN加速网站访问图文指南](https://jishubai.com/wp-content/uploads/2023/02/1676603328-image.png)

2、如果只给单个域名配置证书就填那一个，需要配置泛域名证书就如下图所示添加，选择DNS验证然后提交；文章源自技术白-https://jishubai.com/1210.html

![AWS 最新永久免费CDN服务 — CloudFront配置CDN加速网站访问图文指南](https://jishubai.com/wp-content/uploads/2023/02/1676603740-image.png)

3、转到域名注册商家那边，修改域名DNS解析记录验证（参数在上面提交后有显示），待域名验证成功后，你的SSL证书就签发完成了，然后在自定义SSL里面刷新选中即可；文章源自技术白-https://jishubai.com/1210.html

![AWS 最新永久免费CDN服务 — CloudFront配置CDN加速网站访问图文指南](https://jishubai.com/wp-content/uploads/2023/02/1676604089-image.png)

最后CloudFront CDN加速就已经配置完成，但这样配置只有加速功能，而没有什么防护效果，如果需要防护则要配置AWS WAF功能，不过价格比较贵，这个后续有机会再说明。文章源自技术白-https://jishubai.com/1210.html 文章源自技术白-https://jishubai.com/1210.html

**本站QQ群**：[186764881](https://qm.qq.com/cgi-bin/qm/qr?k=ayGEOoRzzedo2UE-X_fjaVEgIrSEzs8D&jump_from=webapi&authKey=3HmJ0eXHDHak755sZJjNhzDZEQcE+OyHgJvy5hYS0rTYTHUE62r4mSdtTpM9/h9h)   **微信咨询**：lspnbpro（有偿协助，白嫖勿扰)   **杂货铺**：[曙光商店](https://dawnshop.xyz/)

[![列表AD](https://jishubai.com/wp-content/uploads/2023/02/1675248806-%E5%88%97%E8%A1%A8%E5%A4%B4%E9%83%A8AD.png)](https://jishubai.com/1210.html#)