**本章目录:**

-   0x02 进阶实践
    

-   (1) Sonarqube 代码质量检测之 Pipeline Script from SCM
    
-   (2) Gitlab 自动触发构建之 Pipeline Script from SCM
    

-   0x03 入坑与出坑
    

![](https://i0.hdslb.com/bfs/article/4aa545dccf7de8d4a93c2b2b8e3265ac0a26d216.png@progressive.webp)

**实验需求: 拉取代码并指定Tag、采用sonarqube进行代码质量检测并进行构建**

Tips : sonarQube 是sonarQube扫描插件工具, 该工具除了前面直接下载二进制执行文件到Jenkins种(登陆jenkins页面–>系统管理–>全局工具配置)进行配置； 还可以通过自动化构建工具安装，不管是maven项目还是gradle项目都提供了安装sonarQube扫描工具的插件。

plugins {  id "org.sonarqube" version "2.7"}

**实验流程:**

-   Step 0.按照前面的流程在Jenkins中下载并配置好`SonarQube`并且在SonarQube中进行URL设置
    

```
# 关键项:
# (1) Sonarqube 通用配置项
Server base URL : http://sonar.weiyigeek.top:9000/

# (2) Dashboard -> 全局配置 -> SonarQube servers
# 环境变量允许将SonarQube服务器配置注入构建环境变量
Server URL : http://sonar.weiyigeek.top:9000
Server authentication token : 可以参照下面的Step2步骤
```

![WeiyiGeek.Jenkins && Sonarqube](https://i0.hdslb.com/bfs/article/ab1fa66cfcb0aa465620a97aa9c11e79b90fe55a.png@942w_891h_progressive.webp)

-   Step 1.并生成项目标识符名称(`somarqube-test`)并获取令牌`somarqube-test: 4354fc222bee3c2bad2d812b087dabab943a7ab0`
    
-   Step 2.将该Token添加到Jenkins凭据之中`Dashboard -> 凭据 -> 系统 -> 全局凭据 (unrestricted)`选择Secret类型文本 -> Secret(输入上面的Token) -> 描述:somarqube-test-api;
    

```
Secret text6810ea0d-e76a-40cf-9373-5040ed6b5456somarqube-test-apiSecret textsomarqube-test-api
```

![WeiyiGeek.somarqube-test-api](https://i0.hdslb.com/bfs/article/4ce3592704568c2fb2a68a54ed0c1abe896d340f.png@942w_437h_progressive.webp)

-   Step 3.创建一个流水线项目`somarqube-test-pipeline` -> 编写Pipeline Script脚本如下(非常值得注意涵盖的知识较多)
    

```
pipeline {
  agent any
  /* 该块中的变量将写入到Linux环境变量之中作为全局变量，在shell可通过变量名访问，而在script pipeline脚本中通过env.变量名称访问. */
  environment {
    GITLAB_URL = 'git@gitlab.weiyigeek.top:ci-cd/java-maven.git'
    // 代码质量检测项目名称
    SONARQUBE_PROJECTKEY = 'somarqube-test';
    // 代码质量检测反馈超时时间
    SONARQUBE_TIMEOUT = '10';
    // 版本号
    //RELEASE_VERSION="v1.1"
    // 默认操作方式
    //PREJECT_OPERATION="deploy"
  }

  /* 全局参数, 在shell可通过变量名访问，而在script pipeline脚本中通过params.参数名称访问.  */
  parameters {
    // Jenkins -> 原生 Build With Parameters 支持 (BuleOcean不支持gitParameter)
  //   gitParameter name: 'RELEASE_VERSION',
  //                    type: 'PT_BRANCH_TAG',
  //                    branchFilter: 'origin/(.*)',
  //                    defaultValue: 'master',
  //                    selectedValue: 'DEFAULT',
  //                    sortMode: 'DESCENDING_SMART',
//  description: 'Message: 请选择部署的Tags版本?'
  choice(name: 'SONARQUBE', choices: ['False','True'],description: 'Message: 是否进行代码质量检测?')
  string(name: 'RELEASE_VERSION', defaultValue: "", description: 'Message: 请选择部署的Tags版本?')
  choice(name: 'PREJECT_OPERATION', choices: ['deploy', 'rollback', 'redeploy'], description: 'Message: 选择项目操作方式?')
  }

  stages {
    stage ('代码拉取') {
      //注意: 此处得input不能包含在steps中并且只有局部stage块中可用调用，调用方式 ${RELEASE_VERSION} ${PREJECT_OPERATION}
      // input {
      //   message "Title : ${JOB_NAME} 项目构建参数"
      //   ok "完成提交"
      //   parameters {
      //     string(name: 'RELEASE_VERSION', defaultValue: '', description: 'Message: 请选择部署的Tags版本?')
      //     choice(name: 'PREJECT_OPERATION', choices: ['deploy', 'rollback', 'redeploy'], description: 'Message: 选择项目操作方式?')
      //   }
      // }
      steps {
        // (1) git项目拉取
        git branch: "${params.RELEASE_VERSION}", credentialsId: 'b4c8b4e9-2777-44a1-a1ed-e9dc21d37f4f', url: "${env.GITLAB_URL}"

        // (2) 超时时间设置 5 分钟
        timeout(time: 1, unit: 'MINUTES') {
          script {
              // (3) 存储可用版本
              def RELEASE=sh label: 'release',returnStdout: true, script: """\
              git tag -l --column | tr -d ' '  > tag ;
              export a="\$(sed "s#v#, v#g" tag)'";
              echo [\${a#,*}]
              """

              // 由于此处是groovy的变量此处得input通过script块包含只有局部stage块中可用调用(故此处不采用)
              // (4) 定义操作变量(RELEASE_VERSION 与 PREJECT_OPERATION) 注意此处无法影响shell中的变量
              //def RELEASE_VERSION=input message: "${JOB_NAME} 项目版本:\n ${RELEASE}", ok: '完成提交', parameters: [ string(name: 'RELEASE_VERSION', description: 'Message: 请选择部署的Tags版本?')];
              //def PREJECT_OPERATION=input message: "${JOB_NAME} 项目构建参数", ok: '完成提交', parameters: [choice(name: 'PREJECT_OPERATION', choices: ['deploy', 'rollback', 'redeploy'], description: 'Message: 选择项目操作方式?')];

              // (5) 输出变量与操作
              // 例如: RELEASE_VERSION: v1.11 , PREJECT_OPERATION: redeploy. SONARQUBE: True
              echo "RELEASE_VERSION: ${RELEASE_VERSION} , PREJECT_OPERATION: ${PREJECT_OPERATION}. SONARQUBE: ${params.SONARQUBE}"

              // (6) 切换分支版本(其实上面有了branch: "${params.RELEASE_VERSION}"此处显得多余)
              sh label: 'checkout_version', script: '[ -n "${RELEASE_VERSION}" ] &&  git checkout ${RELEASE_VERSION} || { echo -e "切换至指定的tag的版本 tag：${RELEASE_VERSION} 不存在或为空，请检查输入的tag!" && exit 1; }'
          }
        }
     }
   }

    stage ('代码检测') {
      // (7) 判断是否进行代码质量检测
      when {
        // expression { return params.SONARQUBE == "True" }
        anyOf {
          environment name: 'SONARQUBE', value: 'True'
          environment name: 'PREJECT_OPERATION', value: 'deploy'
          environment name: 'PREJECT_OPERATION', value: 'redeploy'
        }
      }

      steps {
        script {
        // (8) Tool 后的名称一定是对应全局工具配置中sonar扫描器的名字进行代码质量检测
        def sonarqubeScanner = tool 'sonarqubescanner';
        withSonarQubeEnv(credentialsId: '6810ea0d-e76a-40cf-9373-5040ed6b5456') {
          // 注意:可以将sonarQube的属性定义在这里，也可以定义在项目文件中然后在这里引用配置文件
          sh "${sonarqubeScanner}/bin/sonar-scanner " +
                "-Dsonar.projectName=${JOB_NAME} " +
                "-Dsonar.projectKey=${SONARQUBE_PROJECTKEY} " +
                "-Dsonar.projectVersion=${RELEASE_VERSION} " +
                "-Dsonar.sourceEncoding=utf8  " +
                "-Dsonar.sources=.  " +
                "-Dsonar.language=java  " +
                "-Dsonar.java.binaries=."
        }

        // (9) 暂停10s等待sonarqube扫描反馈
        sh "sleep ${SONARQUBE_TIMEOUT}"
        // 方式1
        // timeout(1) {
        //     def qg = waitForQualityGate()
        //     if (qg.status != 'OK') {
        //         error "未通过Sonarqube的代码质量阈检查，请及时修改！failure: ${qg.status}"
        //     } else {
        //         echo "successful"
        //     }
        // }

        // 方式2
        timeout(time: 1, unit: 'MINUTES') {
            waitForQualityGate abortPipeline: true
          }
        }
      }
    }

    // (10) 项目构建打包处理
    stage ("项目构建") {
      steps {
       sh label: 'build', script: '/usr/local/maven/bin/mvn clean package -Dmaven.test.skip=true '
     }
    }

    // (11) 进行开发测试部署
    stage ("开发测试部署") {
      steps {
        echo "后续添加......"
      }
    }
  }

  // (12) 全部阶段完成后执行进行工作空间的清理以及消息通知;
  post {
    always {
        echo 'One way or another, I have finished'
        deleteDir() /* clean up our workspace */
    }
    success {
        echo 'I succeeeded!'
    }
    unstable {
        echo 'I am unstable :/'
    }
    failure {
        echo 'I failed :('
    }
    changed {
        echo 'Things were different before...'
    }
  }
}
```

-   Step 4.项目运行以及代码拉取结果反馈
    

```
# git@gitlab.weiyigeek.top:ci-cd/java-maven.git— Print Message <1s
# Git  4s
# 查看可用标签
# git tag -l --column | tr -d ' ' > tag ; export a="$(sed "s#v#, v#g" tag)'"; echo [${a#,*}] — release<1s
# RELEASE_VERSION: v1.11 , PREJECT_OPERATION: deploy. SONARQUBE: True— Print Message <1s
# [ -n "${RELEASE_VERSION}" ] && git checkout ${RELEASE_VERSION} || { echo -e "切换至指定的tag的版本 tag：${RELEASE_VERSION} 不存在或为空，请检查输入的tag!" && exit 1; }— checkout_version  <1s
```

![WeiyiGeek.项目运行以及代码拉取](https://i0.hdslb.com/bfs/article/ca40a587d9fa097435b662b668528020225cdb03.png@942w_419h_progressive.webp)

-   Step 5.代码检测阶段查看(重点), SonarQube analysis api 接口URL(`http://sonar.weiyigeek.top:9000/api/ce/task?id=AXdd7NO4mUDNtrevcJJD`)
    

```
# sonarqubescanner— Use a tool from a predefined Tool Installation <1s
# /usr/local/sonar//bin/sonar-scanner -Dsonar.projectName=somarqube-test-pipeline -Dsonar.projectKey=somarqube-test -Dsonar.projectVersion=v1.11 -Dsonar.sourceEncoding=utf8 -Dsonar.sources=. -Dsonar.language=java -Dsonar.java.binaries=.— Shell Script7s
# sleep 10 — Shell Script10s
# true— Wait for SonarQube analysis to be completed and return quality gate status  <1s
```

![WeiyiGeek.代码检测阶段查](https://i0.hdslb.com/bfs/article/e7de347659f2123acf13287f608e131864201d40.png@942w_702h_progressive.webp)

-   Step 6.综合效果
    

![WeiyiGeek.最终效果](https://i0.hdslb.com/bfs/article/ea9226fe595e8c02c841ef0f06b12a6f7bc5aa6a.png@942w_806h_progressive.webp)

**实验需求：Gitlab 上传自动触发Jenkins构建并通过BlueOcan进行控制构建, 以及与 Gitlab 流水线状态同步**

**实验流程:**

-   Step 1.此处假设您已安装配置Gitlab Authentication plugin、GitLab Plugin这两个插件
    
-   Step 2.到 Gitlab私有仓库中进行生成项目`API Access Token` -> 用户设置 -> 访问令牌 -> 输入您的应用程序的名称 -> 选择相应到期时间 -> 范围: `授予对API的完全读/写访问权，包括所有组和项目、容器注册表和包注册表` -> 然后创建个人访问令牌;
    
-   Step 3.得到api Token(kWL\_9Fw\_nvbxTkpDb9X6)后在Jenkins中添加全局凭据 -> Dashboard -> 凭据 -> 系统 -> 全局凭据 (unrestricted) -> 选择凭据类型为`Gitlab API Token` -> 然后确定即可 -> 他会自动生成一个类似于UUID的一个字符串
    

```
# Value
f49076a8-11e4-4351-a88a-143cfab6555bGitLab API token (gitlab_admin_api)GitLab API tokengitlab_admin
```

![WeiyiGeek.Gitlab-API-Token](https://i0.hdslb.com/bfs/article/f66a983fdbdfd5559e59e2a0efe6558a150f83e2.png@942w_518h_progressive.webp)

-   Step 4.或者直接在全局配置的 Gitlab -> Enable authentication for '/project' end-point -> 应用保存；
    

![WeiyiGeek.Gitlab-Token-配置](https://i0.hdslb.com/bfs/article/57a1c9381cc0d9976cf55214df44f8d1de7d219e.png@942w_557h_progressive.webp)

-   Step 5.在Dashboard -> Gitlab-Pipeline Job 中 -> 构建触发器 -> 勾选`Build when a change is pushed to GitLab.` -> 重新生成打开的合并请求为`On push to source branch` -> Comment (regex) for triggering a build 可以在提交`Jenkins build`字符串进行触发构建编译;
    
-   Step 6.Jenkins 生成 Api Token -> 面板 \_> 用户设置 -> API Token 生成 (APl令牌提供了一种进行经过身份验证的CLI或REST API调用的方法。注意每六个月需要重新生成一次） `11112e147020668570e571fa438439cc60`  
    Tips: 每次重新启动Jenkins时，未使用的遗留令牌的创建日期将被重置，这意味着日期可能不准确。
    

![WeiyiGeek.Jenkins-API-Token](https://i0.hdslb.com/bfs/article/383c086893f659f38068799f233a68e20fd259db.png@942w_783h_progressive.webp)

-   Step 7.在Gitlab -> java-maven 项目 -> 设置 -> WebHooks -> 地址为是前面Build when a change is pushed to GitLab. GitLab webhook URL中的地址 `http://jenkins.weiyigeek.top:8080/project/Gitlab-Pipeline`\-> 输入 `Secret Token` -> 选择`Push events Trigger` -> add Webhook -> 最后进行选择push events测试 -> Hook executed successfully: HTTP 200
    
    注意事项:
    

```
#补充说明
Gitlab Project -> helloworld -> Webhook设置

# Gitlab 设置
URL ：Jenkins Job
Secret Token
Trigger
  Push events  : This URL will be triggered by a push to the repository  (每次提交都将触发-如不需要则不要勾选即可)
  Tag push events : This URL will be triggered when a new tag is pushed to the repository  (非常建议设置tag后才会触发) - 一般开启该事件选项后不建议开启Push evnts
  Comments : This URL will be triggered when someone adds a comment (评论触发)

# 成功 > 控制台输出 #32
# 2021-2-23 上午11:38 CST
# Started by GitLab push by Gitlab WeiyiGeek
```

![WeiyiGeek.GitLab webhook URL](https://i0.hdslb.com/bfs/article/c40f66b8e45b047f575fea02fde70ef9b4c3cedd.png@942w_368h_progressive.webp)

Tips : 此处需要设置允许来自钩子和服务的对本地网络的请求。  
Tips : 注意请根据您的Jenkins站点启用SSL(建议内网也需要注意的)

-   Step 8.此处先使用`Pipeine Script`脚本然后应用保存然后上传v1.11版本到Gitlab，查看是否自动触发Build;
    

```
pipeline {
  agent any
  // 全局选项
  options {
    gitLabConnection('Private-Gitlab')
    gitlabBuilds(builds: ['代码拉取', '代码检测', '项目构建','测试部署','成品归档'])
    timeout
  }

  environment {
    GITLAB_URL = 'git@gitlab.weiyigeek.top:ci-cd/java-maven.git'
    // 代码质量检测项目名称
    SONARQUBE_PROJECTKEY = 'Hello-World';
    // 代码质量检测反馈超时时间
    SONARQUBE_TIMEOUT = '10';
    // 版本号
    //RELEASE_VERSION="v1.1"
    // 默认操作方式
    //PREJECT_OPERATION="deploy"
    // 企业微信WebHookURL
    QYWX_WEBHOOK = 'https://qyapi.weixin.qq.com/cgi-bin/webhook/send?key=c222f3fc-f645-440a-ad24-0ce8d9626fa0'
  }

  /* 全局参数, 在shell可通过变量名访问，而在script pipeline脚本中通过params.参数名称访问.  */
  parameters {
    string(name: 'RELEASE_VERSION', defaultValue: "master", description: 'Message: 请选择部署的Tags版本?',trim: 'True')
  choice(name: 'PREJECT_OPERATION', choices: ['deploy', 'rollback', 'redeploy'], description: 'Message: 选择项目操作方式?')
    choice(name: 'SONARQUBE', choices: ['False','True'],description: 'Message: 是否进行代码质量检测?')
  }

   triggers {
        gitlab(triggerOnPush: true, triggerOnMergeRequest: true, branchFilterType: 'All')
    }

  stages {
    stage ('代码拉取') {
      steps {
        // (1) git项目拉取
        echo "${env.GITLAB_URL} -- ${params.RELEASE_VERSION}"
        // (2) 超时时间设置 5 分钟
        timeout(time: 1, unit: 'MINUTES') {
          script {
              try {
                          echo "${env.GITLAB_URL} -- ${params.RELEASE_VERSION}"

                // 方式1: git branch: 'v1.11', credentialsId: '43287e62-ce5b-489a-9c11-cedf38e16e92', url: "${env.GITLAB_URL}"
                // 方式2:
                checkout([$class: 'GitSCM', branches: [[name: "${params.RELEASE_VERSION}"]], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: 'b4c8b4e9-2777-44a1-a1ed-e9dc21d37f4f', url: "${env.GITLAB_URL}"]]])
                updateGitlabCommitStatus name: '代码拉取', state: 'success'
              } catch(Exception err) {
                updateGitlabCommitStatus name: '代码拉取', state: 'failed'
                // 显示错误信息但还是会进行下一阶段操作
                echo err.toString()    /* hudson.AbortException: Couldn't find any revision to build. Verify the repository and branch configuration for this job. */
                unstable '拉取代码失败'
                // 构建停止
                error "[-Error] : 代码拉取失败\n [-Msg] : ${err.getMessage()} "

              }
              // (3) 存储可用版本
              def RELEASE=sh label: 'release',returnStdout: true, script: """\
              git tag -l --column | tr -d ' '  > tag ;
              export a="\$(sed "s#v#, v#g" tag)'";
              echo [\${a#,*}]
              """

              // (4) 输出变量与操作
              // 例如: RELEASE_VERSION: v1.11 , PREJECT_OPERATION: redeploy. SONARQUBE: True
              echo "RELEASE_VERSION: ${params.RELEASE_VERSION} , PREJECT_OPERATION: ${params.PREJECT_OPERATION}. SONARQUBE: ${params.SONARQUBE}"

              // (5) 切换数据库版本
              //sh label: 'checkout_version', script: '[ -n "${RELEASE_VERSION}" ] &&  git checkout ${RELEASE_VERSION} || { echo -e "切换至指定的tag的版本 tag：${RELEASE_VERSION} 不存在或为空，请检查输入的tag!" && exit 1; }'
              sh "git branch"
          }
        }
     }
   }

    stage ('代码检测') {
      // (6) 判断是否进行代码质量检测
      when {
        expression { return params.PREJECT_OPERATION != "rollback" }
        anyOf {
          environment name: 'SONARQUBE', value: 'True'
          environment name: 'PREJECT_OPERATION', value: 'deploy'
          environment name: 'PREJECT_OPERATION', value: 'redeploy'
        }
      }

      steps {
        script {

        // (7) Tool 后的名称一定是对应全局工具配置中sonar扫描器的名字进行代码质量检测
        def sonarqubeScanner = tool 'sonarqubescanner';
        withSonarQubeEnv(credentialsId: '6810ea0d-e76a-40cf-9373-5040ed6b5456') {
          // 注意:可以将sonarQube的属性定义在这里，也可以定义在项目文件中然后在这里引用配置文件
          sh "${sonarqubeScanner}/bin/sonar-scanner " +
                "-Dsonar.projectName=${JOB_NAME} " +
                "-Dsonar.projectKey=${SONARQUBE_PROJECTKEY} " +
                "-Dsonar.projectVersion=${RELEASE_VERSION} " +
                "-Dsonar.sourceEncoding=utf8  " +
                "-Dsonar.sources=.  " +
                "-Dsonar.language=java  " +
                "-Dsonar.java.binaries=."
        }

        // (8) 暂停10s等待sonarqube扫描反馈
        sleep time: "${env.SONARQUBE_TIMEOUT}", unit: 'NANOSECONDS'
        sh "sleep ${SONARQUBE_TIMEOUT}"
        // 方式1
        // timeout(1) {
        //     def qg = waitForQualityGate()
        //     if (qg.status != 'OK') {
        //         error "未通过Sonarqube的代码质量阈检查，请及时修改！failure: ${qg.status}"
        //     } else {
        //         echo "successful"
        //     }
        // }
        try {
        // 方式2
        timeout(time: 1, unit: 'MINUTES') {
          waitForQualityGate abortPipeline: true
        }

        updateGitlabCommitStatus name: '代码检测', state: 'success'


        } catch(Exception err) {
           updateGitlabCommitStatus name: '代码检测', state: 'failed'
            // 显示错误信息但还是会进行下一阶段操作
            unstable '代码检测失败'
        }
        }
      }
    }

    // (10) 项目构建打包处理
    stage ("项目构建") {
         when {
         expression { return params.PREJECT_OPERATION != "rollback" }
         }
      steps {
           gitlabCommitStatus(connection: gitLabConnection('Private-Gitlab'),name: "项目构建") {
              sh script: '/usr/local/maven/bin/mvn clean package -Dmaven.test.skip=true'
           }

     }
    }

    // (11) 进行开发测试部署
    stage ("测试部署") {
      steps {
        script {
            try {
                sh label: 'deploy', script: '/tmp/script/jenkins-pipeline-gitlab-k8s.sh'
                updateGitlabCommitStatus name: '测试部署', state: 'success'
            }catch(Exception err) {
                echo err.toString()    /* hudson.AbortException: Couldn't find any revision to build. Verify the repository and branch configuration for this job. */
                unstable '部署失败'
                updateGitlabCommitStatus name: '测试部署', state: 'failed'

                // 构建停止
                error "[-Error] : 测试部署失败\n [-Msg] : ${err.getMessage()} "
            }

        }
      }
    }

    stage('成品归档') {
    steps {
       script {
         try {
            archiveArtifacts artifacts: "target/*.war",fingerprint: true, followSymlinks: false, onlyIfSuccessful: true
            updateGitlabCommitStatus name: '成品归档', state: 'success'
         } catch (Exception err) {
            updateGitlabCommitStatus name: '成品归档', state: 'failed'
            echo err.toString()    /* hudson.AbortException: Couldn't find any revision to build. Verify the repository and branch configuration for this job. */

         }
       }
    }
    }
  }



  // (12) POST阶段当所有任务执行后触发进行工作空间的清理以及消息通知;
  post {
    always {
      echo 'One way or another, I have finished'
      qyWechatNotification aboutSend: true, failNotify: true, failSend: true, mentionedMobile: '', startBuild: true, successSend: true, unstableSend: true, webhookUrl: 'https://qyapi.weixin.qq.com/cgi-bin/webhook/send?key=c222f3fc-f645-440a-ad24-0ce8d9626fa0'
      deleteDir()  /* clean up our workspace */

    }
    success {
      echo 'I succeeeded!'
    }
    unstable {
      echo 'I am unstable - 不稳定 :/'
    }
    failure {
      echo 'I failed :('
    }
    changed {
      echo 'Things were different before - 以前的情况不一样...'
    }
  }
}
```

/tmp/script/jenkins-pipeline-gitlab-k8s.sh

```
#!/bin/bash
# Description: Jenkins CI & Kubernetes & Gitlab -> Deploy or Rollback or Redeploy Java Maven Project
DATE=$(date +%Y%m%d-%H%M%S)
WAR_PATH="/nfs/data4/war"
WEBROOT_PATH="/nfs/data4/webapps"
WEB_DIR="${JOB_NAME}-${DATE}-${RELEASE_VERSION}"
WAR_DIR=" ${WAR_PATH}/${WEB_DIR}"
WAR_NAME="${WEB_DIR}.war"
K8S_MATER="WeiyiGeek@10.10.107.202"
K8S_MATER_PORT="20211"
echo "${RELEASE_VERSION}  --------------- ${PREJECT_OPERATION}"


# 部署
deploy () {
  # 1.上传Maven打包的war包到master之中
  scp -P ${K8S_MATER_PORT} ${WORKSPACE}/target/*.war WeiyiGeek@10.10.107.202:${WAR_PATH}/${WAR_NAME}

  # 2.解压&软链接
  ssh -p ${K8S_MATER_PORT} ${K8S_MATER} "unzip ${WAR_PATH}/${WAR_NAME} -d ${WAR_DIR} && \
                                         rm -rf ${WEBROOT_PATH} && \
                                         ln -s ${WAR_PATH}/${WEB_DIR} ${WEBROOT_PATH} && \
                                         kubectl delete pod -l app=java-maven"
}

# 回退
rollback () {
  History_Version=$(ssh -p ${K8S_MATER_PORT} ${K8S_MATER} "find ${WAR_PATH} -maxdepth 1 -type d -name ${JOB_NAME}-*-${RELEASE_VERSION}")
  ssh -p ${K8S_MATER_PORT} ${K8S_MATER} "rm -rf ${WEBROOT_PATH} && \
                                         ln -s ${History_Version} ${WEBROOT_PATH} && \
                                         kubectl delete pod -l app=java-maven"
}


# 重部署
redeploy () {
  # 如果是以前部署过则删除以前部署的项目目录(包括多个文件夹)否则重新部署;
  if [[ "v${GIT_COMMIT}" = "v${GIT_PREVIOUS_SUCCESSFUL_COMMIT}" ]];then
    echo -e "曾经部署过 ${RELEASE_VERSION} 版本，现在正在重新部署!"
    $(ssh -p ${K8S_MATER_PORT} ${K8S_MATER} "find ${WAR_PATH} -d -maxdepth 1 -type d -name ${JOB_NAME}-*-${RELEASE_VERSION} && find ${WAR_PATH} -d -maxdepth 1 -type f -name ${JOB_NAME}-*-${RELEASE_VERSION}.war")
    # ssh -p ${K8S_MATER_PORT} ${K8S_MATER} "rm -rf ${History_Version}"
  fi
  # 物理如何都要重新部署
  deploy
}

# 部署 & 回退 入口(坑-==两边没有空格)
if [[ "${PREJECT_OPERATION}" = "deploy" ]]; then
  # 坑 (防止字符串为空)
#  if [[ "v${GIT_COMMIT}" = "v${GIT_PREVIOUS_SUCCESSFUL_COMMIT}" ]];then
 #   echo -e "您已经部署过 ${RELEASE_VERSION} 版本"
 #   exit 1
 # else
    deploy
 # fi
elif [[ "${PREJECT_OPERATION}" = "rollback" ]];then
  rollback
elif [[ "${PREJECT_OPERATION}" = "redeploy" ]];then
  redeploy
else
  echo -e "无任何操作！停止执行脚本"
  exit 127
fi
```

-   Step 9.功能分析之Git与Gitlab拉取指定分支并切换分支
    

```
#在“源代码管理”部分中：
1. 单击Git

2. 输入您的存储库URL，例如git@your.gitlab.server:gitlab_group/gitlab_project.git
 #在高级设置，设置名称，以origin和的Refspec到+refs/heads/*:refs/remotes/origin/* +refs/merge-requests/*/head:refs/remotes/origin/merge-requests/*

3. 在“分支说明符”中输入：
# 对于单存储库工作流： origin/${gitlabSourceBranch}
# 对于分叉的存储库工作流： merge-requests/${gitlabMergeRequestIid}

4.在其他行为中：
# 点击添加下拉按钮
# 从下拉列表中选择合并，然后再构建
# 将存储库名称设置为origin
# 将“分支”设置为合并为${gitlabTargetBranch}

# 补充:标签时构建
(1) 在GitLab Webhook配置中，添加“标签推送事件”
(2) 在“源代码管理”下的作业配置中：
  1.选择“高级...”并添加“ `+refs/tags/*:refs/remotes/origin/tags/*` ”作为参考规格
  2.您还可以使用“分支说明符”来指定需要构建的标签（例如“ refs / tags / $ {TAGNAME}”示例）
```

简单示例:

```
// 方式1.
git branch: "${params.RELEASE_VERSION}", credentialsId: 'b4c8b4e9-2777-44a1-a1ed-e9dc21d37f4f', url: "${env.GITLAB_URL}"
// git branch: 'v1.11', credentialsId: '43287e62-ce5b-489a-9c11-cedf38e16e92', url: 'git@gitlab.weiyigeek.top:ci-cd/java-maven.git'

//方式2.
checkout([$class: 'GitSCM', branches: [[name: "origin/${params.RELEASE_VERSION}"]], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: 'b4c8b4e9-2777-44a1-a1ed-e9dc21d37f4f', url: '${env.GITLAB_URL}']]])
```

-   Step 9.功能分析之 Jenkins 同步到 Gitlab 流水线之中,并且从Gitlab中可以直接进入Jenkins Job页面查看构建情况;
    

```
# 语法参考: 包含在 steps 块之中
gitlabCommitStatus(connection: gitLabConnection('Private-Gitlab'), name: '${JOB_NAME}') {
  sh label: 'build', script: '/usr/local/maven/bin/mvn clean package -Dmaven.test.skip=true'
}
```

![WeiyiGeek.jenkins与Gitlab流水线](https://i0.hdslb.com/bfs/article/3b711be9d2b6fbea4a15a35d75db52f913fcffc6.png@942w_561h_progressive.webp)

-   Step 10.功能分析之 Jenkins 中成品进行归档, 注意其路径为相对路径及其您生成的项目打包文件格式文件和Gitlab Relase 发布。
    

```
# (1) 成品归档当前路径为 ${WORKSPACE} 变量路径；
archiveArtifacts artifacts: 'target/*.jar', fingerprint: true, followSymlinks: false, onlyIfSuccessful: true # 对于war项目则'target/*war'

# (2) 获取归档成品的URL(注意为了后续不丢失建议采用专门的服务器进行存储或者在jenkins将此次Job编译进行完整留存)
http://jenkins.weiyigeek.top:8080/job/Gitlab-Pipeline/187/artifact/target/hello-world.war

# (3) 项目 -> 发布 (Release) -> 新建发布 -> 编写相关资源 -> 发布资源
# 推荐格式: https://host/namespace/project/releases/:release/downloads/:filepath
```

![WeiyiGeek.Jenkins归档与Release发布](https://i0.hdslb.com/bfs/article/4a89dcf5112937afd2a721a0fc8f6d010d73ed40.png@942w_936h_progressive.webp)

Tips : Gitlab 项目 Release 自动发布 GitLab CI 工具: `https://gitlab.com/gitlab-org/release-cli/-/blob/master/docs/index.md#usage` 后面有时间再扩充;

-   Step 11.同样访问我们的K8s部署的Pod查看是否部署成功URL: `http://10.10.107.202:30089/`;
    

```
Maven - Hello World - v1.13 - Declarative Pipeline jobs for Gitlab WebHook Trigger
访问时间: Tue Feb 02 2021 14:15:24 GMT+0800 (中国标准时间)
Server : Apache Tomcat/8.5.61 | 10.244.0.245
Client : 10.244.0.1 | 10.244.0.1
Document_Root : /usr/local/tomcat/webapps/ROOT/
URL : 10.10.107.202/index.jsp
```

![WeiyiGeek.Jenkins & gitlab 自动触发](https://i0.hdslb.com/bfs/article/7b5b73738eaed2ba86ff8703aa582e872a5eca4c.png@942w_653h_progressive.webp)

**问题1.在BlueOcean中流水线使用的输入类型不支持。请使用 `经典 Jenkins` 参数化构建。**  
问题原因: 在BlueOcean中不支持选择下拉而只支持文本参数;  
文本参数:

-   git\_tags 默认值 描述信息
    
-   deploy\_option 默认值（deploy 、rollback、redeploy） 描述信息（部署&回退&重部署）
    

![WeiyiGeek.BlueOcean输入](https://i0.hdslb.com/bfs/article/09035e5adb152f19ad5ce885c674fa87556b7c26.png@942w_381h_progressive.webp)

**问题2.添加Webhook测试时显示Hook execution failed: URL 'http://jenkins.weiyigeek.top:8080/project/Gitlab-Pipeline' is blocked: Requests to the local network are not allowed**  
错误信息: Hook execution failed: URL 'http://jenkins.weiyigeek.top:8080/project/Gitlab-Pipeline' is blocked: Requests to the local network are not allowed

解决办法: 允许来自钩子和服务的对本地网络的请求。  
操作流程: 管理中心 -> 设置 -> 网络 -> 勾选 `允许Webhook和服务对本地网络的请求` -> 然后输入 钩子和服务可以访问的本地IP地址和域名。

![WeiyiGeek.外发请求设置](https://i0.hdslb.com/bfs/article/a7eb6c20fe7c14682fa84e1e59e34e16b514522a.png@942w_827h_progressive.webp)

**问题3.Jenkinsfile 编写过程中遇到的情况以及解决办法**

-   1.字符串插值处理
    

```
#设置环境变量
environment {
  STATIC_VAR = "静态变量"
  def dynamic_var = ""
}

# 可选的步骤参数
parameters {
  string(name: 'RELEASE_VERSION', defaultValue: "master", description: 'Message: 请选择部署的Tags版本?', trim: 'True')
  choice(name: 'PREJECT_OPERATION', choices: ['deploy', 'rollback', 'redeploy'], description: 'Message: 选择项目操作方式?')
}

#使用环境变量
echo "${env.STATIC_VAR}"
echo "${params.PREJECT_OPERATION}"

#动态设置环境变量
dynamic_var = "动态设置环境变量"
println(dynamic_var)
```

-   2.凭据处理
    

```
// # Secret 文本，带密码的用户名，Secret 文件
withCredentials([string(credentialsId: 'ce228573-ad3d-40e8-a3bf-b9de510080db', variable: 'GITLAB_TOKEN')]) {
  echo GITLAB_TOKEN  # println(GITLAB_TOKEN)
}

// # ssh私钥密钥凭据类型
// # SSH User Private Key / Public
$ cat id_ed25519.pub | base64 -w 0
// c3NoLWVkMjU1MTkgQUFBQUMzTnphQzFsWkRJMU5URTVBQUFBSUIinfozVnFFbyt1NmZXNExZeWN1bk9QKzBubnE3cWNJd3lBWXVtNUp1Qi8gbWFzdGVyQHdlaXlpZ2Vlay50b3AK
echo "c3NoLWVkMjU1MTkgQUFBQUMzTnphQzFsWkRJMU5URTVBQUFBSUIinfozVnFFbyt1NmZXNExZeWN1bk9QKzBubnE3cWNJd3lBWXVtNUp1Qi8gbWFzdGVyQHdlaXlpZ2Vlay50b3AK" | base64 -d
// ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIB3a0sVqEo+u6fW4LYycunOP+0nnq7qcIwyAYum5JuB/ master@weiyigeek.top

script {
  withCredentials([
      sshUserPrivateKey(credentialsId: '1d4e61cb-5d59-4da8-813b-a5fb345770c9', keyFileVariable: 'keyFile', passphraseVariable: 'pass', usernameVariable: 'user'),
      string(credentialsId: '79a78423-e539-4b1f-a872-f0cf1dd3f9fa', variable: 'keyPub')
  ]) {
    // SSH User Private Key
    writeFile file: 'private.key', text: keyFile
    sh "cp \$(cat private.key) ~/.ssh/id_ed25519"

    // SSH User Public
    writeFile file: 'private.pub', text: keyPub
    sh "echo \$(cat private.pub) | base64 -d > ~/.ssh/id_ed25519.pub"

    //其他方式
    //sh "curl -s http://192.168.12.107:8080/id_ed25519.pub -o ~/.ssh/id_ed25519.pub"
  }
}

// # Certificate
withCredentials([certificate(aliasVariable: 'aliase', credentialsId: '6795a628-97a9-4a18-8dcc-1c913e6901d4', keystoreVariable: 'private', passwordVariable: 'pass')]) {
  # // some block
}
```

![WeiyiGeek.SSH密钥与公钥处理](https://i0.hdslb.com/bfs/article/7e85b94f6ce07234efc413f18a09f54c7a88352b.png@942w_962h_progressive.webp)

-   3.额外处理
    

```
# 处理故障
try {
  sh script: "./test.sh"
  timeout(time: 1, unit: 'MINUTES') {
    waitForQualityGate abortPipeline: true
  }
  updateGitlabCommitStatus name: '代码检测', state: 'success'
} catch(Exception err) {
  updateGitlabCommitStatus name: '代码检测', state: 'failed'
  unstable '代码检测失败'
}
```

  
本章至此完毕! 欢迎食用 (●￣(ｴ)￣●)

![](https://i0.hdslb.com/bfs/article/4adb9255ada5b97061e610b682b8636764fe50ed.png@progressive.webp)

>         欢迎各位志同道合的朋友一起学习交流，如文章有误请在下方留下您宝贵的经验知识，个人邮箱地址【master#weiyigeek.top】

![](https://i0.hdslb.com/bfs/article/02db465212d3c374a43c60fa2625cc1caeaab796.png@progressive.webp)

更多文章来源: https://weiyigeek.top 【WeiyiGeek Blog - 为了能到远方，脚下的每一步都不能少】

![](https://i0.hdslb.com/bfs/article/04ec2a1812ca192ea0870ed12a235e1a3ba7b019.png@942w_483h_progressive.webp)

   如果你觉得这个专栏还不错的，请给这篇专栏点个赞、投个币、收个藏、关个注，这将对我有很大帮助！    

如果你觉得这个专栏还不错的，请给这篇专栏点个赞、投个币、收个藏、关个注，这将对我有很大帮助！