This blog entry is about modern Linuxes. In other words RHEL6 equivalents with 2.6.3x kernels and not the ancient RHEL5 with 2.6.18 kernel (wtf?!), which is the most common in enterprises unfortunately. And no, I’m not going to use kernel debuggers or SystemTap scripts here, just plain old “cat /proc/PID/xyz” commands against some useful /proc filesystem entries.

#### Troubleshooting a “slow” process

Here’s one systematic troubleshooting example I _reproduced_ in my laptop. A DBA was wondering why their find command had been running “much slower”, without returning any results for a while. Knowing the environment, we had a hunch, but I got asked about what would be the systematic approach for troubleshooting this – _already ongoing_ – problem **_right now_**.

Luckily the system was running OEL6, so had a pretty new kernel. Actually the 2.6.39 UEK2.

So, let’s do some troubleshooting. First let’s see whether that _find_ process is still alive:

```
[root@oel6 ~]# ps -ef | grep find
root     27288 27245  4 11:57 pts/0    00:00:01 find . -type f
root     27334 27315  0 11:57 pts/1    00:00:00 grep find
```

Yep it’s there – PID 27288 (I’ll use that pid throughout the troubleshooting example).

Let’s start from the basics and take a quick look what’s the bottleneck for this process – if it’s not blocked by anything (for example reading everything it needs from cache) it should be 100% on CPU. If it’s bottlenecked by some IO or contention issues, the CPU usage should be lower – or completely 0%.

```
[root@oel6 ~]# top -cbp 27288
top - 11:58:15 up 7 days,  3:38,  2 users,  load average: 1.21, 0.65, 0.47
Tasks:   1 total,   0 running,   1 sleeping,   0 stopped,   0 zombie
Cpu(s):  0.1%us,  0.1%sy,  0.0%ni, 99.8%id,  0.0%wa,  0.0%hi,  0.0%si,  0.0%st
Mem:   2026460k total,  1935780k used,    90680k free,    64416k buffers
Swap:  4128764k total,   251004k used,  3877760k free,   662280k cached

  PID USER      PR  NI  VIRT  RES  SHR S %CPU %MEM    TIME+  COMMAND
27288 root      20   0  109m 1160  844 D  0.0  0.1   0:01.11 find . -type f
```

Top tells me this process is either 0% on CPU or very close to zero percent (so it gets rounded to 0% in the output). There’s an important difference though, as a process that is completely stuck, not having a chance of getting onto CPU at all vs. a process which is getting out of its wait state every now and then (for example some polling operation times out every now and then and thee process chooses to go back to sleep). So, top on Linux is not a good enough tool to show that difference for sure – but at least we know that this process is not burning serious amounts of CPU time.

Let’s use something else then. Normally when a process seems to be stuck like that (0% CPU usually means that the process is stuck in some blocking system call – which causes the kernel to put the process to sleep) I run `strace` on that process to trace in which system call the process is currently stuck. Also if the process is actually not completely stuck, but returns from a system call and wakes up briefly every now and then, it would show up in strace (as the blocking system call would complete and be entered again a little later):

```
[root@oel6 ~]# strace -cp 27288
Process 27288 attached - interrupt to quit

^C
^Z
[1]+  Stopped                 strace -cp 27288

[root@oel6 ~]# kill -9 %%
[1]+  Stopped                 strace -cp 27288
[root@oel6 ~]# 
[1]+  Killed                  strace -cp 27288
```

Oops, the strace command itself got hung too! It didn’t print any output for a long time and didn’t respond to CTRL+C, so I had to put it into background with CTRL+Z and kill it from there. So much for easy diagnosis.

Let’s try pstack then (on Linux, pstack is just a shell script wrapper around the `GDB` debugger). While pstack does not see into kernel-land, it will still give us a clue about which system call was requested (as usually there’s a corresponding libc library call in the top of the displayed userland stack):

```
[root@oel6 ~]# pstack 27288

^C
^Z
[1]+  Stopped                 pstack 27288

[root@oel6 ~]# kill %%
[1]+  Stopped                 pstack 27288
[root@oel6 ~]# 
[1]+  Terminated              pstack 27288
```

Pstack also got stuck without returning anything!

So we still don’t know whether our process is 100% (hopelessly) stuck or just 99.99% stuck (the process keeps going back to sleep) – and where it’s stuck.

Ok, where else to look? There’s one more commonly available thing to check – the process status and WCHAN fields, which can be reported via good old ps (perhaps I should have ran this command earlier to make sure this process isn’t a zombie by now):

```
[root@oel6 ~]# ps -flp 27288
F S UID        PID  PPID  C PRI  NI ADDR SZ WCHAN  STIME TTY          TIME CMD
0 D root     27288 27245  0  80   0 - 28070 rpc_wa 11:57 pts/0    00:00:01 find . -type f
```

You should run ps multiple times in a row to make sure that the process is still in the same state (you don’t want to get misled by a single sample taken at an unlucky time), but I’m displaying it here only once for brevity.

The process is in state D (uninterruptible sleep) which is usually disk IO related (as the ps man pages also say). And the WCHAN (the function which caused this sleep/wait in this process) field is a bit truncated. I could use a ps option (read the man) to print it out a bit wider, but as this info comes from the proc filesystem anyway, let’s query it directly from the source (again, it would be a good idea to sample this multiple times as we are not sure yet whether our process is completely stuck or just mostly sleeping):

```
[root@oel6 ~]# cat /proc/27288/wchan
rpc_wait_bit_killable

```

Hmm… this process is waiting for some RPC call. RPC usually means that the process is talking to other processes (either in the local server or even remote servers). But we still don’t know why.

#### Is there any movement or is the process completely stuck?

Before we go on to the “meat” of this article, let’s figure out whether this process is completely stuck or not. The /proc/PID/status can tell us that on modern kernels. I have highlighted the interesting bits for our purpose:

```
[root@oel6 ~]# cat /proc/27288/status 
Name:find
State:D (disk sleep)
Tgid:27288
Pid:27288
PPid:27245
TracerPid:0
Uid:0000
Gid:0000
FDSize:256
Groups:0 1 2 3 4 6 10 
VmPeak:  112628 kB
VmSize:  112280 kB
VmLck:       0 kB
VmHWM:    1508 kB
VmRSS:    1160 kB
VmData:     260 kB
VmStk:     136 kB
VmExe:     224 kB
VmLib:    2468 kB
VmPTE:      88 kB
VmSwap:       0 kB
Threads:1
SigQ:4/15831
SigPnd:0000000000040000
ShdPnd:0000000000000000
SigBlk:0000000000000000
SigIgn:0000000000000000
SigCgt:0000000180000000
CapInh:0000000000000000
CapPrm:ffffffffffffffff
CapEff:ffffffffffffffff
CapBnd:ffffffffffffffff
Cpus_allowed:ffffffff,ffffffff
Cpus_allowed_list:0-63
Mems_allowed:00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000001
Mems_allowed_list:0
voluntary_ctxt_switches:9950
nonvoluntary_ctxt_switches:17104

```

The process is in the state D – Disk Sleep (uninterruptible sleep). And look into the number of `voluntary_ctxt_switches` and `nonvoluntary_ctxt_switches` – this tells you how many times the process was taken off CPU (or put back). Then run the same command again a few seconds later and see if those numbers increase. In my case these numbers did not increase, thus I could conclude that the process was completely stuck (well, at least during the few seconds between these commands). So I can be more confident now that this process is completely stuck (and it’s not just flying under the radar by burning 0.04% of CPU all time).

By the way, there are two more locations where to get the context switch counts (and the 2nd one works on ancient kernels as well):

```
[root@oel6 ~]# cat /proc/27288/sched
find (27288, #threads: 1)
---------------------------------------------------------
se.exec_start                      :     617547410.689282
se.vruntime                        :       2471987.542895
se.sum_exec_runtime                :          1119.480311
se.statistics.wait_start           :             0.000000
se.statistics.sleep_start          :             0.000000
se.statistics.block_start          :     617547410.689282
se.statistics.sleep_max            :             0.089192
se.statistics.block_max            :         60082.951331
se.statistics.exec_max             :             1.110465
se.statistics.slice_max            :             0.334211
se.statistics.wait_max             :             0.812834
se.statistics.wait_sum             :           724.745506
se.statistics.wait_count           :                27211
se.statistics.iowait_sum           :             0.000000
se.statistics.iowait_count         :                    0
se.nr_migrations                   :                  312
se.statistics.nr_migrations_cold   :                    0
se.statistics.nr_failed_migrations_affine:                    0
se.statistics.nr_failed_migrations_running:                   96
se.statistics.nr_failed_migrations_hot:                 1794
se.statistics.nr_forced_migrations :                  150
se.statistics.nr_wakeups           :                18507
se.statistics.nr_wakeups_sync      :                    1
se.statistics.nr_wakeups_migrate   :                  155
se.statistics.nr_wakeups_local     :                18504
se.statistics.nr_wakeups_remote    :                    3
se.statistics.nr_wakeups_affine    :                  155
se.statistics.nr_wakeups_affine_attempts:                  158
se.statistics.nr_wakeups_passive   :                    0
se.statistics.nr_wakeups_idle      :                    0
avg_atom                           :             0.041379
avg_per_cpu                        :             3.588077
nr_switches                        :                27054
nr_voluntary_switches              :                 9950
nr_involuntary_switches            :                17104
se.load.weight                     :                 1024
policy                             :                    0
prio                               :                  120
clock-delta                        :                   72
```

Here you need to look into the `nr_switches` number (which equals `nr_voluntary_switches` + `nr_involuntary_switches`).

The total nr\_switches is 27054 in the above output and this is also what the 3rd field in /proc/PID/schedstat shows:

```
[root@oel6 ~]# cat /proc/27288/schedstat 
1119480311 724745506 27054

```

And it doesn’t increase…

#### Peeking into kernel-land using /proc filesystem

So, it looks like our process is pretty stuck :) Strace and pstack were useless. They use ptrace() system call for attaching to the process and peeking into its memory, but as the process was hopelessly stuck, very likely in some system call, so I guess the ptrace() call got hung itself. (BTW, I once tried stracing the strace process itself when attaching to a target process and it crashed the target. You have been warned :)

So, how to see in which system call are we stuck – without strace or pstack? Luckily I was running on a modern kernel – say hi to /proc/PID/syscall!

```
[root@oel6 ~]# cat /proc/27288/syscall 
262 0xffffffffffffff9c 0x20cf6c8 0x7fff97c52710 0x100 0x100 0x676e776f645f616d 0x7fff97c52658 0x390e2da8ea
```

Ok WTF am I supposed to do with that?

Well, these numbers usually stand for something. If it’s “0xAVeryBigNumber” then it’s usually a memory address (and pmap-like tools could be used where they point), but if it’s a small number then perhaps it’s an index into some array – like open file descriptor array (which you can read from `/proc/PID/fd`) or in this case, as we are dealing with system calls – it’s the system call number where your process is currently in! So, this process is stuck in system call #262!

Note that the system call numbers may differ between OS type, OS version and platform, so you should read it from the proper .h file on your flavor of OS. Usually searching for “syscall\*” from /usr/include is a good starting point. On my version and platform (64bit) of Linux the system calls are defined here in `/usr/include/asm/unistd_64.h`:

```
[root@oel6 ~]# grep 262 /usr/include/asm/unistd_64.h 
#define __NR_newfstatat262

```

Here you go! The syscall 262 is something called `newfstatat`. Just run man to find out what it is. Here’s a hint regarding system call names – if man page doesn’t find a call, then try it without possible suffixes and prefixes (for example “man pread” instead of “man pread64”) – in this case, run man without the “new” prefix – `man fstatat`. Or just google.

Anyway, this system call “new-fstat-at” allows you to read file properties pretty much like the usual `stat` system call. And we are stuck in this file metadata reading operation. So we’ve gotten one step further. But we still don’t know why are we stuck there?

Well, [Say Hello To My Little Friend](https://www.youtube.com/watch?v=a_z4IuxAqpE) **/proc/PID/stack**, which allows you to read kernel stack backtraces of a process by just cat’ing a proc file!!!

```
[root@oel6 ~]# cat /proc/27288/stack
[] rpc_wait_bit_killable+0x24/0x40 [sunrpc]
[] __rpc_execute+0xf5/0x1d0 [sunrpc]
[] rpc_execute+0x43/0x50 [sunrpc]
[] rpc_run_task+0x75/0x90 [sunrpc]
[] rpc_call_sync+0x42/0x70 [sunrpc]
[] nfs3_rpc_wrapper.clone.0+0x35/0x80 [nfs]
[] nfs3_proc_getattr+0x47/0x90 [nfs]
[] __nfs_revalidate_inode+0xcc/0x1f0 [nfs]
[] nfs_revalidate_inode+0x36/0x60 [nfs]
[] nfs_getattr+0x5f/0x110 [nfs]
[] vfs_getattr+0x4e/0x80
[] vfs_fstatat+0x70/0x90
[] sys_newfstatat+0x24/0x50
[] system_call_fastpath+0x16/0x1b
[] 0xffffffffffffffff
```

The top function is where in kernel code we are stuck right now – which is exactly what the WCHAN output showed us (note that actually there are some more functions on top of it, such as the kernel `schedule()` function which puts processes to sleep or on CPU as needed, but these are not shown here probably because they’re a result of this wait condition itself, not the cause).

Thanks to the full kernel-land stack of this task we can read it from bottom up to understand how did we end up calling this `rpc_wait_bit_killable` function in the top, which ended up calling the scheduler and put us into sleep mode.

The `system_call_fastpath` in the bottom is a generic kernel system call handler function which has called the kernel code for that newfstatat syscall we issued (`sys_newfstatat`). And then, continuing to go upwards through the “child” functions we end up seeing a bunch of NFS functions in the stack! This is a 100% undeniable proof that we are somewhere under NFS codepath. I’m not saying “in NFS codepath” as when you continue reading upwards past the NFS functions, you see that the topmost NFS function has in turn called some RPC function (rpc\_call\_sync) for talking to some other process – in this case probably the `[kworker/N:N]`, `[nfsiod]`, `[lockd]` or `[rpciod]` kernel IO threads. And we are never getting a reply from those threads for some reason (the usual suspect would be dropped network connection, packets or just network connectivity issues).

To see whether any of the helper threads are stuck on network-related code, you could sample their kernel stacks too, although the kworkers do much more than just NFS RPC communication. During a separate experiment (just copying large files over NFS), I have caught one of the kworkers waiting in networking code:

```
[root@oel6 proc]# for i in `pgrep worker` ; do ps -fp $i ; cat /proc/$i/stack ; done
UID        PID  PPID  C STIME TTY          TIME CMD
root        53     2  0 Feb14 ?        00:04:34 [kworker/1:1]

[] __cond_resched+0x2a/0x40
[] lock_sock_nested+0x35/0x70
[] tcp_sendmsg+0x29/0xbe0
[] inet_sendmsg+0x48/0xb0
[] sock_sendmsg+0xef/0x120
[] kernel_sendmsg+0x41/0x60
[] xs_send_kvec+0x8e/0xa0 [sunrpc]
[] xs_sendpages+0x173/0x220 [sunrpc]
[] xs_tcp_send_request+0x5d/0x160 [sunrpc]
[] xprt_transmit+0x83/0x2e0 [sunrpc]
[] call_transmit+0xa8/0x130 [sunrpc]
[] __rpc_execute+0x66/0x1d0 [sunrpc]
[] rpc_async_schedule+0x15/0x20 [sunrpc]
[] process_one_work+0x13e/0x460
[] worker_thread+0x17c/0x3b0
[] kthread+0x96/0xa0
[] kernel_thread_helper+0x4/0x10
```

It should be possible to enable kernel tracing for knowing which exact kernel thread communicates with other kernel threads, but I won’t go in there in this post – as it’s supposed to be a practical (and simple :) troubleshooting exercise!

#### Diagnosis and “fix”

Anyway, thanks to very easy kernel stack sampling in modern Linux kernels (I don’t know exact version when it was introduced) we managed to systematically figure out where the find command was stuck – in NFS code in the Linux kernel. And when you have NFS-related hangs, the most usual suspects are network issues. If you are wondering how did I reproduce this problem, I mounted a NFS volume from a VM, started the find command and then just suspended the VM. This produces similar symptoms as a network (configuration, firewall) issue where a connection is silently dropped without notifying the TCP endpoints or packets just don’t go through for some reason.

As the top function listed in the stack was one of the “killable” safe-to-kill functions (`rpc_wait_bit_killable`) then we can kill it with `kill -9`:

```
[root@oel6 ~]# ps -fp 27288
UID        PID  PPID  C STIME TTY          TIME CMD
root     27288 27245  0 11:57 pts/0    00:00:01 find . -type f
[root@oel6 ~]# kill -9 27288

[root@oel6 ~]# ls -l /proc/27288/stack
ls: cannot access /proc/27288/stack: No such file or directory

[root@oel6 ~]# ps -fp 27288
UID        PID  PPID  C STIME TTY          TIME CMD
[root@oel6 ~]#
```

The process is gone.

#### Poor-man’s kernel thread profiling

Note that as the /proc/PID/stack looks like just a plain text proc file, you can do [poor-man’s stack profiling](https://tanelpoder.com/2013/02/14/troubleshooting-high-cpu-usage-with-poor-mans-stack-profiler-in-a-one-liner/) also on the kernel threads! This is an example of sampling the current system call and the kernel stack (if in system call) and aggregating it into a semi-hierarchical profile in a poor-man’s way:

```
[root@oel6 ~]# export LC_ALL=C ; for i in {1..100} ; do cat /proc/29797/syscall | awk '{ print $1 }' ; cat /proc/29797/stack | /home/oracle/os_explain -k ; usleep 100000 ; done | sort -r | uniq -c 
     69 running
      1 ffffff81534c83
      2 ffffff81534820
      6 247
     25 180

    100    0xffffffffffffffff 
      1     thread_group_cputime 
     27     sysenter_dispatch 
      3     ia32_sysret 
      1      task_sched_runtime 
     27      sys32_pread 
      1      compat_sys_io_submit 
      2      compat_sys_io_getevents 
     27       sys_pread64 
      2       sys_io_getevents 
      1       do_io_submit 
     27        vfs_read 
      2        read_events 
      1        io_submit_one 
     27         do_sync_read 
      1         aio_run_iocb 
     27          generic_file_aio_read 
      1          aio_rw_vect_retry 
     27           generic_file_read_iter 
      1           generic_file_aio_read 
     27            mapping_direct_IO 
      1            generic_file_read_iter 
     27             blkdev_direct_IO 
     27              __blockdev_direct_IO 
     27               do_blockdev_direct_IO 
     27                dio_post_submission 
     27                 dio_await_completion 
      6                  blk_flush_plug_list
```

This gives you a very rough sample of where in kernel this process spent its time – if at all. The system call numbers are separated, in bold, above – “running” means that this process was in userland (not in a system call) during the sample. So, 69% of the time during the sampling, this process was running userland code. 25% of time in syscall #180 (nfsservctl on my system) and 6% in syscall #247 (waitid).

There are two more “functions” seen in this output – but for some reason they haven’t been properly translated to a function name. Well, this address must stand for something, so let’s try our luck manually:

```
[root@oel6 ~]# cat /proc/kallsyms | grep -i ffffff81534c83
ffffffff81534c83 t ia32_sysret
```

So it looks that these samples are about a 32-bit architecture-compatible system call return function – but as this function isn’t a system call itself (just an internal helper function), perhaps that’s why the /proc/stack routine didn’t translate it. Perhaps these addresses show up because there’s no “read consistency” on the /proc views, when owning threads modify these memory structures and entries, the reading threads may sometimes read data “in-flux”.

Let’ check the other address too:

```
[root@oel6 ~]# cat /proc/kallsyms | grep -i ffffff81534820
[root@oel6 ~]#
```

Nothing? Well, the troubleshooting doesn’t have to stop yet – let’s see if there’s anything interesting around that address. I just removed a couple of trailing characters from the address:

```
[root@oel6 ~]# cat /proc/kallsyms | grep -i ffffff815348 
ffffffff8153480d t sysenter_do_call
ffffffff81534819 t sysenter_dispatch
ffffffff81534847 t sysexit_from_sys_call
ffffffff8153487a t sysenter_auditsys
ffffffff815348b9 t sysexit_audit
```

Looks like the sysenter\_dispatch function starts just 1 byte before the original address displayed by /proc/PID/stack. So we had possibly already executed the one byte (possibly a NOP operation left in there for dynamic tracing probe traps). But nevertheless, looks like these stack samples were in the `sysenter_dispatch` function, which isn’t a system call itself, but a system call helper function.

#### More about stack profilers

Note that there are different types of stack samplers – the Linux Perf, Oprofile tools and Solaris DTrace’s `profile` provider do sample the instruction pointer (EIP on 32bit intel or RIP on x64) and stack pointer (ESP on 32bit and RSP on 64bit) registers of the currently running thread on CPU and walk the stack pointers backwards from there. Therefore, these tools only show threads which happen to be on CPU when sampling!!! This is of course perfect, when troubleshooting high CPU utilization issues, but not useful at all when troubleshooting hung processes or too long process sleeps/waits.

Tools like pstack on Linux,Solaris,HP-UX, procstack (on AIX), ORADEBUG SHORT\_STACK and just reading the /proc/PID/stack pseudofile provide a good addition (not replacement) to the CPU profiling tools – as they access the probed process memory regardless of their CPU scheduling state and read the stack from there. If the process is sleeping, not on CPU, the starting point of the stack can be read from the stored context – which is saved to kernel memory by OS scheduler during the context switch.

Of course, the CPU event profiling tools can often do much more than just a pstack, OProfile, Perf and even DTrace’s CPC provider (in Solaris11) can set up and sample CPU internal performance counters to estimate things like the amount of CPU cycles stalled waiting for main memory, amount of L1/L2 cache misses, etc. Better read what Kevin Closson has to say on these topics: ([Perf](http://kevinclosson.wordpress.com/2013/02/18/using-linux-perf1-to-analyze-database-processes-part-i/), [Oprofile](http://kevinclosson.wordpress.com/2006/12/14/using-oprofile-to-monitor-kernel-overhead-on-linux-running-oracle/))

#### Always-on sampling of thread states

-   Check out my open source [0x.tools](https://0x.tools/) toolset for always-on profiling of Linux production systems!