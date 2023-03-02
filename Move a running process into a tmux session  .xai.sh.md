It’s rare, but sometimes it still happens that I forget to open a tmux or screen session when working with something that is supposed to be quickly done. However, it also happens that “quickly done” turns into “tedious and ugly” and now the process lives longer than it was supposed to and I become afraid of ssh disconnects or something.

So an obvious solution is killing the process and running it in a newly created tmux session — but what if the process ran for a while and I don’t want to kill it because I either lose progress or end up in a mess? Instead of killing and re-running a process, it would be much smoother to just move it into a tmux session. This involves changing the parent of a process, which is not exactly trivial, but thankfully @nelhage made a tool for that: [reptyr](https://github.com/nelhage/reptyr). If you’re interested in how actually achieves its goal, check out his blog posts<sup id="fnref:1" role="doc-noteref" _mstmutation="1"><a href="https://xai.sh/2020/10/16/Move-running-process-into-tmux-session.html#fn:1" rel="footnote">1</a></sup><sup id="fnref:2" role="doc-noteref" _mstmutation="1"><a href="https://xai.sh/2020/10/16/Move-running-process-into-tmux-session.html#fn:2" rel="footnote">2</a></sup>!`reptyr`

As for usage, it is very easy:

1.  Suspend the respective process with `Ctrl-Z`
2.  Send the job to background using `bg`
3.  Take away the ownership from the shell using `disown`
4.  Start or enter your tmux/screen session
5.  Run to attach the process to the current shell`reptyr PID`

It also has some additional useful features, such as TTY-stealing, which is documented in the man page.

Before compiling , make sure to check whether it is in your distributions repository. At the time of writing, this was at least the case for Gentoo, Fedora, and Debian.`reptyr`

**Update (2023-01-05): ptrace scoping**  
A fellow Gentoo enthusiast noticed that on recent systems, the following error occurs when invoking as regular user:`reptyr`

```
Unable to attach to pid 1348999: Operation not permitted
The kernel denied permission while attaching. If your uid matches
the target's, check the value of /proc/sys/kernel/yama/ptrace_scope.
For more information, see /etc/sysctl.d/10-ptrace.conf
```

And this is how I learned about [ptrace scoping](https://www.kernel.org/doc/Documentation/security/Yama.txt), a feature that was added to the Linux kernel in version 3.4.

_The problem_:  
In order to attach to a process, uses the system call, which is used to debug processes. In order to prevent unprivileged users from attaching to processes, the kernel has a feature called “ptrace scoping” which allows you to restrict which users and processes can attach to which processes. The value of determines the scope of :`reptyr``ptrace``/proc/sys/kernel/yama/ptrace_scope``ptrace`

-   `0`: No restrictions, anything can be attached as long as the uids match.
-   `1`: Only the process owner and root can attach to a process. Also, some kind of relationship is required between the processes.
-   `2`: Only root can attach to a process.
-   `3`: No one can attach to a process.

On my Gentoo, it was set to by default, which seems reasonable. As there is no relationship between the process I want to move and the process, is denied the permission to attach via .`1``reptyr``reptyr``ptrace`

So what can we do about this? I assumed that I could use the system call to set to for the process I want to move. However, it seems that I can only set a tracer for the calling process, not for an arbitrary one, which I find rather annoying.`prctl``PR_SET_PTRACER``PR_SET_PTRACER_ANY`

If somebody finds out how to do this, I would be glad to include it here instead of the following “dirty” workaround.

_The workaround_:  
The only remaining option I found so far is temporarily setting to before invoking , which can be done by running . Note that this creates a security problem, as now _all_ processes can be attached. For security reasons, I prefer to reset it to right after the process has been re-attached.`ptrace_scope``0``reptyr``sysctl -w kernel.yama.ptrace_scope=0``1`

___