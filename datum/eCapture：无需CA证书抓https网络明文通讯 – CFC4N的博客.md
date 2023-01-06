eCapture是一款基于eBPF技术实现的用户态数据捕获工具。不需要CA证书，即可捕获https/tls的通讯明文。  
![](https://image.cnxct.com//2022/03/how-ecapture-works.png)

项目在2022年3月中旬创建，一经发布，广受大家喜爱，至今不到两周已经1200多个Star。

![](https://image.cnxct.com/2022/03/ecapture-github-2-scaled.jpg)

## 作用

1.  不需要CA证书，即可捕获HTTPS/TLS通信数据的明文。
2.  在bash审计场景，可以捕获bash命令。
3.  数据库审计场景，可以捕获mysqld/mariadDB的SQL查询。

## 官网

代码仓库见：[https://github.com/ehids/ecapture](https://github.com/ehids/ecapture) ，也可以关注微信公众号「榫卯江湖」获取最新动态。  
![](https://image.cnxct.com/2022/03/wechat-white-search-no-alpha.png)

## 产品架构

eCapture系统用户态程序使用Golang语言开发，具有良好的系统兼容性，无依赖快速部署，更适合云原生场景。  
内核态代码使用C编写，使用clang/llvm编译，生产bpf字节码后，采用`go-bindata`转化为golang语法文件，之后采用`ehids/ebpfmanager`类库，调用bpf syscall进行加载、HOOK、map读取。  
golang编译后，无其他任何依赖即可运行，兼容linux kernel 4.18以上所有版本。

![](https://image.cnxct.com/2022/03/ecapture-architecture-1.png)

## eBPF加载机制

关于eBPF详细加载机制，可到[https://ebpf.io/](https://ebpf.io/) 查阅相关原理。

![](https://image.cnxct.com/2022/03/linux_ebpf_internals.png)

## 实现原理

如工作原理的图所示，在用户态的加密解密函数中下钩子。 tcpdump(libpcap)是在数据包接收到，XDP处理后，进行`clone packet`，进行包的复制，发送给用户态进程。二者工作的所在层不一样。

## 功能介绍

eCapture有三个模块

1.  tls/ssl明文数据捕获
2.  bash命令审计
3.  mysqld数据库审计

第一个功能适用于基于tls/ssl解密需求的运维监控、故障排查、抽样分析场景。

第二个功能适用于安全领域的bash入侵发现场景，这里只是简单的功能，可以在此基础上增加其他功能。

第三个功能适用于数据库审计场景，尤其是做数据安全、数据防泄漏，甚至入侵检测等。同样，可以在此基础上扩充其他功能。

![](https://image.cnxct.com/2022/03/ecapture-can-used-to.png)

查看其使用说明，如下

```
cfc4n@vm-desktop:~/ehids/ecapture$ ./bin/ecapture
NAME:
    ecapture - capture text SSL content without CA cert by ebpf hook.

USAGE:
    ecapture [flags]

VERSION:
    0.1.5-20220325-47edbed

COMMANDS:
    bash        capture bash command
    help        Help about any command
    mysqld56    capture sql queries from mysqld >5.6 .
    tls     alias name:openssl , use to capture tls/ssl text content without CA cert.

DESCRIPTION:
    ecapture是一款无需安装CA证书，即可抓去HTTPS、TLS等明文数据包的工具。
    也可以捕获bash的命令，适用于安全审计场景。包括mysqld的数据库审计等。
    仓库地址: https://github.com/ehids/ecapture

OPTIONS:
      --debug[=false]   enable debug logging
  -h, --help[=false]    help for ecapture
      --hex[=false] print byte strings as hex encoded strings
  -p, --pid=0       if target_pid is 0 then we target all pids
```

其中，有四个全局参数，分别是

-   –debug ， 用于启动调试日志
-   –help ， 查看帮助
-   –hex ，按照hex模式打印字符，用与查看不可见字符
-   –pid ，用于针对特定进程进行数据捕获

## HOOK机制

eCapture采用eBPF uprobe相关函数进行HOOK，故需要目标用户态函数信息，包含函数符号表(symbol table)，函数偏移地址(offset)。 在大部分linux发行版中，使用的二进制可执行文件(ELF)都是包含符号表的；少部分发行版，会去掉ELF中的符号表。那么针对这种场景，就需要用户自行定位目标函数所在ELF/SO中的偏移地址，通过工具的参数来指定。

对于ELF文件，可以将目标类库静态编译到自身，也可以通过动态链接库的方式引用。那么对于这两种形式，eCapture根据不同场景进行自动查找。若查找不到，用户可以通过命令行参数指定。

故eCapture支持`HOOK ELF`，以及`HOOK SO`两种模式。会自动分析ELF文件，读取`.dynamic`和`.dynsym`等段信息，查找相关链接库名以及函数名、偏移地址。

查找原理如下图：

![](https://image.cnxct.com/2022/03/tls-hook-offset-location.png)

## tls/ssl

`ecapture tls`命令用于启动tls/ssl模块，支持了三类tls/ssl加密类库，分别是

-   openssl ，动态链接库名字为`libssl.so`
-   gnutls ，动态链接库名字为`libgnutls.so`
-   nss/nspr ，动态链接库名字为`libnspr4.so`

![](https://image.cnxct.com/2022/03/ecapture-support-tls-ssl.png)

在不同的linux发行版中，因为各种原因，会选择不同的类库。比如`wget`程序，在ubuntu跟centos中就会使用不同的类库。有的是`openssl`，有的是`gnutls`，甚至两个库都引入了。

具体情况，你可以使用`ldd $ELF_PATH | grep -E "tls|ssl|nspr|nss"`来查看一个ELF文件使用类库情况。

```
cfc4n@vm-desktop:~$ ldd `which wget` |grep -E "tls|ssl|nspr|nss"
    libssl.so.1.1 => /lib/x86_64-linux-gnu/libssl.so.1.1 (0x00007f50699f6000)
```

对于firefox、chrome这种进程，需要在程序启动后才能看到tls类库依赖情况，那么，你可以使用`sudo pldd $PID | grep -E "tls|ssl|nspr|nss"` 来查看

```
cfc4n@vm-desktop:~$ ps -ef|grep firefox
cfc4n       6846    1432 45 17:50 ?        00:00:04 /usr/lib/firefox/firefox -new-window
cfc4n@vm-desktop:~$ sudo pldd 6846 |grep -E "tls|ssl|nspr|nss"
/usr/lib/firefox/libnspr4.so
/usr/lib/firefox/libnssutil3.so
/usr/lib/firefox/libnss3.so
/usr/lib/firefox/libssl3.so
/lib/x86_64-linux-gnu/libnss_files.so.2
/lib/x86_64-linux-gnu/libnss_mdns4_minimal.so.2
/lib/x86_64-linux-gnu/libnss_dns.so.2
/usr/lib/firefox/libnssckbi.so
```

eCapture的tls模块命令行参数如下，用户可以使用默认配置外，也可以根据自己环境自行指定。

```
OPTIONS:
      --curl=""       curl or wget file path, use to dectet openssl.so path, default:/usr/bin/curl
      --firefox=""    firefox file path, default: /usr/lib/firefox/firefox.
      --gnutls="" libgnutls.so file path, will automatically find it from curl default.
  -h, --help[=false]    help for tls
      --libssl="" libssl.so file path, will automatically find it from curl default.
      --nspr=""       libnspr44.so file path, will automatically find it from curl default.
      --wget=""       wget file path, default: /usr/bin/wget.
```

同时，使用方法也比较简单，`./ecapture tls --hex`命令即可。

![](https://image.cnxct.com/2022/03/ecapture-tls-wget.jpg)

在linux上，firefox程序中，有很多通讯都使用了`/usr/lib/firefox/libnspr4.so`，但实际上业务请求是可以通过`Socket Thread`进程来发送的。可以通过这个特点来过滤，对于chrome程序，相信细心的你，也能搞定。

![](https://image.cnxct.com/2022/03/ecapture-firefox.jpg)

## bash

笔者在安全部门工作，接到过bash审计需求，其实现方法无非是修改系统类库、使用内核模块等技术实现，对系统稳定性有一定风险。基于eBPF技术实现，可以避开这些问题。这里的bash命令的监控，是作为eBPF技术在安全审计场景中的一个探索。

eCapture在实现时首先查找ENV的$SHELL值，作为bash的二进制文件路径进行HOOK。对于bash加载了`libreadline.so`的场景，也会自动分析，进行符号表查找、offset定位，再进行HOOK。

bash模块的参数有三个，用户可以自定义`bash`、`readlineso`的路径。

```
OPTIONS:
      --bash=""       $SHELL file path, eg: /bin/bash , will automatically find it from $ENV default.
  -h, --help[=false]    help for bash
      --readlineso="" readline.so file path, will automatically find it from $BASH_PATH default.
```

![](https://image.cnxct.com/2022/03/ecapture-bash.jpg)

## mysql/mariadb

与bash模块一样，也是作为数据库审计的一个探索。笔者环境为ubuntu 21.04，mysqld也因为协议关系，使用了衍生的`MariadDB`，用户也可以根据自己实际场景，使用命令行参数进行指定。

mysqld模块，核心原理是HOOK了`dispatch_command`函数，

-   第一个参数为CMD类型，值为COM\_QUERY时，为查询场景，即审计需求的查询类型。
-   第二个参数是THD的结构体，在这里我们用不到。
-   第三个是查询的SQL语句
-   第四个参数是SQL语句的长度，

```
// https://github.com/MariaDB/server/blob/b5852ffbeebc3000982988383daeefb0549e058a/sql/sql_parse.h#L112
dispatch_command_return dispatch_command(enum enum_server_command command, THD *thd,
   char* packet, uint packet_length, bool blocking = true);
```

mysqld审计模块参数如下:

```
OPTIONS:
  -f, --funcname=""           function name to hook
  -h, --help[=false]            help for mysqld56
  -m, --mysqld="/usr/sbin/mariadbd"   mysqld binary file path, use to hook
      --offset=0            0x710410
```

其中，`--mysqld`是用来指定mysqld的路径。 mysqld二进制程序符号表里虽然有`dispatch_command`信息，但`dispatch_command`这个函数名每次编译都是变化的，故不能写死。

eCapture的查找方式是读取mysqld二进制的`.dynamic`段信息，正则语法`\w+dispatch_command\w+`去匹配所有符号信息，找到其函数名、偏移地址，再使用。

你也可以通过objdump命令来查找，再通过命令行参数自行指定funcname。

> mariadbd version : 10.5.13-MariaDB-0ubuntu0.21.04.1  
> objdump -T /usr/sbin/mariadbd |grep dispatch\_command  
> 0000000000710410 g DF .text 0000000000002f35 Base \_Z16dispatch\_command19enum\_server\_commandP3THDPcjbb

即offset为0x710410，函数名为`_Z16dispatch_command19enum_server_commandP3THDPcjbb`。

![](https://image.cnxct.com/2022/03/ecapture-mysqld.jpg)

## 使用

### 下载二进制包

eCapture发布在[https://github.com/ehids/ecapture/releases](https://github.com/ehids/ecapture/releases) ，目前最新版为`eCapture v0.1.5`。

可在linux kernel 4.18以上版本运行。

**二进制包地址：**

[https://github.com/ehids/ecapture/releases/download/v0.1.5/ecapture\_v0.1.5.zip](https://github.com/ehids/ecapture/releases/download/v0.1.5/ecapture_v0.1.5.zip)

**国内加速地址：**  
[https://github.do/https://github.com/ehids/ecapture/releases/download/v0.1.5/ecapture\_v0.1.5.zip](https://github.do/https://github.com/ehids/ecapture/releases/download/v0.1.5/ecapture_v0.1.5.zip)

### 自行编译

代码仓库在[https://github.com/ehids/ecapture](https://github.com/ehids/ecapture) ，你可以自行修改源码编译。

## 演示视频

腾讯视频：

B站：  
[https://www.bilibili.com/video/BV1si4y1Q74a](https://www.bilibili.com/video/BV1si4y1Q74a "https://www.bilibili.com/video/BV1si4y1Q74a")

## Github仓库

[![知识共享许可协议](https://www.cnxct.com/attachments/88x31.png)](http://creativecommons.org/licenses/by-nc-sa/3.0/deed.zh)CFC4N的博客 由 [CFC4N](http://www.cnxct.com/) 创作，采用 [知识共享 署名-非商业性使用-相同方式共享（3.0未本地化版本）许可协议](http://creativecommons.org/licenses/by-nc-sa/3.0/deed.zh)进行许可。基于[https://www.cnxct.com](http://www.cnxct.com/)上的作品创作。转载请注明转自：[eCapture：无需CA证书抓https网络明文通讯](https://www.cnxct.com/what-is-ecapture/)