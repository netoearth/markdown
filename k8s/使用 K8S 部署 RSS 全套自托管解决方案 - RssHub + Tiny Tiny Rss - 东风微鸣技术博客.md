## 前言[](https://ewhisper.cn/posts/60709/#%E5%89%8D%E8%A8%80%20-5)

RSS 是一种描述和同步网站内容的格式，是使用最广泛的 XML 应用。RSS 搭建了信息迅速传播的一个技术平台，使得每个人都成为潜在的信息提供者。发布一个 RSS 文件后，这个 RSS Feed 中包含的信息就能直接被其他站点调用，而且由于这些数据都是标准的 XML 格式，所以也能在其他的终端和服务中使用，是一种描述和同步网站内容的格式。

RSS 广泛用于网上新闻频道，blog 和 wiki。使用 RSS 订阅能更快地获取信息，网站提供 RSS 输出，有利于让用户获取网站内容的最新更新。网络用户可以在客户端借助于支持 RSS 的聚合工具软件，在不打开网站内容页面的情况下阅读支持 RSS 输出的网站内容。

两个目的：

1.  通过 RSS 和 RSS 阅读器作为高效率的 Feed 集合器，免去每日浏览各个网站搜寻信息的时间，发现优质内容，提高阅读效率。
2.  通过 RSS Feed, 获取低频但重要的信息，如重要软件更新，预报预警等。

[Tiny Tiny RSS](https://tt-rss.org/) 是一款基于 PHP 的免费开源 RSS 聚合阅读器。需要自行托管和部署，为基于网页的 RSS 阅读器。

[RSSHub 主页](https://docs.rsshub.app/)

RSSHub 是一个开源、简单易用、易于扩展的 RSS 生成器，可以给任何奇奇怪怪的内容生成 RSS 订阅源。RSSHub 借助于开源社区的力量快速发展中，目前已适配数百家网站的上千项内容

主要目的：

-   将非 rss 格式转换为 rss 以便订阅；正如其 slogan 所言：「🍰 万物皆可 RSS」
    
-   发现更多有趣的订阅源。
    

比如我通过 RssHub 订阅的内容有：

-   哔哩哔哩
-   InfoQ 热门话题
-   本地宝焦点资讯
-   豆瓣 - 正在上映的高分电影
-   所在城区停电通知
-   N 卡驱动更新
-   …

它是作为一个有聚合了很多内容 RSS 源的仓库来使用。

> 📚️ **Reference**:
> 
> 比如我希望订阅 Twitter 上一个名为 DIYgod 的用户的时间线
> 
> 根据 [Twitter 用户时间线路由](https://docs.rsshub.app/social-media.html#twitter) 的文档，路由为 `/twitter/user/:id`，把 `:id` 替换为用户名，得到路径为 `/twitter/user/DIYgod`，再加上域名 `https://rsshub.app`，一个订阅源就生成了：[https://rsshub.app/twitter/user/DIYgod(opens new window)](https://rsshub.app/twitter/user/DIYgod)
> 
> 然后我们可以把 [https://rsshub.app/twitter/user/DIYgod](https://rsshub.app/twitter/user/DIYgod) 添加到任意 RSS 阅读器（当然也可以是 Tiny Tiny RSS) 里来使用
> 
> 其中域名 `https://rsshub.app` 可以替换为你 [自部署](https://docs.rsshub.app/install/) 的域名

另外，如果需要订阅一些特定的内容，比如：

-   bilibili 用户（我自己）关注的内容
-   微博 个人时间线

等就需要将 RssHub 单独部署并进行配置。

## 部署架构[](https://ewhisper.cn/posts/60709/#%E9%83%A8%E7%BD%B2%E6%9E%B6%E6%9E%84)

### Overview[](https://ewhisper.cn/posts/60709/#Overview)

Tiny Tiny RSS 有一个公网 HTTPS 域名（如：[https://ttrss.ewhisper.cn](https://ttrss.ewhisper.cn/)), 我直接登录该域名来进行 RSS 阅读；

Tiny Tiny RSS 订阅源可以来自：

1.  支持 RSS 的网站，比如：[OpenShift 博客](https://cloud.redhat.com/blog) 的对应 RSS 地址为：[https://cloud.redhat.com/blog/rss.xml](https://cloud.redhat.com/blog/rss.xml)
2.  我自己部署的 RssHub, 公网 HTTPS 域名为：[https://rss.ewhisper.cn](https://rss.ewhisper.cn/)

1.  Tiny Tiny RSS 部署在 K8S 集群的 `rss` ns 里；
2.  基于 [Awesome-TTRSS/docker-compose.yml at main · HenryQW/Awesome-TTRSS (github.com)](https://github.com/HenryQW/Awesome-TTRSS/blob/main/docker-compose.yml), 需要部署的组件有：
    1.  tiny tiny rss, 需要有一个 PVC 存储，用于存放 icon
    2.  tiny tiny rss 的 数据库 - postgresql 13, 需要有一个 PVC 存储，用于存放数据库数据。
3.  组件都是单节点部署，不考虑高可用；
4.  Tiny Tiny RSS 通过 Ingress + SVC 对外发布域名；

1.  RssHub 部署在 K8S 集群的 `rss` ns 里；
2.  基于 [RSSHub/docker-compose.yml at master · DIYgod/RSSHub (github.com)](https://github.com/DIYgod/RSSHub/blob/master/docker-compose.yml) , 需要部署的组件有：
    1.  rsshub
    2.  browserless chrome
    3.  redis, 需要有一个 PVC 存储，用于存放缓存数据。
3.  组件都是单节点部署，不考虑高可用；
4.  RssHub 通过 Ingress + SVC 对外发布域名；

## 前提条件[](https://ewhisper.cn/posts/60709/#%E5%89%8D%E6%8F%90%E6%9D%A1%E4%BB%B6%20-3)

1.  需要有属于自己的域名，如：`ewhisper.cn`, 具体的域名为：
    1.  `ttrss.ewhisper.cn`
    2.  `rss.ewhisper.cn`
2.  且该域名已经在国内备案，80 和 443 端口可以正常使用；
3.  该域名托管在 DNSPod 或类似的 DNS 供应商，可以方便地修改 DNS Record;
4.  需要有对应域名的证书，本次需要有：`ttrss.ewhisper.cn` 和 `rss.ewhisper.cn` 的证书，可以是单域名证书，也可以是泛域名证书。
5.  已经搭建好 K8S 集群
6.  K8S 集群有 Ingress Controller
7.  K8S 集群有 StorageClass 或可以提供 PV 存储。（本文 K8S 集群默认提供 `local-path` storageclass)
8.  本次 2 个域名通过 K8S Traefik 的 IngressRoute 进行配置，配置 Ingress 和 证书；
9.  已安装：[K8S 实用工具之五 -kompose - 东风微鸣技术博客 (ewhisper.cn)](https://ewhisper.cn/posts/35291/), 该工具用于将 `docker-compose.yml` 快速转换为 K8S yaml

> 📚️ **Reference:**
> 
> 可以通过 cert-manager 为 dnspod 在 Letsencrypt 上申请免费证书：
> 
> -   [使用 cert-manager 为 dnspod 的域名签发免费证书 | 作者 roc](https://imroc.cc/k8s/trick/cert-manager-webhook-dnspod/)

## 实施[](https://ewhisper.cn/posts/60709/#%E5%AE%9E%E6%96%BD)

其 `docker-compose.yml` 在这里：

[Awesome-TTRSS/docker-compose.yml at main · HenryQW/Awesome-TTRSS (github.com)](https://github.com/HenryQW/Awesome-TTRSS/blob/main/docker-compose.yml),

#### 1\. 修改 `docker-compose`[](https://ewhisper.cn/posts/60709/#1-%20%E4%BF%AE%E6%94%B9%20-docker-compose)

有 2 个地方需要修改：

1.  环境变量：
    1.  `SELF_URL_PATH=https://ttrss.ewhisper.cn/` （你自己的域名）
    2.  `DB_PASS=changeit` (postgresql 数据库密码）
2.  使用 `kompose` 转换，转换前，需要在 `docker-compose.yml` 补充相关信息以保证转换 k8s service 成功，具体为在各个 docker compose 的 service 里加上 `ports` 字段。`docker-compose.yml` 修改的内容见这里：

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br><span>8</span><br><span>9</span><br><span>10</span><br><span>11</span><br><span>12</span><br><span>13</span><br><span>14</span><br><span>15</span><br><span>16</span><br><span>17</span><br><span>18</span><br><span>19</span><br><span>20</span><br></pre></td><td><pre><code><span>version:</span> <span>"3"</span><br><span>services:</span><br>  <span>service.rss:</span><br>    <span>environment:</span><br>      <span>-</span> <span>SELF_URL_PATH=https://ttrss.ewhisper.cn/</span> <span># please change to your own domain</span><br>      <span>-</span> <span>DB_PASS=changeit</span> <span># use the same password defined in `database.postgres`  </span><br><span>...</span><br>  <span>service.mercury:</span><br>    <span>ports:</span><br>      <span>-</span> <span>3000</span><span>:3000</span><br><span>...</span><br>  <span>service.opencc:</span> <span># set OpenCC API endpoint to `service.opencc:3000` on TTRSS plugin setting page</span><br>    <span>ports:</span><br>      <span>-</span> <span>3000</span><span>:3000</span><br><span>...</span><br>  <span>database.postgres:</span><br>    <span>environment:</span><br>      <span>-</span> <span>POSTGRES_PASSWORD=changeit</span><br>    <span>ports:</span><br>      <span>-</span> <span>5432</span><span>:5432</span>      <br></code><p><i></i>YAML</p></pre></td></tr></tbody></table>

为方便查看，相关的 yaml 文件都放在这里了：[east4ming/rsshub-ttrss-k8s-deploy (gitee.com)](https://gitee.com/caseycui/rsshub-ttrss-k8s-deploy)

修改后的 `docker-compose.yml` 在这里查看：[ttrss/docker-compose.yml · east4ming/rsshub-ttrss-k8s-deploy - 码云 - 开源中国 (gitee.com)](https://gitee.com/caseycui/rsshub-ttrss-k8s-deploy/blob/main/ttrss/docker-compose.yml)

#### 2\. 使用 `kompose` 转换[](https://ewhisper.cn/posts/60709/#2-%20%E4%BD%BF%E7%94%A8%20-kompose-%20%E8%BD%AC%E6%8D%A2)

命令如下：

在 `docker-compose.yml` 所在目录下执行：

<table><tbody><tr><td><pre><span>1</span><br></pre></td><td><pre><code>kompose convert -o ./k8s/ --pvc-request-size 2Gi<br></code><p><i></i>BASH</p></pre></td></tr></tbody></table>

> 📝 **Note:**
> 
> `--pvc-request-size 2Gi` 按需调整。

转换后，目录结构如下：（转换后还会生成 NetWorkPolicy 文件，个人认为没必要，就删除掉了相关文件和 label; 另外，生成的文件中有的 字段包含 `.` , 以防万一，都替换为了 `-`):

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br><span>8</span><br><span>9</span><br><span>10</span><br><span>11</span><br><span>12</span><br><span>13</span><br><span>14</span><br></pre></td><td><pre><code>└── ttrss<br>    ├── docker-compose<span>.yml</span><br>    └── k8s<br>        ├── database-postgres-claim0-persistentvolumeclaim<span>.yaml</span><br>        ├── database-postgres-deployment<span>.yaml</span><br>        ├── database<span>.postgres-service</span><span>.yaml</span><br>        ├── feed-icons-persistentvolumeclaim<span>.yaml</span><br>        ├── service-mercury-deployment<span>.yaml</span><br>        ├── service-opencc-deployment<span>.yaml</span><br>        ├── service-rss-deployment<span>.yaml</span><br>        ├── service<span>.mercury-service</span><span>.yaml</span><br>        ├── service<span>.opencc-service</span><span>.yaml</span><br>        └── service<span>.rss-service</span><span>.yaml</span></code><p><i></i>STYLUS</p></pre></td></tr></tbody></table>

除此之外还需要手动创建一个 ingress, 用于对外暴露服务（这里用的是 Traefik CRD - IngressRoute).

1.  Tiny Tiny Rss
    1.  Deployment: `service-rss-deployment.yaml` (🐾这里还需要添加一个 ENV: DB\_HOST, 以通过 DB SVC 访问 DB)
    2.  Service: `service.rss-service.yaml` （用于对接 Ingress, 对外提供服务）
    3.  IngressRoute: `ingress-rss-service.yaml`
2.  DB - PostgreSQL
    1.  Deployment: `database-postgres-deployment.yaml`
    2.  SVC: `database.postgres-service.yaml` (Tiny Tiny Rss 通过该 SVC 连接到 DB)
    3.  PVC: `database-postgres-claim0-persistentvolumeclaim.yaml`（申请持久化存储）
3.  其他服务 - opencc
    1.  Deployment: `service-opencc-deployment.yaml`
    2.  Service: `service-opencc-deployment.yaml`
4.  其他服务 - mercury
    1.  Deployment: `service-mercury-deployment.yaml`
    2.  Service: `service.mercury-service.yaml`

具体的 K8S yaml 内容见这里：[ttrss/k8s · east4ming/rsshub-ttrss-k8s-deploy - 码云 - 开源中国 (gitee.com)](https://gitee.com/caseycui/rsshub-ttrss-k8s-deploy/tree/main/ttrss/k8s)

> 📝 **Note:**
> 
> [ttrss/k8s/ingress-rss-service.yaml · east4ming/rsshub-ttrss-k8s-deploy - 码云 - 开源中国 (gitee.com)](https://gitee.com/caseycui/rsshub-ttrss-k8s-deploy/blob/main/ttrss/k8s/ingress-rss-service.yaml) 的配置沿袭于我的另一篇文章：《 [基于 Traefik 的激进 TLS 安全配置实践 - 东风微鸣技术博客 (ewhisper.cn)](https://ewhisper.cn/posts/14331/)》

#### 3\. 部署[](https://ewhisper.cn/posts/60709/#3-%20%E9%83%A8%E7%BD%B2)

使用 `kubectl` 部署：

<table><tbody><tr><td><pre><span>1</span><br></pre></td><td><pre><code>kubectl -n rss create -f ./k8s/<br></code><p><i></i>BASH</p></pre></td></tr></tbody></table>

#### 4\. 配置 DNS Record[](https://ewhisper.cn/posts/60709/#4-%20%E9%85%8D%E7%BD%AE%20-DNS-Record)

在 DNS 提供商控制台（本例为 DNSPod) 配置对应域名 <[ttrss.ewhisper.cn](http://ttrss.ewhisper.cn/)\> 的 DNS Record:

-   ttrss, A 记录，指向 K8S 集群 Ingress 对应的公网地址

#### 5\. 访问验证[](https://ewhisper.cn/posts/60709/#5-%20%E8%AE%BF%E9%97%AE%E9%AA%8C%E8%AF%81)

访问首页：[https://ttrss.ewhisper.cn/](https://ttrss.ewhisper.cn/) , 默认账户：`admin` 密码：`password`，请第一时间更改。

效果如下：

[![Tiny Tiny Rss](https://pic-cdn.ewhisper.cn/img/2022/03/22/4f10cc52d66e64a92c334af662c2d26a-20220322231202.png)](https://pic-cdn.ewhisper.cn/img/2022/03/22/4f10cc52d66e64a92c334af662c2d26a-20220322231202.png "Tiny Tiny Rss")

RssHub 搭建的步骤几乎和 Tiny Tiny RSS 一样。具体如下：

#### 1\. 修改 `docker-compose`[](https://ewhisper.cn/posts/60709/#1-%20%E4%BF%AE%E6%94%B9%20-docker-compose-2)

使用 `kompose` 转换，转换前，需要在 `docker-compose.yml` 补充相关信息以保证转换 k8s service 成功，具体为在各个 docker compose 的 service 里加上 `ports` 字段。

#### 2\. 使用 `kompose` 转换[](https://ewhisper.cn/posts/60709/#2-%20%E4%BD%BF%E7%94%A8%20-kompose-%20%E8%BD%AC%E6%8D%A2%20-2)

命令如下：

在 `docker-compose.yml` 所在目录下执行：

<table><tbody><tr><td><pre><span>1</span><br></pre></td><td><pre><code>kompose convert -o ./k8s/ --pvc-request-size 2Gi<br></code><p><i></i>BASH</p></pre></td></tr></tbody></table>

> 📝 **Note:**
> 
> `--pvc-request-size 2Gi` 按需调整。

转换后，目录结构如下：

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br><span>8</span><br><span>9</span><br><span>10</span><br><span>11</span><br><span>12</span><br></pre></td><td><pre><code>.<br>├── docker-compose<span>.yml</span><br>└── k8s<br>    ├── browserless-deployment<span>.yaml</span><br>    ├── browserless-service<span>.yaml</span><br>    ├── redis-data-persistentvolumeclaim<span>.yaml</span><br>    ├── redis-deployment<span>.yaml</span><br>    ├── redis-service<span>.yaml</span><br>    ├── rsshub-deployment<span>.yaml</span><br>    └── rsshub-service<span>.yaml</span><p><span>1</span> directory, <span>8</span> files</p></code><p><i></i>STYLUS</p></pre></td></tr></tbody></table>

除此之外还需要手动创建一个 ingress, 用于对外暴露服务（这里用的是 Traefik CRD - IngressRoute).

1.  RssHub
    1.  Deployment: `rsshub-deployment.yaml`
    2.  Service: `rsshub-service.yaml` （用于对接 Ingress, 对外提供服务）
    3.  IngressRoute: `rsshub-ingress.yaml`
2.  Browserless - Chrome
    1.  Deployment: `browserless-deployment.yaml`
    2.  SVC: `browserless-service.yaml` (RssHub 通过该 SVC 连接到 Browserless)
3.  Redis
    1.  Deployment: `redis-deployment.yaml`
    2.  Service: `redis-service.yaml`
    3.  PVC: `redis-data-persistentvolumeclaim.yaml`

具体的 K8S yaml 内容见这里：[rsshub/k8s · east4ming/rsshub-ttrss-k8s-deploy - 码云 - 开源中国 (gitee.com)](https://gitee.com/caseycui/rsshub-ttrss-k8s-deploy/tree/main/rsshub/k8s)

> 📝 **Note:**
> 
> [rsshub/k8s/rsshub-ingress.yaml · east4ming/rsshub-ttrss-k8s-deploy - 码云 - 开源中国 (gitee.com)](https://gitee.com/caseycui/rsshub-ttrss-k8s-deploy/blob/main/rsshub/k8s/rsshub-ingress.yaml) 的配置沿袭于我的另一篇文章：《 [基于 Traefik 的激进 TLS 安全配置实践 - 东风微鸣技术博客 (ewhisper.cn)](https://ewhisper.cn/posts/14331/)》

#### 3\. 部署[](https://ewhisper.cn/posts/60709/#3-%20%E9%83%A8%E7%BD%B2%20-2)

使用 `kubectl` 部署：

<table><tbody><tr><td><pre><span>1</span><br></pre></td><td><pre><code>kubectl -n rss create -f ./k8s/<br></code><p><i></i>BASH</p></pre></td></tr></tbody></table>

#### 4\. 配置 DNS Record[](https://ewhisper.cn/posts/60709/#4-%20%E9%85%8D%E7%BD%AE%20-DNS-Record-2)

在 DNS 提供商控制台（本例为 DNSPod) 配置对应域名 <[rss.ewhisper.cn](http://rss.ewhisper.cn/)\> 的 DNS Record:

-   rss, A 记录，指向 K8S 集群 Ingress 对应的公网地址

#### 5\. 访问验证[](https://ewhisper.cn/posts/60709/#5-%20%E8%AE%BF%E9%97%AE%E9%AA%8C%E8%AF%81%20-2)

如果看到这个页面，证明已经部署成功：

[![RssHub 首页](https://pic-cdn.ewhisper.cn/img/2022/03/22/500404c856f2ffeec06b7af13d6240e3-20220322232745.png)](https://pic-cdn.ewhisper.cn/img/2022/03/22/500404c856f2ffeec06b7af13d6240e3-20220322232745.png "RssHub 首页")

可以通过 TTRss 的页面订阅 RssHub 的源来验证 RssHub 是否正常运行，如下图：

[![TTRSS 订阅信息源](https://pic-cdn.ewhisper.cn/img/2022/03/22/c4730cc23e7547f24c5db6184544dd51-image-20220322233020444.png)](https://pic-cdn.ewhisper.cn/img/2022/03/22/c4730cc23e7547f24c5db6184544dd51-image-20220322233020444.png "TTRSS 订阅信息源")

点击订阅后成功，如下图：

[![订阅效果](https://pic-cdn.ewhisper.cn/img/2022/03/22/56e7b653ebb398171f0266fe31e2133d-20220322233129.png)](https://pic-cdn.ewhisper.cn/img/2022/03/22/56e7b653ebb398171f0266fe31e2133d-20220322233129.png "订阅效果")

证明 RssHub 已经正常运行。

🎉🎉🎉

## 总结[](https://ewhisper.cn/posts/60709/#%E6%80%BB%E7%BB%93%20-18)

通过如上的配置，我们可以通过自己的基于浏览器的 Tiny Tiny RSS 阅读器来订阅并阅读消息，并可以通过 RssHub 来将各种各样的信息转换为可订阅的 Rss 路由。

## 参考资料[](https://ewhisper.cn/posts/60709/#%E5%8F%82%E8%80%83%E8%B5%84%E6%96%99)

-   [rss（简易信息聚合）\_百度百科 (baidu.com)](https://baike.baidu.com/item/rss/24470)
-   [介绍 | RSSHub](https://docs.rsshub.app/)
-   [🐋 Awesome TTRSS | 🐋 Awesome TTRSS (henry.wang)](https://ttrss.henry.wang/zh/)