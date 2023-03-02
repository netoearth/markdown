**编注：**

本文是少数派 [2022 年度征文活动](https://sspai.com/post/77562)的入围文章。本文仅代表作者本人观点，少数派对标题和排版略作调整。

今年我们采用了更加依赖用户反馈数据的奖金结算机制，充电、收藏和阅读量都将不同程度地影响文章的最终排名与稿酬倍率。如果你喜欢这篇文章或内容对你帮助，请尽量通过充电、收藏或评论等方式表达你的支持与赞赏。

___

## 前言

说起电脑操作系统，可能大部分人印象中无外乎是 Windows 和 macOS 两种，很少有人会了解 Linux 是什么，对 Linux 有些了解的人，也基本都觉得 Linux 是服务器专用系统，只有程序员在用，几乎没有人会将其作为个人电脑的操作系统。

然而事实上，还是会有那么一小撮人会将 Linux 系统作为个人电脑的操作系统，其中很多也并不是程序员，而我就是其中一员，最初我只是一个电脑小白，一次偶然的机会接触到了 Linux，起初只是好奇尝鲜，然后便一发不可收拾。

事情要从高考完的那年暑假讲起，马上要上大学的我，准备选购一台属于我自己的笔记本电脑。虽然父母在电脑选购方面没有过多干预，躲过了电脑城的毒手，但因为当时我自认为不玩游戏，而且预算有限，便打算买一台便宜些的轻薄本。年少无知的我，在当时 AMD 锐龙系列 CPU 还未腾飞，轻薄本市场也未开始卷性能的年代，入手了一台 AMD CPU 的轻薄本，现在想想多少有点冤种，不过也多亏了 AMD 对于自家硬件驱动的开放性，让我在安装 Linux 系统的过程中没碰到太多挫折，这些年这台笔记本也没出现过大问题，一直服役到现在，此篇文章便是在这台笔记本上完成的，不过这都是后话了。

事实证明我还真不是游戏党，这些年来各大网游和单机游戏我都很少玩，除了我的世界。我是一个 MC 老玩家，十分喜欢这个游戏，大一时也补票购买了正版 Minecraft，但是 java 版 Minecraft 是出了名的优化差，虽然游戏配置要求不高，但游玩久了存档大起来也难免出现卡顿，我这台轻薄本也渐渐吃力了起来。

就在这时我听说了 Linux，当时的我对 Linux 是啥完全不了解，只是听说 Linux 系统在低配机器上比 Windows 流畅，而且 Minecraft 也有原生 Linux 版本，于是我打算尝试一下。我当时对那么多 Linux 发行版也不了解，就直接选了最多人推荐的 [Ubuntu](https://sspai.com/link?target=https%3A%2F%2Fubuntu.com%2F)，系统的安装并不难，从官网下载 iso，制作启动盘就可以安装了，并且还成功安装并运行了 Minecraft，也确实比在 Windows 上流畅许多。我甚至还得瑟地在 QQ 上发了条动态。

![](https://cdn.sspai.com/2023/01/17/15fac54591615234469b5c8cf823db61.png)

## 因为颜值，迷恋上 Deepin

Ubuntu 的使用，止步于尝鲜，因为当时的我对于 Linux 系统的使用一窍不通，遇到问题也完全不会解决，那时我甚至不懂如何更换镜像源，系统更新只能干等它慢慢下载。等到新鲜劲过了后，便几乎不再碰了。

因为我当时完全不了解 Linux，而且只尝试过 Ubuntu 这一个系统，对 Linux 的图形界面的印象就只有 Ubuntu 那个挺丑的 Gnome 桌面，以至于从此以后都产生了对 Gnome 的恐惧，当时的我认为 Linux 的图形界面就该是简陋又丑陋的，毕竟颜值和流畅度不可得兼。直到我了解到了 [Deepin](https://sspai.com/link?target=https%3A%2F%2Fwww.deepin.org%2Findex%2Fzh)，被它的颜值戳中了，我才知道原来 Linux 系统也可以如此高颜值。于是在许久之后，我决定再次尝试 Linux，这次，我要装 Deepin。

![](https://cdn.sspai.com/2023/01/17/b6e2131e4ee6d6c43923b24897490590.jpeg)

## 又因为 Deepin 太难用，寻找平替

然而我对于 Deepin 的体验并不美好，当时的所谓国产系统并没有受到大众的重视，也并未受到很多资助，所以 Deepin 当初并不好用，有很多 bug，界面也有些卡顿，可能是硬件驱动的原因，网络也有问题。

就在这时我也对 Linux 有了一点了解，知道了 Linux 的底层系统和图形界面是相互独立的，可以为一个系统安装不同的图形界面。我又搜索了一番，找到了一个如何在 Ubuntu 系统上安装 Deepin 的桌面环境 DDE 的教程，于是我照着教程操作，一番操作后，终于在 Ubuntu 系统上体验到了 DDE 桌面环境。

但是我在 Ubuntu 上的 DDE 体验也并不好，当时 [UbuntuDDE](https://sspai.com/link?target=https%3A%2F%2Fubuntudde.com%2F) 这个项目还不存在，DDE 相关软件包也并不在 Ubuntu 的官方软件源中，想要安装，需要添加 ppa，装上之后也遇到了一堆的问题，最后也没能长期使用。

## 接触 Manjaro，误入 Arch 「歧途」

然后我又接触到了 [Manjaro](https://sspai.com/link?target=https%3A%2F%2Fmanjaro.org%2F)，听说这个系统比较好用，最主要的当时 Manjaro 有一个使用 DDE 桌面的社区版本，于是我决定尝试一下，就这样我误入了 Arch 系的「歧途」。

Manjaro DDE 算是我第一个长期使用的发行版了，首先 Manjaro 相对较激进的软件更新，对较新的硬件适配和优化更好，使 DDE 在这个系统上有近乎「完美」的流畅体验，其次因其背靠 Arch Linux，也可以享受到 [AUR](https://sspai.com/link?target=https%3A%2F%2Faur.archlinux.org%2F) 庞大的软件库，基本上你能想到的软件，在 AUR 上都可以找到。因为系统的流畅度比 Windows 好上太多，再加上我常用的软件大部分都陆续找到了替代品，所以从那时开始，我使用 Windows 的频率越来越低。

## 因为 DDE，最终又转到了 Arch Linux

我在个人电脑上使用 Manjaro 前前后后大约有一年多，当时的我虽然已经开始有意识地学习一些 Linux 相关知识，但整体来说还是个小白，还处在遇到问题不会修就重装的状态，之后有一次准备重装时，发现 Manjaro 取消了 DDE 社区版，在官网上下载不到 iso 了。

挣扎了一番后，我最终决定直接安装 [Arch Linux](https://sspai.com/link?target=https%3A%2F%2Farchlinux.org%2F)，Arch Linux 是 Manjaro 的上游发行版，可定制程度很高，但没有图形安装界面，据说安装难度非常高。我参考着官网的[安装教程](https://sspai.com/link?target=https%3A%2F%2Fwiki.archlinux.org%2Ftitle%2FInstallation_guide)和互联网上搜索到的几篇文章，失败了几次后，还是成功安装上了 Arch Linux 并配置好了 DDE 桌面，从 Manjaro 迁移到 Arch，除了安装有些麻烦外，也并没有什么不适应，感觉最大的不同是没有了 Manjaro 自带的图形化应用商店，不过用命令行安装软件也是 Linux 基本操作了，问题不大。

那时的我可能没想到，Arch Linux 结束了我的 distro hopping 之路，成了我使用时间最长的发行版，一直使用到现在。

![](https://cdn.sspai.com/2023/01/17/efa4f9ebfa2533a08bc73c289cb729fd.jpeg)

## 无奈抛弃 DDE

很长一段时间内 DDE 都是我心目中的首选桌面环境，直到 Deepin 20 版本的发布，可能是为了适配触屏设备，Deepin 20 采用了大圆角和特别粗的窗口标题栏的设计，这样的外观我实在爱不起来，再加上 Arch Linux 激进的软件更新，在 Deepin 20 还在测试阶段时，新版的 DDE 桌面就已经进入了 Arch 的 stable 仓库，导致更新后 bug 不断，最终使我忍无可忍决定换掉 DDE。

至于换什么桌面环境，因为使用 Ubuntu 留下的阴影，不想再用 Gnome，xfce、lxqt 之类的桌面环境又觉得太过简陋，最终我决定使用 KDE Plasma，因为这个桌面环境有非常高的可定制性，我还研究着把它配置成了老版本 DDE 的样子。

_**请忽略截图下方的打码，那是某平台的水印，因为这些截图都是我之前发在某社交软件上的，本地早就找不到了**_

![](https://cdn.sspai.com/2023/01/17/bc33388bb2efd48d6300b9725166224c.jpeg)

## 逐渐硬核，沉迷窗口管理器

KDE Plasma 桌面我使用了相当长时间，期间也没遇到什么大问题，我一度以为这就是我的养老配置，不再折腾了。但是随着我对 Linux 认识的深入，也涉足了一些之前未曾触及的领域，比如窗口管理器。

给不了解的人科普一下，Linux 系统完整的图形界面分为底层的 WM（Window Manager，窗口管理器）和上层的 DE（Desktop Environment，桌面环境）两部分，如 KDE Plasma 是 DE，底层的 WM 是 Kwin，同时也存在一些可以独立于 DE 运行的 WM，如 i3wm、dwm、awesome 等等，这些窗口管理器相比于完整的桌面环境，大多可定制性更高，更轻量化，更快，同时学习门槛也更高。

看到那些大佬们自己定制的窗口管理器[配置](https://sspai.com/link?target=https%3A%2F%2Fwww.reddit.com%2Fr%2Funixporn%2F)，全键盘操作，炫酷、高效又优雅，我心中的折腾之魂又燃了起来，于是我开始了研究窗口管理器，我最开始装的是 [i3wm](https://sspai.com/link?target=https%3A%2F%2Fi3wm.org%2F)，并且最终配置的相对比较满意；研究过 [dwm](https://sspai.com/link?target=https%3A%2F%2Fdwm.suckless.org%2F)，配置太复杂最终放弃；后来换成了 [sway](https://sspai.com/link?target=https%3A%2F%2Fswaywm.org%2F)，基本就是 wayland 版本的 i3wm，配置差不多；目前使用的是 [Hyprland](https://sspai.com/link?target=https%3A%2F%2Fhyprland.org%2F)。

因为窗口管理器算是小众中的小众了，所以在中文互联网相关的文章与资料很少，也基本从这时开始，我逐渐习惯了搜索英文资料，阅读英文文档，多少也算是锻炼了英语能力吧（笑）。

## 一次「契机」，促使我删除了 Windows

从最初体验 Ubuntu 开始，我都是双系统安装的 Linux，Windows 与 Linux 各占用一个分区。最初我只是为了尝鲜，所以给 Linux 的分区也不大，只有大约 20G，到后来随着 Linux 的使用频率增加，Windows 的使用频率随之减少，随着一次次重装，给 Linux 分配的分区也越来越大，到最后只给 Windows 留了约 60G 的分区，而且很少打开 Windows 了。

我笔记本的原装硬盘是 256G 的，后来换了一块西部数据的 500G SATA 固态蓝盘，一直正常使用，直到后来听说了部分西部数据硬盘存在冷数据掉速的问题，于是好奇测试了一下我的硬盘速度，然而很不幸地发现，长期不使用的 Windows 分区，已经掉速到连机械硬盘还不如，而且系统十分卡顿，而 Linux 分区没有掉速问题。

考虑到那时的我已经几乎不用 Windows 了，于是我决定一不做二不休，直接把硬盘格式化，全盘安装了 Arch Linux。这次重装前的数据备份尤为痛苦，因为数据掉速，复制的速度实在太慢了。同时这也是我目前为止最后一次在自己的主力机上重装 Arch Linux，距今已经有两年了，首先是 Arch Linux 其实没有刻板印象中那么容易滚挂，再者是随着我知识水平的提升，出现了一些问题我也能够想办法解决了。

## 我的电脑都装了啥

从 Windows 迁移到 Linux 是完全不同的体验，除了少数的跨平台支持的软件，绝大多数 Windows 的软件都需要花时间寻找替代品并适应，另外还有一些软件可以极大地改善 Linux 系统的使用体验，以下是一些我自己经常用的软件及配置，希望可以给感兴趣的朋友一些参考。

系统，当然是 [Arch Linux](https://sspai.com/link?target=https%3A%2F%2Farchlinux.org%2F)，我启用了官方的 `multilib` 仓库，以便安装 steam 和一些 32 位软件，添加了三个第三方仓库，分别是： [archlinuxcn](https://sspai.com/link?target=https%3A%2F%2Fwww.archlinuxcn.org%2Farchlinux-cn-repo-and-mirror%2F)，包含很多国内用户常用的软件；[liquorix](https://sspai.com/link?target=https%3A%2F%2Fliquorix.net%2F)，包含预购建好的 linux-lqx 内核；[chaotic-aur](https://sspai.com/link?target=https%3A%2F%2Faur.chaotic.cx%2F)，包含很多预购建好的 AUR 软件。

系统内核，Arch Linux 官方维护了五个版本的[内核](https://sspai.com/link?target=https%3A%2F%2Fwiki.archlinux.org%2Ftitle%2FKernel)，我曾经使用过原版内核 [linux](https://sspai.com/link?target=https%3A%2F%2Farchlinux.org%2Fpackages%2Fcore%2Fx86_64%2Flinux%2F)，长期支持内核 [linux-lts](https://sspai.com/link?target=https%3A%2F%2Farchlinux.org%2Fpackages%2Fcore%2Fx86_64%2Flinux-lts%2F)，和优化桌面体验和性能的 zen 内核 [linux-zen](https://sspai.com/link?target=https%3A%2F%2Farchlinux.org%2Fpackages%2Fextra%2Fx86_64%2Flinux-zen%2F)，但是都偶尔遇见过桌面卡住的问题，我也不是很确定是否是内核的问题，不过我换成了 linux-lqx 内核后就几乎没有遇见这样的问题了。从前文提到的 liquorix 仓库安装，这个内核修改自 linux-zen 内核，不过针对桌面、多媒体和游戏进行了优化，我个人的体验这个内核稳定性不错，很少遇到问题。

AUR helper，在 Arch Linux 安装来自 [AUR](https://sspai.com/link?target=https%3A%2F%2Faur.archlinux.org%2F) 的软件，标准的方法是去到官网，复制下来地址，用 git clone 下来，再使用 makepkg 构建安装，步骤其实挺繁琐的，而 AUR helper 所做的工作就是把这些步骤自动化，同时也可以作为 Arch 的包管理器 pacman 的前端，代替 pacman 使用。我目前使用的是 [paru](https://sspai.com/link?target=https%3A%2F%2Fgithub.com%2Fmorganamilo%2Fparu)，推荐安装 [paru-bin](https://sspai.com/link?target=https%3A%2F%2Faur.archlinux.org%2Fpackages%2Fparu-bin)，可以省去很多编译的时间，据说 paru 是作者退出 [yay](https://sspai.com/link?target=https%3A%2F%2Fgithub.com%2FJguer%2Fyay) 的开发后使用 rust 重写的，[这里](https://sspai.com/link?target=https%3A%2F%2Fwiki.archlinux.org%2Ftitle%2FAUR_helpers)列出了所有已知的 AUR helper，要注意，AUR helper 并不是 Arch Linux 官方维护的，也不会被收录进官方仓库，同时也要熟悉[手动安装 AUR 软件的方法](https://sspai.com/link?target=https%3A%2F%2Fwiki.archlinux.org%2Ftitle%2FArch_User_Repository%23Installing_and_upgrading_packages)以便于排错。

[keyd](https://sspai.com/link?target=https%3A%2F%2Fgithub.com%2Frvaiya%2Fkeyd)，一个开源的键盘映射程序，通过 [AUR](https://sspai.com/link?target=https%3A%2F%2Faur.archlinux.org%2Fpackages%2Fkeyd-git) 安装。作为一个 vim 用户，需要经常使用 ESC 键，但是现代的键盘上的 ESC 键在左上角，十分不方便，所以很多 vimer 会把 大写锁定键重新映射为 ESC 键，想要映射键盘有很多方法，我使用的是 keyd 这个程序，以 systemd 服务启动，不依赖桌面环境。我的配置主要是互换了 ESC 键与 大写锁定键，同时定义了一个 layer，使用 h j k l 来进行上下左右移动，这也是常用的 vim 键位。

图形环境，前文提到了，我使用的是 [Hyprland](https://sspai.com/link?target=https%3A%2F%2Fhyprland.org%2F) 窗口管理器，通过 AUR 安装的 [hyprland-bin](https://sspai.com/link?target=https%3A%2F%2Faur.archlinux.org%2Fpackages%2Fhyprland-bin) ，Hyprland 是一个 Wayland 窗口管理器，有丝滑的过渡动画，漂亮的模糊效果，还有很高可定制性。[Wayland](https://sspai.com/link?target=https%3A%2F%2Fwayland.freedesktop.org%2F) 是替代 [Xorg](https://sspai.com/link?target=https%3A%2F%2Fx.org%2Fwiki%2F) 的下一代显示服务协议，我当初为了体验 Wayland，从 [i3wm](https://sspai.com/link?target=https%3A%2F%2Fi3wm.org%2F) 迁移到了 [sway](https://sspai.com/link?target=https%3A%2F%2Fswaywm.org%2F)，主要是因为它兼容 i3 的配置文件格式，不过因为 Wayland 还处于早期开发阶段，再加上 sway 的开发团队中可能没有母语为非拉丁系语言的开发者，它对输入法支持不太好，在某些软件中没办法输入中文，需要安装一个特殊的修改版本 [sway-im](https://sspai.com/link?target=https%3A%2F%2Faur.archlinux.org%2Fpackages%2Fsway-im) 才可以解决输入法问题，然而这个软件包又因为最近的系统更新不可用了。我几个月前发现了 Hyprland 项目，那时的 Hyprland 还处于非常早期的开发阶段，在我电脑上安装后十分不稳定，几乎不可用，不过 hyprland 项目的开发进度非常快，现在已经非常可用，同时它的主要开发者貌似是个日本人，因为他们也需要输入法输入日文，所以目前 Hyprland 对输入法的支持比 sway 好很多。

![](https://cdn.sspai.com/2023/01/17/a55a0f011e1e34c0387801a46d03651d.gif)

[piper](https://sspai.com/link?target=https%3A%2F%2Farchlinux.org%2Fpackages%2Fcommunity%2Fany%2Fpiper%2F) 是一个 [libratbag](https://sspai.com/link?target=https%3A%2F%2Fgithub.com%2Flibratbag%2Flibratbag) 的图形化前端，libratbag 是一个开源的游戏外设配置软件，[这里](https://sspai.com/link?target=https%3A%2F%2Fgithub.com%2Flibratbag%2Flibratbag%2Ftree%2Fmaster%2Fdata%2Fdevices)列出了所有适配的设备。我的罗技 G304 鼠标（国外名字叫 G305）可以调整回报率、DPI 和按键功能等。

![](https://cdn.sspai.com/2023/01/17/1aa94203b8a9a0505ae69b0bf10c2720.png)

![](https://cdn.sspai.com/2023/01/17/eecb8d2f31d122d4ef24813a5085c79c.png)

终端模拟器，我目前使用的是 [Alacritty](https://sspai.com/link?target=https%3A%2F%2Farchlinux.org%2Fpackages%2Fcommunity%2Fx86_64%2Falacritty%2F)，可以直接从官方库安装，Alacritty 是一个开源的终端模拟器，支持 GPU 加速，速度很快，并且原生支持 Wayland，shell 使用的是 [zsh](https://sspai.com/link?target=https%3A%2F%2Farchlinux.org%2Fpackages%2Fextra%2Fx86_64%2Fzsh%2F)，可定制性高，并且完全兼容 bash，配置框架使用的 [zimfw](https://sspai.com/link?target=https%3A%2F%2Fgithub.com%2Fzimfw%2Fzimfw)，相比于更加广为人知的 [oh my zsh](https://sspai.com/link?target=https%3A%2F%2Fgithub.com%2Fohmyzsh%2Fohmyzsh)，速度更快，zimfw 的默认配置差不多就开箱即用了，配置结构也很简单，也没什么可讲的。

输入法我一直使用的是 [Fcitx5](https://sspai.com/link?target=https%3A%2F%2Fgithub.com%2Ffcitx%2Ffcitx5)，Fcitx5 是替代 [Fcitx](https://sspai.com/link?target=https%3A%2F%2Fgithub.com%2Ffcitx%2Ffcitx) 的新一代输入法框架，支持很多输入法引擎，[这里](https://sspai.com/link?target=https%3A%2F%2Fwiki.archlinuxcn.org%2Fwiki%2FFcitx5)有 Fcitx5 详细的配置教程。

字体，我使用的全局字体是 [Sarasa Mono SC Nerd](https://sspai.com/link?target=https%3A%2F%2Fgithub.com%2Flaishulu%2FSarasa-Mono-SC-Nerd)，这是一个等宽字体，中文和英文的宽度严格遵守 2:1 的比例，同时集成了 [Nerd Fonts](https://sspai.com/link?target=https%3A%2F%2Fwww.nerdfonts.com%2F) 的图标，非常适合在终端下使用。Arch Linux 已经有人将其放在了 [AUR 上](https://sspai.com/link?target=https%3A%2F%2Faur.archlinux.org%2Fpackages%2Fnerd-fonts-sarasa-mono)，不过原维护者弃坑了，于是我将其接手了，现在是我在维护这个 AUR 包。

浏览器，我使用的是 Firefox，通过 Arch Linux 的官方仓库安装的浏览器本体 [firefox](https://sspai.com/link?target=https%3A%2F%2Farchlinux.org%2Fpackages%2Fextra%2Fx86_64%2Ffirefox%2F)、中文包 [firefox-i18n-zh-cn](https://sspai.com/link?target=https%3A%2F%2Farchlinux.org%2Fpackages%2Fextra%2Fany%2Ffirefox-i18n-zh-cn%2F) 以及广告过滤扩展 [firefox-ublock-origin](https://sspai.com/link?target=https%3A%2F%2Farchlinux.org%2Fpackages%2Fcommunity%2Fany%2Ffirefox-ublock-origin%2F)，并使用了 [Arkenfox](https://sspai.com/link?target=https%3A%2F%2Fgithub.com%2Farkenfox%2Fuser.js) 优化了隐私特性，具体可参考我之前的[文章](https://sspai.com/post/76688)。同时我还安装了 [brave-bin](https://sspai.com/link?target=https%3A%2F%2Faur.archlinux.org%2Fpackages%2Fbrave-bin) 作为备用。

编辑器，很多软件的配置，都需要修改相关的配置文件，这时自然需要一个趁手的文本编辑器。关于 Linux 下谁才是最好的编辑器的争论旷日持久，史称 「编辑器圣战」，有说是 vim 的，有说是 emacs 的，甚至还有说是 vscode 的。而我个人更习惯使用的是轻量、快速、配置自由度高的 vim，或者说是 vim 的 fork 版本 [neovim](https://sspai.com/link?target=https%3A%2F%2Farchlinux.org%2Fpackages%2Fcommunity%2Fx86_64%2Fneovim%2F)，相比于原版 vim，neovim 的代码量更少，更快速，完全兼容 vim 配置格式与插件，同时又有自己的一套 lua 配置格式以及插件生态。vim 拥有一套独特但有高效的快捷键，熟悉后可以大大提高效率。我目前的 neovim 配置主要是参考了这个[视频](https://sspai.com/link?target=https%3A%2F%2Fwww.youtube.com%2Fwatch%3Fv%3Dw7i4amO_zaE%26t%3D1089s)和这个[系列视频](https://sspai.com/link?target=https%3A%2F%2Fwww.youtube.com%2Fplaylist%3Flist%3DPLhoH5vyxr6Qq41NFL4GvhFp-WLd5xzIzZ)，配置了语法高亮，自动补全等等。本文便是我在 neovim 上以 markdown 格式编辑完成的。

Office 套件，除了纯文本文件，有时还会免不了要编辑一些 Office 文档，但是众所周知微软的 Microsoft Office 套件是闭源的收费软件且没有 Linux 版本，虽然有人在 Linux 上通过 wine 兼容层成功运行了 Microsoft Office，但是我从来没有成功过。不过幸运的是 Microsoft Office 在 Linux 下也有很多替代品，国内用户最为熟知的是 [WPS Office](https://sspai.com/link?target=https%3A%2F%2Faur.archlinux.org%2Fpackages%2Fwps-office)，WPS Office 的 Linux 版本没有广告，功能齐全且界面简洁，简直是良心软件，Arch Wiki 上有比较详细的安装及配置[教程](https://sspai.com/link?target=https%3A%2F%2Fwiki.archlinuxcn.org%2Fwiki%2FWPS_Office)。我曾经使用了很长时间的 WPS Office，不过后来我发现 WPS Office 编辑较大的文档时，会比较卡顿，有时甚至崩溃。所以我后来改用了 LibreOffice，在 Arch Linux 官方库中，LibreOffice 有两个版本，一个是 [libreoffice-still](https://sspai.com/link?target=https%3A%2F%2Farchlinux.org%2Fpackages%2Fextra%2Fx86_64%2Flibreoffice-still%2F)，稳定但版本较旧，另一个是 [libreoffice-fresh](https://sspai.com/link?target=https%3A%2F%2Farchlinux.org%2Fpackages%2Fextra%2Fx86_64%2Flibreoffice-fresh%2F)，版本更新功能更多。我个人更推荐安装 fresh 版本。LibreOffice 在默认配置下与 Microsoft Office 兼容性并不太好，但可以修改配置使其与 Microsoft Office 兼容性更好，参考这个[视频](https://sspai.com/link?target=https%3A%2F%2Fwww.youtube.com%2Fwatch%3Fv%3DG0che2Az9hw)。至于缺失的字体，可以去 Windows 的 C 盘下找到字体目录，将所有的 ttf 字体全部复制到 Linux 用户主目录的 `.local/share/fonts` 目录下，就可以使用 Windows 字体了。

屏幕录制，广为熟知的 [OBS Studio](https://sspai.com/link?target=https%3A%2F%2Fobsproject.com%2F) 也有 Linux 版本，我主要是用它的虚拟摄像头功能来应付一些视频会议，除此之外它还可以进行屏幕录制与直播，功能十分强大。OBS Studio 的 Linux 版本只有 Ubuntu PPA 和 [flatpak](https://sspai.com/link?target=https%3A%2F%2Fflathub.org%2Fapps%2Fdetails%2Fcom.obsproject.Studio) 版本是由 OBS 官方维护的，除此之外的都是第三方编译的版本，不过据说其最新的 flatpak 版本在 AMD 机型上有问题，具体参考这个[视频](https://sspai.com/link?target=https%3A%2F%2Fwww.youtube.com%2Fwatch%3Fv%3Dgp3GMwYWeJ4)。相比于使用 flatpak 安装软件，我更偏向于使用发行版原生的包管理器安装软件，然而 Arch Linux 编译的 [obs-studio](https://sspai.com/link?target=https%3A%2F%2Farchlinux.org%2Fpackages%2Fcommunity%2Fx86_64%2Fobs-studio%2F) 因为依赖问题功能并不全，AUR 的 [obs-studio-tytan652](https://sspai.com/link?target=https%3A%2F%2Faur.archlinux.org%2Fpackages%2Fobs-studio-tytan652) 是除了 OBS 官方维护版本之外功能最全的版本了，只缺失了 twitch 和 Youtube 直播的的集成，对国内用户来说影响不大。关于如何在 Hyprland 上启用屏幕录制，可以参考这个[教程](https://sspai.com/link?target=https%3A%2F%2Fgist.github.com%2FPowerBall253%2F2dea6ddf6974ba4e5d26c3139ffb7580)。

游戏，怎么能够忘记最初接触 Linux 的初心呢。我主要玩的是 java 版的 Minecraft，由于官方启动器并不好用，我使用的启动器是 [HMCL](https://sspai.com/link?target=https%3A%2F%2Fgithub.com%2Fhuanghongxun%2FHMCL)，支持多版本隔离管理和模组下载，AUR 的 [hmcl-bin](https://sspai.com/link?target=https%3A%2F%2Faur.archlinux.org%2Fpackages%2Fhmcl-bin) 软件包现在就是我在维护的。对于其他游戏，[steam](https://sspai.com/link?target=https%3A%2F%2Farchlinux.org%2Fpackages%2Fmultilib%2Fx86_64%2Fsteam%2F) 的官方客户端已经在 Arch 官方库里面了，只是需要启用 `multilib` 仓库；[Epic](https://sspai.com/link?target=https%3A%2F%2Fstore.epicgames.com%2F) 和 [GOG](https://sspai.com/link?target=https%3A%2F%2Fwww.gog.com%2F) 平台的游戏可以通过 [Heroic Games Launcher](https://sspai.com/link?target=https%3A%2F%2Faur.archlinux.org%2Fpackages%3FO%3D0%26K%3Dheroic-games-launcher) 游玩，不在以上平台的游戏也可以通过 [lutris](https://sspai.com/link?target=https%3A%2F%2Farchlinux.org%2Fpackages%2Fcommunity%2Fany%2Flutris%2F) 游玩（这个我很少用），另外还有 [ProtonUp-Qt](https://sspai.com/link?target=https%3A%2F%2Faur.archlinux.org%2Fpackages%3FO%3D0%26K%3Dprotonup-qt) 来为 steam、Heroic Games Launcher 还有 lutris 管理 windows 兼容层 wine 和 proton 的版本。这样一来，绝大多数的游戏都可以在 Linux 上正常游玩了。

## 结语

关于这篇文章的结语，我思考了很久。我为什么要抛弃 Windows 转而使用 Linux？是因为其开源免费，还是因为其流畅度高？是因为其可定制性高，还是只是单纯的标新立异？可能都有，我也不好说。但我很确定的是自从误入了 Linux 这个「兔子洞」后，我就再也离不开了，现在如果让我把系统换回 Windows，我可能反而会不适应。

但若是一个普通人来问是否推荐用 Linux，那么我的答案是不推荐。正如那句话所说：「Linux is free if your time is free」，我的建议是，不要贸然决定切换到 Linux，在考虑切换到 Linux 之前，要充分了解 Linux 的各种利与弊，要先问自己，是否愿意花大量时间研究系统与软件配置，是否愿意以及是否有能力阅读大量的文档与教程，其中很多还是英文的，以及在自己的使用场景下，常用的软件是否有 Linux 版本或相似的替代品，最重要的是，切换到 Linux 后，是否能有媲美甚至优于 Windows 的体验。以及不要头铁直接删除 Windows，切换到 Linux 需要一个适应过程，在完全确定自己不需要 Windows 之前，都要保留一份 Windows 的安装，双系统也好，虚拟机也好，都要给自己留一条退路。

\> 下载 [少数派 2.0 客户端](https://sspai.com/page/client)、关注 [少数派公众号](https://sspai.com/s/J71e)，解锁全新阅读体验 📰

\> 实用、好用的 [正版软件](https://sspai.com/mall)，少数派为你呈现 🚀