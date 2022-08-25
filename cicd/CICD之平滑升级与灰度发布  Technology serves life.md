## 简介

在云服务的部署过程中平滑升级和灰度发布是两个非常重要的发布策略，也是运维水平高低的体现。平滑升级主要解决0宕机部署，灰度发布主要解决流量控制。下面对两种策略的目标和实现方式的核心点做了介绍。

## 平滑升级

平滑升级实现在升级部署过程中，不中断对外服务，也就是用户可以持续使用，感觉不到服务的升级，如果升级到新版本如果有问题，也可以在用户使用之前做回退或修复，避免有问题的版本被用户使用。

[![Screen Shot 2016-08-14 at 4.49.20 PM](https://sunzhongkui.files.wordpress.com/2016/08/screen-shot-2016-08-14-at-4-49-20-pm.png?w=640)](https://sunzhongkui.files.wordpress.com/2016/08/screen-shot-2016-08-14-at-4-49-20-pm.png)

**目标**：服务升级过程中保持对外服务不中断，实现0宕机部署。

**实现方式**：其关键在于每一时刻均有至少一个节点还能正常提供服务。步骤如下：

1.  需要部署多个服务节点，将服务分为A、B两批更新
2.  更新服务时，先将A批服务的路由从HAProxy上面摘下来
3.  更新A批服务
4.  A批服务健康检查正常后，重新加入路由
5.  摘下B批服务的路由
6.  更新B批容器
7.  B批服务健康检查正常后，重新加入路由
8.  服务平滑升级完毕

**使用场景**

1.  不影响线上用户使用的情况下，对服务进行升级

## 灰度发布

线上流量通过分流规则访问到不同的后端服务。虽然所有用户访问的都是同一个域名，但是他们真正访问的服务是不同的。

**目标**：线上用户流量可以按照分流规则分配到不同的后端服务。

**原理**：不通版本的服务可以共享同一路由信息，通过调整HAProxy权重的方式来做到按规则分流。

**实现方式**： 例如有服务的两个版本A 和 a，他们使用相同的路由，挂在同一个域名（例如www.example.com）的后端，但是A和a的权重不一样，通过调整权重将流量分配到不同的后端，实现灰度发布。

**使用场景**

1.  A/B测试
2.  服务升级发布时，保持新旧版本并存，测试完毕之前，让测试人员流量进入新版本，线上用户流量进入旧版本

This entry was posted in [cicd](https://sunzhongkui.wordpress.com/tag/cicd/), [ops](https://sunzhongkui.wordpress.com/tag/ops/), [Webservice](https://sunzhongkui.wordpress.com/category/webservice/) and tagged [cicd](https://sunzhongkui.wordpress.com/tag/cicd/), [ops](https://sunzhongkui.wordpress.com/tag/ops/), [WebService](https://sunzhongkui.wordpress.com/tag/webservice-2/). Bookmark the [permalink](https://sunzhongkui.wordpress.com/2016/07/14/cicd%e4%b9%8b%e5%b9%b3%e6%bb%91%e5%8d%87%e7%ba%a7%e4%b8%8e%e7%81%b0%e5%ba%a6%e5%8f%91%e5%b8%83/ "Permalink to [[CICD]]之平滑升级与灰度发布").