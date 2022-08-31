## Neovim 与 vim 的恩怨情仇

2014 年，巴西程序员 Thiago de Arruda Padilha（aka tarruda）因为多次对 vim 提交 feature patch 遭到拒绝。出于对 vim 低效的开发社区的不满，决定众筹一个新项目 —— neovim，尝试解决 vim 当时的问题：

1.  由于 vim 写于 90 年代，20 多年过去，产生了大量的遗留代码，导致程序维护困难
2.  社区新功能开发进度缓慢，vim 开发社区仍然使用邮件 patch 的方式协作，并且对新人极度不友好
3.  作者 Bram Moolenaar 被形容为 vim 社区的独裁者，大量开发者提交的 patch 被拒绝

Neovim 项目发起之初，并不被人们看好，并且被认为是在重复造轮子。vim 作者 Bram 这样评价刚发起的 neovim 项目：

It’s going to be an awful lot of work, with the result that not all systems will be supported, new bugs introduced and what’s the gain for the end user exactly?

但是随着时间的推移，neovim 项目逐渐发展成为一个成熟的项目，并率先提供了多个当时 vim 不支持的新特性：

-   remote plugin，支持使用 python 等第三方语言编写的程序与 nvim 交互，开发插件
-   为 vimscript 提供了异步任务的支持，在此之前 vimscirpt 只能以同步的方式工作，任务卡住会导致 vim 前台卡住
-   支持在 vim 中打开 terminal window
-   重构了 vim 的部分代码，如使用 libuv 库来做多平台兼容，而不是像 vim 那样手动维护，并且使用更加现代化的代码编译工具链

neovim 项目的成功也激发了 bram 对 vim 项目开发的激情，促使 vim 在 7.0 之后极大的加快了新功能开发进度，很快发布了 vim8.0/8.1，把 neovim 实现的大部分新特性在 vim 中也实现了一遍。vim 现在也支持异步任务，terminal 等特性了。所以目前来看 neovim 与 vim 的差异已经很小，大部分第三方插件都能兼容 nvim/vim。

但是在这里还是强烈推荐使用 neovim：

1.  我从 neovim 0.17 版本开始使用（macOS），使用下来，nvim 的稳定性和 vim 基本相当
2.  neovim 始终保持先进性，由于社区开发进度更高效，各种新功能仍然还是先在 neovim 中实现，vim 才会有对应实现，并且现在 neovim 还被 google summer of code 支持，为其添加新特性
3.  neovim 还有一些未被 vim 实现的特性，例如 Remote plugin, virtual text 等..

关于 vim/neovim 的更多比较，各位还可以查阅更多网上的资料。这里还有一篇文章讲述 vim codebase 的问题供参考：链接

## 利用 nvim/vim 新特性，打造更现代化的编辑器

随着 nvim/vim 对异步任务的支持，很多原先 vim 中被大量使用的插件已经逐渐变得过时，这里列举一些更加「先进」的插件，可以作为古老 vim 插件的替代品。

## 文件管理

使用 `Shougo/defx.nvim` 替代 `scrooloose/nerdtree，defx.nvim` 使用 neovim 的 Remote plugin，通过 python3 开发，支持异步，在文件多的情况下打开文件浏览器的速度更加快速。作者 Shougo 是一个高产的 vim 插件作者，同时开发了 denite 等著名插件。但是他写的插件特点就是并不开箱即用，需要大量的配置。这里我列出了我的 defx 配置，同时使用了 `kristijanhusak/defx-git` 和 `kristijanhusak/defx-icons`（需要安装 nerd font）来显示 git 修改和图标。

配置：

```
Plug 'Shougo/defx.nvim', { 'do': ':UpdateRemotePlugins' }
map <silent> - :Defx<CR>
" Avoid the white space highting issue
autocmd FileType defx match ExtraWhitespace /^^/
" Keymap in defx
autocmd FileType defx call s:defx_my_settings()
function! s:defx_my_settings() abort
  IndentLinesDisable
  setl nospell
  setl signcolumn=no
  setl nonumber
  nnoremap <silent><buffer><expr> <CR>
  \ defx#is_directory() ?
  \ defx#do_action('open_or_close_tree') :
  \ defx#do_action('drop',)
  nmap <silent><buffer><expr> <2-LeftMouse>
  \ defx#is_directory() ?
  \ defx#do_action('open_or_close_tree') :
  \ defx#do_action('drop',)
  nnoremap <silent><buffer><expr> s defx#do_action('drop', 'split')
  nnoremap <silent><buffer><expr> v defx#do_action('drop', 'vsplit')
  nnoremap <silent><buffer><expr> t defx#do_action('drop', 'tabe')
  nnoremap <silent><buffer><expr> o defx#do_action('open_tree')
  nnoremap <silent><buffer><expr> O defx#do_action('open_tree_recursive')
  nnoremap <silent><buffer><expr> C defx#do_action('copy')
  nnoremap <silent><buffer><expr> P defx#do_action('paste')
  nnoremap <silent><buffer><expr> M defx#do_action('rename')
  nnoremap <silent><buffer><expr> D defx#do_action('remove_trash')
  nnoremap <silent><buffer><expr> A defx#do_action('new_multiple_files')
  nnoremap <silent><buffer><expr> U defx#do_action('cd', ['..'])
  nnoremap <silent><buffer><expr> . defx#do_action('toggle_ignored_files')
  nnoremap <silent><buffer><expr> <Space> defx#do_action('toggle_select')
  nnoremap <silent><buffer><expr> R defx#do_action('redraw')
endfunction

" Defx git
Plug 'kristijanhusak/defx-git'
let g:defx_git#indicators = {
  \ 'Modified'  : '✹',
  \ 'Staged'    : '✚',
  \ 'Untracked' : '✭',
  \ 'Renamed'   : '➜',
  \ 'Unmerged'  : '═',
  \ 'Ignored'   : '☒',
  \ 'Deleted'   : '✖',
  \ 'Unknown'   : '?'
  \ }
let g:defx_git#column_length = 0
hi def link Defx_filename_directory NERDTreeDirSlash
hi def link Defx_git_Modified Special
hi def link Defx_git_Staged Function
hi def link Defx_git_Renamed Title
hi def link Defx_git_Unmerged Label
hi def link Defx_git_Untracked Tag
hi def link Defx_git_Ignored Comment

" Defx icons
" Requires nerd-font, install at https://github.com/ryanoasis/nerd-fonts or
" brew cask install font-hack-nerd-font
" Then set non-ascii font to Driod sans mono for powerline in iTerm2
Plug 'kristijanhusak/defx-icons'
" disbale syntax highlighting to prevent performence issue
let g:defx_icons_enable_syntax_highlight = 1
```

显示效果如下： [![](https://liaoph.com/img/in-post/modern-vim/defx.png)](https://liaoph.com/img/in-post/modern-vim/defx.png)

## fzf

https://github.com/junegunn/fzf.vim

fzf 是一个 fuzzy search 工具，相比与 ctrlp，它能提供更好的性能，并且扩展性更好，可以集成到其他插件中。类似与 ctrlp 它能提供文件搜索功能，同时还能提供 ctags 代码 symbol 搜索，代码内容搜索等功能。 fzf 的配置相对简单，这里只贴一些的效果图：

文件模糊搜索： [![](https://liaoph.com/img/in-post/modern-vim/fzf-file.gif)](https://liaoph.com/img/in-post/modern-vim/fzf-file.gif) fzf 使用了 terminal + command 的实现方式，可以对其功能进行扩展，例如结合 vim-go 插件搜索代码中的 code symbol： [![](https://liaoph.com/img/in-post/modern-vim/fzf-btags.gif)](https://liaoph.com/img/in-post/modern-vim/fzf-btags.gif) 除了fzf， Shougo 开发的 denite.nvim 也是一个非常流行的 fuzzy search 插件，但是 denite 的配置就更加复杂，这里就进行赘述了。

## 代码补全

在过去，比较流行的补全插件有 `ycm-core/YouCompleteMe` 和 `Shougo/deoplete.nvim`，但是这些插件对不同语言的支持程度都不尽相同，每个语言的补全可能都需要单独配置。其中 YCM 采用 C++ 开发了额外程序，每次更新还需要进行编译。配置，使用，调试都是非常费力且折腾的事情。

随着微软发力开发开源代码编辑器 vscode，同时发布了 Language Server Protocol，这种混乱的局面正在逐渐变得标准化和统一。现在每个语言基本都有对应的 LSP Server 实现。使用 LSP 协议的好处在于，编辑器是需要实现 LSP Client 就可以和 LSP Server 交互，而不需要 care 具体是什么语言。

coc.nvim 就是这样一个采用 LSP 实现的 vim 插件。同时他还利用了 neovim 的 Remote plugin 功能，使用 typescript 开发，能够最小成本的将已有的 vscode 插件进行少量修改适配，即可移植到 vim 中来。

coc.nvim 同时是一个插件化的系统，通过很多众多的插件，还能提供代码补全之外的额外功能。而且 coc.nvim 的开发速度非常快，已经支持了 neovim 刚刚 master 分支上实现的 floating window 功能（这个功能未来也会在 vim 中进行对应实现）。

这里列举一些常用功能：

快速查看函数签名，支持 floating window 展示： [![](https://liaoph.com/img/in-post/modern-vim/coc-k.gif)](https://liaoph.com/img/in-post/modern-vim/coc-k.gif) 这个功能直接映射一个快捷键即可，这里映射为 K：

```
" Use K to show documentation in preview window
function! s:show_documentation()
  if (index(['vim','help'], &filetype) >= 0)
    execute 'h '.expand('<cword>')
  else
    call CocAction('doHover')
  endif
endfunction
nnoremap <silent> K :call <SID>show_documentation()<CR>
```

在补全中提示函数的参数列表： [![](https://liaoph.com/img/in-post/modern-vim/coc-func-param.gif)](https://liaoph.com/img/in-post/modern-vim/coc-func-param.gif) 执行代码检查，并将结果通过 flaoting window 展示： [![](https://liaoph.com/img/in-post/modern-vim/coc-diag.gif)](https://liaoph.com/img/in-post/modern-vim/coc-diag.gif) coc.nvim 还能够安装扩展支持很多额外功能，例如 git 信息显示，括号自动补全，调用第三方 sinppets 等功能。具体的安装配置文档可以参考：coc wiki

## 配置参考

由于篇幅限制，这里只列出一些我常用的一些插件，我所有的 vim 配置文件都在 github 上可以查看，完整的插件和配置可以参考：https://github.com/paco0x/dotfiles/tree/master/vim

## 本文转载于坚实的梦想，如有侵权请联系删除。