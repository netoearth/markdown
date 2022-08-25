1.  [技术文章](https://article.itxueyuan.com/)
2.  [操作系统](https://article.itxueyuan.com/category/os)
3.  [Linux](https://article.itxueyuan.com/category/linux)
4.  正文

## 1\. 前言

目前最常用的环境还是linux的服务器，所以最终选择的是nvim作为自己的首要编辑器，毕竟没有写一些比较大型的项目。在经过多次的摸索后，我还是选择了Neovim + Coc.nvim，放弃了 YCM。

这里假设在没有root权限的情况，考虑的是软件的源码安装（相比之下，直接用各个发行版的命令安装会更加简单）。

## 2.Neovim 和插件安装

### 2.1 Neovim 安装

neovim：[下载地址](https://github.com/neovim/neovim/releases)

选择最新的release 的版本，  
![](https://imgs.itxueyuan.com/1027174-20200709113415644-606718676.png)  
这里除了source code 是源码外，其他的都是编译好的，直接

```
wget https://github.com/neovim/neovim/releases/download/v0.4.3/nvim-linux64.tar.gz
tar -zxvf nvim-linux64.tar.gz
```

然后把neovim路径下的bin加入到~/.bashrc，然后在source一下就算是成功了。

### 2.2 插件安装

#### 1\. Vim-Plug

[vim-plug](https://github.com/junegunn/vim-plug)是一个我很喜欢的vim的插件管理工具，使用下面的命令可以进行安装（其他平台和工具的安装方法地址中有）：

```
sh -c 'curl -fLo "${XDG_DATA_HOME:-$HOME/.local/share}"/nvim/site/autoload/plug.vim --create-dirs 
       https://raw.githubusercontent.com/junegunn/vim-plug/master/plug.vim'
```

这样，这个 `~/.local/share/nvim/site/autoload/plug.vim`就会在你的目录下，并且vim会被调用。

创建nvim的配置文件（这个配置文件和vim的'.vimrc'）一样：

```
mkdir ~/.config/nvim/
nvim ~/.config/nvim/init.vim
```

然后把

```
call plug#begin('~/.vim/plugged')
call plug#end()
```

加入到`init.vim`中，这样以后在call begin和call end 之间加上插件就可以使用了。

**之后的每个插件在init.vim文件中配置好后，要进行保存退出，再次进入nvim,使用命令 :PlugInstall安装**

#### 2\. indentLine

[indentLine](https://github.com/Yggdroot/indentLine)此插件提供的一个可视化的缩进，把`Plug 'Yggdroot/indentLine'`，放到`init.vim`的call begin和call end之间，同时加入一些简单的配置：

```
let g:indent_guides_guide_size            = 1  " 指定对齐线的尺寸
let g:indent_guides_start_level           = 2  " 从第二层开始可视化显示缩进
```

效果如图：

![](https://imgs.itxueyuan.com/1027174-20200709132541645-1504757987.png)

#### 3\. vim-monokai

[vim-monokai](https://github.com/crusoexia/vim-monokai.git)，这个插件是nvim的一个主题，monokai这个配色是我最喜欢的，从开始用sublime的时候我一直都用的是这个主题。

在 call begin 和 call end 之间加上`Plug 'crusoexia/vim-monokai'`，然后把

```
colo monokai
```

加到后面。

效果：

![](https://imgs.itxueyuan.com/1027174-20200709113519534-1520233245.png)

#### 4.vim-airline

[vim-airline](https://github.com/vim-airline/vim-airline)给nvim 提供一个强大的状态栏和标签栏，当打开多个文本时，可以用它进行快速的切换，是一个很强大的工具。

在 call begin 和 call end 之间加上：

```
Plug 'vim-airline/vim-airline'       
Plug 'vim-airline/vim-airline-themes' "airline 的主题
```

然后在`init.vim`里加上一些个性的配置：

```
" 设置状态栏
let g:airline#extensions#tabline#enabled = 1
let g:airline#extensions#tabline#left_alt_sep = '|'
let g:airline#extensions#tabline#buffer_nr_show = 0
let g:airline#extensions#tabline#formatter = 'default'
let g:airline_theme = 'desertink'  " 主题
let g:airline#extensions#keymap#enabled = 1
let g:airline#extensions#tabline#buffer_idx_mode = 1
let g:airline#extensions#tabline#buffer_idx_format = {
        '0': '0 ',
        '1': '1 ',
        '2': '2 ',
        '3': '3 ',
        '4': '4 ',
        '5': '5 ',
        '6': '6 ',
        '7': '7 ',
        '8': '8 ',
        '9': '9 '
       }
" 设置切换tab的快捷键 <> + <i> 切换到第i个 tab
nmap <leader>1 <Plug>AirlineSelectTab1
nmap <leader>2 <Plug>AirlineSelectTab2
nmap <leader>3 <Plug>AirlineSelectTab3
nmap <leader>4 <Plug>AirlineSelectTab4
nmap <leader>5 <Plug>AirlineSelectTab5
nmap <leader>6 <Plug>AirlineSelectTab6
nmap <leader>7 <Plug>AirlineSelectTab7
nmap <leader>8 <Plug>AirlineSelectTab8
nmap <leader>9 <Plug>AirlineSelectTab9
" 设置切换tab的快捷键 <> + <-> 切换到前一个 tab
nmap <leader>- <Plug>AirlineSelectPrevTab
" 设置切换tab的快捷键 <> + <+> 切换到后一个 tab
nmap <leader>+ <Plug>AirlineSelectNextTab
" 设置切换tab的快捷键 <> + <q> 退出当前的 tab
nmap <leader>q :bp<cr>:bd #<cr>
" 修改了一些个人不喜欢的字符
if !exists('g:airline_symbols')
    let g:airline_symbols = {}
endif
let g:airline_symbols.linenr = "CL" " current line
let g:airline_symbols.whitespace = '|'
let g:airline_symbols.maxlinenr = 'Ml' "maxline
let g:airline_symbols.branch = 'BR'
let g:airline_symbols.readonly = "RO"
let g:airline_symbols.dirty = "DT"
let g:airline_symbols.crypt = "CR" 
```

效果为：

![](https://imgs.itxueyuan.com/1027174-20200709113545811-1962783974.png)

[nerdcommenter](https://github.com/preservim/nerdcommenter)是一个很好用的注释工具，在normal和visual模式下，他会对你光标所在行或所选中的多行进行注释和去注释，只需要你按下 <>+<c>+<space>。

在 call begin 和 call end 之间加上`Plug 'scrooloose/nerdcommenter'`，在进行简单的配置

```
"add spaces after comment delimiters by default
let g:NERDSpaceDelims = 1
" python 自动的会多加一个空格
au FileType python let g:NERDSpaceDelims = 0

" Use compact syntax for prettified multi-line comments
let g:NERDCompactSexyComs = 1

" Align line-wise comment delimiters flush left instead of following code indentation
let g:NERDDefaultAlign = 'left'

" Set a language to use its alternate delimiters by default
let g:NERDAltDelims_java = 1

" Add your own custom formats or override the defaults
" let g:NERDCustomDelimiters = { 'c': { 'left': '/**','right': '*/' } }

" Allow commenting and inverting empty lines (useful when commenting a region)
let g:NERDCommentEmptyLines = 1

" Enable trimming of trailing whitespace when uncommenting
let g:NERDTrimTrailingWhitespace = 1

" Enable NERDCommenterToggle to check all selected lines is commented or not
let g:NERDToggleCheckAllLines = 1
```

#### 6\. rainbow

[rainbow](https://github.com/luochen1990/rainbow)是一个提供嵌套括号高亮的一个工具。

在 call begin 和 call end 之间加上`Plug 'luochen1990/rainbow'`  
我对括号的颜色进行了简单的修改：

```
let g:rainbow_active = 1
let g:rainbow_conf = {
   'guifgs': ['darkorange3', 'seagreen3', 'royalblue3', 'firebrick'],
   'ctermfgs': ['lightyellow', 'lightcyan','lightblue', 'lightmagenta'],
   'operators': '_,_',
   'parentheses': ['start=/(/ end=/)/ fold', 'start=/[/ end=/]/ fold', 'start=/{/ end=/}/ fold'],
   'separately': {
       '*': {},
       'tex': {
           'parentheses': ['start=/(/ end=/)/', 'start=/[/ end=/]/'],
       },
       'lisp': {
           'guifgs': ['darkorange3', 'seagreen3', 'royalblue3', 'firebrick'],
       },
       'vim': {
           'parentheses': ['start=/(/ end=/)/', 'start=/[/ end=/]/', 'start=/{/ end=/}/ fold', 'start=/(/ end=/)/ containedin=vimFuncBody', 'start=/[/ end=/]/ containedin=vimFuncBody', 'start=/{/ end=/}/ fold containedin=vimFuncBody'],
       },
       'html': {
           'parentheses': ['start=/v<((area|base|br|col|embed|hr|img|input|keygen|link|menuitem|meta|param|source|track|wbr)[ >])@!z([-_:a-zA-Z0-9]+)(s+[-_:a-zA-Z0-9]+(=("[^"]*"|'."'".'[^'."'".']*'."'".'|[^ '."'".'"><=`]*))?)*>/ end=#</z1># fold'],
       },
       'css': 0,
   }
}
```

效果：  
![](https://imgs.itxueyuan.com/1027174-20200709113600096-1418580234.png)

#### 7\. nerdtree

[nerdtree](https://github.com/preservim/nerdtree)是一个树形的目录管理插件，可以方便在nvim中进行当前文件夹中的文件切换

在 call begin 和 call end 之间加上

```
Plug 'preservim/nerdtree'
Plug 'Xuyuanp/nerdtree-git-plugin'
```

在 call begin 和 call end 之间加上`'preservim/nerdtree'`

进行以下的简单配置

```
" autocmd vimenter * NERDTree  "自动开启Nerdtree
let g:NERDTreeWinSize = 25 "设定 NERDTree 视窗大小
let NERDTreeShowBookmarks=1  " 开启Nerdtree时自动显示Bookmarks
"打开vim时如果没有文件自动打开NERDTree
" autocmd vimenter * if !argc()|NERDTree|endif
"当NERDTree为剩下的唯一窗口时自动关闭
autocmd bufenter * if (winnr("$") == 1 && exists("b:NERDTree") && b:NERDTree.isTabTree()) | q | endif
" 设置树的显示图标
let g:NERDTreeDirArrowExpandable = '+'
let g:NERDTreeDirArrowCollapsible = '-'
let NERDTreeIgnore = ['.pyc$']  " 过滤所有.pyc文件不显示
let g:NERDTreeShowLineNumbers=0 " 是否显示行号
let g:NERDTreeHidden=0     "不显示隐藏文件
""Making it prettier
let NERDTreeMinimalUI = 1
let NERDTreeDirArrows = 1
nnoremap <F3> :NERDTreeToggle<CR> " 开启/关闭nerdtree快捷键
```

默认的情况下是关闭的，可以把`autocmd vimenter * NERDTree`的注释打开，改成默认开启的状态。同时把开关的按键映射到了F3，按下可以打开或关闭。

效果：

![](https://imgs.itxueyuan.com/1027174-20200709113643029-1442004731.png)

#### 8\. tagbar

[tagbar](https://github.com/majutsushi/tagbar)可以用来展示当前的文件的一些函数。

这个插件依赖ctags，需要先安装ctags。从[官网](http://ctags.sourceforge.net/)下载源码，进行编译（如果有root权限直接安装，跳过源码编译）。

```
wget http://prdownloads.sourceforge.net/ctags/ctags-5.8.tar.gz
tar -zxvf ctags-5.8.tar.gz
cd ctags-5.8
./configure --prefix=$PATH # $PATH是你要安装的位置
make -j
make install
```

然后把安装目录里的bin所在的路径加到`~/.bashrc`中，在source一下，输入命令`ctags`，如果输出

```
ctags: No files specified. Try "ctags --help". 
```

那ctags就安装成功了。

然后在`init.vim`中的call begin和call end 之间加入`Plug 'majutsushi/tagbar'`，我这里还设置了tagbar的宽度和按键映射，这样可以通过F4方便的进行打开和关上。在tagbar窗口可以按下回车跳转到指定的函数。

```
let g:tagbar_width=30
nnoremap <silent> <F4> :TagbarToggle<CR> " 将tagbar的开关按键设置为 F4
```

![](https://imgs.itxueyuan.com/1027174-20200709113701958-71124301.png)

#### 9\. vim-cpp-enhanced-highlight

[![](http://imgs.itxueyuan.com/imgs/1-4321-20220707113253)](https://time.geekbang.org/activity/promo?page_name=page_417)

[vim-cpp-enhanced-highlight](https://github.com/octol/vim-cpp-enhanced-highlight)，这个是用来加强C++的高亮的，很少用C++，没仔细看

```
Plug 'octol/vim-cpp-enhanced-highlight'
```

#### 10\. vim-snippets

[honza/vim-snippets](https://github.com/zprhhs/config/blob/master/nvim_vim/init.vim)我用来提高C++的snippets，python 那边使用的coc.nvim提供的，感觉这个东西挺复杂的，也没咋看，但是能用就行。

```
Plug 'honza/vim-snippets'
```

#### 11\. coc.nvim

[coc.nvim](https://github.com/neoclide/coc.nvim)是目前我用过的最舒服的、集代码补全、静态检测、函数跳转等功能的一个引擎，强力推荐！！！

安装coc.nvim，需要安装nodejs。和nvim一样，这个也有编译好的版本，解压后把bin的路径放到放到.bashrc里，之后运行

```
npm install -g neovim
```

安装

```
Plug 'neoclide/coc.nvim', {'branch': 'release'}
```

注意的是，低版本的git会在后面进行更新coc.nvim插件的时候报错，报的啥我也忘了，反正如果更新的时候发现有git的错误，升级一下git就行了。

下面的设置是在他的主页上参考的。

```
" if hidden is not set, TextEdit might fail.
set hidden
" Some servers have issues with backup files, see #649
set nobackup
set nowritebackup

" You will have bad experience for diagnostic messages when it's default 4000.
set updatetime=300

" don't give |ins-completion-menu| messages.
set shortmess+=c

" always show signcolumns
set signcolumn=yes

" Use tab for trigger completion with characters ahead and navigate.
" Use command ':verbose imap <tab>' to make sure tab is not mapped by other plugin.
inoremap <silent><expr> <TAB>
       pumvisible() ? "<C-n>" :
       <SID>check_back_space() ? "<TAB>" :
       coc#refresh()
inoremap <expr><S-TAB> pumvisible() ? "<C-p>" : "<C-h>"

function! s:check_back_space() abort
  let col = col('.') - 1
  return !col || getline('.')[col - 1]  =~# 's'
endfunction

" Use <c-space> to trigger completion.
inoremap <silent><expr> <c-space> coc#refresh()

" Use <cr> to confirm completion, `<C-g>u` means break undo chain at current position.
" Coc only does snippet and additional edit on confirm.
inoremap <expr> <cr> pumvisible() ? "<C-y>" : "<C-g>u<CR>"
" Or use `complete_info` if your vim support it, like:
" inoremap <expr> <cr> complete_info()["selected"] != "-1" ? "<C-y>" : "<C-g>u<CR>"

" Use `[g` and `]g` to navigate diagnostics
nmap <silent> [g <Plug>(coc-diagnostic-prev)
nmap <silent> ]g <Plug>(coc-diagnostic-next)
" Remap keys for gotos
nmap <silent> gd <Plug>(coc-definition)
nmap <silent> gy <Plug>(coc-type-definition)
nmap <silent> gi <Plug>(coc-implementation)
nmap <silent> gr <Plug>(coc-references)

" Use K to show documentation in preview window
nnoremap <silent> K :call <SID>show_documentation()<CR>

function! s:show_documentation()
  if (index(['vim','help'], &filetype) >= 0)
    execute 'h '.expand('<cword>')
  else
    call CocAction('doHover')
  endif
endfunction

" Highlight symbol under cursor on CursorHold
autocmd CursorHold * silent call CocActionAsync('highlight')

" Remap for rename current word
nmap <leader>rn <Plug>(coc-rename)

" Remap for format selected region
xmap <leader>f  <Plug>(coc-format-selected)
nmap <leader>f  <Plug>(coc-format-selected)

augroup mygroup
  autocmd!
  " Setup formatexpr specified filetype(s).
  autocmd FileType typescript,json setl formatexpr=CocAction('formatSelected')
  " Update signature help on jump placeholder
  autocmd User CocJumpPlaceholder call CocActionAsync('showSignatureHelp')
augroup end

" Remap for do codeAction of selected region, ex: `<leader>aap` for current paragraph
xmap <leader>a  <Plug>(coc-codeaction-selected)
nmap <leader>a  <Plug>(coc-codeaction-selected)

" Remap for do codeAction of current line
nmap <leader>ac  <Plug>(coc-codeaction)
" Fix autofix problem of current line
nmap <leader>qf  <Plug>(coc-fix-current)

" Create mappings for function text object, requires document symbols feature of languageserver.
xmap if <Plug>(coc-funcobj-i)
xmap af <Plug>(coc-funcobj-a)
omap if <Plug>(coc-funcobj-i)
omap af <Plug>(coc-funcobj-a)

" Use `:Format` to format current buffer
command! -nargs=0 Format :call CocAction('format')

" Use `:Fold` to fold current buffer
command! -nargs=? Fold :call     CocAction('fold', <f-args>)

" use `:OR` for organize import of current buffer
command! -nargs=0 OR   :call     CocAction('runCommand', 'editor.action.organizeImport')
```

这样，coc.nvim 平台安装完成了，此外，为了提高多种功能，还要进行相关插件的安装，这个放到后面的环境配置里。

#### 12\. nvim的基本配置

```
filetype plugin on
" 设置为双字宽显示，否则无法完整显示如:☆
set ambiwidth=double
set t_ut= " 防止vim背景颜色错误
set showmatch " 高亮匹配括号
set matchtime=1
set report=0
set ignorecase
set nocompatible
set noeb
set softtabstop=4
set shiftwidth=4
set nobackup
set autoread
set nocompatible
set nu "设置显示行号
set backspace=2 "能使用backspace回删
syntax on "语法检测
set ruler "显示最后一行的状态
set laststatus=2 "两行状态行+一行命令行
set ts=4
set expandtab
set autoindent "设置c语言自动对齐
set t_Co=256 "指定配色方案为256
" set mouse=a "设置可以在VIM使用鼠标
set selection=exclusive
" set selectmode=mouse,key
set tabstop=4 "设置TAB宽度
set history=1000 "设置历史记录条数   
" 配色方案
" let g:seoul256_background = 234
colo monokai
set background=dark
set shortmess=atl
" colorscheme desert
"共享剪切板
set clipboard+=unnamed 
set cmdheight=3
if version >= 603
     set helplang=cn
     set encoding=utf-8
endif
set fencs=utf-8,ucs-bom,shift-jis,gb18030,gbk,gb2312,cp936
set termencoding=utf-8
set encoding=utf-8
set fileencodings=ucs-bom,utf-8,cp936
set fileencoding=utf-8
set updatetime=300
set shortmess+=c
set signcolumn=yes

" autocmd FileType json syntax match Comment +//.+$+

set foldmethod=indent " 设置默认折叠方式为缩进
set foldlevelstart=99 " 每次打开文件时关闭折叠

" hi Normal ctermfg=252 ctermbg=none "背景透明
" au FileType gitcommit,gitrebase let g:gutentags_enabled=0
if has("autocmd")
    au BufReadPost * if line("'"") > 1 && line("'"") <= line("$") | exe "normal! g'"" | endif
endif
inoremap jj <Esc> "将jj映射到Esc
```

## 3\. Coc.nvim 环境配置

对于Python，直接装anaconda就行了，同时需要用pip装一下pynvim。

```
pip install pynvim
```

在nvim输入`:checkhealth`，会进行基本环境的检查，保证你用的环境不会出问题即可，我不用python2和ruby，这两是warning就无所谓了。

上面的环境安装结束后，并不能直接使用，还需要进一步的配置coc.nvim。  
coc.nvim 提供了很多有用的扩展工具在[coc extensions](https://github.com/neoclide/coc.nvim/wiki/Using-coc-extensions)中，这些扩展在nvim中通过命令`:CocInstall`进行安装，下面是我安装的一个表：

```
:CocInstall coc-clangd # C++环境插件
:CocInstall coc-cmake  # Cmake 支持
:CocInstall coc-emmet  
:CocInstall coc-git    # git 支持
:CocInstall coc-highlight  # 高亮支持
:CocInstall coc-jedi   # jedi
:CocInstall coc-json   # json 文件支持
:CocInstall coc-python # python 环境支持
:CocInstall coc-sh     # bash 环境支持
:CocInstall coc-snippets # python提供 snippets
:CocInstall coc-vimlsp # lsp
:CocInstall coc-yaml   # yaml
:CocInstall coc-syntax
:CocInstall coc-pairs
```

对于C++，coc.nvim 会自动进行配置的，如果你没有安装clang的话，插件coc-clangd在你第一次打开C++文件时，会提示

```
[coc.nvim] clangd was not found on your PATH. :CocCommand clangd.install will install 10.0.0.
```

按照提示进行，如果不行的话，那只能去装一个clang了，这里参考的是[安装 LLVM 和 Clang](https://www.jianshu.com/p/861c1a630059)：  
如果你有root的权限，可以参考官方的的安装方法更快捷的安装：[http://clangd.llvm.org/installation.html](http://clangd.llvm.org/installation.html)

```
mkdir llvm-source-build
wget https://github.com/llvm/llvm-project/releases/download/llvmorg-9.0.1/llvm-9.0.1.src.tar.xz
tar xvf llvm-9.0.1.src.tar.xz
mv llvm-9.0.1.src llvm
cd llvm/tools/
wget https://github.com/llvm/llvm-project/releases/download/llvmorg-9.0.1/clang-9.0.1.src.tar.xz
tar xvf clang-9.0.1.src.tar.xz
rm clang-9.0.1.src.tar.xz
mv clang-9.0.1.src clang
cd clang/tools/
wget https://github.com/llvm/llvm-project/releases/download/llvmorg-9.0.1/clang-tools-extra-9.0.1.src.tar.xz
tar xvf clang-tools-extra-9.0.1.src.tar.xz
mv clang-tools-extra-9.0.1.src extra
rm clang-tools-extra-9.0.1.src.tar.xz
cd ../../../
mkdir build
cd build/
cmake -G "Unix Makefiles" -DCMAKE_BUILD_TYPE=Release -DLLVM_TARGETS_TO_BUILD="X86" ../ -DCMAKE_INSTALL_PREFIX=$PATH # 改成你的安装路径
make -j
make install
```

测试一下clang和clangd两个命令，检查是否安装成功。  
对于Python的配置，安装好coc-python就算是结束了，进入.py文件的话如果发现jedi报错的话，重新安装一下jedi即可，可能是jedi的版本太低。  
最后，我把代码检测里的那个warning的符号改了， 本来是一个黄色的三角图标，我担心对终端不友好，改成了两个叹号。  
coc.nvim的配置文件位置为：`~/.config/nvim/coc-settings.json`。

```
{
  "diagnostic.warningSign":"!!"
}
```

这样就都配置好了。  
在snaipppts的提示中，可以通过 Ctrl + j 进行移动修改（从https://github.com/neoclide/coc-snippets弄来的，侵删）

![](https://imgs.itxueyuan.com/1027174-20200709141421774-14267257.gif)

在函数里，可以通过按下gd进行函数的跳转，ctrl + o 回调到上次的位置，效果如下图：

![](https://imgs.itxueyuan.com/1027174-20200709131919799-1417378364.gif)  
最后放一下自己目前在用的`init.vim(.vimrc)`（和上面介绍的有改动）:

```
call plug#begin('~/.config/vim_plug/plugged')
    Plug 'honza/vim-snippets'
    Plug 'sainnhe/sonokai'
    Plug 'vim-python/python-syntax'
    Plug 'vim-scripts/ctags.vim'
    " Plug 'crusoexia/vim-monokai'
    Plug 'tmux-plugins/vim-tmux-focus-events'
    Plug 'vim-airline/vim-airline'
    Plug 'scrooloose/nerdcommenter'
    Plug 'vim-airline/vim-airline-themes'
    Plug 'luochen1990/rainbow'
    Plug 'octol/vim-cpp-enhanced-highlight'
    Plug 'Yggdroot/indentLine'
    Plug 'preservim/nerdtree'
    Plug 'Xuyuanp/nerdtree-git-plugin'
    Plug 'neoclide/coc.nvim', {'branch': 'release'}
    Plug 'majutsushi/tagbar'
call plug#end()

filetype plugin on
" 设置为双字宽显示，否则无法完整显示如:☆
set ambiwidth=double
set t_ut= " 防止vim背景颜色错误
set showmatch " 高亮匹配括号
set matchtime=1
set noshowmode " block mode display
set novisualbell noerrorbells
set report=0
set ignorecase
set cursorline "highlight current line
set noeb
set softtabstop=4
set shiftwidth=4
set nobackup
set autoread
set nocompatible
set nu "设置显示行号
set backspace=2 "能使用backspace回删
syntax on "语法检测
set ruler "显示最后一行的状态
set laststatus=2 "两行状态行+一行命令行
set ts=4
set expandtab
set autoindent "设置c语言自动对齐
set t_Co=256 "指定配色方案为256
" set mouse=a "设置可以在VIM使用鼠标
set selection=exclusive
" set selectmode=mouse,key
set tabstop=4 "设置TAB宽度
set history=1000 "设置历史记录条数
set shortmess=atl
set clipboard+=unnamed
set cmdheight=1
if version >= 603
    set helplang=cn
    set encoding=utf-8
endif
set fencs=utf-8,ucs-bom,shift-jis,gb18030,gbk,gb2312,cp936
set termencoding=utf-8
set encoding=utf-8
set fileencodings=ucs-bom,utf-8,cp936
" set updatetime=300
set shortmess+=c
set signcolumn=yes
" reset cursor when vim exits
au VimLeave * set guicursor=a:ver25-blinkon0

" +================================ 可视化缩进 =====================================+ "
" set notermguicolors
let g:sonokai_style = 'shusia'
" let g:sonokai_enable_italic = 1
" let g:sonokai_disable_italic_comment = 1
colo sonokai
set background=dark
hi CursorLine ctermfg=NONE ctermbg=NONE cterm=NONE
hi Normal ctermfg=NONE ctermbg=234 guifg=NONE
hi LineNr ctermfg=yellow ctermbg=236 cterm=NONE
" hi Terminal ctermfg=NONE ctermfg=NONE cterm=NONE
hi EndOfBuffer ctermbg=NONE
" visual mode bg
set foldmethod=indent " 设置默认折叠方式为缩进
set foldlevelstart=99 " 每次打开文件时关闭折叠

" hi Normal ctermfg=252 ctermbg=none "背景透明
" au FileType gitcommit,gitrebase let g:gutentags_enabled=0
if has("autocmd")
    au BufReadPost * if line("'"") > 1 && line("'"") <= line("$") | exe "normal! g'"" | endif
endif

let g:python_highlight_all = 1
" +================================ 可视化缩进 =====================================+ "

let g:indent_guides_enable_on_vim_startup = 0  " 默认关闭
let g:indent_guides_guide_size            = 1  " 指定对齐线的尺寸
let g:indent_guides_start_level           = 2  " 从第二层开始可视化显示缩进

" +================================== NERDTree =======================================+ "
" autocmd vimenter * NERDTree  "自动开启Nerdtree
let g:NERDTreeWinSize = 25 "设定 NERDTree 视窗大小
let NERDTreeShowBookmarks=1  " 开启Nerdtree时自动显示Bookmarks
"打开vim时如果没有文件自动打开NERDTree
" autocmd vimenter * if !argc()|NERDTree|endif
"当NERDTree为剩下的唯一窗口时自动关闭
autocmd bufenter * if (winnr("$") == 1 && exists("b:NERDTree") && b:NERDTree.isTabTree()) | q | endif
" 设置树的显示图标
let g:NERDTreeDirArrowExpandable = '+'
let g:NERDTreeDirArrowCollapsible = '-'
let NERDTreeIgnore = ['.pyc$']  " 过滤所有.pyc文件不显示
let g:NERDTreeShowLineNumbers=0 " 是否显示行号
let g:NERDTreeHidden=0     "不显示隐藏文件
""Making it prettier
let NERDTreeMinimalUI = 1
let NERDTreeDirArrows = 1
" autocmd vimenter *  NERDTreeToggle

" +================================== 按键映射 =======================================+ "

" self key map:
" <leader>s : open key
" <leader>d : close key
" <leader>e : norm key
inoremap <silent> jj <Esc> "将jj映射到Esc
nnoremap <silent> <F4> :TagbarToggle<CR> " 将tagbar的开关按键设置为 F4
nmap <leader>sp :set paste<CR>i
nmap <leader>dp :set nopaste<CR> 
nnoremap ; :
xnoremap ; :
nnoremap <F3> :NERDTreeToggle<CR> " 开启/关闭nerdtree快捷键
nnoremap j gj
nnoremap k gk
nnoremap ^ g^
nnoremap $ g$
nnoremap <leader>dh :noh<CR>

" +================================== coc.nvim  ======================================+ "
" if hidden is not set, TextEdit might fail.
set hidden
" Some servers have issues with backup files, see #649
set nobackup
set nowritebackup

" You will have bad experience for diagnostic messages when it's default 4000.
set updatetime=300

" don't give |ins-completion-menu| messages.
set shortmess+=c

" always show signcolumns
set signcolumn=yes

" Use tab for trigger completion with characters ahead and navigate.
" Use command ':verbose imap <tab>' to make sure tab is not mapped by other plugin.
inoremap <silent><expr> <TAB>
       pumvisible() ? "<C-n>" :
       <SID>check_back_space() ? "<TAB>" :
       coc#refresh()
inoremap <expr><S-TAB> pumvisible() ? "<C-p>" : "<C-h>"

function! s:check_back_space() abort
  let col = col('.') - 1
  return !col || getline('.')[col - 1]  =~# 's'
endfunction

" Use <c-space> to trigger completion.
inoremap <silent><expr> <c-space> coc#refresh()

" Use <cr> to confirm completion, `<C-g>u` means break undo chain at current position.
" Coc only does snippet and additional edit on confirm.
inoremap <expr> <cr> pumvisible() ? "<C-y>" : "<C-g>u<CR>"
" Or use `complete_info` if your vim support it, like:
" inoremap <expr> <cr> complete_info()["selected"] != "-1" ? "<C-y>" : "<C-g>u<CR>"

" Use `[g` and `]g` to navigate diagnostics
nmap <silent> [g <Plug>(coc-diagnostic-prev)
nmap <silent> ]g <Plug>(coc-diagnostic-next)
" Remap keys for gotos
nmap <silent> gd <Plug>(coc-definition)
nmap <silent> gy <Plug>(coc-type-definition)
nmap <silent> gi <Plug>(coc-implementation)
nmap <silent> gr <Plug>(coc-references)

" Use K to show documentation in preview window
nnoremap <silent> K :call <SID>show_documentation()<CR>

function! s:show_documentation()
  if (index(['vim','help'], &filetype) >= 0)
    execute 'h '.expand('<cword>')
  else
    call CocAction('doHover')
  endif
endfunction

" Highlight symbol under cursor on CursorHold
autocmd CursorHold * silent call CocActionAsync('highlight')

" Remap for rename current word
nmap <leader>rn <Plug>(coc-rename)

" Remap for format selected region
xmap <leader>f  <Plug>(coc-format-selected)
nmap <leader>f  <Plug>(coc-format-selected)

augroup mygroup
  autocmd!
  " Setup formatexpr specified filetype(s).
  autocmd FileType typescript,json setl formatexpr=CocAction('formatSelected')
  " Update signature help on jump placeholder
  autocmd User CocJumpPlaceholder call CocActionAsync('showSignatureHelp')
augroup end

" Remap for do codeAction of selected region, ex: `<leader>aap` for current paragraph
xmap <leader>a  <Plug>(coc-codeaction-selected)
nmap <leader>a  <Plug>(coc-codeaction-selected)

" Remap for do codeAction of current line
nmap <leader>ac  <Plug>(coc-codeaction)
" Fix autofix problem of current line
nmap <leader>qf  <Plug>(coc-fix-current)

" Create mappings for function text object, requires document symbols feature of languageserver.
xmap if <Plug>(coc-funcobj-i)
xmap af <Plug>(coc-funcobj-a)
omap if <Plug>(coc-funcobj-i)
omap af <Plug>(coc-funcobj-a)

" Use `:Format` to format current buffer
command! -nargs=0 Format :call CocAction('format')

" Use `:Fold` to fold current buffer
command! -nargs=? Fold :call     CocAction('fold', <f-args>)

" use `:OR` for organize import of current buffer
command! -nargs=0 OR   :call     CocAction('runCommand', 'editor.action.organizeImport')
"
" inoremap <silent><expr> <TAB>
"        pumvisible() ? coc#_select_confirm() :
"        coc#expandableOrJumpable() ? "<C-r>=coc#rpc#request('doKeymap', ['snippets-expand-jump',''])<CR>" :
"        <SID>check_back_space() ? "<TAB>" :
"        coc#refresh()
"
" function! s:check_back_space() abort
"   let col = col('.') - 1
"   return !col || getline('.')[col - 1]  =~# 's'
" endfunction
"
" let g:coc_snippet_next = '<tab>'
" +=================================== tagbar =======================================+ "

let g:tagbar_width=30
" +================================== airline =======================================+ "
let g:airline#extensions#tabline#enabled = 1
let g:airline#extensions#tabline#left_alt_sep = '|'
let g:airline#extensions#tabline#buffer_nr_show = 0
let g:airline#extensions#tabline#formatter = 'default'
let g:airline_theme = 'desertink'
let g:airline#extensions#keymap#enabled = 1
let g:airline#extensions#tabline#buffer_idx_mode = 1
let g:airline#extensions#tabline#buffer_idx_format = {
        '0': '0 ',
        '1': '1 ',
        '2': '2 ',
        '3': '3 ',
        '4': '4 ',
        '5': '5 ',
        '6': '6 ',
        '7': '7 ',
        '8': '8 ',
        '9': '9 '
       }
nmap <leader>1 <Plug>AirlineSelectTab1
nmap <leader>2 <Plug>AirlineSelectTab2
nmap <leader>3 <Plug>AirlineSelectTab3
nmap <leader>4 <Plug>AirlineSelectTab4
nmap <leader>5 <Plug>AirlineSelectTab5
nmap <leader>6 <Plug>AirlineSelectTab6
nmap <leader>7 <Plug>AirlineSelectTab7
nmap <leader>8 <Plug>AirlineSelectTab8
nmap <leader>9 <Plug>AirlineSelectTab9
nmap <leader>- <Plug>AirlineSelectPrevTab
nmap <leader>= <Plug>AirlineSelectNextTab
nmap <leader>q :bp<cr>:bd #<cr>
if !exists('g:airline_symbols')
    let g:airline_symbols = {}
endif
let g:airline_symbols.linenr = "CL" " current line
let g:airline_symbols.whitespace = ''
" let g:airline_left_sep = ']'
" let g:airline_left_alt_sep = ')'
" let g:airline_right_sep = '['
" let g:airline_right_alt_sep = '('
let g:airline_symbols.maxlinenr = 'Ml' "maxline
let g:airline_symbols.branch = 'BR'
let g:airline_symbols.readonly = "RO"
let g:airline_symbols.dirty = "DT"
let g:airline_symbols.crypt = "CR"
" +=============================== NERD Commenter ====================================+ "
"add spaces after comment delimiters by default
let g:NERDSpaceDelims = 1
au FileType python let g:NERDSpaceDelims = 0

" Use compact syntax for prettified multi-line comments
let g:NERDCompactSexyComs = 1

" Align line-wise comment delimiters flush left instead of following code indentation
let g:NERDDefaultAlign = 'left'

" Set a language to use its alternate delimiters by default
let g:NERDAltDelims_java = 1

" Add your own custom formats or override the defaults
" let g:NERDCustomDelimiters = { 'c': { 'left': '/**','right': '*/' } }

" Allow commenting and inverting empty lines (useful when commenting a region)
let g:NERDCommentEmptyLines = 1

" Enable trimming of trailing whitespace when uncommenting
let g:NERDTrimTrailingWhitespace = 1

" Enable NERDCommenterToggle to check all selected lines is commented or not
let g:NERDToggleCheckAllLines = 1


" +=============================== rainbow Parentheses ==============================+ "
" ((((((()))))))
let g:rainbow_active = 1
let g:rainbow_conf = {
   'guifgs': ['#FFE66F', '#00FFFF', '#46A3FF', '#AAAAFF', '#FFB5B5'],
   'ctermfgs': ['lightyellow', 'lightcyan','lightblue', 'lightmagenta'],
   'operators': '_,_',
   'parentheses': ['start=/(/ end=/)/ fold', 'start=/[/ end=/]/ fold', 'start=/{/ end=/}/ fold'],
}
```

内容来源于网络如有侵权请私信删除

### 相关课程

[![](https://imgs.itxueyuan.com/coursePicture/large-20171008223011-course-103-picure.jpg)](https://course.itxueyuan.com/103)

[![](https://imgs.itxueyuan.com/coursePicture/large-20180921180800-course-140-picure.jpg)](https://course.itxueyuan.com/140)