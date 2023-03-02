“工欲善其事必先利其器”, 作为一个 PAAS 平台架构师, 容器相关技术 (docker, k8s 等) 是必不可少的. 本文简单介绍下我自己的 Linux 操作机配置. 提升工作效率, 提高使用体验. ❤[](https://github.githubassets.com/images/icons/emoji/unicode/2764.png?v8)❤[](https://github.githubassets.com/images/icons/emoji/unicode/2764.png?v8)❤[](https://github.githubassets.com/images/icons/emoji/unicode/2764.png?v8)

> ❗[](https://github.githubassets.com/images/icons/emoji/unicode/2757.png?v8) 注意:
> 
> 本文以 CentOS 7.6 为例, RHEL7.6 操作类似.
> 
> Ubuntu 系统操作可以触类旁通. 没啥难度.
> 
> 另外下文中会有一些 " 可选 " 项, 主要是针对一些特殊情况, 如: 需要通过代理连接互联网…

## 更换 OS 软件安装源[](https://ewhisper.cn/posts/55821/#%E6%9B%B4%E6%8D%A2%20OS-%20%E8%BD%AF%E4%BB%B6%E5%AE%89%E8%A3%85%E6%BA%90)

目的: 加快软件下载速度.

可以换成: 阿里, 腾讯, 清华, 中科大…的源.

以清华 Mirror 为例, 操作步骤如下:

> 🔖[](https://github.githubassets.com/images/icons/emoji/unicode/1f516.png?v8) 参考文章:
> 
> 清华大学开源软件镜像站 - CentOS 镜像使用帮助[https://mirrors.tuna.tsinghua.edu.cn/help/centos/](https://mirrors.tuna.tsinghua.edu.cn/help/centos/)

### 操作步骤[](https://ewhisper.cn/posts/55821/#%E6%93%8D%E4%BD%9C%E6%AD%A5%E9%AA%A4)

1.  先备份 CentOS-Base.repo
    
    <table><tbody><tr><td><pre><span>1</span><br></pre></td><td><pre><code>sudo cp /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.bak<br></code><p><i></i>SHELL</p></pre></td></tr></tbody></table>
    
2.  用下面内容覆盖`CentOS-Base.repo`
    
    <table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br><span>8</span><br><span>9</span><br><span>10</span><br><span>11</span><br><span>12</span><br><span>13</span><br><span>14</span><br><span>15</span><br><span>16</span><br><span>17</span><br><span>18</span><br><span>19</span><br><span>20</span><br><span>21</span><br><span>22</span><br><span>23</span><br><span>24</span><br><span>25</span><br><span>26</span><br><span>27</span><br><span>28</span><br><span>29</span><br><span>30</span><br><span>31</span><br><span>32</span><br><span>33</span><br><span>34</span><br><span>35</span><br><span>36</span><br><span>37</span><br><span>38</span><br><span>39</span><br><span>40</span><br><span>41</span><br><span>42</span><br><span>43</span><br><span>44</span><br><span>45</span><br><span>46</span><br><span>47</span><br><span>48</span><br><span>49</span><br><span>50</span><br><span>51</span><br></pre></td><td><pre><code><span># CentOS-Base.repo</span><br><span>#</span><br><span># The mirror system uses the connecting IP address of the client and the</span><br><span># update status of each mirror to pick mirrors that are updated to and</span><br><span># geographically close to the client.  You should use this for CentOS updates</span><br><span># unless you are manually picking other mirrors.</span><br><span>#</span><br><span># If the mirrorlist= does not work for you, as a fall back you can try the</span><br><span># remarked out baseurl= line instead.</span><br><span>#</span><br><span>#</span><p><span>[base]</span><br><span>name</span>=CentOS-<span>$releasever</span> - Base<br><span>baseurl</span>=https://mirrors.tuna.tsinghua.edu.cn/centos/<span>$releasever</span>/os/<span>$basearch</span>/<br><span>#mirrorlist=http://mirrorlist.centos.org/?release=$releasever&amp;arch=$basearch&amp;repo=os</span><br><span>enabled</span>=<span>1</span><br><span>gpgcheck</span>=<span>1</span><br><span>gpgkey</span>=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-<span>7</span></p><p><span>#released updates</span><br><span>[updates]</span><br><span>name</span>=CentOS-<span>$releasever</span> - Updates<br><span>baseurl</span>=https://mirrors.tuna.tsinghua.edu.cn/centos/<span>$releasever</span>/updates/<span>$basearch</span>/<br><span>#mirrorlist=http://mirrorlist.centos.org/?release=$releasever&amp;arch=$basearch&amp;repo=updates</span><br><span>enabled</span>=<span>1</span><br><span>gpgcheck</span>=<span>1</span><br><span>gpgkey</span>=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-<span>7</span></p><p><span>#additional packages that may be useful</span><br><span>[extras]</span><br><span>name</span>=CentOS-<span>$releasever</span> - Extras<br><span>baseurl</span>=https://mirrors.tuna.tsinghua.edu.cn/centos/<span>$releasever</span>/extras/<span>$basearch</span>/<br><span>#mirrorlist=http://mirrorlist.centos.org/?release=$releasever&amp;arch=$basearch&amp;repo=extras</span><br><span>enabled</span>=<span>1</span><br><span>gpgcheck</span>=<span>1</span><br><span>gpgkey</span>=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-<span>7</span></p><p><span>#additional packages that extend functionality of existing packages</span><br><span>[centosplus]</span><br><span>name</span>=CentOS-<span>$releasever</span> - Plus<br><span>baseurl</span>=https://mirrors.tuna.tsinghua.edu.cn/centos/<span>$releasever</span>/centosplus/<span>$basearch</span>/<br><span>#mirrorlist=http://mirrorlist.centos.org/?release=$releasever&amp;arch=$basearch&amp;repo=centosplus</span><br><span>gpgcheck</span>=<span>1</span><br><span>enabled</span>=<span>0</span><br><span>gpgkey</span>=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-<span>7</span></p></code><p><i></i>INI</p></pre></td></tr></tbody></table>
    
3.  更新软件包缓存
    
    <table><tbody><tr><td><pre><span>1</span><br></pre></td><td><pre><code>sudo yum makecache<br></code><p><i></i>SHELL</p></pre></td></tr></tbody></table>
    

## 配置代理(可选)[](https://ewhisper.cn/posts/55821/#%E9%85%8D%E7%BD%AE%E4%BB%A3%E7%90%86%20-%20%E5%8F%AF%E9%80%89)

`sudo vi /etc/profile.d/setproxy.sh`

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br><span>8</span><br><span>9</span><br><span>10</span><br><span>11</span><br><span>12</span><br><span>13</span><br><span>14</span><br><span>15</span><br></pre></td><td><pre><code><span>#</span><span>!/bin/sh</span><br><span></span><br><span>#</span><span> <span>for</span> terminal</span><br>export proxyserveraddr=127.0.0.1<br>export proxyserverport=8080<br>export HTTP_PROXY="http://$proxyserveraddr:$proxyserverport/"<br>export HTTPS_PROXY="http://$proxyserveraddr:$proxyserverport/"<br><span>#</span><span> <span>export</span> FTP_PROXY=<span>"ftp://<span>$proxyserveraddr</span>:<span>$proxyserverport</span>/"</span></span><br><span>#</span><span> <span>export</span> SOCKS_PROXY=<span>"socks://<span>$proxyserveraddr</span>:<span>$proxyserverport</span>/"</span></span><br>export NO_PROXY="localhost,127.0.0.1,localaddress,.localdomain.com"<br>export http_proxy="http://$proxyserveraddr:$proxyserverport/"<br>export https_proxy="http://$proxyserveraddr:$proxyserverport/"<br><span>#</span><span> <span>export</span> ftp_proxy=<span>"ftp://<span>$proxyserveraddr</span>:<span>$proxyserverport</span>/"</span></span><br><span>#</span><span> <span>export</span> socks_proxy=<span>"socks://<span>$proxyserveraddr</span>:<span>$proxyserverport</span>/"</span></span><br>export no_proxy="localhost,127.0.0.1,localaddress,.localdomain.com"<br></code><p><i></i>SHELL</p></pre></td></tr></tbody></table>

`sudo source /etc/profile.d/setproxy.sh`

**YUM 配置代理**

<table><tbody><tr><td><pre><span>1</span><br></pre></td><td><pre><code>echo "proxy=http://127.0.0.1:8080" &gt;&gt; /etc/yum.conf<br></code><p><i></i>SHELL</p></pre></td></tr></tbody></table>

## 安装及配置 Git[](https://ewhisper.cn/posts/55821/#%E5%AE%89%E8%A3%85%E5%8F%8A%E9%85%8D%E7%BD%AE%20Git)

目的: 使用 Git, 毕竟很多资料、代码库和软件都需要通过`git clone`

### 步骤[](https://ewhisper.cn/posts/55821/#%E6%AD%A5%E9%AA%A4%20-2)

1.  `sudo yum install -y git`
    
2.  配置全局用户: `git config --global user.name "<username>"`
    
3.  配置全局 email: `git config --global user.email "<username@example.com>"`
    
4.  (可选): 配置 ssh 认证
    
    1.  🔖[](https://github.githubassets.com/images/icons/emoji/unicode/1f516.png?v8) 参考文档: GitHub - 使用 SSH 连接到 GitHub [https://docs.github.com/cn/github/authenticating-to-github/connecting-to-github-with-ssh](https://docs.github.com/cn/github/authenticating-to-github/connecting-to-github-with-ssh)
5.  (可选): 配置代理 Proxy
    
    <table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br><span>8</span><br><span>9</span><br><span>10</span><br><span>11</span><br><span>12</span><br></pre></td><td><pre><code><span>#</span><span> 查看当前代理设置</span><br>git config --global http.proxy<br><span></span><br><span>#</span><span> 设置当前代理为 http://127.0.0.1::8080 或 socket5://127.0.0.1::8080</span><br>git config --global http.proxy 'http://127.0.0.1::8080'<br>git config --global https.proxy 'http://127.0.0.1::8080'<br>git config --global http.proxy 'socks5://127.0.0.1::8080'<br>git config --global https.proxy 'socks5://127.0.0.1::8080' <br><span></span><br><span>#</span><span> 删除 proxy</span> <br>git config --global --unset http.proxy<br>git config --global --unset https.proxy<br></code><p><i></i>SHELL</p></pre></td></tr></tbody></table>
    
6.  (可选): 配置 Proxy Bypass, 如对应仓库的 origin 需要 Bypass: `git config --add remote.origin.proxy ""`
    

## 优化配置 Shell[](https://ewhisper.cn/posts/55821/#%E4%BC%98%E5%8C%96%E9%85%8D%E7%BD%AE%20Shell)

目的: zsh + plugins, 提供丰富而友好的 shell 体验. 如: 语法高亮, 自动补全, 自动建议, 容器相关插件…

### 安装 zsh[](https://ewhisper.cn/posts/55821/#%E5%AE%89%E8%A3%85%20zsh)

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br></pre></td><td><pre><code>sudo yum install -y zsh<br>zsh --version<br>sudo chsh -s <span>$(<span>which</span> <span>zsh</span>)</span><br># 注销<br></code><p><i></i>REASONML</p></pre></td></tr></tbody></table>

### 安装 powerline[](https://ewhisper.cn/posts/55821/#%E5%AE%89%E8%A3%85%20-powerline)

可以通过 `pip` 安装:

<table><tbody><tr><td><pre><span>1</span><br></pre></td><td><pre><code>pip install powerline-status<br></code><p><i></i>SHELL</p></pre></td></tr></tbody></table>

> 🔖[](https://github.githubassets.com/images/icons/emoji/unicode/1f516.png?v8) 参考文章:
> 
> powerline - Installation: [https://powerline.readthedocs.io/en/latest/installation.html#pip-installation](https://powerline.readthedocs.io/en/latest/installation.html#pip-installation)

### 安装 oh-my-zsh[](https://ewhisper.cn/posts/55821/#%E5%AE%89%E8%A3%85%20-oh-my-zsh)

<table><tbody><tr><td><pre><span>1</span><br></pre></td><td><pre><code>sh -c "$(wget https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh -O -)"<br></code><p><i></i>SHELL</p></pre></td></tr></tbody></table>

> ⚠[](https://github.githubassets.com/images/icons/emoji/unicode/26a0.png?v8) 注意:
> 
> 如果连不上: <[raw.githubusercontent.com](http://raw.githubusercontent.com/)\>, 就从 github 对应的地址: [https://github.com/ohmyzsh/ohmyzsh/blob/master/tools/install.sh](https://github.com/ohmyzsh/ohmyzsh/blob/master/tools/install.sh)把脚本复制下来运行.

### 安装 zsh 插件: `zsh-autosuggestions` 和 `zsh-syntax-highlighting`[](https://ewhisper.cn/posts/55821/#%E5%AE%89%E8%A3%85%20zsh%20%E6%8F%92%E4%BB%B6%20-zsh-autosuggestions-%20%E5%92%8C%20-zsh-syntax-highlighting)

> 🔖[](https://github.githubassets.com/images/icons/emoji/unicode/1f516.png?v8) 参考文档:
> 
> -   zsh-syntax-highlighting [INSTALL.md](http://install.md/): [https://github.com/zsh-users/zsh-syntax-highlighting/blob/master/INSTALL.md](https://github.com/zsh-users/zsh-syntax-highlighting/blob/master/INSTALL.md)

#### `zsh-syntax-highlighting`[](https://ewhisper.cn/posts/55821/#zsh-syntax-highlighting)

1.  clone: `git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting`
2.  在 `~/.zshrc` 激活插件: `plugins=([plugins...] zsh-syntax-highlighting)`
3.  重启 zsh

#### `zsh-autosuggestions`[](https://ewhisper.cn/posts/55821/#zsh-autosuggestions)

1.  clone: `git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions`
2.  在 `~/.zshrc` 激活插件: `plugins=([plugins...] zsh-autosuggestions)`
3.  重启 zsh

### 使用 oh-my-zsh[](https://ewhisper.cn/posts/55821/#%E4%BD%BF%E7%94%A8%20-oh-my-zsh)

编辑 zshrc 文件: `vi ~/.zshrc`

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br><span>8</span><br><span>9</span><br><span>10</span><br><span>11</span><br><span>12</span><br><span>13</span><br><span>14</span><br><span>15</span><br><span>16</span><br><span>17</span><br><span>18</span><br><span>19</span><br><span>20</span><br></pre></td><td><pre><code><span>#</span><span> 修改主题</span><br>ZSH_THEME="agnoster"<br><span></span><br><span>#</span><span> 启用插件</span><br>plugins=(<br>  git<br>  ansible<br>  docker-compose<br>  docker<br>  helm<br>  kubectl<br>  minikube<br>  oc<br>  pip<br>  python<br>  ubuntu<br>  zsh-autosuggestions<br>  zsh-syntax-highlighting<br>)</code><p><i></i>SHELL</p></pre></td></tr></tbody></table>

> 📓[](https://github.githubassets.com/images/icons/emoji/unicode/1f4d3.png?v8) 备注:
> 
> -   helm: k8s 上的镜像包管理工具
> -   minikube: 最小化 K8S 安装工具
> -   oc: K8S 的 RedHat 商业发行版 (OpenShift) 的命令行工具

### 最终效果[](https://ewhisper.cn/posts/55821/#%E6%9C%80%E7%BB%88%E6%95%88%E6%9E%9C)

[![](https://pic-cdn.ewhisper.cn/img/2021/10/31/c4b0eec24ff090d8d6bfe55ee438a91a-fla1bsGqRnSeyvL.png)](https://pic-cdn.ewhisper.cn/img/2021/10/31/c4b0eec24ff090d8d6bfe55ee438a91a-fla1bsGqRnSeyvL.png)

## 按需安装常用软件[](https://ewhisper.cn/posts/55821/#%E6%8C%89%E9%9C%80%E5%AE%89%E8%A3%85%E5%B8%B8%E7%94%A8%E8%BD%AF%E4%BB%B6)

目的: 根据自己需要, 按需安装常用软件和工具

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br></pre></td><td><pre><code>sudo yum -y install dnsmasq httpd haproxy nginx \<br>                    python3 \<br>                    genisoimage libguestfs-tools<br></code><p><i></i>SHELL</p></pre></td></tr></tbody></table>

按需配置服务和开机自启动:

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br></pre></td><td><pre><code>systemctl enable haproxy.service <br>systemctl start haproxy.service <br>...<br></code><p><i></i>SHELL</p></pre></td></tr></tbody></table>

安装 jq, jq 安装链接[https://stedolan.github.io/jq/download/](https://stedolan.github.io/jq/download/). JQ 是个 json 格式化命令行工具, 在日常管理 K8S 中很有用.

### 安装容器类组件[](https://ewhisper.cn/posts/55821/#%E5%AE%89%E8%A3%85%E5%AE%B9%E5%99%A8%E7%B1%BB%E7%BB%84%E4%BB%B6)

#### docker 全家桶[](https://ewhisper.cn/posts/55821/#docker%20%E5%85%A8%E5%AE%B6%E6%A1%B6)

建议直接安装 docker 全家桶, 省心省力

> 🔖[](https://github.githubassets.com/images/icons/emoji/unicode/1f516.png?v8) 参考文档:
> 
> Install Docker Engine on CentOS: [https://docs.docker.com/engine/install/centos/](https://docs.docker.com/engine/install/centos/)

1.  卸载老版本:
    
    <table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br><span>8</span><br></pre></td><td><pre><code><span>$</span><span> sudo yum remove docker \</span><br><span>                  docker-client \</span><br><span>                  docker-client-latest \</span><br><span>                  docker-common \</span><br><span>                  docker-latest \</span><br><span>                  docker-latest-logrotate \</span><br><span>                  docker-logrotate \</span><br><span>                  docker-engine</span><br></code><p><i></i>SHELL</p></pre></td></tr></tbody></table>
    
2.  配置 REPOSITORY
    
    <table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br></pre></td><td><pre><code><span>$</span><span> sudo yum install -y yum-utils</span><br><span></span><br><span>$</span><span> sudo yum-config-manager \</span><br><span>    --add-repo \</span><br><span>    https://download.docker.com/linux/centos/docker-ce.repo</span><br></code><p><i></i>SHELL</p></pre></td></tr></tbody></table>
    
3.  安装:
    
    <table><tbody><tr><td><pre><span>1</span><br></pre></td><td><pre><code><span>$</span><span> sudo yum install docker-ce docker-ce-cli containerd.io</span><br></code><p><i></i>SHELL</p></pre></td></tr></tbody></table>
    
4.  启动:
    
    <table><tbody><tr><td><pre><span>1</span><br></pre></td><td><pre><code><span>$</span><span> sudo systemctl start docker</span><br></code><p><i></i>SHELL</p></pre></td></tr></tbody></table>
    
5.  验证:
    
    <table><tbody><tr><td><pre><span>1</span><br></pre></td><td><pre><code><span>$</span><span> sudo docker run hello-world</span><br></code><p><i></i>SHELL</p></pre></td></tr></tbody></table>
    

#### 其他开源组件[](https://ewhisper.cn/posts/55821/#%E5%85%B6%E4%BB%96%E5%BC%80%E6%BA%90%E7%BB%84%E4%BB%B6)

对于 RedHat 系, 可能要安装多个组件替代:

<table><tbody><tr><td><pre><span>1</span><br></pre></td><td><pre><code>sudo yum -y install buildah podman skopeo<br></code><p><i></i>SHELL</p></pre></td></tr></tbody></table>

> 📓[](https://github.githubassets.com/images/icons/emoji/unicode/1f4d3.png?v8) 备注:
> 
> -   buildah: 构建容器镜像的组件
> -   podman: 运行容器镜像的组件
> -   skopeo: 传输移动容器镜像的组件

### 安装 `kubectl`[](https://ewhisper.cn/posts/55821/#%E5%AE%89%E8%A3%85%20-kubectl)

官方安装文档: [https://kubernetes.io/zh/docs/tasks/tools/install-kubectl/](https://kubernetes.io/zh/docs/tasks/tools/install-kubectl/)

1.  下载: `curl -LO "https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl"`
2.  标记 `kubectl` 文件为可执行：`chmod +x ./kubectl`
3.  将文件放到 PATH 路径下：`sudo mv ./kubectl /usr/local/bin/kubectl`
4.  测试你所安装的版本是最新的：`kubectl version --client`

### 安装 minikube 或 kind[](https://ewhisper.cn/posts/55821/#%E5%AE%89%E8%A3%85%20-minikube-%20%E6%88%96%20-kind)

这里以 minikube 为例:

官方安装文档: [https://kubernetes.io/zh/docs/tasks/tools/install-minikube/](https://kubernetes.io/zh/docs/tasks/tools/install-minikube/)

⚠[](https://github.githubassets.com/images/icons/emoji/unicode/26a0.png?v8) **需要强调的是:**

1.  看中文文档
2.  📓[](https://github.githubassets.com/images/icons/emoji/unicode/1f4d3.png?v8) 说明： 由于国内无法直接连接 [k8s.gcr.io](http://k8s.gcr.io/)，推荐使用阿里云镜像仓库，在 `minikube start` 中添加`--image-repository` 参数。
3.  示例: `minikube start --vm-driver=< 驱动名称 > --image-repository=registry.cn-hangzhou.aliyuncs.com/google_containers`

### 安装 helm v3[](https://ewhisper.cn/posts/55821/#%E5%AE%89%E8%A3%85%20-helm-v3)

二进制 CLI 下载地址[https://github.com/helm/helm/releases/latest](https://github.com/helm/helm/releases/latest)

安装源文档: [https://helm.sh/docs/intro/install/](https://helm.sh/docs/intro/install/)

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br></pre></td><td><pre><code><span>$</span><span> curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3</span><br><span>$</span><span> chmod 700 get_helm.sh</span><br><span>$</span><span> ./get_helm.sh</span><br></code><p><i></i>SHELL</p></pre></td></tr></tbody></table>

### 安装 OpenShift 命令行 `oc`[](https://ewhisper.cn/posts/55821/#%E5%AE%89%E8%A3%85%20OpenShift-%20%E5%91%BD%E4%BB%A4%E8%A1%8C%20-oc)

直接下载二进制 CLI 安装: [https://mirror.openshift.com/pub/openshift-v4/clients/oc/](https://mirror.openshift.com/pub/openshift-v4/clients/oc/)

### 安装 OpenShift for Developer 命令行`odo`[](https://ewhisper.cn/posts/55821/#%E5%AE%89%E8%A3%85%20OpenShift-for-Developer%20%E5%91%BD%E4%BB%A4%E8%A1%8C%20odo)

直接下载二进制 CLI 安装:[https://mirror.openshift.com/pub/openshift-v4/clients/odo/latest/](https://mirror.openshift.com/pub/openshift-v4/clients/odo/latest/)

### 安装 Tekton - K8S 原生 CI/CD 工具[](https://ewhisper.cn/posts/55821/#%E5%AE%89%E8%A3%85%20-Tekton-K8S%20%E5%8E%9F%E7%94%9F%20CI-CD%20%E5%B7%A5%E5%85%B7)

CLI 工具叫做`tkn`, 官方文档: [https://github.com/tektoncd/cli](https://github.com/tektoncd/cli)

安装:

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br></pre></td><td><pre><code><span>#</span><span> Get the tar.xz</span><br>curl -LO https://github.com/tektoncd/cli/releases/download/v0.12.0/tkn_0.12.0_Darwin_x86_64.tar.gz<br><span>#</span><span> Extract tkn to your PATH (e.g. /usr/<span>local</span>/bin)</span><br>sudo tar xvzf tkn_0.12.0_Darwin_x86_64.tar.gz -C /usr/local/bin tkn<br></code><p><i></i>SHELL</p></pre></td></tr></tbody></table>

## 完[](https://ewhisper.cn/posts/55821/#%E5%AE%8C%20-2)

当然, `golang` 环境也是必不可少的.

最后祝大家用的顺手! 💪[](https://github.githubassets.com/images/icons/emoji/unicode/1f4aa.png?v8)💪[](https://github.githubassets.com/images/icons/emoji/unicode/1f4aa.png?v8)💪[](https://github.githubassets.com/images/icons/emoji/unicode/1f4aa.png?v8)