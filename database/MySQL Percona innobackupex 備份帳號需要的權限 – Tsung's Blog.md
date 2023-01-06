[跳至主要內容](https://blog.longwin.com.tw/2022/09/mysql-percona-innobackupex-privilege-permission-2022/#content)

MySQL 要備份資料庫，可以使用 Percona 的 Tookit 的 innobackupex (xtrabackup) 來備份。

要為 innobackupex 建立一個備份專用的使用者，需要哪些權限呢？

Percona 使用 innobackupex、xtrabackup 來做 MySQL 資料的備份，需要開哪些權限才能備份？

官方文件可見此篇：[Connection and Privileges Needed - Percona XtraBackup](https://docs.percona.com/percona-xtrabackup/2.4/using_xtrabackup/privileges.html)

innobackupex 裡面包著 xtrabackup 在執行，需要的權限是一樣的，需要的權限：

-   RELOAD
-   LOCK TABLES
    -   RELOAD and LOCK TABLES (unless the --no-lock option is specified) in order to FLUSH TABLES WITH READ LOCK prior to start copying the files and
-   PROCESS
    -   PROCESS in order to run SHOW ENGINE INNODB STATUS
-   REPLICATION CLIENT
    -   REPLICATION CLIENT in order to obtain the binary log position,
-   下述可選用：
    -   CREATE TABLESPACE in order to import tables (see Restoring Individual Tables)
    -   SUPER in order to start/stop the slave threads in a replication environment.

### Percona innobackupex 授權(Grant)語法

1.  mysql> CREATE USER 'bkpuser'@'localhost' IDENTIFIED BY 's3cret';
2.  mysql> REVOKE ALL PRIVILEGES, GRANT OPTION FROM 'bkpuser'; # 先將所有權限移掉
3.  mysql> GRANT RELOAD, LOCK TABLES, PROCESS, REPLICATION CLIENT ON _._ TO 'bkpuser'@'localhost';
4.  mysql> FLUSH PRIVILEGES;

### Percona innobackupex、xtrabackup 操作方式

-   $ innobackupex --user=DBUSER --password=SECRET /path/to/backup/dir/
-   $ innobackupex --user=LUKE --password=US3TH3F0RC3 --stream=tar ./ | bzip2 -
-   $ xtrabackup --user=DVADER --password=14MY0URF4TH3R --backup --target-dir=/data/bkps/

#### 相關網頁

-   [How innobackupex Works](https://www.percona.com/doc/percona-xtrabackup/1.6/innobackupex/how_innobackupex_works.html#how-ibk-works)

## 文章導覽