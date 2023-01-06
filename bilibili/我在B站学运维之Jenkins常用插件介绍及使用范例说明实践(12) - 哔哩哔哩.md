**本文目录:**

-   0x01 Jenkins 常用插件
    
-   0x02 插件使用说明与范例
    

-   (1) SSH-steps-Plugin
    
-   (2) Gitlab-Plugin
    
-   (3) Kubernetes-plugin
    
-   (4) qy-wechat-notification-plugin
    
-   (5) File Operations - Plugin
    

![](https://i0.hdslb.com/bfs/article/4aa545dccf7de8d4a93c2b2b8e3265ac0a26d216.png@progressive.webp)

Jenkins 持续集成常用插件一览表:

```
# 权限管理
Role-based Authorization Strategy  - 3.1 ： http://wiki.jenkins-ci.org/display/JENKINS/Role+Strategy+Plugin
Authorize Project Plugin - 项目权限管控

# ssh
SSH Credentials
SSH Slaves

# Gitlab
Git (4.5.2) - This plugin integrates Git with Jenkins.
Git client (3.6.0) - Utility plugin for Git support in Jenkins
Gitlab Authentication  (1.10) - This is the an authentication plugin using gitlab OAuth.
GitLab  (1.5.13) - This plugin allows GitLab to trigger Jenkins builds and display their results in the GitLab UI.
Credentials (2.3.14) - This plugin allows you to store credentials in Jenkins.
GitLab Branch Source - 提供分支源代码和文件夹组织功能的GitLab仓库在詹金斯

# Svn
Subversion Plug-in  - svn 仓库专用

# pipeline 编写
Pipeline
Pipeline: Groovy - 2.87
Groovy  - 2.3
Groovy Postbuild  - 2.5

# 质量检测
SonarQube Scanner - 2.13

# 上下游构建
Parameterized Trigger
build-pipeline-plugin  - 可视化 build pipeline 插件

# 构建工具
Maven Integration - 3.8
Config File Provider
Node and Label parameter

# 消息通知
Qy Wechat Notification Plugin - 1.0.2
Email Extension Plugin  # 自带通知功能相对单一

# 容器管理
Kubernetes plugin - Jenkins插件可在Kubernetes集群中运行动态代理。

# 节点添加
SSH Agent - This plugin allows you to provide SSH credentials to builds via a ssh-agent in Jenkins

# 文件目录管理
File Operations - 1.11
```

项目描述: Jenkins流水线步骤，提供SSH工具，如命令执行或文件传输，以实现持续交付。  
项目地址: https://github.com/jenkinsci/ssh-steps-plugin

```
Version: 2.0.0
Released: 2 years ago
Requires Jenkins 2.121.1
```

**1) 在Jenkins中设置一个安全文本(Secret text)票据, 不建议使用明文票据;**

```
# Secret texta2bc53c0-0b68-4fce-9f1f-d04815ae52c1k8s-192.168.12.215Secret textk8s-192.168.12.215
```

Delactive Pipeline Script:

```
pipeline {
  stages {
    stage ('ssh-steps-plugin') {
      steps {
        script {
            try {
              def remote = [:]
              remote.name = 'K8s-Master'
              remote.user = 'root'
              remote.host = '192.168.12.215'
              //remote.password = 'password'  // 明文不安全
              remote.allowAnyHosts = true     // 必须要运行所有主机
              withCredentials([string(credentialsId: 'a2bc53c0-0b68-4fce-9f1f-d04815ae52c1', variable: 'sshpassword')]) {
                remote.password = sshpassword
                writeFile file: 'abc.sh', text: 'ls'
                sshPut remote: remote, from: 'abc.sh', into: '.'  // 非常注意 不能为其它目录只能在当前路径之中
                sshGet remote: remote, from: 'abc.sh', into: 'bac.sh', override: true
                sshScript remote: remote, script: 'abc.sh'
                sshRemove remote: remote, path: 'abc.sh'
                sshCommand remote: remote, command: ' whoami && for i in {1..5}; do echo -n \"Loop \$i \"; date ; sleep 1; done'
              }
            } catch(Exception err) {
                echo err.toString()
                error "[-Error] : ssh-steps-plugin测试失败\n [-Msg] : ${err.getMessage()} "
            }
        }
      }
    }
  }
}
```

输出结果

```
# SSH Steps: sshCommand - Execute command on remote node.7s
# Host key checking is off. It may be vulnerable to man-in-the-middle attacks.
[jsch] Permanently added '192.168.12.215' (RSA) to the list of known hosts.
Connected to 192.168.12.215[192.168.12.215:22] (SSH-2.0-OpenSSH_7.4)
Started command 192.168.12.215#0: whoami
root
Success command 192.168.12.215#0: whoami
Disconnected from 192.168.12.215[192.168.12.215:22]
```

![WeiyiGeek.SSH-STEPS](https://i0.hdslb.com/bfs/article/3af6d14ab2ff64b67835c9350a2c33fb3d28e8a3.png@942w_225h_progressive.webp)

**2) 官网示例中采用密钥进行验证操作**  
描述: 利用Jenkins凭据存储区读取私钥之后再进行ssh主机验证操作，但是需要注意高版本的Openssh的影响。

```
def remote = [:]
remote.name = "node-1"
remote.host = "10.000.000.153"
remote.allowAnyHosts = true

withCredentials([sshUserPrivateKey(credentialsId: 'sshUser', keyFileVariable: 'identity', passphraseVariable: '', usernameVariable: 'userName')]) {
  remote.user = userName
  remote.identityFile = identity
  stage("SSH Steps Rocks!") {
    writeFile file: 'abc.sh', text: 'ls'
    sshCommand remote: remote, command: 'for i in {1..5}; do echo -n \"Loop \$i \"; date ; sleep 1; done'
    sshPut remote: remote, from: 'abc.sh', into: '.'
    sshGet remote: remote, from: 'abc.sh', into: 'bac.sh', override: true
    sshScript remote: remote, script: 'abc.sh'
    sshRemove remote: remote, path: 'abc.sh'
  }
}
```

**3) 远程主机文件上传与删除**  
执行脚本:

```
try {
  def projectProduct = sh returnStdout: true, script: "find ${APP_FILE}"  // returnStatus: true
  def productName = sh returnStdout: true, script: "echo ${JOB_NAME}-${RELEASE_VERSION}${APP_SUFFIX}"
  def remote = [:]
  remote.name = "K8s-Master#${DEPLOY_HOST}"
  remote.user = "weiyigeek" // "${DEPLOY_USER}"
  remote.host = "192.168.12.107" //"${DEPLOY_HOST}"
  remote.port = 20211
  remote.allowAnyHosts = true     // 必须要运行所有主机
  withCredentials([string(credentialsId: "${DEPLOY_CREDENTIALSID}", variable: 'sshpassword')]) {
    remote.password = sshpassword
  }

  if (productName != '' && projectProduct != ''){
    println(projectProduct + "-" +  productName)
    writeFile file: 'projectProduct.sh', text: "${productName}"
  } else {
    error "[-Error] : projectProduct 不能为空!"

  }

  sshPut remote: remote, from: "/home/jenkins/agent/workspace/HelloWorld/target/hello-world.war" , into: "${APP_DIR}"
  sshCommand remote: remote, command: "ls -alh ${APP_DIR}"
}
```

输出结果:

```
target/hello-world.war - hello-world-v1.31.war

projectProduct.sh — Write file to workspace < 1s
+ pwd
/home/jenkins/agent/workspace/HelloWorld

SSH Steps: sshPut - Put a file/directory on remote node.6s
Sending a file/directory to `K8s-Master#192.168.12.215[192.168.12.107]`: from: `/home/jenkins/agent/workspace/HelloWorld/target/hello-world.war` into: /tmp/

SSH Steps: sshCommand - Execute command on remote node.1s

Executing command on K8s-Master#192.168.12.215[192.168.12.107]: ls -alh /tmp/ sudo: false
total 5.9M
-rw-rw-r--  1 weiyigeek weiyigeek 2.0M Feb 23 09:02 hello-world-v1.31.war
```

**4) 采用函数调用的方式**

```
!groovy
def getHost(){
    def remote = [:]
    remote.name = 'mysql'
    remote.host = '192.168.8.108'
    remote.user = 'root'
    remote.port = 22
    remote.password = 'qweasd'
    remote.allowAnyHosts = true
    return remote
}
pipeline {
    agent {label 'master'}
    environment{
        def server = ''
    }
    stages {
        stage('init-server'){
            steps {
                script {
                   server = getHost()
                }
            }
        }
        stage('use'){
            steps {
                script {
                  sshCommand remote: server, command: """
                  if test ! -d aaa/ccc ;then mkdir -p aaa/ccc;fi;cd aaa/ccc;rm -rf ./*;echo 'aa' > aa.log
                  """
                }
            }
        }
    }
}
```

**入坑出坑**  
问题1.使用ssh-steps-plugin插件并且使用Jenkins shh Private 凭据时jsch密钥连接远程Linux报错 com.jcraft.jsch.JSchException: invalid privatekey: \[B@277050dc  
报错信息:

```
exception in thread "main" com.jcraft.jsch.JSchException: invalid privatekey: [B@277050dc
at com.jcraft.jsch.KeyPair.load(KeyPair.java:664)
at com.jcraft.jsch.KeyPair.load(KeyPair.java:561)
at com.jcraft.jsch.IdentityFile.newInstance(IdentityFile.java:40)
at com.jcraft.jsch.JSch.addIdentity(JSch.java:407)
at com.jcraft.jsch.JSch.addIdentity(JSch.java:388)
at com.scc.nanny.ssh.SSH.<init>(SSH.java:59)
at com.scc.nanny.ssh.SSH.main(SSH.java:124)
```

问题原因: 主要原因是生成密钥的时候使用的openssh版本过高导致

```
# 高版本生成的密钥
-----BEGIN OPENSSH PRIVATE KEY-----
b3BlbnNzaC1rZXktdjEAAAAACmFlczI1Ni1jdHIAAAAGYmNyeXB0AAAAGAAAABDRomEs9d
........
AV8l4J6WvCkc3NAAAAEAAAAAEAAAEXAAAAB3NzaC1yc2EAAAADAQABAAABAQDD1C48/OJ3
-----END OPENSSH PRIVATE KEY-----

# ssh-steps-plugin 插件支持的密钥样式
-----BEGIN RSA PRIVATE KEY-----
Proc-Type: 4,ENCRYPTED
DEK-Info: AES-128-CBC,749E5AFD6F06C22EF6FD3AECCB6E540B

FwOlFOtCM+JH3EG7gzDOffkwLysiDCucdUeDZaK08rFSWzKpMwfPD/AZKNi0dqZR
....
-----END RSA PRIVATE KEY-----
```

解决办法: 不能在ssh-steps-plugin插件的版本使用Private验证只能使用密码验证,参考上述1步骤;

官方地址:  
帮助文档: https://github.com/jenkinsci/gitlab-plugin#pipeline-jobs

**Gitlab Triggers Setting**  
如果您想在声明式构建中配置插件支持的任何可选作业触发器请使用触发器块，完整的可配置触发选项列表如下:

```
// 触发器
triggers {
  // gitlab 相关触发
  gitlab(
    triggerOnPush: false,
    triggerOnMergeRequest: true, triggerOpenMergeRequestOnPush: "never",
    triggerOnNoteRequest: true,
    noteRegex: "Jenkins please retry a build",
    skipWorkInProgressMergeRequest: true,
    ciSkip: false,
    setBuildDescription: true,
    addNoteOnMergeRequest: true,
    addCiMessage: true,
    addVoteOnMergeRequest: true,
    acceptMergeRequestOnSuccess: false,
    branchFilterType: "NameBasedFilter",
    includeBranchesSpec: "release/qat",
    excludeBranchesSpec: "",
    pendingBuildName: "Jenkins",
    cancelPendingBuildsOnUpdate: false,
    secretToken: "abcdefghijklmnopqrstuvwxyz0123456789ABCDEF")
}
```

Tips : 注意:您将需要手动运行此作业一次，以便Jenkins读取和设置触发器配置。否则 webhook 将无法触发作业。

**Gitlab PipeLine Status Sync**  
描述: Jenkins 创建的 Pipeline 同步到 Gitlab-CI 流水线之中;

```
pipeline {
  agent any

  options {
    gitLabConnection('your-gitlab-connection-name')
    gitlabBuilds(builds: ['build', 'test', 'deploy'])
  }

  stage {
    stage("build") {
        gitlabCommitStatus("build") {
            // your build steps
        }
      }

      stage("test") {
        gitlabCommitStatus("test") {
            // your test steps
        }
      }
  }

  post {
    failure {
      updateGitlabCommitStatus name: 'build', state: 'failed'
    }
    success {
      updateGitlabCommitStatus name: 'build', state: 'success'
    }
  }
}
```

描述: 通过该插件我们可以实现 Jenkins 管理 Kubernetes 集群以及实现自动化部署;

Jenkins 服务有关 Kubernetes 的插件介绍:

-   1.Kubernetes Credentials 认证插件: 设置连接过程中使用到的信息，包括 Kubernetes Master 的链接地址、证书、用户名和命名空间等
    
-   2.Kubernetes CLI 管理插件: 结合上面的认证插件使用，通过 kubectl 客户端来管理 Kubernetes 服务，但是这个插件并不会在 Jenkins 服务所在主机上安装 Kubectl 工具，所以你需要自行安装。
    
-   3.Kubernetes 插件: 用于将 Jenkins 服务和 Kubernetes 服务结合起来, 使用其插件的前提条件是设置好 Kubernetes 服务的链接配置，并在 Pipeline 中使用相应的指令。该插件提供的指令有 `PodTemplate 、slaveTemplates、kubernetes` 等指令；而不是通过 Kubectl 客户端进行管理
    

Tips : 如果不想使用Kubernetes插件进行管理K8s集群, 我们可以设置一台服务器为 Kubernetes 服务的客户端，配置好 Kubectl 客户端；让 Jenkins 服务通过 SSH 方式连接到客户端执行管理命令。

**Help - 帮助声明:**

-   Kubernetes 插件Github : https://github.com/jenkinsci/kubernetes-plugin
    
-   Kubernetes 插件使用示例: https://github.com/jenkinsci/kubernetes-plugin/tree/master/src/main/resources/org/csanchez/jenkins/plugins/kubernetes/pipeline/samples
    

**基础示例:**

```
pipeline {
  // 确认使用主机/节点机
  agent any /*{ node { label 'master'} }*/

  // 声明使用的工具(便于不同环境构建)
  tools {
    maven 'maven_env'
    jdk   'jdk_k1.8.0_211'
  }

  // 流水线步骤声明
  stages {
    stage ("代码拉取") {
     steps {
      // (1) 代码拉取
      git credentialsId: 'b4c8b4e9-2777-44a1-a1ed-e9dc21d37f4f', url: 'git@gitlab.weiyigeek.top:ci-cd/java-maven.git'
      // (2) 超时时间两分钟
      timeout(time: 2, unit: 'MINUTES') {
          script {
            env.git_version=input message: '选择版本', ok: '通过',parameters: [string (description: '应用版本', name: 'git_version', trim: true)];
            env.deploy_option = input message: '选择操作', ok: 'deploy', parameters: [description: '选择部署环境',choice(name: 'deploy_option', choices: ['deploy', 'rollback', 'redeploy'])];
          }
      }
      // (3) 切换版本
      sh label: 'build', script: '[ -n "${git_version}" ] && git checkout ${git_version} || { echo -e "切换的tag的版本${git_version} 不存在或为空请检查输入的tag!" && exit 1; }'
     }
   }

    stage ("代码质检") {
     steps {
       sh label: 'sonar', returnStatus: true, script: '''/usr/local/bin/sonar-scanner -Dsonar.projectName=${JOB_NAME} -Dsonar.projectKey=Hello-World -Dsonar.sources=.'''
     }
    }

    stage ("项目构建") {
     steps {
      sh label: 'build', script: 'cd /var/lib/jenkins/workspace/maven-pipeline-helloword/ && /usr/local/maven/bin/mvn clean package -Dmaven.test.skip=true '
     }
    }

    stage ("项目部署") {
     steps {
         sh label: 'deploy', script: '/tmp/script/maven-jenkins-ci-script-pipeline-webhook.sh'
      }
    }
  }

  //消息通知: POST阶段当所有任务执行后触发
  post {
    always {
      echo "clear workspace......"
      deleteDir()
      echo "Wechat notification......"
      // 企业微信
      qyWechatNotification aboutSend: true, failNotify: true, failSend: true, mentionedId: 'ALL', mentionedMobile: '', startBuild: true, successSend: true, unstableSend: true, webhookUrl: 'https://qyapi.weixin.qq.com/cgi-bin/webhook/send?key=c222f3fc-f645-440a-ad24-0ce8d9626fa0'
    }
  }
}
```

PS : 钉钉机器人在Pipeline使用示例 https://jenkinsci.github.io/dingtalk-plugin/examples/link.html  

PS : 企业微信插件开源代码(在Pipeline无法获取Job\_Name) https://github.com/jenkinsci/qy-wechat-notification-plugin

描述: 在 Jenkins Pipeline 中，我们经常需要对文件、目录或者tar、zip等压缩包进行操作，比如移动、复制、重命名等等, 而采用sh复制正对于一些安全票据的操作时，会报出以下警告对于强迫症的我是接受不了的并且确实有安全隐患。

```
# 提示1.
The following steps that have been detected may have insecure interpolation of sensitive variables (click here for an explanation):
sh: [keyfile, user]

# 提示2.
Warning: A secret was passed to "sh" using Groovy String interpolation, which is insecure.
 Affected argument(s) used the following variable(s): [keyfile, user]
 See https://****.io/redirect/groovy-string-interpolation for details.
+ cp **** ./id.private
```

解决办法: 我们可以通过安装 `File Operations Plugin` 插件来解决上述问题。  
插件地址: https://plugins.jenkins.io/file-operations  
支持操作:

```
# Array / List of Nested Choice of Objects
fileCopyOperation
fileCreateOperation
fileDeleteOperation
fileDownloadOperation
fileJoinOperation
filePropertiesToJsonOperation
fileRenameOperation
fileTransformOperation
fileUnTarOperation
fileUnZipOperation
fileZipOperation
folderCopyOperation
folderCreateOperation
folderDeleteOperation
folderRenameOperation
```

基础示例:

```
# 1.创建目录
script {
  fileOperations([folderCreateOperation('directoryname')])
}

# 2.文件创建
script {
  fileOperations([fileCreateOperation(fileContent: 'devops', fileName: 'test.json')])
}

# 3.文件复制
# flattenFiles: 如果选中，文件将直接复制到目标位置，而不保留源文件子目录结构。
fileOperations ([
folderCopyOperation(includes: "*", targetLocation: ".", flattenFiles: true)
])


# 4.目录复制
fileOperations ([
folderCopyOperation(sourceFolderPath: "/path/to/src", destinationFolderPath: "/path/to/dest"),
folderCopyOperation(sourceFolderPath: "/path/to/foo", destinationFolderPath: "/path/to/bar")
])
```

至此Jenkins 入门系列教程完毕，后续如工作中还遇到Jenkins方面的技巧，将会在我的个人博客中进行更新哟，欢迎关注我的个人博客。

![](https://i0.hdslb.com/bfs/article/4adb9255ada5b97061e610b682b8636764fe50ed.png@progressive.webp)

> 欢迎各位志同道合的朋友一起学习交流，如文章有误请在下方留下您宝贵的经验知识，个人邮箱地址【master#weiyigeek.top】  

![](https://i0.hdslb.com/bfs/article/02db465212d3c374a43c60fa2625cc1caeaab796.png@progressive.webp)

更多文章来源: https://weiyigeek.top 【WeiyiGeek Blog - 为了能到远方，脚下的每一步都不能少】

![](https://i0.hdslb.com/bfs/article/d25e6027ff6931148f98334450b5e4797c2d0b3e.png@942w_483h_progressive.webp)

如果你觉得这个专栏还不错的，请给这篇专栏点个赞、投个币、收个藏、关个注，这将对我有很大帮助！