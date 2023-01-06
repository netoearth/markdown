“Ubuntu does not have a configuration option to set the scroll speed of the mouse under System Settings. It is a significant disadvantage of this operating system. That’s why users requested to add a new scroll speed control feature through the GNOME development page two years ago.

Still, no official setting is launched in Ubuntu to control the scroll speed of the mouse. Please read this guide if you are an Ubuntu user and want to adjust or change the scroll speed. In this guide, we will explain how to change the mouse scroll speed in Ubuntu.”

Linux users often have problems with changing the mouse scroll speed. As big a problem as it looks, it’s easier to solve and adjust the speed of the mouse. This article will change the mouse scroll speed through two different methods.

### From the Terminal

You only need to run the following curl command, which will directly show a UI on your terminal screen. You can change the mouse scroll speed in Ubuntu from this new pop-up options menu.

bash <(curl \-s http://www.nicknorton.net/mousewheel.sh)

![](https://linuxhint.com/wp-content/uploads/2022/07/word-image-198648-1.png)

This method does not require you to copy the script manually or change the permissions.

### Install Imwheel Manually

Imwheel is a tool by which you can change the behavior of the mouse wheel on a per-program basis. You can use the following steps to change the mouse scroll speed on Ubuntu:

You can install the Imwheel tool on your system by running the following command:

![](https://linuxhint.com/wp-content/uploads/2022/07/word-image-198648-2.png)

Once Imwheel is installed, visit [nicknorton.net](http://www.nicknorton.net/mousewheel.sh) and copy the complete script. Now, paste the script into Text Editor and name the file mousewheel.sh.

![](https://linuxhint.com/wp-content/uploads/2022/07/word-image-198648-3.png)

To set up the .sh file with a suitable code and create the file, go to the location where you have saved your file:

**![](https://linuxhint.com/wp-content/uploads/2022/07/word-image-198648-4.png)**

Now, use the “chmod” command to configure the script for launching the imwheel:

Finally, enter the below command to launch the imwheel from the terminal:

**![](https://linuxhint.com/wp-content/uploads/2022/07/word-image-198648-5.png)**

Doing so will open the dialog box to set mouse scroll speed on your terminal screen, with the help of which you can change your mouse scroll speed in Ubuntu.

![](https://linuxhint.com/wp-content/uploads/2022/07/word-image-198648-6.png)

Using this method, you can change your mouse scroll speed in Ubuntu by following the steps above.

## Wrapping Up

In the above guide, we have explained different methods to change the mouse scroll speed in Ubuntu. The first method is easy and straightforward because you just have to execute a single one to get control of the mouse scroll speed. However, the other one is a manual process using the imwheel tool. So it depends on your system requirements because we have used the above methods in multiple systems. Hence, there are some chances that you may face an issue in using the first method, so you can use the second one.

### About the author

![](https://secure.gravatar.com/avatar/a6dca3a5394c9d066a8078ddf9ede366?s=112&r=g)

A passionate Linux user for personal and professional reasons, always exploring what is new in the world of Linux and sharing with my readers.