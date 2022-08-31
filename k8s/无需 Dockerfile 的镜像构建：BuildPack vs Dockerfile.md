过去的工作中，我们使用微服务、容器化以及服务编排构建了技术平台。为了提升开发团队的研发效率，我们同时还提供了 [[CICD]] 平台，用来将代码快速的部署到 Openshift（企业级的 Kubernetes） 集群。

部署的第一步就是应用程序的容器化，持续集成的交付物从以往的 jar 包、webpack 等变成了容器镜像。容器化将软件代码和所需的所有组件（库、框架、运行环境）打包到一起，进而可以在任何环境任何基础架构上一致地运行，并与其他应用“隔离”。

我们的代码需要从源码到编译到最终可运行的镜像，甚至部署，这一切在 [[CICD]] 的流水线中完成。最初，我们在每个代码仓库中都加入了三个文件，也通过项目生成器（类似 Spring Initializer）在新项目中注入：

-   Jenkinsfile.groovy：用来定义 Jenkins 的 Pipeline，针对不同的语言还会有多种版本
-   Manifest YAML：用于定义 Kubernetes 资源，也就是工作负载及其运行的相关描述
-   Dockerfile：用于构建对象

这个三个文件也需要在工作中不断的演进，起初项目较少（十几个）的时候我们基础团队还可以去各个代码仓库去维护升级。随着项目爆发式的增长，维护的成本越来越高。我们对 [[CICD]] 平台进行了迭代，将“Jenkinsfile.groovy”和 “manifest YAML”从项目中移出，变更较少的 Dockerfile 就保留了下来。

随着平台的演进，我们需要考虑将这唯一的“钉子户” Dockerfile 与代码解耦，必要的时候也需要对 Dockerfile 进行升级。因此调研了一下 buildpacks，就有了今天的这篇文章。

## 什么是 Dockerfile

Docker 通过读取 Dockerfile 中的说明自动构建镜像。Dockerfile 是一个文本文件，包含了由 Docker 可以执行用于构建镜像的指令。我们拿之前用于[测试 Tekton 的 Java 项目](https://github.com/addozhang/tekton-test)的 Dockerfile 为例：

```
FROM openjdk:8-jdk-alpine

RUN mkdir /app
WORKDIR /app
COPY target/*.jar /app/app.jar
ENTRYPOINT ["sh", "-c", "java -Xmx128m -Xms64m -jar app.jar"]
```

### 镜像分层

你可能会听过 Docker 镜像包含了多个层。每个层与 Dockerfile 中的每个命令对应，比如 `RUN`、`COPY`、`ADD`。某些特定的指令会创建一个新的层，在镜像构建过程中，假如某些层没有发生变化，就会从缓存中获取。

在下面的 Buildpack 中也同样通过镜像分层和 cache 来加速镜像的构建。

## 什么是 Buildpack

[BuildPack](https://buildpacks.io/) 是一个程序，它能将源代码转换成容器镜像的并可以在任意云环境中运行。通常 buildpack 封装了单一语言的生态工具链。适用于 Java、Ruby、Go、NodeJs、Python 等。

![buildpacks.io](https://atbug.oss-cn-hangzhou.aliyuncs.com/2021/10/29/cleanshot-20211028-at-0830172x.png)

### Builder 是什么？

一些 buildpacks 按顺序组合之后就是 **builder**，除了 buildpacks， builder 中还加入了 [生命周期](https://buildpacks.io/docs/concepts/components/lifecycle/) 和 stack 容器镜像。

![](https://atbug.oss-cn-hangzhou.aliyuncs.com/2021/10/29/cleanshot-20211028-at-1659142x.png)

stack 容器镜像由两个镜像组成：用于运行 buildpack 的镜像 build image，以及构建应用镜像的基础镜像 run image。如上图，就是 builder 中的运行环境。

### Buildpack 的工作方式

![how buildpack works](https://atbug.oss-cn-hangzhou.aliyuncs.com/2021/10/29/cleanshot-20211028-at-0832072x.png)

每个 buildpack 运行时都包含了两个阶段：

![phases](https://atbug.oss-cn-hangzhou.aliyuncs.com/2021/10/29/cleanshot-20211028-at-0906532x.png)

#### 1\. 检测阶段

通过检查源代码中的某些特定文件/数据，来判断当前 buildpack 是否适用。如果适用，就会进入构建阶段；否则就会退出。比如：

-   Java maven 的 buildpack 会检查源码中是否有 `pom.xml`
-   Python 的 buildpack 会检查源码中是否有 `requirements.txt` 或者 `setup.py` 文件
-   Node buildpack 会查找 `package-lock.json` 文件。

#### 2\. 构建阶段

在构建阶段会进行如下操作：

1.  设置构建环境和运行时环境
2.  下载依赖并编译源码（假如需要的话）
3.  设置正确的 entrypoint 和启动脚本。

比如：

-   Java maven buildpack 在检查到有 `pom.xml` 文件之后，会执行 `mvn clean install -DskipTests`
-   Python buildpack 检查到有 `requrements.txt` 之后，会执行 `pip install -r requrements.txt`
-   Node build pack 检查到有 `package-lock.json` 后执行 `npm install`

## BuildPack 上手

那到底如何在没有 Dockerfile 的情况下使用 builderpack 构建镜像的。看了上面这些，大家基本上也都能了解到这个核心就在 buildpack 的编写和使用的。

其实现在有很多开源的 buildpack 可以用，没有特定定制的情况下无需自己手动编写。比如下面的几个大厂开源并维护的 Buildpacks：

-   [Heroku Buildpacks](https://github.com/heroku/)
-   [Google Buildpacks](https://github.com/GoogleCloudPlatform/buildpacks)
-   [Paketo](https://github.com/paketo-buildpacks)

但是正式详细介绍开源的 buildpacks 之前，我们还是通过自己创建 buildpack 的方式来深入了解 Buildpacks 的工作方式。测试项目呢，我们还是用[测试 Tekton 的 Java 项目](https://github.com/addozhang/tekton-test)。

下面所有的内容都提交到了 [Github](https://github.com/addozhang/buildpacks-sample) 上，可以访问：https://github.com/addozhang/buildpacks-sample 获取相关代码。

最终的目录`buildpacks-sample` 结构如下：

```
├── builders
│   └── builder.toml
├── buildpacks
│   └── buildpack-maven
│       ├── bin
│       │   ├── build
│       │   └── detect
│       └── buildpack.toml
└── stacks
    ├── build
    │   └── Dockerfile
    ├── build.sh
    └── run
        └── Dockerfile
```

### 创建 buildpack

```
pack buildpack new examples/maven \
                         --api 0.5 \
                         --path buildpack-maven \
                         --version 0.0.1 \
                         --stacks io.buildpacks.samples.stacks.bionic
```

看下生成的 `buildpack-maven` 目录：

```
buildpack-maven
├── bin
│   ├── build
│   └── detect
└── buildpack.toml
```

各个文件中都是默认的初试数据，并没有什么用处。需要添加些内容：

**`bin/detect`：**

```
#!/usr/bin/env bash

if [[ ! -f pom.xml ]]; then
    exit 100
fi

plan_path=$2

cat >> "${plan_path}" <<EOL
[[provides]]
name = "jdk"
[[requires]]
name = "jdk"
EOL
```

**`bin/build`：**

```
#!/usr/bin/env bash

set -euo pipefail

layers_dir="$1"
env_dir="$2/env"
plan_path="$3"

m2_layer_dir="${layers_dir}/maven_m2"
if [[ ! -d ${m2_layer_dir} ]]; then
  mkdir -p ${m2_layer_dir}
  echo "cache = true" > ${m2_layer_dir}.toml
fi
ln -s ${m2_layer_dir} $HOME/.m2

echo "---> Running Maven"
mvn clean install -B -DskipTests

target_dir="target"
for jar_file in $(find "$target_dir" -maxdepth 1 -name "*.jar" -type f); do
  cat >> "${layers_dir}/launch.toml" <<EOL
[[processes]]
type = "web"
command = "java -jar ${jar_file}"
EOL
  break;
done
```

**`buildpack.toml`：**

```
api = "0.5"

[buildpack]
  id = "examples/maven"
  version = "0.0.1"

[[stacks]]
  id = "com.atbug.buildpacks.example.stacks.maven"
```

### 创建 stack

构建 Maven 项目，首选需要 Java 和 Maven 的环境，我们使用 `maven:3.5.4-jdk-8-slim` 作为 build image 的 base 镜像。应用的运行时需要 Java 环境即可，因此使用 `openjdk:8-jdk-slim`作为 run image 的 base 镜像。

在 `stacks` 目录中分别创建 `build` 和 `run` 两个目录：

**`build/Dockerfile`**

```
FROM maven:3.5.4-jdk-8-slim

ARG cnb_uid=1000
ARG cnb_gid=1000
ARG stack_id

ENV CNB_STACK_ID=${stack_id}
LABEL io.buildpacks.stack.id=${stack_id}

ENV CNB_USER_ID=${cnb_uid}
ENV CNB_GROUP_ID=${cnb_gid}

# Install packages that we want to make available at both build and run time
RUN apt-get update && \
  apt-get install -y xz-utils ca-certificates && \
  rm -rf /var/lib/apt/lists/*

# Create user and group
RUN groupadd cnb --gid ${cnb_gid} && \
  useradd --uid ${cnb_uid} --gid ${cnb_gid} -m -s /bin/bash cnb

USER ${CNB_USER_ID}:${CNB_GROUP_ID}
```

**`run/Dockerfile`**

```
FROM openjdk:8-jdk-slim

ARG stack_id
ARG cnb_uid=1000
ARG cnb_gid=1000
LABEL io.buildpacks.stack.id="${stack_id}"

USER ${cnb_uid}:${cnb_gid}
```

然后使用如下命令构建出两个镜像：

```
export STACK_ID=com.atbug.buildpacks.example.stacks.maven

docker build --build-arg stack_id=${STACK_ID} -t addozhang/samples-buildpacks-stack-build:latest ./build
docker build --build-arg stack_id=${STACK_ID} -t addozhang/samples-buildpacks-stack-run:latest ./run
```

### 创建 Builder

有了 buildpack 和 stack 之后就是创建 Builder 了，首先创建 `builder.toml` 文件，并添加如下内容：

```
[[buildpacks]]
id = "examples/maven"
version = "0.0.1"
uri = "../buildpacks/buildpack-maven"

[[order]]
[[order.group]]
id = "examples/maven"
version = "0.0.1"

[stack]
id = "com.atbug.buildpacks.example.stacks.maven"
run-image = "addozhang/samples-buildpacks-stack-run:latest"
build-image = "addozhang/samples-buildpacks-stack-build:latest"
```

然后执行命令，**注意这里我们使用了 `--pull-policy if-not-present` 参数，就不需要将 stack 的两个镜像推送到镜像仓库了**：

```
pack builder create example-builder:latest --config ./builder.toml --pull-policy if-not-present
```

### 测试

有了 builder 之后，我们就可以使用创建好的 builder 来构建镜像了。

这里同样加上了 `--pull-policy if-not-present` 参数来使用本地的 builder 镜像：

```
# 目录 buildpacks-sample  与 tekton-test 同级，并在 buildpacks-sample  中执行如下命令
pack build addozhang/tekton-test --builder example-builder:latest --pull-policy if-not-present --path ../tekton-test
```

如果看到类似如下内容，就说明镜像构建成功了（第一次构建镜像由于需要下载 maven 依赖耗时可能会比较久，后续就会很快，可以执行两次验证下）：

```
...
===> EXPORTING
[exporter] Adding 1/1 app layer(s)
[exporter] Reusing layer 'launcher'
[exporter] Reusing layer 'config'
[exporter] Reusing layer 'process-types'
[exporter] Adding label 'io.buildpacks.lifecycle.metadata'
[exporter] Adding label 'io.buildpacks.build.metadata'
[exporter] Adding label 'io.buildpacks.project.metadata'
[exporter] Setting default process type 'web'
[exporter] Saving addozhang/tekton-test...
[exporter] *** Images (0d5ac1158bc0):
[exporter]       addozhang/tekton-test
[exporter] Adding cache layer 'examples/maven:maven_m2'
Successfully built image addozhang/tekton-test
```

启动容器，会看到 spring boot 应用正常启动：

```
docker run --rm addozhang/tekton-test:latest
  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::        (v2.2.3.RELEASE)

 ...
```

## 总结

其实现在有很多开源的 buildpack 可以用，没有特定定制的情况下无需自己手动编写。比如下面的几个大厂开源并维护的 Buildpacks：

-   [Heroku Buildpacks](https://github.com/heroku/)
-   [Google Buildpacks](https://github.com/GoogleCloudPlatform/buildpacks)
-   [Paketo](https://github.com/paketo-buildpacks)

上面几个 buildpacks 库内容比较全面，实现上会有些许不同。比如 Heroku 的执行阶段使用 Shell 脚本，而 Paketo 使用 Golang。后者的扩展性较强，由 Cloud Foundry 基金会支持，并拥有由 VMware 赞助的全职核心开发团队。这些小型模块化的 buildpack，可以通过组合扩展使用不同的场景。

当然还是那句话，自己上手写一个会更容易理解 Buildpack 的工作方式。