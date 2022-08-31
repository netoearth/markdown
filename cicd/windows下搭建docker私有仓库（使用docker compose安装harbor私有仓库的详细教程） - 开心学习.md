## windows下搭建docker私有仓库

**使用docker compose安装harbor私有仓库的详细教程****概述**

　　harbor是什么呢？英文单词的意思是：港湾。港湾用来存放集装箱(货物的)，而docker的由来正是借鉴了集装箱的原理，所以harbor是用于存放docker的镜像，作为镜像仓库使用。官方的说法是：Harbor是一个用于存储和分发Docker镜像的企业级Registry服务器。

　　harbor镜像仓库是由VMware开源的一款企业级镜像仓库，它包括权限管理(RBAC)、LDAP、日志审核、管理界面、自我注册、镜像复制等诸多功能。

**一、harbor特性**

1、基于角色的访问控制：用户和存储库是通过“项目”组织的，用户可以对多个镜像仓库统一命名空间拥有不同的权限。  
2、镜像复制：可以基于具有多个Registry实例之间复制（同步）图像和图表。如果出现任何错误，Harbor会自动重试复制。非常适合于负载平衡、高可用性、多数据中心、混合和多云场景。  
3、LDAP/AD支持：Harbor与现有企业LDAP/AD集成，用于用户身份验证和管理，并支持将LDAP组导入Harbor并为其分配适当的项目角色。  
镜像删除和垃圾收集：镜像可以删除，其空间可以回收。  
4、国际化：对多国语言支持（已拥有中文、英文、德文、日语和俄文）；  
5、图形化用户界面：用户可以轻松浏览、搜索存储库和管理项目。  
6、审计管理：跟踪到存储库的所有操作。  
7、RESTful API：用于大多数管理操作的RESTful API，易于与外部系统集成。一个嵌入式的Swagger用户界面可用于探索和测试API。  
简单部署：提供在线和离线安装程序。此外，可以安装到vSphere平台的(OVA方式)虚拟设备。

**二、Harbor 组件**

1、proxy：Harbor的组件，如注册表、UI和令牌服务，都位于反向代理之后。代理将来自浏览器和Docker客户机的请求转发到各种后端服务。  
2、Registry：负责存储Docker镜像和处理Docker推/拉命令。由于Harbor需要对映像进行访问权限控制，Registry将引导客户机访问令牌服务，以便为每个pull或push请求获取有效的令牌（token）。  
3、Core Service：Harbor的核心功能，主要提供以下服务：  
1）UI：提供图像化的图形用户界面，帮助人户管理镜像和对用户授权。  
2）webhook: 为了及时获取registry上images的状态变化的情况，在Registry上配置webhook，把状态变化传递UI模块；  
3）Token令牌服务：负责根据用户在项目中的角色为每个docker push/pull命令颁发令牌。如果从Docker客户机发送的请求中没有令牌，注册表将把请求重定向到令牌服务。  
4、Datebase：为了给core services提供数据库舒服，负责储存用户权限、审计日志、Docker image分组信息等数据。  
5、Job Services：提供镜像远程负责功能，能把本地镜像同步到其他harbor实例当中。  
6、Log Collector：为了帮助监控Harbor运行，负责手机其他组件的log，供日后分析。

![windows下搭建docker私有仓库（使用docker compose安装harbor私有仓库的详细教程）](http://image.studyofnet.com/1/0128/0401-28649.jpg)

**三、部署环境**

-   ```
    centos-7.6   192.168.8.130
    Docker version 1.19.3
    docker-compose version 1.24.2
    harbor-offline-installer-v1.8.6.tgz
    
    ```
    

**四、安装docker-compose**

-   ```
    方式1:
    [root@centos130 ~]# curl -L https://github.com/docker/compose/releases/download/1.24.1/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
    [root@centos130 ~]# chmod +x /usr/local/bin/docker-compose
    方式2:
    [root@centos130 ~]# wget https://bootstrap.pypa.io/get-pip.py
    [root@centos130 ~]# python get-pip.py
    [root@centos130 ~]# pip install docker-compose
    
    ```
    

**五、卸载docker-compose**

-   ```
    #二进制：
    [root@centos130 ~]rm  /usr/local/bin/docker-compose
    #pip:
    [root@centos130 ~]pip uninstall  docker-compose
    
    ```
    

**六、安装docker**

-   ```
    [root@centos130 ~]# yum install -y yum-utils device-mapper-persistent-data lvm2
    添加一个稳定的源
    [root@centos130 ~]# yum-config-manager --add-repo \
        http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
    [root@centos130 ~]# yum makecache fast
    安装最新稳定版本的docker-ce
    [root@centos130 ~]# yum install -y docker-ce docker-ce-cli containerd.io vim
    [root@centos130 ~]# mkdir /etc/docker && vim /etc/docker/daemon.json
    {
      "registry-mirrors": ["https://yxrgrke0.mirror.aliyuncs.com"],
      "insecure-registries": ["192.168.8.130:5000"],
      "insecure-registries": ["centos130:80"],
      "log-driver": "json-file",
      "log-opts": {
        "max-size": "100m",
        "max-file": "3"
        }
    }
    启动docker
    [root@centos130 ~]# systemctl daemon-reload
    [root@centos130 ~]# systemctl enable docker && systemctl start docker
    [root@centos130 ~]# systemctl status docker
    
    ```
    

**七、安装harbor**

-   ```
    harbor下载地址:
    harbor github 地址 https://github.com/goharbor/harbor
    http://harbor.orientsoft.cn/
    [root@centos130 ~]# wget https://storage.googleapis.com/harbor-releases/release-1.8.6/harbor-offline-installer-v1.8.6.tgz
    [root@centos130 ~]# tar -xf harbor-offline-installer-v1.8.6.tgz -C /usr/local/
    [root@centos130 ~]# cd /usr/local/harbor/
    [root@centos130 ~]# vim harbor.yml
    hostname = centos130
    #这里只是简单的测试，所以只编辑这一行，其他的默认不做修改
    #禁止用户注册
    self_registration = off
    #设置只有管理员可以创建项目
    project_creation_restriction = adminonly
    
    ```
    

**八、执行安装脚本**

-   ```
    [root@centos130 ~]# ./instsll.sh  
    说明：安装报错 找不到docker-proxy 、 docker-runc
    执行
    [root@centos130 ~]# ln -s /usr/libexec/docker/docker-runc-current /usr/bin/docker-runc
    [root@centos130 ~]# ln -s /usr/libexec/docker/docker-proxy-current /usr/bin/docker-proxy
    查看启动的镜像文件
    [root@centos130 ~]# docker-compose ps
    Harbor容器的stop与start：
    [root@centos130 ~]# cd /usr/local/harbor/
    [root@centos130 ~]# docker-compose stop
    [root@centos130 ~]# docker-compose start
    
    ```
    

**九、登录harbor**

到此便安装完成了，直接打开浏览器登陆，并创建my项目：  
默认用户密码是：admin/Harbor12345  

![windows下搭建docker私有仓库（使用docker compose安装harbor私有仓库的详细教程）](http://image.studyofnet.com/1/0128/0401-28650.jpg)  

Shell命令行终端登录harbor仓库

-   ```
    [root@centos130 ~]# harbor上传镜像
    [root@centos130 ~]# docker login centos130:80
    [root@centos130 ~]# docker login -u admin -p Harbor12345 centos130:80  #账号密码： admin/Harbor12345
    Username: admin
    Password: 
    Login Succeeded
    
    ```
    

测试镜像上传

-   ```
    [root@centos130 ~]# docker pull nginx
    [root@centos130 ~]# docker tag nginx:latest centos130:80/my/nginx:latest
    [root@centos130 ~]# docker images
    [root@centos130 ~]# docker push centos130:80/my/nginx:latest
    The push refers to repository [centos130:80/my/nginx]
    55a77731ed26: Pushed 
    71f2244bc14d: Pushed 
    f2cb0ecef392: Pushed 
    latest: digest: sha256:3936fb3946790d711a68c58be93628e43cbca72439079e16d154b5db216b58da size: 948
    
    说明： 格式为: userip/项目名/image名字:版本号   （项目名需要在webui 提前建好）
    [root@centos130 ~]# docker images
    REPOSITORY                       TAG                        IMAGE ID            CREATED             SIZE
    centos130:80/my/nginx:latest     latest                     5a3221f0137b        5 days ago          126MB
    nginx                            latest                     5a3221f0137b        5 days ago          126MB
    删除本地nginx镜像，测试下载
    [root@centos130 ~]# docker pull centos130:80/my/nginx:latest
    
    ```
    

**十、harbor修改端口号**

1、修改docker-compose.yml文件映射为1180端口：

-   ```
    修改配置文件
    [root@centos130 ~]# cat /usr/local/harbor/docker-compose.yml
    
    version: '2.3'
    services:
      log:
        image: goharbor/harbor-log:v1.8.6
        container_name: harbor-log
        restart: always
        dns_search: .
        cap_drop:
          - ALL
        cap_add:
          - CHOWN
          - DAC_OVERRIDE
          - SETGID
          - SETUID
        volumes:
          - /var/log/harbor/:/var/log/docker/:z
          - ./common/config/log/:/etc/logrotate.d/:z
        ports:
          - 127.0.0.1:1514:10514
        networks:
          - harbor
      registry:
        image: goharbor/registry-photon:v2.7.1-patch-2819-v1.8.6
        container_name: registry
        restart: always
        cap_drop:
          - ALL
        cap_add:
          - CHOWN
          - SETGID
          - SETUID
        volumes:
          - /data/registry:/storage:z
          - ./common/config/registry/:/etc/registry/:z
          - type: bind
            source: /data/secret/registry/root.crt
            target: /etc/registry/root.crt
        networks:
          - harbor
        dns_search: .
        depends_on:
          - log
        logging:
          driver: "syslog"
          options:  
            syslog-address: "tcp://127.0.0.1:1514"
            tag: "registry"
      registryctl:
        image: goharbor/harbor-registryctl:v1.8.6
        container_name: registryctl
        env_file:
          - ./common/config/registryctl/env
        restart: always
        cap_drop:
          - ALL
        cap_add:
          - CHOWN
          - SETGID
          - SETUID
        volumes:
          - /data/registry:/storage:z
          - ./common/config/registry/:/etc/registry/:z
          - type: bind
            source: ./common/config/registryctl/config.yml
            target: /etc/registryctl/config.yml
        networks:
          - harbor
        dns_search: .
        depends_on:
          - log
        logging:
          driver: "syslog"
          options:  
            syslog-address: "tcp://127.0.0.1:1514"
            tag: "registryctl"
      postgresql:
        image: goharbor/harbor-db:v1.8.6
        container_name: harbor-db
        restart: always
        cap_drop:
          - ALL
        cap_add:
          - CHOWN
          - DAC_OVERRIDE
          - SETGID
          - SETUID
        volumes:
          - /data/database:/var/lib/postgresql/data:z
        networks:
          harbor:
        dns_search: .
        env_file:
          - ./common/config/db/env
        depends_on:
          - log
        logging:
          driver: "syslog"
          options:  
            syslog-address: "tcp://127.0.0.1:1514"
            tag: "postgresql"
      core:
        image: goharbor/harbor-core:v1.8.6
        container_name: harbor-core
        env_file:
          - ./common/config/core/env
        restart: always
        cap_drop:
          - ALL
        cap_add:
          - SETGID
          - SETUID
        volumes:
          - /data/ca_download/:/etc/core/ca/:z
          - /data/psc/:/etc/core/token/:z
          - /data/:/data/:z
          - ./common/config/core/certificates/:/etc/core/certificates/:z
          - type: bind
            source: ./common/config/core/app.conf
            target: /etc/core/app.conf
          - type: bind
            source: /data/secret/core/private_key.pem
            target: /etc/core/private_key.pem
          - type: bind
            source: /data/secret/keys/secretkey
            target: /etc/core/key
        networks:
          harbor:
        dns_search: .
        depends_on:
          - log
          - registry
        logging:
          driver: "syslog"
          options:  
            syslog-address: "tcp://127.0.0.1:1514"
            tag: "core"
      portal:
        image: goharbor/harbor-portal:v1.8.6
        container_name: harbor-portal
        restart: always
        cap_drop:
          - ALL
        cap_add:
          - CHOWN
          - SETGID
          - SETUID
          - NET_BIND_SERVICE
        networks:
          - harbor
        dns_search: .
        depends_on:
          - log
          - core
        logging:
          driver: "syslog"
          options:
            syslog-address: "tcp://127.0.0.1:1514"
            tag: "portal"
    
      jobservice:
        image: goharbor/harbor-jobservice:v1.8.6
        container_name: harbor-jobservice
        env_file:
          - ./common/config/jobservice/env
        restart: always
        cap_drop:
          - ALL
        cap_add:
          - CHOWN
          - SETGID
          - SETUID
        volumes:
          - /data/job_logs:/var/log/jobs:z
          - type: bind
            source: ./common/config/jobservice/config.yml
            target: /etc/jobservice/config.yml
        networks:
          - harbor
        dns_search: .
        depends_on:
          - redis
          - core
        logging:
          driver: "syslog"
          options:
            syslog-address: "tcp://127.0.0.1:1514"
            tag: "jobservice"
      redis:
        image: goharbor/redis-photon:v1.8.6
        container_name: redis
        restart: always
        cap_drop:
          - ALL
        cap_add:
          - CHOWN
          - SETGID
          - SETUID
        volumes:
          - /data/redis:/var/lib/redis
        networks:
          harbor:
        dns_search: .
        depends_on:
          - log
        logging:
          driver: "syslog"
          options:
            syslog-address: "tcp://127.0.0.1:1514"
            tag: "redis"
      proxy:
        image: goharbor/nginx-photon:v1.8.6
        container_name: nginx
        restart: always
        cap_drop:
          - ALL
        cap_add:
          - CHOWN
          - SETGID
          - SETUID
          - NET_BIND_SERVICE
        volumes:
          - ./common/config/nginx:/etc/nginx:z
        networks:
          - harbor
        dns_search: .
        ports:
          - 1180:80
          - 443:443
          - 4443:4443
        depends_on:
          - postgresql
          - registry
          - core
          - portal
          - log
        logging:
          driver: "syslog"
          options:  
            syslog-address: "tcp://127.0.0.1:1514"
            tag: "proxy"
    networks:
      harbor:
        external: false
    
    ```
    

2、修改/etc/docker/daemon.json文件将80修改为1180端口：

-   ```
    修改daemon配置
    [root@centos130 ~]# cat /etc/docker/daemon.json 
    
    {
      "registry-mirrors": ["https://yxrgrke0.mirror.aliyuncs.com"],
      "insecure-registries": ["192.168.8.130:5000"],
      "insecure-registries": ["centos130:1180"],
      "log-driver": "json-file",
      "log-opts": {
        "max-size": "100m",
        "max-file": "3"
        }
    }
    
    ```
    

3、修改/usr/locat/harbor/harbor.yml文件的hostname

-   ```
    修改hostname配置
    cat /usr/locat/harbor/harbor.yml
    hostname: centos130:1180
    
    ```
    

4、停止harbor，重新启动并生成配置文件

-   ```
    重新初始化
    [root@centos130 ~]# cd /usr/locat/harbor/
    [root@centos130 ~]# docker-compose stop
    [root@centos130 ~]# ./install.sh
    
    ```
    

5、重新启动docker

-   ```
    [root@centos130 ~]# systemctl daemon-reload
    [root@centos130 ~]# systemctl restart docker.service
    验证
    [root@centos130 ~]# docker login centos130:1180
    Username: admin
    Password: Harbor12345
    
    ```
    

到此这篇关于使用docker-compose安装harbor的文章就介绍到这了,更多相关docker compose安装harbor内容请搜索开心学习网以前的文章或继续浏览下面的相关文章希望大家以后多多支持开心学习网！