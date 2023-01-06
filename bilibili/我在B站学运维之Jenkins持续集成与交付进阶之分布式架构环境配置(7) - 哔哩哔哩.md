**本章目录:**

0x00 前言简述

-   1.节点说明
    
-   2.节点连接
    

-   SSH 方式
    
-   JNLP 方式
    

-   3.点明主题
    
-   4.知识扩展
    

0x01 安装部署

-   (0) 分布式架构过程说明
    

-   在 Master 节点中添加 Agent 方式
    

-   (1) 单主机部署配置固定 agent
    

-   Java Web 启动 Agent 方式
    
-   Launch agents via SSH 启动 Agent 方式
    

![](https://i0.hdslb.com/bfs/article/4aa545dccf7de8d4a93c2b2b8e3265ac0a26d216.png@progressive.webp)

## 0x00 前言简述

**Q: 什么是Jenkins的分布式节点构建?**

> 描述: 简单的说就是通过将构建过程分配到从属Slave节点上，从而减轻Master节点的压力，而且可以同时构建多个有点类似负载均衡的概念。  
> 随着现在容器的盛行我们可以将server节点和agent节点在容器或者基于Kubernetes中部署, 可以实现动态的资源分配等等好处。

![WeiyiGeek.Jenkins分布式节点](https://i0.hdslb.com/bfs/article/ac61620cf2584048e35f58b2136c6cb70445f395.png@942w_776h_progressive.webp)

## 1.节点说明

描述: 我们在使用Jenkins的时候一般都会分为server节点与agent节点(也可以叫 slave 节点)。

-   1.server ：主要用于处理调度构建作业，把构建分发到slave节点进行实际执行，监视slave节点的状态（必要时让它们进行上线或者离线），记录和发布构建产物。
    
-   2.agent ： 主要用于处理Job任务等例如编译和发布, agent节点可以分为静态节点和动态节点;
    

**节点类型:**

-   1.静态节点是固定的一台vm虚机或者容器。
    
-   2.动态节点是随着任务的构建来自动创建agent节点。  
    

## 2.节点连接

**agent节点加入的两种方式:**

-   ssh : 在Linux系统中最方便的就是通过SSH启动Jenkins节点,关键是需要再Slave机器中开启sshd服务以及网络连通;
    
-   jnlp ：兼容各种操作系统只要网络可以正常通信都可以采用此种方法(需要安装Java环境);
    

## SSH 方式

环境依赖: SSH Slaves plugin 插件 、SSH Credentials Plugin 插件（管理认证票据）  
添加流程:

-   1）输入Slave节点IP
    
-   2）添加Credentials认证票据(账号密码、或、密钥登陆)
    
-   3）在这里设置的credentials在jenkins的其他需要credentials的地方，可以通过下拉菜单选择使用，比如添加slave时可以直接在Credentials下拉菜单里选择对应的credential就行
    

**用户密码方式添加:**  
添加流程:

-   1）新建节点->输入节点IP
    
-   2）基础信息配置
    
-   3）选择Jenkins凭据
    
-   4）启动代理
    

**私钥方式方式添加:**  
添加流程:(相比于上面的流程唯一不同的是Jenkins凭据选择为`ssh private key`)

-   记住是私钥不是公钥 `cat .ssh/id_rsa` 输入到Private Key之中
    

## JNLP 方式

描述: 与ssh方式是master主动连接slave不同而JNLP方式是slave主动连接master。

Tips : 复制节点在节点环境配置好之后，我们再添加节点就可以复制了修改IP和其他自定义配置即可。

Tips : 在需要Jenkins全局安全配置上开启 Inbound agents 端口 50000/tcp 代理端口, 此端口的作用是便于Agent的jnlp与jenkins的master节点间进行通信;

## 3.点明主题

**Q: 传统Jenkins的server、agent分布式方案有什么缺陷?**

-   1.高可用管理: Master 节点发生单点故障时整个流程都不可用了;
    
-   2.配置管理维护: 每个 Slave节点的配置环境不一样，来完成不同语言的编译打包等操作，但是这些差异化的配置导致管理起来非常不方便，维护起来也是比较费劲
    
-   3.资源分配不均衡：有的 Slave节点要运行的job出现排队等待，而有的Slave节点处于空闲状态
    
-   4.资源浪费： 每台 Slave节点可能是实体机或者VM，当Slave节点处于空闲状态时，也不会完全释放掉资源
    

Tips : 正因为上面的这些种种痛点我们渴望一种更高效更可靠的方式来完成这个 CI/CD 流程，而 Docker 虚拟化容器技术能很好的解决这个痛点，又特别是在 Kubernetes 集群环境下面能够更好来解决上面的问题，所以我们可以引入Kubernates来解决！

Tips : 基于Kubernete的CI/CD可以使用的工具有很多, 比如Jenkins、Gitlab CI已经新兴的drone之类的都是可以，大家可以根据需求进行学习搭建并落地;

**Q: 什么是 Kubernetes ? 它有何作用?**  
答: Kubernetes （简称K8S）是Google开源的容器集群管理系统，在Docker技术的基础上，为容器化的应用提供部署运行、资源调度、服务发现和动态伸缩等一系列完整功能，提高了大规模容器集群管理的便捷性。  
其主要功能如下：

-   1.使用Docker对应用程序包装(package)、实例化(instantiate)、运行(run)。
    
-   2.以集群的方式运行、管理跨机器的容器。以集群的方式运行、管理跨机器的容器。
    
-   3.解决 Docker跨机器容器之间的通讯问题。解决Docker跨机器容器之间的通讯问题。
    
-   4.Kubernetes 的自我修复机制使得容器集群总是运行在用户期望的状态。
    

![WeiyiGeek.Kubernetes 搭建 Jenkins 集群示意图](https://i0.hdslb.com/bfs/article/457cf49cb79ce2dcc7817c6fe032b0d6e50a1499.png@942w_743h_progressive.webp)

PS : 从图上可以看到 `Jenkins Master 和 Jenkins Slave` 以 Pod 形式运行在 Kubernetes 集群的 Node 上，Master 运行在其中一个节点，并且将其配置数据存储到一个 Volume 上去，Slave 运行在各个节点上，并且它不是一直处于运行状态，它会按照需求动态的创建并自动删除。  
PS : 这种方式的工作流程大致为当 Jenkins Master 接受到 Build 请求时，会根据配置的 Label 动态创建一个运行在 Pod 中的 Jenkins Slave 并注册到 Master 上，当运行完 Job 后，这个 Slave 会被注销并且这个 Pod 也会自动删除，恢复到最初状态

**Q: Kubernets方式部署给我们带来什么好处?**

-   1.服务高可用，当 Jenkins Master 出现故障时，Kubernetes 会自动创建一个新的 Jenkins Master 容器，并且将 Volume 分配给新创建的容器，保证数据不丢失，从而达到集群服务高可用。
    
-   2.动态伸缩，合理使用资源，每次运行 Job 时，会自动创建一个 Jenkins Slave，Job 完成后，Slave 自动注销并删除容器，资源自动释放，而且 Kubernetes 会根据每个资源的使用情况，动态分配 Slave 到空闲的节点上创建，降低出现因某节点资源利用率高，还排队等待在该节点的情况。
    
-   3.扩展性好，当 Kubernetes 集群的资源严重不足而导致 Job 排队等待时，可以很容易的添加一个 Kubernetes Node 到集群中，从而实现扩展。
    

**常用 Kubernates + Harbor + Jenkins + gitlab 持续集成方案：**  
Tips : 大致工作流程 `手动 /自动构建 -> Jenkins 调度 K8S API -> 动态生成 Jenkins Slave pod -> Slave pod 拉取 Git 代码／编译／打包镜像 ->推送到镜像仓库 Harbor -> Slave 工作完成，Pod 自动销毁 -> 部署 到测试或生产 Kubernetes平台`。（完全自动化，无需人工干预）

![weiyigeek.CI/CD集成](https://i0.hdslb.com/bfs/article/bbed72885c69fd639fd44539c2bc171db074d8aa.png@942w_438h_progressive.webp)

## 4.知识扩展

**(1) 官方的Jenkins镜像网站**  
Hub Docker Images : https://hub.docker.com/r/jenkins/jenkins/

```
docker pull jenkins/jenkins
```

**(2) Kubernetes 插件官方帮助**  
kubernetes Plugs : https://plugins.jenkins.io/kubernetes/

## 0x01 安装部署

## (0) 分布式架构过程说明

将 Jenkins 的 `Master/Agent` 分布式架构直接部署在宿主机上不是一个很好的选择；但是它作为一个向容器化过度的中间阶段，是必要学习掌握的。

Tips : 整个 Jenkins 服务分布式部署是很简单的其步骤为：

-   部署一个 Jenkins Master 节点。
    
-   通过 Jenkins 的 WEB 页面，在 Master 节点上添加 Agent node 节点。
    
-   添加完 Agent Node 节点后，Jenkins Master 会提供部署 Agent Node 服务的软件包和启动方式；直接找台服务器，根据提示运行就行。
    

## 在 Master 节点中添加 Agent 方式

-   Step 1.新建节点页面的访问路径  
    描述: 在 Jenkins 服务的页面上找到”新建节点“的页面；它的访问路径如下： `Manage Jenkins -》Manage Nodes and Clouds（管理节点）-》New Node（新建节点）`
    
-   Step 2.节点名称和节点类型  
    描述：通过上面的访问路径，进入添加节点的第一个页面。在这里需要填写一下【节点名称】和选择节点类型，一般选择永久节点 \[Permanent Agent\] 即可
    
-   Step 3.在节点的详细设置页面中填写更多信息
    

```
* 配置项 配置项说明/配置信息
* Name/名称 Jenkins Agent 的名称
* of executors / 并发执行数 Agent 节点可以同时执行几个 Job
* Remote root directory / 远程工作目录 从节点上jenkins agent的工作目录，推荐只用绝对路径，如/home//jenkins-agent。注意jenkins要有该目录的读写权限
* Labels / 标签 给Agent节点设置标签；Job 任务可以根据标签选择特定的 Agent 节点执行。
* Usage / Agent 节点的使用方法 有两种方式：1、尽量使用此Agent执行任务；2、只执行标签匹配的任务。
* Lanuch method / Agent 启动方法 有多种选择，这里使用”Launch agent by connecting it to the master“/通过将 Agent 连接到 Master 来启动它
* 其他的配置项可以选择默认状态，也可以更加自己的理解设置。
```

-   Step 4.获取部署 Agent 的方法  
    描述: 上面的步骤操作完成后，就会有个展示配置 Agent 节点的页面。其中提供了两中部署 Agent 的方式，我们选择第二种。
    
-   Step 5.在 Agent 服务器的命令行执行启动命令
    

```
# 方式1.将密码通过命令行直接传入(不安全)
java -jar agent.jar -jnlpUrl http://jenkins.example.com/computer/agent-test/slave-agent.jnlp -secret d4abba3f13324b85ab2997e22c3442045bb86fcd213f79fa01416a5fd0399a18 -workDir ""

# 方式2.将密码信息写入到文件中 Agent 启动方式
echo d4abba3f13324b85ab2997e22c3442045bb86fcd213f79fa01416a5fd0399a18 > secret-file
java -jar agent.jar -jnlpUrl http://jenkins.example.com/computer/agent-test/slave-agent.jnlp -secret @secret-file -workDir ""
```

## (1) 单主机部署配置固定 agent

描述: 添加一个普通、固定(永久)的节点到Jenkins即给 Jenkins 增加了一个普通的永久代理人；之所以叫做“固定”是因为Jenkins没给这种节点提供更高级的集成方式;

-   例如：动态配置。没有其他代理类型能选择的话可以选择该代理类型；
    
-   例如，你在添加不受Jenkins管理的物理机、在Jenkins外部管理的虚拟机等。
    

**环境准备:**

```
$ lsb_release -a
No LSB modules are available.
Distributor ID: Ubuntu
Description:    Ubuntu 20.04.1 LTS
Release:        20.04
Codename:       focal

kubernetes 安装的 Jenkins 2.275 : `http://jenkins.weiyigeek.top:8080/`
```

## Java Web 启动 Agent 方式

**Q: 如何实现Jenkins分布式构建?**

-   Step 1.开启代理程序的TCP端口：Manage Jenkins -> Configure Global Security(全局安全配置) -> 代理 -> 设置为固定的50000端口
    
-   Step 2.在 Manage Jenkins -> Manage Nodes 节点列表 -> 新增节点 -> 节点名称（agent-机器名称）-> 添加一个普通、固定的节点到Jenkins;
    
-   Step 3.此时有两种 Slave 节点连接Master节点的方法 -> (`Launch agent by connecting it to the master` 或者 `通过Java Web启动代理`) ;  
    描述: 使用`Java Web Start`就必须在Agent机器上打开JNLP文件，然后将创建到Jenkins服务器的TCP连接，意味着`不需要Jenkins服务器访问Agent 而是Agent能够链接到Jenkins Server即可`。
    

```
# 在命令行中启动节点
java -jar agent.jar -jnlpUrl http://192.168.12.107:30001/computer/node-1/jenkins-agent.jnlp -secret 52e2e9b37cd4a36310d39984ff461b48ef95b40101a4d34875b7248a7622bd92 -workDir "/home/jenkins"

# Run from agent command line, with the secret stored in a file:
echo 52e2e9b37cd4a36310d39984ff461b48ef95b40101a4d34875b7248a7622bd92 > secret-file
java -jar agent.jar -jnlpUrl http://192.168.12.107:30001/computer/node-1/jenkins-agent.jnlp -secret @secret-file -workDir "/home/jenkins"
```

![WeiyiGeek.Create-Node](https://i0.hdslb.com/bfs/article/6f7502deb25b60d4535388fa675bbe216bc78a0e.png@942w_726h_progressive.webp)

-   Step 4.采用`VM方式`运行agent.jar连接到Jenkins的Server节点, 首先我们知道上面的agent.jar和secret信息等;
    

```
# (1) 我们在一台Linux服务器中下载agent.jar和启动连接。
wget http://192.168.12.107:30001/jnlpJars/agent.jar
java -jar agent.jar -jnlpUrl http://192.168.12.107:30001/computer/node-1/jenkins-agent.jnlp -secret 52e2e9b37cd4a36310d39984ff461b48ef95b40101a4d34875b7248a7622bd92 -workDir "/home/jenkins"

# (2) 运行后输入以下日志
  # INFO: Agent discovery successful
  # Agent address: 192.168.1.200
  # Agent port:    30081
  # Identity:      0b:a1:da:6c:8e:e2:ca:f8:17:f6:b7:ee:cb:ff:84:0d
  # Jul 24, 2020 5:57:29 AM hudson.remoting.jnlp.Main$CuiListener status
  # INFO: Handshaking
  # Jul 24, 2020 5:57:29 AM hudson.remoting.jnlp.Main$CuiListener status
  # INFO: Connecting to 192.168.1.200:30081
  # Jul 24, 2020 5:57:29 AM hudson.remoting.jnlp.Main$CuiListener status
  # INFO: Trying protocol: JNLP4-connect
  # Jul 24, 2020 5:57:29 AM hudson.remoting.jnlp.Main$CuiListener status
  # INFO: Remote identity confirmed: 0b:a1:da:6c:8e:e2:ca:f8:17:f6:b7:ee:cb:ff:84:0d
  # Jul 24, 2020 5:57:30 AM hudson.remoting.jnlp.Main$CuiListener status
  # INFO: Connected   # 表示 成功

# (3) 使用nohup后台运行agent
nohup java -jar agent.jar -jnlpUrl http://192.168.12.107:30001/computer/node-1/jenkins-agent.jnlp -secret 52e2e9b37cd4a36310d39984ff461b48ef95b40101a4d34875b7248a7622bd92 -workDir "/home/jenkins"
```

-   Step 5.采用`Docker方式`运行agent.jar连接到Jenkins的Server节点, 此种方式非常简单拉取镜像和启动镜像；  
    参考连接: https://hub.docker.com/r/jenkins/inbound-agent
    

```
# (1) 拉取镜像
docker pull jenkins/inbound-agent:alpine

# (2) 获取 Jenkins 的 SVC 地址(由于我的Master是采用K8s搭建的)
~$ kubectl get svc -n devops
  # NAME             TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                          AGE
  # jenkins          NodePort    10.99.55.239     <none>        8080:30001/TCP,50000:32001/TCP   19d


# (3) 启动 agent 镜像(注意参数)
# docker run --init jenkins/inbound-agent -url http://jenkins-server:port -workDir=/home/jenkins/agent <secret> <agent name>
~$ docker run -d --name jenkins-agent --init jenkins/inbound-agent:alpine -url http://10.99.55.239:8080 -workDir=/home/jenkins/agent 52e2e9b37cd4a36310d39984ff461b48ef95b40101a4d34875b7248a7622bd92 node-1
c7a00c89f92f16c07d35e572d2ba9da3c17c47a2746e46e3d3225bff418a78f3

# (4) 查看是否启动成功
~$ docker logs c7a00c89f92f16c07d35e572d2ba9da3c17c47a2746e46e3d3225bff418a78f3
  # Feb 03, 2021 11:57:39 AM hudson.remoting.jnlp.Main createEngine
  # INFO: Setting up agent: node-1
  # Feb 03, 2021 11:57:39 AM hudson.remoting.jnlp.Main$CuiListener <init>
  # INFO: Jenkins agent is running in headless mode.
  # Feb 03, 2021 11:57:40 AM hudson.remoting.Engine startEngine
  # INFO: Using Remoting version: 4.6
  # Feb 03, 2021 11:57:40 AM org.jenkinsci.remoting.engine.WorkDirManager initializeWorkDir
  # INFO: Using /home/jenkins/agent/remoting as a remoting work directory
  # Feb 03, 2021 11:57:40 AM org.jenkinsci.remoting.engine.WorkDirManager setupLogging
  # INFO: Both error and output logs will be printed to /home/jenkins/agent/remoting
  # Feb 03, 2021 11:57:40 AM hudson.remoting.jnlp.Main$CuiListener status
  # INFO: Locating server among [http://10.99.55.239:8080]
  # Feb 03, 2021 11:57:40 AM org.jenkinsci.remoting.engine.JnlpAgentEndpointResolver resolve
  # INFO: Remoting server accepts the following protocols: [JNLP4-connect, Ping]
  # Feb 03, 2021 11:57:40 AM hudson.remoting.jnlp.Main$CuiListener status
  # INFO: Agent discovery successful
  #   Agent address: 10.99.55.239
  #   Agent port:    50000
  #   Identity:      8e:c7:1e:e1:39:ee:f4:2a:43:f6:aa:d9:0e:b7:b6:62
  # Feb 03, 2021 11:57:40 AM hudson.remoting.jnlp.Main$CuiListener status
  # INFO: Handshaking
  # Feb 03, 2021 11:57:40 AM hudson.remoting.jnlp.Main$CuiListener status
  # INFO: Connecting to 10.99.55.239:50000
  # Feb 03, 2021 11:57:40 AM hudson.remoting.jnlp.Main$CuiListener status
  # INFO: Trying protocol: JNLP4-connect
  # Feb 03, 2021 11:58:00 AM hudson.remoting.jnlp.Main$CuiListener status
  # INFO: Remote identity confirmed: 8e:c7:1e:e1:39:ee:f4:2a:43:f6:aa:d9:0e:b7:b6:62
  # Feb 03, 2021 11:58:02 AM hudson.remoting.jnlp.Main$CuiListener status
  # INFO: Connected   # 表示连接成功
```

![WeiyiGeek.jenkins-agent-docker](https://i0.hdslb.com/bfs/article/f897015eae37bc93ea84391abbfbcc7fd290aa54.png@942w_612h_progressive.webp)

-   Step 6.采用kubernetes集群以静态的方式部署agent，我们首先编写一个部署文件，并且定义好名称空间、镜像、agent配置信息。同样此处我们创建一个node-2的agent节点;
    

```
# (0) 在命令行中启动节点参数实例 (注意K8s下面的环境变量)
java -jar agent.jar -jnlpUrl http://192.168.12.107:30001/computer/node-2/jenkins-agent.jnlp -secret a06d5adaef69cbaf34e9296d8340d464163ab9f5bbf6b465a290237dc38d45db -workDir "/home/jenkins/agent"

# (1) Jenkins-agent-Deployment 资源清单
cat > Jenkins-agent-Deployment.yaml <<'END'
kind: Deployment
apiVersion: apps/v1
metadata:
  labels:
    k8s-app: jenkins-agent
  name: jenkins-agent-node2
  namespace: devops
spec:
  replicas: 1
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      k8s-app: jenkins-agent-node
  template:
    metadata:
      labels:
        k8s-app: jenkins-agent-node
      namespace: devops
      name: jenkins-agent-node2
    spec:
      containers:
        - name: jenkins-agent
          image: jenkins/inbound-agent:alpine
          imagePullPolicy: IfNotPresent
          resources:
            limits:
              cpu: 1000m
              memory: 2Gi
            requests:
              cpu: 500m
              memory: 512Mi
          env:
            - name: JENKINS_URL
              value: http://10.99.55.239:8080
            - name: JENKINS_SECRET
              value: a06d5adaef69cbaf34e9296d8340d464163ab9f5bbf6b465a290237dc38d45db
            - name: JENKINS_AGENT_NAME
              value: node-2
            - name: JENKINS_AGENT_WORKDIR
              value: /home/jenkins/workspace
END

# JENKINS_URL: url for the Jenkins server, can be used as a replacement to -url option, or to set alternate jenkins URL
# JENKINS_TUNNEL: (HOST:PORT) connect to this agent host and port instead of Jenkins server, assuming this one do route TCP traffic to Jenkins master
# JENKINS_SECRET: agent secret, if not set as an argument
# JENKINS_AGENT_NAME: agent name, if not set as an argument
# JENKINS_AGENT_WORKDIR: agent work directory, if not set by optional parameter -workDir
# JENKINS_WEB_SOCKET: true if the connection should be made via WebSocket rather than TCP


# (2)通过kubectl工具在k8s集群中部署`jenkins agent deployment`并查验状态
kubectl create -f Jenkins-agent-Deployment.yaml
  # deployment.apps/jenkins-agent-node2 created

~/k8s/jenkins$ kubectl get pod -n devops -o wide | grep "jenkins-agent"
  # jenkins-agent-node2-574bb65cb8-5kh7g   1/1     Running   0          29s     172.16.182.237

~/k8s/jenkins$ kubectl logs -f jenkins-agent-node2-574bb65cb8-5kh7g -n devops

# (3) 在传入JENKINS_URL时建议尽量采用域名的方式;
~/k8s/jenkins$ kubectl exec -it jenkins-agent-node2-574bb65cb8-5kh7g -n devops bash
  # bash-5.0$ ping jenkins.devops
  # PING jenkins.devops (10.99.55.239): 56 data bytes
  # bash-5.0$ ping jenkins.devops.svc.cluster.local
  # PING jenkins.devops.svc.cluster.local (10.99.55.239): 56 data bytes
```

![WeiyiGeek.kubernetes-jenkins-agent](https://i0.hdslb.com/bfs/article/ac02c9ad62635b793a7ec16d82f6672349236368.png@942w_653h_progressive.webp)

## Launch agents via SSH 启动 Agent 方式

描述: 当前Jenkins 2.277版本默认是不支持SSH 启动 Agent 方式, 我们需要安装ssh Agent插件;

```
SSH Agent - This plugin allows you to provide SSH credentials to builds via a ssh-agent in Jenkins
```

-   Step 1.首先创建一个凭据存储服务器认证信息。
    
-   Step 2.之后创建一个新的节点添加以下配置。配置ssh的主机和认证信息最后保存(agent配置完成)。
    

![WeiyiGeek.ssh-agent](https://i0.hdslb.com/bfs/article/3b9dcb31a6e0ba3a8baceb39bf82bd13dc1034e1.png@942w_653h_progressive.webp)

-   Step 3.保存后创建一个测试的Pipeline项目
    

```
pipeline {
  // 此处agent指令是必须的
  agent none
  stages {
    // docker 构建的
    stage ('node-1'){
      agent {
        node { label "docker-jenkins-slave"}
      }
      steps {
        echo "----- node-1 ------"
        sh "hostname && ifconfig"
        sh "env"
        echo "----- node-1 end------"
      }
    }
    // k8s 集群中构建的
     stage ('node-2'){
      agent {
        node { label "k8s-node-2"}
      }
      steps {
        echo "----- node-2 ------"
        sh "hostname && ifconfig"
        sh "env"
        echo "----- node-2 end------"
      }
    }
  }
}
```

输出结果:

```
Started by user Jenkins 管理员
Running in Durability level: MAX_SURVIVABILITY
[Pipeline] Start of Pipeline
[Pipeline] stage

[Pipeline] { (node-1)
Running on node-1 in /home/jenkins/workspace/pineline-use-node
[Pipeline] echo
----- node-1 ------
[Pipeline] sh
+ hostname
c7a00c89f92f
+ ifconfig
eth0      Link encap:Ethernet  HWaddr 02:42:AC:11:00:02
          inet addr:172.17.0.2  Bcast:172.17.255.255  Mask:255.255.0.0
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:10783 errors:0 dropped:0 overruns:0 frame:0
          TX packets:14915 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0
          RX bytes:38664627 (36.8 MiB)  TX bytes:5669411 (5.4 MiB)

lo        Link encap:Local Loopback
          inet addr:127.0.0.1  Mask:255.0.0.0
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)

[Pipeline] sh
  # + env
  # JENKINS_HOME=/var/jenkins_home
  # LANGUAGE=en_US:en
  # RUN_CHANGES_DISPLAY_URL=http://192.168.12.107:30001/job/pineline-use-node/3/display/redirect?page=changes
  # HOSTNAME=c7a00c89f92f
  # NODE_LABELS=docker-jenkins-slave node-1
  # HUDSON_URL=http://192.168.12.107:30001/
  # SHLVL=3
  # HOME=/home/jenkins
  # BUILD_URL=http://192.168.12.107:30001/job/pineline-use-node/3/
  # HUDSON_COOKIE=de8bbafb-13fd-42d5-99a4-7ffaebf9a3e2
  # JENKINS_SERVER_COOKIE=durable-8dd62800688ecbc8d9c9e78a15c49a2d
  # WORKSPACE=/home/jenkins/workspace/pineline-use-node
  # JAVA_VERSION=jdk8u272-b10
  # NODE_NAME=node-1
  # RUN_ARTIFACTS_DISPLAY_URL=http://192.168.12.107:30001/job/pineline-use-node/3/display/redirect?page=artifacts
  # STAGE_NAME=node-1
  # EXECUTOR_NUMBER=0
  # RUN_TESTS_DISPLAY_URL=http://192.168.12.107:30001/job/pineline-use-node/3/display/redirect?page=tests
  # BUILD_DISPLAY_NAME=#3
  # HUDSON_HOME=/var/jenkins_home
  # AGENT_WORKDIR=/home/jenkins/agent
  # JOB_BASE_NAME=pineline-use-node
  # PATH=/opt/java/openjdk/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
  # BUILD_ID=3
  # BUILD_TAG=jenkins-pineline-use-node-3
  # LANG=en_US.UTF-8
  # JENKINS_URL=http://192.168.12.107:30001/
  # JOB_URL=http://192.168.12.107:30001/job/pineline-use-node/
  # BUILD_NUMBER=3
  # JENKINS_NODE_COOKIE=a822e652-5870-44ef-b671-d6c1f2afaffb
  # RUN_DISPLAY_URL=http://192.168.12.107:30001/job/pineline-use-node/3/display/redirect
  # HUDSON_SERVER_COOKIE=04b411a999365c6a
  # JOB_DISPLAY_URL=http://192.168.12.107:30001/job/pineline-use-node/display/redirect
  # JOB_NAME=pineline-use-node
  # LC_ALL=en_US.UTF-8
  # JAVA_HOME=/opt/java/openjdk
  # PWD=/home/jenkins/workspace/pineline-use-node
  # WORKSPACE_TMP=/home/jenkins/workspace/pineline-use-node@tmp
  # GITLAB_OBJECT_KIND=none
[Pipeline] echo
----- node-1 end------


[Pipeline] stage
[Pipeline] { (node-2)
Running on node-2 in /home/jenkins/agent/workspace/pineline-use-node
[Pipeline] {
[Pipeline] echo
----- node-2 ------
[Pipeline] sh
+ hostname
jenkins-agent-node2-574bb65cb8-5kh7g
+ ifconfig
eth0      Link encap:Ethernet  HWaddr 5E:A7:08:EA:68:0F
          inet addr:172.16.182.237  Bcast:172.16.182.237  Mask:255.255.255.255
          UP BROADCAST RUNNING MULTICAST  MTU:1480  Metric:1
          RX packets:8149 errors:0 dropped:0 overruns:0 frame:0
          TX packets:11311 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0
          RX bytes:30953238 (29.5 MiB)  TX bytes:4382625 (4.1 MiB)

lo        Link encap:Local Loopback
          inet addr:127.0.0.1  Mask:255.0.0.0
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)

[Pipeline] sh
  # + env
  # JENKINS_HOME=/var/jenkins_home
  # JENKINS_SECRET=a06d5adaef69cbaf34e9296d8340d464163ab9f5bbf6b465a290237dc38d45db
  # agentType=k8s
  # KUBERNETES_PORT=tcp://10.96.0.1:443
  # KUBERNETES_SERVICE_PORT=443
  # LANGUAGE=en_US:en
  # RUN_CHANGES_DISPLAY_URL=http://192.168.12.107:30001/job/pineline-use-node/3/display/redirect?page=changes
  # SONARQUBE_PORT_9000_TCP_ADDR=10.100.233.217
  # HOSTNAME=jenkins-agent-node2-574bb65cb8-5kh7g
  # SHLVL=3
  # NODE_LABELS=k8s-node-2 node-2
  # HUDSON_URL=http://192.168.12.107:30001/
  # HOME=/home/jenkins
  # SONARQUBE_PORT_9000_TCP_PORT=9000
  # BUILD_URL=http://192.168.12.107:30001/job/pineline-use-node/3/
  # SONARQUBE_PORT_9000_TCP_PROTO=tcp
  # HUDSON_COOKIE=c088e600-327b-4e78-a80e-4ba7bd7254f3
  # JENKINS_SERVER_COOKIE=durable-4d6313311ef99675e2bfd51284774a87
  # JENKINS_AGENT_WORKDIR=/home/jenkins/workspace
  # SONARQUBE_SERVICE_HOST=10.100.233.217
  # WORKSPACE=/home/jenkins/agent/workspace/pineline-use-node
  # JAVA_VERSION=jdk8u272-b10
  # NODE_NAME=node-2
  # SONARQUBE_PORT_9000_TCP=tcp://10.100.233.217:9000
  # RUN_ARTIFACTS_DISPLAY_URL=http://192.168.12.107:30001/job/pineline-use-node/3/display/redirect?page=artifacts
  # STAGE_NAME=node-2
  # EXECUTOR_NUMBER=1
  # SONARQUBE_SERVICE_PORT=9000
  # SONARQUBE_PORT=tcp://10.100.233.217:9000
  # RUN_TESTS_DISPLAY_URL=http://192.168.12.107:30001/job/pineline-use-node/3/display/redirect?page=tests
  # BUILD_DISPLAY_NAME=#3
  # JENKINS_PORT_50000_TCP_ADDR=10.99.55.239
  # KUBERNETES_PORT_443_TCP_ADDR=10.96.0.1
  # HUDSON_HOME=/var/jenkins_home
  # AGENT_WORKDIR=/home/jenkins/agent
  # JOB_BASE_NAME=pineline-use-node
  # PATH=/opt/java/openjdk/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
  # JENKINS_SERVICE_HOST=10.99.55.239
  # SONARQUBE_SERVICE_PORT_SONARQUBE=9000
  # JENKINS_PORT_8080_TCP_ADDR=10.99.55.239
  # BUILD_ID=3
  # JENKINS_PORT_50000_TCP_PORT=50000
  # KUBERNETES_PORT_443_TCP_PORT=443
  # JENKINS_SERVICE_PORT_AGENT=50000
  # JENKINS_PORT_50000_TCP_PROTO=tcp
  # BUILD_TAG=jenkins-pineline-use-node-3
  # KUBERNETES_PORT_443_TCP_PROTO=tcp
  # JENKINS_URL=http://192.168.12.107:30001/
  # JENKINS_PORT_8080_TCP_PORT=8080
  # LANG=en_US.UTF-8
  # JOB_URL=http://192.168.12.107:30001/job/pineline-use-node/
  # JENKINS_AGENT_NAME=node-2
  # JENKINS_PORT_8080_TCP_PROTO=tcp
  # BUILD_NUMBER=3
  # JENKINS_NODE_COOKIE=243aa3e8-0de2-4297-b617-8764795f5cab
  # RUN_DISPLAY_URL=http://192.168.12.107:30001/job/pineline-use-node/3/display/redirect
  # JENKINS_PORT=tcp://10.99.55.239:8080
  # JENKINS_SERVICE_PORT=8080
  # HUDSON_SERVER_COOKIE=04b411a999365c6a
  # JOB_DISPLAY_URL=http://192.168.12.107:30001/job/pineline-use-node/display/redirect
  # KUBERNETES_PORT_443_TCP=tcp://10.96.0.1:443
  # JENKINS_PORT_50000_TCP=tcp://10.99.55.239:50000
  # KUBERNETES_SERVICE_PORT_HTTPS=443
  # JOB_NAME=pineline-use-node
  # LC_ALL=en_US.UTF-8
  # PWD=/home/jenkins/agent/workspace/pineline-use-node
  # JENKINS_PORT_8080_TCP=tcp://10.99.55.239:8080
  # JAVA_HOME=/opt/java/openjdk
  # KUBERNETES_SERVICE_HOST=10.96.0.1
  # JENKINS_SERVICE_PORT_WEB=8080
  # WORKSPACE_TMP=/home/jenkins/agent/workspace/pineline-use-node@tmp
  # GITLAB_OBJECT_KIND=none
[Pipeline] echo
----- node-2 end------
[Pipeline] End of Pipeline
Finished: SUCCESS
```

![WeiyiGeek.Pipeline流水线选择](https://i0.hdslb.com/bfs/article/1518ca4cff28fae82df2ec7c2d487ff22d19ab4a.png@942w_716h_progressive.webp)

本章至此完毕！未完请继续期待下一章！

![](https://i0.hdslb.com/bfs/article/4adb9255ada5b97061e610b682b8636764fe50ed.png@progressive.webp)

>         欢迎各位志同道合的朋友一起学习交流，如文章有误请在下方留下您宝贵的经验知识，个人邮箱地址【master#weiyigeek.top】

![](https://i0.hdslb.com/bfs/article/02db465212d3c374a43c60fa2625cc1caeaab796.png@progressive.webp)

更多文章来源: https://weiyigeek.top 【WeiyiGeek Blog - 为了能到远方，脚下的每一步都不能少】

![](https://i0.hdslb.com/bfs/article/04ec2a1812ca192ea0870ed12a235e1a3ba7b019.png@942w_483h_progressive.webp)

   如果你觉得这个专栏还不错的，请给这篇专栏点个赞、投个币、收个藏、关个注，这将对我有很大帮助！    

如果你觉得这个专栏还不错的，请给这篇专栏点个赞、投个币、收个藏、关个注，这将对我有很大帮助！