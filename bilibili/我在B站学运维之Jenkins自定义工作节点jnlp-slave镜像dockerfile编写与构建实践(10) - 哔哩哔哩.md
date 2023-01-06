**本章目录:**

-   0x02 自定义Jenkins工作节点之jenkins-jnlp-slave镜像模板构建
    

-   Dockerfile 构建依赖
    
-   Dockerfile 构建操作
    
-   使用 jenkins-jnlp-slave 镜像
    

## 0x02 自定义Jenkins工作节点之jnlp-slave镜像模板构建

## Dockerfile 构建依赖

自定义的jnlp-slave镜像容器模板主要实现功能:  

-   用户权限控制(sudo)
    
-   ssh 远程连接
    
-   git 代码版本控制
    
-   docker 容器管理
    
-   kubectl 集群管理
    
-   Java 运行环境
    
-   Maven 运行环境
    
-   SonarQube 扫描环境
    
-   Gitlab\_release 上传环境
    
-   中文环境支持
    
-   时区更改配置
    

Tips: 在K8s集群中测试alpine镜像执行相应的安装(需要在Alpine调试新安装软件时使用):

```
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: alpine-app
spec:
  containers:
  - name: alpine-app
    image: alpine:latest
    args:
    - sleep
    - "100000"
EOF

pod/alpine-app created

$ kubectl get pods -o wide
  # NAME                                       READY   STATUS    RESTARTS   AGE    IP              NODE        NOMINATED NODE   READINESS GATES
  # alpine-app                                 1/1     Running   0          116s   10.100.37.194   worker-02   <none>           <none>
```

**Dockerfile**

自定义jnlp容器模板镜像地址: https://hub.docker.com/r/weiyigeek/alpine-jenkins-jnlp

该镜像是Jenkins自定义jnlp容器模板，主要用于Jenkins工作节点容器化使用，主要包含Gitlab\_release发布、 docker 容器管理、kubectl 集群管理功能。我们可以直接在docker 的环境中 `docker pull weiyigeek/alpine-jenkins-jnlp`下来

说明: 镜像构建文件

```
#----------------------------------------------------------------------#
# Title: Base in Alpine Images Create Custom Jenkins Kubernetes jnlp Images
# Author: WeiyiGeek
# WebSite: https://weiyigeek.top
# MainFunction:
#   Install ssh-server docker git openssh tzdata curl tar sudo git ca-certificates wget unzip docker zlib nodejs npm jq
#   Install JDK8
#   - https://www.oracle.com/java/technologies/javase/javase-jdk8-downloads.html
#   - https://github.com/sgerrand/alpine-pkg-glibc/releases/
#   - https://alpine-pkgs.sgerrand.com/sgerrand.rsa.pub
#   Install jnlp
#   - https://repo.jenkins-ci.org/public/org/jenkins-ci/main/remoting/
#   Install Maven
#   - https://apache.osuosl.org/maven/maven-3/${MAVEN_VERSION}/binaries
#   Install SonarqubeScan
#   - https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli-4.5.0.2216-linux.zip
#   Install Gitlab Release Version: 0.0.6
#   - https://gitlab.com/gitlab-org/release-cli/-/releases
#   Install kubernetes cli
#   - kubectl Version: 1.19.3
# Version: alpine-3.13.1
# Release: v1.10
# ChangeLog:
# v1.9 -  增加 中文环境
# v1.10 - 增加 node.js 环境支持
#-------------------------------------------------#

FROM alpine:3.13.1

MAINTAINER  Jenkins Custom Work Node Jnlp Container - <master@weiyigeek.top> - WeiyiGeek

ARG USERNAME=jenkins \
    AGENT_WORKDIR=/home/jenkins \
    BASE_DIR=/usr/local  \
```

完整的镜像的Dockerfile构建文件原文地址: https://mp.weixin.qq.com/s/FWmQNinsYZwyeL3c-6x6tw

**local.md**  

描述：让alpine操作系统中文字符支持;

```
tee local.md <<'END'
en_AG
en_AU
en_BW
en_CA
en_DK
en_GB
en_HK
en_IE
en_IN
en_NG
en_NZ
en_PH
en_SG
en_US
en_ZA
en_ZM
en_ZW
zh_CN
zh_HK
zh_SG
zh_TW
zu_ZA
END
```

**jenkins-agent.sh**  
描述: 容器启动时与`jenkins`利用Java jnlp 进行连接的脚本;

```
# Usage jenkins-agent.sh [options] -url http://jenkins [SECRET] [AGENT_NAME]
# Optional environment variables :
# * JENKINS_TUNNEL : HOST:PORT for a tunnel to route TCP traffic to jenkins host, when jenkins can't be directly accessed over network
# * JENKINS_URL : alternate jenkins URL
# * JENKINS_SECRET : agent secret, if not set as an argument
# * JENKINS_AGENT_NAME : agent name, if not set as an argument
# * JENKINS_AGENT_WORKDIR : agent work directory, if not set by optional parameter -workDir
# * JENKINS_WEB_SOCKET: true if the connection should be made via WebSocket rather than TCP
# * JENKINS_DIRECT_CONNECTION: Connect directly to this TCP agent port, skipping the HTTP(S) connection parameter download.
#                              Value: "<HOST>:<PORT>"
# * JENKINS_INSTANCE_IDENTITY: The base64 encoded InstanceIdentity byte array of the Jenkins master. When this is set,
#                              the agent skips connecting to an HTTP(S) port for connection info.
# * JENKINS_PROTOCOLS:         Specify the remoting protocols to attempt when instanceIdentity is provided.

if [ $# -eq 1 ]; then
        # if `docker run` only has one arguments, we assume user is running alternate command like `bash` to inspect the image
        exec "$@"
else
        # if -tunnel is not provided, try env vars
        case "$@" in
                *"-tunnel "*) ;;
                *)
                if [ ! -z "$JENKINS_TUNNEL" ]; then
                  TUNNEL="-tunnel $JENKINS_TUNNEL"
                fi ;;
        esac

        # if -workDir is not provided, try env vars
        if [ ! -z "$JENKINS_AGENT_WORKDIR" ]; then
                case "$@" in
                        *"-workDir"*) echo "Warning: Work directory is defined twice in command-line arguments and the environment variable" ;;
                        *)
                        WORKDIR="-workDir $JENKINS_AGENT_WORKDIR" ;;
                esac
        fi

        if [ -n "$JENKINS_URL" ]; then
                URL="-url $JENKINS_URL"
        fi

        if [ -n "$JENKINS_NAME" ]; then
                JENKINS_AGENT_NAME="$JENKINS_NAME"
        fi

        if [ "$JENKINS_WEB_SOCKET" = true ]; then
                WEB_SOCKET=-webSocket
        fi

        if [ -n "$JENKINS_PROTOCOLS" ]; then
                PROTOCOLS="-protocols $JENKINS_PROTOCOLS"
        fi

        if [ -n "$JENKINS_DIRECT_CONNECTION" ]; then
                DIRECT="-direct $JENKINS_DIRECT_CONNECTION"
        fi

        if [ -n "$JENKINS_INSTANCE_IDENTITY" ]; then
                INSTANCE_IDENTITY="-instanceIdentity $JENKINS_INSTANCE_IDENTITY"
        fi

        # if java home is defined, use it
        JAVA_BIN="java"
        if [ "$JAVA_HOME" ]; then
                JAVA_BIN="$JAVA_HOME/bin/java"
        fi

        # if both required options are defined, do not pass the parameters
        OPT_JENKINS_SECRET=""
        if [ -n "$JENKINS_SECRET" ]; then
                case "$@" in
                        *"${JENKINS_SECRET}"*) echo "Warning: SECRET is defined twice in command-line arguments and the environment variable" ;;
                        *)
                        OPT_JENKINS_SECRET="${JENKINS_SECRET}" ;;
                esac
        fi

        OPT_JENKINS_AGENT_NAME=""
        if [ -n "$JENKINS_AGENT_NAME" ]; then
                case "$@" in
                        *"${JENKINS_AGENT_NAME}"*) echo "Warning: AGENT_NAME is defined twice in command-line arguments and the environment variable" ;;
                        *)
                        OPT_JENKINS_AGENT_NAME="${JENKINS_AGENT_NAME}" ;;
                esac
        fi

        #TODO: Handle the case when the command-line and Environment variable contain different values.
        #It is fine it blows up for now since it should lead to an error anyway.

        exec $JAVA_BIN $JAVA_OPTS -cp /usr/local/bin/agent.jar hudson.remoting.jnlp.Main -headless $TUNNEL $URL $WORKDIR $WEB_SOCKET $DIRECT $PROTOCOLS $INSTANCE_IDENTITY $OPT_JENKINS_SECRET $OPT_JENKINS_AGENT_NAME "$@"
fi
```

**镜像构建所需软件**  
备注: 在Jenkins 2.277版本中添加一个新的节点中获取匹配当前版本的 agent.jar, 或者是在 jenkins 官网 https://repo.jenkins-ci.org/public/org/jenkins-ci/main/remoting 进行下载;

```
~/k8s/jenkins/jnlp-slave$ ls
agent.jar                      build              jdk-8u281-linux-x64.tar.gz  Jenkins.zip  release-cli-0.6.0-linux-amd64  sonar-scanner-cli-4.5.0.2216-linux.zip
apache-maven-3.6.3-bin.tar.gz  glibc-2.32-r0.apk  jenkins-agent.sh            kubectl      sgerrand.rsa.pub
```

## Dockerfile 构建操作

**操作流程**

```
# (0) 启动一个临时的web服务器（`存放上面的镜像构建所需软件`-非常重要-否则将会导致构建失败）
~/k8s/jenkins/jnlp-slave$ python3 -m http.server 8080
Serving HTTP on 0.0.0.0 port 8080 (http://0.0.0.0:8080/) ...
# 172.17.0.4 - - [30/Mar/2021 17:32:00] "GET /glibc-2.32-r0.apk HTTP/1.1" 200 -
# 172.17.0.4 - - [30/Mar/2021 17:32:00] "GET /glibc-bin-2.32-r0.apk HTTP/1.1" 200 -
# 172.17.0.4 - - [30/Mar/2021 17:32:00] "GET /glibc-i18n-2.32-r0.apk HTTP/1.1" 200 -
# 172.17.0.4 - - [30/Mar/2021 17:32:00] "GET /sgerrand.rsa.pub HTTP/1.1" 200 -
# 172.17.0.4 - - [30/Mar/2021 17:32:00] "GET /locale.md HTTP/1.1" 200 -
# 172.17.0.4 - - [30/Mar/2021 17:32:00] "GET /jdk-8u281-linux-x64.tar.gz HTTP/1.1" 200 -
# 172.17.0.4 - - [30/Mar/2021 17:32:00] "GET /agent.jar HTTP/1.1" 200 -
# 172.17.0.4 - - [30/Mar/2021 17:32:00] "GET /jenkins-agent.sh HTTP/1.1" 200 -
# 172.17.0.4 - - [30/Mar/2021 17:32:00] "GET /apache-maven-3.6.3-bin.tar.gz HTTP/1.1" 200 -
# 172.17.0.4 - - [30/Mar/2021 17:32:00] "GET /sonar-scanner-cli-4.5.0.2216-linux.zip HTTP/1.1" 200 -
# 172.17.0.4 - - [30/Mar/2021 17:32:00] "GET /release-cli-0.6.0-linux-amd64 HTTP/1.1" 200 -
# 172.17.0.4 - - [30/Mar/2021 17:32:00] "GET /kubectl HTTP/1.1" 200 -

# (1) 镜像构建
docker build -t weiyigeek/alpine-jenkins-jnlp .
  # Successfully built b47b6581b712
  # Successfully tagged weiyigeek/alpine-jenkins-jnlp

# (2) 查看构建信息
docker images weiyigeek/alpine-jenkins-jnlp --all
  # REPOSITORY                               TAG           IMAGE ID       CREATED             SIZE
  # harbor.weiyigeek.top/devops/jenkins-jnlp   3.13.1-alpine   b47b6581b712   About an hour ago   715MB

# (3) 推送镜像
docker push weiyigeek/alpine-jenkins-jnlp
  # The push refers to repository [harbor.weiyigeek.top/devops/jenkins-jnlp]
  # 059ea3fbd3b3: Pushed
  # 1119ff37d4a9: Layer already exists
  # 3.13.1-alpine: digest: sha256:1f869c553340c9399c7db9072169600a17ddb0ec41d41d3a4365b8c1571fc201 size: 741

# (4) 采用Ansible各节点拉取镜像
ansible node -m shell -a "docker pull weiyigeek/alpine-jenkins-jnlp"
```

![WeiyiGeek.DockerFile构建自定义Jnlp容器](https://i0.hdslb.com/bfs/article/2d4ad8fcb916f8cd831b2d90aed71a0425c9b626.png@942w_275h_progressive.webp)

描述: 此处还是利用Jenkins的kubernetes插件，让jenkins在进job时动态的生成工作节点(Pod)，并在结束后销毁容器。

-   Step 1.创建流水线 `Kubernetes-jenkins-slave` Job 在流水线中采用Pipeline Script脚本
    

```
pipeline {
  agent {
    kubernetes {
      cloud 'kubernetes'
      namespace 'devops'
      inheritFrom 'jenkins-slave'
      workingDir '/home/jenkins/agent'
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
  - name: 'jnlp'
    image: 'weiyigeek/alpine-jenkins-jnlp'
    imagePullPolicy: 'IfNotPresent'                                     # 镜像拉取策略
    command: ["/bin/sh","-c","/usr/local/bin/jenkins-agent.sh && cat"]  # 重点测试的时候(心酸累)希望读者体验一哈
    tty: true
    volumeMounts:
    - mountPath: "/home/jenkins/.m2"
      name: "volume-0"
    - mountPath: "/var/run/docker.sock"
      name: "volume-1"
    """.stripIndent()
    }
  }
  stages {
    stage ('declarative Pipeline - kubernetes') {
      steps {
        echo "declarative Pipeline - kubernetes"
        sh "mvn -version"
        sh "release-cli -v"
        sh "sonar-scanner -v"
        sh "kubectl version"
        sh "docker --version && sudo docker ps"
      }
    }
  }
}
```

-   Step 2.配置`Pod Templates`模板如下因为我们需要继承一哈
    
  

```
名称：jenkins-slave
命名空间: devops
标签列表: k8s-slave
添加主机卷:
 - 主机: /nfsdisk-31/appstorage/mavenRepo
 - Pod 挂载路径: /home/jenkins/.m2
 - 主机: /var/run/docker.sock
 - Pod 挂载路径: /var/run/docker.sock
```

-   Step 3.在BlueOcen中运行并查看结果
    

```
# (1) 可以看见当进行调度时k8s会动态的拉取镜像并运行，当任务结束后会自动销毁Pod
kubernetes-jenkins-slave-63-01h9j-txdgn-cdwb5   0/1     Pending   0          0s
kubernetes-jenkins-slave-63-01h9j-txdgn-cdwb5   0/1     Pending   0          0s
kubernetes-jenkins-slave-63-01h9j-txdgn-cdwb5   0/1     ContainerCreating   0          0s
kubernetes-jenkins-slave-63-01h9j-txdgn-cdwb5   1/1     Running             0          3s
kubernetes-jenkins-slave-63-01h9j-txdgn-cdwb5   1/1     Terminating         0          13s
kubernetes-jenkins-slave-63-01h9j-txdgn-cdwb5   0/1     Terminating         0          45s
kubernetes-jenkins-slave-63-01h9j-txdgn-cdwb5   0/1     Terminating         0          46s
kubernetes-jenkins-slave-63-01h9j-txdgn-cdwb5   0/1     Terminating         0          46s

# (2) 动态创建的slave启动的Pod脚本
---
kind: "Pod"
metadata:
  annotations:
    buildUrl: "http://jenkins.devops.svc.cluster.local:8080/job/Kubernetes-jenkins-slave/63/"
    runUrl: "job/Kubernetes-jenkins-slave/63/"
  labels:
    jenkins: "slave"
    jenkins/label: "Kubernetes-jenkins-slave_63-01h9j"
    jenkins/label-digest: "0fe6224168b9ce0350e7db8678c10953c2e6f533"
  name: "kubernetes-jenkins-slave-63-01h9j-txdgn-cdwb5"
spec:
  containers:
  - command:
    - "/bin/sh"
    - "-c"
    - "/usr/local/bin/jenkins-agent.sh && cat"
    env:
    - name: "JENKINS_SECRET"
      value: "********"
    - name: "JENKINS_AGENT_NAME"
      value: "kubernetes-jenkins-slave-63-01h9j-txdgn-cdwb5"
    - name: "JENKINS_NAME"
      value: "kubernetes-jenkins-slave-63-01h9j-txdgn-cdwb5"
    - name: "JENKINS_AGENT_WORKDIR"
      value: "/home/jenkins/agent"
    - name: "JENKINS_URL"
      value: "http://jenkins.devops.svc.cluster.local:8080/"
    image: "weiyigeek/alpine-jenkins-jnlp"
    imagePullPolicy: "IfNotPresent"
    name: "jnlp"
    resources:
      limits: {}
      requests:
        memory: "256Mi"
        cpu: "100m"
    tty: true
    volumeMounts:
    - mountPath: "/home/jenkins/.m2"
      name: "volume-0"
    - mountPath: "/var/run/docker.sock"
      name: "volume-1"
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
  - hostPath:
      path: "/var/run/docker.sock"
    name: "volume-1"
  - emptyDir:
      medium: ""
    name: "workspace-volume"

# Running on kubernetes-jenkins-slave-63-01h9j-txdgn-cdwb5 in /home/jenkins/agent/workspace/Kubernetes-jenkins-slave

# 脚本反馈
declarative Pipeline - kubernetes
+ mvn -version
  Apache Maven 3.6.3 (cecedd343002696d0abb50b32b541b8a6ba2883f)
  Maven home: /usr/local/maven
  Java version: 1.8.0_281, vendor: Oracle Corporation, runtime: /usr/local/jdk8/jre
  Default locale: en_US, platform encoding: ANSI_X3.4-1968
  OS name: "linux", version: "5.4.0-42-generic", arch: "amd64", family: "unix"

+ release-cli -v
     version 0.6.0

+ sonar-scanner -v
  INFO: Scanner configuration file: /usr/local/sonar-scanner/conf/sonar-scanner.properties
  INFO: Project root configuration file: NONE
  INFO: SonarScanner 4.5.0.2216
  INFO: Java 1.8.0_281 Oracle Corporation (64-bit)
  INFO: Linux 5.4.0-42-generic amd64

+ kubectl version
  Client Version: version.Info{Major:"1", Minor:"19", GitVersion:"v1.19.6", GitCommit:"fbf646b339dc52336b55d8ec85c181981b86331a", GitTreeState:"clean", BuildDate:"2020-12-18T12:09:30Z", GoVersion:"go1.15.5", Compiler:"gc", Platform:"linux/amd64"}
  Server Version: version.Info{Major:"1", Minor:"19", GitVersion:"v1.19.6", GitCommit:"fbf646b339dc52336b55d8ec85c181981b86331a", GitTreeState:"clean", BuildDate:"2020-12-18T12:01:36Z", GoVersion:"go1.15.5", Compiler:"gc", Platform:"linux/amd64"}

+ docker --version
  Docker version 20.10.3, build 48d30b5b32e99c932b4ea3edca74353feddd83ff

+ sudo docker ps  # 非常注意权限
CONTAINER ID   IMAGE                                                           COMMAND                  CREATED              STATUS          PORTS     NAMES
bd9948326a78   harbor.weiyigeek.top/devops/jenkins-jnlp                          "/bin/sh -c '/usr/lo…"   39 seconds ago       Up 32 seconds             k8s_jnlp_kubernetes-jenkins-slave-68-91jbw-svtg2-3vq9b_devops_6d4bc0d2-b34f-46fc-9c9d-65537cb2bff8_0
84538d2017c2   registry.cn-hangzhou.aliyuncs.com/google_containers/pause:3.2   "/pause"                 About a minute ago   Up 58 seconds             k8s_POD_kubernetes-jenkins-slave-68-91jbw-svtg2-3vq9b_devops_6d4bc0d2-b34f-46fc-9c9d-65537cb2bff8_0

Finished: SUCCESS
```

如需企业实践 Jenkins 流线脚本和上述相关脚本及文件的朋友请访问此处获取: https://weiyigeek.top/wechat.html?key=alpine-jenkins-jnlp

![WeiyiGeek.custom-jenkins-jnlp](https://i0.hdslb.com/bfs/article/2ff0a672a4759ba7d8c5158816d65d38df8913f8.png@942w_713h_progressive.webp)

至此动态工作节点之jenkins-jnlp-slave镜像构建到使用介绍完毕！谢谢大家的支持

![](https://i0.hdslb.com/bfs/article/4adb9255ada5b97061e610b682b8636764fe50ed.png@progressive.webp)

> 欢迎各位志同道合的朋友一起学习交流，如文章有误请在下方留下您宝贵的经验知识，个人邮箱地址【master#weiyigeek.top】  

![](https://i0.hdslb.com/bfs/article/02db465212d3c374a43c60fa2625cc1caeaab796.png@progressive.webp)

更多文章来源: https://weiyigeek.top 【WeiyiGeek Blog - 为了能到远方，脚下的每一步都不能少】

![](https://i0.hdslb.com/bfs/article/d25e6027ff6931148f98334450b5e4797c2d0b3e.png@942w_483h_progressive.webp)

如果你觉得这个专栏还不错的，请给这篇专栏点个赞、投个币、收个藏、关个注，这将对我有很大帮助！