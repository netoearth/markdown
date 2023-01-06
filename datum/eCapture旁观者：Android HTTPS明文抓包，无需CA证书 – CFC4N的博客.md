## 前言

Android上抓包HTTPS是不是越来越难了？高版本无法添加CA证书，抓包软件依赖太多，VPN模式、或HOOK程序时，会被APP检测到。对抗成本愈加增高。有什么万能的工具吗？

是的，[eCapture for Android](https://ecapture.cc/)来了。以后在Android上抓HTTPS通讯包，再也不用安装CA证书了，再也不用下载一堆python依赖环境了，再也不用重打包ssl类库了，再也不用改一堆手机参数了，一键启用，简单明了。

eCapture是一款无需CA证书即可抓获HTTPS明文的软件。支持pcapng格式，支持Wireshark直接查看。基于eBPF技术，仅需root权限，即可一键抓包。  
eCapture中文名旁观者，即 当局者迷，旁观者清。  
![](https://image.cnxct.com/2022/09/ecapture.cc_zh_.png)

2022年年初上海疫情期间，[笔者开始编写并开源](https://www.cnxct.com/what-is-ecapture/)，至今已经半年，GitHub上已经 `4200`个星星。

eCapture是基于eBPF技术实现的抓包软件，依赖系统内核是否支持eBPF。目前支持在操作系统上，支持了X86\_64\\ARM64的Linux kernel 4.18以上内核，支持了ARM64 Android(Linux) kernel 5.4以上版本。最新版是在2022年9月9日发布的v0.4.3版本。

![](https://image.cnxct.com/2022/09/github.com_ehids_ecapture_releases.png)

## 演示视频

下载后，一条命令启动，干净利索。  
`./ecapture tls -w ecapture.pcapng`

先看演示视频，演示环境为Ubuntu 21.04、Android 5.4 （Pixel 6）。

## 模块功能

eCapture支持tls、bash、mysqld、postgres等模块的信息提取与捕获。本文仅讨论`tls`这个HTTPS/TLS明文捕获模块。

## 加密通讯明文捕获–tls模块

`tls`模块在加密通讯类库上，支持了openssl、gnutls、nspr/nss、boringssl等类库。  
但在Android上，pcapng模式只支持boringssl，文本模式则都支持。

## 如何使用eCapture

## 环境依赖

1.  操作系统 Linux kernel 4.18以上，Android kernel 5.4以上。
2.  支持BPF，可选支持BTF（eCapture版本不同）
3.  root权限

## 版本选择

BPF [CO-RE](https://facebookmicrosites.github.io/bpf/blog/2020/02/19/bpf-portability-and-co-re.html)特性为BTF通用的格式，用于做跨内核版本兼容。有的Android手机是没开启BTF。可以查看系统配置确认，`CONFIG_DEBUG_INFO_BTF=y`为开启；`CONFIG_DEBUG_INFO_BTF=n`为关闭；其他为不支持BPF，无法使用eCapture。

```
cfc4n@vm-server:~$# cat /boot/config-`uname -r` | grep CONFIG_DEBUG_INFO_BTF
CONFIG_DEBUG_INFO_BTF=y
```

Android系统上，config是gzip压缩的，且配置文件目录也变了。可使用`zcat /proc/config.gz`命令代替。

eCapture默认发行了支持CO-RE的ELF程序。Android版会发行一个5.4内核不支持BTF（即没有CO-RE）的版本。 下载后，可以通过`./ecapture -v`确认。 非CO-RE版本的version信息中包含编译时的内核版本。

```
# no CO-RE
eCapture version:   linux_aarch64:0.4.2-20220906-fb34467:5.4.0-104-generic
# CO-RE
eCapture version:   linux_aarch64:0.4.2-20220906-fb34467:[CORE]
```

若版本不符合自己需求，可以自行编译，步骤见文末。

## 全局参数介绍

全局参数重点看如下几个

```
root@vm-server:/home/cfc4n/# ecapture -h
      --hex[=false]     print byte strings as hex encoded strings
  -l, --log-file=""       -l save the packets to file
  -p, --pid=0           if pid is 0 then we target all pids
  -u, --uid=0           if uid is 0 then we target all users
```

1.  `--hex` 用于stdout输出场景，展示结果的十六进制，用于查看非ASCII字符，在内容加密、编码的场景特别有必要。
2.  `-l, --log-file=` 保存结果的文件路径。
3.  `-p, --pid=0` 捕获的目标进程，默认为0，则捕获所有进程。
4.  `-u, --uid=0` 捕获的目标用户，默认为0，则捕获所有用户，对Android来说，是很需要的参数。

## 模块参数

```
root@vm-server:/home/cfc4n/project/ssldump# bin/ecapture tls -h
OPTIONS:
      --curl=""       curl or wget file path, use to dectet openssl.so path, default:/usr/bin/curl
      --firefox=""    firefox file path, default: /usr/lib/firefox/firefox.
      --gnutls="" libgnutls.so file path, will automatically find it from curl default.
      --gobin=""  path to binary built with Go toolchain.
  -h, --help[=false]    help for tls
  -i, --ifname="" (TC Classifier) Interface name on which the probe will be attached.
      --libssl="" libssl.so file path, will automatically find it from curl default.
      --nspr=""       libnspr44.so file path, will automatically find it from curl default.
      --port=443    port number to capture, default:443.
      --pthread=""    libpthread.so file path, use to hook connect to capture socket FD.will automatically find it from curl.
      --wget=""       wget file path, default: /usr/bin/wget.
  -w, --write=""  write the  raw packets to file as pcapng format.
```

#### \-i参数

`-i`参数为网卡的名字，Linux上默认为`eth0`，Android上默认为`wlan0`，你可以用这个参数自行指定。

## 输出模式

输出格式支持两种格式，文本跟pcapng文件。有三个参数，

1.  默认，全局参数，输出文本结果到stdout
2.  `-l` 全局参数，保存文本结果的文件路径
3.  `-w` 仅tls模块参数，保存pcapng结果的文件路径

## 类库路径

Linux上支持多种类库，不同类库的路径也不一样。

| 类库 | 参数路径 | 默认值 |
| --- | --- | --- |
| openssl/boringssl | –libssl | Linux自动查找，Android为/apex/com.android.conscrypt/lib64/libssl.so |
| gnutls | –gnutls | Linux自动查找，Android pcapng模式暂未支持 |
| nspr/nss | –nspr | Linux自动查找，Android pcapng模式暂未支持 |

### 文本模式

`-l` 或者不加 `-w` 参数将启用该模式。

支持openssl、boringssl、gnutls、nspr/nss等多种TLS加密类库。  
支持DTLS、TLS1.0至TLS1.3等所有版本的加密协议。  
支持`-p`、`-u`等所有全局过滤参数。

### pcapng模式

`-w` 参数启用该模式，并用`-i`选择网卡名，Linux系统默认为`eth0`,Android系统默认为`wlan0`，

仅支持openssl、boringssl两个类库的数据捕获。暂不支持TLS 1.3协议。

### 类库与参数支持

在Linux系统上，大部分类库与参数都是可以支持的。但在Android系统上，因为内核与ARM架构的原因，支持的参数上，有一定的差异。

#### 不同模式的参数支持

`-p`、`-u`两个全局参数，支持文本模式，不支持pcapng模式。这是因为pcapng模式是使用eBPF TC技术实现。  
![](https://image.cnxct.com/2022/09/how-ecapture-works.png)

| 模式 | \-p | \-u | –libsso | –port |
| --- | --- | --- | --- | --- |
| 文本 | ![✅](https://s.w.org/images/core/emoji/14.0.0/svg/2705.svg) | ![✅](https://s.w.org/images/core/emoji/14.0.0/svg/2705.svg) | ![✅](https://s.w.org/images/core/emoji/14.0.0/svg/2705.svg) | ![✅](https://s.w.org/images/core/emoji/14.0.0/svg/2705.svg) |
| pcapng | ![❌](https://s.w.org/images/core/emoji/14.0.0/svg/274c.svg) | ![❌](https://s.w.org/images/core/emoji/14.0.0/svg/274c.svg) | ![✅](https://s.w.org/images/core/emoji/14.0.0/svg/2705.svg) | ![✅](https://s.w.org/images/core/emoji/14.0.0/svg/2705.svg) |

#### 不同模式的类库以协议支持

| 模式 | openssl（类库） | boringssl（类库） | TLS 1.0/1.1/1.2（协议） | TLS 1.3（协议） |
| --- | --- | --- | --- | --- |
| 文本 | ![✅](https://s.w.org/images/core/emoji/14.0.0/svg/2705.svg) | ![✅](https://s.w.org/images/core/emoji/14.0.0/svg/2705.svg) | ![✅](https://s.w.org/images/core/emoji/14.0.0/svg/2705.svg) | ![✅](https://s.w.org/images/core/emoji/14.0.0/svg/2705.svg) |
| pcapng | ![✅](https://s.w.org/images/core/emoji/14.0.0/svg/2705.svg) | ![✅](https://s.w.org/images/core/emoji/14.0.0/svg/2705.svg) | ![✅](https://s.w.org/images/core/emoji/14.0.0/svg/2705.svg) | ![❌](https://s.w.org/images/core/emoji/14.0.0/svg/274c.svg) |

pcapng模式暂时不支持TLS 1.3，[TLS 1.3密钥捕获功能](https://github.com/ehids/ecapture/pull/143)已经开发完成，只是遇到一些BUG，还在解决中。 笔者不是openssl的专家，对TLS 协议也不太熟。需要补充这两块的知识，解决起来成本比较高，也欢迎对这块擅长的朋友一起来解决。

## 与tcpdump联合使用

eCapture基于eBPF TC，实现了流量捕获，并保存到pcapng文件中。基于eBPF Uprobe实现了TLS Master Secret的捕获。并基于Wireshark的[Decryption Secrets Block (DSB)](https://github.com/pcapng/pcapng/pull/54)标准，实现了[gopacket的DSB功能](https://github.com/google/gopacket/pull/1042)，合并网络包与密钥，保存到pcapng中。

eCapture在网络包捕获上，没有tcpdump强大，不支持丰富的参数。你可以用eCapture捕获master secrets，用tcpdump捕获网络包，然后使用wiresahrk自定义设置密钥文件，配合使用。

![](https://image.cnxct.com/2022/09/ecapture-wireshark-scaled.jpg)

#### 网络包捕获

tcpdump 的常规用法，不再赘述。

#### 密钥捕获

同时启用ecapture ，模式可以选文本或者pcapng，都会保存TLS的master secrets密钥数据到ecapture\_masterkey.log中。

#### 网络包查看

用Wireshark打开网络包文件，设置这个master key文件，之后就可以看到TLS解密后的明文了。

配置路径：`Wireshark` –> `Preferences` –> `Protocols` –> `TLS` –> `(Pre)-Master-Secret log filename`  
![](https://image.cnxct.com/2022/09/wireshark-master-secrets.jpg)

## 参数

### 指定路径

#### 默认路径

在Android上，Google使用了boring ssl类库，也就是C++语言在`libssl`基础上的包装。默认情况下，会使用`/apex/com.android.conscrypt/lib64/libssl.so`路径。

#### APP的类库确认

你可以使用`lsof -p {APP PID}|grep libssl`来确认。 若不是默认路径，则可以使用`--libssl`参数来指定。

## 高级用法

如果你需要查看的APP是自定义SSL类库，那么你可以自助修改eCapture来实现。

### 自定义函数名与offset

首先，需要确定HOOK函数的函数名或者符号表地址。

#### 没有源码

如果你没有类库源码可以通过IDA等软件静态分析、动态调试，确定SSL Write的地址offset。在配置填写在`user/module/probe_openssl.go`文件中，对应的probe配置部分。

```
{
    Section:          "uprobe/SSL_write",
    EbpfFuncName:     "probe_entry_SSL_write",
    AttachToFuncName: "SSL_write",
    UprobeOffset:       0xFFFF00, // TODO
    BinaryPath:       binaryPath,
},
```

### offset自动计算

如果你有源码，则可以自行阅读源码确定函数名或者符号表的地址。对于结构体的成员属性读取，则可以通过`offsetof`宏来自动计算。

```
//  g++ -I include/ -I src/ ./src/offset.c -o off
#include <stdio.h>
#include <stddef.h>
#include <ssl/internal.h>
#include <openssl/base.h>
#include <openssl/crypto.h>

#define SSL_STRUCT_OFFSETS               \
    X(ssl_st, session)              \
    X(ssl_st, s3)              \
    X(ssl_session_st, secret)        \
    X(ssl_session_st, secret_length)  \
    X(bssl::SSL3_STATE, client_random) \
    X(bssl::SSL_HANDSHAKE, new_session) \
    X(bssl::SSL_HANDSHAKE, early_session) \
    X(bssl::SSL3_STATE, hs) \
    X(bssl::SSL3_STATE, established_session) \
    X(bssl::SSL_HANDSHAKE, expected_client_finished_)    

struct offset_test
{
    /* data */
    int t1;
    bssl::UniquePtr<SSL_SESSION> session;
};

int main() {
    printf("typedef struct ssl_offsets { // DEF \n");
#define X(struct_name, field_name) \
    printf("   int " #struct_name "_" #field_name "; // DEF\n");
    SSL_STRUCT_OFFSETS
#undef X
    printf("} ssl_offsets; // DEF\n\n");

    printf("/* %s */\nssl_offsets openssl_offset_%d = { \n",
           OPENSSL_VERSION_TEXT, OPENSSL_VERSION_NUMBER);

#define X(struct_name, field_name)                         \
    printf("  ." #struct_name "_" #field_name " = %ld,\n", \
           offsetof(struct struct_name, field_name)); 
    SSL_STRUCT_OFFSETS
#undef X
    printf("};\n");
    return 0;
}
```

![](https://image.cnxct.com/2022/09/goland-ecapture-1.jpg)  
![](https://image.cnxct.com/2022/09/goland-ecapture-2.jpg)

### 参数提取

对于参数，你需要确认被HOOK函数的参数类型，以便确认读取方式，可以参考`kern/openssl_kern.c`内的`SSL_write`函数实现。

## 编译

### ARM Linux 编译

公有云厂商大部分都提供了ARM64 CPU服务器，笔者选择了腾讯云的。在`广州六区`中，名字叫`标准型SR1`(SR1即ARM 64CPU)，最低配的`SR1.MEDIUM2` 2核2G即满足编译环境。可以按照`按量计费`方式购买，随时释放，比较划算。

操作系统选择`ubuntu 20.04 arm64`。

```
ubuntu@VM-0-5-ubuntu:~$sudo apt-get update
ubuntu@VM-0-5-ubuntu:~$sudo apt-get install --yes wget git golang build-essential pkgconf libelf-dev llvm-12 clang-12  linux-tools-generic linux-tools-common
ubuntu@VM-0-5-ubuntu:~$wget https://golang.google.cn/dl/go1.18.linux-arm64.tar.gz
ubuntu@VM-0-5-ubuntu:~$sudo rm -rf /usr/local/go && sudo tar -C /usr/local -xzf go1.18.linux-arm64.tar.gz
ubuntu@VM-0-5-ubuntu:~$for tool in "clang" "llc" "llvm-strip"
do
sudo rm -f /usr/bin/$tool
sudo ln -s /usr/bin/$tool-12 /usr/bin/$tool
done
ubuntu@VM-0-5-ubuntu:~$export GOPROXY=https://goproxy.cn
ubuntu@VM-0-5-ubuntu:~$export PATH=$PATH:/usr/local/go/bin
```

### 编译方法

1.  `ANDROID=1 make` 命令编译支持core版本的二进制程序。
2.  `ANDROID=1 make nocore`命令编译仅支持当前内核版本的二进制程序。

### 当Wireshark被eBPF增强

`Wireshark` + `eBPF` = `eCapture`

![](https://image.cnxct.com/2022/09/sharkbee.jpeg)

## 贡献者

感谢chriskaliX 、chenhengqi、vincentmli、huzai9527、yihong0618、sfx(4ft35t)等朋友的贡献。  
感谢tiann Weishu在Android这个需求上的推动。  
![](https://image.cnxct.com/2022/09/github.com_ehids_ecapture.png)

## 招聘

Leader直招，没有中间商赚差价。面向RASP、HIDS产品，JVM、Linux内核入侵检测等安全研发职位

1.  JAVA资深研发工程师
2.  golang高级工程师
3.  更多职位见 [https://www.cnxct.com/jobs/](https://www.cnxct.com/jobs/?f=wxg)

[![知识共享许可协议](https://www.cnxct.com/attachments/88x31.png)](http://creativecommons.org/licenses/by-nc-sa/3.0/deed.zh)CFC4N的博客 由 [CFC4N](http://www.cnxct.com/) 创作，采用 [知识共享 署名-非商业性使用-相同方式共享（3.0未本地化版本）许可协议](http://creativecommons.org/licenses/by-nc-sa/3.0/deed.zh)进行许可。基于[https://www.cnxct.com](http://www.cnxct.com/)上的作品创作。转载请注明转自：[eCapture旁观者：Android HTTPS明文抓包，无需CA证书](https://www.cnxct.com/ecapture-for-android/)