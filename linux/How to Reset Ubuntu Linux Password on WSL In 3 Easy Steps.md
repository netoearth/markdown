WSL (Windows Subsystem for Linux) is a handy tool for people who want to enjoy the power of Linux command line from the comfort of Windows.

When you [install Linux using WSL on Windows](https://itsfoss.com/install-bash-on-windows/), you are asked to create a username and password. This user is automatically logged on when you start Linux on WSL.

Now, the problem is that if you haven’t used it for some time, you may forget the account password of WSL. And this will become a problem if you have to use a command with sudo because here you’ll need to enter the password.

![reset wsl password](https://itsfoss.com/wp-content/uploads/2021/06/reset-wsl-password.png)

Don’t worry. You can easily reset it.

## Reset forgotten password for Ubuntu or any other Linux distribution on WSL

To reset the Linux password in WSL, you have to:

-   Switch the default user to root
-   Reset the password for the normal user
-   Switch back the default user to the normal user

Let me show you the steps in detail and with screenshots. If you want a video, you can watch that as well.

<iframe loading="lazy" src="https://www.youtube.com/embed/HYGUz9mC7Hg?version=3&amp;rel=1&amp;showsearch=0&amp;showinfo=1&amp;iv_load_policy=1&amp;fs=1&amp;hl=en-US&amp;autohide=2&amp;wmode=transparent" allowfullscreen="true" sandbox="allow-scripts allow-same-origin allow-popups allow-presentation" width="1290" height="726"></iframe>

### Step 1: Switch to root as default user

It will be wise to note down your account’s normal/regular username. As you can see, my regular account’s username is abhishek.

![username wsl](https://itsfoss.com/wp-content/uploads/2021/06/username-wsl-800x296.png)

Note down the account username

The root user in WSL is unlocked and doesn’t have a password set. This means that you can switch to the root user and then use the power of root to reset the password.

Since you don’t remember the account password, switching to the root user is done by changing the configuration of your Linux WSL application and make it use root user by default.

This is done through Windows Command Prompt and you’ll need to know which command you need to run for your Linux distribution.

This information is usually provided in the description of the distribution app in the [Windows Store](https://www.microsoft.com/en-us/store/apps/windows). This is from where you had downloaded your distribution in the first place.

![wsl distro command](https://itsfoss.com/wp-content/uploads/2021/06/wsl-distro-command-800x602.png)

Know the command to run for your distribution app

From the Windows menu, start the command prompt:

![Start Command Prompt in windows](https://itsfoss.com/wp-content/uploads/2021/06/start-cmd-windows-800x500.jpg)

Start Command Prompt

In here, use your distribution’s command in this fashion. If you were using the Ubuntu app from Windows store, the command would be:

```
ubuntu config --default-user root
```

In the screenshot, I am using Ubuntu 20.04 app from the Windows store. So, I have used ubuntu2004 command.

![wsl set root as default](https://itsfoss.com/wp-content/uploads/2021/06/wsl-set-root-as-default-800x288.png)

Set root as default user in Linux app’s configuration

To save you the trouble, I am listing some distributions and their respective commands in this table:

| Distribution App | Windows Command |
| --- | --- |
| Ubuntu | ubuntu config –default-user root |
| Ubuntu 20.04 | ubuntu2004 config –default-user root |
| Ubuntu 18.04 | ubuntu1804 config –default-user root |
| Debian | debian config –default-user root |
| Kali Linux | kali config –default-user root |

### Step 2: Reset the password for the account

Now, if you start the Linux distribution app, you should be logged in as root. You can reset the password for the normal user account.

Do you remember the username in WSL? If not, you can always check the contents of the /home directory. When you have the username, use this command:

```
passwd username
```

It will ask you to enter a new password. **When you type here, nothing will be displayed on the screen. That’s normal. Just type the new password and hit enter.** You’ll have to retype the new password to confirm and once again, nothing will be displayed on the screen while you type the password.

![resetting wsl password](https://itsfoss.com/wp-content/uploads/2021/06/resetting-wsl-password-800x366.png)

Reset the password for the regular user

Congratulations. The password for the user account has been reset. But you are done just yet. The default user is still root. You should change it back to your regular account user, otherwise it will keep on logging in as root user.

### Step 3: Set regular user as default again

You’ll need the regular account username that you used with the [passwd command](https://linuxhandbook.com/passwd-command/) in the previous step.

Start the Windows command prompt once again. **Use your distribution’s command** in the similar manner you did in the step 1. However, this time, replace root with the regular user.

```
ubuntu config --default-user username
```

![set regular user as default wsl](https://itsfoss.com/wp-content/uploads/2021/06/set-regular-user-as-default-wsl-800x288.png)

Set regular user as default user

Now when you start your Linux distribution app in WSL, you’ll be logged in as the regular user. You have reset the password fresh and can use it to run commands with sudo.

If you forgot the password again in the future, you know the steps to reset it.

## If resetting WSL password is this easy, is this not a security risk?

Not really. You need to have physical access to the computer along with access to the Windows account. If someone already has this much access, she/he can do a lot more than just changing the Linux password in WSL.

## Were you able to reset WSL password?

I gave you the commands and explained the steps. I hope this was helpful to you and you were able to reset the password of your Linux distribution in WSL.

If you are still facing issues or if you have a question on this topic, please feel free to ask in the comment section.

Skip![arrow](https://sdk-canary.apester.com/assets/iconRightArrow.svg)

Skip

Skip ad in

![arrow](https://sdk-canary.apester.com/assets/iconRightArrow.svg)

![](https://itsfoss.com/wp-content/gravatars/2003563996b123e9460848e5ea27f4d6)

Creator of It's FOSS. An ardent Linux user & open source promoter. Huge fan of classic detective mysteries ranging from Agatha Christie and Sherlock Holmes to Detective Columbo & Ellery Queen. Also a movie buff with a soft corner for film noir.