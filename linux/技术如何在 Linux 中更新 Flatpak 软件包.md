![](https://img.linux.net.cn/data/attachment/album/202301/03/154131lop17rnnrkiprkl7.jpg)

我相信几乎所有的 Linux 用户都会保持他们系统的更新。

但这种更新通常是针对默认的 [包管理器](https://itsfoss.com/package-manager/)\[1\]。例如，[更新 Ubuntu](https://itsfoss.com/update-ubuntu/)\[2\] 往往意味着更新所有的 APT 软件包。

然而，还有其他的打包格式，如 Snap 和 Flatpak。Snap 应用程序会自动更新，但 Flatpak 不会。

那么你如何更新 Flatpak 软件包呢？好吧，你可以用这个命令来更新所有已安装和可更新的 Flatpak 包：

```
flatpak update
```

这很简单。但让我再讨论一下关于更新 Flatpak 的一些事情，比如说：

-   更新所有或特定的 Flatpak 包
-   通过软件中心更新 Flatpak 包

让我们先从终端的方法开始。

首先让我从最实用的方法开始，你也应该从这个方法开始。

更新现有的 Flatpak 包的整个目录是很容易的。

输入给定的命令，就可以得到过期包的列表：

```
flatpak update
```

![update flatpak packages in linux](https://img.linux.net.cn/data/attachment/album/202301/03/154134vwxbzv9non79owx5.png)

_update flatpak packages in linux_

你只需输入 `Y` 并按下回车键，就能搞定每一个更新。

要更新特定的软件包，你需要可以更新的软件包的列表。你用的是你之前看到的那个命令。

```
flatpak update
```

![update flatpak packages in linux](https://img.linux.net.cn/data/attachment/album/202301/03/154134vwxbzv9non79owx5.png)

_update flatpak packages in linux_

从输出中复制你要更新的软件包的名称。在以下命令中使用软件包的名称：

```
flatpak update package_name
```

例如，如果你想更新 Telegram，下面的命令可以完成这项工作：

```
flatpak update org.telegram.desktop
```

![update specific package in flatpak](https://img.linux.net.cn/data/attachment/album/202301/03/154134nk1u5l9o1kmw1him.png)

_update specific package in flatpak_

这就完成了。

有 Flatpak 内置支持的发行版会在软件中心提供 Flatpak 应用的更新。Fedora 和 Linux Mint 就是这样的发行版。

但如果你使用的是 Ubuntu，你就需要在 GNOME 软件中心添加 Flatpak 支持：

```
sudo apt install gnome-software-plugin-flatpak
```

完成后，你将在 Ubuntu 中拥有两个软件中心。这是因为默认的软件中心不是 GNOME 的，而是 Snap Store。

从系统菜单中打开这个新的软件中心：

![open software center in ubuntu](https://img.linux.net.cn/data/attachment/album/202301/03/154135z7600a3okhrnj3ck.png)

_open software center in ubuntu_

进入“更新Update”页面，你会发现过时的软件包列表。这包括 APT 和 Flatpak 软件包。

![update flatpak from software center](https://img.linux.net.cn/data/attachment/album/202301/03/154135s1kbbeex4jk7jvji.png)

_update flatpak from software center_

在这里，你可以一次更新所有的软件包，或者你可以有选择地更新什么。

许多 Linux 桌面用户往往忘记更新 Flatpak 软件包，因为它们不包括在定期的系统更新中。

由于 Flatpak 是一个沙盒式的打包解决方案，你可能不会面临任何与过时的软件包有关的问题，但你肯定会错过新的功能和修复。

这就是为什么我建议每隔几周运行一次 Flatpak 更新命令。

我希望你喜欢这个快速的 Flatpak 小技巧。

___

via: [https://itsfoss.com/update-flatpak/](https://itsfoss.com/update-flatpak/)

作者：[Sagar Sharma](https://itsfoss.com/author/sagar/)\[3\] 选题：[lkxed](https://github.com/lkxed)\[4\] 译者：[geekpi](https://github.com/geekpi)\[5\] 校对：[wxy](https://github.com/wxy)\[6\]

本文由 [LCTT](https://github.com/LCTT/TranslateProject)\[7\] 原创编译，[Linux中国](https://linux.cn/article-15408-1.html?utm_source=rss&utm_medium=rss)\[8\] 荣誉推出

___

\[1\]: https://itsfoss.com/package-manager/  
\[2\]: https://itsfoss.com/update-ubuntu/  
\[3\]: https://itsfoss.com/author/sagar/  
\[4\]: https://github.com/lkxed  
\[5\]: https://github.com/geekpi  
\[6\]: https://github.com/wxy  
\[7\]: https://github.com/LCTT/TranslateProject  
\[8\]: https://linux.cn/article-15408-1.html?utm\_source=rss&utm\_medium=rss