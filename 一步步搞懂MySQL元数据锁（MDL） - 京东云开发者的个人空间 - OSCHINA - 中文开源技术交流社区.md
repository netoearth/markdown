某日，路上收到用户咨询，为了清除空间，想删除某 200 多 G 大表数据，且已经确认此表不再有业务访问，于是执行了一条命令‘delete from bigtable’，但好长时间也没删完，经过咨询后，获知 drop table 删除表速度快，而且能彻底释放空间，于是又在另外一个 session 中执行了‘drop table bigtable’命令，但是这个命令并没有快速返回结果，光标一直 hang 在原地不动。最后找我们协助，在登录数据库执行‘show processlist’后发现 drop 语句的状态是‘waiting for table metadata lock’，而之前执行的另外一个 delete 语句依旧能看到，状态为‘updating’，截图如下：

![](https://p3-sign.toutiaoimg.com/tos-cn-i-qvj2lq49k0/74830853296e423e87328b3af887753f~noop.image?_iz=58558&from=article.pc_detail&x-expires=1663900229&x-signature=pD8e3WEQ%2BMm4ZqlXXKycoCDluJY%3D)

到底什么是 metadata lock？这个锁等待是如何产生的？会带来什么影响？最后又如何来解决？今天我们挑 6 **个常见问题**给大家解答一下。

**一、什么是 metadata lock？**

在 MySQL5.5.3 之前，有一个著名的 **bug#989**，大致如下:

```
 session1:  
 BEGIN;
 INSERT INTO t ... ;
 COMMIT;  

 session2:  
 DROP TABLE t;
```

然而上面的操作流程在 binlog 记录的顺序是

```
 DROP TABLE t; 
 BEGIN;  
 INSERT INTO t ... ; 
 COMMIT;
```

很显然备库执行 binlog 时会先删除表 t，然后执行 insert 会报 1032 error，导致复制中断。为了解决该 bug,MySQL 在 5.5.3 引入了 MDL 锁（metadata lock），来保护表的元数据信息，用于解决或者保证 DDL 操作与 DML 操作之间的一致性。

再举一个简单的例子，如果你在查询一个表的过程中，另外一个 session 对该表删除了一个列，那前面的查询到底该显示什么呢？如果在 RR 隔离级别下，事物中再次执行相同的语句还会和之前结果一致吗？为了防止这种情况，表查询开始 MySQL 会在表上加一个锁，来防止被别的 session 修改了表定义，这个锁就叫‘metadata lock’，简称 MDL，翻译成中文也叫‘元数据锁’。

**二、MDL 和行锁有什么区别？**

**metadata lock 是表级锁，是在 server 层加的，适用于所有存储引擎。**所有的 dml 操作都会在表上加一个 metadata 读锁；所有的 ddl 操作都会在表上加一个 metadata 写锁。读锁和写锁的阻塞关系如下：

-   读锁和写锁之间相互阻塞，即同一个表上的 dml 和 ddl 之间互相阻塞。
-   写锁和写锁之间互相阻塞，即两个 session 不能对表同时做表定义变更，需要串行操作。
-   读锁和读锁之间不会产生阻塞。也就是增删改查不会因为 metadata lock 产生阻塞，可以并发执行，日常工作中大家看到的 dml 之间的锁等待是 innodb 行锁引起的，和 metadata lock 无关。

熟悉 innodb 行锁的同学这里可能有点困惑，因为行锁分类和 metadata lock 很类似，也主要分为读锁和写锁，或者叫共享锁和排他锁，读写锁之间阻塞关系也一致。二者最重要的区别一个是表锁，一个是行锁，且行锁中的读写操作对应在 metadata lock 中都属于读锁。

大家也许会奇怪，以前听说普通查询不加锁的，怎么这里又说要加表锁，我们做一个简单测试：

session1：查询前，先看一下 metadata\_locks 表，这个表位于 performance\_schema 下，记录了 metadata lock 的加锁信息。

```
mysql> select * from performance_schema.metadata_locks ;
+-------------+--------------------+----------------+-------------+-----------------------+-------------+---------------+-------------+-------------------+-----------------+----------------+
| OBJECT_TYPE | OBJECT_SCHEMA      | OBJECT_NAME    | COLUMN_NAME | OBJECT_INSTANCE_BEGIN | LOCK_TYPE   | LOCK_DURATION | LOCK_STATUS | SOURCE            | OWNER_THREAD_ID | OWNER_EVENT_ID |
+-------------+--------------------+----------------+-------------+-----------------------+-------------+---------------+-------------+-------------------+-----------------+----------------+
| TABLE       | performance_schema | metadata_locks | NULL        |       139776223308432 | SHARED_READ | TRANSACTION   | GRANTED     | sql_parse.cc:6014 |              54 |             12 |
+-------------+--------------------+----------------+-------------+-----------------------+-------------+---------------+-------------+-------------------+-----------------+----------------+
1 row in set (0.00 sec)
```

session2：执行简单查询，为了让表处于执行状态，这里使用了 sleep 函数。

```
mysql> select sleep(10) from t1;
+-----------+
| sleep(10) |
+-----------+
|         0 |
|         0 |
|         0 |
+-----------+
3 rows in set (30.00 sec)
```

session1：

```
mysql> select * from performance_schema.metadata_locks ;
+-------------+--------------------+----------------+-------------+-----------------------+-------------+---------------+-------------+-------------------+-----------------+----------------+
| OBJECT_TYPE | OBJECT_SCHEMA      | OBJECT_NAME    | COLUMN_NAME | OBJECT_INSTANCE_BEGIN | LOCK_TYPE   | LOCK_DURATION | LOCK_STATUS | SOURCE            | OWNER_THREAD_ID | OWNER_EVENT_ID |
+-------------+--------------------+----------------+-------------+-----------------------+-------------+---------------+-------------+-------------------+-----------------+----------------+
| TABLE       | db1                | t1             | NULL        |       139776154308336 | SHARED_READ | TRANSACTION   | GRANTED     | sql_parse.cc:6014 |              53 |             22 |
| TABLE       | performance_schema | metadata_locks | NULL        |       139776223308432 | SHARED_READ | TRANSACTION   | GRANTED     | sql_parse.cc:6014 |              54 |             13 |
+-------------+--------------------+----------------+-------------+-----------------------+-------------+---------------+-------------+-------------------+-----------------+----------------+
2 rows in set (0.00 sec)
```

此时再次查看 metadata\_lock 表，发现多了一条 t1 的加锁记录，加锁类型为 SHARED\_READ, 且状态是已授予（GRANTED)。大家通常理解的查询不加锁，是指不在表上加 innodb 行锁。

如果在执行 sleep 期间，另外一个 session 执行了一个加字段操作，此时就会产生 metadata lock 锁等待：

session2:

mysql> select sleep(10) from t1;

执行中......

session3：

mysql> alter table t1 add col1 int;

阻塞中......

session1:

```
mysql> show processlist;
+----+-----------------+-----------+------+---------+--------+---------------------------------+-----------------------------+
| Id | User            | Host      | db   | Command | Time   | State                           | Info                        |
+----+-----------------+-----------+------+---------+--------+---------------------------------+-----------------------------+
|  4 | event_scheduler | localhost | NULL | Daemon  | 861577 | Waiting on empty queue          | NULL                        |
| 18 | root            | localhost | db1  | Sleep   |     50 |                                 | NULL                        |
| 19 | root            | localhost | NULL | Query   |      0 | starting                        | show processlist            |
| 20 | root            | localhost | db1  | Query   |     11 | Waiting for table metadata lock | alter table t1 add col1 int |
+----+-----------------+-----------+------+---------+--------+---------------------------------+-----------------------------+
4 rows in set (0.00 sec)
```

显然，id 为 20 的线程还未执行 alter 操作，状态为‘Waiting for table metadata lock’，也就是在等待 session2 的 sleep 操作完成。

**三、MDL 为什么会造成系统崩溃？**

举一个简单例子:

-   session1 启动一个事务，对表 t1 执行一个简单的查询；
-   session2 对 t1 加一个字段；
-   session3 来对 t1 做一个查询；
-   session4 来对 t1 做一个 update;

各个 session 串行操作。

session1：

```
mysql> begin;
Query OK, 0 rows affected (0.00 sec)


mysql> select * from t1 where id=1;
+----+------+------+-------+
| id | name | age  | birth |
+----+------+------+-------+
|  1 | aa   |   10 | NULL  |
+----+------+------+-------+
1 row in set (0.00 sec)
```

session2：

mysql> alter table t1 add col1 int;

阻塞中...

session3:

mysql> select sleep(10) from t1 ;

阻塞中...

session4:

mysql> update t1 set name='aaaa' where id=2;

阻塞中...

也就是由于 session1 的一个事务没有提交，导致 session2 的 ddl 操作被阻塞，session3 和 session4 本身不会被 session1 阻塞，但由于在锁队列中，session2 排队更早，它准备加的是 metadata lock 写锁，阻塞了 session3 和 session4 的读锁。如果 t1 是一个执行频繁的表，show processlist 会发现大量‘waiting for table metadata lock’的线程，数据库连接很快就会消耗完，导致业务系统无法正常响应。

此时如果 session1 提交，是 session2 的 alter 语句先执行还是 session3 和 session4 先执行呢？之前一直以为先到的先执行，当然是 session2 先执行，但经过测试，在 5.7 中，session3 和 session4 先执行，session2 最后执行，也就会出现 alter 长时间无法执行的情况；而在 8.0 中，session2 先执行，session3 和 session4 后执行，由于 5.6 以后 ddl 是 online 的，session2 并不会阻塞 session3 和 session4，感觉这样才是合理的，alter 不会被‘饿死’。

**四、MDL 的生命周期有多长？**

**事务！事务！事务！** 重要的事情说三遍，表上的 metadata lock 的生命周期从事务中的第一条涉及自身的语句开始，到整个事务结束而结束。而 5.5 之前是基于语句的，事务中执行完语句就释放，如果此时另外一个 session 对表做了一个删字段操作，那么就会造成两个问题：

-   ddl 操作如果先于事务完成，那么 binlog 中 ddl 就会排在事务之前，明显和逻辑不符，触发了本文开始提到的 bug。
-   如果是 RR 隔离级别，那么事务中此表第二次执行将无法返回同样的结果，无法满足可重复读的要求。

所以，如果要降低 metadata lock 的锁等待时间，最好要及时提交事务，同时尽量避免大事务。

那么如果发生 metadata lock 锁等待，等待锁的 session 会等待多长时间呢？大家都知道 MySQL 里面行锁等待有个超时时间（参数 innodb\_lock\_wait\_timeout），默认 50s。metadata lock 也有类似参数控制：

```
mysql> show variables like 'lock_wait_timeout'      ;
+-------------------+----------+
| Variable_name     | Value    |
+-------------------+----------+
| lock_wait_timeout | 31536000 |
+-------------------+----------+
1 row in set (0.00 sec)
```

这么长的数字，掰着指头算了半天，居然真的是...... 一年，环游世界一圈回来还得接着等！！！

当然，生产环境中，我们很少会等待 metadata lock 超时，更多的是要想办法把产生 metadata lock 的源头找到，快速提交或者回滚，或者想办法 kill 掉。那么如何找到阻塞的源头呢？

**五、如何快速找到阻塞源头？**

**快速解决问题永远是第一位的**，一旦出现长时间的 metadata lock，尤其是在访问频繁的业务表上产生，通常会导致表无法访问，读写全被阻塞，此时找到阻塞源头是第一位的。这里最重要的表就是前面提到过的  
performance\_schema.metadata\_locks 表。

metadata\_locks 是 5.7 中被引入，记录了 metadata lock 的相关信息，包括持有对象、类型、状态等信息。但 5.7 默认设置是关闭的（8.0 默认打开），需要通过下面命令打开设置：

```
UPDATE performance_schema.setup_instruments SET ENABLED = 'YES', TIMED = 'YES'WHERE NAME = 'wait/lock/metadata/sql/mdl';
```

如果要永久生效，需要在配置文件中加入如下内容：

```
[mysqld]
performance-schema-instrument='wait/lock/metadata/sql/mdl=ON'
```

单纯查询这个表无法得出具体的阻塞关系，也无法得知什么语句造成的阻塞，这里要关联另外两个表 performance\_schema.thread 和  
performance\_schema.events\_statements\_history,thread 表可以将线程 id 和 show processlist 中 id 关联，events\_statements\_history 表可以得到事务的历史 sql，关联后的完整 sql 如下：

```
SELECT
    locked_schema,
    locked_table,
    locked_type,
    waiting_processlist_id,
    waiting_age,
    waiting_query,
    waiting_state,
    blocking_processlist_id,
    blocking_age,
    substring_index(sql_text,"transaction_begin;" ,-1) AS blocking_query,
    sql_kill_blocking_connection
FROM
    (
        SELECT
            b.OWNER_THREAD_ID AS granted_thread_id,
            a.OBJECT_SCHEMA AS locked_schema,
            a.OBJECT_NAME AS locked_table,
            "Metadata Lock" AS locked_type,
            c.PROCESSLIST_ID AS waiting_processlist_id,
            c.PROCESSLIST_TIME AS waiting_age,
            c.PROCESSLIST_INFO AS waiting_query,
            c.PROCESSLIST_STATE AS waiting_state,
            d.PROCESSLIST_ID AS blocking_processlist_id,
            d.PROCESSLIST_TIME AS blocking_age,
            d.PROCESSLIST_INFO AS blocking_query,
            concat('KILL ', d.PROCESSLIST_ID) AS sql_kill_blocking_connection
        FROM
            performance_schema.metadata_locks a
        JOIN performance_schema.metadata_locks b ON a.OBJECT_SCHEMA = b.OBJECT_SCHEMA
        AND a.OBJECT_NAME = b.OBJECT_NAME
        AND a.lock_status = 'PENDING'
        AND b.lock_status = 'GRANTED'
        AND a.OWNER_THREAD_ID <> b.OWNER_THREAD_ID
        AND a.lock_type = 'EXCLUSIVE'
        JOIN performance_schema.threads c ON a.OWNER_THREAD_ID = c.THREAD_ID
        JOIN performance_schema.threads d ON b.OWNER_THREAD_ID = d.THREAD_ID
    ) t1,
    (
        SELECT
            thread_id,
            group_concat(   CASE WHEN EVENT_NAME = 'statement/sql/begin' THEN "transaction_begin" ELSE sql_text END ORDER BY event_id SEPARATOR ";" ) AS sql_text
        FROM
           performance_schema.events_statements_history
        GROUP BY thread_id
    ) t2
WHERE
    t1.granted_thread_id = t2.thread_id \G
   
```

对于前面的例子执行此 sql，得到一个清晰的阻塞关系：

```
               locked_schema: db1
                locked_table: t1
                 locked_type: Metadata Lock
      waiting_processlist_id: 28
                 waiting_age: 227
               waiting_query: alter table t1 add cl3 int
               waiting_state: Waiting for table metadata lock
     blocking_processlist_id: 27
                blocking_age: 252
              blocking_query: select * from t1
sql_kill_blocking_connection: KILL 27
1 row in set, 1 warning (0.00 sec)

```

根据显示结果，processlist\_id 为 27 的线程阻塞了 28 的线程，我们需要 kill 27 即可解锁。

实际上，MySQL 也提供了一个类似的视图来解决 metadata lock 问题，视图名称为 sys.schema\_table\_lock\_waits，但此视图查询结果有 bug，不是很准确，建议大家还是参考上面 sql。

**六、本文开始的案例最终如何解决？**

通过前面的介绍，本文开始的案例产生的过程就很简单了：用户执行了一个全表 delete，在目标表上加了 metadata 读锁，由于表很大，读锁长时间无法释放，后来另外一个 session 执行了 drop table 操作，又需要在表上加 metadata 写锁，由于读写锁互相阻塞，drop 操作只能等待 delete 操作完成才能获得写锁，因此从表面来看，二个命令都长时间没有响应，其实内部一个在执行，一个在等待。

那怎么来解决呢？因为从 show processlist 以及客户描述可以很清楚的知道故障机制，当时建议客户将 delete 操作 kill 掉，等数据回滚完后再执行 drop 操作因为 delete 已经执行了一段时间，回滚过程可能会较长，客户最终 kill delete 后顺利 drop 成功。

**小结**

生产环境大多是 dml 操作，metadata 读锁之间不会产生锁等待，而目前 MySQL 的 ddl 操作大多可以 online 执行，因此即使有写锁，也会很快降级为读锁，所以 ddl 执行期间阻塞 dml 的几率也很小。最容易出现的情况是由于有未完成的事务，导致 ddl metadata 写锁无法加上，只能在锁队列等待，而一旦进入锁队列，写锁又会阻塞其他的读锁，导致数据库连接快速增长，直至消耗殆尽，最终业务受到影响。

为了尽可能避免类似问题，下面是几个小建议：

-   生产环境的任何大表或频繁操作的小表，ddl 都要非常慎重，最好在业务低峰期执行。
-   设计上要尽可能避免大事务，大事务不仅仅会带来各种锁问题，还好引起复制延迟 / 回滚空间爆满等各类问题。
-   要及时提交事务，经常发现客户端设置了事务手工提交，但 sql 执行后忘记点击提交按钮，导致事务长时间无法提交。建议监控实例中的长事务，避免由于各种原因导致事务没有及时提交。

作者：翟振兴