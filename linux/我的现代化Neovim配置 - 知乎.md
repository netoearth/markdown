之前vim和emacs都折腾过不少时间，也用这俩写过几个小项目，最后终于还是在nvim安家了。需要说明一下，vim和neovim不是一个东西，截止到2022.05.04，neovim已经更新到了0.7.0版，已经和vim分道扬镳，本文中介绍的就是基于0.7版nvim的配置，顺便说一下，现在的nvim可以称得上是一个现代化的编辑器了，其可配置的能力不亚于vscode，配置门槛也不高，入手很方便。当然我已经做了很大程度上的配置工作，你可以直接拿去用。

截止到现在我的nvim也大换过配置4次了，现在是第4代的方案，即彻底现代化的配置。

（YouCompleteMe->coc.nvim->nvim-lsp->现在的纯lua配置）

目前我的git仓库状态：

| 分支 | 插件管理 | 补全方案 | 性能 | 实用性 | 扩展性 | 上手难度 |
| --- | --- | --- | --- | --- | --- | --- |
| main | packer | nvim-cmp | 高 | 高 | 高 | 高 |
| coc | packer | coc.nvim | 较高 | 高 | 一般 | 一般 |

main分支使用纯lua进行配置，启动速度、运行性能、扩展性等都是第一流的水平，唯一的难点可能是需要一定时间熟悉，所以我已经做了大量的配置来尽可能做到不折腾开箱即用。

coc分支目前也采用packer做插件管理，启动速度不会受到插件数量的影响，同时与main分支共享了相当一部分的基础插件，只是补全方案使用了coc.nvim。

因为目前nvim-lspconfig的能力相比于coc.nvim在某些语言比如html三板斧下可能还有所欠缺，所以用户可以在面对不同语言时切换不同的分支。

于我自身而言，neovim相比于vscode的优势主要体现在这几个地方：

1.  nvim聚焦于终端和键盘操作，更加优雅方便。
2.  nvim-lsp, treesitter实现了不同语言配置的大一统。
3.  nvim比vscode轻便快速得多。
4.  nvim的配置更加灵活随性，可定制性远高于vscode。
5.  基于lua的众多nvim的新插件功能都相当不错且符合直觉。
6.  很少在windows上写代码。

除了我的配置之外，目前一个比较好的配置是：

## 1\. 展示成品

![](https://pic1.zhimg.com/v2-c76d0db71f58a73a4ce77f084ad86110_b.jpg)

Dashboard

![](https://pic1.zhimg.com/v2-8fc99008bf6e15fec84c7ed8c49b2358_b.jpg)

Telescope frecency

![](https://pic1.zhimg.com/v2-846b33c3571b05703bda3606993510c0_b.jpg)

Command mode下命令补全

![](https://pic1.zhimg.com/v2-0fad502472683b119d5ea5cf130cb054_b.jpg)

搜索时自动分词选择

![](https://pic3.zhimg.com/v2-edfc9bd3087dd537f60a3f756310f476_b.jpg)

K快速查看函数doc

![](https://pic1.zhimg.com/v2-e2a5d62e1f399ba0a54719f87053f7d8_b.jpg)

gd预览函数定义

![](https://pic1.zhimg.com/v2-5e80b0169419429b1c72a7fd1e6d2058_b.jpg)

gD跳转到函数定义处

![](https://pic2.zhimg.com/v2-8522669c9240970d64deb3acc677f949_b.jpg)

gh查看函数引用情况

![](https://pic1.zhimg.com/v2-97aea2475da50ee144b608a4b3a88fa0_b.jpg)

gr修改变量/函数名

![](https://pic4.zhimg.com/v2-a394f85ab69dedb339a27e7306a6bcd7_b.jpg)

code action提示

![](https://pic3.zhimg.com/v2-86ba73992bc24abe197795ee1f35af1a_b.jpg)

当前位置提示

![](https://pic2.zhimg.com/v2-49c4c7d340f781941d9c380d578763b9_b.jpg)

超长函数末尾开头提示

![](https://pic3.zhimg.com/v2-a31a50d8d10f24a2c9d7186deaf5b702_b.jpg)

使用treesitter的范围性自动选择

![](https://pic1.zhimg.com/v2-2edff539f7a7129d01867d3dbb6411f8_b.jpg)

快速光标位置移动

![](https://pic3.zhimg.com/v2-61f3b2898afef6b3069c71fdda5992a2_b.jpg)

git commit历史记录提示

![](https://pic4.zhimg.com/v2-4bb8a2a7b4268a305fc9417d671316e7_b.jpg)

SnipRun代码片段部分执行

![](https://pic2.zhimg.com/v2-458a118276914fd3d5c7af50632b711d_b.jpg)

悬浮终端

![](https://pic1.zhimg.com/v2-0dd41ce6bae8ec05a40b84b7350925e8_b.jpg)

Debug

![](https://pic1.zhimg.com/v2-ec85420069911884febb40b6fcfea9f4_b.jpg)

88个插件

![](https://pic4.zhimg.com/v2-431c82f7f1be4f0bcd745d698d2b0a47_b.jpg)

启动时间43ms

除了展示的这些功能之外：

1.  支持依托于LSP的保存时代码异步format与lint；
2.  支持按照格式自定义代码补全片段；
3.  部分语言支持依托于DAP的debug；
4.  支持依托于Treesitter的代码块可视化选择；
5.  支持平滑滚动与滚动条滚动；
6.  用vim-fugitve和gitui集成了常用的Git操作；

总之就是你想到的想不到的功能我都配置好了，并且尽可能保证高效快速。

默认保存时格式化，可以使用命令`:FormatToggle`来切换启用或禁用此功能。

## 2\. 配置github的ssh key（如果你已经配置过略过本节即可）

由于众所周知的原因，大陆用户访问github会受到限制，经过测试之后发现使用ssh完全不会受到影响（比我使用代理还要快，几乎是瞬间就能完成插件或者treesitter parser的更新）

2.1 添加github的ssh key

Linux/MacOS用户：

先安装openssh（以archlinux为例，其他发行版应该也是要装openssh这个包）：

Windows用户：

手动下载

安装过程一路默认选项Next即可，安装完成之后启动Git窗口，配置以下用户设置

```
git config --global user.name "你的github账户名"
git config --global user.email "你的github账户默认的邮箱地址"
```

这里以后的设置三种系统都一样，Linux/MacOS在终端输入命令操作，Windows在Git窗口中操作即可

之后生成你的ssh-key

```
ssh-keygen -t rsa -b 4096 -C "你的github账户默认的邮箱地址"
```

过程中会要求输入passphrase做进一步的加密保护，自行输入，直接回车表示没有passphrase。

假设你刚刚生成的sshkey的路径为~/.ssh/id\_rsa（windows的路径形如/c/Users/ayamir/.ssh/id\_rsa）

拷贝~/.ssh/id\_rsa.pub的输入到剪切板（即拷贝公钥内容）

选中输出并复制到系统剪切板

2.2 配置github的ssh key

浏览器打开这个网址：[https://github.com/settings/keys](https://link.zhihu.com/?target=https%3A//github.com/settings/keys)

点击 **New SSH key**，之后将拷贝的公钥内容粘贴到key那个输入框中，tile框中的内容可以输入这个key的描述性文字，以我为例输入的是archlinux ssh

之后点击 **Add SSH key**，最后输入你的github密码确认即可

## 3\. 复制配置

首先确保nvim版本大于或等于0.7

Arch可以直接安装：

别的发行版的安装方式可以去wiki查看：

此外还需要安装ripgrep和fd来配合插件的快速搜索：

```
sudo pacman -S ripgrep fd
```

python需要安装neovim模块：

```
pip install neovim --user
```

终端字体需要使用Nerd Font或者其他有icon字体补丁的font：

我个人推荐使用的终端是Kitty和Alacritty

```
sudo pacman -S kitty alacritty
```

（wezterm-git版支持输入中文）

记得安装好字体之后手动修改终端字体

无缝集成上面这几个终端和neovim来提高速度：

还有个更炫酷的选择：neovide（有很花哨的动画效果）

```
paru neovide
# 建议直接通过archlinuxcn装，从AUR容易失败
```

（显卡和CPU好的用起来应该没有压力，比较旧的机器可能比较吃力）

你可以在这里找到我个人的相关配置文件：

安装之后输入

之后的结果应该是这样

![](https://pic4.zhimg.com/v2-7af77738fdf4a1ce9b9958adabf5ee83_b.jpg)

如图即为安装成功

接着克隆我的仓库到

```
git clone git@github.com:ayamir/nvimdots.git ~/.config/nvim/
```

如果需要使用coc分支的：

```
cd ~/.config/nvim
git switch coc
```

克隆完成之后输入nvim，结果应该像这样：

![](https://pic1.zhimg.com/v2-cdd9f20ddf0206b4c2ce42590ca26da4_b.jpg)

开始clone packer

![](https://pic4.zhimg.com/v2-3956bf343ca153beffbd16515876225b_b.jpg)

输入回车

![](https://pic1.zhimg.com/v2-7edb81465cd01fa01b311008041680a0_b.jpg)

耐心等待所有插件clone完毕

很多情况下会像上图中显示的一样，有些插件clone失败，退出之后重新执行：

再次退出并重新进入nvim：

![](https://pic4.zhimg.com/v2-c301d703229dcfe20aeb45b45f03d623_b.jpg)

上图中的错误也经常出现，但是不影响使用

这时就可以使用telescope打开文件开始编辑

## 4\. 具体介绍

Github的介绍永远是最新的，直接看我的Readme和[Wiki](https://link.zhihu.com/?target=https%3A//github.com/ayamir/nvimdots/wiki)即可。

需要记住的一点是使用\`PackerSync\`命令才能让你对插件相关的config的修改生效（对config.lua和plugins.lua文件的修改，修改其他文件不需要）

具体的问题最好去github开issue问