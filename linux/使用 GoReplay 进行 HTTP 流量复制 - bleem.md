## [](https://mritd.com/2021/08/03/use-goreplay-to-record-your-live-traffic/#%E4%B8%80%E3%80%81%E5%AE%89%E8%A3%85 "一、安装")一、安装[](https://mritd.com/2021/08/03/use-goreplay-to-record-your-live-traffic/#%E4%B8%80%E3%80%81%E5%AE%89%E8%A3%85)

GoReplay 采用 Go 编写，其只有一个单独的可执行文件，在官方 [Release](https://github.com/buger/goreplay/releases) 页下载后将其放到 `PATH` 目录即可。

```
wget https://github.com/buger/goreplay/releases/download/v1.2.0/gor_v1.2.0_x64.tar.gz
tar -zxvf gor_v1.2.0_x64.tar.gz
mv gor /usr/local/bin
```

## [](https://mritd.com/2021/08/03/use-goreplay-to-record-your-live-traffic/#%E4%BA%8C%E3%80%81%E5%9F%BA%E6%9C%AC%E4%BD%BF%E7%94%A8 "二、基本使用")二、基本使用[](https://mritd.com/2021/08/03/use-goreplay-to-record-your-live-traffic/#%E4%BA%8C%E3%80%81%E5%9F%BA%E6%9C%AC%E4%BD%BF%E7%94%A8)

GoReplay 命令行整体使用方式为指定输入端和输入端，然后 GoReplay 从输入端将流量复制到输出端。

### [](https://mritd.com/2021/08/03/use-goreplay-to-record-your-live-traffic/#2-1%E3%80%81%E5%AE%9E%E6%97%B6%E6%B5%81%E9%87%8F%E5%A4%8D%E5%88%B6 "2.1、实时流量复制")2.1、实时流量复制[](https://mritd.com/2021/08/03/use-goreplay-to-record-your-live-traffic/#2-1%E3%80%81%E5%AE%9E%E6%97%B6%E6%B5%81%E9%87%8F%E5%A4%8D%E5%88%B6)

GoReplay 输入端可以指定一个 tcp 地址，然后 GoReplay 将该端口流量复制到输出端；下面样例展示从 `127.0.0.0:8000` 复制流量并输出到控制台的样例。

**首先启动一个 HTTP Server，这里直接使用 `python` 的 HTTP Server**

[![](https://cdn.oss.link/markdown/hnOfHX.png)](https://cdn.oss.link/markdown/hnOfHX.png)

**接着再让 gor 监听同样的端口，`--output-stdout` 指定输出端为控制台**

[![](https://cdn.oss.link/markdown/QhFUdM.png)](https://cdn.oss.link/markdown/QhFUdM.png)

**此时通过 `curl` 访问 `python` 的 HTTP Server 可以看到 gor 将 HTTP 请求复制并输出到了控制台**

[![](https://cdn.oss.link/markdown/uilePc.png)](https://cdn.oss.link/markdown/uilePc.png)

同样如果我们通过 `--output-http` 选项将输出端指定为另一个 HTTP Server，那么 gor 会将请求同步复制并发送到输出端 HTTP Server。

[![](https://cdn.oss.link/markdown/vFQiRs.png)](https://cdn.oss.link/markdown/vFQiRs.png)

### [](https://mritd.com/2021/08/03/use-goreplay-to-record-your-live-traffic/#2-2%E3%80%81%E6%B5%81%E9%87%8F%E6%8A%93%E5%8F%96%E4%B8%8E%E9%87%8D%E6%94%BE "2.2、流量抓取与重放")2.2、流量抓取与重放[](https://mritd.com/2021/08/03/use-goreplay-to-record-your-live-traffic/#2-2%E3%80%81%E6%B5%81%E9%87%8F%E6%8A%93%E5%8F%96%E4%B8%8E%E9%87%8D%E6%94%BE)

#### [](https://mritd.com/2021/08/03/use-goreplay-to-record-your-live-traffic/#2-2-1%E3%80%81%E5%9F%BA%E6%9C%AC%E4%BD%BF%E7%94%A8 "2.2.1、基本使用")2.2.1、基本使用[](https://mritd.com/2021/08/03/use-goreplay-to-record-your-live-traffic/#2-2-1%E3%80%81%E5%9F%BA%E6%9C%AC%E4%BD%BF%E7%94%A8)

GoReplay 可以将输出端指定为文件，从而将流量保存到文件中，然后 GoReplay 读取该保存的流量文件并重放到指定的 HTTP Server 中。

**首先通过 `--outpu-file` 选项将请求保存到文件中**

[![](https://cdn.oss.link/markdown/yuxiYQ-1627911532-lA2val.png)](https://cdn.oss.link/markdown/yuxiYQ-1627911532-lA2val.png)

**使用 `--input-file` 选项读取流量信息，然后通过 `--output-http` 选项重放到目标服务器**

[![](https://cdn.oss.link/markdown/cRB3x4-1627911738-7pLgTp.png)](https://cdn.oss.link/markdown/cRB3x4-1627911738-7pLgTp.png)

#### [](https://mritd.com/2021/08/03/use-goreplay-to-record-your-live-traffic/#2-2-2%E3%80%81%E6%89%A9%E5%B1%95%E9%80%89%E9%A1%B9 "2.2.2、扩展选项")2.2.2、扩展选项[](https://mritd.com/2021/08/03/use-goreplay-to-record-your-live-traffic/#2-2-2%E3%80%81%E6%89%A9%E5%B1%95%E9%80%89%E9%A1%B9)

在将流量保存到文件时，默认情况下 GoReplay 以块形式写入文件，并且每个块将生成一个独立的文件名(`test_0.gor`)，如果想要将所有块的流量全部写入一个文件中，**可以设置 `--output-file-append` 为 `true`。**

同时 GoReplay 输出文件名支持日期占位符，例如 `--output-file %Y%m%d.gor` 会生成 `20210801.gor` 这种文件名；所有可用的日期占位符如下:

-   `%Y`: year including the century (at least 4 digits)
-   `%m`: month of the year (01..12)
-   `%d`: Day of the month (01..31)
-   `%H`: Hour of the day, 24-hour clock (00..23)
-   `%M`: Minute of the hour (00..59)
-   `%S`: Second of the minute (00..60)

请求比较多时，将流量保存到文件可能会导致文件很大，这时候可以使用 `.gz` 结尾作为文件名，GoReplay 读取到 `.gz` 后缀后会自动进行 GZip 压缩处理。

```
gor --input-raw :8000 --output-file test.gor.gz
```

如果需要对多个文件进行聚合重放，只需要指定多个文件即可，重放过程中 GoReplay 会自动保持请求顺序:

```
gor --input-file *.gor --output-http http://127.0.0.1:8080
```

在使用文件输入时，GoReplay 还支持压力测试，通过 `test.gor|200%` 这种方式指定的文件名，GoReplay 会以两倍的速率进行请求重放:

```
# Replay from file on 2x speed 
gor --input-file "requests.gor|200%" --output-http "staging.com"
```

### [](https://mritd.com/2021/08/03/use-goreplay-to-record-your-live-traffic/#2-3%E3%80%81%E6%95%B0%E6%8D%AE%E4%B8%A2%E5%A4%B1%E4%B8%8E%E7%BC%93%E5%86%B2%E5%8C%BA "2.3、数据丢失与缓冲区")2.3、数据丢失与缓冲区[](https://mritd.com/2021/08/03/use-goreplay-to-record-your-live-traffic/#2-3%E3%80%81%E6%95%B0%E6%8D%AE%E4%B8%A2%E5%A4%B1%E4%B8%8E%E7%BC%93%E5%86%B2%E5%8C%BA)

GoReplay 采用比较底层的数据包拦截技术，当一个 TCP 数据包到达时内核 GoReplay 会进行拦截；然而数据包可以乱序到达，接下来内核需要重建 TCP 流来保证上层应用能以正确的顺序读取 TCP 数据包，这时候内核就会有一个数据包的缓冲区；默认情况下 Linux 系统的缓冲区为 2M，Windiws 为 1M，当特定的 HTTP 请求数据包超过缓冲区时，GoReplay 就无法正确的拦截(因为 GoReplay 需要一个完整的 HTTP 请求数据包用于保存到文件或者重放)，同时可能会导致请求丢失、请求损坏等问题。

为了解决这种问题，GoReplay 提供了 `--input-raw-buffer-size` 选项用于调整缓冲区大小，例如 `--input-raw-buffer-size 10485760` 选项会将缓冲区调整为 10M。

### [](https://mritd.com/2021/08/03/use-goreplay-to-record-your-live-traffic/#2-4%E3%80%81%E9%80%9F%E7%8E%87%E9%99%90%E5%88%B6 "2.4、速率限制")2.4、速率限制[](https://mritd.com/2021/08/03/use-goreplay-to-record-your-live-traffic/#2-4%E3%80%81%E9%80%9F%E7%8E%87%E9%99%90%E5%88%B6)

某些情况下可能为了方便调试，我们在生产环境抓取流量并镜像到测试环境进行重放；但是可能由于生产环境流量比较大，我们并不需要如此大的请求速率，这时候可以通过速率限制让 GoReplay 帮我们控制请求数量。

**绝对数量限制:** 使用 `--output-http "ADDRESS|N"` 形式的参数时，GoReplay 会保证镜像的流量请求每秒不会超过 “N” 个。

```
# staging.server will not get more than ten requests per second
gor --input-tcp :28020 --output-http "http://staging.com|10"
```

**百分比限制限制:** 使用 `--output-http "ADDRESS|N%"` 形式的参数时，GoReplay 会保证镜像的流量维持在总流量的 “N%”。

### [](https://mritd.com/2021/08/03/use-goreplay-to-record-your-live-traffic/#2-5%E3%80%81%E8%AF%B7%E6%B1%82%E8%BF%87%E6%BB%A4 "2.5、请求过滤")2.5、请求过滤[](https://mritd.com/2021/08/03/use-goreplay-to-record-your-live-traffic/#2-5%E3%80%81%E8%AF%B7%E6%B1%82%E8%BF%87%E6%BB%A4)

在某些时候我们只期望把生产环境的特定流量重放到测试环境，或者禁止一些流量重放到测试环境，这时候我们可以使用 GoReplay 的过滤功能；GoReplay 提供以下选项来提供过滤功能:

-   `--http-allow-header`: 允许重放的 HTTP 头(支持正则)
-   `--http-allow-method`: 允许重放的 HTTP 方法
-   `--http-allow-url`: 允许重放的 URL(支持正则)
-   `--http-disallow-header`: 不允许的 HTTP 头(支持正则)
-   `--http-disallow-url`: 不允许的 HTTP URL(支持正则)

以下是官方给出的命令样例:

```
# only forward requests being sent to the /api endpoint
gor --input-raw :8080 --output-http staging.com --http-allow-url /api

# only forward requests NOT being sent to the /api... endpoint
gor --input-raw :8080 --output-http staging.com --http-disallow-url /api

# only forward requests with an api version of 1.0x
gor --input-raw :8080 --output-http staging.com --http-allow-header api-version:^1\.0\d

# only forward requests NOT containing User-Agent header value "Replayed by Gor"
gor --input-raw :8080 --output-http staging.com --http-disallow-header "User-Agent: Replayed by Gor"

gor --input-raw :80 --output-http "http://staging.server" \
    --http-allow-method GET \
    --http-allow-method OPTIONS
```

### [](https://mritd.com/2021/08/03/use-goreplay-to-record-your-live-traffic/#2-6%E3%80%81%E8%AF%B7%E6%B1%82%E9%87%8D%E5%86%99 "2.6、请求重写")2.6、请求重写[](https://mritd.com/2021/08/03/use-goreplay-to-record-your-live-traffic/#2-6%E3%80%81%E8%AF%B7%E6%B1%82%E9%87%8D%E5%86%99)

有时候可能测试环境的 URL 路径与生产环境完全不同，此时如果直接把生产环境的流量在测试环境重放可能会导致请求路径错误等情况；为此 GoReplay 提供了 URL 重写、参数设置、请求头设置等功能。

**通过 `--http-rewrite-url` 选项进行 URL 重写**

```
# Rewrites all `/v1/user/<user_id>/ping` requests to `/v2/user/<user_id>/ping`
gor --input-raw :8080 --output-http staging.com --http-rewrite-url /v1/user/([^\\/]+)/ping:/v2/user/$1/ping
```

**设置 URL 参数**

```
gor --input-raw :8080 --output-http staging.com --http-set-param api_key=1
```

**设置请求头**

```
gor --input-raw :80 --output-http "http://staging.server" \
    --http-header "User-Agent: Replayed by Gor" \
    --http-header "Enable-Feature-X: true"
```

**Host 头是一个特殊的请求头，默认情况下 GoReplay 会将其自动设置为目标重放地址的域名，如果想关闭这种默认行为请使用 `--http-original-host` 选项。**

## [](https://mritd.com/2021/08/03/use-goreplay-to-record-your-live-traffic/#%E4%B8%89%E3%80%81%E5%85%B6%E4%BB%96%E9%AB%98%E7%BA%A7%E9%85%8D%E7%BD%AE "三、其他高级配置")三、其他高级配置[](https://mritd.com/2021/08/03/use-goreplay-to-record-your-live-traffic/#%E4%B8%89%E3%80%81%E5%85%B6%E4%BB%96%E9%AB%98%E7%BA%A7%E9%85%8D%E7%BD%AE)

### [](https://mritd.com/2021/08/03/use-goreplay-to-record-your-live-traffic/#3-1%E3%80%81%E4%B8%AD%E7%BB%A7%E6%9C%8D%E5%8A%A1%E5%99%A8 "3.1、中继服务器")3.1、中继服务器[](https://mritd.com/2021/08/03/use-goreplay-to-record-your-live-traffic/#3-1%E3%80%81%E4%B8%AD%E7%BB%A7%E6%9C%8D%E5%8A%A1%E5%99%A8)

GoReplay 可以使用中继服务器从而实现链式的流量传递，使用中继服务器时只需要将输出端设置为 TCP 模式，然后中继服务器输入端也设置为 TCP 模式即可:

```
# Run on servers where you want to catch traffic. You can run it on each `web` machine.
gor --input-raw :80 --output-tcp replay.local:28020

# Replay server (replay.local).
gor --input-tcp replay.local:28020 --output-http http://staging.com
```

如果有多个中继服务器，可以使用 `--split-output` 选项让每个抓取流量的 GoReplay 使用轮询算法向每个中继服务器发送流量:

```
gor --input-raw :80 --split-output --output-tcp replay1.local:28020 --output-tcp replay2.local:28020
```

### [](https://mritd.com/2021/08/03/use-goreplay-to-record-your-live-traffic/#3-2%E3%80%81%E8%BE%93%E5%87%BA%E5%88%B0-ElasticSearch "3.2、输出到 ElasticSearch")3.2、输出到 ElasticSearch[](https://mritd.com/2021/08/03/use-goreplay-to-record-your-live-traffic/#3-2%E3%80%81%E8%BE%93%E5%87%BA%E5%88%B0-ElasticSearch)

GoReplay 支持将输出端设置为 ElasticSearch:

```
./gor --input-raw :8000 --output-http http://staging.com --output-http-elasticsearch localhost:9200/gor
```

输出到 ES 时不需要预先创建索引，GoReplay 会自动完成，输出到 ES 后其数据结构如下:

```
type ESRequestResponse struct {
ReqURL               string `json:"Req_URL"`
ReqMethod            string `json:"Req_Method"`
ReqUserAgent         string `json:"Req_User-Agent"`
ReqAcceptLanguage    string `json:"Req_Accept-Language,omitempty"`
ReqAccept            string `json:"Req_Accept,omitempty"`
ReqAcceptEncoding    string `json:"Req_Accept-Encoding,omitempty"`
ReqIfModifiedSince   string `json:"Req_If-Modified-Since,omitempty"`
ReqConnection        string `json:"Req_Connection,omitempty"`
ReqCookies           string `json:"Req_Cookies,omitempty"`
RespStatus           string `json:"Resp_Status"`
RespStatusCode       string `json:"Resp_Status-Code"`
RespProto            string `json:"Resp_Proto,omitempty"`
RespContentLength    string `json:"Resp_Content-Length,omitempty"`
RespContentType      string `json:"Resp_Content-Type,omitempty"`
RespTransferEncoding string `json:"Resp_Transfer-Encoding,omitempty"`
RespContentEncoding  string `json:"Resp_Content-Encoding,omitempty"`
RespExpires          string `json:"Resp_Expires,omitempty"`
RespCacheControl     string `json:"Resp_Cache-Control,omitempty"`
RespVary             string `json:"Resp_Vary,omitempty"`
RespSetCookie        string `json:"Resp_Set-Cookie,omitempty"`
Rtt                  int64  `json:"RTT"`
Timestamp            time.Time
}
```

### [](https://mritd.com/2021/08/03/use-goreplay-to-record-your-live-traffic/#3-3%E3%80%81Kafka-%E5%AF%B9%E6%8E%A5 "3.3、Kafka 对接")3.3、Kafka 对接[](https://mritd.com/2021/08/03/use-goreplay-to-record-your-live-traffic/#3-3%E3%80%81Kafka-%E5%AF%B9%E6%8E%A5)

除了输出到 ES 以外，GoReplay 还支持输出到 Kafka 以及从 Kafka 中读取数据:

```
gor --input-raw :8080 --output-kafka-host '192.168.0.1:9092,192.168.0.2:9092' --output-kafka-topic 'kafka-log'

gor --input-kafka-host '192.168.0.1:9092,192.168.0.2:9092' --input-kafka-topic 'kafka-log' --output-stdout
```