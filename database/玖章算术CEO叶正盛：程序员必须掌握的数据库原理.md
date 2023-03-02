![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E "图片16.png")  

SQL是数据库最常用的语言，也是数据库的核心资源开销，因此SQL优化是程序员的必备技能。

SQL优化通常分3个阶段，我们常称为SQL优化三板斧，这三板斧下去，90%的问题都可以解决。

第一是先找到问题SQL，有两种情况，如果是现在系统有问题，我们可以去看活跃的连接在干什么，把正在运行的SQL取出来。每种数据库通常都提供了查询当前活跃会话的接口，比如MySQL使用show processlist，Oracle可以查询v$session视图信息。如果说问题已经过去了，没有现场，可以通过Slow Log，或者TOP SQL，找到问题，这个需要数据库开启历史SQL记录功能。

第二是分析问题SQL，最主要先看SQL的执行计划，再去看整个数据库的IO访问量是不是符合预期，缓冲命中率怎么样，如果不是99%以上，可能都有问题，有些程序员看到内存都用完了，其实也不是什么问题，比如有10G的内存，如果你的数据量超过10GB，那内存很快都会用满，关键要看缓存命中率。另外是网络IO，多集群的话，需要关注网络传输的延迟和带宽容量。如果有锁，要分析SQL锁的模型。CPU重点要看一下排序、函数计算等操作。

最后是解决问题，就是优化SQL，可以修改SQL提升性能，如果没有索引就要增加索引，如果缓存命中率有问题就要调整内存配置。数据整理也是比较常见的，有些数据库历史数据非常多，影响了效率，要考虑怎么把历史数据归档，提升数据访问效率。

再有是提升硬件性能，最后是分布式改造，分布式改造比较困难，如果前面几个步骤能解决最好，实在不行再去考虑比较复杂的分布式改造。

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E "图片17.png")

接下来重点讲一下执行计划。有些初级程序员往往忽视这个问题，SQL执行计划是描述SQL详细执行路径和算法。

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E "图片18.png")

数据库通常支持使用EXPLAIN语法来查看SQL的执行计划，会展示出SQL具体的执行路径和算法，包括访问表的顺序，每张表使用索引访问还是全表扫描，每次执行预计返回的行数等等，信息会比较详细。

SQL访问单表主要有几种方式，包括：

主键：示例SQL：select \* from t where id=?

唯一索引：select \* from t where uk=?

普通索引：select \* from t where name=？

索引范围扫描：select \* from t where create\_time>?

全表扫描：select \* from t

对于多表连接，通常有以下几种JOIN算法：

**Nested loop join**：适合两张表有非常好的索引访问路径，如：select \* from t1 inner join t2 where t1.id=t2.id and t1.id=?

**Hash Join**：适合两张表都没有好的索引，需要全表扫描，并且是等值关联查询，如：select \* from t1 inner join t2 where t1.id=t2.id

Hash Join需要把一张表加载到内存后，另外一张表再通过Hash匹配到对应的记录，所以会需要临时内存，当临时内存不够时会产生临时磁盘转储的开销。为了解决Hash Join在分布式场景的问题，也诞生了Shuffle Hash Join和Broadcast Hash Join算法。

**Merge Join**：适合两张表都没有好的索引，需要全表扫描，支持非等值关联查询，关联字段最好是有排序，因为Merge Join算法是需要把两边数据排序再Merge 。

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E "图片19.png")

数据库如何生成SQL执行计划，如何选择最优的执行路径，通常是内部有个优化器的组件来完成。优化器类型有RBO和CBO两种。

**RBO**：Rule Base Optimizer，意思是基于规则的优化器逻辑，早期数据库都是采用RBO，实现比较简单，但很依靠程序员对数据库的逻辑理解，在SQL非常复杂的情况下很容易走到糟糕的执行计划。

**CBO**：Cost Base Optimizer，意思是基于成本的逻辑，优化器会计算各种不同的路径成本，最后选择一条最优的路径来执行。

举个例子，我们通常的交通方案有飞机、火车、汽车、轮船、步行，每种方案的速度都不同。如果我们要从杭州去北京，那么该选择那种交通方式？

对于RBO来说，它总是会选择飞机，不管是否发生交通管制，以及去机场的交通是否顺畅。

对于CBO来说，它会评估从出发点到杭州机场/火车站，北京火车站/机场到目的地的交通时间，综合评估一个最短时间的交通方案。

CBO是目前数据库主流的优化器模型，实现难度也更大，需要收集很多统计信息，如每张表的存储空间大小和记录数，字段的区分度等等，在数据经常变化时需要保证统计信息的准确性。

RBO和CBO通常都是在生成执行计划后就不能再修改，即使执行过程中发现严重偏差也不会改变，因此后来有些数据库在CBO的基础上实现了更智能的Adaptive的逻辑，意思是在执行过程中，如果发现有更优路径，那优化器可以调整执行计划。就好像本来选择坐飞机从杭州去北京，但是到了机场，发现航班大量延误，起飞时间完全不确定，这个时候可以调整计划，选择做火车出发。

开源数据库的优化器相对比较弱，商业数据库更领先。国内在这块技术相比国际落后很多。

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E "图片20.png")

接下来讲一下数据库的设计，基础知识是要掌握范式，在教科书里面非常多，这里不多讲 。反范式我觉得是比较有意思，尤其是表设计里面，有非常多的反范式的案例。比如说标记组合是比较常见的，如果有多个标记状态位，可能会用bit的数据类型去存储，这其实就已经违反了第一范式，因为第一范式要求字段的内容要原子性，不能拆分。实际上数据库内核里面，它用了很多bit类型这种反范式来设计元数据，从而提升存储和查询效率。

另外一个是日常习惯字段，拿身份证号码来举例，也是违反了第一范式，因为身份证号码已经包括所在的地区、出生年月、登记顺序以及校验码这4个信息。但是我们实际的表结构设计，还是会把身份证号码当做一个字段，不会说把身份证号码拆成4个字段，否则阅读习惯会不太方便。

还有计算列，典型的违反了第三范式，如商品的金额=单价\*数量，订单总金额，我们一般都会冗余去存储。

数仓里面就更多的反范式的案例，历史数据快照，会把相关的信息都保存下来，也是违反范式，但我们基本都这么干。

拆表也是一种常见设计，比如说一张表里面有常用和不常用的字段，如有个字段是大字段，我们就会把它拆成另外一张表。

这些其实都是比较常见的反范式设计的场景。所以部分场景下我们不用去特别纠结一定要遵循范式，如果说你觉得合理，业务逻辑实现可控，适当的冗余数据其实也没什么问题。

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E "图片21.png")

关于主键选择，比较推荐两种，一种是自增字段，比较适合内部系统的表，如果说是对外的互联网应用需要注意，有被猜测攻击的隐患。我们公司比较喜欢用雪花模型，根据时间序列加上顺序号产生主键，性能比较好，比较安全，实现相对复杂，需要应用程序产生。

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E "图片22.png")

接下来介绍几个常见的数据模型：

首先是平台型互联网数据模型，像淘宝、微信公众号、美团、BOSS直聘、滴滴等等，他们的数据模型都比较类似，主要包括以下对象。

**会员信息**：买家（淘宝、美团）、乘客（滴滴）、读者（公众号）、求职人（BOSS直聘）。

**服务提供者信息**：卖家（淘宝、美团）、博主（公众号）、司机（滴滴）、主播（抖音）、公司（BOSS直聘）

**服务内容信息：**商品、文章、微博、用车、视频、职位

**库存信息**：电商的概念

互联网平台根据以上几张表信息，通过搜索、推荐等各种匹配算法，最后产生订单信息，接着是付款信息、物流/配送信息，最后是服务反馈信息。

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E "图片23.png")

刚才是互联网的模型，企业软件不太一样，企业软件讲究两个东西，一个是安全，一个是流程规范。

安全核心是权限模型设计，比较标准的是RBAC（Role Base Access Control），我们可以参考这套模型演进。核心是理解用户、权限、角色三个重要的对象。按照RBAC设计问题不会太大，有可能会有扩展的东西。比如面向公司的用户，会有组织和部门的关系。权限会扩展出权限分组，角色会有继承等。另外一个是数据域权限，同样的权限与角色，但是能访问的是不同范围的数据，就会有数据域等高级定义。

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E "图片24.png")

我们再看企业软件里比较常见的流程规范，最常见的是工作流引擎。

首先有工单表，不管是请假、权限申请还是生产变更，都有一个工单来存储你做什么事情。

然后是流程模型定义表，例如数据库产生变更的流程，现在做生产发布，变更表结构，要提交变更申请，主管要审批，如果高风险还需要DBA的审批，这就是一个业务流程模型。

流程模型会包括基本定义信息，有哪几个节点，流转路径，根据什么条件流转，哪些人受理等等。

针对每个工单的流程处理，都有工单流转表，消息通知表，再加上用户权限系统，这样就可以形成工作流的待办工作表，基本上就是这么一个套路，大家理解这个概念，在企业级软件里就可以如鱼得水了，不会那么纠结。