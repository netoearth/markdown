### 目的

搭建这个项目的目的在于，自己的项目若想每次发布到线上，每次都需要经过一下这样一段流程化操作。

打包项目->上传到服务器->构建镜像->运行容器。

因此期望通过 **Jenkins** 这样的平台，实现代码一旦 **push** 后，自动完成以上的流程。 搭建期间遇到了许多问题，花了两天时间搭建，希望能帮助一些小伙伴避雷。

本文作为本人学习所记录，仅供参考。

项目地址： [github.com/ylhao666/si…](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fylhao666%2Fsimple-java-maven-app "https://github.com/ylhao666/simple-java-maven-app")

### 实现过程

#### 开发环境

| 软件 | 版本 | 备注 |
| --- | --- | --- |
| Centos | 8.2.2004 | 腾讯云服务器 |
| Docker | 20.10.12 | 运行在云服务器上 |
| Jenkins | latest | **jenkinsci/blueocean** [官方镜像](https://link.juejin.cn/?target=https%3A%2F%2Fhub.docker.com%2Fr%2Fjenkinsci%2Fblueocean "https://hub.docker.com/r/jenkinsci/blueocean") |
| SpringBoot | 2.6.3 | 实现简单的 Web 应用 |

#### 基于 Docker 搭建 Jenkins

搭建 Jenkins 的方式有多种，具体可以查看 [官方文档](https://link.juejin.cn/?target=https%3A%2F%2Fwww.jenkins.io%2Fzh%2Fdoc%2Fbook%2Finstalling%2F "https://www.jenkins.io/zh/doc/book/installing/")，本次使用容器运行，将 Jenkins 部署在 Docker 中，运行语句如下：

```
docker run \
  -dp 8080:8080 \
  -v jenkins-data:/var/jenkins_home \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v "$HOME":/home \
  --restart=always
  jenkinsci/blueocean
复制代码
```

大概解释一下各个参数的意思

1.  `-dp` 映射宿主机 8080 端口到 Jenkins **8080** 端口上，用于访问 Jenkins 主页，同时将 Jenkins 运行在后台
2.  `-v jenkins-data:/var/jenkins_home` 映射卷到 `/var/jenkins_home` 路径，保存了 Jenkins 的基本信息，此项确保重新运行容器后，数据不会被清空，**建议**映射
3.  `-v /var/run/docker.sock:/var/run/docker.sock` 映射 `docker.sock` 文件，这样执行 docker 命令时，响应的是宿主机，具体可以查看[此文章](https://link.juejin.cn/?target=https%3A%2F%2Fcloud.tencent.com%2Fdeveloper%2Farticle%2F1454335 "https://cloud.tencent.com/developer/article/1454335")，**此操作是实现本次部署的关键**
4.  `-v "$HOME":/home` 映射宿主机 `home` 目录到 Jenkins `home` 上
5.  `--restart=always` Docker 重启时，跟随重启

运行成功后，访问 `http://your_ip:8080`，跟随指引完成 Jenkins 的安装

**注意**：安装插件时，选择安装推荐的插件

#### 创建项目

##### 编写 SpringBoot 项目

接下来，基于 **SpringBoot** 搭建一个简易的 Web 应用

引入依赖

```
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-web</artifactId>
</dependency>
复制代码
```

写一个 `HelloController`，提供一个 `/hello` 的接口，用于测试

```
@RestController
public class HelloController {

    @GetMapping("hello")
    public String hello() {
        return "Hello World";
    }

}
复制代码
```

指定默认运行端口，因为 `8080` 已经被 Jenkins 使用了，换成其他端口

```
server:
  port: 9092
复制代码
```

指定构建参数，指定生成的 Jar 包名称，引入 SpringBoot maven 插件，用于构建 Jar 包

```
<build>
  <finalName>My-App</finalName>
  <plugins>
    <plugin>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-maven-plugin</artifactId>
    </plugin>
  </plugins>
</build>
复制代码
```

在本地测试，运行成功，访问 `127.0.0.1:9092/hello`，返回 Hello World

把项目托管到 Github 上，用于被 Jenkins 拉取

[github.com/ylhao666/si…](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fylhao666%2Fsimple-java-maven-app "https://github.com/ylhao666/simple-java-maven-app")

##### Jenkins 创建项目

1.  打开 Jenkins 页面，新建任务

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/30f7a4b66bb1433ab413cdda7687b89d~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.image?)

2.  任务名写项目名，勾选流水线

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e6115239cffa411ea7a0168f582c955c~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.image?)

3.  选择拉取代码的方式，运行时从 Git 代码库拉取代码

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8d39cad535b44f94813d07ca0e47b613~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.image?)

**注意**：需要添加凭证，以便于拉取代码，设置 `ssh` 方式或者 `username & password` 方式都可以

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cfd6a1a3ded443c8a0624a5dce0df0f8~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.image?)

4.  指定构建分支，已经默认的 `Jenkinsfile` 名称

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bee5024a4e724496b05090f48876ebe4~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.image?)

新建项目的初始工作就完成了，接下来开始编写 **Jenkinsfile**，定义流水线逻辑

#### 编写 Jenkinsfile

使用 Jenkins 的关键在于 **Jenkinsfile** 的编写，需要了解一点 `shell`

##### 实现 Jenkinsfile 自动补全

IDEA 似乎不支持 **Jenkinsfile** 的语法提醒，需要手动配置一下，配置完写起来方便一点

1.  获取 **gdsl** 定义文件，首先访问 `http://{{your_ip}}:8080/job/{{your_project_name}}/pipeline-syntax/gdsl` 获取 **gdsl** 文件，复制下来，到项目的 **src.main.java** 目录下，新建 **pipeline.gdsl** 文件，粘贴复制的内容。

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0a1761bc948e44e1b14a8c0af0292306~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.image?)

2.  设置 IDEA ，令其识别 **Jenkinsfile** 文件支持 **Groovy**

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/166b97bbfe9149ed92b4cd31d4b5a012~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.image?)

3.  在项目根目录，新建 **Jenkinsfile** 文件，尝试发现已经可以自动补全

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9e11300c548648c3924589340f09adcf~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.image?)

**注意**：从 **Jenkins** 获取的 `gdsl` 文件可能存在一些未自动补全的字段

可以从 **git** 地址获取完整的 `pileline.gdsl` 文件

##### 编写 Jenkinsfile

本次实现的流水线，首先进行 **Jar** 包的构建，并将其打包成 **Docker 镜像**，然后将镜像运行在**宿主机**的 Docker 容器上

完整的 **Jenkinsfile** 可以在附录查看

**编写构建阶段 Build Stage**

1.  首先定义一个环境变量 `PACKAGE_NAME`，用于 **Maven** 进行打包时使用

```
environment {
    PACKAGE_NAME = 'My-App'
}
复制代码
```

2.  定义 `Build` Stage，将构建过程运行在 **Maven** 容器上

```
// 构建阶段
stage('Build') {
    agent {
        docker {
            image 'maven:3.6.3-slim'
            // 挂载在宿主机上，复用依赖文件
            args '-v /root/.m2:/root/.m2'
        }
    }
}
复制代码
```

由于我们的 **Jenkins** 是运行于宿主机的 Docker 上的，并且在运行时指定了 `docker.sock` 文件的映射，因此构建阶段运行的 **Maven** 容器是运行在宿主机上的，相当于在宿主机上运行 `docker run -v /root/.m2:/root/.m2 maven:3.6.3-slim`，同时映射宿主机的本地 **Maven** 仓库，以便于复用依赖。

3.  指定运行时步骤 `steps`，定义 `build.sh` 脚本，把所有指令放在脚本里执行

```
steps {
    sh 'sh ./jenkins/scripts/build.sh'
    // 暂存 Jar 包，避免不同 agent 下取不到文件
    stash includes: '**/target/*.jar', name: 'jar'
}
复制代码
```

**注意**：因为流水线上不同的 `stage` 可以指定不同的 `agent`，以便运行在不同的环境。所以不同的 `agent` 下，数据是不共享的。因此 `Build` Stage 下构建完成的 Jar 包，可以通过 `stash` 命令进行暂存，后续在 `Deploy` Stage，可以通过 `unstash` 获取。

4.  编写 `build.sh` 脚本

```
# 构建 Jar 包，跳过测试
mvn -B -DskipTests clean package
复制代码
```

`build.sh` 进行的工作很简单，只是对项目进行打包。

**编写测试阶段 Test Stage**

1.  编写 `Test Stage`

```
// 单元测试
stage('Test') {
    steps {
        sh 'sh ./jenkins/scripts/test.sh'
    }
}
复制代码
```

跟 `Build Stage` 一样，将命令抽离到脚本执行

2.  编写 `test.sh` 脚本

```
# test
echo "Test"
复制代码
```

在这里可以对代码做一些测试，**Jenkins** 也支持对测试的结果进行展示，具体可以查看[官方文档](https://link.juejin.cn/?target=https%3A%2F%2Fwww.jenkins.io%2Fzh%2Fdoc%2Fbook%2Fpipeline%2Fjenkinsfile%2F%23test "https://www.jenkins.io/zh/doc/book/pipeline/jenkinsfile/#test")

**编写部署阶段 Deploy Stage**

1.  定义新的环境变量

```
environment {
    IMAGE_NAME = 'my-app'
    IMAGE_VERSION = '1.0.0'
    SERVER_PORT = '7072'
    APP_NAME = 'My-App'
    APP_VERSION = '1.0.0'
}
复制代码
```

用于指定镜像名，镜像版本，服务运行的端口，应用名称，应用版本

2.  定义 `Deploy Stage`

```
// 部署容器
stage('Deploy') {
    steps {
        // 获取 Build Stage 构建的 Jar 包
        unstash 'jar'
        sh 'sh ./jenkins/scripts/deploy.sh'
    }
    post {
        failure {
            echo "部署失败"
        }
    }
}
复制代码
```

定义 `steps`，首先从暂存中获取 `Build Stage` 阶段构建的 Jar 包，然后运行 `deploy.sh` 脚本。 `post` 可以根据不同的运行结果进行不同的响应，这里如果部署失败的话，打印 _部署失败_，可以使用 email 进行告警，具体可以查看[清理和通知](https://link.juejin.cn/?target=https%3A%2F%2Fwww.jenkins.io%2Fzh%2Fdoc%2Fpipeline%2Ftour%2Fpost%2F "https://www.jenkins.io/zh/doc/pipeline/tour/post/")。

```
post {
    failure {
        mail to: 'team@example.com',
             subject: "Failed Pipeline: ${currentBuild.fullDisplayName}",
             body: "Something is wrong with ${env.BUILD_URL}"
    }
}
复制代码
```

3.  定义 **Dockerfile**

```
FROM openjdk:8-jre-slim
ARG PACKAGE_NAME
WORKDIR /app
COPY ${PACKAGE_NAME}.jar ./${PACKAGE_NAME}.jar
RUN echo "java -jar ${PACKAGE_NAME}.jar \${@}" > ./entrypoint.sh 
    && chmod +x ./entrypoint.sh
ENTRYPOINT ["sh", "entrypoint.sh"]
复制代码
```

在项目目录下，创建 `docker/Dockerfile` 文件，用于构建镜像。

大概解释一下 **Dockerfile** 的内容

-   根据 `openjdk:8-jre-slim` 镜像进行构建
-   定义构建参数 `PACKAGE_NAME`，用于构建时传递 Jar 包名称
-   指定工作目录为 `/app`
-   复制 Jar 包到工作目录下
-   定义运行脚本 `entrypoint.sh`，赋予运行权限。`${@}` 用于运行时接受从命令传递的参数。本来期望直接使用 `ENTRYPOINT` 命令，尝试后似乎无法同时接受 `ARG` 已经运行时参数，因此只能使用脚本，具体可以查看[此提问](https://link.juejin.cn/?target=https%3A%2F%2Fstackoverflow.com%2Fquestions%2F49133234%2Fdocker-entrypoint-with-env-variable-and-optional-arguments "https://stackoverflow.com/questions/49133234/docker-entrypoint-with-env-variable-and-optional-arguments")。
-   执行脚本

4.  编写 `deploy.sh` 脚本

```
# 复制 Jar 包到 docker 目录
cp "target/${PACKAGE_NAME}.jar" "docker/${PACKAGE_NAME}.jar"

# 构建镜像
docker build -t "${IMAGE_NAME}:${IMAGE_VERSION}" --build-arg PACKAGE_NAME="${PACKAGE_NAME}" docker

# run container
# 删除旧容器
containerId=$(docker ps -f name="${APP_NAME}-${APP_VERSION}" -aq)
if [ "${containerId}" != "" ]; then
    docker rm -f "${containerId}"
fi

# 运行新容器
docker run --restart=always -dp "${SERVER_PORT}:${SERVER_PORT}" --name "${APP_NAME}-${APP_VERSION}" "${IMAGE_NAME}:${IMAGE_VERSION}" --server.port="${SERVER_PORT}"

# 判断容器运行情况，未运行则抛出异常
docker ps -f name="${APP_NAME}-${APP_VERSION}"
containerId=$(docker ps -f name="${APP_NAME}-${APP_VERSION}" -q)

if [ "${containerId}" = "" ]; then
    exit 42
fi
复制代码
```

-   首先，复制从 `Build Stage` 构建的 Jar 包到 `docker` 目录下
-   开始构建镜像，镜像名从环境变量中获取，同时传递构建参数 `PACKAGE_NAME`，指定上下文为 `docker` 目录
-   根据容器名称获取 **Docker** 中运行的旧容器id，删除旧容器
-   运行新容器，`${SERVER_PORT}:${SERVER_PORT}` 映射运行端口，`"${APP_NAME}-${APP_VERSION}"` 指定容器名称，`--server.port` 传递运行时参数,指定运行端口
-   判断容器运行情况，未运行则抛出异常，终止流水线进行异常告警

**Jenkinsfile** 到这里就编写完了，接下来可以尝试运行流水线。

所有完整的 sh 文件均可以在附录中获取

#### 运行流水线

首先提交代码，然后访问 **Jenkins** 主页，点击刚刚创建的项目

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/661974f6a9dd437cbe724e0765e85f60~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.image?)

点击打开 BlueOcean

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7152d97d447a426e89d839c3d6eb1cf2~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.image?)

点击 Run，开始运行流水线

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/399128a28f53409d9787517282ea1191~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.image?)

可以发现运行成功，同时可以查看每个 `steps` 打印的内容

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7feeb64ff26a45bdae1833b608612bcd~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.image?)

> 绿了绿了芜湖

访问 `{{your_ip}}:7072/hello`，返回 `Hello World`，可以发现已经成功部署到服务器上

#### 设置 Git Webhooks 实现自动化部署

上面已经实现了手动部署，提交完代码后，点击一下就可以自动构建，部署。下面尝试配置自动化部署，这里使用到了 **Git** 的 **Webhooks**。

1.  配置 **Jenkins** 在 **Jenkis** 的配置中，配置 Github 服务器。名称随便填写，API URL 默认即可，**关键是凭证**，需要在 **Github** 中进行申请 _access\_token_，可以点击[这里](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fsettings%2Ftokens%2Fnew "https://github.com/settings/tokens/new")进行申请。按照官方的要求，至少申请如下的权限：

-   **admin:repo\_hook** - for managing hooks (read, write and delete old ones)
-   **repo** - to see private repos
-   **repo:status** - to manipulate commit statuses

把生成得到的 access\_token 填写到凭证里

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/73336c8b9e14419a8ef5bf121eccb8c4~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.image?)

可以点击连接测试，检查是否配置成功

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/04cc961462e8460ab4494af9f362ff5b~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.image?)

2.  设置 **Webhooks** 打开你的 **git** 项目地址，选择 `Settings` - `Webhooks`，大概 url 如下 `github.com/{{your_account}}/{{your_project_name}}/settings/hooks`

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f48a2deb898644c39144d9259d36d8ad~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.image?)

点击 `Add webhook` 新增

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/02353f42ded946dc9bb19030fb90a59a~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.image?)

**Payload URL** 填写 `http://{{your_jenkins_url}}/github-webhook/`，同时选择触发事件，当 `push` 后进行触发，生成后，可以点击尝试触发，查看 **Jenkins** 日志，是否有收到触发事件。

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c5dce2569470418b9ab9166161b3bb09~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.image?)

3.  配置项目触发器

打开任务，查看构建触发器，勾选 `GitHub hook trigger for GITScm polling`，保存

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6b939f2d7f384ac3a1ceb4e5f1c4c750~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.image?)

4.  验证

修改代码，`push` 后，到任务主页，点击 `Github Hook Log`，刷新，可以看到 **Push** 记录

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5a56737ce3014923866e319fc8220839~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.image?)

查看 **Blue Ocean**，可以看到流水线已经在运行中，这样就自动化部署就配置好了。

### 最终效果

最终，我们实现了，代码从 `push` 到构建，到运行的全过程。

> 真不错，终于不用手动进行部署了

如果本文对你有所帮助，就请点个 👍 ，点个关注吧。

### 附录

> Jenkinsfile

```
pipeline {
    agent any
    environment {
        APP_NAME = 'My-App'
        APP_VERSION = '1.0.0'
        PACKAGE_NAME = 'My-App'
    }
    stages {
        // 构建 jar
        stage('Build') {
            agent {
                docker {
                    image 'maven:3.6.3-slim'
                    // 挂载在宿主机上，复用依赖文件
                    args '-v /root/.m2:/root/.m2'
                }
            }
            steps {
                sh 'sh ./jenkins/scripts/build.sh'
                // 暂存 Jar 包，避免不同 agent 下取不到文件
                stash includes: '**/target/*.jar', name: 'jar'
            }
        }

        // 单元测试
        stage('Test') {
            steps {
                sh 'sh ./jenkins/scripts/test.sh'
            }
        }

        // 部署容器
        stage('Deploy') {
            environment {
                IMAGE_NAME = 'my-app'
                IMAGE_VERSION = '1.0.0'
                SERVER_PORT = '7072'
            }
            steps {
                unstash 'jar'
                sh 'sh ./jenkins/scripts/deploy.sh'
            }
            post {
                failure {
                    echo "部署失败"
                }
            }
        }
    }
    //    全局post
    post {
        always {
            echo "Always"
        }
        success {
            echo "Success"
        }
        failure {
            echo "Failure"
        }
    }
}
复制代码
```

> deploy.sh

```
# 校验 Jar 包是否存在
if ! test -f "target/${PACKAGE_NAME}.jar"; then
    echo "${PACKAGE_NAME}.jar 不存在"
    exit 43
fi

echo "复制 Jar 包到 Docker 文件夹"
cp "target/${PACKAGE_NAME}.jar" "docker/${PACKAGE_NAME}.jar"

# 构建镜像
echo "开始构建镜像"
docker build -t "${IMAGE_NAME}:${IMAGE_VERSION}" --build-arg PACKAGE_NAME="${PACKAGE_NAME}" docker

# run container
# 删除旧容器
containerId=$(docker ps -f name="${APP_NAME}-${APP_VERSION}" -aq)
if [ "${containerId}" != "" ]; then
    echo "删除旧容器 ${containerId}"
    docker rm -f "${containerId}"
fi

# 运行新容器
echo "运行新容器, ContainerName: ${APP_NAME}-${APP_VERSION}"
docker run --restart=always -dp "${SERVER_PORT}:${SERVER_PORT}" --name "${APP_NAME}-${APP_VERSION}" "${IMAGE_NAME}:${IMAGE_VERSION}" --server.port="${SERVER_PORT}"

# 判断容器运行情况，未运行则抛出异常
echo "容器运行情况:"
docker ps -f name="${APP_NAME}-${APP_VERSION}"
containerId=$(docker ps -f name="${APP_NAME}-${APP_VERSION}" -q)

if [ "${containerId}" = "" ]; then
    echo "容器未运行"
    exit 42
fi
复制代码
```