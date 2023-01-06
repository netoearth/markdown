_**Brief: Have you created a certificate key or private key with a passphrase and wish to remove it? In this guide, we will show how to remove a passphrase using the openssl command line tool and from an ssh private key.**_

A **passphrase** is a sequence of words used to secure and control access to a private key. It is a key or secret used to [encrypt the file](https://www.tecmint.com/file-and-disk-encryption-tools-for-linux/ "Best File and Disk Encryption Tools for Linux") that contains the actual encryption key.

To use the private key for encryption, for instance for [ssh public-key-based](https://www.tecmint.com/ssh-passwordless-login-using-ssh-keygen-in-5-easy-steps/ "Setup SSH Passwordless Login in Linux") connections, you are required to decrypt the private key file using the decryption key (the **passphrase**) – you are prompted to enter the passphrase.

### Removing a Passphrase from SSL Certificate using OpenSSL

The header of a **TLS/SSL** private key with a passphrase looks like what is shown in the following screenshot. The parameter “**DEK-Info**” stores information required to decrypt the key using the passphrase.

```
$ cat private.pem

```

[![View SSL Private Key Passphrase](https://www.tecmint.com/wp-content/uploads/2022/11/View-SSL-Private-Key-Passphrase.png)](https://www.tecmint.com/wp-content/uploads/2022/11/View-SSL-Private-Key-Passphrase.png)

View SSL Private Key Passphrase

When you or any application such as NGINX webserver is using the private key, which invokes it for encrypting data, you or the application will be prompted to supply the passphrase before the key can be used, for example:

```
$ openssl rsa -in private.pem -outform PEM -pubout -out public.pem

```

[![Enter Private Key Passphrase](https://www.tecmint.com/wp-content/uploads/2022/11/Private-Key-Passphrase.png)](https://www.tecmint.com/wp-content/uploads/2022/11/Private-Key-Passphrase.png)

Enter Private Key Passphrase

To remove the passphrase of an SSL private key using the **openssl** command line tool, simply copy the old file to a new file name. After, the new private key will not have a passphrase as shown in the following screenshot.

```
$ openssl rsa -in private.pem -out private_new.pem 
$ cat private_new.pem 

```

[![Remove SSL Private Key Passphrase](https://www.tecmint.com/wp-content/uploads/2022/11/Remove-SSL-Private-Key-Passphrase.png)](https://www.tecmint.com/wp-content/uploads/2022/11/Remove-SSL-Private-Key-Passphrase.png)

Remove SSL Private Key Passphrase

### Remove Passphrase from SSH Private Key

Usually, when you generate an SSH key pair, you are prompted to set a **passphrase** for the private key as shown in the following screenshot. If you leave it empty, no passphrase is set.

[![Generate SSH Private Key Passphrase](https://www.tecmint.com/wp-content/uploads/2022/11/Generate-SSH-Private-Key-Passphrase.png)](https://www.tecmint.com/wp-content/uploads/2022/11/Generate-SSH-Private-Key-Passphrase.png)

Generate SSH Private Key Passphrase

When you invoke a private ssh key that has a passphrase, before the ssh client can use the key for the connection, it prompts you to supply the passphrase as shown.

```
$ ssh -i .ssh/tecmint tecmint@192.168.56.108

```

[![Enter SSH Private Key Passphrase](https://www.tecmint.com/wp-content/uploads/2022/11/SSH-Private-Key-Passphrase.png)](https://www.tecmint.com/wp-content/uploads/2022/11/SSH-Private-Key-Passphrase.png)

Enter SSH Private Key Passphrase

To remove the passphrase, use the **ssh-keygen** command with the `-p` option which prompts you for the existing passphrase, and `-f` to specify the private key file:

```
$ ssh-keygen -p -f .ssh/tecmint

```

Enter the old passphrase, and leave the new passphrase empty.

[![Remove SSH Private Key Passphrase](https://www.tecmint.com/wp-content/uploads/2022/11/Remove-SSH-Private-Key-Passphrase.png)](https://www.tecmint.com/wp-content/uploads/2022/11/Remove-SSH-Private-Key-Passphrase.png)

Remove SSH Private Key Passphrase

**\[ You might also like: [Basic SSH Command Usage and Configuration in Linux](https://www.tecmint.com/ssh-command-usage/ "Basic SSH Command Usage and Configuration in Linux") \]**

That’s all! Remember that is recommended to use **passphrases** to increase the [security of your SSH keys](https://www.tecmint.com/secure-openssh-server/ "Secure and Harden OpenSSH Server"). To share your thoughts with us about this guide, use the comment form below.

## If You Appreciate What We Do Here On TecMint, You Should Consider:

TecMint is the fastest growing and most trusted community site for any kind of Linux Articles, Guides and Books on the web. Millions of people visit TecMint! to search or browse the thousands of published articles available FREELY to all.

If you like what you are reading, please consider buying us a coffee ( or 2 ) as a token of appreciation.

[![Support Us](https://www.tecmint.com/wp-content/uploads/2015/01/coffee.png)](https://www.buymeacoffee.com/tecmint)

**We are thankful for your never ending support.**