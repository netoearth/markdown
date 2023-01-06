**本章目录:**

-   0x04 Redis 配置文件
    

![](https://i0.hdslb.com/bfs/article/4aa545dccf7de8d4a93c2b2b8e3265ac0a26d216.png@progressive.webp)

本篇水一篇，针对于Redis服务配置文件进行简单翻译，以及在后面记录了Redis常用的配置关键参数，供大家试用。

Redis 版本配置基于: 最新版本 6.2.5  ，当前时间节点: 2021年9月10日 14:31:05  

**官方配置示例文件解析:**

```
# Redis configuration file example.
# ./redis-server /path/to/redis.conf

# Note on units: when memory size is needed, it is possible to specify
# 单元不区分大小写, 它通常采用1k 5GB 4M等形式：
# 1k => 1000 bytes
# 1kb => 1024 bytes
# 1m => 1000000 bytes
# 1mb => 1024*1024 bytes
# 1g => 1000000000 bytes
# 1gb => 1024*1024*1024 bytes

################################## INCLUDES ###################################
# 指定包含其它的配置文件，可以在同一主机上多个Redis实例之间使用同一份配置文件，而同时各个实例又拥有自己的特定配置文件，值得注意是命令“CONFIG REWRITE”不会重写选项“include”。
# include /path/to/local.conf
# include /path/to/other.conf

################################## MODULES #####################################
# 在启动时加载模块可以使用多个loadmodule指令，如果服务器无法加载模块它将中止。
# loadmodule /path/to/my_module.so
# loadmodule /path/to/other_module.so

################################## NETWORK #####################################
# 服务绑定的接口支持IPV6
# bind 192.168.1.100 10.0.0.1     # listens on two specific IPv4 addresses
# bind 127.0.0.1 ::1              # listens on loopback IPv4 and IPv6
# bind * -::*                     # like the default, all available interfaces
# 如果您确定希望您的实例侦听所有接口，只需注释掉以下行。
bind 127.0.0.1 -::1

# 受保护模式是一层安全保护，以避免访问和利用互联网上打开的Redis实例,当且仅当如下情况需要设置为YES
# 1）服务器未使用“bind”指令显式绑定到一组地址。
# 2）未配置密码
protected-mode no

# Accept connections on the specified port, default is 6379 (IANA #815344).
# 如果指定端口 0 Redis将不会侦听TCP套接字。
port 6379

# 在每秒高请求的环境中，您需要高积压工作，以避免客户端连接速度慢的问题。
# TCP listen() backlog.
tcp-backlog 511

# 指定用于侦听传入连接的Unix套接字的路径
# Unix socket.
unixsocket /run/redis.sock
unixsocketperm 700

# 客户端空闲N秒后关闭连接（0表示禁用）
timeout 0

# TCP 存活保持时间: 在Linux上指定的值（以秒为单位）是用于发送ACK的时间段,如果非零则设置SO_KEEPALIVE选项来向空闲连接的客户端发送ACK
tcp-keepalive 300

################################# TLS/SSL #####################################
# By default, TLS/SSL is disabled. To enable it, the "tls-port" configuration.
port 0
tls-port 6379

# PEM格式：配置X.509证书和私钥以及密钥加密字符串，用于向连接的客户端、主机或群集对等方验证服务器。
tls-cert-file redis.crt
tls-key-file redis.key
tls-key-file-pass secret


# PEM格式：通常Redis对服务器功能（接受连接）和客户端功能（从主机复制、建立集群总线连接等）使用相同的证书。
# 有时颁发的证书具有将其指定为仅客户端证书或仅服务器证书的属性。
# 在这种情况下可能需要为传入（服务器）和传出（客户端）连接使用不同的证书。
tls-client-cert-file client.crt
tls-client-key-file client.key
tls-client-key-file-pass secret

# 配置DH参数文件以启用Diffie-Hellman（DH）密钥交换：
tls-dh-params-file redis.dh

# 配置CA证书捆绑包或目录以对TLS/SSL客户端和对等方进行身份验证
tls-ca-cert-file ca.crt
tls-ca-cert-dir /etc/ssl/certs


# 默认情况下，TLS端口上的客户端（包括副本服务器）需要使用有效的客户端证书进行身份验证。
# 如果指定“否-no”，则不需要客户端证书，也不接受客户端证书。
# 如果指定了“可选-optional”，则接受客户端证书，如果提供，则必须有效，但不是必需的。
tls-auth-clients no

# 默认情况下，Redis复制副本不会尝试与其主机建立TLS连接，可采用以下配置启用。
tls-replication yes

# 默认情况下，Redis群集总线使用普通TCP连接，可采用以下配置启用。
tls-cluster yes

# 默认情况下，仅启用TLSv1.2和TLSv1.3，强烈建议禁用正式弃用的旧版本，以减少攻击面。
# 显式指定要支持的TLS版本
tls-protocols "TLSv1.2 TLSv1.3"

# 配置SSL允许的密码,注：此配置仅适用于<=TLSv1.2。
tls-ciphers DEFAULT:!MEDIUM

# 配置允许的TLSv1.3密码套件
tls-ciphersuites TLS_CHACHA20_POLY1305_SHA256

# 默认情况下，服务器遵循客户机的首选项，选择密码时请使用服务器首选项而不是客户端首选项。
tls-prefer-server-ciphers yes

# 默认情况下，TLS会话缓存被启用，以允许支持它的客户端更快、更便宜地重新连接, 使用以下指令禁用缓存。
tls-session-caching no

# 更改缓存的TLS会话的默认数量。零值将缓存设置为无限大小。默认超时为20480
tls-session-cache-size 5000

# 更改缓存TLS会话的默认超时。默认超时为300秒。
tls-session-cache-timeout 60


################################# GENERAL #####################################
# 后台守护进程启动
daemonize yes

# 如果您从upstart或systemd运行Redis，Redis可以与您的监控树交互。选项：
#   supervised no      - no supervision interaction
#   supervised upstart - signal upstart by putting Redis into SIGSTOP mode requires "expect stop" in your upstart job config
#   supervised systemd - signal systemd by writing READY=1 to $NOTIFY_SOCKET on startup, and updating Redis status on a regular basis.
#   supervised auto    - detect upstart or systemd method based on  UPSTART_JOB or NOTIFY_SOCKET environment variables
# 默认值为“否”，要在upstart/systemd下运行，只需取消对以下行的注释：
supervised auto

# 如果指定了pid文件，Redis会在启动时将其写入指定的位置，并在退出时将其删除。
# 守护运行时未指定fidfile默认创建/var/run/redis.pid，而非守护运行时则不会默认创建pid文件
pidfile /var/run/redis_6379.pid

# 指定服务器详细级别
# debug (a lot of information, useful for development/testing)
# verbose (many rarely useful info, but not a mess like the debug level)
# notice (moderately verbose, what you want in production probably)
# warning (only very important / critical messages are logged)
loglevel notice

# 指定日志文件名, 使用空字符串强制Redis登录标准输出, 但使用daemonize，则日志将发送到/dev/null
logfile ""

# 是否启用对系统记录器的日志记录
syslog-enabled no

# 指定系统日志标识。
syslog-ident redis

# 指定系统日志工具。必须是用户或介于LOCAL0-LOCAL7之间。
syslog-facility local0

# 要禁用内置崩溃日志（在需要时可能会产生更干净的内核转储），请取消注释以下内容：
crash-log-enabled no

# 要禁用作为崩溃日志一部分运行的快速内存检查（这可能会让redis更快终止），请取消注释以下内容：
crash-memcheck-enabled no

# 设置数据库的数量，默认数据库是DB 0~DB 15（16个）
databases 16

# 默认情况下，Redis仅在开始记录到标准输出以及标准输出为TTY且系统日志记录被禁用时才显示ASCII art徽标
always-show-logo no

# 默认情况下 Redis 修改流程标题（如“top”和“ps”中所示），以提供一些运行时信息。通过将下面的设置为“否”，可以禁用此选项并保持进程名称为“已执行”。
set-proc-title yes

# 自定义Redis交互式终端标题，模板变量在花括号中指定。支持以下变量：
# {title}           Name of process as executed if parent, or type of child process.
# {listen-addr}     Bind address or '*' followed by TCP or TLS port listening on, or Unix socket if only that's available.
# {server-mode}     Special mode, i.e. "[sentinel]" or "[cluster]".
# {port}            TCP port listening on, or 0.
# {tls-port}        TLS port listening on, or 0.
# {unixsocket}      Unix domain socket listening on, or "".
# {config-file}     Name of configuration file used.
proc-title-template "{title} {listen-addr} {server-mode}"


################################ SNAPSHOTTING  ################################
# 工作目录，例如DB将写入此目录中，使用上面使用“dbfilename”配置指令指定的文件名。
dir ./

# 转储数据库的文件名
dbfilename dump.rdb

# 将数据库保存到磁盘, 如果给定的秒数和对数据库执行的写入操作数同时发生，Redis将保存数据库。
# 分别表示3600秒（15分钟）内有1个更改，300秒（5分钟）内有10个更改以及60秒内有10000个更改。
save 3600 1
save 300 100
save 60 10000

# 默认情况下，如果启用了RDB快照（至少一个保存点），并且最新的后台保存失败，Redis将停止接受写入。
# 但是，如果您已经设置了对Redis服务器和持久性的正确监视，则可能需要禁用此功能，以便即使在磁盘、权限等方面出现问题时，Redis仍能正常工作。
stop-writes-on-bgsave-error yes

# 转储.rdb数据库时使用LZF压缩字符串对象(当空间有多余的时候建议以空间换取时间) - 性能考虑可设置为 no
# 如果要在保存子项中保存一些CPU，请将其设置为“否”，但如果有可压缩的值或键，则数据集可能会更大。
rdbcompression yes

# 是否进行本地数据库rdb文件校验 - 性能考虑可设置为 no
rdbchecksum yes

# 加载RDB或还原负载时，启用或禁用ziplist和listpack等的完全卫生检查，这减少了以后处理命令时断言或崩溃的机会。
# Options:
#   no         - Never perform full sanitation
#   yes        - Always perform full sanitation
#   clients    - Perform full sanitation only for user connections.
#                Excludes: RDB files, RESTORE commands received from the master
#                connection, and client connections which have the skip-sanitize-payload ACL flag.
# 默认值应为“clients”，但由于它当前会通过MIGRATE影响群集重新存储，因此默认情况下会临时设置为“no”。
sanitize-dump-payload no

# 在未启用持久性的实例中删除复制使用的RDB文件，默认情况下此选项处于禁用状态，请注意此选项仅适用于同时禁用AOF和RDB持久性的实例，否则将完全忽略。
rdb-del-sync-files no


################################# REPLICATION #################################
# 主副本复制,使用Replicatof将Redis实例制作为另一台Redis服务器的副本。
#
#   +------------------+      +---------------+
#   |      Master      | ---> |    Replica    |
#   | (receive writes) |      |  (exact copy) |
#   +------------------+      +---------------+
#
# 1) Redis replication is asynchronous, but you can configure a master to
#    stop accepting writes if it appears to be not connected with at least a given number of replicas.
# 2) Redis replicas are able to perform a partial resynchronization with the
#    master if the replication link is lost for a relatively small amount of
#    time. You may want to configure the replication backlog size (see the next sections of this file) with a sensible value depending on your needs.
# 3) Replication is automatic and does not need user intervention. After a
#    network partition replicas automatically try to reconnect to masters and resynchronize with them.
replicaof <masterip> <masterport>

# 如果主机受账号与密码保护（和使用下面的“requirepass”配置指令），则可以在启动复制同步过程之前通知复制副本进行身份验证，否则主机将拒绝复制副本请求。
# 当指定masteruser时，复制副本将使用新的身份验证形式：AUTH<username><password>对其master进行身份验证。
masteruser <username>
masterauth <master-password>

# 当复制副本失去与主机的连接时，或者复制仍在进行时，复制副本可以以两种不同的方式工作：
# 1）如果将“副本服务过时数据”设置为“是”（默认值），则副本仍将响应客户端请求，可能包含过期数据，或者如果这是第一次同步，则数据集可能为空。
# 2）如果副本服务过时数据设置为“否”，则副本将以错误“与正在进行的主机同步”回复所有命令，但以下命令除外：INFO、replicof、AUTH、PING、SHUTDOWN、REPLCONF、ROLE、CONFIG、SUBSCRIBE、UNSUBSCRIBE、PSUBSCRIBE、PUNSUBSCRIBE、PUBLISH、PUBSUB、COMMAND、POST、HOST和LATENCY。
replica-serve-stale-data yes

# 将副本实例配置为接受或不接受写入，将副本实例配置为接受或不接受写入。
replica-read-only yes

# 复制同步策略：磁盘或套接字，传输可以通过两种不同的方式进行：
# 1) Disk-backed: The Redis master creates a new process that writes the RDB file on disk. Later the file is transferred by the parent process to the replicas incrementally.
# 2) Diskless: The Redis master creates a new process that directly writes the RDB file to replica sockets, without touching the disk at all.
# 使用磁盘备份复制，在生成RDB文件的同时，只要生成RDB文件的当前子级完成其工作，就可以将更多副本排入队列并与RDB文件一起提供服务。
# 使用无盘复制，在传输开始后，到达的新复制副本将排队，并且在当前复制副本终止时，将开始新的传输。
# 对于慢速磁盘和快速（大带宽）网络，无盘复制效果更好。
repl-diskless-sync no

# 启用无盘复制时，可以配置服务器等待的延迟，以便生成通过套接字将RDB传输到副本的子级。
repl-diskless-sync-delay 5


# 复制副本可以直接从套接字加载从复制链接读取的RDB，或者将RDB存储到一个文件中，并在完全从主机接收到该文件后读取该文件。
# "disabled"    - Don't use diskless load (store the rdb file to the disk first)
# "on-empty-db" - Use diskless load only when it is completely safe.
# "swapdb"      - Keep a copy of the current db contents in RAM while parsing the data directly from the socket. note that this requires sufficient memory, if you don't have it, you risk an OOM kill.
repl-diskless-load disabled

# 副本以预定义的间隔向服务器发送ping。可以使用xx选项更改此间隔。默认值为10秒。
repl-ping-replica-period 10

# 以下选项设置的复制超时：
# 1) Bulk transfer I/O during SYNC, from the point of view of replica.
# 2) Master timeout from the point of view of replicas (data, pings).
# 3) Replica timeout from the point of view of masters (REPLCONF ACK pings).

# 请务必确保此值大于为repl ping复制周期指定的值，否则每次主副本和复制副本之间的通信量较低时都会检测到超时。默认值为60秒。
repl-timeout 60


# 同步后在副本套接字上禁用TCP_节点延迟？
# 如果选择“是”，Redis将使用更少的TCP数据包和更少的带宽向副本发送数据。但这会增加数据在副本端显示的延迟，对于使用默认配置的Linux内核，延迟可达40毫秒。
# 如果选择“否”，则数据出现在副本端的延迟将减少，但复制将使用更多带宽。
repl-disable-tcp-nodelay no

# Set the replication backlog size.
# 复制积压越大，副本承受断开连接的时间越长，以后能够执行部分重新同步。
repl-backlog-size 1mb

# 主服务器在一段时间内没有连接的副本后，积压工作将被释放。以下选项配置从最后一个复制副本断开连接开始释放积压缓冲区所需的秒数。
repl-backlog-ttl 3600

# Redis Sentinel使用它来选择复制副本，以便在主副本不再正常工作时升级到主副本(副本优先级)。
# 优先级较低的副本被认为更适合升级，因此例如，如果有三个副本的优先级为10、100、25 Sentinel将选择优先级为10的，即最低优先级。
replica-priority 100

# 默认情况下，Redis Sentinel在其报告中包含所有副本.
# "sentinel replicas <master>" 命令将忽略未通知的副本，并且不会向Redis sentinel的客户端公开。
replica-announced yes


# 如果连接的副本少于N个，且延迟小于或等于M秒，则主机可能停止接受写入。
# 此选项不保证N个副本将接受写入，但在没有足够的副本可用的情况下，将丢失写入的暴露窗口限制在指定的秒数内。
# 例如，将一个或另一个设置为0将禁用该功能。默认情况下，要写入的最小副本数设置为0（功能已禁用），最小副本数最大延迟设置为10。
# 例如，要要求至少3个延迟<=10秒的副本，请使用：
min-replicas-to-write 3
min-replicas-max-lag 10


# Redis主机能够以不同的方式列出连接副本的地址和端口。
# 例如，“信息复制”部分提供此信息，除其他工具外，Redis Sentinel使用此信息来发现副本实例，此信息可用的另一个位置是主控器的“角色”命令的输出中。
replica-announce-ip 5.5.5.5
replica-announce-port 1234


############################### KEYS TRACKING #################################
# Redis实现了对客户端缓存值的服务器辅助支持。这是使用一个失效表实现的，该表使用按键名索引的基数键来记住客户机拥有哪些键。
# 参考: https://redis.io/topics/client-side-caching
tracking-table-max-keys 1000000


################################## SECURITY ###################################
# 警告：由于Redis速度非常快，外部用户每秒可以在一个redis上尝试多达100万个密码, 我们可以采用acl进行实现安全。

# Redis ACL users are defined in the following format:
#   user <username> ... acl rules ...
# For example:
#   user worker +@list +@connection ~jobs:* on >ffa9203c493aa99
#
# 特殊用户名“default”用于新连接。如果此用户具有“nopass”规则，则新连接将立即作为“默认”用户进行身份验证，而无需通过AUTH命令提供任何密码。否则，如果“默认”用户未标记为“nopass”
# 这些连接将在未验证状态下启动，并且需要AUTH（或HELLO命令AUTH选项）才能进行验证和验证
#
# The ACL rules that describe what a user can do are the following:
#
#  on           Enable the user: it is possible to authenticate as this user.
#  off          Disable the user: it's no longer possible to authenticate
#               with this user, however the already authenticated connections
#               will still work.
#  skip-sanitize-payload    RESTORE dump-payload sanitation is skipped.
#  sanitize-payload         RESTORE dump-payload is sanitized (default).
#  +<command>   Allow the execution of that command
#  -<command>   Disallow the execution of that command
#  +@<category> Allow the execution of all the commands in such category
#               with valid categories are like @admin, @set, @sortedset, ...
#               and so forth, see the full list in the server.c file where
#               the Redis command table is described and defined.
#               The special category @all means all the commands, but currently
#               present in the server, and that will be loaded in the future
#               via modules.
#  +<command>|subcommand    Allow a specific subcommand of an otherwise
#                           disabled command. Note that this form is not
#                           allowed as negative like -DEBUG|SEGFAULT, but only additive starting with "+".
#  allcommands  Alias for +@all. Note that it implies the ability to execute
#               all the future commands loaded via the modules system.
#  nocommands   Alias for -@all.
#  ~<pattern>   Add a pattern of keys that can be mentioned as part of
#               commands. For instance ~* allows all the keys. The pattern
#               is a glob-style pattern like the one of KEYS.
#               It is possible to specify multiple patterns.
#  allkeys      Alias for ~*
#  resetkeys    Flush the list of allowed keys patterns.
#  &<pattern>   Add a glob-style pattern of Pub/Sub channels that can be
#               accessed by the user. It is possible to specify multiple channel
#               patterns.
#  allchannels  Alias for &*
#  resetchannels   Flush the list of allowed channel patterns.
#  ><password>  Add this password to the list of valid password for the user.
#               For example >mypass will add "mypass" to the list.
#               This directive clears the "nopass" flag (see later).
#  <<password>  Remove this password from the list of valid passwords.
#  nopass       All the set passwords of the user are removed, and the user
#               is flagged as requiring no password: it means that every
#               password will work against this user. If this directive is
#               used for the default user, every new connection will be
#               immediately authenticated with the default user without
#               any explicit AUTH command required. Note that the "resetpass"
#               directive will clear this condition.
#  resetpass    Flush the list of allowed passwords. Moreover removes the
#               "nopass" status. After "resetpass" the user has no associated
#               passwords and there is no way to authenticate without adding
#               some password (or setting it as "nopass" later).
#  reset        Performs the following actions: resetpass, resetkeys, off,
#               -@all. The user returns to the same state it has immediately after its creation.
#

# ACL规则可以按任何顺序指定：例如，您可以从密码开始，然后是标志或密钥模式。但是请注意，加法和减法规则将根据顺序改变含义。
# 例如，请参见以下示例：
#   user alice on +@all -DEBUG ~* >somepassword
#
# 这将允许“alice”使用除DEBUG命令之外的所有命令，因为+@all将所有命令添加到alice可以使用的命令集中，并且随后删除了DEBUG。但是，如果我们颠倒两个ACL规则的顺序，结果将不同：
#   user alice on -DEBUG +@all ~* >somepassword
# the Redis web site at https://redis.io/topics/acl

# ACL LOG: 在下面定义ACL日志的最大条目长度
acllog-max-len 128

# Using an external ACL file, 不在该redis.conf配置acl而是在aclfile指定的路径。
aclfile /etc/redis/users.acl


#从Redis 6开始，“requirepass”只是新ACL系统之上的一个兼容层。选项效果将只是为默认用户设置密码。客户端仍将像往常一样使用AUTH<password>进行身份验证，或者如果遵循新协议，则更明确地使用AUTH default<password>进行身份验证（两者都可以工作。）
requirepass PasswordPassword

# 为了在升级Redis 6.0时确保向后兼容性，acl pubsub默认设置为“AllChannel”权限。
# 为所有现有用户设置了显式发布/订阅后，应取消对以下行的注释。
# acl-pubsub-default resetchannels

# Command renaming (DEPRECATED).可以在共享环境中更改危险命令的名称。
# 例如，CONFIG命令可能会被重命名为难以猜测的内容，因此它仍然可以用于内部使用工具，但不可用于普通客户端。
rename-command CONFIG b840fc02d524045429941cc15f59e41cb7be6c52
# rename-command CONFIG ""


################################### CLIENTS ####################################
# 设置同时连接的客户端的最大数量。- 性能优化参数
# 重要提示：使用Redis Cluster时，最大连接数也与群集总线共享：群集中的每个节点将使用两个连接，一个传入，另一个传出。
# 对于非常大的簇，相应地调整限制的大小非常重要。
# maxclients 10000

############################## MEMORY MANAGEMENT ################################

# 将内存使用限制设置为指定的字节数。
maxmemory <bytes>


# MAXMEMORY策略：当到达MAXMEMORY时，Redis将如何选择要删除的内容。您可以从以下行为中选择一种：
# volatile-lru -> Evict using approximated LRU, only keys with an expire set. (折中)
# allkeys-lru -> Evict any key using approximated LRU.
# volatile-lfu -> Evict using approximated LFU, only keys with an expire set.
# allkeys-lfu -> Evict any key using approximated LFU.
# volatile-random -> Remove a random key having an expire set.
# allkeys-random -> Remove a random key, any key.
# volatile-ttl -> Remove the key with the nearest expire time (minor TTL)
# noeviction -> Don't evict anything, just return an error on write operations. (缺省)
#
# LRU means Least Recently Used
# LFU means Least Frequently Used
#
# LRU、LFU和volatile ttl都是使用近似随机算法实现的。
# 注意：对于上述任何策略，当没有合适的键进行逐出时，Redis将在需要更多内存的写入操作上返回错误。这些命令通常用于创建新键、添加数据或修改现有键。
# 例如：SET、INCR、HSET、LPUSH、SUNIONSTORE、SORT（由于STORE参数），以及EXEC（如果事务包含任何需要内存的命令）。
maxmemory-policy volatile-lru

# 默认值为5会产生足够好的结果, 10非常接近真实的LRU但CPU成本更高。3更快但不是很准确。
maxmemory-samples 5

# 如果写入流量异常大，则可能需要增加此值。降低此值可能会降低延迟，但会降低逐出处理效率
# 0 = minimum latency, 10 = default, 100 = process without regard to latency
maxmemory-eviction-tenacity 10

# 从Redis 5开始，默认情况下，复制副本将忽略其maxmemory设置（除非在故障切换后升级为master或手动）。这意味着密钥的逐出将由主机处理，在主机端的密钥逐出时将DEL命令发送到复制副本。
# replica-ignore-maxmemory yes

# 内存、CPU和延迟之间的折衷：通常设置为“1”的过期“努力”增加到更大的值，直到值“10”。在其最大值时，系统将使用更多的CPU、更长的周期（技术上可能会引入更多的延迟），并将允许系统中仍然存在的已过期密钥更少。
# active-expire-effort 1


############################# LAZY FREEING ####################################

# Redis有两个用于删除关键点的原语。一个称为DEL，是对象的块删除。这意味着服务器停止处理新命令，以便以同步方式回收与对象关联的所有内存。如果删除的键与一个小对象关联，则执行DEL命令所需的时间非常短，与Redis中的大多数其他O（1）或O（log_N）命令相当。但是，如果密钥与包含数百万个元素的聚合值相关联，则服务器可以阻塞很长时间（甚至几秒钟）以完成操作。
# 出于上述原因，Redis还提供了非阻塞删除原语，如UNLINK（非阻塞DEL）和FLUSHDB命令的异步选项，以便在后台回收内存。这些命令在固定时间内执行。另一个线程将尽可能快地增量释放背景中的对象。
# FLUSHDB和FLUSHDB的DEL、UNLINK和ASYNC选项由用户控制。这取决于应用程序的设计，以了解何时使用其中一个是一个好主意。然而，作为其他操作的副作用，Redis服务器有时不得不删除密钥或刷新整个数据库。具体来说，Redis在以下场景中独立于用户调用删除对象：
#
# 1) On eviction, because of the maxmemory and maxmemory policy configurations,
#    in order to make room for new data, without going over the specified  memory limit.
# 2) Because of expire: when a key with an associated time to live (see the
#    EXPIRE command) must be deleted from memory.
# 3) Because of a side effect of a command that stores data on a key that may
#    already exist. For example the RENAME command may delete the old key
#    content when it is replaced with another one. Similarly SUNIONSTORE
#    or SORT with STORE option may delete existing keys. The SET command
#    itself removes any old content of the specified key in order to replace
#    it with the specified string.
# 4) During replication, when a replica performs a full resynchronization with
#    its master, the content of the whole database is removed in order to load the RDB file just transferred.
#
# 在上述所有情况下，默认是以阻塞方式删除对象，就像调用 DEL 一样。 但是，您可以使用以下配置指令专门配置每种情况，以便以非阻塞方式释放内存，就像调用 UNLINK 一样。
lazyfree-lazy-eviction no
lazyfree-lazy-expire no
lazyfree-lazy-server-del no
replica-lazy-flush no

# 也可以在用 UNLINK 调用替换用户代码 DEL 调用并不容易的情况下，使用以下配置指令修改 DEL 命令的默认行为以使其行为与 UNLINK 完全相同：
lazyfree-lazy-user-del no

# FLUSHDB、FLUSHALL 和 SCRIPT FLUSH 支持异步和同步删除，可以通过在命令中传递 [SYNC|ASYNC] 标志来控制。
# 当两个标志都没有传递时，该指令将用于确定是否应异步删除数据。
lazyfree-lazy-user-flush no

################################ THREADED I/O #################################
# Redis 主要是单线程的，但是也有某些线程操作，例如 UNLINK、慢速 I/O 访问和其他在侧线程上执行的操作。
# 现在还可以在不同的 I/O 线程中处理 Redis 客户端套接字读取和写入。 由于写入速度特别慢，Redis 用户通常使用流水线来加快每个内核的 Redis 性能，并生成多个实例以进行更大的扩展。 使用 I/O 线程可以轻松地将 Redis 加速两倍，而无需借助管道或实例分片。
# 默认情况下线程是禁用的，我们建议只在至少有 4 个或更多内核的机器上启用它，至少留下一个备用内核。使用超过 8 个线程不太可能有太大帮助。 我们还建议您仅在确实存在性能问题时才使用线程 I/O，因为 Redis 实例能够使用相当大比例的 CPU 时间，否则使用此功能没有意义。
#
# 因此，例如，如果您有四核盒子，请尝试使用 2 或 3 个 I/O 线程，如果您有 8 核，请尝试使用 6 线程。 为了启用 I/O 线程，请使用以下配置指令（性能优化参数）：
io-threads 4

# 将 io-threads 设置为 1 将像往常一样使用主线程。 当启用 I/O 线程时，我们只使用线程进行写入，即线程化 write(2) 系统调用并将客户端缓冲区传输到套接字，但是也可以使用以下配置指令启用读取线程和协议解析，将其设置为 yes：
io-threads-do-reads no
# 注意 1：此配置指令不能在运行时通过 CONFIG SET 更改。 Aso 此功能当前在 SSL 时不起作用
# 注意 2：如果你想使用 redis-benchmark 测试 Redis 加速，请确保你也在线程模式下运行基准测试本身，使用 --threads 选项来匹配 Redis 线程数，否则你将无法 注意改进。



############################ KERNEL OOM CONTROL ##############################
# 在 Linux 上，可以提示内核 OOM 杀手在内存不足时应该首先杀死哪些进程。
# 启用此功能使Redis 主动控制其所有进程的oom_score_adj 值，具体取决于它们的角色。默认分数将尝试在所有其他进程之前杀死后台子进程，并在主进程之前杀死副本。
# Redis supports three options:
# no:       Don't make changes to oom-score-adj (default).
# yes:      Alias to "relative" see below.
# absolute: Values in oom-score-adj-values are written as is to the kernel.
# relative: Values are used relative to the initial value of oom_score_adj when
#           the server starts and are then clamped to a range of -1000 to 1000.
#           Because typically the initial value is 0, they will often match the absolute values.
oom-score-adj no

oom-score-adj-values 0 200 800


#################### KERNEL transparent hugepage CONTROL ######################
# 当使用 oom-score-adj 时，该指令控制用于主、副本和后台子进程的特定值。 值范围为 -2000 到 2000（更高意味着更有可能被杀死）。
# 非特权进程（不是 root，并且没有 CAP_SYS_RESOURCE 功能）可以自由地增加它们的值，但不能将其降低到其初始设置以下。这意味着将 oom-score-adj 设置为“relative”并将 oom-score-adj-values 设置为正值将始终成功。

# 通常内核透明大页面控件默认设置为“madvise”或“never”（/sys/kernel/mm/transparent_hugepage/enabled），在这种情况下，此配置无效。 在设置为“始终”的系统上，redis 将尝试专门为 redis 进程禁用它，以避免专门针对 fork(2) 和 CoW 的延迟问题。
# 如果出于某种原因您更喜欢保持启用状态，您可以将此配置设置为“no”，并将内核全局设置为“always”。
disable-thp yes

############################## APPEND ONLY MODE ###############################
# 默认情况下，Redis 异步转储磁盘上的数据集。 这种模式在很多应用中已经足够好了，但是 Redis 进程的问题或断电可能会导致几分钟的写入丢失（取决于配置的保存点）。
# appendonly 是另一种持久性模式，可提供更好的持久性。 例如，使用默认的数据 fsync 策略（见后面的配置文件）Redis 可能会在服务器断电等戏剧性事件中丢失一秒钟的写入，或者如果 Redis 进程本身发生问题，则会丢失一次写入，但是 操作系统仍然正常运行。
# AOF 和 RDB 持久化可以同时启用，没有问题。 如果在启动时启用了 AOF，Redis 将加载 AOF，即具有更好持久性保证的文件。
# AOF 持久化参考地址: https://redis.io/topics/persistence
# 指定是否在每次更新操作后进行日志记录，Redis在默认情况下是异步的把数据写入磁盘
appendonly NO
# 指定更新日志文件名，默认为appendonly.aof
appendfilename "appendonly.aof"

# 指定更新日志条件，共有3个可选值：
#   no：表示等操作系统进行数据缓存同步到磁盘（快）
#   always：表示每次更新操作后手动调用fsync()将数据写到磁盘（慢，安全）
#   everysec：表示每秒同步一次（折中，默认值）
# More details please check the following article: http://antirez.com/post/redis-persistence-demystified.html
appendfsync everysec


# 当 AOF fsync 策略设置为 always 或 everysec，并且后台保存过程（后台保存或 AOF 日志后台重写）正在对磁盘执行大量 I/O 时，在某些 Linux 配置中，Redis 可能会在磁盘上阻塞太长时间 fsync() 调用。 请注意，目前没有解决此问题的方法，因为即使在不同的线程中执行 fsync 也会阻塞我们的同步 write(2) 调用。 为了缓解这个问题，可以使用以下选项来防止在 BGSAVE 或 BGREWRITEAOF 正在进行时在主进程中调用 fsync()。
# 这意味着当另一个子进程正在保存时，Redis 的持久性与“appendfsync none”相同。 实际上，这意味着在最坏的情况下（使用默认的 Linux 设置）可能会丢失多达 30 秒的日志。

# 如果您有延迟问题，请将其设为“是”。 否则，将其保留为“否”，从耐用性的角度来看，这是最安全的选择。
no-appendfsync-on-rewrite no

# 自动重写附加文件。 当 AOF 日志大小增长指定百分比时，Redis 能够自动重写日志文件，隐式调用BGREWRITEAOF。
# 这是它的工作原理：Redis 会记住最近一次重写后的 AOF 文件的大小（如果重启后没有发生过重写，则使用启动时的 AOF 大小）。
#此基础尺寸与当前尺寸进行比较。 如果当前大小大于指定的百分比，则触发重写。 此外，您还需要为要重写的 AOF 文件指定最小大小，这对于避免重写 AOF 文件（即使达到百分比增加但仍然很小）很有用。
# 指定百分比为零以禁用自动 AOF 重写功能。
auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb

# 在Redis启动过程中，当AOF数据重新加载回内存时，可能会发现AOF文件在最后被截断。
# 如果aof-load-truncated 设置为yes，则加载一个截断的AOF 文件，Redis 服务器开始发出日志通知用户该事件。否则，如果该选项设置为 no，服务器将因错误而中止并拒绝启动。当该选项设置为 no 时，用户需要在重新启动服务器之前使用“redis-check-aof”实用程序修复 AOF 文件。
# 注意，如果中间会发现AOF文件损坏，服务器还是会报错退出
aof-load-truncated yes

# 重写 AOF 文件时，Redis 能够在 AOF 文件中使用 RDB 前导码，以实现更快的重写和恢复。 当这个选项打开时，重写的 AOF 文件由两个不同的节组成：
#   [RDB file][AOF tail]
# 加载时，Redis识别到AOF文件以“REDIS”字符串开头，加载带前缀的RDB文件，然后继续加载AOF尾部。
aof-use-rdb-preamble yes

################################ LUA SCRIPTING  ###############################

# Lua 脚本的最大执行时间（以毫秒为单位）。如果达到最大执行时间，Redis 将记录脚本在最大允许时间后仍在执行，并开始回复带有错误的查询。
# 如果将其设置为 0 或负值以无警告地无限执行。
lua-time-limit 5000

################################ REDIS CLUSTER  ###############################

# 普通Redis实例不能成为Redis集群的一部分； 只有作为集群节点启动的节点才可以。 要将 Redis 实例作为集群节点启动，启用集群支持
cluster-enabled yes

# 每个 Cluster 节点都有一个集群配置文件。 此文件不适合手动编辑。它由 Redis 节点创建和更新。
# 每个 Redis Cluster 节点都需要不同的集群配置文件。 确保在同一系统中运行的实例没有重叠的集群配置文件名。
cluster-config-file nodes-6379.conf


# 集群节点超时是节点必须不可达的毫秒数才能被视为处于故障状态。 大多数其他内部时间限制是节点超时的倍数。
cluster-node-timeout 15000

# 如果数据看起来太旧，故障主节点的副本将避免启动故障转移。
# 为了获得最大可用性，可以将 cluster-replica-validity-factor 设置为 0 值，这意味着副本将始终尝试对主服务器进行故障转移，无论它们上次与主服务器交互的时间如何。 （但是，他们总是会尝试应用与其偏移等级成正比的延迟）。
# 零是唯一能够保证当所有分区恢复时集群将始终能够继续运行的值。
cluster-replica-validity-factor 10

# 默认值为 1（副本仅在其主服务器至少保留一个副本时才会迁移）。 要禁用迁移，只需将其设置为一个非常大的值或将 cluster-allow-replica-migration 设置为“no”。 可以设置 0 值，但仅用于调试并且在生产中存在危险。
cluster-migration-barrier 1


# 关闭此选项允许使用较少的自动集群配置。 它既禁止迁移到孤立的 master，也禁止从变空的 master 迁移。
# 默认为“是”（允许自动迁移）。
cluster-allow-replica-migration yes


# 默认情况下，如果 Redis 集群节点检测到至少有一个哈希槽未被覆盖（没有可用的节点为其提供服务），它们将停止接受查询。 这样，如果集群部分关闭（例如不再覆盖一定范围的哈希槽），所有集群最终都将变得不可用。 一旦所有插槽再次被覆盖，它就会自动返回可用。
# 但是，有时您希望正在工作的集群子集继续接受对仍被覆盖的键空间部分的查询。为此只需将 cluster-require-full-coverage 选项设置为 no。默认关闭即可(兼容集群容错)
cluster-require-full-coverage no


# 此选项设置为 yes 时，可防止副本在主失败期间尝试故障转移其主。 但是，如果被迫这样做，副本仍然可以执行手动故障转移。
# 这在不同的场景中很有用，尤其是在多个数据中心运营的情况下，如果不是在整个 DC 故障的情况下，我们希望永远不要提升一侧。
cluster-replica-no-failover no


# 此选项设置为 yes 时，允许节点在集群处于关闭状态时为读取流量提供服务，只要它相信它拥有插槽。 这对两种情况很有用。 第一种情况是当应用程序在节点故障或网络分区期间不需要数据一致性时。 一个例子是缓存，只要节点有数据，它就应该能够为它提供服务。
cluster-allow-reads-when-down no


########################## CLUSTER DOCKER/NAT support  ########################

# 在某些部署中，Redis Cluster 节点地址发现失败，因为地址是 NAT-ted 或因为端口被转发（典型情况是 Docker 和其他容器）。
#
# 为了让Redis Cluster在这样的环境中工作，需要一个静态配置，每个节点都知道自己的公共地址。 以下四个选项用于此范围，分别是：
# * cluster-announce-ip
# * cluster-announce-port
# * cluster-announce-tls-port
# * cluster-announce-bus-port

# Example:
# cluster-announce-ip 10.1.1.5
# cluster-announce-tls-port 6379
# cluster-announce-port 0
# cluster-announce-bus-port 6380

################################## SLOW LOG ###################################

# Redis 慢日志是一种记录超过指定执行时间的查询的系统。 执行时间不包括与客户端交谈、发送回复等 I/O 操作，而只是实际执行命令所需的时间（这是命令执行的唯一阶段，线程被阻塞，可以在此期间不处理其他请求）。
# 以下时间以微秒表示，所以1000000相当于一秒。 请注意，负数会禁用慢速日志，而零值会强制记录每个命令。
slowlog-log-slower-than 10000

# slow日志最大长度现在，您可以使用 SLOWLOG RESET 回收慢日志使用的内存。
slowlog-max-len 128

################################ LATENCY MONITOR ##############################
# Redis 延迟监控子系统在运行时对不同的操作进行采样，以收集与 Redis 实例可能的延迟来源相关的数据，通过 LATENCY 命令，用户可以使用这些信息来打印图形和获取报告。
# 系统仅记录在等于或大于通过latency-monitor-threshold 配置指令指定的毫秒数内执行的操作。 当其值设置为零时，延迟监视器关闭。
# 默认情况下延迟监控是禁用的，因为如果您没有延迟问题，它通常不需要，并且收集数据对性能有影响，虽然很小，但可以在大负载下进行测量。 如果需要，可以在运行时使用命令“CONFIG SET latency-monitor-threshold <milliseconds>”轻松启用延迟监控。
latency-monitor-threshold 0

############################# EVENT NOTIFICATION ##############################
# Redis 可以将密钥空间中发生的事件通知 Pub/Sub 客户端。documented at https://redis.io/topics/notifications
# 例如，如果启用了键空间事件通知，并且客户端对存储在数据库 0 中的键“foo”执行 DEL 操作，则将通过 Pub/Sub 发布两条消息：in the Database 0, two messages will be published via Pub/Sub:
# 可以在一组类中选择 Redis 将通知的事件，每个类都由一个字符标识：
# PUBLISH __keyspace@0__:foo del
# PUBLISH __keyevent@0__:del foo
#  K     Keyspace events, published with __keyspace@<db>__ prefix.
#  E     Keyevent events, published with __keyevent@<db>__ prefix.
#  g     Generic commands (non-type specific) like DEL, EXPIRE, RENAME, ...
#  $     String commands
#  l     List commands
#  s     Set commands
#  h     Hash commands
#  z     Sorted set commands
#  x     Expired events (events generated every time a key expires)
#  e     Evicted events (events generated when a key is evicted for maxmemory)
#  t     Stream commands
#  d     Module key type events
#  m     Key-miss events (Note: It is not included in the 'A' class)
#  A     Alias for g$lshzxetd, so that the "AKE" string means all the events
#        (Except key-miss events which are excluded from 'A' due to their
#         unique nature).

# "notify-keyspace-events" 将一个由零个或多个字符组成的字符串作为参数。 空字符串表示禁用通知。
# 示例1：启用列表和通用事件，从事件名称的角度来看，使用：
notify-keyspace-events Elg
# 示例2：获取订阅频道名称__keyevent@0__:expired 的过期密钥流使用：
notify-keyspace-events Ex

# 默认情况下所有通知都是禁用的，因为大多数用户不需要此功能并且该功能有一些开销。请注意，如果您没有指定 K 或 E 中的至少一个，则不会传递任何事件。
notify-keyspace-events ""

############################### GOPHER SERVER #################################
# Redis contains an implementation of the Gopher protocol, as specified in the RFC 1436 (https://www.ietf.org/rfc/rfc1436.txt).
# Gopher 协议在 90 年代后期非常流行。它是 Web 的替代方案，服务器端和客户端的实现都非常简单，Redis 服务器只有 100 行代码来实现这种支持。
# 你现在用 Gopher 做什么？
# 好吧，Gopher 从未真正消亡，最近有一项运动，以便让 Gopher 由纯文本文档组成的更具层次性的内容复活。 有些人想要一个更简单的互联网，其他人则认为主流互联网受到了太多控制，为想要呼吸新鲜空气的人们创造一个替代空间是很酷的。
#
# --- HOW IT WORKS? ---
# Redis Gopher 支持使用Redis 的内联协议，特别是两种无论如何都是非法的内联请求：空请求或任何以“/”开头的请求（没有Redis 命令以这样的斜杠开头）。 正常的 RESP2/RESP3 请求完全脱离了 Gopher 协议实现的路径，并且也照常提供。
#
# 如果您在启用 Gopher 时打开与 Redis 的连接并向其发送一个类似“/foo”的字符串，如果有一个名为“/foo”的键，它将通过 Gopher 协议提供服务。
#
# 为了创建一个真正的 Gopher “洞”（Gopher 谈话中 Gopher 站点的名称），您可能需要一个如下所示的脚本
# https://github.com/antirez/gopher2redis
#
# --- SECURITY WARNING ---
# If you plan to put Redis on the internet in a publicly accessible address
# to server Gopher pages MAKE SURE TO SET A PASSWORD to the instance.
# Once a password is set:
#   1. The Gopher server (when enabled, not by default) will still servecontent via Gopher.
#   2. However other commands cannot be called before the client willauthenticate.
#
# 注意：当启用 'io-threads-do-reads' 时不支持 Gopher，并且应该使用'requirepass'选项来保护你的实例。
# gopher-enabled no

############################### ADVANCED CONFIG ###############################
# 哈希在条目数较少且最大条目不超过给定阈值时使用内存高效的数据结构进行编码。 可以使用以下指令配置这些阈值
hash-max-ziplist-entries 512
hash-max-ziplist-value 64

# 列表也以特殊方式编码，以节省大量空间。 每个内部列表节点允许的条目数可以指定为固定的最大大小或最大元素数。 对于固定的最大大小，使用 -5 到 -1，意思是：
# -5: max size: 64 Kb  <-- not recommended for normal workloads
# -4: max size: 32 Kb  <-- not recommended
# -3: max size: 16 Kb  <-- probably not recommended
# -2: max size: 8 Kb   <-- good
# -1: max size: 4 Kb   <-- good
# 正数意味着每个列表节点存储最多 _exactly_ 的元素数。 性能最高的选项通常是 -2（8 Kb 大小）或 -1（4 Kb 大小），但如果您的用例是独一无二的，请根据需要调整设置。
list-max-ziplist-size -2

# 列表也可以被压缩。压缩深度是从列表的*每一*侧到*排除*压缩的quicklist ziplist节点的数量。 对于快速推送/弹出操作，列表的头部和尾部总是未压缩的。
# 0: disable all list compression
# 1: depth 1 means "don't start compressing until after 1 node into the list,
#    going from either the head or tail"
#    So: [head]->node->node->...->node->[tail]
#    [head], [tail] will always be uncompressed; inner nodes will compress.
# 2: [head]->[next]->node->node->...->node->[prev]->[tail]
#    2 here means: don't compress head or head->next or tail->prev or tail,
#    but compress all nodes between them.
# 3: [head]->[next]->[next]->node->node->...->node->[prev]->[prev]->[tail]
# etc.
list-compress-depth 0

# 仅在一种情况下，集合具有特殊编码：当集合仅由恰好是 64 位有符号整数范围内基数为 10 的整数组成的字符串时。 以下配置设置设置了集合大小的限制，以便使用这种特殊的内存节省编码。
set-max-intset-entries 512

# 与哈希和列表类似，排序集也经过特殊编码以节省大量空间。 仅当排序集的长度和元素低于以下限制时才使用此编码：
zset-max-ziplist-entries 128
zset-max-ziplist-value 64


# HyperLogLog 稀疏表示字节限制，该限制包括 16 字节的标头。 当使用稀疏表示的 HyperLogLog 超过此限制时，它会转换为密集表示。 一个大于 16000 的值是完全没有用的，因为在那个时候密集表示的内存效率更高。
建议值为 ~ 3000，以便在不减慢太多 PFADD 的情况下获得空间高效编码的好处，稀疏编码为 O(N)。 当 CPU 不是问题但空间是，并且数据集由许多基数在 0 - 15000 范围内的 HyperLogLog 组成时，该值可以提高到 ~ 10000。
hll-sparse-max-bytes 3000

# 流宏节点最大大小/项目。 流数据结构是一个大节点的基数树，其中对多个项目进行编码。 使用此配置，可以配置单个节点的大小（以字节为单位），以及在附加新流条目时切换到新节点之前它可能包含的最大项目数。 如果以下任何设置设置为零，则该限制将被忽略，因此例如可以通过将 max-bytes 设置为 0 并将 max-entries 设置为所需值来仅设置最大条目限制。
stream-node-max-bytes 4096
stream-node-max-entries 100

# 指定是否激活重置哈希，默认为开启
# 主动重新散列每 100 毫秒的 CPU 时间使用 1 毫秒，以帮助重新散列主 Redis 哈希表（将顶级键映射到值的表）。 Redis 使用的哈希表实现（参见 dict.c）执行延迟重新哈希：您遇到重新哈希的哈希表的操作越多，执行的重新哈希“步骤”就越多，因此如果服务器空闲，重新哈希永远不会完成 哈希表使用了更多的内存。
# 默认是每秒使用这个毫秒 10 次，以便主动重新哈希主字典，在可能的情况下释放内存。
#  如果不确定：如果您有硬延迟要求并且在您的环境中Redis可以不时回复2毫秒延迟的查询并不是一件好事，请使用“activerehashing no”。
# 如果您没有这样的硬性要求，但希望尽可能快地释放内存，请使用“activerehashing yes”。
activerehashing yes

# 客户端输出缓冲区限制可用于强制断开由于某种原因没有足够快地从服务器读取数据的客户端（一个常见的原因是 Pub/Sub 客户端不能像发布者一样快地消费消息） 他们）。
# 可以为三类不同的客户端设置不同的限制：
#    normal   -> normal clients including MONITOR clients
#    replica  -> replica clients
#    pubsub   -> clients subscribed to at least one pubsub channel or pattern
# Syntax:
#  client-output-buffer-limit <class> <hard limit> <soft limit> <soft seconds>
#
# 一旦达到硬限制，或者如果达到软限制并保持达到指定的秒数（连续），客户端将立即断开连接。
# 例如，如果硬限制是 32 兆字节，软限制是 16 兆字节 / 10 秒，如果输出缓冲区的大小达到 32 兆字节，客户端将立即断开连接，但如果客户端达到 16 兆字节，也会断开连接 并持续突破限制 10 秒。
# 默认情况下，普通客户端不受限制，因为它们不会在不询问的情况下（以推送方式）接收数据，而是在请求之后，因此只有异步客户端可能会创建请求数据比读取数据更快的场景,相反发布订阅和副本客户端有一个默认限制，因为订阅者和副本以推送方式接收数据,硬限制或软限制都可以通过将它们设置为零来禁用。
client-output-buffer-limit normal 0 0 0
client-output-buffer-limit replica 256mb 64mb 60
client-output-buffer-limit pubsub 32mb 8mb 60


# 客户端查询缓冲区累积新命令。 默认情况下，它们被限制为固定数量，以避免协议不同步（例如由于客户端中的错误）将导致查询缓冲区中未绑定的内存使用。 但是，如果您有非常特殊的需求，
# 例如我们巨大的 multi/exec 请求或类似需求，您可以在此处配置它。
client-query-buffer-limit 1gb

# 在Redis协议中，批量请求，即表示单个字符串的元素，通常限制为512 mb。 但是您可以在此处更改此限制，但必须为 1mb 或更大
proto-max-bulk-len 512mb


# Redis 调用一个内部函数来执行许多后台任务，例如超时关闭客户端的连接，清除从未请求过的过期键，等等。
# 并非所有任务都以相同的频率执行，但Redis会根据指定的“hz”值检查要执行的任务。
# 默认情况下，"hz" 设置为 10。提高该值会在 Redis 空闲时使用更多的 CPU，但同时会使 Redis 在有许多键同时过期时更灵敏，并且可以更精确地处理超时。
# 范围在 1 到 500 之间，但是超过 100 的值通常不是一个好主意。 大多数用户应该使用默认值 10，并且仅在需要极低延迟的环境中将其提高到 100。
hz 10

# 通常，拥有一个与连接的客户端数量成正比的 HZ 值是有用的。 例如，为了避免为每个后台任务调用处理太多客户端以避免延迟峰值，这很有用。
# 由于默认的默认 HZ 值保守地设置为 10，Redis 提供并默认启用使用自适应 HZ 值的能力，当有很多连接的客户端时，该值会暂时提高。
# 启用动态HZ时，将使用实际配置的HZ作为基线，但一旦连接更多客户端，将根据需要实际使用配置的HZ值的倍数。 通过这种方式，空闲实例将使用很少的 CPU 时间，而繁忙实例将响应更快。
dynamic-hz yes

# 当子进程重写 AOF 文件时，如果启用以下选项，文件将每生成 32 MB 数据进行 fsync-ed。 这对于以增量方式将文件提交到磁盘并避免大的延迟峰值很有用。
aof-rewrite-incremental-fsync yes

# 当redis保存RDB文件时，如果启用以下选项，文件将每生成32 MB数据进行fsync-ed。 这对于以增量方式将文件提交到磁盘并避免大的延迟峰值很有用。
rdb-save-incremental-fsync yes

# Redis LFU 驱逐（见 maxmemory 设置）可以调整。 然而，最好从默认设置开始，只有在研究如何提高性能以及按键 LFU 如何随时间变化后才更改它们，这可以通过 OBJECT FREQ 命令进行检查。
#
# Redis LFU 实现中有两个可调参数：计数器对数因子和计数器衰减时间。 在更改这两个参数之前，了解这两个参数的含义很重要。
#
# LFU 计数器每个键只有 8 位，最大值为 255，因此重新使用具有对数行为的概率增量。 给定旧计数器的值，当访问一个键时，计数器以这种方式递增：
# 1. A random number R between 0 and 1 is extracted.
# 2. A probability P is calculated as 1/(old_value*lfu_log_factor+1).
# 3. The counter is incremented only if R < P.
#
# 默认的lfu-log-factor 是10，这是一个频率计数器如何随着不同的对数因子的访问次数的变化而变化的表格：
#
# +--------+------------+------------+------------+------------+------------+
# | factor | 100 hits   | 1000 hits  | 100K hits  | 1M hits    | 10M hits   |
# +--------+------------+------------+------------+------------+------------+
# | 0      | 104        | 255        | 255        | 255        | 255        |
# +--------+------------+------------+------------+------------+------------+
# | 1      | 18         | 49         | 255        | 255        | 255        |
# +--------+------------+------------+------------+------------+------------+
# | 10     | 10         | 18         | 142        | 255        | 255        |
# +--------+------------+------------+------------+------------+------------+
# | 100    | 8          | 11         | 49         | 143        | 255        |
# +--------+------------+------------+------------+------------+------------+
#
# 注意：上表是通过运行以下命令获得的：
#   redis-benchmark -n 1000000 incr foo
#   redis-cli object freq foo

# 注意 2：计数器初始值为 5，以便让新对象有机会累积命中。
# 计数器衰减时间是必须经过的时间，以分钟为单位，以便将关键计数器除以 2（或者如果它的值小于 <= 10，则递减）。
 lfu-log-factor 10
# lfu-decay-time 的默认值是 1。一个特殊的值 0 表示每次碰巧被扫描时都会衰减计数器。
lfu-decay-time 1


########################### ACTIVE DEFRAGMENTATION #######################
#
# What is active defragmentation?
# 什么是主动碎片整理？
# -------------------------------
# 主动（在线）碎片整理允许 Redis 服务器压缩内存中小数据分配和释放之间留下的空间，从而允许回收内存。
# 分片是每个分配器（但幸运的是 Jemalloc 除外）和某些工作负载发生的自然过程。 通常需要重新启动服务器以降低碎片，或者至少清除所有数据并再次创建它。 然而，由于 Oran Agra 为 Redis 4.0 实现的这个功能，这个过程可以在运行时以“热”的方式发生，而服务器正在运行。

# 基本上当碎片超过一定程度时（请参阅下面的配置选项）Redis 将开始通过利用某些特定的 Jemalloc 功能在连续内存区域中创建值的新副本（以便了解分配是否导致碎片并在 一个更好的地方），同时，将释放数据的旧副本。 这个过程，对所有键递增重复，将导致碎片回落到正常值。
# -------------------------------
# 重要事项：
# 1. This feature is disabled by default, and only works if you compiled Redis  to use the copy of Jemalloc we ship with the source code of Redis. This is the default with Linux builds.
#
# 2. You never need to enable this feature if you don't have fragmentation issues.
#
# 3. Once you experience fragmentation, you can enable this feature when needed with the command "CONFIG SET activedefrag yes".
# 配置参数能够微调碎片整理过程的行为如果您不确定它们的含义，最好保持默认值不变。

# 已启用活动碎片整理
activedefrag no

# 启动活动碎片整理的最小碎片浪费量
active-defrag-ignore-bytes 100mb

# 启动活动碎片整理的最小碎片百分比
active-defrag-threshold-lower 10

# 我们使用最大努力的最大碎片百分比
active-defrag-threshold-upper 100

# 在 CPU 百分比中进行碎片整理的最小工作量，在达到下限阈值时使用
active-defrag-cycle-min 1

# CPU百分比碎片整理的最大努力，在达到上限时使用
active-defrag-cycle-max 25

# 将从主字典扫描中处理的最大集合/散列集/列表字段数
active-defrag-max-scan-fields 1000

# 用于清除的 Jemalloc 后台线程将默认启用
jemalloc-bg-thread yes

# 可以将 Redis 的不同线程和进程固定到系统中的特定 CPU，以最大限度地提高服务器的性能。在同一主机上运行的多个 Redis 实例将被固定到不同的 CPU。
#通常您可以使用“taskset”命令执行此操作，但是也可以直接通过 Redis 配置来执行此操作，无论是在 Linux 还是 FreeBSD 中。
#可以pin server/IO线程，bio线程，aof重写子进程，bgsave子进程。 指定 cpu 列表的语法与 taskset 命令相同：

# 设置 redis server/io 线程为 cpu 亲和度 0,2,4,6:
server_cpulist 0-7:2

# 将 bio 线程设置为 cpu 亲和度 1,3：
bio_cpulist 1,3

# 设置aof重写子进程为cpu亲和度8、9、10、11：
aof_rewrite_cpulist 8-11

# 设置 bgsave 子进程为 cpu 亲和度 1,10,11
bgsave_cpulist 1,10-11

# 在某些情况下，如果redis检测到系统处于不良状态，它会发出警告甚至拒绝启动，可以通过设置以下配置来抑制这些警告，该配置采用空格分隔的警告列表来抑制
ignore-warnings ARM64-COW-BUG
```

**Redis 服务配常用参数：**

```
# Basic - 基础参数
daemonize yes
supervised auto
databases 16
bind 127.0.0.1 10.20.172.108
port 6379
dir /database/redis
pidfile /var/run/redis.pid

# Security - 安全参数
requirepass www.weiyigeek.top
rename-command CONFIG b840fc02d524045429941cc15f59e41cb7be6c52
rename-command FLUSHDB b840fc02d524045429941cc15f59e41cb7be6c53
rename-command FLUSHALL b840fc02d524045429941cc15f59e41cb7be6c54
rename-command EVAL b840fc02d524045429941cc15f59e41cb7be6c55
rename-command DEBUG b840fc02d524045429941cc15f59e41cb7be6c56
rename-command SHUTDOWN b840fc02d524045429941cc15f59e41cb7be6c57

# Persistent(RDB/AOF) - 持久化配置
save 900 1
save 300 10
save 60 10000
# - RDB 方式
rdbchecksum yes
rdbcompression yes
dbfilename dump-master.rdb
stop-writes-on-bgsave-error no
# - AOF 方式
appendonly yes
appendfilename "appendonly.aof"
appendfsync everysec
no-appendfsync-on-rewrite yes
auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb
aof-rewrite-incremental-fsync yes
aof-load-truncated yes

# Memory Limit and Policy - 内存使用限制与内存策略
maxmemory 2gb
maxmemory-policy volatile-lru

# Master-Slave  - 主从模式关键参数
slaveof 127.0.0.1 6379
masterauth redispass
slave-read-only yes
slave-priority 100
slave-serve-stale-data yes
repl-disable-tcp-nodelay no

# Cluster - 集群模式关键参数
cluster-enabled yes
cluster-config-file  /data/nodes.conf
cluster-node-timeout 5000
# - 当负责一个插槽的主库下线且没有相应的从库进行故障恢复时集群仍然可用
cluster-require-full-coverage no
# - 只有当一个主节点至少拥有其他给定数量个处于正常工作中的从节点的时候，才会分配从节点给集群中孤立的主节点
cluster-migration-barrier 1

# Connection related - 连接超时参数
timeout 300
tcp-keepalive 60

# Lua execute - 脚本执行查询时间
lua-time-limit 5000

# Slow Query - Slow 日志查询与长度
slowlog-log-slower-than 10000
slowlog-max-len 128

# Cache and Buffer - 缓存限制
# client-output-buffer-limit normal 0 0 0
# client-output-buffer-limit slave 256mb 64mb 60
# client-output-buffer-limit pubsub 32mb 8mb 60

# Logging - 日志相关配置
loglevel warning
logfile "/database/redis/logs/redis-6379.log"
```

![](https://i0.hdslb.com/bfs/article/db75225feabec8d8b64ee7d3c7165cd639554cbc.png@progressive.webp)

> 如果你觉得这个专栏还不错的，请给这篇专栏点个赞、投个币、收个藏，这将对我有很大帮助！  
>         文章来源: https://weiyigeek.top  
>         WeiyiGeek Blog - 为了能到远方，脚下的每一步都不能少。  

![](https://i0.hdslb.com/bfs/article/02db465212d3c374a43c60fa2625cc1caeaab796.png@progressive.webp)

欢迎各位志同道合的朋友一起学习交流，如文章有误请在下方留下您宝贵的经验知识，个人邮箱地址【master#weiyigeek.top】