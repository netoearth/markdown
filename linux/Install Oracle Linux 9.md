[Home](https://oracle-base.com/) » [Articles](https://oracle-base.com/articles) » [Linux](https://oracle-base.com/articles/linux) » Here

**This is a developer preview release of Oracle Linux 9. This article will be updated once OL9 is generally available.**

This article provides a pictorial guide for performing a basic installation of [Oracle Linux 9 (OL9)](http://www.oracle.com/us/technologies/linux/overview/index.html).

-   [Basic Installation](https://oracle-base.com/articles/linux/oracle-linux-9-installation#basic_installation)
-   [Network Configuration](https://oracle-base.com/articles/linux/oracle-linux-9-installation#network_configuration)
-   [SELinux](https://oracle-base.com/articles/linux/oracle-linux-9-installation#selinux)
-   [Firewall](https://oracle-base.com/articles/linux/oracle-linux-9-installation#firewall)
-   [SSH](https://oracle-base.com/articles/linux/oracle-linux-9-installation#ssh)

## Basic Installation

1.  Boot ISO image. Use the up arrow to pick the "Install Oracle Linux 9.0.0" option and hit the return key.
    
    ![Boot](https://oracle-base.com/articles/linux/images/ol9/01-boot.jpg)
    
2.  Select the appropriate language and click the "Continue" button.
    
    ![Language](https://oracle-base.com/articles/linux/images/ol9/02-language.jpg)
    
3.  You are presented with the "Installation Summary" screen. You must complete any marked items before you can continue with the installation. Depending on your requirements, you may also want to alter the default settings by clicking on the relevant links.
    
    ![Installation Summary](https://oracle-base.com/articles/linux/images/ol9/03-installation-summary.jpg)
    
    Click the "Installation Destination" link.
    
4.  If you are happy to use automatic partitioning of the whole disk, click the "Done" button to return to the previous screen.
    
    ![Installation Destination](https://oracle-base.com/articles/linux/images/ol9/04-installation-destination.jpg)
    
    If you want to modify the partitioning configuration, check the "Custom" option, click the "Done" button and work through the partitioning screens.
    
5.  Click the "Root Password" link.
    
    ![Installation Summary](https://oracle-base.com/articles/linux/images/ol9/05-installation-summary.jpg)
    
6.  Enter the root password and confirmation, then click the "Done" button to return to the previous screen.
    
    ![Root Password](https://oracle-base.com/articles/linux/images/ol9/06-root-password.jpg)
    
7.  Click the "User Creation" link. You might have to scroll down to see this.
    
    ![Installation Summary](https://oracle-base.com/articles/linux/images/ol9/07-installation-summary.jpg)
    
8.  Enter the user details, and remember to check the "Make this user administrator" checkbox. When you are finished, click the "Done" button.
    
    ![User Creation](https://oracle-base.com/articles/linux/images/ol9/08-user-creation.jpg)
    
9.  If you want anything other than a minimal installation click on the "Software Selection" link and pick the options you require. Once you have completed your selection, click the "Done" button. If you need any other alterations, click the relevant link and fill in the desired details. Once you have completed your alterations to the default configuration, click the "Begin Installation" button.
    
    ![Installation Summary](https://oracle-base.com/articles/linux/images/ol9/09-installation-summary.jpg)
    
10.  Wait for the installation to complete. When prompted, click the "Reboot System" button.
    
    ![Installation Progress](https://oracle-base.com/articles/linux/images/ol9/10-installation-progress.jpg)
    
11.  We did a "Server with GUI" installation, so we get the default GUI login screen. Login with the user you created earlier by clicking on the username.
    
    ![Login](https://oracle-base.com/articles/linux/images/ol9/11-login.jpg)
    
12.  Enter the password and hit the return key.
    
    ![Login Password](https://oracle-base.com/articles/linux/images/ol9/12-login-password.jpg)
    
13.  We are presented with the main screen.
    
    ![Main Screen](https://oracle-base.com/articles/linux/images/ol9/13-main-screen.jpg)
    
14.  If we had done a minimal installation, but later decide we wanted a GUI desktop, we would log in and issue the following commands from the console to install the desktop packages and reboot.
    
    ```
    # dnf groupinstall -y "Server with GUI" --skip-broken
    # systemctl set-default graphical.target
    # dnf update -y
    # reboot
    ```
    

## Network Configuration

-   If you are using DHCP to configure your network settings, then ignore the following network configuration screens, otherwise click the network icon on the top bar and click the "Wired Connected > Wired Settings" link. You are then presented with the "Settings" screen. Flick the switch to "ON" and click the cog icon next to the network of choice.
    
    ![Network](https://oracle-base.com/articles/linux/images/ol9/14-network.jpg)
    
-   Click the IPv4 option, select the "Manual" method and enter the appropriate IP address and subnet mask, default gateway and primary DNS, then click the "Apply" button.
    
    ![Network Devices - IPv4](https://oracle-base.com/articles/linux/images/ol9/15-network-devices-ipv4.jpg)
    
-   Close the "Network" dialog.

## SELinux

-   If the OS is to be used for an Oracle installation, it is easier if Secure Linux (SELinux) is disabled or switched to permissive. To do this edit the "/etc/selinux/config" file, making sure the SELINUX flag is set as follows.
    
    ```
    SELINUX=permissive
    ```
    
    If SELinux is configured after installation, the server will need a reboot for the change to take effect.
    

## Firewall

-   If the OS is to be used for an Oracle installation, you will need to disable or configure the local firewall, as shown [here](https://oracle-base.com/articles/linux/linux-firewall-firewalld). To disable it, do the following as the "root" user.
    
    ```
    # systemctl stop firewalld
    # systemctl disable firewalld
    ```
    

You can configure it later if you wish.

## SSH

-   Make sure the SSH daemon is started using the following commands.
    
    ```
    # systemctl start sshd.service
    # systemctl enable sshd.service
    ```
    

For more information see:

-   [Oracle Linux](https://www.oracle.com/linux/)

Hope this helps. Regards Tim...

[Back to the Top.](https://oracle-base.com/articles/linux/oracle-linux-9-installation#Top)