（题图来自 [CNCF](https://landscape.cncf.io/category=continuous-integration-delivery&format=card-mode&grouping=category))

#### 介绍

本文记录了我试图搭建 CI/CD 工作流的经历。

我想用 Docker Swarm 部署，因为简单。  
kubenetes 暂不考虑，我就1个人，时间成本有点高暂时吃不下。  
顺带一说，CICD 说成大白话就是 "经常测试/经常部署"。没啥神秘或高大上的。

#### 这篇文对谁有用？有什么用？

对谁有用？同样在调查怎么搭建 CI/CD 工作流的程序员  
有什么用？参考这篇文章，节省你的时间

不在乎细节的人可以直接翻到末尾看 "结论"

___

#### 整个 CI/CD 流程其实是

第1步：本地开发时用 Docker 环境，确保开发,测试,部署，全部环境的镜像一致（本来用 Docker 的重点之一就是解决环境不一致问题）如果项目小，可能1个 Dockerfile 就行了。如果稍大一点需要 Dockerfile + docker-compose.yml 2个文件

第2步：构建 Docker image，用于后面的步骤（跑测试和部署）

第3步：跑测试（具体怎么测取决于语言,框架,具体业务）有问题就修问题，没问题就下一步

第4步：部署。测试既然没问题了就部署，拿着新版镜像让 Docker Swarm 或 Kubenetes 更新.

___

第一步没啥可说的我们直接看第二步  
第二步其实就是 docker build 和 docker push  
如果手动在自己电脑跑，当然简单  
用第三方工具自动跑比较麻烦，试过如下工具：

### 1\. Gitlab

需要写一个 `.gitlab-ci.yml` 文件。这个配置文件告诉 Gitlab 怎么跑任务。  
Gitlab 为了跑任务，需要一个叫 Runner 的东西，因为代码总是需要机器来跑的嘛，Runner 就负责跑。

我的 .gitlab-ci.yml 内容如下（先忽略掉，继续看文章，等看完这一整章如果你觉得你对 gitlab ci 比较了解，想看看我可能是哪里配置错了，再来看这个，对于99%的随意阅读读者可忽略这个代码部分）

```
# For this to work, set Gitlab Environment Variable:
# SSH_PRIVATE_KEY_FOR_ROOT_USER_AZURE

stages:
  - build
  - copy
  - run
  
variables:
  HOST: root@137.117.103.xx
  
build-and-push:
 stage: build
 image: docker:stable
 services:
    - docker:dind
 before_script:
    - export DOCKER_HOST="tcp://0.0.0.0:2375"
    - docker info
    - docker login -u [username] -p [password] registry.gitlab.com
 script:
    - docker build -f Dockerfile -t registry.gitlab.com/1c7/rails1 .
    - docker push registry.gitlab.com/1c7/rails1

copy-docker-compose:
  stage: copy
  image: ubuntu:18.04
  before_script:
    - 'which ssh-agent || ( apt-get update -y && apt-get install openssh-client -y )'
    - eval $(ssh-agent -s)
    - echo "${SSH_PRIVATE_KEY_FOR_ROOT_USER_AZURE}" | ssh-add -
    - mkdir -p ~/.ssh
    - echo -e "Host *\n\tStrictHostKeyChecking no\n\n" > ~/.ssh/config
  script:
    - scp docker-compose.yml $HOST:/var/www/docker-compose.yml

run-command-on-remote:
  stage: run
  image: ubuntu:18.04
  before_script:
    - 'which ssh-agent || ( apt-get update -y && apt-get install openssh-client -y )'
    - eval $(ssh-agent -s)
    - echo "${SSH_PRIVATE_KEY_FOR_ROOT_USER_AZURE}" | ssh-add -
    - mkdir -p ~/.ssh
    - echo -e "Host *\n\tStrictHostKeyChecking no\n\n" > ~/.ssh/config
  script:
    - ssh $HOST "mkdir -p /var/www/ && cd /var/www/ && ls && docker stack deploy -c docker-compose.yml rails1-cicd"

```

Runner 的配置位于 Settings -> CI/CD -> Runner。如下图  
![runner](https://1c7.me/content/images/2019/02/runner.jpg)

-   Share Runner：就是 Gitlab 提供的 Runner，能用是能用，  
    但是我没看到怎么配置缓存，所以每次 build image 都是重新来，起码3-5分钟。  
    （而且你可以看到说明里写的是：开源项目不限时间，私有项目限制 2000 CI minute）  
    那行，那就把 Share Runner 停掉，我们用 Specific Runner，做法是在自己的机器上按官方教程安装就是了。  
    （下图可以看到有2个类别）  
    ![12](https://1c7.me/content/images/2019/02/12.jpg)
    
-   Specific Runners：  
    两种安装方式，一种是自己装，一种是 Kubenetes
    

自己装：我不知道哪里配置错了，或者说是 Gitlab 本身的 bug，runner 的确是在运行，但是 CI/CD pipline 总是显示等待 runner pick up job。谷歌无果。时间有限就不折腾这个了。

Kubenetes：提供方其实是 Google Cloud Platform，不是 Gitlab 自己提供的。  
我因为有免费试用的 300 美金就任性的直接开了。实测的时候会有这样那样的问题。  
![error](https://1c7.me/content/images/2019/02/error.jpg)  
解决办法当然是调配置嘛。但是我实在很烦这些东西，先抛弃这个。不能花俩小时在这个破事上。

结论：Gitlab 如果配置文件写的好，Runner 自己弄（跑在自己的机器上）  
可以说是最便宜的方案（仅次于全手工的方案，免费）（如果你自己在开发机上 build image, test, push image，当然是免费的，1个人可以这样做，团队人多了这样做就不方便了)

### 2\. CodeFresh

方便是方便，第一次 build 是8分钟，第二次是 18 秒。说明用了缓存。  
但是这个价格嘛。除了免费，付费版的直接就蹦上了99美金一个月。而且还是 220 build/month 不是无限 build 或者一个更高一些的数字。  
![codefresh](https://1c7.me/content/images/2019/02/codefresh.jpg)

免费版 120 build/month。120次/30天=4次/天。

假设你只有 Master 分支需要部署，而且一天最多4次。免费版够用了。

但假设有 master, dev 分支，每次有变化了就 push 到 dev，然后跑测试。  
确保没问题了就自动 merge master 部署。感觉120次不够用。

如果你就1个人，然后测试都在本地做，就是说仅仅依靠 codefresh 监听 master 分支来打包部署用的镜像。而且一天少于4次，那么就是够用的。

### 3\. Buddy

Buddy 之前试过，操作和界面都很好，是个很赞的工具。  
![buddy1](https://1c7.me/content/images/2019/02/buddy1.jpg)

价格：免费版 120 exection。  
付费版最便宜的： 75美金无限 execution。

如果让我非要 CodeFresh 和 Buddy 二选一，显然选 Buddy 。  
因为75美金比99美金便宜。而且 Buddy 是 Unlimited Executions。

问题是75美金也好贵。

变通方案：  
CodeFresh + Buddy 搭配使用。两边加起来有240次执行。对于个人或者小团队够用了。  
缺点：麻烦

### 4\. Drone.io

这个 CI/CD 工具很赞，简单，好看，可以自己搭建（免费）  
之前的博客写过。我不用是因为文档太差，最新版是 1.0.0-rc1，文档还是 0.8。  
我折腾 .drone.yml 文件一直搞不对（去谷歌查，去问人，去试）非常烦。不想浪费时间。就不用了。

### 5\. Travis

![travis](https://1c7.me/content/images/2019/02/travis.jpg)  
前100个 build 免费，付费版最少69美金。

### 6\. Codeship

![codeship](https://1c7.me/content/images/2019/02/codeship.jpg)  
免费版 100个 build 一个月。  
付费版最便宜 49 美金。

值得一提的是分成了 Pro 和 Basic。区别是支不支持 Docker。  
![codeship-pro](https://1c7.me/content/images/2019/02/codeship-pro.jpg)

### 7\. Jenkins

太丑了+过于复杂，  
丑是因为这个工具很早。网上教程也很多。  
因为太丑不想用。

___

#### 结论

一共看了/试了这几个工具：

-   Gitlab
-   Codefresh
-   Buddy
-   Drone.io
-   Travis
-   Codeship
-   Jenkins

各有优缺点。  
想要免费就得麻烦一点：自己配 Gitlab 或者搭配使用 CodeFresh + Buddy 的免费限额。  
想要便利就得掏钱：75美金的 Buddy

本文蛋疼的点其实就是：独立个人/小团队 没钱又想弄 CI/CD 会有些麻烦，  
有钱就直接用自己想用的工具就行，文章里说的问题都不是问题。

对于10几个人的团队/有钱的团队。每个月的办公室租金/工资/税/杂项开支 花的钱多了去了。  
这99美刀（大概670人民币）才不在乎呢。

### 参考资料

CI/CD 工具：  
[https://github.com/ciandcd/awesome-ciandcd](https://github.com/ciandcd/awesome-ciandcd)  
这里面列的工具比这篇文章里多得多，我也试了一些，有的工具实在太辣鸡了所以我连提都不提.

值得提1-2句的：

-   Teamcity  
    界面还行，免费，但是要自己搭。  
    [https://www.jetbrains.com/teamcity/index.html](https://www.jetbrains.com/teamcity/index.html)
    
-   Bamboo  
    界面还行，30天免费试用，要自己搭  
    [https://www.atlassian.com/software/bamboo](https://www.atlassian.com/software/bamboo)
    
-   cloudbees  
    界面不错，这家公司貌似拥有 CodeShip？  
    看起来主要针对企业用户。有些复杂。有时间再试  
    [https://www.cloudbees.com/](https://www.cloudbees.com/)
    
-   shippable  
    界面凑合，3/10，不想用  
    [https://shippable.com](https://shippable.com/)
    
-   GoCD  
    界面凑合，4/10，首页没有 Pricing 但有个 Download。需要自己搭  
    [https://www.gocd.org/](https://www.gocd.org/)
    
-   Appveyor  
    [https://www.appveyor.com/](https://www.appveyor.com/)  
    开源项目免费，最便宜29美金（14天试用）
    
-   buildkite  
    [https://buildkite.com](https://buildkite.com/)  
    开源项目或是教学用途（学生/老师使用）免费。  
    最便宜的套餐是 15 per user per month。（14天试用）  
    ![buildkit](https://1c7.me/content/images/2019/02/buildkit.jpg)  
    ![bk3](https://1c7.me/content/images/2019/02/bk3.jpg)
    
-   CircleCI  
    [https://circleci.com/](https://circleci.com/)  
    这一家的广告打的很多，我近期（2018年2019年）看到的 Youtube 广告，  
    唯一和 CI/CD 相关的就只有这一家。  
    免费版的是 1,000 build minutes per month。1个 container。  
    2个 container 就直接50刀了  
    self-host 20刀一个用户，最少20个用户。  
    ![circle](https://1c7.me/content/images/2019/02/circle.jpg)  
    ![circle-price](https://1c7.me/content/images/2019/02/circle-price.jpg)  
    ![circle-price2](https://1c7.me/content/images/2019/02/circle-price2.jpg)
    

界面还成，5/10  
![circle-CI](https://1c7.me/content/images/2019/02/circle-CI.jpg)  
我不知道 1000 分钟我能用多久。改天有空再接着试试这个

-   FlowCI  
    [https://flow.ci/](https://flow.ci/)  
    官网是中文的，国人做的。首页界面不错，7/10。  
    功能点第一行：“开源免费”  
    现在2019年2月9号。最新版是2017年年底。而且版本号连1都没有到。  
    Github Star 712。  
    ![flow-ci](https://1c7.me/content/images/2019/02/flow-ci.jpg)  
    给人的感觉就是这个项目半死不活的。很可惜。  
    反正我是不会用的。出了问题都估计没人理的
    
-   ProboCI  
    [https://probo.ci/pricing/](https://probo.ci/pricing/)  
    前2个月免费。最便宜30美金一个月。  
    界面 4/10。首页大标题是 "Continuous Collaboration"  
    看完首页的感觉是这公司还活着么……  
    不想用
    
-   abstruse  
    首页没有 pricing。应该是免费的。  
    首页只提了 CI，没有 CD。  
    感觉可以试一下，但是期望值不会太高。首页非常精简就一个章节。  
    [https://bleenco.io/](https://bleenco.io/)  
    （没错，URL 和名字不匹配）
    
-   concourse  
    [https://concourse-ci.org/](https://concourse-ci.org/)  
    典型的开源项目界面，直接就是文档。  
    感觉不咋样，不想用。
    
-   coveralls  
    [https://coveralls.io/](https://coveralls.io/)  
    界面 3/10，开源项目才免费。我懒得看下去了直接关掉。
    
-   DeployBot  
    [https://deploybot.com/](https://deploybot.com/)  
    现在2019年2月，他们在弄前3个月0.99美金。第四个月开始收费。最便宜的套餐15美金。  
    界面 5/10，有兴趣试试。  
    ![dbot](https://1c7.me/content/images/2019/02/dbot.jpg)
    
-   Azure Devops  
    [https://visualstudio.microsoft.com/team-services/](https://visualstudio.microsoft.com/team-services/)  
    操作起来太让我困惑了，有一定学习成本。  
    ![az](https://1c7.me/content/images/2019/02/az.jpg)  
    ![az2](https://1c7.me/content/images/2019/02/az2.jpg)
    

### 文章

这类 CI/CD 工具的文章我当然也不是第一个写的。  
其他人写的有：

-   [https://github.com/ligurio/awesome-ci](https://github.com/ligurio/awesome-ci)
-   [https://github.com/ciandcd/Continuous-Integration-services/blob/master/continuous-integration-services-list.md](https://github.com/ciandcd/Continuous-Integration-services/blob/master/continuous-integration-services-list.md)
-   [https://en.wikipedia.org/wiki/Comparison\_of\_continuous\_integration\_software](https://en.wikipedia.org/wiki/Comparison_of_continuous_integration_software)

（其实这3个链接都来自前面那个 awesome-cicd 的库）  
话说里面混进去了一些 Code Review 和 Code Style 的工具。┑(￣Д ￣)┍ 和 CICD 毫无关系。

#### 其他

v2ex 的帖子  
[https://www.v2ex.com/t/366439](https://www.v2ex.com/t/366439)  
（大概看看就行，没啥大用）

### CaiCloud

话说在 v2ex 上还看到了这个：[https://caicloud.io/](https://caicloud.io/)  
菜cloud？  
之前只知道小玩家有：青云和灵雀云和 daocloud 和 leancloud。第一次听说这个。  
[https://caicloud.io/methods/cloud](https://caicloud.io/methods/cloud)  
有个"开发测试云"，应该就是 CICD 了  
其他的就是容器云了，kubenetes。

### semaphoreci

[https://semaphoreci.com/](https://semaphoreci.com/)  
这个收费模式和其他的比起来很独特，pay only for what you use.  
每一个月的前20美金免费的。  
$0.00025/sec  
有兴趣试试。  
![sem](https://1c7.me/content/images/2019/02/sem.jpg)

### Argo (kubenetes)

[https://argoproj.github.io/](https://argoproj.github.io/)  
大标题写着 "Get stuff done with Kubernetes"  
看来是不兼容 docker swarm 了，那我就懒得试了  
开源免费软件  
![argo](https://1c7.me/content/images/2019/02/argo.jpg)

### Brigade

[https://brigade.sh/](https://brigade.sh/)  
又是 k8s 专属工具  
![brigade](https://1c7.me/content/images/2019/02/brigade.jpg)

### Skycap

[https://www.cloud66.com/containers/skycap](https://www.cloud66.com/containers/skycap)  
14天免费试用  
最便宜的 Starter plan 是49美金一个月  
[https://www.cloud66.com/containers/skycap/pricing](https://www.cloud66.com/containers/skycap/pricing)  
![cloud66](https://1c7.me/content/images/2019/02/cloud66.jpg)  
他们家产品还挺多的

### Deployhub

[https://www.deployhub.com/](https://www.deployhub.com/)  
到处点了一下有点困惑，应该是开源免费的。  
不确定要不要试用看看  
![deployhub](https://1c7.me/content/images/2019/02/deployhub.jpg)

### Octopus Deploy

[https://octopus.com/trial](https://octopus.com/trial)  
不管是用云还是 self-host 都是30天试用。  
付费计划最便宜45美金一个月。  
![octo](https://1c7.me/content/images/2019/02/octo.jpg)

### Zuul

[https://zuul-ci.org/](https://zuul-ci.org/)  
开源免费。看起来是那种只有几个人在管的很小的项目，不敢用。  
![zuul](https://1c7.me/content/images/2019/02/zuul.jpg)

### Spinnaker

[https://www.spinnaker.io/](https://www.spinnaker.io/)  
大标题 "Continuous Delivery for Enterprise"  
看来是针对大企业用的。右上角没有 Pricing。要自己安装。  
![sp](https://1c7.me/content/images/2019/02/sp.jpg)

### Puppet

[https://puppet.com/products/puppet-pipelines-containers](https://puppet.com/products/puppet-pipelines-containers)  
官网看起来很“企业风”  
下面的介绍写可以 build container 和 deploy to k8s  
免费计划是 up to 5 project。  
注册要填好多东西，懒得试了  
![pupet](https://1c7.me/content/images/2019/02/pupet.jpg)

### Habitus

[https://www.habitus.io/](https://www.habitus.io/)  
开源项目，必须自己安装才能用，Github Star 909  
随便看了下文档也是那种自己写个 yml 文件配置流程的。  
试用比较费时。下次再试。  
![habi](https://1c7.me/content/images/2019/02/habi.jpg)

### Gitkube

[https://gitkube.sh/](https://gitkube.sh/)  
开源免费但是仅针对 k8s.  
![gitkube](https://1c7.me/content/images/2019/02/gitkube.jpg)

## 结论

因为我不可能自己去写 CI/CD 工具  
（大公司才有这样的人才和资源，比如 Airbnb 就是用自己的 in-house 工具, [资料来源1](https://careers.airbnb.com/positions/1391489/),  
"资源来源1"中的原话:

> You will contribute to our in-house CI/CD system, the largest distributed system at Airbnb

市面上所有能找到的 CI/CD 工具我都看过/试过了。

除去丑的不行的工具(Jenkins 等)。  
除去学习成本或时间成本高的工具（需要自己搭自己维护，而且大部分只有30天试用）  
除去价格贵的工具。（都挺贵的，起步价范围是50~100美金）  
除去看起来不靠谱没人用的工具（FlowCI 在2019年还没到 1.0版，而且最新 release 是2017年年底）  
除去14天试用/30天试用的工具  
除去只有开源项目才免费的工具。

我的目标是"尽可能省钱+用起来方便"，  
这类东西不应该占用我太多时间，精力应该放在写业务代码才是。而不是折腾这些东西。  
先把自己的产品搭建起来，等产品有收入了（有钱了），想用什么工具就用什么工具。  
但是在此之前，先省钱

所以我的方案是：

1.  优先用每个月免费限额重置的工具，比如 Codefresh 和 Buddy。
2.  等 CodeFresh/Buddy 的免费限额用干了，用 Gitlab CI/Drone.io

优点：省钱  
缺点：麻烦

小（鸡贼）技巧：如果有多个不互相依赖的项目（比如多个 Rails 项目），  
可以拆分到多个 Github account 里，  
然后用多个 Github account 来登录 Codefresh，  
这样一个项目就有 120次 build 一个月了。

Travis, Codeship, semaphoreci, CircleCI  
这几个目前还不够了解，当备用工具有空试试就行。

如果完全不缺钱（比如一个月有至少100美金的预算）：  
应该会用 Buddy？  
这个得试几个工具才知道，主流的这几个工具都是一个月100美金以下（最便宜的付费版）

#### 全文完

感谢阅读