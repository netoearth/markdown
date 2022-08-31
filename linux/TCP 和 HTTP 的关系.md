大家都知道 HTTP 的底层是 TCP，但是可能仅限于知道，并不是真正理解它们的关系。

平时我们用 chrome devtools 的 Network 工具也只是能分析 HTTP 请求：

![图片](https://mmbiz.qpic.cn/mmbiz_png/YprkEU0TtGhkiczVH5B7ViabSmY4l3WnWNqicW5nRtlqQ6rubZFtcoSHQkkqTF8mNcCda380bicjlUgohAVMuOl7rg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

TCP 层的东西看不见摸不着的，所以对它的理解也模模糊糊。

那怎么能看到 TCP 层的数据包来理清 TCP 和 HTTP 的关系呢？

这里推荐一个抓包工具 WireShark，它能抓取 TCP 层的包：

![图片](https://mmbiz.qpic.cn/mmbiz_png/YprkEU0TtGhkiczVH5B7ViabSmY4l3WnWNDXOcjISCfpbpibeU0pyrJkrjwVNej4J446QXykicCEqWgarQrTYBCicmg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

今天我们就用它来抓包分析下 TCP 和 HTTP 吧！

首先，我们准备这样一段服务端代码：

```
const express = require('express')const app = express()app.get('/', function (req, res) {  res.setHeader('Connection', 'close')  res.end('hello world');})app.listen(4000)
```

用 express 起了一个服务，监听 4000 端口，处理路径为 / 的 get 请求，返回 hello world 的响应体，并设置 Connection: close 的 header。

浏览器访问下：

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

header 和 body 都符合预期。

那 TCP 层都做了什么呢？

我们用 WireShark 抓包分析下：

打开 WireShark 后会看到有个设置按钮：

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

因为我们访问的是 localhost: 4000，所以这里选择本地回环地址那个虚拟网卡，并输入抓包过滤条件为 port 4000：

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

点击 start 开始录制，然后刷新一下浏览器：

这样就能看到抓到的 TCP 数据包：

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

我们一一分析下。

在分析之前需要了解一些 TCP 基础知识：

TCP 的头部是这样的：

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

TCP 是从端口到端口的传输协议，所以开始是源端口和目的端口。

接下来是序列号（sequence number），表示当前包的序号，后面是确认的序列号（acknowledgment number），表示我收到了序号为 xxx 的包。

然后红框标出的部分是 flags 标识位，通过 0、1 表示有没有：

这里我们只会用到其中的 SYN、ACK、FIN：

-   SYN：请求建立一个连接（说明这是链接的开始）
    
-   ACK：表示 ack number 是否是有效的
    
-   FIN：表示本端要断开链接了（说明这是链接的结束）
    

有了这些，我们就知道怎么区分 TCP 链接的开始和结束了。

再看一下抓到的包：

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

有 SYN 标志位的是连接的开始，有 FIN 标志位的是连接的结束，所以我们分为 3 段来看：

首先是连接开始的部分：

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

大家听过 TCP 的三次握手么？说的就是这个。

其中有一个端口是 4000，这个是服务的端口，那另一个端口 57454 明显就是浏览器的端口。

首先是浏览器向服务器发送了一个 SYN 的 TCP 请求，表示希望建立连接，序列号 Seq 是 0。

严格来说，序列号的相对值是 0，绝对值是 2454579144。

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

然后服务器向浏览器发送了一个 SYN 的 TCP 请求，表示希望建立连接，ACK 是 1，代表现在的 ack number 是有效的：

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

这里 ack number 的相对值是 1，绝对值是 2454579145，不就是上个 TCP 数据包的 seq 加 1 么？

TCP 连接中就是通过返回 seq number + 1 作为 ack number 来确认收到的。

然后又返回了一个 seq number 给浏览器，相对值是 0， 绝对值是 2765691269。

浏览器收到后返回了一个 TCP 数据包给服务器，ack number 自然是 2765691270，代表收到了连接请求。

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

这样浏览器和服务器各自向对方发送了 SYN 的建立连接请求，并且都收到了对方的确认，那么 TCP 连接就建立成功了。

这就是 TCP 三次握手的原理！

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

趁热打铁来看下四次挥手的部分：

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

浏览器向服务器发送了有 FIN 标志位的数据包，表示要断开连接，然后服务端返回了 ACK 的包表示确认。

之后服务端发送了 FIN 标志位的数据包给浏览器，表示要断开连接，浏览器也返回了 ACK 的包表示确认。

这样就完成了四次挥手的过程。

当然，具体确认的还是靠 ack number = seq number + 1 来实现的，和上面的一样，就不展开了：

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

我们通过抓包理清了 TCP 连接建立和连接的过程。

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

那么为什么握手是三次，挥手是四次呢？

因为挥手是一个 FIN，一个 ACK，一个 FIN + ACK，一个 ACK：

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

而握手是一个 SYN，一个 ACK + SYN，一个 ACK：

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

不过是因为握手时把 ACK 和 SYN 合并到一个数据包了而已。

那挥手时能合并成三次么？

不能！因为有两个 ack number，怎么合并，冲突了，而握手时只有一个 ack number，自然可以合并。

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

接下来再来看下连接建立后的 http 请求和响应吧：

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

其实一次 HTTP 请求响应会有四个 TCP 数据包，其中两个数据包与滑动窗口有关，这里先不展开了。

我们就看下 HTTP 的那两个包吧：

请求的 seq 是这样的：

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

而响应的 ack 是这样的：

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

相对值是 ack number = seq number + 1 没错，但是绝对值不是：

绝对值 2454579855 = 2454579145 + 710，也就是 ack number = seq number + segment len。

这些细节暂时不用深究。

总之，我们知道了**HTTP 的请求和响应是通过序列号关联在一起的。**

就算同一个 TCP 链接并行发送多个 HTTP 的请求和响应，它们也能找到各自对应的那个。就是通过这个 seq number 和 ack number。

这里为啥链接建立了发送了一个请求就断掉了呢？

我刷新浏览器，请求了两次，发现经历了两次连接的建立、http 请求响应、连接断开：

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

这是因为我设置了 Connection:close 的 header，它的作用就是一次 http 请求响应结束就断开 TCP 链接。

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

我们改成 HTTP 1.1 支持的 keep-alive 试试：

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

设置 Connection 为 keep-alive，然后设置 keep-alive 的细节为 timeout 10 ，也就是 10s 后断开。

重启服务器，再刷新下浏览器试试：

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

可以看到在一个 TCP 连接内发送了多次 http 请求响应。（通过 SYN 开始，FIN 结束）

这就是 keep-alive 的作用。

细心的同学会发现只是浏览器向服务器发送了 FIN 数据包，服务器没有发给浏览器 FIN 数据包。

这是因为 keep-alive 的 header 只是控制的浏览器的断开连接的行为，服务器的断开连接逻辑是独立的。

这样，我们就理清了 HTTP 在 TCP 层面的流程，连接的建立、断开，请求响应，还有 keep-alive。

## 总结

我们平时都是分析 HTTP 请求响应，TCP 对我们来说看不见摸不着的，理解的模模糊糊。

所以今天我们用 WireShark 抓了下 TCP 的包，来理清了 TCP 和 HTTP 的关系。

TCP 是从一个端口到另一个端口的传输控制协议，TCP header 中有序列号 seq number、确认序列号 ack number，还有几个标志位：

-   SYN 标志位代表请求建立连接
    
-   ACK 标志位代表当前确认序列号是有效的。
    
-   FIN 标志位代表请求断开连接
    

然后我们抓了 localhost:4000 的包分析了下 HTTP 请求的 TCP 流程，理清了三次握手（SYN、SYN + ACK、ACK），四次挥手（FIN、ACK、FIN + ACK、ACK）的连接建立、断开的流程。知道了为什么不能三次挥手（因为两个 ACK 冲突了）

然后还理清了同一个 TCP 连接传输的多个 HTTP 请求响应是通过 seq number 和 ack number 来关联的。

之后我们分别测试了 Connection：close 和 Connection：keep-alive 的情况，发现确实 keep-alive 能减少频繁的连接建立和断开，能复用同一个 TCP 链接。

HTTP 是通过 TCP 完成端口到端口的数据传输的。一个 TCP 连接可以传输多个 HTTP 请求、响应。请求和响应的关联是通过 TCP 包的序列号 seq。

理清了 TCP 和 HTTP 的关系，你是否对 HTTP 的理解更深了呢？