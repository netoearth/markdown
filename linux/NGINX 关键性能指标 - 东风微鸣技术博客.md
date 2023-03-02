## RPS[](https://ewhisper.cn/posts/56821/#RPS)

### RPS (Requests Per Second) for HTTP Requests[](https://ewhisper.cn/posts/56821/#RPS-Requests-Per-Second-for-HTTP-Requests)

下表和图形显示了不同 CPU 数量和不同请求大小的 HTTP 请求数量，以千字节（KB）为单位。

| CPUs | 0 KB | 1 KB | 10 KB | 100 KB |
| --- | --- | --- | --- | --- |
| 1 | 145,551 | 74,091 | 54,684 | 33,125 |
| 2 | 249,293 | 131,466 | 102,069 | 62,554 |
| 4 | 543,061 | 261,269 | 207,848 | 88,691 |
| 8 | 1,048,421 | 524,745 | 392,151 | 91,640 |
| 16 | 2,001,846 | 972,382 | 663,921 | 91,623 |
| 32 | 3,019,182 | 1,316,362 | 774,567 | 91,640 |
| 36 | 3,298,511 | 1,309,358 | 764,744 | 91,655 |

[![NGINX HTTP RPS](http://pic-cdn.ewhisper.cn/img/2021/01/14/c7b0f9f5286accc2bf0c26d61a197c61-NGINX-HTTP-RPS.png)](http://pic-cdn.ewhisper.cn/img/2021/01/14/c7b0f9f5286accc2bf0c26d61a197c61-NGINX-HTTP-RPS.png "NGINX HTTP RPS")

### RPS for HTTPS Requests[](https://ewhisper.cn/posts/56821/#RPS-for-HTTPS-Requests)

对于相同的已配置裸机硬件，HTTPS RPS 会低于 HTTP RPS，因为保护机器之间传输的数据所需的数据加密和解密在计算上非常昂贵。

尽管如此，英特尔架构的持续发展 —— 导致服务器具有更快的处理器和更好的内存管理 —— 意味着与专用硬件加密设备相比，用于 CPU 绑定加密任务的软件性能不断提高。

| CPUs | 0 KB | 1 KB | 10 KB | 100 KB |
| --- | --- | --- | --- | --- |
| 1 | 71,561 | 40,207 | 23,308 | 4,830 |
| 2 | 151,325 | 85,139 | 48,654 | 9,871 |
| 4 | 324,654 | 178,395 | 96,808 | 19,355 |
| 8 | 647,213 | 359,576 | 198,818 | 38,900 |
| 16 | 1,262,999 | 690,329 | 383,860 | 77,427 |
| 32 | 2,197,336 | 1,207,959 | 692,804 | 90,430 |
| 36 | 2,175,945 | 1,239,624 | 733,745 | 89,842 |

## Connections per Second[](https://ewhisper.cn/posts/56821/#Connections-per-Second)

每秒连接数（CPS）衡量 NGINX 创建新 TCP 连接到已发出请求的客户端的能力。 客户端发送一系列 HTTP 或 HTTPS 请求，每个请求都建立在新的连接上。 NGINX 解析请求，并为每个请求发送回 0 字节的响应。 满足请求后，连接将关闭。

ℹ️ **备注：**

此测试的 HTTPS 变体通常称为每秒 SSL 事务（SSLTPS）。

### CPS for HTTP Requests[](https://ewhisper.cn/posts/56821/#CPS-for-HTTP-Requests)

该表和图形显示了跨不同 CPU 的 HTTP 请求的 CPS。

| CPUs | CPS |
| --- | --- |
| 1 | 34,344 |
| 2 | 54,368 |
| 4 | 123,164 |
| 8 | 194,967 |
| 16 | 255,032 |
| 32 | 261,033 |
| 36 | 257,277 |

[![HTTP CPS](http://pic-cdn.ewhisper.cn/img/2021/01/14/b600b200534ae9193b8d87170ae3a10b-HTTP-CPS.png)](http://pic-cdn.ewhisper.cn/img/2021/01/14/b600b200534ae9193b8d87170ae3a10b-HTTP-CPS.png "HTTP CPS")

### CPS for HTTPS Requests[](https://ewhisper.cn/posts/56821/#CPS-for-HTTPS-Requests)

表格和图形显示了 HTTPS 请求的 CPS。

| **CPUs** | **CPS** |
| --- | --- |
| 1 | 428 |
| 2 | 869 |
| 4 | 1,735 |
| 8 | 3,399 |
| 16 | 6,676 |
| 24 | 10,274 |
| 36 | 10,067 |

[![SSL TPS](http://pic-cdn.ewhisper.cn/img/2021/01/14/f729934635caf76f58f488f855e35bc1-SSL-TPS.png)](http://pic-cdn.ewhisper.cn/img/2021/01/14/f729934635caf76f58f488f855e35bc1-SSL-TPS.png "SSL TPS")

## Throughput[](https://ewhisper.cn/posts/56821/#Throughput)

这些测试测量 NGINX 能够维持 180 秒的 HTTP 请求的吞吐量（以 Gbps 为单位）。

| CPUs | 100 KB | 1 MB | 10 MB |
| --- | --- | --- | --- |
| 1 | 13 | 48 | 68 |
| 2 | 20 | 69 | 71 |
| 4 | 45 | 67 | 71 |
| 8 | 50 | 68 | 72 |
| 16 | 48 | 66 | 71 |
| 32 | 48 | 66 | 71 |
| 36 | 48 | 66 | 71 |

[![NGINX 吞吐量](http://pic-cdn.ewhisper.cn/img/2021/01/14/1e52d73a24fbeff1b3994a2f6713886a-Throughput.png)](http://pic-cdn.ewhisper.cn/img/2021/01/14/1e52d73a24fbeff1b3994a2f6713886a-Throughput.png "NGINX 吞吐量")

吞吐量与客户端计算机发出的 HTTP 请求的大小成正比。 当文件较大时，NGINX 将获得更高的吞吐量，因为给定请求会导致传输更多数据。 但是，性能达到约 8 个 CPU 的峰值; 对于吞吐量较大的任务，不一定有更多好处。

## 杂记[](https://ewhisper.cn/posts/56821/#%E6%9D%82%E8%AE%B0)

-   在我们测试的 CPU 上可以使用超线程，这意味着可以运行其他 NGINX 工作进程以使用超线程 CPU 的全部容量。 我们没有为此处报告的测试启用超线程，但是我们确实看到在单独的测试中使用超线程可以提高性能。 最值得注意的是，超线程将 SSL TPS 提升了约 50％。
-   此处显示的数字与 OpenSSL 1.0.1 有关。 我们还对 OpenSSL 1.0.2 进行了测试，发现性能提高了 2 倍。 OpenSSL 1.0.1 仍然被广泛使用，但我们建议使用 OpenSSL 1.0.2，以提高安全性和性能。
-   我们还测试了椭圆曲线加密（ECC），但此处显示的结果使用 RSA。 对于加密，尽管 ECC 通常部署在需要提高效率的移动设备上，但 RSA 仍比 ECC 广泛使用。 与标准 RSA 证书相比，我们发现 ECC 的性能提高了 2 到 3 倍，我们建议您考虑实施 ECC。

移至 OpenSSL 1.0.2 和移至 ECC 的结合可能会带来非常强大的性能改进。 此外，我们的结果表明，如果您当前使用 4‑CPU 或 8‑CPU 服务器，则将 16 个 CPU 甚至 32 个 CPU 用于 SSL 可能会带来真正的显着改善。