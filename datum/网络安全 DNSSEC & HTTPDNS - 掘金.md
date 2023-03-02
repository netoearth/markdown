> 图片来源：[unsplash.com/photos/Q1p7…](https://link.juejin.cn/?target=https%3A%2F%2Funsplash.com%2Fphotos%2FQ1p7bh3SHj8 "https://unsplash.com/photos/Q1p7bh3SHj8")

> 作者：念山

## 背景

随着技术的发展，网络环境也变得越来越复杂，而对于一个以网络数据传输提供服务的 App 来讲，在复杂多变的网络环境下安全稳定有效的提供好服务显得尤为重要。而为了提供安全稳定有效的 HTTP 网络服务，我们从网络请求的初始阶段 DNS 解析上保证 DNS 安全性的技术：去分析下苹果2022年 WWDC 讲到的 DNSSEC 技术和我们云音乐现行的 HTTPDNS 并做下对比。

## 什么是 DNS

域名系统（Domain Name System，DNS） 则是将域名解析 IP 地址的一项互联网基础服务，提供该服务的服务器称为 域名服务器（Domain Name Server）。

域名（Domain Name，Domain） 是一个在互联网上标识主机或主机组的名称，相当于 IP 地址的别名，相对于晦涩难记的 IP 地址，域名更显得易于记忆。

### 域名系统是怎么工作的呢

互联网上的域名系统是一个分布式的系统，结构上是一个四层的树状层次结构：

![域名系统](https://p6.music.126.net/obj/wonDlsKUwrLClGjCm8Kx/24871026782/27a5/9cd2/6e09/520658f4c56bc80bdc3ead89f9315697.png)

-   本地域名服务器（Local Name Server，local DNS）：如果通过 DHCP 配置，local DNS 由互联网服务提供商（ISP，如联通、电信）提供
    
-   根域名服务器（Root Name Server）：当 local DNS 查询不到解析结果时，第一步会向它进行查询，并获取顶级域名服务器的IP地址。全球一共有 13 个根域名服务器（除了它们的镜像），它们并不直接用于域名解析，仅用于指出可查询的顶级域名服务器。
    
-   顶级域名服务器（Top-level Name Server）：负责管理在该顶级域名服务器下注册的二级域名，例如\*\*.com 顶级域名服务器\*\*，而 baidu.com 权威服务器是注册在 .com 的权威域名服务器
    
-   权威域名服务器（Authoritative Name Server）：在特定区域内具有唯一性，负责维护该区域内的域名与 IP 地址的映射关系。在 DNS 应答报文中，标志位 AA 标识本次 DNS 记录是否来自权威域名服务器，否则可能来自缓存
    

#### 模拟流程

那以我们主站域名 interface.music.163.com 为例，来看一下 DNS 的过程：

![DNS流程](https://p6.music.126.net/obj/wonDlsKUwrLClGjCm8Kx/24497101540/c5fc/36c2/c7ad/fa43d4a2f914e687061264790e8b3d61.jpg)

-   0、首先检查 DNS 缓存，如果缓存老化或未命中，客户端需要向 local DNS 发送查询请求报文
    
-   1、客户端向 local DNS 发送查询报文 query interface.music.163.com，local DNS 首先检查自身缓存，如果存在记录则直接返回结果，查询结束；如果缓存老化或未命中，则：
    
-   2、local DNS 向根域名服务器发送查询报文 query interface.music.163.com，返回 .com 顶级域名服务器的地址（如果查无记录）
    
-   3、local DNS 向 .com 顶级域名服务器发送查询报文 query interface.music.163.com，返回interface.music.163.com所在的权威域名服务器的地址（如果查无记录）
    
-   4、local DNS 向 interface.music.163.com 权威域名服务器发送查询报文 query interface.music.163.com，得到 ip 地址，存入自身缓存并返回给客户端
    

### DNS 特点

DNS 从 1983 年设计推出到现在已经在全球运作 40 年了，设备支持高，但是由于最初设计的时候没有考虑安全相关的问题，报文无鉴权和加密相关的信息，带来了不稳定的因素。

#### 优势

-   系统支持
-   速度快

#### 劣势

-   中间人攻击
    -   DNS 欺骗
    -   DNS 劫持
-   结构复杂，中间变数多

针对 DNS 不安全的情况，业界也在通过 DNSSEC、HTTPDNS 等技术方案补充来去保证安全提高服务可用。

## 什么是 DNSSEC

DNSSEC 可以说是在原有 DNS 基础之上做的扩展。 网域名称系统安全性扩充 (DNSSEC) 可为网域名称的 DNS (网域名称系统) 加上电子签名，藉此判断来源网路名称的真实性。此功能可以保护网路使用者不受假造 DNS 资料的威胁，让使用者要求正确网址时不会取得其他有意误导或恶意制作的网址。

### DNSSEC 工作的流程

DNSSEC 通过建立信任链来验证记录 ![DNSSEC流程](https://p6.music.126.net/obj/wonDlsKUwrLClGjCm8Kx/24529907702/c2d8/ac95/f653/a703a8d6bb12626d5e8d895bfca737dc.png)

举个例子：设备想要解析 [www.example.org](https://link.juejin.cn/?target=http%3A%2F%2Fwww.example.org "http://www.example.org") 并启用 DNSSEC 验证，过程如下：

-   发送询问 IP 地址、签名和密钥的查询，通过响应可以建立从 IP 地址到密钥 1 的信任关系
    
-   客户端向父区域 org 发送查询，询问可用于验证密钥 1 的记录，建立从密钥 1 到密钥 2 的信任关系
    
-   设备递归地重复这个过程，直到它到达根域
    

### DNSSEC 特点

通过流程总结下来

#### 优势

-   提供校验保证安全

##### 劣势

-   需要路由器支持，路由器必须支持处理大于正常大小的 DNS 包
    -   正常的 DNS 包大小为 512 字节， DNSSEC 则大于 512 字节，国内较多路由器会丢弃超过 512 字节的 DNS 包
-   域名注册服务商配置大部分是付费的
-   云商支持弱
    -   较多云商不支持 DNSSEC 解析
    -   支持 DNSSEC 解析的基本还收费
-   需要权威服务器支持
-   系统兼容差
    -   iOS16+ 支持
    -   macOS Ventura 以上才支持

## 是否要选择 DNSSEC？

DNSSEC 方案对于我们 App 来讲不可控因素太多，且可扩展性较弱所以我们选择了一套同样安全，且更为可控的方案-- HTTPDNS。

## HTTPDNS

相较于 DN S和 DNSSEC，对于客户端来讲有一种可控性更强和更安全的方案：HTTPDNS，而 HTTPDNS 也是我们云音乐现在正在运行的方案。

### 什么是 HTTPDNS

DNSOverHTTP，通过 HTTP 报文请求来去拉取 DNS 的结果，客户端拿到结果后拿 IP 替换请求的 host 并设置 SNI 来去进行IP直连的请求，达到控制请求目的 IP 选择的结果，达到等同 DNS 的功能。

### HTTPDNS 流程

-   客户端请求自己 HTTPDNS 服务器拿到 IP
-   发请求替换 host 为对应的 IP
-   设置 SNI 来添加原始 host 信息

### HTTPDNS 特点

#### 优势

-   可定制化强 : 服务走自己服务器
-   安全 : 走 https 请求

#### 劣势

-   相较于 LocalDNS 时间损耗长：由于采用 HTTP 报文来去传输 domain 与 IP 的映射表
-   需要自己处理替换域名的操作。
-   具有一定的安全风险：iPhone 需要调用私有 API 去设置 SNI

### 安全风险

一个比较大的风险点是在使用私有 API 来去设置 SNI。

在基于目前网络基本使用 NSURLSession 的大背景下，在使用 HTTPDNS 技术的情况下有个重要的一个过程是通过私有 API 来设置 SNI 信息通过 HTTPS 的握手校验。但是私有 API 有随时被封禁的风险，如若被封禁则使用私有 API 来做 SNI 设置的HTTPS请求都不可用。

#### 怎么来规避使用私有 API 这个安全风险呢？

##### 核心问题

NSURLSession 无法控制 Socket 连接，所以只能替换 host 的方式来去处理，如果通过控制 Socket 连接池，就能规避掉 SNI 的问题

##### 新的网络库

我们选择一套新的网络库 Cronet 既能控制 Socket 连接池，又能带来网络性能相关的提升及网络高级特性的支持。

### IP 跑马

HTTPDNS 拿到的是一个 IP 数组，实际发请求的时候只需要一个 IP，那到底使用哪个 IP 来进行建连请求呢？那就需要通过一定的计分标准来给 IP 打分，通过 IP 跑马来拿到最佳 IP。

#### 计分规则

##### 单次请求的评分

单次请求评分按照正确请求和错误请求两种不同的计分方式来进行处理。

###### 错误评分

把网络错误细分为5个等级，不同等级扣分分值不一样

```
typedef NS_ENUM(NSInteger, NENetErrorLevel)
{
    NENetErrorLevelNone, // 无错误，不扣分
    NENetErrorLevelDefault, // 默认，扣1分
    NENetErrorLevelisCancel, //取消不算错误，不扣分
    NENetErrorLevelNormal,  // 普通级别 扣10分
    NENetErrorLevelSerious, // 严重  扣20分
};

// 把对应的NSURLSession的网络报错对应到错误等级进行评分
- (NENetErrorLevel)errorLevelForError:(NSError *)error
{
    NENetErrorLevel errorLevel =NENetErrorLevelNone;
    BOOL isCancel = NO;
    BOOL isChainError = NO;
    BOOL isTimeout = NO;
    NSError *urlError = error;
    
    NSInteger errorCode = urlError.code;
    if ([urlError.domain isEqualToString:NSURLErrorDomain] ||
        [urlError.domain isEqualToString:@"AFNetworkingErrorDomain"])
    {
        if (errorCode == NSURLErrorCannotConnectToHost)
        {
            isChainError = YES;
        }
        isTimeout = errorCode == NSURLErrorTimedOut;
        isCancel = errorCode == NSURLErrorCancelled;
    }
    else if ([urlError.domain isEqualToString:NSPOSIXErrorDomain])
    {
        if (errorCode == 54 || //ECONNRESET
            errorCode == 57 || //ENOTCONN
            errorCode == 60 || //ETIMEDOUT
            errorCode == 61 ||
            errorCode == 65
            )   //ECONNREFUSED
        {
            isChainError = YES;
        }
    }
    
    if (isCancel) {
        errorLevel = NENetErrorLevelisCancel;
    } else if (isChainError) {
        errorLevel = NENetErrorLevelSerious;
    } else if (isTimeout) {
        errorLevel = NENetErrorLevelNormal;
    } else if (error) {
        errorLevel = NENetErrorLevelDefault;
    }
    return errorLevel;
}

复制代码
```

###### 正确的评分

正确的评分采用请求速度来去考量分值，选择基准线速度和时间，在基准线上线浮动进行算分。

耗时<=2.5s 才会去按照基准线进行计算分 耗时>4.5s 开始扣分

```
// 计算公式
- (double)calcllateSuccessMarkWithDataLength:(NSUInteger)dataLength cost:(NSTimeInterval)cost
{
    if (cost == 0) {
        return 0;
    }
    double mark;
    long delta = 0;
    if (cost <= CALL_TIME_GOOD_BENCH_MARK) { //如果<= 2.25s， 才有积分机会
        delta = CALL_TIME_GOOD_BENCH_MARK - cost;
    } else if (cost > CALL_TIME_BAD_BENCH_MARK) { //如果 > 4.5s, 则开始扣分
        delta = CALL_TIME_BAD_BENCH_MARK - cost;
    }
    // 超时不一定是IP有问题
    //通过耗时维度计算出的mark 最低-5分
    mark = MAX(delta,-5);
    if (dataLength > 0 && mark > 0) {
        double speed = dataLength / (cost + 0.0f);
        if (speed < CALL_SPEED_GOOD_BENCH_MARK) { //利用speed， 将请求体小的请求分数降低
            mark = speed / CALL_SPEED_GOOD_BENCH_MARK * mark;
        }
    }
    return mark;
}

复制代码
```

##### IP 总分

IP 总分有两次策略：累计计分、瞬时计分

###### 累计计分

当前网络环境下（WiFi/蜂窝）的所有请求的分值的加和。

###### 瞬时计分

当前网络环境下（WiFi/蜂窝）的当前请求的分值就是IP的最终得分。

#### 选择IP

请求发出的时候根据拿到的 IP 组按照 IP 的跑分拿最高分作为当成请求的 IP。

#### 整体流程图

![跑分整体流程](https://p6.music.126.net/obj/wonDlsKUwrLClGjCm8Kx/24530489195/9c7b/6740/9a6b/2d9afefa2b5da129ff0715631dbd09a4.png)

## 4\. 小结与展望

网络安全不容小觑，各大厂商都在尽力去通过技术手段来去维护网络安全，来给用户提供一个安全可靠的网络环境。我们也希望苹果在后续可以在DNS相关方面提供更开放可由开发人员来去实现策略的 API，在保证网络安全基础上又能让开发人员能有一定的灵活性。针对文中 Cronet 的使用由于还在灰度推进中，等推全后也将梳理篇 Cronet 接入带来的利好及坑点来和大家交流。

## 参考资料

\[1\] [WWDC - 10079](https://link.juejin.cn/?target=https%3A%2F%2Fdeveloper.apple.com%2Fvideos%2Fplay%2Fwwdc2022%2F10079%2F "https://developer.apple.com/videos/play/wwdc2022/10079/")

\[2\] [DNSSEC 协议](https://link.juejin.cn/?target=https%3A%2F%2Fdatatracker.ietf.org%2Fdoc%2Fhtml%2Frfc4956 "https://datatracker.ietf.org/doc/html/rfc4956")

\[3\] [DNS劫持](https://link.juejin.cn/?target=https%3A%2F%2Fhelp.aliyun.com%2Fdocument_detail%2F62842.html "https://help.aliyun.com/document_detail/62842.html")

\[4\] [DNS欺骗](https://link.juejin.cn/?target=https%3A%2F%2Fbaike.baidu.com%2Fitem%2FDNS%25E6%25AC%25BA%25E9%25AA%2597%2F8759836 "https://baike.baidu.com/item/DNS%E6%AC%BA%E9%AA%97/8759836")

\[5\] [Enable encrypted DNS](https://link.juejin.cn/?target=https%3A%2F%2Fdeveloper.apple.com%2Fvideos%2Fplay%2Fwwdc2020%2F10047 "https://developer.apple.com/videos/play/wwdc2020/10047")

\[6\] [Cronet 网络请求流程](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fchromium%2Fchromium%2Fblob%2Fmaster%2Fnet%2Fdocs%2Flife-of-a-url-request.md "https://github.com/chromium/chromium/blob/master/net/docs/life-of-a-url-request.md")

> 本文发布自网易云音乐技术团队，文章未经授权禁止任何形式的转载。我们常年招收各类技术岗位，如果你准备换工作，又恰好喜欢云音乐，那就加入我们 grp.music-fe(at)corp.netease.com !