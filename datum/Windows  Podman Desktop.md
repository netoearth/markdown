## 1\. Installing Podman Desktop on Windows[](https://podman-desktop.io/docs/Installation/windows-install#1-installing-podman-desktop-on-windows "Direct link to heading")

In order to install the latest Podman Desktop application for Windows, visit the [Downloads](https://podman-desktop.io/downloads/windows) section of this website to download the .exe file.

Simply download the file from the [Downloads](https://podman-desktop.io/downloads/windows) section and open it in your Desktop to install Podman Desktop.

## 2\. Installing Podman (if not already present)[](https://podman-desktop.io/docs/Installation/windows-install#2-installing-podman-if-not-already-present "Direct link to heading")

If you don't have Podman installed in your Windows computer, Podman Desktop will prompt you to do so as soon as you open the application. With the latest update, Podman Desktop will be able to install and configure Podman once you click on the 'Install' button on the home page.

NOTE: Podman Engine on Windows is backed by a virtualized Windows Subsystem for Linux (WSLv2) instance. If you don't have it installed already, Podman Desktop will prompt you to do so when you initialize a Podman Machine for the first time. You can read more about installing Podman on Windows [here](https://github.com/containers/podman/blob/main/docs/tutorials/podman-for-windows.md).

![img1](https://podman-desktop.io/assets/images/homescreen-2f222126a2f033005a67a2f187add164.png)

## 3\. Initialize & Start the Podman Machine[](https://podman-desktop.io/docs/Installation/windows-install#3-initialize--start-the-podman-machine "Direct link to heading")

### a. Default Configurations[](https://podman-desktop.io/docs/Installation/windows-install#a-default-configurations "Direct link to heading")

Once Podman is installed, you will see a toggle button at "Home" window that will allow you to initialize a Podman Machine with default configurations. Simply activate the toggle to proceed.

![img2](https://podman-desktop.io/assets/images/initialize-3b40daad029546f9a0efd9f80ed77011.png)

If this is your first time initializing Podman Machine after installation and you do not have WSLv2 installed, Podman Desktop will prompt you to do so (as shown in image below). Click 'OK' and wait for the process to complete.

![img3](https://podman-desktop.io/assets/images/wsl_before_reboot_1-d76d95b3deaf30fc4c5fd3b9d4bb1604.png)

Your device will reboot during the process. You will be again prompted to proceed.

![img4](https://podman-desktop.io/assets/images/wsl_before_reboot_2-42d412b73831e8fadfd1a87129b4b78b.png)

After reboot, the remaining installation process will be performed. Once WSLv2 is installed, you will find that the Podman Machine is initialized.

![img5](https://podman-desktop.io/assets/images/wsl_after_reboot-f66f246d38f9102043c93fc2eabcf150.png)

After initializing a Podman Machine, you should see a toggle to Run Podman. This will start the Podman Machine upon activation.

![img6](https://podman-desktop.io/assets/images/starting-6fc96092ae7b123f1eb13b8c58baa397.png)

### b. Custom Configurations[](https://podman-desktop.io/docs/Installation/windows-install#b-custom-configurations "Direct link to heading")

In order to initialize a Podman Machine with custom configurations, go to "Preferences" on the menu present in the left-side of the application. Under Resources, you will find Podman. Clicking on it shall load the configuration settings for the machine. Enter the values that deem fit for your purpose and click on the "Create" button.

![img7](https://podman-desktop.io/assets/images/create-a5a2302c3eb2d9cde651ea8830e0f398.png)

Once the machine is created, you can click on the Start button in the Machine Settings to start the Machine.

![img8](https://podman-desktop.io/assets/images/machine-d8cdde6d6087c23415bbb1af76d15d1b.png)

### c. Command Line[](https://podman-desktop.io/docs/Installation/windows-install#c-command-line "Direct link to heading")

Using the following two commands in the command line, you can initialize and start a Podman Machine the classic way!

To initialize the machine, the command is

After which, you can start the machine with the command

**Well that's just it. You shall now be all set to use Podman Desktop on Windows!**