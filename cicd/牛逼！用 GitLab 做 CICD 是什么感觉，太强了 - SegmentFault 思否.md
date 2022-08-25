![图片](https://segmentfault.com/img/remote/1460000039667621 "图片")

GitLab [CI/CD](https://link.segmentfault.com/?enc=jVFs3vd080ryk58akmI6dw%3D%3D.XLGhtYRiGpD15Zexub9u96jagdbspXCacsBGjLX1PPb5%2FQnhW4LAefpJvsAMoJQ4XQPWdqsYcRYcWA%2BzKvj9oVjs8qLTQLZBYSjzkYc773t4kMhxXCFmzU47Tcj8gsd%2Bu%2B%2BTUkCtNF68u4d6%2BgxOFaOrbAUQucld0YYeb0NDz4oJvtgPrdurR6AdzVppn7z%2FoGOMaBA%2FTEtCrWCv0kTuXUEmlgn%2F80nL3zDJ4XbZ3dVj2XlgVwc7Cp4ijQtB3H%2BRTp%2FFFYeZ7TIZpSLobODbJwvZ%2FL8clutRb5z%2F%2Fn92u3nqg7pa7vZ1WFbo8LIf27de) 是一个内置在 [GitLab](https://link.segmentfault.com/?enc=LoEBTjoswUxpJ07p%2FEvJUQ%3D%3D.%2FVD6KvT2Gi2l8X6GNaqXVbwlFz%2BeLOx7kRTAmSDkOS0ATTO3u%2F6WzceaHgfUpBEJ%2BVRFVlo4vboyUpXMPH%2BW2g5uj1dQvb7JgprGw0i83m%2BlDVp9Y51QBINFWr5cDtmO0sbnSdFQWWqoKye%2Fvr9dBUSPQbmoeqr6Cs0xe%2Bn8DZcujKFmjHTvh5ELa2cDuJsXGEFcDyKrVuiCUTJdmg6HbHcwksM7bn3FDpUn1U%2FVHHchhRLijiBANMlOrijeGXY%2BeMOEIH5cnB2lkOSK6maRmrL9fVtMbCQWDX0UMfcyMkbpH4YvH7mL9%2BFVUFYHjj8z) 中的工具，用于通过持续方法进行软件开发：

-   Continuous Integration（CI）：[持续集成](https://link.segmentfault.com/?enc=%2BEziH1ukmBQvDK0T2aLv2Q%3D%3D.X3teOiAdqw8D6rdLnZ35WnwR98122J60cHMrRVc9PxaBpfNVMGsLLZGn3DZh3cPOBzFfZvO57VMieeSLF%2F6l1e8vMBBZQNui1ZEpPMx8I59pxs%2BgDsOxL3hyXfA4LRr5Sm4754uZ87MMh0GVtpaLw60MRJLNaTlOeseuNNQ35ji7Mcdqy%2FICBppZeamozs3Rlrfrw9Bl50%2FXW42MgyJ7q6Xv9GxktRHS2bnrJBt4xZ0QuAlOFnkV46PI%2BaW82NcFKiFP281ZpUihtFWlhyiHIS97fhUJA88U4RhK%2BvAjwt%2FpPemDFNhPFdE%2BNmmyH%2Bln)
-   Continuous Delivery（CD）：[持续交付](https://link.segmentfault.com/?enc=ZU3t23LJgfQdWCsh8u%2FnFg%3D%3D.fc76IU%2FemFOf193%2F2Qax3rUrXdySy9JcnhNAlSCLBipZp2IQQtg%2BGFakuyHNA8IqwCQI5n6sFEc6tULOkjZPX2iWZKVJpvhyQFOv455fVwuqUbfdk%2B8eKP6WN0W1wqG81sr%2B0RxNyYs1OojvvIcEe16Tq6%2FLd39Igt9qHCRFznAE9JIpspqo27oRuw83q6gPwcQ8PxPyAqcKwhJjv788a6RbrKjRD06JLqAXxyPTX4RkD2eIBtZhLgNrezKN04D3UKc4CYib6wMOzjhkou86mtqBaviWsxDvAMuKkYOlP0d8FgafH5tlL6xDo4wrm1Qz)
-   Continuous Deployment（CD）：[持续部署](https://link.segmentfault.com/?enc=Gvz6TcKXoOYsS24D7xRjMA%3D%3D.qXEG82XkfbnfPHK1uuCdaH8V64SxoQmNq6%2FHp0VyTvdQk7hFpWDd9O9QyFNRLkP2UU1cC3aZaOvzgLKo6PVnV0CPRcz6uUeS6kNTUsc3ebtOyn1grXTh2GH08A3lbeqMmac%2FhKMFsmcAHPKt75rYsHnmWauJcXA4qgklKfQeFbWGHSPlB04zYLV93%2BBEQozmf0PnI8GPwDBrqPfxLj8zXrMtF27oMY5iU7d%2FCUkHA%2FGM99oFpqyb7mVVq4iE%2Flry%2FY1z0SP7zlPAEc8sjyXsQpI4m8LuWn8IOwPk9VeISgpojhS2O5LrzZ4TKQTkX5Oo)

持续集成的工作原理是将小的代码块推送到 Git 仓库中托管的应用程序代码库中，并且每次推送时，都要运行一系列脚本来构建、测试和验证代码更改，然后再将其合并到主分支中。

持续交付和部署相当于更进一步的 CI，可以在每次推送到仓库默认分支的同时将应用程序部署到生产环境。

这些方法使得可以在开发周期的早期发现 bugs 和 errors，从而确保部署到生产环境的所有代码都符合为应用程序建立的代码标准。

GitLab CI/CD 由一个名为 .gitlab-ci.yml 的文件进行配置，改文件位于仓库的根目录下。文件中指定的脚本由 GitLab Runner 执行。

## GitLab CI/CD 介绍

软件开发的持续方法基于自动执行脚本，以最大程度地减少在开发应用程序时引入错误的机会。从开发新代码到部署新代码，他们几乎不需要人工干预，甚至根本不需要干预。

它涉及到在每次小的迭代中就不断地构建、测试和部署代码更改，从而减少了基于已经存在 bug 或失败的先前版本开发新代码的机会。

Continuous Integration（持续集成），假设一个应用程序，其代码存储在 GitLab 的 Git 仓库中。开发人员每天都要多次推送代码更改。对于每次向仓库的推送，你都可以创建一组脚本来自动构建和测试你的应用程序，从而减少了向应用程序引入错误的机会。这种做法称为持续集成，对于提交给应用程序（甚至是开发分支）的每项更改，它都会自动连续进行构建和测试，以确保所引入的更改通过你为应用程序建立的所有测试，准则和代码合规性标准。

Continuous Delivery（持续交付），持续交付是超越持续集成的更进一步的操作。应用程序不仅会在推送到代码库的每次代码更改时进行构建和测试，而且，尽管部署是手动触发的，但作为一个附加步骤，它也可以连续部署。此方法可确保自动检查代码，但需要人工干预才能从策略上手动触发以必输此次变更。

Continuous Deployment（持续部署），与持续交付类似，但不同之处在于，你无需将其手动部署，而是将其设置为自动部署。完全不需要人工干预即可部署你的应用程序。

## GitLab CI/CD 是如何工作的

为了使用GitLab CI/CD，你需要一个托管在 [GitLab](https://link.segmentfault.com/?enc=1MfsrDX6Md7s8D7jZ6KwzA%3D%3D.JHsqj6vT0jlNC3rCgaB73q8zQaXGC3ti4BUyxCZpJ5JO5MvJ9esvmxqd3QEycZzwbC1ss3TmlucWHTsrN0v1nadKWi5bYBF9TzfrlodkS3q8NiSRFIOUzDsfLpM4x2Yn4X8FhI9rpScgC8qy0Xmk%2BJPA8GCSAYJIG1kIVaHnZGlKlZwuPQ98055sGHif%2FztzsSO%2BOpKO3%2FE0BtNxaLysC5VVTYS4k4p2P8KJKRVIZgPp0nxYRx0swDfmjUXBgqVPdJShDd3xhZJmFoXSZnEsyYIOsC807GjOLh0sb47OCRzL3M7uPbj3NtQiVYUWevWF) 上的应用程序代码库，并且在根目录中的 .gitlab-ci.yml 文件中指定构建、测试和部署的脚本。

在这个文件中，你可以定义要运行的脚本，定义包含的依赖项，选择要按顺序运行的命令和要并行运行的命令，定义要在何处部署应用程序，以及指定是否 要自动运行脚本或手动触发脚本。

为了可视化处理过程，假设添加到配置文件中的所有脚本与在计算机的终端上运行的命令相同。

一旦你已经添加了.gitlab-ci.yml到仓库中，GitLab 将检测到该文件，并使用名为 GitLab Runner 的工具运行你的脚本。该工具的操作与终端类似。

这些脚本被分组到 jobs，它们共同组成一个 Pipeline。一个最简单的 .gitlab-ci.yml 文件可能是这样的：

```
before_script:   
  - apt-get install rubygems ruby-dev -y   
  
run-test:   
  script:   
    - ruby 
```

before\_script 属性将在运行任何内容之前为你的应用安装依赖，一个名为 run-test 的 job（作业）将打印当前系统的 Ruby 版本。二者共同构成了在每次推送到仓库的任何分支时都会被触发的 Pipeline（管道）。

GitLab CI/CD 不仅可以执行你设置的 job，还可以显示执行期间发生的情况，正如你在终端看到的那样：

![image.png](https://segmentfault.com/img/bVcQBwc "image.png")

为你的应用创建策略，[GitLab](https://link.segmentfault.com/?enc=uXJGFAn7g%2B5UDwdx6Usvaw%3D%3D.952s4T8IRQ5TO1aN3T5piOMU3op%2Bb%2FqthVWfnolqtHZdvHPQHVUGRRa3D%2Fh7td4YccJcRYZR8n4fwDX6RMerNgHL3MovmyfTD47EWrJSjCkgPr4id1yNKg1%2F7Myu3XM2lNtdVMMT3igVhrDuTHRDNvYIvHceQgyfofrRoL3IKjstykJlTqfryb6KzpN5phS56FsXR0pVQ7m7cgQsfyDLieggqROIds2jIw%2Bu%2FA1fT8TtMkodKN8Wa2lG6swU2mRY1CiaW2HuQ9U%2BdaWJnuC5hFB8JckiYllOwPiek5hL85vqwe1SL0mfMZKsjK8nYb0b) 会根据你的定义来运行 Pipeline。你的管道状态也会由 GitLab 显示：

![image.png](https://segmentfault.com/img/bVcQBwb "image.png")

最后，如果出现任何问题，可以轻松地回滚所有更改：

![image.png](https://segmentfault.com/img/bVcQBv9 "image.png")

## 基本 CI/CD 工作流程

一旦你将提交推送到远程仓库的分支上，那么你为该项目设置的 CI/CD 管道将会被触发。GitLab CI/CD 通过这样做：

-   运行自动化脚本（串行或并行） 代码Review并获得批准
-   构建并测试你的应用
-   就像在你本机中看到的那样，使用 Review Apps 预览每个合并请求的更改
-   代码 Review 并获得批准
-   合并 feature 分支到默认分支，同时自动将此次更改部署到生产环境
-   如果出现问题，可以轻松回滚

通过 GitLab UI 所有的步骤都是可视化的 。

![图片](https://segmentfault.com/img/remote/1460000039667612 "图片")

## 深入了解CI/CD基本工作流程

如果我们深入研究基本工作流程，则可以在 DevOps 生命周期的每个阶段看到 GitLab 中可用的功能，如下图所示：

Verify：

-   通过持续集成自动构建和测试你的应用程序
-   使用 GitLab 代码质量（GitLab Code Quality）分析你的源代码质量
-   通过浏览器性能测试（Browser Performance Testing）确定代码更改对性能的影响
-   执行一系列测试，比如 Container Scanning，Dependency Scanning，JUnit tests
-   用 Review Apps 部署更改，以预览每个分支上的应用程序更改

Package：

-   用 Container Registry 存储 Docker 镜像
-   用 NPM Registry 存储 NPM 包
-   用 Maven Repository 存储 Maven artifacts
-   用 Conan Repository 存储 Conan 包

Release：

-   持续部署，自动将你的应用程序部署到生产环境
-   持续交付，手动点击以将你的应用程序部署到生产环境
-   用 GitLab Pages 部署静态网站
-   仅将功能部署到一个 Pod 上，并让一定比例的用户群通过 Canary Deployments 访问临时部署的功能（PS：即灰度发布）
-   在 Feature Flags 之后部署功能
-   用 GitLab Releases 将发布说明添加到任意 Git tag
-   使用 Deploy Boards 查看在 Kubernetes 上运行的每个 CI 环境的当前运行状况和状态
-   使用 Auto Deploy 将应用程序部署到 Kubernetes 集群中的生产环境

使用 GitLab CI/CD，还可以：

-   通过 Auto DevOps 轻松设置应用的整个生命周期
-   将应用程序部署到不同的环境
-   安装你自己的 GitLab Runner
-   Schedule pipelines
-   使用安全测试报告（Security Test reports）检查应用程序漏洞

## GitLab CI/CD 快速开始

.gitlab-ci.yml 文件告诉 GitLab Runner 要做什么。一个简单的管道通常包括三个阶段：build、test、deploy，管道在 CI/CD > Pipelines 页面。

###### 创建一个 .gitlab-ci.yml 文件

通过配置 .gitlab-ci.yml 文件来告诉 CI 要对你的项目做什么。它位于仓库的根目录下。

仓库一旦收到任何推送，GitLab 将立即查找 .gitlab-ci.yml 文件，并根据文件的内容在 Runner 上启动作业。

下面是一个 Ruby 项目配置例子：

```
image: "ruby:2.5"  
  
 before_script:  
   - apt-get update -qq && apt-get install -y -qq sqlite3 libsqlite3-dev nodejs  
   - ruby -v  
   - which ruby  
   - gem install bundler --no-document  
   - bundle install --jobs $(nproc)  "${FLAGS[@]}"  
   
 rspec:  
   script:  
     - bundle exec rspec  
  
 rubocop:  
   script:  
     - bundle exec rubocop

```

上面的例子中，定义里两个作业，分别是 rspec 和 rubocop，在每个作业开始执行前，要先执行 before\_script 下的命令。

###### 推送 .gitlab-ci.yml 到 GitLab

```
git add .gitlab-ci.yml  
git commit -m "Add .gitlab-ci.yml"   
git push origin master

```

###### 配置一个 Runner

在 GitLab 中，Runner 运行你定义在 .gitlab-ci.yml 中的作业（job）。

一个 Runner 可以是一个虚拟机、物理机、Docker 容器，或者一个容器集群。

GitLab 与 Runner 之间通过 API 进行通信，因此只需要 Runner 所在的机器有网络并且可以访问 GitLab 服务器即可。

你可以去 Settings ➔ CI/CD 看是否已经有 Runner 关联到你的项目，设置 Runner 简单又直接。

![图片](https://segmentfault.com/img/remote/1460000039667619 "图片")

###### 查看 Pipeline 和 jobs 的状态

在成功配置 Runner 以后，你应该可以看到你最近的提交的状态。

![image.png](https://segmentfault.com/img/bVcQBv7 "image.png")

为了查看所有 jobs，你可以去 Pipelines ➔ Jobs 页面。

![image.png](https://segmentfault.com/img/bVcQBv6 "image.png")

通过点击作业的状态，你可以看到作业运行的日志。

![image.png](https://segmentfault.com/img/bVcQBv4 "image.png")

回顾一下：

-   首先，定义 .gitlab-ci.yml 文件。在这个文件中就定义了要执行的 job 和命令
-   接着，将文件推送至远程仓库
-   最后，配置 Runner，用于运行 job

## Auto DevOps

Auto DevOps 提供了预定义的 CI/CD 配置，使你可以自动检测，构建，测试，部署和监视应用程序。借助 CI/CD 最佳实践和工具，Auto DevOps 旨在简化成熟和现代软件开发生命周期的设置和执行。

借助 Auto DevOps，软件开发过程的设置变得更加容易，因为每个项目都可以使用最少的配置来完成从验证到监视的完整工作流程。只需推送你的代码，GitLab 就会处理其他所有事情。这使得启动新项目更加容易，并使整个公司的应用程序设置方式保持一致。

下面这个例子展示了如何使用 Auto DevOps 将 GitLab.com 上托管的项目部署到 Google Kubernetes Engine。

示例中会使用 GitLab 原生的 Kubernetes 集成，因此不需要再单独手动创建 Kubernetes 集群。

本例将创建并部署一个从 GitLab 模板创建的应用。

###### 从 GitLab 模板创建项目

在创建 Kubernetes 集群并将其连接到 GitLab 项目之前，你需要一个 Google Cloud Platform 帐户。

下面使用 GitLab 的项目模板来创建一个新项目。

![image.png](https://segmentfault.com/img/bVcQBv0 "image.png")

给项目起一个名字，并确保它是公有的。

![image.png](https://segmentfault.com/img/bVcQBvZ "image.png")

###### 从 GitLab 模板创建 Kubernetes 集群

点击 Add Kubernetes cluster 按钮，或者 Operations > Kubernetes。

![图片](https://segmentfault.com/img/remote/1460000039667618 "图片")

![图片](https://segmentfault.com/img/remote/1460000039667614 "图片")

![图片](https://segmentfault.com/img/remote/1460000039667620 "图片")

安装 Helm，Ingress 和 Prometheus。

![图片](https://segmentfault.com/img/remote/1460000039667615 "图片")

###### 启用 Auto DevOps（可选）

Auto DevOps 默认是启用的。

导航栏 Settings > CI/CD > Auto DevOps。

勾选 Default to Auto DevOps pipeline。

最后选择部署策略。

![image.png](https://segmentfault.com/img/bVcQBvX "image.png")

一旦你已经完成了以上所有的操作，那么一个新的 Pipeline 将会被自动创建。为了查看 Pipeline，可以去 CI/CD > Pipelines。

![图片](https://segmentfault.com/img/remote/1460000039667616 "图片")

## 部署应用

到目前为止，你应该看到管道正在运行，但是它到底在运行什么呢？

管道内部分为4个阶段，我们可以查看每个阶段有几个作业在运行，如下图：

构建 -> 测试 -> 部署 -> 性能测试

![图片](https://segmentfault.com/img/remote/1460000039667613 "图片")

现在，应用已经成功部署，让我们通过浏览器查看。

首先，导航到 Operations > Environments。

![image.png](https://segmentfault.com/img/bVcQBvV "image.png")

在 Environments 中，可以看到部署的应用的详细信息。在最右边有三个按钮，我们依次来看一下：

第一个图标将打开在生产环境中部署的应用程序的 URL。这是一个非常简单的页面，但重要的是它可以正常工作！

紧挨着第二个是一个带小图像的图标，Prometheus 将在其中收集有关 Kubernetes 集群以及应用程序如何影响它的数据（在内存/ CPU使用率，延迟等方面）。

![图片](https://segmentfault.com/img/remote/1460000039667617 "图片")

第三个图标是Web终端，它将在运行应用程序的容器内打开终端会话。

## Examples

使用 GitLab CI/CD 部署一个 Spring Boot 应用。

示例 .gitlab-ci.yml

```
image: java:8  
   
 stages:  
   - build  
   - deploy  
   
 before_script:  
   - chmod +x mvnw  
   
 build:  
   stage: build  
   script: ./mvnw package  
   artifacts:  
     paths:  
       - target/demo-0.0.1-SNAPSHOT.jar  
   
 production:  
   stage: deploy  
   script:  
   - curl --location "https://cli.run.pivotal.io/stable?release=linux64-binary&source=github" | tar zx  
   - ./cf login -u $CF_USERNAME -p $CF_PASSWORD -a api.run.pivotal.io  
   - ./cf push  
   only:  
   - master

```

> _原文链接：[https://www.cnblogs.com/cjsbl...](https://link.segmentfault.com/?enc=iHB9kn4RYQxkF1%2Bk7H%2BJUg%3D%3D.EuyeDzhw1oL5EdIhUyEqqo0l0t8aLBgrTYrmQJVhczk%2BXoraKb6R%2BJEes4D8%2BBJD)_

![image](https://segmentfault.com/img/bVbLc6v "image")