Posted: November 8, 2019 %t min read

Image by Gerhild Klinkow @ Pixabay

In the early years of computing, `[telnet](https://www.commandlinux.com/man-page/man1/telnet.1.html)` was used to connect to the command line on remote systems. SSH has replaced `telnet` for remote access needs, and these days when you hear about `telnet`, it is usually when somebody is using the client as a generic network troubleshooting tool.

That’s because, in troubleshooting sessions, sysadmins turn to `telnet` and `[netcat](https://www.commandlinux.com/man-page/man1/nc.1.html)` to test connectivity to service offerings.

## Telnet

See what this process looks like for `telnet`:

```
[dminnich@dminnichlt tmp]$ telnet -4 www.redhat.com 80
Trying 104.117.5.18...
Connected to www.redhat.com.
Escape character is '^]'.
```

Here is a breakdown of the parameters:

-   `-4` means to use IPV4. This flag is not necessary but it made my logs prettier.
-   `www.redhat.com` is the hostname/IP address to connect to.
-   `80` is the port to connect to.

That `Connected to www.redhat.com` line states that the connection was successful. This result means that the server is operational and there is nothing on the network (or any client or server machine) blocking this connection from happening.

You can exit out of `telnet` by pressing Ctrl+\] and then typing `quit`:

```
^]
telnet> quit
Connection closed.
```

Here is an example of a failed connection in `telnet`:

```
[dminnich@dminnichlt tmp]$ telnet -4 www.redhat.com 21
Trying 23.1.49.220...
telnet: connect to address 23.1.49.220: Connection timed out
```

Depending on how the remote network is configured, you may see a `Connection refused` message immediately, or you may have to wait a while to get the `Connection timed out` error. Alternatively, if `telnet` sits there for a few seconds without any output, it is usually safe to assume that the connection will time out, so you can stop the connection attempt by doing a Ctrl+C.

When you see errors like this, it means that any of the following things are wrong:

-   The server daemon isn’t running.
-   The server itself isn’t up.
-   A firewall rule is blocking the connection.
-   There is no network route to the destination.

## Netcat

Now, look at this same process with Netcat (`ncat` on [Red Hat Enterprise Linux](https://www.redhat.com/en/technologies/linux-platforms/enterprise-linux) 8 and related distributions, abbreviated `nc`). Here is an example of a successful connection using Netcat (Ctrl+C will exit the Netcat session:)

```
[dminnich@dminnichlt tmp]$ nc -w3 -4 -v www.redhat.com 80
Ncat: Version 7.70 ( https://nmap.org/ncat )
Ncat: Connected to 104.117.5.18:80.
```

That `-w3` says wait three seconds and then give up, which is a nice Netcat-native feature `telnet` is missing.

Here is what a failed connection looks like in Netcat:

```
[dminnich@dminnichlt tmp]$ nc -w3 -4 -v www.redhat.com 21
Ncat: Version 7.70 ( https://nmap.org/ncat )
Ncat: Connection timed out.
```

Netcat also supports listening on ports for incoming connections, as well as basic port scanning and some other niceties. These features and the fact that lots of operating systems install Netcat but not `telnet` by default are why some sysadmins are starting to use Netcat instead of `telnet` for their troubleshooting needs.

## Interacting with services

One final thing: Both of these tools can interact with the service offerings they connect to. If you type syntactically correct protocol messages and hit Enter, you will receive responses from the service. Here is an example:

```
[dminnich@dminnichlt tmp]$ nc -4 -w3 -v google.com 80
Ncat: Version 7.70 ( https://nmap.org/ncat )
Ncat: Connected to 172.217.5.238:80.
GET /
...
</body></html>
```

You may have to hit Enter a few times in the above example. Interacting with service offerings in this fashion gets complicated fast, especially when encryption comes into play, but if you need to test the internals of something—or if you don’t have a better protocol-specific tool like `curl` around—you can make it work.