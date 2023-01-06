MySQL 的 root 密碼忘了，要怎麼強制重新設定呢？

先記錄官方做法：[MySQL 8.0 Reference Manual :: B.3.3.2 How to Reset the Root Password](https://dev.mysql.com/doc/refman/8.0/en/resetting-permissions.html)

1.  sudo su - mysql # 切換到 mysql user
2.  kill \`cat /mysql-data-directory/host\_name.pid\` # 找到 MySQL pid，kill 掉
3.  ALTER USER 'root'@'localhost' IDENTIFIED BY 'MyNewPass'; # 將此行存成檔案：/home/me/mysql-init
4.  mysqld --init-file=/home/me/mysql-init & # 啟動 MySQL 會執行將 'root'@'localhost' 的密碼換掉
5.  重新啟動 mysqld 即可用修改後的密碼進入

以前的土炮做法：

1.  stop or kill mysqld
2.  mysqld\_safe --skip-grant-tables &
3.  mysql -u root
4.  mysql> use mysql;
5.  mysql> UPDATE user SET Password=PASSWORD("password") WHERE User='root';
6.  mysql> flush privileges;
7.  mysql> quit

## 文章導覽