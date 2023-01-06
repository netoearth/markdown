[Home](https://oracle-base.com/) » [Articles](https://oracle-base.com/articles) » [Linux](https://oracle-base.com/articles/linux) » Here

This article provides a pictorial guide for performing a basic installation of [Fedora 37 (F37)](http://www.fedoraproject.org/).

-   [Basic Installation](https://oracle-base.com/articles/linux/fedora-37-installation#basic-installation)
-   [Network Configuration](https://oracle-base.com/articles/linux/fedora-37-installation#network-configuration)
-   [SELinux](https://oracle-base.com/articles/linux/fedora-37-installation#selinux)
-   [Firewall](https://oracle-base.com/articles/linux/fedora-37-installation#firewall)
-   [SSH](https://oracle-base.com/articles/linux/fedora-37-installation#ssh)

## Basic Installation

1.  Boot from the DVD or ISO image. Use the up arrow to pick the "Install Fedora 36" option and hit the return key.
    
    ![Boot](https://oracle-base.com/articles/linux/images/fedora37/01-boot.jpg)
    
2.  Select the appropriate language, then click the "Continue" button.
    
    ![Language](https://oracle-base.com/articles/linux/images/fedora37/02-language.jpg)
    
3.  You are presented with the "Installation Summary" screen. You must complete any marked items before you can continue with the installation. Depending on your requirements, you may also want to alter the default settings by clicking on the relevant links.
    
    ![Installation Summary](https://oracle-base.com/articles/linux/images/fedora37/03-installation-summary.jpg)
    
    Click the "Installation Destination" link.
    
4.  If you are happy to use automatic partitioning of the whole disk, click the "Done" button to return to the previous screen.
    
    ![Installation Destination](https://oracle-base.com/articles/linux/images/fedora37/04-installation-destination.jpg)
    
    If you want to modify the partitioning configuration, select the "Custom" option and click the "Done" button to work through the partitioning screens.
    
5.  When complete click the "Root Password" link.
    
    ![Installation Summary](https://oracle-base.com/articles/linux/images/fedora37/04.5-installation-summary.jpg)
    
6.  Enter the password details and click the "Done" button.
    
    ![Root Password](https://oracle-base.com/articles/linux/images/fedora37/05-root-password.jpg)
    
7.  Click the "User Creation" link. You might have to scroll to see this.
    
    ![Installation Summary](https://oracle-base.com/articles/linux/images/fedora37/05.5-installation-summary.jpg)
    
8.  Enter the user details and click the "Done" button.
    
    ![User Creation](https://oracle-base.com/articles/linux/images/fedora37/06-user-creation.jpg)
    
9.  Once you have completed your alterations to the default configuration, click the "Begin Installation" button.
    
    ![Installation Summary](https://oracle-base.com/articles/linux/images/fedora37/07-installation-summary.jpg)
    
10.  Wait for the installation to complete and click the "Reboot System" button.
    
    ![Complete](https://oracle-base.com/articles/linux/images/fedora37/08-complete.jpg)
    
11.  Login using the username "root" and the password you specified earlier.
    
    ![Login](https://oracle-base.com/articles/linux/images/fedora37/09-login.jpg)
    
12.  We did a minimal installation, but if you want a GUI desktop with the usual desktop items, log in and issue the following commands from the console to install the desktop packages and reboot.
    
    ```
    # dnf groupinstall -y "Fedora Workstation" --skip-broken
    # systemctl set-default graphical.target
    # dnf update -y
    # reboot
    ```
    

## Network Configuration

The specific instructions will vary a little, depending on the window manager you have installed.

-   If you are using DHCP to configure your network settings, then ignore the following network configuration screens, otherwise click the network icon on the top bar, expand the "Wired" menu item and select the "Wired Settings" link. You are then presented with the "Settings" screen. Flick the switch to on and click the cog icon next to the network of choice.
    
    ![Network](https://oracle-base.com/articles/linux/images/fedora37/10-network.jpg)
    
-   Click the "IPv4" section, select the "Manual" method and enter the appropriate IP address and subnet mask, default gateway and primary DNS, then click the "Apply" button. You may have to scroll down to see some of the options.
    
    ![Network Devices - IPv4](https://oracle-base.com/articles/linux/images/fedora37/11-network-devices-ipv4.jpg)
    
-   Close the network dialog.

## SELinux

-   If the OS is to be used for an Oracle installation, it is easier if Secure Linux (SELinux) is disabled or switched to permissive. To do this edit the "/etc/selinux/config" file, making sure the SELINUX flag is set as follows.
    
    ```
    SELINUX=permissive
    ```
    
    If SELinux is configured after installation, the server will need a reboot for the change to take effect.
    

## Firewall

-   If the OS is to be used for an Oracle installation, it is easier if the firewall is disabled. This can be done by issuing the following commands from a terminal window as the "root" user.
    
    ```
    # systemctl stop firewalld
    # systemctl disable firewalld
    ```
    

You can install and configure it later if you wish.

## SSH

-   Make sure the SSH daemon is started using the following commands.
    
    ```
    # systemctl start sshd.service
    # systemctl enable sshd.service
    ```
    

For more information see:

-   [Fedora](http://www.fedoraproject.org/)

Hope this helps. Regards Tim...

[Back to the Top.](https://oracle-base.com/articles/linux/fedora-37-installation#Top)