\> 全文建立在 MySQL 的存储引擎为 InnoDB 的基础上

先看一条 SQL 如何入库的：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9cc9d61588574e7e9a4b7f2305c7994d~tplv-k3u1fbpfcp-zoom-1.image)

这是一条很简单的更新 SQL，从 MySQL 服务端接收到 SQL 到落盘，先后经过了 MySQL Server 层和 InnoDB 存储引擎。

1.  Server 层就像一个产品经理，分析客户的需求，并给出实现需求的方案。
2.  InnoDB 就像一个基层程序员，实现产品经理给出的具体方案。

在 MySQL” 分析需求，实现方案 “的过程中，还夹杂着内存操作和磁盘操作，以及记录各种日志。

他们到底有什么用处？他们之间到底怎么配合的？MySQL 又为什么要分层呢？InnoDB 里面的那一块 Buffer Pool 又是什么？

我们慢慢分析。

## 分层结构

MySQL 为什么要分为 Server 层和存储引擎两层呢？

这个问题官方也没有给出明确的答案，但是也不难猜，简单来说就是为了 “解耦”。

Server 层和存储引擎各司其职，分工明确，用户可以根据不同的需求去使用合适的存储引擎，多好的设计，对不对？

后来的发展也验证了 “分层设计” 的优越性：MySQL 最初搭载的存储引擎是自研的只支持简单查询的 MyISAM 的前身 ISAM，后来与 Sleepycat 合作研发了 Berkeley DB 引擎，支持了事务。江山代有才人出，技术后浪推前浪，MySQL 在持续的升级着自己的存储引擎的过程中，遇到了横空出世的 InnoDB，InnoDB 的功能强大让 MySQL 倍感压力。

自己的存储引擎打不过 InnoDB 怎么办？

打不过就加入！

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5ef0b96136d74a33ae05fef44b871963~tplv-k3u1fbpfcp-zoom-1.image)

MySQL 选择了和 InnoDB 合作。正是因为 MySQL 存储引擎的插件化设计，两个公司合作的非常顺利，MySQL 也在合作后不久就发布了正式支持 nnoDB 的 4.0 版本以及经典的 4.1 版本。

MySQL 兼并天下模式也成为 MySQL 走向繁荣的一个重要因素。这能让 MySQL 长久地保持着极强竞争力。时至今日，MySQL 依然占据着极高数据库市场份额，仅次于王牌数据库 Oracle。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6ae6b6b16c02486ca4525cbf29f7c986~tplv-k3u1fbpfcp-zoom-1.image)

## Buffer Pool

在 InnoDB 里，有一块非常重要的结构 ——Buffer Pool。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5bf8ef5adeac41eca987f7dc0a357dd9~tplv-k3u1fbpfcp-zoom-1.image)

Buffer Pool 是个什么东西呢？

Buffer Pool 就是一块用于缓存 MySQL 磁盘数据的内存空间。

为什么要缓存 MySQL 磁盘数据呢？

我们通过一个例子说明，我们先假设没有 Buffer Pool，user 表里面只有一条记录，记录的 age = 1，假设需要执行三条 SQL：

1.  事务 A：update user set age = 2
2.  事务 B：update user set age = 3
3.  事务 C：update user set age = 4

如果没有 Buffer Pool，那执行就是这样的：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/259673a70db8439a8cc33c305b33236e~tplv-k3u1fbpfcp-zoom-1.image)

从图上可以看出，每次更新都需要从磁盘拿数据（1 次 IO），修改完了需要刷到磁盘（1 次 IO），也就是每次更新都需要 2 次磁盘 IO。三次更新需要 6 次磁盘 IO。

而有了 Buffer Pool，执行就成了这样：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fc64585ca8ca4e1b8621588d0e1181be~tplv-k3u1fbpfcp-zoom-1.image)

从图上可以看出，只需要在第一次执行的时候将数据从磁盘拿到 Buffer Pool（1 次 IO），第三次执行完将数据刷回磁盘（1 次 IO），整个过程只需要 2 次磁盘 IO，比没有 Buffer Pool 节省了 4 次磁盘 IO 的时间。

当然，Buffer Pool 真正的运转流程没有这么简单，具体实现细节和优化技巧还有很多，由于篇幅有限，本文不做详细描述。

我想表达的是：Buffer Pool 就是将磁盘 IO 转换成了内存操作，节省了时间，提高了效率。

Buffer Pool 是提高了效率没错，但是出现了一个问题，Buffer Pool 是基于内存的，而只要一断电，内存里面的数据就会全部丢失。

如果断电的时候 Buffer Pool 的数据还没来得及刷到磁盘，那么这些数据就丢失了吗？

还是上面的那个例子，如果三个事务执行完毕，在 age = 4 还没有刷到磁盘的时候，突然断电，数据就全部丢掉了：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4c401b92b6764c6490ec820e4439375d~tplv-k3u1fbpfcp-zoom-1.image)

试想一下，如果这些丢失的数据是核心的用户交易数据，那用户能接受吗？

答案是否定的。

那 InnoDB 是如何做到数据不会丢失的呢？

今天的第一个日志 ——redo log 登场了。

## 恢复 - redo log

顾名思义，redo 是重做的意思，redo log 就是重做日志的意思。

redo log 是如何保证数据不会丢失的呢？

就是在修改之后，先将修改后的值记录到磁盘上的 redo log 中，就算突然断电了，Buffer Pool 中的数据全部丢失了，来电的时候也可以根据 redo log 恢复 Buffer Pool，这样既利用到了 Buffer Pool 的内存高效性，也保证了数据不会丢失。

我们通过一个例子说明，我们先假设没有 Buffer Pool，user 表里面只有一条记录，记录的 age = 1，假设需要执行一条 SQL：

1.  事务 A：update user set age = 2

执行过程如下：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/954238d99ccd4f7391539af2e2554bd1~tplv-k3u1fbpfcp-zoom-1.image)

如上图，有了 redo log 之后，将 age 修改成 2 之后，马上将 age = 2 写到 redo log 里面，如果这个时候突然断电内存数据丢失，在来电的时候，可以将 redo log 里面的数据读出来恢复数据，用这样的方式保证了数据不会丢失。

你可能会问，redo log 文件也在磁盘上，数据文件也在磁盘上，都是磁盘操作，何必多此一举？为什么不直接将修改的数据写到数据文件里面去呢？

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/53e9869584054dcd8200ba4520f1a73d~tplv-k3u1fbpfcp-zoom-1.image)

傻瓜，因为 redo log 是磁盘顺序写，数据刷盘是磁盘随机写，磁盘的顺序写比随机写高效的多啊。

这种先预写日志后面再将数据刷盘的机制，有一个高大上的专业名词 ——WAL（Write-ahead logging），翻译成中文就是预写式日志。

虽然磁盘顺序写已经很高效了，但是和内存操作还是有一定的差距。

那么，有没有办法进一步优化一下呢？

答案是可以。那就是给 redo log 也加一个内存 buffer，也就是 redo log buffer，用这种套娃式的方法进一步提高效率。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b96241009bd94a1a994a4894a14cd394~tplv-k3u1fbpfcp-zoom-1.image)

redo log buffer 具体是怎么配合刷盘呢？

在这个问题之前之前，我们先来捋一下 MySQL 服务端和操作系统的关系：

MySQL 服务端是一个进程，它运行于操作系统之上。也就是说，操作系统挂了 MySQL 一定挂了，但是 MySQL 挂了操作系统不一定挂。

所以 MySQL 挂了有两种情况：

1.  MySQL 挂了，操作系统也挂了，也就是常说的服务器宕机了。这种情况 Buffer Pool 里面的数据会全部丢失，操作系统的 os cache 里面的数据也会丢失。
2.  MySQL 挂了，操作系统没有挂。这种情况 Buffer Pool 里面的数据会全部丢失，操作系统的 os cache 里面的数据不会丢失。

OK，了解了 MySQL 服务端和操作系统的关系之后，再来看 redo log 的落盘机制。redo log 的刷盘机制由参数 innodb\_flush\_log\_at\_trx\_commit 控制，这个参数有 3 个值可以设置：

1.  innodb\_flush\_log\_at\_trx\_commit = 1：实时写，实时刷
2.  innodb\_flush\_log\_at\_trx\_commit = 0：延迟写，延迟刷
3.  innodb\_flush\_log\_at\_trx\_commit = 2：实时写，延迟刷

写可以理解成写到操作系统的缓存（os cache），刷可以理解成把操作系统里面的缓存刷到磁盘。

这三种策略的区别，我们分开讨论：

### innodb\_flush\_log\_at\_trx\_commit = 1：实时写，实时刷

这种策略会在每次事务提交之前，每次都会将数据从 redo log 刷到磁盘中去，理论上只要磁盘不出问题，数据就不会丢失。

总结来说，这种策略效率最低，但是丢数据风险也最低。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/aabc806b761a46ee80af0c08ca337568~tplv-k3u1fbpfcp-zoom-1.image)

### innodb\_flush\_log\_at\_trx\_commit = 0：延迟写，延迟刷

这种策略在事务提交时，只会把数据写到 redo log buffer 中，然后让后台线程定时去将 redo log buffer 里面的数据刷到磁盘。

这种策略是最高效的，但是我们都知道，定时任务是有间隙的，但是如果事务提交后，后台线程没来得及将 redo log 刷到磁盘，这个时候不管是 MySQL 进程挂了还是操作系统挂了，这一部分数据都会丢失。

总结来说这种策略效率最高，丢数据的风险也最高。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/35c8e9f764814776aaf0f9dc157e9743~tplv-k3u1fbpfcp-zoom-1.image)

### innodb\_flush\_log\_at\_trx\_commit = 2：实时写，延迟刷

这种策略在事务提交之前会把 redo log 写到 os cache 中，但并不会实时地将 redo log 刷到磁盘，而是会每秒执行一次刷新磁盘操作。

这种情况下如果 MySQL 进程挂了，操作系统没挂的话，操作系统还是会将 os cache 刷到磁盘，数据不会丢失，如下图：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/06c5a2ab9af14cb484f1d14039804853~tplv-k3u1fbpfcp-zoom-1.image)

但如果 MySQL 所在的服务器挂掉了，也就是操作系统都挂了，那么 os cache 也会被清空，数据还是会丢失。如下图：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ddbb161731bd482eb1d2ea7daf482d67~tplv-k3u1fbpfcp-zoom-1.image)

所以，这种 redo log 刷盘策略是上面两种策略的折中策略，效率比较高，丢失数据的风险比较低，绝大多情况下都推荐这种策略。

总结一下，redo log 的作用是用于恢复数据，写 redo log 的过程是磁盘顺序写，有三种刷盘策略，有 innodb\_flush\_log\_at\_trx\_commit 参数控制，推荐设置成 2。

## 回滚 - undo log

我们都知道，InnoDB 是支持事务的，而事务是可以回滚的。

假如一个事务将 age=1 修改成了 age=2，在事务还没有提交的时候，后台线程已经将 age=2 刷入了磁盘。这个时候，不管是内存还是磁盘上，age 都变成了 2，如果事务要回滚，找不到修改之前的 age=1，无法回滚了。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/856fd259687246908ba25793866c9cd9~tplv-k3u1fbpfcp-zoom-1.image)

那怎么办呢？

很简单，把修改之前的 age=1 存起来，回滚的时候根据存起来的 age=1 回滚就行了。

MySQL 确实是这么干的！这个记录修改之前的数据的过程，叫做记录 undo log。undo 翻译成中文是撤销、回滚的意思，undo log 的主要作用也就是回滚数据。

如何回滚呢？看下面这个图：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0f9bf6207edc46878fee0e3f3dec2992~tplv-k3u1fbpfcp-zoom-1.image)

MySQL 在将 age = 1 修改成 age = 2 之前，先将 age = 1 存到 undo log 里面去，这样需要回滚的时候，可以将 undo log 里面的 age = 1 读出来回滚。

需要注意的是，undo log 默认存在全局表空间里面，你可以简单的理解成 undo log 也是记录在一个 MySQL 的表里面，插入一条 undo log 和插入一条普通数据是类似。也就是说，写 undo log 的过程中同样也是要写入 redo log 的。

## 归档 - binlog

undo log 记录的是修改之前的数据，提供回滚的能力。

redo log 记录的是修改之后的数据，提供了崩溃恢复的能力。

那 binlog 是干什么的呢？

binlog 记录的是修改之后的数据，用于归档。

和 redo log 日志类似，binlog 也有着自己的刷盘策略，通过 sync\_binlog 参数控制：

-   sync\_binlog = 0 ：每次提交事务前将 binlog 写入 os cache，由操作系统控制什么时候刷到磁盘
-   sync\_binlog =1 ：采用同步写磁盘的方式来写 binlog，不使用 os cache 来写 binlog
-   sync\_binlog = N ：当每进行 n 次事务提交之后，调用一次 fsync 将 os cache 中的 binlog 强制刷到磁盘

那么问题来了，binlog 和 redo log 都是记录的修改之后的值，这两者有什么区别呢？有 redo log 为什么还需要 binlog 呢？

首先看两者的一些区别：

-   binlog 是逻辑日志，记录的是对哪一个表的哪一行做了什么修改；redo log 是物理日志，记录的是对哪个数据页中的哪个记录做了什么修改，如果你还不了解数据页，你可以理解成对磁盘上的哪个数据做了修改。
-   binlog 是追加写；redo log 是循环写，日志文件有固定大小，会覆盖之前的数据。
-   binlog 是 Server 层的日志；redo log 是 InnoDB 的日志。如果不使用 InnoDB 引擎，是没有 redo log 的。

但说实话，我觉得这些区别并不是 redo log 不能取代 binlog 的原因，MySQL 官方完全可以调整 redo log 让他兼并 binlog 的能力，但他没有这么做，为什么呢？

我认为不用 redo log 取代 binlog 最大的原因是 “没必要”。

为什么这么说呢？

第一点，binlog 的生态已经建立起来。MySQL 高可用主要就是依赖 binlog 复制，还有很多公司的数据分析系统和数据处理系统，也都是依赖的 binlog。取代 binlog 去改变一个生态费力了不讨好。

第二点，binlog 并不是 MySQL 的瓶颈，花时间在没有瓶颈的地方没必要。

## 总结

总结一下：

1.  Buffer Pool 是 MySQL 进程管理的一块内存空间，有减少磁盘 IO 次数的作用。
2.  redo log 是 InnoDB 存储引擎的一种日志，主要作用是崩溃恢复，有三种刷盘策略，有 innodb\_flush\_log\_at\_trx\_commit 参数控制，推荐设置成 2。
3.  undo log 是 InnoDB 存储引擎的一种日志，主要作用是回滚。
4.  binlog 是 MySQL Server 层的一种日志，主要作用是归档。
5.  MySQL 挂了有两种情况：操作系统挂了 MySQL 进程跟着挂了；操作系统没挂，但是 MySQL 进程挂了。

最后，再用一张图总结一下全文的知识点：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/91f5947bf7a941aebeebe219bc1bb43e~tplv-k3u1fbpfcp-zoom-1.image)

## 写在最后

这篇文章写在一年之前，本来觉得是一篇水文没想要发，最近无聊修改了一下发了出来，希望能够用动图的形式帮助到 MySQL 基础不太好的朋友，大神忽略就好。

需要强调的一点是，由于作者水平有限，本文只是浅显的从无到有地阐述了 MySQL 几种日志的大致作用，过程中省略了很多细节，比如 Buffer Pool 的实现细节，比如 undo log 和 MVCC 的关系，比如 binlog buffer、change buffer 的存在，比如 redo log 的两阶段提交。

如果您有任何问题，我们可以探讨，如果您在文中发现错误，还望您指出，万分感谢！

好了，今天的文章就到这里了。

感谢你的阅读！我是 CoderW，我们下期再见。

\> 最后，欢迎关注我的公众号 “CoderW” 一起探讨进步～～～～

## 参考资料

-   《MySQL 实战 45 讲》
-   《从根儿上理解 MySQL》
-   《MySQL 技术内幕 —InnoDB 存储引擎》第 2 版