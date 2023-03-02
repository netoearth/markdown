> **开启掘金成长之旅！这是我参与「掘金日新计划 · 2 月更文挑战」的第 13 天，[点击查看活动详情](https://juejin.cn/post/7194721470063312933 "https://juejin.cn/post/7194721470063312933")**

## 🌳前言

## 💻操作系统的概念

-   首先在开始学习Linux前介绍一下操作系统（OS）的概念和其内部原理，因为对于Linux，就是一种【操作系统】，而且是纯命令行的操作系统，它与Windows不一样的地方在于**Linux没有桌面**，所有的操作都需要用指令去完成，可能很多用惯了Windows图形化界面的用户一下接触Linux会感觉不适应╮(╯▽╰)╭
-   好，稍微介绍了一些Linux，接下来我们来说说对于操作系统的概念，为什么要有操作系统这个东西？它到底是用来干嘛的？很多人可能都只知道Windows是一款操作系统，但是不知道它是用来干嘛的
-   首先对于我们买来的一些笔记本、台式机，里面都有许多的硬件设备，比如说内存条、硬盘、显卡、网卡、声卡等等，其中像显卡、声卡这种硬件设备并不是一插上去就可以使用的，而是需要一种东西叫做==驱动程序==，让他们在计算机中可以工作起来，而且像鼠标和键盘也是一样，都是需要驱动程序才可以进行运作。而管理这些驱动程序的就是**操作系统**
-   在操作系统上还有一层就是**用户**，用户会使用许多的应用软件，这些应用软件也是由操作系统承载的，操作系统也需要去管理这些软件，使他们不出错。**你想要是你在运行一块大型软件的时候黑屏三四次，重启五六次，主机里的硬件还是不是发出怪声，那你的使用感一定不会很好**，所以总的概括一下**操作系统使一款进行软硬件管理的软件**
-   操作系统在对我们来说是很**重要**的，具体我不细讲了，大家可以看看这篇文章【[操作系统基础知识详解](https://link.juejin.cn/?target=https%3A%2F%2Fblog.csdn.net%2Fguorui_java%2Farticle%2Fdetails%2F120943666%3Fops_request_misc%3D%26request_id%3D%26biz_id%3D102%26utm_term%3D%25E6%2593%258D%25E4%25BD%259C%25E7%25B3%25BB%25E7%25BB%259F%26utm_medium%3Ddistribute.pc_search_result.none-task-blog-2~all~sobaiduweb~default-1-120943666.142%255Ev63%255Ejs_top%2C201%255Ev3%255Econtrol_1%2C213%255Ev2%255Et3_control2%26spm%3D1018.2226.3001.4187 "https://blog.csdn.net/guorui_java/article/details/120943666?ops_request_misc=&request_id=&biz_id=102&utm_term=%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F&utm_medium=distribute.pc_search_result.none-task-blog-2~all~sobaiduweb~default-1-120943666.142%5Ev63%5Ejs_top,201%5Ev3%5Econtrol_1,213%5Ev2%5Et3_control2&spm=1018.2226.3001.4187")】

> ![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1a9a4d16b99c4538ab017bd59de40a78~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

## 💻Linux的使用环境介绍

-   好，了解了操作系统的一些基本概念后，我们就可以正式进入Linux的学习，对于Linux的学习，大家可能还没有一个基本概念，要怎么去学呢？要用什么编译器呢？这个我后续会出一期教学讲解
-   有关Linux，你可以去安装一个**虚拟机**，然后在里面装一个Linux系统进行使用，对于初学者这里推荐【CenOS】，版本的话推荐【7.6/8】。但是对于虚拟机的话后期在**维护起来**可能需要很大的精力，所以这里推荐大家使用==云服务器==，只需要选择一下系统镜像即可，买了就能使用，我用的是[腾讯云](https://link.juejin.cn/?target=https%3A%2F%2Fcloud.tencent.com%2Fact%2Fpro%2F2022double11%3FfromSource%3Dgwzcw.7058214.7058214.7058214%26utm_medium%3Dcpc%26utm_id%3Dgwzcw.7058214.7058214.7058214 "https://cloud.tencent.com/act/pro/2022double11?fromSource=gwzcw.7058214.7058214.7058214&utm_medium=cpc&utm_id=gwzcw.7058214.7058214.7058214")的
-   当然在有了服务器之后你还需要安装一个[Xshell](https://link.juejin.cn/?target=https%3A%2F%2Fwww.xshell.com%2Fzh%2Fxshell%2F "https://www.xshell.com/zh/xshell/")，它是Windows界面下访问远端不同系统下的服务器，供我们管理使用服务器，然后就可以在里面执行Linux的各种操作指令

## 🌳基本指令汇总

## 一、【whoami】指令

-   **功能：知晓当前所使用的用户**

首先一进去我们登录到【root】超级用户，这是权限最高的用户（当然还可以进行权限转义），这个时候你在执行操作的时候一定要先确认你是谁，那就是【whoami】这个操作指令，执行了这个操作指令你就可以清楚你当前使用的用户了

> ![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f208e68425cf47d8b8e08bc7a7d84bb0~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

## 二、【pwd】指令

-   **功能：显示当前所处路径**

知道了自己是谁，接下去在Linux中第二重要的一点就是==一定要清楚你当前所在的目录是哪里==，因为你执行的操作都是在这个目录下的，就想下面这个在Windows中当前目录一样

> ![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e8c4dcf127da4a6a91f04d3e6db686cc~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

-   所以我们可以执行一个操作叫【pwd】，这样就可以知晓你在那个目录下，从下面可以看到我在/root用户中

> ![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ddae286bf10c4a0ebe203483b9eeaf5f~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

## 三、【mkdir】指令

-   语法：mkdir \[选项\] dirname...
-   **功能：在当前路径下，创建一个目录/文件夹（windows）**

好，细心的你应该发现了我上面有在【lesson2】的目录下，但是这个目录要怎么创建呢，这个命令就叫做【mkdir】

> ![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/31c790f4d81e4eb1a10b61b270f8fe06~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

-   这里补充一些有关mkdir指令的操作，如果看不懂的没关系，可以先看下去，**因为命令是做汇总的，很难做到难度平缓上升**
-   刚才说到使用【mkdir】指令可以创建目录，那我可以同时创建多个目录吗？我们来试试

> ![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3c68b4f3cfde443db62acf0c06d5ddc7~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

-   可以看到，很明显是不可以的，但是我们在多级目录创建的时候在前面加上【-p】，就发现可以了，再用【tree】命令看一下，就可以知道创建成功了

## 四、【touch】指令

-   语法:touch \[选项\]... 文件...
-   **功能：创建指定的普通文件**。命令参数可更改文档或目录的日期时间，包括存取时间和更改时间，或者新建一个不存在的文件

知道了如何去创建一个目录，那可不可以在这个目录中创建一个文件呢，那当然是可以的，比如说我们创建一个文本文件【File.txt】

> ![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5b292811fae34fdb87771699a9d86507~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

___

好，上面的这一些操作就是带大家入个门，先了解一些简单的操作，接下去的话我要开始介绍一些稍微复杂的指令了，【准备发车，坐稳了】:car:

## 五、【ls】指令

-   语法： ls \[选项\]\[目录或文件\]
-   **功能：显示当前目录下所对应的文件列表（包括目录、普通文本文件...）**
-   常用选项介绍

对于【ls】，我在前面也有涉及过，很简单，就是显示当前目录下的所有文件

> ![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fcf2359ae67140d1a4d94b435083b077~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

但是这样看起来可能不太直观，接下来再介绍一个，叫做【ls -l】，它呢，就是以**list(列表)的形式**，显示文件更多的属性

> ![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5867dd11e95e420dabb21d94030c9365~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

### 1、拓展：文件的概念

-   说到了显示的详细文件信息，而且在上面我们学习了如何去创建一个文件，就给大家普及一下文件的概念
-   首先，我给到两个问题，在windows创建一个空文件，==①这个文件在哪里存着呢？②这个文件要不要占据磁盘空间大小？==
-   好，给出答案，①存在磁盘中 ②要的
-   你可能对第二个问题比较困惑，既然是一个空文件，那为什么会占据磁盘空间呢？在下面我在Windows中新创建的一个空的文本文件，虽然文件里面并没有任何内容，但是其包含各种属性：**文件名称、创建时间、文件类型、文件大小**，对于这些其实都是要占一些存储空间

> ![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/11d5d9823fb1454db3c279ac6d060f97~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

所以我们可以得出一个结论：**【文件 = 内容 + 属性（很多）】**

### 2、命令 - 命令选项

```
ls -l  列出文件的详细信息
复制代码
```

-   好，给大家拓展了一些文件的知识，接下来要解答的疑问是这个【ls -l】,不是说指令就一个连续的英文吗，为什么后面还有个【-l】，这个要再给大家说一下，对于前面的【ls】，确实是一个命令，但是对于后面的【-l】不叫做命令，叫做==命令选项==，对于ls有很多的命令选项，后续在学习的时候都会给大家一一讲到，对于这里的【-l】后面的l大家可以理解为是【list】列表的意思，也就是我们在上面说到过的以列表的形式呈现这个目录中所有的文件
-   对于这个【ls - l】，告诉你个小秘密，其实可以直接简写成【ll】

> ![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/471aca1617da4b4b8ee619aba2d0473b~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

-   对于Linux，还支持多行输入，这个我们在C语言里应该也碰到过，就是内容没输入完成，在下一行还可以继续输入

> ![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/336302df794d4bd09a0e951d23cd3190~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

___

```
ls -a  列出目录下的所有文件，包括以 . 开头的隐含文件
复制代码
```

然后在这个【ls - l】的基础上，再给大家讲一个命令，就是【ls -l -a】，【-a】显示更多的隐藏文件（若一个文件以.开头，就是隐藏文件）

> ![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/187fc4762b7a48198fd8006f94c251c1~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

-   可以看到，除了File.txt，又显示了一些隐藏文件，提前告诉你，==这里【.】指的就是当前目录，【..】指的就是上一级目录==
-   这里的【ls -l -a】，你也可以像【ll】一样简写成【ls -la】，效果也是一样的，而且对于后面的命令选项，循环是可以颠倒的，我们来看看

> ![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f3102faab29f427bb98fe3a90a77885c~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

```
ls -d  将目录象文件一样显示，而不是显示其下的文件。如：ls –d 指定目录
复制代码
```

> ![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ad0062a5c9e64dc5b77e855f182bdad1~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

```
ls -i 输出文件的 i 节点的索引信息。如 ls –ai 指定文件
复制代码
```

> ![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0446f42d286542109adaf5fa41251def~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

```
ls -s 在l文件名后输出该文件的大小。（大小排序，如何找到目录下最大的文件）
复制代码
```

> ![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/71be9f4dd86a45e590d9a91adaa8e209~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

```
ls -k 以 k 字节的形式表示文件的大小。ls –alk 指定文件
复制代码
```

> ![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6939e2ac618e4ff98675fe26316c1f48~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

```
ls -n 用数字的 UID,GID 代替名称。（介绍 UID， GID
复制代码
```

> ![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f928f840c2d44874a024e09ad10ea617~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

```
ls -F 在每个文件名后附上一个字符以说明该文件的类型，“*”表示可执行的普通文件；“/”表示目录；“@”表示符号链接；“|”表示FIFOs；“=”表示套接字(sockets)。（目录类型识别）
复制代码
```

> ![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/aadf1d03f81844d1b41e96efec522785~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

```
ls -r 对目录反向排序
复制代码
```

> ![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9cae657fef204fcb943c2d070b618002~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

```
ls -t 以时间排序
复制代码
```

> ![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d3aed8243a814ae49afb7cff0e6ab13f~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

```
ls -R 列出所有子目录下的文件。(递归) 
复制代码
```

> ![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0cda5210bc85473cba03cc8a9753fd12~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

```
ls -1 一行只输出一个文件
复制代码
```

> ![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/dcc6cdbc6e784b878e19c33cefdacec4~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

-   这个【-d】的功能就是**将目录像文件一样显示**，而不是显示其下的文件

## 六、【cd】指令

好，接下来我们再来讲一个很重要的命令，就是【cd】

-   语法:cd 目录名
-   **功能：改变工作目录。将当前工作目录改变到指定的目录下**

首先一开始，我要给你教你两个命令，就是

-   【 cd.】- ==当前路径==
-   【cd..】- ==上一级路径==（这里是两个点，不是三个点）

### 1、Windows和Linux中的路径区分

-   在介绍【cd】这个指令的之前，先给大家讲讲有关Windows和Linux中的路径有什么不同，上面有给大家列出了很多的路径，我们来和Window是中做一个对照，如下图所示
-   可以看到，在**Linux**中，【路径分隔符】使用的是==左斜杠==，也及时键盘中问号的位置，但是在**Windows**中呢，使用的是==右斜杠==，也即是键盘中|的位置，这个你有注意到吗？

> ![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7c875eff3c8f476ea47c7f9cd3953f18~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

### 2、Linux中多目录的创建【树形结构】

-   在我们平常使用Windows中，都会在一个文件夹中再创建一个文件夹做分类，【方便进行管理】，那么在Linux中，除了【mkdir】创建一个目录外，还可以创建多个目录吗，答案是：**可以的！**
-   我们来试试。可以看到，这里我继续使用【mkdir】命令创建了两个目录

> ![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/dc324995a1c14fefa57e6c66210ef753~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

-   好，接下来我要很多命令一起执行了，怕你看不懂，所以解释得详细一些:heart\_eyes:

> ![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0fb3c8259d674a6aa9c81ea4e1ae5349~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

-   有关这个**tree命令**，大家可能在执行的时候会报出【command not found】，意思就是【未找到命令】，这个的话其实很简单，因为在你的服务器后面没有安装这个命令，所以它是不会有反应的，你只需要以下执行这句命令即可【**yum install -y tree**】，其他的你现在还无需知道

___

-   然后我们再来说说有关Linux下的**目录存储结构**
-   Linux系统中，磁盘上的文件和目录被组织成一棵==目录树==，每个节点都是目录或文件。

> ![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cf9c06cd9ed546f49c6a48b634b7e67a~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

### 3、路径【绝对路径和相对路径】

-   在Windows中，我们为了找到一个文件或者文件夹，通常用什么去定位这个文件呢？？？【**路径**】！！！，Linux下也是如此的，为什么要用路径呢？因为**路径往往具备唯一性**
-   以上面的这个【lib】文件为例，要找到它，也就是要找到【user】，那要怎么找到【user】呢，没错就是在根目录下去找

> ![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6bd5367ab72b44258cd85261093acfd6~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

-   我在【dir1】里创建了一个【dir3】的目录，然后在里面有创建了一个‘【test.c】的文件，接着用【pwd】去查看当然所在目录，可以知晓，这其实就是绝对路径，也就是==从根目录开始，定位文件的路径==
-   **对于绝对路径来说具有唯一性**，为什么这么说呢，从【数据结构】的角度来看，从一个叶子结点一个个往上找它的父亲，一定都是只有一个，因为父亲都是唯一的，所以它的根节点也一定唯一，**因此从根节点开始到这个叶子结点的路径必然唯一**

#### 【cd..】和【ls ..】—— 快速定位

-   那什么是相对路径呢？在介绍相对路径前我要教你一个**快速定位路径的方法**，也就是要用到我们上面的【cd..】这个回到上一级目录的操作，而且对于我们可以通过【ls】配合这个【..】**去快速灵活地查看上一目录的其他子目录下还有什么子目录和文件**，我们来操作看看

> ![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7c9ea0af91b04cf2b97e1c29307afdf9~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

-   从上面的操作我们可以看出，通过【ls】和【..】的配合，完美地灵活定位其他目录中的子目录和文件，这样就可以通过【ls】查看的方式快速了解到其他子目录中有哪些内容
-   接下来我们通过【cd..】和【ls ..】打一个配合，就可以做到**快速传送的功效**

> ![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/46c6624ef53047e2b587f6fbbba53144~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

-   这个【../../dir2】其实指的就是==相对路径==，大家是不是觉得这种路径表示法很方便呢。是的，它是很方便简洁，也不需要一一再去找目录和文件了，但是它有一个缺点，就是【**当前路径可能会发生变化**】
-   此话怎讲呢？我们来看一下下面这个案例。可以看到，我一定是去进行定位的查找，然后cd进去，但是当我将当前的路径做一个变化的时候，就变得紊乱了，因为【cd ../../】完全就没有内容，所以我们可以得出结论：**相对路径只有在特定条件下才有效**

> ![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/36ec5b8335f345bb894b75ee90023bbb~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

___

看完了上面这些案例，接下去我们对绝对路径和相对路径做一个总结吧:snowman:

-   **绝对路径**：麻烦，一般是在某些==配置文件==中，对某种文件进行配置的时候采用
-   **相对路径**：用起来==简单==，后续指令操作的时候，常用的路径定位方案！

> 说得通俗易懂一些就比如你叫张三，坐在教室的第三排第三列，你现在找你的同桌，他坐在第三排第四列，这个就很明确，绝对是可以找到的，指的就是【绝对路径】；然后另一种方式就是你的同桌他不就在你的隔壁嘛，也就类似于cd..的意思，那也就是【相对路径】

### 4、【cd ~】与【cd -】

-   接下再介绍两个有关【cd】的命令，首先介绍【cd ~】,它可以直接进入当前用户（whoami）的**家目录**
-   我现在使用的是【root】超级用户，无论在哪个目录下执行都会直接跳转到【root】用户的目录下，也就是你的家目录。当你使用自己创建的用户时，执行这个命令就会跳转到你的用户目录下。我们来看一下【root】账户的

> ![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/abbc5696a7f141b097516b9f1e3982e4~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

-   可以看到，**无论我在那个路径下，都可以直接跳转到当前用户的路径下**
-   在Linux里有这个当前用户，那么我们平常使用的Windws有没有呢，答案是：有的。我们来看看

> ![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/98d54a7885dd4f6d8356f3a86e7b20c6~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

___

-   好，接下来我们再来玩一个。就是这个【cd -】，这个命令可以**回到最近上一次所处的目录**，我们来看一下

> ![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2c3d82adbb6e4a58b01e7ebcd664df20~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

___

## 知识补充

### ⭐编写代码的记事本文件：nano

-   在C语言中，我们有VS2019、DevC++这样的软件可供我们编写代码，但是Linux里可以写代码吗，答案是：可以的
-   很多安装好的云服务器都是自带【nano】的，我们只需要执行【nano test.c】这个命令就可以进去了

> ![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0d1418cff2484e038a746ae7eb0edbe1~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

-   接着就进入nano所创建的test.c这个文件中，可以开始编辑代码了

> ![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ee34e08f127545f6b822dc04dc5aef63~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

-   一下就是对这个代码文件的一系列操作

> ![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/10869335306f45b390ebd54f65a6c9a5~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

## 七、【stat】指令

-   **功能：查看文件所有的属性**

> ![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/af03ced8b9594c1fb94b122ce622f881~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

-   我们现在主要关注的一点就是这个【ACM时间】，看下面。所以简称ACM时间，不要多想，不是那个【==ACM==】 **Access：最后的访问时间 Modify：文件内容最后修改的时间 Change：文件属性最后修改的时间**
    
-   这指令大家了解一下即可，后面我们讲到再做说明
    

## 七、【rmdir && rm】指令

-   好，接下去再来说一个指令，我们上面说到【mkdir】可以创建一个目录，【touch】可以创建一个文件，但是创建完之后要是我不想要了想将其删除怎么办呢？于是我们就可以使用到这两条命令
-   首先我们来说说【rmdir】，它的作用是**删除一个空目录**。从下图可以看到，若是你要删除的目录为空的话，是删不了的

> ![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b7ab02f3ae354cffbf790e2e4244611a~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

-   那有同学就问，那删不了怎么办呀？那不是就。。。
-   别怕，这不是还有一条指令嘛，【rm】就可以做到。以下是它的常用命令选项

> **\-f 即使文件属性为只读(即写保护)，亦直接删除（强制删除） -i 删除前逐一询问确认 -r 删除目录及其下所有文件（递归式删除）**

> ![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/963e1d05fb314f00b95cfe905ed14dac~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

-   还有一个的话就是这个【-f】，f值得就是【force】暴力的意思，也就是进行一个强制删除，你可以创建一个只读文件，然后就会发现用-r是删除不了的，要使用【-f】才可以进行一个删除。
-   然后下面我通过一个普通用户的账号进行操作，在我当前账户创建一个root超级账户的写保护文件，然后当我去对【rm】删除这个文件时，系统就会给出提示，询问我是否要继续删除，因为这个相当于是root账户的；但若是我们用上面这种【-f】的暴力删除操作，系统就不会给出提示去询问，**直接会进行一个删除**

> ![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/43829068aa094d7381ea81aaf4945274~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

-   当然【-r】和【-f】还可以写在一起，表示暴力递归式地删除一个目录，这里我就不展示了，大家下去可以自己试试看

___

## 八、【man】命令

-   **功能：通过查看联机手册获取帮助**
-   讲到现在，很多同学可能会有些疲倦，觉得这个Linux中的命令怎么会有这么多的参数（命令选项），记不住该怎么办呢？于是这个时候就有一种办法可以让你不需要记这些东西，只需要记住一个命令，那就是这个【man】，然后就可以通过这个【man】命令去查找一些==联机手册==，在这些联机手册中呢，你就可以找到你想要的一些命令的参数
-   接下来我们在Xshell中试试

> ![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/228f37e60404432d97ef4df483b4d73c~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

-   比如说我要再这里查找一个C语言的【printf】有关的参数，就可以直接【man printf】。可以看到，我们这里是查找到了一些相关的信息。如果是要退出和这个手册界面的话按【q】就可以了，腾讯云上是这样，但是其他云服务器上怎样我就不知道了

> ![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0b0bc0f3f7d94aa582b7357c84223639~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

-   可以看到，还有一些其他的内容也是可以查询的

> ![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e979497b54874ef4a7f88df6bb6caeaf~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

-   那其实正规来说我们应该这么写【man 1 printf】和【man 2 fork】。因为这个指令手册一共是分为八章，每一章呢都有其相对应的接口章节，比如说我们的printf，它其实就是一个普通的命令，是属于第一章的
-   既然说到了这个命令，就顺便展示一下:point\_down:

> ![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c47c09e34e9b483caf96271a54c649ff~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

-   然后下面就是刚才所说的man这个指令手册的八章，每一章都有其对应的整体内容总括说明

> 1 是**普通的命令** 2 是**系统调用**,如open,write之类的(通过这个，至少可以很方便的查到调用这个函数，需要加什么头文 件) 3 是**库函数**,如printf,fread4是特殊文件,也就是/dev下的各种设备文件 5 是指**文件的格式**,比如passwd, 就会说明这个文件中各个字段的含义 6 是给**游戏**留的,由各个游戏自己定义 7 是附件还有一些**变量**,比如向environ这种全局变量在这里就有说明 8 是\*\*系统管理用的命令,\*\*这些命令只能由root使用,如ifconfig

-   那有同学说，这不是又要我去记很多内容了，还是记不住怎么办呢，那这个设计者也是有考虑到，如果你连和这个【man】的命令参数都不是很确定的话，其实也是可以查的，就是通过【==man man==】这条指令，我们一起来看看
-   然后就可以看到，通过里面的这个指令手册指南，就可以知道哪个章节所对应哪个内容了，如果看不懂英语的小伙伴可以使用【翻译软件】哦~

> ![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5a9ee1916c794fd7810f181f8ca8ce51~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

-   于是我们就可以这么去使用:point\_down:

> ![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f65222ed193f47cd8c4dabc3c0405e8c~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

-   但是我们有的时候在查询一些接口的时候，可能会这个man手册，因为我们使用的是【云服务器】，它是一个生产环境，即一个真实的线上环境，所以云服务器上的环境内容一般是非常精简的，无法做到像正常的Linux系统那样拥有很丰富的内容，所以我们这时可以考虑自己去进行一个安装，将一些没有的手册内容安装进我们的云服务器，指令大概是下面这样

> 【**yum install -y man-pages**】—— ==超级用户root==

> 【**sudo yum install -y man-pages**】—— ==普通用户user==

> ![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/033f8f15671f432c8f84bf866d47d669~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

___

接下去我讲讲对【文件本身】的一些操作，分别有**拷贝（复制）、移动（剪切）、重命名**

## 九、【cp】指令

**功能: 复制文件或目录（默认只能拷贝文件）**

-   在Windows中，对于一个文件的复制我们可以使用【ctrl + c】和【ctrl + v】，或者是右键单击进行复制和粘贴，但是在Linux下，可以使用【copy】和【paste】进行命令的拷贝和粘贴，但是不可以对文件进行操作
-   对文件进行操作的话要使用到【cp】指令。以下是它的语法格式

**cp （源文件/源目录） （目标文件/目标目录）**

-   我们到Xshell中来看看。这里的【..】指的是上一级目录，我们在上面有讲到过

> ![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/609566c14bbb49caa369fbd170a3134e~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

-   那我们想拷目录怎么办呢？这个【-r】的话尽量是直接更加命令的后面，不要放在最后面，虽然也是允许的，但若是这样写的话在有些操作系统下是过不了的，比如说苹果的mac系统

> ![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3f9a42d0384c4a9999121f8c8a207ab6~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

-   那我现在不想要拷贝完后的这个内容，想把它删掉呢，那我们就可以使用上面的【rm】指令

> ![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6a66888962864982bb0da6f0a686d2d7~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

___

## 十、【mv】指令

-   **功能：可以用来移动文件或者将文件改名**
-   **语法：【mv src(文件/目录) dst(一定是一个目录)】**

### 1、移动文件或目录

-   好，看完基本的功能和语法后，我们来实体演示一下

> ![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2f4561f4d45f4c7e8858f73e06dbb9e2~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

-   那移过去了之后我又想移回来呢？

> ![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/862cf54d2f014a5095e1635034d5d8d6~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

-   好，我现在要移动一个目录，我们一起来看看

> ![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c5b4a750ba5e429da3a84d4f52693e34~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

-   接下去还是一样，把它给移回来

> ![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/71f5e8d1de104623a986008d3725d558~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

-   可以看出，对于文件的这个拷贝和移动的操作，是非常灵活多变的，**这个一定要多练习，做到熟能生巧**

### 2、重命名

-   好，接下去我们再来讲一个操作，既然对文件可以进行拷贝和移动，那么可不可对一个文件进行重命名呢，答案是：可以的

**语法：【mv src(文件) dst(文件)】**

-   对于重命名的话有三种形式

> ①如果后面跟的只是当前的==文件名==【dst和src是同一文件类型】，那就只做**重命名**工作

> ②如果dst为一个==路径==【..上一级路径】，就是纯正的剪切

> ③如果dst为一个【==路径 + 文件==】，在后面又指定了一个不存在的文件名。就会【重命名 + 剪切】

-   我来做个演示

> ![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/dc00b10b0f794031b25fff21806fa83b~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

-   目录操作的话也是类同，你可以自己去试一试

> ![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f68a71119ca54847a7c13f715ef69a5a~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

___

-   好，以上就是对于一些文件的基本操作，我们来小结一下。通过【man】指令，可以查找对应的手册，就可以查看对应的参数；通过【cp】，可以进行文件的拷贝和复制；通过【mv】，可以进行文件的移动和剪切，而且还可以对文件进行重命名
-   接下去我们继续介绍一些文件相关的操作指令，不过接下来要讲的都是对==文件内容==进行的操作，而上面的那些指令则是对==文件本身==进行的一个操作

## 十一、【cat】指令

**功能： 查看目标文件的内容【直接打印全部】** **语法：cat \[选项\]\[文件\]**

-   接下去来说说【cat】指令，对于这个指令的话还是比较直观的，简单一些就是后面直接跟一个文件，然后按回车后就可以直接打印出文件里的内容。我来演示一下
-   首先给文件写点内容进去

> ![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7ecc7797400941f4bd1352e50f9573c2~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

-   可以看到，很直观，就打印出来了

> ![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/872a2a39d4e54328a8dfa6fc43e63a90~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

-   但是这样的话太简单了，我需要你做一些拓展知识:book:

### 拓展

-   首先，对于【cat】，它不仅是可以显示文本的内容，若是你输入【cat】这个命令，那么之后内容就可以由你来输入，你输什么显示屏就给你打印什么

> ![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2615b90db6ec4890bcdfe4fc94d92f0e~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

#### echo命令

-   既然说到了这个输入输出，以及显示，就又可以涉及到一个命令：叫做【echo】，这个你使用man也是可以查找到的。用来在显示器上现实一段字符串

> ![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7e08bfa68d584bfe8e15713cc9b959c4~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

-   也是非常简单，就像这样

> ![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/960858d167374b6d8e3fd82732354f68~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

-   然后下面讲一些和echo相关的操作

#### 输出重定向【覆盖重定向】

-   首先就是输出重定向，就是我可以将一段字符串本文内容写到一个已有的文件中去。**但是这并没有展现重定向的本质**

> ![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a42e284437fa438e8eb1337ab88ac705~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

-   下面我再给一个示例。这就是真正的输出重定向

> ![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/832f527a22384a769e20b80914e020ea~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

#### 追加重定向

-   接下来再来介绍一个，叫做【追加重定向】，刚才在讲述【输出重定向】时，我们知道了使用【echo】可以使得一串文本写入一个文件中，如果目标文件**不存在，会自动创建**，把本来应该显示的内容**写到文件中**
-   那我们多写几条试试。可以看到，多写几条也没有，而且**写不一样的内容还会被覆盖**

> ![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9c37beb046e549eba4130c8876c873a0~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

-   于是我们就可以使用【追加重定向】对本文的内容进行一个扩充

> ![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d1ba07db1f264d63b68d143799f9c4f8~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

#### 输入重定向

-   有输出重定向，一定也有输入重定向，我们来看看

> ![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0f986a7ba7154bb39053a455059ef7a2~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

-   \==下面是它们的区别==

> 1、cat < mylog.txt —— 通过**重定向**获得的文件内容

> 2、cat mylog.txt —— 通过**命令行参数**获得的文件内容

-   除了这种，还有一种指令也可以做输入，叫【wc】，下面是它的一些指令操作，了解一些即可~

> ![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c8e0dd4cfac54b2abfad6005b0fff14c~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

#### 计算机内部文件读写原理

-   既然上面讲到了这么多文件内容的输入和输出，那么接下去我就来说说在计算机内部这个**文件的读写**到底是怎样的，首先你要知道的是，当我们程序中读取内容，从哪里读入的？这个我之前在C语言的时候有讲到过，对于程序，它是从**键盘中读取数据**的，也就是【read】，读取完后再进行一些处理，接着会**写入到显示上**，才会进行一个输出。原理图大概就想下面这样

> ![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1f0ab0c16038467e9a0339a796f9a449~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

-   而我们一开始讲的【输出重定向】呢，就是使用==echo "hello bit" > change4.txt==，将这个内容显示到显示器上，即写入到显示器文件，因此也可以将显示器看作为是一种“文件”，就和我们C语言的程序获取数据使用【scanf】是一个道理，都是从键盘去进行一个获取。
-   讲了这么多文件相关的内容和知识，其实就是为了让你知道

**\--->【Linux下一切皆文件】<---**

___

-   好，谈了这么的补充的内容，也是希望能给到读者更多的了解，能加深对Linux其他内容的掌握，对知识【融会贯通】。接下去我们回去谈谈cat的一些命令选项，一些地方也会使用到

> **\-b 对非空输出行编号 -n 对输出的所有行编号 -s 不输出多行空行**

-   到Xshell中来看看吧

> ![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7baba15d1805482a9810e1f34fb85b0e~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

___

-   接下去说几个操作比较简单的指令，分别是【more】、【less】、【head】、【tail】

## 十二、【more】指令

**语法：more \[选项\]\[文件\]** **功能：more命令，功能类似 cat**

-   在讲【more】之前，我们需要先在一个文件中放入很多的内容。这个指令当做了解，现阶段不多要求

```
cnt=0; while [ $cnt -le 1000 ]; do echo "hello $cnt"; let cnt++; done > f3.txt
复制代码
```

-   然后就可以看到这个文件中写入了1000条内容

> ![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/07d404c85f2942cf9e71ae18a65dff5f~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

-   但是这样拖动观看文件非常不方便，于是我们有专门的指令可以进行观看，就是使用这个【more】

![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9578c021dd594ded8eacb0836425b7ec~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

-   但是对于【more】指令，显示到一个屏幕满了就不显示了，而且只能下翻，不能上翻，在下翻的过程中按回车就可以了，想要退出的话按【q】
-   但是这样还是觉得比较麻烦，所以这个命令我们用得比较少一些，功能远没有下一个指令【less】来的强大。所以我们直接来讲讲【less】吧

## 十三、【less】指令

-   less 工具也是对文件或其它输出进行**分页显示**的工具，应该说是linux正统查看文件内容的工具，**功能极其强大**。
-   less 的用法比起 more 更加的**有弹性**。在 more 的时候，我们并没有办法向前面翻， 只能往后面看
-   但若使用了 less 时，就可以使用 **\[pageup\]\[pagedown\]** 等按键的功能来往前往后翻看文件，更容易用 来查看一个文件的内容！ 除此之外，在 less 里头可以拥有更多的搜索功能，==不止可以向下搜，也可以向上搜==

> 看完这些你是不是很想学学【less】，让我们马上进入吧

-   下面是我是使用上下翻页弄的，大家下去可以自己试试

> ![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9e5dd1b2ed8346a0a31d789ad9b71547~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

-   除此之外，【less】还可以查找某一个固定的内容，只需要【/ + 搜索的内容】即可

> ![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e4128b36586e44dbbf4bc28157213da8~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

-   然后的话自然是它的一些命令选项

> **\-i 忽略搜索时的大小写 -N 显示每行的行号 /字符串：向下搜索“字符串”的功能 ?字符串：向上搜索“字符串”的功能 n：重复前一个搜索（与 / 或 ? 有关） N：反向重复前一个搜索（与 / 或 ? 有关） q:quit**

-   稍微演示两个

> ![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/954a0c0ce2b04da3bbf982cb734b4404~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

> ![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e4beab466d3a4ef083e247cadef442c9~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

-   怎么样，是不是觉得【less】蛮强大的，下去后可以自己试试看，研究一些相关的

___

-   然后我们再来说说【head】和【tail】这两个，在后续也是用的很频繁，也是很不错的两个指令

## 十四、【head】指令

**功能：显示开头某个数量的文字区块**

-   用法很简单，我们来玩玩

> ![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cb55f1d1c1f34af0add2d6ccab07f8aa~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

-   还可以显示指定的行数【**\-n<行数> 显示的行数**】

> ![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/65c01c21dce54ce59e83705ffd0462a2~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

## 十五、【tail】指令

**功能：显示结尾某个数量的文字区块**

-   然后再来看看【tail】

> ![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0a5a0ce745434bfcb834cf4255394d0c~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp) ![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4b80631dd2714472b9f95d8d49214860~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

## 【管道】知识拓展

-   **作用：【级联多条命令，让命令和命令组合完成批量化文本处理的任务】**
-   对于【head】和【tail】我很快就讲完了，这里再给大家拓展一些有关Linux中【管道】的知识，首先来看看下面这条指令

```
cat f3.txt | wc -l
复制代码
```

> ![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3827c1f0c58a4ce2a33776b56cb39486~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

-   接着再根据这条指令给大家讲一讲Linux中有关管道的知识：对于下面这条**复合指令**，是由我们上面所学过的指令所构成的，==第一条==是打印【f3.txt】中的内容；==第二条==指令本来在后面跟上一个文件名，用于显示这个文件中有多少行数，但是我将它们合并到了一起，中间的【|】作为连接它们的一条管道，你可以将这个管道看做为是一个文件，f3.txt中的内容本来应该直接显示到显示器上的，但是现在其中的内容都放到了一个管道里，然后就使用【wc -l】这条指令去这个管道里取数据进行相应的统计
-   这样去写的话我们就不需要将两条指令分开来了，可以写在一起

> ![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/67ae7d92fdf54a599421590691b5e43e~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

-   一个例子还不够形象，接下去我再通过一个管道相关的指令为你做讲解。先来看一下要实现的功能要求

> \==要求：显示f3.txt中500行到520行的内容==

-   刚才我教了你使用【head】显示前n行，使用【tail】显示后n行，但是现在要显示中间的内容，这该怎么搞呢？
-   我们首先可以取到前520行，放到一个文件里，接着再取这520行的后20行，这样其实就可以取出这中间的内容了

> ![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/41d2801ee2924d7baf6e533ab09843e7~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

-   但是这样的话就需要再重新创建一个文件，需要输入两行命令，接下来教你一个很奇妙的方法:bulb:
-   也就是利用我们上面所说的管道来执行，基本的实现思想你已经会了，接下去只要将这些命令做一个**级联**就行

> ![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/01a235fca9474389bb1109f4c438e136~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

-   除此之外，还可以继续添加，这些内容有多少行数。就是加一个【wc -l】

> ![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/598b9aa6123f470fbfac484c45745418~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

## 十六、【date】与时间相关的指令

### 1、date显示

-   下面是直接使用date这个命令显示出的普通的日期格式

> ![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4e213839fff04a03beca9c5cf5db14d7~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

-   这样看很明显比较费力，我们对其进行一个格式化:point\_down:

### 2、date格式化时间

-   下面是一个格式化的常用标记

> **%H : 小时(00..23) %M : 分钟(00..59) %S : 秒(00..61) %X : 相当于 %H:%M:%S %d : 日 (01..31) %m : 月份 (01..12) %Y : 完整年份 (0000..9999) %F : 相当于 %Y-%m-%d**

-   我们使用这些标记来进行一个格式化

> ![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b86804be29e6454dacbe7f7579068b86~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

### 3、date设定时间

-   除了显示当前时间，我们还可以自己来设置时间。你可以将这些命令试着打一下

> **date -s //设置当前时间，只有root权限才能设置，其他只能查看。 date -s 20080523 //设置成20080523，这样会把具体时间设置成空00:00:00 date -s 01:01:01 //设置具体时间，不会对日期做更改 date -s “01:01:01 2008-05-23″ //这样可以设置全部时间 date -s “01:01:01 20080523″ //这样可以设置全部时间 date -s “2008-05-23 01:01:01″ //这样可以设置全部时间 date -s “20080523 01:01:01″ //这样可以设置全部时间**

### 4、时间戳

-   接下去我们来讲一个很重要的东西，叫做【[时间戳](https://link.juejin.cn/?target=https%3A%2F%2Fblog.csdn.net%2Fweixin_45627194%2Farticle%2Fdetails%2F109474670%3Fops_request_misc%3D%25257B%252522request%25255Fid%252522%25253A%252522166902000416782427461118%252522%25252C%252522scm%252522%25253A%25252220140713.130102334..%252522%25257D%26request_id%3D166902000416782427461118%26biz_id%3D0%26utm_medium%3Ddistribute.pc_search_result.none-task-blog-2~all~top_positive~default-1-109474670-null-null.142%255Ev66%255Econtrol%2C201%255Ev3%255Econtrol_1%2C213%255Ev2%255Et3_control2%26utm_term%3D%25E6%2597%25B6%25E9%2597%25B4%25E6%2588%25B3%26spm%3D1018.2226.3001.4187 "https://blog.csdn.net/weixin_45627194/article/details/109474670?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522166902000416782427461118%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=166902000416782427461118&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~top_positive~default-1-109474670-null-null.142%5Ev66%5Econtrol,201%5Ev3%5Econtrol_1,213%5Ev2%5Et3_control2&utm_term=%E6%97%B6%E9%97%B4%E6%88%B3&spm=1018.2226.3001.4187")】，不懂的同学可以先去了解一下。然后我们来看如何将当前的时间转化为时间戳

```
时间->时间戳：date +%s
复制代码
```

> ![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/373b964e5f534e5e8cec61fa65dabc35~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

-   时间戳：格林威治时间**1970年01月01日00时00分00秒**(北京时间1970年01月01日08时00分00秒)起至现在的总秒数
-   所以对于时间戳来说，是有划时代意义的，而且在一些场合下用的也是比较广泛。然后再来看看如何将一个时间戳再转化回我们现在的时间

```
date -d@时间戳
复制代码
```

> ![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/78c0f9ee0cbd44f581e02527db1f2c78~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

### 5、为何需要看时间？

-   题外话：上面讲了看时间的很多方式，那有同学问了，既然现在都有手机、电脑、手表，那为什么还要刻意去通过指令去看时间呢？这个其实就要讲到我们一些日常的工作了，对于系统维护方面的工作，需要进入到一个公司内部封闭的机房，然后在里面进行操作，但是这个机房却是不让带手机进入的，因为管理比较严格，所以你在里面待上几天是看不到时间的，只有Linux的这种命令行可以执行，因此我们需要学会这些

## 十七、【cal】指令

**功能： 用于查看日历等时间信息，如只有一个参数，则表示年份(1-9999)，如有两个参数，则表示月份和年份**

```
cal
复制代码
```

> ![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/399e68692c9f49ad9804b8977e45fb26~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

```
cal -3   显示系统前一个月，当前月，下一个月的月历
复制代码
```

> ![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e6413fd81e7a4ca08f59b93d13a66bca~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

```
cal -j  显示在当年中的第几天（一年日期按天算，从1月1号算起，默认显示当前月在一年中的天数）
复制代码
```

> ![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/38cd046f05de45669d6361cb279c2ae6~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

```
cal -y  显示当前年份的日历
复制代码
```

> ![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7fa3ee8a4e234787860971329584e988~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

> ![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/db824469b4d04874959110289ba4a0af~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

-   可以看到，**很直观、很简单、很震撼**

## 十八、【sort】指令

**作用：对文件中的内容进行排序**【从左向右拿每行的第一个字母的ASCLL码值做比较，默认升序】

> ![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0e61f757a5794509a955383319b278c7~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

-   可以升序，当然也可以降序，在sort后加个【-r】即可

> ![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e4ea2c31b39044008520cff423055f37~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

## 十九、【uniq】指令

**作用：相邻内容去重**

> ![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c2acd923e07c4ea1bae0c84da0572500~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

## Linux搜索三剑客🗡

-   接下去来说说Linux中的搜索指令，莫属这三剑客:point\_down:

### 二十一、【find】指令（灰常重要）

-   Linux下find命令在目录结构中搜索文件，并执行**指定的操作**。
-   Linux下find命令提供了相当多的查找条件，功能很强大。由于find具有强大的功能，所以它的**选项也很 多**，其中大部分选项都值得我们花时间来了解一下。
-   即使系统中含有网络文件系统( NFS)，find命令在该文件系统中同样有效，只你具有相应的权限。
-   在运行一个非常消耗资源的find命令时，很多人都倾向于把它放在后台执行，因为遍历一个大的文件系 统可能会花费很长的时间(这里是指30G字节以上的文件系统)

**功能： 用于在文件树种查找文件，并作出相应的处理（可能访问磁盘）** **语法： find pathname -options** **常用选项：**

-   \-name 按照文件名查找文件

```
find ...(很多东西可以搜)
复制代码
```

\==给出两个示例==

> ![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1c32a33984a34ee9a220a986375b8fa3~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

> ![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5bf3f9abf7474e6b9ecd2549d00b102b~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

由于find命令实在是太多了，放不下，可以看看这篇--->[find指令详解](https://link.juejin.cn/?target=https%3A%2F%2Fblog.csdn.net%2Fm0_46517180%2Farticle%2Fdetails%2F114387542%3Fops_request_misc%3D%25257B%252522request%25255Fid%252522%25253A%252522166902864816800186585768%252522%25252C%252522scm%252522%25253A%25252220140713.130102334..%252522%25257D%26request_id%3D166902864816800186585768%26biz_id%3D0%26utm_medium%3Ddistribute.pc_search_result.none-task-blog-2~all~sobaiduend~default-1-114387542-null-null.142%255Ev66%255Econtrol%2C201%255Ev3%255Econtrol_1%2C213%255Ev2%255Et3_control2%26utm_term%3Dfind%25E6%258C%2587%25E4%25BB%25A4%26spm%3D1018.2226.3001.4187 "https://blog.csdn.net/m0_46517180/article/details/114387542?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522166902864816800186585768%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=166902864816800186585768&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~sobaiduend~default-1-114387542-null-null.142%5Ev66%5Econtrol,201%5Ev3%5Econtrol_1,213%5Ev2%5Et3_control2&utm_term=find%E6%8C%87%E4%BB%A4&spm=1018.2226.3001.4187")

### 二十一、【which】指令

**作用：搜索对应指令的路径**

```
which 指令
复制代码
```

> ![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/75820ebb3f1546608176c4636f72cf09~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

### 二十二、【whereis】指令

**作用：在系统目录下搜索指定的文件、指令、程序或者指定的归档文件、压缩包**

```
whereis 搜索的内容
复制代码
```

> ![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9a156b2a7607405a91e064031b466224~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

___

-   经过上述三剑客的搜索，有同学可能会说，竟然可以搜索那么多东西，指令都能搜得到，是的，大家要记住一点，**在Linux中，指令就是文件**
-   经历了上面这么多指令的学习，很多同学说光是这个指令我就记不住了，还要记这么多命令选项去实现对应的功能。是的，记这些指令确实是比较枯燥，但是这些指令就是要多练习、多打才可以。所以继续下去吧，还有呢（doge）

## 二十三、【alias】指令

**作用：给特定命令起别名**

-   刚才有说到一些命令有很多的命令选项，就像是【ls】，就有12个命令选项，对应它们各自的功能。就想我打的下面这条

```
ls -a -l -i -n
复制代码
```

> ![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1d274f363d7d4e628be95cdf8d30e8c6~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

-   确实，显示出来的内容很丰富，也达到了我们的要求，但是每一次都去这么敲就会感觉很难受，若是你没有很熟练的话就又需要去查看手册
-   这个时候就可以使用到【alias】了，具体就是这么操作

```
alias myls='ls -a -l -i -n'
复制代码
```

-   可以看到，显示的内容是一模一样，完全就剩下了很多精力。所以对于一些我们常用的指令选项，可以将其重起一个别名，方便我们日后使用

> ![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e35946cc98634f1595967aec54cf6c71~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

## 二十四、【grep】指令

**作用：在文件中搜索字符串，将找到的行打印出来** **语法： grep \[选项\] 搜寻字符串 文件**

-   对于【grep】指令，它可以将**指定文本内容**按照特定**关键字**来进行按行==筛选==，我们通过Xshell来看一下

> ![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/71e702448ebd4a5fb1cd55f532559de5~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

-   当然【grep】除了这些还有其他功能可以实现，主要是通过命令选项来完成

> **\-i ：忽略大小写的不同，所以大小写视为相同 -n ：顺便输出行号 -v ：反向选择，亦即显示出没有 '搜寻字符串' 内容的那一行**

-   我来演示一下💻

> ![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/99e54872371b4fae973de59f1880db85~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

## 二十五、【top】指令

**作用：任务管理器【查看CPU占用 】**

> ![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/68673925177b4a42b7a3a892f501ff96~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

## 二十六、【zip/unzip】指令

**功能： 将目录或文件压缩成zip格式** **语法： zip 压缩文件.zip 目录或文件**

-   接下去讲讲有关压缩方面的知识，这一大家应该是比较熟悉的，因此日常会遇到挺多的内容都需要打包成压缩包，而且在打开的时候需要进行一个解压缩，具体我们通过Xshell来看看
    
-   接下去进行一连串的操作，不要眨眼了
    

```
zip [压缩文件名.zip] [要压缩的文件或目录]
复制代码
```

> ![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/84ca15e75b0a4944be2ac114e9d64f35~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

```
unzip [压缩文件名.zip] 
复制代码
```

> ![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d88d219a0ef841f29623418722111f35~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

-   可以看到，🦈啥也没有，这其实是因为你在打包压缩的时候只是拿了一个空壳过来，里面的东西并没有一起做一个封装，因此才会啥都没有，这里打包压缩的时候我们应该使用一个**递归式**的方法进行:book:
    
-   **\-r 递归处理，将指定目录下的所有文件和子目录一并处理**
    

> ![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4816d305b3fb4209a5d87ad04eab6483~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

-   上面的操作是将.zip文件解压到当前目录下，那我们就会想可以不以解压到指定路径下呢，**当然是可以的**

```
unzip 压缩文件 -d 路径
复制代码
```

-   来试试看

> ![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/edcfac03ebd542f4b54105d3a71da77f~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

## 二十七、【wc】指令

**功能：统计指定文件中的字节数、字数、行数，并将统计结果显示输出**

-   这个wc指令我在上面有在将其他指令的时候有提到过，这里再详细介绍一下。下面是它的常用命令选项

```
-l 统计行数
-c 统计字节数
-m 统计字符数。这个标志不能与 -c 标志一起使用
-w 统计字数。一个字被定义为由空白、跳格或换行字符分隔的字符串
-L 打印最长行的长度
--help 显示帮助信息并退出
--version 显示版本信息并退出
复制代码
```

-   然后到Xshell中来演示一下

> ![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9d10644073754c46a71ff47158b4dc12~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

## 二十八、【tar】指令

**功能：打包/解包，不打开它，直接看内容**

-   在上一个指令中，我们讲到了【zip/unzip】这两个压缩和解压缩的命令，但是呢这两个命令只能对以.zip为后缀的压缩文件起效果，在我们日常生活中，一定不止遇到这么一种压缩文件，还有.rar、.tgz、.7z等等，因此为了能够操作更多的压缩文件，我们还需要学习一个指令叫做【tar】对文件的压缩和解压它具有更多的可能性，我们一起来学习一下
-   首先的话你要知道一些它的常用选项，这是比较多的

```
-c ：建立一个压缩文件的参数指令(create 的意思)；
-x ：解开一个压缩文件的参数指令！
-t ：查看 tarfile 里面的文件！
-z ：是否同时具有 gzip 的属性？亦即是否需要用 gzip 压缩？
-j ：是否同时具有 bzip2 的属性？亦即是否需要用 bzip2 压缩？
-v ：压缩的过程中显示文件！这个常用，但不建议用在背景执行过程！
-f ：使用档名，请留意，在 f 之后要立即接档名喔！不要再加参数！
-C ： 解压到指定目录
复制代码
```

-   而且对于这个命令的命令选项的使用，一般都需要两三个一起复用，说两个最常用的
-   首先是==打包==

```
tar -czf 压缩文件.后缀 源文件
复制代码
```

> ![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b4b3c20503e747eeaccef788450239bd~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

-   接着再是==解包==

```
tar -xzf 压缩文件.后缀
复制代码
```

> ![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/183bee5069b94d14bd9edefe5b35fbdd~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

-   当然，还可以将其解包到指定路径下，加个【-C】即可

```
tar -xzf 压缩文件.后缀 -C 具体路径
复制代码
```

> ![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/288882a340ae4d04bdae835fb67df520~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

-   好，到这里我就介绍完了这个命令，那有同学说这个命令选项不是还有很多吗，为什么其他的不介绍，你可以看到，光是打包【-czf】和解包【-xzf】就已经够复杂得了，再加些其他功能基本就要四个一起写，后面如果有需要会继续讲解
-   当然有兴趣的小伙伴可以深入了解一下。我这里给出一些范例

```
tar -cvf /tmp/etc.tar /etc<==仅打包，不压缩！
tar -zcvf /tmp/etc.tar.gz /etc <==打包后，以 gzip 压缩
tar -jcvf /tmp/etc.tar.bz2 /etc <==打包后，以 bzip2 压缩
-ztvf /tmp/etc.tar.gz <==要查阅该 tar file 内的文件
tar-zxvf/tmp/etc.tar.gz <==可以将压缩档在任何地方解开的
tar-N"2005/06/01"-zcvfhome.tar.gz/home <==在/home当中，比2005/06/01新的文件才备份
复制代码
```

## 二十九、【bc】指令

**功能：进行浮点运算**

-   使用这个指令，你就可以在Linux下直接进行一些计算。只需要敲这个指令就行，它会等待你输入

> ![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c1680da5f815444d8fceac22c7f61499~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

-   下面是它的一些操作

> ![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/29259c33fa174d138c7ef97f707dbe07~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

-   不仅如此，bc指令还可以和我们上面说过的管道一起使用

> ![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5d61c79f51ec43b58e3d30f5c6db37f3~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

___

## 三十、【uname】指令

**语法：uname \[选项\]**  
**功能： uname用来获取电脑和操作系统的相关信息**

-   对这个指令我们讲两个命令选项操作

```
uname -a
uname -r
复制代码
```

> ![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3718ec0658814c43a711fda97c6aad7c~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp) ![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e29c319526bd4c8c85160ffb797a0b87~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

-   然后再拓展一下，我们可以直接去查看当前Linux所使用的商业化发行版

```
cat /etc/redhat-release
复制代码
```

![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/97fddc7541b54ac78f45e602d9600d52~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

## 🌳几个重要的热键

## ⌨ 热键【Tab】

-   **\[Tab\]按键---具有『命令补全』和『档案补齐』的功能**
-   接下来给大家介绍一个热键，叫做【Tab】，什么叫做热键呢，不是很热的键，而是使用很频繁的按键，那这个按键具体该怎么使用呢，我们一起来看看

> ![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c5ea4f9e63c74a189b1fb5678a830115~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

-   Tab热键除了可以看到一些以当前字母开始的命令外，还可以用在命令补全中，例如我们下面的touch创建文件这个命令，若是忘记了，就可以使用【Tab】键去做一个提示

> ![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8a9352ef3b5042fdb8ef62f715cc26c8~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

## ⌨ 热键\[Ctrl\]-c

-   **功能： \[Ctrl\]-c按键---让当前的程序『停掉』**

> ![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/83e79e3080cb406387ca1b3f832d5569~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

-   说到这个【ctrl + c】的话，其实是非常有用的，因为我们平常老是会敲错指令或者进入一些界面出不来，这个时候其实无脑🧠\[Ctrl\]-c即可

## ⌨ 热键\[Ctrl\]-d

-   **\[Ctrl\]-d按键---通常代表着：『键盘输入结束(End Of File, EOF 戒 End OfInput)』的意思；另外，他也可以用来取代exit**

> ![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f9550d1870c641e783418596e0729d68~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

## ⌨ 热键\[Ctrl\]-r

-   **\[Ctrl\]-r按键---在历史命令中进行搜索**

> ![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c02c0c79d70e4baeae8d8eaefdca5e0e~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

## ⌨ 热键\[PageUp\]\[PageDown\]

-   **作用：上下翻阅历史指令**
-   这个看下面视频:movie\_camera:【微信手机端看不到】

\[video(video-uiGwHPP4-1669449964665)(type-csdn)(url-[live.csdn.net/v/embed/257…](https://link.juejin.cn/?target=https%3A%2F%2Flive.csdn.net%2Fv%2Fembed%2F257241)(image-https%3A%2F%2Fvideo-community.csdnimg.cn%2Fvod-84deb4%2F7b0aaacb911f4ae8bd1687d6d3ddbc32%2Fsnapshots%2F51c44f366483446baffcc673a54a1eaf-00002.jpg%3Fauth_key%3D4823044404-0-0-eebe3a23d6b49137c3b156633f8e1ba9)(title- "https://live.csdn.net/v/embed/257241)(image-https://video-community.csdnimg.cn/vod-84deb4/7b0aaacb911f4ae8bd1687d6d3ddbc32/snapshots/51c44f366483446baffcc673a54a1eaf-00002.jpg?auth_key=4823044404-0-0-eebe3a23d6b49137c3b156633f8e1ba9)(title-"))\]

## ⌨ 热键history

-   **作用：获取历史所敲过的所有指令**

> ![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4b9b6e8f9aad4e718d762dffed73c19b~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

## 🌳关机

**语法：shutdown \[选项\] \*\* 常见选项：**\*\*

-   接下去说说Linux中的关机，这个命令的话如果Xshell没有出什么大问题就别用了，了解一下即可。还有一点就是我们在使用云服务器的时候尽量不要将其关机，开着就好了。因为这就是一个线上的真实环境。轻易关机可能会导致文件丢失

```
-h     ：将系统的服务停掉后，立即关机。
-r     ：在将系统的服务停掉之后就重新启动
-t sec ： -t 后面加秒数，亦即『过几秒后关机』的意思
复制代码
```

## 🌳扩展命令

-   以下命令作为扩展。上述我所讲解的三十条指令，只是Linux的一小部分罢了，在之后的文章会继续穿插介绍

```
◆安装和登录命令：login、shutdown、halt、reboot、install、mount、umount、chsh、exit、last；
复制代码
```

```
◆文件处理命令：ﬁle、mkdir、grep、dd、ﬁnd、mv、ls、diﬀ、cat、ln；
复制代码
```

```
◆系统管理相关命令：df、top、free、quota、at、lp、adduser、groupadd、kill、crontab；
复制代码
```

```
◆网络操作命令：ifconﬁg、ip、ping、netstat、telnet、ftp、route、rlogin、rcp、ﬁnger、mail、 nslookup；
复制代码
```

```
◆系统安全相关命令：passwd、su、umask、chgrp、chmod、chown、chattr、sudo ps、who；
复制代码
```

```
◆其它命令：tar、unzip、gunzip、unarj、mtools、man、unendcode、uudecode。
复制代码
```

## 🌳总结与提炼