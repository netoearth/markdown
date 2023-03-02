![](https://img.linux.net.cn/data/attachment/album/202301/01/160400h0mmppwwvxd004bm.jpg)

> Flatpak 软件包的一个鲜为人知的特点是，它允许你对已安装的应用程序进行降级。下面是如何使用它的方法。

从技术上讲，小版本或次要更新是为了解决问题。但是，当某些更新破坏你当前的工作流程时，情况可能会变得更糟。

无论是 Flatpak 包还是 Snap，当出现问题时，一切都会在某个时候崩溃。作为一个沙盒打包方案，它可能不会影响整个系统，但如果你遇到一个让你的应用体验变差的错误，你可能会后悔更新。

比如之前 [Black Box](https://itsfoss.com/blackbox-terminal/)\[1\] 的更新就带来了一些错误，无法选择文字！开发人员现在已经解决了这个问题，但在他们没有解决之前，我降级了那个特定的包以使其正常工作。

所以，如果你想降级特定的 Flatpak 应用，你可以按照本指南进行操作。

**免责声明：** 与安装 Flatpak 不同，你需要 `sudo` 权限才能降级 Flatpak 包。如果你的用户没有该权限，你可以按照我们关于 [如何向用户授予 sudo 访问权限](https://itsfoss.com/add-sudo-user-ubuntu/)\[2\] 的详细指南进行操作。

以下是步骤：

第一步是找到要降级的包的应用 ID。你可以列出已安装的软件包轻松找到它：

```
flatpak list --app
```

![find flatpak package id in linux](https://img.linux.net.cn/data/attachment/album/202301/01/160402zbws11busddlhpuz.png)

_find flatpak package id in linux_

记下要降级的包的应用 ID。

这里，我要降级 Black Box，所以我的应用 ID 将是 `com.raggesilver.BlackBox`。

获得应用 ID 后，你需要列出以前的版本。

你可以按照给定的命令语法做到这点：

```
flatpak remote-info --log flathub <Application ID>
```

![find previous releases in flatpak](https://img.linux.net.cn/data/attachment/album/202301/01/160403so7tjrjrfowr772k.png)

_find previous releases in flatpak_

找到首选的先前版本后，复制如上所示的提交的代码。

执行前两个步骤后，你应该有以下内容：

-   包的应用 ID。
-   首选旧版本的提交代码。

现在，你必须将它们放在以下命令中：

```
sudo flatpak update --commit=<commit_code> <Application ID>
```

当我将 Black Box 降级到以前的版本时，我将使用以下命令：

```
sudo flatpak update --commit=c4ef3f4be655cbe2559451a9ef5977ab28139c54bb5adbd7db812f3482bd0db5 com.raggesilver.BlackBox
```

![downgrade flatpak package in linux](https://img.linux.net.cn/data/attachment/album/202301/01/160404rbd2ed8z8vh8vge4.png)

_downgrade flatpak package in linux_

这就完成了！

要检查你是否已成功降级软件包，你可以列出需要更新的软件包（考虑到其他所有内容都是最新的）。它应该包括你最近降级的软件包的名称：

```
flatpak update
```

![downgrade flatpak package](https://img.linux.net.cn/data/attachment/album/202301/01/160404h347dhcjf264ffed.png)

_downgrade flatpak package_

如你所见，Black Box 已过时，需要更新，这意味着包已成功降级！

在本快速教程中，我解释了如何降级 Flatpak 软件包，希望对你有所帮助。

如果你有任何疑问或建议，请在评论中告诉我。

___

via: [https://itsfoss.com/downgrade-flatpak-packages/](https://itsfoss.com/downgrade-flatpak-packages/)

作者：[Sagar Sharma](https://itsfoss.com/author/sagar/)\[3\] 选题：[lkxed](https://github.com/lkxed)\[4\] 译者：[geekpi](https://github.com/geekpi)\[5\] 校对：[wxy](https://github.com/wxy)\[6\]

本文由 [LCTT](https://github.com/LCTT/TranslateProject)\[7\] 原创编译，[Linux中国](https://linux.cn/article-15402-1.html?utm_source=rss&utm_medium=rss)\[8\] 荣誉推出

___

\[1\]: https://itsfoss.com/blackbox-terminal/  
\[2\]: https://itsfoss.com/add-sudo-user-ubuntu/  
\[3\]: https://itsfoss.com/author/sagar/  
\[4\]: https://github.com/lkxed  
\[5\]: https://github.com/geekpi  
\[6\]: https://github.com/wxy  
\[7\]: https://github.com/LCTT/TranslateProject  
\[8\]: https://linux.cn/article-15402-1.html?utm\_source=rss&utm\_medium=rss