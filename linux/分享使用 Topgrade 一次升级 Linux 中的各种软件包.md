![](https://img.linux.net.cn/data/attachment/album/202210/12/152118mo5r6pohnn4o5h56.jpg)

更新 Linux 系统并没有那么复杂，不是吗？毕竟，要更新 Ubuntu 之类的发行版，你只需要使用 `apt update` 和 `apt upgrade` 就行。

如果所有的包都是通过一个包管理器安装的，就会是这样。

但现在情况不再如此。你有经典的 apt/dnf/pacman，还有 Snap、Flatpak、Appimages。不止于此，你还可以使用 PIP（用于 Python）和 Cargo（用于 Rust）安装应用。

使用 Node？ NPM 包需要单独更新。Oh My Zsh？需要单独更新。[Vim 中的插件](https://linuxhandbook.com/install-vim-plugins/)\[1\]、Atom 等也可能不被 apt/dnf/pacman 覆盖。

你现在看到问题了吗？这就是名为 Topgrade 的新工具旨在解决的问题。

[Topgrade](https://github.com/r-darwish/topgrade)\[2\] 是一个 CLI 程序，它会检测你使用的工具，然后运行适当的命令来更新它们。

![Topgrade disable system](https://img.linux.net.cn/data/attachment/album/202210/12/152120q9kktzhly9tgjmb2.png)

_Topgrade disable system_

除了通常的 Linux 包管理器，它还可以检测和更新 Brew、Cargo、PIP、Pihole、Vim 和 Emacs 插件、R 软件包等。你可以在 [维基页面](https://github.com/r-darwish/topgrade/wiki/Step-list)\[3\] 上查看支持的包列表。

-   能够更新来自不同的包管理器的软件包，**包括固件**！
-   你可以如何控制更新包。
-   高度可定制。
-   甚至能够在更新包之前进行概览。

所以不要浪费任何时间，让我们跳到安装。

安装过程非常简单，因为我将使用 Cargo 包管理器。

我们已经有了 [详细指南，其中包含设置 Cargo 包管理器的多种方法](https://itsfoss.com/install-rust-cargo-ubuntu-linux/)\[4\]。所以我将在我的示例中使用 Ubuntu 来快速完成。

因此，让我们以最少方式安装依赖项以及 Cargo：

```
sudo apt install cargo libssl-dev pkg-config
```

安装 Cargo 后，使用给定的命令安装 Topgrade：

```
cargo install topgrade
```

它会抛出一个警告：

![cargo error](https://img.linux.net.cn/data/attachment/album/202210/12/152120naa5q222rrp29n7p.png)

_cargo error_

你只需添加 `cargo` 路径即可运行二进制文件。这可以通过给定的命令来完成，你需要使用你的用户名替换 `sagar`：

```
echo 'export PATH=$PATH:/home/sagar/.cargo/bin' >> /home/sagar/.bashrc
```

现在，重启系统，Topgrade 就可以使用了。但是等等，我们需要安装另一个包来更新 Cargo 以获取最新的包。

```
cargo install cargo-update
```

这样我们完成了安装。

使用 Topgrade 非常简单。使用一个命令，就是这样：

```
topgrade
```

> 此处应有视频，地址： [https://img.linux.net.cn/static/video/topgrade.mp4](https://img.linux.net.cn/static/video/topgrade.mp4)

但这不会给你除了系统包之外的任何控制，但正如我所提到的，你可以将不想更新的仓库列入黑名单。

假设我想排除 Snap 和从默认包管理器下载的包，所以我的命令是：

```
topgrade --disable snap system
```

![Topgrade disable snap system](https://img.linux.net.cn/data/attachment/album/202210/12/152121t3ivtpj5z3p8w3w5.png)

_Topgrade disable snap system_

要进行永久更改，你必须在其配置文件中进行一些更改，这些更改可以通过给定的命令访问：

```
topgrade --edit-config
```

对于此示例，我排除了 Snap 和默认系统仓库：

![configuring Topgrade](https://img.linux.net.cn/data/attachment/album/202210/12/152121hbh8wv114eekm0k4.png)

_configuring Topgrade_

评估将要更新的过时软件包总是一个好主意，我从 Topgrade 的整个目录中找到了这个最有用的选项。

你只需使用带有 `-n` 选项的 `topgrade` 命令，它就会生成过期软件包的摘要。

```
topgrade -n
```

![summery of Topgrade](https://img.linux.net.cn/data/attachment/album/202210/12/152121m29c42zm7y77721h.png)

_summery of Topgrade_

检查需要更新的软件包的一种简洁方法。

在使用 Topgrade 几周后，它成为了我的 Linux 武器库中不可或缺的一部分。 像大多数其他 Linux 用户一样，我只是通过我的默认包管理器更新包。 Python 和 Rust 包被完全忽略了。 感谢 Topgrade，我的系统现在完全更新了。

我知道这不是每个人都想使用的工具。那你呢？愿意试一试吗？

___

via: [https://itsfoss.com/topgrade/](https://itsfoss.com/topgrade/)

作者：[Sagar Sharma](https://itsfoss.com/author/sagar/)\[5\] 选题：[lkxed](https://github.com/lkxed)\[6\] 译者：[geekpi](https://github.com/geekpi)\[7\] 校对：[wxy](https://github.com/wxy)\[8\]

本文由 [LCTT](https://github.com/LCTT/TranslateProject)\[9\] 原创编译，[Linux中国](https://linux.cn/article-15134-1.html?utm_source=rss&utm_medium=rss)\[10\] 荣誉推出

___

\[1\]: https://linuxhandbook.com/install-vim-plugins/  
\[2\]: https://github.com/r-darwish/topgrade  
\[3\]: https://github.com/r-darwish/topgrade/wiki/Step-list  
\[4\]: https://itsfoss.com/install-rust-cargo-ubuntu-linux/  
\[5\]: https://itsfoss.com/author/sagar/  
\[6\]: https://github.com/lkxed  
\[7\]: https://github.com/geekpi  
\[8\]: https://github.com/wxy  
\[9\]: https://github.com/LCTT/TranslateProject  
\[10\]: https://linux.cn/article-15134-1.html?utm\_source=rss&utm\_medium=rss