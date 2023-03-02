Cloudfront是亚马逊AWS(Amazon Web Services)的CDN服务，使用这个cloudfront之前我假设你已经使用过cloudflare，我这里有一篇关于使用cloudflare的文章：[使用CloudFlare免费cdn隐藏服务器ip](https://www.xiebruce.top/997.html)。

写这篇文章是因为我发现目前网上所有关于cloudfront的文章都写的不清不楚，它的使用跟cloudflare区别很大，我也是踩了好多坑才搞好，希望给遇到问题的朋友一点帮助！

注册登录aws后，点击[这里](https://us-east-1.console.aws.amazon.com/cloudfront/v3/home#/welcome)，我们可以看到免费套餐，个人使用的话，只要不是下载和老看4k视频，这个额度是足够的

## 原理解析

### Cloudflare的模式

你需要在你购买域名的网站(比如可能是阿里云，可能是namesilo，可能是freenom等等)那边把你的域名服务器(英文NameServer，一般简称NS)修改为cloudflare的NS服务器，也就是说，需要让cloudflare全面接管你的域名，接管后，你想要添加DNS解析(添加A记录)，就需要在cloudflare中添加，而不是在你购买域名的服务商那边添加，添加解析时，把小云朵的开关打开，就会走cdn，不打开就不会走cdn。

### Cloudfront模式

aws的cloudfront与cloudflare工作模式不同，你无需(也无法)把你的域名交给cloudfront接管，要使用cloudfront做cdn，你需要准备两个域名：

-   1、你要套cdn的域名(A域名)：假设为“aa.test.com”；
-   2、源域名(B域名)：假设为“bb.test.com”。cdn是用于加速用的，你需要告诉cdn你要加速的内容(图片、视频、网站)存放在哪台服务器上，但不能填ip，只能填域名，所以你需要解析一个域名到你要加速的服务器上，这个域名就叫“源域名”。

**Cloudfront cdn加速原理**：就是把B域名(bb.test.com)解析到你的服务器上，然后在cloudfront的“选择源域”中填写B域名，这样cloudfront就知道你要加速的服务器是哪个服务器，最后它会自动生成一个“xxx.cloudfront.net”域名作为加速域名，然后你需要把你的网站域名(即你要套cdn的域名)“aa.test.com”cname到“xxx.cloudfront.net”，最终效果就是你访问“aa.test.com”其实是在访问“xxx.cloudfront.net”(当然这是cname，不是301/302跳转)，而“xxx.cloudfront.net”是cloudfront的域名，它会对应前面B域名所对应的那个网站。

由于cloudfront并不是“接管域名”的方式，所以它只能代理80(http)和443端口(https,包括grpc也能代理，因为grpc基于http2)，不像Cloudflare，除了80和443，它还能代理很多端口。

Cloudflare代理的端口请看[这里](https://developers.cloudflare.com/fundamentals/get-started/reference/network-ports/)，下面我也摘抄下来：

> http端口有：80, 8080, 8880, 2052, 2082,2086, 2095  
> https端口有：443, 2053, 2083, 2087, 2096, 8443  
> Cloudflare支持但不做缓存的端口：2052, 2053, 2082, 2083, 2086, 2087, 2095, 2096, 8880, 8443。

### 举例说明

假设你现在有如下域名和服务器

-   1、假设你想把A域名(aa.test.com)套到cloudflare cdn上，用cdn来加速这个域名；
-   2、那么你需要先把B域名(bb.test.com)解析到你的服务器(11.22.33.44)；
-   3、然后把B域名(bb.test.com)填到cloudfront的“选择源域”里，把A域名(aa.test.com)填到“备用域名(CNAME)”里(这一栏与前一栏不是挨着，需要往下滚动很多才能看到)；
-   4、然后填好其它选项后，最终创建好之后，它会生成一个“xxx.cloudfront.net”格式的域名，这个域名就是cloudfront给你分配的加速域名，比如你之前访问一张图片是这样访问的：aa.test.com/imgs/1.jpg，而用了cdn之后就是：xxx.cloudfront.net/imgs/1.jpg；
-   5、按第4步的说法，虽然cdn可以加速，可是域名就变了，这是不可接受的，这就要用到前面第3步填的“备用域名”了，其实我觉得把它叫“自定义域名”更好一点，你需要添加一条DNS解析(CNAME记录)，把你在“备用域名”那一栏填的那个域名(aa.test.com)添加一个cname，对应到cloudfront给你分配的加速域名“xxx.cloudfront.net”，最终的请求的流向为

### cft真的免费吗？

这是我问客服的

> In this case the services “All data transfer out to Origin” this prices are the charges you incur when data is transferred to your origin using the Put, Post, Patch, Options and Delete HTTP(s) requests. This process is different to the data transfer in free tier, that’s why that services charges on demand.

这是我用aws一个月后的费用(我用的流量少，所以收费少)  
![](https://img.xiebruce.top/2022/10/15/f5b8a19f52610bf5bac47ce899fd5cba.jpg)

这是我2022年9月份的费用(我用的流量少，而且有免费部分)  
![](https://img.xiebruce.top/2022/10/15/9c78dbc1d1e0cb570b3094fff8810f04.jpg)

总的来说，它是有些流量要算钱的，但是有免费的部分，具体以自己使用为准，如果用来经常下载或看电影的，建议谨慎使用，如果只是看看网页查查资料，那可以随便用，就算产生费用也不会有多少。

## 实际操作

### cft的原理

为了让大家对cft原理有一个认识，我把最终添加好cdn的域名截图出来了，如下图所示，我添加了三个域名走cft cdn，每个域名都有三个字段：域名、备用域名、源。  
![image.jpg](https://img.xiebruce.top/2022/10/21/9d413488242ab5d32ea1aa032988a3cb.jpg)

-   **域名**：cft给你分配的，由“14位小写字母或数字”开头+`.cloudfront.net`，比如`a1dpj8y8dvolnz.cloudfront.net`；
-   **备用域名**：你要套CDN的域名；
-   **源**：解析到你服务器ip的域名

你需要做的：

-   1、在你的域名服务商那边解析一个域名(A记录)到你的服务器ip，这个域名用作“源”(注意这个域名不能是你最终要套cdn的域名)；
-   2、当你在cft中添加“分配”后，它会给你生成一个域名(即上图“域名”那一列中的域名)，你需要在你的域名服务商那边添加一个cname，一个cname记录由一个“name”和一个“content”组成，那么“name”就填你真正要套cdn的域名(即上图中的“备用域名”)，而“content”就填上图中的“域名”(即cft给你分配的域名)；
-   3、为什么你“真正套cdn的域名”会被称为“备用域名”呢？因为第二步中的cname如果你不加，直接用cft给你分配的那个域名也是可以的，而添加了cname就能用你自己的域名了，由于原本就有可用的域名(cft分配的)，所以把你自己的要套cdn的那个域名叫“备用域名”；
-   4、申请证书的时候需要在域名服务商那边添加一个cname。

举例：假设你有一个域名叫：test.com，你想解析一个aa.test.com，让它走cft的cdn（即你想把aa.test.com这域名套到cft上），那么你需要按顺序做以下几步：

-   1、在你的域名服务商那边解析一个bb.test.com到你的服务器ip(就是添加一个A记录把bb.test.com解析到你的服务器ip)；
-   2、在aws的cft中[创建一个分配](https://us-east-1.console.aws.amazon.com/cloudfront/v3/home?region=us-west-2#/distributions/create)，创建分配会让你填很多数据和选择很多选项，你要把这个bb.test.com填到“源域”一栏中；
-   3、在“创建分配”页面中找到“备用域名(CNAME) – 可选”，添加一个项目，把你真正要套cdn的域名(即aa.test.com)填到这里；
-   4、你需要在“创建分配”页面点击[请求证书](https://us-east-1.console.aws.amazon.com/acm/home?region=us-east-1#/certificates/request)，然后点击下一步，并填写“完全限定域名”，这个域名就填`*.test.com`即可(除非你的域名是freenom的免费域名，如果是，则需要填完整域名，不能填通配符域名)；
-   5、你需要在域名服务商那边添加一个cname，把你要套cdn的域名(即你填到“备用域名”中的那个域名，在这里是aa.test.com)指向cft给你分配的域名。

**再啰嗦一遍**：真正要套cdn的域名，并不是解析到某个ip的，而是要cname到另一个域名(cft给你分配的)。

了解了原理之后，以下为实际操作。

### Cloudfront网页端操作

从[这里](https://us-east-1.console.aws.amazon.com/cloudfront/v3/home#/welcome)点击”创建CloudFront分配”。

然后在下图的“选择源域”中填写一个“源域名”  
![image.jpg](https://img.xiebruce.top/2022/11/03/7d2c71c6003e19540772493fd143e206.jpg)  
– 1、注意这个域名不能是你要套cdn的域名，这个域名只是用来告诉cloudfront：“喂，你要加速的内容在abc.examle.com服务器中，记得给它加速哟”；  
– 2、它只能填域名，不能直接填ip，所以我们需要解析一个域名指向这个ip(你VPS的ip)，用域名的方式来告诉cloudfront，告诉它要给哪台服务器的内容加速，这个域名我们叫“源域名”，它指向将要被加速的“源服务器”(就是你的VPS)；  
– 3、匹配查看器，其实就是匹配浏览器，“查看器”其实是browser的翻译，本来应该翻译为浏览器的，这aws的翻译也是厉害，不知道为什么会翻译成查看器的。

接下往下，这些按默认就可以  
![](https://img.xiebruce.top/2022/08/28/6cc82a7735bd3a2d6609b0a06b7e7999.jpg)

缓存策略选择“CachingDisabled”(禁用缓存)，源请求策略选择“AllViewer”  
![](https://img.xiebruce.top/2022/08/28/68b166d4789bc355424a8d8a75b8a38b.jpg)

这里都不用填，默认就好  
![](https://img.xiebruce.top/2022/08/28/d84274c0a5951273ddf6a527d8557c21.jpg)

这里“价格与级别”，看你的服务器在哪儿，一般来说第一个和第三个都可以，仅北美和欧洲肯定不行，因为你在国内是亚洲，它不对亚洲加速的话，你要它来干嘛。这个价格可以不用太在意，因为你个人使用不超过免费用量是不会收费的。特别注意这里的“**备用域名(CNAME)**”一栏要填“**你真正要套cdn的域名**”(就是你的网站域名，就是你要加速的域名)  
![](https://img.xiebruce.top/2022/08/28/d9599d9e96a807827f8bbade75615452.jpg)

这里要选择证书，点击“请求证书”来申请一个证书，点击后它会在另一个标签中打开新页面，这个证书不需要我们源域名(bb.test.com)或网站域名(bb.test.com)的证书，就是说这个证书归这个证书，跟你自己域名的证书无关  
![](https://img.xiebruce.top/2022/08/28/3006625823780f44ae5eb9e84a5d8c75.jpg)

___

以下页面是在新标签中打开的，其中“完全限定域名”，其实就是填“你要获取证书的域名”，如果你要申请单域名证书，那就填你要套cdn的那个域名，比如“aa.test.com”，当然你也可以填写“\*.test.com”表示获取“test.com”的通配符证书，这样无论是“aa.test.com”还是“bb.test.com”都以用，我是强烈建议填通配符证书，因为如果你还要添加要套cdn的域名，是可以选择同一个证书的  
![](https://img.xiebruce.top/2022/08/28/24848674b2ab47736b2df8c9ac6586af.jpg)

选择dns验证方式  
![](https://img.xiebruce.top/2022/08/28/f4826b5c78ed3c1e01047ed159f6f038.jpg)

请求成功，点击“查看证书”  
![](https://img.xiebruce.top/2022/08/28/c4b7d5a1f92977d3c815882ecc9e43ba.jpg)

在你域名解析网站那边添加一个cname解析，cname解析需要一个“cname名称”和一个“cname值”，这两个数据在下图中可以看到(如果看不到你就刷新一下)  
![](https://img.xiebruce.top/2022/08/28/11e5c49b5b6ec28cf9aed90a4f3c0a03.jpg)

添加之后，稍微过一会儿你再刷新下图这个页面，“等待验证”会变成“成功”，如果没有变，那就再过一会儿再刷新看看，只有变成绿色的“成功”才可以  
![](https://img.xiebruce.top/2022/08/28/7554a811fd98c67ff8e78068cc0eee23.jpg)

___

回到之前请求证书的地方，点击右侧的“刷新”按钮，然后下拉菜单中就可以选择刚刚请求的证书了  
![](https://img.xiebruce.top/2022/08/28/93029e304220d8b9305b481af5b0af86.jpg)

下图都默认就行，或者把http3也选上  
![image.jpg](https://img.xiebruce.top/2022/10/21/5eb7c61d5afbe72da748c66cb0f13682.jpg)

最终点“创建分配”  
![](https://img.xiebruce.top/2022/08/28/5235204bb2c30413651f75449d82d170.jpg)

可以看到显示创建成功，它会给你分配一个“分配域名”，然后下边还会有一个“备用域名”，这时你就会比较明白什么叫备用域名了，因为它已经给你分配了，你直接用它分配的域名就能用，但是用我们自己的域名通过添加cname后，也是可以走cdn加速的，所以才叫“备用域名”  
![](https://img.xiebruce.top/2022/08/28/39ab6e1db6c9025adb327760702f0bbe.jpg)

这两个域名都可以用来访问你的网站，只不过备用域名的好处就是自定义，更像是你自己的域名，而自动分配的域名就统一是cloudfront的域名(如果是做代理用的话，用cloudfront分配的域名也没什么，就是可能不容易记住)。

如果你想使用自定义域名(即上图的“备用域名”)，**你需要在域名解析服务商那边添加一个cname记录，把自己要套cdn的域名作为名称，而把cloudfront给你分配的域名作为“目标”**，添加之后就可以使用它了。

按照本例的示例域名，aa.test.com就是套了cdn的域名，你用aa.test.com访问你的网站就是走了cdn的，而bb.test.com则是一个用来中转用的域名，真实使用的时候并不使用它

你可以创建很多“分配”来分别代理到你的不同机器，这都是可以的。  
![](https://img.xiebruce.top/2022/09/02/992b641639f6b5bbd2044263c0124fbc.jpg)

如果你想删除一个分配，必须先禁用它，然后等待它生效，当“部署中”变成日期时(不用刷新网页，它会自动变)，禁用就生效了，然后你才能再次选中去删除它，而且删除后，如果再去分配，如果是同一个源域或同一个备用域名，都是无法直接分配的，需要等DNS缓存没了才行，一般要等一小时以上。

### 服务器端操作

Cloudfront会通过443端口和80端口，即https和http协议去请求你的服务器，你必须在你的服务器配置前面所说的“A域名”(套cdn的域名)和“B域名”(源域名)，如果是nginx，则server\_name应该把aa.test.com和bb.test.com都写上

bb.test.com作为源域名，不用说肯定要配置的，不然CDN服务器没法通过这个域名获取到源站的内容，而aa.test.com是cname到xxx.cloudfront.net，理论上来说aa.test.com并没有直接参与访问你服务器的内容，为什么它也要在nginx中配置呢？

原因是，你在浏览器(或其它客户端)发起请求的时候，用的是aa.test.com，虽然DNS那边最终会把cname的域名(即xxx.cloudfront.net)对应的ip返回，但浏览器并不知道这个ip是cname那边的，所以GET请求中header中的Host值还是aa.test.com，这个请求最终会被CDN服务器转发到你的VPS中，那么VPS中的nginx接收到这个请求，就会根据Host值判断应该用哪个server模块来处理你的请求(如果你同一个端口有多个模块的话)，当然如果同一端口只有一个模块，其实server\_name可以不写bb.test.com，因为只有一个它默认就进去了，但建议还是写比较好。

另外，server模块中的证书，一般来说用通配符证书都是能匹配上，如果是单独指定多个域名，申请证书时记得把aa.test.com和bb.test.com的证书都加上。

一般报错502都是由于源站域名不包含在证书里(或证书过期)造成的，详见：[如何排查来自 CloudFront 的 502：“无法满足请求”错误？](https://aws.amazon.com/cn/premiumsupport/knowledge-center/cloudfront-502-errors/)。

参考：https://www.banzhuti.com/free-cdn-amazon-cloudfront-setting.html