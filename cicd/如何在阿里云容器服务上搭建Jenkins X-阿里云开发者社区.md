"Jenkins X is a CI/CD solution for modern cloud applications on Kubernetes." 这是Jenkins社区对于[Jenkins X](https://jenkins-x.io/) 的官方总结和定义。显而易见，它是一套以Jenkins作为核心发动机，以GitOps作为方法论，集成了nexus, docker-registry 和chartmuseum 等一系列交付标准存储组件的持续集成和持续交付解决方案。

下面我们讲介绍如何在阿里云容器服务上快速安装Jenkins X。

1.  首先，需要在 [阿里云容器服务控制台](https://cs.console.aliyun.com/) 创建一个香港集群，如果创建的集群只有一个worker节点，建议添加一台配置不低于8C16G的ECS。
2.  进入集群管理页面，找到 “Master 节点 SSH 连接地址”，SSH登录Master。
3.  安装 git。
    
    ```
    yum install git
    ```
    
4.  安装 jx 。

4.1 定制化 env-kubernetes 来实现在阿里云容器服务kubernetes集群上安装jx。创建~/.jx文件目录。然后下载 cloud-environments repo到.jx文件目录。

```
mkdir -p ~/.jx
cd ~/.jx
git clone https://github.com/qinyujia/cloud-environments.git
```

4.2 在ECS上安装jx客户端。

```
curl -L https://github.com/jenkins-x/jx/releases/download/v1.3.83/jx-linux-amd64.tar.gz | tar xzv 
sudo mv jx /usr/local/bin
```

4.3 更新helm stable repo。

```
helm repo remove stable
helm repo add stable https://kubernetes-charts.storage.googleapis.com
helm repo list
```

4.4 在阿里云容器服务kubernetes集群上安装jx server组件。

```
jx install --provider=kubernetes
```

![1](https://yqfile.alicdn.com/0869ecf176b03965dc1cd1d32956212dfe924689.png "1")

1.  在Jenkins系统设置页面关闭证书验证。例如，打开`http://jenkins.jx.47.89.0.138.nip.io/configure` 页面，选中`Disable https certificate check`。  
    ![2](https://yqfile.alicdn.com/90a837ec18f96152d0ec410e9e659622ac132620.png "2")
2.  接下来就可以查看并使用Jenkins X啦。  
    ![3](https://yqfile.alicdn.com/350db2d1f3c67eaffadf5af6db22c57760d10938.png "3")

![4](https://yqfile.alicdn.com/a59d07d567e6ff219c4633fc97a5ac378192f63f.png "4")