[< 返回新闻公共列表](https://www.pianyun.com/news/)

### 超详细实操教程！在现有K8S集群上安装JenkinsX，极速提升CI/CD体验！ - 云计算

发布时间：2022-12-27 16:32:17

在2018年年初，Jenkins X首次发布，它由Apache Groovy语言的创建者Jame Strachan创建。Jenkins X 是一个高度集成化的 CI/CD 平台，基于 Jenkins 和 Kubernetes 实现，旨在解决微服务体系架构下的云原生应用的持续交付的问题，简化整个云原生应用的开发、运行和部署过程。仅需一条Jenkins X命令，管理员可以创建一个Kubernetes集群，并安装用于管理应用程序、创建流水线并部署一个应用程序到不同的环境中的工具。

Jenkin X还是由插件配置的可扩展自动化[服务器](https://www.pianyun.com/ "服务器")，可充当持续集成（CI）服务器，持续部署（CD）hub和自动化测试。

   
Jenkins X（也称为JX）可以轻松地安装在现有的云提供商上（如GKE、AKS等）。或者，如果你由本地Kubernetes集群，也可以使用Jenkins X。通过jx命令，你可以在本地或远程云提供商（如Google Cloud Platform）上快速部署集群。

本文将教您完成在Ubuntu Server 18.04上运行的现有Kubernetes集群上安装Jenkins X的过程。  
 

## 前期准备

   
我将演示在本地和Google Cloud Platform上部署Kubernetes集群（使用Jenkins X）。为此，您需要：

-   已安装Kubernetes的Ubuntu Server的运行实例。
    
-   一个Google Cloud Platform帐户。
    
-   具有sudo权限的用户。
    
-   网络连接。

   
除此之外，还需一点时间。

让我们开始吧！  
 

## 安装Jenkins X

在Ubuntu上安装Jenkins X十分简单。从Jenkins X Github官方页面（https://github.com/jenkins-x/ ）上下载可执行的二进制文件，然后将其移到正确的目录中。为此，请通过SSH登录到服务器，或直接登录到控制台，在服务器出现bash提示后，输入命令：  
 

```
curl -L "https://github.com/jenkins-x/jx/releases/download/$(curl --silent "https://github.com/jenkins-x/jx/releases/latest" | sed 's#.*tag/\(.*\)\".*#\1#')/jx-linux-amd64.tar.gz" | tar xzv "jx"
```

 

以上命令将下载最新版本的Jenkins X，然后解压二进制文件。命令完成之后，你应该在当前工作目录中看到一个名为jx的可执行文件（如下图）：

 ![超详细实操教程！在现有K8S集群上安装JenkinsX，极速提升CI/CD体验！](https://www.pianyun.com/news/content/Uploads/2022-12-27/46289.jpg)

 为了移动Jenkins X二进制文件，请输入以下命令：

```
sudo mv jx /usr/local/bin
```

 

如果你选择使用一个虚拟机环境来部署一个集群，你必须安装它。为此，你需要安装KVM、KVM-2或VirtualBox。为了简化操作，我们将安装VirtualBox。这会安装X server，但你无需使用它。

 要安装VirtualBox，请输入命令：

```
sudo apt-get install virtualbox -y
```

   
安装将花费一些时间，等安装结束之后，你需要在Ubuntu Server上安装minikube（这将是我们的提供商）。为了完成这一操作，使用以下命令下载必要的文件：  
 

```
wget https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
```

   
使用以下命令更改下载文件的权限：  
 

```
chmod +x minikube-linux-amd64
```

   
使用命令移动（并重命名）文件到适当的目录中：  
 

```
sudo mv minikube-linux-amd64 /usr/local/bin/minikube
```

   
使用以下命令，你应该能看到minikube已经安装完成：  
 

```
minikube version
```

 

下图将展示minikube的版本号：

 ![超详细实操教程！在现有K8S集群上安装JenkinsX，极速提升CI/CD体验！](https://www.pianyun.com/news/content/Uploads/2022-12-27/46290.jpg)

## jx命令部署一个集群

现在，我们要去部署一个集群，这一集群将使用minikube和VirtualBox作为驱动。部署集群的命令如下：  
 

```
jx create cluster minikube
```

   
你将会被问到以下问题：  
 

-   应用于集群的内存量（默认为4096）
    
-   应用于集群的核心数（默认为3）
    
-   磁盘大小（默认为150GB）
    
-   选择驱动程序（从kvm、kvm2、virtualBox、无中选择）

如果你选择以下选项：

-   内存4096
    
-   核心3
    
-   磁盘空间20GB
    
-   VirtualBox驱动程序

   
有效的命令如下：  
 

```
minikube start --memory 4096 --cpus 3 --disk-size 20GB --vm-driver virtualbox --bootstrapper=kubeadm
```

   
你也可以不使用驱动在本地部署一个集群。要完成此操作，你必须使用通过sudo运行jx命令，如：  
 

```
sudo jx create cluster minikube --local-cloud-environment=true
```

   
命令将运行如下：  
 

```
minikube start --memory 4096 --cpus 3 --disk-size 20GB --vm-driver none --bootstrapper=kubeadm
```

   
jx命令将负责提取所有必要的镜像并部署配置的集群。  
 

## 部署到谷歌云

假设您要将集群部署到Google Cloud Platform， Jenkins X也可以实现。在执行此操作之前，你必须首先安装gcloud应用程序。为此，请返回到终端窗口，并使用以下命令下载源文件：  
 

```
wget https://dl.google.com/dl/cloudsdk/channels/rapid/downloads/google-cloud-sdk-265.0.0-linux-x86_64.tar.gz
```

   
使用以下命令解压文件：  
 

```
tar -zxf google-cloud-sdk-*
```

   
使用以下命令进入新创建的目录：  
 

```
cd google-cloud-sdk
```

   
最后，使用命令运行安装程序：  
 

```
./install.sh
```

   
处理完之后，请使用以下命令更新gcloud的所有内容：  
 

```
gcloud components update
```

   
最后，你必须使用以下命令登录到你的Google Cloud Platform帐户：  
 

```
gcloud auth login
```

   
复制链接到浏览器，选择要使用的谷歌账户，然后复制获得的验证码，将其粘贴到命令提示符下，按Enter键。现在，你已经登录Google Cloud Plartform账户，可以发出以下命令：  
 

```
jx create cluster gke --skip-login
```

   
出现提示时，确保选择要使用的Google Cloud Project：

![超详细实操教程！在现有K8S集群上安装JenkinsX，极速提升CI/CD体验！](https://www.pianyun.com/news/content/Uploads/2022-12-27/46291.jpg)

 做出选择并按下Enter键后，系统会提示您选择一个区域：

 ![超详细实操教程！在现有K8S集群上安装JenkinsX，极速提升CI/CD体验！](https://www.pianyun.com/news/content/Uploads/2022-12-27/46292.jpg)

 接着，将问你Jenkins的安装类型（在有Tekton的Serverless Jenkins X 流水线或有Jenkinsfikes的Static Jenks Server中选择）。请注意，使用tekton时，仅支持kaniko作为构建器。

然后，你需要输入名称和邮箱地址以用于git，然后为你的Github账户获取必要的API密钥。之后，集群将部署并可以为你工作。

这就是在现有Kubernetes集群上安装和使用Jenkins X的要旨。这一工具还有许多其他功能，强烈建议你阅读官方文档：

 https://jenkins-x.io/docs/