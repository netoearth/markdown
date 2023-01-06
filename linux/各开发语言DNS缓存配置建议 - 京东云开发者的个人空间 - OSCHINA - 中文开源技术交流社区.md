作者：翟贺龙

## 一、背景

在计算机领域，涉及性能优化动作时首先应被考虑的原则之一便是使用缓存，合理的数据缓存机制能够带来以下收益：

1. 缩短数据获取路径，热点数据就近缓存以便后续快速读取，从而明显提升处理效率；

2. 降低数据远程获取频次，缓解后端数据服务压力、减少前端和后端之间的网络带宽成本；

从 CPU 硬件的多级缓存设计，到浏览器快速展示页面，再到大行其道的 CDN、云存储网关等商业产品，处处应用了缓存理念。

在公网领域，如操作系统、浏览器和移动端 APP 等成熟产品所具备的缓存机制，极大的消解了网络提供商如电信移动联通、内容提供商如各大门户平台和 CDN 厂商直面的服务压力，运营商的 DNS 才能从容面对每秒亿万级的 DNS 解析，网络设备集群才能轻松承担每秒 Tbit 级的互联网带宽，CDN 平台才能快速处理每秒亿万次的请求。

面对公司目前庞大且仍在不断增长的的域名接入规模，笔者所在团队在不断优化集群架构、提升 DNS 软件性能的同时，也迫切需要推动各类客户端环境进行域名解析请求机制的优化，因此，特组织团队成员调研、编写了这篇指南文章，以期为公司、客户及合作方的前端开发运维人员给出合理建议，优化 DNS 整体请求流程，为业务增效。

本文主要围绕不同业务和开发语言背景下，客户端本地如何实现 DNS 解析记录缓存进行探讨，同时基于笔者所在团队对 DNS 本身及公司网络环境的掌握，给出一些其他措施，最终致力于客户端一侧的 DNS 解析请求规范化。

## 二、名词解释

## **1\. 客户端**

本文所述客户端，泛指所有主动发起网络请求的对象，包括但不限于服务器、PC、移动终端、操作系统、命令行工具、脚本、服务软件、用户 APP 等。

## **2\. DNS**

Domain Name System (Server/Service)，域名系统（服务器 / 服务），可理解为一种类数据库服务；

客户端同服务端进行网络通信，是靠 IP 地址识别对方；而作为客户端的使用者，人类很难记住大量 IP 地址，所以发明了易于记忆的域名如 www.jd.com，将域名和 IP 地址的映射关系，存储到 DNS 可供客户端查询；

客户端只有通过向 DNS 发起域名解析请求从而获取到服务端的 IP 地址后，才能向 IP 地址发起网络通信请求，真正获取到域名所承载的服务或内容。

参考：[域名系统](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fdocs.jdcloud.com%2Fcn%2Fjd-cloud-dns%2Fproduct-overview) [域名解析流程](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fdocs.jdcloud.com%2Fcn%2Fjd-cloud-dns%2Fdomain-effect)

## **3\. LDNS**

Local DNS，本地域名服务器；公网接入环境通常由所在网络供应商自动分配（供应商有控制权，甚至可作 DNS 劫持，即篡改解析域名得到的 IP），内网环境由 IT 部门设置自动分配；

通常 Unix、类 Unix、MacOS 系统可通过 /etc/resolv.conf 查看自己的 LDNS，在 nameserver 后声明，该文件亦支持用户自助编辑修改，从而指定 LDNS，如公网常见的公共 DNS 如谷歌 DNS、114DNS 等；纯内网环境通常不建议未咨询 IT 部门的情况下擅自修改，可能导致服务不可用；可参考 `man resolv.conf` 指令结果。

当域名解析出现异常时，同样应考虑 LDNS 服务异常或发生解析劫持情况的可能。

参考：[windows 系统修改 TCP/IP 设置（含 DNS）](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fsupport.microsoft.com%2Fzh-cn%2Fwindows%2F%25E6%259B%25B4%25E6%2594%25B9-tcp-ip-%25E8%25AE%25BE%25E7%25BD%25AE-bd0a07af-15f5-cd6a-363f-ca2b6f391ace)；

## **4\. hosts**

DNS 系统可以动态的提供域名和 IP 的映射关系，普遍存在于各类操作系统的 hosts 文件则是域名和 IP 映射关系的静态记录文件，且通常 hosts 记录优先于 DNS 解析，即本地无缓存或缓存未命中时，则优先通过 hosts 查询对应域名记录，若 hosts 无相关映射，则继续发起 DNS 请求。关于 Linux 环境下此逻辑的控制，请参考下文 C/C++ 语言 DNS 缓存介绍部分。

所以在实际工作中，常利用上述默认特性，将特定域名和特定 IP 映射关系写到 hosts 文件中（俗称 “固定 hosts”），用于绕开 DNS 解析过程，对目标 IP 作针对性访问（其效果与 curl 的 - x 选项，或 wget 的 -e 指定 proxy 选项，异曲同工）；

## **5\. TTL**

Time-To-Live，生存时间值，此概念在多领域适用且可能有不同意义。

本文涉及到 TTL 描述均针对数据缓存而言，可直白理解为已缓存数据的 “有效期”，从数据被缓存开始计，在缓存中存在时长超过 TTL 规定时长的数据被视为过期数据，数据被再次调用时会立刻同权威数据源进行有效性确认或重新获取。

因缓存机制通常是被动触发和更新，故在客户端的缓存有效期内，后端原始权威数据若发生变更，客户端不会感知，表现为业务上一定程度的数据更新延迟、缓存数据与权威数据短时不一致。

对于客户端侧 DNS 记录的缓存 TTL，我们建议值为 60s；同时如果是低敏感度业务比如测试、或域名解析调整不频繁的业务，可适当延长，甚至达到小时或天级别；

## 三、DNS 解析优化建议

## **1\. 各语言网络库对 DNS 缓存的支持调研**

以下调研结果，推荐开发人员参考，以实现自研客户端 DNS 缓存。各开发语言对 DNS 缓存支持可能不一样，在此逐个分析一下。

### C/C++ 语言

#### （1）glibc 的 getaddrinfo 函数

Linux 环境下的 glibc 库提供两个域名解析的函数：gethostbyname 函数和 getaddrinfo 函数，gethostbyname 是曾经常用的函数，但是随着向 IPv6 和线程化编程模型的转移，getaddrinfo 显得更有用，因为它既解析 IPv6 地址，又符合线程安全，推荐使用 getaddrinfo 函数。

函数原型：

```
int getaddrinfo( const char *node, 
                 const char *service,
                 const struct addrinfo *hints,
                 struct addrinfo **res);
```

getaddrinfo 函数是比较底层的基础库函数，很多开发语言的域名解析函数都依赖这个函数，因此我们在此介绍一下这个函数的处理逻辑。通过 strace 命令跟踪这个函数系统调用。

![](https://oscimg.oschina.net/oscnet/up-c8d0beebe8b60bd56cacbbcf87af0e8523f.png)

1）查找 nscd 缓存（nscd 介绍见后文）

我们在 linux 环境下通过 strace 命令可以看到如下的系统调用

```
//连接nscd
socket(PF_LOCAL, SOCK_STREAM|SOCK_CLOEXEC|SOCK_NONBLOCK, 0) = 3
connect(3, {sa_family=AF_LOCAL, sun_path="/var/run/nscd/socket"}, 110) = -1 ENOENT (No such file or directory)
close(3) 

```

通过 unix socket 接口 "/var/run/nscd/socket" 连接 nscd 服务查询 DNS 缓存。

2）查询 /etc/hosts 文件

如果 nscd 服务未启动或缓存未命中，继续查询 hosts 文件，我们应该可以看到如下的系统调用

```
//读取 hosts 文件
open("/etc/host.conf", O_RDONLY)        = 3
fstat(3, {st_mode=S_IFREG|0644, st_size=9, ...}) = 0
...
open("/etc/hosts", O_RDONLY|O_CLOEXEC)  = 3
fcntl(3, F_GETFD)                       = 0x1 (flags FD_CLOEXEC)
fstat(3, {st_mode=S_IFREG|0644, st_size=178, ...}) = 0
```

3）查询 DNS 服务

从 /etc/resolv.conf 配置中查询到 DNS 服务器（nameserver）的 IP 地址，然后做 DNS 查询获取解析结果。我们可以看到如下系统调用

```
//获取 resolv.conf 中 DNS 服务 IP
open("/etc/resolv.conf", O_RDONLY)      = 3
fstat(3, {st_mode=S_IFREG|0644, st_size=25, ...}) = 0
mmap(NULL, 4096, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x7fef2abee000
read(3, "nameserver 114.114.114.114\n\n", 4096) = 25
...
//连到 DNS 服务，开始 DNS 查询
connect(3, {sa_family=AF_INET, sin_port=htons(53), sin_addr=inet_addr("114.114.114.114")}, 16) = 0
poll([{fd=3, events=POLLOUT}], 1, 0)    = 1 ([{fd=3, revents=POLLOUT}])
```

而关于客户端是优先查找 /etc/hosts 文件，还是优先从 /etc/resolv.conf 中获取 DNS 服务器作查询解析，是由 /etc/nsswitch.conf 控制：

```
#/etc/nsswitch.conf 部分配置
...
#hosts:     db files nisplus nis dns
hosts:      files dns
...
```

实际通过 strace 命令可以看到，系统调用 nscd socket 之后，读取 /etc/resolv.conf 之前，会读取该文件

```
newfstatat(AT_FDCWD, "/etc/nsswitch.conf", {st_mode=S_IFREG|0644, st_size=510, ...}, 0) = 0
...
openat(AT_FDCWD, "/etc/nsswitch.conf", O_RDONLY|O_CLOEXEC) = 3
```

4）验证

```
#include <sys/socket.h>
#include <netdb.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <arpa/inet.h>
#include <unistd.h>

int gethostaddr(char * name);

int main(int argc, char *argv[])
{
    if (argc != 2)
    {
        fprintf(stderr, "%s $host", argv[0]);
        return -1;
    }

    int i = 0;
    for(i = 0; i < 5; i++)
    {
        int ret = -1;
        ret = gethostaddr(argv[1]);
        if (ret < 0)
        {
            fprintf(stderr, "%s $host", argv[0]);
            return -1;
        }
        //sleep(5);
    }

    return 0;
}

int gethostaddr(char* name)
{
    struct addrinfo hints;
    struct addrinfo *result;
    struct addrinfo *curr;
    int ret = -1;
    char ipstr[INET_ADDRSTRLEN];
    struct sockaddr_in  *ipv4;

    memset(&hints, 0, sizeof(struct addrinfo));
    hints.ai_family = AF_INET;
    hints.ai_socktype = SOCK_STREAM;

    ret = getaddrinfo(name, NULL, &hints, &result);
    if (ret != 0)
    {
        fprintf(stderr, "getaddrinfo: %s\n", gai_strerror(ret));
        return ret;
    }

    for (curr = result; curr != NULL; curr = curr->ai_next)
    {
        ipv4 = (struct sockaddr_in *)curr->ai_addr;
        inet_ntop(curr->ai_family, &ipv4->sin_addr, ipstr, INET_ADDRSTRLEN);
        printf("ipaddr:%s\n", ipstr);
    }

    freeaddrinfo(result);
    return 0;
}
```

综上分析，getaddrinfo 函数结合 nscd ，是可以实现 DNS 缓存的。

#### （2）libcurl 库的域名解析函数

libcurl 库是 c/c++ 语言下，客户端比较常用的网络传输库，curl 命令就是基于这个库实现。这个库也是调用 getaddrinfo 库函数实现 DNS 域名解析，也是支持 nscd DNS 缓存的。

```
int
Curl_getaddrinfo_ex(const char *nodename,
                    const char *servname,
                    const struct addrinfo *hints,
                    Curl_addrinfo **result)
{
    ...
    error = getaddrinfo(nodename, servname, hints, &aihead);
    if(error)
        return error;
    ...
}

```

### Java

Java 语言是很多公司业务系统开发的主要语言，通过编写简单的 HTTP 客户端程序测试验证 Java 的网络库是否支持 DNS 缓存。测试验证了 Java 标准库中 HttpURLConnection 和 Apache httpcomponents-client 这两个组件。

#### （1）Java 标准库 HttpURLConnection

```
import java.io.BufferedReader;
import java.io.InputStream;
import java.io.InputStreamReader;
import java.io.OutputStream;
import java.net.HttpURLConnection;
import java.net.URL;


public class HttpUrlConnectionDemo {

    public static void main(String[] args) throws Exception {
        String urlString = "http://example.my.com/";

        int num = 0;
        while (num < 5) {
            URL url = new URL(urlString);
            HttpURLConnection conn = (HttpURLConnection) url.openConnection();
            conn.setRequestMethod("GET");
            conn.setDoOutput(true);

            OutputStream os = conn.getOutputStream();
            os.flush();
            os.close();

            if (conn.getResponseCode() == HttpURLConnection.HTTP_OK) {
                InputStream is = conn.getInputStream();
                BufferedReader reader = new BufferedReader(new InputStreamReader(is));
                StringBuilder sb = new StringBuilder();
                String line;
                while ((line = reader.readLine()) != null) {
                    sb.append(line);
                }
                System.out.println("rsp:" + sb.toString());
            } else {
                System.out.println("rsp code:" + conn.getResponseCode());
            }
            num++;
        }
    }
}
```

测试结果显示 Java 标准库 HttpURLConnection 是支持 DNS 缓存，5 次请求中只有一次 DNS 请求。

#### （2）Apache httpcomponents-client

```
import java.util.ArrayList;
import java.util.List;

import org.apache.hc.client5.http.classic.methods.HttpGet;
import org.apache.hc.client5.http.entity.UrlEncodedFormEntity;
import org.apache.hc.client5.http.impl.classic.CloseableHttpClient;
import org.apache.hc.client5.http.impl.classic.CloseableHttpResponse;
import org.apache.hc.client5.http.impl.classic.HttpClients;
import org.apache.hc.core5.http.HttpEntity;
import org.apache.hc.core5.http.NameValuePair;
import org.apache.hc.core5.http.io.entity.EntityUtils;
import org.apache.hc.core5.http.message.BasicNameValuePair;

public class QuickStart {
    public static void main(final String[] args) throws Exception {
        int num = 0;
        while (num < 5) {
            try (final CloseableHttpClient httpclient = HttpClients.createDefault()) {
                final HttpGet httpGet = new HttpGet("http://example.my.com/");
                try (final CloseableHttpResponse response1 = httpclient.execute(httpGet)) {
                    System.out.println(response1.getCode() + " " + response1.getReasonPhrase());
                    final HttpEntity entity1 = response1.getEntity();
                    EntityUtils.consume(entity1);
                }
            }
        num++;
        }
    }
}
```

测试结果显示 Apache httpcomponents-client 支持 DNS 缓存，5 次请求中只有一次 DNS 请求。

从测试中发现 Java 的虚拟机实现一套 DNS 缓存，即实现在 java.net.InetAddress 的一个简单的 DNS 缓存机制，默认为缓存 30 秒，可以通过 networkaddress.cache.ttl 修改默认值，缓存范围为 JVM 虚拟机进程，也就是说同一个 JVM 进程中，30 秒内一个域名只会请求 DNS 服务器一次。同时 Java 也是支持 nscd 的 DNS 缓存，估计底层调用 getaddrinfo 函数，并且 nscd 的缓存级别比 Java 虚拟机的 DNS 缓存高。

```
# 默认缓存 ttl 在 jre/lib/security/java.security 修改，其中 0 是不缓存，-1 是永久缓存
networkaddress.cache.ttl=10

# 这个参数 sun.net.inetaddr.ttl 是以前默认值，目前已经被 networkaddress.cache.ttl 取代

```

### Go

随着云原生技术的发展，Go 语言逐渐成为云原生的第一语言，很有必要验证一下 Go 的标准库是否支持 DNS 缓存。通过我们测试验证发现 Go 的标准库 net.http 是不支持 DNS 缓存，也是不支持 nscd 缓存，应该是没有调用 glibc 的库函数，也没有实现类似 getaddrinfo 函数的功能。这个跟 Go 语言的自举有关系，Go 从 1.5 开始就基本全部由 Go (.go) 和汇编 (.s) 文件写成的，以前版本的 C (.c) 文件被全部重写。不过有一些第三方 Go 版本 DNS 缓存库，可以自己在应用层实现，还可以使用 fasthttp 库的 httpclient。

#### （1）标准库 net.http

```
package main

import (
        "flag"
        "fmt"
        "io/ioutil"
        "net/http"
        "time"
)

var httpUrl string

func main() {
    flag.StringVar(&httpUrl, "url", "", "url")
    flag.Parse()
    getUrl := fmt.Sprintf("http://%s/", httpUrl)

    fmt.Printf("url: %s\n", getUrl)
    for i := 0; i < 5; i++ {
        _, buf, err := httpGet(getUrl)
        if err != nil {
            fmt.Printf("err: %v\n", err)
            return
        }
        fmt.Printf("resp: %s\n", string(buf))
        time.Sleep(10 * time.Second)    # 等待10s发起另一个请求
    }
}

func httpGet(url string) (int, []byte, error) {
    client := createHTTPCli()
    resp, err := client.Get(url)
    if err != nil {
        return -1, nil, fmt.Errorf("%s err [%v]", url, err)
    }
    defer resp.Body.Close()

    buf, err := ioutil.ReadAll(resp.Body)
    if err != nil {
        return resp.StatusCode, buf, err
    }

    return resp.StatusCode, buf, nil
}
func createHTTPCli() *http.Client {
    readWriteTimeout := time.Duration(30) * time.Second
    tr := &http.Transport{
        DisableKeepAlives: true,  //设置短连接
        IdleConnTimeout:   readWriteTimeout,
    }
    client := &http.Client{
        Timeout:   readWriteTimeout,
        Transport: tr,
    }
    return client
}
```

从测试结果来看，net.http 每次都去 DNS 查询，不支持 DNS 缓存。

#### （2）fasthttp 库

fasthttp 库是 Go 版本高性能 HTTP 库，通过极致的性能优化，性能是标准库 net.http 的 10 倍，其中一项优化就是支持 DNS 缓存，我们可以从其源码看到

```
//主要在fasthttp/tcpdialer.go中
type TCPDialer struct {
    ...
    // This may be used to override DNS resolving policy, like this:
    // var dialer = &fasthttp.TCPDialer{
    // Resolver: &net.Resolver{
    // PreferGo:     true,
    // StrictErrors: false,
    // Dial: func (ctx context.Context, network, address string) (net.Conn, error) {
    // d := net.Dialer{}
    // return d.DialContext(ctx, "udp", "8.8.8.8:53")
    // },
    // },
    // }
    Resolver Resolver

    // DNSCacheDuration may be used to override the default DNS cache duration (DefaultDNSCacheDuration)
    DNSCacheDuration time.Duration
    ...
}
```

可以参考如下方法使用 fasthttp client 端

```
func main() {
// You may read the timeouts from some config
readTimeout, _ := time.ParseDuration("500ms")
writeTimeout, _ := time.ParseDuration("500ms")
maxIdleConnDuration, _ := time.ParseDuration("1h")
client = &fasthttp.Client{
ReadTimeout:                   readTimeout,
WriteTimeout:                  writeTimeout,
MaxIdleConnDuration:           maxIdleConnDuration,
NoDefaultUserAgentHeader:      true, // Don't send: User-Agent: fasthttp
DisableHeaderNamesNormalizing: true, // If you set the case on your headers correctly you can enable this
DisablePathNormalizing:        true,
// increase DNS cache time to an hour instead of default minute
Dial: (&fasthttp.TCPDialer{
Concurrency:      4096,
DNSCacheDuration: time.Hour,
}).Dial,
}
sendGetRequest()
sendPostRequest()
}
```

#### （3）第三方 DNS 缓存库

这个是 github 中的一个 Go 版本 [DNS 缓存库](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fgithub.com%2Frs%2Fdnscache)

可以参考如下代码，在 HTTP 库中支持 DNS 缓存

```
r := &dnscache.Resolver{}
t := &http.Transport{
    DialContext: func(ctx context.Context, network string, addr string) (conn net.Conn, err error) {
        host, port, err := net.SplitHostPort(addr)
        if err != nil {
            return nil, err
        }
        ips, err := r.LookupHost(ctx, host)
        if err != nil {
            return nil, err
        }
        for _, ip := range ips {
            var dialer net.Dialer
            conn, err = dialer.DialContext(ctx, network, net.JoinHostPort(ip, port))
            if err == nil {
                break
            }
        }
        return
    },
}
```

### Python

#### （1）requests 库

```
#!/bin/python

import requests

url = 'http://example.my.com/'

num = 0
while num < 5:
    headers={"Connection":"close"}     # 开启短连接
    r = requests.get(url,headers = headers)
    print(r.text)
    num +=1
```

#### （2）httplib2 库

```
#!/usr/bin/env python
import httplib2
http = httplib2.Http()
url = 'http://example.my.com/'

num = 0
while num < 5:
    loginHeaders={
        'User-Agent': 'Mozilla/5.0 (Windows NT 5.1) AppleWebKit/537.36 (KHTML, like Gecko) Maxthon/4.0 Chrome/30.0.1599.101 Safari/537.36',
        'Connection': 'close'  # 开启短连接
    }
    response, content = http.request(url, 'GET', headers=loginHeaders)
    print(response)
    print(content)
    num +=1
```

#### （3）urllib2 库

```
#!/bin/python

import urllib2
import cookielib

httpHandler = urllib2.HTTPHandler(debuglevel=1)
httpsHandler = urllib2.HTTPSHandler(debuglevel=1)
opener = urllib2.build_opener(httpHandler, httpsHandler)
urllib2.install_opener(opener)

loginHeaders={
    'User-Agent': 'Mozilla/5.0 (Windows NT 5.1) AppleWebKit/537.36 (KHTML, like Gecko) Maxthon/4.0 Chrome/30.0.1599.101 Safari/537.36',
    'Connection': 'close' # 开启短连接
}

num = 0
while num < 5:
    request=urllib2.Request('http://example.my.com/',headers=loginHeaders)
    response = urllib2.urlopen(request)
    page=''
    page= response.read()
    print response.info()
    print page
    num +=1

```

Python 测试三种库都是支持 nscd 的 DNS 缓存的 (推测底层也是调用 getaddrinfo 函数)，以上测试时使用 HTTP 短连接，都在 python2 环境测试。

### 总结

针对 HTTP 客户端来说，可以优先开启 HTTP 的 keep-alive 模式，可以复用 TCP 连接，这样可以减少 TCP 握手耗时和重复请求域名解析，然后再开启 nscd 缓存，除了 Go 外，C/C++、Java、Python 都可支持 DNS 缓存，减少 DNS 查询耗时。

> 这里只分析了常用 C/C++、Java、Go、Python 语言，欢迎熟悉其他语言的小伙伴补充。

## 2\. Unix / 类 Unix 系统常用 dns 缓存服务：

在由于某些特殊原因，自研或非自研客户端本身无法提供 DNS 缓存支持的情况下，建议管理人员在其所在系统环境中部署 DNS 缓存程序；

现介绍 Unix / 类 Unix 系统适用的几款常见轻量级 DNS 缓存程序。而多数桌面操作系统如 Windows、MacOS 和几乎所有 Web 浏览器均自带 DNS 缓存功能，本文不再赘述。

P.S. DNS 缓存服务请务必确保随系统开机启动；

### nscd

name service cache daemon 即装即用，通常为 linux 系统默认安装，相关介绍可参考其 manpage：`man nscd`；`man nscd.conf`

（1）安装方法：通过系统自带软件包管理程序安装，如 `yum install nscd`

（2）缓存管理（清除）：

1. `service nscd restart` 重启服务清除所有缓存；

2. `nscd -i hosts` 清除 hosts 表中的域名缓存（hosts 为域名缓存使用的 table 名称，nscd 有多个缓存 table，可参考程序相关 manpage）

### dnsmasq

较为轻量，可选择其作为 nscd 替代，通常需单独安装

（1）安装方法：通过系统自带软件包管理程序安装，如 `yum install dnsmasq`

（2）核心文件介绍（基于 Dnsmasq version 2.86，较低版本略有差异，请参考对应版本文档如 manpage 等）

（3）/etc/default/dnsmasq 提供六个变量定义以支持六种控制类功能

（4）/etc/dnsmasq.d/ 此目录含 README 文件，可参考；目录内可以存放自定义配置文件

（5）/etc/dnsmasq.conf 主配置文件，如仅配置 dnsmasq 作为缓存程序，可参考以下配置

```
listen-address=127.0.0.1                #程序监听地址，务必指定本机内网或回环地址，避免暴露到公网环境
port=53                                 #监听端口
resolv-file=/etc/dnsmasq.d/resolv.conf  #配置dnsmasq向自定义文件内的 nameserver 转发 dns 解析请求
cache-size=150                          #缓存记录条数，默认 150 条，可按需调整、适当增大
no-negcache                             #不缓存解析失败的记录，主要是 NXDOMAIN，即域名不存在
log-queries=extra                       #开启日志记录，指定“=extra”则记录更详细信息，可仅在问题排查时开启，平时关闭
log-facility=/var/log/dnsmasq.log       #指定日志文件

#同时需要将本机 /etc/resolv.conf 第一个 nameserver 指定为上述监听地址，这样本机系统的 dns 查询请求才会通过 dnsmasq 代为转发并缓存响应结果。
#另 /etc/resolv.conf 务必额外配置 2 个 nameserver，以便 dnsmasq 服务异常时支持系统自动重试，注意 resolv.conf 仅读取前 3 个 nameserver
```

（6）缓存管理（清除）：

1. `` kill -s HUP `pidof dnsmasq` `` 推荐方式，无需重启服务

2. `` kill -s TERM `pidof dnsmasq` `` 或 `service dnsmasq stop`

3. `service dnsmasq force-reload` 或 `service dnsmasq restart`

（7）官方文档：[https://thekelleys.org.uk/dnsmasq/doc.html](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fthekelleys.org.uk%2Fdnsmasq%2Fdoc.html)

## 3\. 纯内网业务取消查询域名的 AAAA 记录的请求

以 linux 操作系统为例，常用的网络请求命令行工具常常通过调用 getaddrinfo () 完成域名解析过程，如 ping、telnet、curl、wget 等，但其可能出于通用性的考虑，均被设计为对同一个域名每次解析会发起两个请求，分别查询域名 A 记录（即 IPV4 地址）和 AAAA 记录（即 IPV6 地址）。

因目前大部分公司的内网环境及云上内网环境还未使用 ipv6 网络，故通常 DNS 系统不为内网域名添加 AAAA 记录，徒劳请求域名的 AAAA 记录会造成前端应用和后端 DNS 服务不必要的资源开销。因此，仅需请求内网域名的业务，如决定自研客户端，建议开发人员视实际情况，可将其设计为仅请求内网域名 A 记录，尤其当因故无法实施本地缓存机制时。

## 4\. 规范域名处理逻辑

客户端需严格规范域名 / 主机名的处理逻辑，避免产生大量对不存在域名的解析请求（确保域名从权威渠道获取，避免故意或意外使用随机构造的域名、主机名），因此类请求的返回结果（NXDOMAIN）通常不被缓存或缓存时长较短，且会触发客户端重试，对后端 DNS 系统造成一定影响。