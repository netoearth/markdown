2021-12-20 分类：[网站教程](https://www.banzhuti.com/tutorial/web-tutorial) 阅读(1.02K) 评论(0)

[![亚马逊免费CDN Amazon CloudFront使用图文教程插图](data:image/svg+xml;base64,PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHdpZHRoPSI3MDAiIGhlaWdodD0iMjAzIiB2aWV3Qm94PSIwIDAgNzAwIDIwMyI+PHJlY3Qgd2lkdGg9IjEwMCUiIGhlaWdodD0iMTAwJSIgZmlsbD0iI2NmZDRkYiIvPjwvc3ZnPg== "亚马逊免费CDN Amazon CloudFront使用图文教程插图")](https://cdn.banzhuti.com/2021/12/20211220135950671.jpg)

[亚马逊](https://www.banzhuti.com/tag/613 "【查看含有[亚马逊]标签的文章】")的[Amazon CloudFront](https://www.banzhuti.com/tag/313 "【查看含有[Amazon CloudFront]标签的文章】") 是一种内容分发网络 ([CDN](https://www.banzhuti.com/tag/309 "【查看含有[CDN]标签的文章】")) 服务，旨在获得优异性能、安全性和开发人员便利性。目前除了谷歌、甲骨文等，[亚马逊](https://www.banzhuti.com/tag/613 "【查看含有[亚马逊]标签的文章】")的[免费CDN](https://www.banzhuti.com/tag/117 "【查看含有[免费CDN]标签的文章】")也是很有名的。如何使用呢，这里搬主题就分享一下[亚马逊](https://www.banzhuti.com/tag/613 "【查看含有[亚马逊]标签的文章】")[免费CDN](https://www.banzhuti.com/tag/117 "【查看含有[免费CDN]标签的文章】") [Amazon CloudFront](https://www.banzhuti.com/tag/313 "【查看含有[Amazon CloudFront]标签的文章】")使用图文教程。

AWS最近实施了大免费政策，CloudFront免费层级也从前 12 个月每月 50GB 流出 + 2,000,000 次 HTTP/HTTPS 请求改为永久 1TB 流出 + 10,000,000 次HTTP/HTTPS 请求。诚然 CloudFront 在大陆访问整体效果不佳，但是架不住其免费，用的人多起来总还是有需求的。本文旨在列举使用 CloudFront 分发网站内容时需要注意的地方。

## 使用 CloudFront 需要发工单验证

> Your account must be verified before you can add new CloudFront resources. To verify your account, please contact AWS Support (https://console.aws.amazon.com/support/home#/) and include this error message.

原因：被风控了，实体卡建议立即工单对线，虚拟卡速速爬，等待风控结束再次尝试。

## 先行导入 SSL 证书

我觉得不会有人不用 HTTPS，而使用 HTTPS 需要在创建时选择证书，因此创建 CloudFront 分配前建议先到 https://console.aws.amazon.com/acm/home?region=us-east-1#/certificates/list (AWS Certificate Manager) 导入证书。

**注意，证书必须导入在 region=us-east-1 （弗吉尼亚北部）。**

导入过程我就不细说了。当然，不想导入证书的可以直接向 AWS 请求一张证书，价格为免费，验证级别为 DV，有效期为 396 天，支持多域名和通配符，只能在 AWS 上的部分服务使用（CloudFront 可用，可用服务列表见 https://docs.aws.amazon.com/acm/latest/userguide/acm-services.html）

[![亚马逊免费CDN Amazon CloudFront使用图文教程插图1](data:image/svg+xml;base64,PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHdpZHRoPSIxMDEzIiBoZWlnaHQ9IjI4OCIgdmlld0JveD0iMCAwIDEwMTMgMjg4Ij48cmVjdCB3aWR0aD0iMTAwJSIgaGVpZ2h0PSIxMDAlIiBmaWxsPSIjY2ZkNGRiIi8+PC9zdmc+ "亚马逊免费CDN Amazon CloudFront使用图文教程插图1")](https://cdn.banzhuti.com/2021/12/20211220135955665.png)

## 回源域名设置

CloudFront 回源只能填写域名而不能填写 IP。域名填写一个 A 记录指向源站 IP 的域名即可。注意不可使用要套 [CDN](https://www.banzhuti.com/tag/309 "【查看含有[CDN]标签的文章】") 的域名（比如我想给www.banzhuti.com套 CloudFront，源站 IP 是 1.14.51.4，此时“源域”不能填写 www.banzhuti.com，要填其他指向源站 IP 1.14.51.4 的域名，比如新建一个www2.banzhuti.com解析到 1.14.51.4

[![亚马逊免费CDN Amazon CloudFront使用图文教程插图2](data:image/svg+xml;base64,PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHdpZHRoPSI2MjciIGhlaWdodD0iNTgzIiB2aWV3Qm94PSIwIDAgNjI3IDU4MyI+PHJlY3Qgd2lkdGg9IjEwMCUiIGhlaWdodD0iMTAwJSIgZmlsbD0iI2NmZDRkYiIvPjwvc3ZnPg== "亚马逊免费CDN Amazon CloudFront使用图文教程插图2")](https://cdn.banzhuti.com/2021/12/20211220140000346.png)

协议建议选择“匹配查看器”，“最低源 SSL 协议”建议选 TLSv1.1 或以上

[![亚马逊免费CDN Amazon CloudFront使用图文教程插图3](data:image/svg+xml;base64,PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHdpZHRoPSI2OTUiIGhlaWdodD0iNzA3IiB2aWV3Qm94PSIwIDAgNjk1IDcwNyI+PHJlY3Qgd2lkdGg9IjEwMCUiIGhlaWdodD0iMTAwJSIgZmlsbD0iI2NmZDRkYiIvPjwvc3ZnPg== "亚马逊免费CDN Amazon CloudFront使用图文教程插图3")](https://cdn.banzhuti.com/2021/12/20211220140004190.png)

下面的源路径，除非你知道那是什么，否则不要填；源名称可以自定义，用于标识和记录（相当于 tag）

## 查看器设置

不多说。看图

[![亚马逊免费CDN Amazon CloudFront使用图文教程插图4](data:image/svg+xml;base64,PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHdpZHRoPSI2MjQiIGhlaWdodD0iMzc0IiB2aWV3Qm94PSIwIDAgNjI0IDM3NCI+PHJlY3Qgd2lkdGg9IjEwMCUiIGhlaWdodD0iMTAwJSIgZmlsbD0iI2NmZDRkYiIvPjwvc3ZnPg== "亚马逊免费CDN Amazon CloudFront使用图文教程插图4")](https://cdn.banzhuti.com/2021/12/20211220140009657.png)

## 缓存键和源请求设置

不多说。看图

[![亚马逊免费CDN Amazon CloudFront使用图文教程插图5](data:image/svg+xml;base64,PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHdpZHRoPSI3MzAiIGhlaWdodD0iNTQ4IiB2aWV3Qm94PSIwIDAgNzMwIDU0OCI+PHJlY3Qgd2lkdGg9IjEwMCUiIGhlaWdodD0iMTAwJSIgZmlsbD0iI2NmZDRkYiIvPjwvc3ZnPg== "亚马逊免费CDN Amazon CloudFront使用图文教程插图5")](https://cdn.banzhuti.com/2021/12/20211220140013944.png)

## 域名和边缘证书设置

[![亚马逊免费CDN Amazon CloudFront使用图文教程插图6](https://cdn.banzhuti.com/2021/12/20211220140018710.png "亚马逊免费CDN Amazon CloudFront使用图文教程插图6")](https://cdn.banzhuti.com/2021/12/20211220140018710.png)

关于 TLS 版本与合规问题，请参考 https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/secure-connections-supported-viewer-protocols-ciphers.html

## 根据路径配置缓存与回源行为

分配创建完成以后，可以进入分配配置页面为其他路径的请求设置个别的缓存控制与回源行为控制。可配置项见 Q4

[![亚马逊免费CDN Amazon CloudFront使用图文教程插图7](https://cdn.banzhuti.com/2021/12/20211220140024737.png "亚马逊免费CDN Amazon CloudFront使用图文教程插图7")](https://cdn.banzhuti.com/2021/12/20211220140024737.png)

## 杂项设置

a. 源站 SSL 证书问题  
源站证书必须在有效期内；源站证书必须为受信任证书，自签的或者 Cloudflare 给的什么十年十五年的都不行，否则 502  
b. WebSocket 问题  
ws 只要按上述说明配置即可正常使用。能在 Cloudflare 上用就能在 CloudFront 上用。

除特别注明外，本站所有文章均基于CC-BY-NC-SA 4.0原创，转载请注明出处。  
文章名称：《亚马逊免费CDN Amazon CloudFront使用图文教程》  
文章链接：[https://www.banzhuti.com/free-cdn-amazon-cloudfront-setting.html](https://www.banzhuti.com/free-cdn-amazon-cloudfront-setting.html)

版权免责声明

① 本站提供的资源(插件或主题)均为网上搜集，如有涉及或侵害到您的版权请立即通知我们。  
② 本站所有下载文件，仅用作学习研究使用，请下载后24小时内删除，支持正版，勿用作商业用途。  
③ 因代码可变性，不保证兼容所有浏览器、不保证兼容所有版本的WP、不保证兼容您安装的其他插件。  
④ 本站保证所提供资源(插件或主题)的完整性，但不含授权许可、帮助文档、XML文件、PSD、后续升级等。  
⑤ 由本站提供的资源对您的网站或计算机造成严重后果的本站概不负责。  
⑥ 使用该资源(插件或主题)需要用户有一定代码基础知识！另本站提供**[汉化使用安装教程](https://www.banzhuti.com/wordpress-en-to-cn.html)**，仅供参考。  
⑦ 有时可能会遇到部分字段无法汉化，同时请保留作者汉化宣传信息，谢谢！  
⑧ 本站资源售价只是赞助和汉化辛苦费，收取费用仅维持本站的日常运营所需。  
⑨ 如果喜欢本站资源，欢迎捐助本站开通会员享受优惠折扣，谢谢支持！  
⑩ 如果网盘地址失效，请在相应资源页面下留言，我们会尽快修复下载地址。