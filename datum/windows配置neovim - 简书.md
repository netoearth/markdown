## windows配置neovim

[![](https://upload.jianshu.io/users/upload_avatars/1316672/9970451603e2?imageMogr2/auto-orient/strip|imageView2/1/w/96/h/96/format/webp)](https://www.jianshu.com/u/ef85a8194c9d)

0.3052021.01.24 00:24:38字数 101阅读 5,821

最近在windows下将gvim换成了neovim，配置过程如下

1.  安装neovim

我在windows下使用scoop，所以只需要运行以下命令安装neovim

2.  创建neovim的配置文件

```
ni -type file -force C:\Users\juw1179\AppData\Local\nvim\init.vim
```

3.  我的init.vim的配置文件如下

```
set nocompatible              " be iMproved, required
filetype off                  " required

" set the runtime path to include Vundle and initialize
set rtp+=~/.vim/bundle/Vundle.vim
call vundle#begin()

Plugin 'VundleVim/Vundle.vim'
Plugin 'Lokaltog/vim-powerline'
Plugin '[[Markdown]]'
Plugin 'PProvost/vim-ps1'
Plugin 'MatlabFilesEdition'
Plugin 'posva/vim-vue'
Plugin 'mattn/emmet-vim'
Plugin 'JuliaEditorSupport/julia-vim'
Plugin 'iCyMind/NeoSolarized'

call vundle#end()            " required
filetype plugin indent on    " required
"
" Remember the position of cursor
set viminfo='800,<3000
au BufReadPost * if line("'\"") > 0|if line("'\"") <= line("$")|exe("norm '\"")|else|exe "norm $"|endif|endif

set cursorline
"hi CursorLine ctermbg=darkgrey
set cursorcolumn
set autoindent
set wrap
set nu
set tabstop=2
set shiftwidth=2
set expandtab
set fileencodings=gb18030,gbk,gb2312,utf-8
set termencoding=utf-8
set incsearch " incremental search
set laststatus=2 
set list lcs=tab:\|\ 
syntax enable

" Set Solarized Color Scheme
"let g:solarized_termcolors=256
"set t_Co=256
"set background=dark
colorscheme NeoSolarized
set termguicolors
set hlsearch
set nobackup
```

4.  安装vundle

```
git clone https://github.com/VundleVim/Vundle.vim.git ~/.vim/bundle/Vundle.vim
```

4.  安装插件  
    打开neovim(`nvim`)并运行以下命令

5.  设置文件可以用右键nvim打开  
    编辑注册表

```
Windows Registry Editor Version 5.00
[HKEY_CURRENT_USER\SOFTWARE\Classes\*\shell\Open with Neovim]

[HKEY_CURRENT_USER\SOFTWARE\Classes\*\shell\Open with Neovim\command]
nvim "%1"
```

搞定收工

更多精彩内容，就在简书APP

"小礼物走一走，来简书关注我"

还没有人赞赏，支持一下

[![  ](https://upload.jianshu.io/users/upload_avatars/1316672/9970451603e2?imageMogr2/auto-orient/strip|imageView2/1/w/100/h/100/format/webp)](https://www.jianshu.com/u/ef85a8194c9d)

### 推荐阅读[更多精彩内容](https://www.jianshu.com/)

-   前言 vim-plug 是一个非常优秀的 Vim 插件管理器，但是随着安装的插件越来越多，逐渐发现即使使用 vim...
    
    [![](https://upload.jianshu.io/users/upload_avatars/2222997/500ed1fcf8e2?imageMogr2/auto-orient/strip|imageView2/1/w/48/h/48/format/webp)Whyn](https://www.jianshu.com/u/2baf4dc0fbe6)阅读 4,433评论 0赞 6
    
-   01\. VIM 配置 02. Neovim 配置 03. IdeaVim 进阶配置 04. VsVim 配置 前言...
    
    [![](https://upload.jianshu.io/users/upload_avatars/2222997/500ed1fcf8e2?imageMogr2/auto-orient/strip|imageView2/1/w/48/h/48/format/webp)Whyn](https://www.jianshu.com/u/2baf4dc0fbe6)阅读 17,783评论 2赞 2
    
    [![](https://upload-images.jianshu.io/upload_images/2222997-4eace460f011b255.png?imageMogr2/auto-orient/strip|imageView2/1/w/300/h/240/format/webp)](https://www.jianshu.com/p/88dd20795263)

-   01\. VIM 配置 02. Neovim 配置 03. IdeaVim 进阶配置 04. VsVim 配置 前言...
    
    [![](https://upload.jianshu.io/users/upload_avatars/2222997/500ed1fcf8e2?imageMogr2/auto-orient/strip|imageView2/1/w/48/h/48/format/webp)Whyn](https://www.jianshu.com/u/2baf4dc0fbe6)阅读 3,528评论 0赞 2
    
    [![](https://upload-images.jianshu.io/upload_images/2222997-46f040d2fe5db675.png?imageMogr2/auto-orient/strip|imageView2/1/w/300/h/240/format/webp)](https://www.jianshu.com/p/9536ec0d59a3)
-   windows10下gvim安装deoplete.nvim插件 本文内容主要用作个人备忘，大家选择性参考。 参考链...
    
-   win7 + gvim 打造Python IDE 考虑到有的软件下载地址不能正确访问(原因你懂的)，本文中用到的所...
    
    [![](https://upload.jianshu.io/users/upload_avatars/2891888/4bd50d50-489b-4e12-aeb0-8b933d0bde56.png?imageMogr2/auto-orient/strip|imageView2/1/w/48/h/48/format/webp)尔牛](https://www.jianshu.com/u/9c11ec3db3eb)阅读 6,676评论 3赞 9
    
    [![](https://upload-images.jianshu.io/upload_images/2891888-ca76909b5fdfdc9b.png?imageMogr2/auto-orient/strip|imageView2/1/w/300/h/240/format/webp)](https://www.jianshu.com/p/edd8aad3adc7)
-   [![](https://upload-images.jianshu.io/upload_images/833208-0e025afe22953a64.jpeg?imageMogr2/auto-orient/strip|imageView2/1/w/300/h/240/format/webp)](https://www.jianshu.com/p/06041b29d7fe)
-   版本：ios 1.2.1 亮点： 1.app角标可以实时更新天气温度或选择空气质量，建议处女座就不要选了，不然老想...
    
    [![](https://upload-images.jianshu.io/upload_images/396469-33cff237a20b7ab1.png?imageMogr2/auto-orient/strip|imageView2/1/w/300/h/240/format/webp)](https://www.jianshu.com/p/cb44563a2f9f)
-   我是一名过去式的高三狗，很可悲，在这三年里我没有恋爱，看着同龄的小伙伴们一对儿一对儿的，我的心不好受。怎么说呢，高...
    
    [![](https://cdn2.jianshu.io/assets/default_avatar/9-cceda3cf5072bcdd77e8ca4f21c40998.jpg)小娘纸](https://www.jianshu.com/u/17b36e153d48)阅读 2,382评论 3赞 5
    
-   这些日子就像是一天一天在倒计时 一想到他走了 心里就是说不出的滋味 从几个月前认识他开始 就意识到终究会发生的 只...
    
    [![](https://cdn2.jianshu.io/assets/default_avatar/9-cceda3cf5072bcdd77e8ca4f21c40998.jpg)栗子a](https://www.jianshu.com/u/b2d4cc87478f)阅读 1,159评论 1赞 2
    
    [![](https://upload-images.jianshu.io/upload_images/817171-f2ee63948d7b4347.jpeg?imageMogr2/auto-orient/strip|imageView2/1/w/300/h/240/format/webp)](https://www.jianshu.com/p/b5eb6402a7cd)