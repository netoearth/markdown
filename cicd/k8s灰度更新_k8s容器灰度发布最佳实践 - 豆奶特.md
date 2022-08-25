___

k8s中的容器一般是通过deployment管理的，那么一次滚动升级理论上会更新所有pod，这由deployment资源特性保证的，但在实际的工作场景下，需要灰度发布进行服务验证，即只发布部分节点，这似乎与k8s的deployment原理相违背，但是灰度发布的必要性，运维同学都非常清楚，如何解决这一问题？

最佳实践：  
定义两个不同的deployment，例如：fop-gate和fop-gate-canary，但是管理的pod所使用的镜像、配置文件全部相同，不同的是什么呢？  
答案是：replicas (灰度的fop-gate-canary的replicas是1，fop-gate的副本数是9)

```
cat deployment.yaml
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  {{if eq .system.SERVICE  "fop-gate-canary"}}
  name: fop-gate-canary
  {{else if eq .system.SERVICE "fop-gate"}}
  name: fop-gate
  {{end}}  namespace: dora-apps
  labels:
    app: fop-gate
    team: dora
    type: basic
  annotations:    log.qiniu.com/global.agent: "logexporter"
    log.qiniu.com/global.version: "v2"spec:
  {{if eq .system.SERVICE  "fop-gate-canary"}}
  replicas: 1
  {{else if eq .system.SERVICE "fop-gate"}}
  replicas: 9
  {{end}}
  minReadySeconds: 30
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  template:
    metadata:
      labels:
        app: fop-gate
        team: dora
        type: basic
    spec:
      terminationGracePeriodSeconds: 90
      containers:
      - name: fop-gate
        image: reg.qiniu.com/dora-apps/fop-gate:20190218210538-6-master
...........
```

我们都知道， deployment 会为自己创建的 pod 自动加一个 “pod-template-hash” label 来区分，也就是说，每个deployment只管理自己的pod，不会混乱，那么此时endpoint列表中就会有fop-gate和fop-gate-canary的pod，其他服务调用fop-gate的时候就会同时把请求发到这10个pod上。

灰度发布该怎么做呢？  
最佳实践：创建两个不同pipeline，先灰度发布fop-gate-canary的pipeline，再全局发布fop-gate的pipeline(这里给出的是渲染前的配置文件，注意pipeline不同)：

```
  "fop-gate":    "templates":
    - "dora/jjh/fop-gate/configmap.yaml"
    - "dora/jjh/fop-gate/service.yaml"
    - "dora/jjh/fop-gate/deployment.yaml"
    - "dora/jjh/fop-gate/ingress.yaml"
    - "dora/jjh/fop-gate/ingress_debug.yaml"
    - "dora/jjh/fop-gate/log-applog-configmap.yaml"
    - "dora/jjh/fop-gate/log-auditlog-configmap.yaml"
    "pipeline": "569325e6-6d6e-45ca-b21e-24016a9ef326"

  "fop-gate-canary":    "templates":
    - "dora/jjh/fop-gate/configmap.yaml"
    - "dora/jjh/fop-gate/service.yaml"
    - "dora/jjh/fop-gate/deployment.yaml"
    - "dora/jjh/fop-gate/ingress.yaml"
    - "dora/jjh/fop-gate/log-applog-configmap.yaml"
    - "dora/jjh/fop-gate/log-auditlog-configmap.yaml"
    "pipeline": "15f7dd6a-bd01-41bc-bac5-8266d63fc3a5"
```

注意发布的先后顺序：

![4f9cdb9fd8557a05e7d897fedccb7af0.png](https://img-blog.csdnimg.cn/img_convert/4f9cdb9fd8557a05e7d897fedccb7af0.png)

灰度发布完成后，可以登陆pod查看日志，并观察相关的grafana监控，查看TPS2XX和TPS5XX的变化情况，再决定是否继续发布fop-gate，实现灰度发布的目的

```
➜  dora git:(daixuan) ✗ kubectl get pod -o wide | grep fop-gate
fop-gate-685d66768b-5v6q4          2/2     Running   0          15d     172.20.122.161   jjh304    <none>fop-gate-685d66768b-69c6q          2/2     Running   0          4d21h   172.20.129.52    jjh1565   <none>fop-gate-685d66768b-79fhd          2/2     Running   0          15d     172.20.210.227   jjh219    <none>fop-gate-685d66768b-f68zq          2/2     Running   0          15d     172.20.177.98    jjh322    <none>fop-gate-685d66768b-k5l9s          2/2     Running   0          15d     172.20.189.147   jjh1681   <none>fop-gate-685d66768b-m5n55          2/2     Running   0          15d     172.20.73.78     jjh586    <none>fop-gate-685d66768b-rr7t6          2/2     Running   0          15d     172.20.218.225   jjh302    <none>fop-gate-685d66768b-tqvp7          2/2     Running   0          15d     172.20.221.15    jjh592    <none>fop-gate-685d66768b-xnqn7          2/2     Running   0          15d     172.20.133.80    jjh589    <none>fop-gate-canary-7cb6dc676f-62n24   2/2     Running   0          15d     172.20.208.28    jjh574    <none>➜  dora git:(daixuan) ✗ kubectl exec -it fop-gate-canary-7cb6dc676f-62n24 -c fop-gate bash
root@fop-gate-canary-7cb6dc676f-62n24:/# cd app/auditlog/
root@fop-gate-canary-7cb6dc676f-62n24:/app/auditlog# tail -n5 144 | awk -F'\t' '{print $8}'
200
200
200
200
200
```

![0d09a5206286a3bbd22e81f3c6d16c3d.png](https://img-blog.csdnimg.cn/img_convert/0d09a5206286a3bbd22e81f3c6d16c3d.png)

此外，spinnaker具有发布具有pause、resume、undo功能，实际测试可行  
pause 暂停功能(类似于kubectl rollout pause XXX的功能)  
resume恢复功能(类似于kubectl rollout resume XXX的功能)  
undo取消功能(类似于kubectl rollout undo XXX功能)

spinnaker的这几种功能可以在正常发布服务的过程中发现问题，及时暂停和恢复，注意，spinnaker取消发布一定是针对正在发布的操作，pause状态中的发布无法取消，这与kubectl操作一致

我们尝试执行一次，发布，暂停，恢复，取消 操作，整个过程会产生4个version，每次变动会对应一个新version，因为不管是暂停还是恢复，在spinnaker中都将认为是一次新的发布，会更新version版本![d28536c34d1dfd60699a08cefc3b2665.png](https://img-blog.csdnimg.cn/img_convert/d28536c34d1dfd60699a08cefc3b2665.png)![ba6aa8660bb2599fe63ca9f1566f92e0.png](https://img-blog.csdnimg.cn/img_convert/ba6aa8660bb2599fe63ca9f1566f92e0.png)![1ef4d974837565048dbb76c04f8d4bf5.png](https://img-blog.csdnimg.cn/img_convert/1ef4d974837565048dbb76c04f8d4bf5.png)

总结：k8s中灰度发布最好方法就是定义两个不同的deployment管理相同类型的服务，创建不同的pipeline进行发布管理，避免干扰，同时在正常发布过程中，也可以利用spinnaker的pause，resume，undo等功能进行发布控制。

欢迎关注运维自研堂订阅号，运维自研堂是一个技术分享平台，主要是运维自动化开发：linux、python、django、saltstack、tornado、bootstrap、redis、golang、docker、etcd、k8s、ci/cd、devops等经验分享。

-   容器平台自动化CI/CD流水线实操
    
-   谷歌开源 Kubernetes 原生 CI/CD 构建框架 Tekton
    
-   架构师是怎么炼成的
    
-   IPv6时代对业务的挑战
    
-   如何打造一个安全稳定高效的容器云平台
    
-   深入理解无服务器架构(Faas/Serverless)
    
-   CI/CD 场景价值
    
-   [[云原生]]架构及设计原则
    
-   Jira与Zabbix结合
    
-   【Zabbix】告警事件归档与提取
    
-   【HMonitor】Zabbix告警管理平台
    
-   Zabbix 告警收敛
    
-   Zabbix v3.0微信报警及API使用
    
-   zabbix v3.0安装部署及使用
    
-   Web权限设计
    
-   搭建 kubernetes 容器编排平台
    
-   区块链入门教程
    
-   基于Gogs+Drone搭建的私有CI/CD平台
    
-   WEB架构设计心得
    
-   Docker与CI/CD
    
-   【实战篇】Docker的CI/CD流水线实践
    
-   基于 Harbor 搭建 Docker 私有镜像仓库
    
-   利用helm部署应用到kubernetes
    

开源    创新     共享

投稿&商务合作

Mail：idevops168@163.com       QQ:785249378     微信：Idevops001

![017e9ba1d2d6bd41dfe67ff0e466bf3a.gif](https://img-blog.csdnimg.cn/img_convert/017e9ba1d2d6bd41dfe67ff0e466bf3a.gif)

牛人并不可怕，可怕的是牛人比我们还努力！

![65dfe5b7883dce517f0da5fa148dfdcb.png](https://img-blog.csdnimg.cn/img_convert/65dfe5b7883dce517f0da5fa148dfdcb.png)

长按图片，识别加入我们！

版权声明：本文遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。  
原文链接：https://blog.csdn.net/weixin\_29467373/article/details/112338421

## 更多相关推荐

___

© 2022 All rights reserved by 豆奶特 Powered By WordPress