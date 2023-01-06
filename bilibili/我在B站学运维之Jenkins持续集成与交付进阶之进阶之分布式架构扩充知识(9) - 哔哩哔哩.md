**本章目录:**

-   0x00 前言简述
    
-   0x01 Kubernetes-plugin 使用
    

-   环境依赖
    
-   插件参数
    

-   podTemplate - Pod和容器模板
    
-   container - 容器模板的定义
    

-   Pipeline Support 实践示例
    

-   Scripted Pipeline
    
-   Declarative Pipeline
    

![](https://i0.hdslb.com/bfs/article/4aa545dccf7de8d4a93c2b2b8e3265ac0a26d216.png@progressive.webp)

描述: 有了上一章的基础`"4.Jenkins进阶之分布式架构环境配置"`介绍，本章主要介绍Kubernetes 的插件使用实例, 以及自定义 Slave 的 Jenkins-Jnlp 容器镜像模板(重点)，在进行K8s动态节点生成时拉取使用，极大的节约了系统资源。

描述: 前面已经对Jenkins集成Kubernetes进行部署和配置，本章将进行介绍与实践如何使用该插件以及帮助文档的记录;

说明: Kubernetes-plugin 它是在Kubernetes集群中运行动态代理的Jenkins插件, 该插件为每个启动的代理创建一个Kubernetes Pod，由Docker映像定义运行，并在每次构建后停止。

备注: 代理是使用JNLP启动的，因此预期图像会自动连接到Jenkins主机，以下是重要的环境变量（我们说过不建议使用自己jnlp覆盖默认的jnlp容器）。

```
# jenkins/inbound-agent
JENKINS_URL: Jenkins web interface url
JENKINS_SECRET: the secret key for authentication
JENKINS_AGENT_NAME: the name of the Jenkins agent
JENKINS_NAME: the name of the Jenkins agent (Deprecated. Only here for backwards compatibility)
```

插件地址: https://github.com/jenkinsci/kubernetes-plugin

自定义jnlp容器模板镜像地址: https://hub.docker.com/r/weiyigeek/alpine-jenkins-jnlp

该镜像是Jenkins自定义jnlp容器模板，主要用于Jenkins工作节点容器化使用，主要包含Gitlab\_release发布、 docker 容器管理、kubectl 集群管理功能。我们可以直接在docker 的环境中 `docker pull weiyigeek/alpine-jenkins-jnlp`下来

-   **1.先决条件**
    

-   正在运行的Kubernetes集群1.14或更高版本。对于OpenShift用户，这意味着OpenShift容器平台4.x。
    
-   安装了一个Jenkins实例
    
-   已安装Jenkins Kubernetes插件
    

-   **2.填写Kubernetes插件配置**
    

-   您将打开Jenkins UI并导航至Manage Jenkins-> Configure System-> Cloud-> Kubernetes并输入相应的Kubernetes URL和Jenkins URL，除非Jenkins在Kubernetes中运行否则默认工作（参考上一章）。
    

-   **3.支持的凭证**
    

-   Username/password - 用户名密码
    
-   Secret File (kubeconfig file) - 秘密文件（kubeconfig文件）
    
-   Secret text (Token-based authentication) (OpenShift) - 秘密文本（基于令牌的身份验证）（OpenShift）
    
-   Google Service Account from private key (GKE authentication) - 来自私钥的Google服务帐户（GKE身份验证）
    
-   X.509 Client Certificate - X.509客户端证书
    

描述: podTemplate 是用于创建代理的pod的模板可以通过`用户界面或管道进行配置`, 无论哪种方式它都可以访问以下字段：

```
- 1) cloud ：在Jenkins设置中定义的云的名称。默认为kubernetes
- 2) name : Pod 名称
- 3) namespace : 名称空间设置
- 4) label : 标签值可以设置为唯一的值，以避免跨构建冲突，或省略，并在步骤中定义POD_LABEL。
- 5) yaml : 资源清单声明
- 6) yamlFile : 指定资源清单的文件
- 7) yamlMergeStrategy : 控制yaml定义是覆盖还是与从用声明的pod模板继承的yaml定义合并inheritFrom。默认为override()。
- 8) showRawYaml : 启用或禁用原始Yaml文件的输出默认为true (后续实践)
- 8) serviceAccount : k8s中创建的服务帐户
- 9) nodeSelector ：节点选择器
- 10) nodeUsageMode : 节点调用模式(NORMAL它控制Jenkins是只调度标签表达式匹配的作业，EXCLUSIVE尽可能多地使用该节点。)
- 11) volumes ：为pod定义各种类型的卷并在所有容器中挂载卷
- 12) envVars ：应用于所有容器的环境变量(`envVar 环境变量其值是内联定义的, secretEnvVar 一个环境变量其值是从Kubernetes机密派生的。`)
- 13) imagePullSecrets : 从私有的镜像仓库中拉取镜像的凭据
- 14) annotations : Pod 注解
- 15) InheritFrom ：继承的一个或多个Pod模板的列表
- 16) slaveConnectTimeout : 代理程序联机超时时间（单位s）
- 17) podRetention ：控制保留代理Pod的行为(`Can be 'never()', 'onFailure()', 'always()', or 'default()'`),如果为空则按照下面的active Deadline Seconds参数进行。
- 18) activeDeadlineSeconds ： pod将在这个截止日期过后被删除。
- 19) idleMinutes : 允许Pod保持活动状态以供重用，直到在执行最后一步以来经过了配置的分钟数为止。
- 20) runAsUser : 用户ID，用于将pod中的所有容器运行为。
- 21) runAsGroup : 组ID，用于将Pod中的所有容器运行为。
- 22) hostNetwork : 使用主机网络。
- 23) container  : 用于创建pod容器的容器模板 (见下文) - (重点)。
- 24) defaultContainer : 指定Pod中默认容器则在stages中不用加入container进行指定选择;
```

描述: containerTemplate 是一个将被添加到pod中的容器模板它属于container中数组对象。同样它可以通过用户界面或管道进行配置，并允许你设置以下字段:

**containerTemplate - 容器模板**

-   name : 容器的名称。
    
-   image : 容器的图像。
    
-   envVars : 应用于容器的环境变量（`补充和覆盖在pod级别设置的env var`）。
    

-   envVar 环境变量，其值是内联定义的。
    
-   secretEnvVar 一个环境变量，其值是从Kubernetes机密派生的。
    

-   workingDir : 工作空间目录
    
-   command : 容器将执行的命令。
    
-   args : 传递给命令的参数。
    
-   ttyEnabled : 标记以指示应启用tty。
    
-   livenessProbe : 要添加到容器中的exec liveness探针的参数（不支持httpGet liveness探针）
    
-   ports : 暴露容器上的端口。
    
-   alwaysPullImage : 容器在启动时将拉出图像。
    
-   runAsUser : 用于运行容器的用户ID。
    
-   runAsGroup : 运行容器的组ID。
    

**Liveness Probe Usage**

```
containerTemplate(
  name: 'busybox',
  image: 'busybox',
  ttyEnabled: true,
  command: 'cat',
  livenessProbe: containerLivenessProbe(
    execArgs: 'some --command',
    initialDelaySeconds: 30,
    timeoutSeconds: 1,
    failureThreshold: 3,
    periodSeconds: 10,
    successThreshold: 1
  )
)
```

Tips : 默认情况下`代理连接超时设置为100秒`。在某些情况下您想要设置一个不同的值，如果可以可以将system属性设置`org.csanchez.jenkins.plugins.kubernetes.PodTemplate.connectionTimeout`为一个不同的值

描述： Scripted Pipeline 示例参考:

-   https://github.com/jenkinsci/kubernetes-plugin/tree/master/examples
    
-   https://github.com/jenkinsci/kubernetes-plugin/tree/master/src/test/resources/org/csanchez/jenkins/plugins/kubernetes/pipeline
    

**(1) containerTemplate**

```
podTemplate(
  cloud: 'kubernetes',
  name: 'jenkins-slave',
  namespace: 'devops',
  label: 'k8s-slave',
  // 容器及容器模板定义
  containers: [
    containerTemplate(
      name: 'maven',
      image: 'maven:3.6-jdk-8-alpine',
      ttyEnabled: true,
      command: 'cat',
      privileged: false,
      alwaysPullImage: false,
      workingDir: '/home/jenkins/agent',
      resourceRequestCpu: '100m',
      resourceLimitCpu: '500m',
      resourceRequestMemory: '100Mi',
      resourceLimitMemory: '500Mi',
      envVars: [
        envVar(key: 'MYSQL_ALLOW_EMPTY_PASSWORD', value: 'true')
       //, secretEnvVar(key: 'MYSQL_PASSWORD', secretName: 'mysql-secret', secretKey: 'password')
      ]
      //,ports: [portMapping(name: 'mysql', containerPort: 3306, hostPort: 3306)]
    )
    //     ),
    // containerTemplate(
    //         name: 'jnlp',
    //         image: 'registry.cn-hangzhou.aliyuncs.com/google-containers/jnlp-slave:alpine',
    //         args: '${computer.jnlpmac} ${computer.name}',
    //         command: ''
    //     )
  ]
  ,volumes: [
    hostPathVolume(hostPath: '/nfsdisk-31/appstorage/mavenRepo', mountPath: '/home/jenkins'),
    hostPathVolume(hostPath: '/var/run/docker.sock', mountPath: '/var/run/docker.sock'),
    /*
    emptyDirVolume(mountPath: '/etc/mount1', memory: false),
    secretVolume(mountPath: '/etc/mount2', secretName: 'my-secret'),
    configMapVolume(mountPath: '/etc/mount3', configMapName: 'my-config'),
    hostPathVolume(mountPath: '/etc/mount4', hostPath: '/mnt/my-mount'),
    nfsVolume(mountPath: '/etc/mount5', serverAddress: '127.0.0.1', serverPath: '/', readOnly: true),
    persistentVolumeClaim(mountPath: '/home/jenkins', claimName: 'jenkins', readOnly: false),
    */
  ],
  annotations: [
    podAnnotation(key: "maven-pod", value: "Kubernetes-jenkins-Test")
  ],
  showRawYaml: 'flase'
  //,defaultContainer: 'maven'   # scripted pipeline 不支持该参数
  //,imagePullSecrets: [ 'pull-secret' ]
)
{
 // 注意此次node中则为Pod模板名称
 node ('k8s-slave') {
    // container 中指定Pod中容器名称
    container('maven') {
      stage('maven') {
        container('maven') {
          sh "hostname && ip addr"
          sh ("mvn -version")
          sh "env"
        }
      }
   }
  }
}
```

运行结果:

```
# Still waiting to schedule task
# Waiting for next available executor on 'jenkins-slave-09vf0-sf3k6'
# 等待下一个可用的执行器启动

Created Pod: kubernetes devops/jenkins-slave-rtjxx-q3nhn
[Normal][devops/jenkins-slave-rtjxx-q3nhn][Scheduled] Successfully assigned devops/jenkins-slave-rtjxx-q3nhn to work-223

[Normal][devops/jenkins-slave-rtjxx-q3nhn][Pulled] Container image "maven:3.6-jdk-8-alpine" already present on machine
[Normal][devops/jenkins-slave-rtjxx-q3nhn][Created] Created container maven | Started container maven

[Normal][devops/jenkins-slave-rtjxx-q3nhn][Pulled] Container image "jenkins/inbound-agent:4.3-4" already present on machine
[Normal][devops/jenkins-slave-rtjxx-q3nhn][Created] Created container jnlp | Started container jnlp

Agent jenkins-slave-rtjxx-q3nhn is provisioned from template jenkins-slave-rtjxx

# ----------------------------------
apiVersion: "v1"
kind: "Pod"
metadata:
  annotations:
    maven-pod: "Kubernetes-jenkins-Test"   # 我们添加的注解
    buildUrl: "http://jenkins.devops.svc.cluster.local:8080/job/Kubernetes-jenkins-slave/23/"
    runUrl: "job/Kubernetes-jenkins-slave/23/"
  labels:
    jenkins: "slave"
    jenkins/label-digest: "823e2bb3db243573786b7a360a483f1acc7a0f96"
    jenkins/label: "k8s-slave"
  name: "jenkins-slave-rtjxx-q3nhn"
spec:
  containers:
  - command:
    - "cat"   # 运行的命令
    env:
    - name: "MYSQL_ALLOW_EMPTY_PASSWORD"  # 运行的环境变量
      value: "true"
    image: "maven:3.6-jdk-8-alpine"
    imagePullPolicy: "IfNotPresent"    # 镜像拉取策略
    name: "maven"                      # 容器名称
    resources:
      limits:
        memory: "500Mi"
        cpu: "500m"
      requests:
        memory: "100Mi"
        cpu: "100m"
    tty: true
    volumeMounts:                      # 卷挂载情况
    - mountPath: "/home/jenkins"
      name: "volume-0"
      readOnly: false
    - mountPath: "/var/run/docker.sock"
      name: "volume-1"
      readOnly: false
    - mountPath: "/home/jenkins/agent"
      name: "workspace-volume"
      readOnly: false
    workingDir: "/home/jenkins/agent"
  - env:                                   # jnlp 环境变量
    - name: "JENKINS_SECRET"
      value: "********"
    - name: "JENKINS_AGENT_NAME"
      value: "jenkins-slave-rtjxx-q3nhn"
    - name: "JENKINS_NAME"
      value: "jenkins-slave-rtjxx-q3nhn"
    - name: "JENKINS_AGENT_WORKDIR"
      value: "/home/jenkins/agent"
    - name: "JENKINS_URL"
      value: "http://jenkins.devops.svc.cluster.local:8080/"
    image: "jenkins/inbound-agent:4.3-4"
    name: "jnlp"                         # jenkins-jnlp 容器的名称
    resources:
      limits: {}
      requests:
        memory: "256Mi"
        cpu: "100m"
    volumeMounts:
    - mountPath: "/home/jenkins"
      name: "volume-0"
      readOnly: false
    - mountPath: "/var/run/docker.sock"
      name: "volume-1"
      readOnly: false
    - mountPath: "/home/jenkins/agent"
      name: "workspace-volume"
      readOnly: false
  nodeSelector:
    kubernetes.io/os: "linux"
  restartPolicy: "Never"
  volumes:
  - hostPath:
      path: "/nfsdisk-31/appstorage/mavenRepo"
    name: "volume-0"
  - hostPath:
      path: "/var/run/docker.sock"
    name: "volume-1"
  - emptyDir:
      medium: ""
    name: "workspace-volume"

# ------------------------------
+ hostname
  # jenkins-slave-rtjxx-q3nhn

+ ip addr
  # 1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1000
  #     link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
  #     inet 127.0.0.1/8 scope host lo
  #        valid_lft forever preferred_lft forever
  # 2: tunl0@NONE: <NOARP> mtu 1480 qdisc noop state DOWN qlen 1000
  #     link/ipip 0.0.0.0 brd 0.0.0.0
  # 4: eth0@if162: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1480 qdisc noqueue state UP
  #     link/ether de:c6:df:23:e1:7f brd ff:ff:ff:ff:ff:ff
  #     inet 172.16.24.220/32 brd 172.16.24.220 scope global eth0
  #        valid_lft forever preferred_lft forever
+ mvn -version
  # Apache Maven 3.6.1 (d66c9c0b3152b2e69ee9bac180bb8fcc8e6af555; 2019-04-04T19:00:29Z)
  # Maven home: /usr/share/maven
  # Java version: 1.8.0_212, vendor: IcedTea, runtime: /usr/lib/jvm/java-1.8-openjdk/jre
  # Default locale: en_US, platform encoding: UTF-8
  # OS name: "linux", version: "5.4.0-42-generic", arch: "amd64", family: "unix"

+ env
  # JENKINS_HOME=/var/jenkins_home
  # POD_CONTAINER=maven
  # STAGE_NAME=maven
  # KUBERNETES_SERVICE_PORT=443
  # JAVA_VERSION=8u212
  # MAVEN_HOME=/usr/share/maven
  # MAVEN_CONFIG=/root/.m2
```

Tips : 在PodTemplate模板中设置 `showRawYaml: 'flase'` 则将不会显示资源清单在日志中;

![WeiyiGeek.脚本式流水线k8s插件使用](https://i0.hdslb.com/bfs/article/2901c438e7f2beff104bf0dfa4f08ba03ee417e0.png@942w_389h_progressive.webp)

Tips : 要使用 secretEnvVar 我们必须在 `Dashboard -> 凭据 -> 系统 -> 全局凭据 (unrestricted)` 进行添加一个Secret-text凭据(未能成功复现);

```
Secret text4fc93636-d8f1-4051-8011-d852873fc740mysql-secretSecret textmysql-secret
```

## Declarative Pipeline

描述: 声明式管道支持需要Jenkins 2.66+, 在Declarative Pipeline使用的 kubernetes 选项说明;

```
options are [activeDeadlineSeconds, cloud, containerTemplate, containerTemplates, customWorkspace, defaultContainer, idleMinutes, inheritFrom, instanceCap, label, namespace, nodeSelector, podRetention, serviceAccount, slaveConnectTimeout, supplementalGroups, workingDir, workspaceVolume, yaml, yamlFile, yamlMergeStrategy]
```

示例帮助: https://github.com/jenkinsci/kubernetes-plugin/tree/master/src/main/resources/org/csanchez/jenkins/plugins/kubernetes/pipeline/samples

**示例1.通过yaml文件资源清单创建**

```
pipeline {
  agent {
    kubernetes {
      cloud 'kubernetes'
      namespace 'devops'
      workingDir '/home/jenkins/agent'
      // 继承模板
      inheritFrom 'jenkins-slave'
      defaultContainer 'maven'
      // yamlFile 'KubernetesPod.yaml'
      yaml """\
apiVersion:
kind: Pod
metadata:
  labels:
    jenkins: "slave"
    jenkins/label: 'k8s-slave'
spec:
  containers:
  - name: 'maven'
    image: 'maven:3.6-jdk-8-alpine'
    imagePullPolicy: 'IfNotPresent'     # 镜像拉取策略
    command:
    - cat
    tty: true
    """.stripIndent()
    }
  }
  stages {
    stage ('declarative Pipeline - kubernetes') {
      steps {
        echo "declarative Pipeline - kubernetes"
        container ('maven') {
          sh "echo ${POD_CONTAINER}"
          sh "mvn -version"
        }
      }
    }
  }
}
```

执行结果:

```
[Pipeline] Start of Pipeline
[Pipeline] podTemplate
[Pipeline] {
[Pipeline] node
# 'kubernetes-jenkins-slave-49-fcd1s-m3n3q-t33d8' is offline
# Agent kubernetes-jenkins-slave-49-fcd1s-m3n3q-t33d8 is provisioned from template Kubernetes-jenkins-slave_49-fcd1s-m3n3q
---
kind: "Pod"
metadata:
  annotations:
    buildUrl: "http://jenkins.devops.svc.cluster.local:8080/job/Kubernetes-jenkins-slave/49/"
    runUrl: "job/Kubernetes-jenkins-slave/49/"
  labels:
    jenkins: "slave"
    jenkins/label: "Kubernetes-jenkins-slave_49-fcd1s"
    jenkins/label-digest: "95508e0ca05226ba84989959dc886ad4011cf125"
  name: "kubernetes-jenkins-slave-49-fcd1s-m3n3q-t33d8"
spec:
  containers:
  - command:
    - "cat"
    image: "maven:3.6-jdk-8-alpine"
    imagePullPolicy: "IfNotPresent"
    name: "maven"
    tty: true
    volumeMounts:
    - mountPath: "/home/jenkins/agent"
      name: "workspace-volume"
      readOnly: false
  - env:
    - name: "JENKINS_SECRET"
      value: "********"
    - name: "JENKINS_AGENT_NAME"
      value: "kubernetes-jenkins-slave-49-fcd1s-m3n3q-t33d8"
    - name: "JENKINS_NAME"
      value: "kubernetes-jenkins-slave-49-fcd1s-m3n3q-t33d8"
    - name: "JENKINS_AGENT_WORKDIR"
      value: "/home/jenkins/agent"
    - name: "JENKINS_URL"
      value: "http://jenkins.devops.svc.cluster.local:8080/"
    image: "jenkins/inbound-agent:4.3-4"
    name: "jnlp"
    resources:
      limits: {}
      requests:
        memory: "256Mi"
        cpu: "100m"
    volumeMounts:
    - mountPath: "/home/jenkins/.m2"
      name: "volume-0"
      readOnly: false
    - mountPath: "/home/jenkins/agent"
      name: "workspace-volume"
      readOnly: false
  hostNetwork: false
  nodeSelector:
    kubernetes.io/os: "linux"
  restartPolicy: "Never"
  volumes:
  - hostPath:
      path: "/nfsdisk-31/appstorage/mavenRepo"
    name: "volume-0"
  - emptyDir:
      medium: ""
    name: "workspace-volume"

Running on kubernetes-jenkins-slave-49-fcd1s-m3n3q-t33d8 in /home/jenkins/agent/workspace/Kubernetes-jenkins-slave
  # [Pipeline] {
  # [Pipeline] container
  # [Pipeline] {
  # [Pipeline] stage
  # [Pipeline] { (declarative Pipeline - kubernetes)
[Pipeline] echo
declarative Pipeline - kubernetes
  # [Pipeline] container
  # [Pipeline] {
  # [Pipeline] sh
+ echo maven
maven
  # [Pipeline] sh
+ mvn -version
Apache Maven 3.6.1 (d66c9c0b3152b2e69ee9bac180bb8fcc8e6af555; 2019-04-04T19:00:29Z)
Maven home: /usr/share/maven
Java version: 1.8.0_212, vendor: IcedTea, runtime: /usr/lib/jvm/java-1.8-openjdk/jre
Default locale: en_US, platform encoding: UTF-8
OS name: "linux", version: "5.4.0-42-generic", arch: "amd64", family: "unix"
  # [Pipeline] }
  # [Pipeline] // container
  # [Pipeline] }
  # [Pipeline] // stage
  # [Pipeline] }
  # [Pipeline] // container
  # [Pipeline] }
  # [Pipeline] // node
  # [Pipeline] }
  # [Pipeline] // podTemplate
  # [Pipeline] End of Pipeline
Finished: SUCCESS
```

Tips :默认情况下在容器中运行步骤, 步骤将嵌套在隐式container（name）{...}块中，而不是在jnlp容器中执行。

```
defaultContainer 'maven'
```

Tips : 您也可以使用yamlFile将pod模板保存在单独的KubernetesPod.yaml文件中

```
pipeline {
  agent {
    kubernetes {
      yamlFile 'KubernetesPod.yaml'
    }
  }
  stages {
     ...
  }
}
```

![](https://i0.hdslb.com/bfs/article/4adb9255ada5b97061e610b682b8636764fe50ed.png@progressive.webp)

> 欢迎各位志同道合的朋友一起学习交流，如文章有误请在下方留下您宝贵的经验知识，个人邮箱地址【master#weiyigeek.top】  

![](https://i0.hdslb.com/bfs/article/02db465212d3c374a43c60fa2625cc1caeaab796.png@progressive.webp)

更多文章来源: https://weiyigeek.top 【WeiyiGeek Blog - 为了能到远方，脚下的每一步都不能少】

![](https://i0.hdslb.com/bfs/article/d25e6027ff6931148f98334450b5e4797c2d0b3e.png@942w_483h_progressive.webp)

如果你觉得这个专栏还不错的，请给这篇专栏点个赞、投个币、收个藏、关个注，这将对我有很大帮助！