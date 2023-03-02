_Are you looking for how to install a software package from the Extras Library on EC2? We will help you!_

Here, at Bobcares, we often receive similar AWS queries from our customers as a part of our AWS Support Services.

Today, let’s see the steps followed by our [Support Techs](https://bobcares.com/aws-support-services/) to install a software package from the Extras Library on Amazon EC2.  
  
 

### **Install a software package from the Extras Library on EC2**

   
Amazon Linux Extras is a mechanism in Amazon Linux 2 to enable the consumption of new versions of software packages on a stable operating system. For installing a software package from the Extras library, first, we need to install the **amazon-linux-extras** repository on the instance. Then we need to list the available packages and enable the desired ones. After enabling, then using yum we can install the software packages.

Now, let’s see the steps to install a software package from Amazon Linux Extras in Amazon Linux 2:

1\. First, access the EC2 Linux instance via SSH.

2\. Then run the **which** command to verify the amazon-linux-extras package is installed.

```
$ which amazon-linux-extras
/usr/bin/amazon-linux-extras
```

We can use yum to install the amazon-linux-extras package if it is not installed already.

$ sudo yum install -y amazon-linux-extras

3.Then we need to list the available software packages.

```
$ amazon-linux-extras
0 ansible2 available [ =2.4.2 =2.4.6 ]
2 httpd_modules available [ =1.0 ]
3 memcached1.5 available [ =1.5.1 ]
4 nginx1.12 available [ =1.12.2 ]
5 postgresql10 available [ =10 ]
6 redis4.0 available [ =4.0.5 =4.0.10 ]
7 R3.4 available [ =3.4.3 ]
8 rust1 available [ =1.22.1 =1.26.0 =1.26.1 =1.27.2 =1.31.0 ]
9 vim available [ =8.0 ]
10 ruby2.4 available [ =2.4.2 =2.4.4 ]
11 php7.3 available [ =7.3.0 =7.3.4 =7.3.5 =7.3.8 =7.3.11 =7.3.13 =7.3.14 =7.3.16 ]
```

4\. Now we can enable the desired topic.

The output will show the commands needed for installation.

Here, for example, to enable PHP 7.3, we can use the following command.

```
$ sudo amazon-linux-extras enable php7.3
0 ansible2 available [ =2.4.2 =2.4.6 ]
2 httpd_modules available [ =1.0 ]
3 memcached1.5 available [ =1.5.1 ]
4 nginx1.12 available [ =1.12.2 ]
5 postgresql10 available [ =10 ]
6 redis4.0 available [ =4.0.5 =4.0.10 ]
7 R3.4 available [ =3.4.3 ]
8 rust1 available [ =1.22.1 =1.26.0 =1.26.1 =1.27.2 =1.31.0 ]
9 vim available [ =8.0 ]
10 ruby2.4 available [ =2.4.2 =2.4.4 ]
11 php7.2=latest enabled [ =7.3.0 =7.3.4 =7.3.5 =7.3.8 =7.3.11 =7.3.13 =7.3.14 =7.3.16 ]
_ php7.2 available [ =7.2.22 =7.2.25 =7.2.27 ]

Now you can install:
# yum clean metadata
# yum install php-cli php-pdo php-fpm php-json php-mysqlnd
```

5\. Then we can install the topic using yum.

```
$ sudo yum clean metadata

$ sudo yum install php-cli php-pdo php-fpm php-json php-mysqlnd
```

6\. Now, use the following commands to verify the installation.

```
$ yum list installed php-cli php-pdo php-fpm php-json php-mysqlnd
Loaded plugins: extras_suggestions, langpacks, priorities, update-motd
Installed Packages
php-cli.x86_64 7.2.16-1.amzn2.0.1 @amzn2extra-php7.2
php-fpm.x86_64 7.2.16-1.amzn2.0.1 @amzn2extra-php7.2
php-json.x86_64 7.2.16-1.amzn2.0.1 @amzn2extra-php7.2
php-mysqlnd.x86_64 7.2.16-1.amzn2.0.1 @amzn2extra-php7.2
php-pdo.x86_64 7.2.16-1.amzn2.0.1 @amzn2extra-php7.2

$ which php
/usr/bin/php
```

7\. Also confirm the software version using the following command.

```
$ php --version
PHP 7.2.16 (cli) (built: Aug 3 2021 15:40:35) ( NTS )
Copyright (c) 1997-2018 The PHP Group
```

**\[Need help with more AWS queries? [We’d be happy to assist](https://bobcares.com/aws-support-services/)\]**  
 

### **Conclusion**

   
To conclude, today we discussed the steps followed by our [Support Engineers](https://bobcares.com/aws-support-services/) to help our customers to install a software package from the Amazon Linux Extras library.

var google\_conversion\_label = "owonCMyG5nEQ0aD71QM";