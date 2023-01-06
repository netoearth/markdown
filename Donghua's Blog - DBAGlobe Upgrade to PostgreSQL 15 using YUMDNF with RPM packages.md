### 1\. Install PostgreSQL 15

```
[root@ol8 ~]# dnf install postgresql15-libs postgresql15-server postgresql15-contrib postgresql15
```

### 2\. Initialize the cluster (Use compatible initdb flags that match the old cluster)

```
[postgres@ol8 ~]$ /usr/pgsql-15/bin/initdb -D /var/lib/pgsql/15/data

The files belonging to this database system will be owned by user "postgres".
This user must also own the server process.

The database cluster will be initialized with locale "en_US.UTF-8".
The default database encoding has accordingly been set to "UTF8".
The default text search configuration will be set to "english".

Data page checksums are disabled.

fixing permissions on existing directory /var/lib/pgsql/15/data ... ok
creating subdirectories ... ok
selecting dynamic shared memory implementation ... posix
selecting default max_connections ... 100
selecting default shared_buffers ... 128MB
selecting default time zone ... Asia/Singapore
creating configuration files ... ok
running bootstrap script ... ok
performing post-bootstrap initialization ... ok
syncing data to disk ... ok

initdb: warning: enabling "trust" authentication for local connections
initdb: hint: You can change this by editing pg_hba.conf or using the option -A, or --auth-local and --auth-host, the next time you run initdb.

Success. You can now start the database server using:

    /usr/pgsql-15/bin/pg_ctl -D /var/lib/pgsql/15/data -l logfile start
```

### 3\. Stop existing PostgreSQL 14 service

```
# Find out the service name
[root@ol8 ~]# systemctl |grep postgres
  postgresql-14.service  loaded active running   PostgreSQL 14 database server
```

```
# Stop the service
[root@ol8 ~]# systemctl stop postgresql-14.service
```

```
# Disable the service from future auto-start during reboot
[root@ol8 ~]# systemctl disable postgresql-14.service
Removed /etc/systemd/system/multi-user.target.wants/postgresql-14.service.
```

### 4\. Perform database upgrade

```
[postgres@ol8 ~]$ /usr/pgsql-15/bin/pg_upgrade -b /usr/pgsql-14/bin/ -d /var/lib/pgsql/14/data/ -D /var/lib/pgsql/15/data/

Performing Consistency Checks
-----------------------------
Checking cluster versions                                   ok
Checking database user is the install user                  ok
Checking database connection settings                       ok
Checking for prepared transactions                          ok
Checking for system-defined composite types in user tables  ok
Checking for reg* data types in user tables                 ok
Checking for contrib/isn with bigint-passing mismatch       ok
Creating dump of global objects                             ok
Creating dump of database schemas
                                                            ok
Checking for presence of required libraries                 ok
Checking database user is the install user                  ok
Checking for prepared transactions                          ok
Checking for new cluster tablespace directories             ok

If pg_upgrade fails after this point, you must re-initdb the
new cluster before continuing.

Performing Upgrade
------------------
Analyzing all rows in the new cluster                       ok
Freezing all rows in the new cluster                        ok
Deleting files from new pg_xact                             ok
Copying old pg_xact to new server                           ok
Setting oldest XID for new cluster                          ok
Setting next transaction ID and epoch for new cluster       ok
Deleting files from new pg_multixact/offsets                ok
Copying old pg_multixact/offsets to new server              ok
Deleting files from new pg_multixact/members                ok
Copying old pg_multixact/members to new server              ok
Setting next multixact ID and offset for new cluster        ok
Resetting WAL archives                                      ok
Setting frozenxid and minmxid counters in new cluster       ok
Restoring global objects in the new cluster                 ok
Restoring database schemas in the new cluster
                                                            ok
Copying user relation files
                                                            ok
Setting next OID for new cluster                            ok
Sync data directory to disk                                 ok
Creating script to delete old cluster                       ok
Checking for extension updates                              notice

Your installation contains extensions that should be updated
with the ALTER EXTENSION command.  The file
    update_extensions.sql
when executed by psql by the database superuser will update
these extensions.


Upgrade Complete
----------------
Optimizer statistics are not transferred by pg_upgrade.
Once you start the new server, consider running:
    /usr/pgsql-15/bin/vacuumdb --all --analyze-in-stages

Running this script will delete the old cluster's data files:
    ./delete_old_cluster.sh
```

`Content of update_extensions.sql`

```
--- update_extensions.sql
\connect test
ALTER EXTENSION "pageinspect" UPDATE;
```

`Content of delete_old_cluster.sh`

```
#!/bin/sh
rm -rf '/var/lib/pgsql/14/data'
```

### 5\. Enable and start PostgreSQL 15 service

```
[root@ol8 ~]# systemctl list-unit-files |grep postgres
postgresql-14.service                            disabled        disabled
postgresql-15.service                            disabled        disabled
```

```
[root@ol8 ~]# systemctl enable postgresql-15.service
Created symlink /etc/systemd/system/multi-user.target.wants/postgresql-15.service → /usr/lib/systemd/system/postgresql-15.service.
```

```
[root@ol8 ~]# systemctl start postgresql-15.service
```

### 6\. Post upgrade actions

```
[postgres@ol8 ~]$ /usr/pgsql-15/bin/vacuumdb --all --analyze-in-stages

vacuumdb: processing database "postgres": Generating minimal optimizer statistics (1 target)
vacuumdb: processing database "sample": Generating minimal optimizer statistics (1 target)
vacuumdb: processing database "template1": Generating minimal optimizer statistics (1 target)
vacuumdb: processing database "test": Generating minimal optimizer statistics (1 target)
vacuumdb: processing database "postgres": Generating medium optimizer statistics (10 targets)
vacuumdb: processing database "sample": Generating medium optimizer statistics (10 targets)
vacuumdb: processing database "template1": Generating medium optimizer statistics (10 targets)
vacuumdb: processing database "test": Generating medium optimizer statistics (10 targets)
vacuumdb: processing database "postgres": Generating default (full) optimizer statistics
vacuumdb: processing database "sample": Generating default (full) optimizer statistics
vacuumdb: processing database "template1": Generating default (full) optimizer statistics
vacuumdb: processing database "test": Generating default (full) optimizer statistics
```

```
[postgres@ol8 ~]$ psql -a
psql (15.0)
Type "help" for help.

postgres=# \i update_extensions.sql
\connect test
You are now connected to database "test" as user "postgres".
ALTER EXTENSION "pageinspect" UPDATE;
ALTER EXTENSION
```

```
[postgres@ol8 ~]$ ./delete_old_cluster.sh
```

```
[root@ol8 ~]# dnf remove postgresql14-libs postgresql14 postgresql14-server postgresql14-contrib
```