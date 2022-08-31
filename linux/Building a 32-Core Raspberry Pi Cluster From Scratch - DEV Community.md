A Raspberry Pi is a mini-computer board to which you can connect a monitor, mouse, and keyboard, and install a Linux-based operating system with a GUI. Or you can use it in “headless” mode with no GUI and run, for example, a database server. There are [many usages](https://jira.mariadb.org/browse/MDEV-28632) you can give a Raspberry Pi—from building a Minecraft server to smart mirrors, the possibilities are endless.

[![Mini-computer board](https://res.cloudinary.com/practicaldev/image/fetch/s--Eg5wV6S8--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dz2cdn1.dzone.com/storage/temp/15923637-raspberry-pi-4-model-b.jpg)](https://res.cloudinary.com/practicaldev/image/fetch/s--Eg5wV6S8--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dz2cdn1.dzone.com/storage/temp/15923637-raspberry-pi-4-model-b.jpg)

Since I started to discover MariaDB and learned about database clusters, [Distributed SQL](https://dzone.com/refcardz/getting-started-with-distributed-sql), and [Xpand](https://mariadb.com/products/enterprise/xpand/), the idea of building a Raspberry Pi cluster has been in the back of my head. A cluster like this is a great way to experiment with distributed systems.

In this article, I’ll show you how to build a Raspberry Pi cluster with:

-   8 nodes
    
-   32 cores
    
-   64 GB of RAM
    
-   2TB of storage
    

## [](https://dev.to/alejandro_du/building-a-32-core-raspberry-pi-cluster-from-scratch-29ah#what-you-need)What You Need

If you want to build your own cluster, you don’t have to buy 8 Raspberry Pi devices. Even one device is enough to learn stuff. The instructions in this article are useful even if you plan to configure only one [Raspberry Pi.](https://dzone.com/articles/raspberry-pi-essentials) My advice, however, is to have at least three devices to build your cluster if you truly want to boost your skills in [Linux](https://www.linux.org/) administration, [Ansible](https://www.ansible.com/), [Docker](https://dzone.com/refcardz/getting-started-with-docker-1), [Kubernetes](https://dzone.com/refcardz/advanced-kubernetes), [database clusters](https://mariadb.com/kb/en/what-is-mariadb-galera-cluster/)… you name it!

[![8 x Raspberry Pi 4 Model B (8 GB RAM)](https://res.cloudinary.com/practicaldev/image/fetch/s--dHo00Tff--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dz2cdn1.dzone.com/storage/temp/15923639-8-pies.jpg)](https://res.cloudinary.com/practicaldev/image/fetch/s--dHo00Tff--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dz2cdn1.dzone.com/storage/temp/15923639-8-pies.jpg)

With that in mind, please replace 8 with whatever is your number. Do the same with the specifications of the Raspberry Pi devices if you have different models with less or more RAM or storage. Here’s what I used:

-   8 x [Raspberry Pi 4 Model B](https://www.raspberrypi.com/products/raspberry-pi-4-model-b) (8 GB RAM)
    
-   1 x [Anker 60W Desktop Charger](https://www.amazon.com/Anker-10-Port-Charger-PowerPort-iPhone/dp/B00YRYS4T4) (10-Ports)
    
-   8 x [Short USB Type C cable](https://www.baseusworld.com/products/baseus-simple-hw-quick-charge-charging-data-cable-usb-for-type-c-40w-23cm) (5A, 40W, 23cm)
    
-   8 x [MicroSD card](https://www.westerndigital.com/en-ie/products/memory-cards/sandisk-extreme-pro-uhs-i-microsd#SDSQXCD-256G-GN6MA) (256 GB)
    
-   2 x [Raspberry Pi cluster case](https://www.fruugo.fi/raspberry-pi-multilayer-shell-kit-with-cooling-fan-and-raspberry-pi-heatsink/p-74551370-149649869?language=en) (4 layers with cooling fan)
    

Adapt the quantities to your setup. You can either get one or more [Raspberry Pi power supplies](https://www.raspberrypi.com/products/type-c-power-supply/) if you have very few devices (say 1 or 2) or a multi USB charger. If you go with a multi USB charger, make sure that each port can deliver at least 2.4Am (5V). Pick microSD cards that fit your goals and budget as well. I recommend going with at least 32 GB SD cards. On the cables and case side, pick anything that accommodates your setup. You don’t necessarily need cooling fans, but I recommend them if you plan to leave your devices on for prolonged periods of time.

I also recommend getting all the ingredients before starting “cooking," especially if you want to use multiple Raspberry Pi devices.

## [](https://dev.to/alejandro_du/building-a-32-core-raspberry-pi-cluster-from-scratch-29ah#installing-raspberry-pi-os-headless)Installing Raspberry Pi OS (Headless)

The operating system (OS) we’ll use is [Raspberry Pi OS](https://en.wikipedia.org/wiki/Raspberry_Pi_OS) a Debian-based OS optimized for Raspberry Pi boards. Raspberry Pi OS is installed on the microSD cards, using your computer. You later connect these microSD cards to your Raspberry Pi devices and boot them.

Get all the microSD cards and Raspberry Pi boxes on your desk. You’ll need an SD card adapter (they normally come with microSD cards) or an SD card USB reader if your working computer doesn’t have a slot for SD cards. Bring a sharpie as well.

Download the [Raspberry Pi Imager](https://www.raspberrypi.com/software/) application and install it on your computer. Take a microSD card and connect it to your computer. Open the Raspberry Pi Imager application and click on **CHOOSE OS**. From the list, click on **Raspberry Pi OS (other)** and select **Raspberry Pi OS Lite (64-bit)** if your Raspberry Pi devices are 64-bit or **Raspberry Pi OS Lite (32-bit)** otherwise:

[![Raspberry Pi Imager](https://res.cloudinary.com/practicaldev/image/fetch/s--C5meboNa--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dz2cdn1.dzone.com/storage/temp/15923640-raspberry-pi-os-lite-64.png)](https://res.cloudinary.com/practicaldev/image/fetch/s--C5meboNa--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dz2cdn1.dzone.com/storage/temp/15923640-raspberry-pi-os-lite-64.png)

Click on **CHOOSE STORAGE** and select the microSD card. Double-check that you selected the correct drive, and click on the “gear” icon (advanced options). For hostname use **rpi01**, or something similar. You’ll be naming the devices as **rpi01**, **rp02**, **rp03**, etc.

Check the **Enable SSH** and **Use password authentication** options. Set a username (I’ll leave the default **pi**) and set a secure password.

Check the **Configure wireless LAN** option and enter the name and password of your WiFi connection. Configure your locale settings as well.

Since we have to do this once with each microSD card, make sure to set the **Image customization** options field to: **to always use**. This way the settings will be saved and the process will be easier for the next cards:

[![Raspberry Pi Imager Advanced Options](https://res.cloudinary.com/practicaldev/image/fetch/s--lGAFnNjj--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dz2cdn1.dzone.com/storage/temp/15923641-advanced-options.png)](https://res.cloudinary.com/practicaldev/image/fetch/s--lGAFnNjj--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dz2cdn1.dzone.com/storage/temp/15923641-advanced-options.png)

Now you can click **SAVE** and then **WRITE** to start the process. Once completed, eject the card and connect it to one of the Raspberry Pi devices, put it back into its box, and mark the box with the number **01**. Repeat this process for all of the devices.

## [](https://dev.to/alejandro_du/building-a-32-core-raspberry-pi-cluster-from-scratch-29ah#assembling-the-cluster)Assembling the Cluster

Depending on what kind of cluster case you have, the assembling process might vary. Follow the instructions that come with your case or look for photographs online to get a clear idea of how the devices should be placed in the layers.

[![Assembling the cluster](https://res.cloudinary.com/practicaldev/image/fetch/s--utG-sT-9--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dz2cdn1.dzone.com/storage/temp/15923642-assembling-1.jpg)](https://res.cloudinary.com/practicaldev/image/fetch/s--utG-sT-9--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dz2cdn1.dzone.com/storage/temp/15923642-assembling-1.jpg)

I recommend looking ahead to decide which parts to assemble first. For example, with the case I picked, I had to install the cooling fans before mounting the Raspberry Pi devices on the layers.

Once you mount a Raspberry Pi, use a sharpie to write its number on the layer. This will help you in the future if you have to take a microSD card out for reconfiguration or any other administrative tasks that involve touching the hardware.

[![All of the devices mounted](https://res.cloudinary.com/practicaldev/image/fetch/s--sZjOQ-WH--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dz2cdn1.dzone.com/storage/temp/15923643-assembling-2.jpg)](https://res.cloudinary.com/practicaldev/image/fetch/s--sZjOQ-WH--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dz2cdn1.dzone.com/storage/temp/15923643-assembling-2.jpg)

Once you have all the devices mounted, connect the USB cables to the Raspberry Pi devices and the multi-USB charger. You can optionally use [Power over Ethernet](https://dzone.com/articles/raspberry-pi-4-gives-greater-opportunity-for-iot) and a [network switch](https://en.wikipedia.org/wiki/Network_switch) instead of the USB charger if you prefer, but I’ll live that for you to explore: definitely worth checking. You can either have the charger (or network switch if you go for it) next to your Raspberry Pi rack or attached to it. I used rubber bands to keep the charger attached to the side of the case. It worked fine.

With the USB cables plugged in, you are ready to start all those mini-computers! Connect the charger and switch it on. Depending on the Raspberry Pi models that you use, they might take some time to boot, so be patient. Meanwhile, enjoy the LEDs flashing and fans starting.

[![LEDs flashing and fans starting](https://res.cloudinary.com/practicaldev/image/fetch/s--gXzq9SMK--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dz2cdn1.dzone.com/storage/temp/15923644-24-cores-raspberry-pi-cluster.jpg)](https://res.cloudinary.com/practicaldev/image/fetch/s--gXzq9SMK--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dz2cdn1.dzone.com/storage/temp/15923644-24-cores-raspberry-pi-cluster.jpg)

## [](https://dev.to/alejandro_du/building-a-32-core-raspberry-pi-cluster-from-scratch-29ah#connecting-through-ssh)Connecting Through SSH

If everything went well, you should be able to reach the Raspberry Pi devices through SSH. Give it a try by running the following from your computer:  

```
ssh pi@rpi01.local
```

Enter fullscreen mode Exit fullscreen mode

Introduce your password and answer **yes** to add the device to the list of known hosts. Congrats! Your cluster is live now.

[![Cluster live](https://res.cloudinary.com/practicaldev/image/fetch/s--zZjjnyiA--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dz2cdn1.dzone.com/storage/temp/15923645-ssh.png)](https://res.cloudinary.com/practicaldev/image/fetch/s--zZjjnyiA--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dz2cdn1.dzone.com/storage/temp/15923645-ssh.png)

You may want to run commands to check the hardware information. For example:

-   `lscpu`: Information about the CPU architecture
    
-   `df -H`: File system disk space usage
    
-   `sudo fdisk -l`: Partition information
    
-   `free -m`: Amount of used, free and total amount of RAM
    
-   `cat /proc/version`: Linux kernel information
    

## [](https://dev.to/alejandro_du/building-a-32-core-raspberry-pi-cluster-from-scratch-29ah#manual-configuration-with-raspiconfig)Manual Configuration With raspi-config

If you have one Raspberry Pi, you can connect to it through SSH as described above and use the `raspi-config` utility program to configure various settings. You can modify the WiFi connection (**S1**), change the hostname (**S4**) and user password (**S3**), and expand the filesystem (**A1**), among many other things.

[![Manual Configuration With raspi-config](https://res.cloudinary.com/practicaldev/image/fetch/s--qnwYzVIS--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dz2cdn1.dzone.com/storage/temp/15923646-raspi-config.png)](https://res.cloudinary.com/practicaldev/image/fetch/s--qnwYzVIS--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dz2cdn1.dzone.com/storage/temp/15923646-raspi-config.png)

## [](https://dev.to/alejandro_du/building-a-32-core-raspberry-pi-cluster-from-scratch-29ah#automated-configuration-with-ansible)Automated Configuration With Ansible

If you have more than one Raspberry Pi in your cluster, you might want to automate the configuration process using a tool like [Ansible](https://www.ansible.com/). With Ansible, you can run a command on multiple machines without having to perform repetitive tasks. For example, let’s say we want to expand the filesystem on each Raspberry Pi. This can be done from the command line as follows:  

```
sudo raspi-config --expand-rootfs
```

Enter fullscreen mode Exit fullscreen mode

Without automation, you’d have to SSH to **rpi01.local**, run the command above, and end the SSH session. You’d have to repeat all these steps for **rpi02.local**, **pr03.local**, **rp04.local**, and so forth. Instead, you can simply tell Ansible to run the command for you on all the machines. Let’s see how to do this.

You have to install Ansible on a _control node_ that is connected to your local network. It can be one of the Raspberry Pi devices, your working laptop, or any other machine as long as it runs a Unix-based operating system like Linux or macOS. I happen to have an old laptop that I [repurposed as a dedicated database](https://dzone.com/articles/your-old-laptop-is-your-new-database-server) connected to my network so I used it as the control node (I’ll show how to install a database with replication in the Raspberry Pi devices in a future article). I won’t go through the instructions on how to install Ansible since the process is different on different operating systems. Check the [official documentation](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html) and install it on your laptop or any device that you want to assign as a control node.

Before you start, I recommend generating an example configuration file that you can tweak later as you wish:  

```
sudo su
ansible-config init --disabled -t all > /etc/ansible/ansible.cfg
exit
```

Enter fullscreen mode Exit fullscreen mode

In this file, disable host key checking to simplify the process. Don’t do that in production environments! Change the following line:  

```
;host_key_checking=True
```

Enter fullscreen mode Exit fullscreen mode

to this:  

```
host_key_checking=False
```

Enter fullscreen mode Exit fullscreen mode

Next, you need to define an inventory. This is a list of machines that you want to control with Ansible. The inventory is defined in the **/etc/ansible/hosts** file. For example, you could add the following machines to the inventory using their IP addresses or hostnames:  

```
[mymachines]
foo.example.com
bar.example.com
192.0.2.50
192.0.2.51
```

Enter fullscreen mode Exit fullscreen mode

If the machines have some kind of pattern in their IP addresses or hostnames, you can alternatively use ranges. This is what you can do with your Raspberry Pi devices. Edit the **/etc/ansible/hosts** file by adding the following to the end:  

```
[rpis]
rpi[01-08].local
```

Enter fullscreen mode Exit fullscreen mode

Change the pattern to match your hostnames and numbers. This is equivalent to:  

```
[rpis]
rpi01.local
rpi02.local
rpi03.local
rpi04.local
rpi05.local
rpi06.local
rpi07.local
rpi08.local
```

Enter fullscreen mode Exit fullscreen mode

In this inventory `rpis` is an arbitrary name that you can use to refer to all the Raspberry Pi devices when running commands on them using Ansible.

You need to configure the SSH username that Ansible will use when connecting to the machines. Add the following to the **/etc/ansible/hosts** file:  

```
[rpis:vars]
ansible_user=pi
```

Enter fullscreen mode Exit fullscreen mode

Time to control de machines. A good start is to _ping_ them:  

```
ansible rpis -m ping --ask-pass
```

Enter fullscreen mode Exit fullscreen mode

Type the SSH password. You should see in the output how the “ping” is answered by a “pong” for each machine (I’m showing only one here):  

```
rpi01.local | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
```

Enter fullscreen mode Exit fullscreen mode

Let’s back to the filesystem expansion (although, I think nowadays this is not required anymore, let me know in the comments if you can confirm this). To perform this action on all the machines, run:  

```
ansible rpis -m shell -a "raspi-config --expand-rootfs" --become --ask-pass
```

Enter fullscreen mode Exit fullscreen mode

And as simple as that, all your Raspberry Pi devices have an expanded filesystem. This is a good time to update the system on all the machines:  

```
ansible rpis -m shell -a "apt update -y" --become --ask-pass
ansible rpis -m shell -a "apt upgrade -y" --become --ask-pass
```

Enter fullscreen mode Exit fullscreen mode

You can rebook all the devices as follows:  

```
ansible rpis -m shell -a "reboot" --become --ask-pass
```

Enter fullscreen mode Exit fullscreen mode

And once you are done and want to safely turn all the machines off, just run:  

```
ansible rpis -m shell -a "poweroff" --become --ask-pass
```

Enter fullscreen mode Exit fullscreen mode

## [](https://dev.to/alejandro_du/building-a-32-core-raspberry-pi-cluster-from-scratch-29ah#whats-next)What's Next?

This is only the beginning. You now have a cluster of small computers ready to be controlled by Ansible. You can try setting up any kind of server software, including web servers or databases ([see this example](https://dzone.com/articles/set-up-a-dedicated-database-server-on-raspberry)). There also is much more about Ansible. I merely scratched the surface here using [ad-hoc commands](https://docs.ansible.com/ansible/latest/user_guide/intro_adhoc.html), but you can for example create [playbooks](https://docs.ansible.com/ansible/latest/user_guide/index.html#writing-tasks-plays-and-playbooks) that contain the state in which you want your cluster to be. Explore the [documentation](https://docs.ansible.com/) and have fun!