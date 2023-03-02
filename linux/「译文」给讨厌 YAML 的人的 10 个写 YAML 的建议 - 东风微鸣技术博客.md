配置文件有很多格式: 键、值、和键值对的列表、INI 文件、YAML、JSON、XML 等等。其中，由于一些不同的原因，YAML 有时被认为是特别难以处理的。虽然它反映层次值的能力是重要的，而且它的极简主义可能会让一些人耳目一新，但它对类似 python 的缩进语法的依赖可能会令人沮丧。

然而，开源世界是多样化和灵活的，没有人需要忍受复杂的技术，所以如果您讨厌 YAML，这里有 10 件您可以 (而且应该) 做的事情来让它变得可以忍受。从零开始，任何合理的索引都应该如此。

## 0\. 让你的编辑器来做这件事[](https://ewhisper.cn/posts/7496/#0-%20%E8%AE%A9%E4%BD%A0%E7%9A%84%E7%BC%96%E8%BE%91%E5%99%A8%E6%9D%A5%E5%81%9A%E8%BF%99%E4%BB%B6%E4%BA%8B)

无论您使用什么文本编辑器，都可能有插件使处理语法更容易。如果您的编辑器没有使用 YAML 插件，请找到一个并安装它。您在寻找插件并根据需要配置插件上所付出的努力，将在您下一次编辑 YAML 时得到十倍的回报。

例如， [Atom](http://atom.io/) 编辑器默认带有 YAML 模式，而 GNU Emacs 提供了很少的支持，您可以添加其他包，比如 [yaml-mode](https://github.com/yoshiki/yaml-mode) 来提供帮助。

[![YAML 和空格模式下的 Emacs](https://pic-cdn.ewhisper.cn/img/2021/09/18/23f302d1996fa6cdffab96a7c305fb1d-emacs.jpg)](https://pic-cdn.ewhisper.cn/img/2021/09/18/23f302d1996fa6cdffab96a7c305fb1d-emacs.jpg "YAML 和空格模式下的 Emacs")

如果您最喜欢的文本编辑器没有 YAML 模式，您可以通过小小的配置更改来解决一些不满。例如，GNOME 桌面的默认文本编辑器 Gedit 没有 YAML 模式可用，但它默认提供 YAML 语法高亮显示，并具有可配置的选项卡宽度:

[![在 Gedit 中配置 tab 宽度和输入](https://pic-cdn.ewhisper.cn/img/2021/09/18/34e7c395d6abfa04387b6a6462e0f1a8-gedit.jpg)](https://pic-cdn.ewhisper.cn/img/2021/09/18/34e7c395d6abfa04387b6a6462e0f1a8-gedit.jpg "在 Gedit 中配置 tab 宽度和输入")

使用[`drawspaces`](https://wiki.gnome.org/Apps/Gedit/ShippedPlugins) Gedit 插件包，你可以让空格以前导点的形式可见，从而消除关于缩进级别的任何问题。

花点时间研究一下你最喜欢的文本编辑器。了解编辑器或其社区如何使 YAML 变得更简单，并在您的工作中利用这些特性。你不会后悔的。

## 1\. 使用 linter[](https://ewhisper.cn/posts/7496/#1-%20%E4%BD%BF%E7%94%A8%20-linter)

理想情况下，编程语言和标记语言使用可预测的语法。计算机往往在可预见性方面做得很好，所以在 1978 年发明了 linter 的概念。如果你没有使用 YAML 的 linter，那么是时候采用这个有 40 年历史的传统，使用 `yamllint` 了。

您可以在 Linux 使用发行版的包管理器上安装 [`yamllint`](https://pypi.org/project/yamllint/) 。例如，在 [Red Hat Enterprise Linux](https://www.redhat.com/en/technologies/linux-platforms/enterprise-linux) 8 或 [Fedora](https://getfedora.org/) 上:

<table><tbody><tr><td><pre><span>1</span><br></pre></td><td><pre><code>sudo dnf install yamllint<br></code><p><i></i>SHELL</p></pre></td></tr></tbody></table>

调用 `yamllint` 就像告诉它检查文件一样简单。下面是 `yamllint` 对包含错误的 YAML 文件的响应示例:

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br></pre></td><td><pre><code><span>$</span><span> yamllint errorprone.yaml</span><br>errorprone.yaml<br>23:10     error    syntax error: mapping values are not allowed here<br>23:11     error    trailing spaces  (trailing-spaces)<br></code><p><i></i>SHELL</p></pre></td></tr></tbody></table>

左边的不是时间戳。它是错误的行号和列号。你可能不知道它说的是什么错误，但现在你知道错误的位置了。再看一遍位置，错误的本质就显而易见了。成功是无声的，所以如果您想要基于 lint 的成功的反馈，您可以添加一个带双 & 号 (`&&`) 的有条件的第二个命令。在 POSIX shell 中，如果命令返回 0 以外的任何内容，`&&` 就会失败，因此在成功时，`echo` 命令会清楚地表明这一点。这种策略有些肤浅，但有些用户更喜欢确保命令正确运行，而不是默默地失败。这里有一个例子:

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br></pre></td><td><pre><code><span>$</span><span> yamllint perfect.yaml &amp;&amp; <span>echo</span> <span>"OK"</span></span><br>OK<br></code><p><i></i>SHELL</p></pre></td></tr></tbody></table>

`yamllint` 在成功时之所以如此安静，是因为它在没有错误时返回 0 。

## 2\. 用 Python 编写，而不是 YAML[](https://ewhisper.cn/posts/7496/#2-%20%E7%94%A8%20-Python-%20%E7%BC%96%E5%86%99%EF%BC%8C%E8%80%8C%E4%B8%8D%E6%98%AF%20-YAML)

如果您真的讨厌 YAML，那么停止使用 YAML，至少从字面上来说。您可能会继续使用 YAML，因为这是应用程序接受的唯一格式，但如果唯一的要求是最终使用 YAML，那么就使用其他格式，然后进行转换。Python 以及出色的 [pyyaml](https://pyyaml.org/) 库使这变得很容易，您有两种方法可以选择: 自转换或脚本化。

### 自转换（Self-conversion）[](https://ewhisper.cn/posts/7496/#%E8%87%AA%E8%BD%AC%E6%8D%A2%EF%BC%88Self-conversion%EF%BC%89)

在自转换方法中，数据文件也是生成 YAML 的 Python 脚本。这对于小数据集最有效。只需将您的 JSON 数据写入 Python 变量中，在导入语句前面添加一个 `import` 语句，并以一个简单的三行输出语句结束文件。

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br><span>8</span><br><span>9</span><br><span>10</span><br><span>11</span><br><span>12</span><br><span>13</span><br><span>14</span><br><span>15</span><br><span>16</span><br><span>17</span><br><span>18</span><br><span>19</span><br><span>20</span><br><span>21</span><br><span>22</span><br><span>23</span><br><span>24</span><br><span>25</span><br><span>26</span><br><span>27</span><br><span>28</span><br><span>29</span><br></pre></td><td><pre><code><span>#!/usr/bin/python3</span><br><span>import</span> yaml <p>d={<br><span>"glossary"</span>: {<br>  <span>"title"</span>: <span>"example glossary"</span>,<br>  <span>"GlossDiv"</span>: {<br><span>"title"</span>: <span>"S"</span>,<br><span>"GlossList"</span>: {<br>  <span>"GlossEntry"</span>: {<br><span>"ID"</span>: <span>"SGML"</span>,<br><span>"SortAs"</span>: <span>"SGML"</span>,<br><span>"GlossTerm"</span>: <span>"Standard Generalized Markup Language"</span>,<br><span>"Acronym"</span>: <span>"SGML"</span>,<br><span>"Abbrev"</span>: <span>"ISO 8879:1986"</span>,<br><span>"GlossDef"</span>: {<br>  <span>"para"</span>: <span>"A meta-markup language, used to create markup languages such as DocBook."</span>,<br>  <span>"GlossSeeAlso"</span>: [<span>"GML"</span>, <span>"XML"</span>]<br>  },<br><span>"GlossSee"</span>: <span>"markup"</span><br>}<br>  }<br>}<br>  }<br>}</p><p>f=<span>open</span>(<span>'output.yaml'</span>,<span>'w'</span>)<br>f.write(yaml.dump(d))<br>f.close</p></code><p><i></i>PYTHON</p></pre></td></tr></tbody></table>

使用 Python 运行该文件，生成一个名为 `output.yaml` 的文件。

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br><span>8</span><br><span>9</span><br><span>10</span><br><span>11</span><br><span>12</span><br><span>13</span><br><span>14</span><br><span>15</span><br><span>16</span><br><span>17</span><br></pre></td><td><pre><code><span>$</span><span> python3 ./example.json</span><br><span>$</span><span> cat output.yaml</span><br>glossary:<br>  GlossDiv:<br>GlossList:<br>  GlossEntry:<br>Abbrev: ISO 8879:1986<br>Acronym: SGML<br>GlossDef:<br>  GlossSeeAlso: [GML, XML]<br>  para: A meta-markup language, used to create markup languages such as DocBook.<br>GlossSee: markup<br>GlossTerm: Standard Generalized Markup Language<br>ID: SGML<br>SortAs: SGML<br>title: S<br>  title: example glossary<br></code><p><i></i>SHELL</p></pre></td></tr></tbody></table>

这个输出是完全有效的 YAML，尽管`yamllint` 会发出一个警告，说明文件没有以 `---` 开头，这是您可以在 Python 脚本中或手动调整的内容。

### 脚本转换[](https://ewhisper.cn/posts/7496/#%E8%84%9A%E6%9C%AC%E8%BD%AC%E6%8D%A2)

在这个方法中，使用 JSON 编写代码，然后运行 Python 转换脚本生成 YAML。这比自转换的伸缩性更好，因为它使转换器与数据分离。

Create a JSON file and save it as `example.json`. Here is an example from : 创建一个 JSON 文件并将其保存为 `example.json`。以下是来自 [json.org](https://json.org/example.html) 的一个例子:

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br><span>8</span><br><span>9</span><br><span>10</span><br><span>11</span><br><span>12</span><br><span>13</span><br><span>14</span><br><span>15</span><br><span>16</span><br><span>17</span><br><span>18</span><br><span>19</span><br><span>20</span><br><span>21</span><br><span>22</span><br></pre></td><td><pre><code>{<br>"glossary": {<br>  "title": "example glossary",<br>  "GlossDiv": {<br>"title": "S",<br>"GlossList": {<br>  "GlossEntry": {<br>"ID": "SGML",<br>"SortAs": "SGML",<br>"GlossTerm": "Standard Generalized Markup Language",<br>"Acronym": "SGML",<br>"Abbrev": "ISO 8879:1986",<br>"GlossDef": {<br>  "para": "A meta-markup language, used to create markup languages such as DocBook.",<br>  "GlossSeeAlso": ["GML", "XML"]<br>  },<br>"GlossSee": "markup"<br>}<br>  }<br>}<br>  }<br>}<br></code><p><i></i>SHELL</p></pre></td></tr></tbody></table>

创建一个简单的转换器并将其保存为 `json2yaml.py`。该脚本导入 Python YAML 和 JSON 模块，加载用户定义的 JSON 文件，执行转换，然后将数据写入 `output.yaml`。

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br><span>8</span><br><span>9</span><br><span>10</span><br><span>11</span><br><span>12</span><br></pre></td><td><pre><code><span>#!/usr/bin/python3</span><br><span>import</span> yaml<br><span>import</span> sys<br><span>import</span> json<p>OUT=<span>open</span>(<span>'output.yaml'</span>,<span>'w'</span>)<br>IN=<span>open</span>(sys.argv[<span>1</span>], <span>'r'</span>)</p><p>JSON = json.load(IN)<br>IN.close()<br>yaml.dump(JSON, OUT)<br>OUT.close()</p></code><p><i></i>PYTHON</p></pre></td></tr></tbody></table>

将此脚本保存在系统路径下，并根据需要执行:

<table><tbody><tr><td><pre><span>1</span><br></pre></td><td><pre><code>~/bin/json2yaml.py example.json<br></code><p><i></i>SHELL</p></pre></td></tr></tbody></table>

## 3\. 尽早分析，经常分析[](https://ewhisper.cn/posts/7496/#3-%20%E5%B0%BD%E6%97%A9%E5%88%86%E6%9E%90%EF%BC%8C%E7%BB%8F%E5%B8%B8%E5%88%86%E6%9E%90)

有时从不同的角度看问题是有帮助的。如果您的问题是 YAML，并且您很难可视化数据的关系，那么您可能会发现，临时地将该数据重构为您更熟悉的内容是有用的。

例如，如果您更喜欢字典样式的列表或 JSON，可以使用交互式 Python shell 用两个命令将 YAML 转换为 JSON。假设您的 YAML 文件名为 `mydata.yaml`。

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br></pre></td><td><pre><code><span>$</span><span> python3</span><br><span>&gt;</span><span>&gt;&gt; f=open(<span>'mydata.yaml'</span>,<span>'r'</span>)</span><br><span>&gt;</span><span>&gt;&gt; yaml.load(f)</span><br>{'document': 34843, 'date': datetime.date(2019, 5, 23), 'bill-to': {'given': 'Seth', 'family': 'Kenlon', 'address': {'street': '51b Mornington Road\n', 'city': 'Brooklyn', 'state': 'Wellington', 'postal': 6021, 'country': 'NZ'}}, 'words': 938, 'comments': 'Good article. Could be better.'}<br></code><p><i></i>SHELL</p></pre></td></tr></tbody></table>

还有许多其他的例子，而且有许多在线转换器和本地解析器，所以当数据开始看起来更像一个清单而不是标记时，请不要犹豫重新格式化数据。

## 4\. 阅读规范[](https://ewhisper.cn/posts/7496/#4-%20%E9%98%85%E8%AF%BB%E8%A7%84%E8%8C%83)

我已经远离 YAML 一段时间后, 发现自己再次使用它, 我直接回到 [yaml.org](https://yaml.org/spec/1.2/spec.html) 重读规范。如果你从未读过 YAML 的规范, 你发现 YAML 混乱, 看一眼规范可以提供澄清你永远不知道。该规范非常容易阅读，在 [chapter 6](https://yaml.org/spec/1.2/spec.html#Basic) 中用大量示例详细说明了有效 YAML 的需求。

## 5\. 伪配置（Pseudo-config）[](https://ewhisper.cn/posts/7496/#5-%20%E4%BC%AA%E9%85%8D%E7%BD%AE%EF%BC%88Pseudo-config%EF%BC%89)

在我开始写自己的书 [《在树莓派上开发游戏》(develop Games on the Raspberry Pi, Apress, 2019)](https://www.apress.com/gp/book/9781484241691) 之前，出版商要求我提供一个大纲。你可能认为写提纲很容易。根据定义，它只是章节和章节的标题，没有真正的内容。然而，在发表的 300 页中，最难写的部分是最初的大纲。

YAML 也可以采用同样的方式。您可能对需要记录的数据有一个概念，但这并不意味着您完全理解它们之间的关系。因此，在您开始编写 YAML 之前，请尝试执行一个伪配置。

伪配置就像伪代码。您不必担心结构或缩进、父子关系、继承或嵌套。你只需要在头脑中以你目前理解的方式创建数据迭代。

[![伪配置](https://pic-cdn.ewhisper.cn/img/2021/09/18/ac3aba7c18ecd5a7233163d4786913c2-pseudoyaml.jpg)](https://pic-cdn.ewhisper.cn/img/2021/09/18/ac3aba7c18ecd5a7233163d4786913c2-pseudoyaml.jpg "伪配置")

一旦您在纸上得到了伪配置，就研究它，并将结果转换为有效的 YAML。

## 6\. 解决空格和制表符的争论[](https://ewhisper.cn/posts/7496/#6-%20%E8%A7%A3%E5%86%B3%E7%A9%BA%E6%A0%BC%E5%92%8C%E5%88%B6%E8%A1%A8%E7%AC%A6%E7%9A%84%E4%BA%89%E8%AE%BA)

好吧，也许您不能确定地解决 [空格 vs. 制表符的争论](https://opensource.com/article/18/9/spaces-poll)，但您至少应该在您的项目或组织中解决这个争论。无论您是使用后处理 `sed` 脚本、文本编辑器配置来解决这个问题，还是发誓尊重您的 linter 结果，您的团队中任何接触 YAML 项目的人都必须同意使用空格(符合 YAML 规范)。

任何好的文本编辑器都允许定义多个空格而不是制表符，所以这个选择不会对 tab 键的粉丝产生负面影响。

您可能非常清楚，制表符和空格本质上是不可见的。当一些东西从你的视线中消失时，它很少会出现在你的脑海中，直到你测试并消除了所有「明显」的问题。浪费一个小时在错误的选项卡或空格组上是您创建策略来使用其中一个或另一个的信号，然后开发一个故障安全检查以确保遵从性(例如使用 Git hook 来执行检测)。

## 7\. 少即是多(或多即是少)[](https://ewhisper.cn/posts/7496/#7-%20%E5%B0%91%E5%8D%B3%E6%98%AF%E5%A4%9A%20-%20%E6%88%96%E5%A4%9A%E5%8D%B3%E6%98%AF%E5%B0%91)

有些人喜欢编写 YAML 来强调它的结构。他们积极地缩进以帮助自己可视化数据块。模仿具有显式分隔符的标记语言是一种欺骗。

以下是来自 [Ansible 文档](https://docs.ansible.com/index.html?extIdCarryOver=true&sc_cid=701f2000001Css5AAC) 的一个很好的例子:

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br><span>8</span><br><span>9</span><br><span>10</span><br><span>11</span><br><span>12</span><br><span>13</span><br><span>14</span><br><span>15</span><br></pre></td><td><pre><code><span># Employee records</span><br><span>-</span>  <span>martin:</span><br>        <span>name:</span> <span>Martin</span> <span>D'vloper</span><br>        <span>job:</span> <span>Developer</span><br>        <span>skills:</span><br>            <span>-</span> <span>Python</span><br>            <span>-</span> <span>perl</span><br>            <span>-</span> <span>pascal</span><br><span>-</span>  <span>tabitha:</span><br>        <span>name:</span> <span>Tabitha</span> <span>Bitumen</span><br>        <span>job:</span> <span>Developer</span><br>        <span>skills:</span><br>            <span>-</span> <span>lisp</span><br>            <span>-</span> <span>fortran</span><br>            <span>-</span> <span>erlang</span><br></code><p><i></i>YAML</p></pre></td></tr></tbody></table>

对于一些用户来说，这种方法是布局 YAML 文档的一种很有帮助的方法，而另一些用户则因为看似无缘无故的空白而忽略了这种结构。

如果您拥有并维护一个 YAML 文档，那么您将定义「缩进」的含义。如果水平空白块分散了你的注意力，那么使用 YAML 规范所要求的最小数量的空白。例如，Ansible 文档中的同一个 YAML 可以用更少的缩进表示，而不会失去任何有效性或意义:

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br><span>8</span><br><span>9</span><br><span>10</span><br><span>11</span><br><span>12</span><br><span>13</span><br><span>14</span><br><span>15</span><br></pre></td><td><pre><code><span>---</span><br><span>-</span> <span>martin:</span><br>   <span>name:</span> <span>Martin</span> <span>D'vloper</span><br>   <span>job:</span> <span>Developer</span><br>   <span>skills:</span><br>   <span>-</span> <span>Python</span><br>   <span>-</span> <span>perl</span><br>   <span>-</span> <span>pascal</span><br><span>-</span> <span>tabitha:</span><br>   <span>name:</span> <span>Tabitha</span> <span>Bitumen</span><br>   <span>job:</span> <span>Developer</span><br>   <span>skills:</span><br>   <span>-</span> <span>lisp</span><br>   <span>-</span> <span>fortran</span><br>   <span>-</span> <span>erlang</span><br></code><p><i></i>YAML</p></pre></td></tr></tbody></table>

## 8\. 做一个配方[](https://ewhisper.cn/posts/7496/#8-%20%E5%81%9A%E4%B8%80%E4%B8%AA%E9%85%8D%E6%96%B9)

我非常喜欢重复产生熟悉，但有时重复只会产生重复的愚蠢错误。幸运的是，在公元 396 年，一位聪明的农妇经历了这种现象，并发明了这个 _配方_ 的概念。

如果您发现自己一次又一次地犯 YAML 文档错误，您可以将配方或模板作为注释部分嵌入到 YAML 文件中。当您添加一个节时，复制注释的配方并使用新的真实数据覆盖虚拟数据。例如:

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br><span>8</span><br><span>9</span><br><span>10</span><br><span>11</span><br><span>12</span><br><span>13</span><br><span>14</span><br><span>15</span><br><span>16</span><br><span>17</span><br><span>18</span><br><span>19</span><br><span>20</span><br></pre></td><td><pre><code><span>---</span><br><span># - &lt;common name&gt;:</span><br><span>#   name: Given Surname</span><br><span>#   job: JOB</span><br><span>#   skills:</span><br><span>#   - LANG</span><br><span>-</span> <span>martin:</span><br>  <span>name:</span> <span>Martin</span> <span>D'vloper</span><br>  <span>job:</span> <span>Developer</span><br>  <span>skills:</span><br>  <span>-</span> <span>Python</span><br>  <span>-</span> <span>perl</span><br>  <span>-</span> <span>pascal</span><br><span>-</span> <span>tabitha:</span><br>  <span>name:</span> <span>Tabitha</span> <span>Bitumen</span><br>  <span>job:</span> <span>Developer</span><br>  <span>skills:</span><br>  <span>-</span> <span>lisp</span><br>  <span>-</span> <span>fortran</span><br>  <span>-</span> <span>erlang</span><br></code><p><i></i>YAML</p></pre></td></tr></tbody></table>

## 9\. 换用其他配置格式[](https://ewhisper.cn/posts/7496/#9-%20%E6%8D%A2%E7%94%A8%E5%85%B6%E4%BB%96%E9%85%8D%E7%BD%AE%E6%A0%BC%E5%BC%8F)

总的来说，我是 YAML 的粉丝，但有时 YAML 并不能解决问题。如果您没有被正在使用的应用程序锁定在 YAML 中，那么使用其他配置格式可能会更好。有时配置文件会自动增长，最好将其重构为简单的 Lua 或 Python 脚本。

YAML 是一个很棒的工具，因其极简和简单而在用户中很受欢迎，但它不是您的工具包中唯一的工具。有时候，分道扬镳是最好的选择。YAML 的好处之一是解析库很常见，所以只要您提供迁移选项，您的用户就应该能够轻松地适应。

但是，如果 YAML 是必需的，那么请记住这些技巧，一劳永逸地克服对 YAML 的仇恨!

## 原文链接[](https://ewhisper.cn/posts/7496/#%E5%8E%9F%E6%96%87%E9%93%BE%E6%8E%A5)