![在这里插入图片描述](https://img-blog.csdnimg.cn/9e7aeb8c2aad4e3eb29b26bba8dfe0e2.png)  
创建一家成功的软件公司需要什么？交付有价值的软件并快速交付的能力。我们如何保证这种高速服务？持续交付 (CD) 流程，由完善的持续集成 (CI) 机制支持，以提供完美交付，尤其是当平台组件的数量和依赖性增加时。

这张图片完美地总结了良性 CI/CD 循环，任何 [DevOps](https://so.csdn.net/so/search?q=DevOps&spm=1001.2101.3001.7020) 都应该将其贴在办公桌上：

![图片](https://img-blog.csdnimg.cn/6bd1316de1b345ae84e20ac264e23b1f.png)

在本文中，我们将关注循环的左侧，即产品从代码到测试的过程。

使用源代码时，git 是唯一的选择。事实上，在 BOOM，我们使用来管理代码[生命周期](https://so.csdn.net/so/search?q=%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F&spm=1001.2101.3001.7020)（但 git 选项还包括 Gitea 或 Bitbucket）。每个项目都有自己的存储库，可以由具有不同角色的各种团队成员访问。我们使用“开发”分支构建临时版本，使用主分支构建生产版本。

到目前为止，一切都很好。但是应该如何管理对 git 存储库执行的操作（例如拉取请求和合并）？如何在各种环境中以受控的方式部署代码呢？

答案是CI/CD 工具。

在 BOOM，一开始，我们将 Github Actions 用于 CI，将 Ansible/AWX 用于 CD。当涉及的软件组件很少时，此解决方案有效，但一旦您的路线图在数量和依赖性方面指向分布式软件模型，它就会变得有限。但随着时间的推移，编写库（例如日志库）或包（例如反应组件库）具有多个软件组件的需求变得更加紧迫，需要对整个生态系统进行维护和更有效的管理。

## 选择

在我过去的生活中，我对 [Jenkins](https://so.csdn.net/so/search?q=Jenkins&spm=1001.2101.3001.7020) 有过很深的体验，无论它的优点和缺点。但在 BOOM，我们充满好奇，渴望尝试新技术，看看它们是否符合我们的需求。因此，我们决定与工程团队一起评估和尝试各种解决方案，包括一些 SaaS，其中考虑了以下方面：

在我们测试了许多工具（CircleCI、TravisCI、TeamCity、Bamboo）的评估期之后，我们决定将Drone CI (https://www.drone.io) 作为我们 CI/CD 的核心部分。

Drones 为我们提供了我们所需要的一切，特别是：

-   它是开源的，由一个庞大的社区开发，可能有额外的开发参与；
-   易于安装和维护；
-   它是基于 Docker 的，一切都在容器上运行；
-   原生 Github、Gitlab、Bitbucket（和许多其他）集成；
-   采用基于 yaml 的配置，采用管道即代码原则；
-   它易于扩展（并且在主要云提供商上具有自动扩展功能）；
-   它包括许多由社区维护的工作插件，编写临时插件或扩展并不复杂；
-   开箱即用的 secret 管理（但也可以使用外部系统）；

## Drones 基础

Drones 的基本概念与其他 CI 工具非常相似。在 git 存储库上执行的任何操作都会通过 webhook Drone 触发。如果为特定存储库定义了管道（例如存储库根目录中存在 .drone.yml 文件），Drone 将对其进行分析并执行请求的操作。该决定是通过以下触发器定义做出的：

![图片](https://img-blog.csdnimg.cn/baaf17fe2b614095b755f421b1ed9dad.png)

在这个特定场景中，当且仅当目标分支是“develop”或“master”，并且事件是“pull\_request”或“push”时，管道才会运行。

每个管道都是使用一系列步骤构建的，每个步骤都用语法描述，例如：

![图片](https://img-blog.csdnimg.cn/1d0d96cd5ac446e18fef80a59b94ca65.png)

这很容易阅读。使用镜像 maven:3.6.3-jdk-11 我们执行 mvn clean 和 mvn install。以下无需解释。

![图片](https://img-blog.csdnimg.cn/a6595dfbd5324ba1bf6ee7754f2ccd4f.png)

![图片](https://img-blog.csdnimg.cn/25686277871e4c94a42bd2113ce8663a.png)

但是这些动作是在哪里执行的呢？源代码在哪里？正如我们在开始时所说，在 git 存储库上执行的定义明确的操作会通过 webhook 触发 Drone。Drone 负责克隆 git 存储库内容，与所有容器共享它，为每个容器安装一个特定路径（/drone/src），并在那里设置一个主容器。因此，在这个文件夹中添加文件可以在一个阶段完成，稍后在另一个阶段找到相同的文件，例如前面的 mvn 命令的构建结果可以用于执行单元测试：

![图片](https://img-blog.csdnimg.cn/5e08fffa8ebb494a8ebfcc667a9b8083.png)  
也许另一个可用于执行集成测试：

![图片](https://img-blog.csdnimg.cn/50b9284b3cc04d8faef3fbbd5e19ff56.png)  
如上例所示，我们使用简单的 docker 容器来执行各种步骤，其中大部分是标准容器。也可以通过添加新步骤来构建复杂的管道，直到达到预期的结果。

## Drones 服务

Drone 的强大功能之一是服务的概念。有时，执行特定任务（例如：集成测试）需要支持服务，例如 redis 实例或 postgres 实例。任何使用 SaaS 服务的人都需要使用 docker-in-docker (dind) 功能。使用 Drone，您只需定义一个服务

![图片](https://img-blog.csdnimg.cn/b5580c93a3be45c8bb242f2d05c53388.png)

![图片](https://img-blog.csdnimg.cn/7e7e5a1398a44e668e004482531e8a24.png)

Drone 将负责启动所需的 postgres 实例，然后在管道结束后将其杀死。接下来需要做什么？只需指示测试步骤使用这个 postgres 实例。

![图片](https://img-blog.csdnimg.cn/78264dec52d04716b1af6c872d150113.png)

## Drones 插件

如果没有可用的插件满足您的需求，您可以编写自己的插件。但是什么是 Drones 插件？很简单：它是一个容器运行代码！尽管 Go 是编写插件的首选语言，但也可以使用另一种语言。

我们来看看这一步：

![图片](https://img-blog.csdnimg.cn/aa5e795769b543edb85d2fafd1f1d9e8.png)  
并假设您将标签为 1.1.0 的容器 my-plugin 推送到首选镜像存储库中。

执行此步骤时，Drone 将下载您的插件并运行在定义的 Dockerfile 中找到的内容

![图片](https://img-blog.csdnimg.cn/efc014be9dc348a192b49c6a4b5e49e3.png)  
但是在步骤中定义的值上设置了两个环境变量，称为 DRONE\_FOO 和 DRONE\_BAR。当然，这对于简单的插件来说效果很好，但是当它们更复杂时，最好使用drone-plugin-starter\[1\]并用 Go 编写它。

## 测试和测试报告

让我们回到管道中的测试阶段。如前所述，可以为单元和集成测试添加测试步骤。但是同样的策略也可以应用于添加执行其他类型测试的步骤，例如 cypress 测试、postman 测试等。为这些场景编写步骤是再次启动一个合适的容器并在其中“运行”命令。但是测试报告呢？与 Jenkins 不同，后者使用一个合适的插件将测试结果附加到运行的管道并通过 Jenkins UI 访问它，Drone 只是一个管道执行器。它提供了一个不错的 UI，但它提供了与构建严格相关的信息，仅此而已。那么如何收集测试结果并将其提供给工程团队呢？

我们找到的解决方案是一个名为 Allure Docker Service\[2\] 的开源项目，它提供了一种基于项目存储和组织测试结果的方法。它由 API 层（负责管理内容摄取和管理）和允许轻松直观地浏览报告的 UI 组成。它可以接受各种格式的报告（junit、testng、allure 等），并提供每个项目的趋势视图和每个运行和测试用例的详细视图。执行以下任务很有用：

-   在特定容器中运行各种测试并将测试结果写入共享文件系统；
-   使用内部开发的 Drones 插件，通过 API 将报告发送到我们的 allure-service 实例。

换一种说法，

-   Drones 执行测试
-   Drone 将测试结果发送到 Allure Docker Service
-   通过访问 Allure Docker Service 提供的 Web GUI，工程团队可以使用测试。

例如，在 cypress 测试的具体情况下，这是我们在管道中使用的代码片段

![图片](https://img-blog.csdnimg.cn/845a3f7f5066423ab200496c927d3357.png)  
第一步运行 cypress 测试并将结果以 allure 原生格式存储在 /drone/src/cypress-results/allure 下，而第二步将结果发送到我们系统上的 allure-service。

这似乎是一种解决方法，可以弥补 Drone 只是一个管道执行器这一事实，但根据我的经验，最好的操作方式是让每个平台组件负责一项任务。大型应用程序（例如 Jenkins）在实施更改时可能会出现所有问题都崩溃的问题。同时，松散耦合的组件使得改变一个元素而不改变其他一切成为可能。

## 建筑工件

CI 管道的最终结果应该是可以在任何环境（暂存、预生产、生产等）中使用的工件。目前，我们的平台\[3\]有三种神器：

Docker 镜像存储在 ECR 上，而我们使用 Nexus 存储库管理器 OSS 来存储 npm 包和 java 库。Drone 可以很容易地创建这些工件并将它们推送到适当的位置。例如，在处理 docker 镜像时，使用以下步骤就绰绰有余了：

![图片](https://img-blog.csdnimg.cn/98cc06f5602a44d2b7e8452554b0d479.png)  
因此，将使用 pom.xml 中的版本将新版本的镜像推送到您的 ECR 上。

在 本文中，我们描述了为什么选择 Drone 作为我们的 CD，以及我们如何将它与其他工具一起使用，为我们的工程团队提供一流的体验。过程非常有趣，并不总是那么容易，但我们能够克服各种问题并利用我们建立的生态系统。

![在这里插入图片描述](https://img-blog.csdnimg.cn/ce2cd9f933e9413ba1e2e2fcfe27c9a9.png#pic_center)

在这里还是要推荐下我自己建的Python学习Q群: **746506216**，群里都是学Python的，如果你想学或者正在学习Python ，欢迎你加入，大家都是软件开发党，不定期分享干货（只有Python软件开发相关的），包括我自己整理的一份2022最新的Python进阶资料和零基础教学，欢迎进阶中和对Python感兴趣的小伙伴加入！