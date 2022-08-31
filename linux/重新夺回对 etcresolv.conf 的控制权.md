随着 Linux 的不断发展壮大，涌现出了各种各样的 DNS 自动管理程序，它们都想要直接获得 `/etc/resolv.conf` 的控制权，有些人欣然接受，有些人则无法接受。如果你是无法接受的那一方，那么请继续往下看，我会教你如何识别出是哪些程序在控制你的 `/etc/resolv.conf` 文件，以及如何夺回控制权。

目前能够控制 `/etc/resolv.conf` 文件的工具大概有这么几个：`netconfig`, `NetworkManager`, `resolvconf`, `rdnssd` 和 `systemd-resolved`。如果你的 `/etc/resolv.conf` 文件正在被它们控制，那么你对该文件的任何修改都会在几分钟后被覆盖，或者重启后被恢复成原来的值。

要想重新夺回对 `/etc/resolv.conf` 的控制权，首先就要识别出是谁在控制这个文件。

## 1\. 找出是谁在控制 /etc/resolv.conf

先尝试读取 `/etc/resolv.conf` 开头的注释，注释里一般会标明是谁在操控该文件：

有些工具不会在 `/etc/resolv.conf` 文件中添加注释，从文件内容里找不到任何蛛丝马迹。这时我们需要换种方法，直接查看该文件是否是一个软链接：

如果还是找不到任何线索，那就只能查看系统运行的进程中是否有上面提到的工具。如果还是找不到，那么恭喜你，`resolv.conf` 已经完全掌控在你的手里，你想怎么改就直接改吧。

接下来将会教你如何禁用自动管理 `resolv.conf` 的各种程序。

## 2\. NetworkManager

`NetworkManager` 是最常见的自动配置网络和 DNS 的工具。比如在 Debian 和 Fedora 中它负责配置 `/etc/resolv.conf`。NetworkManager 可以和其他工具共存，即使禁用了所有其他管理 `resolv.conf` 的程序，`NetworkManager` 也会跳出来接管 `resolv.conf`。

可以将 NetworkManager 的主配置部分的选项 `dns` 设置为 `none` 来禁用其对 DNS 的管理功能：

```
$ echo -e "[main]\ndns=none" > /etc/NetworkManager/conf.d/no-dns.conf
$ systemctl restart NetworkManager.service
$ rm /etc/resolv.conf
```

如果配置完了以后没有生效，那么可能存在配置冲突（通常是由 `dnsmasq` 引起的），需要找到冲突的配置：

```
$ grep -ir "\[main\]" /etc/NetworkManager/
```

## 3\. netconfig

如果是 `openSUSE`，`SUSE` 或其他衍生发行版，一般都是由 `netconfig` 来控制 `resolv.conf`。可以通过禁用 `/etc/sysconfig/network/config` 中的 `NETCONFIG_DNS_POLICY` 选项来禁用其对 resolv.conf 的控制：

还要删除 `netconfig` 生成的 `resolv.conf` 文件，并重启系统：

```
$ rm /etc/resolv.conf
$ reboot
```

现在就可以手动创建 `/etc/resolv.conf` 文件随意修改了。

## 4\. resolvconf 和 rdnssd

如果是 `Debian 8.0` 或 `Ubuntu 15.04`，并且启用了 `IPv6`，那么你可能会遇到 [`resolvconf` 和 `rdnssd` 互相争夺 resolv.conf 控制器](https://www.ctrl.blog/entry/how-to-debian-dns-resolv.html)的情况。两个服务都想控制这个文件，每隔几毫秒就会覆盖对方的配置，从而导致间歇性的 DNS 解析中断。可以直接禁用并立即停止这两个服务：

```
$ systemctl disable --now resolvconf.service rdnssd.service
$ rm /etc/resolv.conf
```

最后手动创建 `/etc/resolv.conf` 文件。

## 5\. systemd-resolved

如果是 `Ubuntu 16.10` 或更新的版本，则由 `systemd-resolved` 服务来管理 DNS，可以使用下面的命令来禁用并立即停止该服务：

```
$ systemctl disable --now systemd-resolved.service
$ rm /etc/resolv.conf
```

然后手动创建 `/etc/resolv.conf` 文件。

## 6\. 创建 /etc/resolv.conf

最后的最后，就是手动创建 `/etc/resolv.conf` 文件了，建议权限设置为 `644`。配置示例：

```
nameserver 114.114.114.114
nameserver 223.5.5.5
```

当然，除了 nameserver 外，还有其他的参数可以配置，感兴趣可以 man 一下：

\-------他日江湖相逢 再当杯酒言欢-------