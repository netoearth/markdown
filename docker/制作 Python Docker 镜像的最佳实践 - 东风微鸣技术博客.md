## 概述[](https://ewhisper.cn/posts/25776/#%E6%A6%82%E8%BF%B0)

> 📚️**Reference**:
> 
> [制作容器镜像的最佳实践](https://ewhisper.cn/posts/8023/)

这篇文章是关于制作 Python Docker 容器镜像的最佳实践。(2022 年 12 月更新）  
最佳实践的目的一方面是为了减小镜像体积，提升 DevOps 效率，另一方面是为了提高安全性。希望对各位有所帮助。

这里也再次罗列一下对 Python Docker 镜像也适用的一些通用最佳实践。

-   使用 `LABEL maintainer`
-   标记重要端口
-   设置环境变量
-   使用非 root 用户运行容器进程
-   使用 `.dockerignore` 排除无关文件

### Python 镜像推荐设置的环境变量[](https://ewhisper.cn/posts/25776/#Python-%20%E9%95%9C%E5%83%8F%E6%8E%A8%E8%8D%90%E8%AE%BE%E7%BD%AE%E7%9A%84%E7%8E%AF%E5%A2%83%E5%8F%98%E9%87%8F)

Python 中推荐的常见环境变量如下：

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br></pre></td><td><pre><code><span># 设置环境变量 </span><br><span>ENV</span> PYTHONDONTWRITEBYTECODE <span>1</span><br><span>ENV</span> PYTHONUNBUFFERED <span>1</span><br></code><p><i></i>DOCKERFILE</p></pre></td></tr></tbody></table>

1.  `ENV PYTHONDONTWRITEBYTECODE 1`: 建议构建 Docker 镜像时一直为 `1`, 防止 python 将 pyc 文件写入硬盘
2.  `ENV PYTHONUNBUFFERED 1`: 建议构建 Docker 镜像时一直为 `1`, 防止 python 缓冲 (buffering) stdout 和 stderr, 以便更容易地进行容器日志记录
3.  ❌不再建议使用 `ENV DEBUG 0` 环境变量，没必要。

### 使用非 root 用户运行容器进程[](https://ewhisper.cn/posts/25776/#%E4%BD%BF%E7%94%A8%E9%9D%9E%20-root-%20%E7%94%A8%E6%88%B7%E8%BF%90%E8%A1%8C%E5%AE%B9%E5%99%A8%E8%BF%9B%E7%A8%8B)

出于安全考虑，推荐运行 Python 程序前，创建 非 root 用户并切换到该用户。

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br></pre></td><td><pre><code><span># 创建一个具有明确 UID 的非 root 用户，并增加访问 /app 文件夹的权限。</span><br><span>RUN</span><span> adduser -u 5678 --disabled-password --gecos <span>""</span> appuser &amp;&amp; <span>chown</span> -R appuser /app</span><br><span>USER</span> appuser<br></code><p><i></i>DOCKERFILE</p></pre></td></tr></tbody></table>

### 使用 `.dockerignore` 排除无关文件[](https://ewhisper.cn/posts/25776/#%E4%BD%BF%E7%94%A8%20-dockerignore-%20%E6%8E%92%E9%99%A4%E6%97%A0%E5%85%B3%E6%96%87%E4%BB%B6)

需要排除的无关文件一般如下：

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br><span>8</span><br><span>9</span><br><span>10</span><br><span>11</span><br><span>12</span><br><span>13</span><br><span>14</span><br><span>15</span><br><span>16</span><br><span>17</span><br><span>18</span><br><span>19</span><br><span>20</span><br><span>21</span><br><span>22</span><br><span>23</span><br><span>24</span><br><span>25</span><br><span>26</span><br><span>27</span><br><span>28</span><br><span>29</span><br></pre></td><td><pre><code>**/__pycache__<br>**/*venv<br>**/.classpath<br>**/.dockerignore<br>**/.env<br>**/.git<br>**/.gitignore<br>**/.project<br>**/.settings<br>**/.toolstarget<br>**/.vs<br>**/.vscode<br>**/*.*proj.user<br>**/*.dbmdl<br>**/*.jfm<br>**/bin<br>**/charts<br>**/docker-compose*<br>**/compose*<br>**/Dockerfile*<br>**/node_modules<br>**/npm-debug.log<br>**/obj<br>**/secrets.dev.yaml<br>**/values.dev.yaml<br>*.db<br>.python-version<br>LICENSE<br>README.md<br></code><p><i></i>.DOCKERIGNORE</p></pre></td></tr></tbody></table>

这里选择几个说明下：

1.  `**/__pycache__`: python 缓存目录
2.  `**/*venv`: Python 虚拟环境目录。很多 Python 开发习惯将虚拟环境目录创建在项目下，一般命名为：`.venv` 或 `venv`
3.  `**/.env`: Python 环境变量文件
4.  `**/.git` `**/.gitignore`: git 相关目录和文件
5.  `**/.vscode`: 编辑器、IDE 相关目录
6.  `**/charts`: Helm Chart 相关文件
7.  `**/docker-compose*`: docker compose 相关文件
8.  `*.db`: 如果使用 sqllite 的相关数据库文件
9.  `.python-version`: pyenv 的 .python-version 文件

## 不建议使用 Alpine 作为 Python 的基础镜像[](https://ewhisper.cn/posts/25776/#%E4%B8%8D%E5%BB%BA%E8%AE%AE%E4%BD%BF%E7%94%A8%20-Alpine-%20%E4%BD%9C%E4%B8%BA%20-Python-%20%E7%9A%84%E5%9F%BA%E7%A1%80%E9%95%9C%E5%83%8F)

为什么呢？大多数 Linux 发行版使用 GNU 版本（glibc）的标准 C 库，几乎每个 C 程序都需要这个库，包括 Python。但是 Alpine Linux 使用 musl, Alpine 禁用了 Linux wheel 支持。

理由如下：

-   缺少大量依赖
    -   CPython 语言运行时的相关依赖
    -   openssl 相关依赖
    -   libffi 相关依赖
    -   gcc 相关依赖
    -   数据库驱动相关依赖
    -   pip 相关依赖
-   构建可能更耗时
    -   Alpine Linux 使用 musl，一些二进制 wheel 是针对 glibc 编译的，但是 Alpine 禁用了 Linux wheel 支持。现在大多数 Python 包都包括 PyPI 上的二进制 wheel，大大加快了安装时间。但是如果你使用 Alpine Linux，你可能需要编译你使用的每个 Python 包中的所有 C 代码。
-   基于 Alpine 构建的 Python 镜像反而可能更大
    -   乍一听似乎违反常识，但是仔细一想，因为上面罗列的原因，确实会导致镜像更大的情况。

> 📚️**Reference:**
> 
> [Using Alpine can make Python Docker builds 50× slower (pythonspeed.com)](https://pythonspeed.com/articles/alpine-docker-python/)

这里以这个 [Demo FastAPI Python 程序](https://github.com/east4ming/fastapi-url-shortener) 为例，其基于 Alpine 的 Dockerfile 地址是这个：[https://github.com/east4ming/fastapi-url-shortener/blob/main/Dockerfile.alpine](https://github.com/east4ming/fastapi-url-shortener/blob/main/Dockerfile.alpine)

因为缺少很多依赖，所以在用 pip 安装之前，就需要尽可能全地安装相关依赖：

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br></pre></td><td><pre><code><span>RUN</span><span> <span>set</span> -eux \</span><br><span>    &amp;&amp; apk add --no-cache --virtual .build-deps build-base \</span><br><span>    openssl-dev libffi-dev gcc musl-dev python3-dev \</span><br><span>    &amp;&amp; pip install --upgrade pip setuptools wheel \</span><br><span>    &amp;&amp; pip install --upgrade -r /app/requirements.txt \</span><br><span>    &amp;&amp; <span>rm</span> -rf /root/.cache/pip</span><br></code><p><i></i>DOCKERFILE</p></pre></td></tr></tbody></table>

这里也展示一下基于 Alpine 构建完成后的 镜像未压缩大小：

[![基于 Alpine 的 Python Demo 镜像大小：472 MB](https://pic-cdn.ewhisper.cn/img/2022/12/14/e0b0727904557d08fd1ab7db29308ba4-20221214155509.png)](https://pic-cdn.ewhisper.cn/img/2022/12/14/e0b0727904557d08fd1ab7db29308ba4-20221214155509.png "基于 Alpine 的 Python Demo 镜像大小：472 MB")

△ 基于 Alpine 的 Python Demo 镜像大小：472 MB; 相比之下，基于 slim 的只有 189 MB

在上面代码的这一步，就占用了太多空间：

> 🤔 **思考** :
> 
> 可能上面一段可以精简，但是要判断对于哪个 Python 项目，可以精简哪些包，实在是太难了。

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br><span>8</span><br><span>9</span><br><span>10</span><br><span>11</span><br><span>12</span><br><span>13</span><br><span>14</span><br><span>15</span><br><span>16</span><br><span>17</span><br><span>18</span><br><span>19</span><br><span>20</span><br><span>21</span><br><span>22</span><br><span>23</span><br><span>24</span><br><span>25</span><br><span>26</span><br><span>27</span><br><span>28</span><br><span>29</span><br><span>30</span><br><span>31</span><br><span>32</span><br><span>33</span><br><span>34</span><br></pre></td><td><pre><code>+ apk <span>add</span> --no-cache --virtual .build-deps build-base openssl-dev libffi-dev gcc musl-dev python3-dev<br>fetch https://dl-cdn.alpinelinux<span>.org</span>/alpine/v3<span>.17</span>/main/x86_64/APKINDEX.tar.gz<br>fetch https://dl-cdn.alpinelinux<span>.org</span>/alpine/v3<span>.17</span>/community/x86_64/APKINDEX.tar.gz<br>(<span>1</span>/<span>28</span>) Installing libgcc (<span>12.2</span><span>.1</span>_git20220924-<span>r4</span>)<br>(<span>2</span>/<span>28</span>) Installing libstdc++ (<span>12.2</span><span>.1</span>_git20220924-<span>r4</span>)<br>(<span>3</span>/<span>28</span>) Installing binutils (<span>2.39</span>-<span>r2</span>)<br>(<span>4</span>/<span>28</span>) Installing libmagic (<span>5.43</span>-<span>r0</span>)<br>(<span>5</span>/<span>28</span>) Installing file (<span>5.43</span>-<span>r0</span>)<br>(<span>6</span>/<span>28</span>) Installing libgomp (<span>12.2</span><span>.1</span>_git20220924-<span>r4</span>)<br>(<span>7</span>/<span>28</span>) Installing libatomic (<span>12.2</span><span>.1</span>_git20220924-<span>r4</span>)<br>(<span>8</span>/<span>28</span>) Installing gmp (<span>6.2</span><span>.1</span>-<span>r2</span>)<br>(<span>9</span>/<span>28</span>) Installing isl25 (<span>0.25</span>-<span>r0</span>)<br>(<span>10</span>/<span>28</span>) Installing mpfr4 (<span>4.1</span><span>.0</span>-<span>r0</span>)<br>(<span>11</span>/<span>28</span>) Installing mpc1 (<span>1.2</span><span>.1</span>-<span>r1</span>)<br>(<span>12</span>/<span>28</span>) Installing gcc (<span>12.2</span><span>.1</span>_git20220924-<span>r4</span>)<br>(<span>13</span>/<span>28</span>) Installing libstdc++-dev (<span>12.2</span><span>.1</span>_git20220924-<span>r4</span>)<br>(<span>14</span>/<span>28</span>) Installing musl-dev (<span>1.2</span><span>.3</span>-<span>r4</span>)<br>(<span>15</span>/<span>28</span>) Installing libc-dev (<span>0.7</span><span>.2</span>-<span>r3</span>)<br>(<span>16</span>/<span>28</span>) Installing g++ (<span>12.2</span><span>.1</span>_git20220924-<span>r4</span>)<br>(<span>17</span>/<span>28</span>) Installing make (<span>4.3</span>-<span>r1</span>)<br>(<span>18</span>/<span>28</span>) Installing fortify-headers (<span>1.1</span>-<span>r1</span>)<br>(<span>19</span>/<span>28</span>) Installing patch (<span>2.7</span><span>.6</span>-<span>r8</span>)<br>(<span>20</span>/<span>28</span>) Installing build-base (<span>0.5</span>-<span>r3</span>)<br>(<span>21</span>/<span>28</span>) Installing pkgconf (<span>1.9</span><span>.3</span>-<span>r0</span>)<br>(<span>22</span>/<span>28</span>) Installing openssl-dev (<span>3.0</span><span>.7</span>-<span>r0</span>)<br>(<span>23</span>/<span>28</span>) Installing linux-headers (<span>5.19</span><span>.5</span>-<span>r0</span>)<br>(<span>24</span>/<span>28</span>) Installing libffi-dev (<span>3.4</span><span>.4</span>-<span>r0</span>)<br>(<span>25</span>/<span>28</span>) Installing mpdecimal (<span>2.5</span><span>.1</span>-<span>r1</span>)<br>(<span>26</span>/<span>28</span>) Installing python3 (<span>3.10</span><span>.9</span>-<span>r1</span>)<br>(<span>27</span>/<span>28</span>) Installing python3-dev (<span>3.10</span><span>.9</span>-<span>r1</span>)<br>(<span>28</span>/<span>28</span>) Installing .build-deps (<span>20221214.074929</span>)<br>Executing busybox<span>-1.35</span><span>.0</span>-<span>r29</span>.trigger<br><span>OK:</span> <span>358</span> MiB <span>in</span> <span>65</span> packages<br>...<br></code><p><i></i>AVRASM</p></pre></td></tr></tbody></table>

## 建议使用官方的 python slim 镜像作为基础镜像[](https://ewhisper.cn/posts/25776/#%E5%BB%BA%E8%AE%AE%E4%BD%BF%E7%94%A8%E5%AE%98%E6%96%B9%E7%9A%84%20-python-slim-%20%E9%95%9C%E5%83%8F%E4%BD%9C%E4%B8%BA%E5%9F%BA%E7%A1%80%E9%95%9C%E5%83%8F)

继续上面，所以我是建议：使用官方的 python slim 镜像作为基础镜像

镜像库是这个：[https://hub.docker.com/\_/python](https://hub.docker.com/_/python)

并且使用 `python:<version>-slim` 作为基础镜像，能用 `python:<version>-slim-bullseye` 作为基础镜像更好（因为更新，相对就更安全一些）.

这个镜像不包含默认标签中的常用包，只包含运行 python 所需的最小包。这个镜像是基于 Debian 的。

使用官方 python slim 的理由还包括：

-   稳定性
-   安全升级更及时
-   依赖更新更及时
-   依赖更全
-   Python 版本升级更及时
-   镜像更小

> 📚️**Reference:**
> 
> [The best Docker base image for your Python application (Sep 2022) (pythonspeed.com)](https://pythonspeed.com/articles/base-image-python-docker-images/)

## 一般情况下，Python 镜像构建不需要使用 "多阶段构建"[](https://ewhisper.cn/posts/25776/#%E4%B8%80%E8%88%AC%E6%83%85%E5%86%B5%E4%B8%8B%EF%BC%8CPython-%20%E9%95%9C%E5%83%8F%E6%9E%84%E5%BB%BA%E4%B8%8D%E9%9C%80%E8%A6%81%E4%BD%BF%E7%94%A8%20-%20%E5%A4%9A%E9%98%B6%E6%AE%B5%E6%9E%84%E5%BB%BA)

一般情况下，Python 镜像构建不需要使用 "多阶段构建".

理由如下：

-   Python 没有像 Golang 一样，可以把所有依赖打成一个单一的二进制包
-   Python 也没有像 Java 一样，可以在 JDK 上构建，在 JRE 上运行
-   Python 复杂而散落的依赖关系，在 "多阶段构建" 时会增加复杂度
-   …

如果有一些特殊情况，可以尝试使用 "多阶段构建" 压缩镜像体积：

-   构建阶段需要安装编译器
-   Python 项目复杂，用到了其他语言代码（如 C/C++/Rust)

## pip 小技巧[](https://ewhisper.cn/posts/25776/#pip-%20%E5%B0%8F%E6%8A%80%E5%B7%A7)

使用 pip 安装依赖时，可以添加 `--no-cache-dir` 减少镜像体积：

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br></pre></td><td><pre><code><span># 安装 pip 依赖 </span><br><span>COPY</span><span> requirements.txt .</span><br><span>RUN</span><span> python -m pip install --no-cache-dir --upgrade -r requirements.txt</span><br></code><p><i></i>DOCKERFILE</p></pre></td></tr></tbody></table>

## Python Dockerfile 最佳实践样例[](https://ewhisper.cn/posts/25776/#Python-Dockerfile-%20%E6%9C%80%E4%BD%B3%E5%AE%9E%E8%B7%B5%E6%A0%B7%E4%BE%8B)

最后, 就是基于以上最佳实践的完整样例, 也可以在这里找到: [https://github.com/east4ming/fastapi-url-shortener/blob/main/Dockerfile.slim](https://github.com/east4ming/fastapi-url-shortener/blob/main/Dockerfile.slim)

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br><span>8</span><br><span>9</span><br><span>10</span><br><span>11</span><br><span>12</span><br><span>13</span><br><span>14</span><br><span>15</span><br><span>16</span><br><span>17</span><br><span>18</span><br><span>19</span><br><span>20</span><br><span>21</span><br><span>22</span><br><span>23</span><br><span>24</span><br><span>25</span><br></pre></td><td><pre><code><span>FROM</span> python:<span>3.10</span>-slim<p><span>LABEL</span><span> maintainer=<span>"cuikaidong@foxmail.com"</span></span></p><p><span>EXPOSE</span> <span>8000</span></p><p><span># Keeps Python from generating .pyc files in the container</span><br><span>ENV</span> PYTHONDONTWRITEBYTECODE=<span>1</span></p><p><span># Turns off buffering for easier container logging</span><br><span>ENV</span> PYTHONUNBUFFERED=<span>1</span></p><p><span># Install pip requirements</span><br><span>COPY</span><span> requirements.txt .</span><br><span>RUN</span><span> python -m pip install --no-cache-dir --upgrade -r requirements.txt</span></p><p><span>WORKDIR</span><span> /app</span><br><span>COPY</span><span> . /app</span></p><p><span># Creates a non-root user with an explicit UID and adds permission to access the /app folder</span><br><span>RUN</span><span> adduser -u 5678 --disabled-password --gecos <span>""</span> appuser &amp;&amp; <span>chown</span> -R appuser /app</span><br><span>USER</span> appuser</p><p><span>CMD</span><span> [<span>"uvicorn"</span>, <span>"shortener_app.main:app"</span>, <span>"--host"</span>, <span>"0.0.0.0"</span>]</span></p></code><p><i></i>DOCKERFILE</p></pre></td></tr></tbody></table>

## 总结[](https://ewhisper.cn/posts/25776/#%E6%80%BB%E7%BB%93)

制作 Python Docker 容器镜像的最佳实践。最佳实践的目的一方面是为了减小镜像体积，提升 DevOps 效率，另一方面是为了提高安全性.

最佳实践如下:

-   推荐 2 个 Python 的环境变量
    -   `ENV PYTHONDONTWRITEBYTECODE 1`
    -   `ENV PYTHONUNBUFFERED 1`
-   使用非 root 用户运行容器进程
-   使用 `.dockerignore` 排除无关文件
-   不建议使用 Alpine 作为 Python 的基础镜像
-   建议使用官方的 python slim 镜像作为基础镜像
-   一般情况下, Python 镜像构建不需要使用 "多阶段构建"
-   pip 小技巧: `--no-cache-dir`

希望对大家有所帮助.

最后也感叹一下, 在云原生时代, python 在分发这块, 特别是镜像构建这块, 确实体验、效率、镜像大小等方面差 golang 太多了。😭😭😭

## 📚️参考文档[](https://ewhisper.cn/posts/25776/#%F0%9F%93%9A%EF%B8%8F%E5%8F%82%E8%80%83%E6%96%87%E6%A1%A3)

-   [Using Alpine can make Python Docker builds 50× slower (pythonspeed.com)](https://pythonspeed.com/articles/alpine-docker-python/)
-   [The best Docker base image for your Python application (Sep 2022) (pythonspeed.com)](https://pythonspeed.com/articles/base-image-python-docker-images/)
-   [Multi-stage builds #2: Python specifics (pythonspeed.com)](https://pythonspeed.com/articles/multi-stage-docker-python/)
-   [制作容器镜像的最佳实践 - 东风微鸣技术博客 (ewhisper.cn)](https://ewhisper.cn/posts/8023/)