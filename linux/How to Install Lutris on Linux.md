“If you are a gamer, you will appreciate Lutris, a game manager for Linux systems. The good thing with Lutris is that it avails multiple games in its catalog, and you can easily install any game and update them whenever. The installation is simple, and you don’t need to be a techie to work around it.

The Linux package manager does not support Lutris. Therefore, you need to install Wine to accommodate it first, and this guide will offer you a hands-on tutorial on installing Wine and Lutris to set your gaming path straight.”

## Installing Wine on Ubuntu 20.04

Wine is an open-source tool that enables installing Windows applications on a Linux system. Since Lutris is a Windows application, we must use Wine to install it. To install Wine and its dependencies, use the commands below.

First, check the architecture of your system. You can use the command below for that.

If running an x64 bit system, you need to enable 32-bit architecture before installing Wine, using the command below. If your system is 32-bit, skip this step.

<table><tbody><tr><td><p>1<br></p></td><td><p><span>$ </span><span>sudo</span> <span>dpkg</span> - <span>-add-architecture</span> i386</p></td></tr></tbody></table>

Next, run the command below to download the Wine repository key into your Ubuntu system.

<table><tbody><tr><td><p>1<br></p></td><td><p><span>$ </span><span>wget</span> <span>-nc</span> https:<span>//</span>dl.winehq.org<span>/</span>wine-builds<span>/</span>winehq.key</p></td></tr></tbody></table>

![](https://linuxhint.com/wp-content/uploads/2022/07/image13-4.png)

Once the repository key gets downloaded, you need to add it to your system. For that, use the command below.

<table><tbody><tr><td><p>1<br></p></td><td><p><span>$ </span><span>sudo</span> <span>apt-key add</span> winehq.key</p></td></tr></tbody></table>

![](https://linuxhint.com/wp-content/uploads/2022/07/image5-24.png)

The last step to installing Wine is to add the Wine repository using the command below.

<table><tbody><tr><td><p>1<br></p></td><td><p><span>$ </span><span>sudo</span> apt-add-repository <span>'deb https://dl.winehq.org/wine-builds/ubuntu/focal main'</span></p></td></tr></tbody></table>

![](https://linuxhint.com/wp-content/uploads/2022/07/image8-18.png)

With Wine downloaded and installed, you need to update the apt repository and the Wine repository to the latest stable versions using the below commands.

<table><tbody><tr><td><p>1<br>2<br>3<br></p></td><td><div><p>$ <span>sudo</span> apt update</p><p>$ <span>sudo</span> apt <span>install</span> –install-recommends winehq-stable</p></div></td></tr></tbody></table>

![](https://linuxhint.com/wp-content/uploads/2022/07/image1-41.png)

Once everything is updated, proceed to install Lutris.

## Installing Lutris on Ubuntu 20.04

The only dependency needed was Wine, and since we have successfully installed it, we can now go ahead and install Lutris. Installing Lutris is easy. You only need to add its repository and use the package manager to install it.

Start by adding the **Lutris PPA** using the command given.

<table><tbody><tr><td><p>1<br></p></td><td><p><span>$ </span><span>sudo</span> add-apt-repository ppa:lutris-team<span>/</span>lutris</p></td></tr></tbody></table>

![](https://linuxhint.com/wp-content/uploads/2022/07/image2-36.png)

Next, update the system.

Finally, install Lutris.

<table><tbody><tr><td><p>1<br></p></td><td><p><span>$ </span><span>sudo</span> apt <span>install</span> lutris</p></td></tr></tbody></table>

![](https://linuxhint.com/wp-content/uploads/2022/07/image9-15.png)

Note that we are using Ubuntu as our host system to install Lutris. If you are running a different distro, visit the [Lutris page](https://lutris.net/downloads/) to see the installation steps for your OS.

Open Lutris from applications once its installation completes and begin installing games.

![](https://linuxhint.com/wp-content/uploads/2022/07/image3-35-e1657663867926.png)

## Getting Started With Lutris

To use Lutris for downloading games, you need to create an account. If you have one already, log in to it once Lutris opens. On the other hand, [create an account](https://lutris.net/user/register) if you have none.

![](https://linuxhint.com/wp-content/uploads/2022/07/image12-6.png)

Once logged in, visit the official [Lutris website](https://lutris.net/games/) and search for any game you wish to install.

![](https://linuxhint.com/wp-content/uploads/2022/07/image14-3.png)

Click on the **Install** button, which will open a new link to connect to Lutris.

![](https://linuxhint.com/wp-content/uploads/2022/07/image6-23.png)

Also, you will need to select the installation directory for the downloaded game, and once everything checks, click the **Install** button.

![](https://linuxhint.com/wp-content/uploads/2022/07/image7-23.png)

Follow the installation prompts and allows the process to go to completion. If any dependencies exist, click **Continue** to download them.

![](https://linuxhint.com/wp-content/uploads/2022/07/image4-28.png)

Once the game gets downloaded, you can open it by clicking the **Launch** button.

![](https://linuxhint.com/wp-content/uploads/2022/07/image10-13.png)

You will note a start screen for the game, and it’s time to get your hands dirty and have fun!

![](https://linuxhint.com/wp-content/uploads/2022/07/image11-13.png)

The procedure to download any game is the same. Thanks to Lutris, there is a variety of games that you can download.

## Wrap Up

This guide covered how to install Wine which offers a platform for installing the Lutris application on Ubuntu. We’ve seen the different commands you can use and how to start downloading and installing games on Ubuntu. Hopefully, you now understand how you can install Lutris on Ubuntu. Also, check out the Lutris download page for installation steps for different Linux distributions.