The qemu KVM hypervisor allows a Linux user to install a virtualized operating system that will run very fast indeed once installed.

To install the qemu KVM hypervisor; enter this command.

```
sudo apt-get install qemu-kvm qemu-utils
```

Then create a raw disk image to house your Windows operating system.

```
qemu-img create -f raw win7.img 30G
```

This will take no time at all and this will only require special drivers to be loaded with the hypervisor to enable Windows to use the raw disk image.

Download the ISO image here: [https://fedorapeople.org/groups/virt/virtio-win/direct-downloads/stable-virtio/virtio-win.iso](https://fedorapeople.org/groups/virt/virtio-win/direct-downloads/stable-virtio/virtio-win.iso). This is loaded by our script to load our complete hypervisor setup.

Here is our script. I am using a Windows 7 Ultimate ISO I had on my hard disk. This is installed in the VM.

```
#!/bin/sh
 
export QEMU_AUDIO_DRV=alsa
DISKIMG=~/win7.img
WIN7IMG=~/Downloads/win7.iso
VIRTIMG=~/Downloads/virtio-win-0.1-81.iso
qemu-system-x86_64 --enable-kvm -drive format=raw,file=${DISKIMG},if=virtio -m 4096 \
-net nic,model=virtio -net user -cdrom ${WIN7IMG} \
-drive file=${VIRTIMG},index=3,media=cdrom \
-rtc base=localtime,clock=host -smp cores=4,threads=4 \
-usbdevice tablet -soundhw ac97 -cpu host -vga vmware
```

Once the Virtual Machine has started, and you get to the disk partitioning section, load the drivers from the virtio ISO disk and choose the SCSI driver. This will enable you to partition the drive and continue with the installation. Below is what you should end up with. I know I should not have the crappy Driver Detective software installed, but it is only a VM. The networking drivers are also on the virtio ISO and may be installed after the installation is complete. Just use Device Manager to take care of that. Networking is like Virtualbox in NAT mode. The internal guest OS will have an IP address, but you cannot ping it from the host. But Internet access works like a charm. Installing Firefox is recommended though. Internet Explorer is not very secure.

[![Windows 7 in the qemu KVM.](https://www.securitronlinux.com/wp-content/uploads/2014/03/Screenshot-QEMU-960x566.png)](http://www.securitronlinux.com/wp-content/uploads/2014/03/Screenshot-QEMU.png)

Windows 7 in the qemu KVM.

Give this a try and see how you go!