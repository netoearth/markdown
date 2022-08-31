![Flush DNS Cache](https://vitux.com/wp-content/uploads/flush-dns-cache-ubuntu.png?ezimgfmt=rs:750x375/rscb10/ng:webp/ngcb10)

Are you using the latest version of Ubuntu and looking for a way to clear your DNS cache? Then you’ve come to the right place. In the following tutorial, you will learn how to clear the DNS cache in Ubuntu 20.04 and Ubuntu 22.04. We’ll also explain why you should clear your DNS cache from time to time. So let’s start with the process

## Why and how to clear DNS cache?

A DNS cache can occasionally get corrupted. The reason may be technical errors or other viral attacks that add invalid DNS records to the database, which may result in a user being redirected to another website with a lot of advertisements or even malware when visiting a website. If the cache is corrupted, the user is advised to clear the DNS cache

Some Debian Linux like Ubuntu still uses systemd-resolve. This resolve is already built into the system in Ubuntu and is automatically used by the operating system for many things without the user knowing about it. It is already installed and set up in Ubuntu. All the user has to do is enter the command to flush the DNS and it’s done.

First, you need to open the terminal and type:

```
sudo systemd-resolve - -flush-caches
```

![Flush DNS cache in systemd](https://vitux.com/wp-content/uploads/2019/02/word-image-100.png?ezimgfmt=rs%3Adevice%2Frscb10-1)

When you enter the command, the terminal does not give any confirmation that the cache has been flushed, to confirm that you have to enter another command that would show the user the statistics, the command is as follows:

```
sudo system-resolve - -statistics
```

After you enter the command, the statistics will be displayed in the terminal. If you see that the “current cache size” is zero, you will get confirmation that your DNS cache has been cleared.

If you are using a version of Linux other than Ubuntu, you can also use the following command:

## NSCD command

If you are not using Ubuntu, but another Linux, you can also use nscd. Arch Linux mostly uses nscd. If that’s the case, you just need to type the following command to clear your DNS cache in that Linux.

```
sudo systemctl restart nscd
```

![Restart ncsd](https://vitux.com/wp-content/uploads/2019/02/word-image-101.png?ezimgfmt=rs:727x428/rscb10/ng:webp/ngcb10)

You can use the method described above to clear the DNS cache in Ubuntu. As mentioned earlier, you should clear your DNS cache from time to time because it can cause various problems, such as web pages not loading properly or web scripts not working properly. All these problems are caused by a corrupted DNS cache. Clearing and resetting it will probably fix the problem.

Have you tried the method described above for clearing the DNS cache? Did it work for you? If not, please let us know which method you used in the comments section.