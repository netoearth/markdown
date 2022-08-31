Often even DBAs have misalignment on what is meant by “database size”. As data and databases grow with a rather high rate, it is important to understand the size of the database estate. Worldwide data is expected to [hit 175 zettabytes by 2025](https://www.aparavi.com/resources-blog/data-growth-statistics-blow-your-mind), representing a 61% CAGR. Also, 51% of the data will be in data centers and 49% will be in the public cloud. So, when moving to cloud, it is important to know how much storage is needed for the databases.

Ever wondered which is the biggest database in the world? Here is an interesting link: [10 Largest Databases in the World.](https://jobandwork.asia/10-largest-databases-world-really-know/) “**Really? Sure about that?** How do you actually know if it’s true? Who is the source of information, and can we trust it?” I was wondering exactly the same when checking [the list](https://www.comparebusinessproducts.com/fyi/10-largest-databases-in-the-world).

And also: how should all the data be stored? According to [Data: A Lenovo Solutions Perspective](https://lenovopress.lenovo.com/lp1367-lenovo-solutions-perspective-on-data), the answer is that within the core (i.e. data center or cloud) the classic relational database will still be the dominant approach for many years. Thus, one more reason to properly estimate and calculate the size of your databases.

![](https://juliandontcheff.files.wordpress.com/2022/08/dbs1.gif?w=756)

So, how how to calculate the size of say an Oracle database (check the links at the end for other database brands)? The most common way is to simply calculate the space which the database files physically consume on disk:

```
select sum(bytes)/1024/1024/1024 size_in_GB from dba_data_files;
```

But how about the other files? Like temp, control and redolog files? Are the undo files really part of the database? Do we always want to include the files from SYSTEM and SYSAUX too?

Also, not all this space in the files listed in dba\_data\_files is necessarily allocated. There could be sections of these files that are not used. Then, we can go with this method:

```
select sum(bytes)/1024/1024/1024 size_in_GB from dba_segments;
```

I often prefer using the query which ignores the system, temp and undo data:

```
select nvl(ceil(sum(bytes)/(1024*1024*1024)),0) size_in_GB
from dba_extents
where tablespace_name not in ('SYSTEM','TEMP','SYSAUX') and tablespace_name not like '%UNDO%';
```

Another way used is to only calculate the size of the real data from each schema in the database – the methods above include the indexes too. This requires recent analyze (not estimate but rather compute 100%) for all tables.

```
select owner, nvl(ceil(sum(num_rows*avg_row_len)/(1024*1024*1024)),0) size_in_GB
from dba_tables
where owner not in ('SYSTEM','OUTLN','SYS') group by owner;
```

However, checking the overall size including TEMP and REDO is perhaps the best approach. There is a MOS note, **How to Calculate the Size of the Database ([Doc ID 1360446.1](https://support.oracle.com/epmos/faces/DocumentDisplay?_afrLoop=425589044670906&id=1360446.1))** which is also worth reading. Here is the suggested method to calculate the total database size:

```
select a.data_size+b.temp_size+c.redo_size+d.cont_size "total_size"
from ( select sum(bytes) data_size
       from dba_data_files ) a,
     ( select nvl(sum(bytes),0) temp_size
       from dba_temp_files ) b,
     ( select sum(bytes) redo_size
       from sys.v_$logfile lf, sys.v_$log l
       where lf.group# = l.group#) c,
     ( select sum(block_size*file_size_blks) cont_size
       from v$controlfile ) d;
```

![](https://juliandontcheff.files.wordpress.com/2022/08/dbs2.gif?w=594)

If you have enabled block change tracking and want to be really pedantic with the database size, you should also add the BCT file.

Here are some additional links and a query which will help you find [indexes which are bigger than tables](https://community.oracle.com/tech/developers/discussion/2486121/indexes-which-bigger-than-their-tables).

[Database Size in Oracle](https://sqlserverguides.com/database-size-in-oracle/) by [Bijay Kumar Sahoo](https://sqlserverguides.com/author/fewlines4biju/)

[How to calculate current DB size](https://asktom.oracle.com/pls/apex/asktom.search?tag=how-to-calculate-current-db-size#:~:text=The%20size%20of%20the%20database,this%20space%20is%20necessarily%20allocated.) from asktom

[Estimate the Size of a Database in SQL Server](https://docs.microsoft.com/en-us/sql/relational-databases/databases/estimate-the-size-of-a-database?view=sql-server-ver16)

[How to calculate total size of the database](https://www.complexsql.com/how-to-calculate-total-size-of-the-database/)

[How to calculate the size of a MySQL database](http://www.codediesel.com/mysql/how-to-calculate-the-size-of-a-mysql-database/)

```
select owner "TABLE_OWNER", tablename "TABLE_NAME", tablesize "TABLE SIZE (GB)", indexname "INDEX_NAME", indexsize "INDEX SIZE (GB)", indexsize/tablesize "INDEX/TABLE" from
(
with
tabs as (select owner, segment_name tablename,sum(bytes/1024/1024/1024) tablesize from dba_segments where segment_type='TABLE' group by owner, segment_name),
inds as (select i.owner, i.index_name indexname, i.table_name tablename, sum(s.bytes/1024/1024/1024) indexsize from dba_indexes i join dba_segments s on (i.owner=s.owner and i.index_name=s.segment_name) group by i.owner, i.index_name, i.table_name)
select * from tabs natural join inds where indexsize > tablesize and indexsize>1
)
order by indexsize/tablesize desc;
```

Finally, there is a view in Oracle, called [dba\_hist\_tbspc\_space\_usage](https://docs.oracle.com/en/database/oracle/oracle-database/19/refrn/DBA_HIST_TBSPC_SPACE_USAGE.html#GUID-98A6FF30-1B6B-4A4F-ABD5-D229013E6BD1), which displays historical tablespace usage statistics. So, you can retrieve historical growth of the tablespaces and thus the database as a whole. Check this blog post: [How to retrieve growth history for Oracle tablespaces](https://blog.pythian.com/how-to-retrieve-growth-history-for-oracle-tablespaces/). Note that the history is available for as far back as AWR data is retained. It is considered good practice to keep track of the database size on say weekly basis. You will be always able to answer questions on capacity needs, trends, growth, size, etc.