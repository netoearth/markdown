You can configure several parameters or tunables of Linux (the kernel) to control its behavior, either at boot or on demand while the system is running. **sysctl** is a widely-used command-line utility for modifying or configuring kernel parameters at runtime. You can find the kernel tunables listed under the **/proc/sys/** directory.

It is powered by **procfs** ([proc file system](https://www.tecmint.com/exploring-proc-file-system-in-linux/ "Exploring /proc File System in Linux")), a pseudo file system in Linux and other Unix-like operating systems that provides an interface to kernel data structures. It presents information about processes and additional system information.

The following are 10 useful **sysctl** commands examples that you can use when administering a running Linux system. Note that you need root privileges to run the **sysctl** command, otherwise, use the [sudo command](https://www.tecmint.com/su-vs-sudo-and-how-to-configure-sudo-in-linux/ "Difference Between su and sudo in Linux") when invoking it.  

Table of Contents

1

-   [sysctl Command Examples in Linux](https://www.tecmint.com/sysctl-command-examples/#sysctl_Command_Examples_in_Linux "sysctl Command Examples in Linux")
-   [1\. List All Kernel Parameters in Linux](https://www.tecmint.com/sysctl-command-examples/#1_List_All_Kernel_Parameters_in_Linux "1. List All Kernel Parameters in Linux")
-   [3\. List All Kernel Variable Names](https://www.tecmint.com/sysctl-command-examples/#3_List_All_Kernel_Variable_Names "3. List All Kernel Variable Names")
-   [3\. Find Specific Kernel Variables in Linux](https://www.tecmint.com/sysctl-command-examples/#3_Find_Specific_Kernel_Variables_in_Linux "3. Find Specific Kernel Variables in Linux")
-   [4\. List All Kernel Variables Including Deprecated](https://www.tecmint.com/sysctl-command-examples/#4_List_All_Kernel_Variables_Including_Deprecated "4. List All Kernel Variables Including Deprecated")
-   [5\. List Specific Kernel Variable Value](https://www.tecmint.com/sysctl-command-examples/#5_List_Specific_Kernel_Variable_Value "5. List Specific Kernel Variable Value")
-   [6\. Write Kernel Variable Temporarily](https://www.tecmint.com/sysctl-command-examples/#6_Write_Kernel_Variable_Temporarily "6. Write Kernel Variable Temporarily")
-   [7\. Write Kernel Variable Permanently](https://www.tecmint.com/sysctl-command-examples/#7_Write_Kernel_Variable_Permanently "7. Write Kernel Variable Permanently")
-   [8\. Reload sysctl.conf Variables in Linux](https://www.tecmint.com/sysctl-command-examples/#8_Reload_sysctlconf_Variables_in_Linux "8. Reload sysctl.conf Variables in Linux")
-   [9\. Reload Settings from Custom Configuration Files](https://www.tecmint.com/sysctl-command-examples/#9_Reload_Settings_from_Custom_Configuration_Files "9. Reload Settings from Custom Configuration Files")
-   [10\. Reload Settings that Match Pattern](https://www.tecmint.com/sysctl-command-examples/#10_Reload_Settings_that_Match_Pattern "10. Reload Settings that Match Pattern")

### sysctl Command Examples in Linux

In this guide, we will explain 10 sysctl practical command examples you can use on a Linux system.

### 1\. List All Kernel Parameters in Linux

To list all currently available kernel parameters, run the sysctl command with the `-a` or `--all` flag as shown.

```
$ sudo sysctl -a
OR
$ sudo sysctl --all

```

The variables are displayed in this format:

```
<tunable class>.<tunable> = <value>

```

For example,

```
kernel.ostype = Linux

```

[![Check Kernel Parameters in Linux](https://www.tecmint.com/wp-content/uploads/2023/02/Check-Kernel-Parameters-in-Linux.png)](https://www.tecmint.com/wp-content/uploads/2023/02/Check-Kernel-Parameters-in-Linux.png)

Check Kernel Parameters in Linux

### 3\. List All Kernel Variable Names

To only print variable names without their values, use the `-N` option as shown.

```
$ sudo sysctl -a -N

```

[![Check Kernel Variable Names in Linux](https://www.tecmint.com/wp-content/uploads/2023/02/Check-Kernel-Variable-Names-in-Linux.png)](https://www.tecmint.com/wp-content/uploads/2023/02/Check-Kernel-Variable-Names-in-Linux.png)

Check Kernel Variable Names in Linux

### 3\. Find Specific Kernel Variables in Linux

To find a specific variable, you can filter the output of **sysctl** via the [grep command](https://www.tecmint.com/12-practical-examples-of-linux-grep-command/ "Linux Grep Command"), for example, to filter out any variable associated with **memory** management, you can run the following command:

```
$ sudo sysctl -a | grep memory
OR
$ sudo sysctl --all | grep memory

```

[![Check Kernel Memory Variable in Linux](https://www.tecmint.com/wp-content/uploads/2023/02/Check-Kernel-Memory-Variable-in-Linux.png)](https://www.tecmint.com/wp-content/uploads/2023/02/Check-Kernel-Memory-Variable-in-Linux.png)

Check Kernel Memory Variable in Linux

### 4\. List All Kernel Variables Including Deprecated

**sysctl** command also shows deprecated variables along with the list of all available variables using the `--deprecated` flag as shown.

```
$ sudo sysctl -a --deprecated
OR
$ sudo sysctl -a --deprecated | grep memory

```

### 5\. List Specific Kernel Variable Value

To read a **sysctl** variable and its values, specify the variable name as an argument for the **sysctl** commands as follows. This example shows how to read the `kernel.ostype` variable.

```
$ sudo sysctl kernel.ostype

kernel.ostype = Linux

```

### 6\. Write Kernel Variable Temporarily

To write variables temporarily, simply specify the variable in this format.

```
<tunable class>.<tunable>=<value>

```

The following example shows how to increase the maximum size of the receive queue, which stores frames picked from the ring buffer of the **NIC** (**Network Interface Card**), once they are received from the network. The queue size can be modified using the `net.core.netdev_max_backlog` variable as shown.

```
$ sudo sysctl net.core.netdev_max_backlog
$ sudo sysctl net.core.netdev_max_backlog=1200
$ sudo sysctl net.core.netdev_max_backlog

```

[![Set Kernel Variable Temporarily](https://www.tecmint.com/wp-content/uploads/2023/02/Write-Kernel-Variable-Temporarily.png)](https://www.tecmint.com/wp-content/uploads/2023/02/Write-Kernel-Variable-Temporarily.png)

Set Kernel Variable Temporarily

### 7\. Write Kernel Variable Permanently

**sysctl** can also write variables permanently in a configuration file. To achieve this, use the `-w` option, and specify the configuration file the variable and its value will be appended to, in this case, it is **/etc/sysctl.conf**, the default sysctl configuration file:

```
$ sudo sysctl -w net.core.netdev_max_backlog=1200 >> /etc/sysctl.conf

```

To write files permanently in a custom, specify the location of the file as follows. Sometimes, you can fail to create a file in particular locations even when you invoke the **sysctl** command using the **sudo command**.

In such a case, switch to the root account (if you have the privileges) and run the command again as shown.

```
$ sudo sysctl -w net.core.netdev_max_backlog=1200 >> /etc/sysctl.d/10-test-settings.conf
$ sudo su
# sysctl -w net.core.netdev_max_backlog=1200 >> /etc/sysctl.d/10-test-settings.conf

```

[![Set Kernel Variable Permanently](https://www.tecmint.com/wp-content/uploads/2023/02/Write-Kernel-Variable-Permanently.png)](https://www.tecmint.com/wp-content/uploads/2023/02/Write-Kernel-Variable-Permanently.png)

Set Kernel Variable Permanently

Alternatively, you can create the new configuration file in **/etc/sysctl.d/** directory as shown:

```
$ sudo vim /etc/sysctl.d/10-test-settings.conf

```

Then add the kernel parameters, in it one per line as shown.

```
net.core.netdev_max_backlog = 1200
user.max_net_namespaces = 63067
vm.overcommit_memory = 0

```

Then save the file and close it. To load settings from the custom file you have just created, use the `-p` or `--load` flag.

```
$ sudo sysctl -p /etc/sysctl.d/10-test-settings.conf
OR
$ sudo sysctl --load= /etc/sysctl.d/10-test-settings.conf

```

### 8\. Reload sysctl.conf Variables in Linux

To reload settings from all system configuration files without rebooting, issue the following command.

```
$ sudo sysctl  --system

```

The above command will read all system configuration files from these directories, in this order:

```
/run/sysctl.d/*.conf
/etc/sysctl.d/*.conf
/usr/local/lib/sysctl.d/*.conf
/usr/lib/sysctl.d/*.conf
/lib/sysctl.d/*.conf
/etc/sysctl.conf

```

### 9\. Reload Settings from Custom Configuration Files

You can also reload variable settings from a custom sysctl configuration file as shown.

```
$ sudo sysctl -p/etc/sysctl.d/10-test-settings.conf
OR
$ sudo sysctl --load= /etc/sysctl.d/10-test-settings.conf

```

### 10\. Reload Settings that Match Pattern

To only apply settings that match a certain pattern, use the `-r` or `--pattern` as follows. Note that the pattern uses extended regular expression syntax, here are some examples:

```
$ sudo sysctl --system --pattern '^net.ipv6'
$ sudo sysctl --system -r memory

```

[![Reload Settings that Match Pattern](https://www.tecmint.com/wp-content/uploads/2023/02/Reload-Settings-that-Match-Pattern.png)](https://www.tecmint.com/wp-content/uploads/2023/02/Reload-Settings-that-Match-Pattern.png)

Reload Settings that Match Pattern

In this guide, we have explained 10 **sysctl** command examples you can use to manage a running Linux system. For more information, read the **sysctl** man page (**man sysctl**).

## If You Appreciate What We Do Here On TecMint, You Should Consider:

TecMint is the fastest growing and most trusted community site for any kind of Linux Articles, Guides and Books on the web. Millions of people visit TecMint! to search or browse the thousands of published articles available FREELY to all.

If you like what you are reading, please consider buying us a coffee ( or 2 ) as a token of appreciation.

[![Support Us](https://www.tecmint.com/wp-content/uploads/2015/01/coffee.png)](https://www.buymeacoffee.com/tecmint)

**We are thankful for your never ending support.**