[Home](https://oracle-base.com/) » [Articles](https://oracle-base.com/articles) » [19c](https://oracle-base.com/articles/19c) » Here

[Do not install Oracle on Fedora before reading this!](https://oracle-base.com/articles/linux/do-not-install-oracle-on-fedora-before-reading-this)

This article describes the installation of Oracle Database 19c 64-bit on [Fedora 37 (F37)](http://fedoraproject.org/) 64-bit. The article is based on a server installation with a minimum of 2G swap and secure Linux set to permissive. An example of this type of Linux installation can be seen [here](https://oracle-base.com/articles/linux/fedora-37-installation).

-   [Download Software](https://oracle-base.com/articles/19c/oracle-db-19c-installation-on-fedora-37#download-software)
-   [Hosts File](https://oracle-base.com/articles/19c/oracle-db-19c-installation-on-fedora-37#hosts-file)
-   [Set Kernel Parameters](https://oracle-base.com/articles/19c/oracle-db-19c-installation-on-fedora-37#set-kernel-parameters)
-   [Setup](https://oracle-base.com/articles/19c/oracle-db-19c-installation-on-fedora-37#setup)
-   [Installation](https://oracle-base.com/articles/19c/oracle-db-19c-installation-on-fedora-37#installation)
-   [Database Creation](https://oracle-base.com/articles/19c/oracle-db-19c-installation-on-fedora-37#database-creation)
-   [Post Installation](https://oracle-base.com/articles/19c/oracle-db-19c-installation-on-fedora-37#post-installation)

Related articles.

-   [Oracle Universal Installations (OUI) Silent Installations](https://oracle-base.com/articles/misc/oui-silent-installations)
-   [Database Configuration Assistant (DBCA) : Creating Databases in Silent Mode](https://oracle-base.com/articles/misc/database-configuration-assistant-dbca-silent-mode)

## Download Software

Download the Oracle software from OTN or MOS depending on your support status.

-   [OTN: Oracle Database 19c (19.3) Software (64-bit).](http://www.oracle.com/technetwork/database/enterprise-edition/downloads/index.html)
-   [edelivery: Oracle Database 19c (19.3) Software (64-bit)](http://edelivery.oracle.com/)

## Hosts File

The "/etc/hosts" file must contain a fully qualified name for the server.

```
<IP-address>  <fully-qualified-machine-name>  <machine-name>
```

For example.

```
127.0.0.1       localhost localhost.localdomain localhost4 localhost4.localdomain4
192.168.56.141  fedora37.localdomain  fedora37
```

Set the correct hostname in the "/etc/hostname" file.

```
fedora37.localdomain
```

## Set Kernel Parameters

Add the following lines to the "/etc/sysctl.conf" file, or in a file called "/etc/sysctl.d/98-oracle.conf".

```
fs.file-max = 6815744
kernel.sem = 250 32000 100 128
kernel.shmmni = 4096
kernel.shmall = 1073741824
kernel.shmmax = 4398046511104
kernel.panic_on_oops = 1
net.core.rmem_default = 262144
net.core.rmem_max = 4194304
net.core.wmem_default = 262144
net.core.wmem_max = 1048576
net.ipv4.conf.all.rp_filter = 2
net.ipv4.conf.default.rp_filter = 2
fs.aio-max-nr = 1048576
net.ipv4.ip_local_port_range = 9000 65500
```

Run one of the following commands to change the current kernel parameters, depending on which file you edited.

```
/sbin/sysctl -p
# Or
/sbin/sysctl -p /etc/sysctl.d/98-oracle.conf
```

Add the following lines to a file called "/etc/security/limits.d/oracle-database-server-19c-preinstall.conf" file.

```
oracle   soft   nofile    1024
oracle   hard   nofile    65536
oracle   soft   nproc    16384
oracle   hard   nproc    16384
oracle   soft   stack    10240
oracle   hard   stack    32768
oracle   hard   memlock    134217728
oracle   soft   memlock    134217728
```

Stop and disable the firewall. You can configure it later if you wish.

```
# systemctl stop firewalld
# systemctl disable firewalld
```

Set SELinux to permissive by editing the "/etc/selinux/config" file, making sure the SELINUX flag is set as follows.

```
SELINUX=permissive
```

The server will need a reboot for the change to take effect.

## Setup

The following packages are listed as required. Some are commented out as they are not present in the Fedora repository.

```
#dnf groupinstall "GNOME Desktop" -y
#dnf groupinstall "Development Tools" -y
#dnf groupinstall "Administration Tools" -y
#dnf groupinstall "System Tools" -y
dnf install -y bc    
dnf install -y binutils
#dnf install -y compat-libcap1
dnf install -y compat-libstdc++-33
#dnf install -y dtrace-modules
#dnf install -y dtrace-modules-headers
#dnf install -y dtrace-modules-provider-headers
#dnf install -y dtrace-utils
dnf install -y elfutils-libelf
dnf install -y elfutils-libelf-devel
dnf install -y fontconfig-devel
dnf install -y glibc
dnf install -y glibc-devel
dnf install -y ksh
dnf install -y libaio
dnf install -y libaio-devel
#dnf install -y libdtrace-ctf-devel
dnf install -y libXrender
dnf install -y libXrender-devel
dnf install -y libX11
dnf install -y libXau
dnf install -y libXi
dnf install -y libXtst
dnf install -y libgcc
dnf install -y librdmacm-devel
dnf install -y libstdc++
dnf install -y libstdc++-devel
dnf install -y libxcb
dnf install -y make
dnf install -y net-tools # Clusterware
dnf install -y nfs-utils # ACFS
dnf install -y python # ACFS
dnf install -y python-configshell # ACFS
dnf install -y python-rtslib # ACFS
dnf install -y python-six # ACFS
dnf install -y targetcli # ACFS
dnf install -y smartmontools
dnf install -y sysstat

# Added by me.
yum install -y unixODBC

# compat-libpthread-nonshared.
dnf install -y libnsl2
dnf install -y libnsl2.i686
dnf install -y libxcrypt-compat
dnf install -y http://rpmfind.net/linux/fedora/linux/development/rawhide/Everything/x86_64/os/Packages/c/compat-libpthread-nonshared-2.36.9000-13.fc38.x86_64.rpm

#dnf update -y
```

Create the new groups and users.

```
groupadd -g 54321 oinstall
groupadd -g 54322 dba
groupadd -g 54323 oper
#groupadd -g 54324 backupdba
#groupadd -g 54325 dgdba
#groupadd -g 54326 kmdba
#groupadd -g 54328 asmdba
#groupadd -g 54328 asmoper
#groupadd -g 54329 asmadmin

useradd -u 54321 -g oinstall -G dba,oper oracle
passwd oracle
```

We are not going to use the extra groups, but include them if you do plan on using them.

Create the directories in which the Oracle software will be installed.

```
mkdir -p /u01/app/oracle/product/19.0.0/dbhome_1
mkdir -p /u02/oradata
chown -R oracle:oinstall /u01 /u02
chmod -R 775 /u01 /u02
```

Putting mount points directly under root without mounting separate disks to them is typically a bad idea. It's done here for simplicity, but for a real installation "/" storage should be reserved for the OS.

If you are using X Emulation, login as root and issue the following command.

```
xhost +<machine-name>
```

You will need to add the following symbolic links or the Oracle Universal Installer (OUI) will not start.

```
# Fix for Oracle on Fedora.
rm -f /usr/lib64/libnsl.so.1
rm -f /usr/lib/libnsl.so.1
ln -s /usr/lib64/libnsl.so.3.0.0 /usr/lib64/libnsl.so.1
ln -s /usr/lib/libnsl.so.3.0.0 /usr/lib/libnsl.so.1
```

Set up the environment for the "oracle" user. The "$" characters are escaped using "\\". If you are not creating the file with the `cat` command, you will need to remove the escape characters.

```
mkdir -p /home/oracle/scripts


cat > /home/oracle/scripts/setEnv.sh <<EOF
# Oracle Settings
export TMP=/tmp
export TMPDIR=\$TMP

export ORACLE_HOSTNAME=fedora37.localdomain
export ORACLE_UNQNAME=cdb1
export ORACLE_BASE=/u01/app/oracle
export ORACLE_HOME=\$ORACLE_BASE/product/19.0.0/dbhome_1
export ORA_INVENTORY=/u01/app/oraInvenotry
export ORACLE_SID=cdb1
export PDB_NAME=pdb1
export DATA_DIR=/u02/oradata

export PATH=/usr/sbin:/usr/local/bin:\$PATH
export PATH=\$ORACLE_HOME/bin:\$PATH

export LD_LIBRARY_PATH=\$ORACLE_HOME/lib:/lib:/usr/lib
export CLASSPATH=\$ORACLE_HOME/jlib:\$ORACLE_HOME/rdbms/jlib
EOF

echo ". /home/oracle/scripts/setEnv.sh" >> /home/oracle/.bash_profile

chown -R oracle:oinstall /home/oracle/scripts
```

## Installation

Log into the oracle user. If you are using X emulation then set the DISPLAY environmental variable.

```
DISPLAY=<machine-name>:0.0; export DISPLAY
```

Perform a software-only installation either using interactive mode (GUI) or silent mode and run the root scripts when prompted. Notice the setting of the `CV_ASSUME_DISTID` environment variable, so fake the OS.

```
# Unzip software.
cd $ORACLE_HOME
unzip -oq /path/to/software/LINUX.X64_193000_db_home.zip

# Fix for linking error suggested by Steven Kennedy.
cd $ORACLE_HOME/lib/stubs
mv libc.so libc.so.hide
mv libc.so.6 libc.so.6.hide

# Fake OS.
export CV_ASSUME_DISTID=OEL7.8

# Interactive mode.
#./runInstaller

# Silent mode.
./runInstaller -ignorePrereq -waitforcompletion -silent                        \
    -responseFile ${ORACLE_HOME}/install/response/db_install.rsp               \
    oracle.install.option=INSTALL_DB_SWONLY                                    \
    ORACLE_HOSTNAME=${ORACLE_HOSTNAME}                                         \
    UNIX_GROUP_NAME=oinstall                                                   \
    INVENTORY_LOCATION=${ORA_INVENTORY}                                        \
    SELECTED_LANGUAGES=en,en_GB                                                \
    ORACLE_HOME=${ORACLE_HOME}                                                 \
    ORACLE_BASE=${ORACLE_BASE}                                                 \
    oracle.install.db.InstallEdition=EE                                        \
    oracle.install.db.OSDBA_GROUP=dba                                          \
    oracle.install.db.OSBACKUPDBA_GROUP=dba                                    \
    oracle.install.db.OSDGDBA_GROUP=dba                                        \
    oracle.install.db.OSKMDBA_GROUP=dba                                        \
    oracle.install.db.OSRACDBA_GROUP=dba                                       \
    SECURITY_UPDATES_VIA_MYORACLESUPPORT=false                                 \
    DECLINE_SECURITY_UPDATES=true
```

Run the root scripts when prompted.

```
As a root user, execute the following script(s):
        1. /u01/app/oraInvenotry/orainstRoot.sh
        2. /u01/app/oracle/product/19.0.0/dbhome_1/root.sh
```

You can read more about silent installations [here](https://oracle-base.com/articles/misc/oui-silent-installations).

You are now ready to create a database.

## Database Creation

You create a database using the Database Configuration Assistant (DBCA). The interactive mode will display GUI screens to allow user input, while the silent mode will create the database without displaying any screens, as all required options are already specified on the command line.

```
# Start the listener.
lsnrctl start

# Interactive mode.
# dbca

# Silent mode.
dbca -silent -createDatabase                                                   \
     -templateName General_Purpose.dbc                                         \
     -gdbname ${ORACLE_SID} -sid  ${ORACLE_SID} -responseFile NO_VALUE         \
     -characterSet AL32UTF8                                                    \
     -sysPassword SysPassword1                                                 \
     -systemPassword SysPassword1                                              \
     -createAsContainerDatabase true                                           \
     -numberOfPDBs 1                                                           \
     -pdbName ${PDB_NAME}                                                      \
     -pdbAdminPassword PdbPassword1                                            \
     -databaseType MULTIPURPOSE                                                \
     -memoryMgmtType auto_sga                                                  \
     -totalMemory 2000                                                         \
     -storageType FS                                                           \
     -datafileDestination "${DATA_DIR}"                                        \
     -redoLogFileSize 50                                                       \
     -emConfiguration NONE                                                     \
     -ignorePreReqs
```

You can read more about silent database creation [here](https://oracle-base.com/articles/misc/database-configuration-assistant-dbca-silent-mode).

## Post Installation

Edit the "/etc/oratab" file setting the restart flag for each instance to 'Y'.

```
cdb1:/u01/app/oracle/product/19.0.0/dbhome_1:Y
```

For more information see:

-   [Oracle Database 19c : Installation Guide for Linux](https://docs.oracle.com/en/database/oracle/oracle-database/19/ladbi/index.html)
-   [Automating Database Startup and Shutdown on Linux](https://oracle-base.com/articles/linux/automating-database-startup-and-shutdown-on-linux)
-   [Oracle Universal Installations (OUI) Silent Installations](https://oracle-base.com/articles/misc/oui-silent-installations)
-   [Database Configuration Assistant (DBCA) : Creating Databases in Silent Mode](https://oracle-base.com/articles/misc/database-configuration-assistant-dbca-silent-mode)

Hope this helps. Regards Tim...

[Back to the Top.](https://oracle-base.com/articles/19c/oracle-db-19c-installation-on-fedora-37#Top)