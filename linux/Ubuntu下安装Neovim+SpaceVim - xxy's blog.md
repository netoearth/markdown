前几天装了WSL，身为多年的伪Vim粉~~VS Code宇宙第一~~决定顺便把Vim给搞搞

本来是打算就用原生Vim然后堆plug的，但是既然已经折腾了，就不差这一下了。

因为太久没玩过Ubuntu了，所以上来就是`sudo apt install neovim`，然后报Error，提示 **Unable to locate package neovim** 进Neovim官网看了下[安装教程](https://github.com/neovim/neovim/wiki/Installing-Neovim)，在Ubuntu那一栏可以看到，从18.04开始可以通过PPA来安装了，照着官方教程一顿梭

<table><tbody><tr><td><pre tabindex="0"><code><span>1
</span><span>2
</span><span>3
</span></code></pre></td><td><pre tabindex="0"><code data-lang="bash">sudo add-apt-repository ppa:neovim-ppa/stable
sudo apt-get update
sudo apt-get install neovim
</code></pre></td></tr></tbody></table>

> 老版本的Ubuntu可能需要先安装PPA `sudo apt-get install software-properties-common`  
> 安装完后可以输入`nvim` 打开，当然可以修改下alias，通过vi打开nvim  
> 这里我选择软连接的方式将vi连接到nvim，因为现在wsl系统里的vi和vim命令就是软连接文件，所以我想删掉现在的vi，然后重新软连接到nvim  
> 先`which vi` 找到vi的目录， 比如我的系统中vi文件的目录是`/usr/bin/` 再输入`ls -il` 可以看到vi是个连接文件，指向 **/etc/alternatives/vi**  
> 然后这里我把两个软连接给删掉再建立新的软连接

<table><tbody><tr><td><pre tabindex="0"><code><span>1
</span><span>2
</span><span>3
</span><span>4
</span></code></pre></td><td><pre tabindex="0"><code data-lang="bash">sudo rm -rf /usr/bin/vi 
sudo rm -rf /usr/bin/vim
sudo ln -s /usr/bin/nvim /usr/bin/vi
sudo ln -s /usr/bin/nvim /usr/bin/vim
</code></pre></td></tr></tbody></table>

这时候再输入vi/vim就可以打开nvim了

[![https://cdn.jsdelivr.net/gh/xxy-im/storage@gh-pages/images/neovim.png](https://cdn.jsdelivr.net/gh/xxy-im/storage@gh-pages/images/neovim.png)](https://cdn.jsdelivr.net/gh/xxy-im/storage@gh-pages/images/neovim.png "点击放大")

Neovim

[SpaceVim](https://spacevim.org/cn/)是一个开源的模块化配置集合，可以通过它很方便的打造出适用于各种开发场景的IDE。

<table><tbody><tr><td><pre tabindex="0"><code><span>1
</span></code></pre></td><td><pre tabindex="0"><code data-lang="bash">curl -sLf https://spacevim.org/cn/install.sh <span>|</span> bash
</code></pre></td></tr></tbody></table>

[![https://cdn.jsdelivr.net/gh/xxy-im/storage@gh-pages/images/install-spacevim-1.png](https://cdn.jsdelivr.net/gh/xxy-im/storage@gh-pages/images/install-spacevim-1.png)](https://cdn.jsdelivr.net/gh/xxy-im/storage@gh-pages/images/install-spacevim-1.png "点击放大")

安装SpaceVim

[![https://cdn.jsdelivr.net/gh/xxy-im/storage@gh-pages/images/install-spacevim-2.png](https://cdn.jsdelivr.net/gh/xxy-im/storage@gh-pages/images/install-spacevim-2.png)](https://cdn.jsdelivr.net/gh/xxy-im/storage@gh-pages/images/install-spacevim-2.png "点击放大")

字体安装报错

提示安装完成，打开vim却没有加载出SpaceVim，不知道哪里出现问题，往上翻也只看到几个字体安装的报错，感觉应该是和WSL环境的配置文件有关系，但还是先在网上找了那几个字体报错的解决方法

<table><tbody><tr><td><pre tabindex="0"><code><span>1
</span><span>2
</span><span>3
</span><span>4
</span></code></pre></td><td><pre tabindex="0"><code data-lang="bash"><span># 使mkfontscale和mkfontdir命令正常运行</span>
sudo apt-get install ttf-mscorefonts-installer
<span># 使fc-cache命令正常运行</span>
sudo apt-get install fontconfig 
</code></pre></td></tr></tbody></table>

然后再安装试试  
结果还真是字体的问题，重装下就好了…

[![https://cdn.jsdelivr.net/gh/xxy-im/storage@gh-pages/images/spacevim.png](https://cdn.jsdelivr.net/gh/xxy-im/storage@gh-pages/images/spacevim.png)](https://cdn.jsdelivr.net/gh/xxy-im/storage@gh-pages/images/spacevim.png "点击放大")

SpaceVim

Normal模式下`:SPUpdate` 更新所有插件，`:SPUpdate SpaceVim`可以更新自身

[![https://cdn.jsdelivr.net/gh/xxy-im/storage@gh-pages/images/SUpdate.png](https://cdn.jsdelivr.net/gh/xxy-im/storage@gh-pages/images/SUpdate.png)](https://cdn.jsdelivr.net/gh/xxy-im/storage@gh-pages/images/SUpdate.png "点击放大")

更新所有插件

再次打开vim又 vimproc’s DLL报错，直接`:VimProcInstall`  
或者make一下

<table><tbody><tr><td><pre tabindex="0"><code><span>1
</span><span>2
</span></code></pre></td><td><pre tabindex="0"><code data-lang="bash"><span>cd</span> ~/.SpaceVim/bundle/vimproc.vim/
make
</code></pre></td></tr></tbody></table>

> 有些icon显示不出来，只有个小方框，有可能是因为字体的问题  
> 可以使用`fc-list`命令查看ubuntu中安装的字体  
> SpaceVim默认使用[SourceCodePro Nerd Font Mono](https://github.com/ryanoasis/nerd-fonts/releases)字体

安装Nerd Font

<table><tbody><tr><td><pre tabindex="0"><code><span>1
</span><span>2
</span><span>3
</span><span>4
</span><span>5
</span><span>6
</span></code></pre></td><td><pre tabindex="0"><code data-lang="bash">wget -c https://github.com/ryanoasis/nerd-fonts/releases/download/v2.1.0/SourceCodePro.zip
sudo unzip SourceCodePro -d /usr/share/fonts/SourceCodePro
<span>cd</span> /usr/share/fonts/SourceCodePro
sudo mkfontscale <span># 生成核心字体信息</span>
sudo mkfontdir <span># 生成字体文件夹</span>
sudo fc-cache -fv <span># 刷新系统字体缓存</span>
</code></pre></td></tr></tbody></table>

___

如果使用终端的话需要修改终端的配置  
比如我用的是**Windows Terminal**  
在Windows下安装完[SourceCodePro Nerd Font Mono](https://github.com/ryanoasis/nerd-fonts/releases)字体后需要在**Windows Terminal**配置文件WSL配置下加上  
`"fontFace": "SauceCodePro Nerd Font"`  
注意第一个f小写，然后再重启终端就能看到图标都出来了

[![https://cdn.jsdelivr.net/gh/xxy-im/storage@gh-pages/images/nerd-font.png](https://cdn.jsdelivr.net/gh/xxy-im/storage@gh-pages/images/nerd-font.png)](https://cdn.jsdelivr.net/gh/xxy-im/storage@gh-pages/images/nerd-font.png "点击放大")

更新字体后

看不习惯相对行号，在配置文件中取消

打开~/.SpaceVim.d/init.toml 主题选择 SpaceVim

<table><tbody><tr><td><pre tabindex="0"><code><span>1
</span></code></pre></td><td><pre tabindex="0"><code data-lang="fallback">colorscheme = "SpaceVim"
</code></pre></td></tr></tbody></table>

[![https://cdn.jsdelivr.net/gh/xxy-im/storage@gh-pages/images/colorscheme.png](https://cdn.jsdelivr.net/gh/xxy-im/storage@gh-pages/images/colorscheme.png)](https://cdn.jsdelivr.net/gh/xxy-im/storage@gh-pages/images/colorscheme.png "点击放大")

SpaceVim主题

打开vim，`空格 + f + v + d` (一个个按)，快捷键打开配置文件，空格(space)为自定义快捷键的前缀，按下空格后可以看到所有的自定义快捷键 按照[官方配置](https://spacevim.org/cn/use-vim-as-a-c-cpp-ide/)把需要的加上去就可以了

像clangd，clang这些如果需要的话要先装好才能配置成功，不然vim会报**clangd is not executable** 直接apt安装的clang貌似版本会有点低，所以建议用[官方源](https://apt.llvm.org/)

<table><tbody><tr><td><pre tabindex="0"><code><span>1
</span></code></pre></td><td><pre tabindex="0"><code data-lang="bash">bash -c <span>"</span><span>$(</span>wget -O - https://apt.llvm.org/llvm.sh<span>)</span><span>"</span>
</code></pre></td></tr></tbody></table>

在`/usr/bin` 目录下找到你的clangd安装目录，比如我的是`/usr/bin/clangd-11` 再执行下面命令

<table><tbody><tr><td><pre tabindex="0"><code><span>1
</span></code></pre></td><td><pre tabindex="0"><code data-lang="bash">sudo update-alternatives --install /usr/bin/clangd clangd /usr/bin/clangd-11 <span>100</span>
</code></pre></td></tr></tbody></table>

> SpaceVim默认使用的补全插件是deoplete，愿意折腾的同学也可以改成YCM，注意兼容问题

最后在cpp文件中使用`SPC + l + r`就可以run代码了

> 修改编译命令可参考[Custom Task](https://spacevim.org/documentation/#tasks)

[官方文档](https://spacevim.org/cn/use-vim-as-a-python-ide/)

其实VS Code + Remote一套用起来才更虚服。  
所以上面这些都是瞎折腾，桌面党还是继续老老实实用VS Code