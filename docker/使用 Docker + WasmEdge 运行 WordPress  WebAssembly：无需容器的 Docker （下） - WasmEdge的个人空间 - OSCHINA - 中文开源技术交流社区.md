> 本文翻译自 [Wasm Labs @ VMware OCTO](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fwasmlabs.dev%2F) 的 blog： WebAssembly: Docker without container。这是 Wasm Labs 在 2022 年 12 月 15 日在冬季 Docker Community All Hands 7 的关于 [Docker+WebAssembly 的演讲](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fwww.bilibili.com%2Fvideo%2FBV1314y1w7vp%2F)的文字版。
> 
> 作者：Asen Alexandrov，Wasm Labs 工程师。文中的我们均指作者或 Wasm Labs。本篇文章将更具实践性，将以 PHP 为例带领大家实践 Docker + Wasm。

上篇文章我们了解了[服务端 Wasm 为什么有着重要的作用、什么是 WasmEdge 以及如何让解释型语言编写的程序在 Wasm 里运行](https://my.oschina.net/u/4532842/blog/5698793)，这篇文章，我们将通过动手示例了解在 Docker + Wasm 背景下的 Wasm container 有什么好处以及如何运行一个服务 WordPress 的 php.wasm 镜像。

## 动手示例

让我们开始吧！ 在动手示例中，我们将使用编译为 Wasm 的 PHP 解释器。 我们会：

-   构建一个 Wasm 容器。
-   比较 Wasm 和原生二进制文件。
-   比较传统容器和 Wasm 容器。
-   展示 Wasm 的可移植性

## 前期准备

如果想在本地重现这些示例，你需要使用以下部分或全部内容来准备你的环境：

-   WASI SDK - 从构建 C 代码构建 WebAssembly 应用程序
-   PHP - 为了比较而运行本机 PHP 二进制文件
-   WasmEdge Runtime - 运行 WebAssembly 应用程序
-   Docker Desktop + Wasm (本文写作时，作为稳定 beta 版在 Docker Desktop[4.15 版](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fdocs.docker.com%2Fdesktop%2Frelease-notes%2F%234150)可用) - 能够运行 Wasm 容器

我们还充分运用 [webassembly-language-runtimes](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fgithub.com%2Fvmware-labs%2Fwebassembly-language-runtimes) repo，它提供了将 PHP 解释器构建为 WebAssembly 应用程序的方法。 可以像这样查看 demo 分支：

```
git clone --depth=1 -b php-wasmedge-demo \
   https://github.com/vmware-labs/webassembly-language-runtimes.git wlr-demo
cd wlr-demo

```

## 构建一个 Wasm 容器

第一个示例，我们将展示如何构建基于 C 的应用程序，例如 PHP 解释器。

该构建使用 WASI-SDK 工具集。 它包括一个可以构建到 wasm32-wasi 目标的 clang 编译器，以及在 WASI 之上实现基本 POSIX 系统调用接口的 wasi-libc。 使用 WASI SDK，我们可以从 PHP 的代码库中构建一个用 C 编写的 Wasm 模块，。之后，我们需要一个非常简单的基于 scratch 的 Dockerfile 来制作一个可以使用 Docker+Wasm 运行的 OCI 镜像。

![](https://oscimg.oschina.net/oscnet/up-363675409cbe187ebe0df1e45313a1dea0c.png)

### 构建一个 WASM 二进制码

假设你现在位于 `wlr-demo` 文件夹，这是前期准备工作的一部分，可以运行以下命令来构建 Wasm 二进制文件。

```
export WASI_SDK_ROOT=/opt/wasi-sdk/
export WASMLABS_RUNTIME=wasmedge

./wl-make.sh php/php-7.4.32/ && tree build-output/php/php-7.4.32/bin/

... ( a few minutes and hundreds of build log lines)几分钟和数百行构建日志

build-output/php/php-7.4.32/bin/
├── php-cgi-wasmedge
└── php-wasmedge

```

PHP 是用 _autoconf_ 和 _make 构建的。_ 所以如果你看一眼脚本 `scripts/wl-build.sh` ，你会注意到我们设置了所有相关变量，如 `CC`、`LD`、 `CXX` 等，以使用来自 WASI\_SDK 的编译器。

```
export WASI_SYSROOT="${WASI_SDK_ROOT}/share/wasi-sysroot"
export CC=${WASI_SDK_ROOT}/bin/clang
export LD=${WASI_SDK_ROOT}/bin/wasm-ld
export CXX=${WASI_SDK_ROOT}/bin/clang++
export NM=${WASI_SDK_ROOT}/bin/llvm-nm
export AR=${WASI_SDK_ROOT}/bin/llvm-ar
export RANLIB=${WASI_SDK_ROOT}/bin/llvm-ranlib

```

然后，进一步深入查看 `php/php-7.4.32/wl-build.sh`，可以看到像通常一样，我们使用 autoconf 构建过程。

```
./configure --host=wasm32-wasi host_alias=wasm32-musl-wasi \
   --target=wasm32-wasi target_alias=wasm32-musl-wasi \
   ${PHP_CONFIGURE} || exit 1
...
make -j ${MAKE_TARGETS} || exit 1

```

WASI 是一项正在进行的工作，许多 POSIX 调用仍然不能在它之上实现。 因此，要构建 PHP，我们必须在原始代码库之上应用多个补丁。

我们在上面看到输出二进制文件会转到 `build-output/php/php-7.4.32`。 在下面的示例中，我们将使用专门为 WasmEdge 构建的 `php-wasmedge` 二进制文件，因为它提供服务端 socket 支持，**服务端 socket 支持还不是 WASI 的一部分**。

### 优化二进制码

Wasm 是一个虚拟指令集，因此任何运行时的默认行为都是即时解释这些指令。 当然，这在某些情况下可能会让速度变慢。 因此，为了通过 WasmEdge 获得两全其美的效果，你可以创建一个 AOT（提前编译）优化的二进制文件，它可以在当前机器上原生运行，但仍然可以在其他机器上进行解释。

要创建优化的二进制文件，请运行以下命令：

```
wasmedgec --enable-all --optimize 3 \
   build-output/php/php-7.4.32/bin/php-wasmedge \
   build-output/php/php-7.4.32/bin/php-wasmedge-aot

```

我们在下面的例子中使用这个 `build-output/php/php-7.4.32/bin/php-wasmedge-aot` 二进制码。要了解有关 WasmEdge AOT 优化二进制文件的更多信息，请[查看这里。](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fwasmlabs.dev%2Farticles%2Fdocker-without-containers%2Fbuild-output%2Fphp%2Fphp-7.4.32%2Fbin%2Fphp-wasmedge-aot)

### 构建 OCI 镜像

现在我们有了一个二进制文件，我们可以将它包装在一个 OCI 镜像中。 让我们看一下这个 `images/php/Dockerfile.cli。` 我们需要做的就是复制 Wasm 二进制文件并将其设置为 `ENTRYPOINT`。

```
FROM scratch
ARG PHP_TAG=php-7.4.32
ARG PHP_BINARY=php
COPY build-output/php/${PHP_TAG}/bin/${PHP_BINARY} /php.wasm

ENTRYPOINT [ "php.wasm" ]

```

我们还可以在镜像添加更多内容，当 Docker 运行它时，Wasm 二进制文件可以访问这些内容。 例如，在 `images/php/Dockerfile.server` 中，我们还添加了一些 docroot 内容，在容器启动时由 `php.wasm` 提供服务。

```
FROM scratch
ARG PHP_TAG=php-7.4.32
ARG PHP_BINARY=php
COPY build-output/php/${PHP_TAG}/bin/${PHP_BINARY} /php.wasm
COPY images/php/docroot /docroot

ENTRYPOINT [ "php.wasm" , "-S", "0.0.0.0:8080", "-t", "/docroot"]

```

基于以上文件，我们可以轻松地在本地构建我们的 `php-wasm` 镜像。

```
docker build --build-arg PHP_BINARY=php-wasmedge-aot -t ghcr.io/vmware-labs/php-wasm:7.4.32-cli-aot -f images/php/Dockerfile.cli .
docker build --build-arg PHP_BINARY=php-wasmedge-aot -t ghcr.io/vmware-labs/php-wasm:7.4.32-server-aot -f images/php/Dockerfile.server .

```

## 原生 vs Wasm

现在让我们将原生 PHP 二进制文件与 Wasm 二进制文件在本地和 Docker 容器中分别进行比较。 我们将使用相同的 `index.php` 文件并将运行它时得到的结果与以下内容进行比较：

-   `php`
-   `php-wasmedge-aot`
-   在传统容器中运行的 `php`
-   在 Wasm 容器中运行的 `php-wasmedge-aot`

![](https://oscimg.oschina.net/oscnet/up-2d9cde65bb1ec2d54ded3ea415fc078f3d2.jpg)

在下面所有的示例中，我们使用同样的 `images/php/docroot/index.php` 文件，让我们来看一下。简而言之，该脚本将：

-   使用 `phpversion` 和 `php_uname` 展示解释器版本和它运行的平台
-   打印脚本可以访问的所有环境变量的名称
-   打印一条包含当前时间和日期的问候消息
-   列出根文件夹的内容 `/`

```
<html>
<body>
<h1>Hello from PHP <?php echo phpversion() ?> running on "<?php echo php_uname()?>"</h1>

<h2>List env variable names</h2>
<?php
$php_env_vars_count = count(getenv());
echo "Running with $php_env_vars_count environment variables:\n";
foreach (getenv() as $key => $value) {
    echo  $key . " ";
}
echo "\n";
?>

<h2>Hello</h2>
<?php
$date = getdate();

$message = "Today, " . $date['weekday'] . ", " . $date['year'] . "-" . $date['mon'] . "-" . $date['mday'];
$message .= ", at " . $date['hours'] . ":" . $date['minutes'] . ":" . $date['seconds'];
$message .= " we greet you with this message!\n";
echo $message;
?>

<h2>Contents of '/'</h2>
<?php
foreach (array_diff(scandir('/'), array('.', '..')) as $key => $value) {
    echo  $value . " ";
}
echo "\n";
?>

</body>
</html>


```

### Native PHP 运行 index.js

我们使用本地 `php` 二进制码时，看到一个基于 Linux 的平台。

-   58 个环境变量的列表，脚本可以在需要时访问
-   `/` 中所有文件和文件夹的列表，如果需要，脚本可以再次访问这些文件和文件夹

```
$ php -f images/php/docroot/index.php

<html>
<body>
<h1>Hello from PHP 7.4.3 running on "Linux alexandrov-z01 5.15.79.1-microsoft-standard-WSL2 #1 SMP Wed Nov 23 01:01:46 UTC 2022 x86_64"</h1>

<h2>List env variable names</h2>
Running with 58 environment variables:
SHELL NVM_INC WSL2_GUI_APPS_ENABLED rvm_prefix WSL_DISTRO_NAME TMUX rvm_stored_umask TMUX_PLUGIN_MANAGER_PATH MY_RUBY_HOME NAME RUBY_VERSION PWD NIX_PROFILES LOGNAME rvm_version rvm_user_install_flag MOTD_SHOWN HOME LANG WSL_INTEROP LS_COLORS WASMTIME_HOME WAYLAND_DISPLAY NIX_SSL_CERT_FILE PROMPT_COMMAND NVM_DIR rvm_bin_path GEM_PATH GEM_HOME LESSCLOSE TERM CPLUS_INCLUDE_PATH LESSOPEN USER TMUX_PANE LIBRARY_PATH rvm_loaded_flag DISPLAY SHLVL NVM_CD_FLAGS LD_LIBRARY_PATH XDG_RUNTIME_DIR PS1 WSLENV XDG_DATA_DIRS PATH DBUS_SESSION_BUS_ADDRESS C_INCLUDE_PATH NVM_BIN HOSTTYPE WASMER_CACHE_DIR IRBRC PULSE_SERVER rvm_path WASMER_DIR OLDPWD BASH_FUNC_cr-open%% _

<h2>Hello</h2>
Today, Wednesday, 2022-12-14, at 12:0:36 we greet you with this message!

<h2>Contents of '/'</h2>
apps bin boot dev docroot etc home init lib lib32 lib64 libx32 lost+found media mnt nix opt path proc root run sbin snap srv sys tmp usr var wsl.localhost

</body>
</html>

```

### php-aot-wasm 运行 index.js

如果我们在 WasmEdge 使用 `php-aot-wasm` 我们看到

-   一个 wasi/wasm32 平台
-   没有环境变量，因为没有明确暴露给 Wasm 应用程序
-   Wasm 应用程序未获得对 `/` 的明确访问权限，因此尝试列出其内容失败并出现错误

自然地，为了让 `php-wasmedge-aot` 能够读取 `index.php` 文件，我们必须明确地向 WasmEdge 声明我们想要预先打开 `images/php/docroot` 以便在 Wasm 应用程序的上下文中作为 `/docroot` 进行访问。这显而易见展示了 Wasm 除了可移植性之外的最大优势之一。 我们得到了更佳的安全性，因为除非明确说明，否则无法访问任何内容。

```
$ wasmedge --dir /docroot:$(pwd)/images/php/docroot \
   build-output/php/php-7.4.32/bin/php-wasmedge-aot -f /docroot/index.php


<html>
<body>
<h1>Hello from PHP 7.4.32 running on "wasi (none) 0.0.0 0.0.0 wasm32"</h1>

<h2>List env variable names</h2>
Running with 0 environment variables:


<h2>Hello</h2>
Today, Wednesday, 2022-12-14, at 10:8:46 we greet you with this message!

<h2>Contents of '/'</h2>

Warning: scandir(/): failed to open dir: Capabilities insufficient in /docroot/index.php on line 27

Warning: scandir(): (errno 76): Capabilities insufficient in /docroot/index.php on line 27

Warning: array_diff(): Expected parameter 1 to be an array, bool given in /docroot/index.php on line 27

Warning: Invalid argument supplied for foreach() in /docroot/index.php on line 27


</body>
</html>

```

### 容器中的 PHP 运行 index.js

当我们从一个传统的容器中使用 `php` 时我们看到

-   基于 Linux 的平台
-   脚本有权访问的 14 个环境变量的列表
-   带有当前时间和日期的问候消息
-   包含根文件夹内容的列表 `/`

与在主机上使用 `php` 运行它相比，已经明显有区别，表现更佳。 由于 `/` 的环境变量和内容是 “虚拟的” 并且仅存在于容器内。

```
docker run --rm \
   -v $(pwd)/images/php/docroot:/docroot \
   php:7.4.32-cli \
   php -f /docroot/index.php


<html>
<body>
<h1>Hello from PHP 7.4.32 running on "Linux 227b2bc2f611 5.15.79.1-microsoft-standard-WSL2 #1 SMP Wed Nov 23 01:01:46 UTC 2022 x86_64"</h1>

<h2>List env variable names</h2>
Running with 14 environment variables:
HOSTNAME PHP_INI_DIR HOME PHP_LDFLAGS PHP_CFLAGS PHP_VERSION GPG_KEYS PHP_CPPFLAGS PHP_ASC_URL PHP_URL PATH PHPIZE_DEPS PWD PHP_SHA256

<h2>Hello</h2>
Today, Wednesday, 2022-12-14, at 10:15:35 we greet you with this message!

<h2>Contents of '/'</h2>
bin boot dev docroot etc home lib lib64 media mnt opt proc root run sbin srv sys tmp usr var

</body>
</html>

```

### php-aot-wasm 在一个容器中运行 index.js

如果我们在 WasmEdge 使用 `php-aot-wasm` 我们看到

-   一个 wasi/wasm32 平台
-   只有 2 个基础设施环境变量，使用在 containerd 中运行的 WasmEdge shim 预先设置
-   容器中 `/` 内所有文件和文件夹的列表，明确预打开以供 Wasm 应用程序访问（WasmEdge shim 中的逻辑的一部分）

注意：如果你仔细观察，会发现要从这个镜像运行一个容器，我们必须：

-   通过 `--runtime=io.containerd.wasmedge.v1` 将命令行参数直接传递给 php.wasm 明确声明运行时，而不包括二进制文件本身。 拉到上面可以看到我们可以使用传统的 PHP 容器明确编写完整的命令，包括 php 二进制文件（不是必需的）。

最后一点，即使使用 Docker，Wasm 也加强了运行 index.php 的安全性，因为暴露给它的要少得多。

```
docker run --rm \
   --runtime=io.containerd.wasmedge.v1 \
   -v $(pwd)/images/php/docroot:/docroot \
   ghcr.io/vmware-labs/php-wasm:7.4.32-cli-aot \
   -f /docroot/index.php


<html>
<body>
<h1>Hello from PHP 7.4.32 running on "wasi (none) 0.0.0 0.0.0 wasm32"</h1>

<h2>List env variable names</h2>
Running with 2 environment variables:
PATH HOSTNAME

<h2>Hello</h2>
Today, Wednesday, 2022-12-14, at 11:33:10 we greet you with this message!

<h2>Contents of '/'</h2>
docroot etc php.wasm

</body>
</html>
```

## 传统容器 vs Wasm 容器

我们构建并运行了一个 Wasm 二进制文件，并将其作为容器运行。 我们看到了 Wasm 和传统容器之间的输出差异以及 Wasm 带来的高级 “沙箱隔离”。我们可以轻松看到的两种容器之间的其他差异。

首先，我们将运行两个 daemon 容器，看看我们如何解释有关它们的一些统计信息。 然后我们将检查容器镜像的差异。 ![](https://oscimg.oschina.net/oscnet/up-7843e9711d57da7ba5529b57b66b142cf65.jpg)

### 容器数据

让我们运行两个 daemon 容器 - 一个是从传统的 `php` 镜像，另一个是从 `php-wasm` 镜像。

```
docker run --rm -d \
   -p 8083:8080 -v $(pwd)/images/php/docroot:/docroot \
   php:7.4.32-cli \
   -S 0.0.0.0:8080 -t /docroot
docker run --rm -d \
   --runtime=io.containerd.wasmedge.v1 \
   -p 8082:8080 -v $(pwd)/images/php/docroot:/docroot \
   ghcr.io/vmware-labs/php-wasm:7.4.32-cli-aot 
   -S 0.0.0.0:8080 -t /docroot

```

但是如果我们看 `docker stats`，我们只看到传统容器的数据。这之后可能会变化，因为 Docker+Wasm 现在是 beta 版特性。 所以，如果真的想看看发生了什么，可以改为监视对照组。 每个传统容器都有自己的控制组，如 `docker/ee44...`。另一方面，Wasm 容器作为 `podruntime/docker` 控制组的一部分包含在内，可以间接观察它们的 CPU 或内存消耗。

```
$ systemd-cgtop -kP --depth=10

Control Group           Tasks    %CPU     Memory
podruntime              145      0.1      636.3M
podruntime/docker       145      0.1      636.3M
docker                  2        0.0      39.7M
docker/ee444b...        1        0.0      6.7M 

```

### 镜像大小

首先，探索镜像，我们看到 Wasm 容器镜像比传统镜像小得多。 即使是 `alpine` 版本的 `php` 容器也比 Wasm 容器大。

```
$ docker images


REPOSITORY                     TAG                 IMAGE ID       CREATED          SIZE
php                            7.4.32-cli          680c4ba36f1b   2 hours ago      166MB
php                            7.4.32-cli-alpine   a785f7973660   2 minutes ago    30.1MB
ghcr.io/vmware-labs/php-wasm   7.4.32-cli-aot      63460740f6d5   44 minutes ago   5.35MB

```

这是意料之中的，因为对于 Wasm，我们只需要在容器内添加可执行二进制文件，而对于传统容器，我们仍然需要来自运行二进制文件的操作系统的一些基本库和文件。这种大小差异对于第一次拉取镜像的速度以及进行在本地存储库中占用的空间非常有帮助。

## Wasm 可移植性

Wasm 最大优势之一就是它的可移植性。 当人们想要一个可移植的应用程序时，Docker 已经提供了传统的容器作为一种选择。 然而，除了镜像特别大之外，传统容器还绑定到它们运行的平台架构。 作为程序员，相比许多人都经历过这种坎坷：针对不同的架构，必须构建支持的软件版本，并为每种架构打包对应镜像。

WebAssembly 带来了真正的可移植性。 构建一次二进制文件，就能在任何地方运行它。 作为这种可移植性的证明，我们准备了几个通过我们为 WebAssembly 构建的 PHP 解释器运行 WordPress 的示例。

当 PHP 作为独立的 Wasm 应用程序运行时，它会为 WordPress 提供服务。 它也可以在 Docker+Wasm 容器中运行。 此外，它还能在嵌入 Wasm 运行时的任何应用程序中运行。 在我们的示例中，这是 apache httpd，它可以通过 mod\_wasm 使用 Wasm 应用程序作为内容处理程序。 最后，PHP.wasm 也可以在浏览器中运行。

![](https://oscimg.oschina.net/oscnet/up-ef9463fa205a69118a69fe23c4b14c4b141.jpg)

### 通过 WasmEdge 服务 WordPress

我们为本次演示准备了一个紧凑的 WordPress+Sqlite 示例。 由于它是 `ghcr.io/vmware-labs/php-wasm:7.4.32-server-wordpress` 容器镜像的一部分，我们先将其下载到本地。

此命令将只创建一个临时容器（拉取镜像），将 WordPress 文件复制到 `/tmp/wp/docroot`，然后删除容器。

```
container_id=$(docker create ghcr.io/vmware-labs/php-wasm:7.4.32-server-wordpress) && \
   mkdir /tmp/wp && \
   docker cp $container_id:/docroot /tmp/wp/ && \
   docker rm $container_id

```

现在我们有了 WordPress，让我们添加服务器：

```
wasmedge --dir /docroot:/tmp/wp/docroot \
   build-output/php/php-7.4.32/bin/php-wasmedge-aot \
   -S 0.0.0.0:8085 -t /docroot

```

可以访问 [http://localhost:8085](https://www.oschina.net/action/GoToLink?url=http%3A%2F%2Flocalhost%3A8085%2F) ，使用由 PHP Wasm 解释器服务的 WordPress。

### 通过 Docker+Wasm 服务 WordPress

自然的，有了 Docker 会容易很多。

```
docker run --rm --runtime=io.containerd.wasmedge.v1 \
   -p 8086:8080 -v /tmp/wp/docroot/:/docroot/ \
   ghcr.io/vmware-labs/php-wasm:7.4.32-cli-aot 
   -S 0.0.0.0:8080 -t /docroot

```

可以访问 [http://localhost:8086](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fwasmlabs.dev%2Farticles%2Fdocker-without-containers%2F) 并使用由 PHP Wasm 解释器服务的 WordPress，这回是在 Docker 容器中运行。

### 通过 mod\_wasm in Apache HTTPD 服务 WordPress

Apache HTTPD 是使用最广泛的 HTTP 服务器之一。 现在有了 mod\_wasm，它还可以运行 WebAssembly 应用程序。 为了避免在本地安装和配置它，我们准备了一个容器，其中包含 Apache HTTPD、mod\_wasm 和 WordPress。

```
docker run -p 8087:8080 projects.registry.vmware.com/wasmlabs/containers/php-mod-wasm:wordpress
```

可以访问 [http://localhost:8087](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fwasmlabs.dev%2Farticles%2Fdocker-without-containers%2F) 并使用由 PHP Wasm 解释器服务的 WordPress，它由 Apache HTTPD 中的 mod\_wasm 加载。

### 直接在浏览器中服务 WordPress

访问 [https://wordpress.wasmlabs.dev](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fwordpress.wasmlabs.dev%2F) 获得示例。 你将看到一个框架，其中 PHP Wasm 解释器会现场渲染 WordPress。

## 结论

感谢阅读本文。 需要消化的内容很多，但我们希望本文有助于理解 WebAssembly 的能力以及它如何与你现有的代码库和工具（包括 Docker）结合运行。 期待看到你使用 Wasm 编程！

如果你觉得 WasmEdge 不错，不要忘了给我们点个赞！

[https://github.com/WasmEdge/WasmEdge](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fgithub.com%2FWasmEdge%2FWasmEdge)