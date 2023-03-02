#### 介绍

本文是我使用 Codefresh 做 CI/CD 工具的使用体验。  
本文适合想了解 CodeFresh 的读者。  
题图就是他们官网。  
[可以用我的 refer link 注册 Codefresh](https://g.codefresh.io/signup?ref=rkaDjY5gE)

### 技术栈

Ruby on Rails 5 + Docker Swarm

### 最终目标（我用 CodeFresh 到底想干什么）

我想把 Github + CodeFresh + Docker Swarm 弄一个工作流出来。  
每次 push 到 Github 就触发 CodeFresh build image + 跑测试，测试通过了就更新 Docker Swarm 把新版部署上去

Kubenetes(k8s) 过于复杂我暂时没时间学。

说简单点就是：我只需要写代码和推代码，更新的事情交给 CodeFresh+Docker Swarm 搞定。  
我不需要 ssh 到服务器上跑任何命令。全自动部署。

这里我用了 Ruby on Rails 5 的 sample app。  
因为只是试用而已。  
（就是直接跑 rails new 命令得到的新项目，用了 postgresql 做数据库）

#### 先讲价格

有免费版。（没有14天试用/30天试用的限制）。  
免费版 120 次 Builds/一个月。  
![cf-price](https://1c7.me/content/images/2019/02/cf-price.jpg)  
我不确定够不够我用。

#### 为什么用 Codefresh

我 [之前的文章](https://1c7.me/ci-cd/) 试用了市面上所有能找到的 CI/CD 工具。  
没有银弹，想简单方便就得付费（TravisCI, CircleCI, sem, CodeFresh, Buddy, Codeship)。想免费就得投入时间成本和学习成本加自建机器（Gitlab CI/Drone.io）

但好的地方是 CodeFresh, Buddy 有免费套餐。  
我现在没心情没时间折腾免费方案。所以先试用 CodeFresh。看看是否符合我的使用场景。够不够用。

## 章节

1.  镜像存储位置
2.  机器配置
3.  Trigger
4.  部署到 kubenetes
5.  临时测试环境
6.  查看免费限额（120 build/一个月）

可以直接跳到需要的章节扫读，并不是非得全部看完

#### 使用体验：简单方便

选择 Github 代码库之后，点点鼠标，下一步下一步，  
只要有合适的 Dockerfile 文件可以很快的把镜像打包出来。  
（我已经写好 Dockerfile 并且 commit 到了代码库里）

### 1\. 镜像存储位置

Docker image 构建之后总得有地方存吧？  
默认存在了 Codefresh。  
具体存储位置可以在 Acccount Setting 看到。点击右侧的 Config。  
![cf1](https://1c7.me/content/images/2019/02/cf1.jpg)

可以看到默认是 Codefresh 的 Container Registry，缩写 CFCR  
![cf2](https://1c7.me/content/images/2019/02/cf2.jpg)

可以加其他 Registry  
![cf3](https://1c7.me/content/images/2019/02/cf3.jpg)

之前 [写过文章](https://1c7.me/2019-1-31-china-free-private-docker-registry/) 。国内的阿里云，腾讯云，七牛云，UCloud，都有免费的私有 Container Registry。  
我觉得加上去最大的问题可能就是网速（构建完镜像之后推过去很慢）我没有试，读者可以自己尝试  
![cf4](https://1c7.me/content/images/2019/02/cf4.jpg)

### 2\. 机器配置

可以在 Runtime Enviroment 看到。  
![cf5](https://1c7.me/content/images/2019/02/cf5.jpg)  
免费套餐的 CPU 是2000m（我不知道什么意思）  
内存是4G。

### 3\. Trigger

Trigger 的意思就是如何触发 CodeFresh 的 CI/CD 执行。  
![trigger](https://1c7.me/content/images/2019/02/trigger.jpg)  
自动触发: 有 API 触发， Git 触发，Registry 触发，  
![trigger2](https://1c7.me/content/images/2019/02/trigger2.jpg)

手动触发: 可以点这2个地方  
在 Repo 这里直接点 build  
![manual](https://1c7.me/content/images/2019/02/manual.jpg)  
或者在 pipeline 这里点 run  
![manual2](https://1c7.me/content/images/2019/02/manual2.jpg)

### 4\. 部署到 kubenetes

[根据文档](https://codefresh.io/docs/docs/getting-started/deployment-to-kubernetes-quick-start-guide/) 可以部署到 k8s。  
![k8s2](https://1c7.me/content/images/2019/02/k8s2.jpg)  
文档这里写着建议用 helm。

在主页面的 Kubenetes "Click here to add Kubernetes cluster." 点一下  
![k8s0](https://1c7.me/content/images/2019/02/k8s0.jpg)

会看到如下界面。可以看到都是国外的主流 k8s 服务商。  
![k8s](https://1c7.me/content/images/2019/02/k8s.jpg)

添加 Custom Provider 应该可以添加国内的服务商，比如阿里云腾讯云之类的。  
我没试，因为我不会 k8s  
![k8s-custom](https://1c7.me/content/images/2019/02/k8s-custom.jpg)

### 5\. 临时测试环境

[根据文档](https://codefresh.io/docs/docs/getting-started/on-demand-environments/) ，CodeFresh 提供一个临时测试环境，看下图：  
![code-fresh-on-demand](https://1c7.me/content/images/2019/02/code-fresh-on-demand.jpg)  
注意，仅仅是 quick demos 用的，不是让你来服务用户的。

使用方法：在 Image 这一节点击 Launch  
![launch](https://1c7.me/content/images/2019/02/launch.jpg)

看日志可以看到 Rails 提示连接 PostgreSQL 错误  
![code-db](https://1c7.me/content/images/2019/02/code-db.jpg)  
因为我本地开发就用了 docker compose

database.yml 是这样写的：  
![database](https://1c7.me/content/images/2019/02/database.jpg)

docker-compose.yml 是这样写的：  
![docker-compose](https://1c7.me/content/images/2019/02/docker-compose.jpg)

所以报错了。  
解决办法就是设置环境变量呗，我这里就先不弄了，  
思路是 RAILS\_ENV 设置成 test 或者 development。  
然后把 database.yml 里面的 host 也读环境变量，  
也许可以随便搭一个 Amazon RDS？然后填 host, username, password？  
那么 db:migrate 怎么跑？  
有更简单的方案吗？干脆 dev 和 test 就直接用 SQLite 算了？

跑起来之后可以在 Enviroment 这里看到，点击右上角的 Open App  
![cf-env](https://1c7.me/content/images/2019/02/cf-env.jpg)  
会看到如下界面  
![cf-env2](https://1c7.me/content/images/2019/02/cf-env2.jpg)

注意：免费账号只能开一个 demo 环境，如果已经有了一个就会提示你：  
![one-env](https://1c7.me/content/images/2019/02/one-env.jpg)

### 6\. 查看免费限额（120 build/一个月）

![build-count](https://1c7.me/content/images/2019/02/build-count.jpg)  
在 Account Setting -> Billing 的顶部查看。

build 次数怎么算的？  
我翻了 Codefresh 文档 (包括 FAQ 等地方) 全都没有说。  
实测发现：不论是成功还是失败，只要触发了 build 就算一次。