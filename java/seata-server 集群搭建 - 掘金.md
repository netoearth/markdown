   

[![](https://p3-passport.byteimg.com/img/user-avatar/324dd3d2991d11e0fe6ec9d09bb635a3~100x100.awebp)](https://juejin.cn/user/3553261946938877)

2021年06月29日 11:33 ·  阅读 1865

![seata-server 集群搭建](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f0e91626c1e44779b4b7124f78f66c23~tplv-k3u1fbpfcp-zoom-crop-mark:3024:3024:3024:1702.awebp)  

## seata-server 集群搭建

#### 介绍

Seata分TC、TM和RM三个角色，TC（Server端）为单独服务端部署，TM和RM（Client端）由业务系统集成。 Seata-server 的高可用依赖于`注册中心`、`配置中心`和`数据库`来实现。

Server端存储模式（store.mode）现有file、db、redis三种（后续将引入raft,mongodb），file模式无需改动，直接启动即可，下面专门讲下db启动步骤。

注： file模式为单机模式，全局事务会话信息内存中读写并持久化本地文件root.data，性能较高;

db模式为高可用模式，全局事务会话信息通过db共享，相应性能差些;

redis模式Seata-Server 1.3及以上版本支持,性能较高,存在事务信息丢失风险,请提前配置合适当前场景的redis持久化配置.

下面主要介绍一下基于`nacos`注册中心和`mysql` 数据库的集群高可用的搭建和实现。

#### 原理和架构图

seata-server 的高可用的实现，主要基于db和注册中心，通过db获取全局事务，实现多实例事务共享。通过注册中心来实现seata-server多实例的动态管理。架构图原理图如 下：

![未命名文件.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7eeb7e28cc974325806eead88cd73e23~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

#### 安装步骤

安装环境：centos7、jdk8

-   下载安装包
    
    [点击下载](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fseata%2Fseata%2Freleases "https://github.com/seata/seata/releases")最新的seata-server安装包，目前最新的为Seata 1.4.2
    

![uTools_1623378971538.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4d97c80ef37f4a17bdd174fb01edc583~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

-   解压安装
    
    解压安装包：
    

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2ace573983424cc8a8309f19fd476795~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

新建seata-server 数据库,名称：seata，数据库脚本[：mysql](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fseata%2Fseata%2Fblob%2Fdevelop%2Fscript%2Fserver%2Fdb%2Fmysql.sql "https://github.com/seata/seata/blob/develop/script/server/db/mysql.sql") ,下载执行后：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/27574bc321ad4279853b51bb55776629~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

然后进入seata-server 的conf目录，修改配置file.conf 和 registry.conf 两个配置文件。

修改file.conf文件：把seata-server的存储模式改为db 模式，修改db的数据库连接信息。

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0ac3b2a62f0a49ab999fb59357ada089~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

修改registry.conf 文件：把seata-server 的注册中心和配置中心都修改使用nacos 如下图

注册中心

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1a2cc3fc0407417e8d36af1229f24b92~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

配置中心

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b53217be7724486f8eec20516adf9846~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

初始化配置到 nacos, 新建一个config.txt 的文件，内容如下：

```
service.vgroupMapping.sharding_jdbc_group=default
store.mode=db
store.db.datasource=druid
store.db.dbType=mysql
store.db.driverClassName=com.mysql.jdbc.Driver
store.db.url=jdbc:mysql://127.0.0.1:3306/seata?useUnicode=true&rewriteBatchedStatements=true
store.db.user=root
store.db.password=root
store.db.minConn=5
store.db.maxConn=30
store.db.globalTable=global_table
store.db.branchTable=branch_table
store.db.queryLimit=100
store.db.lockTable=lock_table
store.db.maxWait=5000
复制代码
```

下载执行脚本[：nacos-config.sh](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fseata%2Fseata%2Fblob%2Fdevelop%2Fscript%2Fconfig-center%2Fnacos%2Fnacos-config.sh "https://github.com/seata/seata/blob/develop/script/config-center/nacos/nacos-config.sh"), 然后执行下面的命令：

```
sh nacos-config.sh -h localhost -p 8848 -g SEATA_GROUP -t 5a3c7d6c-f497-4d68-a71a-2e5e3340b3ca -u username -w password
复制代码
```

执行成功后，会把config.txt中的配置推送到nacos中，进行管理，效果如：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cb9da206c48749c0b42b3649d5cfd0ba~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

其他机器，安装seata-server 只需要重复以上步骤即可。

-   启动seata-server 启动master节点,启动命令：sh seata-server.sh -h ip -p 8091 -m db -n 1

启动slave节点,启动命令: sh seata-server.sh -h ip -p 8091 -m db -n 2

```
    -h: 注册到注册中心的ip
    -p: Server rpc 监听端口
    -m: 全局事务会话信息存储模式，file、db、redis，优先读取启动参数 (Seata-Server 1.3及以上版本支持redis)
    -n: Server node，多个Server时，需区分各自节点，用于生成不同区间的transactionId，以免冲突
复制代码
```

启动成功

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bc0a1d6fd602435594f0cfed596e6589~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

在nacos上查看注册情况

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/993d69a1f5b141dcb5f790e8a407efa0~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp) 以上说明，seata-server集群已经搭建成功。

#### SpringCloud集成Seata集群

这里其实和单个seata 集成springcloud 的配置类似，就不多说了，主要提一下，springcloud如何连接seata集群的配置：

```
seata:
  enabled: true # seata 的开关
  application-id: ${spring.application.name}
  tx-service-group: sharding_jdbc_group
  enable-auto-data-source-proxy: false
  service:
    vgroup-mapping:
      sharding_jdbc_group: 'seata-cluster' # seata-server 注册到nacos上时设置的集群名称。
  registry:
    type: nacos
    nacos:
      application: seata-server # seata-server 实例的名称
      server-addr: ${spring.cloud.nacos.discovery.server-addr}
      namespace: 777a9882-1ec7-45bb-9a68-a54b9dee89ac
      group: SEATA_GROUP
  transport:
    compressor: gzip
复制代码
```

相关课程

![「高并发秒杀的设计精要与实现」封面](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d1879744b9094adfbe63498d64efe2ce~tplv-k3u1fbpfcp-no-mark:420:420:300:420.awebp?)

![「Elasticsearch 从入门到实践」封面](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ba4ecdc5afe94cc28c3c700a20524500~tplv-k3u1fbpfcp-no-mark:420:420:300:420.awebp?)

![](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/00ba359ecd0075e59ffbc3d810af551d.svg) 赞

![](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/3d482c7a948bac826e155953b2a28a9e.svg) 收藏