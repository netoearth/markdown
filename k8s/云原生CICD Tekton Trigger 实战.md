Trigger的介绍看 [这里](https://atbug.com/tekton-trigger-glance/).

接上文 [Tekton Pipeline 实战](https://atbug.com/tekton-pipeline-practice/) , 我们为某个项目创建了一个Pipeline, 但是执行时通过 PipelineRun 来完成的. 在 PipelineRun 中我们制定了 Pipepline 以及要使用的 PipelineResource. 但是日常的开发中, 我们更多希望在提交了代码之后开始 Pipeline 的执行. 这时我们就要用到 Tekton Trigger 了.

思路是这样: 代码提交后将`Push Event`发送给`Tekton Trigger EventController`(以下简称 Controller), 然后 Controller 基于的`TriggerBinding`的配置从 payload 中提取信息, 装载在"Params"中作为`TriggerTemplate`的入参. 最后 Controller 创建`PipelineRun`.

## Trigger 相关的资源

### TriggerTemplate

回看[上回用的PipelineRun Yaml](https://atbug.com/tekton-pipeline-practice/#0x06-%E6%89%A7%E8%A1%8C%E6%B5%81%E6%B0%B4%E7%BA%BF), 参数有`revision`, `url`, `imageUrl`和`imageTag`. (imageUrl 与项目名一直)

因此定义`TriggerTemplate`将这 4 个元素作为入参, 然后复用之前的`Pipeline`.

```
apiVersion: tekton.dev/v1alpha1
kind: TriggerTemplate
metadata:
  name: trigger-test-triggertemplate
  namespace: tekton-pipelines
spec:
  params:
    - name: gitrevision
      description: The git revision
      default: master
    - name: gitrepositoryurl
      description: The git repository url
    - name: namespace
      description: The namespace to create the resources
      default: tekton-pielines
    - name: projectname
      description: The project name
    - name: imagetag
      description: The image tag
      default: latest
  resourcetemplates:
    - apiVersion: tekton.dev/v1alpha1
      kind: PipelineRun
      metadata:
        name: tekton-test-pipeline-run-$(uid)
        namespace: $(params.namespace)
      spec:
        serviceAccountName: tekton-test
        params: 
          - name: imageUrl
            value: addozhang/$(params.projectname)
          - name: imageTag
            value: $(params.imagetag)
        pipelineRef:
          name: build-pipeline
        resources:
          - name: git-source
            resourceSpec:
              type: git
              params:
                - name: revision
                  value: $(params.gitrevision)
                - name: url
                  value: $(params.gitrepositoryurl)
```

执行`kubectl apply -f trigger/trigger-template.yaml`

### TriggerBinding

`TriggerTemplate`的入参都可以从`PushEvent`的 payload中通过 JsonPath 表达式来提取. 更多表达式的使用参考[官方的文档](https://github.com/tektoncd/triggers/blob/master/docs/triggerbindings.md#event-variable-interpolation).

_注意: 这里的 payload 格式使用的是 Gitlab 的 PushEvent 定义. 为什么要用 Gitlab 后什么会解释_

```
apiVersion: tekton.dev/v1alpha1
kind: TriggerBinding
metadata:
  name: trigger-test-triggerbinding
  namespace: tekton-pipelines
spec:
  params:
    - name: gitrevision
      value: $(body.after)
    - name: namespace
      value: tekton-pipelines
    - name: gitrepositoryurl
      value: $(body.project.git_http_url)
    - name: projectname
      value: $(body.project.name)
```

执行`kubectl apply -f trigger/trigger-binding.yaml`

### EventListener

`EventController`将`TriggerTemplate`和`TriggerBinding`关联在一起, 资源在创建后会自动创建对应的 pod 和 service.

```
apiVersion: tekton.dev/v1alpha1
kind: EventListener
metadata:
  name: trigger-test-eventlistener
  namespace: tekton-pipelines
spec:
  serviceAccountName: tekton-test //需要访问 API, 用上回创建 serviceaccount
  triggers:
    - bindings:
        - name: trigger-test-triggerbinding
      template:
        name: trigger-test-triggertemplate
```

执行`kubectl apply -f trigger/event-listener.yaml`

提供 WebHook 的访问入口, 为`EventListener`创建`ingress`

```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: el-trigger-test-eventlistener
  namespace: tekton-pipelines
spec:
  rules:
  - host: trigger-test.el.nip.io
    http:
      paths:
      - backend:
          serviceName: el-trigger-test-eventlistener
          servicePort: 8080
```

做完之后记得添加解析`sudo echo "$(minikube ip) trigger-test.el.nip.io" >> /etc/hosts`

## GitLab

这里我们没有继续用 github 作为 CVS, minikube 运行在本地, ingress 地址无法在 github 上解析(webhook 无法工作). 因此只能本地搭建个 gitlab.

使用 Docker Compose 的方式部署, 并制定域名`gitlab.nip.io`.

```
version: '3.6'
services:
  web:
    image: 'gitlab/gitlab-ce:latest'
    restart: always
    hostname: 'gitlab.nip.io'
    environment:
      GITLAB_OMNIBUS_CONFIG: |
       external_url 'http://gitlab.nip.io:8088'
        gitlab_rails['gitlab_shell_ssh_port'] = 8022
        # Add any other gitlab.rb configuration here, each on its own line       
    ports:
      - '8088:8088'
      - '8022:22'
```

记得添加解析

```
sudo echo "$(ifconfig | grep -Eo 'inet (addr:)?([0-9]*\.){3}[0-9]*' | grep -Eo '([0-9]*\.){3}[0-9]*' | grep -v '127.0.0.1' | head -n 1)    gitlab.nip.io" >> /etc/hosts
```

GitLab 的默认账号是`root`, 首次访问`http://gitlab.nip.io:8088`的时候会提示设置密码.

创建项目, 提交代码的操作不多说了. 添加 webhook 前, 要先去"Admin Area » Settings » Network » Outbound Requests"勾选`Allow requests to the local network from web hooks and services`.

### DNS 解析问题

Docker 和 minikube 是在两个独立的虚拟机中运行, 并且我们添加了两个地址解析. 互相访问的时候便会出现 domain 无法解析的问题, 这里是通过部署一个 dnsmasq 作为独立的 dns 解析服务器.

安装和配置 dnsmasq, dnsmasq 会将 `/etc/hosts` 中的记录加入到记录中.

我本地的 minikube 位于`192.168.64.0`网段, 因次 dnsmasq 添加监听地址`192.168.64.1`

```
brew install dnsmasq

cat << EOF >> /usr/local/etc/dnsmasq.conf
strict-order
listen-address=127.0.0.1,192.168.64.1
EOF

sudo brew services start dnsmasq
```

检查一下, `192.168.1.136`是我本机 ip.

```
$ dig gitlab.nip.io @192.168.64.1

; <<>> DiG 9.10.6 <<>> gitlab.nip.io @192.168.64.1
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 63393
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;gitlab.nip.io.INA

;; ANSWER SECTION:
gitlab.nip.io.0INA192.168.1.136

;; Query time: 0 msec
;; SERVER: 192.168.64.1#53(192.168.64.1)
;; WHEN: Thu Feb 13 18:20:19 CST 2020
;; MSG SIZE  rcvd: 58
```

## 测试

推送一个空的提交到代码仓库: `git commit -a -m "trigger commit" --allow-empty && git push origin master`

然后就可以看到新的`PipelineRun`创建并运行.

```
$ tkn pr list
NAME                             STARTED          DURATION    STATUS
tekton-test-pipeline-run-fk2xj   42 seconds ago   ---         Running
```

## 其他文章

Tekton Pipeline 实战: [https://atbug.com/tekton-pipeline-practice](https://atbug.com/tekton-pipeline-practice)