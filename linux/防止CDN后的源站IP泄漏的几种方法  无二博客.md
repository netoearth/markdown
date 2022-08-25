相信使用 Cloudflare 建站的同学，其目的都是为了隐藏服务器 IP，但是有些无良爬虫可通过 HTTP/HTTPS 访问扫描全网 IP 绕过 CDN，暴露证书、同时暴露你的域名。

就比如下面 Nginx 常规 HTTPS 跳转配置就会轻易暴露你的 IP：

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br><span>8</span><br><span>9</span><br></pre></td><td><pre><span><span>server</span> {</span><br><span>    <span>listen</span> <span>80</span>;</span><br><span>    <span>listen</span> [::]:<span>80</span>;</span><br><span>    <span>server_name</span> example.com;</span><br><span></span><br><span>    <span>location</span> / {</span><br><span>        <span>return</span> <span>301</span> https://$server_name$request_uri;</span><br><span>    }</span><br><span>}</span><br></pre></td></tr></tbody></table>

如果你 Nginx 上有这种配置，运行下面命令就可以获取你的域名了（假设你的IP是 1.2.3.4）

`curl -v -k http://1.2.3.4`

或者直接浏览器输入 [http://1.2.3.4](http://1.2.3.4/) 也会跳转到你的域名。解决办法是，直接删除上面 80 端口配置，只保留 443 端口配置。也就是将 HTTPS 作为唯一的回源协议。

Nginx 上开启的端口越少，暴露的风险也就越低，这是显而易见的，就比如 Cloudflare 建站的用户，只需要开启 443 端口就够了。这里又有人问了，443 端口也可以用类似命令扫描呀！对，下面一条命令不止暴露域名，连证书都可以暴露：

`curl -v -k https://1.2.3.4`

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br><span>8</span><br><span>9</span><br><span>10</span><br><span>11</span><br><span>12</span><br><span>13</span><br><span>14</span><br><span>15</span><br><span>16</span><br><span>17</span><br><span>18</span><br><span>19</span><br><span>20</span><br><span>21</span><br><span>22</span><br><span>23</span><br><span>24</span><br><span>25</span><br><span>26</span><br><span>27</span><br><span>28</span><br><span>29</span><br><span>30</span><br><span>31</span><br><span>32</span><br><span>33</span><br><span>34</span><br><span>35</span><br></pre></td><td><pre><span><span>curl</span> -v -k https://<span>1.2</span><span>.3</span><span>.4</span></span><br><span>* <span>Rebuilt</span> <span>URL</span> to: https://<span>1.2</span><span>.3</span><span>.4</span>/</span><br><span>*   <span>Trying</span> <span>1.2</span><span>.3</span><span>.4</span>...</span><br><span>* <span>TCP_NODELAY</span> set</span><br><span>* <span>Connected</span> to <span>1.2</span><span>.3</span><span>.4</span> (<span>1.2</span><span>.3</span><span>.4</span>) <span>port</span> 443 (#0)</span><br><span>* ALPN, offering h2</span><br><span>* ALPN, offering http/1.1</span><br><span>* successfully set certificate verify locations:</span><br><span>*   CAfile: /etc/ssl/certs/ca-certificates.crt</span><br><span>  CApath: /etc/ssl/certs</span><br><span>* TLSv1.3 (<span>OUT</span>), TLS handshake, Client hello (1):</span><br><span>* TLSv1.3 (<span>IN</span>), TLS handshake, Server hello (2):</span><br><span>* TLSv1.2 (<span>IN</span>), TLS handshake, Certificate (11):</span><br><span>* TLSv1.2 (<span>IN</span>), TLS handshake, Server key exchange (12):</span><br><span>* TLSv1.2 (<span>IN</span>), TLS handshake, Server finished (14):</span><br><span>* TLSv1.2 (<span>OUT</span>), TLS handshake, Client key exchange (16):</span><br><span>* TLSv1.2 (<span>OUT</span>), TLS change cipher, Client hello (1):</span><br><span>* TLSv1.2 (<span>OUT</span>), TLS handshake, Finished (20):</span><br><span>* TLSv1.2 (<span>IN</span>), TLS handshake, Finished (20):</span><br><span>* SSL connection using TLSv1.2 / ECDHE-RSA-AES256-GCM-SHA384</span><br><span>* ALPN, server accepted to use http/1.1</span><br><span>* Server certificate:</span><br><span>*  subject: CN=example.com</span><br><span>*  start date: Nov 15 05:41:39 2019 GMT</span><br><span>*  expire date: Nov 14 05:41:39 2020 GMT</span><br><span>*  issuer: CN=example.com</span><br><span>*  SSL certificate verify result: unable to get local issuer certificate (20), continuing anyway.</span><br><span>&gt; GET / HTTP/1.1</span><br><span>&gt; Host: 1.2.3.4</span><br><span>&gt; User-Agent: curl/7.58.0</span><br><span>&gt; Accept: */*</span><br><span>&gt; </span><br><span>* Empty reply from server</span><br><span>* Connection #0 to host 1.2.3.4 left intact</span><br><span>curl: (52) Empty reply from server</span><br></pre></td></tr></tbody></table>

### [](https://1kb.day/posts/nginx_cdn.html#%E5%BC%80%E5%90%AF-ssl-reject-handshake-%E6%8F%92%E4%BB%B6 "开启 ssl_reject_handshake 插件")开启 ssl\_reject\_handshake 插件

可以看见，证书和域名都暴露了。解决方法也很简单，nignx 开启 ssl\_reject\_handshake 插件就能防止 443 端口被扫，下面讲讲使用方法。

Nginx 版本高于等于 1.19.4，才可以使用 ssl\_reject\_handshake 特性来防止 SNI 信息泄露。如果 Nginx 版本太低，可以看看姥爷的这篇文章[自编译 Nginx](https://1kb.day/posts/nginx_make.html).

Nginx 里面新添加一个 server 块，即可开启 ssl\_reject\_handshake，如下：

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br><span>8</span><br><span>9</span><br><span>10</span><br><span>11</span><br><span>12</span><br><span>13</span><br><span>14</span><br><span>15</span><br><span>16</span><br></pre></td><td><pre><span></span><br><span><span>server</span> {</span><br><span>    <span>listen</span> <span>443</span> ssl default_server;</span><br><span>    </span><br><span>    <span>listen</span> [::]:<span>443</span> ssl default_server;</span><br><span>    <span>ssl_reject_handshake</span> <span>on</span>;</span><br><span>}</span><br><span></span><br><span></span><br><span><span>server</span> {</span><br><span>    <span>listen</span> <span>443</span> ssl;</span><br><span>    <span>listen</span> [::]:<span>443</span> ssl;</span><br><span>    <span>server_name</span> example.com;</span><br><span>    <span>ssl_certificate</span> example.com.crt;</span><br><span>    <span>ssl_certificate_key</span> example.com.key;</span><br><span>}</span><br></pre></td></tr></tbody></table>

注，上面插件只适用于 443 端口不适用其它端口。重启 Nginx 后，再运行上面命令，看看还会不会暴露域名。

### [](https://1kb.day/posts/nginx_cdn.html#IP-%E7%99%BD%E5%90%8D%E5%8D%95 "IP 白名单")IP 白名单

实际上还有更直接的方法，那就是配置 Cloudflare IP 白名单，服务端只允许 Cloudflare IP 的请求，其它丢弃掉。下面是利用 iptables 来设置 CF IP 白名单的方法。

有人问，为什么不用 Nginx 内置的 access module 来配置白名单？

因为使用 HTTPS 作为回源协议，爬虫依旧能通过探测证书 SNI 信息来找到你的源站服务器，所以才用 iptables 来配置白名单。

添加cloudflare ips-v4 iptables 白名单的命令：

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br></pre></td><td><pre><span><span>for</span> i <span>in</span> `curl https:<span>//</span>www.cloudflare.com/ips-v4`;</span><br><span>    <span>do</span> iptables -I INPUT -p tcp -m multiport --dports http,https -s <span>$i</span> -j ACCEPT;</span><br><span>done</span><br></pre></td></tr></tbody></table>

添加cloudflare ips-v6 iptables 白名单的命令:

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br></pre></td><td><pre><span><span>for</span> i <span>in</span> `curl https:<span>//</span>www.cloudflare.com/ips-v6`;</span><br><span>    <span>do</span> ip6tables -I INPUT -p tcp -m multiport --dports http,https -s <span>$i</span> -j ACCEPT;</span><br><span>done</span><br></pre></td></tr></tbody></table>

丢弃白名单以外的 ipv4 80,443 tcp 包:

<table><tbody><tr><td><pre><span>1</span><br></pre></td><td><pre><span>iptables -<span>A</span> <span>INPUT</span> -<span>p</span> tcp -m multiport --dports http,https -j DROP</span><br></pre></td></tr></tbody></table>

丢弃白名单以外的 ipv6 80,443 tcp 包:

<table><tbody><tr><td><pre><span>1</span><br></pre></td><td><pre><span>ip6tables -<span>A</span> <span>INPUT</span> -<span>p</span> tcp -m multiport --dports http,https -j DROP</span><br></pre></td></tr></tbody></table>

### [](https://1kb.day/posts/nginx_cdn.html#%E7%AC%AC%E4%B8%89%E7%A7%8D%E5%81%87%E8%AE%BE "第三种假设")第三种假设

其实应该还有第三种方法，俺来假设一下，现在爬虫的 HTTP 版本基本都是 1.1，指定 HTTP 版本号来屏蔽爬虫，理应能达到同样效果。

Nginx 里 server 模块中引入下面规则文件：

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br></pre></td><td><pre><span><span>#限制http版本号，只允许下面3个HTTP版本</span></span><br><span><span>if</span> ($server_protocol !~* <span>"HTTP/2.0|HTTP/3.0|SPDY/3.1"</span>) {</span><br><span>    <span>return</span> <span>403</span></span><br><span>}</span><br></pre></td></tr></tbody></table>

还可以引入以下规则，来抵御 CC 攻击，下面规则比较暴力，非 Cloudflare 用户可以酌情引用。

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br></pre></td><td><pre><span><span>#禁止非 Mozilla/ 请求头的访问</span></span><br><span><span>if</span> ($http_user_agent !~* <span>"Mozilla/"</span>) {</span><br><span>    <span>return</span> <span>403</span></span><br><span>}</span><br></pre></td></tr></tbody></table>

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br></pre></td><td><pre><span>#禁止非GET|<span>HEAD</span>|<span>POST</span>方式的抓取</span><br><span><span>if</span> ($request_method !~ ^(GET|<span>HEAD</span>|<span>POST</span>)$) {</span><br><span>    <span>return</span> <span>403</span>;</span><br><span>}</span><br></pre></td></tr></tbody></table>

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br></pre></td><td><pre><span></span><br><span><span>if</span> ($http_user_agent <span>~* (Scrapy|Curl|HttpClient))</span> {</span><br><span>    <span>return</span> <span>403</span>;</span><br><span>}</span><br></pre></td></tr></tbody></table>

### [](https://1kb.day/posts/nginx_cdn.html#%E7%BB%93%E8%AF%AD "结语")结语

对于爬虫爬取你的真实 IP，上面三种方法，可以三选一。（第三种假设没经过测试，不确保可用性）

俺以前有一点误区，以为在 Cloudflare 上配置相应规则能屏蔽这些爬虫，其实这是不对的，因为它们压根不搭理你域名，只是扫全网IP，探测证书 SNI 信息，所以在 CF 上折腾相关规则完全是无用功。