**本章目录:**

0x00 Redis 性能指标监控

-   (1) 性能指标
    

-   1.基本活动指标：Basic activity
    
-   2.性能指标：Performance
    
-   3.内存指标: Memory
    
-   4.持久性指标: Persistence
    
-   5.错误指标：Error
    
-   6.其他指标说明
    

-   (2) 性能测试工具
    

-   1.redis-benchmark 命令
    
-   2.redisbench 工具
    
-   3.rdb 内存分析工具
    

-   (3) 基准测试实践
    

-   3.1 K8s中单实例redis测试
    

![](https://i0.hdslb.com/bfs/article/4aa545dccf7de8d4a93c2b2b8e3265ac0a26d216.png@progressive.webp)

Redis 服务端常见指标参数:

```
redis-cli -a password info > redis-Performance.txt # 我们可以将redis服务端info相关信息导出到文件之中
  # 2.clients:
  # 3.memory：
  # 4.persistence：
  # 5.stats：通用统计数据
  # 6.Replication：
  # 7.CPU：CPU使用情况
  # 8.cluster：
  # 9.Keypass：键值对统计数量信息
  
10.20.172.108:6379> info  # (1) Redis 服务端信息交互式查看
# Server 服务器运行的环境参数
redis_version:6.2.5
redis_git_sha1:00000000
redis_git_dirty:0
redis_build_id:cb556a016f8668d
redis_mode:standalone
os:Linux 5.11.0-25-generic x86_64
arch_bits:64
multiplexing_api:epoll
atomicvar_api:c11-builtin
gcc_version:9.3.0
process_id:187640
process_supervised:no
run_id:97838216d4fe0de4739e7814b5a2e1d0d32d0982
tcp_port:6379
server_time_usec:1630241617439942
uptime_in_seconds:10930
uptime_in_days:0
hz:10
configured_hz:10
lru_clock:2851665
executable:/opt/databases/redis-6.2.5/src/./redis-server
config_file:/home/weiyigeek/redis/6379/redis-6379.conf
io_threads_active:0

# Clients 客户端相关信息
connected_clients:7
cluster_connections:0
maxclients:10000
client_recent_max_input_buffer:32
client_recent_max_output_buffer:0
blocked_clients:0
tracking_clients:0
clients_in_timeout_table:0

# Memory 服务器运行内存统计数据
used_memory:2050432
used_memory_human:1.96M
used_memory_rss:5140480
used_memory_rss_human:4.90M
used_memory_peak:2253512
used_memory_peak_human:2.15M
used_memory_peak_perc:90.99%
used_memory_overhead:1982152
used_memory_startup:810376
used_memory_dataset:68280
used_memory_dataset_perc:5.51%
allocator_allocated:2204376
allocator_active:2555904
allocator_resident:5230592
total_system_memory:12442619904
total_system_memory_human:11.59G
used_memory_lua:37888
used_memory_lua_human:37.00K
used_memory_scripts:0
used_memory_scripts_human:0B
number_of_cached_scripts:0
maxmemory:0
maxmemory_human:0B
maxmemory_policy:noeviction
allocator_frag_ratio:1.16
allocator_frag_bytes:351528
allocator_rss_ratio:2.05
allocator_rss_bytes:2674688
rss_overhead_ratio:0.98
rss_overhead_bytes:-90112
mem_fragmentation_ratio:2.59
mem_fragmentation_bytes:3153776
mem_not_counted_for_evict:124
mem_replication_backlog:1048576
mem_clients_slaves:0
mem_clients_normal:123000
mem_aof_buffer:128
mem_allocator:jemalloc-5.1.0
active_defrag_running:0
lazyfree_pending_objects:0
lazyfreed_objects:0

# Persistence 持久化数据相关信息
loading:0
current_cow_size:0
current_cow_size_age:0
current_fork_perc:0.00
current_save_keys_processed:0
current_save_keys_total:0
rdb_changes_since_last_save:3
rdb_bgsave_in_progress:0
rdb_last_save_time:1630230687
rdb_last_bgsave_status:ok
rdb_last_bgsave_time_sec:-1
rdb_current_bgsave_time_sec:-1
rdb_last_cow_size:0
aof_enabled:1
aof_rewrite_in_progress:0
aof_rewrite_scheduled:0
aof_last_rewrite_time_sec:0
aof_current_rewrite_time_sec:-1
aof_last_bgrewrite_status:ok
aof_last_write_status:ok
aof_last_cow_size:462848
module_fork_in_progress:0
module_fork_last_cow_size:0
aof_current_size:150
aof_base_size:92
aof_pending_rewrite:0
aof_buffer_length:0
aof_rewrite_buffer_length:0
aof_pending_bio_fsync:0
aof_delayed_fsync:0

# Stats 通用统计数据信息
total_connections_received:25
total_commands_processed:50482
instantaneous_ops_per_sec:4
total_net_input_bytes:2758703
total_net_output_bytes:22330756
instantaneous_input_kbps:0.23
instantaneous_output_kbps:0.55
rejected_connections:0
sync_full:0
sync_partial_ok:0
sync_partial_err:0
expired_keys:0
expired_stale_perc:0.00
expired_time_cap_reached_count:0
expire_cycle_cpu_milliseconds:0
evicted_keys:0
keyspace_hits:1
keyspace_misses:0
pubsub_channels:1
pubsub_patterns:0
latest_fork_usec:310
total_forks:1
migrate_cached_sockets:0
slave_expires_tracked_keys:0
active_defrag_hits:0
active_defrag_misses:0
active_defrag_key_hits:0
active_defrag_key_misses:0
tracking_total_keys:0
tracking_total_items:0
tracking_total_prefixes:0
unexpected_error_replies:0
total_error_replies:8
dump_payload_sanitizations:0
total_reads_processed:48899
total_writes_processed:97139
io_threaded_reads_processed:0
io_threaded_writes_processed:0

# Replication 主从相关指标信息
role:master
connected_slaves:0
master_failover_state:no-failover
master_replid:32da05299a5a36de431b4c05122f7d2b93eca169
master_replid2:3e15749ad586d60bd0d1c93854f6f719a22316ce
master_repl_offset:8915
second_repl_offset:829
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:1
repl_backlog_histlen:8915

# CPU 处理器模式占用信息
used_cpu_sys:12.012184
used_cpu_user:9.453505
used_cpu_sys_children:0.002158
used_cpu_user_children:0.000000
used_cpu_sys_main_thread:11.802969
used_cpu_user_main_thread:9.286577

# Modules 模块加载情况

# Errorstats 错误状态信息
errorstat_NOAUTH:count=6
errorstat_READONLY:count=1
errorstat_WRONGPASS:count=1

# Cluster 集群信息
cluster_enabled:0

# Keyspace 库id与键数量相关信息
db0:keys=1,expires=0,avg_ttl=0
```

![](https://i0.hdslb.com/bfs/article/3b6dff503417ef7db87a840e9bd973c67a1ea1d3.png@900w_381h_progressive.webp)

```
grep -En "connected_clients|conected_laves|master_last_io_seconds_ago|keyspace" redis-Performance.txt
27:connected_clients:7 # # 客户端连接数量
28:connected_slaves:1  # # slave连接数量
128:keyspace_hits:1
129:keyspace_misses:0
```

![](https://i0.hdslb.com/bfs/article/d4707d52ee032388e1c46d2342bed5442e064294.png@842w_314h_progressive.webp)

```
grep -En "latency|instantaneous_ops_per_sec|hi rate" redis-Performance.txt
114:instantaneous_ops_per_sec:3
```

![](https://i0.hdslb.com/bfs/article/b96f2707fd53b6c5fc85cfb6880d0aa028588088.png@942w_297h_progressive.webp)

```
grep -En "used_memory|mem_fragmentation_ratio|evicted_keys|blocked_clients" redis-Performance.txt
32:blocked_clients:0
37:used_memory:2050432
38:used_memory_human:1.96M      # # 内存分配器从操作系统分配的内存总量
39:used_memory_rss:5234688     
40:used_memory_rss_human:4.99M  # # 操作系统看到的内存占用，top命令看到的内存
41:used_memory_peak:2253512
42:used_memory_peak_human:2.15M # # redis内存消耗的峰值
43:used_memory_peak_perc:90.99%
44:used_memory_overhead:1982152
45:used_memory_startup:810376
46:used_memory_dataset:68280
47:used_memory_dataset_perc:5.51%
53:used_memory_lua:37888
54:used_memory_lua_human:37.00K #  # lua脚本引擎占用的内存大小
55:used_memory_scripts:0
56:used_memory_scripts_human:0B
67:mem_fragmentation_ratio:2.63
127:evicted_keys:0
```

## 4.持久性指标: Persistence

![](https://i0.hdslb.com/bfs/article/18617865e23f2af5328ee7e020971de8fc598708.png@942w_221h_progressive.webp)

```
grep -En "rdb_last_save_time|rdb_changes_sice_last_save" redis-Performance.txt
88:rdb_last_save_time:1630230687
89:rdb_changes_since_last_save:0   # 自最后一次持久化以来数据库的更改数
```

![](https://i0.hdslb.com/bfs/article/a3dcd9d905cf34eb295a8975da2a033b46c09e2c.png@942w_264h_progressive.webp)

```
grep -En "rejected_connections|master_link_down_since_seconds" redis-Performance.txt
9:master_link_down_since_seconds:10937
119:rejected_connections:0
keyspace_misses:0    # key值查找失败(没有命中)次数，出现多次可能是被Hacker Attack
```

```
# 1.复制积压缓冲区如果设置得太小，会导致里面的指令被覆盖掉找不到偏移量，从而触发全量同步
repl_backlog_size: 1048576

# 2.通过查看sync_partial_err变量的次数来决定是否需要扩大积压缓冲区，它表示主从半同步复制失败的次数
sync_partial_err:1
```

描述: Redis 性能测试是通过同时执行多个命令实现的,该命令是在 redis 的目录下执行的;

官网参考: https://redis.io/topics/benchmarks

**语法参数:**

```
Usage: redis-benchmark [-h <host>] [-p <port>] [-c <clients>] [-n <requests]> [-k <boolean>]

# 参数说明:
 -h <hostname>      Server hostname (default 127.0.0.1)
 -p <port>          Server port (default 6379)
 -s <socket>        Server socket (overrides host and port)
 -a <password>      Password for Redis Auth
 -c <clients>       Number of parallel connections (default 50)
 -n <requests>      Total number of requests (default 100000)
 -d <size>          Data size of SET/GET value in bytes (default 2)
 --dbnum <db>       SELECT the specified db number (default 0)
 -k <boolean>       1=keep alive 0=reconnect (default 1)
 -r <keyspacelen>   对SET/GET/INCR使用随机键，对SADD使用随机值使用此选项基准将扩展参数内的字符串_rand_int _uu），该参数的指定范围为0到keyspacelen-1之间的12位数字。
 -P <numreq>        Pipeline <numreq> requests. Default 1 (no pipeline).
 -q                 Quiet. Just show query/sec values
 --csv              Output in CSV format
 -l                 Loop. Run the tests forever
 -t <tests>         Only run the comma separated list of tests. The test names are the same as the ones produced as output.
 -I                 Idle mode. Just open N idle connections and wait.
```

**基础实例:**

```
# (1) 同时执行 10000 个请求来检测性能(所有默认测试),通过 -q 参数让结果只显示每秒执行的请求数
$ ./redis-benchmark -h 127.0.0.1 -p 6379 -t set,lpush -n 10000 -q
$ ./redis-benchmark -n 10000  -q 
   # PING_INLINE: 41493.78 requests per second
   # PING_BULK: 44843.05 requests per second
   # SET: 42194.09 requests per second
   # GET: 44052.86 requests per second
   # INCR: 43290.04 requests per second
   # LPUSH: 42194.09 requests per second
   # RPUSH: 42372.88 requests per second
   # LPOP: 42194.09 requests per second
   # RPOP: 42194.09 requests per second
   # SADD: 43668.12 requests per second
   # HSET: 42372.88 requests per second
   # SPOP: 44843.05 requests per second
   # LPUSH (needed to benchmark LRANGE): 42553.19 requests per second
   # LRANGE_100 (first 100 elements): 21367.52 requests per second
   # LRANGE_300 (first 300 elements): 9451.80 requests per second
   # LRANGE_500 (first 450 elements): 6807.35 requests per second
   # LRANGE_600 (first 600 elements): 5350.46 requests per second
   # MSET (10 keys): 36363.64 requests per second

# (2) 运行指定项目的测试，例如我们要求在安静模式下仅运行测试 SET 和 LPUSH 命令
$ redis-benchmark -t set,lpush -n 100000 -q
  # SET: 74239.05 requests per second
  # LPUSH: 79239.30 requests per second


# (3) 指定eval脚本命令进行基准测试
$ redis-benchmark -n 100000 -q script load "redis.call('set','foo','bar')"
  # script load redis.call('set','foo','bar'): 69881.20 requests per second

# (4) 选择密钥空间的大小,默认情况下基准测试针对单个密钥运行,而我们通常可以通过使用大键来模拟更真实的工作负载空间。
# 例如，如果我想运行 100 万次 SET 操作，在 10 万个可能的密钥中为每个操作使用一个随机密钥，
$ redis-cli flushall
$ redis-benchmark -t set -r 100000 -n 1000000

# (5) 默认情况下，每个客户端仅在收到上一个命令的回复时发送下一个命令, Redis 支持流水线，因此可以一次发送多个命令可以想象为并行。
# 例如: 使用 16 个命令的流水线在 MacBook Air 11" 中运行基准测试
redis-benchmark -n 1000000 -t set,get -P 16 -q
  # SET: 403063.28 requests per second
  # GET: 508388.41 requests per second

# (6) 使用 Unix 域套接字形式进行基准测试
$ numactl -C 6 ./redis-benchmark -q -n 100000 -s /tmp/redis.sock -d 256

# (7) 使用 使用 TCP loopback
$ numactl -C 6 ./redis-benchmark -q -n 100000 -d 256
```

![WeiyiGeek.redis-benchmark](https://i0.hdslb.com/bfs/article/7e28248c951fb78773aea7116585c3e6f4797e8c.png@942w_755h_progressive.webp)

**在Redis、Memcached内存数据库基准测试对比:**

```
#!/bin/bash

# BIN=./redis-benchmark
BIN=./mc-benchmark
payload=32
iterations=100000
keyspace=100000

for clients in 1 5 10 20 30 40 50 60 70 80 90 100 200 300
do
    SPEED=0
    for dummy in 0 1 2
    do
        S=$($BIN -n $iterations -r $keyspace -d $payload -c $clients | grep 'per second' | tail -1 | cut -f 1 -d'.')
        if [ $(($S > $SPEED)) != "0" ]
        then
            SPEED=$S
        fi
    done
    echo "$clients $SPEED"
done
```

最后以下是使用`gnuplot`生成的图形形式的结果：

![](https://i0.hdslb.com/bfs/article/901c33916eacfbc1dcd6366ab77caedf518b9217.png@942w_642h_progressive.webp)

**影响基准测试要素**

-   1.工作负载(连接的客户端的数量)
    
-   2.不同版本的Redis
    
-   3.提供服务的服务器物理配置(磁盘、网络、CPU、内存)，在多 CPU 插槽服务器上，Redis 性能取决于 NUMA 配置和进程位置。
    

```
# 不同虚拟化和裸机服务器上的基准测试结果。
* 该测试由 50 个同时执行 200 万个请求的客户端完成。
* 使用环回接口执行测试。
* 使用 100 万个密钥的密钥空间执行测试。
* 测试在使用和不使用流水线（16 个命令流水线）的情况下执行。

# Intel(R) Xeon(R) CPU E5520 @ 2.27GHz（带流水线）/ （无流水线）
$ ./redis-benchmark -r 1000000 -n 2000000 -t get,set,lpush,lpop -P 16 -q # 优于无流水线。
  SET: 552028.75 requests per second
  GET: 707463.75 requests per second
  LPUSH: 767459.75 requests per second
  LPOP: 770119.38 requests per second

$ ./redis-benchmark -r 1000000 -n 2000000 -t get,set,lpush,lpop -q
  SET: 122556.53 requests per second
  GET: 123601.76 requests per second
  LPUSH: 136752.14 requests per second
  LPOP: 132424.03 requests per second
```

-   4.与使用相同硬件不使用虚拟化的情况相比，Redis 在 VM 上运行速度较慢(`推荐物理机按照Redis为首选`)
    
-   5.根据平台的不同，unix 域套接字可以实现比 TCP/IP 环回（例如在 Linux 上）多约 50% 的吞吐量。
    
-   6.与 TCP/IP 环回相比，Unix 域套接字的性能优势在大量使用流水线（即长流水线）时趋于降低。
    
-   7.当使用以太网网络访问 Redis 时，当数据大小保持在以太网数据包大小（约 1500 字节）以下时，使用流水线聚合命令特别有效, 在处理 10 字节、100 字节或 1000 字节的查询几乎会产生相同的吞吐量。
    

描述: 官方推荐的`redis-benchmark`在进行集群的基准测试时，没有办法指定集群模式，此处引入 Redis & Redis Cluster benchmark Tool 更方便对集群基准测试的处理。下载地址: https://github.com/panjiang/redisbench

**redisbench 特点**

-   以 Golang 开发构建
    
-   可以测试redis单实例
    
-   可以测试redis集群
    
-   可以利用多核
    
-   支持同时在多台机器上运行，用于测试大型redis集群（需要相同的机器硬件）
    

**格式语法:**

```
./redisbench -h
-a string   #Redis instance address or Cluster addresses. IP:PORT[,IP:PORT]
-c int      #Clients number for concurrence (default 1)
-cluster    #true: cluster mode, false: instance mode
-d int      #Data size in bytes (default 1000)
-ma string  #addresses for run multiple testers at the same time
-mo int     #the order current tester is in multiple testers
-n int      #Testing times at every client (default 1)
```

**基础示例:**

```
# 测试单实例模式
./redisbench -a 127.0.0.1:6379 -c 200 -n 20000 -d 3

# 测试集群
./redisbench -cluster=true -a 192.168.1.11:6379,192.168.1.11:6380 -c 500 -n 2000 -d 3

# 使用多个测试节点
./redisbench -cluster=true -a 192.168.1.11:6379,192.168.1.11:6380 -c 500 -n 2000 -d 3 -ma 192.168.1.11:9001,192.168.1.11:9002,192.168.1.11:9003 -mo 1 &
./redisbench -cluster=true -a 192.168.1.11:6379,192.168.1.11:6380 -c 500 -n 2000 -d 3 -ma 192.168.1.11:9001,192.168.1.11:9002,192.168.1.11:9003 -mo 2 &
./redisbench -cluster=true -a 192.168.1.11:6379,192.168.1.11:6380 -c 500 -n 2000 -d 3 -ma 192.168.1.11:9001,192.168.1.11:9002,192.168.1.11:9003 -mo 3
```

Tips: 测试结果会自动打印出：请求值，请求时间，TPS 此处不实际演示使用了，感兴趣的朋友可以自行下载测试。

描述: RDR 是解析 redis rdbfile 工具。与redis-rdb-tools相比，RDR 是由golang 实现的，速度更快。

-   分析 Redis 内存中那个 Key 值占用的内存最多
    
-   分析出 Redis 内存中那一类开头的 Key 占用最多，有利于内存优化
    
-   Redis Key 值以 Dashboard 展示，这样更直观
    

安装下载地址: https://github.com/xueqiu/rdr/releases

注意事项:

-   1.linux和windows使用前先添加可执行权限 `chmod +x rdr_linux`
    

基础语法:

```
# RDR 参数解释
show 网页显示 rdbfile 的统计信息
keys 从 rdbfile 获取所有 key
help 帮助
```

基础实例(以Linux为例):

```
#1.分析统计多个 Redis rdb中各种类型的使用占比 
./rdr-linux keys *.rdb
SEARCH:PROJECTS_BY_ID:68
SEARCH:PROJECTS_BY_ID:64
SEARCH:PROJECTS_BY_PARENTID_LIST:0
SEARCH:PROJECTS_BY_PARENTID_LIST:64
SEARCH:PROJECTS_BY_PARENTID_LIST:66
SEARCH:PROJECTS_BY_PARENTID_LIST:68
SEARCH:PROJECTS_BY_ID:66
SEARCH:PROJECTSINFO_BY_PARENTID_LIST:64
ALLOW:SEARCH_BY_SFZH:230103197805153637

./rdr-linux dump dump.rdb
{"CurrentInstance": "dump.rdb",
"LargestKeyPrefixes": {
   "list": [
   {
      "Type": "list",
      "Key": "site",
      "Bytes": 144,
      "Num": 1
   }
],


#2.网页显示分析结果
./rdr-linux show -p 8080 *.rdb
start parsing...
parse dump.rdb  done
parsing finished, please access http://{$IP}:8080
```

## (3) 基准测试实践

**环境说明:**

```
# 运行该Pod的主机节点
osImage: Ubuntu 20.04.1 LTS
kernelVersion: 5.4.0-42-generic
kubeProxyVersion: v1.19.6
kubeletVersion: v1.19.6
containerRuntimeVersion: docker://19.3.14

$ Server 相关信息
redis_version:6.2.5
os:Linux 5.4.0-42-generic x86_64
arch_bits:64
gcc_version:10.3.1
process_id:1
process_supervised:no
tcp_port:6379
hz:10
configured_hz:10
lru_clock:3627023
io_threads_active:0

$ redis 配置的内存限额
maxmemory:1073741824
maxmemory_human:1.00G
maxmemory_policy:volatile-lru

$ cpu 相关信息
Model name: Intel(R) Xeon(R) CPU E3-1220 V2 @ 3.10GHz
物理CPU数: 1
逻辑CPU数: 4
CPU核心数: 4
```

**基准测试:**

```
# 测试1.执行1千万次set命令与get命令，其中每次读取得大小为256字节, 请求客户端数默认50。
~$ redis-benchmark -h 10.102.39.181 -a 123456 -d 256 -t set,get -n 10000000 -q                            
SET: 42445.36 requests per second
GET: 49504.70 requests per second
# - 测试时 Pod 资源峰值
very 1.0s: kubectl top pod -n database redis-cm-0  Tue Sep  7 20:36:50 2021
NAME         CPU(cores)   MEMORY(bytes)
redis-cm-0   848          18Mi

# 测试2.同样执行1千万次set命令与get命令，其中每次读取得大小为256字节，唯一不同的是采用 流水线 -P 16 进行测试（可以看出每秒set、get请求数显著提升）。
~$ redis-benchmark -h 10.102.39.181 -a 123456 -d 256 -t set,get -n 10000000 -P 16 -q                            
SET: 96019.98 requests per second
GET: 316575.91 requests per second

# - 测试时 Pod 资源峰值
very 1.0s: kubectl top pod -n database redis-cm-0  Tue Sep  7 20:46:50 2021
NAME         CPU(cores)   MEMORY(bytes)
redis-cm-0   457m         337Mi
```

**实践测试**

-   1.通过shell pipe 与 redis pipe插入10万数据进行对比
    

```
## Shell-pipe.sh
#!/bin/bash
echo "开始时间: $(date +%s)"
for ((i=0;i<100000;i++));do
  echo -en "helloworld-redis-${i}" | redis-cli -h 10.102.39.181 --no-auth-warning -a 123456 -x set username${i} >> ok.txt
done
echo "完成时间: $(date +%s)"

## redis-pipe.sh
#!/bin/bash
echo "开始时间: $(date +%s)"
python3 redis-set.py >> set-command.txt
cat set-command.txt | redis-cli -h 10.102.39.181 -a 123456 --no-auth-warning --pipe
echo "完成时间: $(date +%s)"

## redis-set.py
tee redis-set.py <<'EOF'
#!/usr/bin/python
for i in range(100000):
  print('set name'+str(i)+' helloworld-redis-'+str(i))
EOF
```

-   2.利用time命令记录了脚本插入的执行效率
    

```
## redis pipe 方式插入数据
~/k8s/benchmark$ time ./redis-pipe.sh
  # 开始时间:1631020862
  # All data transferred. Waiting for the last reply...
  # Last reply received from server.
  # errors: 0, replies: 100000
  # 完成时间: 1631020862

  # real    0m0.466s
  # user    0m0.126s
  # sys     0m0.035s

# Keyspace
db0:keys=100000,expires=0,avg_ttl=0

## shell pipe 方式插入数据
~/k8s/benchmark$ time ./shell-pip.sh
  # 开始时间: 1631021312
  # 完成时间: 1631021921
  # real    10m9.265s # 程序开始至结束总用时(包括CPU)。
  # user    3m44.411s # 程序本身以及调用子进程的时间。
  # sys     1m55.435s # 由程序本身或者间接调用的系统调用执行时间。
# Keyspace
db0:keys=200000,expires=0,avg_ttl=0
```

Tips : 可以从上面的结果看出两种方式real总耗时量相差之巨大，redis pipe方式效率相比较普通shell pipe方式不是一个量级，所以在开发程序中尽量使用`redis pipe`管道方式进行提交数据。

```
# 为了方便后续演示，握又向数据库中插入了80W条数据，只用了大约4s。
开始时间: 1631022423
All data transferred. Waiting for the last reply...
Last reply received from server.
errors: 0, replies: 800000
完成时间: 1631022427
real    0m4.023s
user    0m0.885s
sys     0m0.187s
```

![](https://i0.hdslb.com/bfs/article/db75225feabec8d8b64ee7d3c7165cd639554cbc.png@progressive.webp)

>         如果你觉得这个专栏还不错的，请给这篇专栏点个赞、投个币、收个藏，这将对我有很大帮助！            

欢迎各位志同道合的朋友一起学习交流，如文章有误请在下方留下您宝贵的经验知识，个人邮箱地址【master#weiyigeek.top】

![](https://i0.hdslb.com/bfs/article/02db465212d3c374a43c60fa2625cc1caeaab796.png@942w_progressive.webp)

文章来源: https://weiyigeek.top 【WeiyiGeek Blog - 为了能到远方，脚下的每一步都不能少。】