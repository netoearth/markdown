CloudFlare是全球领先的CDN服务提供商，其提供的免费且足量的服务是诸多站长遭受网络攻击时的“避风港”。

![](https://cdn.luotianyi.vc/wp-content/uploads/2022-03-08_02-35-15.jpg)

在很长一段时间里，博主看到身边很多人在遭受攻击时只是简单地打开`Under Attack`模式，完全没有利用到自定义WAF功能。恰好前段时间在论坛看到了相应的话题，稍作整理，这篇博客就来简单分享下博主觉得比较实用的防火墙规则。

___

首先，简单介绍下今天的主角【防火墙规则】，CF向免费版用户（包括Plesk Plus版）提供了5条防火墙规则，可通过【防火墙】-【防火墙规则】进行配置。**自定义防火墙规则的目的，就是圈定包含一定特征（如IP、UA、地域、提供商等）的可疑对象，并对其进行验证码质询或阻止访问**。充分认识这个目标，后续的所有逻辑都将围绕它来展开。

![](https://cdn.luotianyi.vc/wp-content/uploads/2022-03-06_14-43-15.jpg)

此外，防火墙规则的触发机制是自上而下触发一次，高优先级的规则要放在上部。在规则设置中需要灵活组合匹配条件，`and`需要全部满足、`or`为满足任一条件。下文提供的匹配规则，请通过【编辑表达式】功能修改并输入。

![](https://cdn.luotianyi.vc/wp-content/uploads/2022-03-06_14-24-13.jpg)

最后在开始配置防火墙规则之前，请先前往【防火墙】-【设置】下，将【Privacy Pass支持】关闭，以避免一种绕过质询的可能（尽管如此，免费版的`hCaptcha`依然存在可绕过的方式，所以在较严重的情况下应适当选择阻止部分访问）。

___

★本文参考自以下项目，在此致谢~

> **Hostloc**：[@t9913085](https://hostloc.com/thread-960122-1-1.html)的规则分享和[@欧阳逍遥](https://hostloc.com/forum.php?mod=redirect&goto=findpost&ptid=917540&pid=11301187)的高风险IP列表  
> **GitHub**：[Cloudflare Block Bad Bot Ruleset](https://github.com/XMD0718/cloudflare-block-bad-bot-ruleset)

___

## 一、白名单放行

> **★目标1**：放行已知Bot及白名单IP地址

放在最高优先级的目标是允许已知的正常流量通过防火墙，因为我们后续的配置比如针对机房AS会使得验证施加于GoogleBot之类的搜索引擎爬虫，此外白名单IP可以包括自己的调试IP、已知的善意rss爬虫等。

![](https://cdn.luotianyi.vc/wp-content/uploads/2022-03-08_03-40-33.jpg)

如图，Bot可以直接选用CF提供的【合法机器人爬虫】，IP白名单选用包含以下各项，选择操作为允许。

> **匹配规则**：[点击前往](https://static.lty.fun/%E5%85%B6%E4%BB%96%E8%B5%84%E6%BA%90/CF/WAF/Rule-WhiteList.txt)

___

## 二、IDC ASN验证

> **★目标2**：验证来自常见数据中心的访客

第二条规则，利用的是CF对ASN的判断。这里提到的ASN为自治网络的代码，如AS4134为中国电信。以下列表中基本是数据中心服务商的代号，因为对外提供租赁它们也是常见的网络攻击来源，而非海外真实访客常用的家庭宽带ISP。

![](https://cdn.luotianyi.vc/wp-content/uploads/2022-03-08_04-04-41.jpg)

如图，配置为ASN的包含以下各项，选择操作一般为验证码【质询】或者【JS质询】，在遭受严重攻击选择【阻止】。

> **ASN清单**：[点击前往](https://static.lty.fun/%E5%85%B6%E4%BB%96%E8%B5%84%E6%BA%90/CF/WAF/List-IDCAS.txt)  
> **匹配规则**：[点击前往](https://static.lty.fun/%E5%85%B6%E4%BB%96%E8%B5%84%E6%BA%90/CF/WAF/Rule-IDCAS.txt)

___

## 三、风险IP验证

> **★目标3**：验证收集的高风险IP地址

第三条规则，利用的是威胁分数和IP列表的判断。前者是 Cloudflare 用来确定 IP 信誉的分数，范围由好到差评为0-100；后者由蜜罐等方式抓取，可以结合自己实际表现进行增减。

![](https://cdn.luotianyi.vc/wp-content/uploads/2022-03-08_04-30-11.jpg)

由于提供的IP列表数量较大，直接配置超过了防火墙规则的字符上限，因此需要通过【管理账户】-【配置】-【列表】-【创建新列表】创建一个包含高风险IP的列表（供导入的csv在下方），再在防火墙中直接匹配此列表。  
![](https://cdn.luotianyi.vc/wp-content/uploads/2022-03-08_04-25-30.jpg)

如图，配置为IP源地址、在列表中以及威胁分数大于30。这一条由于覆盖范围较大误伤概率远高于上一条，选择操作建议为【JS质询】，在遭受攻击时再选择【质询】或【阻止】。

> **风险IP清单**：[点击前往](https://static.lty.fun/%E5%85%B6%E4%BB%96%E8%B5%84%E6%BA%90/CF/WAF/List-RiskIP.txt)  
> **风险IP CSV**：[点击前往](https://static.lty.fun/%E5%85%B6%E4%BB%96%E8%B5%84%E6%BA%90/CF/WAF/List-RiskIP.csv)  
> **匹配规则**：[点击前往](https://static.lty.fun/%E5%85%B6%E4%BB%96%E8%B5%84%E6%BA%90/CF/WAF/Rule-RiskIP.txt)

___

## 四、主机名细则

> **★目标4**：对单个主机的细则配置

前面提到了通过`and`和`or`组合规则，对于一些杂项，可以通过主机名`and`去限定区间，这里推荐一些常用的匹配方式。

<table><tbody><tr><td><strong>匹配规则</strong></td><td><strong>解释</strong></td></tr><tr><td>主机名</td><td>针对输入网站域名的配置</td></tr><tr><td>URL路径</td><td>针对访问链接路径中内容的匹配</td></tr><tr><td>国家/地区</td><td>针对访问IP来源地区的匹配</td></tr><tr><td>SSL/HTTPS</td><td>针对是否使用https访问的匹配</td></tr></tbody></table>

前三条内容与以上四个的组合可以很灵活地圈定范围，比如主机名+URL路径可以实现对特定目录、特定文件（比如登录页等）设置更高的验证要求。举个博主自己的例子，博主的静态资源所在目录会同步至海外源站，如下配置，就可以圈定php文件及不符合两个静态文件所在目录的访问，并阻止他们。类似的应用还有很多，可以多去思考和尝试~

![](https://cdn.luotianyi.vc/wp-content/uploads/2022-03-08_05-03-19.jpg)

这里也有前人总结的一些不友好的UA，对于阻止常见的扫描器等有一定的作用。这里没把它单独拿出来设置是博主觉得这是个防君子不防小人的做法，很容易绕过。在设置时，建议通过`and`组合主机名、http访问方式匹配，对全局开放的个人觉得意义不如前几个大，请自行结合实际去考量。

> **风险UA基础规则**：[点击前往](https://static.lty.fun/%E5%85%B6%E4%BB%96%E8%B5%84%E6%BA%90/CF/WAF/Rule-Basic-BadBotUA.txt)  
> **风险UA其他规则**：[点击前往](https://static.lty.fun/%E5%85%B6%E4%BB%96%E8%B5%84%E6%BA%90/CF/WAF/Rule-Additional-BadBotUA.txt)

___

## 五、请求速率限制

从2022年9月开始，CloudFlare开始向免费用户（包括Plesk Plus）提供一条免费的速率限制规则（[官方博客](https://blog.cloudflare.com/unmetered-ratelimiting/)）。该功能是防御CC强有效的功能，可以从【安全性】-【WAF-】【速率限制规则】进入配置页面。

![](https://cdn.luotianyi.vc/wp-content/uploads/2022-11-12_03-40-33.png)

针对免费用户来说可以配置的参数比较少，主要实现的是同个IP【10秒内请求数>n】时对其返回429阻止访问。如图填写`/`可对zone下所有域名进行保护，个人博客正常情况下n建议设置为50，资源更多的站点可以考虑适当放大（以不影响正常访问为准）。

![](https://cdn.luotianyi.vc/wp-content/uploads/2022-11-12_03-45-24.png)

___

## 六、结语

这篇博客对于CF WAF的配置分享到此告一段落了，但是对这些规则的应用仍有正则表达式等更高级的应用方式去探索，更详尽准确的黑名单列表也有待去总结……

每个网站，不论大小，都凝结了一个站长的心血的期望。所以在遇到攻击时，结合实际，准确地发掘并圈定恶意攻击特征，积极利用好这些已有的资源，最大限度减小对正常用户访问的影响是有必要的。

感谢前人的总结和分享，也期待继往开来的你们~

___

\*半原创文章，转载请注明引证出处