**本章目录:**

-   0x01 Jenkins之K8s集群自动化部署单war包应用
    

-   前期准备
    
-   (0) HelloWorld - Tomcat 应用服务器
    
-   (1) Nginx - Web服务器
    
-   (2) Jenkins Pipeline 脚本
    
-   (3) Shell 部署脚本
    
-   (4) 效果预览
    

![](https://i0.hdslb.com/bfs/article/4aa545dccf7de8d4a93c2b2b8e3265ac0a26d216.png@progressive.webp)

**实验目的**  
描述: 采用Jenkins从Gitlab中拉取测试项目以及代码质量检测，然后进行编译构建并利用K8s集群进行WAR包应用部署实践，最后将生成的制品进行归档到Gitlab的Release之中;

动态配置文件或者脚本:

-   setting.xml (Maven编译构建配置文件)
    
-   HelloWorld.yaml (应用部署资源清单)
    
-   options.sh (部署脚本)
    

描述: 假定你已经按照我前面几篇文章中安装好基础环境。

-   Step 1.基础镜像拉取于上传到私有仓库中
    

```
weiyigeek@weiyigeek-107:~/k8s/sonarqube$ docker tag tomcat:8.5.61-jdk8-corretto harbor.weiyigeek.top/devops/tomcat:8.5.61-jdk8-corretto
weiyigeek@weiyigeek-107:~/k8s/sonarqube$ docker push harbor.weiyigeek.top/devops/tomcat:8.5.61-jdk8-corretto
The push refers to repository [harbor.weiyigeek.top/devops/tomcat]
f13244e4918b: Pushed
536f15f78828: Pushed
1c98b6d16fad: Pushed
fdef502bf282: Pushed
cee8d35c645b: Pushed
8.5.61-jdk8-corretto: digest: sha256:110d7391739e868daf8c1bdd03fcb7ffe9eaf2b134768b9162e2cd47d58f7255 size: 1368
```

-   Step 2.资源清单文件
    

```
cat > HelloWorld.yaml <<'END'
apiVersion: v1
kind: Service
metadata:
  name: deploy-maven-svc
spec:
  type: NodePort         # Service 类型
  selector:
    app: java-maven       # 【注意】与deployment资源控制器创建的Pod标签进行绑定;
    release: stabel       # Service 服务发现不能缺少Pod标签，有了Pod标签才能与之SVC对应
  ports:                  # 映射端口
  - name: http
    port: 8080              # cluster 访问端口
    targetPort: 8080        # Pod 容器内的服务端口
    nodePort: 30089
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: deploy-java-maven
spec:
  serviceName: "deploy-maven-svc"
  replicas: 3                # 副本数
  selector:                  # 选择器
    matchLabels:
      app: java-maven   # 匹配的Pod标签非常重要
      release: stabel
  template:
    metadata:
      labels:
        app: java-maven      # 模板标签
        release: stabel
    spec:
      volumes:                # 关键点
      - name: webapps         # 卷名称
        hostPath:             # 采用hostPath卷
          type: DirectoryOrCreate   # 卷类型DirectoryOrCreate: 如果子节点上没有该目录便会进行创建
          path: /nfsdisk-31/test/  # 各主机节点上已存在的目录此处是NFS共享
      - name: timezone     # 容器时区设置
        hostPath:
          path: /usr/share/zoneinfo/Asia/Shanghai
      containers:
      - name: java-maven
        image: harbor.weiyigeek.top/devops/tomcat:8.5.61-jdk8-corretto  # 拉取的镜像
        imagePullPolicy: IfNotPresent
        command: ["bash","-c","cp /tmp/${APPNAME} /usr/local/tomcat/webapps/ROOT.war && catalina.sh run"]
        env:
          - name: APPNAME
            value: HelloWorld-v1.40.war
        ports:
        - name: http         # 此端口在服务中的名称
          containerPort: 8080  # 容器暴露的端口
        volumeMounts:        # 挂载指定卷目录
        - name: webapps      # Tomcat 应用目录
          mountPath: /tmp/
        - name: logs         # Tomcat 日志目录
          mountPath: /usr/local/tomcat/logs
        - name: timezone     # 镜像时区设置
          mountPath: /usr/share/zoneinfo/Asia/Shanghai
  volumeClaimTemplates:      # 卷的体积要求模板此处采用StorageClass存储类主要针对于应用日志的存储;
  - metadata:                # 根据模板自动创建PV与PVC并且进行一一对应绑定;
      name: logs
    spec:
      accessModes: [ "ReadWriteOnce" ]
      storageClassName: managed-nfs-storage # StorageClass存储类
      resources:
        requests:
          storage: 1Gi
END
```

-   Step 1.Nginx 配置文件
    

```
root@468dd7df372e:/# grep -v "#" /etc/nginx/conf.d/default.conf
server {
    listen       80;
    listen  [::]:80;
    server_name  rel.weiyigeek.top;
    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
}
```

-   Step 2.将Jenkins构建的Release进行上传在Nginx 中进行归档所以需要资源清单进行部署;
    

```
cat > jenkins-release-nginx.yaml <<'END'
# apiVersion: extensions/v1beta1
# kind: Ingress
# metadata:
#   annotations:
#     kubernetes.io/ingress.class: nginx
#   labels:
#     app: release
#   name: jenkins-release
#   namespace: devops
# spec:
#   rules:
#   - host: rel.weiyigeek.top
#     http:
#       paths:
#       - backend:
#           serviceName: release-svc
#           servicePort: 80
#         path: /
# ---
apiVersion: v1
kind: Service
metadata:
  name: release-svc
  namespace: devops
spec:
  type: NodePort
  selector:
    app: release
  ports:
  - name: http
    port: 80
    targetPort: 80
    nodePort: 30003
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginxconf
  namespace: devops
data:
  default.conf: |
    server {
      listen       80;
      listen  [::]:80;
      server_name  rel.weiyigeek.top;
      location / {
          root   /usr/share/nginx/html;
          index  index.html index.htm;
      }
      error_page   500 502 503 504  /50x.html;
      location = /50x.html {
          root   /usr/share/nginx/html;
      }
    }

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: release-deployment
  namespace: devops
  labels:
    app: release
spec:
  replicas: 1
  selector:
    matchLabels:
      app: release
  template:
    metadata:
      labels:
        app: release
    spec:
      containers:
      - name: release-nginx
        image: nginx:latest
        imagePullPolicy: IfNotPresent
        ports:
        - name: http
          containerPort:  80
        volumeMounts:
          - name: conf
            mountPath: /etc/nginx/conf.d/
            readOnly: false
          - name: data
            mountPath: /usr/share/nginx/html/
            readOnly: false
      volumes:
      - name: conf
        configMap:
          name: nginxconf
          defaultMode: 0755
      - name: data
        hostPath:
          type: DirectoryOrCreate
          path: /nfsdisk-31/release
END
```

-   Step 3.访问验证
    

```
# 1.测试文件
~$ cd /nfsdisk-31/release && touch index.html
  # echo "Hello World, Boy or Girl;" > index.html

# 2.deployments状态查询与SVC
~$ kubectl get deployments.apps -n devops  release-deployment -o wide
  # NAME                 READY   UP-TO-DATE   AVAILABLE   AGE     CONTAINERS      IMAGES         SELECTOR
  # release-deployment   1/1     1            1           4d11h   release-nginx   nginx:latest   app=release
~$ kubectl get svc -n devops  release-svc
  # NAME          TYPE       CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
  # release-svc   NodePort   10.106.107.201   <none>        80:30003/TCP   4d11h

# 3.访问验证
~/k8s/nginx$ curl rel.weiyigeek.top/index.html
~/k8s/nginx$ curl http://192.168.12.108:30003/index.html
  # Hello World, Boy or Girl;
```

Jenkins 流水线脚本如下:

```
// @ ssh 登陆链接函数
def getSSH() {
  def remote = [:]
  remote.name = "K8S#${DEPLOY_HOST}"
  remote.user = "${DEPLOY_USER}"
  remote.host = "${DEPLOY_HOST}"
  remote.port = 20211
  remote.allowAnyHosts = true
  withCredentials([string(credentialsId: "${DEPLOY_CREDENTIALSID}", variable: 'sshpassword')]) {
    remote.password = sshpassword
  }
  return remote
}

// @ Secure 密钥获取
def getSecure(Ticket_Token) {
  def token = ""
  withCredentials([string(credentialsId: Ticket_Token, variable: 'secure_token')]) {
    token = secure_token
  }
  println token
  return token
}

pipeline {
  /* Job 工作节点的绑定 */
  agent {
    // 1) 采用K8s动态生成Jnlp的Jenkins 节点
    kubernetes {
      cloud 'kubernetes'
      namespace 'devops'
      inheritFrom 'jenkins-slave'
      workingDir '/home/jenkins/agent'
      // yamlFile 'KubernetesPod.yaml' // 指定 yaml 资源清单其内容包含在yaml对象中
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
    image: 'harbor.weiyigeek.top/devops/jenkins-jnlp:3.13.6-alpine'
    imagePullPolicy: 'IfNotPresent'
    command: ["/bin/sh","-c","/usr/local/bin/jenkins-agent.sh && cat"]
    tty: true
    volumeMounts:
    - mountPath: "/home/jenkins/.m2"
      name: "volume-0"
    - mountPath: "/var/run/docker.sock"
      name: "volume-1"
    """.stripIndent()
    }
  }

  /* 全局选项 */
  options {
    // Gitlab 相关设置(连接凭据与Pipeline步骤)
    gitLabConnection('Gitlab-weiyigeek')
    gitlabBuilds(builds: ['代码拉取', '代码检测', '项目构建','项目部署','成品归档','消息通知'])

    // 全局超时设置 30分钟
    timeout(time: 30, unit: 'MINUTES')
  }

  /* Job 自动触发构建参数 */
  triggers {
    // gitlab 项目进行 git Push或者是Comment 触发构建
    gitlab(
      triggerOnPush: false,
      triggerOnMergeRequest: true,
      triggerOpenMergeRequestOnPush: "never",
      noteRegex: "jenkins build",
      branchFilterType: 'All'
    )
  }

  /*
  全局参数, 在shell可通过变量名访问，而在script pipeline脚本中通过params.参数名称访问.  ${params.RELEASE_VERSION}
  */
  parameters {
    string(name: 'RELEASE_VERSION', defaultValue: "master", description: 'Message: 请选择部署的Tags版本?',trim: 'True')
  choice(name: 'PREJECT_OPERATION', choices: ['deploy', 'rollback', 'redeploy'], description: 'Message: 选择项目操作方式?')
    choice(name: 'SONARQUBE', choices: ['False','True'],description: 'Message: 是否进行代码质量检测?')
    choice(name: 'GITLAB_RELEASE', choices: ['False','True'],description: 'Message: 是否进行编译成品发布到Gitlab开发项目Release中?')
  }


  /*
  环境变量设置, 在shell可通过变量名访问，而在script pipeline脚本中通过env.参数名称访问.  ${env.GITLAB_URL}
  */
  environment {
    // # GitLab 仓库相关信息（修改点1）
    GITLAB_URL = 'ssh://git@gitlab.weiyigeek.top:2222/weiyigeek/helloworld.git'
    GITLAB_PROJECTID = '45'
    GITLAB_PUB = '5e2f46d9-6725-4399-847f-dafb3f29d0ce'
    GITLAB_TOKEN = 'ce228573-ad3d-40e8-a3bf-b9de510080db'
    GITLAB_REL = "http://gitlab.weiyigeek.top/api/v4/projects/${GITLAB_PROJECTID}/releases"
    GITLAB_DOWNLOAD = "http://192.168.12.107:30003/${JOB_NAME}/${APP_NAME}"

    // # 应用名称&后缀以及集成结果（修改点2）
    APP_NAME = "${JOB_NAME}-${RELEASE_VERSION}${APP_SUFFIX}"
    APP_SUFFIX = ".war"
    APP_FILE = 'target/*.war'
    APP_DIR = '/nfsdisk-31/test/'
    // # Kubernetes 部署应用相关名称空间以及标签和SVC地址
    K8S_NAMESPACE = "default"
    K8S_SELECTOR = "app=java-maven"
    KS8_ADDRESS = "http://192.168.12.107:30089"

    // # 应用部署主机（修改点3）
    def SERVER = ''
    DEPLOY_CREDENTIALSID = "b846141f-55c2-4b46-98a7-29b9e76af3cc" //12.107
    DEPLOY_HOST = "192.168.12.107"
    DEPLOY_USER = "root"
    DEPLOY_PORT = 20211

    // # 代码质量检测项目名称和检测反馈超时时间
    SONARQUBE_CREDENTIALSID = "18d24c8f-9772-4b3e-934c-f80ed0e96cde"
    SONARQUBE_PROJECTKEY = "${JOB_NAME}"
    SONARQUBE_TIMEOUT = '10';

    // # 企业微信WebHookURL 1150-曾老师
    QYWX_WEBHOOK = 'https://qyapi.weixin.qq.com/cgi-bin/webhook/send?key=7d7f46fa-28c6-40b3-9c45-de243ec51150'
  }


  /* Job 步骤状态 */
  stages {
    stage ('代码拉取') {
      steps {
        // (1) 代码超时时间设置 5 分钟
        timeout(time: 5, unit: 'MINUTES') {
          script {
            try {
              // (1) 动态SSH公与私钥导入到我们动态运行的工作节点之中。
              withCredentials([
                sshUserPrivateKey(credentialsId: '1d4e61cb-5d59-4da8-813b-a5fb345770c9', keyFileVariable: 'keyFile', passphraseVariable: 'pass', usernameVariable: 'user'),
                string(credentialsId: '79a78423-e539-4b1f-a872-f0cf1dd3f9fa', variable: 'keyPub')
              ]) {
              // SSH Private Key
              writeFile file: 'private.key', text: keyFile
              sh "cp \$(cat private.key) ~/.ssh/id_ed25519"

              // SSH Public
              writeFile file: 'private.pub', text: keyPub
              sh "echo \$(cat private.pub) | base64 -d > ~/.ssh/id_ed25519.pub"
              }

              // (2) Gitlab 项目拉取
              checkout([$class: 'GitSCM', branches: [[name: "${params.RELEASE_VERSION}"]], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: "${env.GITLAB_PUB}", url: "${env.GITLAB_URL}"]]])
              // (3) 流水线状态同步
              updateGitlabCommitStatus name: '代码拉取', state: 'success'
            } catch(Exception err) {
              updateGitlabCommitStatus name: '代码拉取', state: 'failed'
              // (4) 显示错误信息但还是会进行下一阶段操作(err.toString()  == unstable '拉取代码失败')
              echo err.toString()
              // 将会停止构建
              error "[-Error] : 代码拉取失败\n [-Msg] : ${err.getMessage()} "
            }

            // (5) 显示当前版本Commit信息以及可用Tag版本
            // def INFO = sh label: 'release_info',returnStdout: true, returnStatus: true, script: """\
            // echo "Git Commit:" && \
            // git branch && \
            // git log --oneline  | grep "${params.RELEASE_VERSION}" && \
            // echo "Available Tag Version:" && \
            // git tag -l --column;
            // """
            // """
            // git tag -l --column | tr -d ' '  > tag ;
            // export a="\$(sed "s#v#, v#g" tag)'";
            // echo [\${a#,*}]
            // """

            // (6) 输出预定变量值
            echo "[-Info] : RELEASE_VERSION: ${params.RELEASE_VERSION} , PREJECT_OPERATION: ${params.PREJECT_OPERATION}, SONARQUBE: ${params.SONARQUBE}, GITLAB_RELEASE: ${params.GITLAB_RELEASE}"

            // (7) 切换数据库版本(Jenkinsfile中使用)
            //sh label: 'checkout_version', script: '[ -n "${RELEASE_VERSION}" ] &&  git checkout ${RELEASE_VERSION} || { echo -e "切换至指定的tag的版本 tag：${RELEASE_VERSION} 不存在或为空，请检查输入的tag!" && exit 1; }'
          }
        }
    }
  }

stage ('代码检测') {
  // (8) 判断是否进行代码质量检测(满足则继续否则跳过)
  when {
    // 如果项目是回滚操作则不进行代码质量检测，然后当SONARQUBE参数为True时进行质量检测，否则不进行。
    expression { return params.PREJECT_OPERATION != "rollback" }
    anyOf {
      environment name: 'SONARQUBE', value: 'True'
    }
  }
  steps {
    script {
    // (9) Tool 后的名称一定是对应全局工具配置中sonar扫描器的名字进行代码质量检测
      def sonarqubeScanner = tool 'Sonar-Scanner';
      withSonarQubeEnv(credentialsId: "${SONARQUBE_CREDENTIALSID}") {
        // 注意: 可以将sonarQube的属性定义在这里，也可以定义在项目文件中然后在这里引用配置文件
        sh "${sonarqubeScanner}/bin/sonar-scanner " +
              "-Dsonar.projectName=${JOB_NAME} " +
              "-Dsonar.projectKey=${SONARQUBE_PROJECTKEY} " +
              "-Dsonar.projectVersion=${RELEASE_VERSION} " +
              "-Dsonar.sourceEncoding=utf8  " +
              "-Dsonar.sources=.  " +
              "-Dsonar.language=java  " +
              "-Dsonar.java.binaries=."

        // (10) 暂停xxs等待sonarqube扫描反馈，根据实际时间进行调整;
        // NANOSECONDS SECONDS
        sleep time: "${SONARQUBE_TIMEOUT}", unit: 'SECONDS'
        // sh "sleep ${SONARQUBE_TIMEOUT}"
      }

      try {
        // 方式1
        // timeout(1) {
        //     def qg = waitForQualityGate()
        //     if (qg.status != 'OK') {
        //         error "未通过Sonarqube的代码质量阈检查，请及时修改！failure: ${qg.status}"
        //     } else {
        //         echo "successful"
        //     }
        // }

        // 方式2 (根据实际时间进行调整)
          timeout(time: 1, unit: 'MINUTES') {
            waitForQualityGate abortPipeline: true
          }
          updateGitlabCommitStatus name: '代码检测', state: 'success'
        } catch(Exception err) {
          updateGitlabCommitStatus name: '代码检测', state: 'failed'
          unstable '代码检测失败'
        }
      }
    }
  }

  // (11) 项目构建打包处理
  stage ("项目构建") {
    // 当操作为 PREJECT_OPERATION 为回滚(rollback) 时不进行构建打包
    when {
      expression { return params.PREJECT_OPERATION != "rollback" }
    }
    steps {
      script {
        try {
          // Maven 配置依赖(动态配置)
          sh script: "curl -s -o settings.xml http://gitlab.weiyigeek.top/weiyigeek/helloworld/-/raw/master/settings.xml"
          sh script: '/usr/local/maven/bin/mvn -s "settings.xml" clean install -Dmaven.test.skip=true'
          // 补充还可以采用此种方式进行更新流水线状态
          // gitlabCommitStatus(connection: gitLabConnection('Gitlab 用户认证凭据'),name: "项目构建") { ... }
          updateGitlabCommitStatus name: '项目构建', state: 'success'
        } catch(Exception err) {
          updateGitlabCommitStatus name: '项目构建', state: 'failed'
          unstable '项目构建失败'
        }
      }
    }
  }

  // (12) 进行开发测试部署
  stage ("项目部署") {
    steps {
      script {
        // ssh 连接函数调用
        SERVER = getSSH()
          try {
            // 项目部署所用得资源清单以及
            sh script: """\
            curl -s http://gitlab.weiyigeek.top/weiyigeek/helloworld/-/raw/master/${JOB_NAME}.yaml | sed s#__APPNAME__#${APP_NAME}#g > ./${JOB_NAME}.yaml && \
            wget http://gitlab.weiyigeek.top/weiyigeek/helloworld/-/raw/master/options.sh -O options.sh
            """
            // ssh 插件调用
            sshPut remote: SERVER, from: "./${JOB_NAME}.yaml" , into: "${APP_DIR}${JOB_NAME}.yaml"
            // 部署脚本
            sh script: "chmod +x options.sh && sh -x ./options.sh"
            updateGitlabCommitStatus name: '测试部署', state: 'success'
          }catch(Exception err) {
            updateGitlabCommitStatus name: '测试部署', state: 'failed'
            unstable '部署失败' // echo err.toString()
            error "[-Error] : 项目部署失败 \n[-Msg] : ${err.getMessage()} "
          }
      }
    }
  }

  // (13) 进行成品归档 archiveArtifacts
  stage('成品归档') {
    when {
      // 当操作为 PREJECT_OPERATION 为rollback(回滚)时 且当 params.GITLAB_RELEASE 为False 时不进行成品归档
      expression { return params.PREJECT_OPERATION != "rollback" && params.GITLAB_RELEASE }
    }
    steps {
      script {
        if ( ! params.RELEASE_VERSION.contains("master") ) {
          try {
            // Jenkins 归档
            archiveArtifacts artifacts: "${APP_FILE}", onlyIfSuccessful: true

            // Gitlab Token 用于 Gitlab - Release 版本分发
            def gitlab_token = getSecure("${GITLAB_TOKEN}")

            // 判断 Gitlab - Release 版本是否存在
            def gitlab_release = sh(returnStatus: true, script: "sudo apk add jq && curl -s --header 'PRIVATE-TOKEN: ${gitlab_token}' ${GITLAB_REL} | jq .[].tag_name | grep -c '${params.RELEASE_VERSION}'")

            // 上传得 Release Nginx web 根路径
            def gitlab_release_dir = "/nfsdisk-31/release/${JOB_NAME}/"

            echo gitlab_token + " --- " + gitlab_release + "====="  + gitlab_release_dir + "---"

            // 根据 gitlab_release 变量判断是否进行release-cli 发布指定版本得 `Release`
            if ( gitlab_release != 0 ) {
              sshCommand remote: SERVER, command: "mkdir -p ${gitlab_release_dir}"
              sshPut remote: SERVER, from: "./${APP_NAME}" ,into: "${gitlab_release_dir}${APP_NAME}"
              sh script: """
              /usr/local/bin/release-cli --server-url "http://gitlab.weiyigeek.top" --private-token "${gitlab_token}" \
              --project-id ${GITLAB_PROJECTID}  create \
              --name "${JOB_NAME}-${params.RELEASE_VERSION}" \
              --tag-name "${params.RELEASE_VERSION}" \
              --description "此 ${JOB_NAME} 项目 ${params.RELEASE_VERSION}版本 来自 Jenkins Build 编译构建上传。" \
              --assets-link '{"name": "${APP_NAME}","url":"${GITLAB_DOWNLOAD}","link_type":"other"}'
              """
            } else {
              echo "[-Tips] :  Gitlab Release 中已存在 ${params.GITLAB_RELEASE} 版本, 将不会提交该Release到Gitlab中"
            }
            updateGitlabCommitStatus name: '成品归档', state: 'success'
          } catch (Exception err) {
            updateGitlabCommitStatus name: '成品归档', state: 'failed'
            echo err.toString()
          }
        } else {
            echo "# 版本不能为 ${param.RELEASE_VERSION} , 将不会进行归档!"
        }
      }
    }
  }
}

  // (12) POST阶段当所有任务执行后触发进行工作空间的清理以及消息通知;
  post {
    // 无论如何，我已经完成了 无论成功与否都将执行
    always {
      script {
        // 企业微信消息通知
        def GIT_COMMIT=sh returnStdout: true, script: "git show --oneline --ignore-all-space --text | head -n 1"
        qyWechatNotification aboutSend: true, failNotify: true, failSend: true, mentionedMobile: '', startBuild: true, successSend: true, unstableSend: true, webhookUrl: "${env.QYWX_WEBHOOK}"
        sh """\
        curl -s ${env.QYWX_WEBHOOK} \
        -H 'Content-Type: application/json' \
        -d '
        {
          "msgtype": "text",
          "text": {
              "content": "项目: ${env.SONARQUBE_PROJECTKEY}, 版本: ${params.RELEASE_VERSION}, 操作: ${params.PREJECT_OPERATION} , 信息: ${GIT_COMMIT}, Gitlab-Release 发布： ${params.GITLAB_RELEASE}, 部署地址: ${env.KS8_ADDRESS}"
          }
        }'
      """
      }
      deleteDir()  /* clean up our workspace */
    }
    success {
      echo '[-Info] : I succeeeded! - 任务成功'
      updateGitlabCommitStatus name: '消息通知', state: 'success'
    }
    unstable {
      echo '[-Info] : I am unstable - 不稳定 :/'
    }
    failure {
      echo '[-Info] : I failed - 任务失败 :( '
      updateGitlabCommitStatus name: '消息通知', state: 'failed'
    }
    changed {
      echo '[-Info] : Things were different before - 与以前的情况不一样...'
    }
  }
}
```

```
#!/bin/sh
# // 基础变量
export K8sMaster="${DEPLOY_USER}@${DEPLOY_HOST}"
export AppName="${APP_FILE##*/}"
export FindAppName=$(find ${WORKSPACE} -maxdepth 2 -type f -name ${AppName})

# // 若操作值不为rollback则执行该判断代码块中的内容。
if [${PREJECT_OPERATION} != "rollback" ];then
  if [ "${FindAppName}" != "" ]; then
    echo "[-Msg]: ${FindAppName} ./${APP_NAME} Copying...."
    cp -rf ${FindAppName} ./${APP_NAME}
  else
    echo "[-Error]: FindAppName param error!"
    exit 1
  fi
fi

# // 判断应用文件是否存在与共享目录中
function judge () {
  export ExistResult=$(ssh -p ${DEPLOY_PORT} ${K8sMaster} "ls ${APP_DIR} | grep ${RELEASE_VERSION} | wc -l")
}

# // 部署操作函数
function deploy () {
  if [[ ${ExistResult} -eq 0 || "${RELEASE_VERSION}" == "master" ]];then
    scp -P ${DEPLOY_PORT} "${APP_NAME}" ${K8sMaster}:${APP_DIR}${APP_NAME}
  else
    echo "[-Msg]: /${APP_NAME} current Deploy....."
  fi

  # 部署脚本
  if [[ "${RELEASE_VERSION}" == "master" ]];then
    ssh -p ${DEPLOY_PORT} ${K8sMaster} "kubectl -n ${K8S_NAMESPACE} delete pod -l ${K8S_SELECTOR}"
  else
    ssh -p ${DEPLOY_PORT} ${K8sMaster} "kubectl apply -f ${APP_DIR}${JOB_NAME}.yaml"
  fi
}

# // 回退操作函数
function rollback () {
  if [[ ${ExistResult} -ne 0 ]];then
    ssh -p ${DEPLOY_PORT} ${K8sMaster} "kubectl apply -f ${APP_DIR}${JOB_NAME}.yaml"
  else
    echo "[-Eroor]: /${APP_NAME} version not exsit....."
    exit 1
  fi
}

# // 重部署操作函数
function redeploy () {
  # 如果是以前部署过则删除以前部署的项目目录,否则重新部署;
  if [[ "v${GIT_COMMIT}" = "v${GIT_PREVIOUS_SUCCESSFUL_COMMIT}" || ${ExistResult} -gt 0 ]];then
    echo -e "[-Message]:  ${RELEASE_VERSION} Version Exsit !"
    ssh -p ${DEPLOY_PORT} ${K8sMaster} "find ${APP_DIR} -d -maxdepth 1 -type f -name ${APP_NAME}"
    scp -P ${DEPLOY_PORT} "${APP_NAME}" ${K8sMaster}:${APP_DIR}${APP_NAME}
  fi

  # 触发重新部署脚本
  deploy
}

# // 判断是否存在历史版本
judge

# 主操作入口判断
case ${PREJECT_OPERATION} in
"deploy")
  echo "1.deploy"
  deploy
;;
"rollback")
  echo "2.rollback"
  rollback
;;
"redeploy")
  echo "3.redeploy"
  redeploy
;;
*)
  echo "[-Error] : ${PREJECT_OPERATION} Param Error not in deploy/rollback/redeploy !"
  exit 1
;;
esac
```

## (4) 效果预览

-   Step 1.在`BlueOcean`进行构建部署参数选择。  
    描述: 我们可以进行初次部署回滚以及重新构建部署,还包括代码质量检测以及成品自动归档的选择控制。
    

![WeiyiGeek.BlueOcean部署](https://i0.hdslb.com/bfs/article/cf1bda1ca2728c06127030dde4e0635935b63fea.png@942w_837h_progressive.webp)

-   Step 2.在K8s集群中进行查看Pod迭代变化。
    

```
/nfsdisk-31/test# kubectl get pod --watch
  # NAME                                      READY   STATUS        RESTARTS   AGE
  # deploy-java-maven-0                       1/1     Running       0          83s
  # deploy-java-maven-1                       1/1     Running       0          80s
  # deploy-java-maven-2                       1/1     Terminating   0          76s
  # deploy-java-maven-2                       0/1     Terminating   0          106s
  # deploy-java-maven-2                       0/1     Pending       0          0s
  # deploy-java-maven-2                       0/1     ContainerCreating   0          0s
  # deploy-java-maven-2                       1/1     Running             0          2s
  # deploy-java-maven-1                       1/1     Terminating         0          112s
  # deploy-java-maven-1                       0/1     Terminating         0          2m26s
  # deploy-java-maven-1                       0/1     Pending             0          0s
  # deploy-java-maven-1                       0/1     ContainerCreating   0          0s
  # deploy-java-maven-1                       1/1     Running             0          3s
  # deploy-java-maven-0                       1/1     Terminating         0          2m32s
```

![WeiyiGeek.实验效果](https://i0.hdslb.com/bfs/article/0002a84cb985807441b6d0db829516119c4036e0.png@942w_596h_progressive.webp)

可以看到我们的项目自动从gitlab仓库中拉取helloworld项目，并根据Job前的输入进行项目MAVEN 构建，生成的war包成品归档，然后将成品推送到K8S集群中进行更新，更新完成后将该war包上传到gitlab release 页面中，以供其它环境拉取使用和备份。

至此本章实践完毕！

![](https://i0.hdslb.com/bfs/article/4adb9255ada5b97061e610b682b8636764fe50ed.png@progressive.webp)

> 欢迎各位志同道合的朋友一起学习交流，如文章有误请在下方留下您宝贵的经验知识，个人邮箱地址【master#weiyigeek.top】  

![](https://i0.hdslb.com/bfs/article/02db465212d3c374a43c60fa2625cc1caeaab796.png@progressive.webp)

更多文章来源: https://weiyigeek.top 【WeiyiGeek Blog - 为了能到远方，脚下的每一步都不能少】

![](https://i0.hdslb.com/bfs/article/d25e6027ff6931148f98334450b5e4797c2d0b3e.png@942w_483h_progressive.webp)

如果你觉得这个专栏还不错的，请给这篇专栏点个赞、投个币、收个藏、关个注，这将对我有很大帮助！