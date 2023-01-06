GIF

![](https://i0.hdslb.com/bfs/article/4a10568f28aa362fc3ddc7f871b07daf847767da.gif)

帅哥（靓仔）、美女，点个关注后续不迷路！

**本章目录**

![](https://i0.hdslb.com/bfs/article/4aa545dccf7de8d4a93c2b2b8e3265ac0a26d216.png@progressive.webp)

-   0x02 HTTPS安全加固指南
    

-   1.连接安全性和加密选择
    
-   2.站点内容安全（Content security）
    
-   3.避免站点额外信息泄露
    
-   4.避免Web服务器中间件脆弱性
    

-   0x03 HTTPS安全与兼容性如何抉择
    
-   0x0n 入坑出坑
    

-   1.通过https证书安全合规检测发现缺少HTTP Strict-Transport-Security响应头
    
-   2.利用 myssl 站点的HTTPS安全报告工具检测，出现证书链不完整降级为B提示
    

![](https://i0.hdslb.com/bfs/article/4aa545dccf7de8d4a93c2b2b8e3265ac0a26d216.png@progressive.webp)

描述: 当你的网站上了 HTTPS 以后，可否觉得网站已经安全了，其实不然前面说过想要部署TLS是非常容易，只需要证书与密钥文件然后根据中间件的配置ssl方法进行配置即可，默认的配置通常不满足https的安全需要，所以我们还需要进一步着重在于 HTTPS 网站的 Header 的相关配置。

**安全加固实践**

-   1.1) 建议部署配置 (HSTS) HTTP Strict Transport Security (`HTTP 严格传输安全HSTS`)响应头，它是TLS的安全网旨在确保即使在配置问题和实施错误的情况下安全性仍然保持不变,如果要使用HSTS我们只需要在网站添加一个新的响应头`Strict-Transport-Security`。再设置该响应头之后将会告诉浏览器只使用 HTTPS 连接到目标服务器，并且还防止一些潜在的中间人攻击，包括 SSL 剥离、会话 cookie 窃取。
    

```
# 语法演示:
Strict-Transport-Security: max-age=<expire-time>Strict-Transport-Security: max-age=<expire-time>; includeSubDomains
Strict-Transport-Security: max-age=<expire-time>; preload

# 配置示例: 浏览器防止指定站点以及将来的子域名都采用 HTTPS，最大年龄为1年(31536000)，同时还允许预加载（加快速度）：
Strict-Transport-Security: max-age=31536000; includeSubDomains; preload

# Nginx
# HSTS (ngx_http_headers_module is required) (63072000 seconds == 两年)
add_header Strict-Transport-Security "max-age=63072000; includeSubDomains; preload" always;
```

-   2.2) 建议部署配置HPKP(HTTP Public Key Pins)响应头, 指示浏览器只与提供的 SSL/TLS 的 HASH 相符或存在于同一证书链的服务器相连接。简单的说如果 SSL/TLS 证书以一种意想不到的方式发生了变化，浏览器就无法连接到主机，这主要是针对受信任证书颁发机构（CA）或流氓 CA 证书颁发的伪造证书，用户可能会被骗安装。
    

> 例如，浏览器连接到 https://weiyigeek.top，它存在这个头。header 告诉浏览器，如果证书 key 匹配，或者在发出证书链中有一个 key 匹配，那么在将来才会再次连接。其他的指令组合是可能的。它们都极大地减少了攻击者在客户端和合法主机之间模拟主机或拦截通信的可能性。

```
Public-Key-Pins: max-age=5184000; pin-sha256="+oZq/vo3KCV0CQPjpdwyInqVXmLiobmUJ3FaDpD/U6c="; pin-sha256="47DEQpj8HBSa+/TImW+5JCeuQeRkm5NMpJWZG3hSuFU="
```

温馨提示: 设置 HSTS header `Strict-Transport-Security` 长的生命周期，即 max-age 的时间不能小于15552000，最好是半年及以上。  
温馨提示: 确定是否需要为您的站点使用 PKP, 如果要是使用它请注意在SSL/TLS密钥需要更新时建立备份计划, 优先创建备份密钥和离线存储。

**安全加固实践**

-   2.1) 避免混合 HTTPS 和 HTTP 内容(`Mixed HTTPS and HTTP Content`), 如果 HTTPS 部署在主站上，请将任何地方的所有内容都 HTTPS 化（全站 HTTPS）。
    
-   2.2) 建议配置良好的内容安全策略`(CSP - Content Security Policy)`响应头, 它可以帮助抵御跨站点脚本（XSS）和其他注入攻击等攻击，现在主流的浏览器都是支持CSP头的。
    

> 一个理想的CSP策略是基于白名单的方法，不允许任何东西，除了明确允许的内容，它还限制了 javascript 的来源和允许操作。

```
# 从限制性策略开始在必要时放松。
Content-Security-Policy: default-src 'none';

# 现在让我们允许自托管 scripts、images、CSS、fonts 和 AJAX，以及 jQuery CDN 托管脚本和 Google Analytics:
Content-Security-Policy: default-src 'none'; script-src 'self' https://code.jquery.com https://www.google-analytics.com; img-src 'self' https://www.google-analytics.com; connect-src 'self'; font-src 'self'; style-src 'self';

# 一个不那么严格的策略
Content-Security-Policy: default-src 'self';

# 更少的限制性策略,然后添加限制，非常不建议如此使用。
Content-Security-Policy: default-src '*';
```

-   2.3) 建议配置`X-Frame-Options`非标准的响应头, 它规定了控制站点是否可以放置在 `<iframe>，<frame> 或 <object>` 标签, 从而防止了clickjacking (点击劫持)攻击。所以启用它时请确定你的网站是否需要被允许呈现在一个 frame 中。
    

```
# HTTP header: 完全不允许使用 sameorigin 拒绝或允许同源框架的选项
X-Frame-Options: deny

# 该响应头可选值
DENY：不能被嵌入到任何iframe或者frame中。
SAMEORIGIN：页面只能被本站页面嵌入到iframe或者frame中
ALLOW-FROM uri：只能被嵌入到指定域名的框架中

# Nginx, 允许 music.163.com 放置在站点之中
add_header X-Frame-Options ALLOW-FROM music.163.com;
```

-   2.4) 建议配置`XSS Protection`(XSS过滤器) 可选, 跨站点脚本（XSS 或 CSS）的保护被构建到大多数流行的浏览器中，例如Chorme、IE等，但除了 Firefox 浏览器之外。
    

```
# HTTP header:
X-Xss-Protection: 1; block

# Nginx:
add_header X-XSS-Protection "1; mode=block";
```

-   2.5) 建议配置`Cache Control`, 其表示缓存页面输出的首选项，适当的值随网站数据的性质而变化，但强烈推荐使用偏好, 如果允许缓存则应该将 max-age 值包含在 Cache-Control 以及 Etag 头文件中，以允许客户端缓存验证。
    

```
# HTTP header:
Cache-Control: public*

# 可选参数:
其中的一个 public，private，no-cache 或 no-store。
```

-   2.6) 建议配置 `Content Type Options`，当浏览器以不同的方式处理来自服务器的文件时，MIME 嗅探就是服务器指令，例如当一个网站承载不受信任的内容（如用户提供的）时这是非常危险的。所以我们可以采用非标准的标头 `X-Content-Type-Options` 选项指示浏览器不做任何模仿指定类型的 MIME。
    

```
# HTTP header:
X-Content-Type-Options: nosniff

# Nginx
add_header X-Content-Type-Options nosniff;
```

-   2.7) 建议验证子资源完整性(`Subresource Integrity`), 浏览器通常从外部域加载大量资源、javascript 和样式表, 例如第三方的js或者css，如果如果外部资源被破坏修改可能导致所以调用的站点存在一系列的脆弱性，所以我们设置外部 javascript 和样式表的完整性属性或者将外部脚本保存到本地路径中进行相对路径调用。
    

```
# 完整性校验,该文件的SHA384值
<script src="https://ajax.googleapis.com/ajax/libs/jquery/1.11.3/jquery.min.js" integrity="sha384-6ePHh72Rl3hKio4HiJ841psfsRJveeS+aLoaEf3BWfS+gTF0XdAqku2ka8VddikM"></script>

# 或者本地相对调用
<script>window.jQuery || document.write('<script src="/jquery.min.js"><\/script>')</script>
```

-   2.8) 建议配置 Iframe Sandbox，如果你的站点存在iframe标签的使用，建议配置 sandbox 属性允许对 iframe 中可以进行的操作进行限制。
    

```
#  设置 iframe 的 sandbox 属性，然后添加所需的权限。
<iframe src="https://weiyigeek.top" sandbox="allow-same-origin allow-scripts"></script>
```

-   2.8) 建议配置 TLS服务器时间同步,我们可使用网络时间协议（NTP）来保持服务器时钟的准确性，这是由于服务器包括所有响应的时间戳，不准确的时钟不会给客户机浏览器带来问题。
    

> ntpdate ntp.aliyun.com  
> 10 Apr 07:39:18 ntpdate\[41371\]: adjust time server 203.107.6.88 offset 0.000460 sec

-   2.9) 建议配置 Cookie Security, 如果你的站点包含敏感信息的 cookie，特别是会话 id，需要标记为安全的，建议会话 cookie 应该与 HttpOnly 值进行标记 和 正确配置 HSTS 响应头。以防止它们被 javascript 访问，这可以防止攻击者利用 XSS 窃取会话 cookie。
    

```
#  标记所有 cookie 安全为HttpOnly
Set-Cookie: Key=Value; path=/; secure; HttpOnly, Key2=Value2; secure; HttpOnly

# 例如, Tomcat7 中进行 HttpOnly 与 secure 配置
# tomcat/conf/server.xml
<Connector port="8080" protocol="HTTP/1.1" connectionTimeout="20000" redirectPort="8443" secure="true" />
# tomcat/conf/web.xml
<session-config>
  <session-timeout>30</session-timeout>
  <cookie-config>
    <http-only>true</http-only>
  </cookie-config>
</session-config>
```

-   2.10) 建议配置`X-Robots-Tag`，使用Meta Robots标记，可以真正阻止搜索引擎显示想保留在搜索结果之外的页面, 使用`X-Robots-Tag HTTP`标头可以达到相同的结果,如前所述，X-Robots-Tag还允许控制如何索引特定文件（类型），从而提供了更大的灵活性。
    
  

```
# 防止搜索引擎显示使用PHP生成的文件.
add_header X-Robots-Tag noindex;

# 假如一个提供.doc文件的网站，但由于特定的原因，不希望搜索引擎为该文件类型建立索引.
location ~* \.(doc|docx|pdf)$ {
  add_header X-Robots-Tag noindex, noarchive, nosnippet;
}
```

温馨提示: 在Nginx中配置缓存策略非常简单, 我们只需要在指定的location规则下,加入类似于`expires`关键参数即可;

**安全加固实践**

-   3.1) 建议将Web服务器的版本号去掉, 例如在 Nginx 中我们可以在 http{} 块中配置加入`server_tokens off;`, 便可以只包含服务器名称但去掉版本号`Server: nginx`；
    
-   3.2) 建议取消掉许多 web 框架设置 HTTP 头 以及 识别框架或版本号，虽然可能对我们访问者无实际用途但对于搜索引擎或者其它Hacker攻击者来说这可是个好东西。所以建议从服务器响应中删除这些标头: `X-Powered-By, X-Runtime, X-Version 和 X-AspNet-Version`。
    

**安全加固实践**

-   4.1) 推荐使用最新稳定的OpenSSL版本, 但是需要注意在升级前请做好回滚措施或者测试，当前2022年4月10日 08:09:17最新的版本为`openssl-1.1.1n`。
    

```
# 编译安装
# https://www.openssl.org/source/
wget -c http://www.openssl.org/source/openssl-1.1.1n.tar.gz
tar -zxf openssl-1.1.1n.tar.gz && cd openssl-1.1.1n
./config --prefix=/usr/local/openssl
make && sudo make install
# lib 库加载到系统
echo "/usr/local/openssl/lib" >> /etc/ld.so.conf.d/libc.conf
ldconfig
# 可以看到当下系统的Openssl版本已经更新到最新
# root@weiyigeek-top:/usr/local/openssl/bin# openssl version
# OpenSSL 1.1.1n  15 Mar 2022
```

-   4.2) 推荐使用最新稳定的Web服务器软件及其版本, 例如 Nginx、Apache、Tomcat等中间件，请选择当前无脆弱性的版本。
    

```
# Nginx 中间件版本查看
$ nginx -V
nginx version: nginx/1.21.6
built by gcc 9.4.0 (Ubuntu 9.4.0-1ubuntu1~20.04)
built with OpenSSL 1.1.1n  15 Mar 2022
TLS SNI support enabled
  configure arguments: --prefix=/usr/local/nginx --with-pcre=../pcre-8.45 --with-zlib=../zlib-1.2.11 --user=ubuntu --group=ubuntu --sbin-path=/usr/sbin/nginx --conf-path=/usr/local/nginx/nginx.conf --pid-path=/usr/local/nginx/nginx.pid --error-log-path=/var/log/nginx/error.log --http-log-path=/var/log/nginx/access.log --lock-path=/var/run/nginx.lock --modules-path=/usr/lib/nginx/modules --http-client-body-temp-path=/var/cache/nginx/client_temp --http-proxy-temp-path=/var/cache/nginx/proxy_temp --http-fastcgi-temp-path=/var/cache/nginx/fastcgi_temp --http-uwsgi-temp-path=/var/cache/nginx/uwsgi_temp --http-scgi-temp-path=/var/cache/nginx/scgi_temp --with-stream --with-stream_realip_module --with-stream_ssl_module --with-stream_ssl_preread_module --with-stream_geoip_module --with-threads --with-mail --with-mail_ssl_module --with-http_addition_module --with-http_auth_request_module --with-http_dav_module --with-http_flv_module --with-http_gunzip_module --with-http_gzip_static_module --with-http_mp4_module --with-http_random_index_module --with-http_realip_module --with-http_secure_link_module --with-http_slice_module --with-http_ssl_module --with-http_stub_status_module --with-http_sub_module --with-http_v2_module --with-compat --with-file-aio --with-cc-opt='-Os -fomit-frame-pointer -g' --with-ld-opt=-Wl,--as-needed,-O1,--sort-common
```

描述: 很多网站HTTPS检测评分都达到了A或者A+, 但在看检测结果的时候发现类似于百度和淘宝这类大用户群的网站居然没有评级到A或者在使用的加密套件上有橙色的加密套件, 这是由于不同的业务侧重点不同，针对于浏览业务不涉及重要敏感信息的系统往往会牺牲一点安全性获取极致的兼容性。

![HTTPS安全与兼容性](https://i0.hdslb.com/bfs/article/002b16dc861c3a62084f4dc799bf710809b3a34e.jpg@942w_572h_progressive.webp)

温馨提示: 如果要适配IE6这款浏览器，那么SSL协议就必须得支持SSL2和SSL3(会受到POODLE漏洞问题影响), 所以除去CBC加密套件那就只剩下RC4系列的加密套件,实际上RC4也是不安全只是为了兼容某些老版浏览器的妥协。  
温馨提示: 如果要放弃IE6并适配兼容XP上的IE8这款浏览器，唯一比较安全的就是3DES系列的加密套件，这差不多少是一种较好的方式。

如果你的服务器需要支持像IE6这种古董级别的浏览器可以采用方式1, 而如果想在保证相对安全性的同时也具有一定的兼容性可以采用方式2, 如果业务比较敏感为保证其安全性，忽略掉其兼容性，建议采用方式3（随着openssl版本迭代进行最好的安全性进行配置）.

**ssl\_ciphers && ssl\_protocols 在Nginx上配置示例**  
方式1.更好的兼容性(支持IE6老古董)

```
ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH;
ssl_protocols SSLv3 TLSv1 TLSv1.1 TLSv1.2 TLSv1.3;
```

方式2.具有相对安全性和一定的兼容性(支持IE8~11)

```
# Nginx
ssl_ciphers EECDH+CHACHA20:EECDH+CHACHA20-draft:EECDH+AES128:RSA+AES128:EECDH+AES256:RSA+AES256:EECDH+3DES:RSA+3DES:!MD5;
ssl_protocols TLSv1 TLSv1.1 TLSv1.2 TLSv1.3;

# old configuration
ssl_protocols TLSv1 TLSv1.1 TLSv1.2 TLSv1.3;
ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384:DHE-RSA-CHACHA20-POLY1305:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA256:ECDHE-ECDSA-AES128-SHA:ECDHE-RSA-AES128-SHA:ECDHE-ECDSA-AES256-SHA384:ECDHE-RSA-AES256-SHA384:ECDHE-ECDSA-AES256-SHA:ECDHE-RSA-AES256-SHA:DHE-RSA-AES128-SHA256:DHE-RSA-AES256-SHA256:AES128-GCM-SHA256:AES256-GCM-SHA384:AES128-SHA256:AES256-SHA256:AES128-SHA:AES256-SHA:DES-CBC3-SHA;

ssl_prefer_server_ciphers on;
```

方式3.建议根据系统中openssl版本中加密套件进行安全性配置，牺牲一定的兼容性。

```
ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDH:AES:EECDH+CHACHA20:EECDH+CHACHA20-draft:EECDH+AES128:RSA+AES128:EECDH+AES256:RSA+AES256:HIGH:!NULL:!aNULL:!eNULL:!EXPORT:!PSK:!ADH:!DES:!3DES:!MD5:!RC4;
ssl_protocols TLSv1 TLSv1.1 TLSv1.2 TLSv1.3;
```

**博主当前配置**

```
# Nginx
ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:ECDHE:ECDH:AES:HIGH:EECDH+CHACHA20:EECDH+CHACHA20-draft:EECDH+AES128:RSA+AES128:EECDH+AES256:RSA+AES256:EECDH+3DES:RSA+3DES:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384:!NULL:!aNULL:!eNULL:!EXPORT:!PSK:!ADH:!DH:!DES:!MD5:!RC4;

# 以下密码需要明确禁止使用
!aNULL，不包含身份验证的DH密码，可能被中间人攻击
!eNULL，不采取密钥交换，也就是明文传输
!EXPORT，美国当年限制加密产品出口遗留的弱密码
!MD5,!RC4,!DES，这些密码已被证明攻破难度低
```

温馨提示: 我们可以访问mozilla公司提供的 SSL Configuration Generator 参考进行相应的配置参考(https://ssl-config.mozilla.org/),例如我们生成具有多种客户端的通用服务器Nginx SSL配置。

```
# generated 2022-04-10, Mozilla Guideline v5.6, nginx 1.26.1, OpenSSL 1.1.1n, intermediate configuration
# https://ssl-config.mozilla.org/#server=nginx&version=1.26.1&config=intermediate&openssl=1.1.1n&guideline=5.6
server {
  listen 80 default_server;
  listen [::]:80 default_server;

  location / {
      return 301 https://$host$request_uri;
  }
}

server {
  listen 443 ssl http2;
  listen [::]:443 ssl http2;

  ssl_certificate /path/to/signed_cert_plus_intermediates;
  ssl_certificate_key /path/to/private_key;
  ssl_session_timeout 1d;
  ssl_session_cache shared:MozSSL:10m;  # about 40000 sessions
  ssl_session_tickets off;

  # curl https://ssl-config.mozilla.org/ffdhe2048.txt > /path/to/dhparam
  ssl_dhparam /path/to/dhparam;

  # intermediate configuration
  ssl_protocols TLSv1.2 TLSv1.3;
  ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384;
  ssl_prefer_server_ciphers off;

  # HSTS (ngx_http_headers_module is required) (63072000 seconds)
  add_header Strict-Transport-Security "max-age=63072000" always;

  # OCSP stapling
  ssl_stapling on;
  ssl_stapling_verify on;

  # verify chain of trust of OCSP response using Root CA and Intermediate certs
  ssl_trusted_certificate /path/to/root_CA_cert_plus_intermediates;

  # replace with the IP address of your resolver
  resolver 127.0.0.1;
}
```

![](https://i0.hdslb.com/bfs/article/648c4ffbad9f4c34cef1e94e913b810edccac9a0.png@942w_651h_progressive.webp)

温馨提示: 我们可以采用`openssl ciphers`命令查看套件支持的协议。

```
openssl ciphers -V 'ECDHE-RSA-AES128-GCM-SHA256:ECDH:AES:EECDH+CHACHA20:EECDH+CHACHA20-draft:EECDH+AES128:RSA+AES128:EECDH+AES256:RSA+AES256:EECDH+3DES:RSA+3DES:CAMELLIA:HIGH:!NULL:!aNULL:!eNULL:!EXPORT:!PSK:!ADH:!DES:!MD5:!RC4' | column -t
```

**解决办法:** 例如，此处在 Nginx 服务器配置 server 段中添加如下内容 `add_header Strict-Transport-Security "max-age=15768000;includeSubDomains;preload";` 以便于让浏览器响应并支持HSTS。

**问题描述:**

-   浏览器的处理流程: 现代的浏览器都有证书自动下载的功能，但很多浏览器在安装后是使用系统内置的证书库，如果你缺失的那张CA证书 (信任的)，在系统的内置证书库中不存在的话，用户第一次访问网站时会显示如下情况:（以Chrome为例）, 即使你的证书确实是可信的，但依旧会显示成不可信，只有等到浏览器自动把缺失的那张CA证书从网上下载下来之后，访问该网站才会显示成可信状态。
    
-   硬件设备的处理流程: 大量的硬件设备并不会像浏览器一样下载CA证书，如果你缺失的CA证书不再这些硬件设备的内置证书库中，那么使用这些硬件设备访问网站就会一直显示你的域名是不可信状态。
    

**缺少证书链的问题和解决办法:** 修复的办法很简单就是在部署证书的时候，把那张缺失的CA证书一并部署。目前一般的证书签发机构在签发证书的时候会把该CA证书一并打包，如果你没有也不同慌，我们可以使用MySSL提供的工具(https://myssl.com/chain\_download.html)补全证书链。

```
# 下载证书链
curl https://myssl.com/api/v1/get_chain/5FFAFB3E5757A13292C4E3C0A045257E915D27DA > full_chain_ecc.crt

# 在 Nginx 中配置证书链
ssl_certificate  /app/ssl/weiyigeek.top_ecc/full_chain_ecc.crt;
ssl_certificate_key  /app/ssl/weiyigeek.top_ecc/weiyigeek.top.key;
```

本文至此完毕，更多技术文章，尽情期待下一章节！

![](https://i0.hdslb.com/bfs/article/4adb9255ada5b97061e610b682b8636764fe50ed.png@progressive.webp)

**已发布的相关历史文章（点击即可进入）**

[我在B站学运维之使用Let'sEncrypt免费颁发及自动续签泛域名(通配符)证书](https://www.bilibili.com/read/cv16065821) 

[我在B站学运维之HTTPS原理介绍以及证书签名的申请配置](https://www.bilibili.com/read/cv16065821)

[我在B站学运维之SSL与TLS协议原理与证书签名多种生成方式实践指南](https://bilibili.com/read/cv16067300)

[我在B站学运维之如何让HTTPS站点评级达到A+? 还得看这篇HTTPS安全优化配置最佳实践](https://www.bilibili.com/read/cv16067510)

![](https://i0.hdslb.com/bfs/article/4adb9255ada5b97061e610b682b8636764fe50ed.png@progressive.webp)

> 欢迎各位志同道合的朋友一起学习交流，如文章有误请在下方留下您宝贵的经验知识，个人邮箱地址【master#weiyigeek.top】或者 个人公众号【WeiyiGeek】联系我。

![](https://i0.hdslb.com/bfs/article/02db465212d3c374a43c60fa2625cc1caeaab796.png@progressive.webp)

更多文章来源于【WeiyiGeek Blog - 为了能到远方，脚下的每一步都不能少】 个人博客。

**个人主页:** 【 https://weiyigeek.top 】  

**博客地址:** 【 https://blog.weiyigeek.top 】

![](https://i0.hdslb.com/bfs/article/813dfe5de3d7e2dc0ff2ce8c38ec097255fe26b1.png@942w_480h_progressive.webp)

https://weiyigeek.top - Always keep a beginner's mind, don't forget the beginner's mind

专栏书写不易，如果您觉得这个专栏还不错的，请给这篇专栏【点个赞、投个币、收个藏、关个注，转个发】，这将对我的肯定，谢谢！。

-   echo  "【点个赞】，动动你那粗壮的拇指或者芊芊玉手，亲！"
    
-   printf("%s", "【投个币】，万水千山总是情，投个硬币行不行，亲！")
    
-   fmt.Printf("【收个藏】，阅后即焚不吃灰，亲！")
    
-   System.out.println("【关个注】，后续浏览查看不迷路哟，亲！")
    
-   console.info("【转个发】，让更多的志同道合的朋友一起学习交流，亲！")
    

GIF

![](https://i0.hdslb.com/bfs/article/11a629d1bc4369dc810216c5dedac871136167d7.gif@1s.webp)

谢谢，各位帅哥、美女四连支持！！这就是我的动力！

↓↓↓ 更多文章，请访问【个人博客站点- https://blog.weiyigeek.top】阅读原文。