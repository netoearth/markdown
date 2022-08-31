共享内存是一个非常有意思的话题，一方面共享内存避免了通讯过程中的内存复制问题，是 Linux IPC 通讯中效率最高的一种。另一方面，因为可以直接对内存甚至其他进程的内存进行修改，利用共享内存可以实现一些常规操作无法做到的奇技淫巧。

从使用方式上讲，Linux 提供了三种共享内存的方式，包括 Unix 味的 POSIX 和 SysV 接口，还提供了直接文件映射内存的 mmap。

本文尝试分别介绍 Linux 共享内存的基本原理，并做一个 “违背祖宗的决定”，如何在 Golang 中使用共享内存🐶。

> Do not communicate by sharing memory, instead, share memory by communicating.
> 
> Golang 是通过通讯代替共享内存的优雅代表，下文仅做试验，不建议日常使用

## mmap

mmap 是 POSIX 规范中的文件映射内存的方法，Linux 并提供了同名系统调用。是一个简单方便的 low level api。

mmap 的参数多样，使用姿势也十分广泛，从本质上讲，mmap 是将一个文件映射到进程的内存中，搭配了共享和私有，文件和匿名两个参数，可以提供非常多样的能力。

![](https://blog.cdn.updev.cn/2021-11-08-mmap.png)

mmap 在使用时可以指定文件或者匿名，指定文件时会在合适的时机把对应文件的指定部分加载到内存中，程序可以直接在内存中获取文件内容；在进行了写操作之后，内核会寻找合适的时机将更改更新到磁盘（脏页落盘）。因此 mmap 也可以使用作一个文件的快速读写。

而如果未指定文件，则 mmap 会默认使用 `/dev/zero` 文件，利用了该设备文件的仅读取出 0 字节流，而且任何写操作均被丢弃的特性。可以利用该特性申请一块被初始化过的内存使用， `malloc` 的内存申请也有 mmap 的功劳。

除了文件的选项，打开文件时，也可以选择打开的方式为共享或者私有，只有在共享模式下，两个进程打开同一个文件才可以视作共享内存。

![](https://blog.cdn.updev.cn/2021-11-08-mmap-scope.png)

mmap 的实现也并不复杂，众所周知 Linux 通过虚拟内存做了内存的虚拟化，进程看到的并不是实际的内存地址，要通过页表等中间层的翻译才能对物理地址进行访问，通过 VMA（`vm_area_struct`）实现了虚拟内存和实际对应的内存段，并与进程相关联。

![](https://blog.cdn.updev.cn/2022-05-29-133853.jpg)

在 Golang 中，`golang.org/x/sys/unix` 提供了 unix 底层 API，下面以这个库展示使用 mmap 共享内存。

首先定义一个简单的 Struct 用来表示共享的内存段，为了方便，数据在字节类型的 `data` 字段中。当然也可以使用 `unsafe.Pointer` 用作 Object 操作。

```
type ShardData struct {
id   int
data []byte
}
```

`unix` 提供了 `Mmap/Munmap` 方法，对某个文件进行映射，可包装工具方法如下：

```
func AttachShmWithMmap(f *os.File, offset int64, length int, prot int, flags int) (ShardData, error) {
data, err := unix.Mmap(int(f.Fd()), offset, length, prot, flags)
if err != nil {
return ShardData{}, err
}
return ShardData{data: data}, nil
}

func DetachShmWithMmap(data ShardData) error {
return unix.Munmap(data.data)
}
```

可运行的组装示例代码：

```
f, err := os.OpenFile("./block", os.O_RDWR, 0666)
if err != nil {
panic(err)
}
defer f.Close()

initData := make([]byte, 1024)
if _, err = f.Write(initData); err != nil {
panic(err)
}

data, err := AttachShmWithMmap(f, 0, 1024, unix.PROT_WRITE|unix.PROT_READ, unix.MAP_SHARED)
if err != nil {
panic(err)
}
defer DetachShmWithMmap(data)
```

## SysV

SystemV， 曾经也被称为 AT&T System V，是 Unix 操作系统众多版本中的一支，也是 Unix 标准的来(fen)源(cha)之一。

和 `mmap` 不同，SysV 为每一块内存设定以一个唯一的 int 类型 key，可以用个使用相同的 Key 获取同一个内存段。只要两个程序使用了相同 key，便可以实现内存段的共享。

SysV 的主要 Api 是四个函数：

1.  `shmget`：创建一个新的共享内存外，也可用于打开一个已存在的共享内存
2.  `shmat`：使用前，附加（attach）内存到进程的地址空间中
3.  `shmdt`：使用后，使共享内存区域与该进程的地址空间分离（detach）
4.  `shmctl`：共享内存控制（加锁、删除等）

![](https://blog.cdn.updev.cn/2022-05-29-CleanShot%202022-05-29%20at%2021.46.15%402x.png)

golang 的 `unix` 库中也提供了 SysV 的使用方式，只需要简单的改下 Attach 和 Detach 函数即可。

```
func AttachShmWithSysV(key, size, flag int) (ShardData, error) {
id, err := unix.SysvShmGet(key, size, flag)
if err != nil {
return ShardData{}, err
}

data, err := unix.SysvShmAttach(id, 0, 0)
if err != nil {
return ShardData{}, err
}

return ShardData{id: id, data: data}, nil
}

func DetachShmWithSysV(data ShardData) error {
if err := unix.SysvShmDetach(data.data); err != nil {
return err
}

_, err := unix.SysvShmCtl(data.id, unix.IPC_RMID, nil)
if err != nil {
return err
}
return nil
}
```

SysV 的共享内存实现是基于 tmpfs，tmpfs 是一个常用的基于内存的 fs，当在 tmpfs 中读写时，fs 中的内容会保存在内存中，当然，内容并非持久的，重启就没了。

系统中没有 mount tmpfs？没关系，内核在初始化时，会自动 mount 一个不可见的 tmpfs，挂载为 shm\_mnt，用来分配共享内存：

```
static struct file_system_type shmem_fs_type = {
        .owner          = THIS_MODULE,
        .name           = "tmpfs",
        .init_fs_context = shmem_init_fs_context,
#ifdef CONFIG_TMPFS
        .parameters     = shmem_fs_parameters,
#endif
        .kill_sb        = kill_litter_super,
        .fs_flags       = FS_USERNS_MOUNT,
};

int __init shmem_init(void)
{
        int error;

        shmem_init_inodecache();

        error = register_filesystem(&shmem_fs_type);
        if (error) {
                pr_err("Could not register tmpfs\n");
                goto out2;
        }

        shm_mnt = kern_mount(&shmem_fs_type);
        if (IS_ERR(shm_mnt)) {
                error = PTR_ERR(shm_mnt);
                pr_err("Could not kern_mount tmpfs\n");
                goto out1;
        }

#ifdef CONFIG_TRANSPARENT_HUGEPAGE
        if (has_transparent_hugepage() && shmem_huge > SHMEM_HUGE_DENY)
                SHMEM_SB(shm_mnt->mnt_sb)->huge = shmem_huge;
        else
                shmem_huge = SHMEM_HUGE_NEVER; /* just in case it was patched */
#endif
        return 0;

out1:
        unregister_filesystem(&shmem_fs_type);
out2:
        shmem_destroy_inodecache();
        shm_mnt = ERR_PTR(error);
        return error;
}
```

## POSIX

POSIX 是 Unix 世界中最流行的应用编程接口。应用编程接口是应用程序和系统调用之间的中间层。

POSIX 共享内存一定程度上也是为了弥补上述两种共享内存的不足：

1.  SysV 使用了一个黑盒的（不可见的 tmpfs）的唯一 key，这点和基于文件描述符的 Unix IO 模型不符
2.  mmap 虽然方便，但是却需要创建一个多进程共享的文件，如果共享的内容无持久化需求，这白白浪费了 IO 资源

POSIX 通过折中的方式进行解决，就是使用挂载于 `/dev/shm` 目录下的专用 tmpfs 文件系统（不再不可见），而且使用文件描述符，利用 `mmap` 对内存进行映射（或说 attach）：

```
$ mount
...
tmpfs on /dev/shm type tmpfs (rw,nosuid,nodev)
...
```

和 SysV 直接操作内存段不同，POSIX 共享内存基本分为两步，一是通过 `shm_open` 打开一段内存（返回文件描述符），然后基于该文件描述符，通过 `mmap` 映射到内存中使用。

因此 POSIX 共享内存的接口也比 SysV 少一半，只有创建和打开的 `shm_open` 和删除的 `shm_unlink` 。

![](https://blog.cdn.updev.cn/2022-05-29-133943.jpg)

`shm_open` 是对 `open` 的简单包装，在打开内存段时，需要传入一个 name，和 SysV 的 key 类似，两个进程使用相同的 name 即可使用相同的内存段。

而 `shm_open` 时传入的 name，经过处理，会被指向 `/dev/shm` 下的一个文件：

```
/* The directory that contains shared POSIX objects.  */
#define SHMDIR _PATH_DEV "shm/"

int
__shm_get_name (struct shmdir_name *result, const char *name, bool sem_prefix)
{
  while (name[0] == '/')
    ++name;
  size_t namelen = strlen (name);

  struct alloc_buffer buffer
    = alloc_buffer_create (result->name, sizeof (result->name));
    
  alloc_buffer_copy_bytes (&buffer, SHMDIR, strlen (SHMDIR));
  
  if (sem_prefix)
    alloc_buffer_copy_bytes (&buffer, "sem.", strlen ("sem."));
    
  alloc_buffer_copy_bytes (&buffer, name, namelen + 1);
  
  if (namelen == 0 || memchr (name, '/', namelen) != NULL
      || alloc_buffer_has_failed (&buffer))
    return -1;
    
  return 0;
}
```

虽然我没有找到 golang 官方库提供的 POSIX 共享内存接口，但由于 POSIX 共享内存的实现非常直白，直接在 `/dev/shm/` 目录下创建文件，使用 `mmap` 映射，就可以使 go 程序使用 POSIX 共享内存。

## 总结

从原理上讲 Linux 共享内存的主要方式只有两种，一是基于文件的 mmap，另一种就是 tmpfs，用一张图描述 Linux 几种实现共享内存的方式：  
![](https://blog.cdn.updev.cn/2022-05-29-134134.jpg)