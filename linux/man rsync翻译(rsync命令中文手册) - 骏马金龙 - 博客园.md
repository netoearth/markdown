**本文为命令rsync的man文档翻译，几乎所有的选项都翻译了，另外关于筛选规则部分只翻译了一部分。由于原文很多地方都比较啰嗦，所以译文中有些内容可能容易让国人疑惑，所以我个人在某些地方加上了注释。若有错误之处，盼请指出。**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
rsync(1)                                                                                           rsync(1)

名称
       rsync - 一个快速、多功能的远程(和本地)文件拷贝工具

摘要
       Local:  rsync [OPTION...] SRC... [DEST]

       Access via remote shell:
         Pull: rsync [OPTION...] [USER@]HOST:SRC... [DEST]
         Push: rsync [OPTION...] SRC... [USER@]HOST:DEST

       Access via rsync daemon:
         Pull: rsync [OPTION...] [USER@]HOST::SRC... [DEST]
               rsync [OPTION...] rsync://[USER@]HOST[:PORT]/SRC... [DEST]
         Push: rsync [OPTION...] SRC... [USER@]HOST::DEST
               rsync [OPTION...] SRC... rsync://[USER@]HOST[:PORT]/DEST


       当仅有一个SRC或DEST参数时将列出源文件列表而不是复制文件。
       
描述
       Rsync是一个快速且功能非常丰富的文件拷贝工具。它可以在本地和远程之间通过shell或rsync
       服务互相拷贝文件。它提供了大量的选项来控制它各方面功能的行为，且在指定待拷贝文件方
       面非常有弹性。它以其增量拷贝算法而出名，只拷贝源和目标不同的文件部分，因此减少网络
       间要传输的数据。Rsync每天都被广泛用于做备份、镜像和当作升级版拷贝命令。

       Rsync使用"quick check"算法(默认)决定文件是否需要被传输，它会找出大小或最后修改时间
       (mtime)不同的文件。当"quick check"算法表明了文件不需要被更新时，任何其他保留属性(译
       者注:除大小和最后修改时间外的属性)都将直接在目标文件上修改。

       rsync的其他特性包括：

       o      支持拷贝链接文件、设备文件、所有权(即所有者和所属组)、属组以及权限

       o      支持类似于GNU tar命令的exclude和exclude-from选项

       o      支持CVS排除模式以忽略相同的文件(译者注：CVS是一种版本控制系统)

       o      可以使用任意透明的远程shell(remote shell)，包括ssh或rsh

       o      不要求超级管理员权限

       o      以pipeline管道模型传输文件以便最小化降低成本

       o      支持匿名或可身份认证的rsync daemon模式(做镜像的理想方式)


一般特性
       Rsync在本地或远程主机之间拷贝文件(但不支持两个远程之间互相拷贝)。

       rsync有两种不同的方式联系远程主机：使用远程shell程序作为传输方式(如ssh或rsh)或直接通
       过TCP联系rsync守护进程。当命令行中指定的源或目标主机后使用了单个冒号(:)时将使用远程
       shell传输模式。当在命令行中指定的源或目标主机后使用双冒号(::)或使用了rsync://这种URL
       时将表示使用TCP联系rsync守护进程，但rsync://方式有一个例外，请参见下文"通过远程SHELL
       连接使用RSYNC-DAEMON特性"段落的内容。

       但有一个特殊情况，如果只给定了源地址没有给定目标地址，则将以类似于"ls -l"的格式输出文
       件列表。

       若给定的源地址和目标地址都不是远程地址，则在本机进行拷贝(见选项--list-only)。

       Rsync命令中，本地端总是扮演"client"角色，远程端总是扮演"server"角色。不要混淆"server"
       和rsync daemon，rsync daemon一定是一个"server"，但是"server"可能是一个rsync daemon也可
       能是远程shell派生出来的进程。

安装
       请阅读README文件来查看安装说明。

       当安装完成后，你可以通过远程shell(也可以通过rsync daemon协议)让rsync与任意你能访问的主
       机进行交流。对于远程传输，现代rsync使用ssh与其他主机进行交流，但是可以配置其他默认的远
       程shell，如rsh或remsh。 

       你也可以通过命令行的"-e"选项或设置RSYNC_RSH环境变量来指定你想要使用的远程shell。

       rsync必须同时装在源主机和目标主机上。

用法
       rsync的使用方法和rcp一样。你必须指定源地址和目标地址，其中一个可能是远程地址。
       
       也许解释语法最好的方式是通过几个示例：

              rsync -t *.c foo:src/

       这将会把当前目录下所有能匹配*.c的文件传输到主机foo上的src目录下。如果远程主机上已经存在
       某些同名文件，rsync的远程更新(rsync-update)协议将会更新哪些有差异的文件。更详细的内容见
       技术报告。

              rsync -avz foo:src/bar /data/tmp

       这将会以递归方式把远程主机foo上的src/bar目录下的所有文件传输到本地主机的/data/tmp目录下。
       这些文件以归档(archive)模式传输，它保证在传输过程中保留符号链接、设备文件、属性、权限、
       所有者、所属组等。另外，在传输过程中会使用压缩功能以减少要传输的数据体积。

              rsync -avz foo:src/bar/ /data/tmp

       使用尾随斜线(/)改变了原本的行为，它避免了在目标地址创建一个额外的目录层次。带有尾随斜线
       时，你可以理解为"拷贝目录的内容"而不是拷贝"拷贝目录名"(译者注：即拷贝目录本身)，但是这两
       种情况都会将目录中包含的文件传输到目标目录下。换句话说，下面两条命令都以相同的方式进行拷
       贝，包括/dest/foo的属性设置。

              rsync -av /src/foo /dest
              rsync -av /src/foo/ /dest/foo

       还需要注意的是，拷贝主机或引用模块的默认目录不需要尾随斜线。例如，下面的命令都拷贝远程
       (默认)目录的内容到本地的"/dest"。

              rsync -av host: /dest
              rsync -av host::module /dest

       你也可以仅使用rsync的仅本地(local-only)模式，此模式下的源地址和目标地址名称中都不需要冒号
       (:)。它行为类似于cp命令的升级版。

       最后，你可以通过移除模块名参数部分的方式列出rsync daemon中所有可用模块。

              rsync somehost.mydomain.com::

       更详细信息见下面段落。

高级用法
       请求从远程主机传输多个文件的语法是通过指定和第一个远程地址格式相同的多个远程地址参数，或
       者可以省略主机名部分。例如，下面的命令都会正常工作：
       (译者注：远程地址中省略主机名时，将取前一个远程地址的主机名作为它的主机名)

              rsync -av host:file1 :file2 host:file{3,4} /dest/
              rsync -av host::modname/file{1,2} host::modname/file3 /dest/
              rsync -av host::modname/file1 ::modname/file{3,4}

       老版本的rsync需要在SRC部分使用引号和空格，如下示例：

              rsync -av host:'dir1/file1 dir2/file2' /dest
              rsync host::'modname/dir1/file1 modname/dir2/file2' /dest

       这种方式在后续rsync版本中将继续(默认)有效，但是它没有第一种方式简便。

       如果你要传输一个文件名中包含了空白字符的文件，可以使用"--protect-args"("-s")选项，也可以
       使用转义符将空白字符转义。例如：

              rsync -av host:'file\ name\ with\ spaces' /dest


连接RSYNC DAEMON
       rsync也可以不使用远程shell作为传输方式。这情况看下，将直接连接远程RSYNC守护进程，一般使
       用的是TCP的873端口。(显然，这要求远程的RSYNC守护进程必须是已运行的，见下文"启动RSYNC服务
       以接受连接请求")

       这种方式的rsync使用方式和远程shell方式一样，除了：

       o      需要使用双冒号"::"分隔主机名和路径，或者使用rsync://的URL格式。

       o      "path"部分的第一个词语是一个模块名(译者注：如hostname::modname/file)。

       o      远程RSYNC守护进程可能会输出你连接它的日期时间。

       o      如果没有指定远程rsync服务的路径名，将列出rsync服务主机上可访问的路径。

       o      如果没有指定本地目标地址，将列出远程rsync服务主机上指定的文件。

       o      必须不能指定"--rsh"("-e")选项。


       以下是拷贝远程模块名为"src"中的所有文件示例：

           rsync -av host::src /dest

       远程daemon上的某些模块可能需要身份验证。如果是这样，在连接时将会被询问输入密码。如果想要
       避免被询问，可以通过设置环境变量RSYNC_PASSWORD的值为你要使用的密码，或者使用选项
       "--password-file"。这非常适用于脚本中。
       
       警告：在某些系统上，环境变量是对所有用户可见的，此时建议使用"--password-file"选项。

       你可以通过web代理(web proxy)的方式与rsync daemon建立连接，只需设置环境变量RSYNC_PROXY的
       值为hostname:port指向你的web代理。但要注意，web代理的配置必须得支持与873端口的代理连接。

       你还可以使用代理程序与rsync daemon建立连接，只需设置环境变量RSYNC_CONNECT_PROG的值为你想
       要运行的命令来代替建立套接字连接。环境变量的值中可能会包含"%H"，它代表rsync命令中所指定
       的主机名(因此如果想在值中包含一个"%"字符，需要使用"%%")。例如：

         export RSYNC_CONNECT_PROG='ssh proxyhost nc %H 873'
         rsync -av targethost1::module/src/ /dest/
         rsync -av rsync:://targethost2/module/src/ /dest/

       上面的命令中使用ssh在proxyhost上运行了nc命令，它将会转发所有数据到目标主机(%H)的873端口。


通过远程SHELL连接使用RSYNC-DAEMON特性
       某些时候使用rsync daemon的各种特性(如可命名的模块)比允许任意套接字连接到系统上(除了真正
       需要使用远程shell的访问)更方便。rsync支持使用远程shell连接到主机上，它会派生出一个单用
       途(single-use)的"daemon"服务用于读取远程用户家目录下的配置文件。如果你想要加密
       daemon-sytle传输的数据，但由于daemon是被远程用户启动的，你无法通过这样的daemon使用像
       chroot这样的功能，也无法修改uid，这时使用远程shell是比较好的。(另一个加密daemon传输的方
       式是，使用ssh建立本地端口到远程主机的隧道，并且在远程主机上配置一个普通的rsync daemon只
       允许从"localhost"发起连接(译者注：其实就是配置ssh的端口转发))

       从用户角度来看，使用远程shell连接使用rsync daemon与连接普通rsync daemon的命令行语法几乎
       相同，唯一例外的是必须在命令行使用--rsh=COMMAND选项设置远程shell程序。(设置RSYNC_RSH环境
       变量不会打开此功能)例如：

           rsync -av --rsh=ssh host::module /dest

       如果需要指定不同用户的远程shell，一定要记住，host前缀"user@"设置的是rsync上的用户(即用于
       需要用户认证的模块)。这意味着必须在ssh命令中使用"-l user"选项来指定远程shell，下面的例子
       中使用了"--rsh"的短格式选项"-e"：

           rsync -av -e "ssh -l ssh-user" rsync-user@host::module /dest

       "ssh-user"将在ssh上生效，而"rsync-user"将用于认证使用"module"模块。(译者注：对于连接目标
       非daemon时，"ssh -l user"和"user@"作用是一样的)
       
       (译者注：远程shell连接使用rsync daemon，和真正的守护进程rsync daemon是不同的，后者是配置
       好后永久监听在后台提供服务，而远程shell使用rsync daemon则是一种临时性单用途的daemon进程，
       虽然也会读取配置文件，但它是由远程shell进程fork出来的子进程，此次连接结束后，此daemon进
       程会自动消逝)

启动RSYNC服务以接受连接请求
       要连接到一个rsync daemon，远程系统上的rsync daemon必须已经运行(或者像inetd一样，已经配置
       了当特殊端口上有连接时会派生出rsync daemon)。关于如何启动一个能处理从套接字进来的连接的
       daemon的完整信息，请看rsyncd.conf(5)，这是rsync daemon的配置文件，它包含如何运行daemon的
       非常详细的信息(包括独立模式(stand-alone)和inetd格式的配置)

       如果你使用的是某种远程shell传输方式，则没有手动启动rsync daemon的必要。

传输过程中的排序
       Rsync总是会在内部传输列表中对指定的文件名进行排序。这将会使得相同目录名的文件被合并在一
       起进行传输，这样一来，去除重复文件名就比较容易，但这可能会让一些人产生疑惑：文件传输时
       的顺序和命令行中给的顺序不一致。

       如果想要让那个某个特殊的文件比其他文件先传输，可以将它们分隔到不同的rsync命令上，或考虑
       使用"--delay-updates"选项(它不会影响传输时的排序，但会使得最后的文件更新(file-updating)
       阶段更迅速)。

示例
       以下是一些我使用rsync的示例。

       要备份包含了大量word文档和邮件文件夹的家目录，使用一个任务计划(cron job)运行：

              rsync -Cavz . arvidsjaur:backup

       每个晚上都将通过PPP连接到主机"arvidsjaur"上的backup目录。

       要同步samba源码树，使用下面的Makefile targets：

           get:
                   rsync -avuzb --exclude '*~' samba:samba/ .
           put:
                   rsync -Cavuzb . samba:samba/
           sync: get put

       这可以让我和连接另一端的CVS目录保持同步。然后再在远程主机上做一些CVS操作，这节省了我大
       量时间，因为远程CVS协议的效率并不高。

       我使用下面的命令在我的"old"和"new"ftp站点之间做一个镜像：

       rsync -az -e ssh --delete ~ftp/pub/samba nimbus:"~ftp/pub/tridge"

       由于设置了cron计划，每隔几小时它就登录一次。

选项汇总
       下面是rsync中可用的命令汇总，关于选项的完整描述，请看后文。

        -v, --verbose               increase verbosity
        -q, --quiet                 suppress non-error messages
            --no-motd               suppress daemon-mode MOTD (see caveat)
        -c, --checksum              skip based on checksum, not mod-time & size
        -a, --archive               archive mode; equals -rlptgoD (no -H,-A,-X)
            --no-OPTION             turn off an implied OPTION (e.g. --no-D)
        -r, --recursive             recurse into directories
        -R, --relative              use relative path names
            --no-implied-dirs       don't send implied dirs with --relative
        -b, --backup                make backups (see --suffix & --backup-dir)
            --backup-dir=DIR        make backups into hierarchy based in DIR
            --suffix=SUFFIX         backup suffix (default ~ w/o --backup-dir)
        -u, --update                skip files that are newer on the receiver
            --inplace               update destination files in-place
            --append                append data onto shorter files
            --append-verify         --append w/old data in file checksum
        -d, --dirs                  transfer directories without recursing
        -l, --links                 copy symlinks as symlinks
        -L, --copy-links            transform symlink into referent file/dir
            --copy-unsafe-links     only "unsafe" symlinks are transformed
            --safe-links            ignore symlinks that point outside the tree
        -k, --copy-dirlinks         transform symlink to dir into referent dir
        -K, --keep-dirlinks         treat symlinked dir on receiver as dir
        -H, --hard-links            preserve hard links
        -p, --perms                 preserve permissions
        -E, --executability         preserve executability
            --chmod=CHMOD           affect file and/or directory permissions
        -A, --acls                  preserve ACLs (implies -p)
        -X, --xattrs                preserve extended attributes
        -o, --owner                 preserve owner (super-user only)
        -g, --group                 preserve group
            --devices               preserve device files (super-user only)
            --copy-devices          copy device contents as regular file
            --specials              preserve special files
        -D                          same as --devices --specials
        -t, --times                 preserve modification times
        -O, --omit-dir-times        omit directories from --times
            --super                 receiver attempts super-user activities
            --fake-super            store/recover privileged attrs using xattrs
        -S, --sparse                handle sparse files efficiently
        -n, --dry-run               perform a trial run with no changes made
        -W, --whole-file            copy files whole (w/o delta-xfer algorithm)
        -x, --one-file-system       don't cross filesystem boundaries
        -B, --block-size=SIZE       force a fixed checksum block-size
        -e, --rsh=COMMAND           specify the remote shell to use
            --rsync-path=PROGRAM    specify the rsync to run on remote machine
            --existing              skip creating new files on receiver
            --ignore-existing       skip updating files that exist on receiver
            --remove-source-files   sender removes synchronized files (non-dir)
            --del                   an alias for --delete-during
            --delete                delete extraneous files from dest dirs
            --delete-before         receiver deletes before xfer, not during
            --delete-during         receiver deletes during the transfer
            --delete-delay          find deletions during, delete after
            --delete-after          receiver deletes after transfer, not during
            --delete-excluded       also delete excluded files from dest dirs
            --ignore-errors         delete even if there are I/O errors
            --force                 force deletion of dirs even if not empty
            --max-delete=NUM        don't delete more than NUM files
            --max-size=SIZE         don't transfer any file larger than SIZE
            --min-size=SIZE         don't transfer any file smaller than SIZE
            --partial               keep partially transferred files
            --partial-dir=DIR       put a partially transferred file into DIR
            --delay-updates         put all updated files into place at end
        -m, --prune-empty-dirs      prune empty directory chains from file-list
            --numeric-ids           don't map uid/gid values by user/group name
            --timeout=SECONDS       set I/O timeout in seconds
            --contimeout=SECONDS    set daemon connection timeout in seconds
        -I, --ignore-times          don't skip files that match size and time
            --size-only             skip files that match in size
            --modify-window=NUM     compare mod-times with reduced accuracy
        -T, --temp-dir=DIR          create temporary files in directory DIR
        -y, --fuzzy                 find similar file for basis if no dest file
            --compare-dest=DIR      also compare received files relative to DIR
            --copy-dest=DIR         ... and include copies of unchanged files
            --link-dest=DIR         hardlink to files in DIR when unchanged
        -z, --compress              compress file data during the transfer
            --compress-level=NUM    explicitly set compression level
            --skip-compress=LIST    skip compressing files with suffix in LIST
        -C, --cvs-exclude           auto-ignore files in the same way CVS does
        -f, --filter=RULE           add a file-filtering RULE
        -F                          same as --filter='dir-merge /.rsync-filter'
                                    repeated: --filter='- .rsync-filter'
            --exclude=PATTERN       exclude files matching PATTERN
            --exclude-from=FILE     read exclude patterns from FILE
            --include=PATTERN       don't exclude files matching PATTERN
            --include-from=FILE     read include patterns from FILE
            --files-from=FILE       read list of source-file names from FILE
        -0, --from0                 all *from/filter files are delimited by 0s
        -s, --protect-args          no space-splitting; wildcard chars only
            --address=ADDRESS       bind address for outgoing socket to daemon
            --port=PORT             specify double-colon alternate port number
            --sockopts=OPTIONS      specify custom TCP options
            --blocking-io           use blocking I/O for the remote shell
            --stats                 give some file-transfer stats
        -8, --8-bit-output          leave high-bit chars unescaped in output
        -h, --human-readable        output numbers in a human-readable format
            --progress              show progress during transfer
        -P                          same as --partial --progress
        -i, --itemize-changes       output a change-summary for all updates
            --out-format=FORMAT     output updates using the specified FORMAT
            --log-file=FILE         log what we're doing to the specified FILE
            --log-file-format=FMT   log updates using the specified FMT
            --password-file=FILE    read daemon-access password from FILE
            --list-only             list the files instead of copying them
            --bwlimit=KBPS          limit I/O bandwidth; KBytes per second
            --write-batch=FILE      write a batched update to FILE
            --only-write-batch=FILE like --write-batch but w/o updating dest
            --read-batch=FILE       read a batched update from FILE
            --protocol=NUM          force an older protocol version to be used
            --iconv=CONVERT_SPEC    request charset conversion of filenames
            --checksum-seed=NUM     set block/file checksum seed (advanced)
        -4, --ipv4                  prefer IPv4
        -6, --ipv6                  prefer IPv6
            --version               print version number
       (-h) --help                  show this help (see below for -h comment)


       Rsync以daemon方式运行时，还可以接受以下选项：

            --daemon                run as an rsync daemon
            --address=ADDRESS       bind to the specified address
            --bwlimit=KBPS          limit I/O bandwidth; KBytes per second
            --config=FILE           specify alternate rsyncd.conf file
            --no-detach             do not detach from the parent
            --port=PORT             listen on alternate port number
            --log-file=FILE         override the "log file" setting
            --log-file-format=FMT   override the "log format" setting
            --sockopts=OPTIONS      specify custom TCP options
        -v, --verbose               increase verbosity
        -4, --ipv4                  prefer IPv4
        -6, --ipv6                  prefer IPv6
        -h, --help                  show this help (if used after --daemon)


选项
       Rsync可以接受长格式选项和段格式选项，下面列出了所有可使用的选项。如果以多种方式指定同一
       个选项，则使用逗号分隔。某些选项只有长格式，没有短格式。如果选项要带参数，参数只能列在长
       格式选项后。如果要指定选项的参数，可以使用"--option=param"的格式，也可以使用空白字符替换
       "="。某些参数可能需要使用引号包围，避免被shell命令行解析。需要记住，文件名中的前导波浪号
       (~)是被shell替换的，因此"--option=~/foo"将不会从波浪号进入家目录(若要如此，将"="移除)


       --help 输出所有选项简短格式的帮助信息并退出。为了兼容老版本，当只使用一个"-h"选项时也会
       输出这些帮助信息。

       --version
              输出rsync的版本号并退出。

       -v, --verbose
              该选项增加了传输过程中的大量信息。默认情况下，rsync以静默(silent)模式工作。单个
              "-v"将给出哪些文件正在被传输的信息，还会在传输结束时给出一个简要总结信息。两个
              "-v"选项(-vv)将给出哪些文件被忽略，并且在传输结束时给出更详细的信息。超过两个
              "-v"选项一般只在调试过程中使用。

              需要注意的是，传输过程中输出的文件名是被"--out-format=%n%L"处理过的，它表示仅显
              示文件名，如果是软链接文件，则还显示它指向谁。当使用一个"-v"选项时，将不会告诉你
              文件的属性改变了。如果明确要列出改变了属性的文件列表(既可以使用"--itemize-change"
              也可以在"--out-format"中增加"%i")，(客户端)输出结果中将总是会增加所有已改变的条目
              清单。更详细信息见"--out-format"选项。

       -q, --quiet
              该选项将减少大量传输过程中的信息，但无法禁止从远程主机产生的信息。该选项适用于
              cron任务计划中。

       --no-motd
              此选项会影响客户端在守护程序传输开始时输出的信息。它会禁止motd信息，但是也会影响
              daemon对"rsync host::"请求显示模块列表的回应信息，因此如果想要请求daemon的模块列
              表，应该忽略该选项。

       -I, --ignore-times
              正常情况下，rsync会忽略文件大小相同且最后修改时间戳相同的文件，该选项关闭"quick 
              check"行为，使得所有的文件都会被更新。
              (译者注：即从增量拷贝变成全量拷贝)

       --size-only
              该选项将修改rsync查找需要被传输文件的"quick check"算法，默认该算法会找出所有大小
              或最后修改时间戳改变文件并传输，使用该选项将仅查找大小改变的文件并传输。在使用了
              某些镜像备份但没有保留精确时间戳的情况下，使用带有该选项的rsync将非常有帮助。

       --modify-window
              当比较时间戳时，rsync会把两者相差不超过该选项所指定的值时认为是相同的时间戳。正常
              情况下该选项的值为0(精确匹配)，但在某些情况下，将此选项的值设置为非0将非常有用。
              特别是与Windows的FAT文件系统(此文件系统有2秒范围内的精确度差异)传输数据时，设置
              "--modify-window=1"将非常有用(允许文件时间戳有1秒的差异)。

       -c, --checksum
              此选项改变了rsync检查文件改变和决定是否要传输的方式。不使用该选项，rsync使用
              "quick check"(默认的)检查发送端和接收端两边文件的大小和最后一次修改时间是否改变。
              使用该选项，将对每个匹配了大小的文件比较128位的校验码。生成校验码意味着两端都会消
              耗大量的磁盘I/O以读取传输队列中文件的数据内容(传输队列早于读取文件数据，即先quick
              check，再生成和比较校验码)，因此该选项会大幅度降低效率。

              发送端生成校验码的时刻是做文件系统扫描以生成可获取文件列表时。接收端生成校验码的
              时刻是在扫描哪些文件发生改变时，它会对那些与发送端文件大小相同的文件生成校验码。
              所以，该选项的结果是：只传输校验码改变或文件大小改变(意味着校验码也改变)的文件。

              注意，rsync默认总是在文件传输完成后再生成全部文件(whole-file)的校验码，并验证传输
              完成的文件是否正确重组。但是使用该选项，它隐含了在传输前做"该文件是否需要更新？"
              的检查，使得文件传输结束后不会自动去验证它们重组的正确性。

              从协议30版本开始(对应的rsync版本从3.0.0开始)使用的校验码是MD5格式的，更老的协议版
              本使用的校验码是MD4格式的。
              
              (译者注：即基于checksum来判断文件是否要同步，而不是基于quick check算法。在两个地
              方会计算checksum：sender端发送文件列表时，接收端的generator判断文件是否要传输时)

       -a, --archive
              该选项等价于"-rlptgoD"选项的组合。它表示使用归档模式并保留几乎所有属性(明显遗漏了
              "-H"选项)。上面的等价选项的唯一例外是指定了"--files-from"选项，它使得"-r"选项被强
              制忽略。

              注意，"-a"选项不保留硬链接属性，因为查找多个硬链接文件是非常昂贵的。若要保留硬链
              接属性，必须与"-a"分开独立使用"-H"选项。

       --no-OPTION
              在选项前面加上前缀"no-"表示关闭该选项的隐含选项功能。不是所有选项都能使用"no-"前
              缀：只有那些隐含了其他选项的选项(如--no-D,--no-perms)或者不同环境下有不同默认值的
              选项(如--no-whole-file,--no-blocking-io,--no-dirs)。"no-"后面即可以接短格式选项，
              也可以接长格式选项(如--no-R等价于--no-relative)

              例如：你想使用"-a"选项但不想使用它的隐含选项"-o"(--owner)，即让"-a"等价于
              "-rlptgD"，可以指定"-a --no-o"(或-a --no-owner)。

              选项的顺序是非常重要的：如果指定"--no-r -a"，则最终会启用"-r"选项，与之相反的是
              "-a --no-r"。同样需要注意的是"--files-from"的副作用，它的位置顺序不重要，因为它影
              响了某些选项的默认行为，并轻微改变了"-a"的意义(见"--files-from"选项以获取更详细说
              明)。

       -r, --recursive
              此选项告诉rsync以递归模式拷贝目录。参见"--dirs"(-d)。

              从rsync 3.0.0开始，现在使用的递归算法是一种增量扫描，比以前少占用很多内存，并且在
              最初扫描完一些目录后就开始进行数据传输。增量扫描仅影响递归算法，不会改变非递归的
              传输类型。同样，只有传输两端的版本都高于3.0.0才会如此。

              某些选项要求rsync知道完整的文件列表，所以它们会禁用增量递归模式。这些选项包括：
              --delete-before,--delete-after,--prune-empty-dirs和--delay-updates。正因为如此，
              如果两端rsync版本都高于3.0.0时，指定"--delete"时的默认删除模式变为
              "--delete-during"(可以使用"--del"或"--del-during"来精确指定删除模式)。同样，选择
              使用"--delete-delay"选项比"--delete-after"会更好。 
              
              增量递归模式可以使用"--no-inc-recursive"(--no-i-r)选项来禁用。

       -R, --relative
              表示使用相对路径。这意味着会将命令行中指定的全路径名而非路径最尾部的文件名发送给
              服务端。当要一次性发送多个不同目录时该选项非常有用。例如，如果使用下面的命令：

                 rsync -av /foo/bar/baz.c remote:/tmp/

              这将会在远程主机上的/tmp/目录下创建一个baz.c文件。如果使用下面的命令：

                 rsync -avR /foo/bar/baz.c remote:/tmp/

              将会在远程主机的/tmp/目录下递归创建foo/bar/baz.c，它保留了命令行中指定的全路径。
              这些额外的路径元素被称为"隐含目录"(如上例中的foo和foo/bar)。

              从rsync 3.0.0开始，rsync总是会发送这些隐含目录作为文件列表中的真实的目录，即使发
              送端的某个路径元素是一个软链接。这使得拷贝全路径文件时，不用担心因为路径中包含了
              软链接而可能出现的非预期的问题(译者注：即链接追踪)。如果要复制服务端符号链接，请
              通过其路径来复制符号链接，并通过实际路径来复制器真实对象。如果rsync版本较老，可
              能需要使用"--no-implied-dirs"选项。

              也可以对所指定的路径限制发送时作为隐含目录的路径信息。从rsync 2.6.7版本开始，
              rsync可以在源路径插入一个点"."，就像这样：

                 rsync -avR /foo/./bar/baz.c remote:/tmp/

              这将会在远程主机上创建/tmp/bar/baz.c。(注意点后面必须跟上斜线，因此"/foo/."将不会
              被缩写)对于更老版本的rsync，可能需要改变目录来限制源路径。例如，推送文件时：

                 (cd /foo; rsync -avR bar/baz.c remote:/tmp/)


              (注意，括号将把两个命令放入子shell中执行，因此cd改变目录不会影响未来的命令)如果使
              用老版本的rsync拉取文件，使用以下惯用格式(但只适用于非daemon的传输)：

                 rsync -avR --rsync-path="cd /foo; rsync" \
                     remote:bar/baz.c /tmp/


       --no-implied-dirs
              该选项影响"--relative"选项的默认行为。当指定该选项时，在传输时不会包含源文件的隐
              含目录。这意味着目标主机上对应路径元素会被保留不变(如果它们存在的话)，并且缺少的
              隐含目录会以默认属性方式被创建。甚至允许目标主机上隐含路径元素和源地址的属性有非
              常大的区别，例如在接收端某文件可能是某个目录的符号链接。

              另外，当rsync要传输的文件为"path/foo/file"时，如果使用"--relative"选项，则目录
              "path"和"path/foo"是隐含目录。如果在目标主机上"path/foo"是一个指向"bar"文件的符号
              连接，接收端的rsync会删除"path/foo"，并重建它为一个目录，然后将接收到的文件放入此
              新目录中。使用"--no-implied-dirs"选项，接收端使用已存在的路径元素更新
              "path/foo/file"，意味着最终会在"path/bar"中创建file文件。另一个实现连接保留功能的
              方法是使用"--keep-dirlinks"选项(也将会使得后续的传输从符号链接定位到目录中)。

              当使用早于3.0.0版本的rsync拉取文件时，如果发送端的路径中包含了符号链接，并且希望
              隐含目录能以普通目录方式被传输时，可能需要使
              用该选项。

       -b, --backup
              当使用该选项时，如果目标路径中已存在需要被传输或需要被删除的文件时将重命名该文件。
              可以使用"--backup-dir"选项控制备份文件的保存路径，使用"--suffix"选项控制备份时追
              加在原文件名后的后缀。
              
              注意，如果不指定"--backup-dir"选项：(1)将隐含"--omit-dir-times"选项(2)如果
              "--delete"选项同时影响该文件，rsync将在排除规则的尾部添加一个起"保护"作用的筛选规
              则(例如，-f "P *~")，这会阻止之前备份的文件被删除。注意，如果你使用了自己定义的筛
              选规则，你可能需要手动插入你的exclude/include规则，并且保证其优先级较高防止被其他
              规则先匹配上而导致失效。

       --backup-dir=DIR
              结合"--backup"选项一起使用，这表示rsync在远端将存储所有备份文件到指定的目录下。这
              可用于增量备份。可以使用"--suffix"选项额外指定备份后缀(否则备份到指定目录的文件将
              使用原文件名)。

              需要注意如果拟制定了一个相对路径，备份目录将会相对到目标目录，因此你可能真正想要
              指定的是一个绝对路径或以"../"开头的路径。如果接收端是rsync daemon，备份目录将无法
              超出模块的路径层次结构，因此请特别注意不要将其删除或复制到其中。

       --suffix=SUFFIX
              该选项可自定义"--backup"(-b)选项的备份文件名后缀，如果没有指定"--backup-dir"选项，
              则默认后缀为"~"，否则后缀为空字符串。

       -u, --update
              该选项将强制忽略在目标路径下已存在且修改时间比源文件更新的文件。(如果已存在的目标
              文件的修改时间和源文件相同，则只在文件大小不同时才会更新)

              注意该选项不会影响软链接或其他特殊文件的拷贝机制。而且，不管两端文件中的数据是否
              相同，考虑发送端和接收端不同的文件格式对于更新来说也是非常重要的。换句话说，如果
              源文件是一个目录，而目标已存在的同名文件却是一个普通文件，则rsync会直接忽略它们的
              时间戳。

              该选项是一种传输规则，不是排除规则，因此不会影响进入file-lists的文件，也因此不会影
              响删除。它仅会限制接收端请求传输的文件。

       --inplace
              该选项会改变当数据需要更新时，rsync传输文件的方式。默认情况下，rysnc会创建一个文件
              的新副本，当此文件传输完成时会将此副本移动到指定的路径下。使用此选项后，将直接把更
              新部分的数据写入到目标文件中。
               
              (译者注：此选项的拷贝机制可以理解为类似于drbd基于块的拷贝机制)
              
              这会带来以下几种影响：

              o      硬链接不会被破坏。这意味着通过其他硬链接文件可以直接访问到新数据。更进一步
                     说，尝试拷贝不同源文件到多重链接的目标文件时，将导致目标数据像"拔河"一样，
                     来来回回地变化。

              o      使用中的二进制程序不会被更新(操作系统会防止这样的事情发生，二进制程序自身也
                     会在尝试数据交换时崩溃)。

              o      在传输过程中，文件的数据会进入不一致状态，并且如果传输被中断或者更新失败时，
                     文件将继续不一致。

              o      rsync无法将数据写入一个无法被更新的文件中。虽然超级管理员可以更新任意文件，
                     但普通用户需要获取文件的写权限才能打开文件并向其中成功写入数据。

              o      如果目标文件中的数据在它被复制到某个位置之前被覆盖，则rsync的增量拷贝效率会
                     降低。如果使用了"--backup"则不会出现这样的问题，因为rsync足够智能，它会使用
                     备份文件作为传输的基准文件。

              警告：不能使用该选项对那些正被其他用户访问的文件，因此在选择使用此选项进行拷贝时需
              要小心谨慎。

              该选项适用于对于那些基于数据块(block-based)改变或向文件尾部追加了数据的大文件，也
              适用于那些安装在磁盘上而非网络上的系统。它也能有效帮助保持写时复制(copy-on-write)
              文件系统的快照。
              
              该选项隐含了"--partial"选项(因为传输中断不会删除文件)，但和"--partial-dir"以及
              "--delay-udpates"选项冲突。

       --append
              该选项使得rsync以追加数据到文件尾部的方式来更新文件，它会假定接收端上已存在的文件
              和发送端文件的前段数据是一致的。如果接收端上文件的大小等于或大于发送端文件的大小，
              则此文件会被忽略。该选项不会干涉不被传输文件的非内容属性(non-content，如权限，所有
              者等)，也不会影响对非普通文件(non-regular)的更新。隐含了"--inplace"选项，但是和
              "--sparse"选项不冲突(因为它总是扩充一个文件的长度)。
              
       --append-verify
              工作方式类似于"--append"选项，但是接收端已存在的数据在验证阶段会被包含在whole-file
              校验码中，如果最后验证阶段失败了，该文件会被传输(rsync将使用正常、非追加的
              "--inplace"模式重发文件)。

       -d, --dirs
              以不递归的方式拷贝目录本身，它不会拷贝目录中的文件。不像"--recursive"选项，只拷贝
              目录中的内容而不拷贝目录本身。除非目录名中使用了"."或者以斜线结尾(如".","dir/.",
              "dir/"等)。既不指定该选项，也不指定"--recursive"选项时，rsync将忽略所有遇到的目录
              (并会向输出这些影响信息)。如果同时指定了"--dirs"和"--recursive"选项，"--recursive"
              将优先生效。

              若未给定"--recursive"选项，"--files-from"或"--list-only"选项会隐含"--dirs"选项，此
              时在列表中能见到所有目录。要想关闭此功能，可以指定"--no-dirs"或"--no-d"选项。

              还有一个比较有用的向后兼容的选项："--old-dirs"(--old-d)。它告诉rsync使用
              "-r --exclude='/*/*'"仅列出目录而不递归。

       -l, --links
              当遇到符号链接时，将在目标路径重新创建符号链接。(译者注：即拷贝符号链接本身)

       -L, --copy-links
              使用该选项时，当遇到符号链接时将拷贝它所指向的目标而不是符号链接本身(译者加：但仅
              只是追踪了链接文件指向文件中的数据，文件名仍然是符号链接文件的文件名。举个例子，
              如果client端a文件-->b文件，则使用该选项拷贝a时，将在receiver端生成a文件，但a文件
              是一个普通文件，其中的数据来源是client端b文件的数据)。老版本的rsync使用该选项还会
              告诉接收端也追踪符号链接到其指向的目标中。在目前的rsync版本中，要实现这样的功能需
              要指定"--keep-dirlinks"(-K)选项。

       --copy-unsafe-links
              This tells rsync to copy the referent of symbolic links that point outside the 
              copied tree. Absolute symlinks are also treated like ordinary files, and so are 
              any symlinks in the source path itself when --relative is used. This option has 
              no additional effect if --copy-links was also specified.

       --safe-links
              This tells rsync to ignore any symbolic links which point outside the copied tree.
              All absolute symlinks are also ignored. Using this option in conjunction with 
              --relative may give unexpected results.

       -k, --copy-dirlinks
              该选项使得sender端将符号链接视为一个目录，就像它真的是一个目录一样。如果你不想让指
              向非目录的符号链接受到影响，可以使用该选项。

              Without this option, if the sending side has replaced a directory with a symlink 
              to a directory, the receiving side will delete anything that is in the way of the 
              new symlink, including a directory hierarchy (as long as --force or --delete is 
              in effect).

              See also --keep-dirlinks for an analogous option for the receiving side.

              --copy-dirlinks applies to all symlinks to directories in the source. If you want 
              to follow only a few specified  symlinks, a trick you can use is to pass them as 
              additional source args with a trailing slash, using --relative to make the paths 
              match up right.  For example:

              rsync -r --relative src/./ src/./follow-me/ dest/


              This works because rsync calls lstat(2) on the source arg as given, and the 
              trailing slash makes lstat(2) follow the symlink, giving rise to a directory in 
              the file-list which overrides the symlink found during the scan of "src/./".

       -K, --keep-dirlinks
              该选项使得receiver端将符号链接视为目录文件，就像它是真的目录一样，但只有它在sender
              端能匹配一个真实目录时才会如此。不使用该选项，receiver端的符号链接将被删除或替换为
              一个真实目录。
              
              例如，假设你要传输一个包含文件"file"的目录"foo"，但是在receiver端上的"foo"是一个指
              向"bar"目录的符号链接。如果不使用该选项，receiver端将删除符号链接"foo"，然后重建它
              为一个目录，然后接收"file"到此目录中。如果使用了该选项，receiver端将保留符号链接，
              然后将"file"存放到"bar"目录中去。
              (译注：上述示例的命令格式为"rsync -r foo user@host:/path"，其中path下有个名为foo
              的链接文件)
              
              需要注意一点：如果使用了"--keep-dirlinks"，你必须信任你所有拷贝中的链接文件。如果
              某个非信任用户要创建属于它自己的符号链接(指向某目录)，在下一次传输过程中，可能会使
              用真实目录替换掉符号链接并影响链接文件所指向目录中的文件内容。对于备份拷贝，你最好
              是用mount的bind功能而不是使用符号链接来改变接收端的目录层次。
              
              参见"--copy-dirlinks"选项，它是在sender端上类似的选项。

       -H, --hard-links
              This tells rsync to look for hard-linked files in the source and link together 
              the corresponding files on the destination.   Without this option, hard-linked 
              files in the source are treated as though they were separate files.

              This option does NOT necessarily ensure that the pattern of hard links on the 
              destination exactly matches that on the source. Cases in which the destination 
              may end up with extra hard links include the following:

              o      If the destination contains extraneous hard-links (more linking than what 
                     is present in the source file list), the  copying
                     algorithm will not break them  explicitly. However, if one or more of the 
                     paths have content differences, the normal file-update process will break 
                     those extra links (unless you are using the --inplace option).

              o      If you specify a --link-dest directory that contains hard links, the 
                     linking of the destination files against the --link-dest files can  
                     cause some paths in the destination to become linked together due to  
                     the --link-dest associations.


              Note that rsync can only detect hard links between files that are inside the 
              transfer set.  If rsync updates a file that has extra hard-link connections to 
              files outside the transfer, that linkage will be broken. If you are tempted to 
              use the --inplace  option to avoid this breakage, be very careful that you know 
              how your files are being updated so that you are certain that no unintended 
              changes happen due to lingering hard links (and see the --inplace option for 
              more caveats).

              If incremental recursion is  active (see  --recursive),  rsync may transfer a 
              missing hard-linked file before it finds  that  another link for that contents 
              exists elsewhere in the hierarchy.  This does not  affect  the accuracy of the 
              transfer (i.e. which files are hard-linked together), just its efficiency (i.e. 
              copying the data for a new, early copy of  a hard-linked file  that could have 
              been found  later in the transfer in another member of the hard-linked set of 
              files).  One way to avoid this inefficiency is to disable incremental recursion 
              using the --no-inc-recursive option.

       -p, --perms
              该选项告诉receiver端的rsync，要将目标文件的权限值设置为何源文件一样(即权限保留)。
              (参见"--chmod"选项以获取rsync修改sender端权限的方式)

              当没有使用该选项时，将以如下方式设置权限值：

              o      已存在的文件继续保留它们的原有权限，尽管"--executability"选项可能会改变文
                     件的执行权限。

              o      对于新文件，将从源文件中获取普通权限值，再配合receiver端文件所在目录的默认
                     ACL权限或umask值决定文件的最终权限，并且会禁用它们的特殊权限位，除非新的目
                     录文件从其父目录中继承了sgid权限。

              因此，当"--perms"和"--executability"选项都被禁用时，rsync的行为和其它文件拷贝工具
              的行为一样，例如cp、tar。

              总结以下：要设置目标文件(包括新文件和已存在的旧文件)的权限值为源文件的权限值，使用
              "--perms"选项。要设置新的目标文件默认权限，请确保"--perms"选项是关闭的，然后使用
              "--chmod=ugo=rwX"(这将保证启用所有非掩码位权限)。如果想以更简单的方式实现后一种情
              况，你可能需要为其定义一个popt别名，例如将下面的命令行放入文件~/.popt中(下面的命令
              中定义了"-Z"选项，并使用了"--no-g"使得目标文件的所属组使用目标目录的默认组)：

                 rsync alias -Z --no-p --no-g --chmod=ugo=rwX

              然后可以在命令行中使用新的选项，例如：

                 rsync -avZ src/ dest/

              (警告：请确保"-a"选项不是跟随在"-Z"后的，否则将重新启用上面已经定义的两个"--no-*"
              选项。)

       -E, --executability
              该选项使得rsync在未指定"--perms"选项时对普通文件保留文件的执行权限(或者不可执行权
              限)。普通文件上开启了"x"才认为有可执行权限。当目标文件已存在且和对应源文件的可执
              行权限值不一样时，rsync将采用如下方式修改权限：

              o      To make a file non-executable, rsync turns off all its ’x’ permissions.

              o      To make a file executable, rsync turns on each ’x’ permission that has a 
                     corresponding ’r’ permission enabled.

              如果指定了"--perms"选项，则该选项被忽略。

       -A, --acls
              使目标文件的ACL属性和源文件的ACL属性一致。该选项隐含了"--perms"选项。
              
       -X, --xattrs
              使目标文件的扩展属性和源文件的扩展属性保持一致。

       --chmod
              该选项使得rsync可以将目标文件的权限设定为此处所指定的权限值，让rsync以为这些指定
              的权限就是源文件的权限。也因此在未配合"--perms"一起使用时该选项无效。

              在chmod(1)的man文档中记录了普通的语法解析规则，你可以通过加上一个前缀"D"来指定该
              权限规则只对目录有效，或者加上前缀"F"指定该权限规则只对普通文件有效。例如，下面的
              例子保证了所有目录都标记了sgid权限，其它人对文件都不可写，所有者和所属组都可写，
              且所有人都有执行权限：

              --chmod=Dg+s,ug+w,Fo-w,+X

              可以指定多个使用逗号分隔的"--chomod"选项。

       -o, --owner
              该选项使得rsync将目标文件的所有者设置为和源文件一样(即保留所有者属性)，但要求接收
              端的rsync是以super user身份运行的(或指定了"--no-super"选项)。如果不指定该选项，目
              标文件的所有者将设置为调用rsync的用户身份(译者注：例如rsync /src name1@host:/path，
              则目标文件的所有者为name1)。

              默认情况下，目标文件的所有者名称由uid匹配而来，但在某些环境下，可能会保留使用uid。
              (详细信息见"--numeric-ids"选项)
              
              (译者注：例如源文件的所有者为name1，其uid=1000，那么将在目标主机上寻找uid=1000所
              对应的用户名，如果能找到则所有者设置为用户名，否则设置为uid=1000)

       -g, --group
              此选项的意义完全同"--owner"，所以不做对应翻译。

       --devices
              该选项使得rsync可以传输字符设备和块设备到目标主机上以重新创建这些设备。该选项要求
              接收端的rsync是以super user身份运行的(译者注：例如，root用户也算是super user，则
              rsync /devicename root@host:/path)，否则该选项失效。(见"--super"和"--fake-super"
              选项)

       --specials
              该选项使得rsync可以传输特殊文件，如命名套接字，命名管道等。

       -D     该选项等价于"--devices --specials"选项组合。

       -t, --times
              该选项告诉rsync将mtime随文件一起传输给receiver，使得目标文件的mtime和源文件一样。
              千万注意，如果不指定该选项，原本排除那些mtime相同的文件而获得的性能提升将不再生
              效；换句话说，如果没有在rsync命令行中使用"-t"或"-a"选项，将导致下一次传输以类似于
              "-I"的方式进行，即更新所有文件(尽管在文件没有真正发生更改的情况下，rsync的增量传
              输算法可以让更新效率很高，但最好还是使用"-t")

       -O, --omit-dir-times
              该选项告诉rsync，当保留mtime(见"--times")时，将忽略目录。如果receiver端正在通过
              NFS共享目录，使用"-O"是一个不错的选择。若指定了"--backup"但未指定"--backup-dir"，
              将隐含该选项。

       --super
              该选项告诉receiver端在进行某些操作时尝试使用super-user身份，尽管receiver端的rsync
              不是以super user身份运行的。这些操作包括：通过"--owner"保留文件所有者，通过
              "--groups"保留文件所属组(包括辅助组)，通过"--devices"选项拷贝设备文件。在receiver
              端未以super user身份调用rsync时，这些选项很有用。如果要关闭super user选项功能，则
              使用"--no-super"

       --fake-super
              如果启用了该选项，rsync将通过对附加在每个文件上的扩展属性(根据实际需要)的保存/恢复
              来模拟super user。扩展属性包括：文件的owner、group、文件的设备信息(设备文件和特殊
              文件被创建为空文本文件)以及所有特殊权限位(suid/sgid/sbit)。

              在不使用super user备份数据时但又想保存ACL属性时，该选项很有用。
              
              "--fake-super"选项默认只影响命令发起端，如果想要通过远程shell影响远程端时，可以指
              定rsync的路径：

                rsync -av --rsync-path="rsync --fake-super" /src/ host:/dest/

              由于本地拷贝时，两端都在本地主机上，该选项将会同时影响本地的sender端和receiver端
              的文件。如果想要避免这样的情况，需要通过指定"localhost"的地址方式来实现拷贝，或者
              可能也可以使用"lsh"远程shell来完成。

              该选项会被"--super"以及"--no-super"选项覆盖。

              其他信息可以参见rsyncd.conf文件中的"fake super"。

       -S, --sparse
              尝试以高效率的方式处理稀疏文件，使得它们在目标主机上占用更少的空间。该选项不能和
              "--inplace"选项一起使用，因为"--inplace"不能向稀疏模式的文件中覆盖数据。

       -n, --dry-run
              该选项使得rsync仅测试运行(并生成和真正运行时几乎一样的输出信息)。该选项常和"-v"、
              "--verbose"、"-i"、"--itemize-changes"选项一起使用，以便查看rsync在这些选项下是如
              何工作的。

              配合"--itemize-changes"时的输出结果应该要和真正运行的结果完全一致(除非人为故意欺
              骗rsync或系统调用失败)。如果输出结果不一致，则出现了bug。配合其他几个选项时，输出
              结果除了在某些方面外应该保持几乎一致。尤其是，dry run不会真的发送数据，因此
              "--progress"将的结果将很可能异常。             

       -W, --whole-file
              使用该选项将使得rsync不再使用增量传输算法，而是传输所有文件。如果源和目标主机之间
              的带宽高于磁盘的带宽(特别是"磁盘"是网络文件系统时)，则该选项比增量传输更有效。当
              源和目标都是本地时，该选项是默认的传输算法，但若受到write batch模式影响，则此算法
              不生效。

              (译者注：假设A主机和B主机之间的网络可以以1000MB/s的速度传输，而目标主机B上磁盘的
              带宽只有500MB/s，显然目标主机在文件重组时从basis file读取数据块的速度不如A发送给
              B快，所以在这种情况下，增量传输不如全量传输)

       -x, --one-file-system
              该选项告诉rsync不能跨文件系统递归(译者注：例如根目录下有mnt目录，mnt常用来做挂载
              点，则递归根目录时，不会递归到mnt里面)。该选项不会限制用户从多个文件系统指定拷贝
              项，仅只是限制rsync在每个目录下进行递归，同时也以类似的限制方式限制删除时receiver
              端递归。需要记住，使用mount命令的"bind"功能绑定了设备文件时，它也被认为是在同一个
              文件系统上。
              
              如果重复指定该选项，rsync将忽略client端所有的挂载点目录。否则，当遇到挂载点时将当
              作是空目录(这些空目录使用已挂载目录的属性，因为挂载点目录下的文件是无法访问的)。
              
              如果指定了"--copy-links"或"--copy-unsafe-links"选项使得rsync"瓦解"符号链接，则符
              号链接所指向的是另一个设备上的目录时将和挂载点一样对待。该选项不会影响指向非目录
              的符号链接。
              (译者注：翻译有点不标准，以下是原文)
              This tells rsync to avoid crossing a filesystem boundary when recursing. This does 
              not limit the user’s ability to specify items to copy from multiple filesystems, 
              just rsync’s recursion through the hierarchy of each  directory that  the  user 
              specified, and also the analogous recursion on the receiving side during deletion. 
              Also keep in mind that rsync treats a "bind" mount to the same device as being on 
              the same filesystem.

             If this option is repeated, rsync omits all mount-point directories from the copy. 
              Otherwise, it includes an empty directory at each mount-point it encounters 
              (using the attributes of the mounted directory because those of the underlying 
              mount-point directory are inaccessible).

              If rsync has been told to collapse symlinks (via --copy-links or 
              --copy-unsafe-links), a symlink to a directory on another device
              is treated like a mount-point.  Symlinks to non-directories are 
              unaffected by this option.

       --existing, --ignore-non-existing
              告诉rsync，如果目标主机上文件或文件所在目录还不存在，则不自动创建它们，即这些文件
              将不被传输。如果该选项和"--ignore-existing"选项一起使用，将不更新任何文件(如果你
              的目的是删除目标主机上的无关文件，这将非常有用)。
              
              (译者注：例如rsync --existing /etc/dhcp/* /tmp，由于/tmp下没有dhcp目录，所以dhcp
              目录和其中的文件都不会被传输到/tmp下)
              
              该选项属于一种transfer rule，而不是exclude rule，因此不会影响进入file list的文件，
              也因此不会影响删除操作。该选项仅对receiver所请求要传输的文件进行了限制。

       --ignore-existing
              该选项告诉rsync忽略对目标主机上已存在的文件的更新(不会忽略已存在的目录，或者什么
              也不做)。见"--existing"选项说明。

              该选项属于一种transfer rule，而不是exclude rule，因此不会影响进入file list的文件，
              也因此不会影响删除操作。该选项仅对receiver所请求要传输的文件进行了限制。              

              对于使用了"--link-dest"选项做备份时，碰巧备份被中断，如果想继续完成备份，则该选项
              有用。因为"--link-dest"选项会拷贝到一个新的目录层次中，使用"--ignore-existing"将
              保证已存在的文件不会被调整。这意味着该选项仅盯着目标上已存在的文件不放。

       --remove-source-files
              该选项告诉rsync移除sender端已经成功传输到receiver端的文件(不包括任何目录文件)。
              
       --delete
              该选项告诉rsync删除receiver端有而sender端没有的文件，但不是删除receiver端所有文件，
              而是只对将要同步的目录生效。你需要明确指定整个目录(如"dir"或"dir/")而不是使用通配
              符来通配目录的内容(如"dir/*")，因为通配符会被shell进行扩展，使得rsync被请求传输单
              个文件而非文件的父目录。被exclude排除的文件也会从delete中排除掉，除非使用了
              "--delete-excluded"选项或者标记了只对sender端匹配上的文件有效(见筛选规则中的
              include/exclude修饰符)。
              (译者注：由于exclude规则先生效，delete时认为源端不存在而目标端存在，使得delete也
              想要删除这些被排除的文件，但默认情况下，在删除时对这些被排除的文件加上了保护规则，
              所以这些文件无法被delete掉，这是一个容易疑惑的地方。要删除这些被排除的文件，只需
              使用选项"--delete-excluded"选项将这些被保护的文件强制取消保护)
               
              (译者注：(1)不会删除receiver端任何目录，即使是子目录也不删除；(2)delete动作是由
              generator进程执行的)
              
              该选项如果错误使用将是非常危险的！强烈建议先使用"--dry-run"(-n)进行测试，以确定将
              删除那些文件。

              如果sender端探测到了任何i/o错误，将自动禁用远程删除功能。这是为了防止sender端的临
              时文件系统故障(如NFS错误)导致大规模删除目标文件。可以指定"--ignore-errors"选项强
              制忽略任何I/O错误。

              "--delete"选项一般可能会配合"--delete-WHEN"的某一种或"--delete-excluded"，它们不
              会冲突。但若未指定任何"--delete-WHEN"时，rsync将默认采用"--delete-during"算法。
              (译者注：即在generator启动后，每处理一个文件列表，就删除该文件列表中需要删除的文
              件，在处理某个文件列表时不会删除别的文件列表中的文件，其实这一点从"-vvvv"的结果中
              很容易获取到)

       --delete-before
              请求在传输开始前执行目标文件删除行为。该选项隐含了"--delete"选项。
              (译者注：传输之前删除指的在处理所有文件列表之前先删除所有文件列表中指定要删除的
              文件，也就是说在generator刚启动时立即删除所有文件列表中待删除文件，而默认的
              --delete则是在generator刚启动时删除第一个文件列表中的待删除文件)

              在传输之前执行删除对于文件系统空间紧俏时是很有帮助的。但是，由于它会在传输开始之
              前发起一段延迟，这一段延迟很可能会使得传输超时(如果指定了"--timeout"选项)，还会
              强制rsync使用老的、非增量的递归算法，此算法要求rsync将所有传输中的文件一次性扫描
              到内存中。

       --delete-during, --del
              请求receiver端的文件删除行为是随着文件传输时逐步执行的。隐含了"--delete"选项。
              (译者注："--delete"没有和"--delete-WHEN"同时使用时，"--delete"默认采用的就是
              "--delete-during"）

       --delete-delay
              请求receiver端的文件删除行为在所有文件列表中的文件都全部传输完成后才删除。在结
              合"--delay-updates"、"--fuzzy"选项一起使用时比较有用，并且相比"--delete-after"
              来说效率更高。

       --delete-after
              请求receiver端的文件删除行为在所有文件列表中的文件都全部传输完成后才删除。和
              "--delete-delay"不同的是，该选项会采用老的、非增量传输的算法将传输中的所有文件
              一次性扫描到内存中，因此效率不高。

       --delete-excluded
              为了删除sender端没有而receiver端有的文件，可以指定该选项告诉rsync即使文件被
              "--exclude"排除了，也要在远程将其删除。

       --ignore-errors
              告诉"--delete"，即使在遇到I/O错误时也要继续。

       --force
              该选项告诉rsync，当某个非空目录要被非目录文件替换时，将此非空目录删除掉。这只有
              在删除行为未激活时才有效。

       --max-delete=NUM
              限制rsync最多能删除NUM个文件或目录，如果突破了该限制，将输出警告信息并以状态码
              25退出。

              可以使用"--max-delete=0"来保证不会在远程删除任何文件，因为只要有删除行为，就会
              警告并退出。

       --max-size=SIZE
              限制rsync传输的最大文件大小。可使用单位后缀，还可以是一个小数值(例如：
              "--max-size=1.5m")。

              该选项是一个传输规则，而不是排除规则，因此不会影响文件进入文件列表，也因此不会影
              响删除行为。它仅仅只是限制了receiver端请求传输的文件。

              有如下可用后缀："K"("KiB")=1024字节、"M"("MiB")、"G"("GiB")。如果想使用1000作为
              换算单位，则使用KB、MB、GB。(注意，所有大写字母都可以使用小写字母替换)。最后，如
              果后缀以"+1"或"-1"结尾，则值表示减小或增大一个字节。

              例如："--max-size=1.5mb-1"表示1499999字节，"--max-size=2g+1"表示2147483649字节。

       --min-size=SIZE
              限制rsync传输的最小文件大小。这可以用于禁止传输小文件或那些垃圾文件。单位的指定
              方法同"--max-size"。

       -B, --block-size=BLOCKSIZE
              该选项强制修改rsync算法将文件划分为数据块时的块大小。一般基于正在更新的文件来选
              择大小值。详细信息见技术报告。
              (译者注：rsync算法的作者说块大小在500-1000时是比较好的选择。对于超过1M的文件绝对
              不要让块大小低于500，否则性能极低。)

       -e, --rsh=COMMAND
              该选项允许你选择用于本地和远程之间的通信的远程shell程序。一般情况下，rsync默认配
              置为使用ssh，但如果在本地网络上，你可能更喜欢使用rsh。

              如果该选项和格式[user@]host::module/path一起使用，则该远程shell命令会在远程主机
              上启动一个rsync daemon(译者注：可以认为是临时模
              拟的rsync daemon进程)，并且所有数据都将通过此远程shell的连接传输，而不是通过网络
              套接字所连接的远程主机上的rsync daemon。见上文"通过远程SHELL连接使用RSYNC-DAEMON
              特性"。

              允许在COMMAND中提供多个远程shell参数，它们将会作为一个整体传递给rsync。多个参数
              必须使用空格(不能是制表符tab或其他任意空白字符)，分隔，也可以使用单引号或双引号
              包围参数来保护空格(但不能使用反斜线)。注意在单引号字符串内部使用多个单引号将返
              回给你一对单引号，同理，双引号也一样(尽管你需要注意哪些引号是shell来解析的，哪
              些引号是rsync解析的)。例如：
              
                  -e 'ssh -p 2234'
                  -e 'ssh -o "ProxyCommand nohup ssh firewall nc -w1 %h %p"'            

              (注意，可以在用户家目录下的.ssh/config文件中自定义ssh的连接选项。)

              你也可以使用RSYNC_RSH环境变量指定远程shell程序，它能接受和"-e"选项一样的值。

              另外请查看会影响该选项的"--blocking-io"选项。

       --rsync-path=PROGRAM
              指定远程机器上要运行的程序以启动远程rsync进程。当远程rsync程序不在默认目录
              /usr/local/bin/rsync下常会使用该选项指定其路径。但要注意，PROGRAM是在shell
              的帮助下运行的，因此它可以是任意程序、脚本或你想运行的命令序列，只要它们不
              会破坏rsync正在使用的标准输入和标准输出即可。

              一个很棘手的例子是在远程机器上设置不同的默认目录以便使用"--relative"选项。例如：

                  rsync -avR --rsync-path="cd /a/b && rsync" host:c/d /e/
                  
              (译者注：以上示例将会在本地主机创建/e/c/d，其中c/d数据来源于远程主机的/a/b/c/d)

       -C, --cvs-exclude
              这是一个很有用的简写排除文件法，用于排除大量不希望在系统之间传输的文件。它使用了
              类似于CVS的算法来决定一个文件是否要被忽略。
              
              exclude列表被初始化为排除以下格式的文件(这些初始化条目被标记为易过期，见后文"筛
              选规则"段落的说明)：

                     RCS SCCS CVS CVS.adm RCSLOG cvslog.* tags TAGS .make.state .nse_depinfo 
                     *~ #* .#* ,* _$* *$ *.old *.bak *.BAK *.orig  *.rej .del-* *.a *.olb *.o 
                     *.obj *.so *.exe *.Z *.elc *.ln core .svn/ .git/ .hg/ .bzr/

              然后，$HOME/.cvsignore文件中的文件列表将会被添加到此exclude列表，此外还会包含
              CVSIGNORE环境变量中指定的所有文件(所有cvsignore名称由空白字符分隔)

              最后，和.cvsignore文件在同一个目录中的文件，如果它们能匹配此文件中列出的规则，则
              也会被排除。不像rsync的筛选/排除规则，这些匹配模式是使用空白字符分隔的。更多cvs
              的模式见cvs(1)的man文档。

              如果将"-C"选项结合"--filter"规则，需要记住，无论"-C"选项处于命令行的哪些位置，这
              些cvs排除规则都将会追加在你所指定的规则之后。这就使得"-C"选项指定的规则比你自行
              指定的规则优先级耕地。如果你想将CVS规则插入到你的筛选规则中的某个位置，那么就不
              要使用"-C"选项，而是使用一种结合方式："--filter:C"或"--filter=-C"。前者启用
              .cvsignore文件中的规则进行每目录扫描，后者则是一次性导入所有上面说所的CVS规则。

       -f, --filter=RULE
              该选项可以让你添加规则，以便从待传输的文件列表中有选择性地排除某些文件。在递归传
              输中，结合该选项是非常有用的。

              你可以在命令行中使用任意多个"--filter"选项以建立要排除的文件列表。如果筛选规则中
              包含了空白字符，需要使用引号包围以防被shell解析。在下文中同样介绍了如何使用下划
              线替代空格来分隔rsync参数和规则。

              该选项的详细信息请参见"筛选规则"段落说明。

       -F     该选项是一种添加到"--filter"规则的简写法，只有两种可能：第一种是单个"-F"选项，此
              时它是以下规则的简写：

                 --filter='dir-merge /.rsync-filter'

              这告诉rsync查找那些分散在目录结构中的所有.rsync-filter文件，然后使用这些规则筛选
              出传输中的文件。第二种是如果重复使用"-F"选项，则它是以下规则的简写：

                 --filter='exclude .rsync-filter'

              该规则将从传输中筛选出.rsync-filter本身。

              关于该选项如何工作的更详细信息，见"筛选规则"段落说明。

       --exclude=PATTERN
              该选项是"--filter"选项的简化格式，默认为排除(exclude)规则，并且它将禁止对普通的
              筛选规则进行解析。

              更详细信息见"筛选规则"段落说明。

       --exclude-from=FILE
              该选项和"--exclude"选项类似，但是它是从包含了排除规则的文件中读取排除规则(每行一
              个规则)。空行以及";"或"#"开头的行为注释行，如果给定的文件为"-"，则表示从标准输入
              中读取排除规则。

       --include=PATTERN
              该选项是"--filter"选项的简化格式，默认为包含(include)规则，并且它将禁止对普通的
              筛选规则进行语法解析。
              
               更详细信息见"筛选规则"段落说明。

       --include-from=FILE
              该选项和"--exclude"选项类似，但是它是从包含了排除规则的文件中读取排除规则(每行一
              个规则)。空行以及";"或"#"开头的行为注释行，如果给定的文件为"-"，则表示从标准输入
              中读取排除规则。

       --files-from=FILE
              该选项可以在FILE中精确指定要传输的文件列表。如果FILE为"-"则表示从标准输入中读取
              文件列表。它还调整了rysnc的默认行为以便能够更简单地指定要传输的文件：

              o     它隐含了"--relative"(-R)选项，所以会保留在文件中每个条目所指定的路径信息。
                    (使用"--no-relative"或"--no-R"关闭该功能)

              o     它隐含了"--dirs"(-d)选项，所以将在目标主机上创建列表中指定的目录而不是悄
                    悄地跳过它们。(使用"--no-dirs"或"--no-d"关闭该功能)

              o     "--archive"(-a)将不再隐含"--recursive"(-r)，因此如果真的要递归到目录中，
                    需要显式指定"--recursive"(-r)。
              
              o     它的副作用是改变了rsync的默认状态，因此"--files-from"选项在命令行中的位置
                    和其它选项的解析无关(例如，"-a"选项放在"--files-from"选项的前后的工作方式
                    是一样的)。

              从FILE中读取的所有文件名都是相对于源目录的相对路径，任何前导斜线都会被移除，也
              无法使用".."进入源目录的上一层次目录。例如：

                 rsync -a --files-from=/tmp/foo /usr remote:/backup

              如果/tmp/foo中包含了字符串"bin"或"/bin"，将在远程主机上创建/backup/bin作为
              /usr/bin所对应的目标文件。如果包含了字符串"bin/"(注意尾随斜线)，则将传输
              /usr/bin目录以及其内的文件。如果指定了"-r"选项，则所有目录结构都会被传输
              (要记住当使用了"--files-from"时，"-r"选项需要显式指定，因为"-a"选项不再隐
              含该选项)。还要注意，适用"--files-from"时，"--relative"选项的默认行为是仅
              复制从文件中读取的路径信息，不再强制复制原规范路径(即此示例中的/usr)

              另外，"--files-from"的文件可以从远程主机上读取，而不一定要从本地主机上读取，
              需只在文件的前面指定"host:"即可。要求"host:"的host必须是rsync两端的某一端，
              为了简写，可以使用简写的前缀":"表示远端主机。例如：
              
                 rsync -a --files-from=srchost:/path/file-list srchost:/ /tmp/copy
                 rsync -a --files-from=:/path/file-list        srchost:/ /tmp/copy

              这将会拷贝在远程主机"srchost"上/path/file-list文件中指定的所有文件。

              注意：对"--files-from"的文件进行排序可以使得rsync效率更高，因为它将使得rsync不
              用再重新读取相邻能共用的路径元素。如果不进行排序，则rsync可能会多次重新扫描路径
              元素(隐含目录)，重复生成文件列表。

       -0, --from0
              告诉rsync从文件中读取规则或文件名时是以空字符(\0)终止的。该选项会影响
              "--exclude-from"、"--include-from"、"--files-from"以及所有在"--filter"
              中指定的规则合并文件。它不会影响"--cvs-exclude"(因为从.cvsignore文件中
              读取的名称都是以空白字符分隔的)。

       -s, --protect-args
              该选项将使得所有发送给远程rsync进程的文件名和大多数选项都不允许被远程shell解析。
              这意味着空格不再分隔文件名，任意非通配特殊字符都不会被翻译(即成为普通字符，
              如：~、$、;、&等)。通配字符将被远程主机上的rsync扩展(正常情况下是由远程shell来
              扩展的)。

       -T, --temp-dir=DIR
              该选项明确receiver端文件重组时的临时目录。默认情况下，receiver端将在文件所在目
              录中创建临时文件。

              在receiver端，如果目标文件所在磁盘分区剩余大小不足以存储待重组文件时，可以使用
              该选项将临时文件存储到其他的分区中，在这种情况下，由于创建的临时文件和目标路径
              不在同一个分区上，所以重组完成时无法直接重命名，而是只能从临时目录中拷贝到目标
              路径下。

              如果你使用该选项的原因不是磁盘空间不足，你可能要将此选项结合"--delay-updates"选
              项一起使用，这将保证所有拷贝的文件都放入目标层次结构的子目录中，并等待传输结束。

       -y, --fuzzy
              该选项告诉rsync，如果目标主机上的basis file缺失，将主动搜索出一个文件作为basis 
              file。目前的算法是在同一目录中模糊搜索目标basis file，搜索的规则是：要么文件大小
              和修改时间完全一致，要么文件名相似。如果搜索到了符合条件的文件，rsync将使用该文
              件作为basis file，这样可能会加速传输速度。
              (译者注：在目标主机上进行文件重组时，会从basis file中拷贝匹配块，在真正的basis 
              file缺失时，模糊搜索出的basis file可能能提供一些匹配块，从而减少sender要发送的
              数据量，加快整个同步过程)
              
              注意，如果使用了"--delete"选项，可能会把潜在的basis file给删除掉，因此要想避免
              种这情况，可以指定"--delete-after"选项，或者直接指定文件名来排除将被删除的文件。
             
       --compare-dest=DIR
              该选项指示rsync使用目标主机上的DIR作为额外的层次结构，以便和传输中的文件做比较
              (如果目标目录中不存在basis file)。如果在DIR中发现了和sender端完全一致的文件，则
              该文件将不会传输到目标目录中。该选项在做稀疏备份时很有用，因为它仅备份从某一次
              更早的备份开始发生了改变的文件。(译者注：以备份目录为比较目录DIR，同步时将比较
              该目录中的文件，最终将仅传输发生了改变的文件到目标目录下)

              从rsync 2.6.4版本开始，可以通过多个"--compare-dest"选项提供多个比较目录，使得
              rsync可以按照为了精确匹配而指定的顺序来搜索列表。如果发现能匹配上但仅只有属性不
              同，将生成一个本地副本然后更新这些属性信息。如果未能匹配上，将选择DIRs中的basis 
              file来提高传输速度。

              如果DIR是相对路径，它将是相对于目标目录的。见"--copy-dest"和"--link-dest"。

       --copy-dest=DIR
              该选项类似于"--compare-dest"，但rsync会从DIR中以本地拷贝的方式拷贝未改变的文件
              到目标目录中。

              可以通过多个"--copy-dest"选项提供多个DIR，使得rsync可以按照为了匹配未修改文件的
              顺序来搜索列表。如果未匹配上，则选择DIR中的basis file以尝试提高传输速度。

              如果DIR是相对路径，它将是相对于目标目录的。见"--copy-dest"和"--link-dest"。

       --link-dest=DIR
              This option behaves like --copy-dest, but unchanged files are hard linked from 
              DIR to the destination directory. The files must be identical in all preserved 
              attributes (e.g. permissions, possibly ownership) in order for the files to be 
              linked together. An example:

                rsync -av --link-dest=$PWD/prior_dir host:src_dir/ new_dir/


              If file’s aren’t linking, double-check their attributes.  Also check if some 
              attributes are getting forced outside of rsync’s control, such a mount option 
              that squishes root to a single user, or mounts a removable drive with generic 
              ownership (such as OS X’s "Ignore ownership on this volume" option).

              Beginning in version 2.6.4, multiple --link-dest directories may be provided, 
              which will cause rsync to search the list in the order specified for an exact 
              match.  If a match is found that differs only in attributes, a local copy is 
              made and the  attributes updated.  If a match is not found, a basis file from 
              one of the DIRs will be selected to try to speed up the transfer.

              This option works best when copying into an  empty destination  hierarchy, as 
              rsync treats existing files as definitive (so it never looks in the link-dest 
              dirs when a destination file already exists), and as malleable  (so it might 
              change the attributes of a destination file, which affects all the hard-linked 
              versions).

              Note that  if you combine this option with --ignore-times, rsync will not 
              link any files together because it only links identical files together as 
              a substitute for transferring the file, never as an additional check after 
              the file is updated.

              If DIR is a relative path, it is relative to the destination directory. See 
              also --compare-dest and --copy-dest.

              Note that rsync versions prior to 2.6.1 had a bug that could prevent --link-dest 
              from working properly for a non-super-user  when -o was specified (or implied 
              by -a).  You can work-around this bug by avoiding the -o option when sending to 
              an old rsync.

       -z, --compress
              使用该选项，rsync将对发送给目标主机的文件数据(file data)进行压缩，这可以减少传输
              的数据量——在某些缓慢的连接中可能比较适用。

              注意，该选项一般情况下可以实现比通过远程shell压缩或传输过程压缩获得更好的压缩比，
              因为它利用了明确不通过连接发送的匹配数据块中的隐含信息。

              请通过"--skip-compress"选项确定不被压缩的默认文件后缀列表。

       --compress-level=NUM
              显式指定"--compress"的压缩级别。如果NUM为非零值，则该选项将隐含"--compress"。

       --skip-compress=LIST
              该选项通过指定后缀格式来决定哪些文件不被压缩。LIST的值为使用斜杠"/"分隔的一个或
              多个文件后缀(不包括点.)

              如果LIST指定的是空字符串，则表示压缩所有文件。

              支持简单的字符类匹配。所谓的字符类由中括号和写在中括号中的一系列的字母组成(如：
              非特殊的字符类[abcz]，特殊的字符类[:alpha:]，注意，短横线"-"在此没有特殊的意义，
              仅表示一个简单的字符)。

              注意，字符"*"和"?"没有特殊的意义。

              例如此处示例指定了6个不压缩的后缀(因为其中一个规则mp[34]匹配了两种后缀)：

                  --skip-compress=gz/jpg/mp[34]/7z/bz2


              默认不被压缩的后缀列表为：(不同版本的rsync可能会有些微改变)

              7z avi bz2 deb gz iso jpeg jpg mov mp3 mp4 ogg rpm tbz tgz z zip

              该后缀列表将会被"--skip-compress"选项指定的后缀列表覆盖，但有一种情况例外：以
              rsync daemon为源拷贝文件时，rsync将在不被压缩的文件列表中增加你指定的后缀文件(
              并且该文件列表可能会被配置为非默认情况)(译者注：也就是说，当从rsync daemon拷贝
              时，指定的压缩忽略后缀不是覆盖原压缩忽略列表，而是追加到原压缩忽略列表)。

       --numeric-ids
              使用该选项后，rsync将传输gid和uid，而不是字符格式的username和groupname然后将其
              映射到两端。

              默认情况下，rsync将使用用户名和组名来决定如何设置文件的所有者和所属组。但要注
              意，即使没有指定"--numeric-ids"选项，也绝不会通过字符格式的username和groupname
              来映射特殊的uid=0和gid=0。

              如果在源主机上的用户或组没有字符格式的名称(译注：即只有uid和gid，没有username
              和groupname)，或者目标主机上没有相同的用户/组，则将使用uid/gid来设置所有权关系。
              请参看rsyncd.conf的man文档中关于"use chroot"的设置说明，以获取关于chroot设置后
              如何影响rsync对用户名和组名的查找能力以及你能对此做些什么。

       --timeout=TIMEOUT
              该选项设置的是最大IO超时时间，单位为秒。如果在指定时间内没有数据被传输，则rsync
              将直接退出。默认值为0表示永不超时。

       --contimeout
              该选项设置rsync成功连接rsync daemon的等待超时时间。若在指定时间内未连接上rsync 
              daemon，rsync将以错误方式退出。

       --address
              默认当rsync连接rsync daemon时，rsync所使用的地址将会绑定到通配符地址上(译注：默
              认通配符地址为0.0.0.0，即可以从任意接口的任意地址上向外发起连接)。使用该选项可以
              明确指定rsync要绑定的IP地址或主机名。请同样参见"--daemon"模式中"--address"选项获
              取详细信息。

       --port=PORT
              该选项可以显式指定一个要使用的tcp端口号而不是默认的873端口。该选项只能在连接rsync
              daemon时的双冒号(::)的语法格式中使用(因为URL rsync://user@host:port/语法本身就可
              以指定端口号，所以不需要该选项明确指定)。请同样参见"--daemon"模式中"--port"选项获
              取详细信息。

       --sockopts
              这个选项可以为希望最大程度地调整系统的人们提供无穷乐趣。你可以设置各种套接字选项
              使得传输速度更快(或更慢！)。关于该选项的可设置项请阅读setsockopt()系统调用man文
              档。默认情况下，该选项没有设置任何值。该选项只影响使用套接字连接rsync daemon的情
              况。在"--daemon"模式中也有该选项。

       --blocking-io
              该选项告诉rsync在启动远程shell传输时使用阻塞I/O模型。如果远程shell为rsh或remsh，
              rsync默认使用阻塞I/O，否则默认将使用非阻塞I/O。(注意，ssh更适合使用非阻塞I/O)

       -i, --itemize-changes
              请求输出一个明细列表，这个列表记录的是每个文件发生了哪些改变，包括属性的改变。它
              完全等价于"--out-format='%i %n%L'"。如果重复该选项，未发生改变的文件也会被输出。
              
              因为几乎不用，所以不做翻译。

       --out-format=FORMAT
              该选项允许你自定义rsync client对每个更新过程产生输出的信息格式。指定的格式为"%"
              加一个单字符。如果指定了"-v"选项，则其默认的格式为"%n%L"(将输出文件名，如果是一
              个链接文件，则输出其指向的对象)。完整的格式列表，见rsyncd.conf的man文档中"log 
              format"段落。

              指定"--out-format"是对每个文件、目录等都输出信息的。
              
              rsync将在文件传输之前输出这些信息，除非请求输出传输过程中的统计信息，这样它会在
              文件传输结束前输出信息，在这种情况下如果同时指定了"--progress"选项，rsync将会同
              时输出正在传输中的文件名。

       --log-file=FILE
              请求将输出信息记录到文件中。类似于"--out-format"。它可以指定记录client以及非
              daemon的server端的日志。若在客户端的命令行上指定该选项，则默认格式为"%i %n%L"。
              以下是一个请求远端来记录日志的示例：

                rsync -av --rsync-path="rsync --log-file=/tmp/rlog" src/ dest/

              该选项对于调试非预期的连接关闭比较有用。

       --log-file-format=FORMAT
              该选项指定要在"--log-file"文件中记录的日志格式(当然，需配合"--log-file"才生效)。
              如果FORMAT指定为空，将不会记录更新的文件信息。完整的格式列表，见rsyncd.conf的
              man文档中"log format"段落。

       --stats
              该选项让rsync打印文件传输过程中的详细统计数据，从中可以看出增量算法的效率如何。

              统计数据包括如下几项：

              o    Number of files是所有"文件"的数量总数，包含目录、字符链接等。

              o    Number of files transferred是普通文件通过rsync增量传输算法更新的数据大小，
                   它不包括创建的目录、字符链接等非文件数据。
  
              o    Total file size是传输中所有文件总大小。该大小不会计算目录和特殊文件大小，
                   但会包含字符链接的大小。 

              o    Total transferred file size是所有已传输文件的总大小。

              o    Literal data是所有非匹配数据块的数据大小，即传输给receiver端的纯文件数据
                   大小。 
              
              o    Matched data是所有匹配块的数据总大小，即receiver端从basis file中拷贝的数
                   据大小。

              o    File list size是sender发送给receiver端的文件列表大小。它比内存中文件列表的
                   内容要小很多，因为sender将其传输给receiver端的时候会将其进行压缩。

              o    File list generation time是sender创建文件列表消耗的时间，单位为秒。老版本
                   的rysnc可能不会输出该项。

              o    File list transfer time是sender将文件列表发送给receiver端消耗的时间，单位秒。 

              o    Total bytes sent是rsync从sender端传输给receiver端的所有数据大小，单位字节。
                   (译者注：包括纯文件数据，信息数据等等非文件数据)

              o    Total  bytes  received是receiver端收到的所有非信息数据大小，单位字节。

       -8, --8-bit-output
              告诉rsync在输出信息中保留所有高位字符不进行转义。几乎用不上，所以不做翻译。

       -h, --human-readable
              以人类可读方式输出信息。如果只指定单个该选项，则K/M/G的转换单位为1000，指定多次，
              则转换单位为1024。

       --partial
              默认rsync在传输中断时会删除只传输了一部分不完整文件(partial file)。在某些环境下，
              可能希望保留这些已传输的部分。"--partial"选项告诉rsync保留这些部分，这可以使得下
              一次传输只传输剩余部分，从而加快传输速度。

       --partial-dir=DIR
              比直接使用"--partial"更好的做法是指定一个存放不完整数据(partial file)的目录DIR，
              下次再传输时，rsync将使用一个文件来寻找到该DIR中的文件，以便提高传输速率，当数据
              真正传输完整且传输真正完成后，该DIR将被删除。

              注意，如果指定了"--whole-file"选项(后者隐含了该选项)，则所有的"partial-dir"中正
              在更新的文件都会被删除，因为rsync此时没有使用增量传输算法来传输文件。

              rsync如果发现DIR不存在，则自动创建(但不是递归创建整个目录路径)，这使得rsync在需
              要时能更方便地使用相对路径在文件所在目录下直接创建partial-dir
              (例如"--partial-dir=.rsync-partial")，并且在partial file被删除时自动移除该目录。

              如果partial-dir的值不是一个绝对路径，rsync将在所有exclude规则尾部追加一个exclude
              规则，用于防止partial file已经存在于sender端，也能防止receiver端过早删除partial 
              file。例如："--partial-dir"将在其他筛选规则的尾部添加一个等价于
              "-f 'P .rsync-partial/'"的规则。

              如果你正在使用自定义的exclude规则，你可能需要在你的exclude/hide/protect规则中添
              加partial-dir相关的条目，因为(1)向其他规则的尾部自动添加规则的行为可能会失效(2)
              你可能希望覆盖rsync的排除选择。例如，如果你想让rsync清除被随意乱放置的剩余partial 
              file，需要指定"--delete-after"并且添加一条"risk"筛选规则，如
              "-f 'R .rsync-partial/'"(不要使用"--delete-before"或"--delete-during"，除非你不想
              让rsync使用剩余的partial file来完成此次传输)。

              重要："--partial-dir"目录不能让其他用户可写，否则有安全隐患。例如避免使用"/tmp"。

              你也可以通过环境变量"RSYNC_PARTIAL_DIR"来设置partial-dir的值。设置该环境变量不会
              强制开启"--partial"，但在指定了"--partial"时会影响它的partial-dir路径。例如，不
              再让"-partial-dir=.rsync-tmp"随同"--progress"一起使用，而是设置
              "RSYNC_PARTIAL_DIR=.rsync-tmp"，然后只需使用"-P"选项来应用".rsync-tmp"目录以完成
              partial传输。只有在以下情况下"--partial"不会查找该环境变量的值：
              (1)使用了选项"--inplace"(因为"--inplace"和"--partial-dir"选项冲突)
              (2)使用了选项"--delay-updates"(见下文)。

       --delay-updates
              该选项将receiver端每个重组的临时文件保留在某个目录中，直到传输结束之前才一次性将
              它们全部重命名为各自对应的目标文件。这样的行为使得所有文件的更新更具有原子性
              (译注：如果你了解数据库事务，就知道原子性是什么意思，最直白地说，具有原子性表示
              要么全部成功，要么全部失败，所以这里更新具有原子性表示要么全部更新成功，要么全部
              更新失败，但由于重命名覆盖目标文件后是不可回滚的，所以这里的原子性并不是那么的严
              格)。默认情况下，这些临时文件将放在每个目标文件所在目录下的".~tmp~"子目录下，但
              如果指定了选项"--partial-dir"，那么将使用该选项指定的目录。请参见"--partial-dir"
              选项获取相关信息。该选项和"--inplace"以及"--append"选项冲突。
       
              该选项使得receiver端使用更多的内存空间，并且需要更多的磁盘空间以存储额外的目标文
              件副本。需要注意，"--partial-dir"不能使用绝对路径，除非你能保证传输中的文件没有
              同名文件(因为如果使用了绝对路径，所有的临时文件都放在那一个目录下，重名文件会先
              后覆盖)，且在目录层次结构中没有挂载点(因为如果无法重命名到指定路径下，延迟更新将
              失败)

       -m, --prune-empty-dirs
              该选项告诉receiver端的rsync从文件列表中删除空目录，包括那些没有文件的空的嵌套目
              录。这对于避免创建一堆无用的目录很有用，例如在sender端使用include/exclude/filter
              规则扫描递归层次时会创建所有层次的目录。

              注意传输规则(如"--min-size")不会影响文件进入文件列表，因此不会让目录成为空目录，
              即使目录中没有任何文件可以匹配上传输规则。

              由于该选项会修剪file list，所以会影响delete对目录的删除行为。但记住，exclude排除
              的文件和目录可以防止现有条目被删除，因为排除规则既隐藏了源文件，又保护了目标文件。

              可以使用全局"protect"筛选规则防止file list中的某个特定目录被修剪。例如，下面的选
              项可以保证目录"emptydir"仍然保留在file list中：

              --filter ’protect emptydir/’

              以下的示例可以从一个层次结构中拷贝其中所有的.pdf文件，并且只在目标主机上创建必
              要的目录来存放这些.pdf文件，并会移除目标上所有多余文件和目录(注意此处使用了非目
              录文件的hide筛选规则替代exclude规则)：

              rsync -avm --del --include=’*.pdf’ -f ’hide,! */’ src/ dest

              若不想删除目标主机上多余文件，可使用更古老的选项"--include='*/' --exclude='*'"
              替代hide筛选规则，不过你不一定能适应它，毕竟它是古老的选项。

       --progress
              该选项告诉rsync显示传输进度信息，这是给那些无聊的用户看的。它隐含了"--verbose"。
              
              如果rsync正在传输的是一个普通文件，将以下面格式显示进度信息：

                    782448  63%  110.64kB/s    0:00:04

              在此例中，receiver重组了sender发送的文件的782448字节的数据或者说重组了该文件的
              63%(译者注：此处的意思是这些数据是文件的纯数据，不包括那些非文件信息类数据)，且
              以110.64kB/s的速率重建文件，如果保持该速率，该文件将在4秒后重建完成。

              若使用的是rsync的增量拷贝算法，那么这些统计数据可能会误导用户。例如，如果sender
              的文件由basis file和另一段放在basis file前面的数据组成，那么当reciever在获取纯文
              件数据时，此处所显示的速率值会急剧下降，而且要完成传输可能会比估计的时间更长，因
              为它还要处理匹配的数据块部分。(译者注：换句话说，该进度信息是sender发送的纯数据
              相关信息，和真正完成同步的进度没有直接关系)              

              当文件传输完成，rsync将使用总结性的行替代进度信息，类似格式如下：

                   1238099 100%  146.38kB/s    0:00:08  (xfer#5, to-check=169/396)

              此示例中，文件大小为1238099字节，整体传输平均速率是146.38kB/s，总共消耗了8秒才传
              输完成。"xfer#5"表示此文件是该rsync会话期间第5个传输的文件，"to-check=169/396"表
              示此次传输的文件列表中共要检查396个文件，其中有169个文件待检查以确定它们是否需要
              传输。
              (译者注：可以认为396是某些文件被排除后待考虑是否传输的总文件数，196则是还剩下196
              个文件还未检查)

       -P     该选项等价于"--partial --progess"选项。指定这两个选项的目的是为了让某一次可能会
              中断的较长传输过程变得更简单。

       --password-file
              该选项让rsync在连接rsync daemon时从密码文件中获取密码。密码文件必须可读。该文件
              中只有第一行是rsync将读取的密码，其他所有行都自动忽略。
              
              该选项无法为远程shell(如ssh)提供密码，至于如何为远程shell提供密码，参考对应远程
              shell的文档(译者注：对于ssh而言，使用公钥认证机制即可)。当使用远程shell访问rsync
              daemon时，该选项只有完成了远程shell的身份验证过程才生效。

       --list-only
              该选项强制rsync仅列出源路径的文件列表而不是进行文件传输。如果rsync命令行中只给出
              了一个地址，将隐含该选项。注意通配符会被shell解析并扩展为rsync的参数。例如：

                  rsync -av --list-only foo* dest/

       --bwlimit=KBPS
              该选项对rsync的传输最大速度进行限速。该限制值是一个平均值，所以在实际传输过程中
              可能会短暂的超出该限制值。设置为0则表示不限速。

       --write-batch=FILE
              记录一个稍后被"--read-batch"读取的文件，该文件可被用于另一个完全一致的目标路径。
              详细信息见"批处理模式"段落信息。
              
              (译者注：通俗地说，就是将源和目标的不同点记录下来保存在FILE中，然后通过
              "--read-batch"读取这些不同点并更新目标文件，如果有多台目标主机上的文件情
              况是完全一致的，则可以通过此FILE一次性应用于所有这些主机，因此称之为"批
              处理模式"。注意，FILE中不仅记录了不同之处，还记录了将要应用于目标主机的
              数据部分，因此它是一个"信息+数据"文件而不仅仅只是小小的信息文件)

       --only-write-batch=FILE
              和"--write-batch"工作方式类似，区别是当生成batch file的时候不会对目标路径做
              任何操作。这使得你可以通过某些方法将所发生改变的信息传输到目标系统上，然后通
              过"--read-batch"应用这些改变。

              可以直接将批处理文件FILE保存到任意的移动存储设备中。

              注意，只有向远程主机推送这些改变时才能节省带宽，因为这样可以让sender端的数据分
              流记录到FILE中，而无需通过连接传输给receiver端。(如果是拉取数据，则sender端为
              remote，因此无法写批处理文件FILE)


       --read-batch=FILE
              读取所有通过"--write-batch"生成的批处理文件FILE并通过其中的"信息+数据"应用于目
              标主机。如果FILE未"-"，则批处理数据将从标准输入中读取。  
              
       --protocol=NUM
              强制指定要使用的协议版本。一般只在创建批处理文件以兼容老版本rsync时可能会用上，
              因此不做翻译。

       --iconv=CONVERT_SPEC
              rsync可以在不同字符集间转换文件名。基本用不会上，因此不做翻译。

       -4, --ipv4 or -6, --ipv6
              限制rsync使用ipv4还是ipv6创建套接字。该选项由于明确表示了要用网络套接字，也就限
              制了只有连接rsync daemon时才生效。

              如果编译rsync时没有将ipv6的功能编译进去，则"--ipv6"无效。通过"--version"的输出
              结果可以知道是否编译了ipv6功能。

       --checksum-seed=NUM
              指定checksum的种子长度。默认种子长度为4字节，应用于每个数据块级和文件级checksum
              的计算。默认checksum的种子由server和其默认的当前系统时间time()计算生成。该选项显
              示指定特定的checksum种子，对应想要重复计算块级或文件级checksum时或者用户想要一个
              更具随机性的checksum种子时比较有用。设置NUM为0将导致rsync使用默认的time()来计算
              种子。


DAEMON OPTIONS
       当启动rsync daemon时，可以指定以下几个选项：

       --daemon
              该选项告诉rsync以daemon方式运行。
              可以在client端使用host::module or rsync://host/module/格式的命令来访问daemon。

              如果标准输入是一个套接字，则rsync被认为是通过inetd方式运行的，否则它将从当前终
              端上分离出来并成为后台守护进程。每次和daemon进行连接时，daemon都会读取配置文件
              (rsyncd.conf)并给出对应的相应。更详细信息见rsyncd.conf的man文档。

       --address
              rsync daemon的绑定地址，默认会绑定在通配地址上(默认为0.0.0.0)。使用"--address"
              选项可以显式指定要绑定的IP地址或主机。可以配合"--config"一起使用来实现rsync虚
              拟主机的功能。更多信息见rsyncd.conf的man文档中"address"段落说明。

       --bwlimit=KBPS
              该选项对rsync的传输最大速度进行限速。该限制值是一个平均值，所以在实际传输过程中
              可能会短暂的超出该限制值。设置为0则表示不限速。

       --config=FILE
              该选项用于指定额外的配置文件来代替默认的配置文件，只有和"--daemon"选项同时使用
              时才有效。默认daemon的配置文件为/etc/rsyncd.conf，除非是通过远程shell启动的临时
              daemon，使用远程shell连接的daemon的默认配置文件是当前目录(一般是$HOME)下的
              rsyncd.conf。

       --no-detach
              当以daemon形式运行时，该选项表示rsync不从终端中将自己分离出来，所以工作在前台。
              在各种daemon管理工具如daemontools、systemd上需要使用。如果rsync是由sshd或inetd
              派生出来的话，则该选项无效。

       --port=PORT
              指定daemon的监听端口，默认为873。

       --log-file=FILE
              该选项告诉rsync daemon使用此处指定的日志文件替代配置文件中"log file"指定的日志
              文件。

       --log-file-format=FORMAT
              该选项告诉rsync daemon使用此处指定的日志格式而不是配置文件中"log format"指定的
              日志格式。该选项会自动开启"transfer logging"，除非它的值为空，因为这样表示关闭
              transfer logging功能。

       --sockopts
              指定套接字选项，将覆盖配置文件配置的套接字选项。
              
       -v, --verbose
              该选项输出rsync daemon启动时的详细信息。它不控制客户端和daemon连接时的信息详细
              程度，因为这些信息详细程度是由client和模块配置段中的"max verbosity"控制的。

       -4, --ipv4 or -6, --ipv6
              告诉rsync使用IPV4还是IPV6创建套接字，然后rsync daemon将监听在对应的地址类型上。
              
              如果编译rsync时没有将ipv6的功能编译进去，则"--ipv6"无效。通过"--version"的输出
              结果可以知道是否编译了ipv6功能。
              
       -h, --help
              如果该选项指定在"--daemon"选项之后，则输出rsync daemon可用选项的简短帮助信息。


FILTER RULES

   (译者注：下面筛选规则的内容很多地方都提到了"传输中根目录"(transfer-root)的概念，所以提前
   在此做个解释。假设执行rsync -r /www/lvm /www/audit remote_host:/path命令，则待传输的目录
   lvm和audit称为传输过程中的根目录，即顶级目录)

       筛选规则可以弹性定义哪些文件需要传输(include)(译者注：前文中出现的传输规则指的就是
       include规则)，以及哪些文件需要跳过(exclude)。这些规则要么直接指定include/exclude匹配
       模式，要么指定一种方式从中获取include/exclude匹配模式(例如从文件中读取规则)。

       对于已经创建好的file list中的文件或目录，rsync会按先后顺序对其中的每一个名称检查是否
       能匹配incluee/exclude规，且先匹配上的规则生效：如果能匹配上exclude规则，则该文件被跳
       过，如果该文件能匹配include或不能匹配任何规则，则不挑过该文件。

       rsync会按照命令行中指定的筛选规则建立一个有序的规则列表。筛选规则语法如下：

              RULE [PATTERN_OR_FILENAME]
              RULE,MODIFIERS [PATTERN_OR_FILENAME]

       你可以选择使用下面所描述的长格式或短格式的RULE名称。如果使用短格式命名的的rule，则分隔
       RULE和MODIFIERS中间的","是可选的。其后的PATTERN或FILENAME必须跟在单个空格或下划线(_)之
       后。以下是规则前缀：

              exclude, - 指定排除规则。
              include, + 指定包含规则。
              merge, . 指定一个从中读取更多规则的merge-file。
              dir-merge, : 指定一个每目录的merge-file。
              hide, H 指定传输过程中需要隐藏的文件。(译者注：exclude的本质就是hide，该类型的
                      规则显然只作用于sender端，下面的"S"同样如此)
              show, S 指定不被隐藏的文件。(译者注：include的本质就是show)
              protect, P 指定保护文件不被删除的规则。
                    (译者注：--delete和--exclude同时使用时，会对被排除的文件加上保护规则。该
                    类型的规则显然只作用于receiver端，下面的"R"同样如此)
              risk, R 指定不被保护的文件，即能被删除的文件。(译者注："--delete-excluded"就是
                      将被保护的文件强制取消保护)
              clear, ! 清空当前include/exclude规则列表。(不带任何参数)

       如果是从一个文件中读取规则，将忽略空行以及使用"#"开头的注释行。

       注意，使用了"--include"或"--exclude"选项后，将不再解析上述所说的规则序列，只允许使用
       "!"表示清空include/exclude规则。如果匹配模式不是使用"- "(减号加一个空格)或"+ "(加号加
       一个空格)开头，则规则将被认为是在字符串的前面加了"+ "(即包含规则)或"- "(即排除规则)前
       缀。实际上"--filter"选项的规则字符串中必须在字符串开头包含短名称或长名称的规则。 

       同时需要注意的是，每个"--filter"、"--include"以及"--exclude"选项都只表示一条规则，如
       果要使用多条规则，你可以重复使用这些选项，或者使用"--filter"选项的merge-file语法，又
       或者是"--include-from"、"--exclude-from"选项。

INCLUDE/EXCLUDE PATTERN RULES
       你可以使用"+"、"-"等规则名称来指定包含和排除文件的规则，以及其他的规则。每个include、
       exclude规则都会对应一个匹配模式用于匹配将要传输
       的文件名。匹配模式有以下几种格式：

       o      如果匹配模式以斜线(/)开头，它表示锚定层次结构中某个特定位置的文件，否则将表示匹
              配路径名的结尾。这有点类似于正则表达式中的行首"^"。因此"/foo"匹配的是"传输中根
              目录"(transfer-root)下的"foo"文件，或者是merge-file目录中的"foo"文件。对于不做
              限制的"foo"匹配模式，由于算法会从顶部开始向下逐层递归，所以它能匹配任意位置名为
              "foo"的文件。但是对于非锚定的"sub/foo"模式，将匹配层次结构中位于sub目录下的foo。
              对于如何指定匹配根"/"的模式见"ANCHORING INCLUDE/EXCLUDE PATTERNS"的段落说明。

       o      若匹配模式以斜线(/)结尾，将只匹配目录，而不匹配普通文件、字符链接以及设备文件。

       o      rsync通过检查匹配模式中是否包含"*"、"?"以及"["符号来决定做简单的字符串匹配还是
              通配符匹配。

       o      单个"*"匹配任意路径元素，但在遇到斜线时终止匹配。

       o      "**"匹配任意路径元素，与"*"不同的是，它能匹配斜线。

       o      "?"匹配任意非斜线的单个字符。

       o      "["表示字符类匹配，例如"[a-z]"，[[:alpha:]]。

       o      在通配匹配模式中，反斜线(\)可以对通配符号进行转义，但如果不是对通配符号使用反斜
              线，则它仅仅只是一个普通的反斜线字符。

       o      如果匹配模式中包含了一个"/"(不包括以斜线结尾的情况)或"**"，则表示对包括前导目录
              的全路径进行匹配。如果匹配模式中不包括"/"或"**"，则表示只对全路径尾部的路径元素
              进行匹配。(注意：使用了递归功能时，"全路径"可能是从最顶端开始向下递归的某中间一
              段路径)。

       o      对于"dir_name/***"来说，它将匹配dir_name下的所有层次的文件。

       注意，当使用"--recursive"(-r)选项(-a隐含该选项)时，每个子路径元素会自顶向下逐层，
       被访问因此include/exclude匹配模式会对每个子路径元素的全路径名进行递归(例如，要包
       含"/foo/bar/baz"，则"/foo"和"/foo/bar"必须不能被排除)。实际上，排除匹配模式在发现
       有文件要传输时，此文件所在目录层次的排除遍历会被短路。如果排除了某个父目录，则更
       深层次的include模式匹配将无效，因为rsync从排除的那个父目录位置开始不会再向下遍历。
       这在使用尾随"*"时尤为重要。例如，下面的例子不会正常工作：

              + /some/path/this-file-will-not-be-found
              + /file-is-included
              - *

       由于父目录"some"被规则"*"所排除，所以会失败，rsync绝不会访问"some"或"some/path"中的任
       何文件。一种解决方式是请求包含层次结构中的所有目录，只需使用一个规则"+ */"(需放在"- *"
       规则的前面)即可，可能还需要使用"--prune-empty-dirs"选项。另一解决方式是为所有需要被访
       问的父目录
       增加特定包含规则。例如，下面的规则可以正常工作：

              + /some/
              + /some/path/
              + /some/path/this-file-is-found
              + /file-also-included
              - *

       以下是一些exclude/include规则匹配模式：

       o   "- *.o"将排除所有文件名能匹配"*.o"的文件。
           
       o   "- /foo"将排除"传输中根目录"(transfer-root)下名为"foo"的文件或目录。
           
       o   "- foo/"将排除所有名为"foo"的目录。 
           
       o   "- /foo/*/bar"将排除"传输中根目录"(transfer-root)下"foo"目录再向下两层的"bar"文件。
           
       o   "- /foo/**/bar"将排除transfer-root下"foo"目录再向下递归任意层次后名为"bar"的文件。
           (译者注："**"匹配任意多个层次的目录) 
           
       o   同时使用"+  */"、"+ *.c"和"- *"，将只包含所有目录和C源码文件，除此之外的所有文件
           和目录都被排除。(参见选项"--prune-empty-dirs")
           
       o   同时使用"+ foo/"、"+ foo/bar.c"和"- *"将只包含"foo"目录和"foo/bar.c"。("foo"目录
           必须显式包含，否则将被排除规则"- *"排除掉)

       以下是"+"或"-"后可接的修饰符：

       o   "/"指定include/exclude规则需要与当前条目的绝对路径进行匹配。例如，"-/ /etc/passwd"
           将在任意传输/etc目录中文件时刻都排除passwd文件。而对于"-/ subdir/foo"规则，当传输
           "subdir"目录中文件时，将总是排除"foo"文件，即使"foo"文件可能是在transfer-root中的。

       o   "!"指定如果模式匹配失败，则include/exclude规则生效。例如，"-! */"将排除所有非目录
           文件。(译者注：反向匹配的意思，"- */"规则是排除所有目录，那些非目录文件就匹配不上，
           加上"!"，即"-! */"，则表示匹配不上的这些非目录文件被匹配上)

       o   "C"表示将所有的全局CVS排除规则插入到普通排除规则中，而不再使用"-C"选项来插入。其后
           不能再接其他参数。

       o   "s"表示规则只应用于sender端。当某规则作用于sender端时，它可以防止文件被传输。默认
           情况下，所有规则都会作用于两端，除非使用了"--delete-exclude"选项，这样规则将只作用
           于sender端。请参见"hide"(H)和"show"(S)规则，这是指定sender端include/exclude规则的
           另一种方式。

       o   "r"表示规则只应用于receiver端。当规则作用于receiver端，它可以防止文件被删除。更多
           信息见上面的"s"修饰符。另请参见"P"和"R"规则，它们是指定receiver端include/exclude
           规则的另一种方式。

       o   "p"表示此规则是易过期的，这意味着将忽略正在被删除的目录。例如，"-C"选项的默认规则
           是以CVS风格进行排除，且"*.o"文件会被标记为易过期，这将不会阻止在源端移除的目录在
           目标端上被删除。
           (译者注：也就是说在源端删除的目录在目标端上也会被删除。以下是原文，翻译也许有误)
           A p indicates that a rule is perishable, meaning that it is ignored in directories 
           that are being deleted. For instance, the -C option’s default rules that exclude 
           things like "CVS" and "*.o" are marked as perishable, and will not prevent a
           directory that was removed on the source from being deleted on the destination.

MERGE-FILE FILTER RULES
       你可以通过指定"."将某文件中的规则合并在规则集合中，或指定":"(dir-merge)将目录下文件中
       的规则合并在规则集合中。

       有两种合并文件的方式：单实例合并"."和每目录合并":"。单实例合并文件仅读取一次，其中的规
       则被合并到筛选列表中替代"."所代表的规则。对于每目录合并文件，rsync将扫描每个目录以遍历
       指定名称的规则文件，并合并规则文件中的内容到当前规则列表中。每目录合并的规则文件必须创
       建在sender端，因为会扫描sender端来决定哪些文件要被传输。如果想让规则文件作用于receiver
       端不想被删除的文件，则需要将规则文件传输到receiver端(见下面的"PER-DIRECTORY RULES AND 
       DELETE")。

       一些示例：

              merge /etc/rsync/default.rules
              . /etc/rsync/default.rules
              dir-merge .per-dir-filter
              dir-merge,n- .non-inherited-per-dir-excludes
              :n- .non-inherited-per-dir-excludes

       以下是单实例合并"."和每目录合并":"后可接的修饰符：

       o      "-"表示规则文件中只有exclude规则，可以在文件中使用注释行。 

       o      "+"表示规则文件中只有include规则，可以在文件中使用注释行。
       
       o      "C"指定的是CVS分格的规则。它将会开启"n"、"w"和"-"，也允许指定规则列表清空修饰符
              "!"。若不提供任何文件名，则默认为".cvsignore"。

       o      "e"表示排除合并文件使其不被传输。
              例如"dir-merge,e .rules"等价于"dir-merge .rules"+"- .rules"。

       o      "n"表示规则不被子目录继承。

       o      "w"表示规则是使用空白符号分隔的而不是默认的换行符。显然，这种方式下合并文件中不
              能使用注释行。
              注意：分隔规则前缀的空格是有特殊意义的，例如"- foo + bar"会被解析为两条规则。

       o      此外，你可能还需要指定上文所述的"+"、"-"规则的修饰符，以便对这些从文件中读取的规
              则具有修饰符集(除了"!"修饰符)。例如"merge,-/ .excl"将把".excl"文件中的内容视为绝
              对路径排除，而"dir-merge,s .filt"和":sC"都将使得所有每目录规则仅作用于sender端。

       每目录合并规则会被所有的子目录继承，除非指定了"n"修饰符。每个子目录的规则会置放在继承
       规则的前面，使得最新的规则比继承的规则有更高的优先级。整个dir-merge的规则集合会在指定
       的merge-file中进行合并，因此可通过前面在全局规则列表中指定的规则来覆盖dir-merge规则。
       当从每目录文件中读取到了列表清除规则"!"，将仅从当前的合并文件中清除掉继承规则。

       阻止dir-merge中的单个规则被继承的另一种方式是使用前导斜线(/)进行锚定。每目录合并规则中
       的锚定规则是相对于merge-file目录的，因此"/foo"将仅匹配从dir-merge规则中找到的目录中的
       "foo"文件。

       以下是一个示例，需要指定[--fileter=".file"]：

              merge /home/user/.global-filter
              - *.gz
              dir-merge .rules
              + *.[ch]
              - *.o

       这将会在规则列表的头部合并/home/user/.global-file文件中内容，还会合并所有的".rules"文
       件中的规则。在目录扫描开始之前读取的所有规则都遵循全局锚定规则(例如，前导斜线匹配
       transfer-root)。

       If a per-directory merge-file is specified with a path that is a parent directory of
       the first transfer directory, rsync will scan all the parent dirs from that starting 
       point to the transfer directory for the indicated per-directory file.  For instance,
       here is a  common filter (see -F):

              --filter=': /.rsync-filter'


       That rule tells rsync to scan for the file .rsync-filter in all directories from the 
       root down through the parent directory of the transfer prior to the start of the normal 
       directory scan of the file in the directories that are sent as a part of the transfer. 
       (Note:  for an rsync daemon, the root is always the same as the module’s "path".)

       Some examples of this pre-scanning for per-directory files:

              rsync -avF /src/path/ /dest/dir
              rsync -av --filter=': ../../.rsync-filter' /src/path/ /dest/dir
              rsync -av --filter=': .rsync-filter' /src/path/ /dest/dir


       The first two commands above will look for ".rsync-filter" in "/" and "/src" before 
       the normal scan begins looking for the file in "/src/path" and its subdirectories. 
       The last command avoids the parent-dir scan and only looks for the ".rsync-filter" 
       files in each directory that is a part of the transfer.

       If you want to include the contents of a ".cvsignore" in your patterns, you should 
       use the rule ":C", which creates a dir-merge of the .cvsignore file, but parsed in 
       a CVS-compatible manner.  You can use this to affect where the --cvs-exclude (-C) 
       option’s inclusion of the per-directory .cvsignore file gets placed into your rules 
       by putting the ":C" wherever you like in your filter rules. Without this, rsync would 
       add the dir-merge rule for the .cvsignore file at the end of all your other rules 
       (giving it a lower priority than your command-line rules).  For example:

              cat < out.dat


       then  look  at  out.dat. If everything is working correctly then out.dat should be a 
       zero length file. If you are getting the above error from rsync then you will probably 
       find that out.dat contains some text or data. Look at the contents and try to work out 
       what is  producing it. The most common cause is incorrectly configured shell startup 
       scripts (such as .cshrc or .profile) that contain output statements for non-interactive
       logins.

       If you are having trouble debugging filter patterns, then try specifying the -vv 
       option. At this level of verbosity rsync will show  why each individual file is 
       included or excluded.


ENVIRONMENT VARIABLES
       CVSIGNORE
             The CVSIGNORE environment variable supplements any ignore patterns in .cvsignore 
             files. See  the  --cvs-exclude  option  for  more details.

       RSYNC_ICONV
              Specify a default --iconv setting using this environment variable. 
              (First supported in 3.0.0.)

       RSYNC_RSH
              The RSYNC_RSH environment variable allows you to override the default shell 
              used as the transport for rsync.  Command line options are permitted after 
              the command name, just as in the -e option.

       RSYNC_PROXY
              The RSYNC_PROXY environment variable allows you to redirect your rsync 
              client to use a web proxy when connecting to a rsync daemon. You should 
              set RSYNC_PROXY to a hostname:port pair.

       RSYNC_PASSWORD
              Setting RSYNC_PASSWORD to the required password allows you to run authenticated 
              rsync connections to an rsync daemon without user intervention. Note that this 
              does not supply a password to a remote shell transport such as ssh; to learn 
              how to do that,  consult the remote shell’s documentation.

       USER or LOGNAME
              The  USER or LOGNAME environment variables are used to determine the default 
              username sent to an rsync daemon.  If neither is set, the username defaults to 
              "nobody".

       HOME   The HOME environment variable is used to find the user’s default .cvsignore 
              file.


FILES
       /etc/rsyncd.conf or rsyncd.conf

SEE ALSO
       rsyncd.conf(5)


VERSION
       This man page is current for version 3.0.9 of rsync.

INTERNAL OPTIONS
       The options --server and --sender are used internally by rsync, and should never 
       be typed by a user under normal circumstances. Some awareness of these options may 
       be needed in certain scenarios, such as when setting up a login that can only run 
       an rsync command. For instance, the support directory of the rsync distribution has 
       an example script named rrsync (for restricted rsync) that can be used with a 
       restricted ssh login.



```

![复制代码](https://common.cnblogs.com/images/copycode.gif)