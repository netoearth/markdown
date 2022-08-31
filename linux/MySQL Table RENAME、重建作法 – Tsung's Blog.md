[跳至主要內容](https://blog.longwin.com.tw/2022/08/mysql-table-rename-import-data-2022/#content)

MySQL 刪除資料後，空間並不會釋放出來，可以使用 OPTIMIZE TABLE 來釋放空間，OPTIMIZE 等同於 TABLE Copy & RENAME，所以會有大量 IO。

我是採用兩種方式來做，依照資料量大小來做選擇。(一樣會有大量IO，但是可以自己掌握，缺點是執行時會有短暫的時間可能漏資料)

不過資料量過大的，還是建議另外處理，這種作法是非常不得已的。

資料量小，可以瞬間新增完畢(大概10萬筆以下)

1.  CREATE TABLE table\_new LIKE table\_now;
2.  RENAME TABLE table\_now TO table\_old;
3.  RENAME TABLE table\_new TO table\_now;
4.  INSERT table\_now SELECT \* FROM table\_new;

資料量中大，資料先新增後，再來 RENAME，再來補中間漏的資料

1.  CREATE TABLE table\_new LIKE table\_now;
2.  INSERT table\_new SELECT \* FROM table\_now;
3.  RENAME TABLE table\_now TO table\_old;
4.  RENAME TABLE table\_new TO table\_now;
5.  INSERT table\_new SELECT \* FROM table\_now WHERE id > xxx;
6.  可能需要用：INSERT IGNORE table\_new SELECT \* FROM talbe\_now WHERE id > xxx;

上述 INSERT table\_new SELECT \* FROM table\_now; 可能會遇到下述錯誤訊息：

-   ERROR 1062 (23000): Duplicate entry 'AA' for key 'PRIMARY'

預設遇到 Duplicate Key (ERROR 1062 (23000): Duplicate entry 'AA' for key 'PRIMARY') 就會停止 INSERT，遇到 Duplicate Key 跳過繼續寫入其它語法，可以使用下述(INSERT IGNORE)：

-   INSERT **IGNORE** table\_new SELECT \* FROM talbe\_now;

## 文章導覽