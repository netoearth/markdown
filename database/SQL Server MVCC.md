![图片](https://mmbiz.qpic.cn/mmbiz_jpg/RGa0fiaTicQOpicESkNicCfia3IgOAOavX2LTicGKqLewf9nq49OUSnVWsoRsQJeCu1Z0mXKvpZPSpfBiaZbJzibyq14xA/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1)

ACID 是事务四大属性，理论上 Serializable 隔离级别是最能保证事务属性的。但在实际场景中，Serializable 隔离级别不利用高并发的访问。关系型数据库为了能在高并发下保证数据的一致性，提供了一种叫 Multi-Version Concurrency Control (MVCC)的机制。主流的关系型数据库系统都实现了 MVCC，只是实现的方式不一样。

**你知道 SQL Server 是如何实现 MVCC 的吗？**

SQL Server 数据库引擎使用锁或者行版本控制机制来确保事务的完整性，并在多个用户同时访问数据时保持数据库的一致性。锁和行版本控制可以防止用户读取未提交的数据，还可以防止多个用户尝试同时更改同一数据。如果不进行锁定或行版本控制，对数据执行的查询可能会返回数据库中尚未提交的数据，从而产生意外的结果。

下面，我们将一步步进行测试，理解 SQL Server 如何实现 MVCC 的。

```
--创建简单的测试数据
```

们构建2个事务，按顺序从上至下执行，分别对SQL定义了编号 SQL01~SQL05，以方便我们说明，后面测试皆以此模板为参考。

<table><tbody><tr><td><span><strong><span>事务一<br></span></strong></span></td><td><span><strong><span>事务二<br></span></strong></span></td></tr><tr><td><p><span>begin tran</span></p><p><span>--SQL01</span></p><p><span>select * from</span><span> dbo.Test</span></p><p><span>where </span><span>name = </span><span>'kk'</span></p></td><td><br></td></tr><tr><td><br></td><td><p><span>begin tran</span></p><p><span>--SQL02</span></p><p><span>update </span><span>dbo.Test</span></p><p><span>set </span><span>info = </span><span>'更改'</span></p><p><span>where </span><span>name = </span><span>'kk'</span></p></td></tr><tr><td><p><span>--SQL03</span></p><p><span>select</span><span> *&nbsp;</span></p><p><span>from </span><span>dbo.Test </span><span>WITH(READPAST)</span></p><p><span>where </span><span>name =</span><span> 'kk'</span></p><p><span>--SQL04</span></p><p><span>select </span><span>*&nbsp;</span></p><p><span>from</span><span> dbo.Test </span><span>WITH(NOLOCK)</span></p><p><span>where </span><span>name =</span><span> 'kk'</span></p><p><span>--SQL05</span></p><p><span>select </span><span>* </span><span>from </span><span>dbo.Test</span></p><p><span>where </span><span>name = </span><span>'kk'</span></p></td><td><br></td></tr></tbody></table>

**测试一：**

首先是最常见的隔离级别 READ COMMITTED ，了解事务之间是如何隔离的。把事务隔离级别更改为 READ COMMITTED，并按照示例中的2个事务顺序执行。

```
SET TRANSACTION ISOLATION LEVEL READ COMMITTED;
```

**现象：**

-   SQL01开始正常执行，结果正常；
    
-   SQL02可正常执行，更改数据；
    
-   SQL03可正常执行，但无查询结果（该行被排除在外）；
    
-   SQL04可正常执行，结果为更新后的值；
    
-   SQL05发生堵塞，请求共享锁，等待事务二结束；
    

**结论：**在隔离级别 READ COMMITTED下，事务中数据的更改只有在提交的情况下才能被读取，即不可重复读。这种情况在 SQL Server 是比较常见的，很容易引起堵塞或者死锁，不利于高并发。

**测试二：**

关系型数据库对数据一致性的保障，还有一种隔离脚本叫可重复读（REPEATABLE READ），那我们更改为这个隔离级别会怎样呢？我们更改事务隔离级别，其他步骤与上面的测试一样。

```
SET TRANSACTION ISOLATION LEVEL REPEATABLE READ
```

**现象：**

-   SQL01开始正常执行，结果正常；
    
-   SQL02发生堵塞，请求排他锁，等待事务一结束；
    
-   SQL03可正常执行，结果为更新前的值；
    
-   SQL04可正常执行，结果为更新前的值；
    
-   SQL05可正常执行，结果为更新前的值；
    

**结论：**在隔离级别 REPEATABLE READ下，同一个事务可以重复读取了，但是其他事务的更改却被堵塞了。也就是说，REPEATABLE READ 中查询的“优先级”高，READ COMMITTED 中更改的“优先级”高。那这样的隔离级别似乎也没多大好处。

我们可以看看这两个事务的锁请求情况。  

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

**测试三：**

除了四大事务隔离级别 Read uncommitted、Read committed、Repeatable read、Serializable ，SQL Server 提供了一种独特的快照隔离级别 Snapshot，以此来实现行版本控制，也就是我们常说的MVCC。该级别在读操作期间不使用共享锁。必须将 ALLOW\_SNAPSHOT\_ISOLATION 数据库选项设置为 ON，事务才能在快照隔离下运行。

```
SET TRANSACTION ISOLATION LEVEL SNAPSHOT
```

**现象：**

-   SQL01开始正常执行，结果正常；
    
-   SQL02可正常执行，更改数据；
    
-   SQL03执行错误，此隔离级别不支持READPAST；
    
-   SQL04可正常执行，结果为更新后的值；
    
-   SQL05可正常执行，结果为更新前的值；
    

**结论：**在快照隔离级别Snapshot下，同一个事务可以重复读取，读取的是更改之前的版本值，并且也不再堵塞其他事务的更新，实现了MVCC。

我们看看事务一中获取的锁情况，发现只有一个数据库共享锁，也就是所谓的“读锁”。事务一并没有更细粒度的锁，也就不会堵塞其他事务的更改操作了。

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

其实，事务使用的是行版本控制而不是共享锁，tempdb 数据库必须具有足够的空间用于版本存储区。在 tempdb 已满的情况下，更新操作将停止生成版本，并继续执行，但是因为所需的特定行版本不再存在，读取操作可能会失败。这将影响诸如触发器、MARS 和联机索引的操作。只要活动事务需要访问行版本，就必须存储行版本。后台线程每隔一分钟删除一次不再需要的行版本，从而释放 tempdb 中的版本空间。

因此，在大批量数据操作时，尽量分多批事务进行处理，并及时提交/回滚事务，避免tempdb中的行版本数据过大。

查看tempdb行版本记录大小，此时可以看到版本存储区域分配了数据页。

```
SELECT
```

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

Snapshot 是特定的事务隔离级别，事务使用行版本控制提供事务级读取一致性。

**测试四：**

SQL Server 还有另一种隔离级别 read committed snapshot，是在隔离级别 READ COMMITTED中设置数据库 READ\_COMMITTED\_SNAPSHOT为ON时出现，READ\_COMMITTED 事务使用行版本控制提供语句级读取一致性。

```
ALTER DATABASE dbname SET READ_COMMITTED_SNAPSHOT ON;
```

**现象：**

-   SQL01开始正常执行，结果正常；
    
-   SQL02可正常执行，更改数据；
    
-   SQL03可正常执行，结果为更新前的值；
    
-   SQL04可正常执行，结果为更新后的值；
    
-   SQL05可正常执行，结果为更新前的值；
    

**结论：**与“测试三”一样，在快照隔离级别read committed snapshot下，同一个事务可以重复读取，读取的是更改之前的版本值，并且也不再堵塞其他事务的更新，实现了MVCC。

___

以上我们已经测试了四种事务隔离级别，其中两种隔离级别实现了MVCC。在行版本控制中，大大提高了数据库的并发访问。

<table><tbody><tr><td><span>测试一</span></td><td><span>SET TRANSACTION ISOLATION LEVEL READ COMMITTED</span></td></tr><tr><td><span>测试二</span></td><td><span>SET TRANSACTION ISOLATION LEVEL REPEATABLE READ</span></td></tr><tr><td><span>测试三</span></td><td><p><span>SET TRANSACTION ISOLATION LEVEL SNAPSHOT</span></p><p><span>ALTER DATABASE dbname SET ALLOW_SNAPSHOT_ISOLATION ON</span></p></td></tr><tr><td><span>测试四</span></td><td><p><span>SET TRANSACTION ISOLATION LEVEL READ COMMITTED</span></p><p><span>ALTER DATABASE dbname SET READ_COMMITTED_SNAPSHOT ON</span></p></td></tr></tbody></table>

对于大多数应用程序，建议使用行版本控制的读提交隔离（read committed snapshot）而不是快照隔离（snapshot），原因如下：

-   读提交隔离比快照隔离消耗更少的tempdb空间。
    
-   读提交隔离适用于分布式事务，而快照隔离则不然。
    
-   读提交隔离适用于大多数现有应用程序，无需任何更改，仍然使用默认隔离级别read committed。行版本控制由数据库选项READ\_COMMITTED\_SNAPSHOT控制。
    
-   读提交隔离不检查更新冲突，快照隔离容易受到更新冲突的影响。
    
-   读提交隔离提供语句级读取一致性。事务中的每个语句执行时，都会获取一个新的数据快照并保持每个语句的一致性，直到语句完成执行。快照隔离提供事务级别的读取一致性
    

**最近文章推荐：**

-   [SQL Server 安装与访问](http://mp.weixin.qq.com/s?__biz=MzIwMDkwNDA3MA==&mid=2247484747&idx=1&sn=95c3f09cd60fbd15974f211f4aaf7d33&chksm=96f759a2a180d0b4f75dfb929a43ccca1e700f8f71984197305fe8ec261d383d9b94d7af3c42&scene=21#wechat_redirect)  
    
-   [SQL Server 登录账号与数据库用户](http://mp.weixin.qq.com/s?__biz=MzIwMDkwNDA3MA==&mid=2247484712&idx=1&sn=e7c6de6af959816a35254246dbd34798&chksm=96f759c1a180d0d76fad41eafd73521d70ebc35464c4e71fec97660cbfe1b1ac150554e1dd11&scene=21#wechat_redirect)  
    
-   [Windows 电源计划对 SQL 查询影响](http://mp.weixin.qq.com/s?__biz=MzIwMDkwNDA3MA==&mid=2247484689&idx=1&sn=294cc788906933c14306b7848a1f8f25&chksm=96f759f8a180d0eee800fa0ba077702ff8a5709ab1815c6e6fae74fa6a3b0798fe04b3db1e66&scene=21#wechat_redirect)  
    
-   [SQL Server 存储过程最佳实践](http://mp.weixin.qq.com/s?__biz=MzIwMDkwNDA3MA==&mid=2247484676&idx=1&sn=4c04f63bff733eda0e9adaba7d6805ef&chksm=96f759eda180d0fbbc52c7b33dc572b3f824ce8ebd8b477b1eec41db5c4392d68aad8b4b8b4b&scene=21#wechat_redirect)  
    
-   [SQL Server 数据库精确点还原](http://mp.weixin.qq.com/s?__biz=MzIwMDkwNDA3MA==&mid=2247484670&idx=1&sn=bdb8c11df9c197eedd5887f7dcebedbb&chksm=96f75817a180d101336909b9cd5008723e2e8bc26d5c97e4130ec66c4863ad82a6839da755c1&scene=21#wechat_redirect)  
    
-   [SQL Server NUL 设备](http://mp.weixin.qq.com/s?__biz=MzIwMDkwNDA3MA==&mid=2247484662&idx=1&sn=7fe5dec370f21314296cdef04bf50dc3&chksm=96f7581fa180d10994f5f8fa91c6982180d18e25f514569001852138fc60f09cbc6686882c7f&scene=21#wechat_redirect)  
    
-   [SQL Server AlwaysOn AG 无AD域无DNS的跨子网集群实战](http://mp.weixin.qq.com/s?__biz=MzIwMDkwNDA3MA==&mid=2247484656&idx=1&sn=5fc437334c05061031a30c12d671e08c&chksm=96f75819a180d10f1d7db2d16e79508aaebc01c6f8960eaa3a2d4faf67953583f4b62befc812&scene=21#wechat_redirect)  
    

**历史**文章推荐：****

-   [Windows Cluster 投票权问题](http://mp.weixin.qq.com/s?__biz=MzIwMDkwNDA3MA==&mid=2247484580&idx=1&sn=8cc419155b1eae36658635b60e1a6013&chksm=96f7584da180d15bb1ca47011f3ccfdd4b2d2d60832401a501d8310a3aa7d795074392e12f7e&scene=21#wechat_redirect)  
    
-   [Windows Cluster 分布式算法](http://mp.weixin.qq.com/s?__biz=MzIwMDkwNDA3MA==&mid=2247484564&idx=1&sn=a5d35f13d0186b019f30a3df1074caeb&chksm=96f7587da180d16b8de3da7e86e22af799e00d9bbf56105cae93d8b1f5f56eb68f2c11696194&scene=21#wechat_redirect)  
    
-   [SQL Server 高可用方案介绍](http://mp.weixin.qq.com/s?__biz=MzIwMDkwNDA3MA==&mid=2247484556&idx=1&sn=cbc6b6e32b81e6742ddd829c1feb8cb2&chksm=96f75865a180d173e9537b6c02c8147e9fd415b34c1da3022e68923d9d1046d8efa5ab7aabcd&scene=21#wechat_redirect)  
    
-   [SQL Server 增量数据同步](http://mp.weixin.qq.com/s?__biz=MzIwMDkwNDA3MA==&mid=2247484114&idx=1&sn=606a5ceaff089837c117b5fcee766a1d&chksm=96f75e3ba180d72d1a5b512107aee8d1d8269bf0411b9fccda23a55eed1d1d5b35841be1ed85&scene=21#wechat_redirect)  
    
-   [SQL Server 配置 Zabbix 监控](http://mp.weixin.qq.com/s?__biz=MzIwMDkwNDA3MA==&mid=2247484046&idx=1&sn=5fb66d86b102a6205b2e6500ff547d2f&chksm=96f75e67a180d771673d034a1a7e6da21049e54864e525042fa8578c2545aef26d2f346f3764&scene=21#wechat_redirect)
    

___

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)