此文是前段时间笔记的整理，之前自己对这方面的关注不够，因此做下记录。

___

有太多的文章介绍如何**运行**容器，然而如何**停止**容器的文章相对少很多。

根据运行的应用类型，应用的停止过程非常重要。如果应用要写文件，停止前要保证正确刷新数据并关闭文件；如果是 HTTP 服务，要确保停止前处理所有未完成的请求。

## 信号

信号是 Linux 内核与进程以及进程间通信的一种方式。针对每个信号进程都有个默认的动作，不过进程可以通过定义信号处理程序来覆盖默认的动作，除了 `SIGSTOP` 和 `SIGKILL`。二者都不能被捕获或重写，前者用来将进程暂停在当前状态，而后者则是从内核层面立即杀掉进程。

有两个比较重要的进程 `SIGTERM` 和 `SIGKILL`。`SIGTERM` 是优雅地关闭命令，`SIGKILL` 则是暴力的关闭命令。比如 Docker，容器会先收到 `SIGTERM` 信号，10s 后会收到 `SIGKILL` 信号。

还有很多其他的信号，只是限定于特定的上下文。

## 中断

硬件的中断就像操作系统的信号。通常发生在硬件想要向操作系统注册事件时。操作系统必须立即停止运行，并处理中断。

比较常见的中断例子就是键盘中断，比如按下 `ctrl+z` 或者 `ctrl+c`。Linux 将其分别转换成 `SIGTSTP` 和 `SIGINT`。硬件中断过去通常用来处理键盘和鼠标输入，但如今被用作操作系统软件驱动层面的信号轮训。

## Docker

前面说了这么多终于来到 Docker，容器的独特之处在于通常只运行一个进程。即使是单进程，容器内 PID 为 1 的进程也具有 init 系统的特殊规则和职责。

PID 1 在 Linux 中非常重要，通常是 init 进程。通常进程在收到 `SIGTERM` 信号后，假如不对信号进程处理，会快速退出。但 PID 1 的进程收到 `SIGTERM` 之后假如不对信号进行处理则**什么都不会做**。

容器内 PID 1 通常有两种情况： shell 进程 PID 为 1 和你的进程 PID 为 1。分别对应着 shell 和 exec 格式的命令。

### shell 格式

Dockerfile 有个特点，就是如果不使用 [JSON 格式](https://docs.docker.com/engine/reference/builder/#run) 来指定容器命令，会通过 shell 以 `fork` 的形式来执行命令，也就是 `/bin/sh -c`。

-   `docker run`（宿主机上）
    -   `/bin/sh -c`（PID 1，容器内）
        -   `/loop.sh` （PID 2，容器内）

这种格式的命令特点是不会向业务进程发送信号。比如发送给 shell 的 `SIGTERM` 信号不会转发给子进程，而是等待子进程的退出。唯一杀死容器的方式就是发送 `SIGKILL` 信号，或者碰巧子进程自己崩溃。

所以应该尽量避免使用这种方式，

### exec 格式

这个就是 Dockerfile 的推荐语法了，你的进程会立即启动并作为容器的初始化进程，然后就有了下面的进程树：

-   `docker run`（宿主机上）
    -   `/loop.sh`（PID 1，容器内）

说了这么多，很多人觉得不够直观。我们会用示例应用来进行说明，但在这之前简单说下如何发送信号来停止容器。

## 发送信号

有几种方式来停止容器。

### docker stop

默认情况下 `docker stop` 命令会向容器发送 `SIGTERM` 信号，然后等待 `10s`，如果容器没停止再发送 `SIGKILL` 信号。

在 Dockerfile 中，可以通过 `STOPSIGNAL` 指令来设置默认的退出信号，比如 `STOPSIGNAL SIGKILL` 将退出信号设置为 `SIGKILL`。或者在 `docker run` 是通过 `--stop-signal` 参数来覆盖镜像中的 `STOPSIGNAL` 设置。

### docker kill

默认情况下 `docker kill` 会直接杀死容器，不给容器任何机会进行优雅停止，这里发出的就是 `SIGKILL` 信号。

当然 `docker kill` 可以通过 `--signal` 来指定要发送的信号，类似 Linux 的 `kill` 命令：

```
docker kill ----signal=SIGTERM foo
```

### docker rm -f

通常情况下 `docker rm` 用来删除已经停止的容器，但是加上 `--force`（简写 `-f`）会强制删除正在运行的容器。同样，也不会给容器任何优化停止的机会。

## 信号处理

我们使用一个简单的应用对 shell 和 exec 两种格式做下对比。在这个应用中，对 `SIGTERM` 进行处理：收到信号后退出。

```
#!/usr/bin/env sh
trap 'exit 0' SIGTERM
while true; do :; done
```

接下来我们使用两种不同格式的 `CMD` 来构建镜像。

### shell 格式

使用下面的 Dockerfile 来构建镜像 _term_。

```
FROM alpine:3.15.0
COPY loop.sh /
CMD /loop.sh
```

执行下面的命令构建镜像、启动容器、停止容器。

```
docker build -t term .
docker run --name term -d term
docker stop term
```

此时你会发现容器并没有立刻停止，而是大约 10s 之后才被停止。可以通过命令查看容器的退出状态：

```
docker inspect -f '{{.State.ExitCode}}' term
137
```

`137 = 128 + 9` 说明容器的退出信号是 `SIGKILL`。

### exec 格式

调整下 Dockerfile，将 `CMD` 修改为推荐的 JSON 格式：

```
FROM ubuntu:trusty
COPY loop.sh /
CMD ["/loop.sh"]
```

执行下面的命令构建镜像、启动容器、停止容器。（需要先执行 `docker rm term` 删除之前停止的容器）

```
docker build -t term .
docker run --name term -d term
docker stop term
```

此时容器会立刻退出。查看容器的退出状态：

```
docker inspect -f '{{.State.ExitCode}}' term
0
```

## 总结

`docker rm -f` 和 `docker kill` 干掉容器很容器，但是为了实现容器的优雅退出，应该使用 `docker stop` 命令，同时 Dockerfile 中应尽量避免使用 shell 格式设置 `ENTRYPONT` 或者 `CMD`。