## Nexus配置Docker镜像源代理

[![](https://cdn2.jianshu.io/assets/default_avatar/12-aeeea4bedf10f2a12c0d50d626951489.jpg)](https://www.jianshu.com/u/abe1470cd694)

0.5242020.01.10 14:26:24字数 553阅读 5,605

本文是以Nexus 3.18.1为例介绍如何配置docker代理（proxy）仓库。

**Nexus端配置**

1、首先，在nexus上创建一个proxy类型的docker 仓库。

![](https://upload-images.jianshu.io/upload_images/20828550-ece9b326bbb4b83b.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

创建proxy类型的docker仓库

2、入下图，在“Remote storage”填入**https://registry-1.docker.io  ，**“Docker Index”**选择 Use Docker Hub，其他项保持默认。**

![](https://upload-images.jianshu.io/upload_images/20828550-c381c0efbcb14207.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

proxy类型的docker仓库设置-1

3、点击页面底部的“Create repository”菜单创建这个repository。

![](https://upload-images.jianshu.io/upload_images/20828550-21297cb8b83c0dab.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

proxy类型的docker仓库设置-2

4、创建一个docker仓库组，将上一步我们创建的docker proxy加入到组里。

**请注意3个红框处**，需要注意的是**HTTP**这里，如果你是用容器方式启动的nexus，那么你需要额外映射一个端口出来（在启动nexus容器的时候），比如我的启动nexus容器的命令如下，使用-p命令将容器里的端口映射出来。

docker run -d -p 172.23.2.200:8095:8081 -p 8097:8097 **\-p 28443:28443**  --privileged=true -v /mirror-folder-nexus318/nexus318/data:/nexus-data --name nexus318 sonatype/nexus3:latest

![](https://upload-images.jianshu.io/upload_images/20828550-640e929cf7d02c8f.png)

docker仓库组设置-1

5、把刚才创建的docker-proxy（忽略图中的docker-test，只是一个示例）加入到组里来，点击save即可。

![](https://upload-images.jianshu.io/upload_images/20828550-e17e141c2ae81464.png)

docker仓库组设置-2

至此，nexus端配置完成，下面我们对客户端（拉取镜像端）进行配置。

___

___

## 客户端配置

这里我们的客户端是CentOS 7，具体配置步骤如下：

1、首先在centos7上安装了docker环境，即能够在命令行里正常的敲docker命令并有回显。

2、在客户端机器上配置vim /etc/docker/daemon.json文件，如果系统上没有该文件则创建之，该文件内容如下：

{

"registry-mirrors":\["http://_**172.23.2.200**_:28443"\],

"insecure-registries":\["172.23.2.200:28443"\]

}

其中，斜体加粗的IP为nexus代理的地址，28443位你在docker仓库组中设置的HTTP端口。

![](https://upload-images.jianshu.io/upload_images/20828550-04a554017e440ba6.png)

配置daemon.json

3、重启客户端上的docker服务，如下：

    systemctl daemon-reload

    systemctl restart docker

4、之后，就可以通过如下命令拉取（pull）docker镜像了。

    命令格式为 ：  docker pull postgres:latest    ，postgres:latest为镜像名和tag.

![](https://upload-images.jianshu.io/upload_images/20828550-f597bc37ce25e40b.png)

拉取镜像成功

所有配置完。

更多精彩内容，就在简书APP

![](https://upload.jianshu.io/images/js-qrc.png)

"小礼物走一走，来简书关注我"

[![  ](https://cdn2.jianshu.io/assets/default_avatar/5-33d2da32c552b8be9a0548c7a4576607.jpg)](https://www.jianshu.com/u/d2cbf65179fe)共1人赞赏

[![  ](https://cdn2.jianshu.io/assets/default_avatar/12-aeeea4bedf10f2a12c0d50d626951489.jpg)](https://www.jianshu.com/u/abe1470cd694)

[Lzy\_e50e](https://www.jianshu.com/u/abe1470cd694 "Lzy_e50e")云计算从业者，喜欢关注和研究云计算行业相关的技术，并乐于分享工作实践中的所感所得，与大家一起共...

总资产0.36共写了553字获得4个赞共1个粉丝

-   和清纯老婆闪婚的20天 我撞伤了一个老伯，不但没担责任，他还要把女儿嫁给我。 没想到婚礼当天，曾被我捉奸在床的前女...
    
    [![](https://upload.jianshu.io/users/upload_avatars/4790772/388e473c-fe2f-40e0-9301-e357ae8f1b41.jpeg)茶点故事](https://www.jianshu.com/u/0f438ff0a55f)阅读 17944评论 2赞 13
    
-   放贷风波疫情爆发，我和同学的妻子被封控在一起美艳人妻却不知，我对她早已仰慕已久… 早几年的时候我跟发小刘权开了个小...
    

-   林欣蜷缩在监狱围墙里的一角，双眼无神地盯着另一边围墙上的铁丝网，几只小鸟正在上面盘旋着。 她在里面已经快呆了一个月...
    
    [![](https://upload.jianshu.io/users/upload_avatars/27310729/6489c52e-a21e-4692-8e9a-a374936edd84.jpg)以左a](https://www.jianshu.com/u/739f064dcce0)阅读 1217评论 0赞 7
    
-   我是他体贴入微，乖巧懂事的金丝雀。 上位后，当着满座宾朋，我播放了他曾经囚禁、霸凌我，还逼疯了我妈妈视频。 1 我...
    
    [![](https://upload.jianshu.io/users/upload_avatars/4790772/388e473c-fe2f-40e0-9301-e357ae8f1b41.jpeg)茶点故事](https://www.jianshu.com/u/0f438ff0a55f)阅读 2905评论 0赞 1
    
-   我曾亲眼目睹近百具死婴尸体,都被挖了眼睛和内脏，悬挂在铁丝上风干，最后磨成粉...... 我叫曹文静，今年25岁，...
    
-   寡妇与驴 我曾处理过一桩骇人听闻的案件，一个寡妇跟一头驴死在了她家的小院里。 死前这一人一驴都服用过某种药物，而且...
    
-   我的男朋友很帅很帅，温柔体贴，但是他和我闺蜜上了床。 还在陵园给我立了一个碑。 恐怖的是，我男朋友自己的墓碑就在我...
    
    [![](https://upload.jianshu.io/users/upload_avatars/4790772/388e473c-fe2f-40e0-9301-e357ae8f1b41.jpeg)茶点故事](https://www.jianshu.com/u/0f438ff0a55f)阅读 2475评论 0赞 3
    
-   被退婚后，我嫁给了当朝新帝。 本只想在后宫低调度日，却不料新帝是个恋爱脑。 放着家世优越的皇后和貌美如花的贵妃不要...
    
-   1、美女医生进监狱 疫情刚开始的那年，我和护士小白，被紧急调到西城监狱支援防控。 这里是男子监狱，那些囚犯很多都是...
    
    [![](https://upload.jianshu.io/users/upload_avatars/4790772/388e473c-fe2f-40e0-9301-e357ae8f1b41.jpeg)茶点故事](https://www.jianshu.com/u/0f438ff0a55f)阅读 2307评论 0赞 1
    
-   我在学生会的时候，拉赞助，被合作方领导骚扰。 这事还被人撞见，传了出去。 而我妈听见了这些闲话，气势汹汹地杀来。 ...
    
    [![](https://upload.jianshu.io/users/upload_avatars/4790772/388e473c-fe2f-40e0-9301-e357ae8f1b41.jpeg)茶点故事](https://www.jianshu.com/u/0f438ff0a55f)阅读 2866评论 0赞 1
    
-   我吃了整整三年的避子药，因为怀过皇嗣的妃嫔都死了，无一例外。三年前帝后大婚，洞房花烛的第二天早上我就毅然决然饮下了...
    
-   我跟我前夫在商场里面给小孩买衣服，他现在的老婆不停地给他打电话。 前夫没办法，只能撒谎说自己在公司开会。 对方听清...
    
    [![](https://upload.jianshu.io/users/upload_avatars/4790772/388e473c-fe2f-40e0-9301-e357ae8f1b41.jpeg)茶点故事](https://www.jianshu.com/u/0f438ff0a55f)阅读 4417评论 0赞 0
    
-   我是一名男催乳师，新顾客竟是母亲的闺蜜。她竟邀请我与她丈夫三人同睡...... 1 我是一名男催乳师。 一个备受争...
    
-   《网恋社死现场》 一、 面基当天，我把谈了三个月的网恋对象拉黑了。 事情是这样的。 那天我刚上完早课，叼了个冰棒走...
    
    [![](https://upload.jianshu.io/users/upload_avatars/4790772/388e473c-fe2f-40e0-9301-e357ae8f1b41.jpeg)茶点故事](https://www.jianshu.com/u/0f438ff0a55f)阅读 934评论 0赞 0
    
-   送上门的女学生 文/六酒 我在一所大专当英语老师的过程中，为了能够顺利毕业，我班里一个清纯动人的女学生找上我，说愿...
    
    [![](https://upload.jianshu.io/users/upload_avatars/4790772/388e473c-fe2f-40e0-9301-e357ae8f1b41.jpeg)茶点故事](https://www.jianshu.com/u/0f438ff0a55f)阅读 1311评论 0赞 3
    
-   皇帝的白月光自尽了，我却瞧着他一点也不伤心。 那日我去见贵妃最后一面，却惊觉她同我长得十分相似。 回宫后我的头就开...
    
    [![](https://upload.jianshu.io/users/upload_avatars/4790772/388e473c-fe2f-40e0-9301-e357ae8f1b41.jpeg)茶点故事](https://www.jianshu.com/u/0f438ff0a55f)阅读 768评论 0赞 0
    
-   谁能想到有朝一日，逼宫这种事会发生在我身边。 被逼走的是我亲妈，始作俑者是我亲小姨。 为了争得我的抚养权，母亲放弃...
    
    [![](https://upload.jianshu.io/users/upload_avatars/4790772/388e473c-fe2f-40e0-9301-e357ae8f1b41.jpeg)茶点故事](https://www.jianshu.com/u/0f438ff0a55f)阅读 972评论 0赞 2
    
-   我被心上人灌下晕药，送到了新科状元的床上。 一年后的雨水，我被人毒死，扔进枯井之中。 死前，我竟然听到了撕心裂肺的...
    

### 推荐阅读[更多精彩内容](https://www.jianshu.com/)

-   1\. 摘要 本文是辉哥Docker入门的一些摘要和资源分享，涉及DOCKER入门，框架原理,镜像制作和资源列表等内...
    
    [![](https://upload.jianshu.io/users/upload_avatars/1190574/93697e8d8af5.jpeg?imageMogr2/auto-orient/strip|imageView2/1/w/48/h/48/format/webp)笔名辉哥](https://www.jianshu.com/u/347463663a86)阅读 4,269评论 0赞 74
    
-   一、实验背景 Docker中要使用镜像，一般会从本地、docker Hup公共仓库和其它第三方公共仓库中下载镜像，...
    
-   Nexus简介 Nexus是一个多功能的仓库管理器，是企业常用的私有仓库服务器软件。目前常被用来作为Maven私服...
    
-   Docker值得关注的特性： o 文件系统隔离：每个进程容器运行在一个完全独立的根文件系统里。 o 资源隔离：系统...
    
-   概述 虚拟化技术 真一次编译到处运行 Docker 最初是 dotCloud 公司创始人 Solomon Hyke...