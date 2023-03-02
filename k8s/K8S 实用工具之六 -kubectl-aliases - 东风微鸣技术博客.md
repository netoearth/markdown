## 开篇[](https://ewhisper.cn/posts/53971/#%E5%BC%80%E7%AF%87%20-7)

📜 **引言** ：

-   磨刀不误砍柴工
-   工欲善其事必先利其器

-   第一篇：《[K8S 实用工具之一 - 如何合并多个 kubeconfig？](https://ewhisper.cn/posts/46789/)》
-   第二篇：《[K8S 实用工具之二 - 终端 UI K9S](https://ewhisper.cn/posts/48546/)》
-   第三篇：《[K8S 实用工具之三 - 图形化 UI Lens](https://ewhisper.cn/posts/33163/)》
-   第四篇：《[K8S 实用工具之四 - kubectl 实用插件](https://ewhisper.cn/posts/60907/)》
-   第五篇：《[K8S 实用工具之五 -kompose](https://ewhisper.cn/posts/35291/)

## `ahmetb/kubectl-aliases`[](https://ewhisper.cn/posts/53971/#ahmetb-kubectl-aliases)

就是 **一大堆** 的 kubectl alias，目的就是省下敲一长串 kubectl 命令的时间。

地址在这里：[ahmetb/kubectl-aliases](https://github.com/ahmetb/kubectl-aliases)

### 示例[](https://ewhisper.cn/posts/53971/#%E7%A4%BA%E4%BE%8B)

例如：

<table><tbody><tr><td><pre><span>1</span><br></pre></td><td><pre><code><span>alias</span> ksysgdepwslowidel=<span>'kubectl --namespace=kube-system get deployment --watch --show-labels -o=wide -l'</span><br></code><p><i></i>BASH</p></pre></td></tr></tbody></table>

完整的有多少呢，近 800 多个… 以下只是一小部分：

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br><span>8</span><br><span>9</span><br><span>10</span><br><span>11</span><br><span>12</span><br><span>13</span><br><span>14</span><br><span>15</span><br><span>16</span><br><span>17</span><br><span>18</span><br></pre></td><td><pre><code><span>alias</span> k=<span>'kubectl'</span><br><span>alias</span> kg=<span>'kubectl get'</span><br><span>alias</span> kgpo=<span>'kubectl get pod'</span><p><span>alias</span> ksysgpo=<span>'kubectl --namespace=kube-system get pod'</span></p><p><span>alias</span> krm=<span>'kubectl delete'</span><br><span>alias</span> krmf=<span>'kubectl delete -f'</span><br><span>alias</span> krming=<span>'kubectl delete ingress'</span><br><span>alias</span> krmingl=<span>'kubectl delete ingress -l'</span><br><span>alias</span> krmingall=<span>'kubectl delete ingress --all-namespaces'</span></p><p><span>alias</span> kgsvcoyaml=<span>'kubectl get service -o=yaml'</span><br><span>alias</span> kgsvcwn=<span>'kubectl get service --watch --namespace'</span><br><span>alias</span> kgsvcslwn=<span>'kubectl get service --show-labels --watch --namespace'</span></p><p><span>alias</span> kgwf=<span>'kubectl get --watch -f'</span><br>...</p></code><p><i></i>BASH</p></pre></td></tr></tbody></table>

完整列表看 [这里](https://github.com/ahmetb/kubectl-aliases/blob/master/.kubectl_aliases)

### 安装[](https://ewhisper.cn/posts/53971/#%E5%AE%89%E8%A3%85%20-2)

您可以直接下载 bash/zsh 的 `.kubectl_aliases` 文件，并保存到您的 `$HOME` 目录。

然后加到 `.bashrc/.zshrc` 中：

<table><tbody><tr><td><pre><span>1</span><br></pre></td><td><pre><code>[-f ~/.kubectl_aliases] &amp;&amp; <span>source</span> ~/.kubectl_aliases<br></code><p><i></i>BASH</p></pre></td></tr></tbody></table>

### 缩写解释[](https://ewhisper.cn/posts/53971/#%E7%BC%A9%E5%86%99%E8%A7%A3%E9%87%8A)

-   `k`\= `kubectl`
    -   **`sys`**\=`--namespace kube-system`
-   commands:
    -   **`g`**\=`get`
    -   **`d`**\=`describe`
    -   **`rm`**\=`delete`
    -   **`a`**:`apply -f`
    -   **`ak`**:`apply -k`
    -   **`k`**:`kustomize`
    -   **`ex`**: `exec -i -t`
    -   **`lo`**: `logs -f`
-   resources:
    -   **`po`**\=pod, **`dep`**\=`deployment`, **`ing`**\=`ingress`, **`svc`**\=`service`, **`cm`**\=`configmap`, **`sec`**\=`secret`, **`ns`**\=`namespace`, **`no`**\=`node` \*\*
-   flags:
    -   output format: **`oyaml`**, **`ojson`**, **`owide`**
    -   **`all`**: `--all` or `--all-namespaces` depending on the command
    -   **`sl`**: `--show-labels`
    -   **`w`**\=`-w/--watch`
-   value flags (should be at the end):
    -   **`n`**\=`-n/--namespace`
    -   **`f`**\=`-f/--filename`
    -   **`l`**\=`-l/--selector`

## That’s All[](https://ewhisper.cn/posts/53971/#That%E2%80%99s-All-2)

🎉🎉🎉

## 参考链接[](https://ewhisper.cn/posts/53971/#%E5%8F%82%E8%80%83%E9%93%BE%E6%8E%A5%20-2)

[ahmetb/kubectl-aliases](https://github.com/ahmetb/kubectl-aliases)