chrony 是网络时间协议（NTP）的通用实现。

chrony 包含两个程序：chronyd 是一个可以在启动时启动的守护程序。chronyc 是一个命令行界面程序，用于监视 chronyd 的性能并在运行时更改各种操作参数。

与其它时间同步软件的对比：[https://chrony.tuxfamily.org/comparison.html](https://chrony.tuxfamily.org/comparison.html)

## 一、安装与配置

```
yum -y install chrony

systemctl enable chronyd
systemctl start chronyd

vim /etc/chrony.conf
```

**chrony.conf 默认配置**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
# 使用 pool.ntp.org 项目中的公共服务器。以server开，理论上想添加多少时间服务器都可以。
# Use public servers from the pool.ntp.org project.
# Please consider joining the pool (http://www.pool.ntp.org/join.html).
server 0.centos.pool.ntp.org iburst
server 1.centos.pool.ntp.org iburst
server 2.centos.pool.ntp.org iburst
server 3.centos.pool.ntp.org iburst

# 根据实际时间计算出服务器增减时间的比率，然后记录到一个文件中，在系统重启后为系统做出最佳时间补偿调整。
# Record the rate at which the system clock gains/losses time.
driftfile /var/lib/chrony/drift

# 如果系统时钟的偏移量大于1秒，则允许系统时钟在前三次更新中步进。
# Allow the system clock to be stepped in the first three updates if its offset is larger than 1 second.
makestep 1.0 3

# 启用实时时钟（RTC）的内核同步。
# Enable kernel synchronization of the real-time clock (RTC).
rtcsync

# 通过使用 hwtimestamp 指令启用硬件时间戳
# Enable hardware timestamping on all interfaces that support it.
#hwtimestamp *

# Increase the minimum number of selectable sources required to adjust the system clock.
#minsources 2

# 指定 NTP 客户端地址，以允许或拒绝连接到扮演时钟服务器的机器
# Allow NTP client access from local network.
#allow 192.168.0.0/16

# Serve time even if not synchronized to a time source.
#local stratum 10

# 指定包含 NTP 身份验证密钥的文件。
# Specify file containing keys for NTP authentication.
#keyfile /etc/chrony.keys

# 指定日志文件的目录。
# Specify directory for log files.
logdir /var/log/chrony

# 选择日志文件要记录的信息。
# Select which information is logged.
#log measurements statistics tracking
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

## 二、使用

ntp 服务器：[https://www.cnblogs.com/jhxxb/p/10579816.html](https://www.cnblogs.com/jhxxb/p/10579816.html)

### 1.服务端配置

chrony.conf 修改两处

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
# Use public servers from the pool.ntp.org project.
# Please consider joining the pool (http://www.pool.ntp.org/join.html).
server s1a.time.edu.cn iburst
server ntp.aliyun.com iburst

# Allow NTP client access from local network.
allow 192.168.8.0/24
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

开启同步

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
systemctl enable chronyd
systemctl restart chronyd

# 查看时间同步状态
timedatectl status
# 开启网络时间同步
timedatectl set-ntp true
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

### 2.客户端配置

chrony.conf 修改两处

```
# Use public servers from the pool.ntp.org project.
# Please consider joining the pool (http://www.pool.ntp.org/join.html).
server 192.168.8.5 iburst

# Allow NTP client access from local network.
allow 192.168.8.5
```

开启同步

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
systemctl enable chronyd
systemctl restart chronyd

# 查看时间同步状态
timedatectl status
# 开启网络时间同步
timedatectl set-ntp true
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

## 三、命令

chronyc 用法

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
# 查看 ntp_servers
chronyc sources -v

# 查看 ntp_servers 状态
chronyc sourcestats -v

# 查看 ntp_servers 是否在线
chronyc activity -v

# 查看 ntp 详细信息
chronyc tracking -v
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

修改时区

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
# 查看日期时间、时区及 NTP 状态
timedatectl

# 查看时区列表
timedatectl list-timezones
timedatectl list-timezones |  grep  -E "Asia/S.*"

# 修改时区
timedatectl set-timezone Asia/Shanghai

# 修改日期时间（可以只修改其中一个）
timedatectl set-time "2019-09-19 15:50:20"

# 开启 NTP
timedatectl set-ntp true/flase
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

___

**[https://chrony.tuxfamily.org/](https://chrony.tuxfamily.org/)**

**[https://www.linuxprobe.com/centos7-chrony-time.html](https://www.linuxprobe.com/centos7-chrony-time.html)**