## 

简单使用gogs+drone搭建ci/cd自动化部署

 [![](https://cdn.learnku.com/uploads/avatars/21295_1648867994.jpg!/both/100x100) 晴空 的个人博客](https://learnku.com/blog/fivenull) / 198 / 0 / 创建于 4个月前 / 更新于 4个月前

日常的开发工作中，如果没有专业的运维工程师，我们的 CI/CD 工作会非常的困难，尤其是二三线城市，很多公司的技术团队是没有全职的运维工程师，这时候，掌握代码提交和自动化代码部署的能力，就会显得更有竞争力。  
首先，什么是 CI/CD  
**CI : 持续集成**  
通俗一点说，就是把代码提交到代码托管平台  
**CD : 持续交付 + 持续部署**  
简单的说，就是自动化轻松的部署到生产坏境，且全程自动更新代码

具体实现，需要借助 docker，使用 `docker-compose` 来编排容器  
新建 docker-compose.yml 文件  
以下内容可以拿来直接使用，需要确保 3000、10022、8080 三个端口没有占用，且外网可以访问，且创建 drone 同名数据库数据表，  
帐号密码为 drone 和 b7RLreDWrycdnBA3  
如想修改，修改 `DRONE_DATABASE_DATASOURCE` 配置项

```
version: '3'
services:
  gogs:
    image: gogs/gogs:0.12.3
    container_name: gogs
    ports:
      # 安装配置gogs需要访问3000的端口
      - "3000:3000"
      - "10022:22"
    volumes:
      - /data/gogs:/data
    restart: always

  drone-server:
    image: drone/drone:latest
    container_name: drone-server
    ports:
      # 访问drone的时候需要访问的8080的端口
      - "8080:80"
      - "8443:443"
    volumes:
      # 挂载外部的目录
      - /data/drone:/var/lib/drone/
      - /var/run/docker.sock:/var/run/docker.sock
    restart: always
    environment:
      - DRONE_DEBUG=true
      # 启动日志，默认是关闭的
      - DRONE_LOGS_TRACE=true
      # 启动 debug 日志，默认是关闭的
      - DRONE_LOGS_DEBUG=true
      - DRONE_OPEN=true
      # 设置 drone-server 使用的 host 名称，可以是 ip 地址加端口号；容器中可以使用容器名称代替
      - DRONE_SERVER_HOST=drone-server
      - DRONE_GIT_ALWAYS_AUTH=false
      # 开启 gogs
      - DRONE_GOGS=true
      - DRONE_GOGS_SKIP_VERIFY=false
      # gogs 服务地址，使用容器名 + 端口号---注意点这里需有前面的HTTP
      - DRONE_GOGS_SERVER=http://gogs:3000
      # drone 的提供者，本项目中为 gogs服务
      - DRONE_PROVIDER=gogs
      # 配置 drone 数据库
      - DRONE_DATABASE_DRIVER=mysql
      # 配置 drone 数据库文件
      - DRONE_DATABASE_DATASOURCE=drone:b7RLreDWrycdnBA3@tcp(172.17.0.1:3306)/drone?parseTime=true
      # 协议，可选 http、https
      - DRONE_SERVER_PROTO=http
      # 秘钥信息设置，主要是用在 drone-server 与 drone-agent 之间的 RPC 请求
      - DRONE_RPC_SECRET=secret
      # 秘钥信息设置，主要是用在 drone-server 与 drone-agent 直接的请求
      - DRONE_SECRET=secret

  drone-agent:
    image: drone/agent:latest
    container_name: drone-agent
    depends_on:
      - drone-server
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    restart: always
    environment:
      - DRONE_DEBUG=true
      # 开始日志，默认是关闭的
      - DRONE_LOGS_TRACE=true
      # 启动 debug 日志，默认是关闭的
      - DRONE_LOGS_DEBUG=true
      # 设置 drone-server 使用的 host 名称，可以是 ip 地址加端口号；容器中可以使用容器名称代替
      - DRONE_RPC_SERVER=http://drone-server
      # 秘钥，用于 drone-server 与 drone-agent 之间的 RPC 请求
      - DRONE_RPC_SECRET=secret
      - DRONE_SERVER=drone-server:9000
      # 秘钥，用于 drone-server 与 drone-agent 直接的请求
      - DRONE_SECRET=secret
      - DRONE_MAX_PROCS=5
      - DROCK_HOST=tcp://127.0.0.1:2375
```

运行编排文件

可以使用 `Portainer` 来可视化管理 docker  
下载镜像

```
docker pull portainer/portainer
```

运行

```
docker run -d -p 9000:9000 --restart=always -v /var/run/docker.sock:/var/run/docker.sock --name prtainer portainer/portainer
```

首次进入 portainer 管理界面需要设置登录账号密码，进入后按提示，进入容器管理界面，访问地址为`服务器IP:9000`，如下图

![简单使用gogs+drone搭建ci/cd自动化部署](https://cdn.learnku.com/uploads/images/202203/18/21295/NwPCqLqh4a.png!large)  
表示已经运行成功

**运行 gogs**  
访问地址为`服务器IP:3000`

![简单使用gogs+drone搭建ci/cd自动化部署](https://cdn.learnku.com/uploads/images/202203/18/21295/EDBZFPODCO.png!large)

![简单使用gogs+drone搭建ci/cd自动化部署](https://cdn.learnku.com/uploads/images/202203/18/21295/6W8840GJfp.png!large)

在用户设置 - SSH 秘钥里上传电脑秘钥

![简单使用gogs+drone搭建ci/cd自动化部署](https://cdn.learnku.com/uploads/images/202203/18/21295/kVb04rG6u6.png!large)

之后找一个 vue 项目上传上去

![简单使用gogs+drone搭建ci/cd自动化部署](https://cdn.learnku.com/uploads/images/202203/18/21295/8UFXGo4vST.png!large)

**设置 drone**  
访问 `服务器IP:8080` 打开 drone 界面

![简单使用gogs+drone搭建ci/cd自动化部署](https://cdn.learnku.com/uploads/images/202203/18/21295/8eJPbyaQSI.png!large)

如果打不开，去查看容器日志，解决问题

![简单使用gogs+drone搭建ci/cd自动化部署](https://cdn.learnku.com/uploads/images/202203/18/21295/mxJOWmiyM6.png!large)

输入 gogs 里设置的管理员帐号密码

![简单使用gogs+drone搭建ci/cd自动化部署](https://cdn.learnku.com/uploads/images/202203/18/21295/Wgy2L43WrD.png!large)

drone 会自动同步 gogs 里的仓库

![简单使用gogs+drone搭建ci/cd自动化部署](https://cdn.learnku.com/uploads/images/202203/18/21295/OXeaGQJakE.png!large)

![简单使用gogs+drone搭建ci/cd自动化部署](https://cdn.learnku.com/uploads/images/202203/18/21295/q4MFzgzzqX.png!large)  
点击，激活仓库

![简单使用gogs+drone搭建ci/cd自动化部署](https://cdn.learnku.com/uploads/images/202203/18/21295/edOJVyrHWd.png!large)  
这个时候，我们回到 gogs 里，在仓库设置 - 管理 web 钩子中，就会看到 gogs 已经自动帮我们设置好了

![简单使用gogs+drone搭建ci/cd自动化部署](https://cdn.learnku.com/uploads/images/202203/18/21295/nLMSJ0aByp.png!large)  
点击编辑，进入查看详情

![简单使用gogs+drone搭建ci/cd自动化部署](https://cdn.learnku.com/uploads/images/202203/18/21295/r2Elta4lfv.png!large)

**编写.drone.yml 文件**  
需要注意两点  
1、后缀为 yml  
2、文件是隐藏文件，所以是。开头的  
在项目根目录，创建`.drone.yml` 文件

```
kind: pipeline
type: docker
name: build
steps:
- name: 编译文件
  image: node:10.16
  pull: if-not-exists # always never
  commands:
    - node -v
    - npm -v
    - yarn --version
    - yarn config set cache-folder .yarn-cache
    - yarn install
    - yarn run build

- name: 同步文件
  image: drillster/drone-rsync
  settings:
    user: root
    key:
      from_secret: ssh_key
    hosts:
      - 172.17.0.1
    # 来源项目目录
    source: ./dist/*
    # 目标服务器目录
    target: /www/wwwroot/www
    script:
      - cd /www/wwwroot/www
      - ls

 - name: 钉钉推送
   pull: if-not-exists # always never
   image: guoxudongdocker/drone-dingtalk
   settings:
     token:
       from_secret: dingding_token
     type: markdown
     message_color: true
     message_pic: true
     sha_link: true
   when:
     status: [failure, success]
```

需要注意的是，这里用到了 drone 配置的 secret

![简单使用gogs+drone搭建ci/cd自动化部署](https://cdn.learnku.com/uploads/images/202203/19/21295/lTp7oFAtqa.png!large)

特别注意的是  
ssh\_key 的值，是 **秘钥**，不是公钥  
是宿主机的 `/root/.ssh/id_rsa` 文件里的所有内容  
需要全部复制进去  
之后，把宿主机公钥的内容，复制到宿主机的 authorized\_keys 文件里  
这两步很重要

之后，`git push` 提交，将触发构建任务  
（钉钉推送这里被我注释了代码，所以看不到构建任务）  
![简单使用gogs+drone搭建ci/cd自动化部署](https://cdn.learnku.com/uploads/images/202203/19/21295/yCVzvn6c4g.png!large)

之后等待即可，完成，收工

> 本作品采用[《CC 协议》](https://learnku.com/docs/guide/cc4.0/6589)，转载必须注明作者和本文链接