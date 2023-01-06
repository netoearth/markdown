![Snapcraft logo](https://www.linuxuprising.com/ezoimgfmt/1.bp.blogspot.com/-FrgqhIXVbiA/XK3gHv8m3hI/AAAAAAAACng/FQrKyTGZCEoH3yki5kJvF7V7W6RTe-2vwCLcBGAs/s640/snapcraft_green-red_hex.jpeg?ezimgfmt=rs%3Adevice%2Frscb272-1 "Snapcraft logo")

I was using Disk Usage Analyzer recently to see if I could free up some space on my Ubuntu 18.10 desktop, when I noticed that the `/var/lib/snapd/snaps/` folder was quite large.  
  
While investigating how I could free up some space / clear the snap cache from the `/var/lib/snapd/snaps/` folder without removing the snap packages I had installed, I found out that by default, 3 snap versions are stored by the system after snap package updates. Meaning that for each installed snap package that had at least 2 updates, I had 3 revisions stored on my system, taking up quite a bit of disk space.

There is a **snap option** (starting with snapd version 2.34), called [`refresh.retain`](https://docs.snapcraft.io/system-options/87#retain), **to set the maximum number of a snap's revisions stored by the system after the next refresh, which can be set to a number between 2 and 20**. You can change this from the default value of 3 to 2 by using:

```
sudo snap set system refresh.retain=2
```

**_Related, but for Flatpak packages: [How To Remove Unused Flatpak Runtimes To Free Up Disk Space](https://www.linuxuprising.com/2019/02/how-to-remove-unused-flatpak-runtimes.html)_**

**But what if you want to remove all versions kept on the system for all snap packages that had updates?** This is [a script](https://superuser.com/a/1330590) created by Popey, Community Manager in Ubuntu Engineering at Canonical, to remove ALL old versions of snaps, only keeping the current active version (updated with `LANG=en_US.UTF-8` so it works with non-English locales, thanks to William in the comments):

```
#!/bin/bash
# Removes old revisions of snaps
# CLOSE ALL SNAPS BEFORE RUNNING THIS
set -eu

LANG=en_US.UTF-8 snap list --all | awk '/disabled/{print $1, $3}' |
    while read snapname revision; do
        snap remove "$snapname" --revision="$revision"
    done
```

Using this script should free up some significant disk space (depending on the number of snap packages installed on your system, and if they had updates since they were installed). In my case, the script reduced the size of the `/var/lib/snapd/snaps/` folder by more than 50%.

Update: It looks like this script only works with English. For other languages you'll need to replace `/disabled/` in the command with its translation to your language.

To use this script create a file named `remove-old-snaps`, paste the contents from the code block above, save the file in your home directory, and make it executable using:

```
chmod +x remove-old-snaps
```

  
Run the script with `sudo` to remove old snap revisions (make sure to close all running snaps before running the script):

```
sudo ./remove-old-snaps
```

This is the script running on my system, removing old snap package revisions:

```
$ sudo ./remove-old-snaps

atom (revision 223) removed
atom (revision 222) removed
bitwarden (revision 15) removed
bitwarden (revision 16) removed
canonical-livepatch (revision 50) removed
canonical-livepatch (revision 54) removed
chromium (revision 607) removed
chromium (revision 660) removed
core (revision 6531) removed
core (revision 6405) removed
core18 (revision 719) removed
core18 (revision 731) removed
gallery-dl (revision 36) removed
gallery-dl (revision 167) removed
gimp (revision 110) removed
gimp (revision 113) removed
```