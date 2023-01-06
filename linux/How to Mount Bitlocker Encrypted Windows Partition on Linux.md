Here’s the scenario. My system came with Windows 10 Pro and that came with BitLocker encryption. I [installed Ubuntu in the dual boot mode even with the BitLocker encryption](https://itsfoss.com/dual-boot-ubuntu-windows-bitlocker/) enabled for Windows.

You can easily access the Windows files from within Linux. No hi-fi stuff here. Just go to the file manager and click on the Windows partition which is located usually under the “Other Locations” tab.

![mount encrypted windows partition in linux](https://itsfoss.com/wp-content/uploads/2021/11/mount-encrypted-windows-partition-in-linux-800x476.png)

Mounting Windows partition through the file manager in Linux desktop

The process is not too complicated with BitLocker encrypted Windows partition as well. It’s just that when you try to mount the Windows partition, it will ask for the password.

![password needed for encrypted windows drive mount in linux](https://itsfoss.com/wp-content/uploads/2021/11/password-needed-for-encrypted-windows-drive-mount-in-Linux.png)

Password required for encrypted Windows drive mount in Linux

It works though. In my case, I entered the 48 digit BitLocker recovery password and it decrypted the Windows partition and mounted it without any issue in Ubuntu 21.10 with GNOME 40.

Try your BitLocker password. If that does not work, try the recovery password. For normal Windows 10 Pro users, the recovery password is stored in your Microsoft account.

Enter the recovery and you’ll see that Windows partition and its files are accessible now. Checking the “Remember Password” box is also a time saver for further usage.

![encrypted windows partition mounted in linux](https://itsfoss.com/wp-content/uploads/2021/11/encrypted-windows-partition-mounted-in-Linux-800x491.png)

Encrypted Windows partition now mounted in Linux

If the above does not work for you or if you are stuck with the command line, there is an alternative method.

This method involves using a tool called [Dislocker](https://github.com/Aorimn/dislocker).

## Mount BotLocker encrypted Windows partition in Linux with Dislocker \[Command Line Method\]

Dislocker process works in two parts. The first part decrypts the BitLocker encryption and gives a file named dislocker-file. This is basically a virtual NTFS partition. The second part is basically mounting the virtual NTFS partition you just got.

You’ll need either the BitLocker password or the recovery password to decrypt the encrypted drive.

Let’s see the steps in details.

### Step 1: Install Disclocker

Dislocker is available in the repositories of most Linux distributions. Please use your distribution’s package manager to install it.

On Ubuntu and Debian based distributions, use this command:

```
sudo apt install dislocker
```

![install dislocker ubuntu](https://itsfoss.com/wp-content/uploads/2021/11/install-dislocker-ubuntu.png)

Installing Dislocker in Ubuntu

### Step 2 : Create mount points

You’ll need to create two mount points. One for where Dislocker will generate the dislocker-file and the other one which will mount this dislocker-file (virtual filesystem) as a loop device.

There is no naming restrictions and you may name these mount directories anything you want.

Use these commands one by one:

```
sudo mkdir -p /media/decrypt
sudo mkdir -p /media/windows-mount
```

![creating mount points for dislocker](https://itsfoss.com/wp-content/uploads/2021/11/creating-mount-points-for-dislocker.png)

Creating mount points for dislocker

### Step 3: Get the partition info which needs to be decrypted

You need the name of the Windows partition. You can use the file explorer or GUI tools like Gparted.

![show device name gparted](https://itsfoss.com/wp-content/uploads/2021/11/show-device-name-gparted-800x416.png)

Get the partition name

In my case, the Windows partition is /dev/nvme0n1p3. It will be different for your system. You may also use command line for this purpose.

```
sudo lsblk
```

### Step 4: Decrypt the partition and mount

You have everything set up. Now comes the real part.

I**f you have the BitLocker password**, use the dislocker command in this fashion (replace <partition\_name> and <password> with actual values):

```
sudo dislocker <partition_name> -u<password> -- /media/decrypt
```

Please note that **there is no space between u and password**.

If you only have the recovery password, use the command in this fashion (replace <partition\_name> and <recovery\_password> with actual values):

```
sudo dislocker <partition_name> -p<recovery_password> -- /media/decrypt
```

Again, there is **no space between p and password**.

It should not take a long time in decrypting the partition. You should see the dislocker-file in the designated mount point, /media/decrypt in our case. Now mount this dislocker-file:

```
sudo mount -o loop /media/decrypt/dislocker-file /media/windows-mount
```

![mount dislocker decrypted windows partition](https://itsfoss.com/wp-content/uploads/2021/11/mount-dislocker-decrypted-windows-partition.png)

You are done. Your BitLocker encrypted Windows partition is decrypted and mounted in Linux. You can access it from the file explorer as well.

![discloker mount encrypted windows partition](https://itsfoss.com/wp-content/uploads/2021/11/discloker-mount-encrypted-windows-partition-800x483.png)

Mounting Dislocker decrypted Windows partition with file manager

### Troubleshooting tips for wrong fs type error

If you get an error like this:

```
mount: /media/windows-mount: wrong fs type, bad option, bad superblock on /dev/loop35, missing codepage or helper program, or other error.
```

You should specify the filesystem while mounting.

For NTFS, use:

```
sudo mount -t ntfs-3g -o loop /media/decrypt/dislocker-file /media/windows-mount
```

For exFAT, use:

```
sudo mount -t exFAT-fuse -o loop /media/decrypt/dislocker-file /media/windows-mount
```

### Unmount the Windows partition

You can unmount the mounted partition from the file manager. Just **click the unmount symbol** beside the partition named windows-mount.

Otherwise, unmount command is always there for you.

```
sudo umount /media/decrypt
sudo umount /media/windows-mount
```

I hope it helps you. If you still have questions or suggestions, please let me know in the comments.

Skip![arrow](https://sdk.apester.com/assets/iconRightArrow.svg)

Skip

Skip ad in

![arrow](https://sdk.apester.com/assets/iconRightArrow.svg)

![](https://itsfoss.com/wp-content/gravatars/2003563996b123e9460848e5ea27f4d6)

Creator of It's FOSS. An ardent Linux user & open source promoter. Huge fan of classic detective mysteries ranging from Agatha Christie and Sherlock Holmes to Detective Columbo & Ellery Queen. Also a movie buff with a soft corner for film noir.