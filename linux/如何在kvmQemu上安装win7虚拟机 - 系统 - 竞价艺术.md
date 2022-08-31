出处：[https://github.com/hpaluch/hpaluch.github.io/wiki/Install-Windows7-on-KVM-Qemu](https://github.com/hpaluch/hpaluch.github.io/wiki/Install-Windows7-on-KVM-Qemu)

If you plan to install Windows 7 on KVM/Qemu Hypervisor you would need to resolve few problems:

-   lockup on boot (ending with _Starting Windows_ and animated logo)
-   need to have VirtIO disk and drivers for good IO performance

## Setup for Ubuntu 16.04 LTS 64-bit

Install at least following packages in Ubuntu:

```
sudo apt-get install virt-manager qemu-kvm libvirt-bin ubuntu-vm-builder bridge-utils virt-viewer 
```

> NOTE: You may need to Logout/Login after above installation to get membership in `libvirtd` group

## Requirements

To install Windows 7 you need:

-   Windows 7 installation DVD in ISO file (licensed original of course!)
-   VirtIO driver disk - download from [https://fedorapeople.org/groups/virt/virtio-win/direct-downloads/archive-virtio/virtio-win-0.1.126-2/virtio-win-0.1.126.iso](https://fedorapeople.org/groups/virt/virtio-win/direct-downloads/archive-virtio/virtio-win-0.1.126-2/virtio-win-0.1.126.iso) or look at  
    [https://fedoraproject.org/wiki/Windows\_Virtio\_Drivers](https://fedoraproject.org/wiki/Windows_Virtio_Drivers) for recent version of VirtIO drivers

> NOTE: According to [https://fedoraproject.org/wiki/Windows\_Virtio\_Drivers](https://fedoraproject.org/wiki/Windows_Virtio_Drivers)  
> QXL Graphics driver is included in _virtio-win-0.1.103-1_ and later  
> Therefore we use 0.1.126 here (to fulfill this requirement)

## Creating Win7 VM

-   Start _Virtual Machine Manager_ (either from your favorite Desktop (LXDE), or issuing `virt-manager` from terminal).
-   Click on Icon _Create a new virtual machine_
-   Keep _Local install media (ISO image or CDROM)_ selection and click on _Forward_
-   Select _Use ISO image_ and _Browse..._ to your Win7 installation DVD

> NOTE: OS auto-detection failed in my case, so don't worry - you are not alone

-   Uncheck _automatically detect OS..._
-   select in listboxes:
    
    -   OS Type: _Windows_
    -   Version: _Microsoft Windows 7_
-   Click on _Forward_
-   Use proper CPU and Memory for Win7, reasonable minimums are:
    
    -   1x CPU
    -   2048MB of RAM
-   Keep selected _Enable storage_ and _Create disk image..._ - recommended size is 30GB or more.
-   Click on _Forward_
    
    -   change _Name:_ to something reasonable
    -   Select check-box _Customize configuration before install_
-   Click on _Finish_

Now we need to adjust our configuration:

-   Click on _IDE Disk 1_
    
    -   expand _Advanced_
    -   change _Disk bus_ to _VirtIO_
    -   click on "Apply"
-   click on _Display Spice_
    
    -   change _Type:_ to _VNC Server_
    -   click on _Apply_
-   click on _Video QXL_
    
    -   change _Model:_ to _Cirrus_
    -   click on _Apply_
-   click on _NIC: ..._
    
    -   change _Device Model:_ to _VirtIO_
    -   click on _Apply_
-   Remove _Sound: ich6_
-   click on _Add hardware_
    
    -   select _Storage_
    -   select _Device Type:_ to _CDROM Device_
    -   click on _Finish_
-   click on _IDE CDROM 1_ and change:
    
    -   click on _Connect_ and select your Windows 7 installation DVD ISO
    -   enable checkbox _Shareable_
    -   click on _Apply_
-   click on your _IDE CDROM 2_ and change:
    
    -   click on Connect and select your `virtio-win-0.1.118.iso`
    -   enable checkbox _Shareable_
    -   click on _Apply_

> NOTE: There is some rumor, that ordering DVD as Driver ISO First and OS ISO last will  
> automatically load required drivers - but I had never such luck...

-   click on _Boot options_
    
    -   check _Enable Boot Menu_
    -   enable and reorder these entries so that boot menu looks like:
        
        -   IDE CDROM 1
        -   VirtIO Disk 1
        -   _...others..._
    -   click on Apply

> Now we need to add (temporarily) secondary QXL adapater, otherwise the Windows graphics drivers  
> will be unable to install.
> 
> However - primary adapter must be (temporarily) Cirrus - otherwise Windows would have no chance  
> to display installation setup.

Add QXL using:

-   Click on _Add Hardware_
    
    -   Select _Graphics_
    -   Keep _Type:_ to _Spice Server_
    -   Click _Finish_
-   again Click on _Add Hardware_
    
    -   click on _Video_
    -   Change _Model:_ to _QXL_
    -   click on _Finish_

_Finally_ you can click on _Begin installation_

-   After a while a window _Install Windows_ should appear
-   Click on _Next_
-   click on _Install Now_

> NOTE: At any time you can use _Ctrl-Alt_ to release mouse from KVM to X-Window

-   confirm license terms (but rather don't read them if you want to sleep well)
-   Click on _Custom (Advanced)_ installation Type

> NOTE: there _should_ be warning `No drives were found` - it is expected...

-   do this to fix missing Disk Drive problem:
    
    -   click on _Load Driver_
    -   click on _Browse_
    -   On DVD labeled _virtio-win-0.1.1_ Expand path to `\viostor\w7\amd64`
    -   confirm driver _Red Hat VirtIO SCSI controller_ clicking on _Next_
-   now there should be visible _Disk 0 unallocated space_
-   just click on _Next_ - Windows should start Copying and Expanding files - the installation  
    itself should take about 30minutes

After few reboots you should have Windows 7 desktop ready.  
However there are few things that need to be fixed...

## Installing QEMU drivers

Windows 7 knows nothing about VirtIO (or KVM at all) - so you need to install  
proper drivers (with exception of Disk driver - it was already done in setup part).

-   hold Win+R and type `devmgmt.msc` to Run box
-   a _Device Manager_ should appear
-   Right-click on _Other Devices_ -> _Ethernet Controller_ (with question mark)
    
    -   Click on _Update Driver Software..._
    -   choice _Browse my computer for driver software_
    -   click on _Browse..._ and choice your CD with _virtio-win-0.1.1_ label
    -   ensure that _Include subfolders_ is checked on - click on Next
    -   confirm _always trust sofwtare from "Red Hat, Inc."_

Now you should have installed _RedHat VirtIO Ethernet adapter_

Repeat above steps for remaining _Other devices_:

1.  _PCI Device_ -> should install _VirtIO Balloon Driver_
2.  _PCI Simple Communications Controller_ -> _VirtIO-Serial Driver_
3.  _Video Controller_ -> _Red Hat QXL GPU_

## Windows Guest Agent

In your windows guest run from your _virtio-win-0.1.1_ ISO CD Guest agent - in a path like  
`\guest-agent\qemu-ga-64.msi`

## Enable Accelerated Display Driver

-   Run again Device Manager
-   Verify that QXL is detected and working proberly - there should be a device in:
    
    -   _Display Adapters_ -> _Red Hat QXL GPU_
-   Shutdown your Windows Guest (thanks to Guest Agent it should be even possible to use  
    _Virtual Machine Manager_ -> _Shut Down_)
-   Switch Manager to _i_ icon (_Show Virtual Hardware Details_)
-   Remove these devices:
    
    -   _Display VNC_
    -   _Video Cirrus_

Power on your Windows Guest - it should work with QXL/Spice now...

Spice can be verified using `virsh` for example:

```
virsh
virsh # list
 Id    Name                           State
----------------------------------------------------
 4     win7tutor                      running

virsh # domdisplay win7tutor
spice://127.0.0.1:5900
```

In Windows Guest you can verify or change your Adapater settings using:

-   Right-Click somewhere on Windows Desktop
-   click on _Screen resolution_
-   You should see _Display:_ like: _1\. (Default Monitor) on Red Hat QXL GPU_

That's all.

## Resources

-   [http://www.linux-kvm.org/page/Windows7Install](http://www.linux-kvm.org/page/Windows7Install)