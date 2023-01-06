> 官方文档：[https://seata.io/zh-cn/docs/ops/deploy-guide-beginner.html](https://seata.io/zh-cn/docs/ops/deploy-guide-beginner.html)

[Seata](https://so.csdn.net/so/search?q=Seata&spm=1001.2101.3001.7020) 的架构中，一共有三个角色：

-   TC (Transaction Coordinator) - 事务协调者：维护全局和分支事务的状态，驱动全局事务提交或回滚。
-   TM (Transaction Manager) - 事务管理器：定义全局事务的范围：开始全局事务、提交或回滚全局事务。
-   RM (Resource Manager) - 资源管理器：管理分支事务处理的资源，与TC交谈以注册分支事务和报告分支事务的状态，并驱动分支事务提交或回滚。

其中，`TC（Server端）为单独部署的 Server 服务端`，TM 和 RM （Client端）由业务系统集成使用。

## 一、Seata Server环境搭建

使用 Docker 部署 Seata Server：[https://seata.io/zh-cn/docs/ops/deploy-by-docker-142.html](https://seata.io/zh-cn/docs/ops/deploy-by-docker-142.html)

下面使用 Docker安装 TC（Server端）。

**Server端存储模式（store.mode）支持三种：**

-   file：(默认)单机模式，全局事务会话信息内存中读写并持久化本地文件root.data。
-   db：高可用模式，全局事务会话信息通过db共享。
-   redis：Seata-Server 1.3及以上版本支持。

**资源目录：**

> 下载Seata源码：[https://github.com/seata/seata/tree/v1.5.1](https://github.com/seata/seata/tree/v1.5.1)

在Seata源码的script目录中有三个文件夹：

-   client：存放client端sql脚本 (包含 undo\_log表) ，参数配置
-   config-center：各个配置中心参数导入脚本，config.txt(包含server和client，原名nacos-config.txt)为通用参数文件
-   server：server端数据库脚本 (包含 lock\_table、branch\_table 与 global\_table) 及各个容器配置

## 1、拉取镜像

1）查找镜像

```
[root@centos7 ~]# docker search seataio/seata-server
```

2）拉取镜像

本地没有镜像时拉取，有的话就省略这一步。

```
[root@centos7 ~]# docker pull seataio/seata-server:1.5.1
```

3）创建并启动容器

启动 TC（Server端）。

```
[root@centos7 ~]# docker run -d --name seata-tc-server_v1.5.1 -p 192.168.198.110:7091:7091 seataio/seata-server:1.5.1
b1b2edf1edef77d8909791e562d6e990386cc2b27a52215dbcd4e309f32fc7e4

```

查看启动日志：

```
[root@centos7 ~]# docker logs -f b1b2edf1edef
```

4）进入启动好的服务

```
[root@centos7 ~]# docker exec -it b1b2edf1edef sh
/seata-server # ls
classes       libs          resources     sessionStore
/seata-server # cd resources/
/seata-server/resources # ls
META-INF                 README.md                application.yml          io                       logback-spring.xml
README-zh.md             application.example.yml  banner.txt               logback                  lua

```

## 2、建库建表

创建 seata\_server数据库。执行该SQL脚本：\\script\\server\\db\\mysql.sql。

> SQL脚本：[https://github.com/seata/seata/tree/v1.5.1/script/server/db](https://github.com/seata/seata/tree/v1.5.1/script/server/db)

```
-- -------------------------------- The script used when storeMode is 'db' --------------------------------
-- the table to store GlobalSession data
CREATE TABLE IF NOT EXISTS `global_table`
(
    `xid`                       VARCHAR(128) NOT NULL,
    `transaction_id`            BIGINT,
    `status`                    TINYINT      NOT NULL,
    `application_id`            VARCHAR(32),
    `transaction_service_group` VARCHAR(32),
    `transaction_name`          VARCHAR(128),
    `timeout`                   INT,
    `begin_time`                BIGINT,
    `application_data`          VARCHAR(2000),
    `gmt_create`                DATETIME,
    `gmt_modified`              DATETIME,
    PRIMARY KEY (`xid`),
    KEY `idx_status_gmt_modified` (`status` , `gmt_modified`),
    KEY `idx_transaction_id` (`transaction_id`)
) ENGINE = InnoDB
  DEFAULT CHARSET = utf8mb4;

-- the table to store BranchSession data
CREATE TABLE IF NOT EXISTS `branch_table`
(
    `branch_id`         BIGINT       NOT NULL,
    `xid`               VARCHAR(128) NOT NULL,
    `transaction_id`    BIGINT,
    `resource_group_id` VARCHAR(32),
    `resource_id`       VARCHAR(256),
    `branch_type`       VARCHAR(8),
    `status`            TINYINT,
    `client_id`         VARCHAR(64),
    `application_data`  VARCHAR(2000),
    `gmt_create`        DATETIME(6),
    `gmt_modified`      DATETIME(6),
    PRIMARY KEY (`branch_id`),
    KEY `idx_xid` (`xid`)
) ENGINE = InnoDB
  DEFAULT CHARSET = utf8mb4;

-- the table to store lock data
CREATE TABLE IF NOT EXISTS `lock_table`
(
    `row_key`        VARCHAR(128) NOT NULL,
    `xid`            VARCHAR(128),
    `transaction_id` BIGINT,
    `branch_id`      BIGINT       NOT NULL,
    `resource_id`    VARCHAR(256),
    `table_name`     VARCHAR(32),
    `pk`             VARCHAR(36),
    `status`         TINYINT      NOT NULL DEFAULT '0' COMMENT '0:locked ,1:rollbacking',
    `gmt_create`     DATETIME,
    `gmt_modified`   DATETIME,
    PRIMARY KEY (`row_key`),
    KEY `idx_status` (`status`),
    KEY `idx_branch_id` (`branch_id`),
    KEY `idx_xid_and_branch_id` (`xid` , `branch_id`)
) ENGINE = InnoDB
  DEFAULT CHARSET = utf8mb4;

CREATE TABLE IF NOT EXISTS `distributed_lock`
(
    `lock_key`       CHAR(20) NOT NULL,
    `lock_value`     VARCHAR(20) NOT NULL,
    `expire`         BIGINT,
    primary key (`lock_key`)
) ENGINE = InnoDB
  DEFAULT CHARSET = utf8mb4;

INSERT INTO `distributed_lock` (lock_key, lock_value, expire) VALUES ('AsyncCommitting', ' ', 0);
INSERT INTO `distributed_lock` (lock_key, lock_value, expire) VALUES ('RetryCommitting', ' ', 0);
INSERT INTO `distributed_lock` (lock_key, lock_value, expire) VALUES ('RetryRollbacking', ' ', 0);
INSERT INTO `distributed_lock` (lock_key, lock_value, expire) VALUES ('TxTimeoutCheck', ' ', 0);
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/2f6de75808cf406f96e979d9801c9091.png)

## 3、Nacos(注册&配置中心)部署Seata Server

使用 Nacos作为Seata Server端的注册和配置中心。Seata Server存储模式等配置通过Nacos配置中心来管理。

### 3.1 配置Nacos注册中心

将Seata Server注册到 Nacos。

**1）修改 application.yml文件**

进入容器，修改 application.yml文件。

```
seata:
  registry:
    # support: nacos 、 eureka 、 redis 、 zk  、 consul 、 etcd3 、 sofa
    type: nacos
    nacos:
      application: seata-server
      server-addr: 192.168.xxx.xxx:8848
      username: nacos
      password: nacos
      namespace: b4d0832b-a7b0-44c2-8ce5-1abe676a4736
```

### 3.2 配置Nacos配置中心

**1）修改 application.yml文件**

进入容器，修改application.yml文件。

```
seata:
  config:
    # support: nacos 、 consul 、 apollo 、 zk  、 etcd3
    type: nacos
    nacos:
      server-addr: 192.168.xxx.xxx:8848
      namespace: b4d0832b-a7b0-44c2-8ce5-1abe676a4736
      username: nacos
      password: nacos
      data-id: seataServer.properties
```

**2）添加seataServer.properties文件**

Seata从v1.4.2版本开始，支持从一个Nacos dataId配置项中获取所有配置信息。所以，在nacos配置中心中新建配置，dataId为 seataServer.properties配置项。

![在这里插入图片描述](https://img-blog.csdnimg.cn/03248f802a6b4a07830e0b17d9990a68.png)

> SeataServer.properties内容为/seata/script/config-center/config.txt 中的配置信息，修改为自己的配置信息。

简单配置如下：

```
store.mode=db
store.db.datasource=druid
store.db.dbType=mysql
store.db.driverClassName=com.mysql.jdbc.Driver
store.db.url=jdbc:mysql://192.168.xxx.xxx:3306/seata_server?useUnicode=true&characterEncoding=UTF-8&serverTimezone=Asia/Shanghai
store.db.user=root
store.db.password=123456
store.db.minConn=5
store.db.maxConn=30
store.db.globalTable=global_table
store.db.branchTable=branch_table
store.db.queryLimit=100
store.db.lockTable=lock_table
store.db.maxWait=5000
```

## 4、启动Seata Server

上面配置好之后，重新启动容器。

1）访问 Seata控制台，

访问：http://ip:7091/，账号密码都是 seata。

![在这里插入图片描述](https://img-blog.csdnimg.cn/18036f56b7d548208cade33084ec25b3.png)  
2）访问 Nacos控制台，

查看 Seata-Server注册成功。

![在这里插入图片描述](https://img-blog.csdnimg.cn/5c9d267e07284c93a15df664f19b6e0e.png)  
到此，Seata Server 搭建成功。

> – 求知若饥，虚心若愚。