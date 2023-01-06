## 概述

这是一篇放了两三年的存货文章了，因为没有啥特别的，所以一直没有放出来。最近因为在做一个需求的时候用到了一些 HTTPS 的内容，所以又翻到了这篇文章，所以简单总结了一下就发布了，没啥特别的，就是老生常谈 HTTPS 建立过程的事情，本来应该还可以扩展介绍一下 HTTP2 和 QUIC 的相关内容，但是基于目前写的意愿不是很大，所以就没有展开来聊聊了。

## TLS 和 SSL

### 什么是 SSL

SSL是安全套接字层（Secure Socket Layer）的缩写，简而言之，它是保持互联网连接安全的标准技术，保护两个系统之间发送的任何敏感数据，防止犯罪分子读取和修改任何传输的信息，包括潜在的个人详细资料。这两个系统可以是一个服务器和一个客户端（例如，一个购物网站和浏览器）或服务器到服务器（例如，一个有个人身份信息或工资信息的应用程序）。

它通过确保用户和网站之间或两个系统之间传输的任何数据都不可能被读取。它使用加密算法对传输中的数据进行窜改，防止黑客在连接中读取数据。这些信息可能是任何敏感或个人的信息，包括信用卡号码和其他财务信息、姓名和地址。

### 什么是 TLS

TLS（Transport Layer Security）只是SSL的一个更新、更安全的版本。我们仍然将我们的安全证书称为SSL，因为它是一个更常用的术语，但当你从 DigiCert 购买 SSL 时，你实际上是在购买最新的TLS 证书，可选择 ECC、RSA 或 DSA加密。

当网站由SSL证书保护时，HTTPS（超文本传输协议安全）出现在URL中。点击浏览器栏上的锁符号可以查看证书的详细信息，包括签发机构和网站所有者的公司名称。

### TCP 连接

https 需要先进行 TCP 的连接建立，然后再进行 TLS 握手，最后再发送消息。

三次握手和四次挥手

### RSA 握手

#### 会话密钥的生成

1.  Client 生成随机数 1，将使用的协议版本，支持的加密套件以及随机数，通过发送 hello 数据包给 Server
2.  Server 生成随机数 2，将首选的密码套件，服务器证书以及随机数 2 响应给 Client（明文）
    -   这里 Client 会通过 server 证书签发的 CA 证书校验 Server 证书的有效性，确保真的是 server 证书
3.  Client 生成随机数 3，所谓的 “准 master secret”，然后通过服务器证书中的 **公钥** 加密发送给服务器
4.  Server 通过 **私钥** 解密获取 “准 master secret”（唯一一次使用私钥）
5.  Client 和 Server 都有客户端随机数、服务器随机数和准主密钥。将这三个输入组合起来可得到 “会话主密钥（master secret）”，会话期间的所有后续通信都用这些新密钥进行加密。

| **图 1：RSA 证书握手过程** |
| --- |
| ![](https://cdn.pyer.dev/blog-cdn/2021/02/03/16/27/24/143e8e03a882.jpg) |
| From: [Cloudflare](https://www.cloudflare.com/learning/ssl/keyless-ssl/) |

#### Keyless SSL

如果将应用部署在云上，但是私钥不存在在云上（出于安全考虑放在本地），那么第 4 步中需要的私钥也就需要放在云上，所谓的 keyless 就是在第 4 步的时候，云将加密的 “预主密钥” 转发给自己部署的服务器，然后由服务器进行解密，再返回给云，过程为：

| **图 2：Keyless SSL 握手过程** |
| --- |
| ![](https://cdn.pyer.dev/blog-cdn/2021/02/03/16/35/19/ee6ec65dac65.jpg) |
| From: [Cloudflare](https://www.cloudflare.com/learning/ssl/keyless-ssl/) |

#### 一点思考

-   为什么要生成 3 个随机数

在第一次握手时，发送了一个 Client Random，然后 Server 回了一个 Server Random，这两个 Random 都是不安全的；在 Client 收到 Server 证书之后，通过证书加密了一个 Premaster Secret，这个是安全的，那为什么最终的 session key 要使用 3 个 Random，而不是直接使用最后一个加密了的 Random 就可以了？

解答：主要是处于对 [**重放攻击**](https://en.wikipedia.org/wiki/Replay_attack) 的考虑，如果只使用最后一次 **准 master secret**，那么如果有一个中间人重放了这个请求，那么这条连接就会使用同样的密钥（因为对密码学不是很了解，所以不知道这样的伤害程度有多高，但是，从看的一些文章来看这不是一个好的实践）。所以，这个时候，如果还有一个 server 的随机数，那么被重放攻击时，因为了 server 随机数的存在，就可以抵御重放攻击了，因为每次重放客户端的请求得到的会话主密钥都是不一样的（server random 不一样）。

### ECDHE 握手

1.  Client 生成随机数 1，将使用的协议版本，支持的加密套件列表以及随机数，通过发送 hello 数据包给 Server
2.  Server 使用其私钥对 Client Random、Server Random 及 _Server DH 参数_ 进行加密，加密后的数据用作服务器的数字签名，从而确定服务器具有与 SSL 证书中的公钥相匹配的私钥，除了这个加密数据，还包含服务器的 SSL 证书，选定的密码套件以及 Server Random。（加密，唯一一次使用密钥的地方）
3.  Client 使用公钥解密服务器的消息并验证 SSL 证书。然后，使用 Client DH 参数回复
4.  双方使用 Client DH 参数和 Server DH 参数，彼此分开计算预主机密
5.  他们将此预主机密与客户端随机数和服务器随机数组合以获取会话密钥

| **图 3：ECDHE 握手过程** |
| --- |
| ![](https://cdn.pyer.dev/blog-cdn/2021/02/03/16/40/32/2ffb98197cf3.jpg) |
| From: [Cloudflare](https://www.cloudflare.com/learning/ssl/keyless-ssl/) |

#### Keyless SSL

| **图 4：ECHDE Keyless 握手过程** |
| --- |
| ![](https://cdn.pyer.dev/blog-cdn/2021/02/03/16/43/16/7e315d83e2f8.jpg) |
| From: [Cloudflare](https://www.cloudflare.com/learning/ssl/keyless-ssl/) |

## 什么是前向保密？

前向保密确保加密的数据即使私钥公开时仍保持加密，这也称为“完美前向保密”。如果每个通信会话使用唯一的会话密钥，并且会话密钥与私钥分开生成，则前向保密是可能的。如果单个会话密钥受损，则攻击者只能解密该会话；所有其他会话都将保持加密状态。

在为前向保密设置的协议中，私钥在初始握手过程中用于身份验证，否则不用于加密。短暂 Diffie-Hellman 握手将会话密钥与私钥分开生成，因此具有前向保密。

相反，RSA 没有前向保密；在私钥受损的情况下，攻击者可以解密过去对话的会话密钥，因为他们可以解密明文形式的预主机密以及客户端随机数和服务器随机数。通过将这三者结合起来，攻击者可以得到任何指定的会话密钥。

___

简单理解：如果密钥丢了，以前的历史记录（包括握手过程都被人记录下来了）会不会被破解。如果能破解，那么就不是，不能，那么就是前向保密。

## 数字证书

### 证书的组成

首先，有一个权威的证书签发机构，称为CA——全球就那么几个公司比较权威啦，这个机构，先用RSA产生一对公私钥，私钥自己留着藏起来，然后用自己的私钥对自己的公钥进行签名，生成所谓的数字证书。

这个过程大概是这样的：

-   server 生成一个文件，文件内容大概是这样的（以下内容都是明文。我们称为内容P）：
    -   server 的域名
    -   签发者 ID：谁签发的证书
    -   Subject：也就是这个证书签发给谁。这里 subject 和 domain 应该相同
    -   有效日期（包括开始日期和结束日期，精确到秒）
    -   签发日期
    -   公钥内容
    -   其他信息：例如 server 的提供者所在的国家，地区等
-   然后使用 hash 算法，对内容 P 进行 hash 计算，得到一个 hash 值 H
-   然后使用签发机构的私钥对 H 进行 RSA 加密，得到签名信息 S（ 数字签名/ Digital Signature）
    -   这个步骤称为签名，就是用私钥对某公开内容的 hash 值进行加密。
-   然后将 P，S 连成一个文件，这个文件就是所谓的数字证书了。

## Ref

-   [Cloudflare 关于 https 连接建立的过程介绍](https://www.cloudflare.com/zh-cn/learning/ssl/what-happens-in-a-tls-handshake/)
-   [通过抓包分析 https 的连接建立过程](http://www.moserware.com/2009/06/first-few-milliseconds-of-https.html)