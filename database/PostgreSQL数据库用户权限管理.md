![图片](https://mmbiz.qpic.cn/mmbiz_jpg/YibW6hdF6Wb9DiaGYdesRPfR3tzcNycV1swDeph9RrYJrxWk0wzKTkqZcrrZ8x0mVZ3ibc6VC4Ce1TNstQ5VKxFgQ/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1)

**启动数据库**

```
[postgres@cjc-db-01 ~]$ pg_ctl -D /pg/data -l /pg/log/pg.log start
```

**登录数据库**

```
[postgres@cjc-db-01 ~]$ psql
```

**查看pg\_roles表字段**

```
postgres=# \d pg_roles
```

或

```
postgres=#
```

**查看角色信息**

```
postgres=# select rolname,rolsuper,rolcreatedb from pg_roles;
```

**在PostgreSQL数据库中，角色和用户没有区别，一个角色就是一用户，这块和Oracle数据库有一定差异。**

**相关描述如下：**

```
CREATE USER is now an alias for CREATE ROLE. 
```

**查看create user帮助信息**

```
postgres=# \h create user;
```

查看create role帮助信息，和create user完全一样。

```
postgres=# \h create role;
```

**参数说明：**

```
1.SUPERUSER |NOSUPERUSER
```

**创建用户**

```
create user chen LOGIN;
```

**创建角色**

```
create role cjc SUPERUSER PASSWORD '1';
```

**查看用户已经创建**

```
postgres=# select rolname,rolsuper,rolcreatedb from pg_roles;
```

**指定用户登录**

```
[postgres@cjc-db-01 ~]$ psql -h 172.16.6.137 -p 5678 -U chen -W postgres
```

```
postgres=> select current_user;
```

注意：创建用户方式创建出来的用户默认有 LOGIN 权限，而创建角色创建出来的用户没有 LOGIN 权限。

```
[postgres@cjc-db-01 ~]$ psql -h 172.16.6.137 -p 5678 -U cjc -W postgres
```

**授予登录权限**

```
postgres=# grant connect on database postgres to cjc;
```

**授权语句不对，还是不能登录**

```
[postgres@cjc-db-01 ~]$ psql -h 172.16.6.137 -p 5678 -U cjc -W postgres
```

**修改角色**

```
postgres=# alter role cjc login superuser;
```

**可以登录了**

```
[postgres@cjc-db-01 ~]$ psql -h 172.16.6.137 -p 5678 -U cjc -W postgres
```

```
postgres=# select current_user;
```

**为用户指定表空间**

**查看表空间**

```
postgres=# \db
```

**创建表空间**

```
postgres=# \h create tablespace
```

**注意：**

```
1.表空间的名称不能以 'pg_' 开头，它们是系统表空间的保留名称；
```

**创建空目录**

```
[postgres@cjc-db-01 data]$ pwd
```

**创建表空间，有WARNING，新增的表空间不能在数据路径里面**

```
postgres=# CREATE TABLESPACE cjctbs OWNER cjc LOCATION '/pg/data/cjctbs';
```

**自动生成的文件**

```
[postgres@cjc-db-01 PG_10_201707211]$ pwd
```

**删除表空间**

```
postgres=# drop tablespace cjctbs;
```

**创建表空间,重新指定路径，没有WARNING**

```
[postgres@cjc-db-01 pg]$ mkdir /pg/tbs/cjctbs -p
```

**自动生成目录**

```
[postgres@cjc-db-01 PG_10_201707211]$ pwd
```

**创建数据库**

**指定用户和表空间**

```
postgres=# create database cjcdb owner cjc tablespace cjctbs;
```

**查看新生成的文件**

```
[postgres@cjc-db-01 16396]$ pwd
```

**查询数据库**

```
postgres=# \l+
```

**查询数据库物理存储位置**

```
cjcdb=# select oid,datname from pg_database where datname = 'cjcdb';
```

**根据oid进行查找**

```
[postgres@cjc-db-01 16396]$ pwd
```

**查看指定数据库的大小**

```
postgres=# select pg_database_size('cjcdb');
```

**查看指定数据库的大小**

```
postgres=# select pg_size_pretty(pg_database_size('cjcdb'));
```

**查看数据库的大小**

```
postgres=# select pg_database.datname, pg_database_size(pg_database.datname) AS size from pg_database; 
```

**查询数据库默认表空间**

```
cjcdb=# select datname,dattablespace from pg_database where datname='cjcdb'; 
```

**登录数据库**

```
[postgres@cjc-db-01 ~]$ psql -h 172.16.6.137 -p 5678 -U cjc -W cjcdb
```

**建表**

```
cjcdb=# create table t1(id int,name char(10));
```

**查询表物理存储位置**

```
cjcdb=# select oid,relfilenode from pg_class where relname = 't1';
```

**根据relfilenode进行查找**

```
[postgres@cjc-db-01 16396]$ pwd
```

**查询表所在表空间**

**新增表**

```
create table t2(id int,name char(10));
```

**没表空间信息？**

```
cjcdb=# \d+ t3
```

**修改表所在表空间**

```
alter table t1 set tablespace cjctbs;
```

**查询表结构**

```
cjcdb=# \d t1
```

**插入数据**

```
cjcdb=# insert into t1 values(1,'a'),(2,'aaa');
```

**查看当前数据库下所有表**

```
postgres=# \c cjcdb
```

**查看表大小**

```
cjcdb=# select pg_relation_size('t1'); 
```

**查看表大小**

```
cjcdb=# select pg_size_pretty(pg_relation_size('t1')); 
```

**创建索引**

```
cjcdb=# create index i_t1_id on t1(id);
```

**查看表的总大小，包括索引大小**

```
cjcdb=# select pg_size_pretty(pg_total_relation_size('t1'));
```

**查看当前数据库下所有索引**

```
cjcdb=# \di
```

**查看单个索引大小**

```
cjcdb=# select pg_size_pretty(pg_relation_size('i_t1_id'));
```

**查询表空间大小**

```
cjcdb=# select pg_size_pretty(pg_tablespace_size('cjctbs'));
```

**当前chen只有登录权限**

```
[postgres@cjc-db-01 ~]$ psql -h 172.16.6.137 -p 5678 -U chen -W cjcdb
```

**没有查询权限**

```
cjcdb=> select * from t1;
```

**查看授权语句**

```
postgres=# \h grant;
```

**表级别授权**

```
[postgres@cjc-db-01 ~]$ psql -h 172.16.6.137 -p 5678 -U cjc -W cjcdb
```

可以查询

```
[postgres@cjc-db-01 ~]$ psql -h 172.16.6.137 -p 5678 -U chen -W cjcdb
```

不能执行其他操作

```
cjcdb=> insert into t1 values(3,'xxx');
```

**回收权限**

```
[postgres@cjc-db-01 ~]$ psql -h 172.16.6.137 -p 5678 -U cjc -W cjcdb
```

**库级别授权**

```
cjcdb=# GRANT ALL ON DATABASE cjcdb to chen;
```

**查询权限  
**

**查看用户表权限**

```
cjcdb=# select * from information_schema.table_privileges where grantee='cjc';
```

**查看usage权限表**

```
select * from information_schema.usage_privileges where grantee='cjc';
```

**查看存储过程函数相关权限表**

```
select * from information_schema.routine_privileges where grantee='cjc';
```

**###chenjuchao 20230118 10:20###**

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)