摘要：越来越多的人在MySQL数据库使用UUID作为表的主键。大家都知道，对于MySQL的InnoDB存储引擎，主键非常重要！它对性能、内存和磁盘空间的影响巨大。

原文：https://blogs.oracle.com/mysql/post/mysql-uuids

作者： Frédéric Descamps，Oracle公司MySQL社区经理，知名MySQL布道师 。

![图片](https://mmbiz.qpic.cn/mmbiz_png/PO81lMIQTpj9eO1VrME1BSJjJmibe3wP3XPmEugYKUYcwc2JGKLG8t9vyEnmwDvEAk5fJWlmPVHl25S4d1D40ibg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

关于译者，姚远：

-   Oracle ACE（Oracle和MySQL数据库方向）
    
-   华为云MVP
    
-   《MySQL 8.0运维与优化》的作者
    
-   中国唯一一位Oracle高可用大师
    
-   拥有包括 Oracle 10g和12c OCM在内的20+数据库相关认证。
    
-   曾任IBM公司数据库部门经理
    
-   现在一家第三方公司任首席数据库专家，服务2万+客户。
    

01

—

小心两个坑

在InnoDB中使用UUID作为主键需要考虑两个问题：

1.  UUID的返回值通常是随机的，而InnoDB的表实质是以主键组织存储的索引，插入新的记录会造成表的再平衡。
    
2.  主键包含在每个二级索引中，过长的主键会浪费磁盘和内存的空间。
    

    让我们看看这个例子：

```
MySQL > CREATE TABLE my_table ( 
```

现在，让我们插入2个新记录：

```
MySQL > INSERT INTO my_table (name, beers) VALUES ("Luis",1), ("Miguel",5);
```

我们查看一下这个表的内容：

```
MySQL > SELECT * FROM my_table;
```

我们可以看到，两个新记录不是插入表格的末尾，而是插入到中间。InnoDB必须移动两个旧记录才能插入它们之前的两个新记录。在这个小表上（所有记录都在同一个页面上），不会造成任何问题，但设想一下，如果这个表是1TB大！

此外，如果我们使用VARCHCAR数据类型保存UUID，每个字段可能需要146字节（一些utf8字符最多可以占用4个字节+标记VARCHAR结束的2个字节）：

```
MySQL > EXPLAIN SELECT * FROM my_table WHERE 
```

02

—

## 解决方案

MySQL用户可以遵循一些最佳实践来避免这两个问题：

1.  使用数据类型BINARY(16)来存储UUID占用的空间比较小。
    
2.  使用函数UUID\_TO\_BIN(..., swap\_flag)对UUID进行转换，这里将swap\_flag设置为“1”可以让生成的UUID单向正增长，这里的时间戳的低部分和高部分（分别为十六进制数字的第一组和第三组）被交换，参见：https://dev.mysql.com/doc/refman/8.0/en/miscellaneous-functions.html#function\_uuid-to-bin
    

不断地重复执行下面的语句，看到得到结果不一定是正向增长的：

```
mysql> select hex(uuid_to_bin(uuid()));
```

如果加上swap\_flag设置为“1”，看到得到结果一直是正向增长的：

```
mysql> select hex(uuid_to_bin(uuid(),1));
```

让我们通过下面的例子来了解如何实现：

```
MySQL > CREATE TABLE my_table2 ( 
```

表里面存储的UUID是二进制的，我们使用BIN\_TO\_UUID对其进行解码：

```
MySQL > SELECT BIN_TO_UUID(uuid,1), name, beers FROM my_table2;
```

注意BIN\_TO\_UUID的swap\_flag设置为“1”并不是为了让结果看起来单向增长，而是为了把UUID\_TO\_BIN函数对时间的交换改成正确的。

现在我们可以验证，当我们添加新条目时，它们会添加到表格的末尾：

```
MySQL > INSERT INTO my_table2 (name, beers) VALUES ("Scott",1), ("Lenka",5); 
```

我们可以把swap\_flag设置为“1”进行解码UUID：

```
MySQL > SELECT BIN_TO_UUID(uuid,1), name, beers FROM my_table2;
```

当然，现在主键的大小更小，固定为16字节：

```
MySQL > EXPLAIN SELECT * FROM my_table2 
```

只有这16字节被添加到所有的二级索引中，这样二级索引对长度也变小了。

03

—

## UUID 的版本

目前有两个流行UUID版本，分别是v1和v4。UUID v1的说明参见：https://www.rfc-editor.org/rfc/rfc4122.html。

-   UUID v1：是一个通用的唯一标识符，使用时间戳和生成它的计算机的MAC地址生成。
    
-   UUID v4：是一个使用随机数生成的通用唯一标识符。
    

使用UUID v4，无法生成任何顺序输出，因此不推荐使用UUID v4做为InnoDB表的主键。

04

—

## 总结

总之，如果想在MySQL中使用UUID，请遵循以下建议：

-   使用UUID v1，而不是UUID v4。
    
-   使用BINARY(16)来存储UUID存储UUID。
    
-   使用函数UUID\_TO\_BIN(..., swap\_flag)对UUID进行转换，这里将swap\_flag设置为“1”。
    

欢迎加我的微信👇

**![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)**

**近期热文**

[**我学英语的绝招**](http://mp.weixin.qq.com/s?__biz=Mzk0MjMzMjE3OA==&mid=2247484043&idx=1&sn=0691c1bf019aece7c8eabba0edc54996&chksm=c2c582f6f5b20be07f0d6a8e8da6e7fef3ea7d4da5355cfa439f29e00b0e083e247d5ed27dab&scene=21#wechat_redirect)  

想快速学好英语吗？来看看一个IT人是如何达到英语专业八级的水平的。

[**学习oracle数据库只需要看一本书**](http://mp.weixin.qq.com/s?__biz=Mzk0MjMzMjE3OA==&mid=2247484062&idx=1&sn=393851d6a505f893a7831c001adc8187&chksm=c2c582e3f5b20bf5ab83b5e33e01ad14b257b052b21ec0126dd9857db8b0d49c04487c55ce5b&scene=21#wechat_redirect) 

实际上学好Oracle数据库只需要看一本书，让Oracle ACE来告诉你如何学好Oracle数据库？

**[赠书福利 《MySQL 8.0 运维与优化》重磅发布](https://mp.weixin.qq.com/s?__biz=MzIwMDUxOTExMA==&mid=2647864560&idx=1&sn=48073ac8a5adf445f216d5ada95b7333&scene=21#wechat_redirect)** 

**[刚刚上市10天就卖了一千](https://mp.weixin.qq.com/s?__biz=MzIwMDUxOTExMA==&mid=2647864560&idx=1&sn=48073ac8a5adf445f216d5ada95b7333&scene=21#wechat_redirect)**[本](https://mp.weixin.qq.com/s?__biz=MzIwMDUxOTExMA==&mid=2647864560&idx=1&sn=48073ac8a5adf445f216d5ada95b7333&scene=21#wechat_redirect)，京东和当当都卖断了货！

[**MySQL 8.0 性能优化艺术25讲**](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=Mzk0MjMzMjE3OA==&action=getalbum&album_id=2413528557064126465#wechat_redirect)

B站上最火的**MySQL性能优化课程。**

**[**Linux系统管理录像22讲**](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=Mzk0MjMzMjE3OA==&action=getalbum&album_id=2540141121310867457#wechat_redirect)  
零基础Linux课程**

[**中国第一个Oracle高可用认证大师？**](http://mp.weixin.qq.com/s?__biz=Mzg5ODY3NDMxOA==&mid=2247484363&idx=1&sn=ba26cd49e5fbc749448c5df3592d5fd4&chksm=c05fb82af728313c130f7305dd88377a689cae9144cd74028ad5e68969d06cea48a6578808d4&scene=21#wechat_redirect)  

点击“在看”可以阅读我翻译的其他文章👇