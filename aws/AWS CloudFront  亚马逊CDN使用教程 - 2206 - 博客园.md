**步骤：**

1\. 打开CloudFront服务之后，首先需要选择分发类型。分发类型分为一般的静态文件分发和流媒体分发，即Web和RTMP，我的站点即选择Web类型。

![](https://img2018.cnblogs.com/blog/1171977/201811/1171977-20181105165537584-1549565443.png)

2\. CDN回源设定。

a. 首先要填写源站地址Origin Domain Name，源站可以是一个AWS S3，也可是是一个普通web站点。对于web站点，你要填写的的便是该web站点的域名，不支持直接填写IP；需要注意的是，该域名不能与站点提供给用户访问的域名一致，而是一个单独的回源域名。例如对于我的站点，其域名为www.dancen.com，我新注册了一个域名wwwcdn.dancen.com用于CDN回源。为什么回源域名不能与站点域名一致呢，很好理解，以我的站点为例，当用户访问站点www.dancen.com时，通过DNS系统查询得到站点的CNAME记录，CDN再通过对CNAME的解析到达边缘节点；当需要回源时，CDN又会访问回源域名，如果回源域名也是www.dancen.com，那就形成了一个循环，www.dancen.com -> CDN -> www.dancen.com，而且这是个死循环  
\---------------------  
**b.** 其次，需要选择回源协议，对安全要求不高的话，选择http only即可。

![](https://img2018.cnblogs.com/blog/1171977/201811/1171977-20181105165815096-1772429845.png)

3\. 缓存设定。

**a.** 需要指定用户访问的协议，http和https、重定向http到https、仅https，根据需要选择。

**b.** http方法，默认选择GET，HEAD即可。

**c.** 需要注意的是对象缓存Object Caching的设定，该设定用来指定 CloudFront 为 Web 分配缓存对象的时长。

Use Origin Cache Headers选项表示以源站http header中`Cache-Control max-age` 或 `Cache-Control s-maxage` 指令或者 `Expires` 标头字段的设定为准；

Customize选项中，最短、最长和默认 TTL 值来控制 CloudFront 在缓存中保留对象的时长 (以秒为单位)，超过该时长后才会将另一个请求转发到源。标头值还确定浏览器在缓存中保留对象的时长，超过该时长后才会将另一个请求转发到 CloudFront。

**d.** Query String Forwarding and Caching设定。CDN中缓存的对象是以查询的URL来区分的，该项的默认选择为NONE，表示忽略URL 中？之后的参数，有效提高文件缓存命中率，提升分发效率，这种情况下，

http://www.dancen.com/t.png?v1

http://www.dancen.com/t.png?v2

会被认为是同样的请求，当t.png已经被CDN缓存，并且没有过期时，即使服务器上的t.png被修改了，用户也获取不到新版的t.png。

虽然AWS官方推荐使用不同的文件名来对文件进行版本控制，但为了管理方便，我打算通过参数来对缓存进行版本控制，因此该项我设置为保留所有参数，即Forward all, cache based on all，这种情况下，

http://www.dancen.com/t.png?v1

http://www.dancen.com/t.png?v2

是两个不同的请求，当我修改了服务器上的t.png时，客户端只需要修改URL的参数，就能够获取到最新的t.png。

![](https://img2018.cnblogs.com/blog/1171977/201811/1171977-20181105170150163-2109355321.png)

4\. 分发设定。

a. 价格类别，价格类别由分发区域决定，最高档即全球分发，最低档即只在AWS的大本营北美进行分发。

b. 源站域名Alternate Domain Names，即我的站点提供给用户访问的域名，www.dancen.com，而该域名的CNAME，便要设定为最后CloudFront为我们生成的CDN域名：xxxxxxx.cloudfront.net。

c. SSL证书设置。

默认的ColudFront证书Default CloudFront Certificat。如果你让用户通过最后CloudFront为我们生成的CDN域名：xxxxxxx.cloudfront.net直接访问你的站点，那选择该选项。

自定义SSL证书。如果你让用户通过你自己的域名，例如www.dancen.com访问你的站点，那你就得为CloudFront提供该站点的证书，该证书可以通过ACM（AWS [Certificate Manager](https://console.aws.amazon.com/acm/home?region=us-east-1)）导入，具体流程参照本文最厚的的附文部分。

d. 创建分发，点击Create Distribution按钮即可。

![](https://img2018.cnblogs.com/blog/1171977/201811/1171977-20181105170404932-1600847221.png)

5 获取CloudFront为我们生成的CDN域名。

创建分发后，待Status变为Deployed，说明CloudFront配置生效，这个过程可能耗费20分钟左右。

CloudFront会生成一个域名Domain Name：xxxxxxx.cloudfront.net，我们需要在域名提供商的DNS系统把源站域名，如

www.dancen.com的CNAME记录设定为该域名。在CMD窗口执行nslookup命令检查你的域名，如www.dancen.com，能否解析出CDN提

供的IP，确认CNAME设定是否生效，生效后CDN加速便算是开启成功，使用你的域名，如www.dancen.com测试通过即可。

![](https://img2018.cnblogs.com/blog/1171977/201811/1171977-20181105170646514-1560923007.png)

![](https://img2018.cnblogs.com/blog/1171977/201811/1171977-20181105170700466-722188258.png)

附：

通过AWS的ACM（AWS [Certificate Manager](https://console.aws.amazon.com/acm/home?region=us-east-1)）服务导入站点SSL证书。ACM服务可以直接请求证书，也可以导入证书，这里假设你的源站已经使用https，有了SSL证书，因此只讲述导入证书的步骤。

1\. 打开ACM服务，点击导入证书。

![](https://img2018.cnblogs.com/blog/1171977/201811/1171977-20181105171058266-258507383.png)

2\. 填写证书内容。

到你的站点服务器，查看证书内容，并填入ACM。例如fullchain.pem文件即为证书，打开之，第一个begin-end即为证书正文，第二个begin-end即为证书链；privkey.pem文件即为私匙。填写完毕后，点击审核并导入即可

![](https://img2018.cnblogs.com/blog/1171977/201811/1171977-20181105171426201-2057504310.png)

![](https://img2018.cnblogs.com/blog/1171977/201811/1171977-20181105171439679-1346631680.png)

![](https://img2018.cnblogs.com/blog/1171977/201811/1171977-20181105171454430-213537191.png)