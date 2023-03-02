我们的系统依赖一个第三方的服务，该服务是通过IP限制访问权限的。出于安全考虑，我们的系统会校验证书，因此我们采用Nginx反向代理去访问该服务。该服务迁移到cloud上之后，我们系统出现了问题。

Nginx的配置文件如下所示：

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br><span>8</span><br><span>9</span><br><span>10</span><br><span>11</span><br><span>12</span><br><span>13</span><br><span>14</span><br><span>15</span><br><span>16</span><br><span>17</span><br><span>18</span><br><span>19</span><br><span>20</span><br><span>21</span><br><span>22</span><br></pre></td><td><pre><span>http {</span><br><span>    upstream backend.example.com {</span><br><span>        server backend1.example.com:443;</span><br><span>   }</span><br><span></span><br><span>    server {</span><br><span>        listen      80;</span><br><span>        server_name www.example.com;</span><br><span></span><br><span>        location /upstream {</span><br><span>            proxy_pass                    https://backend.example.com;</span><br><span>            proxy_ssl_certificate         /etc/nginx/client.pem;</span><br><span>            proxy_ssl_certificate_key     /etc/nginx/client.key;</span><br><span>            proxy_ssl_protocols           TLSv1 TLSv1.1 TLSv1.2;</span><br><span>            proxy_ssl_ciphers             HIGH:!aNULL:!MD5;</span><br><span>            proxy_ssl_trusted_certificate /etc/nginx/trusted_ca_cert.crt;</span><br><span></span><br><span>            proxy_ssl_verify        on;</span><br><span>            proxy_ssl_verify_depth  2;</span><br><span>        }</span><br><span>    }</span><br><span>}</span><br></pre></td></tr></tbody></table>

## [](https://www.realks.com/2021/03/07/nginx-proxy-ssl-server-name/#Debug "Debug")Debug

在出现线上问题的时候，第一时间检查了配置文件的改动记录 ⇒ 配置文件没有改到。

接着检查了，证书是否过期⇒没有过期。

通过域名curl请求 ⇒ 第三方服务正常，证书正确。

在我们没有任何改动的情况下，第三方服务也是单纯地做了迁移，然后就挂了。

思考了很久之后，Nginx没有任何改动，那么出问题的一定不是我们。那么出问题一定是第三方，最直接的方式是联系第三方提供商，联系渠道很繁琐，时间成本也会很高。为了快速修复问题，我们决定盲调Bug。

## [](https://www.realks.com/2021/03/07/nginx-proxy-ssl-server-name/#%E5%B0%9D%E8%AF%95%E8%A7%A3%E5%86%B3 "尝试解决")尝试解决

第一次尝试，已知curl请求一切正常。尝试着将upstream干掉，直接写道proxy\_path中。依旧是失败，错误日志中出现了”[https://1.1.1.1:443](https://1.1.1.1/) TLS handshake failed”。推断，proxy\_path会将域名转换成ip。

第二次尝试，尝试检查curl ip来访问服务。失败。推断，IP和域名指向了两个不同的服务。到这里算是找到root cause了。

解决思路：反向代理时候，通过域名访问而不是IP访问。

查询文档，发现可以使用配置项：`proxy_ssl_server_name`，该配置项默认值是off，需要将一些内容写到配置文件中：

<table><tbody><tr><td><pre><span>1</span><br></pre></td><td><pre><span>proxy_ssl_server_name on;</span><br></pre></td></tr></tbody></table>

第三次尝试，将`proxy_ssl_server_name on`写入到配置文件中。成功。

服务器名称指示（Server Name Indication, SNI）是对TLS协议的扩展。在握手过程开始时，通过该协议，客户端指示其尝试连接的主机名。该协议允许服务器上的同一个IP和TCP端口拥有多个证书，因此允许同一个IP地址为多个HTTPS网站提供服务，且无需所有网站使用相同的证书。