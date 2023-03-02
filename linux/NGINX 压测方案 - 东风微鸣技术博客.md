> [NGINX Sizing 指南: 如何进行测试](https://www.nginx.com/blog/nginx-plus-sizing-guide-how-we-tested/)

## 一 拓扑和硬件[](https://ewhisper.cn/posts/1151/#%E4%B8%80%20-%20%E6%8B%93%E6%89%91%E5%92%8C%E7%A1%AC%E4%BB%B6)

### 1.1 拓扑[](https://ewhisper.cn/posts/1151/#1-1-%20%E6%8B%93%E6%89%91)

所有测试均使用三个独立的机器完成，这几台机器通过双 40 GbE 链路连接在一个简单，平坦的 2 层网络中. (下图软件版本有更新)

[![用于 NGINX 性能测试的标准端到端拓扑结构](https://pic-cdn.ewhisper.cn/img/2021/09/26/16992bc6e0e1a2129636c0843e234e8b-nginx-plus-sizing-guide-topology.png)](https://pic-cdn.ewhisper.cn/img/2021/09/26/16992bc6e0e1a2129636c0843e234e8b-nginx-plus-sizing-guide-topology.png "用于 NGINX 性能测试的标准端到端拓扑结构")

为了从客户端机器生成流量，我们使用了 [`wrk`](https://github.com/wg/wrk) —— 类似于[`ab`](https://httpd.apache.org/docs/2.4/programs/ab.html)（ApacheBench）的性能测试工具。所有流量都被定向到 NGINX 反向代理服务器，后者将连接转发到 Web 服务器后端。

### 1.2 硬件[](https://ewhisper.cn/posts/1151/#1-2-%20%E7%A1%AC%E4%BB%B6)

| 机器 | CPU | 网络 | 内存 |
| --- | --- | --- | --- |
| 客户端 | 16 real cores(非超线程) | 略 | 16GB |
| 反向代理服务器 | 16 real cores(非超线程) | 略 | 16GB |
| 应用服务器 | 16 real cores(非超线程) | 略 | 16GB |

## 二 使用的软件[](https://ewhisper.cn/posts/1151/#%E4%BA%8C%20-%20%E4%BD%BF%E7%94%A8%E7%9A%84%E8%BD%AF%E4%BB%B6)

-   [`wrk`](https://github.com/wg/wrk) 在客户端计算机上运行 4.0.0 版生成了 NGINX 代理的流量。我们按照这些 [说明](https://github.com/wg/wrk/wiki/Installing-Wrk-on-Linux) 安装了它。

## 三 测试方法[](https://ewhisper.cn/posts/1151/#%E4%B8%89%20-%20%E6%B5%8B%E8%AF%95%E6%96%B9%E6%B3%95)

测试了 NGINX 作为具有不同 CPU 数量的反向代理服务器的性能。一个 NGINX `worker` 进程消耗一个 CPU，因此为了测量不同数量的 CPU 的性能，我们改变了 NGINX `worker` 进程的数量，用两个, 四个，八个等等 `worker` 进程重复测试。

## 四 性能指标[](https://ewhisper.cn/posts/1151/#%E5%9B%9B%20-%20%E6%80%A7%E8%83%BD%E6%8C%87%E6%A0%87)

测量以下性能指标:

-   **每秒请求数（RPS）** - 衡量处理 HTTP 请求的能力。在测试中，每个客户端通过 keepalive 连接发送 1 KB 大小文件的请求。反向代理服务器处理每个请求，并通过另一个 keepalive 连接将其转发到 Web 服务器。
-   **每秒 SSL/TLS 事务数（TPS）** - 衡量处理新 SSL / TLS 连接的能力。在测试中，每个客户端都会发送一系列 HTTPS 请求，每个请求都在一个新连接上。反向代理服务器解析请求并通过已建立的 keepalive 连接将它们转发到 Web 服务器。Web 服务器为每个请求发回 0 字节的响应。
-   **吞吐量** - 测量 NGINX 在通过 HTTP 提供 1 MB 文件时可以承受的吞吐量。

## 五 运行测试[](https://ewhisper.cn/posts/1151/#%E4%BA%94%20-%20%E8%BF%90%E8%A1%8C%E6%B5%8B%E8%AF%95)

要生成所有客户端流量，我们使用 `wrk` 工具的以下选项：

-   `-c` 选项指定要创建的 TCP 连接数。对于我们的测试，我们将其设置为 50 个连接。
-   `-d` 选项指定生成流量的时间。我们每次运行测试 3 分钟。
-   `-t` 选项指定要创建的线程数。我们指定了一个线程。

通过 `taskset` 指令, 充分使用每个 CPU ，可以将单个 `wrk` 进程固定到 CPU。与增加 `wrk` 线程数相比，此方法产生更一致的结果。

### 5.1 每秒请求数 (Requests Per Second)[](https://ewhisper.cn/posts/1151/#5-1-%20%E6%AF%8F%E7%A7%92%E8%AF%B7%E6%B1%82%E6%95%B0%20-Requests-Per-Second)

为了测量每秒请求数（RPS），运行以下脚本：

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br></pre></td><td><pre><code><span>for</span> i <span>in</span> `seq 1 number-of-CPUs`; <span>do</span><br>    taskset -c <span>$i</span> wrk -t 1 -c 50 -d 180s http://Reverse-Proxy-Server-IP-address/1kb.bin<br><span>done</span><br></code><p><i></i>BASH</p></pre></td></tr></tbody></table>

该测试每个 CPU 产生了 `wrk` 的一个副本。每个副本创建 50 个 TCP 连接，并在其上连续请求 1 KB 文件 3 分钟（180 秒）。

### 5.2 每秒 SSL/TLS 事务数[](https://ewhisper.cn/posts/1151/#5-2-%20%E6%AF%8F%E7%A7%92%20-SSL-TLS-%20%E4%BA%8B%E5%8A%A1%E6%95%B0)

为了测量每秒 SSL/TLS 事务（TLS），我们运行了以下脚本：

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br></pre></td><td><pre><code><span>for</span> i <span>in</span> `seq 1 number-of-CPUs`; <span>do</span><br>    taskset -c <span>$i</span> wrk -t 1 -c 50 -d 180s -H <span>'Connection: close'</span> https://Reverse-Proxy-Server-IP-address/0kb.bin<br><span>done</span><br></code><p><i></i>BASH</p></pre></td></tr></tbody></table>

该测试使用和先前的测试相同的 `-c`，`-d` 和 `-t`值，但有两个显着的不同，因为本次焦点在于 SSL/TLS 连接的处理：

-   客户端为每个请求打开和关闭连接（`-H` 选项设置 `Connection:` `close` HTTP 标头）。
-   请求的文件是 0 字节而不是 1 KB 的大小。

### 5.3 吞吐量[](https://ewhisper.cn/posts/1151/#5-3-%20%E5%90%9E%E5%90%90%E9%87%8F)

为了测量吞吐量，我们运行了以下脚本：

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br></pre></td><td><pre><code><span>for</span> i <span>in</span> `seq 1 number-of-CPUs`; <span>do</span><br>    taskset -c <span>$i</span> wrk -t 1 -c 50 -d 180s http://Reverse-Proxy-Server-IP-address/1mb.bin<br><span>done</span><br></code><p><i></i>BASH</p></pre></td></tr></tbody></table>

与第一个测试的唯一区别是文件更大, 大小为 1 MB。我们发现使用 10 MB 的较大文件大小并未提高整体吞吐量。

> **在 Linux 上安装 wrk**
> 
> wrk 需要 `openssl dev` 包和 `gcc/dev` 堆栈。
> 
> 以下是关于如何在 Linux 上安装 wrk 的简要说明。
> 
> **CentOS/RedHat/Fedora**
> 
> <table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br></pre></td><td><pre><code>sudo yum groupinstall <span>'Development Tools'</span><br>sudo yum install -y openssl-devel git <br>git <span>clone</span> https://github.com/wg/wrk.git wrk<br><span>cd</span> wrk<br>make<br>＃将可执行文件移动到 PATH 中的某个位置<br>sudo cp wrk /somewhere/<span>in</span>/your/PATH<br></code><p><i></i>BASH</p></pre></td></tr></tbody></table>
> 
> 安装构建工具，openssl dev libs（包括头文件）和 git。然后使用 git 下载 wrk 并构建。