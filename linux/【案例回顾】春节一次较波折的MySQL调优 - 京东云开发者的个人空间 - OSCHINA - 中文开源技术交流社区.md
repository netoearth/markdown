春节长假某日，阳光明媚，春暖花开，恰逢冬奥会开幕，想着一定是一个黄道吉日，必能顺风顺水。没想到却遇到一个有点小波折的客户报障。

01

故障起因

故障起因是客户前一天从自建 MySQL 迁移到云上 RDS，在执行某个并发较高的业务时出现了大量锁等待，客户当时升级了实例到最高规格，但故障依旧。客户反馈升级后的实例规格比自建实例高了一倍，自建实例上从未发生过类似情况。后客户根据当时的业务故障模拟了现场，主要是并发执行如下存储过程的时候性能很差：

![](https://p3-sign.toutiaoimg.com/tos-cn-i-qvj2lq49k0/e3f2ae4438754015a286277b760d8663~noop.image?_iz=58558&from=article.pc_detail&x-expires=1662600674&x-signature=JYrbUlE7RGglFUd4y9K2LPLItMc%3D)

02

初步诊断

从存储过程的逻辑看，比较简单，主要涉及两个 SQL，一个从表 t（隐藏了真实表名）中 meeting\_id 根据传入参数值查询，具体的入参由字符型变量 p\_meeting\_id 带入；另外一个根据 meeting\_id 和刚查出的 phone\_id 去更新 t 中的 phone\_id 为 phone\_id+3。表 t 数据量约 40w 左右。

第一感觉这是个简单问题，估计两个 SQL 的 meeting\_id 索引没有生效，查询表上索引后果然发现 meeting\_id 和 phone\_id 上没有索引，建议客户在两个字段上分别创建了索引，且 meeting\_id 为主键。此时用户执行模拟的并发脚本反馈速度有了明显提升，200 个并发最高执行时间 40s 左右，但模拟 500 个并发的时候，超过了 8 分钟还没有执行完。用户反馈在自建 MySQL 上并发 500 执行都是秒级完成。此时在控制台看，这个存储过程在慢查询日志中批量出现，且扫描行数巨大，客户端已经完全 hang 住:

![](https://p3-sign.toutiaoimg.com/tos-cn-i-qvj2lq49k0/004b718615e54cb08e7cb4ef2208961c~noop.image?_iz=58558&from=article.pc_detail&x-expires=1662600674&x-signature=Ap0OOwUknYlSmkCMdAsDEvMgfU0%3D)

03

进一步优化

虽然优化有了初步的效果， 但距离客户自建环境性能描述还差距很大，由于并发高， 从监控看测试期间 CPU 到了 100%，怀疑参数 innodb\_thread\_concurrency 的设置可能不当。此参数的作用是控制 InnoDB 的并发线程上限。也就是说，一旦并发线程数达到这个值，InnoDB 在接收到新请求的时候，就会进入等待状态，直到有线程退出。RDS 默认值为 0，也就是没有限制上限，在高并发的场景下可能会产生较多的上下文切换，导致 CPU 升高。和客户咨询了一下，他们自建环境的值设置为 32，建议他们将 RDS 的值也改为 32 再看看效果。客户很快反馈，修改后的确有效果，500 个并发在 3 分钟内完成，没有再发生 hang 住不动的情况，性能有了进一步的提升。但参数 innodb\_thread\_concurrency 进一步调整效果不明显。

04

加 trace 诊断

客户看到性能不断提升也很有信心，但和自建环境差距还是很大，还有哪里可能有问题？突然想到，创建索引后，在控制台的慢查询列表中看到很多存储过程的调用 sql，且扫描记录数巨大，如果是走 meeting\_id 唯一索引，应该扫描很少的记录数才对，难道没有走索引？或者没有走 meeting\_id 主键索引？联系客户，希望提供测试环境登陆测试。

在测试环境，首先希望验证一下两个 SQL 的执行计划到底是怎么样的。登陆实例后，分别对两个存储过程中的 SQL 执行 explain，发现走的确实是主键（meeting\_id）：

![](https://p3-sign.toutiaoimg.com/tos-cn-i-qvj2lq49k0/07cdd8853b1f491f89c651ecf51bccce~noop.image?_iz=58558&from=article.pc_detail&x-expires=1662600674&x-signature=59f0C64RHSl0JsYk%2FzhaGOVLldk%3D)

为了进一步确认 SQL 在存储过程中的实际执行计划，修改了一下测试的存储过程逻辑，加入了 SQL 执行的 explain 结果和实际执行的 trace，过程中主要增加的代码如下：

![](https://p3-sign.toutiaoimg.com/tos-cn-i-qvj2lq49k0/ba0573e548e74c3c9851d0b3f8523bd2~noop.image?_iz=58558&from=article.pc_detail&x-expires=1662600674&x-signature=joA11Z5gf4LTDmW%2BDaQQbNux8vI%3D)

执行计划结果如下：

![](https://p3-sign.toutiaoimg.com/tos-cn-i-qvj2lq49k0/bec13dabb5324b268c53a0fe6c1bf09b~noop.image?_iz=58558&from=article.pc_detail&x-expires=1662600674&x-signature=Q4lQhV25tEpJ86%2BeiOyPD5wYYMo%3D)

从结果看，两个 SQL 居然真的没有走主键 meeting\_id 索引，而是都走了 phone\_id 这个普通的二级索引，其中第一个查询 SQL 走的索引全扫描，扫描记录数 rows 为 397399，和表的记录数一致，显然走了全索引扫描，虽然比全表扫描好一些，但效率仍然低下；另外一个 update 的 SQL 走了正常的索引扫描，rows 只有 2，性能高效。为什么两个 SQL 没有走 meeting\_id 这个主键索引呢？看 trace 打印的部分内容：

![](https://p3-sign.toutiaoimg.com/tos-cn-i-qvj2lq49k0/4d21da66b3a34eb280ac723cd1fa3bd2~noop.image?_iz=58558&from=article.pc_detail&x-expires=1662600674&x-signature=dOKjOK%2Fr7PSQRXWwRTKCW4r59LE%3D)

trace 显示两个 SQL 在优化器分析时，将 meeting\_id 做了隐式转换，转换函数为 convert ('meeting\_id' using utf8mb4)，也就是将 meeting\_id 做了字符集的转换，熟悉索引机制的同学都清楚，这种情况下优化器是不会走 meeting\_id 索引的。这也可以解释了客户第一次创建索引的时候为啥有性能提升，但效果并不明显，原因就是只有 update 语句真正用到了索引带来的性能提升，而且是 phone\_id 索引带来的提升，不是性能更高的主键 meeting\_id。

05

真相大白

现在聚焦到最关键的问题，meeting\_id 为啥要做字符集的隐式转换？查看了一下实例相关字符集的设置：

1.  表和列的字符集都为 utf8；
2.  表所在库的字符集为 utf8mb4；
3.  server 字符集（（character\_set\_server））为 utf8
4.  character\_set\_client/character\_set\_connection/character\_set\_results 为 utf8mb4

果然，server、database、table 的字符集不完全一致，猜想一下实际流程应该是这样的：存储过程中传入的字符参数字符集为 utf8mb4，和表中字符集为 utf8 的字段 meeting\_id 比较时，meeting\_id 做了字符集的隐式转换，转换为 utf8mb4 后再和输入参数比较，从而导致 meeting\_id 上的索引无法使用。

根据这个猜测，建议用户将表的字符集更改为 utf8mb4，这样应该可以避免字符集的转换。由于这个功能还未上线，用户直接对 表做了字符集的修改：

```
alter table zm_meeting convert to character set utf8mb4;
```

修改后让用户再次测试，预期效果终于出现，并发 500 测试在秒级完成，trace 查看执行计划，都走了 meeting\_id 的主键索引，隐式转换也随之消失，性能问题得到了彻底解决。

06

后续思考

存储过程的入参为啥使用了 utf8mb4？这是本次案例的核心，查阅 mysql 文档，存储过程介绍里面有一段描述：

![](https://p3-sign.toutiaoimg.com/tos-cn-i-qvj2lq49k0/297be36231fc49688e657d0fdcef58e2~noop.image?_iz=58558&from=article.pc_detail&x-expires=1662600674&x-signature=pimhA8CO7tfVGp54rxf%2FJESQouM%3D)

简单说，就是存储过程的字符型参数，如果没有显式指定字符集，默认将会使用所在数据库的字符集，而本案例中表所在的数据库字符集为 utf8mb4，所以参数默认使用了 utf8mb4，导致了匹配过程的隐式转换。存储过程外直接写 SQL 为什么没有这种情况发生，我猜测比较的字符串应该会自动匹配‘=’左边表字段的字符集。

既然这样，理论上直接修改参数的字符集应该也可以达到同样结果，简单测试下，将存储过程参数加上表上的字符集属性：

```
CREATE  PROCEDURE `zm_sp_next_phone_id`(IN `p_meeting_id` VARCHAR(36) character set utf8)
```

测试结果如我们预期，不会产生隐式转换，执行计划正确。

问题虽然解决了，原因也找到了，但反思一下整个过程，如果用户的 server、库、表字符集能够保持一致，将完全可以避免这个故障。与字符集相关的类似故障也可以大概率避免，所以客户侧还是要有一定的设计规范；产品侧如果有一定的检查规则可以帮客户发现类似的隐患，对提升客户体验也是一种很有价值的服务。

作者：翟振兴