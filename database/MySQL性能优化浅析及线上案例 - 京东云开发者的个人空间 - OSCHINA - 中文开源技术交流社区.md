### 作者：京东健康 孟飞

### **1、 数据库性能优化的意义**

业务发展初期，数据库中量一般都不高，也不太容易出一些性能问题或者出的问题也不大，但是当数据库的量级达到一定规模之后，如果缺失有效的预警、监控、处理等手段则会对用户的使用体验造成影响，严重的则会直接导致订单、金额直接受损，因而就需要时刻关注数据库的性能问题。

### **2、 性能优化的几个常见措施**

数据库性能优化的常见手段有很多，比如添加索引、分库分表、优化连接池等，具体如下：

| **序号** | **类型** | **措施** | **说明** | | 1 | 物理级别 | 提升硬件性能 | 将数据库安装到更高配置的服务器上会有立竿见影的效果，例如提高 CPU 配置、增加内存容量、采用固态硬盘等手段，在经费允许的范围可以尝试。 | | 2 | 应用级别 | 连接池参数优化 | 我们大部分的应用都是使用连接池来托管数据库的连接，但是大部分都是默认的配置，因而配置好超时时长、连接池容量等参数就显得尤为重要。 1、 如果链接长时间被占用，新的请求无法获取到新的连接，就会影响到业务。 2、 如果连接数设置的过小，那么即使硬件资源没问题，也无法发挥其功效。之前公司做过一些压测，但就是死活不达标，最后发现是由于连接数太小。 | | 3 | 单表级别 | 合理运用索引 | 如果数据量较大，但是又没有合适的索引，就会拖垮整个性能，但是索引是把双刃剑，并不是说索引越多越好，而是要根据业务的需要进行适当的添加和使用。 缺失索引、重复索引、冗余索引、失控索引这几类情况其实都是对系统很大的危害。 | | 4 | 库表级别 | 分库分表 | 当数据量较大的时候，只使用索引就意义不大了，需要做好分库分表的操作，合理的利用好分区键，例如按照用户 ID、订单 ID、日期等维度进行分区，可以减少扫描范围。 | | 5 | 监控级别 | 加强运维 | 针对线上的一些系统还需要进一步的加强监控，比如订阅一些慢 SQL 日志，找到比较糟糕的一些 SQL，也可以利用业务内一些通用的工具，例如 druid 组件等。 |

### **3、 MySQL 底层架构**

首先了解一下数据的底层架构，也有助于我们做更好优化。

![](https://s3.cn-north-1.jdcloud-oss.com/shendengbucket1/2023-01-02-16-00mTKWmzeBgcPR0JT.png)

**一次查询请求的执行过程**

我们重点关注第二部分和第三部分，第二部分其实就是 Server 层，这层主要就是负责查询优化，制定出一些执行计划，然后调用存储引擎给我们提供的各种底层基础 API，最终将数据返回给客户端。

### **4、MySQL 索引构建过程**

目前比较常用的是 InnoDB 存储引擎，本文讨论也是基于 InnoDB 引擎。我们一直说的加索引，那到底什么是索引、索引又是如何形成的呢、索引又如何应用呢？这个话题其实很大也很小，说大是因为他底层确实很复杂，说小是因为在大部分场景下程序员只需要添加索引就好，不太需要了解太底层原理，但是如果了解不透彻就会引发线上问题，因而本文平衡了大家的理解成本和知识深度，有一定底层原理介绍，但是又不会太过深入导致难以理解。

首先来做个实验：

创建一个表，目前是只有一个主键索引

CREATE TABLE \`t1\`(

a int NOT NULL,

b int DEFAULT NULL,

c int DEFAULT NULL,

d int DEFAULT NULL,

e varchar(20) DEFAULT NULL,

PRIMARYKEY(a)

)ENGINE=InnoDB

插入一些数据：

insert into test.t1 values(4,3,1,1,'d');

insert into test.t1 values(1,1,1,1,'a');

insert into test.t1 values(8,8,8,8,'h');

insert into test.t1 values(2,2,2,2,'b');

insert into test.t1 values(5,2,3,5,'e');

insert into test.t1 values(3,3,2,2,'c');

insert into test.t1 values(7,4,5,5,'g');

insert into test.t1 values(6,6,4,4,'f');

MYSQL 从磁盘读取数据到内存是按照一页读取的，一页默认是 16K，而一页的格式大概如下。

![](https://s3.cn-north-1.jdcloud-oss.com/shendengbucket1/2023-01-02-16-08QWTVsoP8D7X66W9.png)

每一页都包括了这么几个内容，首先是页头、其次是页目录、还有用户数据区域。

1）刚才插入的几条数据就是放到这个用户数据区域的，这个是按照主键依次递增的单向链表。

2）页目录这个是用来指向具体的用户数据区域，因为当用户数据区域的数据变多的时候也就会形成分组，而页目录就会指向不同的分组，利用二分查找可以快速的定位数据。

**当数据量变多的时候，那么这一页就装不下这么多数据，就要分裂页，而每页之间都会双向链接，最终形成一个双向链表。**

页内的单向链表是为了查找快捷，而页间的双向链表是为了在做范围查询的时候提效，下图为示意图，其中其二页和第三页是复制的第一页，并不真实。

![](https://s3.cn-north-1.jdcloud-oss.com/shendengbucket1/2023-01-02-16-13nSpjcdYaNZFuSgf.png)

而如果数据还继续累加，光这几个页也不够了，那就逐步的形成了一棵树，也就是说索引 B-Tree 是随着数据的积累逐步构建出来的。

![](https://s3.cn-north-1.jdcloud-oss.com/shendengbucket1/2023-01-02-16-14yvxfngBP1iHWkdE.png)

最下边的一层叫做叶子节点，上边的叫做内节点，而叶子节点中存储的是全量数据，这样的树就是**聚簇索引**。一直有同学的理解是说索引是单独一份而数据是一份，其实 MySQL 中有一个原则就是**数据即索引、索引即数据**，真实的数据本身就是存储在聚簇索引中的，所谓的**回表就是回的聚簇索引**。

但是我们也不一定每次都按照主键来执行 SQL 语句，大部分情况下都是按照一些业务字段来，那就会形成别的索引树，例如，如果按照 b,c,d 来创建的索引就会长这样。

![](https://s3.cn-north-1.jdcloud-oss.com/shendengbucket1/2023-01-02-16-19g31nbf4qQPfj62gk.png)

推荐 1 个网站，可以可视化的查看一些算法原型：

目录：

[https://www.cs.usfca.edu/~galles/visualization/Algorithms.html](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fwww.cs.usfca.edu%2F%7Egalles%2Fvisualization%2FAlgorithms.html)

B + 树

[https://www.cs.usfca.edu/~galles/visualization/BPlusTree.html](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fwww.cs.usfca.edu%2F%7Egalles%2Fvisualization%2FBPlusTree.html)

而在 MySQL 官网上介绍的索引的叶子节点是双向链表。

![](https://s3.cn-north-1.jdcloud-oss.com/shendengbucket1/2023-01-02-16-50Xp2gUL29TfINoHGQ.png)

**关于索引结构的小结：**

对于 B-Tree 而言，叶子节点是没有链接的，而 B+Tree 索引是单向链表，但是 MySQL 在 B+Tree 的基础之上加以改进，形成了双向链表，双向的好处是在处理 > <，between and 等 ' 范围查询 ' 语法时可以得心应手。

### **5、MySQL 索引的一些使用规范**

1、 只为用于搜索、排序或分组的列创建索引。

重点关注 where 语句后边的情况

2、 当列中不重复值的个数在总记录条数中的占比很大时，才为列建立索引。

例如手机号、用户 ID、班级等，但是比如一张全校学生表，每条记录是一名学生，where 语句是查询所有’某学校‘的学生，那么其实也不会提高性能。

3、 索引列的类型尽量小。

无论是主键还是索引列都尽量选择小的，如果很大则会占据很大的索引空间。

4、 可以只为索引列前缀创建索引，减少索引占用的存储空间。

alter table single\_table add index idx\_key1(key1(10))

5、 尽量使用覆盖索引进行查询，以避免回表操作带来的性能损耗。

select key1 from single\_table order by key1

6、 为了尽可能的少的让聚簇索引发生页面分裂的情况，建议让主键自增。

7、 定位并删除表中的冗余和重复索引。

冗余索引：

单列索引：（字段 1）

联合索引：（字段 1 字段 2）

重复索引：

在一个字段上添加了普通索引、唯一索引、主键等多个索引

### **6、 执行计划**

![](https://s3.cn-north-1.jdcloud-oss.com/shendengbucket1/2023-01-02-17-04AsiLDlzuIVThmzJ.png)

其中常用的是：

possible\_keys: 可能用到的索引

key: 实际使用的索引

rows：预估的需要读取的记录条数

### **7、 线上案例**

**案例 1：**

在建设互联网医院系统中，问诊单表当时量级 23 万左右，其中有一个 business\_id 字符串字段，这个字段用来记录外部订单的 ID，并且在该字段上也加了索引，但是 ' 根据该 ID 查询详情 ' 的 SQL 语句却总是时好时坏，性能不稳定，快则 10ms，慢则 2 秒左右，SQL 大体如下：

**select 字段 1、字段 2、字段 3 from nethp\_diag where business\_Id = ?**

因为 business\_id 是记录第三方系统的订单 ID，为了兼容不同的第三方系统，因而设计成了**字符串类型**，但如果传入的是一个数字类型是无法使用索引的，因为 MySQL 只能将**字符串转数字，而不能将数字转字符串**，由于外部的 ID 有的是数字有的是字符串，因而导致索引一会可以走到，一会走不到，最终导致了性能的不稳定。

**案例 2：**

在某次大促的当天，突然接到 DBA 运维的报警，说数据库突然流量激增，CPU 也打到 100% 了，影响了部分线上功能和体验，遇到这种情况当时大部分人都比较紧张，下图为当时的数据库流量情况：

![](https://s3.cn-north-1.jdcloud-oss.com/shendengbucket1/2023-01-02-17-09aunMVpr2IPGjAXF.png)

![](https://s3.cn-north-1.jdcloud-oss.com/shendengbucket1/2023-01-02-17-11RVc65tq5461v8q461.png)

相关 SQL 语句：

<!-- 统计医患下过去 24 小时内开的电子病历总数 -->

<select id="getCountByDPAndTime" resultType="integer">

select count(1)

from jdhe\_medical\_record

where status = 1 and is\_test = #{isTest,jdbcType=INTEGER} and electric\_medical\_record\_status in (2,3)

**<if test="patientId != null">**

and patient\_id = #{patientId,jdbcType=BIGINT}

**</if>**

**<if test="doctorPin != null">**

and doctor\_pin = #{doctorPin,jdbcType=VARCHAR}

**</if>**

and created >#{dateStart,jdbcType=TIMESTAMP};

</select>

**当时的索引情况**

![](https://s3.cn-north-1.jdcloud-oss.com/shendengbucket1/2023-01-02-17-132CN00k0HWsH13ENJ.png)

**当时的执行计划**

![](https://s3.cn-north-1.jdcloud-oss.com/shendengbucket1/2023-01-02-17-13PY13kip9nioeMPsB.png)

其实在 patientId 和 doctor\_pin 两个字段上是有索引的，但是由于线上情况的改变，导致 test 判断没有进入，这样的通用查询导致这两个字段没有设置上，进而导致了数据库扫描的量激增，对数据库产生了很大压力。

**案例 3：**

2020 年某日上午收到数据库 CPU 异常报警，对线上有一定的影响，后续检查数据库 CPU 情况如下，从 7 点 51 分开始，CPU 从 8% 瞬间达到 99.92%，丝毫没有给程序员留任何情面。

![](https://s3.cn-north-1.jdcloud-oss.com/shendengbucket1/2023-01-02-17-17zTgCyHN6nuyNyzu.png)

当时的 SQL 语句：

select rx\_id, rx\_create\_time from nethp\_rx\_info where rx\_status = 5 and status = 1 and rx\_product\_type = 0 and (parent\_rx\_id = 0 or parent\_rx\_id is null) and business\_type != 7 and vender\_id = 8888 order by rx\_create\_time asc limit 1;

当时的索引情况:

PRIMARY KEY (\`id\`), UNIQUE KEY \`uniq\_rx\_id\` (\`rx\_id\`), KEY \`idx\_diag\_id\` (\`diag\_id\`), KEY \`idx\_doctor\_pin\` (\`doctor\_pin\`) USING BTREE, KEY \`idx\_rx\_storeId\` (\`store\_id\`), KEY \`idx\_parent\_rx\_id\` (\`parent\_rx\_id\`) USING BTREE, KEY \`idx\_rx\_status\` (\`rx\_status\`) USING BTREE, KEY \`idx\_doctor\_status\_type\` (\`doctor\_pin\`, \`rx\_status\`, \`rx\_type\`), KEY \`idx\_business\_store\` (\`business\_type\`, \`store\_id\`), KEY \`idx\_doctor\_pin\_patientid\` (\`patient\_id\`, \`doctor\_pin\`) USING BTREE, KEY \`idx\_rx\_create\_time\` (\`rx\_create\_time\`)

当时这张表量级 2000 多万，而当这条慢 SQL 执行较少的时候，数据库的 CPU 也就下来了，恢复到了 49.91%，基本可以恢复线上业务，从而表象就是线上间歇性的一会可以开方一会不可以，这条 SQL 当时总共执行了 230 次，当时的 CPU 情况也是忽高忽低，伴随这条 SQL 语句的执行情况，从而最终证明 CPU 的飙升是由于这条慢 SQL。当线上业务逻辑复杂的时候，你很难第一时间知道到底是由于那条 SQL 引起的，这个就需要对业务非常熟悉，对 SQL 很熟悉，否则就会白白浪费大量的排查时间。

最后的排查结果：

在头天晚上的时候**添加了一条索引 rx\_create\_time**，当时没事，但是第二天却出了事故。

![](https://s3.cn-north-1.jdcloud-oss.com/shendengbucket1/2023-01-02-17-19ZhrSo19Lv0WHjjjw.png)

加索引前后走的索引不同，一个是走的 rx\_status（处方审核状态）单列索引，一个是走的 rx\_create\_time (处方提交事件) 单列索引，这个就要回到业务，因为处方状态是个枚举，且枚举范围不到 10 个，也就说线上 29,000,000 的数据量也就是被分成了不到 10 份，rx\_status=5 的值是其中一份，因而通过这个索引就可以命中很多行，这是业务规则，再套用 MySQL 的特性，主要是以下几条：

**1、没加新索引 rx\_create\_time 的时候**，由于 order by 后边没有索引，就看 where 条件中是否有合适的索引，查询选择器选定 rx\_status 这个单列索引，而 rx\_status=5 这个条件下限制的数据行在索引中是连续，即使需要的 rx\_id 不在索引中，再回主键聚簇索引也来得及，由于 order by 后边没有索引，所以走磁盘级别的排序 filesort，高峰积压的时候处方就 1 万到 2 万，跑到了 100ms, 白天低谷的时候几百单也就 20ms。

**2、新加索引之后，就分两种情况：**

2.1、加索引是在晚上，当前命中的行数比较少，由于当天晚上的时候待审核的处方确实很少，也就是 rx\_status=5 的确实很少，查询优化器感觉反正没多少行，排序不重要，因而就还是选择 rx\_status 索引。

2.2、第二天白天，待审核的处方数量很多了（rx\_status=5 的数据量多了），当时可以命中几万数据，如果当前命中的行数比较多，查询优化器就开始算成本，感觉排序的成本会更高，那就优先保排序吧，所以就选择 rx\_create\_time 这个字段，但是这个索引树上没有别的索引字段的信息，没办法，几乎每条数据都要回表，进而引发了灾难。

### **8、 推荐用书**

这本书以一种诙谐幽默的风格写了 MySQL 的一些运行机制，非常适合阅读，理解成本大幅降低。

[https://item.jd.com/13009316.html](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fitem.jd.com%2F13009316.html)

![](https://s3.cn-north-1.jdcloud-oss.com/shendengbucket1/2023-01-02-17-29r17QEqCZh5Q17ja.png)

[https://item.jd.com/10066181997303.html](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fitem.jd.com%2F10066181997303.html)

![](https://s3.cn-north-1.jdcloud-oss.com/shendengbucket1/2023-01-02-17-30OTMpja1zOi6QD6C.png)

### **9、一些感悟**

关于数据库的性能优化其实是一个很复杂的大课题，很难通过一篇帖子讲的很全面和深刻，这也就是为什么我的标题是‘浅析’，程序员的成长一定是要付出代价和成本，因为只有真的在一线切身体会到当时的紧张和压力，对于一件事情才能印象深刻，但反之也不能太过于强调代价，如果可以通过一些别人的分享就可以规避一些自己业务的问题和错误的代价也是好的。