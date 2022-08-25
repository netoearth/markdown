在公司搭建内部 GitLab 平台后，前端活动项目从 SVN 迁移到 GitLab。本文介绍如何基于 GitLab CI/CD 实现自动化构建及发布。

在从 SVN 迁移到 GitLab 和接入 GitLab CI/CD 的过程中，特别感谢发布系统和服务端同学的大力支持。

**一、目前的构建、发布流程**

在这部分，我们先给出使用 GitLab CI/CD 的收益，然后分别介绍使用 GitLab CI/CD 之前以及之后的构建、发布流程。

**1\. 团队收益**

1\. 发布时间由平均 5 分钟减少到 1.5 分钟。从全程 5 分钟的手动操作，到只需合并分支代码、自动化构建及发布的 1.5 分钟。

2\. 前端构建放到 CI/CD 中，解决了本地构建可能导致线上代码打包后不一致的问题。

**2\. 使用 GitLab CI/CD 前的构建、发布**

**2.1 流程**

1\. 开发完成后，在本地进行 build，build 后提交打包后的 HTML 文件。

2\. 打开发布系统，发布 HTML 文件。

**2.2 痛点**

1\. 发布代码的操作流程长，耗时。开发完成后，发布代码时，需要经过大量的手动操作，加上发布系统慢，每次发布大约需要 5 分钟。

2\. 每个人本地环境不一致，线上代码不同的人发布有不一致的风险。

**2.3 需要手动操作的费时点**

一共需要至少 8 次手动操作

本地构建：

1\. 执行 build 命令，等待 build 完成。

2\. build 完成后，提交打包后的 HTML 文件。

发布代码：

1\. 打开发布系统

2\. 选择发布项目及环境

3\. 打开发布页面

4\. 选择发布文件

5\. 填写发布信息

6\. 点击确认发布

**3\. 使用 GitLab CI/CD 后的构建、发布**

发布代码 1 步到位：只需将开发分支合并至发布环境对应的分支，提交分支后，GitLab CI/CD 自动进行构建、发布。

**二、什么是 GitLab CI/CD**

这部分我们先简要介绍下 GitLab CI/CD，然后介绍如何从零搭建一个 GitLab CI/CD。

**1\. 简要介绍 GitLab CI/CD**

如下图所示，提交代码到 GitLab 后，满足指定条件后会触发 pipeline 进行自动化构建、发布。

pipeline 可以理解为构建任务，里面可以包含多个流程，如下载依赖、运行测试、编译、部署。pipeline 什么时候触发，分为几个流程，每个流程做什么，是在项目的 .gitlab-ci.yml 文件中定义。

对于我们的活动项目而言，pipeline 主要包括了，依赖下载，build 代码，发布到服务器这三个过程。后续实践部分会具体介绍。

**2\. GitLab CI/CD 整体流程**

GitLab CI/CD 的 pipeline 具体流程和操作在 .gitlab-ci.yml 文件中申明，触发 pipeline 后，由 GitLab Runner 根据 .gitlab-ci.yml 文件运行，运行结束后将返回至 GitLab 系统。

**2.1 .gitlab-ci.yml 文件**

.gitlab-ci.yml 文件是一个申明式文件，用于定义 GitLab CI/CD 流程分为几个阶段，每个阶段分别干什么。

关于具体干什么、怎么干，主要使用命令行和脚本操作，稍后会在实践部分做细致的介绍。

如果涉及一些逻辑的话，会使用脚本，我们的项目主要使用 Shell 脚本，Python 脚本。

**2.1 GitLab Runner**

GitLab Runner 是 CI 的执行环境，负责执行 gitlab-ci.yml 文件，并将结果返回给 GitLab 系统。Runner 具体可以有多种形式，[docker](https://cloud.tencent.com/product/tke?from=10680)、虚拟机或 shell，在注册 runner 时选定方式。

**3\. 从零搭建一个 GitLab CI/CD**

为了解整个流程，可以在 GitLab 上建一个项目，跑一跑整个 CI/CD 流程。

**3.1 新建一个 GitLab 项目**

1\. 登录 GitLab：https://gitlab.com/

2\. 新建一个自己的项目

**3.2 配置 Runner**

GitLab 提供了一些共享的 Runner，我们可以不用处理 Runner。

**3.3 新建 .gitlab-ci.yml 文件**

1\. 拉取项目到本地

2\. 在项目根目录新建 .gitlab-ci.yml 文件

3\. 提交 .gitlab-ci.yml 文件

4\. 在项目的 CI/CD 中，可以看到 CI/CD 的运行情况

.gitlab-ci.yml 文件示例

```
image: node
# 定义 stages
stages:
  - build
  - test
# 定义 job
 build 阶段:
  stage: build
  script:
    - echo "build stage"
# 定义 job
发布到测试环境:
  stage: test
  script:
    - echo "test stage"
```

**三、新项目接入 GitLab CI/CD 流程**

团队有新项目需要接入 GitLab CI/CD，首先申请 GitLab 项目，再让 GitLab 系统维护者帮忙配置 Runner，编写 .gitlab-ci.yml 文件，触发构建即可看 CI/CD 情况。

1\. 新建 GitLab 项目

2\. 配置 GitLab Runner

3\. .gitlab-ci.yml 文件

目前已有两个新的项目路接入了 GitLab CI/CD，接入情况不错，根据文档进行操作过程比较顺利。

**四、GitLab CI/CD 实践**

在实践部分，这里着重介绍 GitLab Runner 和 .gitlab-ci.yml 文件，主要的流程及遇到的问题和解决方案包含在 .gitlab-ci.yml 文件的介绍过程中。

**1\. GitLab Runner**

GitLab Runner 一般由 GitLab 系统维护者管理，配置后，同类项目可以共享，一般不需要进行修改。这里不进行具体介绍，主要介绍下使用过程中的注意点，具体使用可参考 GitLab Runner 文档。（https://docs.gitlab.com.cn/runner/）

**1.1 GitLab Runner 使用流程**

1\. 下载 GitLab Runner

2\. 注册 GitLab Runner

3\. 使用 GitLab Runner

**1.2 GitLab Runner 注意点**

在使用 Runner 的过程中，我们遇到了一些问题，下面简要介绍问题及解决方案，不做具体介绍。

**1.2.1 配置 Runner 后，push 代码，出发了 pipeline，但一直处于Pending状态**

错误信息是：This job is stuck, because you don’t have any active runners that can run this job

注册的 Runner，默认情况下，不会运用没有 tag 的 job，可以在 Settings→CI/CD→Runners Settings，去掉 Runner untagged jobs 即可。

**1.2.2 GitLab Runner 的类型**

有三种类型的 Runner，Shared Runners 在整个系统所有项目都可以使用，Group Runners 注册后，同一个项目下的不同代码库共享，Specific Runners 需要给项目单独配置，使用 Specific Runners 注意考虑是否需要关闭 Shared Runners、和 Group Runners。

1\. Shared Runners

2\. Specific Runners

3\. Group Runners

**1.2.3 在 GitLab CI 中使用 docker**

在部署到阿里云时，需要在 GitLab CI/CD 中使用 docker 打镜像发布。可以参考 Building Docker images with GitLab CI/CD（https://docs.gitlab.com/ee/ci/docker/using\_docker\_build.html）

**1.2.4 在 GitLab CI/CD 中访问 Runner 宿主机目录**

我们使用的 Runner executor 是 Dokcer，在 Dokcer volumes 中配置需要访问的目录。

**2\. .gitlab-ci.yml 文件**

活动项目 .gitlab-ci.yml 文件如下，下面主要通过活动项目的 .gitlab-ci.yml 文件来介绍我们的实践过程、.gitlab-ci.yml 详细的用法，可参考 GitLab CI/CD Pipeline Configuration Reference 文档（https://docs.gitlab.com/ee/ci/yaml/README.html）

**2.1 .gitlab-ci.yml 文件介绍**

image 是执行 CI/CD 依赖的 Docker 基础镜像。镜像中有 Node、Yarn、Dalp（内部 rsync 工具）。

stages 中定义了我们的 pipeline 分为以下几个过程:

1\. 下载依赖阶段 pre\_build

2\. 构建阶段 build

3\. 发布阶段 deploy

stage 申明当前的阶段，在 stages 中使用

variables 用于定义变量

before\_script 执行 script 前的操作

script 当前 stage 需要执行的操作

changes 指定 stage 触发条件

refs 指定 stage 触发的分支

```
image: registry.huajiao.com/gitlab-ci/node-yarn:v1.4

variables:
  # $CI_PROJECT_PATH  ： 项目id,用于项目唯一区分本项目与其它项目
  # $CI_PROJECT_DIR   ： 本地项目路径
  # $PROCESS_PATH     ： 临时文件目录(包括日志和一些临时文件)
  NODE_MODULES_PATH: /runner-cache/frontend/$CI_PROJECT_PATH/$CI_BUILD_REF_NAME/node_modules

stages:
  - pre_build
  - build
  - deploy

下载依赖:
  before_script:
    # 无 node_modules 文件时，新建 node_modules 文件
    - /bin/bash ./ci/mkdir.sh $NODE_MODULES_PATH
    # 软链 node_modules 到宿主机
    - ln -s $NODE_MODULES_PATH .
    - cd webpack@next
  stage: pre_build
  script:
    - echo "yarn install"
    - yarn install  --network-timeout 60000
  only:
    changes:
      - webpack@next/package.json
    refs:
      - test
      - test-99
      - test-128
      - master
      - ci
      - feature/ci-test

构建:
  stage: build
  variables:
    CI_COMMIT_BEFORE_SHA_PATH: /mnt/gv0/gitlab-runner-cache/$CI_PROJECT_PATH
    CI_COMMIT_BEFORE_SHA_FILE_NAME: $CI_BUILD_REF_NAME.sh
    CI_COMMIT_BEFORE_SHA_FILE: /mnt/gv0/gitlab-runner-cache/$CI_PROJECT_PATH/$CI_BUILD_REF_NAME.sh
  before_script:
    # 建存此次 CI CI_COMMIT_SHA 的文件
    - /bin/bash ./ci/mkfile.sh $CI_COMMIT_BEFORE_SHA_PATH $CI_COMMIT_BEFORE_SHA_FILE_NAME
    # 软链 node_modules 到宿主机
    - ln -s $NODE_MODULES_PATH .
    - rm -rf php/share/*
    - cd webpack@next
  script:
    # 缓存上次ci
    - source $CI_COMMIT_BEFORE_SHA_FILE
    - echo "CI_COMMIT_BEFORE_SHA=$CI_COMMIT_SHA" > $CI_COMMIT_BEFORE_SHA_FILE
    - python3 ../ci/build.py   # 编译
    - /bin/bash ../ci/commit.sh   # 提交编译结果
  only:
    changes:
      - www_src/**/*
    refs:
      - test
      - test-99
      - test-128
      - master
      - ci


测试发布:
  stage: deploy
  variables:
    PROCESS_PATH: /mnt/gv0/gitlab-runner-cache/deploy/process/$CI_JOB_ID  # 目录不要换，用于日志服务器获取日志展示
  script:
    - mkdir $PROCESS_PATH # 建立发布临时路径，存放发布配置中间文件和结果日志用
    - dplt $CI_PROJECT_DIR/.deploy_test.yml $CI_PROJECT_PATH $CI_PROJECT_DIR/php/ $PROCESS_PATH
    # dplt 发布yml配置
    - echo "发布完成，错误日志查看http://new.admin.wolffy.qihoo.net/log?path="$PROCESS_PATH
    - echo `ls $PROCESS_PATH/*.log`
  only:
    changes:
      - php/**/*
    refs:
      - test
```

**2.2 下载依赖阶段（pre\_build stage）**

下载依赖的方案是：当 package.json 文件发生变化时，触发 pre\_build stage，执行 yarn install。下载的 node\_modules 放在宿主机下，执行时通过软链获取依赖。

**2.3 构建阶段（build stage）**

构建阶段，分为 3 部分

1\. diff 文件变化

2\. 前端 build

3\. commit build 后结果

**2.3.1 diff 文件变化**

每次 CI 时，将当前 CI commit SHA（CI\_COMMIT\_SHA 变量）存在文件中，存为 CI\_COMMIT\_BEFORE\_SHA 变量， diff 时，git diff 当前 CI 与上次 commit SHA 的变化。

**2.3.2 前端 build**

根据 git diff 的变化情况，确定本次需要打包的项目。

**2.3.3 commit 打包后生成的 HTML 文件**

在 GitLab CI/CD 提交代码时，使用 Git 凭证存储，提交打包后的 HTML 文件。Git 凭证存储细节可参考凭证存储文档(https://git-scm.com/book/zh/v2/Git-%E5%B7%A5%E5%85%B7-%E5%87%AD%E8%AF%81%E5%AD%98%E5%82%A8)

**2.4 发布阶段（deploy stage）**

发布阶段，使用内部的 rsync 工具 dplt 将打包后的 HTML 文件部署。dplt 可配置集群、机器列表。

**五、目前的问题及后续方向**

1\. 对于一个[持续集成](https://cloud.tencent.com/product/coding-ci?from=10680)，虽然实现了自动构建和发布，但缺少关键的测试环节。

2\. 借助于 GitLab CI/CD，我们实现了线上环境的一致，但本地开发环境和线上环境仍然不一致，可能存在本地没有问题，线上出现问题的情况。

3\. 从 SVN 切到 Git 开发后，分支管理策略，分支命名规范，commit message 规范，CodeReview 等，在团队中需要逐步规范。

**六、主要参考资料**

持续集成是什么？(http://www.ruanyifeng.com/blog/2015/09/continuous-integration.html)

什么是 CI/CD？(https://www.redhat.com/zh/topics/devops/what-is-ci-cd)

GitLab Docs(https://docs.gitlab.com/)

Introduction to CI/CD with GitLab(https://docs.gitlab.com/ee/ci/introduction/)

用 GitLab CI 进行持续集成(https://scarletsky.github.io/2016/07/29/use-gitlab-ci-for-continuous-integration/)

如何实现前端工程的持续集成与持续部署？(https://www.zhihu.com/question/60194439)

基于 GitLab CI 的前端工程CI/CD实践(https://github.com/giscafer/front-end-manual/issues/27)

文章分享自微信公众号：

![](https://open.weixin.qq.com/qr/code?username=gh_8dfd4ad91e5a)

本文参与 [腾讯云自媒体分享计划](https://cloud.tencent.com/developer/support-plan) ，欢迎热爱写作的你一起参与！

如有侵权，请联系

cloudcommunity@tencent.com

删除。