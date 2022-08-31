**点击上方“IT那活儿****”公众号，关注后了解更多内容，不管IT什么活儿，干就完了！！！**

## **harbor概述**

容器技术越来越或火，越成熟，容器应用的开发和运行始终离不开可靠的镜像管理，虽然Docker官方也提供了公共的镜像仓库，但是从安全和效率等方面考虑，部署我们私有环境的Registry也是非常必要的。 

所以Harbor孕育而生，**Harbor是由VMware公司开源的企业级的Docker Registry管理项目**，它包括权限管理(RBAC)、LDAP、日志审核、管理界面、自我注册、镜像复制和中文支持等功能。

## **Harbor的主要功能**

-   **基于角色的访问控制：**用户与Docker镜像仓库通过“项目”进行组织管理，一个用户可以对多个镜像仓库在同一命名空间（project）里有不同的权限。  
    
-   **基于镜像的复制策略：**镜像可以在多个Registry实例中复制（可以将仓库中的镜像同步到远程的Harbor，类似于MySQL主从同步功能），尤其适合于负载均衡，高可用，混合云和多云的场景。
    
-   **图形化用户界面：**用户可以通过浏览器来浏览，检索当前Docker镜像仓库，管理项目和命名空间。
    
-   **审计管理：**所有针对镜像仓库的操作都可以被记录追溯，用于审计管理。
    
-   **部署简单：** 提供在线和离线两种安装工具， 也可以安装到vSphere平台虚拟设备。
    

## **harbor搭建**

**1\. 搭建准备**  

-   Linux CentOS 7
    
-   Docker（20.10.9）下载地址：https://download.docker.com/linux/static/stable/x86\_64/
    
-   Docker-compose（v2.6.0）  
    
    https://github.com/docker/compose/releases
    
-   Harbor（1.10.10）  
    
    https://github.com/goharbor/harbor/releases
    
-   **服务配置**
    
    最低要求：CPU2核/内存4G/硬盘40G；
    
    推荐：CPU4核/内存8G/硬盘300G。
    

**2\. 开始搭建**

**2.1 docker安装**

将安装包文件上传到服务器目录并解压。

![](https://oss-emcsprod-public.modb.pro/wechatSpider/modb_20220802_635ba238-1201-11ed-ac0b-38f9d3cd240d.png)

将解压出来的docker文件内容移动到 usr/bin/ 目录下。

![](https://oss-emcsprod-public.modb.pro/wechatSpider/modb_20220802_6363adde-1201-11ed-ac0b-38f9d3cd240d.png)

将docker注册为service，在/etc/systemd/system目录下创建docker.service文件，并配置如下内容保存。

![](https://oss-emcsprod-public.modb.pro/wechatSpider/modb_20220802_6369d9f2-1201-11ed-ac0b-38f9d3cd240d.png)

添加文件权限并启动docker，执行如下命令。

![](https://oss-emcsprod-public.modb.pro/wechatSpider/modb_20220802_6370d32e-1201-11ed-ac0b-38f9d3cd240d.png)

安装成功，查看版本。

![](https://oss-emcsprod-public.modb.pro/wechatSpider/modb_20220802_637bb050-1201-11ed-ac0b-38f9d3cd240d.png)

**2.2 docker-compose安装**

上传docker-compose并复制到/usr/bin/目录下，并给执行权限。

![](https://oss-emcsprod-public.modb.pro/wechatSpider/modb_20220802_6387d042-1201-11ed-ac0b-38f9d3cd240d.png)

安装成功，查看版本。

![](https://oss-emcsprod-public.modb.pro/wechatSpider/modb_20220802_6391a130-1201-11ed-ac0b-38f9d3cd240d.png)

**2.3 harbor安装**

上传harbor压缩包，并解压。

![](https://oss-emcsprod-public.modb.pro/wechatSpider/modb_20220802_639a0ac8-1201-11ed-ac0b-38f9d3cd240d.png)

进入harbor目录给予以下文件执行权限：

![](https://oss-emcsprod-public.modb.pro/wechatSpider/modb_20220802_63a16566-1201-11ed-ac0b-38f9d3cd240d.png)

修改harbor.yml文件。

**注：以实际行数为准，只供参考**。

-   修改第五行 hostname 为主机ip或域名；
    
-   注释第13行 https；
    
-   注释第15行 port: 443；
    
-   修改第34行 harbor\_admin\_password，设置一个admin用户的密码；
    
-   修改第47行 data\_volume，修改harbor的数据目录；
    
-   修改第120行 location，修改harbor的日志目录。
    

启动harbor，执行harbor目录下的install.sh文件：

![](https://oss-emcsprod-public.modb.pro/wechatSpider/modb_20220802_63aeef74-1201-11ed-ac0b-38f9d3cd240d.png)

验证是否成功。

![](https://oss-emcsprod-public.modb.pro/wechatSpider/modb_20220802_63b84114-1201-11ed-ac0b-38f9d3cd240d.png)

在/etc/docker/下创建daemon.json文件，并配置阿里镜像加速以及修改harbor地址为可信任源。

![](https://oss-emcsprod-public.modb.pro/wechatSpider/modb_20220802_63c5168c-1201-11ed-ac0b-38f9d3cd240d.png)

通过浏览器搜索在harbor.yml里设置的hostsname进行访问。

![](https://oss-emcsprod-public.modb.pro/wechatSpider/modb_20220802_63cc90b0-1201-11ed-ac0b-38f9d3cd240d.png)

默认密码为：admin/Harbor12345。

## **harbor维护**

<table><tbody><tr><td><section><strong><span>容器</span></strong></section></td><td><section><strong><span>功能</span></strong></section></td></tr><tr><td><section><span>harbor-core</span></section></td><td><section><span>配置管理中心</span></section></td></tr><tr><td><section><span>harbor-db</span></section></td><td><section><span>PG数据库</span></section></td></tr><tr><td><section><span>harbor-jobservice</span></section></td><td><section><span>负责镜像复制</span></section></td></tr><tr><td><section><span>harbor-log</span></section></td><td><section><span>记录操作日志</span></section></td></tr><tr><td><section><span>harbor-portal</span></section></td><td><section><span>Web管理页面和API</span></section></td></tr><tr><td><section><span>nginx</span></section></td><td><section><span>前端代理，负责前端页面和镜像上传/下载转发</span></section></td></tr><tr><td><section><span>redis</span></section></td><td><section><span>会话</span></section></td></tr><tr><td><section><span>registryctl</span></section></td><td><section><span>镜像存储</span></section></td></tr></tbody></table>

-   容器数据持久化目录（可更改）：/data
    
-   日志文件目录（可更改）：/var/log/harbor
    
-   数据库做好定期备份。
    

## **配置harbor主从复制**

**1\. 主备**  

-   简单，主挂了切到备Harbor；
    
-   同一时间只有一台提供服务；
    
-   适合少量镜像下载。
    

**2\. 双主**

-   双向配置复制；
    
-   两台同时提供服务；
    
-   前面增加负载均衡器。
    

**3\. 一主多从**

-   多个从同步主；
    
-   适合多地区业务、大量镜像下载需求。
    

**4\. 主从配置步骤**

![](https://oss-emcsprod-public.modb.pro/wechatSpider/modb_20220802_63dc4640-1201-11ed-ac0b-38f9d3cd240d.png)

-   **提供者**：选择Harbor；
    
-   **目标名**：自定义，可填一个具有声明意义的名称；
    
-   **描述**：自定义，可备注目标仓库的一些信息；
    
-   **目标URL**：对端仓库的地址；
    
-   **访问ID**：目标仓库的用户名；
    
-   **访问密码**：访问ID的密码；
    
-   **验证远程证书**：如果对端用的是自签证书或者非信任证书的话不要勾选此项。
    

![](https://oss-emcsprod-public.modb.pro/wechatSpider/modb_20220802_63e4347c-1201-11ed-ac0b-38f9d3cd240d.png)

![](https://oss-emcsprod-public.modb.pro/wechatSpider/modb_20220802_63edaa84-1201-11ed-ac0b-38f9d3cd240d.png)

-   **名称**：自定义，有声明意义；
    
-   **描述**：自定义，备注规则中仓库信息；
    
-   **复制模式**
    
    **Push-base****d**：从本地仓库推送到远程仓库，双主模式两个harbor同时配置；
    
    **Pull-based**：从远程仓库拉去到本地仓库，一主多从模式的从库可配置，主从模式从库可配置；
    
-   **目标仓库/源仓库**：对端仓库地址，在仓库管理中添加；
    
-   **资源过滤器**：指定根据什么来同步；
    
-   **触发模式**：一般选择事件驱动，这样主库已更新就会同步。
    

## **docker基础命令使用以及镜像下载上传**

**1\. 主机连接harbor仓库命令**  

docker login 目标仓库地址：端口

输入账号密码：

![](https://oss-emcsprod-public.modb.pro/wechatSpider/modb_20220802_63f5446a-1201-11ed-ac0b-38f9d3cd240d.png)

**2\. 退出连接**

docker logout 目标仓库地址：端口

![](https://oss-emcsprod-public.modb.pro/wechatSpider/modb_20220802_640036f4-1201-11ed-ac0b-38f9d3cd240d.png)

**3\. 从docker仓库下载任意镜像**

docker pull 镜像名称。

![](https://oss-emcsprod-public.modb.pro/wechatSpider/modb_20220802_6406ee4a-1201-11ed-ac0b-38f9d3cd240d.png)

**4\. 查看下载镜像**

```
docker images
```

![](https://oss-emcsprod-public.modb.pro/wechatSpider/modb_20220802_640dc792-1201-11ed-ac0b-38f9d3cd240d.png)  

**5\. 查看已启动的镜像**

docker ps  -a=历史启动过的镜像。

![](https://oss-emcsprod-public.modb.pro/wechatSpider/modb_20220802_641a168c-1201-11ed-ac0b-38f9d3cd240d.png)

**6\. 镜像重命名**

docker tag 原镜像名称 仓库地址/项目名称/镜像:镜像版本。

![](https://oss-emcsprod-public.modb.pro/wechatSpider/modb_20220802_64276de6-1201-11ed-ac0b-38f9d3cd240d.png)

**7\. 上传镜像到私有仓库及验证**

docker push 仓库地址/项目名称/镜像:镜像版本。

![](https://oss-emcsprod-public.modb.pro/wechatSpider/modb_20220802_6431d916-1201-11ed-ac0b-38f9d3cd240d.png)![](https://oss-emcsprod-public.modb.pro/wechatSpider/modb_20220802_643992c8-1201-11ed-ac0b-38f9d3cd240d.png)

![](https://oss-emcsprod-public.modb.pro/wechatSpider/modb_20220802_64415436-1201-11ed-ac0b-38f9d3cd240d.png)

## 本文作者：罗 啸（上海新炬王翦团队）

## 本文来源：“IT那活儿”公众号

![](https://oss-emcsprod-public.modb.pro/wechatSpider/modb_20220802_644c0f7a-1201-11ed-ac0b-38f9d3cd240d.png)