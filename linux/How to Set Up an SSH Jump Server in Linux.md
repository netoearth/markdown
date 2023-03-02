A **jump host** (also known as a **jump server**) is an intermediary host or an SSH gateway to a remote network, through which a connection can be made to another host in a different security zone, for example, a demilitarized zone (**DMZ**). It bridges two dissimilar security zones and offers controlled access between them.

A **jump host** should be highly secured and monitored especially when it spans a private network and a **DMZ** with servers providing services to users on the internet.

A classic scenario is connecting from your desktop or laptop from inside your company’s internal network, which is highly secured with firewalls to a DMZ. In order to easily manage a server in a DMZ, you may access it via a **jump host**.

In a nutshell, an **SSH Jump** server is a Linux server that is used as a gateway to other Linux servers on a private network over the SSH Protocol.

In this article, we will demonstrate how to access a remote Linux server via a **jump host**, and also we will configure the necessary settings in your per-user SSH client configurations.

Table of Contents

1

-   [SSH Jump Server Setup](https://www.tecmint.com/access-linux-server-using-a-jump-host/#SSH_Jump_Server_Setup "SSH Jump Server Setup")

-   [Reasons for Configuring an SSH Jump Server](https://www.tecmint.com/access-linux-server-using-a-jump-host/#Reasons_for_Configuring_an_SSH_Jump_Server "Reasons for Configuring an SSH Jump Server")
-   [How To Create a Simple SSH Jump Server](https://www.tecmint.com/access-linux-server-using-a-jump-host/#How_To_Create_a_Simple_SSH_Jump_Server "How To Create a Simple SSH Jump Server")
-   [Dynamic Jumphost List](https://www.tecmint.com/access-linux-server-using-a-jump-host/#Dynamic_Jumphost_List "Dynamic Jumphost List")
-   [Multiple Jumphosts List](https://www.tecmint.com/access-linux-server-using-a-jump-host/#Multiple_Jumphosts_List "Multiple Jumphosts List")
-   [Static Jumphost List](https://www.tecmint.com/access-linux-server-using-a-jump-host/#Static_Jumphost_List "Static Jumphost List")
-   [Making SSH Jump Server More Secure](https://www.tecmint.com/access-linux-server-using-a-jump-host/#Making_SSH_Jump_Server_More_Secure "Making SSH Jump Server More Secure")

#### SSH Jump Server Setup

Consider the following scenario.

[![SSH Jump Host](https://www.tecmint.com/wp-content/uploads/2018/11/SSH-Jump-Host.png)](https://www.tecmint.com/wp-content/uploads/2018/11/SSH-Jump-Host.png)

SSH Jump Host

For more clarity, below is a simple setup demonstrating the role of an SSH Jump server.

[![SSH Jump Server Setup](https://www.tecmint.com/wp-content/uploads/2018/11/SSH-Jump-Server-Setup.png)](https://www.tecmint.com/wp-content/uploads/2018/11/SSH-Jump-Server-Setup.png)

SSH Jump Server Setup

### Reasons for Configuring an SSH Jump Server

A **Jump** server provides a gateway to your infrastructure and reduces the potential attack surface to your resources. It also provides transparent management of devices as well as a single point of entry to your resources.

Keep in mind that as you incorporate a jump server into your infrastructure, ensure that the server is hardened, otherwise it would be as good as not using one. We will come back to this later in this tutorial.

### How To Create a Simple SSH Jump Server

Let us now focus on how you can create a simple **SSH Jump** server. Here is our simple setup.

-   Originating IP: 105.68.76.85.
-   Jump Server IP (we’ll call this host-jump): 173.82.232.55.
-   Destination IP (we’ll call this host\_destination): 173.82.227.89.

In the above scenario, you want to connect to **HOST 2 (173.82.227.89)**, but you have to go through **HOST 1 (173.82.232.55)**, because of firewalling, routing, and access privileges. There is a number of valid reasons why jump hosts are needed.

### Dynamic Jumphost List

The simplest way to connect to a target server via a **jump host** is using the `-A` and `-J` flags from the command line. This tells ssh to make a connection to the jump host and then establish a TCP forwarding to the target server, from there (make sure you’ve [Passwordless SSH Login](https://www.tecmint.com/ssh-passwordless-login-using-ssh-keygen-in-5-easy-steps/) between machines).

```
$ ssh -A -J user@jump-server  user@destination server

```

For example, in our setup, we have the user called **james** configured on the **Jump Server** and **tecmint** configured on the destination or target system.

The command will look as follows from the originating IP.

```
$ ssh -A -J james@173.82.232.55 tecmint@173.82.227.89

```

The command will prompt you for the jump server’s user password, followed by the target system’s password upon which you will be granted access to the target system.

[![Connect Server Using SSH Jump Host](https://www.tecmint.com/wp-content/uploads/2018/11/Connect-Server-Using-SSH-Jump-Host.png)](https://www.tecmint.com/wp-content/uploads/2018/11/Connect-Server-Using-SSH-Jump-Host.png)

Connect Server Using SSH Jump Host

If **usernames** or **ports** on machines differ, specify them on the terminal as shown.

```
$ ssh -J username@host1:port username@host2:port  

```

### Multiple Jumphosts List

The same syntax can be used to make jumps over multiple servers.

```
$ ssh -J username@host1:port,username@host2:port username@host3:port

```

### Static Jumphost List

Static jumphost list means, that you know the **jumphost** or **jumphosts** that you need to connect a machine. Therefore you need to add the following static jumphost ‘routing’ in `~/.ssh/config` file and specify the host aliases as shown.

```
### First jumphost. Directly reachable
Host vps1
  HostName vps1.example.org

### Host to jump to via jumphost1.example.org
Host contabo
  HostName contabo.example.org
  ProxyJump vps1

```

Now try to connect to a target server via a **jump host** as shown.

```
$ ssh -J vps1 contabo

```

[![Login to Target Host via Jumphost](https://www.tecmint.com/wp-content/uploads/2018/11/login-into-target-host-via-jumphost.png)](https://www.tecmint.com/wp-content/uploads/2018/11/login-into-target-host-via-jumphost.png)

Login to Target Host via Jumphost

The second method is to use the **ProxyCommand** option to add the **jumphost** configuration in your `~.ssh/config` or `$HOME/.ssh/config` file as shown.

In this example, the target host is **contabo**, and the **jumphost** is **vps1**.

```
Host vps1
HostName vps1.example.org
IdentityFile ~/.ssh/vps1.pem
User ec2-user

Host contabo
HostName contabo.example.org
IdentityFile ~/.ssh/contabovps
Port 22
User admin
Proxy Command ssh -q -W %h:%p vps1

```

Save the changes and exit the file. To apply the changes, restart the SSH daemon.

```
$ sudo systemctl restart ssh

```

Let’s explore the options used in the configuration file:

-   `-q` – This stands for quiet mode. It suppresses warnings and diagnostic messages.
-   `-W` – Requests that standard input and output on the client be forwarded to HOST on PORT over the secure channel.
-   `%h` – Specifies the host to connect to.
-   `%p` – Specified the port to connect to on the remote host.

To ‘**jump**‘ to the destination system from your Originating IP through the Jump Server, just run the following command:

```
$ ssh contabo

```

The above command will first open an ssh connection to **vps1** in the background affected by the **ProxyCommand**, and thereafter, start the ssh session to the target server **contabo**.

**\[ You might also like: [How to Connect Remote Linux via SSH ProxyJump and ProxyCommand](https://www.tecmint.com/ssh-proxyjump-and-ssh-proxycommand/ "Use SSH ProxyJump and SSH ProxyCommand") \]**

### Making SSH Jump Server More Secure

One of the ways of making this setup more secure is by copying the Public SSH key from the Originating system to the **Jump Server**, and then finally to the target system, and then disabling password authentication. Check out our guide on [how to enable SSH passwordless authentication](https://www.tecmint.com/ssh-passwordless-login-using-ssh-keygen-in-5-easy-steps/ "Setup SSH Passwordless Login in Linux").

In addition, check out [SSH server hardening tips](https://www.tecmint.com/secure-openssh-server/ "Secure and Harden OpenSSH Server").

Also, ensure that no sensitive data is housed within the **Jump** server as this could lead to leakage of access credentials such as usernames and passwords leading to a system-wide breach.

For more information, see the ssh man page or refer to: [OpenSSH/Cookbxook/Proxies and Jump Hosts](https://en.wikibooks.org/wiki/OpenSSH/Cookbook/Proxies_and_Jump_Hosts).

That’s all for now! In this article, we have demonstrated how to access a remote server via a jump host. Use the feedback form below to ask any questions or share your thoughts with us.

## If You Appreciate What We Do Here On TecMint, You Should Consider:

TecMint is the fastest growing and most trusted community site for any kind of Linux Articles, Guides and Books on the web. Millions of people visit TecMint! to search or browse the thousands of published articles available FREELY to all.

If you like what you are reading, please consider buying us a coffee ( or 2 ) as a token of appreciation.

[![Support Us](https://www.tecmint.com/wp-content/uploads/2015/01/coffee.png)](https://www.buymeacoffee.com/tecmint)

**We are thankful for your never ending support.**