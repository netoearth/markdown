## 容器操作系统类型[](https://ewhisper.cn/posts/20535/#%E5%AE%B9%E5%99%A8%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F%E7%B1%BB%E5%9E%8B)

### Busybox[](https://ewhisper.cn/posts/20535/#Busybox)

集成了一百多个最常用 Linux 命令和工具的软件工具箱.

包含`cat` `echo` `grep` `find` `mount` `telnet` 等

> Busybox 是 Linux 系统的瑞士军刀

### Debian/Ubuntu[](https://ewhisper.cn/posts/20535/#Debian-Ubuntu)

### CentOS/Fedora[](https://ewhisper.cn/posts/20535/#CentOS-Fedora)

### CoreOS[](https://ewhisper.cn/posts/20535/#CoreOS)

[官网链接](https://coreos.com/)

Linux 发行版, 针对容器技术.

## 创建自定义操作系统的镜像[](https://ewhisper.cn/posts/20535/#%E5%88%9B%E5%BB%BA%E8%87%AA%E5%AE%9A%E4%B9%89%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F%E7%9A%84%E9%95%9C%E5%83%8F)

### 基于 commit 命令创建[](https://ewhisper.cn/posts/20535/#%E5%9F%BA%E4%BA%8E%20commit-%20%E5%91%BD%E4%BB%A4%E5%88%9B%E5%BB%BA)

支持用户提交自己对容器的修改, 并生成新的镜像. 命令格式为:

`docker commit CONTAINER [REPOSITORY[:TAG]]`

#### 创建步骤[](https://ewhisper.cn/posts/20535/#%E5%88%9B%E5%BB%BA%E6%AD%A5%E9%AA%A4)

1.  使用 OS 镜像创建容器
    
2.  配置软件源为国内软件源, 如 ali
    
    <table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br><span>8</span><br><span>9</span><br><span>10</span><br></pre></td><td><pre><code>deb http:<span>//mi</span>rrors.aliyun.com<span>/ubuntu/</span> xenial main restricted universe multiverse<br>deb http:<span>//mi</span>rrors.aliyun.com<span>/ubuntu/</span> xenial-security main restricted universe multiverse<br>deb http:<span>//mi</span>rrors.aliyun.com<span>/ubuntu/</span> xenial-updates main restricted universe multiverse<br>deb http:<span>//mi</span>rrors.aliyun.com<span>/ubuntu/</span> xenial-proposed main restricted universe multiverse<br>deb http:<span>//mi</span>rrors.aliyun.com<span>/ubuntu/</span> xenial-backports main restricted universe multiverse<br>deb-src http:<span>//mi</span>rrors.aliyun.com<span>/ubuntu/</span> xenial main restricted universe multiverse<br>deb-src http:<span>//mi</span>rrors.aliyun.com<span>/ubuntu/</span> xenial-security main restricted universe multiverse<br>deb-src http:<span>//mi</span>rrors.aliyun.com<span>/ubuntu/</span> xenial-updates main restricted universe multiverse<br>deb-src http:<span>//mi</span>rrors.aliyun.com<span>/ubuntu/</span> xenial-proposed main restricted universe multiverse<br>deb-src http:<span>//mi</span>rrors.aliyun.com<span>/ubuntu/</span> xenial-backports main restricted universe multiverse<br></code><p><i></i>AWK</p></pre></td></tr></tbody></table>
    
3.  执行 `apt-get update` 更新软件包缓存
    
4.  通过 `apt-get` 安装服务(比如安装 ssh): `apt-get install openssh-server`
    
5.  创建目录：需要目录 `/var/run/sshd` 存在, 手动创建: `mkdir -p /var/run/sshd`. 此时已经可以启动服务: `/usr/sbin/sshd -D &`
    
6.  修改服务配置, **取消 pam 登陆限制**: `sed -ri 's/session required pam_loginuid.so/#session required pam_loginuid.so/g' /etc/pam.d/sshd`
    
7.  其他操作：在 root 用户目录下创建 `.ssh` 目录, 并复制需要登陆的公钥信息 (1. 直接从用户目录下的 _.ssh/id\_rsa.pub_ 文件拷贝 2. `ssh-keygen -t rsa` 生成) 到 `authorized_keys` 中: `mkdir -p /root/.ssh && vi /root/.ssh/authorized_keys` . 公钥信息格式示例如下:
    
    <table><tbody><tr><td><pre><span>1</span><br></pre></td><td><pre><code><span>ssh</span>-rsa AAAAB<span>3</span>NzaC<span>1</span>yc<span>2</span>EAAAADAQABAAABAQCtuqN<span>2</span>zGhhVBTVCCoNa<span>8</span>hPvGu<span>3</span>xo<span>8</span>+UsqG+AxW<span>0</span>jEUvQYhr<span>6</span>/IEXiIAk<span>41</span>HzjeEZVYKGGr<span>08</span>Jh<span>8</span>n<span>5</span>xxmBW<span>4</span>AyH/<span>1</span>DaU<span>1</span>Ej<span>3</span>m<span>0</span>dOuZ<span>09</span>HAUJfY<span>7</span>WnrtO<span>8</span>GrZtQT<span>2</span>KhI<span>6</span>P<span>2</span>pwnOJU<span>3</span>fm<span>6</span>eRLLVzL<span>2</span>oSyhBQ<span>8</span>ca/njwAyHXOVJiPOpO<span>3</span>cokOPa<span>2</span>BzziWqslmFKyWQdaf<span>6</span>rBwYKF+<span>2</span>eoFrVk<span>0</span>QepoJtc<span>6</span>OfgIyuQEi+gJXste<span>6</span>QiPJRYgFQoYlv/bzYnnrG<span>7</span>Zs<span>0</span>qVCi<span>6</span>SfIRF<span>7</span>twVXUNW/hkPbGxsKZTLAvITS<span>3</span>tOR<span>2</span>nRt<span>6</span>pibT<span>46</span>RM/+ebiuT<span>0</span>fZ/e/xl<span>3</span>w<span>4</span>QygGTB<span>2</span>Xl casey@ubuntu<br></code><p><i></i>APACHE</p></pre></td></tr></tbody></table>
    
8.  其他步骤：创建自动启动的 SSH 服务的可执行文件 `run.sh` , 并添加可执行权限: `vi /run.sh; chmod +x run.sh`
    
    <table><tbody><tr><td><pre><span>1</span><br><span>2</span><br></pre></td><td><pre><code><span>#!/bin/bash</span><br>/usr/sbin/sshd -D<br></code><p><i></i>BASH</p></pre></td></tr></tbody></table>
    
9.  最后, 退出容器 `exit`
    

#### 保存镜像[](https://ewhisper.cn/posts/20535/#%E4%BF%9D%E5%AD%98%E9%95%9C%E5%83%8F)

`sudo docker commit <container id> ubuntu-sshd`

#### 启动镜像[](https://ewhisper.cn/posts/20535/#%E5%90%AF%E5%8A%A8%E9%95%9C%E5%83%8F)

`sudo docker run -p 10122:22 -d ubuntu /run.sh`

#### 使用[](https://ewhisper.cn/posts/20535/#%E4%BD%BF%E7%94%A8)

可以通过 ssh 服务进行连接

`ssh <container ip> -p 10122 -l root`

### 使用 Dockerfile 创建(重点关注)[](https://ewhisper.cn/posts/20535/#%E4%BD%BF%E7%94%A8%20-Dockerfile-%20%E5%88%9B%E5%BB%BA%20-%20%E9%87%8D%E7%82%B9%E5%85%B3%E6%B3%A8)

> [Docker hun 上我的 ubuntu-sshd 镜像](https://hub.docker.com/r/caseycui/ubuntu-sshd/)

#### Dockerfile 示例[](https://ewhisper.cn/posts/20535/#Dockerfile%20%E7%A4%BA%E4%BE%8B)

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br><span>8</span><br><span>9</span><br><span>10</span><br><span>11</span><br><span>12</span><br><span>13</span><br><span>14</span><br><span>15</span><br><span>16</span><br><span>17</span><br><span>18</span><br><span>19</span><br><span>20</span><br><span>21</span><br><span>22</span><br><span>23</span><br><span>24</span><br><span>25</span><br><span>26</span><br><span>27</span><br><span>28</span><br><span>29</span><br><span>30</span><br><span>31</span><br><span>32</span><br><span>33</span><br><span>34</span><br><span>35</span><br><span>36</span><br><span>37</span><br><span>38</span><br><span>39</span><br><span>40</span><br><span>41</span><br><span>42</span><br><span>43</span><br><span>44</span><br></pre></td><td><pre><code><span>## OS 镜像</span><br><span>FROM</span> ubuntu:<span>16.04</span><p><span>## 维护者</span><br><span>LABEL</span><span> maintainer=<span>"CaseyCui cuikaidong@foxmail.com"</span></span></p><p><span>## 1. 备份官方源</span><br><span>## 2. 创建 /var/run/sshd 文件夹（正常启动 SSH 服务需要）</span><br><span>## 3. 创建 root 用户目录下.ssh 目录</span><br><span>RUN</span><span> mv /etc/apt/sources.list /etc/apt/sources.list.default \</span><br><span>    &amp;&amp; mkdir -p /var/run/sshd \</span><br><span>    &amp;&amp; mkdir -p /root/.ssh</span></p><p><span>## 拷贝阿里镜像源信息到 /etc/apt/sources.list</span><br><span>COPY</span><span> sources.list /etc/apt/sources.list</span></p><p><span>## 拷贝公钥到 /root/.ssh 目录</span><br><span>COPY</span><span> authorized_keys /root/.ssh/authorized_keys</span></p><p><span>## 拷贝运行脚本 run-sshd.sh 到根目录下</span><br><span>COPY</span><span> run-sshd.sh /run-sshd.sh</span></p><p><span>## 1. 安装 openssh-server</span><br><span>## 2. 修改时区为 ** 中国 / 上海 **(ubuntu 新版本需要安装 *tzdata*，通过 ** 链接 ** 方式使之生效)</span><br><span>## 3. 修改 SSH 服务的安全登录配置，取消 pam 登录限制</span><br><span>## 4. run-sshd.sh 增加执行权限</span><br><span>RUN</span><span> apt-get update \</span><br><span>    &amp;&amp; DEBIAN_FRONTEND=noninteractive apt-get install -yq --no-install-recommends \</span><br><span>        openssh-server \</span><br><span>        tzdata \</span><br><span>    &amp;&amp; rm -rf /var/lib/apt/lists/* \</span><br><span>    &amp;&amp; ln -fs /usr/share/zoneinfo/Asia/Shanghai /etc/localtime \</span><br><span>    &amp;&amp; dpkg-reconfigure -f noninteractive tzdata \  </span><br>    &amp;&amp; sed -ri <span>'s/session    required     pam_loginuid.so/#session    required     pam_loginuid.so/g'</span> /etc/pam.d/sshd \<br>    &amp;&amp; chmod +x /<span>run</span><span>-sshd.sh</span></p><p><span>## 开放端口</span><br><span>EXPOSE</span> <span>22</span></p><p><span>## 增加挂载点 /tmp</span><br><span>VOLUME</span><span> /tmp</span></p><p><span>## 设置启动命令</span><br><span>CMD</span><span> [<span>"/run-sshd.sh"</span>]</span></p></code><p><i></i>DOCKERFILE</p></pre></td></tr></tbody></table>

> **自动化修改时区**
> 
> ubuntu 16.04 之前:
> 
> `echo "Asia/Shanghai" > /etc/timezone && dpkg-reconfigure -f noninteractive tzdata`
> 
> ubuntu 16.04 及之后:
> 
> `&& ln -fs /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && dpkg-reconfigure -f noninteractive tzdata`
> 
> 前提: 需要安装 tzdata 包:
> 
> `apt-get -yq install tzdata`  
> **ENV 环境变量**
> 
> ENV 环境变量全局生效, 有时可能会有负面效果.
> 
> 如: `ENV DEBIAN_FRONTEND noninteractive` 会把所有操作设置为非交互式的.
> 
> 尽量不要像上边那样使用, 建议的用法是: 在有需要时, 和执行的命令一起执行, 如:
> 
> <table><tbody><tr><td><pre><span>1</span><br><span>2</span><br></pre></td><td><pre><code><span>RUN</span><span> apt-get update &amp;&amp; \</span><br><span>    DEBIAN_FRONTEND=noninteractive apt-get install -yq --no-install-recommends tzdata <span># -q: quiet</span></span><br></code><p><i></i>DOCKERFILE</p></pre></td></tr></tbody></table>

#### .dockerignore 文件[](https://ewhisper.cn/posts/20535/#dockerignore%20%E6%96%87%E4%BB%B6)

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br></pre></td><td><pre><code>## 忽略文件夹 .git/<br>.git<p>## 忽略临时文件<br>*.swp</p></code><p><i></i>CLEAN</p></pre></td></tr></tbody></table>

#### [run.sh](http://run.sh/) 脚本[](https://ewhisper.cn/posts/20535/#run-sh-%20%E8%84%9A%E6%9C%AC)

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br></pre></td><td><pre><code><span>##!/bin/bash</span><br>/usr/sbin/sshd -D<br></code><p><i></i>BASH</p></pre></td></tr></tbody></table>

#### 创建镜像[](https://ewhisper.cn/posts/20535/#%E5%88%9B%E5%BB%BA%E9%95%9C%E5%83%8F%20-3)

在 Dockerfile 目录下执行:

`sudo docker build -t caseycui/ubuntu-sshd .`