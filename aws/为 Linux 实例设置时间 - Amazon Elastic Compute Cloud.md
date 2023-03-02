## 为 Linux 实例设置时间

对于许多服务器任务和进程来说，准确一致的时间参考是非常重要的。大多数系统日志包含时间戳，您可以用来确定问题发生的时间以及事件发生的顺序。如果您使用 AWS CLI 或 AWS 开发工具包从您的实例发送请求，这些工具会以您的名义签署请求。如果您的实例的日期和时间设置不正确，签名中的日期可能与请求的日期不匹配，进而导致 AWS 拒绝请求。

Amazon 提供 Amazon Time Sync Service，该服务可从所有 EC2 实例访问，同样由其他 AWS 服务使用。该服务在每个 AWS 区域中使用一组与卫星连接的原子参考时钟，以通过网络时间协议 (NTP) 提供准确的当前协调世界时 (UTC) 全球标准时间读数。Amazon Time Sync Service 自动消除在 UTC 中添加的任何闰秒。

Amazon Time Sync Service 是通过 NTP（IPv4 地址为 `169.254.169.123` 或 IPv6 地址为 `fd00:ec2::123`）为 VPC 中运行的任何实例提供的。IPv6 地址仅可在 [基于 Nitro 系统构建的实例](https://docs.aws.amazon.com/zh_cn/AWSEC2/latest/UserGuide/instance-types.html#ec2-nitro-instances) 上访问。您的实例不需要访问互联网，并且您不必配置安全组规则或网络 ACL 规则以允许进行访问。最新版本的 Amazon Linux 2 和 Amazon Linux AMI 默认情况下与 Amazon Time Sync Service 同步。

可以使用以下步骤通过 `chrony` 客户端在实例上配置 Amazon Time Sync Service。或者，您也可以使用外部 NTP 源。有关 NTP 和公共时间源的更多信息，请访问 [http://www.ntp.org/](http://www.ntp.org/)。实例需要访问 Internet，外部 NTP 时间源才能正常工作。

对于 Windows 实例，请参阅[为 Windows 实例设置时间](https://docs.aws.amazon.com/AWSEC2/latest/WindowsGuide/windows-set-time.html)。

**主题**

-   [配置具有 IPv4 地址的 EC2 实例的时间](https://docs.aws.amazon.com/zh_cn/AWSEC2/latest/UserGuide/set-time.html#configure-amazon-time-service-amazon-linux-IPv4)
-   [配置具有 IPv6 地址的 EC2 实例的时间](https://docs.aws.amazon.com/zh_cn/AWSEC2/latest/UserGuide/set-time.html#configure-amazon-time-service-amazon-linux-IPv6)
-   [在 Amazon Linux 上更改时区](https://docs.aws.amazon.com/zh_cn/AWSEC2/latest/UserGuide/set-time.html#change_time_zone)
-   [比较时间戳](https://docs.aws.amazon.com/zh_cn/AWSEC2/latest/UserGuide/set-time.html#compare-timestamps-with-clockbound)

## 配置具有 IPv4 地址的 EC2 实例的时间

此部分介绍如何根据 Linux 分发版的类型，为具有 IPv4 地址的 EC2 实例设置时间。

**主题**

-   [在 Amazon Linux AMI 上配置 Amazon Time Sync Service](https://docs.aws.amazon.com/zh_cn/AWSEC2/latest/UserGuide/set-time.html#configure-amazon-time-service-amazon-linux)
-   [在 Ubuntu 上配置 Amazon Time Sync Service](https://docs.aws.amazon.com/zh_cn/AWSEC2/latest/UserGuide/set-time.html#configure-amazon-time-service-ubuntu)
-   [在 SUSE Linux 上配置 Amazon Time Sync Service](https://docs.aws.amazon.com/zh_cn/AWSEC2/latest/UserGuide/set-time.html#configure-amazon-time-service-suse)

### 在 Amazon Linux AMI 上配置 Amazon Time Sync Service

在 Amazon Linux 2 上，`chrony` 已安装并配置为使用 Amazon Time Sync Service IP 地址。

对于 Amazon Linux AMI，您必须编辑 `chrony` 配置文件以添加 Amazon Time Sync Service 的服务器条目。

###### 配置 实例以使用 Amazon Time Sync Service

1.  连接到您的实例并卸载 NTP 服务。
    
    ```
    [ec2-user ~]$ sudo yum erase 'ntp*'
    ```
    
2.  安装 `chrony` 软件包。
    
    ```
    [ec2-user ~]$ sudo yum install chrony
    ```
    
3.  使用任何文本编辑器（如 `/etc/chrony.conf` 或 **vim**）打开 **nano** 文件。确认该文件包含以下行：
    
    ```
    server 169.254.169.123 prefer iburst minpoll 4 maxpoll 4
    ```
    
    如果该行存在，则已配置 Amazon Time Sync Service，您可以转到下一步。如果不存在，请在该文件中已包含的任何其他 `server` 或 `pool` 语句后面添加该行，然后保存您的更改。
    
4.  重启 `chrony` 守护程序 (`chronyd`)。
    
    ```
    [ec2-user ~]$ sudo service chronyd restart
    ```
    
    ```
    Starting chronyd:                                          [  OK  ]
    ```
    
    在 RHEL 和 CentOS (最高版本为 6) 上，服务名称是 `chrony` 而不是 `chronyd`。
    
5.  使用 `chkconfig` 命令可将 `chronyd` 配置为在每次系统启动时启动。
    
    ```
    [ec2-user ~]$ sudo chkconfig chronyd on
    ```
    
6.  确认 `chrony` 使用 `169.254.169.123` IP 地址同步时间。
    
    ```
    [ec2-user ~]$ chronyc sources -v
    ```
    
    ```
    210 Number of sources = 7
            
              .-- Source mode  '^' = server, '=' = peer, '#' = local clock.
             / .- Source state '*' = current synced, '+' = combined , '-' = not combined,
            | /   '?' = unreachable, 'x' = time may be in error, '~' = time too variable.
            ||                                                 .- xxxx [ yyyy ] +/- zzzz
            ||      Reachability register (octal) -.           |  xxxx = adjusted offset,
            ||      Log2(Polling interval) --.      |          |  yyyy = measured offset,
            ||                                \     |          |  zzzz = estimated error.
            ||                                 |    |           \
            MS Name/IP address         Stratum Poll Reach LastRx Last sample               
            ===============================================================================
            ^* 169.254.169.123               3   6    17    43    -30us[ -226us] +/-  287us
            ^- ec2-12-34-231-12.eu-west>     2   6    17    43   -388us[ -388us] +/-   11ms
            ^- tshirt.heanet.ie              1   6    17    44   +178us[  +25us] +/- 1959us
            ^? tbag.heanet.ie                0   6     0     -     +0ns[   +0ns] +/-    0ns
            ^? bray.walcz.net                0   6     0     -     +0ns[   +0ns] +/-    0ns
            ^? 2a05:d018:c43:e312:ce77:>     0   6     0     -     +0ns[   +0ns] +/-    0ns
            ^? 2a05:d018:dab:2701:b70:b>     0   6     0     -     +0ns[   +0ns] +/-    0ns
    ```
    
    在返回的输出中，`^*` 指示首选的时间源。
    
7.  验证 `chrony` 报告的时间同步指标。
    
    ```
    [ec2-user ~]$ chronyc tracking
    ```
    
    ```
    Reference ID    : A9FEA97B (169.254.169.123)
            Stratum         : 4
            Ref time (UTC)  : Wed Nov 22 13:18:34 2017
            System time     : 0.000000626 seconds slow of NTP time
            Last offset     : +0.002852759 seconds
            RMS offset      : 0.002852759 seconds
            Frequency       : 1.187 ppm fast
            Residual freq   : +0.020 ppm
            Skew            : 24.388 ppm
            Root delay      : 0.000504752 seconds
            Root dispersion : 0.001112565 seconds
            Update interval : 64.4 seconds
            Leap status     : Normal
    ```
    

### 在 Ubuntu 上配置 Amazon Time Sync Service

您必须编辑 `chrony` 配置文件以添加 Amazon Time Sync Service 的服务器条目。

###### 配置 实例以使用 Amazon Time Sync Service

1.  连接到您的实例并使用 `apt` 安装 `chrony` 软件包。
    
    ```
    ubuntu:~$ sudo apt install chrony
    ```
    
    如有必要，请先运行 `sudo apt update` 以更新您的实例。
    
2.  使用任何文本编辑器（如 `/etc/chrony/chrony.conf` 或 **vim**）打开 **nano** 文件。在该文件中已包含的任何其他 `server` 或 `pool` 语句前面添加以下行，然后保存您的更改：
    
    ```
    server 169.254.169.123 prefer iburst minpoll 4 maxpoll 4
    ```
    
3.  重新启动 `chrony` 服务。
    
    ```
    ubuntu:~$ sudo /etc/init.d/chrony restart
    ```
    
    ```
    Restarting chrony (via systemctl): chrony.service.
    ```
    
4.  确认 `chrony` 使用 `169.254.169.123` IP 地址同步时间。
    
    ```
    ubuntu:~$ chronyc sources -v
    ```
    
    ```
    210 Number of sources = 7
                
                  .-- Source mode  '^' = server, '=' = peer, '#' = local clock.
                 / .- Source state '*' = current synced, '+' = combined , '-' = not combined,
                | /   '?' = unreachable, 'x' = time may be in error, '~' = time too variable.
                ||                                                 .- xxxx [ yyyy ] +/- zzzz
                ||      Reachability register (octal) -.           |  xxxx = adjusted offset,
                ||      Log2(Polling interval) --.      |          |  yyyy = measured offset,
                ||                                \     |          |  zzzz = estimated error.
                ||                                 |    |           \
                MS Name/IP address         Stratum Poll Reach LastRx Last sample
                ===============================================================================
                ^* 169.254.169.123               3   6    17    12    +15us[  +57us] +/-  320us
                ^- tbag.heanet.ie                1   6    17    13  -3488us[-3446us] +/- 1779us
                ^- ec2-12-34-231-12.eu-west-     2   6    17    13   +893us[ +935us] +/- 7710us
                ^? 2a05:d018:c43:e312:ce77:6     0   6     0   10y     +0ns[   +0ns] +/-    0ns
                ^? 2a05:d018:d34:9000:d8c6:5     0   6     0   10y     +0ns[   +0ns] +/-    0ns
                ^? tshirt.heanet.ie              0   6     0   10y     +0ns[   +0ns] +/-    0ns
                ^? bray.walcz.net                0   6     0   10y     +0ns[   +0ns] +/-    0ns
    ```
    
    在返回的输出中，`^*` 指示首选的时间源。
    
5.  验证 `chrony` 报告的时间同步指标。
    
    ```
    ubuntu:~$ chronyc tracking
    ```
    
    ```
    Reference ID    : 169.254.169.123 (169.254.169.123)
                Stratum         : 4
                Ref time (UTC)  : Wed Nov 29 07:41:57 2017
                System time     : 0.000000011 seconds slow of NTP time
                Last offset     : +0.000041659 seconds
                RMS offset      : 0.000041659 seconds
                Frequency       : 10.141 ppm slow
                Residual freq   : +7.557 ppm
                Skew            : 2.329 ppm
                Root delay      : 0.000544 seconds
                Root dispersion : 0.000631 seconds
                Update interval : 2.0 seconds
                Leap status     : Normal
    ```
    

### 在 SUSE Linux 上配置 Amazon Time Sync Service

从 [https://software.opensuse.org/package/chrony](https://software.opensuse.org/package/chrony) 安装 chrony。

使用任何文本编辑器（如 `/etc/chrony.conf` 或 **vim**）打开 **nano** 文件。确认该文件包含以下行：

```
server 169.254.169.123 prefer iburst minpoll 4 maxpoll 4
```

如果此行不存在，请添加它。注释掉任何其他服务器或池行。打开 yaST 并启用 chrony 服务。

## 配置具有 IPv6 地址的 EC2 实例的时间

此部分介绍了如果您为使用 IPv6 地址的 EC2 实例配置 Amazon Time Sync Service，在 [配置具有 IPv4 地址的 EC2 实例的时间](https://docs.aws.amazon.com/zh_cn/AWSEC2/latest/UserGuide/set-time.html#configure-amazon-time-service-amazon-linux-IPv4) 中介绍的流程有何不同。它没有解释整个 Amazon Time Sync Service 配置流程。IPv6 地址仅可在 [基于 Nitro 系统构建的实例](https://docs.aws.amazon.com/zh_cn/AWSEC2/latest/UserGuide/instance-types.html#ec2-nitro-instances) 上访问。

我们不建议在您的 chrony.conf 文件中同时使用 IPv4 地址和 IPv6 地址条目。IPv4 和 IPv6 NTP 数据包来自您的实例的同一个本地服务器。如果同时使用来自 IPv4 端点的一些数据包和来自 IPv6 端点的一些数据包，则很可能会得到混合的结果。

根据您使用的 Linux 发行版，当您到达编辑 chrony.conf 文件的步骤时，您将使用 Amazon Time Sync Service的 IPv6 端点 (`fd00:ec2::123`) 而不是 IPv4 端点 (`169.254.169.123`)：

```
server fd00:ec2::123 prefer iburst minpoll 4 maxpoll 4
```

保存文件并确认 `chrony` 使用 `fd00:ec2::123` IPv6 地址同步时间：

```
[ec2-user ~]$ chronyc sources -v
```

在输出中，如果您看到 `fd00:ec2::123` IPv6 地址，则配置完成。

## 在 Amazon Linux 上更改时区

Amazon Linux 实例默认设置为 UTC（协调世界时）时区。您可以将实例上的时间更改为本地时间或网络中的其他时区。

此信息适用于 Amazon Linux。有关其他发布版本的信息，请参阅特定于该版本的文档。

###### 要更改 Amazon Linux 2 实例上的时区

1.  查看系统的当前时区设置。
    
    ```
    [ec2-user ~]$ timedatectl
    ```
    
2.  列出可用的时区。
    
    ```
    [ec2-user ~]$ timedatectl list-timezones
    ```
    
3.  设置选定的时区。
    
    ```
    [ec2-user ~]$ sudo timedatectl set-timezone America/Vancouver
    ```
    
4.  （可选）通过运行 **timedatectl** 命令，确认当前时区已更新为新时区。
    
    ```
    [ec2-user ~]$ timedatectl
    ```
    

###### 要更改 Amazon Linux 实例上的时区

1.  确定将在实例上使用的时区。`/usr/share/zoneinfo` 目录包含时区数据文件的层次结构。浏览该位置的目录结构，查找针对您的时区的文件。
    
    ```
    [ec2-user ~]$ ls /usr/share/zoneinfo
    Africa      Chile    GB         Indian       Mideast   posixrules  US
    America     CST6CDT  GB-Eire    Iran         MST       PRC         UTC
    Antarctica  Cuba     GMT        iso3166.tab  MST7MDT   PST8PDT     WET
    Arctic      EET      GMT0       Israel       Navajo    right       W-SU
    ...
    ```
    
    该位置的部分条目是目录 (如 `America`)，这些目录包含针对特定城市的时区文件。查找要用于实例的城市 (或时区中的一个城市)。
    
2.  使用新时区更新 `/etc/sysconfig/clock` 文件。在此示例中，我们使用洛杉矶的时区数据文件 `/usr/share/zoneinfo/America/Los_Angeles`。
    
    1.  使用您常用的文本编辑器（如 `/etc/sysconfig/clock` 或 **vim**）打开 **nano** 文件。您需要在编辑器命令中使用 **sudo**，因为 `/etc/sysconfig/clock` 归 `root` 所有。
        
        ```
        [ec2-user ~]$ sudo nano /etc/sysconfig/clock
        ```
        
    2.  查找 `ZONE` 条目，将其更改为时区文件 (忽略路径的 `/usr/share/zoneinfo` 部分)。例如，要更改为洛杉矶时区，请将 `ZONE` 条目更改为以下内容：
        
        ```
        ZONE="America/Los_Angeles"
        ```
        
        请勿将 `UTC=true` 条目更改为其他值。此条目用于硬件时钟；如果您在实例上设置了其他时区，则无需调整此条目。
        
    3.  保存文件，退出文本编辑器。
        
3.  在 `/etc/localtime` 与时区文件之间创建一个符号链接，以便实例在引用本地时间信息时找到此时区文件。
    
    ```
    [ec2-user ~]$ sudo ln -sf /usr/share/zoneinfo/America/Los_Angeles /etc/localtime
    ```
    
4.  重启系统，以便所有服务和应用程序接受新时区信息。
    
    ```
    [ec2-user ~]$ sudo reboot
    ```
    
5.  （可选）使用 **date** 命令确认当前时区已更新为新时区。当前时区将显示在输出中。在下面的示例中，当前时区是 PDT，它指的是洛杉矶时区。
    
    ```
    [ec2-user ~]$ date
    Sun Aug 16 05:45:16 PDT 2020
    ```
    

## 比较时间戳

如果您使用的是 Amazon Time Sync Service，就可以将 Amazon EC2 实例上的时间戳与 ClockBound 进行比较，以确定事件的真实时间。ClockBound 可以测定 EC2 实例的时钟的准确性，并允许您检查一个给定的时间戳是早于还是晚于实例的当前时钟。在确定整个 EC2 实例中的事件与事务的顺序和一致性方面，这些信息十分重要，并且不会收到各个实例的地理位置的影响。

ClockBound 是一个开源守护进程和开源库。要了解关于 ClockBound 的详情，包括安装说明，请参阅 [GitHub](https://github.com/aws/clock-bound) 上的 _ClockBound_。