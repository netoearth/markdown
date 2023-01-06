**本章目录:**

0x04 基础使用

-   HelloWorld 自由风格软件项目
    
-   Gitlab 集成配置与实践
    
-   Maven 集成配置与实践
    
-   SonarQube 集成配置与实践
    

![](https://i0.hdslb.com/bfs/article/4aa545dccf7de8d4a93c2b2b8e3265ac0a26d216.png@progressive.webp)

Q: Jenkins Free Style 基本使用?  
目的: 构建一个自由风格的软件项目, 这是Jenkins的主要功能Jenkins将会结合任何SCM和任何构建系统来构建你的项目, 甚至可以构建软件以外的系统.

**操作流程:**

Step 1.项目创建

> (1) 面板 -> 新建一个任务 -> 任务名称 freestyle-helloworld -> 保存  
> (2) Dashboard -> 项目名称 freestyle-helloworld -> General

```
# 重要的配置项
丢弃旧的构建：(保持构建的天数 / 保持构建的最大个数)
参数化构建过程: 比如添加的环境变量或者API密钥等等
```

> (3) Dashboard -> 项目名称 freestyle-helloworld > 构建 > 执行 Shell 命令

```
echo "FreeStyle - HelloWorld"
touch hello-world.txt
```

![WeiyiGeek.自由风格的软件项目](https://i0.hdslb.com/bfs/article/0347f9bec8154e62d6dde6ba01d3b8ba98cd1255.png@942w_356h_progressive.webp)

Step 2.工程 `freestyle-helloworld` 构建与控制台输出查看;  

![WeiyiGeek.工程构建与控制台输出](https://i0.hdslb.com/bfs/article/d5cd8ead86ae47225b4a6ae767c1b941a7872d03.png@942w_479h_progressive.webp)

PS : 如果是蓝色圆点表示构建成功, 如果是红色圆点表示构建失败!

Step 3.构建后的工作区存放目录

```
jenkins:/var/lib/jenkins/workspace/freestyle-helloworld$ ls
  # hello-world.txt
jenkins:/var/lib/jenkins/workspace/freestyle-helloworld$ cat hello-world.txt
```

**简单说明：**  
Q: Jenkins 为啥要集成Gitlab?

> 答：由于我们需要依托于Jenkins将Gitlab上的项目获取至本地，为后续网站的代码发布工作做好准备;

Q: Jenkins 如何集成Gitlab?

> 答: 由于Jenkins 只是一个调度平台，所以需要安装和Gitlab相关的插件即可完成集成;

Jenkins 与 Gitlab 集成思路

-   1.开发提交代码 至 Gitlab
    
-   2.Jenkins 安装 Gitlab 所需插件
    

```
Credentials Plugin (2.3.14) - This plugin allows you to store credentials in Jenkins.
Git client plugin (3.6.0) - Utility plugin for Git support in Jenkins
Git plugin (4.5.0) - This plugin integrates Git with Jenkins.
Gitlab Authentication plugin (1.10) - This is the an authentication plugin using gitlab OAuth.
Gitlab Hook Plugin (1.4.2) - Enables Gitlab web hooks to be used to trigger SMC polling on Gitlab projects
GitLab Plugin (1.5.13) - This plugin allows GitLab to trigger Jenkins builds and display their results in the GitLab UI.
```

![WeiyiGeek.Gitlab关联插件](https://i0.hdslb.com/bfs/article/5871441b42d1f3ebab07258d4052b4e4a3ae3546.png@942w_401h_progressive.webp)

-   3.Jenkins 创建 FreeStyle 项目，然后配置Gitlab仓库对应的地址;
    

**操作流程:**

-   **Step 1.首先需要将我们所用的项目代码推送到Gitlab（此处假设您已安装Gitlab）之中;**
    

> (1) 管理中心 -> 群组 -> 新建群组 -> 完善信息 -> 创建群组  
> (2) 管理中心 -> 项目 -> 创建项目 -> 完善仓库信息 -> 创建项目  
> (3) 推送现有文件夹到Gitlab之中

```
cd existing_folder
git init
git remote add origin http://gitlab.weiyigeek.top/ci-cd/blog.git
git add .
git commit -m "Initial commit"
git push -u origin master
  # remote: Resolving deltas: 100% (829/829), done.
  # To http://gitlab.weiyigeek.top/ci-cd/blog.git
  #  * [new branch]      master -> master
  # Branch 'master' set up to track remote branch 'master' from 'origin'.
```

![WeiyiGeek.gitlab](https://i0.hdslb.com/bfs/article/e6fe5bf9b0fa38f62db8d130145fc5809e851192.png@942w_432h_progressive.webp)

-   **Step 2.此处假设您已经按照上面`Gitlab 集成配置`安装了相应的插件, 并且配置好gitlab域名访问的解析记录;**
    

```
# 此处由于没有自己搭建DNS，则将其写入到Jenkins 服务器的hosts文件中进行相应的IP与域名绑定
cat /etc/hosts
  # 127.0.0.1 localhost
  # 127.0.1.1 gitlab
  # 10.10.107.201 gitlab.weiyigeek.top
```

-   **Step 3.Jenkins 创建 FreeStyle 项目，然后配置Gitlab仓库对应的地址(`ssh 的方式`)**
    

```
# (1) 在Gitlab中添加当前机器ssh密钥(此处以Jenkins用户为例)
jenkins@jenkins:~$ssh-keygen -t ed25519 -C "jenkins@weiyigeek.top"
~$ ls ~/.ssh/id_ed25519
  # id_ed25519      id_ed25519.pub

# (2) 添加公钥到gitlab中
# > 用户设置 -> SSH Keys -> jenkins-CI
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAINFb2ALOnOpePb7e/tss4/ZKxNHb3srh7NXntW5jAWSf jenkins@weiyigeek.top

# (3) 主机认证指纹绑定
jenkins@jenkins:~$ git ls-remote -h -- git@gitlab.weiyigeek.top:ci-cd/blog.git HEAD
  # The authenticity of host 'gitlab.weiyigeek.top (10.10.107.201)' can't be established.
  # ECDSA key fingerprint is SHA256:cmG5Ces+96qG9EEaU1b/tJuTR1re7XDXUU4jg7asna4.
  # Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
```

![WeiyiGeek.SSH Keys](https://i0.hdslb.com/bfs/article/f9ad22ec407e03ed860e39e0502c535e2020ba43.png@942w_312h_progressive.webp)

-   **Step 4.Jenkins > FreeStyle 项目 > 源码管理 > Git > Repositories 设置 > Credentials 设置**  
    PS : 如果是非Jenkins用户下公钥对Gitlab该项目有访问权限时，可以通过 Credentials 添加其认证的密钥即可;
    

```
# (1) Weiyigeek 用户密钥 (此时假设Gitlab已经添加该公钥)
weiyigeek@jenkins:~$ cat /home/weiyigeek/.ssh/id_ed25519
  # -----BEGIN OPENSSH PRIVATE KEY-----
  # b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAAAMwAAAAtzc2gtZW
  # QyNTUxOQAAACAIxH1a4EivkJOSAXqd6LUE8tUvRfgPwwn1Erb7r728cwAAAJgANcy8ADXM
  # vAAAAAtzc2gtZWQyNTUxOQAAACAIxH1a4EivkJOSAXqd6LUE8tUvRfgPwwn1Erb7r728cw
  # AAAECLw4Wnle7Md7fc5NQN6EohejUx0XMHKfdYXejLAw2/NwjEfVrgSK+Qk5IBep3otQTy
  # 1S9F+A/DCfUStvuvvbxzAAAAFG1hc3RlckB3ZWl5aWdlZWsudG9wAQ==
  # -----END OPENSSH PRIVATE KEY-----

# (2) 进行 Credentials 相应的设置
```

![WeiyiGeek.Weiyigeek 用户密钥](https://i0.hdslb.com/bfs/article/db89e2d2c335ef979f81b5e00ed8ec56e46f739a.png@942w_669h_progressive.webp)

**Step 5.首先手动实现设置NFS共享存储以及 Kubernetes 部署 StatefulSet 资源清单，通过Nginx访问我们的blog;**

基础配置:

```
# (1) NFS 服务器配置
root@nfs$ showmount -e
  # /nfs/data4 *
/nfs/data4# ls -alh /nfs/
  # drwxrwxrwx  2 weiyigeek weiyigeek 4.0K Nov 19 13:27 data4
/nfs/data4# mkdir /nfs/data4/tags/
/nfs/data4# ln -s /nfs/data4/tags/ /nfs/data4/web
/nfs/data4# chown -R nobody:weiyigeek /nfs/data4/

# (2) Node 1 && Node 2
sudo mkdir -pv /nfs/data4/
sudo chown -R weiyigeek:weiyigeek /nfs/data4/
sudo mount.nfs -r 10.10.107.202:/nfs/data4/ /nfs/data4/
k8s-node-4:/nfs/data4$ ls -alh  # 权限查看
  # total 12K
  # drwxrwxrwx 3 weiyigeek weiyigeek 4.0K Dec 24 15:44 .
  # drwxr-xr-x 3 root   root 4.0K Dec 24 15:47 ..
  # drwxr-xr-x 2 weiyigeek weiyigeek 4.0K Dec 24 15:44 tags
  # lrwxrwxrwx 1 weiyigeek weiyigeek   16 Dec 24 15:44 web -> /nfs/data4/tags/
```

资源清单:

```
cat > jenkins-gitlab-ci.yaml <<'EOF'
apiVersion: v1
kind: Service
metadata:
  name: deploy-blog-svc
spec:
  type: NodePort         # Service 类型
  selector:
    app: blog-html        # 【注意】与deployment资源控制器创建的Pod标签进行绑定;
    release: stabel       # Service 服务发现不能缺少Pod标签，有了Pod标签才能与之SVC对应
  ports:                  # 映射端口
  - name: http
    port: 80              # cluster 访问端口
    targetPort: 80        # Pod 容器内的服务端口
    nodePort: 30088
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: deploy-blog-html
spec:
  serviceName: "deploy-blog-svc"
  replicas: 3                # 副本数
  selector:                  # 选择器
    matchLabels:
      app: blog-html   # 匹配的Pod标签非常重要
      release: stabel
  template:
    metadata:
      labels:
        app: blog-html      # 模板标签
        release: stabel
    spec:
      volumes:                # 关键点
      - name: web
        hostPath:             # 采用hostPath卷
          type: DirectoryOrCreate   # 卷类型DirectoryOrCreate: 如果子节点上没有该目录便会进行创建
          path: /nfs/data4/web      # 各主机节点上已存在的目录此处是NFS共享
      - name: timezone     # 容器时区设置
        hostPath:
          path: /usr/share/zoneinfo/Asia/Shanghai
      containers:
      - name: blog-html
        image: harbor.weiyigeek.top/test/nginx:v3.0  # 拉取的镜像
        imagePullPolicy: IfNotPresent
        ports:
        - name: http         # 此端口在服务中的名称
          containerPort: 80  # 容器暴露的端口
        volumeMounts:        # 挂载指定卷目录
        - name: web
          mountPath: /usr/share/nginx/html
        - name: logs
          mountPath: /var/log/nginx
        - name: timezone
          mountPath: /usr/share/zoneinfo/Asia/Shanghai
  volumeClaimTemplates:    # 卷的体积要求模板此处采用StorageClass存储类
  - metadata:              # 根据模板自动创建PV与PVC并且进行一一对应绑定;
      name: logs
    spec:
      accessModes: [ "ReadWriteOnce" ]
      storageClassName: managed-nfs-storage # StorageClass存储类
      resources:
        requests:
          storage: 1Gi
EOF
```

部署资源清单:

```
~/K8s/Day12$ kubectl apply -f jenkins-gitlab-ci.yaml
service/deploy-blog-svc unchanged
statefulset.apps/deploy-blog-html configured

~/K8s/Day12$ kubectl get services -o wide
NAME              TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE    SELECTOR
deploy-blog-svc   NodePort    10.104.74.36   <none>        80:30088/TCP   5m9s   app=blog-html,release=stabel
kubernetes        ClusterIP   10.96.0.1      <none>        443/TCP        49d    <none>

~/K8s/Day12$ kubectl get pod -o wide
NAME                                      READY   STATUS    RESTARTS   AGE   IP             NODE          NOMINATED NODE   READINESS GATES
deploy-blog-html-0                        1/1     Running   0          72s   10.244.0.193   Master
deploy-blog-html-1                        1/1     Running   0          57s   10.244.1.182   k8s-node-4
deploy-blog-html-2                        1/1     Running   0          49s   10.244.2.84    k8s-node-5
```

效果查看:

```
curl http://10.10.107.202:30088/host.html
  # Hostname: deploy-blog-html-0 ,Image Version: 3.0, Nginx Version: 1.19.4
```

-   **Step 6.采用Tag方式发布和回退项目**
    

Q: 为什么要让项目支持Tag版本方式上线?

> 答: 采用 Tag 的方式可以直观的知道我们部署的项目版本，同时也便于进行回退;  
> 比如: 第一次上线v1.1第二次上线v1.2，如果此时上线的v1.2出现文件，那么我们可以快速回退至上一个版本v1.1;

使用Tag方式发布与回退思路:

> 1.开发如果需要发布新版本，必须将当前版本打上一个标签。  
> 2.Jenkins需要让其脚本支持传参，比如用户传递v1.1则拉取项目的v1.1标签。

实践操作:

> (1) 首先需要安装 `Git Parameter` 插件(`增加了从项目中配置的git存储库中选择分支、标记或修订的能力。`)，然后配置Jenkins参数化构建过程，让用户在构建时选择对应的Tag版本;  
> 丢弃旧的构建 > 保持构建的最大个数 为 10 个  
> 参数化构建过程 > Git 参数 > git\_version （变量） -> 参数类型 Tags -> 默认值 (origin/master) -> 排序方式 (DESCENDING SMART) 倒序 `防止 Tags 顺序乱`;  
> 选项参数 (选择) > deploy\_option （变量） -> 选项 deploy|rollback  
> 源码管理 > Git > Repository URL:git@gitlab.weiyigeek.top:ci-cd/blog.git > Credentials (密钥认证) -> 分支机构建设 （`指定分支（为空时代表any）`）-> 引入 ${git\_version} 变量  
> 构建 > 执行shell > /bin/sh -x /tmp/script/blog-script.sh  
> 应用 > 保存

![WeiyiGeek.jenkins项目创建](https://i0.hdslb.com/bfs/article/24aad096d09fbbc752cf8552b7e3ca33e8054054.png@942w_627h_progressive.webp)

> (2) 部署和回退脚本以及自动化发布重复构建的问题处理编写

touch /tmp/script/blog-script.sh && chmod a+x $\_

```
#!/bin/bash
# Description: Jenkins CI & Kubernetes & Gitlab -> Deploy or Rollback Blog HTML
DATE=$(date +%Y%m%d-%H%M%S)
TAG_PATH="/nfs/data4/tags"
WEB_PATH="/nfs/data4/web"
WEB_DIR="blog-${DATE}-${git_version}"
K8S_MATER="weiyigeek@10.10.107.202"
K8S_MATER_PORT="20211"


# 部署
deploy () {
  # 1.打包工作目录中的项目
  cd ${WORKSPACE}/ && \
  tar -zcf /tmp/${WEB_DIR}.tar.gz ./*

  # 2.上传打包的项目到master之中
  scp -P ${K8S_MATER_PORT} /tmp/${WEB_DIR}.tar.gz  weiyigeek@10.10.107.202:${TAG_PATH}

  # 3.解压&软链接
  ssh -p ${K8S_MATER_PORT} ${K8S_MATER} "mkdir -p ${TAG_PATH}/${WEB_DIR} && \
                                         tar -xf ${TAG_PATH}/${WEB_DIR}.tar.gz -C ${TAG_PATH}/${WEB_DIR} && \
                                         rm -rf ${WEB_PATH} && \
                                         ln -s ${TAG_PATH}/${WEB_DIR} ${WEB_PATH} && \
                                         kubectl delete pod -l app=blog-html"
}

# 回退
rollback () {
  Previous_Version=$(ssh -p ${K8S_MATER_PORT} ${K8S_MATER} "find ${TAG_PATH} -maxdepth 1 -type d -name blog-*-${git_version}")
  ssh -p ${K8S_MATER_PORT} ${K8S_MATER} "rm -rf ${WEB_PATH} && \
                                         ln -s ${Previous_Version} ${WEB_PATH} && \
                                         kubectl delete pod -l app=blog-html"
}


# 部署 & 回退 入口(坑-==两边没有空格)
if [[ "${deploy_option}" = "deploy" ]]; then
  # 坑 (防止字符串为空)
  if [[ "v${GIT_COMMIT}" = "v${GIT_PREVIOUS_SUCCESSFUL_COMMIT}" ]];then
    echo -e "您已经部署过 ${git_version} 版本"
    exit 1
  else
    deploy
  fi
elif [[ "${deploy_option}" = "rollback" ]];then
  rollback
else
  exit 127
fi
```

> (3) Jenkins 服务器 与 Kubernetes Master 机器中的普通用户进行 ssh 公钥认证登录;

```
# copy 公钥 到 对端主机
ssh-copy-id -p 20211 weiyigeek@10.10.107.202
  # /usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/var/lib/jenkins/.ssh/id_ed25519.pub"
  # authorized only. All activity will be monitored and reported.
  # weiyigeek@10.10.107.202's password:

  # Number of key(s) added: 1

  # Now try logging into the machine, with:   "ssh -p '20211' 'weiyigeek@10.10.107.202'" and check to make sure that only the key(s) you wanted were added.
```

> (4) 为我们的Blog打三个Tag并验证构建脚本

```
$ cd ~/workspace/freestyle-blog
jenkins:~/workspace/freestyle-blog$ git config --global user.email "jenkins@weiygeek.top"
jenkins:~/workspace/freestyle-blog$ git config --global user.name "Weiyigeek"
vim index.html
git add .
git commit -m "v1.1"
git tag v1.1 -m "Jenkins CI test - v1.1"
git push origin HEAD:master
git push origin v1.1
  # Total 0 (delta 0), reused 0 (delta 0)
  # To gitlab.weiyigeek.top:ci-cd/blog.git
  # * [new tag]         v1.1 -> v1.1
```

> (5) 进行项目构建 -> Build With Paramters 查看 部署的版本以及是部署还是回退 -> 如下图所示拉取的指定 Tags ;

![WeiyiGeek.项目构建](https://i0.hdslb.com/bfs/article/5c8b76a47235269ba88f475937bc874f2db4dc49.png@942w_573h_progressive.webp)

> (6) 项目构建部署验证 访问`http://10.10.107.202:30088/`博客地址

```
Built Branches
    v1.3: Build #4 of Revision 4f80504ee142cb9866e4a0988cdc1e1693cfd7aa (v1.3)
    v1.2: Build #3 of Revision 4e1dd6eded12d7cad3740d2461c3e784ad712632 (v1.2)
    v1.1: Build #2 of Revision cd2d5778292ff2cbb7d4ac1b82c684223a38761b (v1.1)

# 如果已经部署过该项目则显示:
  # + '[' deploy==deploy ']'
  # + [[ v4f80504ee142cb9866e4a0988cdc1e1693cfd7aa = \v\4\f\8\0\5\0\4\e\e\1\4\2\c\b\9\8\6\6\e\4\a\0\9\8\8\c\d\c\1\e\1\6\9\3\c\f\d\7\a\a ]]
  # + echo -e '您已经部署过 v1.3 版本'
  # 您已经部署过 v1.3 版本
```

![WeiyiGeek.项目构建部署验证](https://i0.hdslb.com/bfs/article/ba152e32322a6e29f064d2944c3b94f067d53aa8.png@942w_371h_progressive.webp)

> (6) 项目回退验证 > 请选择您要部署的RELEASE版本`v1.1`, 然后选择部署或是回退;

![WeiyiGeek.项目回退验证](https://i0.hdslb.com/bfs/article/de1012277ab267b2401a25eff897644926d1fe83.png@942w_624h_progressive.webp)

**基本概述:**  
Q: 什么是Java项目?

> 答：简单来说就是使用Java编写的代码，我们将其称为Java项目；

Q: 为什么Java项目需要使用Maven编译?

> 答: 由于Java编写的Web服务代码是无法直接在服务器上运行，需要使用Maven工具进行打包;  
> 简单理解: Java 源代码就像汽车的一堆散件，必须经过工厂的组装才能完成一辆完整的汽车，这里组装汽车可以理解是 Maven 编译过程;

Q: 在实现自动化构建Java项目时，先实现手动构建Java项目;

> 答: 因为想要实现自动化发布代码，就必须手动进行一次构建，既是熟悉过程又是优化我们的部署脚本;  
> 大致流程: 源码 -> gitlab -> Maven 手动构建 -> 最后推送构建 -> 发布;

```
# Maven 拉取 Jar 的几种途径
                -> 国外 Maven Jar 服务器
Gitlab -> Maven -> 国内 Maven Jar 镜像服务器
                -> 企业 内部 Maven Jar 私服服务器 (可以双向同步)
```

自动部署 Java 项目至 kubernetes 集群流程步骤：

-   (1) Jenkins 安装 Maven Integration 插件，使Jenkins支持Maven项目的构建
    
-   (2) Jenkins 配置 JDK 路径以及 Maven 路径
    
-   (3) Jenkins 创建 Maven 项目然后进行构建
    
-   (4) 编写自动化上线脚本推送至 kubernetes 集群
    
-   (5) 优化部署脚本使其支持上线与回滚以及，相同版本重复构建
    

**基础环境配置:**

-   Step 1. Jenkins 服务器中 Java 与 Maven 环境配置与 Maven 插件安装;
    

```
# Java 环境
~$ ls /usr/local/jdk1.8.0_211/
~$ java -version
  # java version "1.8.0_211"
  # Java(TM) SE Runtime Environment (build 1.8.0_211-b12)
  # Java HotSpot(TM) 64-Bit Server VM (build 25.211-b12, mixed mode)

# Maven 环境 (两种方式)
# ~$ sudo apt install maven # 方式1
~$ wget https://mirrors.tuna.tsinghua.edu.cn/apache/maven/maven-3/3.6.3/binaries/apache-maven-3.6.3-bin.tar.gz
~$ sudo tar -zxvf apache-maven-3.6.3-bin.tar.gz -C /usr/local
~$ ls /usr/local/apache-maven-3.6.3/
  # bin  boot  conf  lib  LICENSE  NOTICE  README.txt
~$ sudo ln -s /usr/local/apache-maven-3.6.3/ /usr/local/maven  # 非常值得学习(变版本却不变执行路径)
~$ /usr/local/maven/bin/mvn -v
  # Apache Maven 3.6.3 (cecedd343002696d0abb50b32b541b8a6ba2883f)
  # Maven home: /usr/local/maven
  # Java version: 1.8.0_211, vendor: Oracle Corporation, runtime: /usr/local/jdk1.8.0_211/jre
  # Default locale: en_US, platform encoding: UTF-8
  # OS name: "linux", version: "5.4.0-46-generic", arch: "amd64", family: "unix"
~$ export PATH=/usr/local/maven/bin/:$PATH  # 临时
~$ export PATH=${JAVA_HOME}/bin:/usr/local/maven/bin/:$PATH # 加入 /etc/Profile永久
~$ mvn -v

# Maven Integration 插件 安装
# PS : 提供了Jenkins和Maven的深度集成:根据快照在项目之间自动触发，各种Jenkins发布者的自动配置(Junit)。

# Jenkins 中 Maven 集成配置
> 面板 -> 全局工具配置 -> JDK 配置 -> 新增JDK -> 别名 jdk_env -> JAVA_HOME /usr/local/jdk1.8.0_211/
                                -> 新增Maven -> 别名 maven_env -> MAVEN_HOME /usr/local/jdk1.8.0_211/
```

![WeiyiGeek.Maven 集成配置](https://i0.hdslb.com/bfs/article/07c22f3849f953fcbe03a0d0fbf591e9beae0830.png@942w_936h_progressive.webp)

-   Step 2.手动我们的Java的Maven项目, 测试Jenkins服务器上的Maven是否可以正常编译War包并运行在单机的Tomcat8 之中;
    

Maven 镜像配置(您也可选择阿里云的Maven仓库地址加快拉取速度):

```
<!-- # (1) 修改官方的Maven仓库地址为本地私服; -->
158   <mirrors>
159 <!-- Maven 私服地址配置 -->
160     <mirror>
161         <id>weiyigeek-maven</id>
162         <mirrorOf>central</mirrorOf>
163         <name>Private maven</name>
164         <url>http://maven.weiyigeek.top:8081/repository/maven-public/</url>
165     </mirror>
166   </mirrors>

<!-- # (2) Default: ${user.home}/.m2/repository 缓存目录 -->
53   <localRepository>/path/to/local/repo</localRepository>
```

Maven代码项目程序拉取:

```
$ git clone git@gitlab.weiyigeek.top:ci-cd/java-maven.git
  # Cloning into 'java-maven'...
  # The authenticity of host 'gitlab.weiyigeek.top (10.10.107.201)' can't be established.
$ ~/code/java-maven$ ls
pom.xml  src  target
```

构建与运行测试:

```
~/code/java-maven$ mvn clean package -Dmaven.test.skip=true   # 跳过测试用例
  # [INFO] Building war: /home/weiyigeek/code/java-maven/target/hello-world.war
  # [INFO] WEB-INF/web.xml already added, skipping
  # [INFO] ------------------------------------------------------------------------
  # [INFO] BUILD SUCCESS
  # [INFO] ------------------------------------------------------------------------
  # [INFO] Total time:  7.225 s
  # [INFO] Finished at: 2020-12-25T09:23:20Z
  # [INFO] ------------------------------------------------------------------------

~/code/java-maven$ ls -alh /home/weiyigeek/code/java-maven/target/hello-world.war  # 生成的war
-rw-r--r-- 1 weiyigeek weiyigeek 2.0M Dec 25 09:23 /home/weiyigeek/code/java-maven/target/hello-world.war

# 将生成的 hello-world.war 复制到本地的 Tomcat 的Wabapps目录之中他会自动解压
```

![WeiyiGeek.Maven-hello-world](https://i0.hdslb.com/bfs/article/1f0d4cbb92c7e3b069d7776a0ad30e6b6f77cbaf.png@942w_368h_progressive.webp)

-   Step 3.在 Kubernets 上 准备接受 Jenkins 推送过来的war包的项目环境;  
    基础配置:
    

```
# (1) NFS 服务器上配置 (PS:由于测试资源有限 则 nfs 与 k8s master 在同一台机器上)
root@nfs$ showmount -e
  # /nfs/data4 *
/nfs/data4# ls -alh /nfs/
  # drwxrwxrwx  2 weiyigeek weiyigeek 4.0K Nov 19 13:27 data4
/nfs/data4# mkdir /nfs/data4/{war,webapps}/
/nfs/data4# ln -s /nfs/data4/war/ROOT /nfs/data4/webapps/ROOT
/nfs/data4# chown -R weiyigeek:weiyigeek /nfs/data4/

# (2) Node 1 && Node 2 都要执行
sudo mkdir -pv /nfs/data4/
sudo chown -R weiyigeek:weiyigeek /nfs/data4/
sudo mount.nfs -r 10.10.107.202:/nfs/data4/ /nfs/data4/

# (3) master 节点拉取 tomcat 此处选择 8.5.61-jdk8-corretto 并上传到内部私有harbor之中
master$ docker pull tomcat:8.5.61-jdk8-corretto
  # 8.5.61-jdk8-corretto: Pulling from library/tomcat
  # ...
  # Status: Downloaded newer image for tomcat:8.5.61-jdk8-corretto
  # docker.io/library/tomcat:8.5.61-jdk8-corretto
master$ docker tag tomcat:8.5.61-jdk8-corretto harbor.weiyigeek.top/test/tomcat:8.5.61-jdk8-corretto
master$ docker push  harbor.weiyigeek.top/test/tomcat:8.5.61-jdk8-corretto
  # The push refers to repository [harbor.weiyigeek.top/test/tomcat]
  # f13244e4918b: Pushed
  # 536f15f78828: Pushed
  # 1c98b6d16fad: Pushed
  # fdef502bf282: Pushed
  # cee8d35c645b: Pushed
  # 8.5.61-jdk8-corretto: digest: sha256:110d7391739e868daf8c1bdd03fcb7ffe9eaf2b134768b9162e2cd47d58f7255 size: 1368

# (4) 节点拉取验证
k8s-node-4:~$ docker pull harbor.weiyigeek.top/test/tomcat:8.5.61-jdk8-corretto
k8s-node-5:~$ docker pull harbor.weiyigeek.top/test/tomcat:8.5.61-jdk8-corretto

# (5) 将 jenkins 中构建好的war包 上传到 Master 节点 的 /nfs/data4/war 目录之中
jenkins$ scp -P 20211 /var/lib/jenkins/workspace/Maven-HelloWorld/target/hello-world.war weiyigeek@10.10.107.202:/nfs/data4/war
# **************WARNING**************
# Authorized only. All activity will be monitored and reported.
# hello-world.war

# (6) 解压war并建立软链接
master$ /nfs/data4/war$ unzip hello-world.war -d hello-world-v1.0
  # Archive:  hello-world.war
master$ /nfs/data4/war$ ls
  # hello-world-v1.0  hello-world.war
master$ /nfs/data4$ ln -s /nfs/data4/war/hello-world-v1.0/ /nfs/data4/webapps && ls /nfs/data4/webapps # 坑太多该目录不能存在
  # index.jsp  META-INF  WEB-INF
```

资源清单: (`同样此处采用了HostPath的Volumns卷(war项目)与StorageClass存储类(访问日志)`)

```
cat > jenkins-ci-gitlab-to-kubernetes.yaml <<'EOF'
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
          path: /nfs/data4/webapps  # 各主机节点上已存在的目录此处是NFS共享
      - name: timezone     # 容器时区设置
        hostPath:
          path: /usr/share/zoneinfo/Asia/Shanghai
      containers:
      - name: java-maven
        image: harbor.weiyigeek.top/test/tomcat:8.5.61-jdk8-corretto  # 拉取的镜像
        imagePullPolicy: IfNotPresent
        ports:
        - name: http         # 此端口在服务中的名称
          containerPort: 8080  # 容器暴露的端口
        volumeMounts:        # 挂载指定卷目录
        - name: webapps      # Tomcat 应用目录
          mountPath: /usr/local/tomcat/webapps/ROOT
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
EOF
```

资源清单构建:

```
# (1) svc 与 stafefulset 资源创建
~/K8s/Day12$ kubectl apply -f jenkins-ci-gitlab-to-kubernetes.yaml
  # service/deploy-maven-svc created
  # statefulset.apps/deploy-java-maven created

# (2) 查看创建的资源
~/K8s/Day12$ kubectl get svc | grep "deploy-maven-svc" && kubectl get pod -l app=java-maven -o wide
  # deploy-maven-svc   NodePort    10.101.97.223    8080:30089/TCP  7m24s
  # NAME                  READY   STATUS    RESTARTS   AGE   IP             NODE
  # deploy-java-maven-0   1/1     Running   0          10s   10.244.0.212   weiyigeek-ubuntu
  # deploy-java-maven-1   1/1     Running   0          25s   10.244.1.199   k8s-node-4
  # deploy-java-maven-2   1/1     Running   0          29s   10.244.2.102   k8s-node-5

# (3) 访问验证
master$ curl -s http://10.101.97.223:8080 | grep "<p>"
  # <p> Server : Apache Tomcat/8.5.61  | 10.244.2.102 </p>
  # <p> Client : 10.244.0.0 | 10.244.0.0</p>
  # <p> Document_Root : /usr/local/tomcat/webapps/ROOT/  <br/><br/> URL : 10.101.97.223/index.jsp  </p>
master$ curl -s http://10.101.97.223:8080 | grep "<p>"
  # <p> Server : Apache Tomcat/8.5.61  | 10.244.0.212 </p>
  # <p> Client : 10.244.0.1 | 10.244.0.1</p>
  # <p> Document_Root : /usr/local/tomcat/webapps/ROOT/  <br/><br/> URL : 10.101.97.223/index.jsp  </p>
master$ curl -s http://10.101.97.223:8080 | grep "<p>"
  # <p> Server : Apache Tomcat/8.5.61  | 10.244.1.199 </p>
  # <p> Client : 10.244.0.0 | 10.244.0.0</p>
  # <p> Document_Root : /usr/local/tomcat/webapps/ROOT/  <br/><br/> URL : 10.101.97.223/index.jsp  </p>
```

-   Step 4.下面正餐开始了，在 Jenkins 控制台 `点击新建任务 -> 输入任务名称(Maven-HelloWorld) -> 选择构建一个Maven项目`
    

常规设置: 丢弃旧的构建 10 个、参数化构建过程 `Git 参数 git_version | 选项参数 deploy_option （deploy|rollback|redeploy）` (设置与上面相同 PS 注意排序方式为智能排序)

关键点: Maven Build -> `Goals and options : clean package -Dmaven.test.skip=true`

构建后脚本设置: Post Steps -> 执行 Shell 命令 -> `/bin/bash -x /tmp/script/maven-jenkins-ci-script.sh`\-> 应用并保存

构建测试结果:

```
# 项目拉取
Running as SYSTEM
Building in workspace /var/lib/jenkins/workspace/Maven-HelloWorld
The recommended git tool is: NONE
using credential b4c8b4e9-2777-44a1-a1ed-e9dc21d37f4f
Cloning the remote Git repository
Cloning repository git@gitlab.weiyigeek.top:ci-cd/java-maven.git

<===[JENKINS REMOTING CAPACITY]===>channel started
Executing Maven: -B -f /var/lib/jenkins/workspace/Maven-HelloWorld/pom.xml clean package -Dmaven.test.skip=true
# 扫描项目并下载依赖
[INFO] Scanning for projects...
[INFO]
[INFO] -------------------< com.weiyigeek.main:hello-world >-------------------
[INFO] Building hello-world Maven Webapp 1.1-SNAPSHOT
[INFO] --------------------------------[ war ]---------------------------------
[INFO] Downloading from weiyigeek-maven: http://maven.weiyigeek.top:8081/repository/maven-public/org/apache/maven/plugins/maven-clean-plugin/2.5/maven-clean-plugin-2.5.pom

# 打包应用
[INFO] Packaging webapp
[INFO] Assembling webapp [hello-world] in [/var/lib/jenkins/workspace/Maven-HelloWorld/target/hello-world]
[INFO] Processing war project
[INFO] Copying webapp resources [/var/lib/jenkins/workspace/Maven-HelloWorld/src/main/webapp]
[INFO] Webapp assembled in [26 msecs]
[INFO] Building war: /var/lib/jenkins/workspace/Maven-HelloWorld/target/hello-world.war
[INFO] WEB-INF/web.xml already added, skipping
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  6.756 s
[INFO] Finished at: 2020-12-25T13:55:45Z
[INFO] ------------------------------------------------------------------------
Waiting for Jenkins to finish collecting data
[JENKINS] Archiving /var/lib/jenkins/workspace/Maven-HelloWorld/pom.xml to com.weiyigeek.main/hello-world/1.1-SNAPSHOT/hello-world-1.1-SNAPSHOT.pom
[JENKINS] Archiving /var/lib/jenkins/workspace/Maven-HelloWorld/target/hello-world.war to com.weiyigeek.main/hello-world/1.1-SNAPSHOT/hello-world-1.1-SNAPSHOT.war
channel stopped
Finished: SUCCESS
```

-   Step 5.Jenkins CI 部署Shell脚本编写,脚本需求部署、回退、可重复构建;  
    脚本与权限: `su - "jenkins" -c "touch /tmp/script/maven-jenkins-ci-script.sh && chmod a+x /tmp/script/maven-jenkins-ci-script.sh"`
    
  

```
#!/bin/bash
# Description: Jenkins CI & Kubernetes & Gitlab -> Deploy or Rollback or Redeploy Java Maven Project
DATE=$(date +%Y%m%d-%H%M%S)
WAR_PATH="/nfs/data4/war"
WEBROOT_PATH="/nfs/data4/webapps"
WEB_DIR="${JOB_NAME}-${DATE}-${git_version}"
WAR_DIR="${WAR_PATH}/${WEB_DIR}"
WAR_NAME="${WEB_DIR}.war"
K8S_MATER="weiyigeek@10.10.107.202"
K8S_MATER_PORT="20211"

# 部署
deploy () {
  # 1.上传Maven打包的war包到master之中
  scp -P ${K8S_MATER_PORT} ${WORKSPACE}/target/*.war  weiyigeek@10.10.107.202:${WAR_PATH}/${WAR_NAME}

  # 2.解压&软链接
  ssh -p ${K8S_MATER_PORT} ${K8S_MATER} "unzip ${WAR_PATH}/${WAR_NAME} -d ${WAR_DIR} && \
                                         rm -rf ${WEBROOT_PATH} && \
                                         ln -s ${WAR_PATH}/${WEB_DIR} ${WEBROOT_PATH} && \
                                         kubectl delete pod -l app=java-maven"
}

# 回退
rollback () {
  History_Version=$(ssh -p ${K8S_MATER_PORT} ${K8S_MATER} "find ${WAR_PATH} -maxdepth 1 -type d -name ${JOB_NAME}-*-${git_version}")
  ssh -p ${K8S_MATER_PORT} ${K8S_MATER} "rm -rf ${WEBROOT_PATH} && \
                                         ln -s ${History_Version} ${WEBROOT_PATH} && \
                                         kubectl delete pod -l app=java-maven"
}


# 重部署
redeploy () {
  # 如果是以前部署过则删除以前部署的项目目录,否则重新部署;
  if [[ "v${GIT_COMMIT}" = "v${GIT_PREVIOUS_SUCCESSFUL_COMMIT}" ]];then
    echo -e "曾经部署过 ${git_version} 版本，现在正在重新部署!"
    History_Version=$(ssh -p ${K8S_MATER_PORT} ${K8S_MATER} "find ${WAR_PATH} -maxdepth 1 -type d -name ${JOB_NAME}-*-${git_version}")
    ssh -p ${K8S_MATER_PORT} ${K8S_MATER} "rm -rf ${History_Version}"
  fi
  # 物理如何都要重新部署
  deploy
}

# 部署 & 回退 入口(坑-==两边没有空格)
if [[ "${deploy_option}" = "deploy" ]]; then
  # 坑 (防止字符串为空)
  if [[ "v${GIT_COMMIT}" = "v${GIT_PREVIOUS_SUCCESSFUL_COMMIT}" ]];then
    echo -e "您已经部署过 ${git_version} 版本"
    exit 1
  else
    deploy
  fi
elif [[ "${deploy_option}" = "rollback" ]];then
  rollback
elif [[ "${deploy_option}" = "redeploy" ]];then
  redeploy
else
  echo -e "无任何操作！停止执行脚本"
  exit 127
fi
```

-   Step 6.修改h1标签为我们的项目打几个Tag标签然后Push到仓库;
    

![WeiyiGeek.show tags](https://i0.hdslb.com/bfs/article/29816acf5fd9068d0be8db0d16d3d1fac708a398.png@942w_399h_progressive.webp)

-   Step 7.验证我们的构建的Maven项目以及推送构建的war到kubernetes集群之中(激动人心的时刻即将到来);
    

PS: 查看任务控制台输出信息确定脚本部署没有任何问题，之后访问`http://10.10.107.202:30089/`查看编译的war运行在kunbernets中部署Tomcat应用的结果;

![WeiyiGeek.jenkins-ci-deploy](https://i0.hdslb.com/bfs/article/3594b4b39ce37715acb4335cae15e74e62bb5dfa.png@942w_458h_progressive.webp)

-   Step 8.回滚测试 构建参数: `deploy_option->rollback , git_version -> v1.1`
    

```
# 构建推送反馈
[Maven-HelloWorld] $ /bin/sh -xe /tmp/jenkins6930799554885763742.sh
+ /bin/bash -x /tmp/script/maven-jenkins-ci-script.sh
++ date +%Y%m%d-%H%M%S
+ DATE=20201227-021208
+ WAR_PATH=/nfs/data4/war
+ WEBROOT_PATH=/nfs/data4/webapps
+ WEB_DIR=Maven-HelloWorld-20201227-021208-v1.1
+ WAR_DIR='/nfs/data4/war/Maven-HelloWorld-20201227-021208-v1.1'
+ WAR_NAME=Maven-HelloWorld-20201227-021208-v1.1.war
+ K8S_MATER=weiyigeek@10.10.107.202
+ K8S_MATER_PORT=20211
+ [[ rollback = \d\e\p\l\o\y ]]
+ [[ rollback = \r\o\l\l\b\a\c\k ]]  # 进行回滚操作
+ rollback
++ ssh -p 20211 weiyigeek@10.10.107.202 'find /nfs/data4/war -maxdepth 1 -type d -name Maven-HelloWorld-*-v1.1'
**************WARNING**************
Authorized only. All activity will be monitored and reported.
+ History_Version=/nfs/data4/war/Maven-HelloWorld-20201227-020140-v1.1  # 历史版本
+ ssh -p 20211 weiyigeek@10.10.107.202 'rm -rf /nfs/data4/webapps &&                                          ln -s /nfs/data4/war/Maven-HelloWorld-20201227-020140-v1.1 /nfs/data4/webapps &&                                          kubectl delete pod -l app=java-maven'
**************WARNING**************
Authorized only. All activity will be monitored and reported.
pod "deploy-java-maven-0" deleted
pod "deploy-java-maven-1" deleted
pod "deploy-java-maven-2" deleted
Finished: SUCCESS

# 应用反馈
http://10.10.107.202:30089/
  # Maven - Hello World - v1.1
  # 访问时间: Sun Dec 27 2020 10:12:18 GMT+0800 (中国标准时间)
  # Server : Apache Tomcat/8.5.61 | 10.244.1.203
  # Client : 10.244.0.0 | 10.244.0.0
  # Document_Root : /usr/local/tomcat/webapps/ROOT/
  # URL : 10.10.107.202/index.jsp
```

-   Step 9.重部署测试 构建参数: `deploy_option->redeploy , git_version -> v1.2`
    

```
# (1) 我们先来看看 war 目录的v1.2版本的文件与目录名称
/nfs/data4/war$ ls -alh | grep "-v1.2"
drwxrwxr-x 4 weiyigeek weiyigeek 4.0K Dec 27 10:09 Maven-HelloWorld-20201227-020939-v1.2
-rw-r--r-- 1 weiyigeek weiyigeek 2.0M Dec 27 10:09 Maven-HelloWorld-20201227-020939-v1.2.war

# (2) 项目重部署控制台信息输出
[Maven-HelloWorld] $ /bin/sh -xe /tmp/jenkins8520069315177369772.sh
channel stopped
+ /bin/bash -x /tmp/script/maven-jenkins-ci-script.sh
++ date +%Y%m%d-%H%M%S
+ DATE=20201227-021934
+ WAR_PATH=/nfs/data4/war
+ WEBROOT_PATH=/nfs/data4/webapps
+ WEB_DIR=Maven-HelloWorld-20201227-021934-v1.2
+ WAR_DIR='/nfs/data4/war/Maven-HelloWorld-20201227-021934-v1.2'
+ WAR_NAME=Maven-HelloWorld-20201227-021934-v1.2.war
+ K8S_MATER=weiyigeek@10.10.107.202
+ K8S_MATER_PORT=20211
+ [[ redeploy = \d\e\p\l\o\y ]]
+ [[ redeploy = \r\o\l\l\b\a\c\k ]]
+ [[ redeploy = \r\e\d\e\p\l\o\y ]]
+ redeploy   # 进入了重部署脚本选择
+ [[ ve8d88cf3e222b79259edcfb7ca48cee7b079ee08 = \v\e\8\d\8\8\c\f\3\e\2\2\2\b\7\9\2\5\9\e\d\c\f\b\7\c\a\4\8\c\e\e\7\b\0\7\9\e\e\0\8 ]]
+ echo -e '曾经部署过 v1.2 版本，现在正在重新部署!'
曾经部署过 v1.2 版本，现在正在重新部署!
++ ssh -p 20211 weiyigeek@10.10.107.202 'find /nfs/data4/war -maxdepth 1 -type d -name Maven-HelloWorld-*-v1.2'
+ History_Version=/nfs/data4/war/Maven-HelloWorld-20201227-020939-v1.2
+ ssh -p 20211 weiyigeek@10.10.107.202 'rm -rf /nfs/data4/war/Maven-HelloWorld-20201227-020939-v1.2' # 删除了原来v1.2版本的war解压目录

# 重新构建
+ deploy
+ scp -P 20211 /var/lib/jenkins/workspace/Maven-HelloWorld/target/hello-world.war weiyigeek@10.10.107.202:/nfs/data4/war/Maven-HelloWorld-20201227-021934-v1.2.war
# 解压上传的war 并创建软连接
+ ssh -p 20211 weiyigeek@10.10.107.202 'unzip /nfs/data4/war/Maven-HelloWorld-20201227-021934-v1.2.war -d  /nfs/data4/war/Maven-HelloWorld-20201227-021934-v1.2 &&                                          rm -rf /nfs/data4/webapps &&                                          ln -s /nfs/data4/war/Maven-HelloWorld-20201227-021934-v1.2 /nfs/data4/webapps &&                                          kubectl delete pod -l app=java-maven'
Archive:  /nfs/data4/war/Maven-HelloWorld-20201227-021934-v1.2.war
   creating: /nfs/data4/war/Maven-HelloWorld-20201227-021934-v1.2/META-INF/
....
  inflating: /nfs/data4/war/Maven-HelloWorld-20201227-021934-v1.2/META-INF/maven/com.weiyigeek.main/hello-world/pom.properties
pod "deploy-java-maven-0" deleted
pod "deploy-java-maven-1" deleted
pod "deploy-java-maven-2" deleted

Finished: SUCCESS

# (3) 应用反馈
Maven - Hello World - v1.2
访问时间: Sun Dec 27 2020 10:20:03 GMT+0800 (中国标准时间)
Server : Apache Tomcat/8.5.61 | 10.244.2.107
Client : 10.244.0.0 | 10.244.0.0
Document_Root : /usr/local/tomcat/webapps/ROOT/
URL : 10.10.107.202/index.jsp
```

PS : 至此 Maven 与 Jenkins 集成实践完成

**补充介绍: 除了上面在项目的Pom.xml中设置内部Maven服务器，我们可以可以采样下面两种方式指定配置文件**

-   方式1.通过 curl 在构建前下载 Git 版本控制的 Maven 自定义的 setting.xml 配置文件（在我后面K8s中构建时会看见）。
    
-   方式2.采用 `Managed file(可以配置 Maven 的全局配置文件和用户配置文件)` 和 `Maven Integration` 插件
    

```
// Config File Provider 插件使用配置文件
configFileProvider([configFile(fileId:'maven-global-settings',variable:'MAVEN_GLOBAL_ENV')]) {
sh "mvn -s $MAVEN_GLOBAL_ENV clean install"
}

// Pipeline Maven Integration 插件使用配置文件
withMaven(
  // Maven installation declared in the Jenkins "Global Tool Configuration"
  maven: 'maven-3',
  // Maven settings.xml file defined with the Jenkins Config File Provider Plugin
  // We recommend to define Maven settings.xml globally at the folder level using
  // navigating to the folder configuration in the section "Pipeline Maven Configuration / Override global Maven configuration"
  // or globally to the entire master navigating to  "Manage Jenkins / Global Tools Configuration"
  mavenSettingsConfig: 'my-maven-settings') {
    // Run the maven build
    sh "mvn clean verify"

  } // withMaven will discover the generated Maven artifacts, JUnit Surefire & FailSafe & FindBugs & SpotBugs reports...
```

描述: 在Jenkins持续集成中中可以在构建代码前对我们项目进行一个代码质量扫描检测, 此处我们引入SonarQube进行实现;

**操作流程:**

-   Step 0.在 SonarQube Web中进行认证 Token 生成：(手工设置)\_添加项目 -> 项目标识 -> 创建一个令牌 (Jenkins) -> 得到Token;
    

```
Jenkins: 755eeb453cb28f96aa9d4ea14334da7287c2e840
# 此令牌用于执行分析时认证时使用，如果这个令牌存在问题，可以随时在你的用户账号下取消这个令牌。
```

-   Step 1.插件安装: 系统管理 -> 插件管理 -> `SonarQube Scanner for Jenkins`（）
    

![WeiyiGeek.SonarQube Scanner for Jenkins](https://i0.hdslb.com/bfs/article/a98ad35c7c585c49ae3a2244a566cfeba9d399ef.png@942w_342h_progressive.webp)

-   Step 2.在Jenkins系统管理->系统设置以及全局工具管理->上配置SonarQube相关配置(服务端地址，以及客户端工具地址);
    

```
# (1) 服务端
Dashboard -> 配置 -> SonarQube servers -> Add SonarQube 设置名称和SonarQube地址 -> 添加Token凭据（类型：Secret Text） -> Jenklins-Connet-Sonarqube-Token

# (2) 客户端
Dashboard -> 全局工具配置 -> SonarQube Scanner -> 名称（sonarqube-scanner） -> 工具路径 （/usr/local/sonar）-> 保存应用
```

![WeiyiGeek.Jenkins上SonarQube配置](https://i0.hdslb.com/bfs/article/54e15b5b79d0b3f27778b03708b764e0600974a1.png@942w_746h_progressive.webp)

-   Step 3.应用项目分析实战在分析项目中配置`Analysis Properties`\->保存和应用-> 此时项目中会多一个SonarQube的图标;  
    PS : 下述图中有误应该是在`Pre Steps阶段(Execute SonarQube Scanner，其次 Build 构建即可)`
    

```
# 构建前进行分析配置（名称、唯一标识、检测目录）
sonar.projectName=${JOB_NAME}
sonar.projectKey=Jenkis
sonar.source=.


# 已配置不需要了
# sonar.host.url=http://sonar.weiyigeek.top:9000 \
# sonar.login=755eeb453cb28f96aa9d4ea14334da7287c2e840
```

![WeiyiGeek.SonarQube Analysis Properties](https://i0.hdslb.com/bfs/article/5e552d8d6a786417fa89ec589cf7e90f73847335.png@942w_693h_progressive.webp)

-   Step 4.新建立一个V1.7Tag并上传到Gitlab之中，之后进行检测与构建操作->返回工程点击SonarQube 图标进行项目检测结果页面;
    

```
WeiyiGeek@WeiyiGeek MINGW64 /e/EclipseProject/hello-world (master)
$ git add .
$ git commit -m "v1.7"
  # [master 0f50b10] v1.7
  # 1 file changed, 1 insertion(+), 1 deletion(-)
$ git tag -a "v1.7" -m "v1.7"
$ git push
  # Enumerating objects: 11, done.
  # Counting objects: 100% (11/11), done.
  # .....
  # To http://gitlab.weiyigeek.top/ci-cd/java-maven.git
  #    3bfb942..0f50b10  master -> master

$ git push origin v1.7
  # ...
  # To http://gitlab.weiyigeek.top/ci-cd/java-maven.git
  # * [new tag]         v1.7 -> v1.7
```

![WeiyiGeek.SonarQube项目检测结果](https://i0.hdslb.com/bfs/article/9e2684211f1cba6335d489c8ca535d1fe0e63fd7.png@942w_644h_progressive.webp)

-   Step 5.项目在K8s集群中部署结果
    

```
# https://10.10.107.202:30089/
Maven - Hello World - v1.7
访问时间: Sun Jan 03 2021 13:20:34 GMT+0800 (中国标准时间)
Server : Apache Tomcat/8.5.61 | 10.244.2.111
Client : 10.244.0.0 | 10.244.0.0
Document_Root : /usr/local/tomcat/webapps/ROOT/
URL : 10.10.107.202/index.jsp
```

原文地址（1）: https://mp.weixin.qq.com/s/\_ybyBhr6DQKePuRxkh5yFQ

原文地址（2）:https://mp.weixin.qq.com/s/DdTwyWF9gM9gBS4M72HXYg

![](https://i0.hdslb.com/bfs/article/db75225feabec8d8b64ee7d3c7165cd639554cbc.png@progressive.webp)

>               如果你觉得这个专栏还不错的，请给这篇专栏点个赞、投个币、收个藏、关个注，这将对我有很大帮助！     
> 
>               欢迎各位志同道合的朋友一起学习交流，如文章有误请在下方留下您宝贵的经验知识，个人邮箱地址【master#weiyigeek.top】

![](https://i0.hdslb.com/bfs/article/02db465212d3c374a43c60fa2625cc1caeaab796.png@progressive.webp)

更多文章来源: https://weiyigeek.top 【WeiyiGeek Blog - 为了能到远方，脚下的每一步都不能少】 

![](https://i0.hdslb.com/bfs/article/fd8645896d98c4d98503adf2e586e5f992a9a2c6.png@942w_483h_progressive.webp)

个人主页：https://weiyigeek.top