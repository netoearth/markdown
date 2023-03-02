_**Brief: In this advanced guide, we will discuss some practical examples of the dd command. After following this guide, advanced users will be able to work with the block devices comfortably from the command line interface.**_

In Linux, [everything is a file](https://www.tecmint.com/explanation-of-everything-is-a-file-and-types-of-files-in-linux/ "Everything is a File in Linux") and the block devices are not an exception to it. Often time, we need to work with block devices. As Linux users, we perform a variety of operations on block devices, such as, – taking a [backup of a disk or partition](https://www.tecmint.com/clone-linux-partitions/ "Clone a Partition or Hard drive in Linux"), backing up Master Boot Record (MBR), making a [bootable USB drive](https://www.tecmint.com/linux-bootable-usb-creators/ "Create Bootable USB in Linux"), and the list goes on.

Certainly, we can use graphical tools to perform all these operations. However, most Linux administrators prefer to use the **dd command** due to its rich functionality and robustness.

In this advanced guide, we will learn about the **dd command** to convert and copy files. However, unlike the [cp command](https://www.tecmint.com/cp-command-examples/ "How to Copy Files and Directories in Linux") most of the time it’s used with block devices.

In this guide, first, we will understand the usage of the **dd command** with basic examples then we will discuss some advanced use cases.

Table of Contents

1

-   [dd Command Syntax](https://www.tecmint.com/dd-command-examples/#dd_Command_Syntax "dd Command Syntax")
-   [1\. How to Copy a File in Linux](https://www.tecmint.com/dd-command-examples/#1_How_to_Copy_a_File_in_Linux "1. How to Copy a File in Linux")
-   [2\. How to Convert Text from Lowercase to Uppercase](https://www.tecmint.com/dd-command-examples/#2_How_to_Convert_Text_from_Lowercase_to_Uppercase "2. How to Convert Text from Lowercase to Uppercase")
-   [3\. How to Convert Text from Uppercase to Lowercase](https://www.tecmint.com/dd-command-examples/#3_How_to_Convert_Text_from_Uppercase_to_Lowercase "3. How to Convert Text from Uppercase to Lowercase")
-   [4\. Avoid Overwriting Destination File in Linux](https://www.tecmint.com/dd-command-examples/#4_Avoid_Overwriting_Destination_File_in_Linux "4. Avoid Overwriting Destination File in Linux")
-   [5\. Append Data in a File Using dd Command](https://www.tecmint.com/dd-command-examples/#5_Append_Data_in_a_File_Using_dd_Command "5. Append Data in a File Using dd Command")
-   [6\. Skip Bytes or Characters While Reading the Input File](https://www.tecmint.com/dd-command-examples/#6_Skip_Bytes_or_Characters_While_Reading_the_Input_File "6. Skip Bytes or Characters While Reading the Input File")
-   [7\. Backup Linux Disk Partition Using dd Command](https://www.tecmint.com/dd-command-examples/#7_Backup_Linux_Disk_Partition_Using_dd_Command "7. Backup Linux Disk Partition Using dd Command")
-   [8\. Restore Linux Disk Partition Using dd Command](https://www.tecmint.com/dd-command-examples/#8_Restore_Linux_Disk_Partition_Using_dd_Command "8. Restore Linux Disk Partition Using dd Command")
-   [9\. Backup Entire Linux Hard Drive Using dd Command](https://www.tecmint.com/dd-command-examples/#9_Backup_Entire_Linux_Hard_Drive_Using_dd_Command "9. Backup Entire Linux Hard Drive Using dd Command")
-   [10\. Restore the Linux Hard Drive Using dd Command](https://www.tecmint.com/dd-command-examples/#10_Restore_the_Linux_Hard_Drive_Using_dd_Command "10. Restore the Linux Hard Drive Using dd Command")
-   [11\. Backup Master Boot Record Using dd Command](https://www.tecmint.com/dd-command-examples/#11_Backup_Master_Boot_Record_Using_dd_Command "11. Backup Master Boot Record Using dd Command")
-   [12\. Restore Master Boot Record Using dd Command](https://www.tecmint.com/dd-command-examples/#12_Restore_Master_Boot_Record_Using_dd_Command "12. Restore Master Boot Record Using dd Command")
-   [13\. Copy CD/DVD Drive Content Using dd Command](https://www.tecmint.com/dd-command-examples/#13_Copy_CDDVD_Drive_Content_Using_dd_Command "13. Copy CD/DVD Drive Content Using dd Command")
-   [14\. Create a Bootable USB Drive Using dd Command](https://www.tecmint.com/dd-command-examples/#14_Create_a_Bootable_USB_Drive_Using_dd_Command "14. Create a Bootable USB Drive Using dd Command")
-   [15\. How to Show the Progress Bar](https://www.tecmint.com/dd-command-examples/#15_How_to_Show_the_Progress_Bar "15. How to Show the Progress Bar")
    -   -   [Conclusion](https://www.tecmint.com/dd-command-examples/#Conclusion "Conclusion")

So let’s get started.

### dd Command Syntax

The most common syntax of the **dd command** is as follows:

```
$ dd [if=] [of=]

```

In the above syntax:

-   **if** – represents the input or source file.
-   **of** – represents the output or destination file.

**Precaution** – It is important to note that the dd command must be used very cautiously while working with the block devices. A small mistake can cause permanent data loss. Hence it is strongly commended to try out these operations on the test machine first.

### 1\. How to Copy a File in Linux

One of the basic use of the **dd command** is to copy a file into a current directory. Let’s understand by creating a simple text file:

```
$ echo "this is a sample text file" > file-1.txt

```

Now, let’s create a copy of it using the **dd command**:

```
$ dd if=file-1.txt of=file-2.txt

```

In this example, the `if` parameter represents the source file whereas the `of` parameter represents the destination file.

[![Copy File with dd Command](https://www.tecmint.com/wp-content/uploads/2023/01/Copy-File-with-dd-Command.png)](https://www.tecmint.com/wp-content/uploads/2023/01/Copy-File-with-dd-Command.png)

Copy File with dd Command

Isn’t it exactly similar to the **cp command**? Then what’s so special about the **dd command**?

The **dd command** is much more powerful than the regular **cp command**. The latter sections of the tutorial discuss some of its advanced use cases.

### 2\. How to Convert Text from Lowercase to Uppercase

The **dd command** allows us to perform case conversion. To achieve this we can use the **conv** parameter with it.

To understand this, first, display the contents of the **file-1.txt** file:

```
$ cat file-1.txt

this is a sample text file

```

Now, let’s convert the file contents to the upper case using the following command:

```
$ dd if=file-1.txt of=upper-case.txt conv=ucase

```

In this example, the `conv=ucase` option is used to convert the lowercase letters to uppercase letters.

Finally, verify the contents of the newly created file:

```
$ cat upper-case.txt

THIS IS A SAMPLE TEXT FILE

```

[![Convert Text from Lowercase to Uppercase in Linux](https://www.tecmint.com/wp-content/uploads/2023/01/Convert-Lowercase-to-Uppercase-Text.png)](https://www.tecmint.com/wp-content/uploads/2023/01/Convert-Lowercase-to-Uppercase-Text.png)

Convert Text from Lowercase to Uppercase in Linux

### 3\. How to Convert Text from Uppercase to Lowercase

In a similar way, we can use the **dd command** to convert the upper case letters into lower case letters:

Let’s use the `conv=lcase` option to convert the upper case letters to lower case:

```
$ dd if=upper-case.txt of=lower-case.txt conv=lcase

```

Now, let’s display the contents of the newly created file and verify that the conversion has been done correctly:

```
$ cat lower-case.txt

this is a sample text file

```

[![Convert Text to Lowercase in Linux](https://www.tecmint.com/wp-content/uploads/2023/01/Convert-Text-to-Lowercase-in-Linux.png)](https://www.tecmint.com/wp-content/uploads/2023/01/Convert-Text-to-Lowercase-in-Linux.png)

Convert Text to Lowercase in Linux

### 4\. Avoid Overwriting Destination File in Linux

By default, the **dd command** replaces the destination file, which means it will overwrite the file if it exists at the destination with the same name.

However, we can disable this default behavior using the `conv=excl` option as shown.

```
$ dd if=file-1.txt of=file-2.txt conv=excl

dd: failed to open ‘file-2.txt’ File exists

```

Here, we can see that the **dd command** has aborted the operations because the file with the same name is present at the destination.

### 5\. Append Data in a File Using dd Command

Sometimes, we want to update the file in an append mode, which means the new contents should be added to the end of the destination file.

We can achieve this by combining the two flags – `oflag=append` and `conv=notrunc`. Here, the `oflag` represents the output flag whereas the `notrunc` option is used to disable the truncation at the destination.

To understand this, first, let’s create a new text file:

```
$ echo "append example demo" > dest.txt

```

Next, let’s append the contents to the **dest.txt** file using the following command:

```
$ dd if=file-1.txt of=dest.txt oflag=append conv=notrunc

```

Now, let’s check the contents of the **dest.txt** file:

```
$ cat dest.txt 

append example demo
this is a sample text file

```

[![Append Data to File Using dd Command](https://www.tecmint.com/wp-content/uploads/2023/01/Append-Data-to-File-Using-dd-Command.png)](https://www.tecmint.com/wp-content/uploads/2023/01/Append-Data-to-File-Using-dd-Command.png)

Append Data to File Using dd Command

### 6\. Skip Bytes or Characters While Reading the Input File

We can instruct the **dd command** to skip the first few characters while reading the input file using the **ibs** and **skip** options.

First, let’s display the contents of the **file-1.txt** file:

```
$ cat file-1.txt

this is a sample text file

```

Next, let’s skip the first 8 characters by using the following command:

```
$ dd if=file-1.txt of=file-2.txt ibs=8 skip=1

```

Now, let’s verify the contents of the **file-2.txt** file:

```
$ cat file-2.txt

a sample text file

```

[![Skip Characters from File Using dd](https://www.tecmint.com/wp-content/uploads/2023/01/Skip-Bytes-from-File.png)](https://www.tecmint.com/wp-content/uploads/2023/01/Skip-Bytes-from-File.png)

Skip Characters from File Using dd

In the above output, we can see that the command has skipped the first 8 characters.

### 7\. Backup Linux Disk Partition Using dd Command

So far we discussed the basic examples of the **dd command** that doesn’t require root access. Now, let’s see some advanced use cases.

Just like files, we can take the backup of the disk partition using the **dd command**. For example, the below command takes a backup of the **/dev/sda1** partition to **partition-bkp.img**:

```
$ sudo dd if=/dev/sda1 of=partition-bkp.img

```

[![Backup Partition in Linux Using dd Command](https://www.tecmint.com/wp-content/uploads/2023/01/Backup-Linux-Partition-using-dd.png)](https://www.tecmint.com/wp-content/uploads/2023/01/Backup-Linux-Partition-using-dd.png)

Backup Partition in Linux Using dd Command

### 8\. Restore Linux Disk Partition Using dd Command

In the previous example, we backed up the **/dev/sda1** partition to the **partition-bkp.img** file.

Now, let’s restore it to the **/dev/sdb1** partition using the following command:

```
$ sudo dd if=partition-bkp.img of=/dev/sdb1

```

[![Restore Linux Partition Using dd Command](https://www.tecmint.com/wp-content/uploads/2023/01/Restore-Linux-Partition-using-dd.png)](https://www.tecmint.com/wp-content/uploads/2023/01/Restore-Linux-Partition-using-dd.png)

Restore Linux Partition Using dd Command

It is important to note that, the size of the destination partition must be equal to or larger than the backup size.

### 9\. Backup Entire Linux Hard Drive Using dd Command

The disk drive can have multiple partitions. So taking and restoring backup per partition can become time-consuming as the number of partitions increases. To overcome this limitation, we can back up the entire disk drive just like the partitions.

So, let’s take the backup of the **/dev/sda** disk using the following command:

```
$ sudo dd if=/dev/sda of=disk-bkp.img

```

[![Backup Entire Linux Hard Drive Using dd Command](https://www.tecmint.com/wp-content/uploads/2023/01/Backup-Linux-Hard-Disk-using-dd.png)](https://www.tecmint.com/wp-content/uploads/2023/01/Backup-Linux-Hard-Disk-using-dd.png)

Backup Entire Linux Hard Drive Using dd Command

The above command takes back up the entire disk including its partitions.

### 10\. Restore the Linux Hard Drive Using dd Command

Just like partitions, we can restore the backup of the entire disk. In the previous example, we backed up the entire disk to the **disk-bkp.img** file. Now, let’s use the same to restore it on the **/dev/sdb** disk.

First, let’s delete all the partitions from the **/dev/sdb** disk and verify that all the partitions have been deleted:

```
$ lsblk /dev/sdb

```

Next, let’s restore the backup on the **/dev/sdb** drive using the following command:

```
$ sudo dd if=disk-bkp.img of=/dev/sdb

```

Finally, verify that the partition has been created on the **/dev/sdb** disk:

```
$ lsblk /dev/sdb

```

[![Restore Linux Hard Drive Using dd Command](https://www.tecmint.com/wp-content/uploads/2023/01/Restore-Linux-Hard-Disk-using-dd.png)](https://www.tecmint.com/wp-content/uploads/2023/01/Restore-Linux-Hard-Disk-using-dd.png)

Restore Linux Hard Drive Using dd Command

### 11\. Backup Master Boot Record Using dd Command

The Master Boot Record (MBR) is located in the first sector of the boot disk. It stores information about the disk partitions. We can use the **dd command** as shown below to take a back of it:

```
$ sudo dd if=/dev/sda of=mbr.img bs=512 count=1

```

The above command takes back up the first 512 bytes i.e. one sector.

[![Backup MBR Using dd Command](https://www.tecmint.com/wp-content/uploads/2023/01/Backup-MBR-using-dd.png)](https://www.tecmint.com/wp-content/uploads/2023/01/Backup-MBR-using-dd.png)

Backup MBR Using dd Command

It is important to note that, the above command must be executed on the boot disk.

### 12\. Restore Master Boot Record Using dd Command

In the previous example, we backed up the Master Boot Record (MBR). Now, let’s restore it on the **/dev/sdb** disk using the following command:

```
$ sudo dd if=mbr.img of=/dev/sdb

```

[![Restore MBR Using dd Command](https://www.tecmint.com/wp-content/uploads/2023/01/Restore-MBR-using-dd.png)](https://www.tecmint.com/wp-content/uploads/2023/01/Restore-MBR-using-dd.png)

Restore MBR Using dd Command

### 13\. Copy CD/DVD Drive Content Using dd Command

Similar to the partitions and disks, we can use the dd command to copy contents from the CD or DVD drive. So let’s use the below command to do the same:

```
$ sudo dd if=/dev/cdrom of=alma-minimal.iso

```

In Linux, the CD/DVD drive is represented by the **/dev/cdrom** device. Hence we are using it as a source file.

Now, let’s verify that the contents have been copied successfully by verifying its [checksum command](https://www.tecmint.com/generate-verify-check-files-md5-checksum-linux/ "Generate and Verify Files with Checksum Command"):

```
$ sha256sum alma-minimal.is

```

[![Copy CD/DVD Contents Using dd Command](https://www.tecmint.com/wp-content/uploads/2023/01/Copy-CD-DVD-Contents-Using-dd-Command.png)](https://www.tecmint.com/wp-content/uploads/2023/01/Copy-CD-DVD-Contents-Using-dd-Command.png)

Copy CD/DVD Contents Using dd Command

### 14\. Create a Bootable USB Drive Using dd Command

In the previous example, we created an iso image of the Alma Linux. Now let’s use it to create a bootable USB drive:

```
$ sudo dd if=alma-minimal.iso of=/dev/sdb

```

[![Create Bootable USB Using dd Command](https://www.tecmint.com/wp-content/uploads/2023/01/Create-Bootable-USB-Using-dd-Command.png)](https://www.tecmint.com/wp-content/uploads/2023/01/Create-Bootable-USB-Using-dd-Command.png)

Create Bootable USB Using dd Command

It is important to note that, the above command must be executed with the correct USB drive.

### 15\. How to Show the Progress Bar

By default, the **dd command** doesn’t show the progress while doing the copy operation. However, we can override this default behavior using the status option.

So let’s use the `status=progress` option with the **dd command** to show the progress bar:

```
$ sudo dd if=alma-minimal.iso of=/dev/sdb status=progress

```

[![Show dd Command Progress](https://www.tecmint.com/wp-content/uploads/2023/01/Show-dd-Command-Progress.png)](https://www.tecmint.com/wp-content/uploads/2023/01/Show-dd-Command-Progress.png)

Show dd Command Progress

##### Conclusion

In this article, we discussed some practical examples of the **dd command**. Advanced users can refer to these examples in day-to-day life while working with Linux systems. However, we have to be very careful while executing these commands. Because a small mistake can overwrite the contents of the entire disk.

_**Do you know of any other best example of the dd command in Linux? Let us know your views in the comments below.**_