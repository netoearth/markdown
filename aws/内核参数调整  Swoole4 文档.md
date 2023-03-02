[![](https://wenda-1252906962.file.myqcloud.com/images/ad/crmeb-3.jpg)![](https://wiki.swoole.com/_images/ico.png)](https://github.crmeb.net/u/Swoole)

📝 更新时间：2022-12-19 11:55 ，[修改](https://github.com/swoole/docs/edit/main/zh-ch//other/sysctl.md)

## [内核参数调整](https://wiki.swoole.com/#/other/sysctl?id=%e5%86%85%e6%a0%b8%e5%8f%82%e6%95%b0%e8%b0%83%e6%95%b4)

## [ulimit 设置](https://wiki.swoole.com/#/other/sysctl?id=ulimit-%e8%ae%be%e7%bd%ae)

`ulimit -n` 要调整为 100000 甚至更大。 命令行下执行 `ulimit -n 100000` 即可修改。如果不能修改，需要设置 `/etc/security/limits.conf`，加入

```
* soft nofile 262140
* hard nofile 262140
root soft nofile 262140
root hard nofile 262140
* soft core unlimited
* hard core unlimited
root soft core unlimited
root hard core unlimited
```

注意，修改 `limits.conf` 文件后，需要重启系统生效

## [内核设置](https://wiki.swoole.com/#/other/sysctl?id=%e5%86%85%e6%a0%b8%e8%ae%be%e7%bd%ae)

`Linux` 操作系统修改内核参数有 3 种方式：

-   修改 `/etc/sysctl.conf` 文件，加入配置选项，格式为 `key = value`，修改保存后调用 `sysctl -p` 加载新配置
-   使用 `sysctl` 命令临时修改，如：`sysctl -w net.ipv4.tcp_mem="379008 505344 758016"`
-   直接修改 `/proc/sys/` 目录中的文件，如：`echo "379008 505344 758016" > /proc/sys/net/ipv4/tcp_mem`

> 第一种方式在操作系统重启后会自动生效，第二和第三种方法重启后失效

### [net.unix.max\_dgram\_qlen = 100](https://wiki.swoole.com/#/other/sysctl?id=netunixmax_dgram_qlen-100)

swoole 使用 unix socket dgram 来做进程间通信，如果请求量很大，需要调整此参数。系统默认为 10，可以设置为 100 或者更大。或者增加 worker 进程的数量，减少单个 worker 进程分配的请求量。

### [net.core.wmem\_max](https://wiki.swoole.com/#/other/sysctl?id=netcorewmem_max)

修改此参数增加 socket 缓存区的内存大小

```
net.ipv4.tcp_mem  =   379008       505344  758016
net.ipv4.tcp_wmem = 4096        16384   4194304
net.ipv4.tcp_rmem = 4096          87380   4194304
net.core.wmem_default = 8388608
net.core.rmem_default = 8388608
net.core.rmem_max = 16777216
net.core.wmem_max = 16777216
```

### [net.ipv4.tcp\_tw\_reuse](https://wiki.swoole.com/#/other/sysctl?id=netipv4tcp_tw_reuse)

是否 socket reuse，此函数的作用是 Server 重启时可以快速重新使用监听的端口。如果没有设置此参数，会导致 server 重启时发生端口未及时释放而启动失败

### [net.ipv4.tcp\_tw\_recycle](https://wiki.swoole.com/#/other/sysctl?id=netipv4tcp_tw_recycle)

使用 socket 快速回收，短连接 Server 需要开启此参数。此参数表示开启 TCP 连接中 TIME-WAIT sockets 的快速回收，Linux 系统中默认为 0，表示关闭。打开此参数可能会造成 NAT 用户连接不稳定，请谨慎测试后再开启。

## [消息队列设置](https://wiki.swoole.com/#/other/sysctl?id=%e6%b6%88%e6%81%af%e9%98%9f%e5%88%97%e8%ae%be%e7%bd%ae)

当使用消息队列作为进程间通信方式时，需要调整此内核参数

-   kernel.msgmnb = 4203520，消息队列的最大字节数
-   kernel.msgmni = 64，最多允许创建多少个消息队列
-   kernel.msgmax = 8192，消息队列单条数据最大的长度

## [FreeBSD/MacOS](https://wiki.swoole.com/#/other/sysctl?id=freebsdmacos)

-   sysctl -w net.local.dgram.maxdgram=8192
-   sysctl -w net.local.dgram.recvspace=200000 修改 Unix Socket 的 buffer 区尺寸

## [开启 CoreDump](https://wiki.swoole.com/#/other/sysctl?id=%e5%bc%80%e5%90%af-coredump)

设置内核参数

```
kernel.core_pattern = /data/core_files/core-%e-%p-%t
```

通过 ulimit -c 命令查看当前 coredump 文件的限制

```
ulimit -c
```

如果为 0，需要修改 /etc/security/limits.conf，进行 limit 设置。

> 开启 core-dump 后，一旦程序发生异常，会将进程导出到文件。对于调查程序问题有很大的帮助

## [其他重要配置](https://wiki.swoole.com/#/other/sysctl?id=%e5%85%b6%e4%bb%96%e9%87%8d%e8%a6%81%e9%85%8d%e7%bd%ae)

-   net.ipv4.tcp\_syncookies=1
-   net.ipv4.tcp\_max\_syn\_backlog=81920
-   net.ipv4.tcp\_synack\_retries=3
-   net.ipv4.tcp\_syn\_retries=3
-   net.ipv4.tcp\_fin\_timeout = 30
-   net.ipv4.tcp\_keepalive\_time = 300
-   net.ipv4.tcp\_tw\_reuse = 1
-   net.ipv4.tcp\_tw\_recycle = 1
-   net.ipv4.ip\_local\_port\_range = 20000 65000
-   net.ipv4.tcp\_max\_tw\_buckets = 200000
-   net.ipv4.route.max\_size = 5242880

## [查看配置是否生效](https://wiki.swoole.com/#/other/sysctl?id=%e6%9f%a5%e7%9c%8b%e9%85%8d%e7%bd%ae%e6%98%af%e5%90%a6%e7%94%9f%e6%95%88)

如：修改 `net.unix.max_dgram_qlen = 100` 后，通过

```
cat /proc/sys/net/unix/max_dgram_qlen
```

如果修改成功，这里是新设置的值。

[![](https://wenda-1252906962.file.myqcloud.com/images/ad/crmeb-3.jpg)![](https://wiki.swoole.com/_images/ico.png)](https://github.crmeb.net/u/Swoole)