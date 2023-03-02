原文：http://blog.163.com/hlz\_2599/blog/static/142378474201341341339314/

参考：http://man.linuxde.net/iperf

Iperf是一个网络性能测试工具。Iperf可以测试TCP和UDP带宽质量。Iperf可以测量最大TCP带宽，具有多种参数和UDP特性。 Iperf可以报告带宽，延迟抖动和数据包丢失。利用Iperf这一特性，可以用来测试一些网络设备如路由器，防火墙，交换机等的性能。

Iperf有两种版本，windows版和linux版本。

linux版本更新快，最新版本为iperf 3.0，下载地址为[http://code.google.com/p/iperf/downloads/list](http://code.google.com/p/iperf/downloads/list) ，

windows版本官方更新的最新版本为1.7（打包在jperf中），下载地址为：[http://sourceforge.net/projects/iperf/files/jperf/jperf%202.0.0/](http://sourceforge.net/projects/iperf/files/jperf/jperf%202.0.0/) ，

不过在网络上找到了移植版本iperf2.5。Iperf还有一个图形界面程序叫做Jperf，使用JPerf程序能简化了复杂命令行参数的构造，而且 它还保存测试结果,同时实时图形化显示结果。当然，JPerf可以测试TCP和UDP带宽质量。JPerf可以测量最大TCP带宽，具有多种参数和UDP 特性。JPerf可以报告带宽，延迟抖动和数据包丢失。为了测试的准确性，尽量使用linux环境测试。

Iperf的主要功能如下：

## TCP

-   测量网络带宽
    
-   报告MSS/MTU值的大小和观测值
    
-   支持TCP窗口值通过套接字缓冲
    
-   当P线程或Win32线程可用时，支持多线程。客户端与服务端支持同时多重连接
    

## UDP

-   客户端可以创建指定带宽的UDP流
    
-   测量丢包
    
-   测量延迟
    
-   支持多播
    
-   当P线程可用时，支持多线程。客户端与服务端支持同时多重连接（不支持Windows）
    

## 其他

-   在适当的地方，选项中可以使用K（kilo-）和M（mega-）。例如131072字节可以用128K代替。
    
-   可以指定运行的总时间，甚至可以设置传输的数据总量。
    
-   在报告中，为数据选用最合适的单位。
    
-   服务器支持多重连接，而不是等待一个单线程测试。
    
-   在指定时间间隔重复显示网络带宽，波动和丢包情况。
    
-   服务器端可作为后台程序运行。
    
-   服务器端可作为Windows 服务运行。
    
-   使用典型数据流来测试链接层压缩对于可用带宽的影响。
    
-   支持传送指定文件，可以定性和定量测试
    

## Iperf使用方法

1.  安装Iperf
    
    1.  对于windows版的Iperf，直接将解压出来的iperf.exe和cygwin1.dll复制到%systemroot%目录即可
        
    2.  对于linux版的Iperf，请使用如下命令安装
        
        gunzip -c iperf-<version>.tar.gz | tar -xvf -
        
        cd iperf-<version>
        
        ./configure
        
        make
        
        make install
        
2.  使用Iperf（以windows版本为例）
    
    在命令提示符中输入iperf命令即可运行Iperf，使用命令Iperf –help可以查看iperf的帮助
    
    ![](http://img0.ph.126.net/NlrsFWqdAIaSIDcRZcnGgA==/3171378562699005273.jpg)
    
3.  Iperf参数介绍
    
    <table><colgroup><col> <col></colgroup><tbody><tr><td><p><span><strong>命令行选项</strong></span></p></td><td><p><span><strong>描述</strong></span></p></td></tr><tr><td colspan="2"><p><span>客户端与服务器共用选项</span></p></td></tr><tr><td><p><span>-f, --format [bkmaBKMA]</span></p></td><td><p><span>格式化带宽数输出。支持的格式有：<br>'b' = bits/sec 'B' = Bytes/sec<br>'k' = Kbits/sec 'K' = KBytes/sec<br>'m' = Mbits/sec 'M' = MBytes/sec<br>'g' = Gbits/sec 'G' = GBytes/sec<br>'a' = adaptive bits/sec 'A' = adaptive Bytes/sec<br>自适应格式是kilo-和mega-二者之一。除了带宽之外的字段都输出为字节，除非指定输出的格式，默认的参数是a。<br>注 意：在计算字节byte时，Kilo = 1024， Mega = 1024^2，Giga = 1024^3。通常，在网络中，Kilo = 1000， Mega = 1000^2， and Giga = 1000^3，所以，Iperf也按此来计算比特（位）。如果这些困扰了你，那么请使用-f b参数，然后亲自计算一下。</span></p></td></tr><tr><td><p><span>-i, --interval #</span></p></td><td><p><span>设置每次报告之间的时间间隔，单位为秒。如果设置为非零值，就会按照此时间间隔输出测试报告。默认值为零。</span></p></td></tr><tr><td><p><span>-l, --len #[KM]</span></p></td><td><p><span>设置读写缓冲区的长度。TCP方式默认为8KB，UDP方式默认为1470字节。</span></p></td></tr><tr><td><p><span>-m, --print_mss</span></p></td><td><p><span>输出TCP MSS值（通过TCP_MAXSEG支持）。MSS值一般比MTU值小40字节。通常情况</span></p></td></tr><tr><td><p><span>-p, --port #</span></p></td><td><p><span>设置端口，与服务器端的监听端口一致。默认是5001端口，与ttcp的一样。</span></p></td></tr><tr><td><p><span>-u, --udp</span></p></td><td><p><span>使用UDP方式而不是TCP方式。参看-b选项。</span></p></td></tr><tr><td><p><span>-w, --window #[KM]</span></p></td><td><p><span>设置套接字缓冲区为指定大小。对于TCP方式，此设置为TCP窗口大小。对于UDP方式，此设置为接受UDP数据包的缓冲区大小，限制可以接受数据包的最大值。</span></p></td></tr><tr><td><p><span>-B, --bind host</span></p></td><td><p><span>绑定到主机的多个地址中的一个。对于客户端来 说，这个参数设置了出栈接口。对于服务器端来说，这个参数设置入栈接口。这个参数只用于具有多网络接口的主机。在Iperf的UDP模式下，此参数用于绑 定和加入一个多播组。使用范围在224.0.0.0至239.255.255.255的多播地址。参考-T参数。</span></p></td></tr><tr><td><p><span>-C, --compatibility</span></p></td><td><p><span>与低版本的Iperf使用时，可以使用兼容模式。不需要两端同时使用兼容模式，但是强烈推荐两端同时使用兼容模式。某些情况下，使用某些数据流可以引起1.7版本的服务器端崩溃或引起非预期的连接尝试。</span></p></td></tr><tr><td><p><span>-M, --mss #[KM}</span></p></td><td><p><span>通过TCP_MAXSEG选项尝试设置TCP最大信息段的值。MSS值的大小通常是TCP/IP头减去40字节。在以太网中，MSS值 为1460字节（MTU1500字节）。许多操作系统不支持此选项。</span></p></td></tr><tr><td><p><span>-N, --nodelay</span></p></td><td><p><span>设置TCP无延迟选项，禁用Nagle's运算法则。通常情况此选项对于交互程序，例如telnet，是禁用的。</span></p></td></tr><tr><td><p><span>-V (from v1.6 or higher)</span></p></td><td><p><span>绑定一个IPv6地址。<br>服务端：$ iperf -s –V<br>客户端：$ iperf -c &lt;Server IPv6 Address&gt; -V<br>注意：在1.6.3或更高版本中，指定IPv6地址不需要使用-B参数绑定，在1.6之前的版本则需要。在大多数操作系统中，将响应IPv4客户端映射的IPv4地址。</span></p></td></tr><tr><td colspan="2"><p><span>服务器端专用选项</span></p></td></tr><tr><td><p><span>-s, --server</span></p></td><td><p><span>Iperf服务器模式</span></p></td></tr><tr><td><p><span>-D (v1.2或更高版本)</span></p></td><td><p><span>Unix平台下Iperf作为后台守护进程运行。在Win32平台下，Iperf将作为服务运行。</span></p></td></tr><tr><td><p><span>-R(v1.2或更高版本，仅用于Windows)</span></p></td><td><p><span>卸载Iperf服务（如果它在运行）。</span></p></td></tr><tr><td><p><span>-o(v1.2或更高版本，仅用于Windows)</span></p></td><td><p><span>重定向输出到指定文件</span></p></td></tr><tr><td><p><span>-c, --client host</span></p></td><td><p><span>如果Iperf运行在服务器模式，并且用-c参数指定一个主机，那么Iperf将只接受指定主机的连接。此参数不能工作于UDP模式。</span></p></td></tr><tr><td><p><span>-P, --parallel #</span></p></td><td><p><span>服务器关闭之前保持的连接数。默认是0，这意味着永远接受连接。</span></p></td></tr><tr><td colspan="2"><p><span>客户端专用选项</span></p></td></tr><tr><td><p><span>-b, --bandwidth #[KM]</span></p></td><td><p><span>UDP模式使用的带宽，单位bits/sec。此选项与-u选项相关。默认值是1 Mbit/sec。</span></p></td></tr><tr><td><p><span>-c, --client host</span></p></td><td><p><span>运行Iperf的客户端模式，连接到指定的Iperf服务器端。</span></p></td></tr><tr><td><p><span>-d, --dualtest</span></p></td><td><p><span>运行双测试模式。这将使服务器端反向连接到客户端，使用-L 参数中指定的端口（或默认使用客户端连接到服务器端的端口）。这些在操作的同时就立即完成了。如果你想要一个交互的测试，请尝试-r参数。</span></p></td></tr><tr><td><p><span>-n, --num #[KM]</span></p></td><td><p><span>传送的缓冲器数量。通常情况，Iperf按照10秒钟发送数据。-n参数跨越此限制，按照指定次数发送指定长度的数据，而不论该操作耗费多少时间。参考-l与-t选项。</span></p></td></tr><tr><td><p><span>-r, --tradeoff</span></p></td><td><p><span>往复测试模式。当客户端到服务器端的测试结束时，服务器端通过-l选项指定的端口（或默认为客户端连接到服务器端的端口），反向连接至客户端。当客户端连接终止时，反向连接随即开始。如果需要同时进行双向测试，请尝试-d参数。</span></p></td></tr><tr><td><p><span>-t, --time #</span></p></td><td><p><span>设置传输的总时间。Iperf在指定的时间内，重复的发送指定长度的数据包。默认是10秒钟。参考-l与-n选项。</span></p></td></tr><tr><td><p><span>-L, --listenport #</span></p></td><td><p><span>指定服务端反向连接到客户端时使用的端口。默认使用客户端连接至服务端的端口。</span></p></td></tr><tr><td><p><span>-P, --parallel #</span></p></td><td><p><span>线程数。指定客户端与服务端之间使用的线程数。默认是1线程。需要客户端与服务器端同时使用此参数。</span></p></td></tr><tr><td><p><span>-S, --tos #</span></p></td><td><p><span>出栈数据包的服务类型。许多路由器忽略TOS字段。你可以指定这个值，使用以"0x"开始的16进制数，或以"0"开始的8进制数或10进制数。<br>例如，16进制'0x10' = 8进制'020' = 十进制'16'。TOS值1349就是：<br>IPTOS_LOWDELAY minimize delay 0x10<br>IPTOS_THROUGHPUT maximize throughput 0x08<br>IPTOS_RELIABILITY maximize reliability 0x04<br>IPTOS_LOWCOST minimize cost 0x02</span></p></td></tr><tr><td><p><span>-T, --ttl #</span></p></td><td><p><span>出栈多播数据包的TTL值。这本质上就是数据通过路由器的跳数。默认是1，链接本地。</span></p></td></tr><tr><td><p><span>-F (from v1.2 or higher)</span></p></td><td><p><span>使用特定的数据流测量带宽，例如指定的文件。<br>$ iperf -c &lt;server address&gt; -F &lt;file-name&gt;</span></p></td></tr><tr><td><p><span>-I (from v1.2 or higher)</span></p></td><td><p><span>与-F一样，由标准输入输出文件输入数据。</span></p></td></tr><tr><td colspan="2"><p><span>杂项</span></p></td></tr><tr><td><p><span>-h, --help</span></p></td><td><p><span>显示命令行参考并退出 。</span></p></td></tr><tr><td><p><span>-v, --version</span></p></td><td><p><span>显示版本信息和编译信息并退出。</span></p></td></tr></tbody></table>
    

1.  用Iperf测试路由器的性能
    

1.  **测试单线程TCP**
    

-   在服务端运行iperf，输入命令iperf –s –p 12345 –i 1 –M 以在本机端口12345上启用iperf
    
    ![](http://img1.ph.126.net/zQWPF_47aIcjPQRosPx84A==/6597302863122534214.jpg)
    
-   在客户端运行iperf，输入命令iperf –c _server-ip_ –p _server-port_ –i 1 –t 10 –w 20K，其中参数说明如下：
    
    \-c：客户端模式，后接服务器ip
    
    \-p：后接服务端监听的端口
    
    \-i：设置带宽报告的时间间隔，单位为秒
    
    \-t：设置测试的时长，单位为秒
    
    \-w：设置tcp窗口大小，一般可以不用设置，默认即可
    

测试后截图如下：

　　![](http://img0.ph.126.net/tzgWiwPTtYkZt-VKTtrxpg==/1935984889915935720.jpg)

客户端截图

![](http://img1.ph.126.net/8Tq0eUNK-vFd5vwSTGKLPg==/1682094460922930116.jpg)

服务端截图

其中：Interval表示时间间隔。Transfer表示时间间隔里面转输的数据量。Bandwidth是时间间隔里的传输速率。最后一行是本次测试的统计。测试可知带宽平均为89.9Mbit/s。

1.  **测试多线程TCP**
    
    在客户端添加-P参即可测试多线程的TCP性能，如下为使用两个线程的测试情况
    
    ![](http://img2.ph.126.net/4PU_NZfw11Lfk6NWeGlt1A==/6597793245309021740.jpg)
    
    客户端
    
    ![](http://img1.ph.126.net/mxX5KYjBR1uaUUIF25I6yw==/288511851228466648.jpg)
    
2.  **测试单线程UDP（默认带宽）**
    

-   在服务端运行iperf，输入命令iperf –s -u –p 12345 –i 1 以在本机端口12345上启用iperf，并运行于udp模式
    
-   在客户端运行iperf，输入命令iperf -c server-ip -p server-port -i 1 -t 10 -b，其中参数说明如下：
    
    \-c：客户端模式，后接服务器ip
    
    \-p：后接服务端监听的端口
    
    \-i：设置带宽报告的时间间隔，单位为秒
    
    \-t：设置测试的时长，单位为秒
    
    \-b：设置udp的发送带宽，单位bit/s
    

![](http://img2.ph.126.net/xLQ8882vlX-Rd9O_z4tJJQ==/1883349069271049615.jpg)

客户端

![](http://img2.ph.126.net/mcC2mJpB6dYWOvSoIZ-WGA==/6597722876564840099.jpg)

服务端

　　　　　其中，Jitter为抖动，lost/total为丢包数量，Datagrams为包数量。

1.  **测试单线程UDP（带宽为10Mbit/s）**
    
    设置客户端带宽为10M即可，使用参数-b指定
    

　　　　　　　　　　　　　　　　　　　　　　　　　　![](http://img1.ph.126.net/Y7S4YnzrZ4qaWDRDzzeWDQ==/3381358895325141868.jpg)

客户端

![](http://img1.ph.126.net/c4T74btXNH7G2eJSPz2-aw==/1884756444154596553.jpg)

服务端

1.  **测试多线程UDP**
    
    与多线程TCP类似，只需要客户端使用-P参数指定线程个数即可
    

1.  **测试UDP的双向传输**
    
    客户端使用参数-d以运行双测试模式，客户端会与服务端进行udp往返测试。可以使用-L参数指定本端双测试监听的端口。
    

![](http://img0.ph.126.net/RwibI0hja7Bd-LfvC3S94A==/3154490064096366291.jpg)

客户端

![](http://img0.ph.126.net/oiOttVMAGyi70BtQg4koUA==/3169126762885320009.jpg)服务端

1.  **测试UDP往复传输**
    
    与双向传输类似，使用参数-r以运行交互模式，仍然可以使用-L参数指定交互的端口。
    

![](http://img0.ph.126.net/1IeWWN4C3mZT3nSxMdxO5Q==/6598107705634352961.jpg)

客户端

![](http://img0.ph.126.net/8geXV5uD0vmF7caVzx6SxQ==/1982146786096484775.jpg)

服务端

1.  **分布式测试**
    
    使用多台电脑或使用一台电脑的多个IP地址测试。当使用一台电脑的多个iP地址测试时，可以使用-B命令绑定网卡的某一个ip地址以测试
    

1.  安装Iperf
    
    1.  对于windows版的Iperf，直接将解压出来的iperf.exe和cygwin1.dll复制到%systemroot%目录即可
        
    2.  对于linux版的Iperf，请使用如下命令安装
        
        gunzip -c iperf-<version>.tar.gz | tar -xvf -
        
        cd iperf-<version>
        
        ./configure
        
        make
        
        make install
        
2.  使用Iperf（以windows版本为例）
    
    在命令提示符中输入iperf命令即可运行Iperf，使用命令Iperf –help可以查看iperf的帮助
    
    ![](http://img0.ph.126.net/NlrsFWqdAIaSIDcRZcnGgA==/3171378562699005273.jpg)
    
3.  Iperf参数介绍
    
    <table><colgroup><col> <col></colgroup><tbody><tr><td><p><span><strong>命令行选项</strong></span></p></td><td><p><span><strong>描述</strong></span></p></td></tr><tr><td colspan="2"><p><span>客户端与服务器共用选项</span></p></td></tr><tr><td><p><span>-f, --format [bkmaBKMA]</span></p></td><td><p><span>格式化带宽数输出。支持的格式有：<br>'b' = bits/sec 'B' = Bytes/sec<br>'k' = Kbits/sec 'K' = KBytes/sec<br>'m' = Mbits/sec 'M' = MBytes/sec<br>'g' = Gbits/sec 'G' = GBytes/sec<br>'a' = adaptive bits/sec 'A' = adaptive Bytes/sec<br>自适应格式是kilo-和mega-二者之一。除了带宽之外的字段都输出为字节，除非指定输出的格式，默认的参数是a。<br>注 意：在计算字节byte时，Kilo = 1024， Mega = 1024^2，Giga = 1024^3。通常，在网络中，Kilo = 1000， Mega = 1000^2， and Giga = 1000^3，所以，Iperf也按此来计算比特（位）。如果这些困扰了你，那么请使用-f b参数，然后亲自计算一下。</span></p></td></tr><tr><td><p><span>-i, --interval #</span></p></td><td><p><span>设置每次报告之间的时间间隔，单位为秒。如果设置为非零值，就会按照此时间间隔输出测试报告。默认值为零。</span></p></td></tr><tr><td><p><span>-l, --len #[KM]</span></p></td><td><p><span>设置读写缓冲区的长度。TCP方式默认为8KB，UDP方式默认为1470字节。</span></p></td></tr><tr><td><p><span>-m, --print_mss</span></p></td><td><p><span>输出TCP MSS值（通过TCP_MAXSEG支持）。MSS值一般比MTU值小40字节。通常情况</span></p></td></tr><tr><td><p><span>-p, --port #</span></p></td><td><p><span>设置端口，与服务器端的监听端口一致。默认是5001端口，与ttcp的一样。</span></p></td></tr><tr><td><p><span>-u, --udp</span></p></td><td><p><span>使用UDP方式而不是TCP方式。参看-b选项。</span></p></td></tr><tr><td><p><span>-w, --window #[KM]</span></p></td><td><p><span>设置套接字缓冲区为指定大小。对于TCP方式，此设置为TCP窗口大小。对于UDP方式，此设置为接受UDP数据包的缓冲区大小，限制可以接受数据包的最大值。</span></p></td></tr><tr><td><p><span>-B, --bind host</span></p></td><td><p><span>绑定到主机的多个地址中的一个。对于客户端来 说，这个参数设置了出栈接口。对于服务器端来说，这个参数设置入栈接口。这个参数只用于具有多网络接口的主机。在Iperf的UDP模式下，此参数用于绑 定和加入一个多播组。使用范围在224.0.0.0至239.255.255.255的多播地址。参考-T参数。</span></p></td></tr><tr><td><p><span>-C, --compatibility</span></p></td><td><p><span>与低版本的Iperf使用时，可以使用兼容模式。不需要两端同时使用兼容模式，但是强烈推荐两端同时使用兼容模式。某些情况下，使用某些数据流可以引起1.7版本的服务器端崩溃或引起非预期的连接尝试。</span></p></td></tr><tr><td><p><span>-M, --mss #[KM}</span></p></td><td><p><span>通过TCP_MAXSEG选项尝试设置TCP最大信息段的值。MSS值的大小通常是TCP/IP头减去40字节。在以太网中，MSS值 为1460字节（MTU1500字节）。许多操作系统不支持此选项。</span></p></td></tr><tr><td><p><span>-N, --nodelay</span></p></td><td><p><span>设置TCP无延迟选项，禁用Nagle's运算法则。通常情况此选项对于交互程序，例如telnet，是禁用的。</span></p></td></tr><tr><td><p><span>-V (from v1.6 or higher)</span></p></td><td><p><span>绑定一个IPv6地址。<br>服务端：$ iperf -s –V<br>客户端：$ iperf -c &lt;Server IPv6 Address&gt; -V<br>注意：在1.6.3或更高版本中，指定IPv6地址不需要使用-B参数绑定，在1.6之前的版本则需要。在大多数操作系统中，将响应IPv4客户端映射的IPv4地址。</span></p></td></tr><tr><td colspan="2"><p><span>服务器端专用选项</span></p></td></tr><tr><td><p><span>-s, --server</span></p></td><td><p><span>Iperf服务器模式</span></p></td></tr><tr><td><p><span>-D (v1.2或更高版本)</span></p></td><td><p><span>Unix平台下Iperf作为后台守护进程运行。在Win32平台下，Iperf将作为服务运行。</span></p></td></tr><tr><td><p><span>-R(v1.2或更高版本，仅用于Windows)</span></p></td><td><p><span>卸载Iperf服务（如果它在运行）。</span></p></td></tr><tr><td><p><span>-o(v1.2或更高版本，仅用于Windows)</span></p></td><td><p><span>重定向输出到指定文件</span></p></td></tr><tr><td><p><span>-c, --client host</span></p></td><td><p><span>如果Iperf运行在服务器模式，并且用-c参数指定一个主机，那么Iperf将只接受指定主机的连接。此参数不能工作于UDP模式。</span></p></td></tr><tr><td><p><span>-P, --parallel #</span></p></td><td><p><span>服务器关闭之前保持的连接数。默认是0，这意味着永远接受连接。</span></p></td></tr><tr><td colspan="2"><p><span>客户端专用选项</span></p></td></tr><tr><td><p><span>-b, --bandwidth #[KM]</span></p></td><td><p><span>UDP模式使用的带宽，单位bits/sec。此选项与-u选项相关。默认值是1 Mbit/sec。</span></p></td></tr><tr><td><p><span>-c, --client host</span></p></td><td><p><span>运行Iperf的客户端模式，连接到指定的Iperf服务器端。</span></p></td></tr><tr><td><p><span>-d, --dualtest</span></p></td><td><p><span>运行双测试模式。这将使服务器端反向连接到客户端，使用-L 参数中指定的端口（或默认使用客户端连接到服务器端的端口）。这些在操作的同时就立即完成了。如果你想要一个交互的测试，请尝试-r参数。</span></p></td></tr><tr><td><p><span>-n, --num #[KM]</span></p></td><td><p><span>传送的缓冲器数量。通常情况，Iperf按照10秒钟发送数据。-n参数跨越此限制，按照指定次数发送指定长度的数据，而不论该操作耗费多少时间。参考-l与-t选项。</span></p></td></tr><tr><td><p><span>-r, --tradeoff</span></p></td><td><p><span>往复测试模式。当客户端到服务器端的测试结束时，服务器端通过-l选项指定的端口（或默认为客户端连接到服务器端的端口），反向连接至客户端。当客户端连接终止时，反向连接随即开始。如果需要同时进行双向测试，请尝试-d参数。</span></p></td></tr><tr><td><p><span>-t, --time #</span></p></td><td><p><span>设置传输的总时间。Iperf在指定的时间内，重复的发送指定长度的数据包。默认是10秒钟。参考-l与-n选项。</span></p></td></tr><tr><td><p><span>-L, --listenport #</span></p></td><td><p><span>指定服务端反向连接到客户端时使用的端口。默认使用客户端连接至服务端的端口。</span></p></td></tr><tr><td><p><span>-P, --parallel #</span></p></td><td><p><span>线程数。指定客户端与服务端之间使用的线程数。默认是1线程。需要客户端与服务器端同时使用此参数。</span></p></td></tr><tr><td><p><span>-S, --tos #</span></p></td><td><p><span>出栈数据包的服务类型。许多路由器忽略TOS字段。你可以指定这个值，使用以"0x"开始的16进制数，或以"0"开始的8进制数或10进制数。<br>例如，16进制'0x10' = 8进制'020' = 十进制'16'。TOS值1349就是：<br>IPTOS_LOWDELAY minimize delay 0x10<br>IPTOS_THROUGHPUT maximize throughput 0x08<br>IPTOS_RELIABILITY maximize reliability 0x04<br>IPTOS_LOWCOST minimize cost 0x02</span></p></td></tr><tr><td><p><span>-T, --ttl #</span></p></td><td><p><span>出栈多播数据包的TTL值。这本质上就是数据通过路由器的跳数。默认是1，链接本地。</span></p></td></tr><tr><td><p><span>-F (from v1.2 or higher)</span></p></td><td><p><span>使用特定的数据流测量带宽，例如指定的文件。<br>$ iperf -c &lt;server address&gt; -F &lt;file-name&gt;</span></p></td></tr><tr><td><p><span>-I (from v1.2 or higher)</span></p></td><td><p><span>与-F一样，由标准输入输出文件输入数据。</span></p></td></tr><tr><td colspan="2"><p><span>杂项</span></p></td></tr><tr><td><p><span>-h, --help</span></p></td><td><p><span>显示命令行参考并退出 。</span></p></td></tr><tr><td><p><span>-v, --version</span></p></td><td><p><span>显示版本信息和编译信息并退出。</span></p></td></tr></tbody></table>