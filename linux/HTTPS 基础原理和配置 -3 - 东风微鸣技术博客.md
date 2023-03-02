书接上文：[HTTPS 基础原理和配置 - 2](https://ewhisper.cn/posts/8080/)，接下来介绍：

1.  配置 NGINX
2.  后端 HTTPS
3.  检查配置
4.  配置 HSTS
5.  OCSP Stapling

重要部分来了。如何使用这些选项并配置 NGINX?

## 一、NGINX 的 HTTPS 配置[](https://ewhisper.cn/posts/57169/#%E4%B8%80%E3%80%81NGINX-%20%E7%9A%84%20-HTTPS-%20%E9%85%8D%E7%BD%AE)

这里有一些基本的原语（或叫做指令），你可以使用:`ssl_certificate`、`ssl_certificate_key`、`ssl_protocols` 和`ssl_ciphers`。

### 1.1 NGINX 配置参数（OpenSSL）[](https://ewhisper.cn/posts/57169/#1-1-NGINX-%20%E9%85%8D%E7%BD%AE%E5%8F%82%E6%95%B0%EF%BC%88OpenSSL%EF%BC%89)

在开始之前: NGINX 处理 TLS 的方式是使用 OpenSSL，我相信你已经在新闻中听说过这个库。它因 Heartbleed 和其他一些漏洞而闻名。它确实是最广泛使用的内置加密库。这是 NGINX 用于加密的。

因此，在服务器上要做的一件事是检查正在使用的 OpenSSL 版本。你可能不想使用 0.9.8 之类的版本。你希望在 1.0.1p 或 1.0.2 范围内使用，因为多年来他们已经修复了许多 bug。你永远不知道下一个 OpenSSL 漏洞什么时候出现，但至少现在它是非常可靠的(1.0.1p)。它还拥有所有现代加密算法。

### 1.2 NGINX 配置证书链和私钥[](https://ewhisper.cn/posts/57169/#1-2-NGINX-%20%E9%85%8D%E7%BD%AE%E8%AF%81%E4%B9%A6%E9%93%BE%E5%92%8C%E7%A7%81%E9%92%A5)

所以，当你在 NGINX 中设置你的服务器部分时，`ssl_certificate` 就是你的证书链。这是你的证书加上所有的信任链一直到根证书。然后你还需要提供你的私钥。

`ssl_certificate_key` 就是你的私钥。

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br><span>8</span><br><span>9</span><br><span>10</span><br><span>11</span><br><span>12</span><br><span>13</span><br><span>14</span><br><span>15</span><br><span>16</span><br><span>17</span><br><span>18</span><br></pre></td><td><pre><code><span>server</span> {<br>    <span>listen</span> <span>443</span> ssl http2;<br>    <span>listen</span> [::]:<span>443</span> ssl http2;<p>    <span>ssl_certificate</span> /path/to/signed_cert_plus_intermediates;<br>    <span>ssl_certificate_key</span> /path/to/private_key;<br>    <span>ssl_session_timeout</span> <span>1d</span>;<br>    <span>ssl_session_cache</span> shared:MozSSL:<span>10m</span>;  <span># about 40000 sessions</span><br>    <span>ssl_session_tickets</span> <span>off</span>;</p><p>    <span># curl https://ssl-config.mozilla.org/ffdhe2048.txt &gt; /path/to/dhparam</span><br>    <span>ssl_dhparam</span> /path/to/dhparam;</p><p>    <span># intermediate configuration</span><br>    <span>ssl_protocols</span> TLSv1.<span>2</span> TLSv1.<span>3</span>;<br>    <span>ssl_ciphers</span> ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384;<br>    <span>ssl_prefer_server_ciphers</span> <span>on</span>;<br>}</p></code><p><i></i>NGINX</p></pre></td></tr></tbody></table>

### 1.3 额外选项[](https://ewhisper.cn/posts/57169/#1-3-%20%E9%A2%9D%E5%A4%96%E9%80%89%E9%A1%B9)

你还可以添加一些与会话恢复有关的额外选项。如前所述，当你第一次建立 TLS 连接时，需要额外的两次往返，因为你必须完成整个握手和交换证书。如果你以前连接过一个客户端，并且他们已经缓存了会话传输所使用的密钥，那么你可以只恢复该会话。这是一个称为会话恢复的特性。

你只需要一个超时来说明你希望将会话保留多长时间，以及这些会话的缓存可以有多大。在本例中，默认是 10 MB 的会话; 那应该够你用很长时间了。共享缓存是首选，因为这样你就可以在所有 NGINX worker 之间共享它们。

例如，如果你的一个 worker 是最初建立连接的那个，而第二个连接被建立到另一个 NGINX worker，你仍然可以恢复连接。还有另一个选项叫做「session ticket」。它只在 Chrome 内核浏览器和 Firefox 中使用，但本质上是一样的。你必须生成一个随机的 48 字节文件，但我建议暂时只使用会话缓存。

### 1.4 NGINX 的协议和密码配置[](https://ewhisper.cn/posts/57169/#1-4-NGINX-%20%E7%9A%84%E5%8D%8F%E8%AE%AE%E5%92%8C%E5%AF%86%E7%A0%81%E9%85%8D%E7%BD%AE)

作为一个非常明显的下一步，你必须列出希望支持的协议和密码。在这种情况下，上文是 Mozilla 推荐的密码，以及从 1.2 版到 1.3 版的 TLS 协议。

### 1.5 其他[](https://ewhisper.cn/posts/57169/#1-5-%20%E5%85%B6%E4%BB%96)

我提到过你如何协商你选择的密码; 你可以选择客户端的选择或服务器的选择。最好选择服务器的选择。这里有一个指令: `ssl_prefer_server_ciphers`——始终打开它。

### 1.6 同一证书多域[](https://ewhisper.cn/posts/57169/#1-6-%20%E5%90%8C%E4%B8%80%E8%AF%81%E4%B9%A6%E5%A4%9A%E5%9F%9F)

如果你有多个站点，并且它们使用相同的证书，那么你实际上可以分解 HTTP 定义。你可以在顶层使用 SSL 证书，在底层使用不同的服务器。这里你需要记住的一件事是如果你有 _[example.com](http://example.com/)_ 和 _[example.org](http://example.org/)_ ，你必须有一个证书对这两个名字都有效，这样才能工作。

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br><span>8</span><br><span>9</span><br><span>10</span><br><span>11</span><br><span>12</span><br><span>13</span><br><span>14</span><br></pre></td><td><pre><code><span>ssl_certificate</span>multiSAN.crt;<br><span>ssl_certificate_key</span>multiSAN.key;<p><span>server</span> {<br>    <span>listen</span><span>443</span> ssl;<br>    <span>server_name</span>www.example.com;<br>    ...<br>}</p><p><span>server</span> {<br>    <span>listen</span><span>443</span> ssl;<br>    <span>server_name</span>www.example.org;<br>    ...<br>}</p></code><p><i></i>NGINX</p></pre></td></tr></tbody></table>

以上基本上就是为 NGINX 设置 HTTPS。

更高级的话题是: 如何使用 NGINX 作为其他 HTTPS 服务的代理?

[![后端加密](https://pic-cdn.ewhisper.cn/img/2021/09/24/e3ebdf5f6a0c424302907cb9c38e0be1-20210924114009.png)](https://pic-cdn.ewhisper.cn/img/2021/09/24/e3ebdf5f6a0c424302907cb9c38e0be1-20210924114009.png "后端加密")

我们称之为后端加密。所以，你的访客访问你的 NGINX 服务器是完全加密的。NGINX 背后发生了什么? 在这种情况下，NGINX 必须充当浏览器访问你的后端服务。

### 2.1 NGINX 后端配置[](https://ewhisper.cn/posts/57169/#2-1-NGINX-%20%E5%90%8E%E7%AB%AF%E9%85%8D%E7%BD%AE)

这可以在 NGINX 中以类似的方式配置。也有类似于 `ssl_protocols` 和 `ssl_ciphers` 的指令; 在本例中，你将它放在一个代理下。NGINX 作为客户端将使用 `proxy_ssl_protocols` 和 `proxy_ssl_cipher` 指令。

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br><span>8</span><br><span>9</span><br><span>10</span><br><span>11</span><br></pre></td><td><pre><code><span>http</span> {<br>    <span>server</span> {<br>        <span>proxy_ssl_protocols</span>TLSv1.<span>2</span> TLSv1.<span>3</span>;<span># 协议</span><br>        <span>proxy_ssl_ciphers</span>ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384;<span># 密码套件</span><br>        <span>proxy_ssl_trusted_certificate</span>/etc/ssl/certs/trusted_ca_cert.crt;<span># 受信任的 CA</span><p>                <span>proxy_ssl_verify</span><span>on</span>;<br>        <span>proxy_ssl_verify_depth</span><span>2</span>;<br>        proxy_ssl_session-<span>reuse</span><span>on</span>;<br>    }<br>}</p></code><p><i></i>NGINX</p></pre></td></tr></tbody></table>

我建议使用完全相同的密码套件和协议集。这里的主要区别是客户端对服务器进行身份验证。所以，在浏览器的情况下，你有一组你信任的证书颁发机构，而 NGINX 作为客户端，你也需要一组你信任的证书颁发机构。

### 2.2 受信任的 CA 的选项[](https://ewhisper.cn/posts/57169/#2-2-%20%E5%8F%97%E4%BF%A1%E4%BB%BB%E7%9A%84%20-CA-%20%E7%9A%84%E9%80%89%E9%A1%B9)

你可以使用两种不同的理念来实现这一目标，一种是创建你自己的内部证书颁发机构并在内部管理它。这有点棘手，但它更便宜，也更容易管理，因为你可以为任何服务颁发证书，并将它们颁发给你拥有并完全控制的证书颁发机构。在这种情况下，`proxy_ssl_trusted_certificate` 将被设置为你的证书颁发机构。

或者，你可以使用我在 [上一篇文章](https://ewhisper.cn/posts/8080/#4-3-%20%E6%88%91%E5%A6%82%E4%BD%95%E8%8E%B7%E5%BE%97%E4%B8%80%E4%B8%AA%E8%AF%81%E4%B9%A6%EF%BC%9F) 中描述的相同技术。你可以为你所有的服务购买证书，然后如果你的 NGINX 需要信任它们，它可以信任浏览器信任的同一套证书颁发机构。

对于 Ubuntu，磁盘上有一个列表，里面包含了几乎所有平台的所有证书。但是，如果你正在构建一组需要相互通信的大型服务，那么就很难为这些域颁发证书。你必须向证书颁发机构证明所有权才能实际获得证书。

我推荐内部 CA 机制。最困难的部分是——如何保证证书颁发机构的安全? 如何保证证书颁发机构的私钥的安全? 你可以通过使用脱机计算机和特殊管理员来实现这一点，但无论哪种情况，都存在一些挑战。

## 三、检查配置[](https://ewhisper.cn/posts/57169/#%E4%B8%89%E3%80%81%E6%A3%80%E6%9F%A5%E9%85%8D%E7%BD%AE)

NGINX 设置了 HTTPS。如何检查它的配置是否正确?

[![ssl labs 评分](https://pic-cdn.ewhisper.cn/img/2021/09/24/83be4a2b1ca6f8cd1023668b45763ba7-check-ssllabs.png)](https://pic-cdn.ewhisper.cn/img/2021/09/24/83be4a2b1ca6f8cd1023668b45763ba7-check-ssllabs.png "ssl labs 评分")

SSL Labs 是人们最喜欢的网站检查工具之一。SSL Labs 是 Qualys 运营的一个网站; 你只要输入你的域名，它就会运行所有类型的浏览器，所有类型的 SSL 连接，它会告诉你哪些设置正确，哪些设置错误。

在本例中，我们检查了一个名为 [badSSL.com](http://badssl.com/) 的网站，它列举了所有可能导致 HTTPS 配置混乱的不同方法。你可以用 SSL Labs 扫描每一个，它会告诉你每一个有什么问题。在本例中，给出的等级是 **C**，因为它支持 SSL v3.0。

这里还提到了其他一些东西，你们可以修改，但在我如何设置 NGINX 的描述中，如果你这样设置，你基本上会得到 **A**。这意味着证书协议支持、密钥交换和密码强度都是顶级的。

[![ssl labs 评分](https://pic-cdn.ewhisper.cn/img/2021/09/24/d705ad08fdbf2a820bfdfc20d935be13-image-20210924115648206.png)](https://pic-cdn.ewhisper.cn/img/2021/09/24/d705ad08fdbf2a820bfdfc20d935be13-image-20210924115648206.png "ssl labs 评分")

### 3.1 CFSSL 扫描[](https://ewhisper.cn/posts/57169/#3-1-CFSSL-%20%E6%89%AB%E6%8F%8F)

这对互联网网站非常有效; 如果你有防火墙或 NGINX 后面的服务，CloudFlare 构建了这个叫做 CFSSL 扫描的工具。你可以在内部基础架构中使用它; 它是开源的，在 [cloudflare/cfssl: CFSSL: Cloudflare’s PKI and TLS toolkit (github.com)](https://github.com/cloudflare/cfssl)上。它将做本质上与 SSL Labs 相同的事情，只是在你的基础设施内。它会告诉你什么是对的，什么是错的。

[![CFSSL 扫描](https://pic-cdn.ewhisper.cn/img/2021/09/24/4ad0942f624387c5d5040c8dc8312426-check-cfssl.png)](https://pic-cdn.ewhisper.cn/img/2021/09/24/4ad0942f624387c5d5040c8dc8312426-check-cfssl.png "CFSSL 扫描")

## 四、加分项：配置 HSTS[](https://ewhisper.cn/posts/57169/#%E5%9B%9B%E3%80%81%E5%8A%A0%E5%88%86%E9%A1%B9%EF%BC%9A%E9%85%8D%E7%BD%AE%20-HSTS)

之前提到过得到 **A** 的方法，那么 **A+** 呢? 事实证明，SSL Labs 就会给一个 **A+**，当你有了一个叫做 HSTS (Hypertext Strict Transport Security， 超文本严格传输安全) 的特性。

### 4.1 什么是 HSTS？[](https://ewhisper.cn/posts/57169/#4-1-%20%E4%BB%80%E4%B9%88%E6%98%AF%20-HSTS%EF%BC%9F)

本质上，这是一个 HTTP 头，你可以添加到你的请求，告诉浏览器总是通过 HTTPS 访问这个站点。即使他们最初是通过 HTTP 访问的，也总是重定向到 HTTPS。

然而，这实际上有一点危险，因为如果你的 SSL 配置中断或证书过期，那么访问者将无法访问该站点的纯 HTTP 版本。你还可以做一些更高级的事情。就是将你的站点添加到预加载列表中。Chrome 和火狐浏览器都有一个列表，所以如果你注册了，他们就永远不会通过 HTTP 访问你的网站。

### 4.2 为什么要这么做？[](https://ewhisper.cn/posts/57169/#4-2-%20%E4%B8%BA%E4%BB%80%E4%B9%88%E8%A6%81%E8%BF%99%E4%B9%88%E5%81%9A%EF%BC%9F)

SSL Labs 将给你一个 **A+** 如果其他一切正确的。你需要正确地设置 `includeSubdomains` 的 HSTS(这意味着它适用于所有子域名)，并且它必须有至少 6 个月的有效期，这使得它非常危险。这是因为如果你改变了你的配置，浏览器将会记住这六个月。所以你必须保持你的 HTTPS 配置正常工作。

这是一件好事，因为它可以防止任何人在中间修改它。使用 HSTS，浏览器甚至不会有机会访问你的 HTTP 端，因此人们不会在这方面干扰你的站点。所以 HSTS 是一个非常可靠的方法。

### 4.3 风险[](https://ewhisper.cn/posts/57169/#4-3-%20%E9%A3%8E%E9%99%A9)

正如我提到的，有几个风险：

-   阻止人们通过 HTTP 访问站点
-   如果 HTTPS 配置异常（比如证书过期），站点就无法访问了

### 4.4 NGINX 配置 HSTS[](https://ewhisper.cn/posts/57169/#4-4-NGINX-%20%E9%85%8D%E7%BD%AE%20-HSTS)

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br><span>8</span><br><span>9</span><br><span>10</span><br><span>11</span><br><span>12</span><br><span>13</span><br><span>14</span><br><span>15</span><br><span>16</span><br><span>17</span><br><span>18</span><br><span>19</span><br><span>20</span><br><span>21</span><br><span>22</span><br><span>23</span><br><span>24</span><br><span>25</span><br><span>26</span><br><span>27</span><br><span>28</span><br><span>29</span><br><span>30</span><br><span>31</span><br></pre></td><td><pre><code><span>server</span> {<br>    <span>listen</span> <span>443</span> ssl http2;<br>    <span>listen</span> [::]:<span>443</span> ssl http2;<p>    <span>ssl_certificate</span> /path/to/signed_cert_plus_intermediates;<br>    <span>ssl_certificate_key</span> /path/to/private_key;<br>    <span>ssl_session_timeout</span> <span>1d</span>;<br>    <span>ssl_session_cache</span> shared:MozSSL:<span>10m</span>;  <span># about 40000 sessions</span><br>    <span>ssl_session_tickets</span> <span>off</span>;</p><p>    <span># curl https://ssl-config.mozilla.org/ffdhe2048.txt &gt; /path/to/dhparam</span><br>    <span>ssl_dhparam</span> /path/to/dhparam;</p><p>    <span># intermediate configuration</span><br>    <span>ssl_protocols</span> TLSv1.<span>2</span> TLSv1.<span>3</span>;<br>    <span>ssl_ciphers</span> ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384;<br>    <span>ssl_prefer_server_ciphers</span> <span>off</span>;</p><p>    <span># HSTS (ngx_http_headers_module is required) (15768000 seconds)</span><br>    <span>add_header</span> Strict-Transport-Security <span>"max-age=15768000"</span> always;</p><p>    <span># OCSP stapling</span><br>    <span>ssl_stapling</span> <span>on</span>;<br>    <span>ssl_stapling_verify</span> <span>on</span>;</p><p>    <span># verify chain of trust of OCSP response using Root CA and Intermediate certs</span><br>    <span>ssl_trusted_certificate</span> /path/to/root_CA_cert_plus_intermediates;</p><p>    <span># replace with the IP address of your resolver</span><br>    <span>resolver</span> <span>127.0.0.1</span>;<br>}</p></code><p><i></i>NGINX</p></pre></td></tr></tbody></table>

要设置它，只需在 NGINX 的服务器配置中添加一个头文件，上面写着 `Strict-Transport-Security`，并给它一个最大时间（`max-age`）。在本例中，它被设置为 6 个月(这是预加载列表所需的最低时间)。你还可以在这里添加其他指令，如:includeSubdomains 和 preload，这意味着可以接受这个指令并将其添加到预加载列表中。这就是得到 A+ 的方法。

## 五、又一加分项：配置 OCSP 装订（OCSP Stapling）[](https://ewhisper.cn/posts/57169/#%E4%BA%94%E3%80%81%E5%8F%88%E4%B8%80%E5%8A%A0%E5%88%86%E9%A1%B9%EF%BC%9A%E9%85%8D%E7%BD%AE%20-OCSP-%20%E8%A3%85%E8%AE%A2%EF%BC%88OCSP-Stapling%EF%BC%89)

这是一些人喜欢使用的另一个附加功能，它实际上可以帮助加快连接速度。

### 5.1 什么是 OCSP Stapling[](https://ewhisper.cn/posts/57169/#5-1-%20%E4%BB%80%E4%B9%88%E6%98%AF%20-OCSP-Stapling)

正如我前面提到的，建立一个 TLS 连接需要有很多来回的。我没有提到的是，这些证书不仅可以过期失效，还可以被吊销。

因此，如果你丢失了你的私钥，出现了漏洞，或者有其他人非法拥有了你的私钥，那么你必须去你的证书颁发机构并撤销这个密钥。有几种机制可以告诉浏览器证书已被吊销; 它们都有点粗略，但最流行的是 OCSP(Online Certificate Status Protocol，在线证书状态协议)。

发生的情况是: 当浏览器收到证书时，它还必须检查它是否被吊销了。于是它联系了证书颁发机构，问「这个证书还有效吗?」他们会回答「是」或「不是」。这本身就是另一组连接，你需要查找 CA 的 DNS，你需要连接到 CA，这对你的网站来说是额外的减速。

HTTPS 不只需要三次往返，你还需要 OCSP。因此，OCSP 装订允许服务器获取证书未过期的证明。在后台，获取这个表示「是的，证书是好的」的 OCSP 响应，然后将它放入握手中。这样客户端就不需要实际接触 CA 并获取它。

### 5.2 会快多少？[](https://ewhisper.cn/posts/57169/#5-2-%20%E4%BC%9A%E5%BF%AB%E5%A4%9A%E5%B0%91%EF%BC%9F)

完整 HTTPS 网站访问过程示例如下：

1.  DNS（1334ms）
2.  TCP 握手（240ms）
3.  SSL 握手（376ms）
4.  跟踪证书链（1011ms）
5.  到 CA 的 DNS 记录（300ms）
6.  到 CA 的 TCP （407ms）
7.  到 CA 的 OCSP 第一次（598 ms）
8.  到 CA 的 TCP 第二次（317ms）
9.  到 CA 的 OCSP 第二次（444ms）
10.  完成 SSL 握手（1270ms）

配置了 OCSP Stapling，上文的 `5-9` 步可以省略，这可以节省约 30% 的访问一个 HTTPS 网站的连接时间。

### 5.3 NGINX 配置 OCSP Stapling[](https://ewhisper.cn/posts/57169/#5-3-NGINX-%20%E9%85%8D%E7%BD%AE%20-OCSP-Stapling)

见上文（4.4 NGINX 配置 HSTS)，用 NGINX 也很容易设置。有一个 OCSP Stapling 的指令。装订验证（Stapling verify）是指在装订证书之后对其进行验证。正如我前面（2.2 受信任的 CA 的选项）提到的，使用代理，你必须信任 CA。你可以从 CA 获得一个文件，并通过可信证书部分添加到该文件中。

## 总结[](https://ewhisper.cn/posts/57169/#%E6%80%BB%E7%BB%93%20-7)

以上就是配置 NGINX 和 OCSP 装订、HSTS 和 SSL 代理的方法。

正如我提到的，在 2008 年， TLS v1.2 是最新和最好的。近期，他们推出了新版本，TLS v1.3。

HTTPS 是一个不断变化的领域，我们的配置及最佳实践可能也需要进行相应调整。