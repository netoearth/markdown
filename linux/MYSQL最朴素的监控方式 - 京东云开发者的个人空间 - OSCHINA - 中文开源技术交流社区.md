对于当前数据库的监控方式有很多，分为数据库自带、商用、开源三大类，每一种都有各自的特色；而对于 mysql 数据库由于其有很高的社区活跃度，监控方式更是多种多样，不管哪种监控方式最核心的就是监控数据，获取得到全面的监控数据后就是灵活的展示部分。那我们今天就介绍一下完全采用 mysql 自有方式采集获取监控数据，在单体下达到最快速、方便、损耗最小。

本次文章完全使用 mysql 自带的 show 命令实现获取，从 connects、buffercache、lock、SQL、statement、Database throughputs、serverconfig7 大方面全面获取监控数据。

### 1 连接数（Connects）

-   最大使用连接数：show status like ‘Max\_used\_connections’
-   当前打开的连接数：show status like ‘Threads\_connected’

### 2 缓存（bufferCache）

-   未从缓冲池读取的次数：show status like ‘Innodb\_buffer\_pool\_reads’
-   从缓冲池读取的次数：show status like ‘Innodb\_buffer\_pool\_read\_requests’
-   缓冲池的总页数：show status like ‘Innodb\_buffer\_pool\_pages\_total’
-   缓冲池空闲的页数：show status like ‘Innodb\_buffer\_pool\_pages\_free’
-   缓存命中率计算：（1-Innodb\_buffer\_pool\_reads/Innodb\_buffer\_pool\_read\_requests）\*100%
-   缓存池使用率为：((Innodb\_buffer\_pool\_pages\_total-Innodb\_buffer\_pool\_pages\_free）/Innodb\_buffer\_pool\_pages\_total）\*100%

### 3 锁（lock）

-   锁等待个数：show status like ‘Innodb\_row\_lock\_waits’
-   平均每次锁等待时间：show status like ‘Innodb\_row\_lock\_time\_avg’
-   查看是否存在表锁：show open TABLES where in\_use>0；有数据代表存在锁表，空为无表锁

备注：锁等待统计得数量为累加数据，每次获取得时候可以跟之前得数据进行相减，得到当前统计得数据

### 4 SQL

-   查看 mysql 开关是否打开：show variables like ‘slow\_query\_log’，ON 为开启状态，如果为 OFF，set global slow\_query\_log=1 进行开启
-   查看 mysql 阈值：show variables like ‘long\_query\_time’，根据页面传递阈值参数，修改阈值 set global long\_query\_time=0.1
-   查看 mysql 慢 sql 目录：show variables like ‘slow\_query\_log\_file’
-   格式化慢 sql 日志：mysqldumpslow -s at -t 10 /export/data/mysql/log/slow.log  
    注：此语句通过 jdbc 执行不了，属于命令行执行。  
    意思为：显示出耗时最长的 10 个 SQL 语句执行信息，10 可以修改为 TOP 个数。显示的信息为：执行次数、平均执行时间、SQL 语句

备注：当 mysqldumpslow 命令执行失败时，将慢日志同步到本地进行格式化处理。

### 5 statement

-   insert 数量：show status like ‘Com\_insert’
-   delete 数量：show status like ‘Com\_delete’
-   update 数量：show status like ‘Com\_update’
-   select 数量：show status like ‘Com\_select’

### 6 吞吐（Database throughputs）

-   发送吞吐量：show status like ‘Bytes\_sent’
-   接收吞吐量：show status like ‘Bytes\_received’
-   总吞吐量：Bytes\_sent+Bytes\_received

### 7 数据库参数（serverconfig）

show variables

![](https://img1.jcloudcs.com/developer.jdcloud.com/8325e861-eeac-4931-a228-788274237af120220628151134.png)

### 8 慢 SQL

慢 SQL 指的是 MySQL 慢查询，具体指运行时间超过 long\_query\_time 值的 SQL。  
我们常听 MySQL 中有二进制日志 binlog、中继日志 relaylog、重做回滚日志 redolog、undolog 等。针对慢查询，还有一种慢查询日志 slowlog，用来记录在 MySQL 中响应时间超过阀值的语句。慢 SQL 对实际生产业务影响是致命的，所以测试人员在性能测试过程中，对数据库 SQL 语句执行情况实施监控，给开发提供准确的性能优化意见显得尤为重要。那怎么使用 Mysql 数据库提供的慢查询日志来监控 SQL 语句执行情况，找到消耗较高的 SQL 语句，以下详细说明一下慢查询日志的使用步骤：

-   确保打开慢 SQL 开关 slow\_query\_log

![](https://img1.jcloudcs.com/developer.jdcloud.com/1b2668e5-5b21-4c52-a2f3-8eccaf38cdcb20220628151158.png)

-   设置慢 SQL 域值 long\_query\_time  
    这个 long\_query\_time 是用来定义慢于多少秒的才算 “慢查询”，注意单位是秒，我通过执行 sql 指令 set long\_query\_time=1 来设置了 long\_query\_time 的值为 1, 也就是执行时间超过 1 秒的都算慢查询，如下：

![](https://img1.jcloudcs.com/developer.jdcloud.com/78b3dd23-8420-4819-93fe-c61dc5ddd20120220628151241.png)

-   查看慢 SQL 日志路径

![](https://img1.jcloudcs.com/developer.jdcloud.com/bdac706c-85eb-410a-9cc0-c32ba1e11efc20220628151255.png)

-   通过慢 sql 分析工具 mysqldumpslow 格式化分析慢 SQL 日志  
    mysqldumpslow 慢查询分析工具，是 mysql 安装后自带的，可以通过./mysqldumpslow —help 查看使用参数说明

![](https://img1.jcloudcs.com/developer.jdcloud.com/c011c55d-1549-4a41-83b7-8ac63a5ce1f920220628151311.png)

常见用法：

1.  取出使用最多的 10 条慢查询  
    ./mysqldumpslow -s c -t 10 /export/data/mysql/log/slow.log
2.  取出查询时间最慢的 3 条慢查询  
    ./mysqldumpslow -s t -t 3 /export/data/mysql/log/slow.log

注意：使用 mysqldumpslow 的分析结果不会显示具体完整的 sql 语句，只会显示 sql 的组成结构；  
假如: SELECT _FROM sms\_send WHERE service\_id=10 GROUP BY content LIMIT 0, 1000;  
mysqldumpslow 命令执行后显示：  
Count: 2 Time=1.5s (3s) Lock=0.00s (0s) Rows=1000.0 (2000), vgos\_dba\[vgos\_dba\]@\[10.130.229.196\]SELECT_ FROM sms\_send WHERE service\_id=N GROUP BY content LIMIT N, N

**mysqldumpslow 的分析结果详解：**

-   Count：表示该类型的语句执行次数，上图中表示 select 语句执行了 2 次。
-   Time：表示该类型的语句执行的平均时间（总计时间）
-   Lock：锁时间 0s。
-   Rows：单次返回的结果数是 1000 条记录，2 次总共返回 2000 条记录。

通过这个工具就可以查询出来哪些 sql 语句是慢 SQL，从而反馈研发进行优化，比如加索引，该应用的实现方式等。

**常见慢 SQL 排查**

1.  不使用子查询  
    SELECT _FROM t1 WHERE id (SELECT id FROM t2 WHERE name=’hechunyang’);  
    子查询在 MySQL5.5 版本里，内部执行计划器是这样执行的：先查外表再匹配内表，而不是先查内表 t2，当外表的数据很大时，查询速度会非常慢。  
    在 MariaDB10/MySQL5.6 版本里，采用 join 关联方式对其进行了优化，这条 SQL 会自动转换为 SELECT t1._ FROM t1 JOIN t2 ON t1.id = t2.id;  
    但请注意的是：优化只针对 SELECT 有效，对 UPDATE/DELETE 子 查询无效， 生产环境尽量应避免使用子查询。
2.  避免函数索引  
    SELECT _FROM t WHERE YEAR(d) >= 2016;  
    由于 MySQL 不像 Oracle 那样⽀持函数索引，即使 d 字段有索引，也会直接全表扫描。  
    应改为 > SELECT_ FROM t WHERE d >= ‘2016-01-01’;
3.  用 IN 来替换 OR 低效查询  
    慢 SELECT _FROM t WHERE LOC\_ID = 10 OR LOC\_ID = 20 OR LOC\_ID = 30;  
    高效查询 > SELECT_ FROM t WHERE LOC\_IN IN (10,20,30);
4.  LIKE 双百分号无法使用到索引  
    SELECT _FROM t WHERE name LIKE ‘%de%’;  
    使用 SELECT_ FROM t WHERE name LIKE ‘de%’;
5.  分组统计可以禁止排序  
    SELECT goods\_id,count(_) FROM t GROUP BY goods\_id;  
    默认情况下，MySQL 对所有 GROUP BY col1，col2… 的字段进⾏排序。如果查询包括 GROUP BY，想要避免排序结果的消耗，则可以指定 ORDER BY NULL 禁止排序。  
    使用 SELECT goods\_id,count (_) FROM t GROUP BY goods\_id ORDER BY NULL;
6.  禁止不必要的 ORDER BY 排序  
    SELECT count(1) FROM user u LEFT JOIN user\_info i ON u.id = i.user\_id WHERE 1 = 1 ORDER BY u.create\_time DESC;  
    使用 SELECT count (1) FROM user u LEFT JOIN user\_info i ON u.id = i.user\_id;

### 9 总结

-   任何东西不应过重关注其外表，要注重内在的东西，往往绚丽的外表下会有对应的负担和损耗。
-   mysql 数据库的监控支持通过 SQL 方式从 performance\_schema 库中访问对应的表数据，前提是初始化此库并开启监控数据写入。
-   对于监控而言，不在于手段的多样性，而需要明白监控的本质，以及需要的监控项内容，找到符合自身项目特色的监控方式。
-   在选择监控工具对 mysql 监控时，需要关注监控工具本身对于数据库服务器的消耗，不要影响到其自身的使用。

**作者：安甲舒**