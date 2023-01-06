 

kubekey 2.0 的一个重大的功能，就是实现离线安装。目前已经到了rc3的阶段，这里就整理一下我的理解。

-   https://blog.k8s.li/pass-tob-k8s-offline-deploy.html

上面这篇文章，其实非常经典，介绍离线安装的关键点，对着来看kubekey实现的功能，就可以更好理解。

-   系统 OS 的 rpm/deb 包：ipvsadm、conntrack 等；
-   如 kubelet、kubectl、kubeadm、crictl ,docker-ce, containerd
-   组件容器镜像：如 kube-apiserver、kube-proxy、coredns、calico、flannel 等；

我们就来看看kubekey如何解决上面的3个问题。

## 操作系统依赖

其实就是如何把依赖的包，搞成一个repo，对于kubekey来说，他是通过制作一个iso，挂在到节点上，添加临时的repo，解决k8s的安装，对系统的依赖问题。

Docker和contained，并不是才有repo的方式安装，所以对于kubekey来说，repo，只需要解决系统的k8s的依赖就可以。

-   https://github.com/kubesphere/kubekey/blob/master/docs/manifest-example.md

```
spec:
  arches: 
  - amd64
  operatingSystems: 
  - arch: amd64
    type: linux
    id: ubuntu
    version: "20.04"
    osImage: Ubuntu 20.04.3 LTS
    repository: 
      iso:
        localPath: 
        url: https://github.com/pixiake/k8s-dependencies/releases/download/v1.0.0/ubuntu-20.04-amd64-debs.iso
  - arch: amd64
    type: linux
    id: centos
    version: "7"
    osImage: CentOS Linux 7 (Core)
    repository:
      iso:
        localPath:
        url: https://github.com/pixiake/k8s-dependencies/releases/download/v1.0.0/centos-7-amd64-rpms.iso
```

后续kubekey会提供相应的工具，来制作这个iso，原理其实和木子文章介绍应该是一样的。

## kubelet 等二进制

k8s的二进制包，kubekey 1.21，已经解决了这个问题，默认会做kk的目录下，创建一个kubekey的目录，目录下，如何已经存放好相应的二进制包，他就直接使用，如果没有，kk就会下载。通过ansbile 分发到各个节点的相应目录下。

所以kk会根据你需要k8s版本，下载相应的

-   kubeadm
-   kubelet
-   kubectl
-   cni

还有相关的

-   docker or contained
-   crictl

当然也会包括docker 相应的mirror，镜像仓库的设置。

## 组件容器镜像

这个是最复杂，涉及到镜像仓库，证书，镜像。要做的工作比较多。对于kubekey来说，

-   初始化一个registry 或者harbor
-   下载需要的镜像，导入到镜像仓库
-   解决证书问题，所有的节点请求私有仓库，拉取镜像。

目前kubekey 2.0 ，已经基本都实现了。可以很流畅的把k8s和kubesphere都实现离线安装。