Docker本质上就是虚拟化方法的一种，是一个开源的应用容器引擎，让开发者可以打包他们的应用以及依赖包到一个可移植的容器中，然后发布到任何Linux机器上，当然也可以基于Linux实现虚拟化，容器是完全使用沙箱机制，相互之间不会有任何接口。

docker支持随机创建、销毁，当任务需要创建一台实例时，只需要在设备池中虚拟化出一台资源机，挂载私有网络，便完成了实例创建。当然，服务器除了硬件资源，还需要软件资源，包含操作系统、容器、中间件等配合，因此，实例化docker之后，我们还得创建操作系统、安装我们的容器配置。

![](https://obs-emcsapp-public.obs.cn-north-4.myhwclouds.com/wechatSpider/modb_20200715_103043.png)![](https://obs-emcsapp-public.obs.cn-north-4.myhwclouds.com/wechatSpider/modb_20200715_103044.png)

那么，能否把这些所需要的内容进行统一打包，然后顺势“解压”到服务器上，直接完成基本环境部署，这就需要镜像文件的协助。

镜像

我们回想一下winXP,win7盛行的年代，当年“雨林木风”大火的年代，凡是安装过操作系统的兄弟们，都或多或少听说过ghost，那个蓝色，旁边有个小精灵的框框。我们将gho文件预先存放在磁盘里，或者光盘中，甚至U盘启动盘中，蓝色框框出现后，选中gho文件，对其进行还原。待安装完成后，我们的操作系统就安装好了，当然，驱动什么的，需要自行解决（如果是个人电脑备份的gho便可以任他蓝屏都不怕，当然，前提是资料不要在系统盘）。  

![](https://obs-emcsapp-public.obs.cn-north-4.myhwclouds.com/wechatSpider/modb_20200715_103045.png)

ghost这种方式，只是安装OS系统方式其中的一种，还有一种ISO文件，可以双击打开，这种更为方便，打开后，按步骤执行，会覆盖系统盘的全部内容。从而替换当前操作系统。这些，都是镜像文件的体现方式。

镜像（Mirroring）是冗余的一种类型，一个磁盘上的数据在另一个磁盘上存在一个完全相同的副本即为镜像。  

镜像是一种文件存储形式，可以把许多文件做成一个镜像文件，与GHOST等程序放在一个盘里用GHOST等软件打开后，又恢复成许多文件，RAID1和RAID10使用的就是镜像。常见的镜像文件格式有ISO、BIN、IMG、TAO、DAO、CIF、FCD。

现在云计算中，镜像使用更为广泛。配合虚拟资源生成，镜像集成了操作系统、nginx，tomcat，jenkins，sdk等。按照不同设备功能，对服务器进行不同版本软件集成。

正是因为有了镜像文件，才大大减少了虚拟服务器创建的工作量。才有了如今，各大公有云厂商的一键生成服务器的操作。

上面说了这么多，相信大家都知道今天本作者的主题是什么了，那就是镜像仓库选型比较。

DockerRepository介绍

DockerRepository为Docker仓库，是集中存放镜像的地方。  

DockerRegistry是Docker仓库的注册服务器，实际上Docker仓库镜像的管理，分发，用户认证都是通过Registry来完成，就是所谓的Docker仓库管理工具。

Docker官方为用户提供了一个公共仓库DockerHub，大部分需求镜像都可以通过在Docker Hub中直接下载镜像来实现。但面对这镜像使用的方便，安全问题及网络延迟等原因，Docker也满足自建私有仓库管理的需求，提供的RegistryDocker私有仓库镜像，同时也有一些不错的第三方Docker私有仓库管理工具涌现而出。

Docker私有仓库管理工具

Registry

Dockerregistry作为Docker官方为使用者提供的一个私有仓库管理，可以用于在本地管理我们自己的镜像。其拉取镜像方式的便利有了较大的提高，安全性问题也由网络转向本地，增强了可控性。

测试配置

Dockerregistry部署方式简单，Dockerhub上有提供官方镜像，直接可部署为Docker容器。

![](https://obs-emcsapp-public.obs.cn-north-4.myhwclouds.com/wechatSpider/modb_20200715_103046.png)上图为部署测试使用nginx镜像上传到registry私有仓库中，其展示方式使用URL在浏览器中打开，也可在Linux服务器上使用curl命令直接访问URL。其不同的展示样式需要更改URL，并且没有直接管理Docker镜像的工具，同样需要curl命令修改URL的参数来完成，操作不方便。

Docker-Registry-Web

此工具为DockerHub上第三方Docker私有仓库WebUI镜像，搜索名称为：hyper/Docker-registry-web。其包含身份验证服务及事件记录功能，版本基于Dockerregistry v2。

测试配置

部署方式也是基于Docker镜像来完成，该工具展示方式为Web界面。后端私有仓库有DockerRegistry支持。其主要功能就是提供可查看及管理的WebUI。

![](https://obs-emcsapp-public.obs.cn-north-4.myhwclouds.com/wechatSpider/modb_20200715_103047.png)

上图所示为WebRegistry打开界面，可以清楚看到本地私有仓库地址，及仓库中已有的镜像。

![](https://obs-emcsapp-public.obs.cn-north-4.myhwclouds.com/wechatSpider/modb_20200715_103048.png)

上图为点击详镜像名称，进而可查看镜像的详细信息。

注：测试Docker-Registry-Web配置为非认证方式，需要在源端服务器上配置Docker允许私有仓库地址。

Docker-Registry-Web除支持镜像展示基本功能外，还可支持镜像删除管理，及身份验证服务等，身份验证服务包括basic验证及证书验证等方式。其功能配置方式不做一一展示请参考官网信息。

参考：https://hub.Docker.com/r/hyper/Docker-registry-web

Nexus

Nexus号称是世界上最流行的仓库管理软件(Theworld's most popular repository),Nuexu3能够支持Maven、npm、Docker、YUM、Helm等格式数据的存储和发布；并且能够与Jekins、SonaQube和Eclipse等工具进行集成。

Nexus3支持作为宿主和代理存储库的Docker存储库，可以直接将这些存储库暴露给客户端工具；也可以以存储库组的方式暴露给客户端工具，存储库组是合并了多个存储库的内容的存储库，能够通过一个URL将多个存储库暴露给客户端工具，从而便于用户的使用。通过Nexus3自建能够有效减少访问获取镜像的时间和对带宽的使用，并能够通过自有的镜像仓库共享企业自己的镜像。

Nexus有两个版本Nexus RepositoryOSS和NexusRepositoryPro,其中OSS版本是免费,Pro专业版需要收费.OSS对于基础的仓库管理已经足够使用。本文中使用的Nexus展示是基于OSS版本。

![](https://obs-emcsapp-public.obs.cn-north-4.myhwclouds.com/wechatSpider/modb_20200715_103049.png)

测试配置

Nexus登录使用默认密码（admin/admin123）,登录权限为管理员。登录之后才可以看到配置按钮（齿轮按钮），获取修改配置的权限。

Nexus中配置Docker私有仓库有三种类型：

-   hosted(本地类型)镜像资源的提交和拉取都基于本地存储；
    
-   proxy(代理类型)本地不做数据存储,可以和hosted配合；
    
-   group(组合类型)可以组合多个hosted和proxy按顺序提供统一访问地址;
    

注：本文测试Nexus版本：NexusOSS 3.24.0-02

![](https://obs-emcsapp-public.obs.cn-north-4.myhwclouds.com/wechatSpider/modb_20200715_103051.png)

如上图所示，opensource-v1为创建的Docker仓库项目（基于hosted类型），在此基础上我们对该项目进行配置管理，主要涉及到仓库端口，api版本允许，匿名提交权限，仓库存储位置，仓库权限等。其中还涉及到规则配置，是Nexus安全性管理中重要的一项。

![](https://obs-emcsapp-public.obs.cn-north-4.myhwclouds.com/wechatSpider/modb_20200715_103052.png)

如上图中展示，Nginx为测试push提交到Nexus上的镜像，测试方式在服务器上对镜像打tag标识，登录已配置的Nexus仓库，然后push提交。

注：该Nexus仓库配置为http方式，需要在源端服务器上配置Docker允许私有仓库地址。

在围绕仓库管理的为中心的理念里，Nexus又加入的重要的仓库安全管理模块，及系统管理工具模块。如下图中展示：

![](https://obs-emcsapp-public.obs.cn-north-4.myhwclouds.com/wechatSpider/modb_20200715_103053.png)

![](https://obs-emcsapp-public.obs.cn-north-4.myhwclouds.com/wechatSpider/modb_20200715_103054.png)

![](https://obs-emcsapp-public.obs.cn-north-4.myhwclouds.com/wechatSpider/modb_20200715_103055.png)

上图描述：Nexus支持工具中日志管理功能。

![](https://obs-emcsapp-public.obs.cn-north-4.myhwclouds.com/wechatSpider/modb_20200715_103056.png)

上图描述：Nexus支持工具中服务资源监控功能。

Nexus出色的界面化管理，及强大仓库管理功能，全面的安全管理配置，简洁高效的系统管理工具，让Nexus成为一款优秀的仓库管理软件，并且其开源版本也得到更多用户的支持与信赖。

Harbor

Harbor是一个用于存储和分发Docker镜像的企业级Registry服务器，通过添加一些企业必需的功能特性，例如安全、标识和管理等，扩展了开源DockerDistribution。作为一个企业级私有Registry服务器，Harbor提供了更好的性能和安全。提升用户使用Registry构建和运行环境传输镜像的效率。

Harbor的每个组件都是以Docker容器的形式构建的，可以使用DockerCompose来进行部署，当然，如果你的环境中使用了Kubernetes，Harbor也提供了Kubernetes的配置文件。

Harbor共由8个容器组成：

-   ui：Harbor的核心服务;
    
-   log：运行着Rsyslog的容器，进行日志收集;
    
-   mysql：由官方MySQL镜像构成的数据库容器;
    
-   nginx：使用Nginx做反向代理;
    
-   registry：官方的Docker Registry;
    
-   adminserver：Harbor的配置数据管理器;
    
-   jobservice：Harbor的任务管理服务;
    
-   redis：用于存储Session;
    

注：本文中测试Harbor版本：v1.10.3-6990ccaa

测试配置

![](https://obs-emcsapp-public.obs.cn-north-4.myhwclouds.com/wechatSpider/modb_20200715_103058.png)

如上图所示为Harbor登录界面，Harbor可以使用默认登录用户密码（admin/Harbor12345）。

Harbor页面中，主要包括项目管理，日志查看，及系统管理；其中主要涉及到Docker仓库管理的配置在系统管理项中。

![](https://obs-emcsapp-public.obs.cn-north-4.myhwclouds.com/wechatSpider/modb_20200715_103059.png)

如上图中所示opensourcev2为测试新建的仓库项目（仓库分支），可以配置仓库访问权限，角色管理等。

![](https://obs-emcsapp-public.obs.cn-north-4.myhwclouds.com/wechatSpider/modb_20200715_103100.png)

如上图所示，memcache镜像为测试上传push到Harbor仓库中的展示，其页面内容描述了当前镜像的详细信息，镜像具有复制，添加标签，tag复制及删除功能。

值得一提的是Harbor的仓库管理模块和复制管理模块。仓库管理中可以添加其他仓库源中的镜像，利用复制管理模块可以将其已添加到仓库管理中的镜像复制到Harbor仓库中。  

如下图展示（图表1/2/3）：

![](https://obs-emcsapp-public.obs.cn-north-4.myhwclouds.com/wechatSpider/modb_20200715_103101.png)

图表1![](https://obs-emcsapp-public.obs.cn-north-4.myhwclouds.com/wechatSpider/modb_20200715_103102.png)

图表2![](https://obs-emcsapp-public.obs.cn-north-4.myhwclouds.com/wechatSpider/modb_20200715_103103.png)

图表3

Harbor系统管理中，还有其他功能可选，例如自我注册，项目定额，审查服务，垃圾清理，配置管理，中文支持等。Harbor作为专业的Docker仓库管理工具，致力于企业级Registry服务器发展，其完善的配置，独特的功能，良好的界面展示，及友好的中文支持都受到广大用户的支持与青睐。  

详细功能对比

![](https://obs-emcsapp-public.obs.cn-north-4.myhwclouds.com/wechatSpider/modb_20200715_103104.png)

总结

通过对这几款私有仓库管理的认识，可以明确看到不同工具之间的功能对比，Registry虽然功能少，但确是官方最早发行支持的私有仓库。像Nexus3,和Harbor是具有强烈的专业性，以其丰富的功能设计来赢得用户的认可。行业内还有其他仓库管理工具流行，本文只采集了这四种来测试对比。希望通过此文章中对不同私有仓库管理工具的认知，结合自身实际情况，现场实际环境，选择适合场景的仓库管理工具。

文中参考引用信息：https://www.jianshu.com/p/c6223b292b01