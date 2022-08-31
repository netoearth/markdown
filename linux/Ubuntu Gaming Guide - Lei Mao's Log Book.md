## [](https://leimao.github.io/blog/Ubuntu-Gaming-Guide/#Introduction "Introduction")Introduction

Playing games on Ubuntu is not news. Some games on Steam were supported by Linux natively and Wine provides compatibility to Windows games. However, previously it has some restrictions for your OS environment. People often failed due to driver mismatch or missing of libraries, as you can see from the [issues](https://github.com/ValveSoftware/steam-for-linux/issues) on the GitHub steam repository.

My gaming experience on Ubuntu 18.04 was not very successful neither. I was not able to turn on Steam, not even mentioning Windows games in Wine, with the drivers I installed for Deep Learning. Although I have tried many tricks, I was only able to play some Linux native games, such as [MineTest](http://www.minetest.net/), but not other games. To play my favorite games, such as World of Warcraft, CS:GO, and Dota 2, I would have to reboot my computer and switch to Windows 10.

Recently I found that I could start to play the games on Steam and the Windows games in Wine on Ubuntu with the latest official NVIDIA drivers. I also tried the way I configured my Ubuntu for both deep learning and gaming on several different PCs with different hardware configurations, and all of them worked pretty well. So I am very excited to write this blog post on how to configure Ubuntu for both deep learning and gaming. Now that I could do both development and gaming on Ubuntu, I feel I could hardly go back to Windows.

## [](https://leimao.github.io/blog/Ubuntu-Gaming-Guide/#Protocols "Protocols")Protocols

In this protocol, I assume the OS is Ubuntu 18.04. The other Linux distributions might also work but I just did not test them.

### [](https://leimao.github.io/blog/Ubuntu-Gaming-Guide/#NVIDIA-CUDA-and-Driver-Installation "NVIDIA CUDA and Driver Installation")NVIDIA CUDA and Driver Installation

Deep learning developers need to install both NVIDIA driver and CUDA. Usually we install the CUDA package directly from the official NVIDIA driver website, and that package would include the driver package as well. While installing CUDA and driver via this way makes future upgrades easy, it did not support gaming on Ubuntu in my case previously, at least for the driver series 418. Recently, I found upgrading to the latest driver series 430 supports my Ubuntu gaming demand without sacrificing the environment configuration for deep learning.

Here is how you will install the latest NVIDIA CUDA and driver. You go to [`https://developer.nvidia.com/cuda-downloads`](https://developer.nvidia.com/cuda-downloads), select your PC architecture, Linux distribution, and Version, and finally make sure to select `deb (network)` for Installer Type. Then you will get the installation guide to install both the latest CUDA and driver.

[![NVIDIA CUDA and Driver Installation on Ubuntu 18.04](https://leimao.github.io/images/blog/2019-08-07-Ubuntu-Gaming-Guide/cuda_installation.png "NVIDIA CUDA and Driver Installation on Ubuntu 18.04")

NVIDIA CUDA and Driver Installation on Ubuntu 18.04

](https://leimao.github.io/images/blog/2019-08-07-Ubuntu-Gaming-Guide/cuda_installation.png)

Concretely, for Ubuntu 18.04, we run the following commands to install.

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br></pre></td><td><pre><span>$ wget http://developer.download.nvidia.com/compute/cuda/repos/ubuntu1804/x86_64/cuda-repo-ubuntu1804_10.1.168-1_amd64.deb</span><br><span>$ sudo dpkg -i cuda-repo-ubuntu1804_10.1.168-1_amd64.deb</span><br><span>$ sudo apt-key adv --fetch-keys https://developer.download.nvidia.com/compute/cuda/repos/ubuntu1804/x86_64/7fa2af80.pub</span><br><span>$ sudo apt-get update</span><br><span>$ sudo apt-get install cuda</span><br></pre></td></tr></tbody></table>

To verify the installation is successful. We run `nvidia-smi` in the terminal.

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br><span>8</span><br><span>9</span><br><span>10</span><br><span>11</span><br></pre></td><td><pre><span>$ nvidia-smi</span><br><span>Wed Aug  7 21:48:46 2019       </span><br><span>+-----------------------------------------------------------------------------+</span><br><span>| NVIDIA-SMI 430.26       Driver Version: 430.26       CUDA Version: 10.2     |</span><br><span>|-------------------------------+----------------------+----------------------+</span><br><span>| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |</span><br><span>| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |</span><br><span>|===============================+======================+======================|</span><br><span>|   0  GeForce RTX 208...  Off  | 00000000:01:00.0  On |                  N/A |</span><br><span>| 41%   42C    P0    56W / 260W |   3357MiB / 11016MiB |     28%      Default |</span><br><span>+-------------------------------+----------------------+----------------------+</span><br></pre></td></tr></tbody></table>

In this case, we could see that the driver version is `430.26` and the CUDA version is `10.2`.

### [](https://leimao.github.io/blog/Ubuntu-Gaming-Guide/#NVIDIA-CUDA-and-Driver-Upgrade "NVIDIA CUDA and Driver Upgrade")NVIDIA CUDA and Driver Upgrade

Once the NVIDIA CUDA and driver were installed via the way I described above, to upgrade in the future, we could simply run the following command.

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br></pre></td><td><pre><span>$ sudo apt-get update</span><br><span>$ sudo apt-get upgrade cuda</span><br></pre></td></tr></tbody></table>

### [](https://leimao.github.io/blog/Ubuntu-Gaming-Guide/#Steam-Installation "Steam Installation")Steam Installation

Because Steam games use OpenGL as the graphics interface on Ubuntu and it seems that it only supports 32-bit OpenGL. Players with 64-bit systems have to install some weird `i386` libraries, such as `libgl1-mesa-dri:i386` and `libgl1-mesa-glx:i386` etc. There is no such information officially from the Steam installation guide, and it is often very hard to troubleshoot what is missing for Steam for beginners.

Now we could install steam with `apt`, and all the dependencies will also be installed automatically.

<table><tbody><tr><td><pre><span>1</span><br></pre></td><td><pre><span>$ sudo apt-get install steam</span><br></pre></td></tr></tbody></table>

To run Steam, simply run the following command in the terminal.

<table><tbody><tr><td><pre><span>1</span><br></pre></td><td><pre><span>$ steam</span><br></pre></td></tr></tbody></table>

[![Steam on Ubuntu 18.04](https://leimao.github.io/images/blog/2019-08-07-Ubuntu-Gaming-Guide/steam.png "Steam on Ubuntu 18.04")

Steam on Ubuntu 18.04

](https://leimao.github.io/images/blog/2019-08-07-Ubuntu-Gaming-Guide/steam.png)

### [](https://leimao.github.io/blog/Ubuntu-Gaming-Guide/#Lutris-Installation "Lutris Installation")Lutris Installation

[Lutris](https://lutris.net/) supports a variety of Windows applications and games, including Blizzard games, using Wine. To install Lutris, please follow the [instructions](https://lutris.net/downloads/) on the official website.

Concretely, for Ubuntu 18.04, we run the following commands to install.

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br></pre></td><td><pre><span>$ sudo add-apt-repository ppa:lutris-team/lutris</span><br><span>$ sudo apt-get update</span><br><span>$ sudo apt-get install lutris</span><br></pre></td></tr></tbody></table>

To run Lutris, simply run the following command in the terminal.

<table><tbody><tr><td><pre><span>1</span><br></pre></td><td><pre><span>$ lutris</span><br></pre></td></tr></tbody></table>

[![Lutris on Ubuntu 18.04](https://leimao.github.io/images/blog/2019-08-07-Ubuntu-Gaming-Guide/lutris.png "Lutris on Ubuntu 18.04")

Lutris on Ubuntu 18.04

](https://leimao.github.io/images/blog/2019-08-07-Ubuntu-Gaming-Guide/lutris.png)

### [](https://leimao.github.io/blog/Ubuntu-Gaming-Guide/#Others "Others")Others

There are of course other platforms, such as [GOG](https://www.gog.com/). But I am not going to cover all of them.

## [](https://leimao.github.io/blog/Ubuntu-Gaming-Guide/#Games "Games")Games

I played several games with the highest graphics settings on my PC which has a ROG SWIFT PG27UQ 120Hz refresh rate 4K resolution monitor and a NVIDIA RTX 2080 TI GPU.

### [](https://leimao.github.io/blog/Ubuntu-Gaming-Guide/#Dota-2-from-Steam "Dota 2 from Steam")Dota 2 from Steam

[![Dota 2 on Ubuntu 18.04](https://leimao.github.io/images/blog/2019-08-07-Ubuntu-Gaming-Guide/dota2.png "Dota 2 on Ubuntu 18.04")

Dota 2 on Ubuntu 18.04

](https://leimao.github.io/images/blog/2019-08-07-Ubuntu-Gaming-Guide/dota2.png)

### [](https://leimao.github.io/blog/Ubuntu-Gaming-Guide/#World-of-Warcraft-from-Lutris "World of Warcraft from Lutris")World of Warcraft from Lutris

[![World of Warcraft on Ubuntu 18.04](https://leimao.github.io/images/blog/2019-08-07-Ubuntu-Gaming-Guide/wow.png "World of Warcraft on Ubuntu 18.04")

World of Warcraft on Ubuntu 18.04

](https://leimao.github.io/images/blog/2019-08-07-Ubuntu-Gaming-Guide/wow.png)

## [](https://leimao.github.io/blog/Ubuntu-Gaming-Guide/#Updates "Updates")Updates

### [](https://leimao.github.io/blog/Ubuntu-Gaming-Guide/#2-23-2020 "2/23/2020")2/23/2020

Steam only uses 32bit NVIDIA drivers. Sometimes, the NVIDIA driver installed using the official methods described above would miss i386 drivers. If this happens, we have to either roll back to the last NVIDIA driver which comes with i386 packages and support Steam using the official methods or add ppa driver repository and use the latest driver supported which usually comes with i386 packages.

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br></pre></td><td><pre><span>sudo add-apt-repository ppa:graphics-drivers/ppa</span><br><span>sudo apt remove --purge *nvidia*</span><br><span>sudo apt autoremove</span><br><span>sudo apt install nvidia-driver<span>-440</span></span><br><span>sudo reboot</span><br></pre></td></tr></tbody></table>

Make sure that the i386 packages are installed. Otherwise Steam would not run on Ubuntu. For some users who have conflicting added apt repositories, i386 packages would not be installed even though they are in the repository. To resolve this, remove the conflicting NVIDIA apt repositories from Ubuntu Software & Updates.

Do remember to balance your work and game.