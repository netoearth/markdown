Terminals come to us from the history of `UNIX` systems. Basically, terminals provided an API for the console utils (physical ones!) to generalize interaction with users. It includes ways of reading input and writing to it in **two** modes:

-   the **canonical** mode (default) – input is buffered line by line and read into after a new line char `\n` occurs;
-   the **noncanonical** mode – an application can read terminal input a character at a time. For example `vi`, `emacs` and `less` use this mode.

Nowadays, with the widespread use of rich graphical UIs, the significance of the terminals are lesser than it was, but still, we use this protocol implicitly every time we start an ssh connection.

The are a bunch of files under `/dev/` directory that represents different types of terminals:

-   `/dev/tty*` – physical consoles;
-   `/dev/ttyS*` – serial connections;
-   `/dev/pts/*` – pseudoterminals.

Also, the `/proc/tty/drivers` file contains other supported drivers.

So, in order to determine what terminal file the current shell session is using, we have a `tty` cli tool (`[man 1 tty](https://man7.org/linux/man-pages/man1/tty.1.html)`).

On my remote ssh connection:

For a physical console:

We can also open a virtual device `/dev/tty` to get a `fd` of the controlling terminal if it exists for the current process.

## Pseudoterminal (`devpts`) [#](https://biriukov.dev/docs/fd-pipe-session-terminal/4-terminals-and-pseudoterminals/#pseudoterminal-devpts)

In order to make it possible to use a terminal remotely, the Linux kernel provides a feature called pseudoterminal or `devpts` ([https://www.kernel.org/doc/html/latest/filesystems/devpts.html)](https://www.kernel.org/doc/html/latest/filesystems/devpts.html%29).

It allows us to build terminal emulators and use them instead of a real terminal, where an application expects a terminal device. Using pseudoterminals we can build terminal proxies, record screen sessions and mock user input. You can think about pseudoterminal like as a special type of Inter-process communication (IPC). It’s a bidirectional communication channel. All operations that can be applied to a terminal device can also be applied to a `pts` device end, including something that doesn’t make sense. For example: changing the speed of connection transforms into a no-op internally.

Pseudoterminal consists of 2 parts:

1.  A `ptmx` part which is a leader for the pseudoterminal. This end is used to emulate the user input and read back the program output.
2.  A `pts` is a secondary end. This part is given to an application that needs a terminal.

The following image shows how an ssh client uses pseudoterminals to establish remote access.

![ssh client, sshd server and two pairs of pseudoterminals](https://biriukov.dev/docs/fd-pipe-session-terminal/images/sshd-pseudo-terminal.png)

Image 4. – `ssh` client, `sshd` server and two pairs of pseudoterminals

-   ❶ – We usually have a graphical UI on our local host and use some kind of `xterm` to run a local console. The UI subsystem receives all keyboard inputs, so it opens a pseudoterminal and redirects it into its `ptmx` device. The other side of the terminal is what an `xterm` emulator sees.
-   ❷ – The local `bash` process creates a new foreground process group (job). It’s running in the foreground, so a `ssh` gets full control of the terminal. `ssh` client is a special terminal program that leverages the full power of terminals. It sets the terminal into the raw mode. Thus, the future `CTRL-C` and `CTRL-Z` combinations do not affect the local ssh process. Instead, all such commands will be sent to the remote side of the ssh connection and interpreted there. When the `ssh` client exits, it returns all settings back.
-   ❸, ❼ – The communication between the `ptmx` and `pts` happens in the kernel and is hidden from our eyes.
-   ❹ – `sshd` server is listening to new connections. It makes a `fork` for each connected user and checks their credentials.
-   ❺ – The `sshd` process creates a new pseudoterminal pair. It basically connects the `ptmx` side and the client tcp socket.
-   ❻ – Then the `sshd` process makes a `fork()`, opens a corresponding new `pts`, starts a new session (`setsid()`), opens our `pts` making it the controlling terminal of the session and duplicates standard file descriptors 0,1 and 2 with the `pts` descriptor. Now it’s ready to call `execve()` to start `bash` shell.

Let’s emulate the above with a small example. We are creating a new session with a new pseudoterminal pair and write the `stdin` into a file on disk.

```
import os
import time
import sys

print(f"parent: {os.getpid()}")

ptmx, secondary = os.openpty()

pid = os.fork()
if not pid:
print(f"child: {os.getpid()}")
os.close(ptmx)
os.setsid()
name = os.ttyname(secondary)
print(name)
s = os.open(name, os.O_RDWR)
os.dup2(s, 0)
os.dup2(s, 1)
os.dup2(s, 2)
os.close(secondary)
os.close(s)
with open('/tmp/file.txt', "w") as f:
    for l in sys.stdin:
        f.write(l)
        f.flush()
time.sleep(999999)
else:
os.close(secondary)
os.write(ptmx, b"text\n")
os.waitpid(-1,0)
```

Run it:

```
$ python3 ./terminal.py
parent: 8776
child: 8777
/dev/pts/3
```

Check the file:

File descriptors show that all is good:

```
$ ls -l  /proc/8776/fd
lrwx------ 1 vagrant vagrant 64 Jul 12 21:45 0 -> /dev/pts/0
lrwx------ 1 vagrant vagrant 64 Jul 12 21:45 1 -> /dev/pts/0
lrwx------ 1 vagrant vagrant 64 Jul 12 21:45 2 -> /dev/pts/0
lrwx------ 1 vagrant vagrant 64 Jul 12 21:45 3 -> /dev/ptmx
```

```
$ ls -l  /proc/8777/fd
lrwx------ 1 vagrant vagrant 64 Jul 12 21:45 0 -> /dev/pts/3
lrwx------ 1 vagrant vagrant 64 Jul 12 21:45 1 -> /dev/pts/3
lrwx------ 1 vagrant vagrant 64 Jul 12 21:45 2 -> /dev/pts/3
l-wx------ 1 vagrant vagrant 64 Jul 12 21:45 3 -> /tmp/file.txt
```

## Terminal settings [#](https://biriukov.dev/docs/fd-pipe-session-terminal/4-terminals-and-pseudoterminals/#terminal-settings)

The `ptmx` and `pts` devices share terminal attributes (`termios`) and window size (`winsize`) structures.

The current setting of a terminal can be obtained and updated by the `stty` command:

As we discussed earlier, the background jobs can print to the `stdout` by default. However, we can change it by setting `TOSTOP` flag for the terminal. If we do that, the background process group will receive a `SIGTTOU` signal from the kernel. The default handler for this signal is stop.

And run some background job:

```
$ yes | grep y &
[1] 10694
[vagrant@archlinux post2]$ jobs -l
[1]+ 10693 Stopped (tty output)yes
 10694                   | grep y
```

And return it back:

We also can return back to the default setting by using the “sane” parameter:

## Handling Terminal Signals [#](https://biriukov.dev/docs/fd-pipe-session-terminal/4-terminals-and-pseudoterminals/#handling-terminal-signals)

As we discussed, the kernel can send some terminal signals to foreground and background processes. Some of them we already touched:

-   `SIGTTIN` – a background process tried to read from a terminal.
-   `SIGTTOU` – a background process tries to write to a terminal when the `tostop` flag is set or a background process asks to send it to the foreground.
-   `SIGTSTP` – a default response to a `CTRL-Z` pressed combination.

The noncanonical programs such as `vi`, `emacs` and `less`, need to handle all the above signals in order to reset terminal settings back and forth, redraw a terminal content and place the cursor in the right place.

Another interesting terminal signal we haven’t seen is the `SIGWINCH` signal. A foreground process receives it when size of a terminal window has changed. Usually, a program uses `ioctl()` with the `TIOCGWINSZ` operation to get the current size in its signal handler. For example:

```
import time
import signal
import termios
import fcntl
import struct

def signal_handler(signum, frame):
   packed = fcntl.ioctl(0, termios.TIOCGWINSZ, struct.pack('HHHH', 0, 0, 0, 0))
   rows, cols, h_pixels, v_pixels = struct.unpack('HHHH', packed)
   print(rows, cols, h_pixels, v_pixels)

signal.signal(signal.SIGWINCH, signal_handler)

time.sleep(9999)
```

And if you start it and play with size of terminal window:

```
$ python3 ./size.py
32 85 1360 1280
32 72 1152 1280
32 74 1184 1280
32 86 1376 1280
32 83 1328 1280
33 73 1168 1320
```

## `screen` and `tmux` [#](https://biriukov.dev/docs/fd-pipe-session-terminal/4-terminals-and-pseudoterminals/#screen-and-tmux)

`screen` (`[man 1 screen](https://man7.org/linux/man-pages/man1/screen.1.html)`) and `tmux` (`[man 1 tmux](https://man7.org/linux/man-pages/man1/tmux.1.html)`) are usually used for protecting shell sessions between connections. It is also widely used for long-running jobs and better ssh user client experience. Both use pseudoterminals to multiplex a single physical terminal (or terminal window) between multiple processes (multiple shell sessions). In this section, we will talk about `tmux`, but the `screen` is almost the same in all discussed topics here.

On the first start, `tmux` starts a server with a set of `ptmx` corresponding to its panes. Clients of `tmux` (`tmux attach`) use a default unix socket to find and connect to the server:

```
$ ls -lad /tmp/tmux-1000/default
srwxrwx--- 1 vagrant vagrant 0 Jul 14 12:57 /tmp/tmux-1000/default
```

where `1000` is a user id `UID`.

`tmux` doesn’t do any `tcsetpgrp()` calls, because any panel or window creates a new pair of terminals.

So let’s demonstrate it. After we `ssh` to a box, we have our bash with `PID` 11761 :

```
$ ls -la /proc/$$/fd
lrwx------ 1 vagrant vagrant 64 Jul 14 12:08 0 -> /dev/pts/0
lrwx------ 1 vagrant vagrant 64 Jul 14 12:08 1 -> /dev/pts/0
lrwx------ 1 vagrant vagrant 64 Jul 14 12:08 2 -> /dev/pts/0
```

Let assume, that we already have a working `tmux` session on the server. So if we check the `ps` command, we can see it with `PID` 11781.

```
...      
├─sshd(11619)───sshd(11749)───sshd(11760)───bash(11761)
...
└─tmux: server(11780)───bash(11781)
```

Now let’s attach to the `tmux` session:

We get a new bash `PID` from the above `ps` output and a new pseudoterminal:

```
$ ls -la /proc/$$/fd
lrwx------ 1 vagrant vagrant 64 Jul 14 12:10 0 -> /dev/pts/1
lrwx------ 1 vagrant vagrant 64 Jul 14 12:10 1 -> /dev/pts/1
lrwx------ 1 vagrant vagrant 64 Jul 14 12:10 2 -> /dev/pts/1
```

If we check the pseudoterminal `devpts` folder `/dev/pts/`:

```
$ ls -la /dev/pts/
crw--w----  1 vagrant tty  136, 0 Jul 14 12:11 0
crw--w----  1 vagrant tty  136, 1 Jul 14 12:11 1
c---------  1 rootroot   5, 2 Jul  9 21:14 ptmx
```

we can see that there are 2 pseudoterminals, one is our ssh client, and the other is our `tmux`.

Let’s observe the `tmux` server’s file descriptors:

```
$ ls -la /proc/11780/fd
lrwx------ 1 vagrant vagrant 64 Jul 14 12:09 0 -> /dev/null
lrwx------ 1 vagrant vagrant 64 Jul 14 12:09 1 -> /dev/null
lrwx------ 1 vagrant vagrant 64 Jul 14 12:09 2 -> /dev/null
…
lrwx------ 1 vagrant vagrant 64 Jul 14 12:09 5 -> /dev/pts/0
…
lrwx------ 1 vagrant vagrant 64 Jul 14 12:09 8 -> /dev/ptmx
```

We see it has an open `/dev/ptmx` to control the terminal on `/dev/pts/1`, and a `/dev/pts/0` to read our input and write output back to our ssh connection.

Now, if we detach, we can still see the bash process in the ps output:

```
$ pstree -p
tmux: server(11780)───bash(11781)
```

It’s left there and is waiting for us. Talkinh about the `tmux` server, it closed `/dev/pts/0` because we returned back control of the ssh terminal and it doesn’t need to read and write to it anymore:

```
$ ls -la /proc/11780/fd
lrwx------ 1 vagrant vagrant 64 Jul 14 12:09 0 -> /dev/null
lrwx------ 1 vagrant vagrant 64 Jul 14 12:09 1 -> /dev/null
lrwx------ 1 vagrant vagrant 64 Jul 14 12:09 2 -> /dev/null
…
lrwx------ 1 vagrant vagrant 64 Jul 14 12:09 8 -> /dev/ptmx
```

Also, if we take a look the `procfs`, we will find out that `tmux` server has its own session and process group. It makes sense, it should not depend on any of the active terminal connections.

```
$ cat /proc/11780/stat | cut -d " " -f 1,5,6,7,8,9 | tr ' ' '\n' | paste <(echo -ne "pid\nppid\npgid\nsid\ntty\ntgid\n") -
pid    11780
ppid   1
pgid   11780
sid    11780
tty    0
tgid   -1
```

If we open one more session, we will see one more shell process:

```
└─tmux: server(11780)─┬─bash(11781)
                      └─bash(11846)
```

And open file descriptors of the server:

```
$ ls -la /proc/11780/fd
lrwx------ 1 vagrant vagrant 64 Jul 14 12:09 0 -> /dev/null
lrwx------ 1 vagrant vagrant 64 Jul 14 12:09 1 -> /dev/null
lrwx------ 1 vagrant vagrant 64 Jul 14 12:18 10 -> /dev/pts/2
lrwx------ 1 vagrant vagrant 64 Jul 14 12:20 11 -> /dev/ptmx
...
lrwx------ 1 vagrant vagrant 64 Jul 14 12:09 7 -> /dev/pts/0
lrwx------ 1 vagrant vagrant 64 Jul 14 12:09 8 -> /dev/ptmx
```

The overall schema described above could be presented in the below image 5:

![tmux client-server architecture](https://biriukov.dev/docs/fd-pipe-session-terminal/images/tmux-sshd-ssh-client-pseudoterminal.png)

Image 5. – `tmux` client-server architecture

`tmux` server opens as many pseudoterminals as needed, but none of them is a controlling terminal. It is possible to do it in several ways, and the most simple one is to open a `/dev/pts/ptmx` with the `O_NOCTTY` open flag (`[man 2 open](https://man7.org/linux/man-pages/man2/open.2.html)`).

In order to set a demonize, the `tmux` server [made a single `fork()`](https://github.com/tmux/tmux/blob/6c2bf0e22119804022c8038563b9865999f5a026/compat/daemon.c#L44) and `setsid()`:

```
int
daemon(int nochdir, int noclose)
{
    int fd;

    switch (fork()) {
    case -1:
    return (-1);
    case 0:
    break;
    default:
    _exit(0);
    }

    if (setsid() == -1)
    return (-1);

    if (!nochdir)
    (void)chdir("/");

    if (!noclose && (fd = open(_PATH_DEVNULL, O_RDWR, 0)) != -1) {
    (void)dup2(fd, STDIN_FILENO);
    (void)dup2(fd, STDOUT_FILENO);
    (void)dup2(fd, STDERR_FILENO);
    if (fd > 2)
    (void)close (fd);
    }

#ifdef __APPLE__
    daemon_darwin();
#endif
    return (0);
}
```

It makes the `tmux` server immune to terminal terminations and signals logic I described earlier.

## Pseudoterminal proxy [#](https://biriukov.dev/docs/fd-pipe-session-terminal/4-terminals-and-pseudoterminals/#pseudoterminal-proxy)

As I mentioned earlier, we could think about pseudoterminals as a proxy. The reasonable question is can we leverage them in our day-to-day scripting routines? The answer, as you can guess, is yes. There are two incredible tools: `expect` (`[man 1 expect](https://man7.org/linux/man-pages/man1/expect.1.html)`) and `script` (`[man 1 script](https://man7.org/linux/man-pages/man1/script.1.html)`) that uses pseudoterminals in the proxy mode and are super helpful in writing basic automation.

### `expect` [#](https://biriukov.dev/docs/fd-pipe-session-terminal/4-terminals-and-pseudoterminals/#expect)

The `expect` program uses a pseudoterminal to allow an interactive terminal-oriented program to be driven from a script file. Let’s assume we need to automate an `ssh` connection in a shell script. We want to insert username and password when the `ssh` client asks for them. We can easily achieve this with `expect`:

```
#!/usr/bin/expect

set timeout 20

set host [lindex $argv 0]
set username [lindex $argv 1]
set password [lindex $argv 2]

spawn ssh "$username\@$host"

expect "password:"
send "$password\r";

interact
```

And test it:

```
[local ~] $ ./ssh.exp 192.168.0.1 vagrant vagrant
spawn ssh vagrant@192.168.0.1
vagrant@192.168.0.1's password:
[remote ~]$
```

where ”`vagrant`“ is our username and password.

### `script` [#](https://biriukov.dev/docs/fd-pipe-session-terminal/4-terminals-and-pseudoterminals/#script)

Another task is to record a terminal session. The pseudoterminals are used in the `script` program, which records all of the input and output that occurs during a shell session.

Record to file:

```
$ script --timing=time.txt script.log
```

Replay:

```
$ scriptreplay --timing=time.txt script.log
```

## Changing a process’s controlling terminal [#](https://biriukov.dev/docs/fd-pipe-session-terminal/4-terminals-and-pseudoterminals/#changing-a-processs-controlling-terminal)

And lastly, I want to show you one more fascinating tool `reptyr` [https://github.com/nelhage/reptyr](https://github.com/nelhage/reptyr). Imagine, you forgot to start a `screen` or `tmux` session and have run a long-running script. Using `reptyr` you can move it under a `screen` or `tmux` session without a restart!

It uses `ptrace` systemcall to change the session id of the running process.

> We use `ptrace` to attach to a target process and force it to execute code of our own choosing in order to open the new terminal, and `dup2` it over stdout and stderr.

More info about it could be found in the detailed author’s blog post:

[https://blog.nelhage.com/2011/02/changing-ctty/](https://blog.nelhage.com/2011/02/changing-ctty/)

How it works tl;dr:

> While we have `mutt` captured with `ptrace`, we can make it `fork` a dummy child, and start tracing that child, too. We’ll make the child `setpgid` to make it into its own process group, and then get mutt to `setpgid` itself into the child’s process group. mutt can then `setsid`, moving into a new session, and now, as a session leader, we can finally `ioctl(TIOCSCTTY)` on the new terminal, and we win.