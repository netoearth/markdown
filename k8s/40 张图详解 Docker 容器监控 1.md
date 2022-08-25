#### 在公众号后台回复：**JGNB**，可获取杰哥原创的 PDF 手册。

### 前言

在企业中，通常业务是不允许随意停止的，否则将给企业带来巨大的经济损失。

运维工程师要保证业务正常运行，就必须利用工具时刻监控业务的运行状态，容器中的业务也不例外。

除了容器自身的监控命令外，还有一些针对容器的动态特征而开发的第三方监控工具。

本章将对容器监控及其相关内容进行详解。

### Docker 监控命令

在容器中，通常可以通过执行命令或利用第三方工具，获取当前容器中的数据并将数据呈现给用户。

安装完成的 Docker 自带一些用于监控容器的子命令，这是 Docker 开发者为用户提供的容器监控方式。

#### docker ps 命令

docker ps 命令是之前中讲过的命令，用来查看容器状态，示例代码如下：

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

另外，通过 docker container ls命令也可以达到相同的效果，示例代码如下：

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

注意，若是 docker container ls命令执行失败，更新 Docker 版本即可。

#### docker top 命令

docker top 命令用于查看容器中的进程，示例代码如下：

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

以上示例通过 docker top 命令添加容器 ID 号查看到了容器内进程。

除此之外，还可以在命令中添加容器名称，达到相同的效果，示例代码如下：

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

在 docker top 命令中添加参数即可显示特定的进程信息，此处以 -u 参数为例，示例代码如下：

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

以上示例通过给 docker top 命令添加 -u 参数，将 sysdig 容器的进程信息以用户为主的格式显示出来。

#### docker stats 命令

docker stats 命令用于查询容器的各项资源的消耗情况，示例代码如下：

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

以上示例执行了 docker stats 命令，在终端通过一个动态列表显示出各个容器的资源使用情况，如 cpu 使用率、内存、容器网络等信息。

在没有限制容器内存的情况下，此处将会显示宿主机的内存。

此处的动态列表有一项明显的不足，就是只能显示容器 ID 号，不显示容器名称。

但只要在命令中添加容器名称，即可查看指定容器的信息，示例代码如下：

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

Docker 自带的容器监控命令能够灵活捕捉容器的实时信息，且使用方便。

但它们无法反映容器资源占用的趋势，且只能显示有限的数据。

### Sysdig

Sysdig 是一款命令行监控工具，因其轻量级的特点深受广大用户的喜爱。

Sysdig 就像放大镜，使用户可以更清晰地看到宿主机与容器的各项行为。

它相当于多种 Linux 监控工具的合集，如 strace、htop、lsof 等，将这些工具的功能及查询结果整合到同一个界面中，供用户操作。

Sysdig 为在 Docker Hub 中提供了容器镜像，用户可以将 Sysdig 以容器的形式运行，示例代码如下：

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

以上示例中，Sysdig 容器以挂载宿主机目录的方式收集系统信息，并给予其足够的系统权限。

注意，该命令中必须使用绝对路径，否则会在执行时出错。

容器启动后将直接进入容器终端，若通过 Ctrl+P+Q 组合键退出容器或者容器在后台运行，通过 exec 命令即可进入 Sysdig 容器，示例代码如下：

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

在 Sysdig 容器中，通过以下命令启动 Sysdig 监控：

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

执行成功之后，将显示 Sysdig 功能界面，如图所示。

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

功能界面中不仅有各项资源的使用信息，下方还有各类选项，用户可以根据不同要求从不同角度去监控不同类型的资源。

按 F2 键或者单击 Views 选项，进入监控选项列表，如图所示。

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

在该界面中，左边列出了 Sysdig 的各个监控项，右边是关于监控项的说明。

通过键盘方向键即可移动界面中的光标，从而切换监控项。

下面将光标移动到 Containers 项，按回车键或者双击该选项，进入容器监控界面，如图所示。

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

若是用户觉得图中的内容太过繁琐，或者难以理解，可以按 F7 键，进入数据说明界面。

其中有对各项数据的解释，能帮助用户更快掌握 Sysdig 的使用方法，如图所示。

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

进入该界面之后，按任意键即可退出。

另外，在监控界面中还可以指定按照某一项数据进行排序，单击列表中某一项数据的表头即可。

此处以内存为例，按照占用内存排序的监控列表，如图所示。

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

若要查看单个容器内部信息，将光标移动到该容器信息上，按回车键即可，如图所示。

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

再移动光标到指定信息，按回车键还可以查看容器进程中的线程信息，如图所示。

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

若要返回上一级，在键盘上按退格键即可。

为方便用户管理，Sysdig 还支持搜索功能，通过 Ctrl+F 组合键即可启动该功能，再输入关键字即可查询。此处关键字以 usr 为例，如图所示。

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

如果在操作过程中遇到问题，可以按 F1 键或者单击某选项，进入帮助文档。

帮助文档详细介绍了 Sysdig 的操作方式，供用户学习。

若动态列表变化太快，导致用户无法准确查看到信息，可以按 P 键将列表暂停。

Sysdig 为用户提供了较为全面的监控视角，但其本质是命令行工具，缺乏更具直观性的监控角度。

### Weave Scope

Weave Scope 为用户提供了更直观的监控视角，它将整个监控以图形界面的形式呈现出来。

#### 安装 Weave Scope

首先，下载 Weave Scope 的二进制安装包到指定的路径下，示例代码如下：

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

Weave Scope 安装包的本质是一个脚本，所以需要赋予其执行权限，示例代码如下：

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

然后，通过命令执行该脚本，示例代码如下：

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

此时，Weave Scope 监控已经开启，通过浏览器访问系统提示中的 http://192.168.77.128:4040/ 即可进入监控界面。

在进入界面之前先查看容器状态，示例代码如下：

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

从以上示例中可以看到，宿主机中增加了一个被命名为 weavescope 的新容器，这说明 Weave Scope 以容器的方式在宿主机中运行。

下面根据提示进入 Weave Scope 界面，如图所示。

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

图中，宿主机中的所有容器都以图形的形式呈现出来，更加便于用户管理。

#### 监控容器

在 Weave Scope 界面中，宿主机上的容器被分为多个种类，默认不显示 Weave Scope 本身的容器。

若要查看所有容器，就需要在界面左下角的选项中进行操作，如图所示。

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

在左下方单击 All 选项，即可查看宿主机中所有运行中的容器，如图所示。

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

若要查看容器的资源占用情况，需要在界面上方选项中进行操作，如图所示。

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

在上方单击 CPU 选项，即可显示界面中容器的 CPU 使用情况，如图所示。

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

单击 CPU 选项之后，CPU 使用情况将会以液位高度的形式在容器图标上显示。

此时，将鼠标指针移动到容器图标之上，即可显示具体数据，如图所示。

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

若要查看某一容器的详细信息，单击该容器图标即可，如图所示。

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

其中，容器的详细信息包括以下各项：

-   Status：CPU 与内存的实时状态曲线图。
    
-   Info：镜像、镜像标签、命令等信息。
    
-   Processes：该容器中实时运行的进程信息。
    
-   Docker labels：维护人员或容器的启动命令等信息。
    
-   Image：该容器的镜像信息。
    

容器详细信息界面中，有一行可对该容器直接进行操作的选项，如图所示。

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

图中的选项从左到右分别表示：

-   通过 docker attach 命令进入容器终端；
    
-   通过 docker exec 命令进入容器终端；
    
-   通过 docker restart 命令重新启动容器；
    
-   通过 docker pause 命令暂停容器；
    
-   通过 docker stop 命令终止容器。
    

有了这些选项，用户就不需要在终端中输入命令，直接单击选项即可对容器进行操作。

若需要执行这些选项之外的操作，可通过选项进入容器终端完成。

#### 监控宿主机

Weave Scope 为用户提供广阔的监控视角，除了监控容器，还可以对宿主机进行监控。

单击界面上方的 Hosts 选项，即可查看宿主机，如图所示。

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

与容器操作相同，单击宿主机图标即可查看其详细信息，如图所示。

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

与容器相比，宿主机的 Staus 项中增加了负载信息。

详细信息还包含了宿主机中的容器信息，单击容器名称即可查看容器的详细信息。

宿主机的信息中只有一个供用户对其进行操作的选项，单击即可进入宿主机终端，如图所示。

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

#### 多宿主机监控

在企业生产环境中，通常需要使用多台宿主机部署容器业务，所以容器监控也需要同时监控多台宿主机。

而 Weave Scope 恰好拥有多宿主机监控的功能，下面通过示例来演示该功能的使用方式。

首先，准备两台安装了 Weave Scope 的服务器，并分别在启动命令中添加两个服务器的 IP 地址进行启动，示例代码如下：

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

根据两台宿主机中启动命令的执行结果，无论是访问 http://192.168.77.128:4040/ 还是 http://192.168.77.130:4040/，都可以监控到两台宿主机，如图所示。

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

单击界面上方的 Containers 选项，查看所有宿主机中的容器，如图所示。

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

为了便于用户分辨，Weave Scope 在每个容器图标下的容器名称后都会标注该容器所属宿主机的主机名。

在生产环境中部署大量容器，需要对某一容器进行操作时，可以使用 Weave Scope 界面左上角的搜索功能，对该容器进行搜索。

此处以关键字 "reg" 为例，搜索结果如图所示。

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

另外，Weave Scope 还支持逻辑条件搜索。

例如，搜索 CPU 占用率大于 1% 的容器，在搜索栏中输入 "cpu>1 即可，如图所示。

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

界面的右下角有四个选项，从左到右前三个是调试界面显示的选项，最后一个是 Weave Scope 的帮助选项，如图所示。

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

单击帮助选项，即可查看 Weave Scope 的帮助文档，如图所示。

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

### 总结

本章讲解了 Docker 自带的监控命令，以及一些第三方监控软件的安装与使用。

其中，Sysdig 是一款优秀的命令行监控工具；

Weave Scope 不仅操作简单，还为用户提供了更为直观的图形界面。

希望大家通过本篇文章的学习能够熟练掌握 Docker 容器的监控方式，以保证容器中业务的正常运行。

作者 | 飞向星的客机

来源 | CSDN博客

### 推荐阅读

[Docker 镜像详细讲解](http://mp.weixin.qq.com/s?__biz=MzAwMjg1NjY3Nw==&mid=2247529576&idx=1&sn=b79f6ac0ac7e80f346061015e6276136&chksm=9ac637e2adb1bef4e4106736a3861411cc85a97b2102d229abdd9d354b956654bc6f9f8aca11&scene=21#wechat_redirect)

[超值一篇分享，Docker：从入门到实战过程全记录](http://mp.weixin.qq.com/s?__biz=MzAwMjg1NjY3Nw==&mid=2247527367&idx=1&sn=3d82aa094fd833ec8cd6494c5d1187e4&chksm=9ac6284dadb1a15b599b35383d1177f7f3b82dfaf747126e57c499c4c9eaa1314cd204559145&scene=21#wechat_redirect)  

[深入理解 Docker 网络原理](http://mp.weixin.qq.com/s?__biz=MzAwMjg1NjY3Nw==&mid=2247526308&idx=1&sn=47570e62b36bc1d6c2ed57bac6d4e8ea&chksm=9ac6242eadb1ad384940bbee3951aaf5172adeb7e3c5188b582996912f0704425652ff071503&scene=21#wechat_redirect)  

[能从入门到精通的 Docker 学习指南](http://mp.weixin.qq.com/s?__biz=MzAwMjg1NjY3Nw==&mid=2247524569&idx=1&sn=93610b13485f0d02d1d98fb51b68c32f&chksm=9ac62353adb1aa45b4b603400638374fc161c1e469ce1140d457ae358d80f92109b5c1a35019&scene=21#wechat_redirect)  

[史上讲解最好的 Docker 教程，从入门到精通（建议收藏的教程）](http://mp.weixin.qq.com/s?__biz=MzAwMjg1NjY3Nw==&mid=2247517692&idx=1&sn=7f65fed24267116e9a314d5a420cb808&chksm=9ac6c676adb14f6003e62b7c70327342798763d7fc417e6b1728ea787c5b6d0624b25e9959e3&scene=21#wechat_redirect)  

[Docker 极简入门指南，10 分钟就能看懂~](http://mp.weixin.qq.com/s?__biz=MzAwMjg1NjY3Nw==&mid=2247514906&idx=1&sn=9d0dcfced19cd6edbc3028a75d343637&chksm=9ac6f890adb1718604610465822801a46c5df43997ff99b9b0026fd66d055aca105b3e74c75e&scene=21#wechat_redirect)

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)