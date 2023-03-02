> 这几天把公司测试环境 Nginx 切换到了 Caddy，在实际切换过程中还是有一点小问题，但是目前感觉良好，这里记录一些细节。

## [](https://mritd.com/2021/08/20/switching-rrom-nginx-too-caddy/#%E4%B8%80%E3%80%81%E4%B8%BA%E4%BB%80%E4%B9%88%E8%A6%81%E5%88%87%E6%8D%A2 "一、为什么要切换")一、为什么要切换[](https://mritd.com/2021/08/20/switching-rrom-nginx-too-caddy/#%E4%B8%80%E3%80%81%E4%B8%BA%E4%BB%80%E4%B9%88%E8%A6%81%E5%88%87%E6%8D%A2)

大部分情况我们的生产环境使用一个域名，为了保证隔离性我们会在测试环境采用另一个域名(偷偷透露一下，测试环境买 `*.link` 域名，国内能备案还贼便宜)；然而我们不太舍得掏钱去给测试域名再买个证书，所以一直 ACME 大法。

众所周知这个玩意的证书 3 个月需要续签一次，脚本式续签然后 nginx reload 有时候还不太靠谱，总之内部环境复杂下脚本式操作还是有点风险，所以最后决定 Caddy 一把梭一劳永逸了。

## [](https://mritd.com/2021/08/20/switching-rrom-nginx-too-caddy/#%E4%BA%8C%E3%80%81%E5%88%87%E6%8D%A2%E4%B8%AD%E6%B6%89%E5%8F%8A%E5%88%B0%E7%9A%84%E7%BB%86%E8%8A%82 "二、切换中涉及到的细节")二、切换中涉及到的细节[](https://mritd.com/2021/08/20/switching-rrom-nginx-too-caddy/#%E4%BA%8C%E3%80%81%E5%88%87%E6%8D%A2%E4%B8%AD%E6%B6%89%E5%8F%8A%E5%88%B0%E7%9A%84%E7%BB%86%E8%8A%82)

### [](https://mritd.com/2021/08/20/switching-rrom-nginx-too-caddy/#2-1%E3%80%81%E8%A7%84%E5%88%99%E5%8C%B9%E9%85%8D "2.1、规则匹配")2.1、规则匹配[](https://mritd.com/2021/08/20/switching-rrom-nginx-too-caddy/#2-1%E3%80%81%E8%A7%84%E5%88%99%E5%8C%B9%E9%85%8D)

在某个站点中我们采用了 Nginx 判断 User-Agent 来处理访问到底是移动端还是桌面端，说实话我比较讨厌这种骚这种东西:

```
map $http_user_agent $is_desktop {
    default 0;
    ~*linux.*android|windows\s+(?:ce|phone) 0; # exceptions to the rule
    ~*spider|crawl|slurp|bot 1; # bots
    ~*windows|linux|os\s+x\s*[\d\._]+|solaris|bsd 1; # OSes
}

map $is_desktop $is_mobile {
    1 0;
    0 1;
}

server {
    # reverse proxy
    location / {
        if ($is_mobile) {
            rewrite ^ https://$host/h5 redirect;
            break;
        }
        proxy_pass http://backend;
        include conf.d/common/proxy.conf;
    }
}
```

一开始通过查找 Caddy 文档发现 Caddy 也是支持 map 的:

```
map {host}             {my_placeholder}  {magic_number} {
example.com        "some value"      3
foo.example.com    "another value"
(.*)\.example.com  "${1} subdomain"  5

~.*\.net$          -                 7
~.*\.xyz$          -                 15

default            "unknown domain"  42
}
```

在实际配置时发现其实这个问题只需要用自定义规则匹配器判断一下是不是移动端即可:

```
@mobile {
    header_regexp User-Agent (?i)linux.*android|windows\s+(?:ce|phone)
    not path_regexp ^.+\.(?:css|cur|js|jpe?g|gif|htc|ico|png|html|xml|otf|ttf|eot|woff|woff2|svg)$
    not path /web/*
}
rewrite @mobile /h5/{path}?{query}
```

在后续编写匹配规则时发现 Caddy 的匹配规则确实是非常强大，在官方的 [Request Matchers](https://caddyserver.com/docs/caddyfile/matchers#request-matchers) 文档页面上可以找到基本上满足所有需求的匹配器，从请求头到请求方法、协议、请求路径，从标准匹配到通配符、正则匹配基本上样样俱全，甚至支持代码式的 [CEL (Common Expression Language)](https://github.com/google/cel-spec) 表达式匹配；多个匹配还可以自定义命名作为业务相关的匹配器使用。

### [](https://mritd.com/2021/08/20/switching-rrom-nginx-too-caddy/#2-2%E3%80%81%E8%A7%84%E5%88%99%E9%87%8D%E5%86%99 "2.2、规则重写")2.2、规则重写[](https://mritd.com/2021/08/20/switching-rrom-nginx-too-caddy/#2-2%E3%80%81%E8%A7%84%E5%88%99%E9%87%8D%E5%86%99)

在 Nginx 中 rewrite 指令是多种行为的，比如可以进行 URL 隐式改写，也可以返回 301、307 等重定向代码；但是在 Caddy 中这两种行为被划分为两个指令:

-   rewrite: 内部重写，对 URL、参数等进行内部替换，浏览器地址将保持不变
-   redir: 重定向，返回 HTTP 状态码让客户端自行重定向到新页面

#### [](https://mritd.com/2021/08/20/switching-rrom-nginx-too-caddy/#2-2-1%E3%80%81rewrite "2.2.1、rewrite")2.2.1、rewrite[](https://mritd.com/2021/08/20/switching-rrom-nginx-too-caddy/#2-2-1%E3%80%81rewrite)

针对于地址的隐式重写 rewrite 指令其语法规则如下:

```
rewrite [<matcher>] <to>
```

匹配器就是全局标准的匹配器定义，可以使用内置的，也可以组合内置匹配器为自定义匹配器，这个匹配器比 Nginx 强大太多；`to` 中分为三种情况:

-   **只替换 PATH: `rewrite /abc /bcd`:**

这种情况下，rewrite 根据 “匹配器” 确定匹配路径，然后完全替换为最后一个路径；**最后面的路径可以使用 `{path}` 占位符引用原始路径。**

-   **只替换 请求参数: `rewrite /api ?a=b`:**

这种情况下，Caddy 以 `?` 作为分隔符，如果 `?` 后面有内容就意味着将请求参数替换为后面的请求参数；**最后面的请求参数可以通过 `{query}` 引用原始请求参数。**

-   **全部替换: `rewrite /abc /bcd?{query}&custom=1`:**

这种情况下，Caddy 根据 “匹配器” 匹配会即替换请求路径也替换请求参数，当然两个占位符也都是可用的。

**需要注意的是: `rewrite` 只做重写，不会中断请求链，这意味着最终返回结果根据后续的请求匹配来决定。**

#### [](https://mritd.com/2021/08/20/switching-rrom-nginx-too-caddy/#2-2-2%E3%80%81redir "2.2.2、redir")2.2.2、redir[](https://mritd.com/2021/08/20/switching-rrom-nginx-too-caddy/#2-2-2%E3%80%81redir)

redir 用于向客户端声明显式的重定向，即返回特定重定向状态码，其语法如下:

```
redir [<matcher>] <to> [<code>]
```

匹配器就不说了，全都一样；\*\*`<to>` 这个参数会作为 `Location` 头部值返回，其中可以使用占位符引用原始变量:\*\*

```
redir * https://example.com{uri}
```

`code` 部分分为四种情况:

-   一个 `3xx` 的自定义状态码
-   `temporary`: 返回 302 临时重定向
-   `permanent`: 返回 301 永久重定向
-   `html`: 使用 HTML 文档方式重定向

例如将所有请求永久重定向到新站点:

```
redir https://example.com{uri} permanent
```

这里面 HTML 方式是比较难理解的，这起源于一个规范，具体如下:

> HTTP 协议中重定向机制是应该优先采用的创建重定向映射的方式，但是有时候 Web 开发者对于服务器没有控制权，或者无法对其进行配置。针对这些特定的应用情景，Web 开发者可以在精心制作的 HTML 页面的
> 
> 部分添加一个元素，并将其 http-equiv 属性的值设置为 refresh 。当显示页面的时候，浏览器会检测该元素，然后跳转到指定的页面。

在源码中如果使用了 `html` 重定向方式，Caddy 会返回一个 HTML 页面以满足上述方式的情况下让浏览器自行刷新:

```
var body string
switch code {
case "permanent":
code = "301"
case "temporary", "":
code = "302"
case "html":
// Script tag comes first since that will better imitate a redirect in the browser's
// history, but the meta tag is a fallback for most non-JS clients.
const metaRedir = `<!DOCTYPE html>
<html>
<head>
<title>Redirecting...</title>
<script>window.location.replace("%s");</script>
<meta http-equiv="refresh" content="0; URL='%s'">
</head>
<body>Redirecting to <a href="%s">%s</a>...</body>
</html>
`
safeTo := html.EscapeString(to)
body = fmt.Sprintf(metaRedir, safeTo, safeTo, safeTo, safeTo)
code = "302"
default:
codeInt, err := strconv.Atoi(code)
if err != nil {
return nil, h.Errf("Not a supported redir code type or not valid integer: '%s'", code)
}
if codeInt < 300 || codeInt > 399 {
return nil, h.Errf("Redir code not in the 3xx range: '%v'", codeInt)
}
}
```

#### [](https://mritd.com/2021/08/20/switching-rrom-nginx-too-caddy/#2-2-3%E3%80%81uri "2.2.3、uri")2.2.3、uri[](https://mritd.com/2021/08/20/switching-rrom-nginx-too-caddy/#2-2-3%E3%80%81uri)

`uri` 指令是一个特殊指令，它与 `rewrite` 类似，不同的是它用于对 URI 重写更加方便，其语法如下:

```
uri [<matcher>] strip_prefix|strip_suffix|replace|path_regexp \
<target> \
[<replacement> [<limit>]]
```

**语法中第二个参数为一个动词，用来定义如何替换 URI:**

-   `strip_prefix`: 从路径中去除前缀
-   `strip_suffix`: 从路径中去除后缀
-   `replace`: 在整个 URI 路径中执行子替换(例如 `/a/b/c/d` 替换为 `/a/1/2/d`)
-   `path_regexp`: 在路径中进行正则替换

以下为一些样例:

```
# 去除 "/api/v1" 前缀
uri strip_prefix /api/v1

# 去除 ".html" 后缀
uri strip_suffix .html

# 子路径替换 "/v1" => "/v2"
uri replace /v1/ /v2/

# 正则替换 "/api/v数字" => "/api"
uri path_regexp /api/v\d /api
```

其中在使用 `replace` 时最后面可以跟一个数字，代表从 URI 中找找替换多少次，默认为 `-1` 即全部替换。

### [](https://mritd.com/2021/08/20/switching-rrom-nginx-too-caddy/#2-3%E3%80%81WebSocket-%E4%BB%A3%E7%90%86 "2.3、WebSocket 代理")2.3、WebSocket 代理[](https://mritd.com/2021/08/20/switching-rrom-nginx-too-caddy/#2-3%E3%80%81WebSocket-%E4%BB%A3%E7%90%86)

在 Nginx 配置中，如果想要代理 WebSocket 链接，我们需要增加以下设置:

```
proxy_set_header Upgrade $http_upgrade;
proxy_set_header Connection "upgrade";
```

但是在 Caddy 中一切变得更加简单… 简单到就是我们啥也不用干，自动支持:

> Websocket proxying “just works” in v2; there is no need to “enable” websockets like in v1.

### [](https://mritd.com/2021/08/20/switching-rrom-nginx-too-caddy/#2-4%E3%80%81URL-%E7%BC%96%E7%A0%81 "2.4、URL 编码")2.4、URL 编码[](https://mritd.com/2021/08/20/switching-rrom-nginx-too-caddy/#2-4%E3%80%81URL-%E7%BC%96%E7%A0%81)

在使用路径匹配器时，URL 默认是被解码的，例如:

```
# 中文已经被解码，需要直接写解码后的字符串才能匹配到
redir /2016/03/22/Java-内存之直接内存 https://mritd.com/2016/03/22/java-memory-direct-memory permanent
```

**至于反向代理 `reverse_proxy` 传出时的编码暂时还没有遇到，还需要测试一下。**

### [](https://mritd.com/2021/08/20/switching-rrom-nginx-too-caddy/#2-5%E3%80%81%E5%BC%BA%E5%88%B6-HTTP "2.5、强制 HTTP")2.5、强制 HTTP[](https://mritd.com/2021/08/20/switching-rrom-nginx-too-caddy/#2-5%E3%80%81%E5%BC%BA%E5%88%B6-HTTP)

有些站点可能默认就是 HTTP 的，我们也不期望以 HTTPS 方式访问；但是 Caddy 默认会为站点进行 ACME 证书申请，而申请不下来证书时又访问不了；这种情况下只需要在站点地址上强制写明 HTTP 协议即可:

```
http://example.com {
    reverse_proxy ...
}
```

### [](https://mritd.com/2021/08/20/switching-rrom-nginx-too-caddy/#2-6%E3%80%81%E4%BB%A3%E7%90%86-HTTPS "2.6、代理 HTTPS")2.6、代理 HTTPS[](https://mritd.com/2021/08/20/switching-rrom-nginx-too-caddy/#2-6%E3%80%81%E4%BB%A3%E7%90%86-HTTPS)

如果想要代理 HTTPS 服务，那么只需要在 `reverse_proxy` 中填写 HTTPS 地址即可；**不过与 Nginx 不同，Caddy 的 TLS 校验默认是开着的，所以如果后端 HTTPS 证书过期等情况可能导致 Caddy 返回 502 错误；** 这种情况可以通过 `transport` 进行关闭:

```
reg.example.com {
    handle {
        request_body {
            max_size 1G
        }
        reverse_proxy {
            to https://172.16.11.40:443
            transport http {
                # SNI
                tls_server_name reg.example.link
                # 关闭后端 TLS 验证
                tls_insecure_skip_verify
            }
        }
    }
}
```

### [](https://mritd.com/2021/08/20/switching-rrom-nginx-too-caddy/#2-7%E3%80%81%E8%87%AA%E5%AE%9A%E4%B9%89%E8%AF%81%E4%B9%A6 "2.7、自定义证书")2.7、自定义证书[](https://mritd.com/2021/08/20/switching-rrom-nginx-too-caddy/#2-7%E3%80%81%E8%87%AA%E5%AE%9A%E4%B9%89%E8%AF%81%E4%B9%A6)

如果已经有自己的证书，而不期望 Caddy 自动申请，那么只需要在 `tls` 指令后加上证书即可:

```
reg.example.com {
    handle {
        ...
    }
    
    # 使用自定义证书
    tls cert.pem key.pem
}
```

### [](https://mritd.com/2021/08/20/switching-rrom-nginx-too-caddy/#2-8%E3%80%81%E6%97%A5%E5%BF%97%E6%89%93%E5%8D%B0 "2.8、日志打印")2.8、日志打印[](https://mritd.com/2021/08/20/switching-rrom-nginx-too-caddy/#2-8%E3%80%81%E6%97%A5%E5%BF%97%E6%89%93%E5%8D%B0)

Caddy 的日志系统与 Nginx 完全不同，Caddy 日志按照 Namespace 划分，**在站点配置中默认为只可以打印当前站点的请求日志，如果需要打印例如反向代理的上游地址等需要在全局日志配置中配置。** 日志这一块一句两句说不清，推荐直接看官方文档以及日志实现逻辑，如果懂 go 的话可以看看 [uber-go/zap](https://github.com/uber-go/zap) 这个日志框架；下面是按文件分开打印请求日志和上游日志的样例:

```
# Global options
{
    # 打印反向代理 upstream 信息日志(upstream 这个位置随便起名)
    log upstream {
        level DEBUG
        format json {
            time_format "iso8601"
        }
        output file /data/logs/upstream.log {
            roll_size 100mb
            roll_keep 3
            roll_keep_for 7d
        }
        # 需要指定 Namespace 才能打印
        include "http.handlers.reverse_proxy"
    }
}

example.com {
    # 打印站点请求日志
    log {
        format json {
            time_format "iso8601"
        }
        output file "/data/logs/example.com.log" {
            roll_size 100mb
            roll_keep 3
            roll_keep_for 7d
        }
    }
}
```

### [](https://mritd.com/2021/08/20/switching-rrom-nginx-too-caddy/#2-9%E3%80%81TLS-%E7%89%88%E6%9C%AC%E4%B8%8D%E6%94%AF%E6%8C%81 "2.9、TLS 版本不支持")2.9、TLS 版本不支持[](https://mritd.com/2021/08/20/switching-rrom-nginx-too-caddy/#2-9%E3%80%81TLS-%E7%89%88%E6%9C%AC%E4%B8%8D%E6%94%AF%E6%8C%81)

很不幸的是我们有一个 TLS 1.1 兼容的服务，当切换到 Caddy 后 TLS 1.1 已经不被支持，目前 Caddy 的 TLS 兼容性最小为 TLS 1.2，最大为 TLS 1.3:

**`protocols <min> [<max>]`**

> `protocols` specifies the minimum and maximum protocol versions. Default min: `tls1.2`. Default max: `tls1.3`.

## [](https://mritd.com/2021/08/20/switching-rrom-nginx-too-caddy/#%E4%B8%89%E3%80%81%E5%88%87%E6%8D%A2%E6%80%BB%E7%BB%93 "三、切换总结")三、切换总结[](https://mritd.com/2021/08/20/switching-rrom-nginx-too-caddy/#%E4%B8%89%E3%80%81%E5%88%87%E6%8D%A2%E6%80%BB%E7%BB%93)

总结一句话: 匹配器舒服，配置行为明确，配置引用少写一万行，其他的坑继续踩。