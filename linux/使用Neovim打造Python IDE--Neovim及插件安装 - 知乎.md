## Neovim和Vim抉择

Vim作为一款超过20年的文本编辑器.因为其众多的插件和例如代码折叠,垂直分割等实用功能.直至今日依然被很多程序员作为首选的文本编辑器.但作为一款有点年纪的软件,其也面临着很多问题: - 其堆积了超过300k行C89代码,已经很少有人能够看的懂了 - Bram Moolenaar作为vim的作者及唯一维护者,在合并分支的时候需要格外小心.因为一旦合并,新的代码就必须要他进行维护

基于以上因素,巴西程序员 Thiago de Arruda Padilha（aka tarruda）以下面目标为宗旨,重构了vim:

1.  简化维护以此来提高bug修复和新功能的合并
2.  将工作分配到其他开发者
3.  无需任何核心代码即可对用户界面进行修改
4.  基于协同处理来提高插件架构的扩展性,用户可以使用任何语言来编写插件.

通过这些目标,新的开发人员将很快加入社区,为用户改进编辑器.如今,已经有越来越多的人选择使用Neovim来代替Vim.作为一位vim拥护者我也在2020年选择了neovim.替换的原因无非是使用neovim同vim几乎没有差异,且越来越多的插件开发者编写为neovim编写插件.特别是看到[ncm2](https://link.zhihu.com/?target=https%3A//github.com/ncm2/ncm2)作者roxma这句

> Note that vim8 support is simply a bonus. It's not the goal of NCM2.

更加坚定了我的想法.

**注:这个系列文章我都是以Ubuntu18.04为准,有些命令可能不适用您的操作系统**

Neovim安装很简单,下面我介绍下主流的操作系统安装方法,未提及的操作系统请查阅neovim官网.

### Ubuntu18.04

```
sudo apt-get install neovim
```

### MacOS

### CentOs7

### Fedora

```
sudo dnf install -y neovim
```

## Neovm python支持

安装完Neovim后.为了打造Python IDE.我们需要安装Neovim Python模块

在终端输入`nvim`后我们即可进入到Neovim界面.然后输入`:CheckHealth`,找到下图的内容

![](https://pic3.zhimg.com/v2-b928df529709396bf01306c4722d0e16_b.jpg)

通过图片上面信息,我们就可以发现Neovim的Python2和Python3模块没有被安装.按照上面提示输入如下命令即可

```
pip2 install neovim
pip3 install neovim
```

如果出现下面内容,则说明你的Neovim Python模块已经安装完成.

![](https://pic3.zhimg.com/v2-06301b2934d4fa6f47d32616a29acaaa_b.jpg)

## Neovim配置

创建配置文件

```
mkdir ~/.config/nvim
nvim ~/.config/nvim/init.vim
```

## Neovim插件安装

在安装Neovim插件前,我们先安装vim-plug来管理我们的插件.

在terminal输入下面命令即可一键安装vim-plug

```
curl -fLo ~/.vim/autoload/plug.vim --create-dirs \
    https://raw.githubusercontent.com/junegunn/vim-plug/master/plug.vim
```

更多有关vim-plug配置请参考我的另一篇文章[使用Neovim打造Python IDE--Vim-Plug配置介绍](https://link.zhihu.com/?target=http%3A//www.stilesyu.com/index.php/2020/02/07/neovim-vim-plug/)

这是我neovim配置文件供大家参考



> 更多优质的文章请订阅我的公众号【生活少数派】或者访问我的[个人博客](https://link.zhihu.com/?target=https%3A//www.stilesyu.com/)

## 参考资料

**这份配置文件部分我参考了其他博客,但是因为距离写这篇文章过去很长一段时间,很难寻找出处.在此我说声抱歉.如果里面有您的内容,您可以在下方留言,我会补充出处**

[Github-neovim-wiki](https://link.zhihu.com/?target=https%3A//github.com/neovim/neovim/wiki/Introduction) [NeoVim 科普，21世纪的Vim](https://zhuanlan.zhihu.com/p/21364426) [Vim-维基中文](https://link.zhihu.com/?target=https%3A//zh.wikipedia.org/wiki/Vim)