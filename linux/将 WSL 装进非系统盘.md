一日一技 | 想体验 WSL 但系统盘不够大？简单几步就可以将 WSL 装进非系统盘

在上一篇关于 WSL 功能的文章的评论中，有读者提到了安装 WSL 到非系统盘是一件困难的事情。

**关联阅读：**[《为 WSL 配置这些新功能，不用虚拟机也能体验完整 Linux》](https://sspai.com/post/74167)

确实，现在很多人系统盘都比较小，一方面是因为 SSD 容量相对较小，另一方面则是系统盘往往会被各种应用程序的数据文件占据掉很多空间（没错就是在说你小而美）。而在 Windows 系统的默认情况下，WSL 2 会被安装到系统盘中，这就会让我们本来就不富裕的系统盘空间雪上加霜，所以只能退而求其次不用 WSL 2了。

但爱折腾的我怎么又能放弃 WSL 这个利器呢？经过我的一番研究，发现无论是 WSL 1 还是 WSL 2 都可以把它们安装到非系统盘里，如果你也有这样的困扰不妨跟着一起试试吧。

WSL 经历了两个大版本，分别称为 WSL1 和 WSL2。前者是通过兼容层方式转译系统调用，而后者通过虚拟机的方式运行完整的 Linux 内核。

将 WSL 1 安装到非系统盘的步骤很简单：首先[前往这里](https://store.rg-adguard.net/)下载 WSL 1 的 Linux 发行版安装包，将后缀名改为 zip 并进行解压，解压后可得到名为 `install.tar.gz` 的文件。

![](https://cdn.sspai.com/2022/07/22/article/5c8ff014d99044f5b25a17f7fb6c08a7?imageView2/2/w/1120/q/90/interlace/1/ignore-error/1)

接下来，前往[这里](https://github.com/DDoSolitary/LxRunOffline/releases)下载并解压缩 lxrunoffline 软件；接下来打开终端以后，输入以下命令进行安装 WSL 1 到非系统盘：

```
lxrunoffline i -n <WSL名称> -d <安装路径> -f <安装包路径>.tar.gz
```

进度条走完以后，WSL 1 就安装到非系统盘中了。使用 LxRunOffline 新安装的 WSL 1 默认是以 root 用户登录。

但是，WSL2 中分离了入口文件和磁盘存储文件，并不随着发行版安装包分发。当无法找到 ubuntu.exe 入口程序的时候，就无法进行下一步了。而一个正确安装的 WSL2 环境，Linux 的根系统文件树都在一个 ext4.vhdx 的虚拟磁盘中。

![](https://cdn.sspai.com/2022/07/22/article/62633e0d6dcb3a131b5e8174d1dffdfe?imageView2/2/w/1120/q/90/interlace/1/ignore-error/1)

除了 ext4.vhdx 都是压缩包内的文件

## 兼容 WSL1 和 WSL2 的新方法

好消息是，WSL 的工具集也在不断更新，现在官方的 wsl.exe 工具已经能够支持导入导出 Linux 发行版了。我们可以通过这个功能来实现将 WSL2 迁移到别的系统盘符上。

假设你已经安装好了 WSL2 的环境，如果还没安装可以参考[《为 WSL 配置这些新功能，不用虚拟机也能体验完整 Linux》](https://sspai.com/post/74167)。

首先要确认我们电脑上安装了什么发行版，主要是确认具体的发行版在电脑上的名称，通过 `wsl.exe --list` 列出即可。我这里的具体名字为「Ubuntu-20.04」。

```
PS C:\Users\someone> wsl.exe --list
Windows Subsystem for Linux Distributions:
Ubuntu-20.04 (Default)
```

接下来，我们首先需要导出该发行版到一个压缩包中；再反注册该发行版；接着再一个新的安装位置重新导入该发行版。接下来用 D 盘来做一个具体的例子，假设我首先创建了一个空白的路径 `D:\WSL\Ubuntu`，然后执行：

```
PS C:\Users\someone> cd D:\WSL
PS D:\WSL> wsl.exe --export Ubuntu-20.04 Ubuntu-20.04.tar
PS D:\WSL> wsl.exe --unregister Ubuntu-20.04
PS D:\WSL> wsl.exe --import Ubuntu-20.04 D:\WSL\Ubuntu Ubuntu-20.04.tar
```

导入之后，wsl.exe 工具会自动注册导入发行版的位置，可以通过 `wsl.exe --list` 命令再次查看。这时我们就已经将 WSL 2 安装到了 `D:\WSL\Ubuntu` 中，这个方法同样对 WSL 1 有效。

如果你希望使用 `wsl.exe` 命令直接进入刚刚转移到 D 盘的那个开发版，那么`wsl.exe --set-default Ubuntu-20.04`可以帮助到你。但我个人不喜欢这个指令，因为进入 WSL2 以后，终端会默认切换到当前 Windows 路径下，还需要再多一步手动切换到 WSL 2 的 home 目录中。我目前使用的办法是通过补全功能敲 `ubuntu2004.exe` 命令直接进入 WSL 2 的 home 目录。

![](https://cdn.sspai.com/2022/07/22/article/652934c64cfb222b440e689f83916bb1?imageView2/2/w/1120/q/40/interlace/1/ignore-error/1)

两种进入 WSL2 的命令方式

`ubuntu2004.exe` 命令可以直接进入 WSL 2 的 home 目录的原因其实很简单，因为它并不会跟随发行版安装包分发。在仔细查找后发现，它会出现在 AppData 中，但文件本身不大，所以我们不用动它。

```
PS D:\WSL> Get-Command ubuntu2004.exe

CommandType     Name                                               Version    Source
-----------     ----                                               -------    ------
Application     ubuntu2004.exe                                     0.0.0.0    C:\Users\someone\AppData\Local\Microsoft\WindowsApps\ubuntu2004.exe
```

然后就是常规的 Linux 命令行配置和捣鼓了，少数派上有很多文章可供参考，可以自行搜索，希望你也可以来试一试 Linux 这个操作系统。

\> 下载 [少数派 2.0 客户端](https://sspai.com/page/client)、关注 [少数派公众号](https://sspai.com/s/J71e)，解锁全新阅读体验 📰 

\> 实用、好用的 [正版软件](https://sspai.com/mall)，少数派为你呈现 🚀

© 本文著作权归作者所有，并授权少数派独家使用，未经少数派许可，不得转载使用。

[![正弦定理](https://cdn.sspai.com/article/6e5b5ff3-b2be-228c-03ec-9e61f3e9e18e.jpeg?imageMogr2/auto-orient/quality/95/thumbnail/!84x84r/gravity/Center/crop/84x84/interlace/1)](https://sspai.com/u/wsine/updates)

用写作来摸鱼。 🔗：https://blog.wsine.top/