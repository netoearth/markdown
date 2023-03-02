## 使用vagrant搭建k8s集群--创建虚拟机环境

[![](https://cdn2.jianshu.io/assets/default_avatar/6-fd30f34c8641f6f32f5494df5d6b8f3c.jpg)](https://www.jianshu.com/u/accede74be6f)

0.0872020.04.25 21:27:49字数 1,241阅读 465

## 创建虚拟机群

安装部署virtualbox、vagrant，安装完成后，准备创建虚拟机群，本次搭建规划为`1个master + 2个slave + 1个docker-register`的模式。

### 虚拟机配置参数

| hostname | IP | CPU | 内存 |
| --- | --- | --- | --- |
| master | 10.110.0.11 | 2 | 2G |
| slave-1 | 10.110.0.12 | 1 | 2G |
| slave-2 | 10.110.0.13 | 1 | 2G |
| docker-register | 10.110.0.14 | 1 | 2G |

虚拟机群规划的资源完成后，我们在宿主机的目录下创建一个Vagrantfile的文件，这个文件就是创建虚拟机群组的配置文件。

### 编写配置文件

这里我们一步步编写Vagrantfile文件，以便于更好的理解该配置文件的意义以及作为参考：

```
Vagrant.configure("2") do |config|
  config.vm.define "master" do |master|
  
  end
  config.vm.define "worker-1" do |worker_1|
  
  end
  config.vm.define "worker-2" do |worker_2|
  
  end
  config.vm.define "docker-register" do |docker-register|
  
  end
end
```

-   `Vagrant.configure("2")`是一个固定格式，不需要修改，其含义可以查看vagrant的文档获取；
-   `config.vm.define "master"`定义了一个虚拟机，为master；

配置虚拟机的参数都是大同小异，这里只列出`master`这台虚拟机的配置信息，其他的虚拟机的配置参数可以根据前面规划的内容进行修改或根据自己实际情况做出修改，master的基本配置信息如下：

```
Vagrant.configure("2") do |config|
  config.vm.define "master" do |master|
    master.vm.box = "centos/7"
    master.vm.hostname = "master"
    master.vm.network "private_network", ip: "10.110.0.11"
    master.vm.provider "virtualbox" do |v|
      v.memory = 2048
      v.cpus = 2
    end
  end
end
```

-   `master.vm.box = "centos/7"`定义了虚拟机采用的镜像文件是`centos/7`。执行`vagrant up`时，vagrant会根据Vagrantfile的内容，查看本地环境是否存在对应镜像，如果不存在则从vagrant的官方仓库中下载。还可以使用以下两种方式获取vagrant的镜像文件：
    
    -   使用`vagrant init centos/7`的方式下载镜像到本地；
    -   手动下载镜像，使用`vagrant add box <image_file> --name <local_image_name>`将下载的文件导入到本地的box列表中；
    
    之后，可以使用`vagrant box list`查看本地环境的image列表。
    
-   `master.vm.hostname`和`master.vm.network`是为虚拟机配置了hostname和私有网络，关于public\_network的配置，后续有相关的说明；
    
-   `master.vm.provider "virtualbox"`该项配置则声明了vagrant使用的虚拟化后端是virtualbox，并且后续的配置项对虚拟机使用的计算资源做出了配置。
    

最后，我们为虚拟机配置public\_network，以便于外网访问虚拟机：

```
Vagrant.configure("2") do |config|
  config.vm.network "public_network", bridge: "eth0"

```

-   `config.vm.network "public_network", bridge: "eth0"`该配置与private\_network的配置类似，区别在与网络模式这里选择`bridge`，参数则是宿主机上与外界连通的网卡的名称。

### 配置虚拟机支持root登录

一般来说，linux环境下的ssh都不支持root用户直接访问，但是，在后续创建集群的过程中，有许多的操作需要root权限才能执行，这样一来，直接使用root用户登录虚拟机对我们来说是十分方便的，这一步，我们在Vagrantfile文件中增加相应的配置，使得虚拟机在创建完成后既可以支持root用户通过ssh链接：

```
$sshd_change = <<-SCRIPT
echo change sshd_config to allow public key authentication & relaod sshd...
sed -i 's/\#PermitRootLogin yes/PermitRootLogin yes/' /etc/ssh/sshd_config
sed -i 's/\#PubkeyAuthentication yes/PubkeyAuthentication yes/' /etc/ssh/sshd_config
sed -i 's/PasswordAuthentication no/PasswordAuthentication yes/' /etc/ssh/sshd_config
systemctl reload sshd
SCRIPT

Vagrant.configure("2") do |config|
  config.vm.network "public_network", bridge: "eth0"

  config.vm.define "master" do |master|
    master.vm.box = "centos/7"
    master.vm.hostname = "k8s-master-01"
    master.vm.network "private_network", ip: "10.110.0.11"
    master.vm.provision "shell", inline: $sshd_change
    master.vm.provider "virtualbox" do |v|
      v.memory = 2048
      v.cpus = 2
    end
  end
```

-   `sshd_change`是一个我们编写的脚本名称，脚本内容就是`SCRIPT`标签中的内容；
-   `master.vm.provision "shell", inline: $sshd_change`这一项配置表明创建虚拟机的过程中，修改了sshd\_config文件中的信息，修改的内容就是脚本中描述的信息。

完成整个Vagrantfile的编写后，保存该文件，在该文件存在的路径执行`vagrant up`命令，开始创建虚拟机群。创建过程中没有错误后，等待一段时间，终端没有输出信息后，则说明虚拟机群创建完成了，使用`vagrant status`命令查看虚拟机状态及数量是否与期望的一致。

## 配置宿主免密登录虚拟机

创建好的虚拟机，可以使用用户名vagrant和密码vagrant通过ssh链接。但是，为了方便我们可以对虚拟机进行免密登录配置，配置方式和一般linux配置免密登录的方式一样，即将宿主机的ssh-id复制到各个虚拟机上，使用命令`ssh-copy-id vagrant@<ip>`即可。也可以将虚拟机的hostname和IP映射，写入到宿主机的hosts文件中，即将下列内容追加到/etc/hosts文件中：

```
10.110.0.11 master
10.110.0.12 slave-1
10.110.0.13 slave-2
10.110.0.14 docker-register
```

在终端中输入`ssh-copy-id root@master`就可以将宿主机的`ssh_key`复制到对应的虚拟机master上，成功后就可以冲宿主机免密登录到虚拟机上。

## 结束

创建完虚拟机群之后，一个良好的习惯就是，为各个虚拟机创建快照，以便于后期在部署K8S集群的过程中，出现异常后，快速的恢复虚拟机，使用`vagrant snapshot save master k8s-master`这种格式，就可以快速的创建虚拟机快照了。

更多精彩内容，就在简书APP

"小礼物走一走，来简书关注我"

还没有人赞赏，支持一下

[![  ](https://cdn2.jianshu.io/assets/default_avatar/6-fd30f34c8641f6f32f5494df5d6b8f3c.jpg)](https://www.jianshu.com/u/accede74be6f)

[LineWay](https://www.jianshu.com/u/accede74be6f "LineWay")懂一点Python，懂一点Docker，还懂一点Go.

总资产22共写了2838字获得8个赞共7个粉丝

### 推荐阅读[更多精彩内容](https://www.jianshu.com/)

-   一、Vagrant 介绍 Vagrant 是一个软件，可以自动化虚拟机的安装和配置流程，用来管理虚拟机，如 Vir...
    
    [![](https://upload-images.jianshu.io/upload_images/13236273-91f44d6e82171410.png?imageMogr2/auto-orient/strip|imageView2/1/w/300/h/240/format/webp)](https://www.jianshu.com/p/31b2a6ce0924)
-   开发需要在各种系统上进行开发任务，运维则需要在各种系统上学习工具使用。因此，虚拟机恐怕也是 IT 人员最常使用的工...
    
    [![](https://cdn2.jianshu.io/assets/default_avatar/7-0993d41a595d6ab6ef17b19496eb2f21.jpg)李广慧](https://www.jianshu.com/u/XRLrHk)阅读 5,219评论 3赞 24
    
    [![](https://upload-images.jianshu.io/upload_images/817-b31c4f3331547569.png?imageMogr2/auto-orient/strip|imageView2/1/w/300/h/240/format/webp)](https://www.jianshu.com/p/a87a37d73202)
-   1\. Vagrant 的介绍 虚拟开发环境 平常我们经常会遇到这样的问题：在开发机上面开发完毕程序，放到正式环境之...
    
    [![](https://cdn2.jianshu.io/assets/default_avatar/1-04bbeead395d74921af6a4e8214b4f61.jpg)斐波那契额](https://www.jianshu.com/u/80ea0a768c26)阅读 1,521评论 1赞 12
    
    [![](https://upload-images.jianshu.io/upload_images/4371731-88069b256f33728f.png?imageMogr2/auto-orient/strip|imageView2/1/w/300/h/240/format/webp)](https://www.jianshu.com/p/75e6da2d5fb9)
-   1.Vagrant Vagrant是一个虚拟机管理和配置工具，所以虚拟机系统还得靠专门的虚拟化软件，Vagrant...
    
    [![](https://upload.jianshu.io/users/upload_avatars/12842279/a84a22d5-268e-49e9-b753-643097ff324e.jpg?imageMogr2/auto-orient/strip|imageView2/1/w/48/h/48/format/webp)中v中](https://www.jianshu.com/u/532767262a94)阅读 7,564评论 0赞 2
    
    [![](https://upload-images.jianshu.io/upload_images/12842279-2f6f83de1315e745.png?imageMogr2/auto-orient/strip|imageView2/1/w/300/h/240/format/webp)](https://www.jianshu.com/p/8d22b5c51984)
-   更新了，更新了...... 经过郑少的打磨，今天跟大家分享几个不错的美化案例。 让你在设计年终总结的时候有一些灵感...
    
    [![](https://upload.jianshu.io/users/upload_avatars/4786780/98edffff-a345-4885-9a72-6a03204af031.jpg?imageMogr2/auto-orient/strip|imageView2/1/w/48/h/48/format/webp)郑少PPT](https://www.jianshu.com/u/47b9764d2127)阅读 4,124评论 9赞 84
    
    [![](https://upload-images.jianshu.io/upload_images/4786780-d3730278d1306acc.png?imageMogr2/auto-orient/strip|imageView2/1/w/300/h/240/format/webp)](https://www.jianshu.com/p/ca8e0d7138d7)
-   飞雪夜，靠床头。品阅君书，妾心惆怅失梦愁。北南相去万千州。为生活奋进，好温柔。 用韵: （七尤；平） 头，愁，州，柔。
    
    [![](https://upload.jianshu.io/users/upload_avatars/3773289/ad2e8876a610.jpg?imageMogr2/auto-orient/strip|imageView2/1/w/48/h/48/format/webp)润泽杂夥](https://www.jianshu.com/u/e1c6270303a6)阅读 220评论 2赞 10
    
    [![](https://upload-images.jianshu.io/upload_images/3773289-2c5dee7ca3e8d041.jpg?imageMogr2/auto-orient/strip|imageView2/1/w/300/h/240/format/webp)](https://www.jianshu.com/p/6b3cafd3d5a4)
-   回到魔都，恢复上班的前三个月，一切相安无事，我跟雅薰还是依然保持工作关系，只是增添了几分陌生。 我只能沉默地继续努...
    
    [![](https://upload.jianshu.io/users/upload_avatars/593218/06bf27b9-6fd9-4a73-a925-2b034108e0d4.jpg?imageMogr2/auto-orient/strip|imageView2/1/w/48/h/48/format/webp)代码胖](https://www.jianshu.com/u/f156f305df50)阅读 422评论 0赞 1
    
-   1.【太7:6】不要把圣物给狗，也不要把你们的珍珠丢在猪前，恐怕它践踏了珍珠，转过来咬你们。  2.不要论断...
    
    [![](https://cdn2.jianshu.io/assets/default_avatar/8-a356878e44b45ab268a3b0bbaaadeeb7.jpg)胡涂格格](https://www.jianshu.com/u/899198d570fc)阅读 252评论 0赞 0
    
-   你笑起来的样子像三月的太阳暖和于是漫山的花盛开 你笑起来的样子像六月的星空澄澈于是霓虹的闹抛却 你笑起来的样子像九...
    
    [![](https://upload.jianshu.io/users/upload_avatars/11119469/a41dc409-efd9-45b7-ab7b-4f3eec17d4bf.jpg?imageMogr2/auto-orient/strip|imageView2/1/w/48/h/48/format/webp)黄南北](https://www.jianshu.com/u/669bc51f5954)阅读 832评论 11赞 41