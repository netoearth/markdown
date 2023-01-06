　　　　　　　　　　　　　**Docker镜像管理基础篇**

　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　**作者：尹正杰**

**版权声明：原创作品，谢绝转载！否则将追究法律责任。**

**一.Docker Images** 

```
Docker镜像还有启动容器所需要的文件系统及其内容，因此，其用于创建并启动docker容器。
```

**1>.采用分层构建机制，最底层为bootfs，向上为rootfs**

****![](https://img2018.cnblogs.com/blog/795254/201910/795254-20191016191218874-401491389.png)****

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
bootfs：　　用于系统引导的文件系统，包括bootloader和kernel，容器启动完成后会被卸载以节约内存资源。

rootfs：　　如上图所示，rootfs位于bootfs之上，表现为docker容器的根文件系统。　　传统模式中，系统启动之时，内核挂载rootfs时会首先将其挂载为"只读"模式，完整性自检完成后将其重新挂载为读写模式。　　docker中，rootfs由内核挂载为"只读"模式，而后通过"联合挂载"技术额外挂载一个"可写"层。
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

**2>.Docker Image Layer**

**![](https://img2018.cnblogs.com/blog/795254/201910/795254-20191016191638013-1882653117.png)**

```
如上图所示，位于下层的镜像称为父镜像（parent image），最底层的称为基础镜像（base image）。

最上层为"可读写"层，其下的均为"只读"层。
```

**二.Docker的分层镜像所支持的文件系统**

**1>.auFS**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
　　镜像的分层构建和联合挂载依赖于转有的文件系统。　　　　advanced multi-layered unification filesystem(高级多层统一文件系统，简称Aufs),用于为Linux文件系统实现"联合挂载"。　　aufs是之前UnionFS的重新实现，2006年由Junjiro Okajima开发。（据说aufs源代码有3w多行，而ext文件系统也才5k多行左右，作者曾4次申请向Linux内核合并都被拒，因此我们要使用这种文件系统需要自己给内核打补丁。）　　Docker最初使用aufs作为容器文件系统层，它目前仍作为存储后端之一来支持。　　aufs的竞争产品是overlayfs，后者自从3.18版本开始被合并到Linux内核。　　docker的分层镜像，除了aufs，docker还支持btrfs，devicemapper和vfs等(在Ubantu系统下，docker默认Ubuntu的aufs；而在CentOS 7上，用的是devicemapper)。
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

**2>.Devicemapper**

**![](https://img2018.cnblogs.com/blog/795254/201910/795254-20191016194248271-104739313.png)**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
　　Device Mapper 是Linux2.6内核中支持逻辑卷管理的通用设备映机制，它为实现用于存储资源管理的块设备驱动提供了一个高度模块化的内核架构。　　 在内核中它通过一个一个模块化的target driver（如上图中的linear,mirror,snapshot,multipath）插件实现对IO请求的过滤或者重新定向等工作，当前已经实现的target driver插件包括软raid，软加密，逻辑卷条带，多路径，镜像，快照等。　　如下图所示，在这诸多"插件"中，有一种叫Thin Provisioning Snapshot,Docker正是使用了Thin Provisioning的Snapshot的技术实现了类似auFS的分层镜像。　　由于Devicemapper并不太稳定，性能很差，我们了解即可，无需深入了解，感兴趣的小伙伴可自行查阅资料。
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

![](https://img2018.cnblogs.com/blog/795254/201910/795254-20191016194319174-1498521844.png)

**3>.overlay2**

```
　　我们知道Devicemapper并不太稳定，性能极差，在最新版本安装中的docker中，我们使用"docker info"命令可以看到"Storage Driver"为"overlay2"。

　　overlay2是一种抽象的二级文件系统，它需要建构在本地文件系统(CentOS 7.x版本中使用"docker info"命令可以看到"Backing FileSystem"为"xfs")之上，CentOS 7默认是xfs文件系统，因此overlay2是默认是建立在xfs之上的，了解即可。
```

**三.Registry**

**1>.什么是Registry**

```
　　我们去构建镜像时，镜像做好之后应该有一个统一存放位置，我们称之为Docker仓库，Registry是存放Docker镜像的仓库（官方默认仓库在"https://hub.docker.com"），Registry分私有和公有两种。Images和Registry之间默认使用的时https协议，当然如果你非要指定为http协议也是可以的。
　　启动容器时，docker daemon会试图从本地获取相关的镜像；本地镜像不存在时，其将从Registry中下载该镜像并保存到本地。
　　　　Registry用于保存docker镜像，包括镜像的层次结构和元数据。用户可自建Registry，也可使用官方的Docker Hub。
```

**2>.Docker Registry分类**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
　　Sponsor Registry:　　　　第三方的registry，供客户端和Docker社区使用。
　　　　　　Mirror Registry:　　　　第三方的registry，只让客户使用。
　　　　　　Vendor Registry:　　　　由发布Docker镜像的供应商提供的registry。
　　　　　　Private Registry:　　　　通过设有防火墙和额外的安全层的私有实体提供的registry。
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

**3>.Registry包括Repository和Index**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
　　Repository:
　　　　由某特定的docker镜像的所有迭代版本组成的镜像仓库；
　　　　一个Registry中可以存在多个Repository：
　　　　　　Repository可分为"顶层仓库"和"用户仓库"；
　　　　　　用户仓库名称格式为"用户名/仓库名"。
　　　　每个仓库可以包含多个Tag（标签），每个标签对应一个镜像；
　　　　　　Index：
　　　　　　维护用户账户，镜像的校验以及公共命名空间的信息；
　　　　　　相当于为Registry提供了一个完成用户认证等功能的检索接口。
　　如下图所示，Docker Registry中的镜像通常由开发人员制作，而后推送至"公共"或"私有"Registry上保存，供其它人员使用，例如"部署"到生产环境。
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

 ![](https://img2018.cnblogs.com/blog/795254/201910/795254-20191016213837779-173225942.png)

**四.Docker Hub功能概述**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
　　Docker Hub是Docker官方默认镜像服务器，如果有需要的小伙伴可以自行去官方注册，链接地址：https://hub.docker.com。除了官方默认的镜像服务还有一个著名的镜像地址：https://quay.io，除此之外，国内也有很多优秀的镜像地址，比如阿里源，清华源等。

　　接下来我们简单对Docker Hub的功能做一个简单描述：
　　　　Image Repositories（镜像仓库）:
　　　　　　存放自己的镜像仓库。

　　　　Automated builds（自动构建）：
　　　　　　当仓库的源代码发生改变时它可以自动创建新的镜像。

　　　　Webhooks（web钩子）：
　　　　　　和Automated builds有关，当代码被成功push到repository后会触发的动作称为webhooks。

　　　　Organization（组织）：
　　　　　　创建工作组来协同工作。

　　　　GitHub and Bitbucket Integration：
　　　　　　可以与GitHub或者Bitbucket进行整合。
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

**五.镜像相关的操作**

**1>.镜像的生成途径**

```
目前有三者中途径可以制作镜像：　　1>.Dockerfile　　2>.基于容器制作　　3>.Docker Hub automated builds（其实还是基于Dockerfile实现的）
```

![](https://img2018.cnblogs.com/blog/795254/201910/795254-20191016222826155-876647045.png)

**2>.基于容器做镜像**

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node101.yinzhengjie.org.cn ~]# docker commit --help　　　　　　　　#查看该命令的帮助信息

Usage:    docker commit [OPTIONS] CONTAINER [REPOSITORY[:TAG]]

Create a new image from a container's changes

Options:
  -a, --author string    Author (e.g., "John Hannibal Smith <hannibal@a-team.com>")
  -c, --change list      Apply Dockerfile instruction to the created image
  -m, --message string   Commit message
  -p, --pause            Pause container during commit (default true)
[root@node101.yinzhengjie.org.cn ~]# 
```

\[root@node101.yinzhengjie.org.cn ~\]# docker commit --help　　　　　　　　#查看该命令的帮助信息

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@node101.yinzhengjie.org.cn ~]# docker run --name b1 -it busybox　　　　#启动一个docker容器，不要停止它
/ # 
/ # mkdir -pv /data/html
created directory: '/data/'
created directory: '/data/html'
/ # 
/ # vi /data/html/index.html
/ # 
/ # cat /data/html/index.html　　　　　　　　　　　　　　　　　　　　　　　　　　　　#对b1这个容器做了一些修改后，咱们需要另外开启一个终端
<h1>Busybox httpd server.[Jason Yin dao ci yi you !!!]</h1>
/ # 
/ # 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

![](https://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif)

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@node101.yinzhengjie.org.cn ~]# docker container ls　　　　　　#不难发现b1容器正在运行状态
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
098572441abd        busybox             "sh"                     6 minutes ago       Up 6 minutes                            b1
b16c5b671f31        redis:4-alpine      "docker-entrypoint.s…"   5 hours ago         Up 5 hours          6379/tcp            redis01
ce713ceaa586        nginx:1.14-alpine   "nginx -g 'daemon of…"   5 hours ago         Up 5 hours          80/tcp              web01
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# docker image ls　　　　　　　　#查看现有镜像
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
busybox             latest              19485c79a9bb        6 weeks ago         1.22MB
redis               4-alpine            442c73e7ad1d        8 weeks ago         20.4MB
nginx               1.14-alpine         8a2fb25a19f5        6 months ago        16MB
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# docker commit -p b1　　　　　　#创建一个镜像，建议创建镜像时添加"-p"选项，目的是创建镜像时暂停当前容器的运行
sha256:76d5e6c143b2566fd67b86badc4d526542ce65f9caf85978d213fddb6475c7fa
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# docker image ls　　　　　　　　#很显然创建成功了，但是没有REPOSITORY和TAG相关信息。
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
<none>              <none>              76d5e6c143b2        3 seconds ago       1.22MB
busybox             latest              19485c79a9bb        6 weeks ago         1.22MB
redis               4-alpine            442c73e7ad1d        8 weeks ago         20.4MB
nginx               1.14-alpine         8a2fb25a19f5        6 months ago        16MB
[root@node101.yinzhengjie.org.cn ~]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

![](https://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif)

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@node101.yinzhengjie.org.cn ~]# docker image ls　　　　　　　　#很显然创建成功了，但是没有REPOSITORY和TAG相关信息。
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
<none>              <none>              76d5e6c143b2        3 seconds ago       1.22MB
busybox             latest              19485c79a9bb        6 weeks ago         1.22MB
redis               4-alpine            442c73e7ad1d        8 weeks ago         20.4MB
nginx               1.14-alpine         8a2fb25a19f5        6 months ago        16MB
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# docker tag 76d5e6c143b2 jason/httpd:v0.1-1　　　　　　#我们第一次为自建的镜像打标签
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# docker image ls
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
jason/httpd         v0.1-1              76d5e6c143b2        4 minutes ago       1.22MB
busybox             latest              19485c79a9bb        6 weeks ago         1.22MB
redis               4-alpine            442c73e7ad1d        8 weeks ago         20.4MB
nginx               1.14-alpine         8a2fb25a19f5        6 months ago        16MB
[root@node101.yinzhengjie.org.cn ~]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

![](https://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif)

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@node101.yinzhengjie.org.cn ~]# docker image ls
REPOSITORY TAG IMAGE ID CREATED SIZE
jason/httpd v0.1-1 76d5e6c143b2 4 minutes ago 1.22MB
busybox latest 19485c79a9bb 6 weeks ago 1.22MB
redis 4-alpine 442c73e7ad1d 8 weeks ago 20.4MB
nginx 1.14-alpine 8a2fb25a19f5 6 months ago 16MB
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]#
[root@node101.yinzhengjie.org.cn ~]# docker tag jason/httpd:v0.1-1 jason/httpd:latest　　　　#我们第二次为自建的镜像打标签，这说明一个镜像可以设置多个标签
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# docker image ls
REPOSITORY TAG IMAGE ID CREATED SIZE
jason/httpd latest 76d5e6c143b2 5 minutes ago 1.22MB
jason/httpd v0.1-1 76d5e6c143b2 5 minutes ago 1.22MB
busybox latest 19485c79a9bb 6 weeks ago 1.22MB
redis 4-alpine 442c73e7ad1d 8 weeks ago 20.4MB
nginx 1.14-alpine 8a2fb25a19f5 6 months ago 16MB
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]#
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node101.yinzhengjie.org.cn ~]# docker image ls　　　　　　#不难发现，“jason/httpd:v0.1-1“和“jason/httpd:latest”两个镜像都指向了同一个IMAGE ID,即这2个镜像表示为同一个镜像，TAG分别对该镜像有2次引用而已。
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
jason/httpd         latest              76d5e6c143b2        17 hours ago        1.22MB
jason/httpd         v0.1-1              76d5e6c143b2        17 hours ago        1.22MB
busybox             latest              19485c79a9bb        6 weeks ago         1.22MB
redis               4-alpine            442c73e7ad1d        8 weeks ago         20.4MB
nginx               1.14-alpine         8a2fb25a19f5        6 months ago        16MB
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# docker image rm jason/httpd:latest　　　　#我们删除一个对IMAGE ID为"76d5e6c143b2"的引用，他不会真的去删除该镜像，因为还有另一个TAG在使用该镜像哟。
Untagged: jason/httpd:latest
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# docker image ls
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
jason/httpd         v0.1-1              76d5e6c143b2        17 hours ago        1.22MB
busybox             latest              19485c79a9bb        6 weeks ago         1.22MB
redis               4-alpine            442c73e7ad1d        8 weeks ago         20.4MB
nginx               1.14-alpine         8a2fb25a19f5        6 months ago        16MB
[root@node101.yinzhengjie.org.cn ~]# 
```

\[root@node101.yinzhengjie.org.cn ~\]# docker image rm jason/httpd:latest　　#我们删除一个对IMAGE ID为"76d5e6c143b2"的引用，他不会真的去删除该镜像，因为还有另一个TAG在使用该镜像哟。

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@node101.yinzhengjie.org.cn ~]# docker image ls　　　　　　　　#查看我们自定义的镜像
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
jason/httpd         v0.1-1              76d5e6c143b2        17 hours ago        1.22MB
busybox             latest              19485c79a9bb        6 weeks ago         1.22MB
redis               4-alpine            442c73e7ad1d        8 weeks ago         20.4MB
nginx               1.14-alpine         8a2fb25a19f5        6 months ago        16MB
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# docker run --name t1 -it jason/httpd:v0.1-1 　　　　#使用咱们自定义的镜像启动一个名称为t1的容器观察是否有我们之前的测试数据。
/ # 
/ # ls /data/html/　　　　　　　　　　  #果不其然，该目录是存在的，该目录是咱们基于busybox镜像创建的目录
index.html
/ # 
/ # cat /data/html/index.html 　　　　#不仅如此，里面的文件内容也是咱们上面修改的，至此，咱们的基于容器创建镜像就是如此的简单。
<h1>Busybox httpd server.[Jason Yin dao ci yi you !!!]</h1>
/ # 
/ # 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

![](https://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif)

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@node101.yinzhengjie.org.cn ~]# docker inspect busybox　　　　　　　　　　　　  #查看busybox镜像的元数据信息，因为咱们基于它创建的自定义镜像，尤其观察"Cmd属性"。
[
    {
        "Id": "sha256:19485c79a9bbdca205fce4f791efeaa2a103e23431434696cc54fdd939e9198d",
        "RepoTags": [
            "busybox:latest"
        ],
        "RepoDigests": [
            "busybox@sha256:fe301db49df08c384001ed752dff6d52b4305a73a7f608f21528048e8a08b51e"
        ],
        "Parent": "",
        "Comment": "",
        "Created": "2019-09-04T19:20:16.230463098Z",
        "Container": "e30cd53834b3dfdb989c63cc73f4f31f404c7a6a0c0e9d6b9e3e8451edd72596",
        "ContainerConfig": {
            "Hostname": "e30cd53834b3",
            "Domainname": "",
            "User": "",
            "AttachStdin": false,
            "AttachStdout": false,
            "AttachStderr": false,
            "Tty": false,
            "OpenStdin": false,
            "StdinOnce": false,
            "Env": [
                "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
            ],
            "Cmd": [
                "/bin/sh",
                "-c",
                "#(nop) ",
                "CMD [\"sh\"]"　　　　　　#不难发现，基于该镜像创建的文件默认会继承sh属性。
            ],
            "ArgsEscaped": true,
            "Image": "sha256:758a17a836a4c09586a291c928d1f0561320e252d07c4749e14338daefe84b18",
            "Volumes": null,
            "WorkingDir": "",
            "Entrypoint": null,
            "OnBuild": null,
            "Labels": {}
        },
        "DockerVersion": "18.06.1-ce",
        "Author": "",
        "Config": {
            "Hostname": "",
            "Domainname": "",
            "User": "",
            "AttachStdin": false,
            "AttachStdout": false,
            "AttachStderr": false,
            "Tty": false,
            "OpenStdin": false,
            "StdinOnce": false,
            "Env": [
                "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
            ],
            "Cmd": [
                "sh"
            ],
            "ArgsEscaped": true,
            "Image": "sha256:758a17a836a4c09586a291c928d1f0561320e252d07c4749e14338daefe84b18",
            "Volumes": null,
            "WorkingDir": "",
            "Entrypoint": null,
            "OnBuild": null,
            "Labels": null
        },
        "Architecture": "amd64",
        "Os": "linux",
        "Size": 1219782,
        "VirtualSize": 1219782,
        "GraphDriver": {
            "Data": {
                "MergedDir": "/var/lib/docker/overlay2/b770b052c1df66b22f2bbc171e637e15b1cc3ff49b4ae3eca3d32f5ad1b778d3/merged",
                "UpperDir": "/var/lib/docker/overlay2/b770b052c1df66b22f2bbc171e637e15b1cc3ff49b4ae3eca3d32f5ad1b778d3/diff",
                "WorkDir": "/var/lib/docker/overlay2/b770b052c1df66b22f2bbc171e637e15b1cc3ff49b4ae3eca3d32f5ad1b778d3/work"
            },
            "Name": "overlay2"
        },
        "RootFS": {
            "Type": "layers",
            "Layers": [
                "sha256:6c0ea40aef9d2795f922f4e8642f0cd9ffb9404e6f3214693a1fd45489f38b44"
            ]
        },
        "Metadata": {
            "LastTagTime": "0001-01-01T00:00:00Z"
        }
    }
]
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node101.yinzhengjie.org.cn ~]# docker inspect jason/httpd:v0.1-1　　　　　　#查看咱们自建镜像的元数据信息，尤其观察"Cmd"的属性
[
    {
        "Id": "sha256:76d5e6c143b2566fd67b86badc4d526542ce65f9caf85978d213fddb6475c7fa",
        "RepoTags": [
            "jason/httpd:v0.1-1"
        ],
        "RepoDigests": [],
        "Parent": "sha256:19485c79a9bbdca205fce4f791efeaa2a103e23431434696cc54fdd939e9198d",
        "Comment": "",
        "Created": "2019-10-17T05:04:21.325141974Z",
        "Container": "098572441abd3b00688d9542119a5a46132f8ec2c8d8cf9219e9b47a042787a0",
        "ContainerConfig": {
            "Hostname": "098572441abd",
            "Domainname": "",
            "User": "",
            "AttachStdin": true,
            "AttachStdout": true,
            "AttachStderr": true,
            "Tty": true,
            "OpenStdin": true,
            "StdinOnce": true,
            "Env": [
                "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
            ],
            "Cmd": [
                "sh"　　　　　　#我们发现它继承自busybox镜像中的sh。
            ],
            "Image": "busybox",
            "Volumes": null,
            "WorkingDir": "",
            "Entrypoint": null,
            "OnBuild": null,
            "Labels": {}
        },
        "DockerVersion": "19.03.3",
        "Author": "",
        "Config": {
            "Hostname": "098572441abd",
            "Domainname": "",
            "User": "",
            "AttachStdin": true,
            "AttachStdout": true,
            "AttachStderr": true,
            "Tty": true,
            "OpenStdin": true,
            "StdinOnce": true,
            "Env": [
                "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
            ],
            "Cmd": [
                "sh"
            ],
            "Image": "busybox",
            "Volumes": null,
            "WorkingDir": "",
            "Entrypoint": null,
            "OnBuild": null,
            "Labels": {}
        },
        "Architecture": "amd64",
        "Os": "linux",
        "Size": 1219914,
        "VirtualSize": 1219914,
        "GraphDriver": {
            "Data": {
                "LowerDir": "/var/lib/docker/overlay2/b770b052c1df66b22f2bbc171e637e15b1cc3ff49b4ae3eca3d32f5ad1b778d3/diff",
                "MergedDir": "/var/lib/docker/overlay2/06fb92a39e00ab668989827564a7902fbfae860640917d05796e2eccaf192d72/merged",
                "UpperDir": "/var/lib/docker/overlay2/06fb92a39e00ab668989827564a7902fbfae860640917d05796e2eccaf192d72/diff",
                "WorkDir": "/var/lib/docker/overlay2/06fb92a39e00ab668989827564a7902fbfae860640917d05796e2eccaf192d72/work"
            },
            "Name": "overlay2"
        },
        "RootFS": {
            "Type": "layers",
            "Layers": [
                "sha256:6c0ea40aef9d2795f922f4e8642f0cd9ffb9404e6f3214693a1fd45489f38b44",
                "sha256:8191d8bde55887d65a8851cdfa7fd613161baca8c91bf4eb78e32290836a6e6b"
            ]
        },
        "Metadata": {
            "LastTagTime": "2019-10-17T13:10:07.061823165+08:00"
        }
    }
]
[root@node101.yinzhengjie.org.cn ~]# 
```

\[root@node101.yinzhengjie.org.cn ~\]# docker inspect jason/httpd:v0.1-1　　　　　　#查看咱们自建镜像的元数据信息，尤其观察"Cmd"的属性

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node101.yinzhengjie.org.cn ~]# docker commit -h
Flag shorthand -h has been deprecated, please use --help

Usage:    docker commit [OPTIONS] CONTAINER [REPOSITORY[:TAG]]

Create a new image from a container's changes

Options:
-a, --author string Author (e.g., "John Hannibal Smith <hannibal@a-team.com>")
-c, --change list Apply Dockerfile instruction to the created image
-m, --message string Commit message
-p, --pause Pause container during commit (default true)
[root@node101.yinzhengjie.org.cn ~]#
[root@node101.yinzhengjie.org.cn ~]# docker image ls
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
jason/httpd         v0.1-1              76d5e6c143b2        17 hours ago        1.22MB
busybox             latest              19485c79a9bb        6 weeks ago         1.22MB
redis               4-alpine            442c73e7ad1d        8 weeks ago         20.4MB
nginx               1.14-alpine         8a2fb25a19f5        6 months ago        16MB
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# docker commit -a "YinZhengjie <y1053419035@qq.com>" -c 'CMD ["/bin/httpd","-f","-h","/data/html"]' -p b1 jason/httpd:v0.2   #创建镜像时指定默认的CMD命令。 
sha256:78fb6601880fdaa19439d2288b9f5800eb2e8e079a7ed212c380540e0e2c1246
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# docker image ls
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
jason/httpd         v0.2                78fb6601880f        16 seconds ago      1.22MB
jason/httpd         v0.1-1              76d5e6c143b2        17 hours ago        1.22MB
busybox             latest              19485c79a9bb        6 weeks ago         1.22MB
redis               4-alpine            442c73e7ad1d        8 weeks ago         20.4MB
nginx               1.14-alpine         8a2fb25a19f5        6 months ago        16MB
[root@node101.yinzhengjie.org.cn ~]# 
 
```

\[root@node101.yinzhengjie.org.cn ~\]# docker commit -a "YinZhengjie <y1053419035@qq.com>" -c 'CMD \["/bin/httpd","-f","-h","/data/html"\]' -p b1 jason/httpd:v0.2 　　#创建镜像时指定默认的CMD命令。

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@node101.yinzhengjie.org.cn ~]# docker image ls　　　　　　　　　　　　　　　　　　#我们发现咱们自建的镜像已经成功啦
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
jason/httpd         v0.2                78fb6601880f        5 minutes ago       1.22MB
jason/httpd         v0.1-1              76d5e6c143b2        17 hours ago        1.22MB
busybox             latest              19485c79a9bb        6 weeks ago         1.22MB
redis               4-alpine            442c73e7ad1d        8 weeks ago         20.4MB
nginx               1.14-alpine         8a2fb25a19f5        6 months ago        16MB
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# docker run --name t2 jason/httpd:v0.2　　　　　　#紧接着我们基于咱们自建的镜像启动容器，发现阻塞到当前命令行了，发现不会自动进入到一个交互式命令行了，这很正常，因为咱们在创建镜像时指定的命令为"/bin/httpd"，此时我们需要另开一个终端查看容器是否正常运行。
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node101.yinzhengjie.org.cn ~]# docker container ls　　　　　　　　#我们发现咱们t2容器是在正常运行的
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
4c436db6d283        jason/httpd:v0.2    "/bin/httpd -f -h /d…"   21 seconds ago      Up 21 seconds                           t2
098572441abd        busybox             "sh"                     17 hours ago        Up 17 hours                             b1
b16c5b671f31        redis:4-alpine      "docker-entrypoint.s…"   22 hours ago        Up 22 hours         6379/tcp            redis01
ce713ceaa586        nginx:1.14-alpine   "nginx -g 'daemon of…"   22 hours ago        Up 22 hours         80/tcp              web01
[root@node101.yinzhengjie.org.cn ~]# 
```

\[root@node101.yinzhengjie.org.cn ~\]# docker container ls　　　　　　　　#我们发现咱们t2容器是在正常运行的

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node101.yinzhengjie.org.cn ~]# docker inspect busybox　　　　　　#查看busybox镜像的元数据信息，注意观察“Cmd”的属性哟~
[
    {
        "Id": "sha256:19485c79a9bbdca205fce4f791efeaa2a103e23431434696cc54fdd939e9198d",
        "RepoTags": [
            "busybox:latest"
        ],
        "RepoDigests": [
            "busybox@sha256:fe301db49df08c384001ed752dff6d52b4305a73a7f608f21528048e8a08b51e"
        ],
        "Parent": "",
        "Comment": "",
        "Created": "2019-09-04T19:20:16.230463098Z",
        "Container": "e30cd53834b3dfdb989c63cc73f4f31f404c7a6a0c0e9d6b9e3e8451edd72596",
        "ContainerConfig": {
            "Hostname": "e30cd53834b3",
            "Domainname": "",
            "User": "",
            "AttachStdin": false,
            "AttachStdout": false,
            "AttachStderr": false,
            "Tty": false,
            "OpenStdin": false,
            "StdinOnce": false,
            "Env": [
                "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
            ],
            "Cmd": [
                "/bin/sh",
                "-c",
                "#(nop) ",
                "CMD [\"sh\"]"
            ],
            "ArgsEscaped": true,
            "Image": "sha256:758a17a836a4c09586a291c928d1f0561320e252d07c4749e14338daefe84b18",
            "Volumes": null,
            "WorkingDir": "",
            "Entrypoint": null,
            "OnBuild": null,
            "Labels": {}
        },
        "DockerVersion": "18.06.1-ce",
        "Author": "",
        "Config": {
            "Hostname": "",
            "Domainname": "",
            "User": "",
            "AttachStdin": false,
            "AttachStdout": false,
            "AttachStderr": false,
            "Tty": false,
            "OpenStdin": false,
            "StdinOnce": false,
            "Env": [
                "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
            ],
            "Cmd": [
                "sh"
            ],
            "ArgsEscaped": true,
            "Image": "sha256:758a17a836a4c09586a291c928d1f0561320e252d07c4749e14338daefe84b18",
            "Volumes": null,
            "WorkingDir": "",
            "Entrypoint": null,
            "OnBuild": null,
            "Labels": null
        },
        "Architecture": "amd64",
        "Os": "linux",
        "Size": 1219782,
        "VirtualSize": 1219782,
        "GraphDriver": {
            "Data": {
                "MergedDir": "/var/lib/docker/overlay2/b770b052c1df66b22f2bbc171e637e15b1cc3ff49b4ae3eca3d32f5ad1b778d3/merged",
                "UpperDir": "/var/lib/docker/overlay2/b770b052c1df66b22f2bbc171e637e15b1cc3ff49b4ae3eca3d32f5ad1b778d3/diff",
                "WorkDir": "/var/lib/docker/overlay2/b770b052c1df66b22f2bbc171e637e15b1cc3ff49b4ae3eca3d32f5ad1b778d3/work"
            },
            "Name": "overlay2"
        },
        "RootFS": {
            "Type": "layers",
            "Layers": [
                "sha256:6c0ea40aef9d2795f922f4e8642f0cd9ffb9404e6f3214693a1fd45489f38b44"
            ]
        },
        "Metadata": {
            "LastTagTime": "0001-01-01T00:00:00Z"
        }
    }
]
[root@node101.yinzhengjie.org.cn ~]# 
```

\[root@node101.yinzhengjie.org.cn ~\]# docker inspect busybox　　　　　　#查看busybox镜像的元数据信息，注意观察“Cmd”的属性哟~

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node101.yinzhengjie.org.cn ~]# docker inspect t2　　　　　　　　　 #查看咱们启动的t2容器的元数据信息，可以和"busybox"镜像稍作对比，注意观察"Cmd"的属性哟~
[
    {
        "Id": "4c436db6d283cbc3b48a7d5bd3221a0c85115aa80d39d79f77408a663f982aeb",
        "Created": "2019-10-17T22:03:56.226968309Z",
        "Path": "/bin/httpd",
        "Args": [
            "-f",
            "-h",
            "/data/html"
        ],
        "State": {
            "Status": "running",
            "Running": true,
            "Paused": false,
            "Restarting": false,
            "OOMKilled": false,
            "Dead": false,
            "Pid": 34800,
            "ExitCode": 0,
            "Error": "",
            "StartedAt": "2019-10-17T22:03:56.601818034Z",
            "FinishedAt": "0001-01-01T00:00:00Z"
        },
        "Image": "sha256:78fb6601880fdaa19439d2288b9f5800eb2e8e079a7ed212c380540e0e2c1246",
        "ResolvConfPath": "/var/lib/docker/containers/4c436db6d283cbc3b48a7d5bd3221a0c85115aa80d39d79f77408a663f982aeb/resolv.conf",
        "HostnamePath": "/var/lib/docker/containers/4c436db6d283cbc3b48a7d5bd3221a0c85115aa80d39d79f77408a663f982aeb/hostname",
        "HostsPath": "/var/lib/docker/containers/4c436db6d283cbc3b48a7d5bd3221a0c85115aa80d39d79f77408a663f982aeb/hosts",
        "LogPath": "/var/lib/docker/containers/4c436db6d283cbc3b48a7d5bd3221a0c85115aa80d39d79f77408a663f982aeb/4c436db6d283cbc3b48a7d5
bd3221a0c85115aa80d39d79f77408a663f982aeb-json.log",        "Name": "/t2",
        "RestartCount": 0,
        "Driver": "overlay2",
        "Platform": "linux",
        "MountLabel": "",
        "ProcessLabel": "",
        "AppArmorProfile": "",
        "ExecIDs": null,
        "HostConfig": {
            "Binds": null,
            "ContainerIDFile": "",
            "LogConfig": {
                "Type": "json-file",
                "Config": {}
            },
            "NetworkMode": "default",
            "PortBindings": {},
            "RestartPolicy": {
                "Name": "no",
                "MaximumRetryCount": 0
            },
            "AutoRemove": false,
            "VolumeDriver": "",
            "VolumesFrom": null,
            "CapAdd": null,
            "CapDrop": null,
            "Capabilities": null,
            "Dns": [],
            "DnsOptions": [],
            "DnsSearch": [],
            "ExtraHosts": null,
            "GroupAdd": null,
            "IpcMode": "private",
            "Cgroup": "",
            "Links": null,
            "OomScoreAdj": 0,
            "PidMode": "",
            "Privileged": false,
            "PublishAllPorts": false,
            "ReadonlyRootfs": false,
            "SecurityOpt": null,
            "UTSMode": "",
            "UsernsMode": "",
            "ShmSize": 67108864,
            "Runtime": "runc",
            "ConsoleSize": [
                0,
                0
            ],
            "Isolation": "",
            "CpuShares": 0,
            "Memory": 0,
            "NanoCpus": 0,
            "CgroupParent": "",
            "BlkioWeight": 0,
            "BlkioWeightDevice": [],
            "BlkioDeviceReadBps": null,
            "BlkioDeviceWriteBps": null,
            "BlkioDeviceReadIOps": null,
            "BlkioDeviceWriteIOps": null,
            "CpuPeriod": 0,
            "CpuQuota": 0,
            "CpuRealtimePeriod": 0,
            "CpuRealtimeRuntime": 0,
            "CpusetCpus": "",
            "CpusetMems": "",
            "Devices": [],
            "DeviceCgroupRules": null,
            "DeviceRequests": null,
            "KernelMemory": 0,
            "KernelMemoryTCP": 0,
            "MemoryReservation": 0,
            "MemorySwap": 0,
            "MemorySwappiness": null,
            "OomKillDisable": false,
            "PidsLimit": null,
            "Ulimits": null,
            "CpuCount": 0,
            "CpuPercent": 0,
            "IOMaximumIOps": 0,
            "IOMaximumBandwidth": 0,
            "MaskedPaths": [
                "/proc/asound",
                "/proc/acpi",
                "/proc/kcore",
                "/proc/keys",
                "/proc/latency_stats",
                "/proc/timer_list",
                "/proc/timer_stats",
                "/proc/sched_debug",
                "/proc/scsi",
                "/sys/firmware"
            ],
            "ReadonlyPaths": [
                "/proc/bus",
                "/proc/fs",
                "/proc/irq",
                "/proc/sys",
                "/proc/sysrq-trigger"
            ]
        },
        "GraphDriver": {
            "Data": {
                "LowerDir": "/var/lib/docker/overlay2/f4767de9acdb4315ed8d017ff4aedc362cc3dc62b64b9201e640f96b9d2d6324-init/diff:/var/l
ib/docker/overlay2/06fb92a39e00ab668989827564a7902fbfae860640917d05796e2eccaf192d72/diff:/var/lib/docker/overlay2/b770b052c1df66b22f2bbc171e637e15b1cc3ff49b4ae3eca3d32f5ad1b778d3/diff",                "MergedDir": "/var/lib/docker/overlay2/f4767de9acdb4315ed8d017ff4aedc362cc3dc62b64b9201e640f96b9d2d6324/merged",
                "UpperDir": "/var/lib/docker/overlay2/f4767de9acdb4315ed8d017ff4aedc362cc3dc62b64b9201e640f96b9d2d6324/diff",
                "WorkDir": "/var/lib/docker/overlay2/f4767de9acdb4315ed8d017ff4aedc362cc3dc62b64b9201e640f96b9d2d6324/work"
            },
            "Name": "overlay2"
        },
        "Mounts": [],
        "Config": {
            "Hostname": "4c436db6d283",
            "Domainname": "",
            "User": "",
            "AttachStdin": false,
            "AttachStdout": true,
            "AttachStderr": true,
            "Tty": false,
            "OpenStdin": false,
            "StdinOnce": false,
            "Env": [
                "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
            ],
            "Cmd": [
                "/bin/httpd",
                "-f",
                "-h",
                "/data/html"
            ],
            "Image": "jason/httpd:v0.2",
            "Volumes": null,
            "WorkingDir": "",
            "Entrypoint": null,
            "OnBuild": null,
            "Labels": {}
        },
        "NetworkSettings": {
            "Bridge": "",
            "SandboxID": "ee2c356d9910e07a7d98b32a08d67e7876bcdbd129cf73562364d50e73247a18",
            "HairpinMode": false,
            "LinkLocalIPv6Address": "",
            "LinkLocalIPv6PrefixLen": 0,
            "Ports": {},
            "SandboxKey": "/var/run/docker/netns/ee2c356d9910",
            "SecondaryIPAddresses": null,
            "SecondaryIPv6Addresses": null,
            "EndpointID": "e91c73767eeb0a982dae99287f5be2536ac7f973dc10ffa889d2f2cef17412ac",
            "Gateway": "172.17.0.1",
            "GlobalIPv6Address": "",
            "GlobalIPv6PrefixLen": 0,
            "IPAddress": "172.17.0.5",
            "IPPrefixLen": 16,
            "IPv6Gateway": "",
            "MacAddress": "02:42:ac:11:00:05",
            "Networks": {
                "bridge": {
                    "IPAMConfig": null,
                    "Links": null,
                    "Aliases": null,
                    "NetworkID": "c2802fc2d2c3198bd6d5257cb83043aabd76e3ecaeea8daafe1b0bb065efb9d4",
                    "EndpointID": "e91c73767eeb0a982dae99287f5be2536ac7f973dc10ffa889d2f2cef17412ac",
                    "Gateway": "172.17.0.1",
                    "IPAddress": "172.17.0.5",
                    "IPPrefixLen": 16,
                    "IPv6Gateway": "",
                    "GlobalIPv6Address": "",
                    "GlobalIPv6PrefixLen": 0,
                    "MacAddress": "02:42:ac:11:00:05",
                    "DriverOpts": null
                }
            }
        }
    }
]
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# 
```

\[root@node101.yinzhengjie.org.cn ~\]# docker inspect t2　　　　　　　　　 #查看咱们启动的t2容器的元数据信息，可以和"busybox"镜像稍作对比，注意观察"Cmd"的属性哟~

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@node101.yinzhengjie.org.cn ~]# docker inspect t2 | grep IPAddress　　#查看t2容器的元数据信息并过滤IP地址用于访问httpd服务。
            "SecondaryIPAddresses": null,
            "IPAddress": "172.17.0.5",
                    "IPAddress": "172.17.0.5",
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# curl 172.17.0.5　　　　　　　　　　　　#是不是很神奇？我们启动t2容器后，就可以直接访问t2容器里面的服务啦！
<h1>Busybox httpd server.[Jason Yin dao ci yi you !!!]</h1>
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

**六.将自建镜像push到Docker Hub仓库中**

**1>.点击“Create Repository”(自行注册一下docker hub官网的id，大约不到2分钟就能搞定）**

**![](https://img2018.cnblogs.com/blog/795254/201910/795254-20191017155624812-197684082.png)**

**2>.填写创建仓库的相关信息后，并点击"Create"**

**![](https://img2018.cnblogs.com/blog/795254/201910/795254-20191017160146634-881294068.png)**

**3>.Docker Hub中的Repository就此创建成功啦**

**![](https://img2018.cnblogs.com/blog/795254/201910/795254-20191017160359286-2089213784.png)**

**4>.使用命令行登录docker Hub的官网（需要账号密码）**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@node101.yinzhengjie.org.cn ~]# docker login --help

Usage: docker login [OPTIONS] [SERVER]　　　　　　　　#注意，如果登录的是Docker Hub则不需要指定服务器地址。
Log in to a Docker registry.
If no server is specified, the default is defined by the daemon.

Options:
-p, --password string Password
--password-stdin Take the password from stdin
-u, --username string Username
[root@node101.yinzhengjie.org.cn ~]#
[root@node101.yinzhengjie.org.cn ~]# docker login -u yinzhengjie2019　　　　　　#输入用户名和密码
Password: 
WARNING! Your password will be stored unencrypted in /root/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded　　　　　　#如果出现这条信息说明登录成功啦！
[root@node101.yinzhengjie.org.cn ~]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

**5>.将自建镜像push到Docker Hub服务器上去(注意：镜像名称需要包含你docker hub用户名的字样)**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@node101.yinzhengjie.org.cn ~]# docker image ls　　　　　　#查看现有镜像
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
jason/httpd         v0.2                78fb6601880f        41 minutes ago      1.22MB
jason/httpd         v0.1-1              76d5e6c143b2        18 hours ago        1.22MB
busybox             latest              19485c79a9bb        6 weeks ago         1.22MB
redis               4-alpine            442c73e7ad1d        8 weeks ago         20.4MB
nginx               1.14-alpine         8a2fb25a19f5        6 months ago        16MB
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# docker push jason/httpd　　#我想把所有"jason/httpd"镜像推送出去，这里没有指明版本，因此我想把所有版本都推送上去。
The push refers to repository [docker.io/jason/httpd]
8191d8bde558: Preparing 
6c0ea40aef9d: Preparing 
denied: requested access to the resource is denied　　　　　　　　#哎呀，推送失败啦！原因是咱们的镜像名称和docker hub登录用户名称的前缀不一致导致！
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# docker tag jason/httpd:v0.1-1 yinzhengjie2019/httpd:v0.1-1　　　　#查明原因后，我们需要把现有的镜像从新更改名称
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# docker tag jason/httpd:v0.2 yinzhengjie2019/httpd:v0.2
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# docker image ls　　　　　　　　#经过一番操作后，已经把"jason/httpd"的所有tag更名成功啦。
REPOSITORY              TAG                 IMAGE ID            CREATED             SIZE
yinzhengjie2019/httpd   v0.2                78fb6601880f        46 minutes ago      1.22MB
jason/httpd             v0.2                78fb6601880f        46 minutes ago      1.22MB
jason/httpd             v0.1-1              76d5e6c143b2        18 hours ago        1.22MB
yinzhengjie2019/httpd   v0.1-1              76d5e6c143b2        18 hours ago        1.22MB
busybox                 latest              19485c79a9bb        6 weeks ago         1.22MB
redis                   4-alpine            442c73e7ad1d        8 weeks ago         20.4MB
nginx                   1.14-alpine         8a2fb25a19f5        6 months ago        16MB
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# docker push yinzhengjie2019/httpd　　　　　　#此时我们在用符合docker hub镜像名称推送镜像即可成功。
The push refers to repository [docker.io/yinzhengjie2019/httpd]
8191d8bde558: Pushed 
6c0ea40aef9d: Pushed 
v0.1-1: digest: sha256:c5fb206d8741582ce04f2914edee272dafed8fccde3055751375cf7885531a51 size: 734
8191d8bde558: Layer already exists 
6c0ea40aef9d: Layer already exists 
v0.2: digest: sha256:d8933526601ca1e4d078fad21cbace2eacd24f774bc0ad46d4119d13a457e086 size: 734
[root@node101.yinzhengjie.org.cn ~]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

**6>.镜像推送成功后，我们可以在Docker Hub的Repositories中查看(如下图所示)**

**![](https://img2018.cnblogs.com/blog/795254/201910/795254-20191017162429002-336337591.png)**

**七.将自建镜像推送到阿里云上的仓库**

**1>.点击"创建镜像仓库"(首先登录阿里云，并进入管理中心，链接地址为:"https://cr.console.aliyun.com"）**

![](https://img2018.cnblogs.com/blog/795254/201910/795254-20191017163637858-907251751.png)

**2>.填写对应的仓库信息**

![](https://img2018.cnblogs.com/blog/795254/201910/795254-20191017163932428-1911613671.png)

**3>.选择"本地仓库"**

![](https://img2018.cnblogs.com/blog/795254/201910/795254-20191017164039602-412927653.png)

**4>.镜像仓库创建成功**

![](https://img2018.cnblogs.com/blog/795254/201910/795254-20191017164129415-2135740821.png)

**5>.点击上图已经创建好的httpd仓库的"管理"按钮，可以看到该仓库的使用方式**

**![](https://img2018.cnblogs.com/blog/795254/201910/795254-20191017164738515-2002094061.png)**

**6>.为仓库设置固定登录密码**

**![](https://img2018.cnblogs.com/blog/795254/201910/795254-20191017165037505-2135693286.png)**

**7>.使用刚刚修改的固定密码在命令行中登录**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@node101.yinzhengjie.org.cn ~]# docker logout 　　　　　　#从Docker Hub服务器中退出登录
Removing login credentials for https://index.docker.io/v1/
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# docker login --username=153****5200 registry.cn-beijing.aliyuncs.com　　　　#根据案例官网的提示信息进行登录
Password: 
WARNING! Your password will be stored unencrypted in /root/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded　　　　　　　　#很显然，咱们登录成功啦!
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

**8>.推送本地镜像到阿里云（需要按照阿里云的要求修改对应的标签）**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@node101.yinzhengjie.org.cn ~]# docker image ls
REPOSITORY              TAG                 IMAGE ID            CREATED             SIZE
jason/httpd             v0.2                78fb6601880f        About an hour ago   1.22MB
yinzhengjie2019/httpd   v0.2                78fb6601880f        About an hour ago   1.22MB
yinzhengjie2019/httpd   v0.1-1              76d5e6c143b2        18 hours ago        1.22MB
jason/httpd             v0.1-1              76d5e6c143b2        18 hours ago        1.22MB
busybox                 latest              19485c79a9bb        6 weeks ago         1.22MB
redis                   4-alpine            442c73e7ad1d        8 weeks ago         20.4MB
nginx                   1.14-alpine         8a2fb25a19f5        6 months ago        16MB
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# docker tag jason/httpd:v0.2 registry.cn-beijing.aliyuncs.com/yinzhengjie2019/httpd:v0.2　　　　#根据案例官网的提示修改符合的TAG方能推送成功！
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# docker image ls
REPOSITORY                                               TAG                 IMAGE ID            CREATED             SIZE
jason/httpd                                              v0.2                78fb6601880f        About an hour ago   1.22MB
yinzhengjie2019/httpd                                    v0.2                78fb6601880f        About an hour ago   1.22MB
registry.cn-beijing.aliyuncs.com/yinzhengjie2019/httpd   v0.2                78fb6601880f        About an hour ago   1.22MB
jason/httpd                                              v0.1-1              76d5e6c143b2        18 hours ago        1.22MB
yinzhengjie2019/httpd                                    v0.1-1              76d5e6c143b2        18 hours ago        1.22MB
busybox                                                  latest              19485c79a9bb        6 weeks ago         1.22MB
redis                                                    4-alpine            442c73e7ad1d        8 weeks ago         20.4MB
nginx                                                    1.14-alpine         8a2fb25a19f5        6 months ago        16MB
[root@node101.yinzhengjie.org.cn ~]#
[root@node101.yinzhengjie.org.cn ~]# docker push registry.cn-beijing.aliyuncs.com/yinzhengjie2019/httpd　　　　　　#很显然，咱们自定义镜像推送成功啦!
The push refers to repository [registry.cn-beijing.aliyuncs.com/yinzhengjie2019/httpd]
8191d8bde558: Pushed 
6c0ea40aef9d: Pushed 
v0.2: digest: sha256:d8933526601ca1e4d078fad21cbace2eacd24f774bc0ad46d4119d13a457e086 size: 734
[root@node101.yinzhengjie.org.cn ~]#
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

**9>.在阿里云的web界面查看对应的镜像是否上传成功啦**

**![](https://img2018.cnblogs.com/blog/795254/201910/795254-20191017165648735-1305166096.png)**

**八.镜像的导入和导出**

**1>.在一个节点中导出镜像**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@node101.yinzhengjie.org.cn ~]# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
jason/httpd         v0.2                78fb6601880f        11 hours ago        1.22MB
jason/httpd         v0.1-1              76d5e6c143b2        28 hours ago        1.22MB
busybox             latest              19485c79a9bb        6 weeks ago         1.22MB
redis               4-alpine            442c73e7ad1d        8 weeks ago         20.4MB
nginx               1.14-alpine         8a2fb25a19f5        6 months ago        16MB
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# docker save -o myimages.gz jason/httpd:v0.1-1 jason/httpd:v0.2　　　　#我们将2个镜像打包到一个文件中
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# ll
total 1920
-rw-------. 1 root root 1461760 Oct 18 17:01 myimages.gz
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

**2>.将打包好的镜像压缩包发到另一个节点中**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@node101.yinzhengjie.org.cn ~]# scp myimages.gz  node102.yinzhengjie.org.cn:~
The authenticity of host 'node102.yinzhengjie.org.cn (172.30.1.102)' can't be established.
ECDSA key fingerprint is SHA256:ARXihOo9IJ8fnmJT91072JnBkrKxiTnsLYCyKaoxu4c.
ECDSA key fingerprint is MD5:7a:1c:f3:c1:16:e2:13:e1:da:a0:48:59:83:df:31:f9.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added 'node102.yinzhengjie.org.cn,172.30.1.102' (ECDSA) to the list of known hosts.
root@node102.yinzhengjie.org.cn's password: 
myimages.gz                                                                   100% 1428KB 147.7MB/s   00:00    
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

**3>.在另一个节点导入镜像**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@node102.yinzhengjie.org.cn ~]# systemctl start docker　　　　　　#需要先启动docker服务才能使用它的管理服务
[root@node102.yinzhengjie.org.cn ~]# 
[root@node102.yinzhengjie.org.cn ~]# docker image ls　　　　　　　　　　#很显然，该节点没有镜像文件
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
[root@node102.yinzhengjie.org.cn ~]# 
[root@node102.yinzhengjie.org.cn ~]# ll
total 1428
-rw-------. 1 root root 1461760 Oct 18 17:29 myimages.gz
[root@node102.yinzhengjie.org.cn ~]# 
[root@node102.yinzhengjie.org.cn ~]# 
[root@node102.yinzhengjie.org.cn ~]# docker load -i myimages.gz 　　　#我们使用docker命令将node101.yinzhengjie.org.cn节点的包加载一下
Loaded image: jason/httpd:v0.1-1
Loaded image: jason/httpd:v0.2
[root@node102.yinzhengjie.org.cn ~]# 
[root@node102.yinzhengjie.org.cn ~]# docker image ls　　　　　　　　　　#我们发现镜像直接被加载成功了，无需去仓库下载咱们就可以使用啦~
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
jason/httpd         v0.2                78fb6601880f        12 hours ago        1.22MB
jason/httpd         v0.1-1              76d5e6c143b2        29 hours ago        1.22MB
[root@node102.yinzhengjie.org.cn ~]# 
[root@node102.yinzhengjie.org.cn ~]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

当你的才华还撑不起你的野心的时候，你就应该静下心来学习。当你的能力还驾驭不了你的目标的时候，你就应该沉下心来历练。问问自己，想要怎样的人生。 欢迎加入基础架构自动化运维：598432640，大数据SRE进阶之路：959042252，DevOps进阶之路：526991186