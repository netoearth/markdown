 2022.7.17  [Linux](https://waynerv.com/categories/linux/)  2820  6 分钟  28

## 目录

1.  [理论依据](https://waynerv.com/posts/how-to-reduce-docker-image-size/#%E7%90%86%E8%AE%BA%E4%BE%9D%E6%8D%AE)
2.  [分析工具](https://waynerv.com/posts/how-to-reduce-docker-image-size/#%E5%88%86%E6%9E%90%E5%B7%A5%E5%85%B7)
    1.  [docker history](https://waynerv.com/posts/how-to-reduce-docker-image-size/#docker-history)
    2.  [dive](https://waynerv.com/posts/how-to-reduce-docker-image-size/#dive)
3.  [优化技巧](https://waynerv.com/posts/how-to-reduce-docker-image-size/#%E4%BC%98%E5%8C%96%E6%8A%80%E5%B7%A7)
    1.  [分阶段构建与从零构建](https://waynerv.com/posts/how-to-reduce-docker-image-size/#%E5%88%86%E9%98%B6%E6%AE%B5%E6%9E%84%E5%BB%BA%E4%B8%8E%E4%BB%8E%E9%9B%B6%E6%9E%84%E5%BB%BA)
    2.  [避免产生无用的文档或缓存](https://waynerv.com/posts/how-to-reduce-docker-image-size/#%E9%81%BF%E5%85%8D%E4%BA%A7%E7%94%9F%E6%97%A0%E7%94%A8%E7%9A%84%E6%96%87%E6%A1%A3%E6%88%96%E7%BC%93%E5%AD%98)
    3.  [及时清理不需要的文件](https://waynerv.com/posts/how-to-reduce-docker-image-size/#%E5%8F%8A%E6%97%B6%E6%B8%85%E7%90%86%E4%B8%8D%E9%9C%80%E8%A6%81%E7%9A%84%E6%96%87%E4%BB%B6)
    4.  [合并多个镜像层](https://waynerv.com/posts/how-to-reduce-docker-image-size/#%E5%90%88%E5%B9%B6%E5%A4%9A%E4%B8%AA%E9%95%9C%E5%83%8F%E5%B1%82)
    5.  [复制文件的同时修改元信息](https://waynerv.com/posts/how-to-reduce-docker-image-size/#%E5%A4%8D%E5%88%B6%E6%96%87%E4%BB%B6%E7%9A%84%E5%90%8C%E6%97%B6%E4%BF%AE%E6%94%B9%E5%85%83%E4%BF%A1%E6%81%AF)
4.  [参考链接](https://waynerv.com/posts/how-to-reduce-docker-image-size/#%E5%8F%82%E8%80%83%E9%93%BE%E6%8E%A5)

最近在接手一个新项目后，将原本 1.6GB 的镜像精简到了 600 多MB，直接进入了贤者时间，特地记录下优化过程中总结的一些经验。

## [](https://waynerv.com/posts/how-to-reduce-docker-image-size/#%E7%90%86%E8%AE%BA%E4%BE%9D%E6%8D%AE)[理论依据](https://waynerv.com/posts/how-to-reduce-docker-image-size/#contents:%E7%90%86%E8%AE%BA%E4%BE%9D%E6%8D%AE)

镜像的本质是镜像层和运行配置文件组成的压缩包，构建镜像是通过运行 Dockerfile 中的 `RUN` 、`COPY` 和 `ADD` 等指令生成镜像层和配置文件的过程。

在我去年写的博客 [容器技术原理(一)：从根本上认识容器镜像 | Shall We Code?](https://waynerv.com/posts/container-fundamentals-learn-container-with-oci-spec/#%E9%95%9C%E5%83%8F%E8%A7%84%E8%8C%83) 中，详细解释了镜像组成、联合文件系统及其工作方式等理论，本文不再赘述，只从中提取和镜像体积有关的关键点：

-   `RUN`、`COPY` 和 `ADD` 指令会在已有镜像层的基础上创建一个新的镜像层，执行指令产生的所有文件系统变更会在指令结束后作为一个镜像层整体提交。
-   镜像层具有 `copy-on-write` 的特性，如果去更新其他镜像层中已存在的文件，会先将其复制到新的镜像层中再修改，造成双倍的文件空间占用。
-   如果去删除其他镜像层的一个文件，只会在当前镜像层生成一个该文件的删除标记，并不会减少整个镜像的实际体积。

上述理论可以通过如下 Dockerfile 来验证：

<table><tbody><tr><td><pre tabindex="0"><code><span>1
</span><span>2
</span><span>3
</span><span>4
</span><span>5
</span></code></pre></td><td><pre tabindex="0"><code data-lang="docker"><span><span><span>FROM</span><span> alpine:latest</span><span>
</span></span></span><span><span><span></span><span>COPY</span> resource.tar /<span>
</span></span></span><span><span><span></span><span>RUN</span> touch /resource.tar<span>
</span></span></span><span><span><span></span><span>RUN</span> rm -f /resource.tar<span>
</span></span></span><span><span><span></span><span>ENTRYPOINT</span> <span>[</span><span>"/bin/ash"</span><span>]</span><span>
</span></span></span></code></pre></td></tr></tbody></table>

我们在 Dockerfile 中简单地添加、修改和删除某个资源文件，然后构建镜像查看其镜像层信息：

<table><tbody><tr><td><pre tabindex="0"><code><span>1
</span><span>2
</span><span>3
</span><span>4
</span><span>5
</span><span>6
</span><span>7
</span><span>8
</span><span>9
</span></code></pre></td><td><pre tabindex="0"><code data-lang="bash"><span><span>$ docker build -t test-image -f Dockerfile .
</span></span><span><span>$ docker <span>history</span> test-image:latest
</span></span><span><span>IMAGE          CREATED              CREATED BY                                      SIZE      COMMENT
</span></span><span><span>95f1695b2904   About a minute ago   /bin/sh -c <span>#(nop)  ENTRYPOINT ["/bin/ash"]      0B</span>
</span></span><span><span>1780448c656f   About a minute ago   /bin/sh -c rm -f /resource.tar                  0B
</span></span><span><span>a85d29bf7738   About a minute ago   /bin/sh -c touch /resource.tar                  135MB
</span></span><span><span>6dac335fa653   <span>4</span> minutes ago        /bin/sh -c <span>#(nop) COPY file:66065d6e23e0bc52…   135MB</span>
</span></span><span><span>e66264b98777   <span>7</span> weeks ago          /bin/sh -c <span>#(nop)  CMD ["/bin/sh"]              0B</span>
</span></span><span><span>&lt;missing&gt;      <span>7</span> weeks ago          /bin/sh -c <span>#(nop) ADD file:8e81116368669ed3d…   5.53MB</span>
</span></span></code></pre></td></tr></tbody></table>

在 `docker history` 的输出结果中可以看到：

-   `RUN touch /resource.tar` 指令只是修改了文件的元信息，但依然将整个文件拷贝到了新的镜像层中。
-   `RUN rm -f /resource.tar` 指令虽然删除了文件，并且该文件在运行容器时不可见，但依然在前两个镜像层中以及最终的镜像中存在。

## [](https://waynerv.com/posts/how-to-reduce-docker-image-size/#%E5%88%86%E6%9E%90%E5%B7%A5%E5%85%B7)[分析工具](https://waynerv.com/posts/how-to-reduce-docker-image-size/#contents:%E5%88%86%E6%9E%90%E5%B7%A5%E5%85%B7)

给代码做性能调优时，首先要借助 Profiling 工具找到代码的性能瓶颈，对于优化镜像体积也是如此。下面介绍两个可以分析镜像体积的工具：

### [](https://waynerv.com/posts/how-to-reduce-docker-image-size/#docker-history)[docker history](https://waynerv.com/posts/how-to-reduce-docker-image-size/#contents:docker-history)

docker 自带的 `docker history` 命令，该命令可以展示所有镜像层的创建时间、指令以及体积等较为基础的信息，但对于复杂的镜像则有些乏力。使用方式见上方的示例。

### [](https://waynerv.com/posts/how-to-reduce-docker-image-size/#dive)[dive](https://waynerv.com/posts/how-to-reduce-docker-image-size/#contents:dive)

第三方的 `dive` 工具，该工具可以分析镜像层组成，并列出每个镜像层所包含的文件列表，可以很方便地定位到影响镜像体积的构建指令以及具体文件。

以 `golang:1.16` 镜像为例，首先[安装 dive](https://github.com/wagoodman/dive#installation)，然后执行 `dive golang:1.16`，输出如下：

![dive image](https://waynerv.com/posts/how-to-reduce-docker-image-size/docker-image-dive.png)

如上图所示，在左侧选中镜像层后，在右侧的文件树视图中可以清晰地看到该层的具体文件，并能够筛选相比上一层新增、更新或删除的文件。在选中的镜像层中，由于执行了 `apt-get` 安装编译依赖，因此在 `/usr/lib` 目录下新增了 150MB 依赖库文件。

## [](https://waynerv.com/posts/how-to-reduce-docker-image-size/#%E4%BC%98%E5%8C%96%E6%8A%80%E5%B7%A7)[优化技巧](https://waynerv.com/posts/how-to-reduce-docker-image-size/#contents:%E4%BC%98%E5%8C%96%E6%8A%80%E5%B7%A7)

下面介绍一些优化效果比较显著的优化技巧。

### [](https://waynerv.com/posts/how-to-reduce-docker-image-size/#%E5%88%86%E9%98%B6%E6%AE%B5%E6%9E%84%E5%BB%BA%E4%B8%8E%E4%BB%8E%E9%9B%B6%E6%9E%84%E5%BB%BA)[分阶段构建与从零构建](https://waynerv.com/posts/how-to-reduce-docker-image-size/#contents:%E5%88%86%E9%98%B6%E6%AE%B5%E6%9E%84%E5%BB%BA%E4%B8%8E%E4%BB%8E%E9%9B%B6%E6%9E%84%E5%BB%BA)

**分阶段构建**（multi-stage builds）和**从零构建**（build from scratch）是优化镜像体积的基本手段和必备技巧。

该方法将镜像构建过程区分为构建和运行环境，在构建环境安装编译器等依赖并编译所需的二进制包，然后将其复制到仅包含必要运行依赖的运行环境中。 该技巧对 golang 这种能够编译静态二进制文件的语言效果尤为明显，我们可以将编译产生的二进制文件放到 `scratch` 镜像中运行（`scratch` 是一个特殊的空镜像）：

<table><tbody><tr><td><pre tabindex="0"><code><span>1
</span><span>2
</span><span>3
</span><span>4
</span><span>5
</span><span>6
</span><span>7
</span><span>8
</span></code></pre></td><td><pre tabindex="0"><code data-lang="docker"><span><span><span>FROM</span><span> golang</span><span>
</span></span></span><span><span><span></span><span>COPY</span> hell0.go .<span>
</span></span></span><span><span><span></span><span>ENV</span> <span>CGO_ENABLED</span><span>=</span><span>0</span>
</span></span><span><span><span>RUN</span> go build hello.go<span>
</span></span></span><span><span><span>
</span></span></span><span><span><span></span><span>FROM</span><span> scratch</span><span>
</span></span></span><span><span><span></span><span>COPY</span> --from<span>=</span><span>0</span> /go/hello .<span>
</span></span></span><span><span><span></span><span>CMD</span> <span>[</span><span>"./hello"</span><span>]</span><span>
</span></span></span></code></pre></td></tr></tbody></table>

如果直接使用 golang 镜像作为运行环境，其镜像体积通常接近 1 个 G，其中大部分文件都不是在运行容器时所必要的。 将编译结果拷贝到运行环境后，体积只有几十 kb~mb 不等，如果需要在运行容器中保留基本的系统工具，可以考虑使用 alpine 镜像作为运行环境。

### [](https://waynerv.com/posts/how-to-reduce-docker-image-size/#%E9%81%BF%E5%85%8D%E4%BA%A7%E7%94%9F%E6%97%A0%E7%94%A8%E7%9A%84%E6%96%87%E6%A1%A3%E6%88%96%E7%BC%93%E5%AD%98)[避免产生无用的文档或缓存](https://waynerv.com/posts/how-to-reduce-docker-image-size/#contents:%E9%81%BF%E5%85%8D%E4%BA%A7%E7%94%9F%E6%97%A0%E7%94%A8%E7%9A%84%E6%96%87%E6%A1%A3%E6%88%96%E7%BC%93%E5%AD%98)

docker 镜像不应该包含文档、缓存等对运行容器没有作用的内容。

1.  避免在本地保留安装缓存。
    
    大部分包管理器会在安装时缓存下载的资源以备之后使用，以 pip 为例，会将下载的响应和构建的中间文件保存在 `~/.cache/pip` 目录，应使用 `--no-cache-dir` 选项禁用默认的缓存行为。
    
2.  避免安装文档。
    
    部分包管理器提供了选项可以不安装附带的文档，如 dnf 可使用 `--nodocs` 选项。
    
3.  避免缓存包索引。
    
    部分包管理器在执行安装之前，会尝试查询所有已启用仓库的包列表、版本等元信息缓存在本地作为索引。个别仓库的索引缓存可达到 150 M 以上。 我们应该仅在安装包时查询索引，并在安装完成后清理，不应该在单独的指令中执行 `yum makecache` 这类缓存索引的命令。
    

### [](https://waynerv.com/posts/how-to-reduce-docker-image-size/#%E5%8F%8A%E6%97%B6%E6%B8%85%E7%90%86%E4%B8%8D%E9%9C%80%E8%A6%81%E7%9A%84%E6%96%87%E4%BB%B6)[及时清理不需要的文件](https://waynerv.com/posts/how-to-reduce-docker-image-size/#contents:%E5%8F%8A%E6%97%B6%E6%B8%85%E7%90%86%E4%B8%8D%E9%9C%80%E8%A6%81%E7%9A%84%E6%96%87%E4%BB%B6)

运行容器时不需要的文件，一定要在创建的同一层清理，否则依然会保留在最终的镜像中。

通过包管理安装包，通常会产生大量的缓存文件，一定要在同一 `RUN` 指令的结尾处立刻清理。在安装依赖数量较多时，可以节省大量的缓存空间。

以 `dnf` 为例：

<table><tbody><tr><td><pre tabindex="0"><code><span>1
</span><span>2
</span><span>3
</span></code></pre></td><td><pre tabindex="0"><code data-lang="docker"><span><span><span>RUN</span> dnf install -y --nodocs &lt;PACKAGES&gt; <span>\
</span></span></span><span><span><span></span>  <span>&amp;&amp;</span> dnf clean all <span>\
</span></span></span><span><span><span></span>  <span>&amp;&amp;</span> rm -rf /var/cache/dnf<span>
</span></span></span></code></pre></td></tr></tbody></table>

以 `apt` 为例：

<table><tbody><tr><td><pre tabindex="0"><code><span>1
</span><span>2
</span><span>3
</span><span>4
</span></code></pre></td><td><pre tabindex="0"><code data-lang="docker"><span><span><span>RUN</span> apt-get update <span>\
</span></span></span><span><span><span></span>  <span>&amp;&amp;</span> apt-get install -y &lt;PACKAGES&gt; <span>\
</span></span></span><span><span><span></span>  <span>&amp;&amp;</span> rm -rf /var/lib/apt/lists/*<span>
</span></span></span><span><span><span></span><span># 官方的 ubuntu/debian 镜像 apt-get 会在安装后自动执行 clean 命令</span><span>
</span></span></span></code></pre></td></tr></tbody></table>

### [](https://waynerv.com/posts/how-to-reduce-docker-image-size/#%E5%90%88%E5%B9%B6%E5%A4%9A%E4%B8%AA%E9%95%9C%E5%83%8F%E5%B1%82)[合并多个镜像层](https://waynerv.com/posts/how-to-reduce-docker-image-size/#contents:%E5%90%88%E5%B9%B6%E5%A4%9A%E4%B8%AA%E9%95%9C%E5%83%8F%E5%B1%82)

上文解释过，应该避免在不同镜像层中更新文件而造成额外的体积占用。当构建的层数很多且执行指令较复杂时，很难避免在不同的镜像层中更新文件，可通过以下手段精简这部分额外体积：

1.  在最终生成镜像时将所有镜像层合并成一层，在 `docker build` 命令中使用 `—squash` 即可实现（需要[开启 docker daemon 的实验性功能](https://docs.docker.com/engine/reference/commandline/dockerd/#description)）。以本文开头的 Dockerfile 为例：
    
    <table><tbody><tr><td><pre tabindex="0"><code><span> 1
    </span><span> 2
    </span><span> 3
    </span><span> 4
    </span><span> 5
    </span><span> 6
    </span><span> 7
    </span><span> 8
    </span><span> 9
    </span><span>10
    </span></code></pre></td><td><pre tabindex="0"><code data-lang="docker"><span><span>$ docker build -t squash-image --squash -f Dockerfile . <span>
    </span></span></span><span><span><span></span>$ docker <span>history</span> squash-image<span>
    </span></span></span><span><span><span></span>IMAGE          CREATED        CREATED BY                                      SIZE      COMMENT<span>
    </span></span></span><span><span><span></span>55ded8881d63   <span>9</span> hours ago                                                    0B        merge sha256:95f1695b29044522250de1b0c1904aaf8670b991ec1064d086c0c15865051d5d to sha256:e66264b98777e12192600bf9b4d663655c98a090072e1bab49e233d7531d1294<span>
    </span></span></span><span><span><span></span>&lt;missing&gt;      <span>11</span> hours ago   /bin/sh -c <span>#(nop)  ENTRYPOINT ["/bin/ash"]      0B</span><span>
    </span></span></span><span><span><span></span>&lt;missing&gt;      <span>11</span> hours ago   /bin/sh -c rm -f /resource.tar                  0B<span>
    </span></span></span><span><span><span></span>&lt;missing&gt;      <span>11</span> hours ago   /bin/sh -c touch /resource.tar                  0B<span>
    </span></span></span><span><span><span></span>&lt;missing&gt;      <span>11</span> hours ago   /bin/sh -c <span>#(nop) COPY file:66065d6e23e0bc52…   0B</span><span>
    </span></span></span><span><span><span></span>&lt;missing&gt;      <span>7</span> weeks ago    /bin/sh -c <span>#(nop)  CMD ["/bin/sh"]              0B</span><span>
    </span></span></span><span><span><span></span>&lt;missing&gt;      <span>7</span> weeks ago    /bin/sh -c <span>#(nop) ADD file:8e81116368669ed3d…   5.53MB</span><span>
    </span></span></span></code></pre></td></tr></tbody></table>
    
    最终生成的镜像只有一个镜像层，包含最后实际存在的文件系统，在合并所有镜像层的过程中，相当于禁用了 `copy-on-write` 特性。
    
    这种做法的坏处在于，镜像在保存和分发时是可以复用镜像层的，推送镜像时会跳过镜像仓库已存在的镜像层，拉取镜像时会跳过本地已拉取过的镜像层，而合并成一层后则失去了这种优势。
    
    对于可能和其他共用镜像层的场景，可以采取下面一种方式。
    
2.  分阶段构建，将部分中间镜像层压缩成一层作为基础镜像。 在开发团队内部，我们往往会在官方镜像的基础上添加或更新部分依赖，然后作为团队内部统一使用的基础镜像，这种复用方式可以大大减少实际占用的镜像体积。 更进一步，我们可以将这类基础镜像压缩成一层。下面以 golang 官方镜像为例：
    
    <table><tbody><tr><td><pre tabindex="0"><code><span>1
    </span><span>2
    </span><span>3
    </span><span>4
    </span><span>5
    </span></code></pre></td><td><pre tabindex="0"><code data-lang="docker"><span><span><span>FROM</span><span> golang:1.16 as base</span><span>
    </span></span></span><span><span><span>
    </span></span></span><span><span><span></span><span>FROM</span><span> scratch</span><span>
    </span></span></span><span><span><span></span><span>COPY</span> --from<span>=</span>base / /<span>
    </span></span></span><span><span><span></span><span>ENTRYPOINT</span> <span>[</span><span>"/bin/bash"</span><span>]</span><span>
    </span></span></span></code></pre></td></tr></tbody></table>
    
    压缩成一层后，`golang:1.16` 的镜像体积从 919MB 变成 913MB，官方镜像已经做了很多优化所以节省空间十分有限，但对于开发团队内部制作的基础镜像，这种优化往往会带来意外惊喜。
    

### [](https://waynerv.com/posts/how-to-reduce-docker-image-size/#%E5%A4%8D%E5%88%B6%E6%96%87%E4%BB%B6%E7%9A%84%E5%90%8C%E6%97%B6%E4%BF%AE%E6%94%B9%E5%85%83%E4%BF%A1%E6%81%AF)[复制文件的同时修改元信息](https://waynerv.com/posts/how-to-reduce-docker-image-size/#contents:%E5%A4%8D%E5%88%B6%E6%96%87%E4%BB%B6%E7%9A%84%E5%90%8C%E6%97%B6%E4%BF%AE%E6%94%B9%E5%85%83%E4%BF%A1%E6%81%AF)

首先将可执行文件添加到镜像内，然后再修改文件的执行权限和所属用户，这类 COPY-RUN 指令在 Dockerfile 中十分常见：

<table><tbody><tr><td><pre tabindex="0"><code><span>1
</span><span>2
</span></code></pre></td><td><pre tabindex="0"><code data-lang="docker"><span><span><span>COPY</span> output/hello /usr/bin/hello<span>
</span></span></span><span><span><span></span><span>RUN</span> chmod +x /usr/bin/hello <span>&amp;&amp;</span> chown normal:normal /usr/bin/hello<span>
</span></span></span></code></pre></td></tr></tbody></table>

但由于修改文件元信息也会将文件复制到新的镜像层，以上指令会产生两份相同的文件。在文件体积较大时，会显著增加整个镜像的体积。 事实上，我们可以在复制文件的同时完成对文件元信息的修改，`COPY` 和 `ADD` 指令都提供了修改元信息的 `--chmod` 和 `--chown` 选项：

<table><tbody><tr><td><pre tabindex="0"><code><span>1
</span></code></pre></td><td><pre tabindex="0"><code data-lang="docker"><span><span><span>COPY</span> --chmod<span>=</span><span>755</span> --chown<span>=</span>normal:normal output/hello /usr/bin/hello<span>
</span></span></span></code></pre></td></tr></tbody></table>

`--chmod` 特性目前还未添加到官方文档，使用前需要[开启 docker 的 buildkit 特性](https://docs.docker.com/develop/develop-images/build_enhancements/)（在 `docker build` 命令前添加 `DOCKER_BUILDKIT=1` 即可），目前只支持 `--chmod=755` 和 `--chmod=0755` 这种设置方法，不支持 `--chmod=+x`。

## [](https://waynerv.com/posts/how-to-reduce-docker-image-size/#%E5%8F%82%E8%80%83%E9%93%BE%E6%8E%A5)[参考链接](https://waynerv.com/posts/how-to-reduce-docker-image-size/#contents:%E5%8F%82%E8%80%83%E9%93%BE%E6%8E%A5)

-   [Best practices for writing Dockerfiles | Docker Documentation](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)
-   [How to build tiny container images | Enable Sysadmin](https://www.redhat.com/sysadmin/tiny-containers)
-   [COPY --chmod reduced the size of my container image by 35%](https://blog.vamc19.dev/posts/dockerfile-copy-chmod/)
-   [Docker Images : Part I - Reducing Image Size](https://www.ardanlabs.com/blog/2020/02/docker-images-part1-reducing-image-size.html)

updatedupdated2022-07-172022-07-17

[Docker](https://waynerv.com/tags/docker/)