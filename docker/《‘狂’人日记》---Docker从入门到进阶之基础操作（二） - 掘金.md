   

[![](https://p26-passport.byteacctimg.com/img/user-avatar/e3b82b14bcbe9861a2b50f64dcd89de1~300x300.image)](https://juejin.cn/user/3611262085773719)

![《‘狂’人日记》---Docker从入门到进阶之基础操作（二）](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/67d93340e78e4cc0a7b10a3f59c2c065~tplv-k3u1fbpfcp-zoom-crop-mark:3024:3024:3024:1702.awebp?)  

携手创作，共同成长！这是我参与「掘金日新计划 · 8 月更文挑战」的第24天，[点击查看活动详情](https://juejin.cn/post/7123120819437322247 "https://juejin.cn/post/7123120819437322247")

## 书接上文，讲到了Docker的安装以及ps 和 run命令，那么这里就为大家介绍镜像的命令

## 1.镜像常用命令

## 1.1 使用语法和参数

```
[root@base ~]# docker image --help

Usage:  docker image COMMAND

Manage images

Options:
      --help   Print usage

Commands:
  build       从Dockerfile构建映像
  history     显示image的历史
  import      从压缩文件中导入内容以创建文件系统映像
  inspect     显示一个或多个image的详细信息
  load        从tar存档文件或STDIN加载映像
  ls          image列表
  prune       删除未使用的image
  pull        从注册表中提取映像或存储库
  push        将映像或存储库推入注册表
  rm          删除一个或多个image
  save        保存一个或多个image到tar存档(默认流到STDOUT)
  tag         创建指向SOURCE_IMAGE的标签TARGET_IMAGE

Run 'docker image COMMAND --help' for more information on a command.

复制代码
```

## 1.2 常用命令介绍(部分)

### 1.2.1 ls --- image列表

### 1.2.1.1 使用语法

```
Usage:  docker image ls [OPTIONS] [REPOSITORY[:TAG]]

List images

Aliases:
  ls, images, list

Options:
  -a, --all             显示所有图像(默认隐藏中间图像)
      --digests         显示摘要
  -f, --filter filter   根据提供的条件过滤输出
      --format string   使用Go模板打印漂亮的图片
      --help            打印使用
      --no-trunc        不截断输出
  -q, --quiet           只显示数字id
复制代码
```

#### 1.2.1.2 -a 列出所有Image

```
[root@base ~]# docker image ls -a
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
centos-7            latest              7e989e1e46e7        2 weeks ago         380 MB
docker.io/nginx     latest              b692a91e4e15        3 weeks ago         142 MB
[root@base ~]# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
centos-7            latest              7e989e1e46e7        2 weeks ago         380 MB
docker.io/nginx     latest              b692a91e4e15        3 weeks ago         142 MB


复制代码
```

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0bdee5be0f3b4bdb90c21574ab76df3a~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

#### 1.2.1.3 -q 只显示数字id

```
[root@base ~]# docker image ls -q
7e989e1e46e7
b692a91e4e15

复制代码
```

### 1.2.2 rm --- 删除一个或多个image

#### 1.2.2.1 使用语法

```
Usage:  docker image rm [OPTIONS] IMAGE [IMAGE...]

Remove one or more images

Aliases:
  rm, rmi, remove

Options:
  -f, --force      强制移除image
      --help       Print usage
      --no-prune   Do not delete untagged parents
复制代码
```

#### 1.2.2.2 -f 删除镜像

```
[root@base ~]# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
centos-7            latest              7e989e1e46e7        2 weeks ago         380 MB
docker.io/nginx     latest              b692a91e4e15        3 weeks ago         142 MB
[root@base ~]# docker image rm -f 7e989
Untagged: centos-7:latest
Deleted: sha256:7e989e1e46e7f23498f0ca7da2a9ec7794ff4d7f06b77f7ef6e90222cdb3d346
[root@base ~]# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
docker.io/nginx     latest              b692a91e4e15        3 weeks ago         142 MB

复制代码
```

#### 1.2.2.3 rmi --- 等同于 image rm

```
[root@base ~]# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
docker.io/nginx     latest              b692a91e4e15        3 weeks ago         142 MB
[root@base ~]# docker rmi b692
Untagged: docker.io/nginx:latest
Untagged: docker.io/nginx@sha256:ecc068890de55a75f1a32cc8063e79f90f0b043d70c5fcf28f1713395a4b3d49
Deleted: sha256:b692a91e4e1582db97076184dae0b2f4a7a86b68c4fe6f91affa50ae06369bf5
Deleted: sha256:20fe57e949a4f70bf714590d9d5a78d158d12f4619d148619427a86dfc2e5a7a
Deleted: sha256:042a89e2d80d230c47e0f2add6e13a5958cf18b039f04a3751200937ef76ba03
Deleted: sha256:9e20f968300754f2d3ace5b726448b9a4249bb8196aded53b36ae8b6d3e8c174
Deleted: sha256:15e9cede496de643a978a58c0b49ef7beea83b368dfefc2e46fa0e8dd589f099
Deleted: sha256:d2850ddb0c4ca9f6289f624b27f987873e556c41250f5c5ed47a69c6c2529e4b
Deleted: sha256:92a4e8a3140f7a04a0e5a15793adef2d0e8889ed306a8f95a6cfb67cecb5f212
[root@base ~]# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE

复制代码
```

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/765bdccc85bd49b9a7b7ef7fa02cdca4~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

### 1.2.3 build --- 从Dockerfile构建映像

#### 1.2.3.1 使用语法(常用)

```
Usage:  docker image build [OPTIONS] PATH | URL | -

Build an image from a Dockerfile

Options:
      --build-arg list             设置构建时变量(默认[])
      --cache-from stringSlice     将镜像视为缓存源
      --cgroup-parent string       可选父容器cgroup
      --compress                   使用gzip压缩构建上下文
      --cpu-period int             限制CPU CFS(完全公平调度程序)周期
      --cpu-quota int              限制CPU CFS(完全公平调度程序)配额
  -c, --cpu-shares int             CPU份额(相对权重)
      --cpuset-cpus string         允许执行的cpu(0-3、0、1)
      --cpuset-mems string         允许执行的MEMs (0- 3,0,1)
      --disable-content-trust      跳过图像验证(默认为true)
  -f, --file string                Dockerfile的名称(默认为'PATH/Dockerfile')
      --force-rm                   总是删除中间容器
      --help                       Print usage
      --isolation string           容器隔离技术
      --label list                 设置image的元数据(默认[])
  -m, --memory string              内存限制
      --memory-swap string         交换限制等于内存加上交换:'-1'来启用无限交换
      --network string             在构建期间为RUN指令设置网络模式(默认为"default")
      --no-cache                   在构建映像时不使用缓存
      --pull                       总是尝试拉出更新版本的image
  -q, --quiet                      关闭构建输出并在成功时打印image ID
      --rm                         在成功构建后移除中间容器(默认为true)
      --security-opt stringSlice   安全选项
      --shm-size string            “/dev/shm”的大小，默认为64MB
  -t, --tag list                   以' Name:tag'格式命名和可选的标记(default [])
      --ulimit ulimit              Ulimit选项(默认[])
  -v, --volume list                设置构建时绑定挂载(默认[])

复制代码
```

#### 1.2.3.2 构建一个带有openssh-server的容器镜像

```
[root@base ~]# cat Dockerfile
FROM centos:7

RUN yum install -y openssh-server sudo
RUN sed -i 's/UsePAM yes/UsePAM no/g' /etc/ssh/sshd_config

#RUN useradd admin
#RUN echo "admin:admin" | chpasswd
RUN echo "root:123456" | chpasswd
#RUN echo "admin  ALL=(ALL)    ALL" >> /etc/sudoers

RUN ssh-keygen -t dsa -f /etc/ssh/ssh_host_dsa_key
RUN ssh-keygen -t rsa -f /etc/ssh/ssh_host_rsa_key

RUN mkdir /var/run/sshd
EXPOSE 22
CMD ["/usr/sbin/sshd", "-D"]
[root@base ~]# docker build -t ssh-server .

复制代码
```

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/55ff9ed6562149e6b3640651567bae3a~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0dfb99444f5e4de195cdd7587529cef4~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

#### 1.2.3.3 从构建的镜像拉起容器

```
[root@base ~]# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
ssh-server          latest              4e28f6e4fc6a        15 minutes ago      390 MB
docker.io/centos    7                   eeb6ee3f44bd        11 months ago       204 MB
[root@base ~]# docker run -dit --name sshd -p 10022:22 ssh-server
53b462f31348b9fc0bdda53084f374782d1c073155c5445434d555d988a6264d
[root@base ~]# ssh localhost:10022
ssh: Could not resolve hostname localhost:10022: Name or service not known
[root@base ~]# docker ps -a
CONTAINER ID        IMAGE               COMMAND               CREATED             STATUS                     PORTS                   NAMES
53b462f31348        ssh-server          "/usr/sbin/sshd -D"   11 seconds ago      Up 10 seconds              0.0.0.0:10022->22/tcp   sshd
88958f0f4fc8        7e989e1e46e7        "/bin/bash"           2 weeks ago         Exited (137) 2 weeks ago                           centos7


复制代码
```

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2d50fe42a9fe462fb26e251d8b640763~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

![](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/00ba359ecd0075e59ffbc3d810af551d.svg) 3

![](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/3d482c7a948bac826e155953b2a28a9e.svg) 收藏