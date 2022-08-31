xargs 是一个非常有用的命令，奇怪的是很多人并不知道这个命令。它最大的功能是并行执行批量命令。通过利用 cpu 的多核计算能力，你可以将自己的工作效率提高 10 倍！时间是世界上最珍贵的东西，xargs 极大地节约了我们的时间，这就是我喜欢 xargs 的地方！

### 批量删除文件

删除文件最简单的方法是 `rm ./*`，当要删除的文件过多时会报 `Argument list too long` 这个错误。原因是 bash 会将 `rm ./*` 匹配到的文件都展开，生成一个非常长的 command，linux 内核对 command 长度有限制。这时我们就可以使用 xargs 来删除文件

```
ls ./* | xargs rm -rf
```

xargs 会将匹配到的文件以合理的长度传给 `rm -rf`，相当于是将文件分组删除。

一个类似的场景是批量 kill 进程。比如你的 chrome 浏览器不响应了，你想重启 chrome，可以用 xargs 将 chrome 进程批量杀掉。

```
ps aux | grep '/Applications/Google Chrome.app' | awk '{print $2}' | xargs kill -9
```

用 ps 和 grep 找到所有的 chrome 进程，再用 awk 将进程 ip 通过管道传给 xargs，最后 xargs 将 进程 ip 传给 kill。boom，所有的 chrome 进程都消失了。

### 服务器批量管理

经过刚才的热身，你已经对 xargs 有了大概的了解。现在我们看一个稍微复杂点的场景。我们要在多个服务器上执行相同的命令。做的方法比较多，比如你可以用 shell 脚本，也可以用专业的服务器集群维护工具。如果命令比较简单，用 xargs 会非常方便。

假如我们有三个服务器

-   192.168.0.1
-   192.168.0.2
-   192.168.0.3

我们想看看这三个服务器上 nginx 进程的情况，我们可以用这个命令来实现

```
echo "192.168.0.1 192.168.0.2 192.168.0.3" \
| xargs -n 1 -I __ip__ ssh root@__ip__ 'ps aux | grep nginx'
```

这个命令通过管道将 ip 列表传给 xargs，`-n 1` 表示 xargs 每次消费一个 ip。`-I __ip__` 指定每次消费的 ip 使用的替换字符串，每消费一个 ip，xargs 就会将后面的 `__ip__` 替换成对应的 ip。

实际效果相当于

```
ssh root@192.168.0.1 'ps aux | grep nginx'
ssh root@192.168.0.2 'ps aux | grep nginx'
ssh root@192.168.0.3 'ps aux | grep nginx'
```

我们现在要从这三台服务器上下载`/var/log/cron`文件到本地，可以这么做

```
echo "192.168.0.1 192.168.0.2 192.168.0.3" \
| xargs -n 1 -P 4 -I __ip__ scp root@__ip__:/var/log/cron ./__ip__.cron
```

这里我们用到了一个新的参数 `-P 4`，它的作用是使用最多 4 个进程并行执行后面的命令。假如服务器有非常多，要下载的文件也比较大，我们就可以用这个参数来并行从服务器上下载文件，同时又能控制进程数量。

### 一个更复杂的例子

我们现在想分析一下所有服务器上 nginx 日志，找到所有访问 nginx 服务的 ip，并对 ip 做计数统计。

服务器上有切割并压缩好的 nginx 日志文件，这些日志文件在 `/var/log/nginx` 下面，文件名是 `access.log.[0-9]+.gz`。一种方法是我们在每个服务器上单独统计 ip 的计数，然后再将每个服务器统计的结果汇总到一起。另一种方法是我们将 nginx 日志拉到本地，在本地做分析。在 nginx 服务器负载不高的情况下，我们可以使用第一种方法。

首先，我们在每个服务器上找到 nginx 日志文件，解压并找到所有 ip，然后做 ip 计数。我们将这个功能写在 `run_in_server.sh` 脚本里

```
#!/bin/bash

process() {
    zcat $1 | awk '{print $1}' | sort | uniq -c > /tmp/$1.ip_count
}
export -f process

cd /var/log/nginx && find ./ -type f -name "access.log.*.gz" | xargs -n 1 -P 2 bash -c 'process "$@"' _
cat /tmp/*.ip_count | awk '{ip_counts[$2] += $1} END {for (ip in ip_counts) print ip, ip_counts[ip]}'
rm -rf /tmp/*.ip_count
```

这个脚本在每个服务器上执行。

首先定义了一个函数 `process`。这个函数将传进来的文件解压，并统计 ip 次数，最后将结果写到 `/tmp/` 下以 `.ip_count` 结尾的临时文件中。

接着脚本里先找到所有的 nginx 日志文件，将日志文件通过 xargs 依次传给 `process` 函数。这里的最大并行数是 2，可以根据 nginx 服务器的负载情况进行调整。

最后将临时结果通过 awk 做汇总。

我们再写另外一个脚本 `run_in_local.sh`

```
#!/bin/bash

process() {
    scp ./run_in_server.sh root@$1:/tmp/run_in_server.sh
    ssh root@$1 bash /tmp/run_in_server.sh > /tmp/$1.ip_count
}
export -f process

cat ip_list | xargs -n 1 -P 8 -I __ip__ scp ./run_in_server.sh root@__ip__:/tmp/run_in_server.sh
cat /tmp/*.ip_count | awk '{ip_counts[$2] += $1} END {for (ip in ip_counts) print ip, ip_counts[ip]}' > ./ip_count_result
rm -rf /tmp/*.ip_count
```

这个脚本是在本地执行。

首先将 run\_in\_server.sh 下发到每个服务器上并运行，接着将每个服务器的结果写到本地的 /tmp/ 下面，然后用 awk 将结果汇总，最后清理临时文件。

不管你的服务器有多少台，我们通过两个简单的脚本，就高效的完成了一个非常复杂的工作，而 xargs 在这里面扮演了非常重要的角色。

### One more thing

最后一个例子有没有觉得很眼熟？是不是和 map/reduce 的 word count 例子很像？没错，我们用了两个简单的 shell 脚本就实现了一个非常高效的 map/reduce 系统，这个系统充分利用了所有计算和存储资源，可很短的时间内完成了一个复杂的分布式计算。

我想，看完这篇文章，聪明的你会很快把 xargs 用起来，也会把 xargs 介绍给更多的人！

## 文章导航