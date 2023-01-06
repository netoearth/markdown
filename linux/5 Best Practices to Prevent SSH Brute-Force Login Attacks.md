Servers running **SSH** are usually a soft target for brute-force attacks. Hackers are constantly coming up with innovative software tools and bots for automating brute-force attacks which further increase the risk of intrusion.

In this guide, we explore some of the tips that you can implement to safeguard your SSH servers from brute-force attacks on [RHEL-based Linux distributions](https://www.tecmint.com/redhat-based-linux-distributions/ "The Best RedHat-based Linux Distributions") and [Debian derivatives](https://www.tecmint.com/debian-based-linux-distributions/ "Best Debian-based Linux Distributions").

### Disable SSH Password Authentication and Enable SSH-Key Authentication

The default authentication method for **SSH** is username/password authentication. But as we have seen, password authentication is prone to brute-force attacks. To be on the safe side, it is recommended to implement key-based SSH authentication where authentication is made possible by public and private SSH key pairs. The private key remains on the client’s PC while the public key is copied to the server.

During SSH key authentication, the server checks whether the client PC possesses the private key. If the check is successful, a shell session is created or the command sent to the remote server is executed successfully. We have a comprehensive guide on [how to configure SSH key-based authentication](https://www.tecmint.com/configure-ssh-passwordless-rhel-9/ "Configure SSH Passwordless Authentication on RHEL").

Even after setting up Key-based authentication, your server is still susceptible to brute-force attacks for the simple reason that password authentication is still active. This needs to be disabled.

Therefore, edit the default SSH configuration file.

```
$ sudo vim /etc/ssh/sshd_config

```

Set the **PasswordAuthentication** parameter to `no` as shown.

```
PasswordAuthentication no

```

[![Disable SSH Password Authentication](https://www.tecmint.com/wp-content/uploads/2012/11/Disable-SSH-Password-Authentication.png)](https://www.tecmint.com/wp-content/uploads/2012/11/Disable-SSH-Password-Authentication.png)

Disable SSH Password Authentication

Then save the file and reload SSH to apply the changes.

```
$ sudo systemctl reload ssh

```

### Implement Fail2ban Intrusion Prevention Tool

Written in **Python**, [Fail2ban](https://www.fail2ban.org/wiki/index.php/Main_Page "Fail2ban for SSH") is an open-source intrusion prevention framework that scans log files of services for authentication failures and bans IPs that repeatedly fail password authentication checks for a specified amount of time.

**Fail2ban** constantly monitors server log files for intrusion attempts and other nefarious activity, After a predefined number of authentication failures – in most cases, 3 failed login attempts – Fail2ban automatically blocks the remote host from accessing the server, and the host is kept in a ‘**Jail**‘ for a specific duration of time.

In doing so, **Fail2ban** significantly reduces the rate of incorrect password authentication attempts. Check out our guide on how you can [install and configure Fail2ban on Linux](https://www.tecmint.com/use-fail2ban-to-secure-linux-server/ "Use Fail2ban to Secure Your Linux Server") to secure your server from Bruteforce attacks.

### Limit Maximum Number of SSH Authentication Attempts

Another simple way of safeguarding your server from brute-force attacks is by limiting the number of SSH login attempts. By default, this is set to **3**, but if by any chance this is set to a higher value, set it to 3 connection attempts at most.

For example, to set the maximum connection attempts to 3 set the **MaxAuthTries** parameter to **3** as shown

```
MaxAuthTries = 3

```

Once again, save the changes and reload the SSH service.

```
$ sudo systemctl reload ssh

```

### Implement TCP Wrappers to Limit SSH Access From Clients

**TCP** wrappers is a library that provides a host-based **Access Control List** (**ACL**) that restricts access to TCP services by remote clients based on their IP addresses

Remote hosts from accessing services on the system. TCP wrappers uses the **/etc/hosts.allow** and **/etc/hosts.deny** configuration files (in that order) to determine if the remote client is allowed to access a specific service or not.

Usually, these files are commented out and all hosts are allowed through the TCP wrappers layer. Rules for allowing access to a given service are placed in the **/etc/hosts.allow** file and take precedence over the rules in the /**etc/hosts.deny** file.

Best practice recommends blocking all incoming connections. Therefore, open the **/etc/hosts.deny** file.

```
$ sudo vim /etc/hosts.deny

```

Add the following line.

```
ALL: ALL

```

Save the changes and exit the file.

Then access the **/etc/hosts.allow** file.

```
$ sudo vim /etc/hosts.deny

```

Configure the hosts or domains that can connect to the server via SSH as shown. In this example, we are allowing only two remote hosts to connect to the server (**173.82.227.89** and **173.82.255.55**) and denying the rest.

```
sshd: 173.82.227.89 173.82.255.55
sshd: ALL: DENY

```

[![Limit SSH Access to Clients](https://www.tecmint.com/wp-content/uploads/2012/11/Limit-SSH-Access-to-Clients.png)](https://www.tecmint.com/wp-content/uploads/2012/11/Limit-SSH-Access-to-Clients.png)

Limit SSH Access to Clients

Save the changes and exit the configuration file.

To test it out, try to connect to the server from a host that is not among those you have allowed access to. You should get a permission error as shown.

```
$ ssh root@173.82.235.7

kex_exchange_identification: read: Connection reset by peer
Connection reset by 173.82.235.7 port 22
lost connection

```

### Implement SSH Two Factor Authentication

**Two Factor Authentication** provides an added security layer to password authentication, thereby making your server more secure from brute-force attacks. A widely-used **Two Factor Authentication** solution is **Google Authenticator App** and we have a well-documented guide on how you can [set up Two Factor Authentication](https://www.tecmint.com/ssh-two-factor-authentication/ "Setup Two-Factor Authentication For SSH In Linux").

##### Conclusion

This was a summary of 5 best practices that you can implement to prevent **SSH Brute Force** login attacks and ensure your server’s safety. You can also read [How to secure and harden OpenSSH Server](https://www.tecmint.com/secure-openssh-server/ "Secure and Harden OpenSSH Server").

## If You Appreciate What We Do Here On TecMint, You Should Consider:

TecMint is the fastest growing and most trusted community site for any kind of Linux Articles, Guides and Books on the web. Millions of people visit TecMint! to search or browse the thousands of published articles available FREELY to all.

If you like what you are reading, please consider buying us a coffee ( or 2 ) as a token of appreciation.

[![Support Us](https://www.tecmint.com/wp-content/uploads/2015/01/coffee.png)](https://www.buymeacoffee.com/tecmint)

**We are thankful for your never ending support.**