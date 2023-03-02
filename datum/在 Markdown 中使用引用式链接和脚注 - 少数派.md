**Matrix 首页推荐** 

[Matrix](https://sspai.com/matrix) 是少数派的写作社区，我们主张分享真实的产品体验，有实用价值的经验与思考。我们会不定期挑选 Matrix 最优质的文章，展示来自用户的最真实的体验和观点。   
文章代表作者个人观点，少数派仅对标题和排版略作修改。

___

## 引用式链接

众所周知，[Markdown 中的链接](https://sspai.com/link?target=https%3A%2F%2Fdaringfireball.net%2Fprojects%2Fmarkdown%2Fsyntax%23link) 是这样写的：

```
This is [an example](http://example.com/ "Title") inline link.

[This link](http://example.net/) has no title attribute.
```

这种广泛使用的链接写作方式被称为行内链接（inline link）。实际上，Markdown 中的链接还有另一种不太常用的写法，即引用式链接（reference-style link），它的形式如下：

```
This is [an example] [id] reference-style link.

[id]: http://example.com/  "Optional Title Here"
```

可以发现，引用式链接在正文中使用了两个方括号，第一个方括号包裹超链接的文本部分，也就是这里的 `[an example]`，紧接着其后的是一个可有可无的空格，接下来使用第二个方括号包裹该链接的标识符，这里用 `[id]` 表示。

在下方的链接部分，首先出现链接的标识符 `[id]`，然后是一个半角冒号 `:`，接着是一个或多个空格（也可以是 Tab 键），然后是链接的 URL，最后是一个可选的链接标题，其格式与行内链接的标题格式一致，使用圆括号 `()`、双引号 `"` 或单引号 `'` 包裹。除此之外，引用式链接还可以省略链接标识符，即省略第二个方括号中的内容，这种链接叫作隐式链接（implicit link），其写作方式如下所示：

```
[Google][]

[Google]: https://www.google.com
```

与行内链接相比，引用式链接有什么好处？John Gruber 在介绍 Markdown 基本语法时就给出了解释：

> The point of reference-style links is not that they’re easier to write. The point is that with reference-style links, your document source is vastly more readable.
> 
> 引用式链接的意义并不在于它们更容易书写，关键在于使用引用式链接可以大大提高 Markdown 源文件的可读性。

的确，在 Markdown 源文件中有大量链接，并且这些链接非常冗长且不够美观的情况下，行内链接不仅会明显降低源文件的可读性，甚至还会干扰我们的写作思路，而引用式链接则可以在很大程度避免这个问题。除此之外，写作过程中，我们往往不想打断写作思路去网上搜索链接，这种情况下就可以先放置一个引用式链接占位符，方便后续进行补充。如果忘记填写链接也没有关系，[Markdown 语法检查器](https://sspai.com/prime/story/markdown-linter-a-primer) 会以错误的形式标注出来，提醒我们修改。

![一份 Markdown 文件，分别使用行内链接和引用式链接的效果对比，很明显右侧引用式链接的可读性更强](https://cdn.sspai.com/2022/12/28/article/fcd932fe80736e573a5babb27fba8dbd?imageView2/2/w/1120/q/90/interlace/1/ignore-error/1)

一份 Markdown 文件，分别使用行内链接和引用式链接的效果对比，很明显右侧引用式链接的可读性更强

**关联阅读：**[给你的 Markdown 挑挑刺——语法检查器入门与进阶](https://sspai.com/prime/story/markdown-linter-a-primer)

### 使用 Keyboard Maestro 插入引用式链接

几天前，少数派编辑 [@PlatyHsu](https://sspai.com/u/platyhsu/updates) 在会员群中「墙裂推荐」使用引用式链接（他称之为「分体式链接」）的时候，我评论说「手动加一个链接标识符有点麻烦」。不过后来我意识到，与引用式链接带来的便利相比，这点麻烦还是可以接受的，而我要做的就是让书写引用式链接的过程变得更加简单。为此，我制作了一个 [Keyboard Maestro](https://sspai.com/link?target=https%3A%2F%2Fwww.keyboardmaestro.com%2F) 动作（⬇️ [点击下载](https://sspai.com/link?target=https%3A%2F%2Fp15.p3.n0.cdn.getcloudapp.com%2Fitems%2FOAu2Q1mJ%2Ffc7518c1-b6d8-46cc-973c-776fbbf69cde.kmmacros)），帮助我在 Markdown 中写作时快速插入引用式链接。

![快速插入引用式链接的 Keyboard Maestro 动作](https://cdn.sspai.com/2022/12/28/article/c8d60b2cccc711c55eaa8b21d7bc1e26?imageView2/2/w/1120/q/90/interlace/1/ignore-error/1)

快速插入引用式链接的 Keyboard Maestro 动作

这个 Keyboard Maestro 动作的主要原理是生成一个随机链接标识符（这里设置的是 3 位数，UUID 由大写英文字母和阿拉伯数字组成），以避免手动编辑过程中可能出现重复的情况。然后按照引用式链接的格式，将标识符和 URL 插入到对应的位置。

使用方法很简单，首先将链接的 URL 复制到剪切板中，然后在 Markdown 文件中移动光标选中要插入链接的文本，接着按下快捷键 `⌥ + ⇧ + K`（你也可以修改为其他快捷键），就会自动将剪切板中的链接作为一个引用式链接插入。需要注意的是，如果你使用 [一句话换一行](https://sspai.com/post/73957) 的写作方式，则可能需要将插入的链接文本上下移动几行；如果你在 [VS Code](https://sspai.com/link?target=https%3A%2F%2Fcode.visualstudio.com%2F) 中使用 [Vim](https://sspai.com/link?target=https%3A%2F%2Fwww.vim.org%2F)，请不要在 Normal Mode 下执行此动作，而需要首先按下 `i` 或 `a` 进入 Insert Mode，然后再执行这个 Keyboard Maestro 动作。

![使用 Keyboard Maestro 动作快速插入引用式链接](https://cdn.sspai.com/2022/12/28/article/12229804bed1dccdccf78a6b63cef334?imageView2/2/w/1120/q/90/interlace/1/ignore-error/1)

使用 Keyboard Maestro 动作快速插入引用式链接

### 在 Pandoc’s Markdown 中使用引用式链接

如果你需要将已有 Markdown 中的行内链接转换为引用式链接，可以使用 [Pandoc](https://sspai.com/link?target=https%3A%2F%2Fpandoc.org%2F) 的 `--reference-links` [选项](https://sspai.com/link?target=https%3A%2F%2Fpandoc.org%2FMANUAL.html%23option--reference-links)，例如：

```
pandoc --reference-links --reference-location=block input.md -o output.md
```

这行命令中，`--reference-links` 指定使用引用式链接，而不是默认的行内链接，`--reference-location=block` 表明引用链接的位置是 `block`，也就是放置在每个段落的下方，另外两个 [可选项](https://sspai.com/link?target=https%3A%2F%2Fpandoc.org%2FMANUAL.html%23option--reference-location) 是 `section`（放置在当前章节的末尾） 和 `document`（放置在文档末尾，默认选项）。输入文件为 `input.md`，输出文件为 `output.md`。

值得一提的是，Pandoc 还提供了一个 [扩展](https://sspai.com/link?target=https%3A%2F%2Fpandoc.org%2FMANUAL.html%23extension-shortcut_reference_links) `shortcut_reference_links`，支持省略引用式链接的第二个方括号，也就是你可以在 Markdown 中这样简写引用式链接（转换为 `markdown` 时，这是一个默认开启的选项）：

```
See [my website].

[my website]: http://foo.bar.baz
```

另外，如果你习惯在引用式链接的两个方括号之间加上一个空格，即 `[foo] [bar]`，可以在使用 Pandoc 转换时开启 `spaced_reference_links` [这个扩展](https://sspai.com/link?target=https%3A%2F%2Fpandoc.org%2FMANUAL.html%23extension-spaced_reference_links)。

**关联阅读：**[Pandoc 从入门到精通，你也可以学会这一个文本转换利器](https://sspai.com/post/77206)

## 引用式脚注

尽管标准 Markdown 不支持脚注，但很多 Markdown 变种都支持。一般来说，大多数 Markdown 方言和编辑器支持行内脚注和引用式脚注这两种方式，而后者使用更为普遍：

```
Here is a footnote reference,[^1] and another.[^longnote]

[^1]: Here is the footnote.

[^longnote]: Here's one with multiple blocks.
```

使用相对较少的行内脚注的书写方式：

```
Here is an inline note.^[Inline notes are easier to write, since
you don't have to pick an identifier and move down to type the
note.]
```

容易发现，脚注与链接的写法非常相似。行内脚注的书写方式很简洁，引用式脚注的可读性更好，但需要在写作时手动添加一个标识符。为了避免手动编辑导致标识符可能出现的冲突，我们再次借助 Keyboard Maestro 动作来随机生成标识符（⬇️ [点击下载](https://sspai.com/link?target=https%3A%2F%2Fp15.p3.n0.cdn.getcloudapp.com%2Fitems%2FJruyDYRv%2F585c8c91-e0b3-414d-bffc-b3c5b3042d07.kmmacros)）。

![用于插入引用式脚注的 Keyboard Maestro 动作](https://cdn.sspai.com/2022/12/28/article/8b8121fba4929d4ea289f056dcd06ba1?imageView2/2/w/1120/q/90/interlace/1/ignore-error/1)

用于插入引用式脚注的 Keyboard Maestro 动作

这个 Keyboard Maestro 动作的思路与插入引用式链接的动作类似，此处不再赘述。它的使用方式同样非常简单，你只需将光标定位到需要插入脚注的位置，然后按下快捷键 `⌥ + ⇧ + J`，它就会自动插入脚注标识符，并将光标定位到填写脚注内容的位置。

![使用 Keyboard Maestro 动作插入引用式脚注](https://cdn.sspai.com/2022/12/28/article/608d68409a373d3144195d7a54dc97c5?imageView2/2/w/1120/q/90/interlace/1/ignore-error/1)

使用 Keyboard Maestro 动作插入引用式脚注

## 小结

@PlatyHsu 说：「会写 Markdown 的人很多，但写得好 Markdown 的人却很少」，的确，尽管 Markdown 简单易上手，但写好它还是需要不断学习的。而在 Markdown 中使用引用式链接和脚注，可以算是一种值得学习的写作习惯，因为这可以让你的 Markdown 文档的易读性大大提高，更有利于后续的编辑和修订。本文介绍了这两种「引用式写作方式」，以及如何借助 Keyboard Maestro 或 Pandoc 来快速插入或转换引用式链接和脚注，以提高写作效率。如果你有更好的方法或者想要补充的内容，欢迎在评论区分享。

\> 下载 [少数派 2.0 客户端](https://sspai.com/page/client)、关注 [少数派公众号](https://sspai.com/s/J71e)，解锁全新阅读体验 📰

\> 实用、好用的 [正版软件](https://sspai.com/mall)，少数派为你呈现 🚀