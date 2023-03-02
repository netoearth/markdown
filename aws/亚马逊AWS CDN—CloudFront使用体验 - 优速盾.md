[优速盾-小U](https://www.cdnb.net/bbs/gerenzhongxin/cdnb) • 2022年8月3日 12:53 • [使用教程](https://www.cdnb.net/bbs/archives/category/yousudun/shiyongjiaocheng) • 阅读 375

## [](https://www.cdnb.net/bbs/tag/15)**[**cdn**](https://www.cdnb.net/bbs/go?_=fafb2576f8aHR0cHM6Ly93d3cuY2RuYi5uZXQv)**概念

CDN通过广泛的网络节点分布，提供快速、稳定、安全、可编程的全球[**内容分发**](https://www.cdnb.net/bbs/go?_=fafb2576f8aHR0cHM6Ly93d3cuY2RuYi5uZXQv)加速服务，支持将网站、音视频、下载等内容分发至接近用户的节点，使用户可就近取得所需内容，提高用户访问的响应速度和成功率（阿里云产品介绍）。

对于海外服务器，网上介绍的CDN服务商基本就cloudflare，imperva，和亚马逊的cloudfront。cloudfront支持S3存储桶和自定义源域名，配置起来也很方便。国内貌似也有支持[**CDN加速**](https://www.cdnb.net/bbs/tag/37)的服务商，不过节点可能没有那么多。

## 具体配置

登录AWS后，搜索CloudFront，进入CloudFront控制台。点‘创建分配’。

[![](https://www.cdnb.net/bbs/wp-content/uploads/2022/08/20210517184315931.png "亚马逊AWS CDN—CloudFront使用体验插图1")](https://www.cdnb.net/bbs/wp-content/uploads/2022/08/20210517184315931.png)

因为绝大部分都是Web方式，感觉步骤一就挺鸡肋的（根本没得选），点‘入门’直接到步骤二

[![](https://www.cdnb.net/bbs/wp-content/uploads/2022/08/20210517184650258.png "亚马逊AWS CDN—CloudFront使用体验插图3")](https://www.cdnb.net/bbs/wp-content/uploads/2022/08/20210517184650258.png)

这一页看起来配置很多，其实主要就两项（AWS的特色？）：源域名，备用域名。

### 源域名

需要加速的[**源服务器**](https://www.cdnb.net/bbs/tag/31)的域名。注意。这个域名不能和提供给用户访问的自定义域名一样，原因参考这篇文章：[AWS CloudFront / 亚马逊CDN使用教程 – 2206 – 博客园](https://www.cdnb.net/bbs/go?_=33c97399adaHR0cHM6Ly93d3cuY25ibG9ncy5jb20vdzIyMDYvcC85OTEwMjQ2Lmh0bWw%3D "AWS CloudFront / 亚马逊CDN使用教程 - 2206 - 博客园")  
有S3存储桶的直接在列表里选择，直接输入域名就是自定义的，路径非必选。

[![](https://www.cdnb.net/bbs/wp-content/uploads/2022/08/20210827115329665.png "亚马逊AWS CDN—CloudFront使用体验插图5")](https://www.cdnb.net/bbs/wp-content/uploads/2022/08/20210827115329665.png)

[![](https://www.cdnb.net/bbs/wp-content/uploads/2022/08/a6d1c2b6b54c47088a09f78ef53f6455.png "亚马逊AWS CDN—CloudFront使用体验插图7")](https://www.cdnb.net/bbs/wp-content/uploads/2022/08/a6d1c2b6b54c47088a09f78ef53f6455.png)

另外协议选了https或匹配查看器的话，源域名也要配置nginx的ssl证书

### 备用域名

如果需要通过自定义域名访问（如：http://res.example.com），就要填写备用域名，导入自定义ssl证书，或使用ACM公用证书。 这一步会将路由指向CloudFront生成的域名，还需再去route53配置CNames指向生成的dxxxxxxxxx.cloudfront.net

[![](https://www.cdnb.net/bbs/wp-content/uploads/2022/08/20210519112152160.png "亚马逊AWS CDN—CloudFront使用体验插图9")](https://www.cdnb.net/bbs/wp-content/uploads/2022/08/20210519112152160.png)

创建完毕后，也可以直接使用CloudFront随机生成的域名：dxxxxxxxxx.cloudfront.net

如果是用https访问，要选择自定义域名对应的ssl证书。

如果是网页应用，默认根对象（类似主页）也可以配置一下。

[![](https://www.cdnb.net/bbs/wp-content/uploads/2022/08/20210519112836793.png "亚马逊AWS CDN—CloudFront使用体验插图11")](https://www.cdnb.net/bbs/wp-content/uploads/2022/08/20210519112836793.png)

点击‘创建分配’，等到状态变为‘已部署’，就可使用[CDN加速](https://www.cdnb.net/bbs/tag/37 "CDN加速")了。

[![](https://www.cdnb.net/bbs/wp-content/uploads/2022/08/2021051810045882.png "亚马逊AWS CDN—CloudFront使用体验插图13")](https://www.cdnb.net/bbs/wp-content/uploads/2022/08/2021051810045882.png)

## 验证效果

先向[源服务器](https://www.cdnb.net/bbs/tag/31 "源服务器")上传一张高清大图，然后用postman请求

未开启CDN加速：

[![](https://www.cdnb.net/bbs/wp-content/uploads/2022/08/20210519114105416.png "亚马逊AWS CDN—CloudFront使用体验插图15")](https://www.cdnb.net/bbs/wp-content/uploads/2022/08/20210519114105416.png)

开启CDN加速后：

[![](https://www.cdnb.net/bbs/wp-content/uploads/2022/08/20210519114202891.png "亚马逊AWS CDN—CloudFront使用体验插图17")](https://www.cdnb.net/bbs/wp-content/uploads/2022/08/20210519114202891.png)

也可以直接用浏览器验证，不过记得先清一下浏览器缓存，不然看不出效果

无CDN加速：

[![](https://www.cdnb.net/bbs/wp-content/uploads/2022/08/20210519114505104.png "亚马逊AWS CDN—CloudFront使用体验插图19")](https://www.cdnb.net/bbs/wp-content/uploads/2022/08/20210519114505104.png)

加速后：

[![](https://www.cdnb.net/bbs/wp-content/uploads/2022/08/20210519114528933.png "亚马逊AWS CDN—CloudFront使用体验插图21")](https://www.cdnb.net/bbs/wp-content/uploads/2022/08/20210519114528933.png)

查看ip地址，确实不再是从源服务器请求资源

[![](https://www.cdnb.net/bbs/wp-content/uploads/2022/08/20210519114748666.png "亚马逊AWS CDN—CloudFront使用体验插图23")](https://www.cdnb.net/bbs/wp-content/uploads/2022/08/20210519114748666.png)

## 总结

对于网站，音视频，下载类应用，CDN可以很好的提速，但是如果是提供api接口，逻辑算法类应用，CDN并不适用。

原文链接：https://blog.csdn.net/qq\_42760638/article/details/116919205?ops\_request\_misc=%257B%2522request%255Fid%2522%253A%2522165918321816782388050817%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fblog.%2522%257D&request\_id=165918321816782388050817&biz\_id=0&utm\_medium=distribute.pc\_search\_result.none-task-blog-2~blog~first\_rank\_ecpm\_v1~times\_rank-12-116919205-null-null.nonecase&utm\_term=[cdn](https://www.cdnb.net/bbs/tag/15 "cdn")

原创文章，作者：优速盾-小U，如若转载，请注明出处：https://www.cdnb.net/bbs/archives/168

[Nginx反向代理谷歌](https://www.cdnb.net/bbs/archives/5560 "Nginx反向代理谷歌")

上一篇 2022年8月3日 12:48

[网站如何防御 CC 和 DDOS 攻击](https://www.cdnb.net/bbs/archives/6616 "网站如何防御 CC 和 DDOS 攻击")

下一篇 2022年8月3日 13:49