## 1\. 新的改变

我们对[[Markdown]]编辑器进行了一些功能拓展与语法支持，除了标准的Markdown编辑器功能，我们增加了如下几点新功能，帮助你用它写博客：

1.  **全新的界面设计** ，将会带来全新的写作体验；
2.  在创作中心设置你喜爱的代码高亮样式，Markdown **将代码片显示选择的高亮样式** 进行展示；
3.  增加了 **图片拖拽** 功能，你可以将本地的图片直接拖拽到编辑区域直接展示；
4.  全新的 **KaTeX数学公式** 语法；
5.  增加了支持**甘特图的mermaid语法[1](https://developer.aliyun.com/article/947125?spm=a2c6h.12873639.article-detail.89.51e55a6armJDJQ#fn-1)** 功能；
6.  增加了 **多屏幕编辑** Markdown文章功能；
7.  增加了 **焦点写作模式、预览模式、简洁写作模式、左右区域同步滚轮设置** 等功能，功能按钮位于编辑区域与预览区域中间；
8.  增加了 **检查列表** 功能。

## 2\. 功能快捷键

-   撤销：Ctrl/Command + Z
-   重做：Ctrl/Command + Y
-   加粗：Ctrl/Command + B
-   斜体：Ctrl/Command + I
-   标题：Ctrl/Command + Shift + H
-   无序列表：Ctrl/Command + Shift + U
-   有序列表：Ctrl/Command + Shift + O
-   检查列表：Ctrl/Command + Shift + C
-   插入代码：Ctrl/Command + Shift + K
-   插入链接：Ctrl/Command + Shift + L
-   插入图片：Ctrl/Command + Shift + G

## 3.合理的创建标题，有助于目录的生成

直接输入1次#，并按下space后，将生成1级标题。 输入2次#，并按下space后，将生成2级标题。 以此类推，我们支持6级标题。有助于使用`TOC`语法后生成一个完美的目录。

## 4\. 如何改变文本的样式

_强调文本_ _强调文本_

**加粗文本** **加粗文本**

\==标记文本==

~删除文本~

> 引用文本

H~2~O is是液体。

2^10^ 运算结果是 1024.

## 5\. 插入链接与图片

> ! \[\] () 在小括号里边填写图片的连接即可进行超级链接。

当然，我们为了让用户更加便捷，我们增加了图片拖拽功能。

## 6\. 如何插入一段漂亮的代码片

去[博客设置](https://link.juejin.cn/?target=https%3A%2F%2Fmp.csdn.net%2Fconfigure)页面，选择一款你喜欢的代码片高亮样式，下面展示同样高亮的 `代码片`.

```
// An highlighted block
var foo = 'bar';
复制代码
```

## 7\. 生成一个适合你的列表

-   项目

-   项目

-   项目

1.  项目1
2.  项目2
3.  项目3

-   计划任务
-   完成任务

## 8\. 创建一个表格

一个简单的表格是这么创建的：

## 9\. 设定内容居中、居左、居右

使用`:---------:`居中 使用`:----------`居左 使用`----------:`居右

## 10\. SmartyPants

SmartyPants将ASCII标点字符转换为“智能”印刷标点HTML实体。例如：

## 11\. 创建一个自定义列表

Markdown :  Text-to-HTML conversion tool

Authors :  程云博 :  Java学术趴

## 12\. 如何创建一个注脚

一个具有注脚的文本。\[^2\] 。具体使用方法参考文章下方的官网链接。

## 13.注释也是必不可少的

Markdown将文本转换为 HTML。

## 14.KaTeX数学公式

您可以使用渲染LaTeX数学表达式 [KaTeX](https://link.juejin.cn/?target=https%3A%2F%2Fkhan.github.io%2FKaTeX%2F):

![微信截图_20220607160053.png](https://ucc.alicdn.com/pic/developer-ecology/391c725ef90a4f8e90d7e9e532a080fa.png "微信截图_20220607160053.png")

> 你可以找到更多关于的信息 **LaTeX** 数学表达式，参考文章下方的官网链接

## 15.新的甘特图功能，丰富你的文章

![微信截图_20220607160251.png](https://ucc.alicdn.com/pic/developer-ecology/46ac222ed3a147a591f6947a5927da9e.png "微信截图_20220607160251.png")

-   关于 **甘特图** 语法，点击文章最下方的官网链接地址进行查看。

## 16.UML 图表

可以使用UML图表进行渲染。 [Mermaid](https://link.juejin.cn/?target=https%3A%2F%2Fmermaidjs.github.io%2F). 例如下面产生的一个序列图： ![微信截图_20220607160329.png](https://ucc.alicdn.com/pic/developer-ecology/c9ad633b99bb4e8886ebd98308dcaf0a.png "微信截图_20220607160329.png")

这将产生一个流程图:

![微信截图_20220607160346.png](https://ucc.alicdn.com/pic/developer-ecology/bbc8793f2a654b19999a0e1a76403f0d.png "微信截图_20220607160346.png")

-   关于 **Mermaid** 语法，参考文章下方的官网链接。

## 17\. FLowchart流程图

我们依旧会支持flowchart的流程图：

![微信截图_20220607160601.png](https://ucc.alicdn.com/pic/developer-ecology/511b3aa8b80043edbca25ce915710ad3.png "微信截图_20220607160601.png")

-   关于 **Flowchart流程图** 语法，参考文章下方的官网链接。

## 18\. 导出与导入

### 导出

如果你想尝试使用此编辑器, 你可以在此篇文章任意编辑。当你完成了一篇文章的写作, 在上方工具栏找到 **文章导出** ，生成一个.md文件或者.html文件进行本地保存。

### 导入

如果你想加载一篇你写过的.md文件或者.html文件，在上方工具栏可以选择导入功能进行对应扩展名的文件导入， 继续你的创作。

## 19.如果大家想进一步了解的话，下边提供了官方文档的阅读连接。

1.[mermaid语法说明](https://link.juejin.cn/?target=https%3A%2F%2Fmermaidjs.github.io%2F)[↩](https://developer.aliyun.com/article/947125?spm=a2c6h.12873639.article-detail.89.51e55a6armJDJQ#fnref-1)