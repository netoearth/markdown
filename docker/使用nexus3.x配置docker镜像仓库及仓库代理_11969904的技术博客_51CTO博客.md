## 背景

我们一直使用  [harbor](https://github.com/vmware/harbor) 作为docker的镜像仓库，但Harbor只能作为私有仓库，当需要 [Docker Hub](https://hub.docker.com/) 或  [Google Cloud Containers](https://console.cloud.google.com/gcr/images/google-containers/GLOBAL) 上的镜像时，我们只能自己手动pull，重新打tag，再push到harbor上。

当需要拉取多个镜像时，这样相当麻烦，尤其是我们使用`Kubespray`来部署`Kubernetes`集群，仅仅准备镜像就需要花费很多时间。

我们希望有一个Docker仓库，能同时托管私有镜像，还能代理访问公共的镜像仓库。

正好我们在使用`Nexus`作为Maven的仓库，同时`nexus3`提供了`Docker`, `yum`, `apt`, `npm`, `ruby gems`, `pypi` 等诸多类型的仓库功能。

经过技术调研，`Nexus3`完全可以达到我们的预期。

Nexus3 提供了的3种类型的Docker仓库，前两者都可以创建多个仓库，最后一个则可以将他们全部聚合到一个URL来访问。

-   docker (hosted): 自托管
-   docker (proxy): 代理
-   docker (group): 聚合

## 配置Docker仓库

## 运行一个`nexus3`服务器

```
curl -SLO https://sonatype-download.global.ssl.fastly.net/repository/repositoryManager/3/nexus-3.12.1-01-unix.tar.gztar -C /data/nexus3 -xf nexus-3.12.1-01-unix.tar.gzexport RUN_AS_USER=root/data/nexus3/exus-3.12.1-01/bin/nexus start1.2.3.4.5.
```

默认端口是`8081`, 用户名/密码：`admin/admin123`

登录nexus3，点击`设置`齿轮，并定位到左侧的Repository：

## 创建一个`hosted`类型的docker仓库

Hosted类型仓库用作我们的私有仓库，替代harbor的功能。

-   创建专用的`blob`为了仓库数据的独立性和安全性，我们可以给每一个repository创建一个独立的`Blob`块存储。  
    点击`Repository`下面的 `Blob Stores` - `Create blob store`:

-   **Name**：填写一个易于辨认的名字

-   **Path**：会自动生成并补全。默认在Nexus安装目录下面的`sonatype-work/nexus3/blobs/`下，也可以修改到其它目录或磁盘，甚至可以是NFS或者cephfs的目录。

-   创建`hosted`类型的私有仓库  
    点击`Repository`下面的 `Repositories` - `Create repository` - `docker(hosted)` :

-   **Name**: 输入一个简洁直观的名字
-   **Online**: 勾选。这个开关可以设置这个Docker repo是在线还是离线。
-   **Repository Connectors**下面包含`HTTP`和`HTTPS`两种类型的port。  
    有什么用呢？说明讲得很清楚：  
    连接器允许docker客户端直接连接到docker仓库，并实现一些请求操作，如`docker pull`, `docker push`, API查询等。但这个连接器不是必需的，尤其是我们后面会用`group`类型的docker仓库来聚合它。  
    因此，这里我们不勾选并且不填写端口，Nexus就不会启动一个监听到某个端口的连接器。  
    当然，如果你希望直接访问该仓库，你可以填写一个端口如：8090，然后在`daemon.json`中设置  
    `"insecure-registries": [172.xx.xxx.xxx:8090"]`, 重启docker后，通过如下命令访问：  
    `docker push 172.xx.xxx.xxx:8090/centos:7.5.1804`**Force basic authentication**  
    勾选。这样的话就不允许匿名访问了，执行`docker pull`或 `docker push`之前，都要先登录：`docker login`
-   **Docker Registry API Support**Docker registry默认使用的是`API v2`, 但是为了兼容性，我们可以勾选启用`API v1`。
-   **Storage**Blob store，我们下拉选择前面创建好的专用blob：`blob-docker-private`
-   **Hosted**开发环境，我们运行重复发布，因此Delpoyment policy 我们选择`Allow redeploy`。

-   截图展示

![使用nexus3.x配置docker镜像仓库及仓库代理_代理服务器](https://s2.51cto.com/images/blog/202207/28150456_62e235180fa1b1236.jpg?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184 "image")

## 创建一个`proxy`类型的docker仓库

proxy类型仓库，可以帮助我们访问不能直接到达的网络，如另一个私有仓库，或者国外的公共仓库，如Google cloud registry。

对于代理`Docker hub`, 官方有简要的文档可以参考，就不再细写，请参考 [Proxy Repository for Docker](https://help.sonatype.com/repomanager3/private-registry-for-docker/proxy-repository-for-docker)

下面以创建一个`Google cloud registry`的代理为例，简要写一下如何创建`proxy`类型的docker仓库。

-   创建一个专用的`blob`
-   创建一个`proxy`类型的仓库
-   **Name**:`proxy-google-containers`
-   **Repository Connectors**: 不设置。
-   **Proxy**

-   **Remote Storage**: Google cloud registry的地址：`https://gcr.io`配置docker hub的proxy时，这里填写:`https://registry-1.docker.io`
-   **Docker Index**:`Use proxy registry(specified above)`配置docker hub的Docker Index时，点选`Use Docker Hub`或者填写：`https://index.docker.io/`

-   **Storage**:`blob-google-containers`

-   截图

## 创建一个`group`类型的docker仓库

group类型的docker仓库，是一个聚合类型的仓库。它可以将前面我们创建的3个仓库聚合成一个URL对外提供服务，可以屏蔽后端的差异性，实现类似透明代理的功能。

Group类型创建过程类似于上面的proxy类型。

-   名字比较简单：registry
-   启用了一个监听在`80`端口的`http`连接器；
-   **Storage**：选择专用的blob存储`blob-docker-group`；
-   **group**: 将左边可选的3个仓库，添加到右边的`members`下；

-   截图

![使用nexus3.x配置docker镜像仓库及仓库代理_私有仓库_02](https://s2.51cto.com/images/blog/202207/28150506_62e2352296e4135533.jpg?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184 "image")

## 如何使用仓库

-   配置代理服务器根据我们上面配置，我们还无法直接使用仓库，我们还得配置一个代理服务器。  
    在`Setting` - `System` - `HTTP` 下，设置一个代理服务器。
-   代理服务器我们使用 [V2Ray](https://github.com/v2ray)搭建，这里不详述。
-   使用仓库  
    根据以上配置，这个仓库就有一个可以使用的URL了,可以使用下面的命令直接pull上面push的镜像：

```
docker pull 172.xx.xxx.xxx/centos:7.5.1804 1.
```

以上行为会优先从本地仓库（docker-private）查找，没有再从代理仓库(docker-hub 和 google-containers)查找。

我们再来pull一个 google-containers的镜像看看：

```
docker pull 172.xx.xxx.xxx/google-containers/kubernetes-dashboard-amd64:v1.8.31.
```

当然，因为我们的仓库禁用了匿名用户访问，你需要先登录`docker registry`:

```
docker login 172.xx.xxx.xxx -u admin -p xxxxxxxxxx1.
```

成功了。

同理，我们可以拉取docker hub上的镜像，只要把url中的域名，替换成我们自己的域名就可以了。

最后我们看下镜像仓库的存储结构：

可以看到，Docker Hub的官方仓库镜像，被放到了`libray` 目录下，GCR的则新建了一个`google-containers` 的目录，而其他的用户的镜像，则会被放在用户名同名的目录下，目录名与远程目录一致。

值得一提的是，我们可以使用这个仓库从Docker Hub和 Google Cloud Registry下载镜像，但是我们不能通过这个仓库推送镜像到远程仓库。我们推送的所有镜像都会被存储在私有仓库内。