CloudFlare是全球知名的CDN服务提供商，其与诸多云服务商构建的带宽联盟为末端用户提供了极大的流量优惠。本文将结合博主在Backblaze（B2）和阿里云OSS使用上的实践探索，简单展示在搭配使用CloudFlare CDN和保护源对象存储的一些技巧，希望大家能够喜欢~

![](https://cdn.luotianyi.vc/wp-content/uploads/2022-02-02_10-58-56.jpg)

顺便值此新春佳节之际，感谢诸位一直以来对博主的支持，简单地新的一年祝大家阖家欢乐、学习进步、事业有成哦啊♪(´▽｀)~

___

## 一、概述

### 带宽联盟

带宽联盟是由CloudFlare主导组建的旨在节省云计算服务商间数据流量开销的组织，换句话说就是节省IDC间的流量传输，减少各IDC通过基础ISP向互联网传输资源支付的费用。在这个联盟中，受益最大的就是CF和末端用户，CF借此巩固自身的互联网带宽权益地位，更有底气向ISP压低带宽价格，用户则获得了来自服务商的流量费用豁免。目前根据实际测试，带宽联盟的流量节省方式主要是CF与服务商构建对等互联或选择低价的ISP/IXP完成互联。

![](https://cdn.luotianyi.vc/wp-content/uploads/2022-02-02_09-31-09.png)

虽然加入带宽联盟的服务商有二十余家，不过是否向末端用户提供优惠仍取决于各服务商自身的政策。值得一提的是，像AWS和Google这样在互联权益地位很高的服务商，对带宽联盟基本是持拒绝或仅提供基础支持的态度，这是因为利益的冲突驱使他们不愿向CF转让自己的权益优势，从IDC的角度来看这是无可厚非的。

> #官方简介  
> [https://www.cloudflare.com/zh-cn/bandwidth-alliance](https://www.cloudflare.com/zh-cn/bandwidth-alliance)

### 支持的服务商

正如上文所言，并不是所有参与带宽联盟的服务商都向用户提供这样的优惠，以下推荐是博主和朋友亲自测试确认的结果，欢迎大家进行补充。

**（1）阿里云**：阿里云OSS已确认除迪拜和中国大陆的地域外流量传出至CF边缘节点不计费。计费项目包括存储费用和请求费用，国内版产品定价（[点击前往](https://www.aliyun.com/price/product#/oss/detail/ossbag)），国际版产品定价（[点击前往](https://www.alibabacloud.com/zh/product/oss/pricing)）。注意，仅有国际版客户拥有每月1亿次的免费读请求额度。其优点是功能完善，地域众多且网络对大陆友好，有各类资源包供选购；缺点是费用偏高，计费方式复杂。

> #官方说明  
> [https://www.aliyun.com/product/news/detail?id=17749](https://www.aliyun.com/product/news/detail?id=17749)

**（2）Backblaze（B2）**：Cloud Storage已确认流量传出至CF边缘节点不计费。计费项目包括存储费用（0.005/G·月）及列目录、读取的B/C类请求费用（0.004/万次），产品定价（[点击前往](https://www.backblaze.com/b2/cloud-storage-pricing.html)）。其优点是价格非常友好且计费灵活；缺点是地域不可选、功能很少、可用区网络质量较差。

> #官方说明  
> [https://www.backblaze.com/b2/solutions/content-delivery.html](https://www.backblaze.com/b2/solutions/content-delivery.html)

**（3）DigitalOcean**：Spaces Object Storage已确认流量传出至CF边缘节点不计费。计费项目只有存储费用，不计请求费用，但需要选择5USD/月的基础套餐，产品定价（[点击前往](https://www.digitalocean.com/products/spaces)）。博主并没有亲自使用这个服务因此不做过多评价，显而易见的优点是计费非常简单；缺点是不够灵活，仅有一种包计费模式。

> #官方说明  
> [https://www.digitalocean.com/community/questions/bandwidth-alliance-status](https://www.digitalocean.com/community/questions/bandwidth-alliance-status)

___

## 二、连接CloudFlare

连接到CloudFlare需要为存储桶绑定域名，然后在CloudFlare设置CNAME到桶域名。像阿里云OSS、腾讯云COS等经过自行开发的是可以直接通过相应的域名绑定页面进行绑定，其余的AWS S3、Scaleway之类的是通过设置bucket名称为域名进行绑定，只有B2选择的方式非常特别。

![](https://cdn.luotianyi.vc/wp-content/uploads/2022-02-03_01-19-03.png)

另外建议为自定义的域名在CloudFlare添加一条页面规则，将SSL设置为灵活（即HTTP回源），因为多数对象存储不支持自定义域名SSL访问，选用HTTPS回源的话是不必要的（但是B2是不允许HTTP访问的就必须要设置为HTTPS回源）。其余的页面规则可以按照自己按照其需求自行添加，在此不做赘述。

![](https://cdn.luotianyi.vc/wp-content/uploads/2022-02-03_01-29-22.jpg)

### 阿里云OSS等

阿里云绑定自定义域名很简单，在【传输管理】-【域名管理】中，点击绑定域名按流程即可完成绑定。需要在CloudFlare指向的桶域名，也可以在概览中的外网访问Bucket域名找到。

![](https://cdn.luotianyi.vc/wp-content/uploads/2022-02-03_01-22-07.jpg)

### Backblaze

B2选择的绑定方式就非常特别了，在bucket文件管理页面上传一个文件，点击右侧信息按钮可以在其中获得`Friendly URL`，将你的域名CNAME指向链接中域名比如`f004.backblazeb2.com`后，就可以将链接中域名替换为自己的域名实现访问。B2的服务器是向任意域名接入，bucket间域名公用仅以目录区分，这样存在的隐患十分明显，文件链接完全公开而且自定义域名可被其他用户使用。

![](https://cdn.luotianyi.vc/wp-content/uploads/2022-02-03_01-53-16.jpg)

CloudFlare近期的【转换规则】可以完美解决这个问题，在【创建转换规则】下选择【重写URL】，传入匹配建议如图选择相应主机名，随后在路径下找到重写到，选择Dynamic动态模式，按如图修改`/file/lms-example`为即需要隐藏的目录填入框内即可。这样访问链接中bucket路径便被隐藏了，既美观也减少了被滥用的可能。

<table><tbody><tr><td data-settings="show"></td><td><div><p><span>#隐藏目录中/file/lms-example部分</span></p><p><span>concat</span><span>(</span><span>"/file/lms-example"</span><span>,</span><span> </span><span>http</span><span>.</span><span>request</span><span>.</span><span>uri</span><span>.</span><span>path</span><span>)</span></p></div></td></tr></tbody></table>

同样的，利用转换规则隐藏部分目录也可以用于其他的对象存储服务商，因为绝大多数的对象存储公共访问链接是有迹可循的，被有心之人找出并利用可能会产生巨额的流量费用。我们可以在bucket中新建一个用于存放对外访问的文件夹，然后与B2的操作一样将该目录通过动态重写隐藏掉，这些可以自行发挥。

___

## 三、隐藏Bucket特征

### 设置错误页

S3协议的bucket在公开读的权限下会默认展示桶内文件列表和路径，特征明显而且并不友好。绝大多数的对象存储都支持设置静态网站`index.html`和错误页`404.html`的设置，比如阿里云OSS在【基础设置】-【静态页面】下。这里随手找了两个单页供选择和参考（[点击前往](https://static.lty.fun/?%E9%9D%99%E6%80%81%E8%B5%84%E6%BA%90/%E5%8D%95%E9%A1%B5/%E9%94%99%E8%AF%AF%E9%A1%B5)）。

![](https://cdn.luotianyi.vc/wp-content/uploads/2022-02-03_02-21-01.jpg)

显然，像B2这种简单粗暴的是又一次没有这样的功能，可以如图使用URL重写规则将主页静态定向至index.html。404页面的功能没有一个比较好的办法，不过B2的友好访问链接不会展示目录列表和bucket信息，影响并不大。

![](https://cdn.luotianyi.vc/wp-content/uploads/2022-02-03_08-11-44.jpg)

### 隐藏bucket标头

在对象存储的标头中会包含有一些bucket信息，可以通过控制台的Network选项卡暴露出来，也是对对象存储特征的一个暴露点。

![](https://cdn.luotianyi.vc/wp-content/uploads/2022-02-03_10-13-24.jpg)

通过CloudFlare转换规则中的【修改响应头】，能够简单地实现响应头的去除，同时还可以加入跨域请求头等需要的标头。

![](https://cdn.luotianyi.vc/wp-content/uploads/2022-02-03_10-08-11.jpg)

目前B2和OSS可以隐藏的标头大致如下，其他对象存储请根据实际情况去查看和配置，在此就不一一列出了。

<table><tbody><tr><td><strong>阿里云OSS</strong></td><td><strong>Backblaze B2</strong></td></tr><tr><td>x-oss-hash-crc64ecma</td><td>x-bz-content-sha1</td></tr><tr><td>x-oss-object-type</td><td>x-bz-file-id</td></tr><tr><td>x-oss-request-id</td><td>x-bz-file-name</td></tr><tr><td>x-oss-server-time</td><td>x-bz-info-src_last_modified_millis</td></tr><tr><td>x-oss-storage-class</td><td>x-bz-upload-timestamp</td></tr></tbody></table>

### 禁止访问文件

同样的，通过页面规则中的URL重写功能，我们也能够实现诸如对特定目录、特定后缀（如图`.php`）的禁止访问，重写至错误页面即可。不过需要注意的是，转换规则是下方规则优先级更高，而页面规则是上方规则优先级更高。

![](https://cdn.luotianyi.vc/wp-content/uploads/2022-02-03_10-11-43.jpg)

___

## 四、CloudFlare安全规则

### 页面规则

CloudFlare页面规则前文有提到，免费版提供3条规则，在这里也可以设置一些边缘缓存有效期、缓存内容等等细致的配置。博主来自对象存储的静态资源一般进行如下的配置，如果有特殊的需要可以自行结合实际去探索。

![](https://cdn.luotianyi.vc/wp-content/uploads/2022-02-03_08-23-16.jpg)

### 防火墙规则

CloudFlare防火墙有5条免费的防火墙规则，入口在【防火墙】-【防火墙规则】，能够实现很细致的安全规则，比如图中配置即为开启验证码。其他的项目在这里暂时不做过多的讲解，等后续有时间，我们再展开详细聊一聊CloudFlare防火墙的规则应用。

![](https://cdn.luotianyi.vc/wp-content/uploads/2022-02-03_08-20-43.jpg)

___

## 五、结语

此外还有一些对象存储比如Scaleway提供每个月免费75G的空间和流量且不计请求，腾讯云COS的带宽联盟优惠政策也在推进，选择不止局限于博主提到的几家，能够以最低的成本满足自己需求的才是最佳的选择。

关于对象存储搭配CloudFlare使用的探索到这里就结束了，回想起来有一些的尝试并没有什么具体意义，也没有起到很大的作用。这篇博客就在此简单做一个记录吧，希望其中的一些技巧能够对诸位的使用有一些启发。其他的使用技巧，也欢迎与博主进一步交流~

___

\*原创文章，转载请注明出处