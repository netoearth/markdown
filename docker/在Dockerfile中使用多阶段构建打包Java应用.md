多阶段构建指在Dockerfile中使用多个FROM语句，每个FROM指令都可以使用不同的基础镜像，并且是一个独立的子构建阶段。使用多阶段构建打包Java应用具有构建安全、构建速度快、镜像文件体积小等优点，本文以Github上的Java Maven项目为例，结合阿里云容器镜像服务（ACR）的镜像构建服务，介绍如何进行多阶段构建。

## 前提条件

-   开通[阿里云容器镜像服务](https://cr.console.aliyun.com/)。
-   请准备一个托管在[Github](https://github.com/)、[Gitlab](https://gitlab.com/)、[Bitbucket](https://bitbucket.org/)或者[阿里云Code](https://code.aliyun.com/)平台上的Java源代码仓库。

## 背景信息

**镜像构建的通用问题**

镜像构建服务使用Dockerfile来帮助用户构建最终镜像，但在具体实践中，存在一些问题：

-   Dockerfile编写有门槛
    
    开发者（尤其是Java）习惯了语言框架的编译便利性，不知道如何使用Dockerfile构建应用镜像。
    
-   镜像容易臃肿
    
    构建镜像时，开发者会将项目的编译、测试、打包构建流程编写在一个Dockerfile中。每条Dockerfile指令都会为镜像添加一个新的图层，从而导致镜像层次深，镜像文件体积特别大。
    
-   存在源码泄露风险
    
    打包镜像时，源代码容易被打包到镜像中，从而产生源代码泄漏的风险。
    

**多阶段构建优势**

针对Java这类的编译型语言，使用Dockerfile多阶段构建，具有以下优势：

-   保证构建镜像的安全性
    
    当您使用Dockerfile多阶段构建镜像时，需要在第一阶段选择合适的编译时基础镜像，进行代码拷贝、项目依赖下载、编译、测试、打包流程。在第二阶段选择合适的运行时基础镜像，拷贝基础阶段生成的运行时依赖文件。最终构建的镜像将不包含任何源代码信息。
    
-   优化镜像的层数和体积
    
    构建的镜像仅包含基础镜像和编译制品，镜像层数少，镜像文件体积小。
    
-   提升构建速度
    
    使用构建工具（Docker、Buildkit等），可以并发执行多个构建流程，缩短构建耗时。
    

以[Java Maven项目](https://github.com/zlseu-edu/simple-java-maven-app)为例，在Java Maven项目中新建Dockerfile文件，并在Dockerfile文件添加以下内容。

**说明** 该Dockerfile文件使用了二阶段构建。

-   第一阶段：选择Maven基础镜像（Gradle类型也可以选择相应Gradle基础镜像）完成项目编译，拷贝源代码到基础镜像并运行RUN命令，从而构建Jar包。
-   第二阶段：拷贝第一阶段生成的Jar包到OpenJDK镜像中，设置CMD运行命令。

```
# First stage: complete build environment
FROM maven:3.5.0-jdk-8-alpine AS builder

# add pom.xml and source code
ADD ./pom.xml pom.xml
ADD ./src src/

# package jar
RUN mvn clean package

# Second stage: minimal runtime environment
From openjdk:8-jre-alpine

# copy jar from the first stage
COPY --from=builder target/my-app-1.0-SNAPSHOT.jar my-app-1.0-SNAPSHOT.jar

EXPOSE 8080

CMD ["java", "-jar", "my-app-1.0-SNAPSHOT.jar"]
```

## 步骤二：绑定源代码仓库

在容器镜像服务控制台，绑定您托管的代码仓库，以下内容以GitHub为例。

1.  登录[容器镜像服务控制台](https://cr.console.aliyun.com/)。
2.  在顶部菜单栏，选择所需地域。
3.  在左侧导航栏，选择实例列表。
4.  在实例列表页面单击个人版实例。
5.  在个人版实例管理页面左侧导航栏中选择。
6.  在GitHub栏单击绑定账号，在弹出的对话框中单击点击前往源代码仓库登录，跳转到GitHub。
7.  在授权界面，单击Authorize AliyunDeveloper。
    
    ![授权](https://static-aliyun-doc.oss-accelerate.aliyuncs.com/assets/img/zh-CN/0965359951/p129691.png)
    
    返回至代码源界面，GitHub栏中显示已绑定，表示绑定成功。
    

## 步骤三：新建镜像仓库

1.  登录[容器镜像服务控制台](https://cr.console.aliyun.com/)。
2.  在顶部菜单栏，选择所需地域。
3.  在左侧导航栏，选择实例列表。
4.  在实例列表页面单击个人版实例。
5.  在个人版实例管理页面左侧导航栏中选择，然后单击创建镜像仓库。
6.  设置镜像仓库信息。
    
    ![仓库信息](https://static-aliyun-doc.oss-accelerate.aliyuncs.com/assets/img/zh-CN/0965359951/p129695.png)
    
     
    | 配置项 | 描述 |
    | --- | --- |
    | 地域 | 镜像仓库所在区域，目前阿里云容器镜像服务已经全球23个区域开服。 |
    | 命名空间 | 仓库所属命名空间。一个镜像仓库必须且仅属于一个命名空间。一个命名空间下可以包含多个镜像仓库。 |
    | 仓库名称 | 输入仓库名称。 |
    | 仓库类型 | 仓库类型分为公开和私有。无论公开还是私有类型仓库，推送镜像都需要先进行登录。
    -   公开：拉取镜像时可以免登录，直接通过网络拉取。
    -   私有：必须要通过Docker客户端进行登录，才能拉取镜像。
    
     |
    | 摘要 | 简要描述信息。 |
    | 描述信息 | 完整描述信息。 |
    
7.  单击下一步，设置代码源。
    
    -   代码源：将代码源设为GitHub，选择[步骤二：绑定源代码仓库](https://help.aliyun.com/document_detail/173175.htm?spm=a2c4g.11186623.0.0.168f3e06OhjuMu#section-00k-o0q-dor)中绑定的源代码仓库。
    -   构建设置 ：
        1.  代码变更时自动构建镜像：当分支有代码提交后会自动触发构建规则。
        2.  海外机器构建：构建时会在海外机房构建，构建成功后推送到指定地域。
        3.  不使用缓存：每次构建时会强制重新拉取基础依赖镜像，可能会增加构建时间。
    
8.  单击创建镜像仓库。
    
    选择，在镜像仓库界面查找到创建的仓库，表示创建镜像仓库成功。
    

## 步骤四：执行构建

1.  在左侧导航栏中选择，单击仓库名称或目标仓库操作列的管理，进入仓库详情页面。
2.  单击左侧导航栏中的构建，在构建规则设置区域的单击添加规则。
3.  设置构建规则。
    
    ![构建规则](https://static-aliyun-doc.oss-accelerate.aliyuncs.com/assets/img/zh-CN/0965359951/p129721.png)
    
     
    | 配置项 | 描述 |
    | --- | --- |
    | 类型 | 定义了推送代码到托管仓库，触发构建规则的事件。目前有Branch和Tag两种类型的推送。 |
    | Branch/Tag | 设置构建的代码分支。 |
    | Dockerfile目录 | 设置Dockerfile文件所在的目录。这里的目录指的是相对目录，以代码分支的根目录为父目录。 |
    | Dockerfile文件名 | 设置Dockerfile文件名，默认为Dockerfile。 |
    | 镜像版本 | 设置镜像版本。 |
    
4.  单击确认，返回至构建页面。
5.  在构建规则设置区域中，找到创建的规则，单击目标规则对应操作列的立即构建。
    
    在构建日志区域中找到构建记录，当构建状态显示成功，表示构建成功。
    

## 执行结果

-   查看构建的镜像
    
    在个人版实例管理页面左侧导航栏中选择，单击仓库名称或目标仓库操作列的管理，进入仓库详情页面。单击镜像版本，查看构建的镜像。
    
-   查看镜像层数
    
    使用Docker客户端登录镜像仓库，执行以下命令，拉取构建的镜像并检查镜像层数。可以发现仅包含了第一阶段编译Jar包，大小增加有限，镜像层数控制在四层。
    
    ```
    docker pull registry.cn-hangzhou.aliyuncs.com/docker-builder/java-multi-stage:master
    
    docker inspect registry.cn-hangzhou.aliyuncs.com/docker-builder/java-multi-stage:master | jq -s .[0][0].RootFS
    {
      "Type": "layers",
      "Layers": [
        "sha256:f1b5933fe4b5f49bbe8258745cf396afe07e625bdab3168e364daf7c956b6b81",
        "sha256:9b9b7f3d56a01e3d9076874990c62e7a516cc4032f784f421574d06b18ef9aa4",
        "sha256:edd61588d12669e2d71a0de2aab96add3304bf565730e1e6144ec3c3fac339e4",
        "sha256:2be89a7ccd49878fb5f09d6dfa5811ce09fc1972f0a987bbb5a006992aa3dff3"
      ]
    }
    ```
    
-   运行镜像，查看运行结果
    
    使用Docker客户端登录镜像仓库，执行以下命令，运行镜像，查看运行结果。
    
    ```
    docker run -ti registry.cn-hangzhou.aliyuncs.com/docker-builder/java-multi-stage:master
    Hello World!
    ```