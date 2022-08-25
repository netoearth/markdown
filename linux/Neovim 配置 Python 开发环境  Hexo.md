### [](https://ncfun.gitee.io/2019/11/06/Neovim%E9%85%8D%E7%BD%AEPython%E5%BC%80%E5%8F%91%E7%8E%AF%E5%A2%83/#%E5%AE%89%E8%A3%85 "安装")安装

### [](https://ncfun.gitee.io/2019/11/06/Neovim%E9%85%8D%E7%BD%AEPython%E5%BC%80%E5%8F%91%E7%8E%AF%E5%A2%83/#%E6%B7%BB%E5%8A%A0Python%E6%94%AF%E6%8C%81 "添加Python支持")添加`Python`支持

`NeoVim`原生是不带各种语言支持的，需要自己去安装和关联

### [](https://ncfun.gitee.io/2019/11/06/Neovim%E9%85%8D%E7%BD%AEPython%E5%BC%80%E5%8F%91%E7%8E%AF%E5%A2%83/#%E6%B7%BB%E5%8A%A0Python%E8%B7%AF%E5%BE%84 "添加Python路径")添加`Python`路径

编辑`init.vim`

bash

```
let g:python3_host_prog = '/Library/Frameworks/Python.framework/Versions/3.7/bin/python3'
```

注意：虚拟环境一定要是绝对路径！不能用`~/`这样的

### [](https://ncfun.gitee.io/2019/11/06/Neovim%E9%85%8D%E7%BD%AEPython%E5%BC%80%E5%8F%91%E7%8E%AF%E5%A2%83/#%E6%A3%80%E6%9F%A5%E7%8A%B6%E6%80%81 "检查状态")检查状态

正确输出：

![image](https://ncfun.gitee.io/2019/11/06/Neovim%E9%85%8D%E7%BD%AEPython%E5%BC%80%E5%8F%91%E7%8E%AF%E5%A2%83/14006958-ffd96f334bf46fb8.png)

**image**

## [](https://ncfun.gitee.io/2019/11/06/Neovim%E9%85%8D%E7%BD%AEPython%E5%BC%80%E5%8F%91%E7%8E%AF%E5%A2%83/#Neovim%E8%AE%BE%E7%BD%AE "Neovim设置")Neovim设置

bash

```
set nocompatible "关闭与vi的兼容模式
set number  "显示绝对行号
set rnu  "显示相对行号
set nowrap    "不自动折行
set showmatch    "显示匹配的括号
set scrolloff=3        "距离顶部和底部3行
set encoding=utf-8  "通用的 utf8 编码，避免乱码
set fenc=utf-8      "编码
set mouse=a        "启用鼠标
set hlsearch        "搜索高亮
set cc=80 "标尺线

set foldmethod=indent  " 代码折叠
set foldcolumn=0            "设置折叠区域的宽度
setlocal foldlevel=1        "设置折叠层数为
set foldlevelstart=99       "打开文件是默认不折叠代码
noremap <space> za  "设置快捷键为空格

syntax on    "语法高亮
filetype plugin indent on  "开启自动识别文件类型，并根据文件类型加载不同的插件和缩进规则
set clipboard=unnamed  "yy直接复制到系统剪切板（For macvim）
cmap w!! w !sudo tee >/dev/null %  "w!!写入只读文件

nnoremap <C-j> 4j  "ctr + j 每次移动4行
nnoremap <C-k> 4k
```

## [](https://ncfun.gitee.io/2019/11/06/Neovim%E9%85%8D%E7%BD%AEPython%E5%BC%80%E5%8F%91%E7%8E%AF%E5%A2%83/#%E6%8F%92%E4%BB%B6%E5%AE%89%E8%A3%85 "插件安装")插件安装

### [](https://ncfun.gitee.io/2019/11/06/Neovim%E9%85%8D%E7%BD%AEPython%E5%BC%80%E5%8F%91%E7%8E%AF%E5%A2%83/#%E5%AE%89%E8%A3%85%E6%8F%92%E4%BB%B6%E7%AE%A1%E7%90%86%E5%99%A8 "安装插件管理器")安装插件管理器

推荐 [vim-plug](https://github.com/junegunn/vim-plug)

bash

```
curl -fLo ~/.local/share/nvim/site/autoload/plug.vim --create-dirs \
https://raw.githubusercontent.com/junegunn/vim-plug/master/plug.vim
```

### [](https://ncfun.gitee.io/2019/11/06/Neovim%E9%85%8D%E7%BD%AEPython%E5%BC%80%E5%8F%91%E7%8E%AF%E5%A2%83/#%E6%BF%80%E6%B4%BB%E6%8F%92%E4%BB%B6%E7%AE%A1%E7%90%86%E5%99%A8 "激活插件管理器")激活插件管理器

配置文件添加：

bash

```
call plug#begin('~/.config/nvim/plugged')
  Plug 'Yggdroot/indentLine'  "缩进指示线
call plug#end()
```

### [](https://ncfun.gitee.io/2019/11/06/Neovim%E9%85%8D%E7%BD%AEPython%E5%BC%80%E5%8F%91%E7%8E%AF%E5%A2%83/#%E9%85%8D%E8%89%B2%E4%B8%BB%E9%A2%98%EF%BC%9Agruvbox "配色主题：gruvbox")配色主题：[gruvbox](https://github.com/morhetz/gruvbox)

bash

```
Plug 'morhetz/gruvbox'
```

设置

### [](https://ncfun.gitee.io/2019/11/06/Neovim%E9%85%8D%E7%BD%AEPython%E5%BC%80%E5%8F%91%E7%8E%AF%E5%A2%83/#%E8%87%AA%E5%8A%A8%E8%A1%A5%E5%85%A8%EF%BC%9Adeoplete-deoplete-jedi "自动补全：deoplete + deoplete-jedi")自动补全：[deoplete](https://github.com/Shougo/deoplete.nvim) + [deoplete-jedi](https://github.com/deoplete-plugins/deoplete-jedi)

bash

```
Plug 'Shougo/deoplete.nvim', { 'do': ':UpdateRemotePlugins' }
Plug 'deoplete-plugins/deoplete-jedi'
```

`deoplete` 是一款自动补全的引擎，具体对某种编程语言的自动补全支持，要安装对应的source 才能真正工作。  
`deoplete-jedi` 是 Python 的 Source，需要补全其它语言在[这里](https://github.com/Shougo/deoplete.nvim/wiki/Completion-Sources)查找相应的资源。

插件设置：

```
" 自启动
let g:deoplete#enable_at_startup = 1

" 自动补全提示默认 ctrl-n 下翻页，改成 tab
inoremap <expr><tab> pumvisible() ? "\<c-n>" : "\<tab>"

" 自动补全提示默认 ctrl-p 下翻页，改成 s-tab
inoremap <expr><S-tab> pumvisible() ? "\<c-p>" : "\<tab>"

" 函数方法 Preview 的窗口自动关闭
autocmd InsertLeave,CompleteDone * if pumvisible() == 0 | pclose | endif

" Preview 窗口设为在当前窗口下面打开
set splitbelow
```

`deoplete-jedi` 默认配置就很好了

### [](https://ncfun.gitee.io/2019/11/06/Neovim%E9%85%8D%E7%BD%AEPython%E5%BC%80%E5%8F%91%E7%8E%AF%E5%A2%83/#%E4%BB%A3%E7%A0%81%E9%AB%98%E4%BA%AE%EF%BC%9Asemshi "代码高亮：semshi")代码高亮：[semshi](https://github.com/numirias/semshi)

bash

```
Plug 'numirias/semshi', {'do': ':UpdateRemotePlugins'}
```

### [](https://ncfun.gitee.io/2019/11/06/Neovim%E9%85%8D%E7%BD%AEPython%E5%BC%80%E5%8F%91%E7%8E%AF%E5%A2%83/#%E8%87%AA%E5%8A%A8%E7%BC%A9%E8%BF%9B%EF%BC%9Aindentpython "自动缩进：indentpython")自动缩进：[indentpython](https://github.com/vim-scripts/indentpython)

bash

```
Plug 'vim-scripts/indentpython.vim'
```

### [](https://ncfun.gitee.io/2019/11/06/Neovim%E9%85%8D%E7%BD%AEPython%E5%BC%80%E5%8F%91%E7%8E%AF%E5%A2%83/#%E8%87%AA%E5%8A%A8%E8%A1%A5%E5%85%A8%E6%8B%AC%E5%8F%B7%EF%BC%9Aauto-pairs "自动补全括号：auto-pairs")自动补全括号：[auto-pairs](https://github.com/jiangmiao/auto-pairs)

bash

```
Plug 'jiangmiao/auto-pairs' " 自动补全括号和引号等
```

### [](https://ncfun.gitee.io/2019/11/06/Neovim%E9%85%8D%E7%BD%AEPython%E5%BC%80%E5%8F%91%E7%8E%AF%E5%A2%83/#%E7%8A%B6%E6%80%81%E6%A0%8F%EF%BC%9Avim-airline "状态栏：vim-airline")状态栏：[vim-airline](https://github.com/vim-airline/vim-airline)

bash

```
Plug 'vim-airline/vim-airline' " 状态栏
```

`vim-airline`提供了很多主题来个性化状态栏，不同主题的样子可以参见[这里](https://github.com/vim-airline/vim-airline/wiki/Screenshots)。  
更改vim-airline 主题方式很简单，首先安装插件[vim-airlinetheme](https://github.com/vim-airline/vim-airline-themes):

bash

```
Plug 'vim-airline/vim-airline-themes'
```

然后在 Nvim 配置文件中，加入以下设置：

bash

```
let g:airline_theme='<theme>' " <theme> 代表某个主题的名称
```

### [](https://ncfun.gitee.io/2019/11/06/Neovim%E9%85%8D%E7%BD%AEPython%E5%BC%80%E5%8F%91%E7%8E%AF%E5%A2%83/#%E6%B3%A8%E9%87%8A%E6%8F%92%E4%BB%B6%EF%BC%9Anerdcommenter "注释插件：nerdcommenter")注释插件：[nerdcommenter](https://github.com/scrooloose/nerdcommenter)

bash

```
Plug 'scrooloose/nerdcommenter'
```

插件设置：

bash

```
" 注释符后面自动添加空格
let g:NERDSpaceDelims = 1

" 取消注释后删除注释符后的空格
let g:NERDTrimTrailingWhitespace = 1

" 启用NERDCommenterToggle以检查所有选定的行是否已注释
let g:NERDToggleCheckAllLines = 1

" 允许注释和倒空行（在注释区域时很有用）
let g:NERDCommentEmptyLines = 1
```

### [](https://ncfun.gitee.io/2019/11/06/Neovim%E9%85%8D%E7%BD%AEPython%E5%BC%80%E5%8F%91%E7%8E%AF%E5%A2%83/#%E4%BB%A3%E7%A0%81%E6%A3%80%E6%9F%A5%E5%B7%A5%E5%85%B7%EF%BC%9AAle "代码检查工具：Ale")代码检查工具：[Ale](https://github.com/dense-analysis/ale)

bash

```
Plug 'dense-analysis/ale'
```

插件设置：

bash

```
let g:ale_sign_error = '>>'  " 自定义错误标志
let g:ale_sign_warning = '!'  " 自定义警告标志
" 指定修复 pep8 错误的 fixer
let g:ale_fixers = {'python': ['remove_trailing_lines', 'trim_whitespace', 'autopep8']}
nnoremap <C-S-l> :ALEFix<CR>  "修复语法和格式错误 ctr + shift + l"
```

### [](https://ncfun.gitee.io/2019/11/06/Neovim%E9%85%8D%E7%BD%AEPython%E5%BC%80%E5%8F%91%E7%8E%AF%E5%A2%83/#%E6%96%87%E4%BB%B6%E6%A0%91%EF%BC%9Anerdtree "文件树：nerdtree")文件树：[nerdtree](https://github.com/Xuyuanp/git-nerdtree)

bash

```
Plug 'scrooloose/nerdtree'
```

### [](https://ncfun.gitee.io/2019/11/06/Neovim%E9%85%8D%E7%BD%AEPython%E5%BC%80%E5%8F%91%E7%8E%AF%E5%A2%83/#%E6%98%BE%E7%A4%BA-Git-%E5%8F%98%E6%9B%B4%E7%8A%B6%E6%80%81%EF%BC%9Anerdtree-git-plugin "显示 Git 变更状态：nerdtree-git-plugin")显示 Git 变更状态：[nerdtree-git-plugin](https://github.com/Xuyuanp/nerdtree-git-plugin)

bash

```
Plug 'Xuyuanp/nerdtree-git-plugin'
```