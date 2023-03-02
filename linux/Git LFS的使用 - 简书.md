## Git LFS的使用

[![](https://cdn2.jianshu.io/assets/default_avatar/15-a7ac401939dd4df837e3bbf82abaa2a8.jpg)](https://www.jianshu.com/u/f442407d3901)

62017.12.26 23:56:39字数 495阅读 224,114

[参考资料](https://link.jianshu.com/?t=https%3A%2F%2Fcoding.net%2Fhelp%2Fdoc%2Fgit%2Fgit-lfs.html)

Git LFS 是 Github 开发的一个 Git 的扩展，用于实现 Git 对大文件的支持

  

![](https://upload-images.jianshu.io/upload_images/1059995-670f795346b86292?imageMogr2/auto-orient/strip|imageView2/2/w/1000/format/webp)

git-lfs

### 使用目的

在游戏开发过程中,设计资源占用了很大一部分空间. 像png,psd等文件是二进制(blob)的,体积也很庞大.  
但git的diff/patch等是基于文件行的.对于二进制文件来说. git需要存储每次commit的改动.  
每次当二进制文件修改,发生变化的时候. 都会产生额外的提交量.导致clone和pull的数据量大增.在线仓库的体积也会迅速增长.

![](https://upload-images.jianshu.io/upload_images/1059995-c9ddfd907277e8df.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

git-grow

LFS(Large File Storage) 就是为了解决这一问题而产生的工具.  
它将你所标记的大文件保存至另外的仓库,而在主仓库仅保留其轻量级指针.  
那么在你检出版本时,根据指针的变化情况下更新对应的大文件.而不是在本地保存所有版本的大文件

![](https://upload-images.jianshu.io/upload_images/1059995-7e78b1cc5ceb6c1f.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

store

### 安装

> 注意：安装 Git LFS 需要 Git 的版本不低于 1.8.5

#### _Linux_

1.  `curl -s https://packagecloud.io/install/repositories/github/git-lfs/script.deb.sh | sudo bash`  
    \`
2.  `sudo apt-get install git-lfs`
3.  `git lfs install`

#### _Mac_

1.  安装HomeBrew `/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"`
2.  `brew install git-lfs`
3.  `git lfs install`

#### _Windows_

1.  下载安装 [windows installer](https://link.jianshu.com/?t=https%3A%2F%2Fgithub.com%2Fgithub%2Fgit-lfs%2Freleases)
2.  运行 windows installer
3.  在命令行执行 `git lfs install`

### 使用

1.  执行 `git lfs install` 开启lfs功能
2.  使用 `git lfs track` 命令进行大文件追踪 例如`git lfs track "*.png"` 追踪所有后缀为png的文件
3.  使用 `git lfs track` 查看现有的文件追踪模式
4.  提交代码需要将`gitattributes`文件提交至仓库. 它保存了文件的追踪记录
5.  提交后运行`git lfs ls-files` 可以显示当前跟踪的文件列表
6.  将代码 push 到远程仓库后，LFS 跟踪的文件会以『Git LFS』的形式显示:
7.  clone 时 使用'git clone' 或 `git lfs clone`均可

更多精彩内容，就在简书APP

![](https://upload.jianshu.io/images/js-qrc.png)

"小礼物走一走，来简书关注我"

还没有人赞赏，支持一下

[![  ](https://cdn2.jianshu.io/assets/default_avatar/15-a7ac401939dd4df837e3bbf82abaa2a8.jpg)](https://www.jianshu.com/u/f442407d3901)

总资产5共写了6838字获得114个赞共20个粉丝

-   和清纯老婆闪婚的20天 我撞伤了一个老伯，不但没担责任，他还要把女儿嫁给我。 没想到婚礼当天，曾被我捉奸在床的前女...
    
    [![](https://upload.jianshu.io/users/upload_avatars/4790772/388e473c-fe2f-40e0-9301-e357ae8f1b41.jpeg)茶点故事](https://www.jianshu.com/u/0f438ff0a55f)阅读 343评论 2赞 4
    
-   儿媳妇怀着八个月的胎，我那脑子缺根筋的儿子带回来个妖艳贱货。 他一纸离婚协议甩自己原配面前：「离婚！」 真就我惯着...
    
    [![](https://upload.jianshu.io/users/upload_avatars/4790772/388e473c-fe2f-40e0-9301-e357ae8f1b41.jpeg)茶点故事](https://www.jianshu.com/u/0f438ff0a55f)阅读 147评论 0赞 1
    
-   我被篮球砸伤了脚踝，被送到医院后，竟然遇到我老公全家，陪另一个女人产检。 平时对我的冷言冷语的小姑子吴彤彤挽着小三...
    
    [![](https://upload.jianshu.io/users/upload_avatars/4790772/388e473c-fe2f-40e0-9301-e357ae8f1b41.jpeg)茶点故事](https://www.jianshu.com/u/0f438ff0a55f)阅读 122评论 0赞 1
    
-   谈了三年的男朋友向我求婚的那天，他飞机失事的白月光竟然活着回来了。 我跟着他一路跑到楼梯口，看着他们深情相拥在一起...
    
    [![](https://upload-images.jianshu.io/upload_images/15878160-eb4fb54f8ac7aad3.jpg?imageMogr2/auto-orient/strip|imageView2/1/w/150/h/120/format/webp)](https://www.jianshu.com/p/0fd162ed0426)
-   我是他体贴入微，乖巧懂事的金丝雀。 上位后，当着满座宾朋，我播放了他曾经囚禁、霸凌我，还逼疯了我妈妈视频。 1 我...
    
    [![](https://upload-images.jianshu.io/upload_images/4790772-d47beae92feda3d4.jpg?imageMogr2/auto-orient/strip|imageView2/1/w/150/h/120/format/webp)](https://www.jianshu.com/p/348696104d87)
-   2006年的一个雪夜，我亲眼看着自己的妹妹倒在血泊中。 她死了，葬在白雪里。 这是她最好的结局。 我手里捏着带血的...
    
    [![](https://upload-images.jianshu.io/upload_images/4790772-80b3fa6281a8af3f.jpeg?imageMogr2/auto-orient/strip|imageView2/1/w/150/h/120/format/webp)](https://www.jianshu.com/p/3a1958255a4a)
-   2009年8月27日夜间，在广东省江门市境内，破获一起电信诈骗组织。 自8月20日开始，江门市警方通过IP查询、侦...
    
    [![](https://upload.jianshu.io/users/upload_avatars/4790772/388e473c-fe2f-40e0-9301-e357ae8f1b41.jpeg)茶点故事](https://www.jianshu.com/u/0f438ff0a55f)阅读 107评论 0赞 1
    
    [![](https://upload-images.jianshu.io/upload_images/4790772-c23438736057a2ca.jpeg?imageMogr2/auto-orient/strip|imageView2/1/w/150/h/120/format/webp)](https://www.jianshu.com/p/3d55645df488)
-   正文 铊”化学符号TI，原子序数为81，是一种伴生元素，被广泛用于电子、军工、化工等各方面。无色无味，易溶于水，最...
    
    [![](https://upload.jianshu.io/users/upload_avatars/4790772/388e473c-fe2f-40e0-9301-e357ae8f1b41.jpeg)茶点故事](https://www.jianshu.com/u/0f438ff0a55f)阅读 141评论 0赞 1
    
-   我的男朋友很帅很帅，温柔体贴，但是他和我闺蜜上了床。 还在陵园给我立了一个碑。 恐怖的是，我男朋友自己的墓碑就在我...
    
    [![](https://upload-images.jianshu.io/upload_images/4790772-72d6c4a051346583.jpg?imageMogr2/auto-orient/strip|imageView2/1/w/150/h/120/format/webp)](https://www.jianshu.com/p/4b29db56d238)
-   1、美女医生进监狱 疫情刚开始的那年，我和护士小白，被紧急调到西城监狱支援防控。 这里是男子监狱，那些囚犯很多都是...
    
-   我在学生会的时候，拉赞助，被合作方领导骚扰。 这事还被人撞见，传了出去。 而我妈听见了这些闲话，气势汹汹地杀来。 ...
    
    [![](https://upload-images.jianshu.io/upload_images/4790772-724f1fd47f4b9afe.jpg?imageMogr2/auto-orient/strip|imageView2/1/w/150/h/120/format/webp)](https://www.jianshu.com/p/6e1afb7777ad)
-   我跟我前夫在商场里面给小孩买衣服，他现在的老婆不停地给他打电话。 前夫没办法，只能撒谎说自己在公司开会。 对方听清...
    
    [![](https://upload-images.jianshu.io/upload_images/4790772-241a1c9fae8951e8.jpg?imageMogr2/auto-orient/strip|imageView2/1/w/150/h/120/format/webp)](https://www.jianshu.com/p/72014d444acc)
-   疫情期间，我和竹马被迫一起居家隔离。 凌晨，我穿着吊带睡衣在厨房遇到了他，四目相对间，我下意识捂着胸口嚎叫了起来。...
    
    [![](https://upload-images.jianshu.io/upload_images/4790772-c72e8766daecf26a.jpeg?imageMogr2/auto-orient/strip|imageView2/1/w/150/h/120/format/webp)](https://www.jianshu.com/p/875b89eba857)
-   周颢程养鱼翻车了。 我和他的另一个女友联合起来报复他。 我上午约见面，她就下午约。 她要口红，我就要粉底液。 周颢...
    
-   人人都说尚书府的草包嫡子修了几辈子的福气，才能尚了最受宠的昭宁公主。 只可惜公主虽容貌倾城，却性情淡漠，不敬公婆，...
    
-   《网恋社死现场》 一、 面基当天，我把谈了三个月的网恋对象拉黑了。 事情是这样的。 那天我刚上完早课，叼了个冰棒走...
    
-   送上门的女学生 文/六酒 我在一所大专当英语老师的过程中，为了能够顺利毕业，我班里一个清纯动人的女学生找上我，说愿...
    
    [![](https://upload-images.jianshu.io/upload_images/4790772-0f0cdf03185827a0.png?imageMogr2/auto-orient/strip|imageView2/1/w/150/h/120/format/webp)](https://www.jianshu.com/p/bedff5b474cf)
-   作者：猫打滚儿 文案： 八岁那年，我被我爸当成诬陷村长的诱饵，我最崇拜的叔叔是他的同谋，那夜之后，这三个男人都失踪...
    
-   皇帝的白月光自尽了，我却瞧着他一点也不伤心。 那日我去见贵妃最后一面，却惊觉她同我长得十分相似。 回宫后我的头就开...
    
-   谁能想到有朝一日，逼宫这种事会发生在我身边。 被逼走的是我亲妈，始作俑者是我亲小姨。 为了争得我的抚养权，母亲放弃...
    
    [![](https://upload-images.jianshu.io/upload_images/4790772-ef416415d0dee221.jpg?imageMogr2/auto-orient/strip|imageView2/1/w/150/h/120/format/webp)](https://www.jianshu.com/p/e00864bde0c1)
-   我被心上人灌下晕药，送到了新科状元的床上。 一年后的雨水，我被人毒死，扔进枯井之中。 死前，我竟然听到了撕心裂肺的...
    

### 被以下专题收入，发现更多相似内容

### 推荐阅读[更多精彩内容](https://www.jianshu.com/)

-   1.git的安装 1.1 在Windows上安装Git msysgit是Windows版的Git，从https:/...
    
    [![](https://upload.jianshu.io/users/upload_avatars/4889822/4b23a204-6949-480b-9483-ca64f5b5e501.jpg?imageMogr2/auto-orient/strip|imageView2/1/w/48/h/48/format/webp)落魂灬](https://www.jianshu.com/u/9efe5acbf3d0)阅读 12,209评论 4赞 54
    
    [![](https://upload-images.jianshu.io/upload_images/4889822-566528249b3593b9?imageMogr2/auto-orient/strip|imageView2/1/w/300/h/240/format/webp)](https://www.jianshu.com/p/29b392fba2b9)
-   本文为 Git教程的学习笔记，教程源自廖雪峰的博客。这是一个由浅入深，学完后能立刻上手的Git教程。另，附上另一本...
    
    [![](https://upload.jianshu.io/users/upload_avatars/763193/1d19245e4d82.jpeg?imageMogr2/auto-orient/strip|imageView2/1/w/48/h/48/format/webp)七弦桐语](https://www.jianshu.com/u/9c881a52bca6)阅读 5,832评论 5赞 47
    
    [![](https://upload-images.jianshu.io/upload_images/763193-3ebabd44cfe74d13.jpg?imageMogr2/auto-orient/strip|imageView2/1/w/300/h/240/format/webp)](https://www.jianshu.com/p/382abb427ca9)
-   第一章 安装Git工具 下载GitHub for Windows,直接点击安装，安装完成后，可以看到“Git Sh...
    
    [![](https://cdn2.jianshu.io/assets/default_avatar/7-0993d41a595d6ab6ef17b19496eb2f21.jpg)不圆的石头](https://www.jianshu.com/u/cddadfa5d25e)阅读 11,587评论 5赞 63
    
    [![](https://upload-images.jianshu.io/upload_images/3958688-32b2b525c5ef4919.png?imageMogr2/auto-orient/strip|imageView2/1/w/300/h/240/format/webp)](https://www.jianshu.com/p/c07ccdfba068)
-   第二章 选择器 1、p.warning （没有空格）其class属性包含warning的p段落 区别于 p .wa...
    
    [![](https://cdn2.jianshu.io/assets/default_avatar/14-0651acff782e7a18653d7530d6b27661.jpg)风色透明](https://www.jianshu.com/u/b08711611464)阅读 362评论 0赞 1
    
-   今夜，为了给菜哥践行，兄弟们一起到水世界来玩，吃完东西玩了会，感觉无趣，于是就玩了下王者，菜哥呢就在玩手机斗地主，...
    
    [![](https://upload-images.jianshu.io/upload_images/8915265-a1a14f7d5feeabd3.jpg?imageMogr2/auto-orient/strip|imageView2/1/w/300/h/240/format/webp)](https://www.jianshu.com/p/fe98a4f8c84f)