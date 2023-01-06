View 可以縮短 SQL 撰寫的長度，只是做個類似轉換的動作，後面該做的還是跑不掉的。

不過若有新舊版程式，例如要將這個 Table Name 加上「年月」，但是原始 Table 暫時還要可以讀取，就可以靠 View 來達成。

Create View 文件與語法

-   MySQL 官方文件：[MySQL 8.0 Reference Manual :: 13.1.23 CREATE VIEW Statement](https://dev.mysql.com/doc/refman/8.0/en/create-view.html)
    -   CREATE VIEW \[db\_name.\]view\_name \[(column\_list)\] AS  
        select-statement;
    -   範例：  
        CREATE VIEW transactions AS  
        SELECT id\_number, name, transaction\_date FROM customers;

想將 Table Name 加上「年月」，會需要 RENAME Table，然後再加上一個 View 來取代原始 Table，讓其它程式還可以讀取，要怎麼做呢？

步驟如下：

1.  RENAME Table (加上年月)
    -   RENAME TABLE table\_name TO table\_name\_202201;
2.  建立 VIEW (取代原始 Table name)
    -   CREATE VIEW table\_name\_202201 AS SELECT \* FROM table\_name;
    -   若一次想要把兩個都撈進來 (不過這個速度很慢)
    -   CREATE VIEW table\_name AS SELECT \* FROM table\_name\_202201 UNION ALL SELECT \* FROM table\_name\_202202;
3.  程式都已經改完後
4.  DROP View
    -   DROP VIEW table\_name;

## 文章導覽