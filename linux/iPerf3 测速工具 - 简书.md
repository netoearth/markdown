## iPerf3 测速工具

[![](https://upload.jianshu.io/users/upload_avatars/1934827/99488e08-19c3-47a3-bd7d-cfef8fc6377d.jpg?imageMogr2/auto-orient/strip|imageView2/1/w/96/h/96/format/webp)](https://www.jianshu.com/u/c668a382b97b)

0.2072020.05.03 15:24:38字数 612阅读 22,597

Tags：iPerf3 网络 测速 教程 路由器 内网

## 0\. 瞎 bb

iPerf - The ultimate speed test tool for TCP, UDP and SCTP  
iPerf - TCP、UDP 和 SCTP 的终极速度测试工具

一直想测试内网路由器的速度，看看极限在哪，于是找到了这款工具 iPerf3。

由于网上的教程众多，本文会比较简略。

## 1\. 准备

官网链接：[https://iperf.fr/iperf-download.php](https://links.jianshu.com/go?to=https%3A%2F%2Fiperf.fr%2Fiperf-download.php)  
我选择了 **Windows 64 bits** 和 **Magic iPerf**，分别用于我的 **Windows**(192.168.50.167) 和 **Android**(192.168.50.223)。

下载操作可能需要梯子。

Windows 上，解压压缩包到任意路径，命令行进入该路径（Tips：最简单的办法就是，资源管理器打开目录，在导航栏路径中直接输入 cmd 并回车），然后直接运行 iperf3.exe 附带参数即可。

```
D:\iperf-3.1.3-win64>iperf3.exe -h
Usage: iperf [-s|-c host] [options]
       iperf [-h|--help] [-v|--version]
...
```

Android 上就更简单了，左上角切换到 iPerf3，中间输入参数，右上角切换到 Started 就可以了。但是为了方便理解，下述操作涉及命令行的时候，我会把文件名附带上，并且加上 $ 符号（模拟 Linux 终端）。

```
$ ./iperf3 -h
Usage: iperf [-s|-c host] [options]
  iperf [-h|--help] [-v|--version]
...
```

## 2\. 简单测试

### 2.1 TCP 上行测试

```
# Server(Windows) 输入
D:\iperf-3.1.3-win64>iperf3.exe -s
```

```
# Client(Android) 输入
$ ./iperf3 -c 192.168.50.167
```

```
# Server(Windows) 输出
[ID] Interval        Transfer    Bandwidth
[ 5] 0.00-10.07 sec  0.00 Bytes  0.00 bits/sec  sender
[ 5] 0.00-10.07 sec  280 MBytes  234 Mbits/sec  receiver
```

```
# Client(Android) 输出
[ID] Interval        Transfer    Bandwidth
[ 4] 0.00-10.03 sec  282 MBytes  236 Mbits/sec  sender
[ 4] 0.00-10.03 sec  280 MBytes  234 Mbits/sec  receiver
```

**由于是 Client 向 Server 上传数据，所以此时 sender 代表 Client 向 Server 发送的数据总和，receiver 代表 Server 从 Client 接受的数据总和。**

至于为什么 sender 与 receiver 会不一样，个人的推测是丢包。

如果使用 -P 参数开启多线程模式，线程数特别大时，sender 与 receiver 的差会进一步拉大。

### 2.2 TCP 下行测试

```
# Server(Windows) 输入
D:\iperf-3.1.3-win64>iperf3.exe -s
```

```
# Client(Android) 输入
$ ./iperf3 -c 192.168.50.167 -R
```

```
# Server(Windows) 输出
[ID] Interval        Transfer    Bandwidth
[ 5] 0.00-10.04 sec  421 MBytes  352 Mbits/sec  sender
[ 5] 0.00-10.04 sec  0.00 Bytes  0.00 bits/sec  receiver
```

```
# Client(Android) 输出
[ID] Interval        Transfer    Bandwidth
[ 4] 0.00-10.00 sec  421 MBytes  352 Mbits/sec  sender
[ 4] 0.00-10.00 sec  421 MBytes  352 Mbits/sec  receiver
```

**由于是 Client 从 Server 下载数据，所以此时 sender 代表 Client 从 Server 接受的数据总和，receiver 代表 Server 向 Client 发送的数据总和。**

### 2.3 UDP 上行测试

```
# Server(Windows) 输入
D:\iperf-3.1.3-win64>iperf3.exe -s
```

```
# Client(Android) 输入
$ ./iperf3 -c 192.168.50.167 -u -b 300M
```

```
# Server(Windows) 输出
[ID] Interval        Transfer    Bandwidth      Jitter    Lost/Total Datagrams
[ 5] 0.00-10.03 sec  0.00 Bytes  0.00 bits/sec  0.080 ms  0/45646 (0%)
```

```
# Client(Android) 输出
[ID] Interval        Transfer    Bandwidth      Jitter    Lost/Total Datagrams
[ 5] 0.00-10.00 sec  357 MBytes  299 Mbits/sec  0.080 ms  0/45646 (0%)
```

### 2.4 UDP 下行测试

**注意，这里的 Windows 和 Android 对调了 C/S 身份。**

```
# Server(Android) 输入
$ ./iperf3 -s
```

```
# Client(Windows) 输入
D:\iperf-3.1.3-win64>iperf3.exe -c 192.168.50.223 -u -b 500M
```

```
# Server(Android) 输出
[ID] Interval        Transfer    Bandwidth      Jitter    Lost/Total Datagrams
[ 5] 0.00-10.09 sec  0.00 Bytes  0.00 bits/sec  0.123 ms  5888/36477 (16%)
```

```
# Client(Windows) 输出
[ID] Interval        Transfer    Bandwidth      Jitter    Lost/Total Datagrams
[ 5] 0.00-10.00 sec  596 MBytes  500 Mbits/sec  0.123 ms  5888/36477 (16%)
```

**此时能够发现，多了个 Lost 数据，这应该就是 UDP 丢包了。**

## 3\. 关于压力测试

**加大压力，减少丢包。**

TCP：在 2.2 的基础上，增加 -P 参数开启多线程下载，并且输出的 sender 与 receiver 的差要尽可能的小。

```
# Client(Android) 输入
$ ./iperf3 -c 192.168.50.167 -R -P 16
```

UDP：在 2.4 的基础上，调整 -b 参数控制带宽大小，并且 Lost 丢包要要尽可能的小。

```
# Client(Windows) 输入
D:\iperf-3.1.3-win64>iperf3.exe -c 192.168.50.223 -u -b 1000M
```

## 4\. Usage message

```
D:\iperf-3.1.3-win64>iperf3.exe -h
Usage: iperf [-s|-c host] [options]
       iperf [-h|--help] [-v|--version]

Server or Client:
  -p, --port      #         server port to listen on/connect to
  -f, --format    [kmgKMG]  format to report: Kbits, Mbits, KBytes, MBytes
  -i, --interval  #         seconds between periodic bandwidth reports
  -F, --file name           xmit/recv the specified file
  -B, --bind      <host>    bind to a specific interface
  -V, --verbose             more detailed output
  -J, --json                output in JSON format
  --logfile f               send output to a log file
  -d, --debug               emit debugging output
  -v, --version             show version information and quit
  -h, --help                show this message and quit
Server specific:
  -s, --server              run in server mode
  -D, --daemon              run the server as a daemon
  -I, --pidfile file        write PID file
  -1, --one-off             handle one client connection then exit
Client specific:
  -c, --client    <host>    run in client mode, connecting to <host>
  -u, --udp                 use UDP rather than TCP
  -b, --bandwidth #[KMG][/#] target bandwidth in bits/sec (0 for unlimited)
                            (default 1 Mbit/sec for UDP, unlimited for TCP)
                            (optional slash and packet count for burst mode)
  -t, --time      #         time in seconds to transmit for (default 10 secs)
  -n, --bytes     #[KMG]    number of bytes to transmit (instead of -t)
  -k, --blockcount #[KMG]   number of blocks (packets) to transmit (instead of -t or -n)
  -l, --len       #[KMG]    length of buffer to read or write
                            (default 128 KB for TCP, 8 KB for UDP)
  --cport         <port>    bind to a specific client port (TCP and UDP, default: ephemeral port)
  -P, --parallel  #         number of parallel client streams to run
  -R, --reverse             run in reverse mode (server sends, client receives)
  -w, --window    #[KMG]    set window size / socket buffer size
  -M, --set-mss   #         set TCP/SCTP maximum segment size (MTU - 40 bytes)
  -N, --no-delay            set TCP/SCTP no delay, disabling Nagle's Algorithm
  -4, --version4            only use IPv4
  -6, --version6            only use IPv6
  -S, --tos N               set the IP 'type of service'
  -Z, --zerocopy            use a 'zero copy' method of sending data
  -O, --omit N              omit the first n seconds
  -T, --title str           prefix every output line with this string
  --get-server-output       get results from server
  --udp-counters-64bit      use 64-bit counters in UDP test packets

[KMG] indicates options that support a K/M/G suffix for kilo-, mega-, or giga-

iperf3 homepage at: http://software.es.net/iperf/
Report bugs to:     https://github.com/esnet/iperf
```

禁止转载，如需转载请通过简信或评论联系作者。

[![  ](https://upload.jianshu.io/users/upload_avatars/1934827/99488e08-19c3-47a3-bd7d-cfef8fc6377d.jpg?imageMogr2/auto-orient/strip|imageView2/1/w/100/h/100/format/webp)](https://www.jianshu.com/u/c668a382b97b)

[Tosh](https://www.jianshu.com/u/c668a382b97b "Tosh")毕业于中国 TOP500 高校；ACM 塑料鶸；全沾工程师；精通 Bash、C#、C/C++、...

总资产1共写了5029字获得8个赞共11个粉丝