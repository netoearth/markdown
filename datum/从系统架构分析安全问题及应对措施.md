在日常生产生活中，我们常说，“安全第一”、“安全无小事”。围绕着安全问题，在各行各业都有对各类常见安全问题的解决方案和突发安全问题的应急预案。在互联网、软件开发领域，我们日常工作中对各类常见的安全问题又有哪些常见的解决方案呢？在此，结合经典架构图做一个梳理。

![](https://p3-sign.toutiaoimg.com/tos-cn-i-qvj2lq49k0/f21cb9b1681b40ef97d8fb0d535b7a1c~noop.image?_iz=58558&from=article.pc_detail&x-expires=1663121188&x-signature=VuPGraI4PbtBAlZbkEpQbgYBHmA%3D)

经典架构图

  
下面，结合上述的经典架构图，对数据存储、微服务接口、外网数据传输及 APP 层可能出现的安全问题进行分析，并给出一些常见的应对措施。

**1 数据存储**

为了保证数据存储的安全，对于敏感数据在进行存储时，需要进行加密存储，同时，敏感数据建议在全公司进行收口管理，便于统一管理。对敏感数据进行加密存储时，常见的加密方式有可逆加密和不可逆加密，分别适用于不同的敏感数据。

**1.1 可逆加密或对称加密**

可逆加密，即通过对密文进行解密后，能把密文解密还原出明文。对称加密算法加密和解密用到的密钥是相同的，这种加密方式加密速度非常快，适合经常发送数据的场合，缺点是密钥的传输比较麻烦。比如：网络购物的收货地址、姓名、手机号等就适合用该加密方式，常用的对称加密算法有 DES、AES，下面以 AES 为例说明对称加密的过程。

![](https://p3-sign.toutiaoimg.com/tos-cn-i-qvj2lq49k0/43a8bd70b7d34b3da6850d412be08962~noop.image?_iz=58558&from=article.pc_detail&x-expires=1663121188&x-signature=XYbbYEElxGBWn4hxqMEw0CuDi28%3D)

在该加解密中，对于秘钥 K 的生成需要加解密双方共同制定并妥善保管。通常，我们会把该秘钥 K 存储在需要使用加解密程序的进程内，便于在程序使用时直接进行使用。

**1.2 不可逆加密**

不可逆加密，即不需要解密出明文，如：用户的密码。不可逆加密常用的算法有 RES、MD5 等，在此以 MD5 为例进行说明。但大家都知道，MD5 算法是存在碰撞的，即不同的明文通过 MD5 加密后，存在相同的密文。因此，直接使用 MD5 对密码进行加密在生产上是不严谨的，通常还需要配合盐（salt）进行使用。对于盐的使用，也有一定的技巧，一种盐值是固定的，即所有的明文在进行加密时都使用相同的盐进行加密；另一种是结合具体的业务场景，用可变盐值，比如：就密码加密而言，可以把用户名的部分或全部作为盐值，和密码进行一起加密后存储。

![](https://p3-sign.toutiaoimg.com/tos-cn-i-qvj2lq49k0/49e9079931ff4b1290f9ead496dd8ea8~noop.image?_iz=58558&from=article.pc_detail&x-expires=1663121188&x-signature=R43bqt0I4usw8%2Fb1pBQlKnQAz1w%3D)

**2 微服务接口**

微服务的安全，需要从请求鉴权和请求容量限制这 2 个方面来考虑。对于请求鉴权，可以设置请求 IP 黑名单的方式，对该 IP 的所有请求或全部放行或全部拒绝，该方式的粒度较粗。而如果要做得较细粒度一些，可以针对具体的 API 进行 token 鉴权，相比粗粒度该方式会控制得比较精准；

```
<jsf:consumer id="setmentService" interface="com.jd.lfsp.jsf.service.SetmentService"
                  protocol="jsf" alias="${jsf.consumer.alias}" timeout="${jsf.consumer.timeout}" retries="0">
        <jsf:parameter key="token" value="${jsf.consumer.token}" hide="true" />
</jsf:consumer>
```

除了对请求鉴权外，在实际的生产中，还可以对请求容量进行限制，对请求容量进行限制时，可以按 QPS 进行限制，也可以对每天的最大请求次数进行限制。在 jsf 平台管理端，可以对具体的方法进行请求的 QPS 限流。

![](https://p3-sign.toutiaoimg.com/tos-cn-i-qvj2lq49k0/bce36771e05c418fa11eb904d7b5299c~noop.image?_iz=58558&from=article.pc_detail&x-expires=1663121188&x-signature=VLe83YTrKn1U037IP%2FqK7REDNxk%3D)

**3 数据传输**

数据传输主要分为数据通过前端 APP 的请求，进入到服务网关前和进入服务网关后这俩部分，对于数据已经进入服务网关后，这属于机房内的数据传输，通常这类加密意义不大，对这类的数据传输的安全需要建立相应的内部安全机制及流程规范，通过制度措施来保证。而数据在进入服务网关前，对数据的安全传输有哪些可做的。在数据请求进入服务网关前，通常我们有基于 SSL 协议的传输加密以及现在普遍通用的 HTTPS 加密。

![](https://p3-sign.toutiaoimg.com/tos-cn-i-qvj2lq49k0/fa6f1948d0cd4959b7db07c36c77d06d~noop.image?_iz=58558&from=article.pc_detail&x-expires=1663121188&x-signature=G3KxVAQ8AcPdUhTCU3oZvbMluvc%3D)

HTTPS 也是 HTTP 和 SSL 协议的结合体，所以在数据传输中，SSL 协议扮演了至关重要的角色。那 SSL 协议的工作过程是怎么样的，他是怎么保证数据传输过程中的安全的呢？下面为大家解析一下 SSL 协议的工作过程。

![](https://p3-sign.toutiaoimg.com/tos-cn-i-qvj2lq49k0/44ce1a0500bc480893d5d500474dbc1d~noop.image?_iz=58558&from=article.pc_detail&x-expires=1663121188&x-signature=VJ4OWTDOvlKMDE13kO6EuvLesKE%3D)

**SSL 客户端与 SSL 服务端验证的过程如下：**

1.  SSL 客户端向 SSL 服务端发送随机消息 ClientHello 的同时把自己支持的 SSL 版本、加密算法、秘钥交换算法、MAC 算法等信息一并发送；
2.  SSL 服务端收到 SSL 客户端的请求后，确定本次通信采用的 SSL 版本及加密组件和 MAC 算法，并通过 ServerHello 发送给 SSL 客户端；
3.  SSL 服务端将携带自己公钥信息的数字证书通过 Certificate 发送给 SSL 客户端；
4.  SSL 服务端通过 ServerHelloDone 消息通知 SSL 客户端版本和加密组件协商结束，开始进行秘钥交换；
5.  SSL 客户端验证 SSL 服务端发送的证书合法后，利用证书中的公钥加密随机数生成 ClientKeyExchange 发送给 SSL 服务端；
6.  SSL 客户端发送 ChangeCipherSpec 消息，通知 SSL 服务端后续将用协商好的秘钥及加密组件和 MAC 值；
7.  SSL 客户端计算已交互的握手消息的 hash 值，利用协商好的秘钥和加密组件加密 hash 值，并通过 Finished 消息发送给 SSL 服务端，SSL 服务端用相同的方法计算已交互的 hash 值，并与 Finished 消息进行对比，二者相同且 MAC 值相同，则秘钥和加密组件协商成功；
8.  同样地，SSL 服务端也通过 ChangeCipherSpec 消息通知客户端后续报文将采用协商好的秘钥及加密组件和 MAC 算法；
9.  SSL 服务端端计算已交互的握手消息的 hash 值，利用协商好的秘钥和加密组件加密 hash 值，并通过 Finished 消息发送给 SSL 客户，SSL 客户端用相同的方法计算已交互的 hash 值，并与 Finished 消息进行对比，二者相同且 MAC 值相同，则秘钥和加密组件协商成功；

通过上面的这个交互过程，我们可以看出，在使用 SSL 的过程中，除了客户端（浏览器）跟服务器之间的通讯外，其他的任何第三方想要获取到协商的秘钥是比较困难的。即使有比较厉害的人获取到了，基于目前用户在某个网站上的时效性，会影响我们对应秘钥的时效性，因此，造成的破坏性也比较有限。

**4 APP**

在 APP 层的安全问题，需要结合服务端一并来解决，在这主要介绍验证码这种形式。验证码作为一种人机识别手段，其主要作用是区分正常人操作还是机器的操作，拦截恶意行为。当前互联网中，大多数系统为了更好地提供服务，通常都需要用户进行注册。注册后，用户每次在使用系统时需要进行登录，登录过程中，为了防止系统非法使用，通常都需要用户进行登录操作，登录过程中，常用的验证方式主要通过验证码进行验证，当前比较常用的验证码有以下几种类型。

**4.1 短信验证码**

目前用得比较广泛的一种验证码形式，输入有效的手机号后，系统给手机号发送相应的短信验证码完成验证。

![](https://p9-sign.toutiaoimg.com/tos-cn-i-qvj2lq49k0/ea16585c0e59422ea0c460a8125c5df0~noop.image?_iz=58558&from=article.pc_detail&x-expires=1663121188&x-signature=TzvXzImNNg4zD9l4MBJvoYDZ8OQ%3D)

**4.2 语音验证码**

通过输入有效的手机号，系统给手机号拨打电话后，用语音播报的方式完成验证码的验证。

![](https://p3-sign.toutiaoimg.com/tos-cn-i-qvj2lq49k0/0bcbffd3ad744c17bb2fdf4eed4d7e43~noop.image?_iz=58558&from=article.pc_detail&x-expires=1663121188&x-signature=4vzuILqdz2MrFPsGbNUQ1QG7jYU%3D)

**4.3 图片验证码**

较传统的验证码验证方式，由系统给出验证码在页面显示，在进行页面提交时，验证码一并提交到系统后台验证。

![](https://p3-sign.toutiaoimg.com/tos-cn-i-qvj2lq49k0/856d88d9f7794585b3753bcbd16be6b3~noop.image?_iz=58558&from=article.pc_detail&x-expires=1663121188&x-signature=U%2Fx5U78NNgC3J7gL3GA%2FG7pgyYI%3D)

**4.4 语义验证码**

比较新颖的一种验证码形式，但是该种方式相比较而言对用户不是特别友好，需要慎用。

![](https://p3-sign.toutiaoimg.com/tos-cn-i-qvj2lq49k0/34724030f96948ad85adf32f78dc46fa~noop.image?_iz=58558&from=article.pc_detail&x-expires=1663121188&x-signature=eNM9sGPTybcu7D%2FXXTkA3ok4lvA%3D)

除了上述的几种目前常用的验证码外，还有文本验证码、拼图验证码、问题类验证码等，在此就不再一一列举，大家如果感兴趣可以自己去搜索、学习。

这主要从系统的架构上，分析了日常工作中我们所接触到的比较常见的一些安全问题及其应对措施，在实际工作的安全问题远不止这里提到的内容。希望在日常工作中，我们大家都绷紧安全的神经，时刻关注自己工作中的各类潜在的安全问题，争取把安全问题消灭在系统发布前。

**5 参考文献**

**SSL 是如何加密传输的数据的：**  
[\[技术每日说\] - SSL 是如何加密传输的数据的！](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fevergreen-tree.github.io%2Farticles%2F2016-05%2Fdaily-ssl-rsa-des-algorithm)

**名词解释：**

-   SSL：（Secure Socket Layer，安全套接字层），位于可靠的面向连接的网络层协议和应用层协议之间的一种协议层。SSL 通过互相认证、使用数字签名确保完整性、使用加密确保私密性，从而实现客户端和服务器之间的安全通讯。该协议由两层组成：SSL 记录协议和 SSL 握手协议。
-   HTTPS：（全称：Hypertext Transfer Protocol over Secure Socket Layer），是以安全为目标的 HTTP 通道，简单讲是 HTTP 的安全版（HTTP+SSL）。即 HTTP 下加入 SSL 层，HTTPS 的安全基础是 SSL，因此加密的详细内容就需要 SSL。

___

**作者：廖宗雄**