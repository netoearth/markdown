### 简介

GitHub：[https://github.com/Qihoo360/wayne](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fgithub.com%2FQihoo360%2Fwayne)

Wayne是一个Kubernetes的可视化管理平台，通过直观的页面操作便可完成Kubernetes中资源的创建、部署等操作。

采用微内核架构，通过插件化的方式将不同功能尽量的分离，更利于各种定制化功能的扩展。

在此基础上，融入了部门、项目的概念，通过RBAC的方式细化了资源控制的权限，适合建立企业内部的私有云平台。

![](https://www.helloworld.net/img/default-avatar.png)

### 功能特性

-   可视化操作：提供直观、简便的方式操作Kubernetes集群，减小学习成本，快速上线业务。
-   多样的编辑模式：支持图形化编辑，也支持Json、Yaml两种高级定制化编辑模式。
-   微内核架构：采用可扩展的插件化方式开发，定制化选择特性功能，更方便的集成符合企业需求的新功能。
-   多集群管理：可以同时管理多个Kubernetes集群，更方便地管理多个集群。
-   丰富的权限管理：将资源抽象化为部门、项目级别，角色的权限可以更细化的控制，适用于多部门、多项目的统一集中管理。
-   多种登录模式：支持企业级LDAP登录、支持OAuth2登录，支持数据库登录多种模式。
-   完备的审计：所有操作都会有完整的审计功能，方便追踪操作历史。
-   开放平台：支持APIKey开放平台，用户可自主申请相关APIKey并管理自己的项目。
-   多层次监控：提供多级别的监控统计信息，实时关注集群的运行状态。

### 组件

-   Web UI: 提供完整的业务开发和平台运维功能体验。
-   Worker: 扩展一系列基于消息队列的功能，例如 Audit 和 Webhooks 等审计组件。

### 项目依赖

-   Golang 1.9+(installation manual)
-   Docker 17.05+ (installation manual)
-   Bee (installation manual) (请务必使用链接版本，不要使用 beego 官方版本，存在一些定制)
-   Node.js 8+ and npm 5+ (installation with nvm)
-   MySQL 5.6+ (Wayne 主要数据都存在 MySQL 中)
-   RabbitMQ (可选，如需扩展审计功能，例如操作审计和 Webhooks 等，则需部署)

### 架构

项目整体采用前后端分离的方案实现。

-   前端采用Angular框架进行数据交互和展示，使用Ace编辑器进行Kubernetes资源模版编辑。
-   后端采用Beego框架做数据接口处理，持久层采用MySQL存储，使用client-go与Kubernetes进行交互。

![](https://oscimg.oschina.net/oscnet/f4e37b778e262831eccc5ead36c28b30b84.png)

## 基于Kubernetes部署Wayne

### 安装git

### **安装 docker-compose**

查看docker-compose版本

### 快速启动

#### 克隆代码仓

#### 启动MySQL（可选）

如果没有可用的 MySQL 服务，可以通过 docker-compose 快速创建：

创建完成后，停止MySQL服务器，创建配置文件：

编辑 dev.conf ，写入数据库相关配置（请修改为数据库实际地址）

_数据未进行持久化，生产环境一定要做数据持久化，避免数据丢失 。可修改 wayne 目录下的 docker-compose.yaml 文件实现，详细请了解 docker volumes的应用。_

进入Wayne根目录，后台启动MySQL服务

**启动Wayne服务**

修改 Wayne 访问端口为8081，默认 端口已被 8080 已被 kubernetes apiserver 占用

进入 Wayne 跟目录，启动 Wayne 服务

_“ Wayne配置文件路径：/etc/kubernetes/wayne/src/backend/conf ”_

Wayne启动后，可以通过 http://192.168.90.82:8080/admin 访问本地 Wayne, 默认管理员账号 admin:admin

### 扩展应用

将wayne和mysql服务设置为开机启动

赋予 /etc/rd.d/rc.local执行权限

添加启动脚本