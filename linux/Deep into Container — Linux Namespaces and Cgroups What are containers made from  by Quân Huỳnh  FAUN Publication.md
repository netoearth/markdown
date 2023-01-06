If we do DevOps, we are probably familiar with Kubernetes, Docker, and Containers. But have we ever wondered what the hell is docker? What are containers? Docker is a container? Docker is not a container and I will explain what it is in this post.

![](https://miro.medium.com/max/700/0*o-tX5NYWCI_725Bt.png)

## Container

Containers are a technology that allows us to run the process in an independent environment with other processes on the same computer. So how does the container do that?

To do that, the container is built from a few new features of the Linux kernel, of which the two main features are “namespaces” and “cgroups”.

## Linux Namespaces

This is a feature of Linux that allows us to create something like a virtual machine, quite similar to the function of virtual machine tools. This main feature makes our process completely separate from the other processes.

Linux namespaces are a bunch of different kinds:

-   The PID namespace allows us to create separate processes.
-   The networking namespace allows us to run the program on any port without conflict with other processes running on the same computer.
-   Mount namespace allows you to mount and unmount the filesystem without affecting the host filesystem.

Creating a Linux namespace is quite simple, we use a package called `unshare` to create a separate process.

```
sudo unshare --fork --pid --mount-proc bash
```

It will create a separate process and assign the bash shell to it.

```
root@namespace:~#
```

Try running the `ps` CLI.

```
root@namespace:~# ps auxUSER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMANDroot         1  0.0  0.0  23104  4852 pts/0    S    21:54   0:00 bashroot        12  0.0  0.0  37800  3228 pts/0    R+   21:57   0:00 ps aux
```

We will see it has only two processes is running, `bash` and `ps`.

You open another terminal on the server and type the command `ps aux`.

```
hmquan@server:~$ ps auxUSER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND...root        43  0.0  0.0   7916   828 pts/0    S    21:54   0:00 unshare --fork --pid --mount-proc bash...
```

You will see the `unshare` process is running, you can think it similar to the containers listed when you run the `docker ps` command.

To exit the namespace, type `exit`.

```
root@namespace:~# exit
```

Now when you run the `ps aux` command again on the server, we will see that the `unshare` process earlier is gone.

## Cgroups

We could have created a process separate from the other process with Linux namespaces. But if we create multiple namespaces, then how can we limit the resources of each namespace so that it doesn’t take up the resources of another namespace?

Luckily, in 2007 some people developed [cgroups](https://en.wikipedia.org/wiki/Cgroups) just for us. This is a Linux feature that allows you to limit the resources of a process. Cgroups will determine the limit of CPU and Memory that a process can use. To create cgroup, we will use `cgcreate`.

Before using `cgcreate`, we need to install `cgroup-tools`.

Ubuntu and Debian.

```
sudo apt-get install cgroup-tools
```

CentOS.

```
sudo yum install libcgroup
```

Then, we run the following command to create cgroup.

```
sudo cgcreate -g memory:my-process
```

One folder is created at the path /sys/fs/cgroup/memory.

```
$ ls /sys/fs/cgroup/memory/my-processcgroup.clone_children               memory.memsw.failcntcgroup.event_control                memory.memsw.limit_in_bytescgroup.procs                        memory.memsw.max_usage_in_bytesmemory.failcnt                      memory.memsw.usage_in_bytesmemory.force_empty                  memory.move_charge_at_immigratememory.kmem.failcnt                 memory.oom_controlmemory.kmem.limit_in_bytes          memory.pressure_levelmemory.kmem.max_usage_in_bytes      memory.soft_limit_in_bytesmemory.kmem.tcp.failcnt             memory.statmemory.kmem.tcp.limit_in_bytes      memory.swappinessmemory.kmem.tcp.max_usage_in_bytes  memory.usage_in_bytesmemory.kmem.tcp.usage_in_bytes      memory.use_hierarchymemory.kmem.usage_in_bytes          notify_on_releasememory.limit_in_bytes               tasksmemory.max_usage_in_bytes
```

We will see a lot of files, these are the files that define the limit of the process, the file that we are interested in now is `memory.kmem.limit_in_bytes`, it will define the memory limit of a process, and the value used in bytes.

For example, we will create a process have a memory limit is 50Mi.

```
sudo echo 50000000 >  /sys/fs/cgroup/memory/my-process/memory.limit_in_bytes
```

Then, run the following command to use cgroup.

```
hmquan@server:~$ sudo cgexec -g memory:my-process bashroot@cgroup:~#
```

Now the process created by cgroup will have a memory limit is 50Mi.

## Cgroups with namespace

And now, we can use cgroups with namespaces to create an independent process and limit the resources it can use.

For example.

```
hmquan@server:~$ sudo cgexec -g cpu,memory:my-process unshare -uinpUrf --mount-proc sh -c "/bin/hostname my-process && chroot mktemp -d /bin/sh"
```

Result.

```
root@my-process:~# echo "Hello from in a container"Hello from in a container
```

So, the container is a combination of two features cgroups and namespace, although in fact, it may have some other things, basically cgroups and namespace are the two main features.

## What is docker

So what is docker? Docker is just a tool to help us interact with the underlying container technology. To be more precise, Docker helps us easily create containers instead of having to do many things.

And to interact with the containers, Docker uses the container runtime. We will talk about it in the next post.

## Conclusion

So we have learned about what containers are built of. If you have any questions or need further clarification, you can ask in the comment section below.

If you like my article you can support me by [buying me a coffee](https://www.buymeacoffee.com/hmquan08017) ☕️, thank you.

![](https://miro.medium.com/max/700/0*mKF735hj2ij2cAY4.png)

## If this post was helpful, please click the clap 👏 button below a few times to show your support for the author 👇

## 🚀Developers: Learn and grow by keeping up with what matters, [JOIN FAUN.](https://faun.to/8zxxd)