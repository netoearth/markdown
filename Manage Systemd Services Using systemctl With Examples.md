**All the major Linux distributions, such as Ubuntu, Fedora, etc use the systemd init system today for managing and controlling various services while the system is running. In this guide, we explain the systemd commands you can use to manage systemd services using systemctl.**

-   [Service management concept](https://www.debugpoint.com/systemd-systemctl-service/#Service_management_concept "Service management concept")
-   [Important Unit files and description](https://www.debugpoint.com/systemd-systemctl-service/#Important_Unit_files_and_description "Important Unit files and description")
-   [Where are the unit files stored?](https://www.debugpoint.com/systemd-systemctl-service/#Where_are_the_unit_files_stored "Where are the unit files stored?")
    -   [Examples](https://www.debugpoint.com/systemd-systemctl-service/#Examples "Examples")
-   [Manage Systemd Services Using systemctl](https://www.debugpoint.com/systemd-systemctl-service/#Manage_Systemd_Services_Using_systemctl "Manage Systemd Services Using systemctl ") 
    -   [List Services](https://www.debugpoint.com/systemd-systemctl-service/#List_Services "List Services")
    -   [Show Service Status](https://www.debugpoint.com/systemd-systemctl-service/#Show_Service_Status "Show Service Status")
    -   [Start a service](https://www.debugpoint.com/systemd-systemctl-service/#Start_a_service "Start a service")
    -   [Stop a service](https://www.debugpoint.com/systemd-systemctl-service/#Stop_a_service "Stop a service")
    -   [Restart and reload a service](https://www.debugpoint.com/systemd-systemctl-service/#Restart_and_reload_a_service "Restart and reload a service")
    -   [Enable and disable a service](https://www.debugpoint.com/systemd-systemctl-service/#Enable_and_disable_a_service "Enable and disable a service")

## Service management concept

Systemd is the init system and service manager of modern Linux systems. The init system is the first process that starts when you power on your system and keeps running until it is shut down.

The main purpose of systemd as init system is to initialize various system components just after the Linux kernel is booted at the beginning. Additionally, when the system is running, it also manages various services and daemons such as ssh daemon, network manager, etc.

The systemd works on the unit files. There are different types of unit files based on the purpose and resources. For example, the services have unit files with `.service` extensions, whereas device unit files have `.device` extensions. The [systemctl](https://www.man7.org/linux/man-pages/man1/systemctl.1.html) command of systemd is used to manage the unit files. 

## Important Unit files and description

<table><tbody><tr><td><strong>Units</strong></td><td><strong>extensions</strong></td><td><strong>descriptions</strong></td></tr><tr><td>Service unit</td><td>.service</td><td>A system service.</td></tr><tr><td>Target unit</td><td>.target</td><td>A group of systemd units.</td></tr><tr><td>Automount unit</td><td>.automount</td><td>A file system automount point.</td></tr><tr><td>Device unit</td><td>.device</td><td>A device file recognized by the kernel.</td></tr><tr><td>Mount unit</td><td>.mount</td><td>A file system mount point.</td></tr><tr><td>Path unit</td><td>.path</td><td>A file or directory in a file system.</td></tr><tr><td>Scope unit</td><td>.scope</td><td>An externally created process.</td></tr><tr><td>Slice unit</td><td>.slice</td><td>A group of hierarchically organized units that manage system processes.</td></tr><tr><td>Snapshot unit</td><td>.snapshot</td><td>A saved state of the systemd manager.</td></tr><tr><td>Socket unit</td><td>.socket</td><td>An inter-process communication socket.</td></tr><tr><td>Swap unit</td><td>.swap</td><td>A swap device or a swap file.</td></tr><tr><td>Timer unit</td><td>.timer</td><td>A systemd timer.</td></tr></tbody></table>

## Where are the unit files stored?

The unit files are stored in two places ideally in a Linux system. The files created at run time, and boot time are stored in `/run/systemd/system/`. The unit files you create manually (e.g. `systemctl enable` command) is stored at `/etc/systemd/system/`.

The `/etc/systemd/system/` path takes precedence over the run time unit files present at `/run/systemd/system/`.

There is another path where systemd keeps the system’s copy of the service unit files – `/lib/systemd/system`.

### Examples

![run systemd system - unit files](https://www.debugpoint.com/wp-content/uploads/2020/12/run-systemd-system-unit-files-1024x208.jpg)

run systemd system – unit files

![etc systemd system - user enabled unit files](https://www.debugpoint.com/wp-content/uploads/2020/12/etc-systemd-system-user-enabled-unit-files-1024x360.jpg)

etc systemd system – user enabled unit files

First, let’s look at how you can list the services, check their status, etc. This is required if you want to understand the state of your system for investigation purposes. 

To list out all systemd active units, you can use the below command.

```
systemctl list-units 
```

![systemctl list-units (1)](https://www.debugpoint.com/wp-content/uploads/2020/12/systemctl-list-units-1-1024x199.jpg)

systemctl list-units (1)

![systemctl list-units (2)](https://www.debugpoint.com/wp-content/uploads/2020/12/systemctl-list-units-2-1024x201.jpg)

systemctl list-units (2)

This command gives a large output with the below headings. Scroll using arrow keys (up, down, left, and right) to view the entire output. 

-   UNIT – Name of the systemd unit
-   Load – Reflects whether systemd parsed the config file of the unit and it is loaded in memory
-   ACTIVE – Unit state (high-level status). 
-   SUB – Unit state (low-level status)
-   DESCRIPTION – Description of the unit.

For example, the ACTIVE value can be active whereas the SUB level can be many such as running, listening, dead, and active.

More example commands:

```
systemctl list-units --all
```

```
systemctl list-units --all --state=inactive
```

### List Services

To list all units of type service, use the below command. 

```
systemctl list-units --type=service
```

![list loaded services](https://www.debugpoint.com/wp-content/uploads/2020/12/list-loaded-services-1024x255.jpg)

list loaded services

You can combine additional parameters as well. For example, if you want to find out the services that are running, use the below command.

```
systemctl list-units --type=service --state=running
```

![list all running services](https://www.debugpoint.com/wp-content/uploads/2020/12/list-all-running-services-1024x455.jpg)

list all running services

The other combinations of `state` values which you can use are –

-   active
-   inactive
-   running
-   dead
-   exited
-   plugged
-   tentative
-   listening
-   waiting

![more examples of state values](https://www.debugpoint.com/wp-content/uploads/2020/12/more-examples-of-state-values-1024x590.jpg)

more examples of state values

Additionally, you can list all unit files using the below commands. 

```
systemctl list-unit-files
```

Managing unit files are a different topic which I will cover in the next article.

### Show Service Status

To find out the status of a service, use the `below command. For this guide,` I have used the `NetworkManager` service, which controls the network management of a system. **You can replace the NetworkManager with your own service name**. **All the following commands require sudo privilege**. If you do not know the service name, check the above commands to display all service units, then filter out using the `grep` command.

```
systemctl status NetworkManager.service
```

Note that you can omit the .service at the end of the service name. The systemd is clever enough to understand the command!

![status of a service](https://www.debugpoint.com/wp-content/uploads/2020/12/status-of-a-service-1024x297.jpg)

status of a service

In the above example, you can see all information is provided in the command. It also has the CGroup value to identify the user groups, which is needed in a server environment.

**Some more example commands:**

```
systemctl is-active NetworkManager.service
```

```
systemctl is-enabled NetworkManager.service
```

```
systemctl is-failed NetworkManager.service
```

### Start a service

Starting a systemd service is easy. Run the below command with start and service name. The command starts the service.

```
sudo systemctl start NetworkManager.service
```

### Stop a service

Using the stop parameter, you can stop a service. For example:

```
sudo systemctl stop NetworkManager.service
```

Be cautious while stopping a service. Make sure you know what you are doing.

### Restart and reload a service

The systemd provides options to configure a service so that it can restart or reload without restarts. The restart parameter restarts the services, and the reload uses configuration files

```
sudo systemctl restart NetworkManager.service
```

```
sudo systemctl reload NetworkManager.service
```

![restart a service](https://www.debugpoint.com/wp-content/uploads/2020/12/restart-a-service.jpg)

restart a service

### Enable and disable a service

If you created a new service or installed applications that work via services, starting them using the start command will not enable it again in the next boot. So, if you want the systemd to start a service when the system boots, you need to use the enable command. The same is true for disabling when you want to disable a service at boot time.

```
sudo systemctl enable NetworkManager.service
```

The enable command creates a symlink from /etc/systemd/system to the target locations.

To disable the service, use the below command.

```
sudo systemctl disable NetworkManager.service
```

The [systemd](https://www.debugpoint.com/tag/systemd) services and their associated commands have many additional options as well. The systemd is a robust and important component of modern Linux systems. I hope this guide helps you to troubleshoot your desktop or servers when required in Linux. There are many additional features that systemd provides – such as target files, modifying the service unit files, etc. – which I will cover in the next set of articles. All the articles are tagged with systemd for easy browsing.