**本章目录:**

0x01 Redis 安全优化

-   1.Security
    

-   非特权运行
    
-   文件权限
    
-   接口绑定
    
-   更改默认服务端口
    
-   认证配置
    
-   禁用特定命令
    
-   日志记录
    
-   防范字符串转义和 NoSQL 注入
    
-   防范由外部客户端精心挑选的输入触发的攻击
    
-   防火墙限制访问
    
-   禁止redis中存储敏感的明文数据
    
-   Redis 安全配置总结示例
    

-   2.Performance Optimization
    

-   关键优化项
    
-   Redis 性能优化总结示例
    

![](https://i0.hdslb.com/bfs/article/4aa545dccf7de8d4a93c2b2b8e3265ac0a26d216.png@progressive.webp)

描述: Redis提供的访问控制、代码安全问题、选择恶意输入可从外部触发的攻击等功能，需要我们运维人员进行相应的配置提高安全性。

描述: Redis 不需要 root 权限即可运行，建议以仅用于此目的的非特权redis用户身份运行它，此种方式能最大程度防止`CONFIG SET/GET` 目录和其他类似运行时配置指令的可能性。。

```
#设置一个单独的redis账户很有必要，redis crackit就利用到了root用户的特性来重置authorized_keys。首先创建一个redis账户，然后通过该账户启动。
$ useradd redis
$ setsid sudo -u redis redis-server /etc/redis.conf
$ ps -elf|grep redis   #可以看到是redis用户启动
  # 4 S root      9048     1  0  80   0 - 59753 poll_s 19:43 ?        00:00:00 sudo -u redis redis-server /etc redis.conf
  # 4 S redis     9049  9048  0  80   0 - 38471 ep_pol 19:43 ?        00:00:00 redis-server
```

描述: 因为redis密码明文存储在配置文件中，所以我们需要限制redis文件目录访问权限，如设置redis的主目录权限为700`(rwx------)`,如果redis.conf配置文件独立于redis主目录权限修过为600`(rw-------)`。

例如:  

```
# 文件权限
chmod 700 /opt/redis/redis-5.0.4/
chmod 600 /etc/redis.conf

# 所属者、组
chown redis:redis /etc/redis.conf
chown redis:redis /opt/redis/redis-5.0.4/
```

描述: 除了网络中受信任的客户端之外，每个人都应该拒绝访问 Redis 端口，因此运行 Redis 的服务器应该只能由使用 Redis 实现应用程序的计算机直接访问。

假如服务器有两个网络接口(一个A区域、一个B区域)，如果只需要A区域的机器访问则只绑定到A区域网络接口中，如服务器自身访问则只绑定到本地回环接口上。

```
# 通过在redis.conf文件中添加如下一行，可以将 Redis 绑定到单个接口：
bind 127.0.0.1 192.168.1.200
```

Tips：注意除了您可以绑定IPV4以为你还可绑定IPV6  

描述: 除了我们可以指定绑定的接口外，我们还可以更改默认的redis服务端口，可以防止黑客针对于默认Redis服务扫描探测。

```
# 将默认的服务断开从6379变成63791
port 63791
```

描述: 为Redis服务端设置一个认证密码是非常必须，下面讲解 Redis 配置密码认证的几种方式总结:

**操作流程:**

```
# 1.通过redis.conf文件中进行配置，此种方式修改后需要重启Redis。
vim /etc/redis.conf
requirepass WeiyiGeek  # WeiyiGeek 即认证密码
masterauth  WeiyiGeek  # 配置主节点认证密码, 注意若master配置了密码则slave也要配置相应的密码参数否则无法进行正常复制的

# 2.通过命令行进行配置，此种方式的优点无需重启Redis。
redis 127.0.0.1:6379[1]> config set requirepass my_redis
OK
redis 127.0.0.1:6379[1]> config get requirepass
1) "requirepass"
2) "my_redis"
```

**使用密码验证登陆Redis服务器:**

```
# 方式1:密码明文会被记录到系统命令执行历史中(极其不推荐/不安全)
redis-cli -h 127.0.0.1 -p 6379 -a WeiyiGeek

# 方式2:交互式进行配置
redis-cli -h 127.0.0.1 -p 6379
redis 127.0.0.1:6379> auth WeiyiGeek # OK
```

非常注意: AUTH 命令与其他所有 Redis 命令一样，以未加密的方式发送，因此它无法防范对网络有足够访问权限以执行窃听的攻击者, 所以对应高敏感的数据建议配置TLS 支持(Redis 在所有通信通道上都可选地支持 TLS)以加密数据与命令传输。

描述: 我们可以禁用 Redis 中的命令或将它们重命名为不可猜测的名称，以便普通客户端仅限于指定的一组命令，比如漏洞就利用config/save两个命令完成攻击 。

由于redis无用户权限限制，建议将危险的命令使用rename配置项进行禁用或重命名，这样外部不了解重命名规则攻击者，就不能执行这类命令`FLUSHDB, FLUSHALL, KEYS, PEXPIRE, DEL, CONFIG, SHUTDOWN, BGREWRITEAOF, BGSAVE, SAVE, SPOP, SREM, RENAME, DEBUG, EVAL`

例如: 普通用户可能无法调用Redis `CONFIG 命令`来更改实例的配置，但提供和删除实例的系统应该能够这样做。

```
# redis.conf 配置文件
# 方式1.CONFIG / FLUSHALL命令被重命名为一个不可猜测的名称
rename-command CONFIG b840fc02d524045429941cc15f59e41cb7be6c52
rename-command FLUSHALL b840fc02d524045429941cc15f59e41cb7be6c53

# 方式2.通过将其重命名为空字符串来完全禁用它（或任何其他命令）
rename-command CONFIG ""
rename-command FLUSHALL  ""
```

Tips: 注意配置后需要重新redis-server服务。

描述: 为Redis创建访问(或Debug)日志(根据需求设置)，在建立Redis蜜罐时，如果有攻击尝试时，就开业及时发现监控redis安全状态, 以及可以监控`cmdstat_*`指标信息报警;

```
# 执行info commandstats 看出命令执行的次数、命令耗费的 CPU 时间(单位毫秒)、执行每个命令耗费的平均 CPU 时间(单位毫秒)
cmdstat_get:calls=2,usec=15,usec_per_call=7.50
cmdstat_select:calls=1,usec=9,usec_per_call=9.00
cmdstat_keys:calls=4,usec=1948,usec_per_call=487.00
cmdstat_auth:calls=3123,usec=8291,usec_per_call=2.65
```

**日志记录配置:**

```
logfile "/usr/local/redis/redis.log" #日志文件存放目录
loglevel verbose  #记录访问信息
```

描述: Redis 协议没有字符串转义的概念，所以一般情况下使用普通客户端库是不可能注入的, 但有可能会通过EVAL和EVALSHA命令执行的 Lua 脚本来构造恶意脚本。

```
> eval "return {KEYS[1],KEYS[2],ARGV[1],ARGV[2]}" 2 key1 key2 first second
```

Tips : EVALSHA 通过其 SHA1 摘要评估缓存在服务器端的脚本。脚本使用SCRIPT LOAD命令缓存在服务器端。该命令在其他方面与EVAL相同。

描述: 有可能攻击者构造恶意的数据结构插入到 Redis 数据库中, 这可能会触发Redis 内部实现的数据结构的病态（最坏情况）算法复杂性。

例如，攻击者可以通过 Web 表单将一组已知散列到同一桶的字符串提供到散列表中，以便将 O(1)预期时间（平均时间）变为O(N )最坏的情况，消耗比预期更多的 CPU，并最终导致拒绝服务。

解决办法: 为了防止这种特定的攻击，Redis 对哈希函数使用了每次执行的伪随机种子。

描述: 前面针对Redis-server服务层面进行安全配置，此处针对网络层面进行限制，只允许指定的IP地址进行访问，在主机上配置防火墙的优点是防止同一网段的东西流量。

在Linux上系统防火墙设置命令:

```
iptables -A INPUT -s x.x.x.x -p tcp --dport 6379 -j ACCEPT  #如果需要其他机器访问或者设置了slave模式，那就记得加上相应的防火墙设置(Centos6)
firewall-cmd --add-rich-rule="rule family="ipv4" source address="x.x.x.x" port protocol="tcp" port="6379" accept" --permanent  #(Centos7)
```

在Windows上系统防火墙设置命令:

```
New-NetFirewallRule -Name "redis-server-access" -DisplayName "redis-server" -Description "redis-server 客户端访问规则" -Direction Inbound -LocalPort 6379 -RemoteAddress x.x.x.x -Protocol TCP -Action Allow -Enabled True
Get-NetFirewallRule -Name "redis-server-access"  | Format-Table
```

禁止redis中存储敏感的明文数据

描述: Redis设计旨在提供高性能的KV服务，至少目前在权限访问控制和数据持久化方面比较弱化，所以从应用层面上，不建议使用Redis来存储敏感信息，例如鉴权的密码。

安全配置示例:

```
# 配置文件 vim /etc/redis/redis.conf
# 1.信任的内网运行,尽量避免有公网访问（如果存在内网中其他固定IP则需要设置防火墙）
bind 127.0.0.1

# 2.绑定redis监听的网络接口(通过redis配置项bind,可同时绑定多个IP), 把6379改为其他得端口(或者采用unix管道进行数据管理)
port 63791

# 3.开启redis密码认证,并设置高复杂度密码设置，因查询效率高，auth这种命令每秒能处理10w次以上(所以需要增加强度)
# echo -e "weiyigeek"|sha256sum
requirepass 097575a79efcd7ea7b1efa2bcda78a4fc7cbd0820736b2f2708e72c3d21f8b61

# 4.日志文件存放目录以及记录redis访问信息。
# debug (a lot of information, useful for development/testing)
# verbose (many rarely useful info, but not a mess like the debug level)
# notice (moderately verbose, what you want in production probably) 默认
# warning (only very important / critical messages are logged)
logfile "/usr/local/redis/redis.log"
loglevel verbose

# 5.默认情况下，启用保护模式。只有在以下情况下才应禁用(no)它
# - 您确定希望其他主机的客户端连接到Redis
# - 即使没有配置身份验证，也没有特定的接口集
# - 使用“bind”指令显式列出。
protected-mode yes

# 6.重命名特殊命令（根据需求）
# `FLUSHDB, FLUSHALL, KEYS, PEXPIRE, DEL, CONFIG, SHUTDOWN, BGREWRITEAOF, BGSAVE, SAVE, SPOP, SREM, RENAME, DEBUG, EVAL`
rename-command CONFIG b840fc02d524045429941cc15f59e41cb7be6c52
rename-command FLUSHDB b840fc02d524045429941cc15f59e41cb7be6c53
rename-command FLUSHALL b840fc02d524045429941cc15f59e41cb7be6c54
rename-command EVAL b840fc02d524045429941cc15f59e41cb7be6c55
rename-command DEBUG b840fc02d524045429941cc15f59e41cb7be6c56
rename-command SHUTDOWN b840fc02d524045429941cc15f59e41cb7be6c7
```

## 2.Performance Optimization

描述: Redis开发和运维人员更加关注的是Redis本身的一些配置优化，例如AOF和RDB的配置优化、数据结构的配置优化等，但是对于操作系统是否需要针对Redis做一些配置优化不甚了解或者不太关心，然而事实证明一个良好的系统操作配置能够为Redis服务良好运行保驾护航。

-   **Step 1.vm.overcommit\_memory 最佳实践**  
    Redis在启动时可能会出现这样的日志, 然后弄清楚什么是overcommit?  
    描述: Linux 操作系统对大部分申请内存的请求都回复yes以便能运行更多的程序。因为申请内存后并不会马上使用内存，这种技术叫做overcommit。
    

```
# 如果Redis在启动时有上面的日志，说明`vm.overcommit_memory=0`，Redis提示把它设置为1。
# WARNING overcommit_memory is set to 0! Background save may fail under low memory condition. To fix this issue add 'vm.overcommit_memory = 1' to /etc/sysctl.conf and then reboot or run the
command 'sysctl -w vm.overcommit_memory=1' for this to take effect.

# 注意：本文的可用内存代表物理内存与swap之和。
# 最佳实践
- Redis建议把这个值设置为1是为了让fork能够在低内存下也执行成功(设置合理的maxmemory保证机器有20%~30%的闲置内存)。
- 集中化管理aof重写和rdb的bgsave。
```

Tips : 日志中的 Background save 代表的是 bgsave 和 bgrewriteaof，如果当前可用内存不足，操作系统应该如何处理fork。如果vm.overcommit\_memory=0，代表如果没有可用内存，就申请内存失败，对应到Redis就是fork执行失败，在Redis的日志会出现：`Cannot allocate memory`

-   **Step 2.vm.swapniess 最佳实践**  
    描述: swap对于操作系统来比较重要，当物理内存不足时，可以swap out一部分内存页，以解燃眉之急。但世界上没有免费午餐，swap空间由硬盘提供，对于需要高并发、高吞吐的应用来说，磁盘IO通常会成为系统瓶颈。在Linux中，并不是要等到所有物理内存都使用完才会使用到swap，系统参数swppiness会决定操作系统使用swap的倾向程度。swappiness的取值范围是0~100，swappiness的值越大，说明操作系统可能使用swap的概率越高，swappiness值越低，表示操作系统更加倾向于使用物理内存。
    

如果Linux > 3.5的情况下 vm.swapniess=1 (`宁愿swap也不要OOM killer`) 否则 vm.swapniess=0 (`宁愿OOM killer也不用swap`) 从而实现如下两个目标：

> 1.物理内存充足时候，使Redis足够快。  
> 2.物理内存不足时候，避免Redis死掉(如果当前Redis为高可用，死掉比阻塞更好)。

运维提示：OOM(Out Of Memory) killer机制是指Linux操作系统发现可用内存不足时，强制杀死一些用户进程（非内核进程），来保证系统有足够的可用内存进行分配。

-   **Step 3.kernel.mm.transparent\_hugepage.enabled 最佳实践**  
    Redis在启动时可能会看到如下日志：
    

```
WARNING you have Transparent Huge Pages (THP) support enabled in your kernel. This will create latency and memory usage issues with Redis. To fix this issue run the command 'echo never > /sys/kernel/mm/transparent_hugepage/enabled' as root, and add it to your /etc/rc.local in order to retain the setting after a reboot. Redis must be restarted after THP is disabled.
```

Tips : 从提示看Redis建议修改Transparent Huge Pages (THP)的相关配置，Linux kernel在2.6.38内核增加了Transparent Huge Pages (THP)特性 ，支持大内存页(2MB)分配，默认开启。当开启时可以降低fork子进程的速度，但fork之后，每个内存页从原来4KB变为2MB，会大幅增加重写期间父进程内存消耗。同时每次写命令引起的复制内存页单位放大了512倍，会拖慢写操作的执行时间，导致大量写操作慢查询。

因此Redis日志中建议将此特性进行禁用，禁用方法如下：`echo never > /sys/kernel/mm/transparent_hugepage/enabled`

-   **Step 4.Transparent Huge Pages**  
    Redis在启动时可能会看到如下日志：`WARNING you have Transparent Huge Pages (THP) support enabled in your kernel. This will create latency and memory usage issues with Redis. To fix this issue run the command 'echo never > /sys/kernel/mm/transparent_hugepage/enabled' as root, and add it to your /etc/rc.local in order to retain the setting after a reboot. Redis must be restarted after THP is disabled.`
    

从提示看Redis建议修改Transparent Huge Pages (THP)的相关配置，Linux kernel在2.6.38内核增加了Transparent Huge Pages (THP)特性 ，支持大内存页(2MB)分配，默认开启。当开启时可以降低fork子进程的速度，但fork之后，每个内存页从原来4KB变为2MB，会大幅增加重写期间父进程内存消耗。同时每次写命令引起的复制内存页单位放大了512倍，会拖慢写操作的执行时间，导致大量写操作慢查询。例如简单的incr命令也会出现在慢查询中。因此Redis日志中建议将此特性进行禁用，禁用方法如下：

```
# 配置机器重启后THP配置依然生效
tee -a /etc/rc.local <<'EOF'
echo never >  /sys/kernel/mm/transparent_hugepage/enabled
EOF
```

  

-   **Step 5.OOM killer 优化配置**  
    OOM killer会在可用内存不足时选择性的杀掉用户进程,它会为每个用户进程设置一个权值，这个权值越高，被“下手”的概率就越高，反之概率越低。每个进程的权值存放在`/proc/{progress_id}/oom_score`中，这个值是受`/proc/{progress_id}/oom_adj`的控制，oom\_adj在不同的Linux版本的最小值不同，可以参考Linux源码中oom.h(从-15到-17)
    

当`oom_adj`设置为最小值时，该进程将不会被OOM killer杀掉，设置方法如下:

```
# 命令
echo {value} > /proc/${process_id}/oom_adj

# 脚本
for redis_pid in $(pgrep -f "redis-server")
do
  echo -17 > /proc/${redis_pid}/oom_adj
done
```

-   **Step 6.设置其打开文件数句柄数以及单个用户最大进程数**  
    描述: 下面得参数主要设置是单个进程能够使用得Linux最大文件句柄数, 解决在高并发的情况下不会异常报错。在Redis官方提到的建议
    

```
# You requested maxclients of 10000 requiring at least 10032 max file descriptors.
第一行：Redis建议把open files至少设置成10032，那么这个10032是如何来的呢？因为maxclients的默认是10000，这些是用来处理客户端连接的，除此之外，Redis内部会使用最多32个文件描述符，所以这里的10032 = 10000 + 32。

# Redis can’t set maximum open files to 10032 because of OS error: Operation not permitted.
第二行：Redis不能将open files设置成10032，因为它没有权限设置。

# Current maximum open files is 4096. Maxclients has been reduced to 4064 to compensate for low ulimit. If you need higher maxclients increase ‘ulimit –n’.
第三行：当前系统的open files是4096，所以maxclients被设置成4096-32=4064个，如果你想设置更高的maxclients，请使用ulimit -n来设置。从上面的三行日志分析可以看出open files的限制优先级比maxclients大。
```

    解决办法:

```
# 临时
ulimit –Sn 10032
# 永久
tee etc/security/limits.conf <<'EOF'
*  soft    nofile          10032
*  hard    nofile          10032
*  soft    nproc           65535
*  hard    nproc           65535
EOF
```

-   **Step 7.TCP backlog 日志队列优化**  
    描述: Redis 默认的 tcp-backlog 为511 我们可以通过修改配置 tcp-backlog 进行调整，如果Linux的tcp-backlog 小于Redis设置的 tcp-backlog，那么在Redis启动时会看到如下日志：
    

```
# WARNING: The TCP backlog setting of 511 cannot be enforced because /proc/sys/net/core/somaxconn is set to the lower value of 128.
```

    解决方法:

```
# 查看
cat /proc/sys/net/core/somaxconn
128

# 修改
echo 511 > /proc/sys/net/core/somaxconn
```

-   **Step 8.保证Redis服务器时钟的一致性**  
    描述: 我们知道像Redis Sentinel和Redis Cluster这两种需要多个Redis实例的类型，可能会涉及多台服务器。虽然Redis并没有对多个服务器的时钟有严格的要求，但是假如多个Redis实例所在的服务器时钟不一致，对于一些异常情况的日志排查是非常困难的，例如Redis Cluster的故障转移，如果日志时间不一致，对于我们排查问题带来很大的困扰(注：但不会影响集群功能，集群节点依赖各自时钟)。一般公司里都会有NTP服务用来提供标准时间服务，从而达到纠正时钟的效果
    

    例如：每小时的同步1次NTP服务

    0 \* \* \* \* /usr/sbin/ntpdate ntp.xx.com \> /dev/null 2\>&1

**系统优化配置**

```
# - 设置内存分配策略
sudo sysctl -w vm.overcommit_memory=1

# - 尽量使用物理内存(速度快)针对内核版本大于>=3.x （宁愿swap也不要OOM killer）
sudo sysctl -w vm.swapniess=1

# - 禁用 THP 特性减少内存消耗
echo never > /sys/kernel/mm/transparent_hugepage/enabled

# - OOM killer 特性优化
for redis_pid in $(pgrep -f "redis-server")
do
  echo -17 > /proc/${redis_pid}/oom_adj
done

# - 设置其打开文件数句柄数以及单个用户最大进程数
tee etc/security/limits.conf <<'EOF'
*  soft    nofile          10032
*  hard    nofile          10032
*  soft    nproc           65535
*  hard    nproc           65535
EOF

# - SYN队列长度设置此参数可以容纳更多等待连接的网络。
echo 511 > /proc/sys/net/core/somaxconn
sudo sysctl -w net.ipv4.tcp_max_syn_backlog=2048

# - 每个小时同步一次时间
0 * * * * /usr/sbin/ntpdate ntp.xx.com > /dev/null 2>&1
```

**应用配置优化**

```
# 最大客户端上限连接数（需根据实际情况调整与系统的open files有关，其数量值为open files(10032) - 32）
maxclients 10000

# 集群配置优化关键项
# 集群超时时间，如果此时间设置太小时由于网络波动可能会导致进行重新选Master的操作
cluster-node-timeout 5000
# 主节点写入后必须同步到一台从上，防止数据丢失的有效方法(要求是其从节点必须>=1)
min‐replicas‐to‐write 1
```

**应用使用中优化**

```
# (1) 查询执行时间指的是不包括像客户端响应(talking)、发送回复等 IO 操作，而单单是执行一个查询命令所耗费的时间
redis> SLOWLOG LEN   # 管理 redis 的慢日志查看当前日志的数量
redis> SLOWLOG RESET # 清空 slowlog 此时上面 LEN 变成 0

# (2) 断开耗时连接
# 列出所有已连接客户端
redis 127.0.0.1:6379> CLIENT LIST
addr=127.0.0.1:43501 fd=5 age=10 idle=0 flags=N db=0 sub=0 psub=0 multi=-1 qbuf=0 qbuf-free=32768 obl=0 oll=0 omem=0 events=r cmd=client
# 杀死当前客户端的连接
redis 127.0.0.1:6379> CLIENT KILL 127.0.0.1:43501
OK
```

![](https://i0.hdslb.com/bfs/article/db75225feabec8d8b64ee7d3c7165cd639554cbc.png@progressive.webp)

>             如果你觉得这个专栏还不错的，请给这篇专栏点个赞、投个币、收个藏，这将对我有很大帮助！          
> 
> 欢迎各位志同道合的朋友一起学习交流，如文章有误请在下方留下您宝贵的经验知识，个人邮箱地址【master#weiyigeek.top】

![](https://i0.hdslb.com/bfs/article/02db465212d3c374a43c60fa2625cc1caeaab796.png@progressive.webp)

文章来源: https://weiyigeek.top 【WeiyiGeek Blog - 为了能到远方，脚下的每一步都不能少】