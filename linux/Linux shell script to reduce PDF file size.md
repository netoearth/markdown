Here is a handy and useful Linux and Unix shell script that reduce PDF file size using Ghostscript. No need to upload your PDF file to the shady third-party website. Just do it from the terminal. I tested it with both CentOS and Ubuntu/Debian Linux.  
  
A sample shell script reduce the file size of a scanned PDF file under Linux. It should work with other Unix-like systems too as long as Ghostscript installed. For example macOS or FreeBSD.

## Unix and Linux shell script to reduce PDF file size

```
#!/bin/sh
 
# http://www.alfredklomp.com/programming/shrinkpdf
# Licensed under the 3-clause BSD license:
#
# Copyright (c) 2014-2019, Alfred Klomp
# All rights reserved.
#
# Redistribution and use in source and binary forms, with or without
# modification, are permitted provided that the following conditions are met:
# 1. Redistributions of source code must retain the above copyright notice,
#    this list of conditions and the following disclaimer.
# 2. Redistributions in binary form must reproduce the above copyright notice,
#    this list of conditions and the following disclaimer in the documentation
#    and/or other materials provided with the distribution.
# 3. Neither the name of the copyright holder nor the names of its contributors
#    may be used to endorse or promote products derived from this software
#    without specific prior written permission.
#
# THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
# AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
# IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE
# ARE DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE
# LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR
# CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF
# SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS
# INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN
# CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE)
# ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED OF THE
# POSSIBILITY OF SUCH DAMAGE.
 
#
# Modified by Vivek Gite to suit my needs
#
shrink ()
{
gs\
  -q -dNOPAUSE -dBATCH -dSAFER\
  -sDEVICE=pdfwrite\
  -dCompatibilityLevel=1.3\
  -dPDFSETTINGS=/screen\
  -dEmbedAllFonts=true\
  -dSubsetFonts=true\
  -dAutoRotatePages=/None\
  -dColorImageDownsampleType=/Bicubic\
  -dColorImageResolution=$3\
  -dGrayImageDownsampleType=/Bicubic\
  -dGrayImageResolution=$3\
  -dMonoImageDownsampleType=/Subsample\
  -dMonoImageResolution=$3\
  -sOutputFile="$2"\
  "$1"
}
 
check_smaller ()
{
# If $1 and $2 are regular files, we can compare file sizes to
# see if we succeeded in shrinking. If not, we copy $1 over $2:
if [ ! -f "$1" -o ! -f "$2" ]; then
return 0;
fi
ISIZE="$(echo $(wc -c "$1") | cut -f1 -d\ )"
OSIZE="$(echo $(wc -c "$2") | cut -f1 -d\ )"
if [ "$ISIZE" -lt "$OSIZE" ]; then
echo "Input smaller than output, doing straight copy" >&2
cp "$1" "$2"
fi
}
 
usage ()
{
echo "Reduces PDF filesize by lossy recompressing with Ghostscript."
echo "Not guaranteed to succeed, but usually works."
echo "  Usage: $1 infile [outfile] [resolution_in_dpi]"
}
 
IFILE="$1"
 
# Need an input file:
if [ -z "$IFILE" ]; then
usage "$0"
exit 1
fi
 
# Output filename defaults to "-" (stdout) unless given:
if [ ! -z "$2" ]; then
OFILE="$2"
else
OFILE="-"
fi
 
# Output resolution defaults to 72 unless given:
if [ ! -z "$3" ]; then
res="$3"
else
res="90"
fi
 
shrink "$IFILE" "$OFILE" "$res" || exit $?
 
check_smaller "$IFILE" "$OFILE"
```

## How to Reduce PDF File Size in Linux when using bash shell

Download and install the script in your system. Set up executable permissions using the chmod command:  
`chmod +x ~/bin/shrinkpdf.sh`  
Run it as follows:  
`~/bin/shrinkpdf.sh input.pdf output.pdf   ~/bin/shrinkpdf.sh input.pdf output.pdf 100   ~/bin/shrinkpdf.sh input.pdf output.pdf 120`  
Use the ls to list file size:  
`ls -lh input.pdf   ls -lh output.pdf`  
![Unix and Linux shell script to reduce PDF file size](https://bash.cyberciti.biz/wp-content/uploads/2019/11/Unix-and-Linux-shell-script-to-reduce-PDF-file-size-.png)![Unix and Linux shell script to reduce PDF file size](https://bash.cyberciti.biz/wp-content/uploads/2019/11/Unix-and-Linux-shell-script-to-reduce-PDF-file-size-.png)

## Comparing original vs compressed pdf file quality

1.  [input.pdf](https://bash.cyberciti.biz/wp-content/uploads/2019/11/input.pdf) (original file \[1018K\])
2.  [output.pdf](https://bash.cyberciti.biz/wp-content/uploads/2019/11/output.pdf) (compressed file \[217K\])

## Conclusion

I often use this tiny script to reduce the file size of a scanned PDF file that I need to email/upload or store it on my NAS server for future reference. Most government or third party sites put restrictions on PDF file size uploads. Hence, using this script can help you reduce the size and upload files. The source of original shell script is [here](http://www.alfredklomp.com/programming/shrinkpdf/)

  

| Category | List of Unix and Linux commands |
| --- | --- |
| Download managers | [wget](https://www.cyberciti.biz/tips/linux-wget-your-ultimate-command-line-downloader.html "Wget Command in Linux with Examples") |
| Driver Management | [Linux Nvidia driver](https://www.cyberciti.biz/faq/tag/linux-nvidia-driver/ "Linux Nvidia Driver How To - Linux / Unix Q & A from nixCraft") |
| Documentation | [help](https://bash.cyberciti.biz/guide/Help_command "Help command - Linux Shell Scripting Wiki") • [mandb](https://bash.cyberciti.biz/guide/Mandb_command "Mandb command - Linux Shell Scripting Wiki") • [man](https://bash.cyberciti.biz/guide/Man_command "Man command - Linux Shell Scripting Wiki") • [pinfo](https://www.cyberciti.biz/open-source/command-line-hacks/linux-command-pinfo-for-colorful-info-pages/ "pinfo - Read Linux Info Documentation in Colors") |
| Disk space analyzers | [df](https://www.cyberciti.biz/faq/df-command-examples-in-linux-unix/ "How to use df command in Linux / Unix {with examples}") • [duf](https://www.cyberciti.biz/open-source/command-line-hacks/duf-disk-usage-free-utility-for-linux-bsd-macos-windows/ "duf - Disk Usage/Free Utility for Linux, BSD, macOS & Windows") • [ncdu](https://www.cyberciti.biz/open-source/install-ncdu-on-linux-unix-ncurses-disk-usage/ "How to install ncdu on Linux / Unix to see disk usage") • [pydf](https://www.cyberciti.biz/tips/unix-linux-bsd-pydf-command-in-colours.html "Unix / Linux: See Colourised Filesystem Disk Space Usage with pydf") |
| File Management | [cat](https://www.cyberciti.biz/faq/linux-unix-appleosx-bsd-cat-command-examples/ "cat Command in Linux / Unix with examples") • [cp](https://www.cyberciti.biz/faq/copy-command/ "Linux Copy File Command [ cp Command Examples ]") • [less](https://bash.cyberciti.biz/guide/Less_command "Less command - Linux Shell Scripting Wiki") • [mkdir](https://www.cyberciti.biz/faq/linux-make-directory-command/ "Linux: How to Make a Directory Command") • [more](https://bash.cyberciti.biz/guide/More_command "More command - Linux Shell Scripting Wiki") • [tree](https://www.cyberciti.biz/faq/linux-show-directory-structure-command-line/ "Linux see directory tree structure using tree command") |
| Firewall | [Alpine Awall](https://www.cyberciti.biz/faq/how-to-set-up-a-firewall-with-awall-on-alpine-linux/ "How To Set Up a Firewall with Awall on Alpine Linux") • [CentOS 8](https://www.cyberciti.biz/faq/how-to-set-up-a-firewall-using-firewalld-on-centos-8/ "How to set up a firewall using FirewallD on CentOS 8") • [OpenSUSE](https://www.cyberciti.biz/faq/set-up-a-firewall-using-firewalld-on-opensuse-linux/ "How to set up a firewall using FirewallD on OpenSUSE Linux") • [RHEL 8](https://www.cyberciti.biz/faq/configure-set-up-a-firewall-using-firewalld-on-rhel-8/ "How to set up a firewall using FirewallD on RHEL 8") • [Ubuntu 16.04](https://www.cyberciti.biz/faq/howto-configure-setup-firewall-with-ufw-on-ubuntu-linux/ "How to set up a UFW firewall on Ubuntu 16.04 LTS server") • [Ubuntu 18.04](https://www.cyberciti.biz/faq/how-to-setup-a-ufw-firewall-on-ubuntu-18-04-lts-server/ "How to setup a UFW firewall on Ubuntu 18.04 LTS server") • [Ubuntu 20.04](https://www.cyberciti.biz/faq/how-to-configure-firewall-with-ufw-on-ubuntu-20-04-lts/ "How To Configure Firewall with UFW on Ubuntu 20.04 LTS") |
| Linux Desktop apps | [Skype](https://www.cyberciti.biz/faq/how-to-install-skype-application-on-linux/ "How to install Skype application on Linux") • [Spotify](https://www.cyberciti.biz/faq/how-to-install-spotify-application-on-linux/ "How to install Spotify application on Linux") • [VLC 3](https://www.cyberciti.biz/faq/how-to-install-vlc-3-application-vetinari-on-linux/ "How to install VLC 3 application (Vetinari) on Linux") |
| Modern utilities | [bat](https://www.cyberciti.biz/open-source/bat-linux-command-a-cat-clone-with-written-in-rust/ "bat Linux command - A cat clone with written in Rust") • [exa](https://www.cyberciti.biz/open-source/command-line-hacks/exa-a-modern-replacement-for-ls-written-in-rust-for-linuxunix/ "exa a modern replacement for ls command in rust for Linux/Unix") |
| Network Utilities | [NetHogs](https://www.cyberciti.biz/faq/linux-find-out-what-process-is-using-bandwidth/ "Linux See Bandwidth Usage Per Process With Nethogs Tool") • [dig](https://www.cyberciti.biz/faq/linux-unix-dig-command-examples-usage-syntax/ "Linux and Unix dig Command Examples") • [host](https://www.cyberciti.biz/faq/linux-unix-host-command-examples-usage-syntax/ "Linux and Unix host Command Examples") • [ip](https://www.cyberciti.biz/faq/linux-ip-command-examples-usage-syntax/ "Linux ip Command Examples") • [nmap](https://www.cyberciti.biz/security/nmap-command-examples-tutorials/ "Nmap Command Examples For Linux Sys/Network Admins") • [ping](https://www.cyberciti.biz/faq/unix-ping-command-examples/ "UNIX ping Command Examples") |
| OpenVPN | [CentOS 7](https://www.cyberciti.biz/faq/centos-7-0-set-up-openvpn-server-in-5-minutes/ "CentOS 7 Set Up OpenVPN Server In 5 Minutes") • [CentOS 8](https://www.cyberciti.biz/faq/centos-8-set-up-openvpn-server-in-5-minutes/ "CentOS 8 Set Up OpenVPN Server In 5 Minutes") • [Debian 10](https://www.cyberciti.biz/faq/debian-10-set-up-openvpn-server-in-5-minutes/ "Debian 10 Set Up OpenVPN Server In 5 Minutes") • [Debian 8/9](https://www.cyberciti.biz/faq/install-configure-openvpn-server-on-debian-9-linux/ "Install and Configure an OpenVPN on Debian 9 In 5 Minutes") • [Ubuntu 18.04](https://www.cyberciti.biz/faq/ubuntu-18-04-lts-set-up-openvpn-server-in-5-minutes/ "Ubuntu 18.04 LTS Set Up OpenVPN Server In 5 Minutes") • [Ubuntu 20.04](https://www.cyberciti.biz/faq/ubuntu-20-04-lts-set-up-openvpn-server-in-5-minutes/ "Ubuntu 20.04 LTS Set Up OpenVPN Server In 5 Minutes") |
| Package Manager | [apk](https://www.cyberciti.biz/faq/10-alpine-linux-apk-command-examples/ "10 Alpine Linux apk Command Examples") • [apt-get](https://www.cyberciti.biz/tips/linux-debian-package-management-cheat-sheet.html "Ubuntu/Debian Linux apt-get package management cheat sheet") • [apt](https://www.cyberciti.biz/faq/ubuntu-lts-debian-linux-apt-command-examples/ "apt Command Examples for Ubuntu/Debian for new users") • [yum](https://www.cyberciti.biz/faq/rhel-centos-fedora-linux-yum-command-howto/ "How to use yum command on CentOS/RHEL") |
| Processes Management | [bg](https://www.cyberciti.biz/faq/unix-linux-bg-command-examples-usage-syntax/ "Linux / Unix: bg Command Examples") • [chroot](https://www.cyberciti.biz/faq/unix-linux-chroot-command-examples-usage-syntax/ "Linux / Unix: chroot Command Examples") • [cron](https://www.cyberciti.biz/faq/how-do-i-add-jobs-to-cron-under-linux-or-unix-oses/ "How To Add Jobs To cron Under Linux or UNIX") • [disown](https://www.cyberciti.biz/faq/unix-linux-disown-command-examples-usage-syntax/ "Linux / Unix: disown Command Examples") • [fg](https://www.cyberciti.biz/faq/unix-linux-fg-command-examples-usage-syntax/ "Linux / Unix: fg Command Examples") • [glances](https://www.cyberciti.biz/faq/linux-install-glances-monitoring-tool/ "Linux: Keep An Eye On Your System With Glances Monitor") • [gtop](https://www.cyberciti.biz/howto/gtop-awesome-system-monitoring-dashboard-for-terminal/ "gtop: Awesome system monitoring dashboard for Linux/macOS Unix terminal") • [iotop](https://www.cyberciti.biz/hardware/linux-iotop-simple-top-like-io-monitor/ "Linux iotop Check Whats Stressing & Increasing Load On Hard Disks") • [jobs](https://www.cyberciti.biz/faq/unix-linux-jobs-command-examples-usage-syntax/ "Linux / Unix: jobs Command Examples") • [killall](https://www.cyberciti.biz/faq/unix-linux-killall-command-examples-usage-syntax/ "Linux / Unix: killall Command Examples") • [kill](https://www.cyberciti.biz/faq/unix-kill-command-examples/ "Linux / UNIX: Kill Command Examples") • [pidof](https://www.cyberciti.biz/faq/linux-pidof-command-examples-find-pid-of-program/ "Linux pidof Command Examples To Find PID of A Program/Command") • [pstree](https://www.cyberciti.biz/faq/unix-linux-pstree-command-examples-shows-running-processestree/ "Linux/Unix: pstree Command Examples: See A Tree Of Processes") • [pwdx](https://www.cyberciti.biz/faq/unix-linux-pwdx-command-examples-usage-syntax/ "Linux / Unix: pwdx Command Examples") • [time](https://www.cyberciti.biz/faq/unix-linux-time-command-examples-usage-syntax/ "Linux / Unix: time Command Examples") • [vtop](https://www.cyberciti.biz/faq/how-to-install-and-use-vtop-graphical-terminal-activity-monitor-on-linux/ "How to install and use vtop graphical terminal monitor on Linux") |
| Searching | [ag](https://www.cyberciti.biz/open-source/command-line-hacks/ag-supercharge-string-search-through-directory-hierarchy/ "ag - supercharge string search through a directory on a Linux") • [egrep](https://www.cyberciti.biz/faq/grep-regular-expressions/ "Regular expressions in grep ( regex ) with examples") • [grep](https://www.cyberciti.biz/faq/howto-use-grep-command-in-linux-unix/ "How to use grep command in Linux/ Unix with examples") • [whereis](https://www.cyberciti.biz/faq/unix-linux-whereis-command-examples-to-locate-binary/ "Linux / Unix whereis Command Examples") • [which](https://www.cyberciti.biz/faq/unix-linux-which-command-examples-syntax-to-locate-programs/ "Linux / Unix: which Command Examples To Find Out A Program File ") |
| Shell builtins | [compgen](https://www.cyberciti.biz/open-source/command-line-hacks/compgen-linux-command/ "compgen: An Awesome Command To List All Linux Commands") • [echo](https://bash.cyberciti.biz/guide/Echo_Command "Echo Command - Linux Shell Scripting Wiki") • [printf](https://bash.cyberciti.biz/guide/Printf_command "Printf command - Linux Shell Scripting Wiki") |
| System Management | [reboot](https://www.cyberciti.biz/faq/howto-reboot-linux/ "Reboot Linux System Command") • [shutdown](https://www.cyberciti.biz/faq/howto-shutdown-linux/ "How To Shutdown Linux Using Command Line") |
| Terminal/ssh | [tty](https://www.cyberciti.biz/faq/linux-unix-appleosx-bsd-what-tty-command/ "Find Out What tty Im Using tty command under Linux / Unix") |
| Text processing | [cut](https://bash.cyberciti.biz/guide/Cut_command "Cut command - Linux Shell Scripting Wiki") • [rev](https://bash.cyberciti.biz/guide/Rev_command "Rev command - Linux Shell Scripting Wiki") |
| User Environment | [exit](https://bash.cyberciti.biz/guide/Exit_command "Exit command - Linux Shell Scripting Wiki") • [who](https://www.cyberciti.biz/faq/unix-linux-who-command-examples-syntax-usage/ "Linux / Unix who Command Examples To List Users on The Systems") |
| User Information | [groups](https://www.cyberciti.biz/faq/unix-linux-groups-command-examples-syntax-usage/ "Linux / Unix: groups Command Examples") • [id](https://www.cyberciti.biz/faq/unix-linux-id-command-examples-usage-syntax/ "Linux / Unix id Command Examples") • [lastcomm](https://www.cyberciti.biz/faq/linux-unix-lastcomm-command-examples-usage-syntax/ "Linux / Unix: lastcomm Command Examples") • [last](https://www.cyberciti.biz/faq/linux-unix-last-command-examples/ "Linux / Unix: last Command Examples ") • [lid/libuser-lid](https://www.cyberciti.biz/faq/linux-lid-command-examples-syntax-usage/ "Linux lid (libuser-lid) Command Examples") • [logname](https://www.cyberciti.biz/faq/unix-linux-logname-command-examples-syntax-usage/ "Linux / Unix: logname Command Examples To Display Loginname") • [members](https://www.cyberciti.biz/faq/linux-members-command-examples-usage-syntax/ "Linux members Command Examples") • [users](https://www.cyberciti.biz/faq/unix-linux-users-command-examples-syntax-usage/ "Linux / Unix: users Command Examples") • [whoami](https://www.cyberciti.biz/faq/unix-linux-whoami-command-examples-syntax-usage/ "Linux / Unix: whoami Command Examples") • [w](https://www.cyberciti.biz/faq/unix-linux-w-command-examples-syntax-usage-2/ "Linux / Unix: w Command Examples ") |
| WireGuard VPN | [Alpine](https://www.cyberciti.biz/faq/how-to-set-up-wireguard-vpn-server-on-alpine-linux/ "Alpine Linux set up WireGuard VPN server") • [CentOS 8](https://www.cyberciti.biz/faq/centos-8-set-up-wireguard-vpn-server/ "CentOS 8 set up WireGuard VPN server") • [Debian 10](https://www.cyberciti.biz/faq/debian-10-set-up-wireguard-vpn-server/ "Debian 10 set up WireGuard VPN server") • [Firewall](https://www.cyberciti.biz/faq/how-to-set-up-wireguard-firewall-rules-in-linux/ "How To Set Up WireGuard Firewall Rules in Linux") • [Ubuntu 20.04](https://www.cyberciti.biz/faq/ubuntu-20-04-set-up-wireguard-vpn-server/ "How to set up WireGuard VPN server on Ubuntu 20.04") |