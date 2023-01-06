闪回查询可以让使用者查询到以往某个时间点上的表中的内容，这对于某些场合是非常有用处的，甚至是救命的功能。比如如果不小心误删除了表中的数据，就可以通过闪回查询查到删除时间点之前的数据，这样可以直接将这些数据再恢复回来。

比如这样一个让人惋惜的场景（delete的SQL请千万不要在生产环境中运行）。

现在你有这么一张表，记录了账户姓名和账户金额。

```
MogDB=# select * from accounts; name | amount------+-------- A    |    100 B    |    100 C    |    100 D    |     99(4 rows)
```

你本来想执行一条SQL，去删除金额等于99元的账户信息。正常执行的话，应该删除1条记录。为了演示效果，先用select来代替delete。

```
MogDB=#  select * from accounts where amount=99; name | amount------+-------- D    |     99(1 row)
```

但是在键盘上减号“-”和等号“=”紧挨着，你的手指太胖了，误按到了减号，所以现在你给数据库发出去的命令是这样的。

```
delete from accounts where amount-99;
```

为了演示，还是用select来代替delete。

```
#  select * from accounts where amount-99; name | amount------+-------- A    |    100 B    |    100 C    |    100(3 rows)
```

恐怖的事情发生了，除了金额真的等于99的那条数据没有返回，其它所有的数据都返回了。也就意味着，如果这是上面的那条delete命令，你删除了表中所有金额不等于99的账户。

好消息是，在MogDB 3.0中，已经修改了对减号这种危险语法的校验，现在执行同样的SQL，将会报错。

```
MogDB=# delete from accounts where amount-99;ERROR:  argument of WHERE must be type boolean, not type integerLINE 1: delete from accounts where amount-99;                                   ^
```

但是在社区版的openGauss3.0或者MySQL或者MariaDB中，这样的危险语法仍然是可以正常执行的。

```
gsql ((openGauss 3.0.0 build 02c14696) compiled at 2022-04-01 18:28:23 commit 0 last mr  release)Non-SSL connection (SSL connection is recommended when requiring high-security)Type "help" for help.omm=# select version();                                          version-------------------------------------------------------------------------------------------- (openGauss 3.0.0 build 02c14696) compiled at 2022-04-01 18:28:23 commit 0 last mr  release(1 row)omm=# create table accounts (name varchar2, amount int);CREATE TABLEomm=# insert into accounts values ('A',100),('B',100),('C',100),('D',99);INSERT 0 4omm=# delete from accounts where amount-99;DELETE 3omm=# select * from accounts; name | amount------+-------- D    |     99(1 row)
```

不管是什么误操作，如果真的发生了，那么在MogDB 3.0中还有闪回查询可以用。

闪回查询及相关实现在MogDB 3.0中发生了一些变化。

1.  只对存储引擎为ustore的表起作用，默认的astore已经不再支持闪回查询。
2.  需要先手工设置undo\_retention\_time。该参数表示回滚段中旧版本数据的保留时间，等同于允许闪回查询的时间跨度。该参数默认是0，任何闪回查询都会遇到`restore point not found`报错。

创建一张ustore的accounts表。

```
MogDB=# create table accounts (name varchar2, amount int) with (storage_type=ustore);CREATE TABLE
```

修改undo\_retention\_time参数，该参数单位是秒，86400表示是24小时。该参数允许最大值2147483647，相当于大约68年。当然你不会设置这么大，因为这会导致回滚段中保留过多旧数据。

```
MogDB=# alter system set undo_retention_time=86400;ALTER SYSTEM SET
```

插入一些测试数据，就用前面提到的账户和金额。

```
MogDB=#  insert into accounts values ('A',100),('B',100),('C',100),('D',99);INSERT 0 4
```

现在，因为某种误操作，你删除了所有金额不等于99的账户记录。

```
MogDB=# delete from accounts where amount<>99;DELETE 3MogDB=#  select * from accounts; name | amount------+-------- D    |     99(1 row)
```

当你发现这个失误的时候，可能是在1分钟以后，也可能是在1小时以后，这没关系，但是不要超过24小时（因为undo\_retention\_time设置了24小时），否则就找不回来了。

检查当前的时间戳，然后预估你执行误操作时候的时间戳，比如用当前时间戳减去5分钟，为了简便演示，我在发出删除命令之前，检查了系统的时间戳。

```
MogDB=# select sysdate;       sysdate--------------------- 2022-09-02 13:19:24(1 row)
```

发起闪回查询。

```
MogDB=# select * from accounts timecapsule TIMESTAMP to_timestamp('2022-09-02 13:19:24','YYYY-MM-DD HH24:MI:SS'); name | amount------+-------- A    |    100 B    |    100 C    |    100 D    |     99(4 rows)
```

真正的恢复步骤，可以是按照删除命令的条件，利用闪回查询将所有被误删除的记录创建为一张临时表，然后再从临时表中将所有记录插入回原表中。

```
MogDB=# create table tmp_accounts as select * from accounts timecapsule TIMESTAMP to_timestamp('2022-09-02 13:19:24','YYYY-MM-DD HH24:MI:SS') where amount<>99;INSERT 0 3MogDB=# select * from tmp_accounts; name | amount------+-------- A    |    100 B    |    100 C    |    100(3 rows)MogDB=# insert into accounts select * from tmp_accounts;INSERT 0 3MogDB=# select * from accounts; name | amount------+-------- D    |     99 A    |    100 B    |    100 C    |    100(4 rows)
```