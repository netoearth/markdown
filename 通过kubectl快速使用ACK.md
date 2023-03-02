阿里云容器服务Kubernetes版（简称容器服务ACK）提供高性能的容器化应用管理服务，让您轻松高效地在云端运行Kubernetes容器化应用。本文将指导您如何通过kubectl在ACK集群中快速部署并公开一个容器化Demo应用，并监控应用的运行情况。

## 背景信息

-   本教程中所使用的Demo应用ACK-Cube为一个线上魔方游戏，该游戏将通过容器镜像部署到ACK Pro版集群中。完成本教程后，您将得到一个ACK Pro版集群和魔方游戏应用。![cube](https://help-static-aliyun-doc.aliyuncs.com/assets/img/zh-CN/2613290361/p324047.png)
-   Demo应用的容器镜像基于[开源项目](https://github.com/bsehovac/the-cube)构建，镜像地址为`registry.cn-hangzhou.aliyuncs.com/acr-toolkit/ack-cube:1.0`。
-   Kubectl是标准的Kubernetes命令行管理工具，通过kubectl您可以连接和管理ACK集群。关于kubectl命令行的更多信息，请参见[kubectl](https://kubernetes.io/docs/user-guide/kubectl/)。
-   CloudShell是阿里云推出的云命令行工具，您可以在容器服务ACK控制台上打开CloudShell，无需任何安装或配置操作，可以直接在CloudShell中通过kubectl管理集群。

## 前提条件

-   您已注册阿里云账号并完成实名认证，请参见[注册阿里云账号](https://help.aliyun.com/document_detail/324609.htm#task-2012947 "在使用阿里云之前，您需要先注册一个阿里云账号。")和[个人实名认证](https://help.aliyun.com/document_detail/324614.htm#task-2020003 "为了确保您可以正常使用阿里云产品和服务，您需要完成个人实名认证。")。
-   您已了解Kubernetes的相关基本概念，请参见[与原生Kubernetes名词对照](https://help.aliyun.com/document_detail/113088.htm#concept-ib1-kqf-hhb "本文主要为您介绍容器服务ACK与原生Kubernetes名词对照情况。")。你还可以通过[CNCF × Alibaba 云原生技术公开课](https://edu.aliyun.com/course/1651/lesson/list)，深入学习Kubernetes。

## 操作流程

![workflow-kubectl](https://help-static-aliyun-doc.aliyuncs.com/assets/img/zh-CN/4732290361/p324032.png)

## 操作视频

首次使用时，您需要开通容器服务ACK，并为其授权相应云资源的访问权限。

1.  登录[容器服务ACK开通页面](https://common-buy.aliyun.com/?spm=5176.2020520152.0.0.75a361b1SJ9jQT&commodityCode=csk_propayasgo_public_cn)。
2.  阅读并选中容器服务ACK服务协议。
3.  单击立即开通。
4.  登录[容器服务管理控制台](https://cs.console.aliyun.com/)。
5.  在容器服务需要创建默认角色页面，单击前往RAM进行授权进入[云资源访问授权](https://ram.console.aliyun.com/#/role/authorize?request=%7B%22ReturnUrl%22:%22https:%2F%2Fcs.console.aliyun.com%2F%22,%22Service%22:%22CS%22,%22Requests%22:%7B%22request1%22:%7B%22RoleName%22:%22AliyunCSManagedLogRole%22,%22TemplateId%22:%22AliyunCSManagedLogRole%22%7D,%22request2%22:%7B%22RoleName%22:%22AliyunCSManagedCmsRole%22,%22TemplateId%22:%22AliyunCSManagedCmsRole%22%7D,%22request3%22:%7B%22RoleName%22:%22AliyunCSManagedCsiRole%22,%22TemplateId%22:%22AliyunCSManagedCsiRole%22%7D,%22request4%22:%7B%22RoleName%22:%22AliyunCSManagedVKRole%22,%22TemplateId%22:%22AliyunCSManagedVKRole%22%7D,%22request5%22:%7B%22RoleName%22:%22AliyunCSClusterRole%22,%22TemplateId%22:%22Cluster%22%7D,%22request6%22:%7B%22RoleName%22:%22AliyunCSServerlessKubernetesRole%22,%22TemplateId%22:%22ServerlessKubernetes%22%7D,%22request7%22:%7B%22RoleName%22:%22AliyunCSKubernetesAuditRole%22,%22TemplateId%22:%22KubernetesAudit%22%7D,%22request8%22:%7B%22RoleName%22:%22AliyunCSManagedNetworkRole%22,%22TemplateId%22:%22AliyunCSManagedNetworkRole%22%7D,%22request9%22:%7B%22RoleName%22:%22AliyunCSDefaultRole%22,%22TemplateId%22:%22Default%22%7D,%22request10%22:%7B%22RoleName%22:%22AliyunCSManagedKubernetesRole%22,%22TemplateId%22:%22ManagedKubernetes%22%7D,%22request11%22:%7B%22RoleName%22:%22AliyunCSManagedArmsRole%22,%22TemplateId%22:%22AliyunCSManagedArmsRole%22%7D%7D%7D)页面，然后单击同意授权。
    
    完成以上授权后，刷新控制台即可使用容器服务ACK。
    

## 步骤二：创建ACK Pro版集群

本步骤指导您如何快速创建一个ACK Pro版集群，因此参数配置大多以默认为主。关于创建集群的详细参数描述，请参见[创建ACK Pro版集群](https://help.aliyun.com/document_detail/176833.htm#task-skz-qwk-qfb "ACK的Pro托管版集群相比原标准托管版集群进一步提升了集群的可靠性、安全性和调度性能，并且支持赔付标准的SLA，适合生产环境下有着大规模业务，对稳定性和安全性有高要求的企业客户。本文介绍如何通过容器服务控制台创建ACK Pro托管版集群。")。

1.  登录[容器服务管理控制台](https://cs.console.aliyun.com/)。
2.  在控制台左侧导航栏中，单击集群。
3.  在集群列表页面中，单击页面右上角的创建集群。
4.  在ACK托管版页签下，配置以下相关的集群参数，未说明参数保留默认设置即可。
    
     
    | 参数 | 说明 |
    | --- | --- |
    | 集群名称 | 填写集群的名称。示例：ACK-Demo。 |
    | 集群规格 | 选择集群规格。示例：Pro版。关于ACK Pro版集群的详细信息，请参见[ACK Pro版集群概述](https://help.aliyun.com/document_detail/173290.htm#concept-2558837 "ACK Pro托管版集群是在ACK标准托管版基础上针对企业大规模生产环境进一步增强了可靠性、安全性，并且提供可赔付的SLA的Kubernetes集群。")。 |
    | 地域 | 选择集群所在的地域。示例：华北2（北京）。 |
    | 专有网络 | Kubernetes集群仅支持运行于专有网络，因此您需要为集群指定专有网络VPC，且该VPC必须与集群处于同一地域。
    示例：在华北2（北京）地域下创建名为vpc-ack-demo的VPC。通过单击创建专有网络进行创建，详情请参见[创建和管理专有网络](https://help.aliyun.com/document_detail/65398.htm#task-1012575 "专有网络VPC（Virtual Private Cloud）是您专有的云上私有网络。您可以完全掌控自己的专有网络，例如选择IP地址范围、配置路由表和网关等。专有网络创建后，您也可以添加附加IPv4网段扩充专有网络的网段。本文为您介绍如何创建和管理专有网络。")。
    
     |
    | 虚拟交换机 | 选择用于集群节点间通信的交换机。示例：在vpc-ack-demo的VPC下创建一个名为vswitch-ack-demo的虚拟交换机，并选择使用该交换机。
    
    通过单击创建虚拟交换机进行创建，详情请参见[使用交换机](https://help.aliyun.com/document_detail/65387.htm#task-1012575 "交换机（vSwitch）是组成专有网络的基础网络模块，用来连接不同的云资源。成功创建交换机后，您可以在交换机中创建云资源、绑定自定义路由表或者绑定网络ACL。本文介绍如何使用交换机。")。
    
     |
    | API Server访问 | 设置集群API Server是否可在公网访问，当您需要从公网远程管理集群时，需要配置弹性公网IP（EIP）。
    
    示例：选中使用EIP暴露API Server。
    
     |
    
5.  单击下一步：节点池配置，配置以下相关参数，未说明参数保留默认设置即可。
    
     
    | 参数 | 说明 |
    | --- | --- |
    | 实例规格 | 为集群选配所使用的节点。为了保证集群的稳定性，建议的实例规格为：vCPU ≥ 4核，内存 ≥ 8 GiB。关于如何选型以及规格介绍，请参见[ECS选型](https://help.aliyun.com/document_detail/98886.htm#concept-yww-f2t-zfb "本文介绍构建Kubernetes集群时该如何选择ECS类型以及选型的注意事项。")和[实例规格族](https://help.aliyun.com/document_detail/25378.htm#concept-sx4-lxv-tdb "实例是能够为您的业务提供计算服务的最小单位，不同的实例规格可以提供的计算能力不同。本文为您介绍在售的所有ECS实例规格族，包括每种实例规格族的特点、在售规格和适用场景。")。
    您可通过设置vCPU和内存的大小，或者直接搜索实例规格，选用该规格的节点。
    
     |
    | 数量 | 根据需要设置集群的节点数量。示例：2个，防止单点故障。 |
    | 系统盘 | 选择节点所使用的系统盘。示例：ESSD云盘，40 GiB（最小规格）。 |
    | 登录方式 | 选择登录节点的方式，并输入密码。示例：设置密码。 |
    
6.  单击下一步：组件配置，所有组件使用默认配置。
7.  单击下一步：确认配置，然后选中并阅读服务协议，单击创建集群。
    
    **说明** 集群的创建时间一般约为10分钟。创建完成后，在集群列表页面，可以看到新创建的集群。
    

## 步骤三：连接集群

本步骤指导您如何通过kubectl客户端或CloudShell快速连接到ACK集群。更多信息，请参见[通过kubectl工具连接集群](https://help.aliyun.com/document_detail/86494.htm#task-2076136 "除了通过容器服务控制台来管理集群之外，您还可以通过Kubernetes命令行工具kubectl来管理集群以及应用。本文介绍如何通过kubectl客户端连接ACK集群。")和[在CloudShell上通过kubectl管理Kubernetes集群](https://help.aliyun.com/document_detail/100650.htm#task-y24-gbm-2gb "CloudShell是阿里云推出的云命令行工具，您可以使用CloudShell工具在任意浏览器上运行CloudShell命令管理阿里云资源。本文为您介绍如何在容器服务ACK控制台上利用CloudShell通过kubectl管理集群。")。

方式一：通过kubectl客户端连接集群

1.  登录[容器服务管理控制台](https://cs.console.aliyun.com/)。
2.  在控制台左侧导航栏中，单击集群。
3.  在集群列表页面中，单击目标集群名称（即ACK-Demo）。
4.  在集群信息页面，单击连接信息页签，复制公网访问页签下的内容（即集群的公网访问凭证）。
5.  将复制的集群凭证内容粘贴至$HOME/.kube目录下的config文件中，保存并退出。
    
    **说明** 如果$HOME/目录下没有.kube目录和config文件，请自行创建。
    
6.  执行kubectl命令验证集群的连通性。
    
    以查询命名空间为例，执行以下命令。
    
    预期输出：
    
    ```
    NAME              STATUS   AGE
    arms-prom         Active   4h39m
    default           Active   4h39m
    kube-node-lease   Active   4h39m
    kube-public       Active   4h39m
    kube-system       Active   4h39m
    ```
    

方式二：通过CloudShell连接集群

1.  登录[容器服务管理控制台](https://cs.console.aliyun.com/)。
2.  在控制台左侧导航栏中，单击集群。
3.  在集群列表页面中，选择目标集群右侧操作列下的。
    
    等待数秒启动成功之后，您就可以在CloudShell界面通过kubectl命令来管理集群和应用。
    

## 步骤四：部署并公开应用

本步骤指导您如何通过kubectl在新创建的ACK集群中快速部署一个无状态应用（Deployment），并将其通过LoadBalancer类型服务（Service）公开。关于服务公开的详细操作，请参见[通过使用自动创建SLB的服务公开应用](https://help.aliyun.com/document_detail/197444.htm#task-1942045 "当您没有可用的SLB时，Cloud Controller Manager（CCM）组件可以为LoadBalancer类型服务自动创建SLB，并对其进行管理。本文以Nginx应用为例，介绍如何通过使用自动创建SLB的服务来公开应用。")。

1.  使用以下示例应用的YAML内容，创建名为ack-cube.yaml的文件。
    
    ```
    apiVersion: apps/v1 # for versions before 1.8.0 use apps/v1beta1
    kind: Deployment
    metadata:
      name: ack-cube #应用名称。
      labels:
        app: ack-cube
    spec:
      replicas: 2 #设置副本数量。
      selector:
        matchLabels:
          app: ack-cube  #对应服务中Selector的值需要与其一致，才可以通过服务公开此应用。
      template:
        metadata:
          labels:
            app: ack-cube
        spec:
          containers:
          - name: ack-cube
            image: registry.cn-hangzhou.aliyuncs.com/acr-toolkit/ack-cube:1.0 #替换为您实际的镜像地址，格式为：<image_name:tags>。
            ports:
            - containerPort: 80 #需要在服务中暴露该端口。
            resources:
              limits: #设置资源限制。
                cpu: '1'
                memory: 1Gi
              requests: #设置所需资源
                cpu: 500m
                memory: 512Mi        
    ```
    
2.  执行以下命令，部署Demo应用ack-cube。
    
    ```
    kubectl apply -f ack-cube.yaml
    ```
    
3.  执行以下命令，确认示例应用状态正常。
    
    ```
    kubectl get deployment ack-cube
    ```
    
    预期输出：
    
    ```
    NAME       READY   UP-TO-DATE   AVAILABLE   AGE
    ack-cube   2/2     2            2           96s
    ```
    
4.  使用以下示例服务的YAML内容，创建名为ack-cube-svc.yaml的文件。将`selector`修改为ack-cube.yaml示例应用文件中`matchLabels`的值（本示例为`app: ack-cube`），从而将该服务关联至后端应用。
    
    ```
    apiVersion: v1
    kind: Service
    metadata:
      labels:
        app: ack-cube
      name: ack-cube-svc
      namespace: default
    spec:
      ports:
      - port: 80
        protocol: TCP
        targetPort: 80
      selector:
        app: ack-cube # 需要与Deployment YAML文件中的matchLabels的值一致。
      type: LoadBalancer
    ```
    
5.  执行以下命令创建名为ack-cube-svc的服务，并通过其公开应用。
    
    ACK会自动创建一个公网SLB，并将其绑定至该服务。
    
    ```
    kubectl apply -f ack-cube-svc.yaml
    ```
    
6.  执行以下命令确认LoadBalancer类型的服务创建成功。
    
    示例应用将通过EXTERNAL-IP字段的IP地址向公网公开。
    
    ```
    kubectl get svc ack-cube-svc
    ```
    
    预期输出：
    
    ```
    NAME           TYPE           CLUSTER-IP      EXTERNAL-IP     PORT(S)        AGE
    ack-cube-svc   LoadBalancer   172.16.72.161   47.94.xx.xx   80:31547/TCP   32s
    ```
    

## 步骤五：测试应用

在浏览器地址栏输入该服务EXTERNAL-IP字段的IP地址，即可开始魔方游戏。

## 步骤六：监控应用

关于如何监控应用的运行情况，请参见[步骤五：监控应用](https://help.aliyun.com/document_detail/309542.htm#section-kvn-181-pxw)。

## 相关文档

-   为了保证应用能够动态调整所需容器资源，您可以配置容器水平伸缩（HPA）、定时容器水平伸缩（CronHPA）、容器垂直伸缩（HPA）等，详情请参见[弹性伸缩概述](https://help.aliyun.com/document_detail/176660.htm#concept-1918532 "弹性伸缩是根据业务需求和策略，经济地自动调整弹性计算资源的管理服务。本文介绍弹性伸缩的背景信息和弹性伸缩涉及的组件。")。
-   除了通过服务（Service）公开应用，您还可以通过路由（Ingress）实现对应用的七层网络路由控制，详情请参见[创建Ingress路由](https://help.aliyun.com/document_detail/86536.htm#task-1796525 "Ingress是Kubernetes中的一个资源对象，用来管理集群外部访问集群内部服务的方式。您可以通过Ingress资源来配置不同的转发规则，从而根据转发规则访问集群内Pod。本文介绍如何通过控制台和Kubectl方式创建、查看、更新和删除Ingress。")。
-   除了观测容器性能，您还可以观测集群基础设施、应用性能和用户业务，详情请参见[可观测性体系概述](https://help.aliyun.com/document_detail/203344.htm#task-2042182 "可观测性指如何从外部输出推断及衡量系统内部状态。Kubernetes可观测性体系包含监控和日志两部分，监控可以帮助开发者查看系统的运行状态，而日志可以协助问题的排查和诊断。本文介绍阿里云容器服务ACK可观测性生态分层和各层的可观测能力，以帮助您更好地对容器服务可观测性生态有一个全面的认识。")。
-   为了避免产生不必要的费用，请及时清理不需要的集群，详情请参见[删除集群](https://help.aliyun.com/document_detail/86498.htm#task-2372991 "您可以通过容器服务管理控制台删除不再使用的集群。")。