**Matrix 首页推荐** 

[Matrix](https://sspai.com/matrix) 是少数派的写作社区，我们主张分享真实的产品体验，有实用价值的经验与思考。我们会不定期挑选 Matrix 最优质的文章，展示来自用户的最真实的体验和观点。 

文章代表作者个人观点，少数派仅对标题和排版略作修改。

___

过去半年，我认真地体验了 Obsidian（黑曜石）这款软件，深度使用了一段时间之后便果断退订了 Ulysses，将全部的文档都迁移到了 Obsidian。由于其平台的开放性，Obsidian 的定位不仅仅是笔记软件，更适合作为 PKM 工具使用。

关于 Obsidian 使用介绍的文章和视频已经数不胜数，通过阅读这些优秀的经验分享，让我快速掌握 Obsidian 的使用技巧。之前在 Obsidian 官方论坛里看到一篇文章 「[VIM Mode - Quality of Life Improvements](https://sspai.com/link?target=https%3A%2F%2Fforum.obsidian.md%2Ft%2Fvim-mode-quality-of-life-improvements%2F429)」，介绍了在 Obsidian 里如何高效地使用 Vim Mode ，而 Obsidian 吸引我的其中最重要的一点就是支持 Vim Mode 。过去我一直在各种 IDE 里使用 Vim Mode，已经熟悉了 Vim Mode 操作习惯和便利，仍记得在支持 Vim Mode 的 Markdown 笔记软件里，我还使用过一段时间的 [Haroopad](https://sspai.com/link?target=http%3A%2F%2Fpad.haroopress.com%2F)，可惜软件很久没有更新了。Obsidian 经过 2 年多迭代和各类 Vim Mode 周边插件支持，目前支持的 Vim Mode 能力已经能够我满足日常笔记和写作的需要，这篇文章就总结一下目前我是如何在 Obsidian 配置和使用 Vim Mode 以及对其周边插件进行完善。

### 1\. Vimrc Support

想深度使用 Vim Mode 一定会用到 Vimrc Support 这个插件，配合自定义 .obsidian.vimrc 文件，实现丰富的 Vim Mode 键盘映射管理。经过各种摸索，我自己的配置文件目前如下：[.obsidian.vimrc](https://sspai.com/link?target=https%3A%2F%2Fgist.github.com%2Fjiyee%2Fcfa8dc2f37359004b34820543f691db3)，接下来展开来介绍一下里面具体的配置。

#### 按行跳转光标

```
    " Have j and k navigate visual lines rather than logical ones
    nmap j gj
    nmap k gk

    " I like using H and L for beginning/end of line
    nmap H ^
    nmap L $
    " nmap J 5j
    " nmap K 5k
```

#### 按段落跳转光标

```
    " Moving to next/prev paragraph
    nmap [ {
    nmap ] }
```

#### 按 Heading 跳转光标

```
    " Moving to next/prev heading
    exmap nextHeading jsfile .obsidian.markdown-helper.js {jumpHeading(true)}
    exmap prevHeading jsfile .obsidian.markdown-helper.js {jumpHeading(false)}
    nmap g] :nextHeading
    nmap g[ :prevHeading
```

#### 按 Tab 跳转光标

```
    " Emulate Tab Switching https://vimhelp.org/tabpage.txt.html#gt
    " requires Pane Relief: https://github.com/pjeby/pane-relief
    exmap tabnext obcommand pane-relief:go-next
    nmap gt :tabnext
    exmap tabprev obcommand pane-relief:go-prev
    nmap gT :tabprev
    " Same as CMD+\
    nmap g\ :tabnext
```

#### 光标跳转到上次编辑的位置

另外一个常用的功能是光标快速回退到上次编辑的位置，在 Obsidian 官方论坛里也有用户提出需求「[Hotkey jump to previous caret position](https://sspai.com/link?target=https%3A%2F%2Fforum.obsidian.md%2Ft%2Fhotkey-jump-to-previous-caret-position%2F10817)」，找到一个很巧妙的实现， `nmap g; u<C-r>`，限制是只能回退一步，且 Editing/Preview 模式切换之后会失灵。另外，Vim Mode 已经内置支持 `gi` 跳转到上次编辑的位置并切换到 insert 模式。

#### `zz` 键盘映射屏幕移动位置至 70%

`zz` 键盘映射默认会将文档当前行移动到屏幕中间位置，但是我习惯将当前编辑行移动到屏幕中上的位置，结合 jsfile 稍微调整一下 `zz` 屏幕移动位置。

```
    exmap scrollToCenterTop70p jsfile .obsidian.markdown-helper.js {scrollToCursor(0.7)}
    nmap zz :scrollToCenterTop70p
```

#### Surround Text with surround

Vimrc Support 插件支持了 surround 特性，进一步搭配 [Admonition](https://sspai.com/link?target=https%3A%2F%2Fgithub.com%2Fvalentine195%2Fobsidian-admonition) 和 [code block from selections](https://sspai.com/link?target=https%3A%2F%2Fgithub.com%2Fdy-sh%2Fobsidian-code-block-from-selection) 插件，无论在 normal mode 或 visual mode 下都可以通过 `sa{x}` 指令直接创建 admonition block 。

```
    " Surround Admonition
    " https://github.com/esm7/obsidian-vimrc-support/discussions/146
    exmap CodeBlockAdmonitionNote obcommand code-block-from-selection:17f30753-d5f4-4953-abed-5027a25ede58
    map san :CodeBlockAdmonitionNote
    exmap CodeBlockSelectionAdmonitionNote jscommand { editor.setSelections([selection]); this.app.commands.commands['code-block-from-selection:17f30753-d5f4-4953-abed-5027a25ede58'].callback() }
    vmap san :CodeBlockSelectionAdmonitionNote
```

![](https://cdn.sspai.com/2023/01/31/bf1e80938c7aee397571278a6b753dd1.gif)

键盘映射 san 直接创建 Admonition Block

#### 搭配 [Zoom](https://sspai.com/link?target=https%3A%2F%2Fgithub.com%2Fvslinko%2Fobsidian-zoom) 插件，按 Heading 进行 Zoom In 和 Zoom Out

```
    " Zoom in/out
    exmap zoomIn obcommand obsidian-zoom:zoom-in
    exmap zoomOut obcommand obsidian-zoom:zoom-out
    nmap zi :zoomIn
    nmap zo :zoomOut

    nmap &a :zoomOut
    nmap &b :nextHeading
    nmap &c :zoomIn
    nmap &d :prevHeading
    nmap z] &a&b&c
    nmap z[ &a&d&c
```

![](https://cdn.sspai.com/2023/01/31/24d8939ae33e42cbc2a61a80b937d11e.gif)

键盘映射 zi 按 Heading 进行 Zoom In

#### 搭配 [Stille](https://sspai.com/link?target=https%3A%2F%2Fgithub.com%2Fmichaellee%2Fstille) 插件，切换 Focus 模式

```
    exmap toggleStille obcommand obsidian-stille:toggleStille
    nmap zs :toggleStille
    nmap ,s :toggleStille
```

![](https://cdn.sspai.com/2023/01/31/922096645abad05f01ccd7c10b4164e7.gif)

键盘映射 zs 切换到 Focus 模式

#### 直接在指定 IDE 打开 .obsidiain.vimrc 配置文件

在开发过程中，需要经常编辑 .obsidian.vimrc 配置文件，为了快速地在 Sublime Text 中打开配置文件，我找到了一个 workaround 的方法，利用 [Shell commands](https://sspai.com/link?target=https%3A%2F%2Fgithub.com%2FTaitava%2Fobsidian-shellcommands) 插件配置注册一个命令行 `/usr/local/bin/subl {{vault_path}}/.obsidian.vimrc` 命令，就可以通过命令直接唤起 Sublime Text 并打开 .obsidian.vimrc 文件。

![](https://cdn.sspai.com/2023/01/31/bebbf07ba210e99eea2d1d76e00d42e2.png)

#### Vim Mode 状态栏显示异常 Bugfix

Vimrc Support 插件 0.8.0 版本存在新打开文件之后 Vim Mode 在状态栏显示异常的问题，我提了第一个 [PR](https://sspai.com/link?target=https%3A%2F%2Fgithub.com%2Fesm7%2Fobsidian-vimrc-support%2Fpull%2F159) ，在最新的 0.9.0 版本修复了此 bug ，还顺带修复了 Fixed Normal Mode Layout 功能。提交完 PR 之后，在 GitHub 上也开始尝试解答部分 issue。

### 2\. Obsidian Vim Reading View Navigation

这个插件实现了在 Preview 模式下 `j/k/gg/G` 等光标跳转指令，不过该插件没有发布到 Community plugins 市场，只能手动安装。[obsidian-vim-reading-view-navigation Allows navigating Obsidian's Reading View with j, k, gg and G](https://sspai.com/link?target=https%3A%2F%2Fgithub.com%2Fkometenstaub%2Fobsidian-vim-reading-view-navigation)

### 3\. Obsidian Vim Yank Highlighter

高亮 `yy` 复制行，同样没有发布到 Community plugins 市场，只能手动安装。[obsidian-vim-yank-highlight Highlights the current yank](https://sspai.com/link?target=https%3A%2F%2Fgithub.com%2Fkometenstaub%2Fobsidian-vim-yank-highlight)

## 中文编辑体验优化

常见的 Vim Mode 实现里对于中文的支持一般都是非常差的，「[Vim 的中文支持及解决思路](https://sspai.com/post/71322)」这篇文章总结了在 Vim 中文编辑的问题和解决思路。得益于 Obsidian 的插件开放性，使得针对中文编辑体验进行专项优化成为可能。

### 1\. 中英文输入法自动切换

使用 Vim Mode 中文写作，第一个遇到的瓶颈问题就是中英文输入法切换，在切换到 insert 模式自动切换到中文输入法，回到 normal 模式自动切换到英文输入法。目前，已经有一个插件  [obsidian-vim-im-switch-plugin](https://sspai.com/link?target=https%3A%2F%2Fgithub.com%2Fyuanotes%2Fobsidian-vim-im-switch-plugin) 实现了该功能，需要搭配 [fcitx-remote-for-osx](https://sspai.com/link?target=https%3A%2F%2Fgithub.com%2Fxcodebuild%2Ffcitx-remote-for-osx) 这个命令行工具使用。我直接修改了源文件 main.m，替换默认输入法配置（com.apple.keylayout.ABC 和 im.rime.inputmethod.Squirrel.Rime），重新编译安装即可使用。不过该插件存在 bug，经常出现切换失灵的问题，提了 [PR](https://sspai.com/link?target=https%3A%2F%2Fgithub.com%2Fyuanotes%2Fobsidian-vim-im-switch-plugin%2Fpull%2F19) 修复，暂时还没有合入主干发布新版本。

![](https://cdn.sspai.com/2023/01/31/178ea1cf7aa6e2a958289737949ed282.gif)

insert/normal 模式切换和输入法切换同步

### 2\. 按中文分词跳转光标

在 Vim Mode 使用过程中会遇到的另一个瓶颈问题是对于中文分词的逻辑跟英文不一样，「[Vim 的中文支持及解决思路](https://sspai.com/post/71322)」这篇文章里详细讲述了可能的三种解决思路。在 Community plugins 市场里找了一圈也没有现成的解决方案，不过发现有一个插件 [cm-chs-patch](https://sspai.com/link?target=https%3A%2F%2Fgithub.com%2Faidenlx%2Fcm-chs-patch) 实现了中文分词和双击选中的功能，于是花了一点时间在此基础上实现了 Vim Mode 按中文分词跳转光标的能力（[支持 Vim 模式 by jiyee · Pull Request 19](https://sspai.com/link?target=https%3A%2F%2Fgithub.com%2Faidenlx%2Fcm-chs-patch)[）。最新的 1.10.0 版本已](https://sspai.com/link?target=)经支持按中文分词跳转光标。不过，因为不能直接修改 Obsidian 里内置的 vim.js 内部实现，只能将 vim.js 的内部实现 fork 出来，好在相关实现属于比较稳定的部分，之后的升级适配应该不存在太大的困难。

![](https://cdn.sspai.com/2023/01/31/ea13e099300da2ab2ad7377ad19209f9.gif)

按中文分词跳转光标

### 3\. 按中文标点跳转光标

此外，在中文编辑过程中经常需要按断句跳转或编辑，开发支持了 `f<charactor>` 和 `t<charactor>` 键盘映射输入英文标点跳转到对应的中文标点，例如在英文输入法状态下，输入 `f,` 会跳转到下一个 `,` 或者 `，`，我自己觉得这个特性还蛮实用的，在分词粒度和段落粒度之间补齐了语句粒度。因为实现相似，同样提交到了 [cm-chs-patch](https://sspai.com/link?target=https%3A%2F%2Fgithub.com%2Faidenlx%2Fcm-chs-patch) 插件，不过我觉得放在的这个插件里并不是最恰当的，只是当时提 PR 的时候一起带上了。

![](https://cdn.sspai.com/2023/01/31/cb8ba042a326c7fc843089c572344700.gif)

在英文输入法状态下，输入英文标点跳转到对应的中文标点

## 自定义插件开发

此外，根据我自己日常使用中的习惯还依赖部分特性没有很好的支持，我正在着手开发一个 [Obsidian-VimEx](https://sspai.com/link?target=https%3A%2F%2Fgithub.com%2Fjiyee%2FObsidian-VimEx) 插件，目前包括以下几个特性：

### File Rename Modal

之前通过定义 `gr` 和 `<C-w>r` 键盘映射到文件重命名功能，在启用 「Show inline title」选项之后，默认的 Rename file 行为会是跳转到文档头部的标题块，且标题文字处于全选状态，因为标题修改是一个高频的行为，就实现了一个简单的文件重命名弹窗，确保文件重命名动作不会干扰到现有的编辑状态。

```
" rename file modal
exmap renameFile obcommand Obsidian-VimEx:file-rename-modal
nmap gr :renameFile
```

![](https://cdn.sspai.com/2023/01/31/b41536059a4507008493203452b476c3.gif)

键盘映射 gr 文件重命名弹窗

### Mark 标记

我之前一直很少用 Vim Mode 里的 Mark 功能，主要的原因是经常忘记 Mark 的标记名称。如果能直接在编辑器里可视化标记 Mark 标记名称或者有一个 Mark 标记名称列表的话，对于跨段落的来回跳转会方便很多。目前已经实现的是，在行号右侧添加自定义的 Mark 标记名称，会联动 `m{x}` 键盘映射。

![](https://cdn.sspai.com/2023/01/31/a9307292a158f8492b7e41ea9f4d44bf.gif)

键盘映射 m{x} 同步展示标记名称

#### .obsidian.vimrc 可视化配置（计划中）

配置 .obsidian.vimrc 过程中发现想找到一个合适的参考配置其实挺不方便的，只有通过 GitHub 或论坛搜索的方式。如果能可视化配置，将全部特性都列出来，直接配置指令即可，甚至能够以社区共享配置的话就更方便了。这个可能会作为一个功能加到插件里。

___

最后，还有其他特性需求或建议欢迎留言或者提 [issue](https://sspai.com/link?target=https%3A%2F%2Fgithub.com%2Fjiyee%2FObsidian-VimEx%2Fissues)，共同完善 Obsidian + Vim Mode 体验。

#### 关联阅读

-   [Vim 的中文支持及解决思路](https://sspai.com/post/71322)

\> 下载 [少数派 2.0 客户端](https://sspai.com/page/client)、关注 [少数派公众号](https://sspai.com/s/J71e)，解锁全新阅读体验 📰

\> 实用、好用的 [正版软件](https://sspai.com/mall)，少数派为你呈现 🚀