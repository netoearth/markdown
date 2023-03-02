## 开篇[](https://ewhisper.cn/posts/46789/#%E5%BC%80%E7%AF%87%20-3)

📜 **引言** ：

-   磨刀不误砍柴工
-   工欲善其事必先利其器

K8S 集群规模，有的公司倾向于少量大规模 K8S 集群，也有的公司会倾向于大量小规模的 K8S 集群。

如果是第二种情况，是否有一个简单的 `kubectl` 命令来获取一个 kubeconfig 文件并将其合并到 `~/.kube/config` 文件作为一个额外的上 context?

🔥 **提示** ：

Kubeconfig 文件会包含 Kubernetes 集群的以下信息：

-   集群
-   上下文（context）
-   用户

有以下解决方案：

## 解决方案[](https://ewhisper.cn/posts/46789/#%E8%A7%A3%E5%86%B3%E6%96%B9%E6%A1%88)

### 方案一：`KUBECONFIG` 环境变量指向多个文件[](https://ewhisper.cn/posts/46789/#%E6%96%B9%E6%A1%88%E4%B8%80%EF%BC%9AKUBECONFIG-%20%E7%8E%AF%E5%A2%83%E5%8F%98%E9%87%8F%E6%8C%87%E5%90%91%E5%A4%9A%E4%B8%AA%E6%96%87%E4%BB%B6)

通过在 KUBECONFIG 环境变量中指定多个文件，可以临时将 KUBECONFIG 文件组合在一起，并在 `kubectl` 中使用。

如下，那么是在 kubeconfig 是在内存中做的合并：

<table><tbody><tr><td><pre><span>1</span><br></pre></td><td><pre><code><span>export</span> KUBECONFIG=~/.kube/config:~/anotherconfig <br></code><p><i></i>BASH</p></pre></td></tr></tbody></table>

### 方案二：`flatten`[](https://ewhisper.cn/posts/46789/#%E6%96%B9%E6%A1%88%E4%BA%8C%EF%BC%9Aflatten)

直接如下：

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br></pre></td><td><pre><code><span>export</span> KUBECONFIG=~/.kube/config:~/anotherconfig <br>kubectl config view --flatten<br></code><p><i></i>BASH</p></pre></td></tr></tbody></table>

如果需要，还可以管道输出到另外一个新文件。

-   `--flatten`：将生成的 kubeconfig 文件扁平化为自包含的输出（用于创建可移植的  
    kubeconfig 文件）

### 方案三：`kubectl` 插件 `konfig`[](https://ewhisper.cn/posts/46789/#%E6%96%B9%E6%A1%88%E4%B8%89%EF%BC%9Akubectl-%20%E6%8F%92%E4%BB%B6%20-konfig)

`kubectl` 有个 `krew` 插件包管理器，可以通过 `krew` 安装 `konfig` 实用插件来管理 kubeconfig。

#### 实用工具：`krew`[](https://ewhisper.cn/posts/46789/#%E5%AE%9E%E7%94%A8%E5%B7%A5%E5%85%B7%EF%BC%9Akrew)

**什么是 `krew`**:

Krew 是 `kubectl` 命令行工具的插件管理器。

Krew 可以帮助你:

-   发现 kubectl 插件
-   将它们安装到您的机器上
-   并保持安装的插件是最新的

目前在 `krew` 上有 [164 个 `kubectl` 插件](https://krew.sigs.k8s.io/plugins/)。

Krew 可以在所有主要平台上工作，比如 macOS、Linux 和 Windows。

Krew 还可以帮助 `kubectl` 插件开发者: 你可以很容易地在多个平台上打包和发布你的插件，并且可以通过 `krew` 集中的插件库来发现它们。

**安装**

Krew 本身是一款通过 Krew 安装和更新的 kubectl 插件（是的，krew 自托管）。

Bash 或 ZSH shell 安装：

1.  如果需要用代理，请先配置 proxy，操作指南：[Advanced Configuration · Krew](https://krew.sigs.k8s.io/docs/user-guide/advanced-configuration/#custom-network-proxy)
    
2.  确认已安装 `git`
    
3.  下载并安装 `krew`:
    
    <table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br><span>8</span><br><span>9</span><br></pre></td><td><pre><code>(<br>  <span>set</span> -x; <span>cd</span> <span>"<span>$(mktemp -d)</span>"</span> &amp;&amp;<br>  OS=<span>"<span>$(uname | tr '[:upper:]' '[:lower:]')</span>"</span> &amp;&amp;<br>  ARCH=<span>"<span>$(uname -m | sed -e 's/x86_64/amd64/' -e 's/\(arm\)\(64\)\?.*/\1\2/' -e 's/aarch64$/arm64/')</span>"</span> &amp;&amp;<br>  KREW=<span>"krew-<span>${OS}</span>_<span>${ARCH}</span>"</span> &amp;&amp;<br>  curl -fsSLO <span>"https://github.com/kubernetes-sigs/krew/releases/latest/download/<span>${KREW}</span>.tar.gz"</span> &amp;&amp;<br>  tar zxvf <span>"<span>${KREW}</span>.tar.gz"</span> &amp;&amp;<br>  ./<span>"<span>${KREW}</span>"</span> install krew<br>)<br></code><p><i></i>BASH</p></pre></td></tr></tbody></table>
    
4.  添加 `krew` 到 `PATH`: `export PATH="${KREW_ROOT:-$HOME/.krew}/bin:$PATH"` 并重启 shell
    
5.  运行 `kubectl krew` 来验证
    
6.  要看完整的插件列表，运行：`kubectl krew search`
    

#### 实用工具：`konfig`[](https://ewhisper.cn/posts/46789/#%E5%AE%9E%E7%94%A8%E5%B7%A5%E5%85%B7%EF%BC%9Akonfig)

安装：`kubectl krew install konfig`

`krew` 插件 `konfig` 可以帮助你管理 `~/.kube/config`。

使用 `konfig` 插件的语法如下:

<table><tbody><tr><td><pre><span>1</span><br></pre></td><td><pre><code>kubectl konfig import -s new.yaml<br></code><p><i></i>BASH</p></pre></td></tr></tbody></table>

## 总结[](https://ewhisper.cn/posts/46789/#%E6%80%BB%E7%BB%93%20-6)

今天分享了 2 个实用插件：

1.  **krew**：`kubectl` 插件管理器
2.  **konfig**：kubeconfig 配置管理插件

🎉🎉🎉