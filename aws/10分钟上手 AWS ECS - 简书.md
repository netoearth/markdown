## 10分钟上手 AWS ECS

[![](https://cdn2.jianshu.io/assets/default_avatar/15-a7ac401939dd4df837e3bbf82abaa2a8.jpg)](https://www.jianshu.com/u/99f3318bcc01)

[刘思宁](https://www.jianshu.com/u/99f3318bcc01)IP属地: 山东

0.2282017.12.21 16:49:53字数 1,448阅读 8,952

你在用 docker 么？你是否感觉在国内更新 docker 经常失败？下载 docker image 速度缓慢？

如果你想解决这个问题，可以尝试使用 AWS 提供的 Elastic Container Service（简称 ECS）把你的 docker 容器放在云上。

## 把 docker 镜像交给 AWS —— 创建 docker image 仓库

通常，如果你想要在服务器上构建一个镜像，你可能会通过 ssh 登录服务器后，执行 `docker build` 命令，或者从 docker hub 上下载镜像。如果使用 ECS 服务，你可以直接把镜像交给 AWS 设置的 image 仓库。

首先，我们在 ECS 服务中创建一个仓库。和 docker hub 或者 github 仓库一样，我们在网页上创建一个空仓库，然后把我们的资源 push 到仓库的地址。

在进入 Elastic Container Service 的控制台后，点击「存储库」选项卡，并点击「创建存储库」。

在下一页，给你的存储库起个名字。比如我起的名字叫 `example-image`：

![](https://upload-images.jianshu.io/upload_images/1269937-8ec1881b4aa0df4f.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

然后点击下一步，会有一个详细的上传引导，按照这个引导一步一步做，当你本地的终端里显示镜像 push 完成后，点击浏览器里的完成。你就能看到你上传的镜像了。

![](https://upload-images.jianshu.io/upload_images/1269937-09ec04b94eefa3a8.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

如果你不知道用什么镜像，可以看我写的[这个 Dockerfile，一个基于 nginx 的很简单的镜像](https://link.jianshu.com/?t=https%3A%2F%2Fgithub.com%2Fliusining%2Faws-ecs-example)。

## 找个地方承载你的服务 —— 创建集群

在上一步，我们已经准备好了要使用的镜像，但是还没有服务器可以让我们来启动这个镜像。为了弄一台能运行 docker 的服务器，我们需要在 ECS 控制台中创建一个集群（尽管这个集群只有 1 台服务器）。

那么，点击「集群」选项卡，并点击「创建集群」：

![](https://upload-images.jianshu.io/upload_images/1269937-a491b5a93de872cd.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

之后会有很多设置项需要填写，我们在这里只填写最少必要的设置项，也就是第一屏的设置项。如图，我把集群名称设置为`example-cluster`。

![](https://upload-images.jianshu.io/upload_images/1269937-a9c411a058b2aa14.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

之后，滑到页面地图，点击创建。然后等待集群创建好，你就有了一个可以运行 docker 的 ECS 集群。

## 定义如何运行你的容器 —— 创建任务定义

有了 ECS 集群后，你就可以把容器放到上面运行了。但是在运行之前，你需要先定义一下，你要如何运行这些容器。

那么，我们就进入「任务定义」 选项卡，并点击「创建新任务定义」：

![](https://upload-images.jianshu.io/upload_images/1269937-8378d10fc6bdf9d6.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

之后又是一些设置项，但对我们来说，重要的只有两个，一个是起名，一个是添加容器。我把这个任务起名为`example-task`，然后点击添加容器。

在添加容器的时候，最基本的是前 5 项，更详细的说明我写在图中。后面还有很多选项，其中大部分都可以看做是从 docker 命令演化而来的，在本教程中不详细介绍。填写之后，点击右下角的创建：

![](https://upload-images.jianshu.io/upload_images/1269937-d4a92bb467092749.jpg)

这样，我们就在任务中添加了一个容器作为任务的一部分。接着点击页面下方的「创建」，完成整个任务的创建。

## 启动容器和配套设施 —— 创建服务

这是运行容器前的最后一步，也就是把刚才定义的任务运行起来。在 AWS 的词典里，这叫创建服务<sup><a href="https://www.jianshu.com/p/d9a3093bd151#fn1" id="fnref1">[1]</a></sup>。

要想创建服务，我们需要进入刚刚创建的集群：

![](https://upload-images.jianshu.io/upload_images/1269937-7342922a1533754c.jpg)

在「服务」选项卡（1号位置）下，点击「创建」。之后会设置任务的参数，只需要填选第一屏的内容就好了。

![](https://upload-images.jianshu.io/upload_images/1269937-b77c8de06945c494.jpg)

之后所有的设置项，我们都留作默认的。所以一路点击下一步，并创建服务，直到页面显示「服务已创建」。这个时候，我们可以去查看服务状态：

![](https://upload-images.jianshu.io/upload_images/1269937-6f8b43114e4f3011.jpg)

如果在服务的控制面板中，显示刚刚启动的任务为「RUNNING」状态，代表着我们成功的启动了在「创建任务定义」阶段设置的容器。

为了查看实际效果，我们回到集群的控制面板，去查看 ECS 服务器的地址（公有 DNS 或者 公有 IP）：

![](https://upload-images.jianshu.io/upload_images/1269937-d1e4aaa0b7a0bd5a.jpg)

我得到的实际效果是这样的：

![](https://upload-images.jianshu.io/upload_images/1269937-1e24816fe5a7058c.jpg)

## 后话

恭喜，到这里你可能已经在使用自己的第一个 ECS 了。但是后续的更改要如何进行呢？

如果你想使用更多的服务器，就在集群控制面板中做「扩展 ECS 实例」的操作。  
如果你想想更新容器运行的代码，就把用新打包的镜像推到仓库中，然后对任务「创建新修订」，然后更新服务。

![](https://upload-images.jianshu.io/upload_images/1269937-39e68bdd01617f12.jpg)

image

最后，我要道歉的是，我对 ECS 服务也并不是很熟悉，无法达到生产环境要求的稳定性，可用性。如果有疑问，或者质疑，欢迎留言。

## 参考

[Running Docker In Production Using AWS ECS](https://link.jianshu.com/?t=https%3A%2F%2Fyoutu.be%2Fzp7gUCgyS34)

___

1.  或者说，运行任务是创建服务的一部分，因为创建服务的过程中，还可以配置 auto scaling，负载均衡等超越运行任务的部分。 [↩](https://www.jianshu.com/p/d9a3093bd151#fnref1)
    

更多精彩内容，就在简书APP

"打赏2元，我将拼命干活"

还没有人赞赏，支持一下

[![  ](https://cdn2.jianshu.io/assets/default_avatar/15-a7ac401939dd4df837e3bbf82abaa2a8.jpg)](https://www.jianshu.com/u/99f3318bcc01)

[刘思宁](https://www.jianshu.com/u/99f3318bcc01 "刘思宁")这世界和我们想象的不一样<br><br>微信公众号：刘思宁 <br>微博：刘思宁io <br>...

总资产70共写了13.1W字获得19,881个赞共4,266个粉丝

### 被以下专题收入，发现更多相似内容

### 推荐阅读[更多精彩内容](https://www.jianshu.com/)

-   Docker — 云时代的程序分发方式 要说最近一年云计算业界有什么大事件？Google Compute Engi...
    
-   转载自 http://blog.opskumu.com/docker.html 一、Docker 简介 Docke...
    
    [![](https://upload.jianshu.io/users/upload_avatars/2795885/067251d5-3831-4949-a47b-83fdd4698f3d.png?imageMogr2/auto-orient/strip|imageView2/1/w/48/h/48/format/webp)极客圈](https://www.jianshu.com/u/bf4b9a52ab6d)阅读 10,148评论 0赞 120
    
    [![](https://upload-images.jianshu.io/upload_images/2795885-48df501d45ad57e9.png?imageMogr2/auto-orient/strip|imageView2/1/w/300/h/240/format/webp)](https://www.jianshu.com/p/1c5fef69897f)
-   写这篇文章主要是为了今后毕业论文素材上的整理，同时对docker进行巩固温习。大纲: docker简介docker...
    
    [![](https://upload.jianshu.io/users/upload_avatars/764980/0e1d68d52315.jpg?imageMogr2/auto-orient/strip|imageView2/1/w/48/h/48/format/webp)胡图仙人](https://www.jianshu.com/u/e24c217f739c)阅读 6,858评论 2赞 96
    
    [![](https://upload-images.jianshu.io/upload_images/764980-9d00087087057bb6.png?imageMogr2/auto-orient/strip|imageView2/1/w/300/h/240/format/webp)](https://www.jianshu.com/p/7e4796cd35a0)
-   七律/苏铁 作者：心博、图片：网络 铁树能开实慢然，峥嵘岁月往如烟。 朝来夜去八千度，斗转星移二十年。 羽翼叶姿娇...
    
    [![](https://upload.jianshu.io/users/upload_avatars/8882454/dbed3952-ef0a-4839-b9bb-99bdb9bfbf03.jpg?imageMogr2/auto-orient/strip|imageView2/1/w/48/h/48/format/webp)心博1](https://www.jianshu.com/u/79a3fbed207d)阅读 512评论 1赞 4
    
    [![](https://upload-images.jianshu.io/upload_images/8882454-550ad136f106962c.jpg?imageMogr2/auto-orient/strip|imageView2/1/w/300/h/240/format/webp)](https://www.jianshu.com/p/53fc022501b1)
-   过年回家待了一段时间，正好是深冬，吃上热腾腾的饭菜，就是每天眼巴巴儿的最期盼的事。 我老家在老河口市，属于湖北省襄...
    
    [![](https://upload.jianshu.io/users/upload_avatars/3388066/d39da48f6023?imageMogr2/auto-orient/strip|imageView2/1/w/48/h/48/format/webp)余小头](https://www.jianshu.com/u/41adf4fb9581)阅读 529评论 0赞 1
    
    [![](https://upload-images.jianshu.io/upload_images/3388066-9cc92fbc00964e03.jpg?imageMogr2/auto-orient/strip|imageView2/1/w/300/h/240/format/webp)](https://www.jianshu.com/p/1d16e9afbb21)