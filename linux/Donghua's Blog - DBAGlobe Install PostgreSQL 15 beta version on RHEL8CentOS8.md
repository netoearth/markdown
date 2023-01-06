### Step 1: Modify existing `/etc/yum.repos.d/pgdg-redhat-all.repo` to include beta release for v15. Replace the `enable=0` to `enabled=1`.

```
[pgdg15-updates-testing]
name=PostgreSQL 15 for RHEL / Rocky $releasever - $basearch - Updates testing
baseurl=https://download.postgresql.org/pub/repos/yum/testing/15/redhat/rhel-$releasever-$basearch
enabled=1
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-PGDG
repo_gpgcheck = 1
```

If you haven't installed the PostgreSQL Yum Repo yet, execute following command to install it for your OS version. Choose a different link if your are not using RHEL 8/CentOS 8. ([https://www.postgresql.org/download/linux/redhat/](https://www.postgresql.org/download/linux/redhat/))

```
yum install https://download.postgresql.org/pub/repos/yum/reporpms/EL-8-x86_64/pgdg-redhat-repo-latest.noarch.rpm
```

### Step 2: Install PostgreSQL Server and Client

1.  Get list of enabled YUM repositories.

```
yum repolist
```

```
[root@9cced9bedc1c ~]# yum repolist
repo id                                       repo name
appstream                                     CentOS Linux 8 - AppStream
baseos                                        CentOS Linux 8 - BaseOS
extras                                        CentOS Linux 8 - Extras
pgdg-common                                   PostgreSQL common RPMs for RHEL / Rocky 8 - x86_64
pgdg14                                        PostgreSQL 14 for RHEL / Rocky 8 - x86_64
pgdg15-updates-testing                        PostgreSQL 15 for RHEL / Rocky 8 - x86_64 - Updates testing
```

2.  List available packages in `pgdg15-updates-testing` Repo.

```
# yum --disablerepo="*" --enablerepo="pgdg15-updates-testing" list available
```

```
[root@9cced9bedc1c ~]# yum --disablerepo="*" --enablerepo="pgdg15-updates-testing" list available
Last metadata expiration check: 0:02:57 ago on Sun 17 Jul 2022 07:10:56 AM UTC.
Available Packages
postgis33_15.x86_64                        3.3.0-beta2_1.rhel8                 pgdg15-updates-testing
postgis33_15-client.x86_64                 3.3.0-beta2_1.rhel8                 pgdg15-updates-testing
postgis33_15-devel.x86_64                  3.3.0-beta2_1.rhel8                 pgdg15-updates-testing
postgis33_15-docs.x86_64                   3.3.0-beta2_1.rhel8                 pgdg15-updates-testing
postgis33_15-gui.x86_64                    3.3.0-beta2_1.rhel8                 pgdg15-updates-testing
postgis33_15-utils.x86_64                  3.3.0-beta2_1.rhel8                 pgdg15-updates-testing
postgresql15.x86_64                        15.0-beta2_3PGDG.rhel8              pgdg15-updates-testing
postgresql15-contrib.x86_64                15.0-beta2_3PGDG.rhel8              pgdg15-updates-testing
postgresql15-devel.x86_64                  15.0-beta2_3PGDG.rhel8              pgdg15-updates-testing
postgresql15-docs.x86_64                   15.0-beta2_3PGDG.rhel8              pgdg15-updates-testing
postgresql15-libs.x86_64                   15.0-beta2_3PGDG.rhel8              pgdg15-updates-testing
postgresql15-llvmjit.x86_64                15.0-beta2_3PGDG.rhel8              pgdg15-updates-testing
postgresql15-plperl.x86_64                 15.0-beta2_3PGDG.rhel8              pgdg15-updates-testing
postgresql15-plpython3.x86_64              15.0-beta2_3PGDG.rhel8              pgdg15-updates-testing
postgresql15-pltcl.x86_64                  15.0-beta2_3PGDG.rhel8              pgdg15-updates-testing
postgresql15-server.x86_64                 15.0-beta2_3PGDG.rhel8              pgdg15-updates-testing
postgresql15-test.x86_64                   15.0-beta2_3PGDG.rhel8              pgdg15-updates-testing
```

3.  Install PostgreSQL Server and Client

```
yum --disablerepo="*" --enablerepo="pgdg15-updates-testing" -y install postgresql15-server.x86_64 postgresql15.x86_64 postgresql15-contrib.x86_64
```

```
[root@9cced9bedc1c ~]# yum --disablerepo="*" --enablerepo="pgdg15-updates-testing" -y install postgresql15-server.x86_64 postgresql15.x86_64 postgresql15-contrib.x86_64
Last metadata expiration check: 0:03:57 ago on Sun 17 Jul 2022 07:10:56 AM UTC.
Dependencies resolved.
=====================================================================================================
 Package                   Arch        Version                     Repository                   Size
=====================================================================================================
Installing:
 postgresql15              x86_64      15.0-beta2_3PGDG.rhel8      pgdg15-updates-testing      1.8 M
 postgresql15-contrib      x86_64      15.0-beta2_3PGDG.rhel8      pgdg15-updates-testing      771 k
 postgresql15-server       x86_64      15.0-beta2_3PGDG.rhel8      pgdg15-updates-testing      6.8 M
Installing dependencies:
 postgresql15-libs         x86_64      15.0-beta2_3PGDG.rhel8      pgdg15-updates-testing      311 k

Transaction Summary
=====================================================================================================
Install  4 Packages

Total download size: 9.7 M
Installed size: 39 M
Downloading Packages:
(1/4): postgresql15-libs-15.0-beta2_3PGDG.rhel8.x86_64.rpm           169 kB/s | 311 kB     00:01
(2/4): postgresql15-contrib-15.0-beta2_3PGDG.rhel8.x86_64.rpm        354 kB/s | 771 kB     00:02
(3/4): postgresql15-15.0-beta2_3PGDG.rhel8.x86_64.rpm                706 kB/s | 1.8 MB     00:02
(4/4): postgresql15-server-15.0-beta2_3PGDG.rhel8.x86_64.rpm         4.2 MB/s | 6.8 MB     00:01
-----------------------------------------------------------------------------------------------------
Total                                                                2.8 MB/s | 9.7 MB     00:03
Running transaction check
Transaction check succeeded.
Running transaction test
Transaction test succeeded.
Running transaction
  Preparing        :                                                                             1/1
  Installing       : postgresql15-libs-15.0-beta2_3PGDG.rhel8.x86_64                             1/4
  Running scriptlet: postgresql15-libs-15.0-beta2_3PGDG.rhel8.x86_64                             1/4
  Installing       : postgresql15-15.0-beta2_3PGDG.rhel8.x86_64                                  2/4
  Running scriptlet: postgresql15-15.0-beta2_3PGDG.rhel8.x86_64                                  2/4
  Running scriptlet: postgresql15-server-15.0-beta2_3PGDG.rhel8.x86_64                           3/4
  Installing       : postgresql15-server-15.0-beta2_3PGDG.rhel8.x86_64                           3/4
  Running scriptlet: postgresql15-server-15.0-beta2_3PGDG.rhel8.x86_64                           3/4
  Installing       : postgresql15-contrib-15.0-beta2_3PGDG.rhel8.x86_64                          4/4
  Running scriptlet: postgresql15-contrib-15.0-beta2_3PGDG.rhel8.x86_64                          4/4
  Verifying        : postgresql15-15.0-beta2_3PGDG.rhel8.x86_64                                  1/4
  Verifying        : postgresql15-contrib-15.0-beta2_3PGDG.rhel8.x86_64                          2/4
  Verifying        : postgresql15-libs-15.0-beta2_3PGDG.rhel8.x86_64                             3/4
  Verifying        : postgresql15-server-15.0-beta2_3PGDG.rhel8.x86_64                           4/4

Installed:
  postgresql15-15.0-beta2_3PGDG.rhel8.x86_64      postgresql15-contrib-15.0-beta2_3PGDG.rhel8.x86_64
  postgresql15-libs-15.0-beta2_3PGDG.rhel8.x86_64 postgresql15-server-15.0-beta2_3PGDG.rhel8.x86_64

Complete!
```

Alternatively, you can manually install PostgreSQL RPM files used for beta testing here: [https://download.postgresql.org/pub/repos/yum/testing/15/redhat/rhel-8-x86\_64/](https://download.postgresql.org/pub/repos/yum/testing/15/redhat/rhel-8-x86_64/)

### Step 3: Create Database

```
/usr/pgsql-15/bin/postgresql-15-setup initdb
```

```
[root@9cced9bedc1c ~]# /usr/pgsql-15/bin/postgresql-15-setup initdb
Initializing database ... OK
```

### Step 4: Enable Autostart

```
systemctl enable postgresql-15
systemctl start postgresql-15
```

```
[root@9cced9bedc1c ~]# systemctl status postgresql-15.service
● postgresql-15.service - PostgreSQL 15 database server
   Loaded: loaded (/usr/lib/systemd/system/postgresql-15.service; enabled; vendor preset: disabled)
   Active: active (running) since Sun 2022-07-17 07:22:40 UTC; 1min 20s ago
     Docs: https://www.postgresql.org/docs/15/static/
  Process: 3901 ExecStartPre=/usr/pgsql-15/bin/postgresql-15-check-db-dir ${PGDATA} (code=exited, st>
 Main PID: 3906 (postmaster)
    Tasks: 7 (limit: 48480)
   Memory: 19.4M
   CGroup: /system.slice/postgresql-15.service
           ├─3906 /usr/pgsql-15/bin/postmaster -D /var/lib/pgsql/15/data/
           ├─3909 postgres: logger
           ├─3910 postgres: checkpointer
           ├─3911 postgres: background writer
           ├─3913 postgres: walwriter
           ├─3914 postgres: autovacuum launcher
           └─3915 postgres: logical replication launcher

Jul 17 07:22:39 9cced9bedc1c.mylabserver.com systemd[1]: Starting PostgreSQL 15 database server...
Jul 17 07:22:40 9cced9bedc1c.mylabserver.com postmaster[3906]: 2022-07-17 07:22:40.054 UTC [3906] LO>
Jul 17 07:22:40 9cced9bedc1c.mylabserver.com postmaster[3906]: 2022-07-17 07:22:40.054 UTC [3906] HI>
Jul 17 07:22:40 9cced9bedc1c.mylabserver.com systemd[1]: Started PostgreSQL 15 database server.
```

### Step 5: Verify Server via `psql` client.

```
[postgres@9cced9bedc1c log]$ psql
psql (15beta2)
Type "help" for help.

postgres=# select version();
                                                  version

-----------------------------------------------------------------------------------------------------
-------
 PostgreSQL 15beta2 on x86_64-pc-linux-gnu, compiled by gcc (GCC) 8.5.0 20210514 (Red Hat 8.5.0-10),
64-bit
(1 row)
```