   

[![](https://p3-passport.byteimg.com/img/user-avatar/da0521c2de64e55cd974f2c7aa6d5e16~100x100.awebp)](https://juejin.cn/user/3439705793762375)

2022年10月07日 23:59 ·  阅读 409

## 一.引言

Github是日常开发中绕不开的问题，我们日常使用github进行多人项目开发是否标准？是否还在一顿梭哈git pull,git push和git merge?今天我将完整的讲解一套标准的github极简工作流程，带大家看看标准的github提交姿势，最后带领各位爷们区分merge和rebase的区别。助力各位平步青云，一步日天。

## 二.成员项目初始化

**1.githu建立一个新项目**

首先我们模拟项目管理员在github建立一个新的项目仓库，为了简单模拟多人开发，下面我将列举两位同学合作开发的过程。 ![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/48283588113e4d5dbc95f1b4c630e8bb~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

**2\. 员工zs和ls初始项目环境**

使用两个文件夹分别模拟员工zs和ls的电脑，员工进入项目组后会依次进行如下操作

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f3c2a4df75c84851a991ded801e495c7~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

1.  clone项目到自己本地（仅列举zs同学，ls同上）
    
    ```
     git clone xxxx (xxx代表项目远程地址)
     
    复制代码
    ```
    
2.  建立自己项目分支，此处假设以个人名称作为自己的开发分支
    
    ```
     //git checkout -b xxx(xxx代表分支名称)
    
     git checkout -b feat/zs
    
    复制代码
    ```
    

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/604c20eec832494ebe4a0645dc5af7e0~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

新建分支后，可以发现已经自动切换到新建的分支上了。

3.  将自己新建的分支同步到远程github上
    
    ```
    //git push origin xxx(分支名称)
    git push origin feat/zs
    
    复制代码
    ```
    

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2154858603c8490885b3c060e989a34e~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

可以发现zs同学新建立的分支已经成功同步到远端了。

## 三.成员开始开发

zs在自己分支干完活后，会执行如下操作。

1.写一个小功能/模块后，将自己的代码发布到远端github仓库上，保持同步。（千万不要写一大堆代码再提交，不然最后下班的肯定是你），记得是同步到自己分支上。

```
git add .
git commit -m "feat: zs干完了"
git push origin feat/zs
复制代码
```

2.在自己分支开发时如果发现main分支代码更新了（一般会通知），我们需要去切换到main分支，拉取main分支最新的代码，然后回到自己分支上，将最新的main分支代码合并到你的个人分支上

```
//切换主分支
git switch main
//获取主分支最新的代码
git pull origin main
//切换自己分支
git switch feat/zs
//将main分支最新代码合并到自己分支上，有冲突解决冲突
git merge main
//将自己分支代码提交到远程github
git push origin feat/zs
复制代码
```

经过上面1和2的循环操作，你发现自己干完了，可以摸鱼了。（摸你妈个头，代码pr?）接下来就是最后一步了，我们需要向主分支提交pull request。

![09371ded35d08ef40b048d00d6ca283.jpg](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/216fb19d3d5b494090794102471f2e70~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

3.  你的老大摸完鱼后，发现你的消息通知，心里念叨这个逼代码写的这么快，下次给你多派点活。然后骂骂咧咧的在你的分支测试没有问题后，将你的pull request合并到主分支上，如下图有三种合并方式，这里出现了两个词语 rebase和squash。我们先不谈，后面会讲解，假设领导直接使用第一种常规方式merge将你的分支合并到了主分支上。

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/47d2436ed15d4def9d1e5fc65eab50a6~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

4.  github结构图

其中白线是主分支，核心。蓝色是zs同学开发路线，绿色是ls同学开发路线，他们都采用的是merge的方式进行项目的合并，项目老大可以清晰的看到每个同学每次提交的功能点。但是这仅仅是两个同学，如果是5位？10位？这个图看起来就有点费老大了。因此采用merge的方式适合于项目分支不多，参与人数少的开发实践。当然这个总结也比较片面，毕竟使用push,pull,merge梭哈的人数占比还是相当多的。

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a4a01fec8c004847bf1a3f32597af4ab~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

## 四.git rebase和squash

上面我们遗留了一个问题，rebase和squash。这两个吊毛就是今天的主角，也是大肠比较正规的代码管理手段。相当一部分大肠面试官也爱用merge和rebase的问题来刁难我们这些吊毛。说着说着我就留下了眼泪，大家学会了一定不要刁难同行啊。不好意思岔开话题了，大伙再忍忍，马上就出来了（嘿嘿嘿）。

> 此处仍然是zs和ls两个同学啊，我们直接跳过初始化，假设他们已经完成了上面项目初始化，各自创建好了分支。下面的例子仍然以zs同学为例子进行

1.zs同学在自己分支写了一个功能,提交到自己分支，同步github

```
git add .
git commit -m "feat: 功能1"
git push origin feat/zs
复制代码
```

2\. zs同学准备继续写新功能时，发下老大通知大家更新下main分支最新代码，这个时候老大要求必须使用rebase

```
    //切换主分支
    git switch main
    //拉取main最新代码
    git pull origin main
    //切换自己分支
    git switch feat/zs
    //将main合并到自己zs分支，继续开发
    git rebase main
    
    
复制代码
```

此处rebase和merge大家先理解成一个功能，都是将目标分支代码合并到指定分支上。rebase和merge不同的是

merge是将main分支直接串联到zs分支上，形成一次提交，而rebase是告诉main分支，你别和我串联一起，你把你最新更新的部分直接拷贝给我，我将它们整合一起放在我的分支节点上。可能大家疑惑这不是一样？你在这绕圈放屁？不都是将main更新的部分合并到了zs分支？别急嘛，后面看了结构图大家就懂了。

3.zs同学持续进行1和2操作。完成自己开发后在github上向老大提出pull request请求，老大看到后选择了squash and merge选项将zs同学提交的代码合并到了主分支上。

此处squash and merge和merge有何不同？其主要作用就是压缩的意思，将多次commit代码提交节点压缩成一次。由于我们每次commit 提交的代码并不都是有意义的，可能大多数是添加一了一行代码，修改了一个字符就提交一次commit。对我们个人来说可能用来欲盖弥彰告诉老大我今天commit好多次，我是个好员工。但是从项目管理和维护角度看，你就是在堆垃圾。因此我们需要将多次commit合并成一次核心的提交上去就行。

4.  github结构图

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d8d4cbf4fae249b68f5489fd07d7de07~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

这个图很好的解释了merge和rebase的区别，一个是直接串联main分支合并，一个是单独将main分支代码拿过来合并。并且经过squash后，项目commit也变得十分清晰。

所以说，当项目分支多，功能复杂时，采用rebase和squash and merge是十分有必要的选择。

## 总结

上面是我的个人小小总结，由于本人还在学校，并未参加过多人合作公司项目，只参加过小组项目（代码分支管理混乱）。这篇文章总结目的是规范我的开发设计，同时也希望带给大家帮助。如果您感觉有用，解答了您的疑惑，那麻烦免费的举报点一点啊。

相关课程

![cover](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1646d797b8c34de6b4d323657f430635~tplv-k3u1fbpfcp-zoom-mark-crop-v2:0:0:160:224.awebp)

![cover](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/22cb14cfcc6047419eb4e1bfea62a57d~tplv-k3u1fbpfcp-zoom-mark-crop-v2:0:0:160:224.awebp)

![](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/00ba359ecd0075e59ffbc3d810af551d.svg) 21

![](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/3d482c7a948bac826e155953b2a28a9e.svg) 收藏