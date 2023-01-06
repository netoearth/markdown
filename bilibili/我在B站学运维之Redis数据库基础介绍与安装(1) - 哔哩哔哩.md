**本章目录:**  

-   0x00 Redis 介绍
    

-   Redis 特点
    
-   Redis 优势
    
-   Redis 与其他K-V存储异同
    
-   Redis 补充说明
    

-   0x01 Redis单实例安装部署
    

-   1.Windows
    
-   2.Linux
    

-   0x02 Redis 常用工具命令
    

-   redis-server 命令
    
-   redis-cli 命令
    

-   0x03 Redis 内置命令实践
    

![](https://i0.hdslb.com/bfs/article/4aa545dccf7de8d4a93c2b2b8e3265ac0a26d216.png@progressive.webp)

描述: `Remote Dictionary Server(Redis)` 是由`Salvatore Sanfilippo`开发者写的一个`key-value`存储系统。

Redis是一种开源（BSD 许可）,使用ANSI C语言编写、内存中数据结构存储，用作数据库、缓存和消息代理。Redis 提供了诸如字符串、散列、列表、集合、带范围查询的排序集合、位图、超级日志、地理空间索引和流等数据结构。

Redis 内置复制、Lua 脚本、LRU 驱逐、事务和不同级别的磁盘持久化，并通过 Redis Sentinel 和 Redis Cluster 自动分区提供高可用性。

它通常被称为数据结构服务器它有五种类型值（value）: `字符串(String), 哈希(Hash), 列表(list), 集合(sets) 和 有序集合(sorted sets)`等类型。

-   非关系型数据库
    
-   支持数据的持久化,可以将内存中的数据保存在磁盘中,重启的时候可以再次加载进行使用。
    
-   不仅仅支持简单的`key-value`类型的数据,同时还提供list,set,zset,hash等数据结构的存储。
    
-   支持数据的备份即`master-slave`主从模式的数据备份。
    

-   是一个开源的 `key-value` 存储系统,并且性能高体现在IO读写(R>W)
    
-   R是非常轻量级,一个空Redis实例占用的内在只有1M左右,所以不用担心多个Redis实例会额外占用很多内存。
    
-   丰富得数据类型`(String/Hash/List/sets/Sorted sets)`
    
-   所有得操作都是原子性得(要么成功要么失败完全不执行),且多个操作支持事务即原子性(通过MULTI和EXEC指令包起来)
    

-   R不是一个普通的键值存储，它实际上是一个数据结构服务器，支持不同类型的值。
    
-   R有更为复杂得数据结构并提供事务处理机制(原子性操作)
    
-   R运行在内存中但是可以持久化到磁盘之中,在对数据集进行高速读写时需要权衡内存(数据量不能大于硬件内存)
    
-   在磁盘格式方面他们是紧凑的以追加的方式产生的,因为他们并不需要进行随机访问
    

-   官网文档: https://redis.io/documentation
    
-   Redis下载: http://redis.io/download
    

Tips: 稳定版本完全遵循通常的`major.minor.patch`语义版本控制模式。

描述: Redis 在 Windows 下安装是非常之简单(非常不推荐)，可以按照以下步骤进行安装并测试使用。

**安装步骤:**

-   Step 1.Redis 官方不建议在 windows 下使用 Redis，所以官网没有 windows 版本可以下载。还好微软团队维护了开源的 windows 版本，虽然只有 3.2 版本，对于普通测试使用足够了。
    

> 下载地址: https://github.com/MicrosoftArchive/redis/releases

-   Step 2.下载完成之后双击按着引导流程安装。  
    

![WeiyiGeek.win-redis](https://i0.hdslb.com/bfs/article/35527aec575bdf0172a1c4df1d0876d965fef44c.png@942w_366h_progressive.webp)

-   Step 3.Redis 现在可以使用了，下面打开 Redis 程序目录会存储如下可执行相关文件
    

文件介绍：

```
* redis-server.exe：服务端程序，提供 redis 服务

* redis-cli.exe: 客户端程序，通过它连接 redis 服务并进行操作

* redis-check-dump.exe：RDB 文件修复工具

* redis-check-aof.exe：AOF 文件修复工具

* redis-benchmark.exe：性能测试工具，用以模拟同时由 N 个客户端发送 M 个 SETs/GETs 查询 (类似于 Apache 的 ab 工具)

* redis.windows.conf： 配置文件，将 redis 作为普通软件使用的配置，命令行关闭则 redis 关闭

* redis.windows-service.conf：配置文件，将 redis 作为系统服务的配置
```

-   Step 4.我们单击 redis-server.exe，采用默认redis.windows.conf文件进行，配置启动 Redis 服务。
    

```
# 方式1.命令行指定redis服务端配置文件
redis-server.exe redis.windows.conf

# 方式2.安装 redis 到 windows 服务
redis-server --service-install redis.windows.conf
```

![WeiyiGeek.redis-server](https://i0.hdslb.com/bfs/article/f94bd0694a0f80be946f8fe050b2be62d25ecf4f.png@942w_603h_progressive.webp)

Tips : 注：可以把 redis 的路径加到系统的环境变量里，这样就省得再输路径了，后面的那个 redis.windows.conf 可以省略，如果省略，会启用默认的参数。

-   Step 5.如果要以windows服务来运行管理redis，可以采用如下
    

```
# 执行以下命令启动再次 redis：
redis-server --service-start

# 停止 redis 服务：
redis-server --service-stop

# 最后，测试一下 redis 是否能够正常使用：切换到 redis 目录下：E:\tools\redis-3.2.100  下：
redis-cli.exe -h 127.0.0.1 -p 6379
```

描述: 在Linux中有多种方法安装Redis，如`源码编译安装`或者`采用系统三方软件包命令`下载安装。

**环境准备:**

CentOS Linux release 7.6.1810 (Core)  

Linux WeiyiGeek 3.10.0-957.1.3.el7.x86\_64

**安装步骤：**

```
# 1.官网下载redis source code并编译redis Make
$ wget https://download.redis.io/releases/redis-6.2.5.tar.gz
$ tar xzf redis-6.2.5.tar.gz
$ cd redis-6.2.5
$ make
$ make test
  # \o/ All tests passed without errors!
  # Cleanup: may take some time... OK
  # make[1]: 离开目录“/opt/databases/redis-6.2.5/src”

# 2.编译后的redis服务程序redis-server,还有用于测试的客户端程序redis-cli,两个程序位于安装目录 src 目录下：
redis-server  #服务端
redis-cli     #客户端


# 3.启动服务和配置文件
$ cp redis.conf /etc/redis.conf
$ ln -s /opt/redis/redis-6.2.5/src/redis-server /usr/bin/redis-server  #建立软链接
$ ln -s /opt/redis/redis-6.2.5/src/redis-cli /usr/bin/redis-cli
$ redis-server /etc/redis.conf  #利用配置文件启动redis

#4.验证服务是不是成功
#使用测试客户端程序redis-cli和redis服务交互了
$ redis-cli
redis> ping        # PONG  测试redis状态
redis> set foo bar
redis> get foo     # "bar"
```

**From the official Ubuntu PPA**

```
You can install the latest stable version of Redis from the redislabs/redis package repository. Add the repository to the apt index, update it and install:
$ sudo add-apt-repository ppa:redislabs/redis
$ sudo apt-get update
$ sudo apt-get install redis
```

**From Snapcraft**  
You can install the latest stable version of Redis from the Snapcraft marketplace: `$ sudo snap install redis`

描述: 它是redis服务启动的命令，提供数据库存储服务。

**语法格式:**

```
Usage: ./redis-server [/path/to/redis.conf] [options] [-]
       ./redis-server - (read config from stdin)
       ./redis-server -v or --version
       ./redis-server -h or --help
       ./redis-server --test-memory <megabytes>

Examples:
       ./redis-server (run the server with default conf)
       ./redis-server /etc/redis/6379.conf
       ./redis-server --port 7777
       ./redis-server --port 7777 --replicaof 127.0.0.1 8888
       ./redis-server /etc/myredis.conf --loglevel verbose -
       ./redis-server /etc/myredis.conf --loglevel verbose

Sentinel mode:
       ./redis-server /etc/sentinel.conf --sentinel
```

**基础实例:**

```
# (1) 指定配置文件并且启动参数设置最大连接数
$ redis-server --maxclients 100000 /etc/redis.conf

# (2) 使用 TLS 模式手动运行 Redis 服务器(您可以指定port 0完全禁用非 TLS 端口)
$ redis-server --tls-port 6379 --port 0 \
--tls-cert-file ./tests/tls/redis.crt \
--tls-key-file ./tests/tls/redis.key \
--tls-ca-cert-file ./tests/tls/ca.crt

# (3) 指定用户启动redis-server
setsid sudo -u redis redis-server /etc/redis/redis.conf
```

描述: redis-cli 是Redis命令行界面，一个简单的程序，允许直接从终端向Redis发送命令，并读取服务器发送的回复。

**命令安装:**

```
# Ubuntu、Debian
apt install redis-cli

# CentOS
yum install redis-cli
```

**两种主要模式:**

-   交互模式，其中有一个 REPL（读取评估打印循环），用户可以在其中键入命令并获得回复；以及另一种模式，其中命令作为 的参数发送redis-cli、执行并打印在标准输出上。
    
-   交互模式下，redis-cli具有基本的行编辑功能，提供良好的打字体验。
    

**Syntax & Argument:**

```
$ redis-cli -h localhst -p 6379 redis-Command
  -h ip : Redis-server 主机地址
  -p Port : Redis-server 服务端口
  -a Auth : 认证密码
  -n [0-15] : Redis 数据库 db0~db15
  -x : 读取STDIN的最后一个参数。
  -u <uri>：选项和有效的 URI来提供部分或全部信息：
  -r <count> : 第一个说明运行命令的次数
  -i <delay> : 第二个配置不同命令调用之间的延迟，以秒为单位（可以指定十进制数，如 0.1 以表示 100 毫秒）
  --rdb 提供远程备份工具，允许将 RDB 文件从任何 Redis 实例传输到运行 redis-cli. 要使用此模式
  --slave : salve 模式允许检查主服务器在复制流中向其从属服务器发送的内容
  --scan : 扫描Keyname描键空间
  --bigkeys ：大键扫描
  --pattern : 底层模式匹配功能可用通配符
  --latency : 使用此选项，CLI 运行一个循环，在该循环中将PING命令发送到 Redis 实例，并测量获得回复的时间(100秒执行一次)。
  --latency-histor : 有时研究最大和平均延迟如何随时间演变是有用的。(15s执行一次)
  --latency-dist : 使用彩色终端显示一系列延迟
  --intrinsic-latency <test-time> : 测试的时间以秒为单位，并指定redis-cli应检查当前运行的系统延迟的秒数。
  --lru-test : 用作带有LRU eviction的缓存,模拟命中率对于正确配置缓存非常有用
  --raw : 对应答使用原始格式(当STDOUT不是tty时默认)。
  --no-raw: 不使用原始格式
  --cluster: 集群操作命令
  --cert: 证书
  --key: 密钥
  --cacert: CA证书

# Examples:
  $ redis-cli -h host -p port -a password     #链接其他Redis主机
  $ redis-cli -h host -p port -a password -c  #以Redis集群模式访问
  $ redis-cli -h host -p port shutdown        #可以通过杀进程的方式强制关闭服务也可采用下面这种

  $ redis-cli --scan --pattern '*:12345*'
  $ redis-cli --raw  #redis-cli --raw
  $ redis-cli --eval myscript.lua key1 key2 , arg1 arg2 arg3

  $ cat /etc/passwd | redis-cli -x set mypasswd
  $ redis-cli get mypasswd

  $ redis-cli -r 100 lpush mylist x
  $ redis-cli -r 100 -i 1 info | grep used_memory_human:
```

Tips: 默认情况下，redis-cli使用普通 TCP 连接连接到 Redis。您可以使用`SSL / TLS`启用--tls选项，连同`--cacert`或 `--cacertdir`配置一个受信任的根证书捆绑或目录。

如果目标服务器需要使用客户端证书进行身份验证，您可以使用--cert和指定证书和相应的私钥--key。

**实际案例:**

-   Step 1.命令行操作
    

```
# 0.命令行标准输出、原始输出、文件输出、CSV 输出
$ redis-cli incr mycounter        # (integer) 7
$ redis-cli incr mycounter > /tmp/output.txt && cat /tmp/output.txt # 8
$ redis-cli --raw incr mycounter  # 9
$ redis-cli lpush mylist a b c d  && redis-cli --csv lrange mylist 0 -1 # "d","c","b","a"

# 1.命令行验证redis服务状态
$ redis-cli -h 127.0.0.1 -p 6390 ping  # PONG
$ redis-cli -h 127.0.0.1 -p 6390-a password -c  ping # PONG 集群模式访问
$ redis-cli -u redis://pssw0rd@redis-16379.hosted.com:16379/0 ping # PONG

# 2.从其他程序获取输入，例如为了/etc/services在我的计算机上为文件内容设置一个 Redis 键
$ redis-cli -x set foo < /etc/services # OK
$ redis-cli getrange foo 0 50
"#\n# Network services, Internet style\n#\n# Note that "

# 3.通过管道符给 redis 服务端发送命令,类似于是由用户交互输入的一样。
$ cat /tmp/commands.txt
  # set foo 100
  # incr foo
  # append foo xxx
  # get foo
$ cat /tmp/commands.txt | redis-cli
  # OK
  # (integer) 101
  # (integer) 6
  # "101xxx"
$ echo "ping\ninfo" | redis-cli -a 123456


# 4.连续运行相同的命令,默认情况下，间隔（或延迟）设置为 0，因此命令会尽快执行：
$ redis-cli -r 5 incr foo
  # (integer) 1
  # (integer) 2
  # (integer) 3
  # (integer) 4
  # (integer) 5

# 如要永远运行相同的命令则需使用`-r -1`,此时我们可以随时间监控 RSS 内存大小
$ redis-cli -a 123456 -r -1 -i 1 INFO | grep rss_human
  # used_memory_rss_human:4.64M

# 5.运行 Lua 脚本(在Redis 3.2后出现的Lua调试工具)
$ cat /tmp/script.lua
return redis.call('set',KEYS[1],ARGV[1])    # foo将填充KEYS数组，bar该ARGV数组。
$ redis-cli --eval /tmp/script.lua foo , bar # OK

# 6.命令行查看指定主机地址以及db0序号的库中的所有key的名称
$ redis-cli -h 127.0.0.1 -a weiyigeek.top -n 0 keys "*"
  #  1) "SEARCH:PROJECTSINFO_BY_PARENTID_LIST:95"
  #  2) "ALLOW:SEARCH_BY_SFZH:test"
  #  3) "SEARCH:PROJECTS_BY_ID:100"

# 7.Redis 数据库 db0 迁移到其它Redis数据库指定 dbn 中(非常值得学习)
$ redis-cli -h localhost -a weiyigeek.top -n 0 keys "*" | while read key
do
  redis-cli -h localhost -a weiyigeek.top -n 0 --raw dump $key | perl -pe 'chomp if eof' | redis-cli -h localhost -a weiyigeek.top -n 12 -x restore $key 0
done
# - perl -pe 'chomp if eof' : 删除文件中最后一个换行符
# 执行效果:
  # $ redis-cli -h localhost
  # localhost:6379> AUTH weiyigeek.top
  # OK
  # localhost:6379> select 12
  # OK
  # localhost:6379[12]> DBSIZE
  # (integer) 65
```

**特殊模式:**

```
# 用于显示有关 Redis 服务器的连续统计信息的监控工具。
$ redis-cli -i 5 --stat  # 修改其默认刷新评率
------- data ------ --------------------- load -------------------- - child -
keys       mem      clients blocked requests            connections
506        1015.00K 1       0       25 (+1)             7
506        3.40M    51      0       60461 (+60436)      57

# 用作键空间分析器,它扫描数据集的大键，但也提供有关数据集包含的数据类型的信息。
# 在输出的第一部分中报告了遇到的每个大于先前较大键（相同类型）的新键,摘要部分提供有关 Redis 实例内数据的一般统计信息。
$ redis-cli --bigkeys
[00.00%] Biggest string found so far 'key-419' with 3 bytes
[05.14%] Biggest list   found so far 'mylist' with 100004 items
[35.77%] Biggest string found so far 'counter:__rand_int__' with 6 bytes
[73.91%] Biggest hash   found so far 'myobject' with 3 fields

-------- summary -------
Sampled 506 keys in the keyspace!
Total key length in bytes is 3452 (avg len 6.82)
Biggest string found 'counter:__rand_int__' has 6 bytes
Biggest   list found 'mylist' has 100004 items
Biggest   hash found 'myobject' has 3 fields

# 获取库中所有Key的命令，此处使用--scan已非阻塞的方式进行扫描Keyspace
$ redis-cli -a 123456  --scan  | head -6
"mylist"
"demo1"
"myhash"
"key:__rand_int__"
"demo"
"counter:__rand_int__"
$ redis-cli --scan --pattern '*-11*'  # SCAN命令的底层模式匹配功能--pattern
key-114
key-117

# 监控Redis中执行的命令,它将打印 Redis 实例接收到的所有命令
$ redis-cli monitor # OK
1460100081.165665 [0 127.0.0.1:51706] "set" "foo" "bar"

# 监控Redis实例的延迟查询，并了解延迟的最大值、平均值和分布。
$ redis-cli --latency  # 每秒发生 100 次，并且在控制台中实时更新统计信息
min: 0, max: 1, avg: 0.19 (427 samples)
$ redis-cli --latency-history  # 研究最大和平均延迟如何随时间演变,其原理与--latency，但每 15 秒（默认情况下）从头开始一个新的采样会话
min: 0, max: 1, avg: 0.14 (1314 samples) -- 15.01 seconds range
$ redis-cli --latency-dist # 表示不同样本百分比的彩色输出，以及表示不同延迟数字的不同 ASCII 字符
$ ./redis-cli --intrinsic-latency 5 # 此命令必须在要运行 Redis 服务器的计算机上执行，而不是在其他主机上执行。(只能本地测试)
Max latency so far: 1 microseconds.

# RDB 文件的远程备份(可用于迁移备份),简单但有效的方法可确保您拥有 Redis 实例的灾难恢复 RDB 备份
$ redis-cli --rdb /tmp/dump.rdb
  # SYNC sent to master, writing 13256 bytes to '/tmp/dump.rdb'
  # Transfer finished with success.
$ echo $? # 1

# 它允许检查主服务器在复制流中向其从属服务器发送的内容，以便将写入传播到其副本
$ redis-cli --slave
  # SYNC with master, discarding 13256 bytes of bulk transfer...
  # SYNC done. Logging commands from master.
  # "PING"
  # "SELECT","0"
  # "set","foo","bar"
  # "PING"
  # "incr","mycounter"

# 执行 LRU 模拟根据键的数量和为缓存分配的内存量（通过maxmemory指令指定），缓存命中和未命中的数量会发生变化(该程序每秒显示统计数据)。
# 使用--lru-test则需要指定Key数量以及配置一个maxmemory
$ ./redis-cli --lru-test 10000000
120750 Gets/sec | Hits: 48774 (40.39%) | Misses: 71976 (59.61%)
122500 Gets/sec | Hits: 49052 (40.04%) | Misses: 73448 (59.96%)
127000 Gets/sec | Hits: 50870 (40.06%) | Misses: 76130 (59.94%) # 59% 的失误率可能是不可接受的。所以我们知道100MB的内存是不够的。让我们尝试使用半千兆字节。
# 如您所见，在最初几秒钟内，缓存开始填充。未命中率后来稳定到我们在很长一段时间内可以预期的实际数字：
140000 Gets/sec | Hits: 135376 (96.70%) | Misses: 4624 (3.30%)
141250 Gets/sec | Hits: 136523 (96.65%) | Misses: 4727 (3.35%)
140250 Gets/sec | Hits: 135457 (96.58%) | Misses: 4793 (3.42%)
140500 Gets/sec | Hits: 135947 (96.76%) | Misses: 4553 (3.24%)

# CLI 能够仅使用PUBLISH命令在 Redis Pub/Sub 通道中发布消息
$ redis-cli psubscribe '*'
$ redis-cli PUBLISH mychannel mymessag
```

![WeiyiGeek.Pub/Sub](https://i0.hdslb.com/bfs/article/0e4ecde781c788f3b1d72e568e15d5b45dcc0896.png@942w_338h_progressive.webp)

-   Step 2.交互式操作
    

```
$ redis-cli -h localhost

# 1.认证
auth 123456

# 2.验证状态
ping  # PONG

# 3.处理连接和重新连接
# connect通过指定我们要连接的主机名和端口，在交互模式下使用该命令可以连接到不同的实例：
connect weiyigeek.top 6379

# 4.通常在检测到断开连接后，CLI 总是会尝试透明地重新连接：如果尝试失败，则显示错误并进入断开连接状态。
# 以下是断开和重新连接的示例：
127.0.0.1:6379> debug restart
Could not connect to Redis at 127.0.0.1:6379: Connection refused
not connected> ping # PONG
127.0.0.1:6379> (now we are connected again)

# 5.执行重新连接时，redis-cli自动重新选择上次选择的数据库编号。
$ redis-cli
127.0.0.1:6379> multi
OK
127.0.0.1:6379(TX)> ping  # QUEUED

# 6.编辑、历史、完成和提示
# 因为redis-cli使用 linenoise线编辑库，所以一直具备线编辑能力，不依赖libreadline或其他可选库。
# 历史记录在 CLI 重新启动之间保留在用户主目录内调用的.rediscli_history文件中。
127.0.0.1:6379> Z<TAB>  # 通过按箭头键（向上和向下）访问已执行命令的历史记录，以及通过按 TAB 键来执行命令名称补全。

# 7.运行相同的命令 N 次
127.0.0.1:6379> 5 incr mycounter
(integer) 1
....
(integer) 5

# 8.命令帮助显示有关给定类别的所有命令
help @<category>
# @generic、@list、@set、@sorted_set、@hash、 @pubsub、@transactions、@connection、@server、@scripting、 @hyperloglog


# 9.Redis Server 配置首选项的两种方式：
# - 1.直接在交换模式下使用`:set`命令
# - 2.通过在 CLI 中键入命令或将其添加到.redisclirc文件中来设置以下首选项:
:set hints   - 启用语法提示
:set nohints - 禁用语法提示
```

**学习参考:**

-   Redis Lua调试器文档: https://redis.io/topics/ldb
    

描述: Redis 在交互模式中命令使用说明请参考如下地址(https://redis.io/commands)

**内置命令基础使用演示:**

```
# 连接Redis数据库(单机、集群)
redis-cli -h 127.0.0.1
redis-cli -h 127.0.0.1 -c
> keys *  # 未认证: (error) NOAUTH Authentication required.

# Redis 登陆验证
> auth weiyigeek.top
OK

# ping 测试链接是否存活(状态为PONG则表示正常)
> ping
PONG


# info 服务器得统计信息(单体、集群)
[1]> info
# Server
  # redis_version:5.0.8
  # redis_git_sha1:00000000
  # redis_git_dirty:0
  # redis_build_id:ce75a617c591114f
  # redis_mode:standalone
  # os:Linux 4.18.0-147.5.1.el8_1.x86_64 x86_64
[1]> info replication  # 集群


# select 选择redis数据库号(db0~15),可以选择任意一个仓库进行数据库存取
> select 1
> select 16
(error) ERR DB index is out of range

# dbsize 返回当前数据库中的Key数目
> select 1
[1]> DBSIZE
(integer) 2
[1]> dbsize   # 命令不区分大小写
(integer) 2

# 设置与获取 key
127.0.0.1:6379> set foo bar   # 设置 key
127.0.0.1:6379> keys *        # 获取所有得keys
127.0.0.1:6379> get foo       # 获取key得值 "bar"
redis> DBSIZE                 # 返回当前数据库的 key 的数量 (integer) 6
redis> DEBUG OBJECT foo       # DEBUG OBJECT key


# move命令将Redis键移植到其它库中
# 例如：将db0中的mykey键移动到1号仓库，然后进入db1进行查看键
> move mykey 1
> select 1
> keys *


# flushdb 删除当前选择与所有数据库中的所有key(非常小心，等同于rm -rf /)
[1]> FLUSHDB #删除当前数据库的所有key
[1]> DBSIZE
(integer) 0
redis> FLUSHALL #删除所有数据库的所有key
redis> DBSIZE
(integer) 0


# COMMAND 命令相关
redis> COMMAND          # 返回所有的Redis命令的详细信息，以数组形式展示
redis> COMMAND COUNT    # 返回命令总数 (integer) 200
redis> COMMAND GETKEYS MSET a b c d e f   #获取给定命令的所有键
redis> COMMAND INFO get set eval  #获取 redis 命令的描述信息


# 关闭 redis 服务器(server) 并保存数据
redis> SHUTDOWN [NOSAVE] [SAVE]

# 关闭并从客户端退出当前连接
redis[1]> quit

# 内置一部分shell命令
redis> echo "Redis Test-WeiyigGeek"    # 指定字符串输出(执行打印字符串)
redis> time                 # 返回当前服务器时间
1) "1555514501"
2) "504765"

# 实时打印出 Redis 服务器接收到的命令，调试用
redis> MONITOR

# 查询执行时间指的是不包括像客户端响应(talking)、发送回复等 IO 操作，而单单是执行一个查询命令所耗费的时间
redis> SLOWLOG LEN  #管理 redis 的慢日志 ,查看当前日志的数量 (integer) 14
redis> SLOWLOG RESET #清空 slowlog OK 上面 LEN 变成0


# Redis 数据备份与恢复
redis > SAVE            # SAVE 命令在 redis 安装目录中创建dump.rdb文件
redis > CONFIG GET dir  # 需将备份文件 (dump.rdb) 移动到 redis 安装目录并启动服务即可
1) "dir"
2) "/usr/local/redis/bin"  # 输出的 redis 安装目录
redis > BGSAVE             # 后台执行保存

# 返回最近一次 Redis 成功将数据保存到磁盘上的时间，以 UNIX 时间戳格式表示
redis> lastsave
(integer) 1555514754


# 通过其CONFIG命令可以查看或设置配置项
redis> CONFIG RESETSTAT   #重置服务器得统计信息
redis> CONFIG GET *   #查看所有配置
13) "pidfile"
14) "/var/run/redis_6379.pid"
> CONFIG GET loglevel
1) "loglevel"
2) "notice"
redis > CONFIG SET loglevel "notice" #修改配置
redis > CONFIG set requirepass "runoob" # 通过 redis 的配置文件或者在redis-cli设置密码参数
redis > CONFIG REWRITE     # 将修改写入到 redis.conf 中
OK

# 登陆Redis-server的客户端信息查看
> CLIENT id    #当前连接ID (integer) 1257
> CLIENT setname weiyi  #设置服务名称
> CLIENT getname     #获取通过 CLIENT SETNAME 命令设置的服务名称 "weiyi"
> CLIENT PAUSE 100  #阻塞客户端命令一段时间
> CLIENT LIST  #返回连接到 redis 服务的客户端列表
id=1256 addr=127.0.0.1:41306 fd=7 name=weiyi age=708 idle=0 flags=N db=0 sub=0 psub=0 multi=-1 qbuf=26 qbuf-free=32742 obl=0 oll=0 omem=0 events=r cmd=client
> CLIENT kill 127.0.0.1:41306 #关闭客户端连接
OK
```

![](https://i0.hdslb.com/bfs/article/db75225feabec8d8b64ee7d3c7165cd639554cbc.png@progressive.webp)

>     如果你觉得这个专栏还不错的，请给这篇专栏点个赞、投个币、收个藏，这将对我有很大帮助！  
>     文章来源: https://weiyigeek.top  
>     WeiyiGeek Blog - 为了能到远方，脚下的每一步都不能少。  

![](https://i0.hdslb.com/bfs/article/02db465212d3c374a43c60fa2625cc1caeaab796.png@progressive.webp)

欢迎各位志同道合的朋友一起学习交流，如文章有误请在下方留下您宝贵的经验知识，个人邮箱地址【master#weiyigeek.top】