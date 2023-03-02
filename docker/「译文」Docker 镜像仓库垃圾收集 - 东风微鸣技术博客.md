> 👉️**URL:** [https://docs.docker.com/registry/garbage-collection/](https://docs.docker.com/registry/garbage-collection/)
> 
> 📝**Description:**
> 
> High level discussion of 垃圾收集

从 v2.4.0 开始，垃圾收集器命令包含在注册表二进制文件中。本文档描述了这个命令的作用以及如何和为什么应该使用它。

## Debian 上运行垃圾收集[](https://ewhisper.cn/posts/9434/#Debian-%20%E4%B8%8A%E8%BF%90%E8%A1%8C%E5%9E%83%E5%9C%BE%E6%94%B6%E9%9B%86)

<table><tbody><tr><td><pre><span>1</span><br></pre></td><td><pre><code>/usr/bin/docker-registry garbage-collect --dry-run /etc/docker/registry/config.yml<br></code><p><i></i>BASH</p></pre></td></tr></tbody></table>

输出示例如下：

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br><span>8</span><br><span>9</span><br><span>10</span><br><span>11</span><br><span>12</span><br><span>13</span><br></pre></td><td><pre><code><span>emqx</span>/emqx-edge<br><span>emqx</span>/emqx-edge: marking manifest sha<span>256</span>:daf<span>0</span>f<span>342</span>c<span>71</span>cdf<span>6238</span>cf<span>3</span>c<span>56</span>a<span>7</span>cfe<span>6</span>ca<span>7333</span>a<span>62</span>b<span>400328</span>c<span>0</span>c<span>81</span>b<span>469</span>ccd<span>629</span>e<span>66</span><br><span>emqx</span>/emqx-edge: marking blob sha<span>256</span>:<span>30</span>c<span>03992</span>a<span>89</span>eb<span>819</span>aba<span>2931</span>bcbb<span>88163</span>fb<span>4</span>e<span>9</span>ed<span>31</span>c<span>839</span>de<span>8060</span>ec<span>56</span>a<span>66884113</span><br><span>emqx</span>/emqx-edge: marking blob sha<span>256</span>:be<span>307</span>f<span>383</span>ecc<span>62</span>b<span>27</span>a<span>29</span>b<span>599</span>c<span>3</span>fc<span>9</span>d<span>3129693</span>a<span>798</span>e<span>7</span>fcce<span>614</span>f<span>09174</span>cfe<span>2</span>d<span>354</span><br><span>emqx</span>/emqx-edge: marking blob sha<span>256</span>:<span>9</span>fb<span>745</span>ef<span>40</span>e<span>3</span>f<span>0</span>afd<span>369751</span>ee<span>44471</span>f<span>6</span>c<span>219438391</span>c<span>8852</span>b<span>30</span>e<span>450</span>e<span>15736</span>e<span>71</span>e<br><span>emqx</span>/emqx-edge: marking blob sha<span>256</span>:<span>95</span>ccf<span>8</span>f<span>331</span>e<span>107472</span>d<span>0009407862</span>e<span>4084</span>b<span>5897</span>dd<span>12</span>ec<span>105</span>b<span>1</span>a<span>823</span>e<span>37185</span>ff<span>072</span><br><span>emqx</span>/emqx-edge: marking blob sha<span>256</span>:cd<span>0335</span>bd<span>06052</span>d<span>8</span>ed<span>0</span>cf<span>75</span>dced<span>1</span>b<span>4</span>b<span>73</span>d<span>64</span>a<span>6</span>a<span>66</span>fdea<span>02860</span>ff<span>21</span>b<span>5</span>bed<span>675893</span><br><span>emqx</span>/emqx-edge: marking blob sha<span>256</span>:b<span>9</span>e<span>8</span>afc<span>4</span>fb<span>5</span>ee<span>2</span>fdd<span>2</span>c<span>476</span>dc<span>04583</span>b<span>3</span>b<span>9881883</span>f<span>171</span>b<span>120</span>c<span>0</span>ab<span>60430</span>d<span>81</span>ef<span>63</span>e<br><span>emqx</span>/emqx-edge: marking blob sha<span>256</span>:d<span>6</span>f<span>1281450</span>cb<span>81992</span>e<span>0</span>b<span>7003</span>b<span>4</span>f<span>1588</span>b<span>2</span>b<span>1</span>d<span>355075</span>d<span>1092396349</span>ba<span>688</span>d<span>662</span>ef<br><span>emqx</span>/emqx-edge: marking blob sha<span>256</span>:<span>5</span>b<span>1</span>be<span>9</span>d<span>5</span>d<span>246</span>a<span>49</span>dd<span>67255</span>d<span>32</span>cb<span>0</span>ac<span>67</span>b<span>7315</span>b<span>9</span>e<span>7</span>ecaf<span>8</span>f<span>87</span>fd<span>44</span>c<span>7</span>b<span>1</span>fe<span>7</span>a<span>368</span><p><span>9</span> blobs marked, <span>0</span> blobs eligible for deletion</p></code><p><i></i>APACHE</p></pre></td></tr></tbody></table>

然后执行：

<table><tbody><tr><td><pre><span>1</span><br></pre></td><td><pre><code>/usr/bin/docker-registry garbage-collect /etc/docker/registry/config.yml<br></code><p><i></i>BASH</p></pre></td></tr></tbody></table>

本次没有 blob 可被删除

## 关于 垃圾收集[](https://ewhisper.cn/posts/9434/#%E5%85%B3%E4%BA%8E%20-%20%E5%9E%83%E5%9C%BE%E6%94%B6%E9%9B%86)

在 Docker 注册表的上下文中，垃圾收集是从文件系统中删除不再被清单引用的 blob 的过程。blob 可以同时包含层和清单。

注册表数据可能会占用相当大的磁盘空间。此外，当需要确保文件系统中不再存在某些层时，垃圾收集可以作为安全考虑因素。

## 垃圾收集实践[](https://ewhisper.cn/posts/9434/#%E5%9E%83%E5%9C%BE%E6%94%B6%E9%9B%86%E5%AE%9E%E8%B7%B5)

文件系统层按其在注册表中的内容地址存储。这有许多优点，其中之一是数据只存储一次，并由清单引用。查看 [这里](https://docs.docker.com/registry/compatibility/#content-addressable-storage-cas) 了解更多细节。

层因此在清单之间共享；每个清单维护对该层的一个引用。只要一个层被一个清单引用，它就不能被垃圾收集。

可以使用注册表 API 删除清单和层（请参阅 [这里](https://docs.docker.com/registry/spec/api/#deleting-a-layer) 和 [这里](https://docs.docker.com/registry/spec/api/#deleting-an-image) 的 API 文档以了解详细信息）。这个 API 删除了对目标的引用，并使它们符合垃圾收集的条件。这也使得它们无法通过 API 读取。

如果删除了一个层，则在运行垃圾收集时将其从文件系统中删除。如果清单被删除，如果没有其他清单引用它们，那么它所引用的层将从文件系统中删除。

### 实例[](https://ewhisper.cn/posts/9434/#%E5%AE%9E%E4%BE%8B)

在这个例子中，manifest A 引用了两个层：a 和 b。manifest B 引用了层 a 和层 c。

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br></pre></td><td><pre><code>A <span>-----&gt; a &lt;----- B</span><br>    \<span>--&gt; b     |</span><br>         c &lt;<span>--/</span><br></code><p><i></i>ADA</p></pre></td></tr></tbody></table>

清单 B 通过 API 被删除：

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br></pre></td><td><pre><code><span>A</span> -----&gt; <span>a</span>     <span>B</span><br>    \--&gt; <span>b</span><br>         c<br></code><p><i></i>CSS</p></pre></td></tr></tbody></table>

在这个状态层中，c 不再有引用，并且有资格进行垃圾收集。层 a 删除了一个引用，但没有垃圾收集，因为它仍然被清单 a 引用。代表清单 B 的 blob 有资格进行垃圾收集。

在垃圾收集运行之后，清单 A 和它的 blobs 仍然保留。

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br></pre></td><td><pre><code>A <span>-----&gt; a</span><br>    \<span>--&gt; b</span><br></code><p><i></i>ADA</p></pre></td></tr></tbody></table>

### 关于垃圾收集的更多细节[](https://ewhisper.cn/posts/9434/#%E5%85%B3%E4%BA%8E%E5%9E%83%E5%9C%BE%E6%94%B6%E9%9B%86%E7%9A%84%E6%9B%B4%E5%A4%9A%E7%BB%86%E8%8A%82)

垃圾收集分两个阶段运行。首先，在“标记”阶段，该进程扫描注册表中的所有清单。从这些清单，它构造了一组内容地址摘要。这个集合是“标记集”，表示不删除的集合。其次，在“扫描”阶段，进程扫描所有的 blob，如果 blob 的内容地址摘要不在标记集中，则进程删除它。

垃圾收集分两个阶段运行。首先，在 " 标记 "（mark）阶段，该进程扫描了注册表中的所有清单。从这些清单中，它构建了一个内容地址摘要集。这个集合是 “标记集”，表示不删除的 Blobs 集合。其次，在 “清理”（sweep）阶段，该进程扫描所有的 blob，如果一个 blob 的内容地址摘要不在标记集中，该进程将删除它。

> 注意：您应该确保注册表处于只读模式或根本不运行。如果您在运行垃圾收集时上传图像，则存在图像层被错误删除导致图像损坏的风险。

这种类型的垃圾收集被称为 stop-the-world 垃圾收集。

## 运行垃圾收集[](https://ewhisper.cn/posts/9434/#%E8%BF%90%E8%A1%8C%E5%9E%83%E5%9C%BE%E6%94%B6%E9%9B%86)

垃圾收集可以按如下方式运行

<table><tbody><tr><td><pre><span>1</span><br></pre></td><td><pre><code>bin/registry garbage-collect [--dry-run] /path/to/config.yml<br></code><p><i></i>BASH</p></pre></td></tr></tbody></table>

garbage-collect 命令接受一个 `--dry-run` 参数，该参数在不删除任何数据的情况下打印标记和扫描阶段的进度。使用 `info` 日志级别运行可以清楚地指示哪些项目适合删除。

config.yml 格式如下：

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br></pre></td><td><pre><code><span>version:</span> <span>0.1</span><br><span>storage:</span><br>  <span>filesystem:</span><br>    <span>rootdirectory:</span> <span>/registry/data</span><br></code><p><i></i>YAML</p></pre></td></tr></tbody></table>

_将注册表日志级别设置为 `info` 的演练垃圾收集的示例输出_:

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br><span>8</span><br><span>9</span><br><span>10</span><br><span>11</span><br><span>12</span><br><span>13</span><br></pre></td><td><pre><code><span>hello</span>-world<br><span>hello</span>-world: marking manifest sha<span>256</span>:fea<span>8895</span>f<span>450959</span>fa<span>676</span>bcc<span>1</span>df<span>0611</span>ea<span>93823</span>a<span>735</span>a<span>01205</span>fd<span>8622846041</span>d<span>0</span>c<span>7</span>cf<br><span>hello</span>-world: marking blob sha<span>256</span>:<span>03</span>f<span>4658</span>f<span>8</span>b<span>782</span>e<span>12230</span>c<span>1783426</span>bd<span>3</span>bacce<span>651</span>ce<span>582</span>a<span>4</span>ffb<span>6</span>fbbfa<span>2079428</span>ecb<br><span>hello</span>-world: marking blob sha<span>256</span>:a<span>3</span>ed<span>95</span>caeb<span>02</span>ffe<span>68</span>cdd<span>9</span>fd<span>84406680</span>ae<span>93</span>d<span>633</span>cb<span>16422</span>d<span>00</span>e<span>8</span>a<span>7</span>c<span>22955</span>b<span>46</span>d<span>4</span><br><span>hello</span>-world: marking configuration sha<span>256</span>:<span>690</span>ed<span>74</span>de<span>00</span>f<span>99</span>a<span>7</span>d<span>00</span>a<span>98</span>a<span>5</span>ad<span>855</span>ac<span>4</span>febd<span>66412</span>be<span>132438</span>f<span>9</span>b<span>8</span>dbd<span>300</span>a<span>937</span>d<br><span>ubuntu</span><p><span>4</span> blobs marked, <span>5</span> blobs eligible for deletion<br><span>blob</span> eligible for deletion: sha<span>256</span>:<span>28</span>e<span>09</span>fddaacbfc<span>8</span>a<span>13</span>f<span>82871</span>d<span>9</span>d<span>66141</span>a<span>6</span>ed<span>9</span>ca<span>526</span>cb<span>9</span>ed<span>295</span>ef<span>545</span>ab<span>4559</span>b<span>81</span><br><span>blob</span> eligible for deletion: sha<span>256</span>:<span>7</span>e<span>15</span>ce<span>58</span>ccb<span>2181</span>a<span>8</span>fced<span>7709</span>e<span>9893206</span>f<span>0937</span>cc<span>9543</span>bc<span>0</span>c<span>8178</span>ea<span>1</span>cf<span>4</span>d<span>7</span>e<span>7</span>b<span>5</span><br><span>blob</span> eligible for deletion: sha<span>256</span>:<span>87192</span>bdbe<span>00</span>f<span>8</span>f<span>2</span>a<span>62527</span>f<span>36</span>bb<span>4</span>c<span>7</span>c<span>7</span>f<span>4</span>eaf<span>9307</span>e<span>4</span>b<span>87</span>e<span>8334</span>fb<span>6</span>abec<span>1765</span>bcb<br><span>blob</span> eligible for deletion: sha<span>256</span>:b<span>549</span>a<span>9959</span>a<span>664038</span>fc<span>35</span>c<span>155</span>a<span>95742</span>cf<span>12297672</span>ca<span>0</span>ae<span>35735</span>ec<span>027</span>d<span>55</span>bf<span>4</span>e<span>97</span><br><span>blob</span> eligible for deletion: sha<span>256</span>:f<span>251</span>d<span>679</span>a<span>7</span>c<span>61455</span>f<span>06</span>d<span>793</span>e<span>43</span>c<span>06786</span>d<span>7766</span>c<span>88</span>b<span>8</span>c<span>24</span>edf<span>242</span>b<span>2</span>c<span>08</span>e<span>3</span>c<span>3</span>f<span>599</span></p></code><p><i></i>APACHE</p></pre></td></tr></tbody></table>