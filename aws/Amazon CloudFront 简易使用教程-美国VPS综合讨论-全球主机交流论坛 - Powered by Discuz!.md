AWS 最近实施了大免费政策，CloudFront 免费层级也从前 12 个月每月 50GB 流出 + 2,000,000 次 HTTP/HTTPS 请求改为永久 1TB 流出 + 10,000,000 次HTTP/HTTPS 请求。诚然 CloudFront 在大陆访问整体效果不佳，但是架不住其免费，用的人多起来总还是有需求的。本文旨在列举使用 CloudFront 分发网站内容时需要注意的地方。

0\. 使用 CloudFront 需要发工单验证

> Your account must be verified before you can add new CloudFront resources. To verify your account, please contact AWS Support (https://console.aws.amazon.com/support/home#/) and include this error message.

原因：被风控了，实体卡建议立即工单对线，虚拟卡速速爬，等待风控结束再次尝试。

1\. 先行导入 SSL 证书  
我觉得不会有人不用 HTTPS，而使用 HTTPS 需要在创建时选择证书，因此创建 CloudFront 分配前建议先到 https://console.aws.amazon.com/acm/home?region=us-east-1#/certificates/list (AWS Certificate Manager) 导入证书。注意，证书必须导入在 region=us-east-1 （弗吉尼亚北部）。导入过程我就不细说了。当然，不想导入证书的可以直接向 AWS 请求一张证书，价格为免费，验证级别为 DV，有效期为 396 天，支持多域名和通配符，只能在 AWS 上的部分服务使用（CloudFront 可用，可用服务列表见 https://docs.aws.amazon.com/acm/latest/userguide/acm-services.html）  
![](https://p.itxe.net/images/2021/12/20/image.png)

2\. 源域  
CloudFront 回源只能填写域名而不能填写 IP。域名填写一个 A 记录指向源站 IP 的域名即可。注意不可使用要套 CDN 的域名（比如我想给 blog.iks.moe 套 CloudFront，源站 IP 是 1.14.51.4，此时“源域”不能填写 blog.iks.moe，要填其他指向源站 IP 1.14.51.4 的域名，比如新建一个 qcloud-shimokitazawa.iks.moe 解析到 1.14.51.4  
![](https://p.itxe.net/images/2021/12/20/imaged1f51be93fab90ea.png)

协议建议选择“匹配查看器”，“最低源 SSL 协议”建议选 TLSv1.1 或以上  
![](https://p.itxe.net/images/2021/12/20/image96196e68aaaaee09.png)

下面的源路径，除非你知道那是什么，否则不要填；源名称可以自定义，用于标识和记录（相当于 tag）

3\. 查看器  
不多说。看图  
![](https://p.itxe.net/images/2021/12/20/image1834ff853f93fb44.png)

4\. 缓存键和源请求  
不多说。看图  
![](https://p.itxe.net/images/2021/12/20/image0d5ba68f57f3649d.png)

5\. 域名和边缘证书  
![](https://p.itxe.net/images/2021/12/20/image90224ad89f24664b.png)  
关于 TLS 版本与合规问题，请参考 https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/secure-connections-supported-viewer-protocols-ciphers.html

6\. 根据路径配置缓存与回源行为  
分配创建完成以后，可以进入分配配置页面为其他路径的请求设置个别的缓存控制与回源行为控制。可配置项见 Q4  
![](https://p.itxe.net/images/2021/12/20/imagec12f4051493a7caa.png)

7\. 杂项  
a. 源站 SSL 证书问题  
源站证书必须在有效期内；源站证书必须为受信任证书，自签的或者 Cloudflare 给的什么十年十五年的都不行，否则 502  
b. WebSocket 问题  
ws 只要按上述说明配置即可正常使用。能在 Cloudflare 上用就能在 CloudFront 上用。