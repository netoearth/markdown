   

[![](https://p26-passport.byteacctimg.com/img/mosaic-legacy/3795/3033762272~300x300.image)](https://juejin.cn/user/2058748597121191)

2022年03月23日 22:38 ·  阅读 133

入门 [CICD]、 gitlab pipeline](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/933fdd00eaac4e0fb8bbc496f7fd8c74~tplv-k3u1fbpfcp-zoom-crop-mark:3024:3024:3024:1702.awebp?)  

#什么是 [[CICD]]

## 软件开发测试面临的问题

在过去，我们开发发布软件通常需要比较长的一个周期。原因是我们需要非常多的人工流程来保障我们的软件质量。这些人工流程包括代码提交、合并、开发阶段测试、部署、内测、灰度发布、正式发布。所有的这些流程都会耗费大量人力，并且人工进行操作通常会引发因为操作不规范或粗心大意带来的许多问题。 现在越来越多的公司发布软件新版本的频次越来越高，有的甚至做到每周发一个版本。在传统人工开发流程下，是很难保证软件的质量的。所以 CI（Continuous integration） 、CD（Continuous delivery）即持续集成、持续发布的概念应运而生。

## CI

持续集成要求我们的开发每天将自己的代码频繁的提交进代码仓库。每次提交都会自动触发自动化测试、编译、生成结果的流程。当 CI 中自动化测试的任务出现了问题，会及时的自动通知到对应的开发人员。这样开发可以更快的提前发现自己的代码问题，并且进行修复。 CI 非常依赖当前自动化测试的程度，自动化测试覆盖率越高越能提前发现问题。当然，因为每次提交代码都要等待自动化任务的结束，所以也不宜设置全量的 case 进行作为 CI 任务。 一旦 CI 任务失败了，开发应该立即进行修复再提交。

## CD

持续部署要求我们的代码变更（包括 bug 修复、新功能等），及时的持续的第一时间发布到用户。我们应该保证我们的代码是随时可以发布的。开发人员更多关注 CI 以保证提交的代码质量，运维运营人员更多关注 CD。 CI 的流程的结果是通过保证质量的代码提交进行待发布部署的代码产物的生成，CD 则关注于如何自动化的将这些代码生成产物部署到各个环境（开发环境、测试环境、生产环境）。

## 现有成熟的解决方案

Jenkins:免费开源 Azure Pipelines：微软的产品 Cloud Build：Google的产品 GitLab [[CICD]]：Gitlab的产品 Bammboo：Atlassian的产品 #gitlab [[CICD]] 使用入门

## gitlab pipeline 介绍

Gitlab pipeline 是 Gitlab [[CICD]] 的功能组件，在 Gitlab 的侧边栏你就可以看到它。Gitlab pipeline 主要包含两个概念： Stage 和 Job。 Job 是定义具体做什么事情，通过编写 shell 脚本来实现具体要做什么事情。 Job 的运行需要 Gitlab Runner，runner 需要提供 Job 运行需要的环境，如 node、java等。 runner 可以是一台远端服务器，也可以是本地机器、也可以是虚拟机或者 docker。 Gitlab 官网提供了各种环境的安装教程。

Stage 是定义什么时候运行一个 Job。 通过 Stage 和 Job 的配合就可以完成 CI、CD 的流程定义。比如可以定义一个测试 Stage、一个编译 Stage、一个发布 Stage。这样我们提交了代码后就会自动化的在 Gitlab Runner 上面进行各个 Stage 的任务了。从而实现 CDCD。

Gitlab Pipeline Job支持定义各种触发条件，比如可以只在提交 MR 时触发（用于拦截 CI 任务失败的 MR 请求，这时候代码还不会合并到目标分之）。也可以定义只能手动触发，比如部署到生产环境的任务需要手动触发。 ![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1de3e020f25945b0878ace41514de64e~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

## gitlab [[CICD]] 使用教程

简单来说就是先搭建 gitlab runner，让 CI、CD 的 Job 有地方运行。然后就是定义各 Stage 的任务。就这么简单。

### 搭建 gitlab runner

我们可以通过 docker 进行搭建，也可以通过真机器进行搭建。搭建流程见 [官网链接](https://link.juejin.cn/?target=https%3A%2F%2Fdocs.gitlab.com%2Frunner%2F "https://docs.gitlab.com/runner/")。 比如下面看看如何通过 docker 安装 Gitlab runner

```
docker run -d --name gitlab-runner --restart always \
-v /srv/gitlab-runner/config:/etc/gitlab-runner \
-v /var/run/docker.sock:/var/run/docker.sock \ 
gitlab/gitlab-runner:latest
复制代码
```

这个命令会自动拉取最新的 gitlab runner 的 docker 镜像并启动。 安装完成后需要进行 gitlab runner 的注册。

```
docker run --rm -it -v /srv/gitlab-runner/config:/etc/gitlab-runner gitlab/gitlab-runner register
复制代码
```

运行这个命令后会提示需要输入：

1.  gitlab url 和 注册 token： 在 gitlab 页面上就可以看到

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ef63d90a03f04924a2e3412a62578bd3~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?) 2. 描述信息 3. tag 信息，任务可以指定 tag 在对应的 runner 上运行 4. 其他信息使用默认值即可

### gitlab [[CICD]] 配置文件

.gitlab-ci.yml 是 gitlab 的 [[CICD]] 配置文件，它需要和源码一起进行提交。

### 测试基本运行

```
test-job:
  script:
  - echo 'hello world'
复制代码
```

### 搭建一个自动测试、构建、测试包发布，的流程

通过一个简单的示例演示搭建一个自动测试、构建、测试包发布，的流程

#### 配置自动执行测试

```
stages:
  - test
test-job:
  stage: test
  script:
    - echo 'test is running'
    - npm install
    - npm run test
复制代码
```

#### 配置自动执行编译

```
stages:
  - test
  - build
test-job:
  stage: test
  script:
    - echo 'test is running'
    - npm run test
build-job:
  stage: build
  script:
    - echo 'build is running'
    - npm run build
复制代码
```

#### 配置自动执行测试包发布

```
stages:
  - test
  - build
  - deploy
test-job:
  stage: test
  script:
    - echo 'test is running'
    - npm run test
build-job:
  stage: build
  script:
    - echo 'build is running'
    - npm run build
deploy-job:
  stage: deploy
  script:
    - echo 'deploy is running'
    - npm run deploy
复制代码
```

最后我们提交代码就会以此运行测试 （CI）、编译（CI）、发布（CD）这样的任务了。当然以上只是一个简单的示范，现实中会比这复杂许多。 仅供入门参考。

文章被收录于专栏：

![cover](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bb9f5f9dd673434aad4a97f10bbe0bef~tplv-k3u1fbpfcp-no-mark:160:160:160:120.awebp?)

![](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/00ba359ecd0075e59ffbc3d810af551d.svg) 2

![](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/3d482c7a948bac826e155953b2a28a9e.svg) 收藏