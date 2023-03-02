## [](https://mritd.com/2021/05/29/mysql-schema-diff/#%E4%B8%80%E3%80%81%E5%AE%89%E8%A3%85-mysql-schema-diff "一、安装 mysql-schema-diff")一、安装 mysql-schema-diff[](https://mritd.com/2021/05/29/mysql-schema-diff/#%E4%B8%80%E3%80%81%E5%AE%89%E8%A3%85-mysql-schema-diff)

Ubuntu 20.04 系统使用如下命令安装:

```
apt install libmysql-diff-perl -y
```

安装完成后使用 `--help` 应该能看到相关提示

```
➜ ~ mysql-schema-diff --help
Usage: mysql-schema-diff [ options ] <database1> <database2>

Options:
  -?,  --help                show this help
  -A,  --apply               interactively patch database1 to match database2
  -B,  --batch-apply         non-interactively patch database1 to match database2
  -d,  --debug[=N]           enable debugging [level N, default 1]
  -l,  --list-tables         output the list off all used tables
  -o,  --only-both           only output changes for tables in both databases
  -k,  --keep-old-tables     don't output DROP TABLE commands
  -c,  --keep-old-columns    don't output DROP COLUMN commands
  -n,  --no-old-defs         suppress comments describing old definitions
  -t,  --table-re=REGEXP     restrict comparisons to tables matching REGEXP
  -i,  --tolerant            ignore DEFAULT, AUTO_INCREMENT, COLLATE, and formatting changes
  -S,  --single-transaction  perform DB dump in transaction

  -h,  --host=...            connect to host
  -P,  --port=...            use this port for connection
  -u,  --user=...            user for login if not current user
  -p,  --password[=...]      password to use when connecting to server
  -s,  --socket=...          socket to use when connecting to server

for <databaseN> only, where N == 1 or 2,
       --hostN=...           connect to host
       --portN=...           use this port for connection
       --userN=...           user for login if not current user
       --passwordN[=...]     password to use when connecting to server
       --socketN=...         socket to use when connecting to server

Databases can be either files or database names.
If there is an ambiguity, the file will be preferred;
to prevent this prefix the database argument with `db:'.
```

## [](https://mritd.com/2021/05/29/mysql-schema-diff/#%E4%BA%8C%E3%80%81%E7%94%9F%E6%88%90%E5%B7%AE%E5%BC%82-SQL "二、生成差异 SQL")二、生成差异 SQL[](https://mritd.com/2021/05/29/mysql-schema-diff/#%E4%BA%8C%E3%80%81%E7%94%9F%E6%88%90%E5%B7%AE%E5%BC%82-SQL)

安装完成后可直接使用该工具生成**差异 SQL 文件**，`mysql-schema-diff` 工具使用如下:

```
Usage: mysql-schema-diff [ options ] <database1> <database2>

Options:
  -?,  --help                show this help
  -A,  --apply               interactively patch database1 to match database2
  -B,  --batch-apply         non-interactively patch database1 to match database2
  -d,  --debug[=N]           enable debugging [level N, default 1]
  -l,  --list-tables         output the list off all used tables
  -o,  --only-both           only output changes for tables in both databases
  -k,  --keep-old-tables     don't output DROP TABLE commands
  -c,  --keep-old-columns    don't output DROP COLUMN commands
  -n,  --no-old-defs         suppress comments describing old definitions
  -t,  --table-re=REGEXP     restrict comparisons to tables matching REGEXP
  -i,  --tolerant            ignore DEFAULT, AUTO_INCREMENT, COLLATE, and formatting changes
  -S,  --single-transaction  perform DB dump in transaction

  -h,  --host=...            connect to host
  -P,  --port=...            use this port for connection
  -u,  --user=...            user for login if not current user
  -p,  --password[=...]      password to use when connecting to server
  -s,  --socket=...          socket to use when connecting to server

for <databaseN> only, where N == 1 or 2,
       --hostN=...           connect to host
       --portN=...           use this port for connection
       --userN=...           user for login if not current user
       --passwordN[=...]     password to use when connecting to server
       --socketN=...         socket to use when connecting to server
```

**通俗的说，通过 `--hostN` 等参数指定两个数据库地址，例如 `--password1` 指定第一个数据库密码，`--password2` 指定第二个数据库密码；然后最后仅跟 `数据库1[.表名] 数据库2[.表名]`，表名如果不写则默认对比两个数据库。**以下为样例命令:

```
mysql-schema-diff \
    --host1 127.0.0.1 --port1 3300 \
    --user1 bleem --password1=Bleem77965badf \
    --host2 127.0.0.1 --port2 3306 \
    --user2 bleem --password2=asnfskdf667asd8 \
    testdb testdb
```

注意，首次运行后请根据生成的 SQL 判断**对比是否正确**，比如说**想把比较新的测试库更改同步到生产库**，那么 SQL 里全是**DROP 字样的删除动作，这说明 `--hostN` 等参数指定反了(变成了生产库同步测试库)，此时只需要将 `--hostN` 参数调换一下即可(1改成2，2改成1)，这样生成的 SQL 就会变为 ADD 字样的添加动作。**

## [](https://mritd.com/2021/05/29/mysql-schema-diff/#%E4%B8%89%E3%80%81%E7%BD%91%E7%BB%9C%E9%97%AE%E9%A2%98 "三、网络问题")三、网络问题[](https://mritd.com/2021/05/29/mysql-schema-diff/#%E4%B8%89%E3%80%81%E7%BD%91%E7%BB%9C%E9%97%AE%E9%A2%98)

mysql-schema-diff 工具需要在运行时能同时连接两个数据库，常规情况下可以通过 SSH 打洞来临时解决访问问题；**如果实在无法打通网络环境，mysql-schema-diff 还支持文件对比**，以下为一些文件对比的示例:

```
# compare table definitions in two files
mysql-schema-diff db1.mysql db2.mysql

# compare table definitions in a file 'db1.mysql' with a database 'db2'
mysql-schema-diff db1.mysql db2

# interactively upgrade schema of database 'db1' to be like the
# schema described in the file 'db2.mysql'
mysql-schema-diff -A db1 db2.mysql

# compare table definitions in two databases on a remote machine
mysql-schema-diff --host=remote.host.com --user=myaccount db1 db2

# compare table definitions in a local database 'foo' with a
# database 'bar' on a remote machine, when a file foo already
# exists in the current directory
mysql-schema-diff --host2=remote.host.com --password=secret db:foo bar
```