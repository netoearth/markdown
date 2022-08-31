A software repository or “**repo**” is a central location for keeping and maintaining RPM software packages for Redhat Linux distribution, from which users can download and install packages on their Linux servers.

**Repositories** are generally stored on a public network, which can be accessed by multiple users on the internet. However, you can create your own local repository on your server and access it as a single user or allow access to other machines on your local LAN (Local Area Network) using HTTP web server.

The advantage of creating a local repository is that you don’t require an internet connection to install software packages or updates.

[YUM (Yellowdog Updater Modified)](https://www.tecmint.com/20-linux-yum-yellowdog-updater-modified-commands-for-package-mangement/) or [DNF (Dandified YUM)](https://www.tecmint.com/dnf-commands-for-fedora-rpm-package-management/) is a widely used software package management utility for [RPM (RedHat Package Manager)](https://www.tecmint.com/20-practical-examples-of-rpm-commands-in-linux/) based Linux systems, which makes software installation easy on Red Hat/CentOS Linux.

In this article, we will explain how to setup a local **YUM/DNF** repository on **RHEL 8** using the installation DVD or ISO file. We will also show you how to find and install software packages on client **RHEL 8** machines using **Nginx HTTP** server.

#### Our Testing Environment

```
Local Repository Server: RHEL 8 [192.168.0.106]
Local Client Machine: RHEL 8 [192.168.0.200]

```

### Step 1: Install Nginx Web Server

**1.** First, install the **Nginx HTTP** server using the [DNF package manager](https://www.tecmint.com/dnf-commands-for-fedora-rpm-package-management/) as follows.

```
# dnf install nginx

```

[![Install Nginx on RHEL 8](https://www.tecmint.com/wp-content/uploads/2019/05/Install-Nginx-on-RHEL-8.png)](https://www.tecmint.com/wp-content/uploads/2019/05/Install-Nginx-on-RHEL-8.png)

Install Nginx on RHEL 8

**2.** Once **Nginx** installed, you can start, enable the service to auto start at boot time and verify the status using the following commands.

```
# systemctl start nginx
# systemctl enable nginx
# systemctl status nginx

```

[![Verify Nginx on RHEL 8](https://www.tecmint.com/wp-content/uploads/2019/05/Verify-Nginx-on-RHEL-8.png)](https://www.tecmint.com/wp-content/uploads/2019/05/Verify-Nginx-on-RHEL-8.png)

Verify Nginx on RHEL 8

**3.** Next, you need to open **Nginx** ports **80** and **443** on your firewall.

```
# firewall-cmd --zone=public --permanent --add-service=http
# firewall-cmd --zone=public --permanent --add-service=https
# firewall-cmd --reload

```

[![Open Nginx Port on RHEL 8 Firewall](https://www.tecmint.com/wp-content/uploads/2019/05/Open-Nginx-Port-on-RHEL-8-Firewall.png)](https://www.tecmint.com/wp-content/uploads/2019/05/Open-Nginx-Port-on-RHEL-8-Firewall.png)

Open Nginx Port on RHEL 8 Firewall

**4.** Now you can verify that your **Nginx** server is up and running by going to the following URL on your web browser, a default Nginx web page will be displayed.

```
http://SERVER_DOMAIN_NAME_OR_IP

```

[![Verify Nginx Installation on RHEL 8](https://www.tecmint.com/wp-content/uploads/2019/05/Verify-Nginx-Installation-on-RHEL-8.png)](https://www.tecmint.com/wp-content/uploads/2019/05/Verify-Nginx-Installation-on-RHEL-8.png)

Verify Nginx Installation on RHEL 8

### Step 2: Mounting RHEL 8 Installation DVD/ISO File

**5.** Create a local repository mount point under **Nginx** document root directory `/var/www/html/` and mount the downloaded **RHEL 8 DVD ISO** image under `/mnt` directory.

```
# mkdir /var/www/html/local_repo
# mount -o loop rhel-8.0-x86_64-dvd.iso /mnt  [Mount Download ISO File]
# mount /dev/cdrom /mnt                       [Mount DVD ISO File from DVD ROM]

```

**6.** Next, copy ISO files locally under `/var/www/html/local_repo` directory and verify the contents using [ls command](https://www.tecmint.com/tag/linux-ls-command/).

```
# cd /mnt
# tar cvf - . | (cd /var/www/html/local_repo/; tar xvf -)
# ls -l /var/www/html/local_repo/

```

[![Verify Contents of RHEL 8 ISO Files](https://www.tecmint.com/wp-content/uploads/2019/05/Verify-RHEL-8-ISO-Files.png)](https://www.tecmint.com/wp-content/uploads/2019/05/Verify-RHEL-8-ISO-Files.png)

Verify Contents of RHEL 8 ISO Files

### Step 3: Configuring Local Repository

**7.** Now it is time to configure the local repository. You need to create the local repository configuration file in the `/etc/yum.repos.d/` directory and set the appropriate permissions on the file as shown.

```
# touch /etc/yum.repos.d/local-rhel8.repo
# chmod  u+rw,g+r,o+r  /etc/yum.repos.d/local-rhel8.

```

**8.** Then open the file for editing using your [favorite command line text editor](https://www.tecmint.com/linux-command-line-editors/).

```
# vim /etc/yum.repos.d/local.repo

```

**9.** Copy and paste the following content in the file.

```
[LocalRepo_BaseOS]
name=LocalRepo_BaseOS
metadata_expire=-1
enabled=1
gpgcheck=1
baseurl=file:///var/www/html/local_repo/
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-redhat-release

[LocalRepo_AppStream]
name=LocalRepo_AppStream
metadata_expire=-1
enabled=1
gpgcheck=1
baseurl=file:///var/www/html/local_repo/
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-redhat-release

```

Save the changes and exit the file.

**10.** Now you need to install required packages for creating, configuring and managing your local repository by running the following command.

```
# yum install createrepo  yum-utils
# createrepo /var/www/html/local_repo/

```

### Step 4: Testing Local Repository

**11.** In this step, you should run a cleanup of temporary files kept for repositories by using the following command.

```
# yum clean all
OR
# dnf clean all

```

**12.** Then verify that the created repositories appear in the list of enabled repositories.

```
# dnf repolist
OR
# dnf repolist  -v  #shows more detailed information 

```

[![Verify Local Repository in RHEL 8](https://www.tecmint.com/wp-content/uploads/2019/05/Verify-Local-Repository-in-RHEL-8.png)](https://www.tecmint.com/wp-content/uploads/2019/05/Verify-Local-Repository-in-RHEL-8.png)

Verify Local Repository in RHEL 8

**13.** Now try to install a package from the local repositories, for example install [Git command line tool](https://www.tecmint.com/use-git-version-control-system-in-linux/) as follows:

```
# dnf install git

```

[![Install Package from Local Yum Repository](https://www.tecmint.com/wp-content/uploads/2019/05/Install-Package-from-Local-Yum-Repository.png)](https://www.tecmint.com/wp-content/uploads/2019/05/Install-Package-from-Local-Yum-Repository.png)

Install Package from Local Yum Repository

Looking at the output of the above command, the **git** package is being installed from the **LocalRepo\_AppStream** repository as shown in the screenshot. This proves that the local repositories are enabled and are working fine.

### Step 5: Setup Local Yum Repository on Client Machines

**14.** Now on your **RHEL 8** client machines, add your local repos to the YUM configuration.

```
# vi /etc/yum.repos.d/local-rhel8.repo 

```

Copy and paste the configuration below in the file. Make sure to replace `baseurl` with your server IP address or domain.

```
[LocalRepo_BaseOS]
name=LocalRepo_BaseOS
enabled=1
gpgcheck=0
baseurl=http://192.168.0.106

[LocalRepo_AppStream]
name=LocalRepo_AppStream
enabled=1
gpgcheck=0
baseurl=http://192.168.0.106

```

Save the file and start using your local YUM mirrors.

**15.** Next, run the following command to see your local repos in the list of available YUM repos, on the client machines.

```
# dnf repolist

```

[![Verify Local Repository in RHEL 8 Client](https://www.tecmint.com/wp-content/uploads/2019/05/Verify-Local-Repository-in-RHEL-8-Client.png)](https://www.tecmint.com/wp-content/uploads/2019/05/Verify-Local-Repository-in-RHEL-8-Client.png)

Verify Local Repository in RHEL 8 Client

That’s all! In this article, we have shown how to create a local **YUM/DNF** repository in RHEL 8, using the installation DVD or ISO file. Do not forget to reach us via the feedback form below for any questions or comments.

## If You Appreciate What We Do Here On TecMint, You Should Consider:

TecMint is the fastest growing and most trusted community site for any kind of Linux Articles, Guides and Books on the web. Millions of people visit TecMint! to search or browse the thousands of published articles available FREELY to all.

If you like what you are reading, please consider buying us a coffee ( or 2 ) as a token of appreciation.

[![Support Us](https://www.tecmint.com/wp-content/uploads/2015/01/coffee.png)](https://www.buymeacoffee.com/tecmint)

**We are thankful for your never ending support.**