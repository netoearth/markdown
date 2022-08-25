DevOps漫谈之二，谈一谈在PHP项目如何应用CI/CD提升开发效率和规范开发与发布以及运维、运营之间的平衡。既然是漫谈就无法做到完全的无误，各位看官权当一种思路，抛砖引玉了。点击最下方关键词可以查找该系列的文章。

DevOps是一种方法论，是一组过程、方法与系统的统称。那么DevOps在PHP中的应用其实就是这一整套方法论在PHP上的实现和流程化，最终达到在PHP项目上DevOps理论需实现的目标---促进开发、运维和测试（QA）多部门之间的沟通、协作与整合，打破传统开发和运营之间的壁垒和鸿沟。结合PHP项目的特点，以下做一些PHP项目中应用CI/CD的流程（Pipeline）分析。

## CI/CD碎碎恋

什么是CI/CD[系列一](https://blog.jjonline.cn/linux/238.html)文章已经咬文嚼字的介绍过，会意就好，这里再补充两张图。

![CI/CD](https://blog.jjonline.cn/Upload/image/201901/20190131164953.png "CI/CD")

上图分3个区块，即CI/CD的宏观流程，为了形象呈现这里的两个CD拆分开了，查找了不少资料发现在具体的DevOps实践中将两个CD合并成了1个步骤（节点或要点）来理解更为便捷。整个流程中一直有一个关键词`Automation`，也就是自动。约莫的拿这张图分析下CI/CD

### 1、左侧为CI（Continuous Integration）的内容

源码和版本控制(`SCM`)下开发者提交代码，并自动触发服务端的源码构建，CI服务器触发Build过程和运行测试（不局限与单元测试，各种代码质量保证的测试均包含）。似乎有点儿绕来绕去的还有专业名词，云里雾里的感jio，简单的描述即开发者提交代码到git或svn上，自动触发源码的构建（`Build`）、集成（`Integration`）及各种测试（`Test`）。这里的`SCM`即`Software Configuration Management`的简写，一直以来我以为SCM指的是`Source Code/Control Management`，其实也差不多啦，就是源码、版本控制工具，例如：`svn`、`git`都是SCM的一种（蔬菜是SCM，胡萝卜是git、白萝卜是svn来理解啦，是那么个意思就好）；这里的`Build`和`Integration`不同的编程语言里可以理解成各自不同的动作，传统的C/C++项目乃至java项目均有Build过程，大致理解是那个意思即可，就是从源码构造成二进制程序或源码与二进制文件之间的中间形式文件的过程，PHP项目是不需要Build过程的，不过可以借用Build这种概念做一些准备性的“构建”动作。

### 2、中间CD持续交付（Continuous Delivery）的内容

CI之后`Release`（释出、推出、生成）可发布的软件或包，交付给客户或将可执行程序`deploy`到各种环境中，这里的deploy很让人迷惑，若要咬文嚼字的话理解成“部署”是不够精确的，若真要较留一段英文解释吧，这个靠悟性了。

```
To deploy (from the French deployer) is "to spread out or arrange strategically." Long used in the context of military strategy, it has now gained currency in information technology. In its IT context, deployment encompasses all the processes involved in getting new software or hardware up and running properly in its environment, including installation, configuration, running, testing, and making necessary changes. The word implementation is sometimes used to mean the same thing.-----谷歌的半吊子翻译如下：部署（来自法语中的deployer）是“分散或策略性地安排”。 它长期用于军事战略，现在已经在信息技术方面获得了成功。 在IT领域中，部署包括在其环境中正常启动和运行新软件或硬件所涉及的所有过程，包括安装，配置，运行，测试和进行必要的更改。 实施这个词有时用来表示同样的事情。
```

### 3、右边CD持续部署（Continuous Deployment）的内容

这个就不难理解了，自动化的部署到预发、生产环境，或发布给公共开放的用户们使用。因为持续部署的概念中突出了部署到正式环境中，很多时候对持续交付的理解就成了持续交付给内部的非开发团队，譬如质量保障、QA等做内部测试用。所以很多时候说CI/CD的时候，里面的CD是不会做详细的进一步区分的，就是持续交付和持续部署两个词的含义。

另外一张图，里面的CD就没有再进一步区分了，看看就好。

![CI/CD](https://blog.jjonline.cn/Upload/image/201902/20190201120659.png "CI/CD")

## PHP用应用CI/CD的思考

CI/CD的内涵和外延太广泛了，各人的理解又是有各种差异和各自亮点的，不必纠结于咬文嚼字。

PHP的脚本语言的天然特性，使得开发者编码完毕无需Build编译的过程，文件复制拷贝到web目录下，请求一下就可以立即看到开发的结果。那么PHP这种便捷性之下，CI/CD还有存在的必要性么？

![Pipeline](https://blog.jjonline.cn/Upload/image/201902/20190201121331.png "Pipeline")

在CI/CD理论的实践中，都是作为一些连续的、自动的步骤来处理的。查阅相关英文资料时常会看到`Pipeline`这个词汇，直白的翻译就是管道，这个词在英文中还是一个动词属性--传递途径，不做多的分析，上图所示整个CI/CD实现的流程像不像一个自来水流动的管道？是那么个意思就好。

### 那么对于PHP项目如何应用CI/CD呢？

以下是个人的一些思考。

#### 1、代码的版本控制和运行环境统一

这个无需多言，许多团队开发的项目中经常遇到各种坑和麻烦，集中统一的代码版本管理工具大多都已使用上了。

代码管理基本上就是二选一：svn或git。当然还有其他类型的工具，大多时候可能就是git了，git本身就是分布式的，没有过多的权限和功能限定的话git本身的分布式就够了，当然还有有免费开源的功能强大的gitlab，如果不是私有项目github、gitee都是可以的。

运行环境统一这个，似乎很少有团队涉及到，因为各个开发者的系统可能是不一样的，也很难做到限定，不过条件允许的情况下统一开发、测试和生产各运行运行环境也是必要的。现在虚拟化、容器化技术也不断显现威力，合理利用这些技术实现运行时环境的统一并不是什么难事儿，就算是传统的安装方式，提供统一的安装脚本或统一的安装说明都是一种进步。对于PHP项目来说，统一的运行环境无非PHP版本和扩展以及配置项、MySQL版本和配置项、Web服务器的选择和配置等等。

#### 2、代码风格检查和规范和自动化单元测试（Unit Test）

PHP的脚本语言特性导致了编码风格的多样性和编码习惯的丰富多彩，对于编码风格哪种更为优雅其实没必要争论，这方面Go语言做的很好，直接官方提供统一的编码排版工具，大有“谁也别争，按我的要求来就好”的意味，也确实解决了不少没必要的争论和精力消耗。

PHP的编码规范有一个PHP-FIG组织提的PSR规范，本着聊胜于无的原则，团队统一遵循即可。也有各种实现了这些规范的自动排版和检查的工具如：[CodeSniffer](https://github.com/squizlabs/PHP_CodeSniffer)、[PHP-CS-Fixer](https://github.com/FriendsOfPHP/PHP-CS-Fixer)等

关于PHP里的单元测试，近几年似乎更多的是偏向于QA层面的，PHP里也是有发源于Junit的[phpunit](https://phpunit.de/)的，关于单元测试这块儿，因为不涉及业务的开发，仅仅是对所开发的代码的一种自动化按单元执行的测试代码的开发，还存在一些误解和作用性上面的空白。有一个新的岗位叫：测试开发工程师简称测开，据说比普通的码畜待遇要好很多。

#### 3、自动化部署至开发、测试环境进一步测试和验证

这一块有很多可用的方案和工具，[Jenkins](https://jenkins.io/)、[Travis-CI](https://travis-ci.org/)以及Gitlab提供的[CI/CD](https://about.gitlab.com/product/continuous-integration/)等等，核心在于选择哪一种工具并合理配置运行起来。这一块儿工具已经很多，而且大多都是完整或不仅仅实现了CI/CD功能的。选择团队最佳和最熟悉的工具即可，实现自动化的环境构建、部署和依赖测试等等。对于PHP项目来说，大多时候环境都是提前搭建好的，这块儿可能更多就是部署测试代码和自动化测试；当然大型分布式的从申请新的机器到机器环境搭建、代码部署和进入集群开始服务整个流程本身也是包含在CI/CD的理念里的，所以怎么用如何用其实还是具备很大的倾向性和喜好在里面的。

#### 4、可选的自动化部署至生产环境

这其实是传统开发过程中的统一部署和发布，这方面的工具Jenkins应用的最多，国人开发的[瓦力](https://walle-web.io/)部署系统也有不少团队使用，更多的大厂以定制开发的工具居多。

对于没有完整实践经验的开发团队而言，不可能完整的实现CI/CD的理念的，至少在PHP项目上当前是不可能的，所以自动部署至生产环境本文作为可选项。绝大多数时候部署至生产环境俗称“发版”，你见过哪家公司敢在最终线上部署这一步完整实践CI/CD理念的么？顶多用上一套部署系统，开发、测试、运维各方均没有问题的时候，手动去触发执行自动部署。所以说人是不可靠的，也是可靠的。

\-----

参考资料：

1、https://dzone.com/articles/learn-how-to-setup-a-[[cicd]]-pipeline-from-scratch

2、https://blog.docker.com/2018/02/ci-cd-with-docker-ee/

3、https://ithelp.ithome.com.tw/articles/10204538

4、https://gblogs.cisco.com/ch-tech/how-to-build-devops-cloud-agnostic-cicd-pipeline-prospect/

5、https://gblogs.cisco.com/ch-tech/cloud-agnostic-cicd-pipeline-for-devops-building-blocks/

6、https://gblogs.cisco.com/ch-tech/how-to-devops-cloud-agnostic-cicd-pipeline-process-steps/

7、http://www.zhujinhe.com/archives/31