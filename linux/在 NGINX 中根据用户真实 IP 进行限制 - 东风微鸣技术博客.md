## 需求[](https://ewhisper.cn/posts/36827/#%E9%9C%80%E6%B1%82%20-4)

**需要根据用户的真实 IP 进行限制, 但是 NGINX 前边还有个 F5, 导致 `deny` 指令不生效.**

阻止用户的真实 IP **不是** `192.168.14.*` 和 `192.168.15.*` 的访问请求.

## 实现[](https://ewhisper.cn/posts/36827/#%E5%AE%9E%E7%8E%B0%20-2)

最简单的实现如下:

📓[](https://github.githubassets.com/images/icons/emoji/unicode/1f4d3.png?v8) 前置条件:

需要 nginx 前边的 load balancer 设备 (如 F5) 开启 `X-Forwarded-For` 支持.

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br></pre></td><td><pre><code><span>proxy_set_header</span>   X-Forwarded-For  $proxy_add_x_forwarded_for;<p><span>if</span> ($proxy_add_x_forwarded_for !<span>~ "192\.168\.1[45]")</span>  {<br>    <span>return</span> <span>403</span>;<br>}      </p></code><p><i></i>NGINX</p></pre></td></tr></tbody></table>

说明如下:

-   `proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;` 获取请求头 `X-Forwarded-For` 中的用户真实 IP, 并附加到 `$proxy_add_x_forwarded_for` 变量
-   `if...`
    -   `(...)` 变量 `$proxy_add_x_forwarded_for` 不匹配正则 `192\.168\.1[45]` (即 `192.168.14.*` 和 `192.168.15.*`)
    -   `return 403`, 如果上边的条件满足, 返回 403
    -   即: 如果真实 IP 不是 `192.168.14.*` 和 `192.168.15.*`, 返回 403.

如果有更复杂的需求, 可以参考这个示例:

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br><span>8</span><br><span>9</span><br><span>10</span><br><span>11</span><br><span>12</span><br><span>13</span><br><span>14</span><br><span>15</span><br><span>16</span><br></pre></td><td><pre><code><span>proxy_set_header</span> HOST $http_host;<br><span>proxy_set_header</span>   X-Forwarded-For  $proxy_add_x_forwarded_for;<p><span>if</span> ($http_host <span>~ "yourdomain.hypernode.io:8443")</span>  {<br>  <span>set</span> $block_me_now A;<br>}</p><p> <span>if</span> ($proxy_add_x_forwarded_for != YOURIP) {<br>  <span>set</span> $block_me_now <span>"<span>${block_me_now}</span>B"</span>;<br>}</p><p>  <span>if</span> ($block_me_now = AB) {<br>    <span>return</span> <span>403</span>;<br>    break;<br>}</p></code><p><i></i>NGINX</p></pre></td></tr></tbody></table>

## 为啥 `deny` 配置不起作用?[](https://ewhisper.cn/posts/36827/#%E4%B8%BA%E5%95%A5%20-deny-%20%E9%85%8D%E7%BD%AE%E4%B8%8D%E8%B5%B7%E4%BD%9C%E7%94%A8)

🤔[](https://github.githubassets.com/images/icons/emoji/unicode/1f914.png?v8) 疑问: 为啥以下的配置不起作用?

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br></pre></td><td><pre><code><span>allow</span> <span>192.168.14.0</span>/<span>24</span>;<br><span>allow</span> <span>192.168.15.0</span>/<span>24</span>;<br><span>deny</span> all;<br></code><p><i></i>NGINX</p></pre></td></tr></tbody></table>

根据 nginx 官方文档, `deny` 指令是根据「client address」进行限制的.

📓[](https://github.githubassets.com/images/icons/emoji/unicode/1f4d3.png?v8) 引用:

The `ngx_http_access_module` module allows limiting access to certain **client addresses**.

而「client address」对应的变量是: `$remote_addr`

📓[](https://github.githubassets.com/images/icons/emoji/unicode/1f4d3.png?v8) 引用:

`$remote_addr`:  
 client address

**关于 `$remote_addr`**:

是 nginx 与客户端进行 TCP 连接过程中，获得的客户端真实地址. Remote Address 无法伪造，因为建立 TCP 连接需要三次握手，如果伪造了源 IP，无法建立 TCP 连接，更不会有后面的 HTTP 请求

`remote_addr` 代表客户端的 IP，但它的值不是由客户端提供的，而是服务端根据客户端的 ip 指定的，当你的浏览器访问某个网站时，假设中间没有任何代理，那么网站的 web 服务器（Nginx，Apache 等）就会把 `remote_addr` 设为你的机器 IP，如果你用了某个代理(其实 F5 就是这个反向代理)，那么你的浏览器会先访问这个代理，然后再由这个代理转发到网站，这样 web 服务器就会把 `remote_addr` 设为这台代理机器的 IP。

但是实际某些特殊场景中，我们即使有代理，也需要将 `$remote_addr` 设置为真实的用户 IP，以便记录在日志当中，当然 nginx 是有这个功能，但是需要编译的时候添加 `--with-http_realip_module` 这个模块，默认是没有安装的。(我也没有安装)