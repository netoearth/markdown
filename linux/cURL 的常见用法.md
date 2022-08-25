![图片](https://mmbiz.qpic.cn/mmbiz_svg/5mxuSU5RGhYuRXhJNhIN4WVNzpxr3kriaWDk8jhX1iauvtYuWqyrJP5u5ibXicmiboe6Cd4mbU425eD0J8ic5okmKN5RWHunNk6gic5/640?wx_fmt=svg&wxfrom=5&wx_lazy=1&wx_co=1)

cURL Logo

**cURL** 是一个开源免费项目，主要是命令行工具 **cURL** 和 libcurl，**cURL** 可以处理任何网络传输协议，但是不涉及任何具体的**数据处理**。

**cURL** 支持的通信协议非常丰富，如 DICT，FILE，FTP，FTPS，GOPHER，HTTP，HTTPS，IMAP，IMAPS，LDAP，LDAPS，MQTT，POP3，POP3S，RTMP， RTMPS，RTSP，SCP，SFTP，SMB，SMBS，SMTP，SMTPS，TELNET 以及 TFTP。查看 cURL 源代码可以访问官方 Github。

如果安装 **cURL** 呢？

ubuntu / Debian.

```
sudo apt install curl
```

CentOS / Fedora.

```
sudo yum install curl
```

Windows.

如果你已经安装了 Git，那么 Git Bash 自带 **cURL** . 如果作为开发者你 git 都没有，那么只能官方手动下载。

### 1\. 请求源码

直接 curl 。

```
$ curl http://wttr.in/
```

上面请求的示例网址是一个天气网站，很有意思，会根据你的请求 ip 信息返回你所在位置的天气情况。

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

curl wttr.in

写这篇文字时我所在的上海正在下雨，窗外飘雨无休无止。

### 2\. 文件下载

使用 `-o` 保存文件，类似于 wget 命令，比如下载 README 文本保存为 readme.txt 文件。如果你需要自定义文件名，可以使用 `-O`自定使用 url 中的文件名。

```
$ curl -o readme.txt https://mirrors.nju.edu.cn/kali/README  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current                                 Dload  Upload   Total   Spent    Left  Speed100   159  100   159    0     0   1939      0 --:--:-- --:--:-- --:--:--  1939
```

下载文件会显示下载状态，如数据量大小、传输速度、剩余时间等。可以使用 `-s` 参数禁用进度表。

```
$ curl -o readme.txt https://mirrors.nju.edu.cn/kali/README  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current                                 Dload  Upload   Total   Spent    Left  Speed100   159  100   159    0     0   1939      0 --:--:-- --:--:-- --:--:--  1939$ $ curl -o readme.txt https://mirrors.nju.edu.cn/kali/README -s
```

也可以使用 `--process-bar` 参数让进度表显示为进度条。

```
$ curl -o readme.txt https://mirrors.nju.edu.cn/kali/README --progress-bar########################################################################################## 100.0%
```

**cURL** 作为强大的代名词，断点续传自然手到擒来，使用 `-C -` 参数即可。下面是断点续传下载 ubuntu20.04 镜像的例子。

```
$ curl -O https://mirrors.nju.edu.cn/ubuntu-releases/20.04/ubuntu-20.04-desktop-amd64.iso --progress-bar##                                                                                               1.7%^C$ curl -C - -O https://mirrors.nju.edu.cn/ubuntu-releases/20.04/ubuntu-20.04-desktop-amd64.iso --progress-bar###                                                                                              2.4%^C$ curl -C - -O https://mirrors.nju.edu.cn/ubuntu-releases/20.04/ubuntu-20.04-desktop-amd64.iso --progress-bar###                                                                                               2.7%^C$ 
```

什么？下载时不想占用太多网速？使用 `--limit-rate` 限个速吧。

```
curl -C - -O https://mirrors.nju.edu.cn/ubuntu-releases/20.04/ubuntu-20.04-desktop-amd64.iso --limit-rate 100k
```

什么？你又要从 FTP 服务器下载文件了？不慌。

```
curl -u user:password -O ftp://ftp_server/path/to/file/
```

### 3\.  Response Headers

使用 `-i` 参数显示 Response Headers 信息。使用 `-I` 可以只显示 Response Headers 信息。

```
$ curl -I http://wttr.inHTTP/1.1 200 OKServer: nginx/1.10.3Date: Sat, 30 May 2020 09:57:03 GMTContent-Type: text/plain; charset=utf-8Content-Length: 8678Connection: keep-aliveAccess-Control-Allow-Origin: *
```

### 4\. 请求方式(GET/POST/...)

使用 `-X` 轻松更改请求方式。

```
$ curl -X GET http://wttr.in$ curl -X POST http://wttr.in$ curl -X PUT http://wttr.in...
```

### 5\. 请求参数

以传入参数 `name`  值为 `未读代码` 为例。

Get 方式参数直接url拼接参数。

```
$ curl -X GET http://wttr.in?name=未读代码
```

Post 方式使用 `--data` 设置参数。

```
$ curl -X POST --data "name=未读代码" http://wttr.in
```

请求时也可以自定义 **header** 参数，使用 `--harder` 添加。

```
$ curl --header "Content-Type:application/json" http://wttr.in
```

### 6\. 文件上传

**cURL** 的强大远不止此，表单提交，上传文件内容也不在话下，只需要使用 `-F`  或者 `-D`参数，`-F` 会自动加上请求头 `Content-Type: multipart/form-data` ，而 `-D` 则是 `Content-Type : application/x-www-form-urlencoded`.

比如上传一个 protrait.jpg 图片。

```
$ curl -F profile=@portrait.jpg https://example.com/upload
```

提交一个具有 name 和 age 参数的 form 表单。

```
curl -F name=Darcy -F age=18 https://example.com/upload
```

参数对应的内容也可以从文件中读取。

```
curl -F "content=<达西的身世.txt" https://example.com/upload
```

上传时同时指定内容类型。

```
curl -F "content=<达西的身世.txt;type=text/html" https://example.com/upload
```

上传文件的和其他参数一起。

```
curl -F 'file=@"localfile";filename="nameinpost"' example.com/upload
```

### 7\. 网址通配

**cURL** 可以实现多个网址的匹配，你可以使用 `{}` 结合逗号分割来标识使用 url 中的某一段，也可以使用 `[]` 来表示范围参数。

```
# 请求 www.baidu.com 和  pan.baidu.com 和 fanyi.baidu.com$ curl http://{www,pan,fanyi}.baidu.com# 虚构网址1-10开头的baidu.com，然后请求$ curl http://[1-10].baidu.com# 虚构网址a-z开头的baidu.com，然后请求$ curl http://[a-z].baidu.com
```

这种方式有时候还是很有用处的，比如说你发现了某个网站的 url 规律。

### 8\. 使用 cookie

请求时使用 `-c` 参数存储响应的 cookie，使用 `-b` 可以在请求时带上指定 cookie.

```
$ curl -c wdbyte_cookies http://www.wdbyte.com$ curl -b wdbyte_cookes http://www.wdbyte.com
```

### 总结

以上就是 **cURL** 的常见用法了，最后告诉你一个小技巧，Chrome、Firefox 等浏览器可以直接拷贝请求为 cURL 语句。保存之后下次请求测试非常方便。

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

Chrome 复制 cURL 请求

**参考资料**

1.  https://curl.haxx.se/docs/manpage.html