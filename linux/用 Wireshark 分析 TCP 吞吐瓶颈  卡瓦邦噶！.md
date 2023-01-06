Debug 网络质量的时候，我们一般会关注两个因素：延迟和吞吐量（带宽）。延迟比较好验证，Ping 一下或者 [mtr](https://www.kawabangga.com/posts/4275) 一下就能看出来。这篇文章分享一个 debug 吞吐量的办法。

看重吞吐量的场景一般是所谓的长肥管道(Long Fat Networks, LFN, [rfc7323](https://datatracker.ietf.org/doc/html/rfc7323)). 比如下载大文件。吞吐量没有达到网络的上限，主要可能受 3 个方面的影响：

1.  发送端出现了瓶颈
2.  接收端出现了瓶颈
3.  中间的网络层出现了瓶颈

发送端出现瓶颈一般的情况是 buffer 不够大，因为发送的过程是，应用调用 syscall，将要发送的数据放到 buffer 里面，然后由系统负责发送出去。如果 buffer 满了，那么应用会阻塞住（如果使用 block 的 API 的话），直到 buffer 可用了再继续 write，生产者和消费者模式。

发送端出现瓶颈一般都比较好排查，甚至通过应用的日志看何时阻塞住了即可。大部分情况都是第 2，3 种情况，比较难以排查。这种情况发生在，发送端的应用已经将内容写入到了系统的 buffer 中，但是系统并没有很快的发送出去。

TCP 为了优化传输效率（注意这里的传输效率，并不是单纯某一个 TCP 连接的传输效率，而是整体网络的效率），会:

1.  保护接收端，发送的数据不会超过接收端的 buffer 大小 (Flow control)。数据发送到接受端，也是和上面介绍的过程类似，kernel 先负责收好包放到 buffer 中，然后上层应用程序处理这个 buffer 中的内容，如果接收端的 buffer 过小，那么很容易出现瓶颈，即应用程序还没来得及处理就被填满了。那么如果数据继续发过来，buffer 存不下，接收端只能丢弃。
2.  保护网络，发送的数据不会 overwhelming 网络 (Congestion Control, 拥塞控制), 如果中间的网络出现瓶颈，会导致长肥管道的吞吐不理想；

对于接收端的保护，在两边连接建立的时候，会协商好接收端的 buffer 大小 (receiver window size, rwnd), 并且在后续的发送中，接收端也会在每一个 ack 回包中报告自己剩余和接受的 window 大小。这样，发送端在发送的时候会保证不会发送超过接收端 buffer 大小的数据。（意思是，发送端需要负责，receiver 没有 ack 的总数，不会超过 receiver 的 buffer.）

对于网络的保护，原理也是维护一个 Window，叫做 Congestion window，拥塞窗口，cwnd, 这个窗口就是当前网络的限制，发送端不会发送超过这个窗口的容量（没有 ack 的总数不会超过 cwnd）。

怎么找到这个 cwnd 的值呢？

这个就是关键了，默认的算法是 cubic, 也有其他算法可以使用，比如 Google 的 [BBR](https://www.kawabangga.com/posts/4086).

主要的逻辑是，慢启动(Slow start), 发送数据来测试，如果能正确收到 receiver 那边的 ack，说明当前网络能容纳这个吞吐，将 cwnd x 2，然后继续测试。直到下面一种情况发生：

1.  发送的包没有收到 ACK
2.  cwnd 已经等于 rwnd 了

第 2 点很好理解，说明网络吞吐并不是一个瓶颈，瓶颈是在接收端的 buffer 不够大。cwnd 不能超过 rwnd，不然会 overload 接收端。

对于第 1 点，本质上，发送端是用丢包来检测网络状况的，如果没有发生丢包，表示一切正常，如果发生丢包，说明网络处理不了这个发送速度，这时候发送端会直接将 cwnd 减半。

但实际造成第 1 点的情况并不一定是网络吞吐瓶颈，而可能是以下几种情况：

1.  网络达到了瓶颈
2.  网络质量问题丢包
3.  中间网络设备延迟了包的送达，导致发送端没有在预期时间内收到 ACK

2 和 3 原因都会造成 cwnd 下降，无法充分利用网络吞吐。

以上就是基本的原理，下面介绍如何定位这种问题。

## rwnd 查看方式

这个 window size 直接就在 TCP header 里面，抓下来就能看这个字段。

[![](https://www.kawabangga.com/wp-content/uploads/2022/08/tcp-handshake-factor.png)](https://www.kawabangga.com/wp-content/uploads/2022/08/tcp-handshake-factor.png)

但是真正的 window size 需要乘以 factor, factor 是在 [TCP 握手节点通过 TCP Options 协商的](https://en.wikipedia.org/wiki/TCP_window_scale_option)。所以如果分析一条 TCP 连接的 window size，必须抓到握手阶段的包，不然就不可以知道协商的 factor 是多少。

[![](https://www.kawabangga.com/wp-content/uploads/2022/08/tcp-rwnd-size.png)](https://www.kawabangga.com/wp-content/uploads/2022/08/tcp-rwnd-size.png)

## cwnd 查看方式

Congestion control 是发送端通过算法得到的一个动态变量，会实时调整，并不会体现在协议的传输数据中。所以要看这个，必须在发送端的机器上看。

在 Linux 中可以使用 `ss -i` 选项将 TCP 连接的参数都打印出来。

[![](https://www.kawabangga.com/wp-content/uploads/2022/08/tcp-cwnd-ss.png)](https://www.kawabangga.com/wp-content/uploads/2022/08/tcp-cwnd-ss.png)

这里展示的单位是 [TCP MSS.](https://www.cloudflare.com/learning/network-layer/what-is-mss/) 即实际大小是 1460bytes \* 10.

Wireshark 提供了非常实用的统计功能，可以让你一眼就能看出当前的瓶颈是发生在了哪里。但是第一次打开这个图我不会看，一脸懵逼，也没查到资料要怎么看。好在我[同事](https://github.com/royzhr)会，他把我教会了，我在这里记录一下，把你也教会。

首先，打开的方式如下：

[![](https://www.kawabangga.com/wp-content/uploads/2022/08/wireshark-open-tcptrace.png)](https://www.kawabangga.com/wp-content/uploads/2022/08/wireshark-open-tcptrace.png)

然后你会看到如下的图。

[![](https://www.kawabangga.com/wp-content/uploads/2022/08/wireshark-tcptrace-howtoread.png)](https://www.kawabangga.com/wp-content/uploads/2022/08/wireshark-tcptrace-howtoread.png)

首先需要明确，tcptrace 的图表示的是单方向的数据发送，因为 tcp 是双工协议，两边都能发送数据。其中最上面写了你当前在看的图数据是从 10.0.0.1 发送到 192.168.0.1 的，然后按右下角的按钮可以切换看的方向。

X轴表示的是时间，很好理解。

然后理解一下 Y 轴表示的 Sequence Number, 就是 TCP 包中的 Sequence Number，这个很关键。图中所有的数据，都是以 Sequence Number 为准的。

所以，你如果看到如上图所示，那么说明你看反了，因为数据的 Sequence Number 并没有增加过，说明几乎没有发送过数据，需要点击 Switch Direction。

[![](https://www.kawabangga.com/wp-content/uploads/2022/08/wireshark-real-dataflow.png)](https://www.kawabangga.com/wp-content/uploads/2022/08/wireshark-real-dataflow.png)

这就对了，可以看到我们传输的 Sequence Number 在随着时间增加而增加。

这里面有 3 条线，含义如下：

[![](https://www.kawabangga.com/wp-content/uploads/2022/08/wireshark-tcptrace-read.png)](https://www.kawabangga.com/wp-content/uploads/2022/08/wireshark-tcptrace-read.png)

除此之外，另外还有两种线：

[![](https://www.kawabangga.com/wp-content/uploads/2022/08/tcptrace-sack.png)](https://www.kawabangga.com/wp-content/uploads/2022/08/tcptrace-sack.png)

需要始终记住的是 Y 轴是 Sequence Number，红色的线表示 SACK 的线表示这一段 Sequence Number 我已经收到了，然后配合黄色线表示 ACK 过的 Sequence Number，那么发送端就会知道，在中间这段空挡，包丢了，红色线和黄色线纵向的空白，是没有被 ACK 的包。所以，需要重新传输。而蓝色的线就是表示又重新传输了一遍。

学会了看这些图，我们可以认识几种常见的 pattern：

### 丢包

[![](https://www.kawabangga.com/wp-content/uploads/2022/08/wireshark-packet-loss.png)](https://www.kawabangga.com/wp-content/uploads/2022/08/wireshark-packet-loss.png)

很多红色 SACK，说明接收端那边重复在说：中间有一个包我没有收到，中间有一个包我没有收到。

### 吞吐受到接收 window size 限制

[![](https://www.kawabangga.com/wp-content/uploads/2022/08/rwnd-too-low.png)](https://www.kawabangga.com/wp-content/uploads/2022/08/rwnd-too-low.png)

从这个图可以看出，黄色的线（接收端一 ACK）一上升，蓝色就跟着上升（发送端就开始发），直到填满绿色的线（window size）。说明网络并不是瓶颈，可以调大接收端的 buffer size.

### 吞吐受到网络质量限制

[![](https://www.kawabangga.com/wp-content/uploads/2022/08/wireshark-cwnd-low.png)](https://www.kawabangga.com/wp-content/uploads/2022/08/wireshark-cwnd-low.png)

从这张图中可以看出，接收端的 window size 远远不是瓶颈，还有很多空闲。

[![](https://www.kawabangga.com/wp-content/uploads/2022/08/wireshark-cwnd-low-zoom.png)](https://www.kawabangga.com/wp-content/uploads/2022/08/wireshark-cwnd-low-zoom.png)

放大可以看出，中间有很多丢包和重传，并且每次只发送一点点数据，这说明很有可能是 cwnd 太小了，受到了拥塞控制算法的限制。

本文中用到的抓包文件可以从这里下载(credit: [https://www.youtube.com/watch?v=yUmACeSmT7o](https://www.youtube.com/watch?v=yUmACeSmT7o)):

1.  [https://www.cloudshark.org/captures/f5eb7c033728](https://www.cloudshark.org/captures/f5eb7c033728)
2.  [https://www.cloudshark.org/captures/c967765aef38](https://www.cloudshark.org/captures/c967765aef38)

其他的一些参考资料：

1.  [https://www.stackpath.com/edge-academy/what-is-cwnd-and-rwnd/](https://www.stackpath.com/edge-academy/what-is-cwnd-and-rwnd/)
2.  [https://www.baeldung.com/cs/tcp-flow-control-vs-congestion-control](https://www.baeldung.com/cs/tcp-flow-control-vs-congestion-control)
3.  [https://www.cs.cornell.edu/courses/cs4450/2020sp/lecture21-congestion-control.pdf](https://www.cs.cornell.edu/courses/cs4450/2020sp/lecture21-congestion-control.pdf)
4.  [https://www.mi.fu-berlin.de/inf/groups/ag-tech/teaching/2011-12\_WS/L\_19531\_Telematics/08\_Transport\_Layer.pdf](https://www.mi.fu-berlin.de/inf/groups/ag-tech/teaching/2011-12_WS/L_19531_Telematics/08_Transport_Layer.pdf)
5.  [https://wiki.aalto.fi/download/attachments/69901948/TCP-CongestionControlFinal.pdf](https://wiki.aalto.fi/download/attachments/69901948/TCP-CongestionControlFinal.pdf)
6.  [https://paulgrevink.wordpress.com/2017/09/08/about-long-fat-networks-and-tcp-tuning/](https://paulgrevink.wordpress.com/2017/09/08/about-long-fat-networks-and-tcp-tuning/)

帮这位老师的项目打一个广告：[https://github.com/royzhr/spate](https://github.com/royzhr/spate) 一个大流量网络性能测试工具。