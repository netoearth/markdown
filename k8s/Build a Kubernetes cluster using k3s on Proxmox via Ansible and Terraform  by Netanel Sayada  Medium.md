![](https://miro.medium.com/max/1400/1*jL6SE1nSaPQb4EOWGnbZpw.jpeg)

This combination is absolutely amazing!

i always wanted to build a mini pc cluster at my home, but haven't find the right hardware yet. i saw a lot of the self-hosted community running their cluster on Proxmox, so i decided to give it a go… my way of curse :)

Proxmox is an open-source hypervisor that have enterprise capabilities and a large community behind it.

For Terraform and Ansible, i always like the idea of infrastructure as code (iac) and Terraform and Ansible just make it easy to accomplish.

the idea here was to be able to spin up a k3s cluster with minimum effort so i can spin it up and down for ever project that i would like to run.

lets see how to set it up!

## System requirements

-   [The deployment environment must have Ansible 2.4.0+](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html)
-   [Terraform installed](https://learn.hashicorp.com/tutorials/terraform/install-cli)
-   [Proxmox server](https://www.proxmox.com/en/)

## Proxmox setup

This setup is relaying on [cloud-init](https://cloudinit.readthedocs.io/en/latest/index.html) images.

Using cloud-init image save us a ton of time and it’s works great! I use ubuntu focal image, but you can use whatever distro you like.

to configure the cloud-init image you will need to connect to a Linux server and run the following (these tools cannot be installed on the Proxmox server):

install image tools on the server:

```
apt-get install libguestfs-tools
```

Get the image that you would like to work with. you can browse to [https://cloud-images.ubuntu.com](https://cloud-images.ubuntu.com/) and select any other version that you would like to work with. for Debian, got to [https://cloud.debian.org/images/cloud/](https://cloud.debian.org/images/cloud/).

```
wget https://cloud-images.ubuntu.com/focal/current/focal-server-cloudimg-amd64.img
```

so now we can edit our base image an install Proxmox agent — this is necessary if we want Terraform to work properly. if you have other packages that you need on your base image, this will be the place to add them. it can take some time for the image to update.

```
virt-customize -a focal-server-cloudimg-amd64.img --install qemu-guest-agent
```

now that we have the image updated, we need to move it to the Proxmox server. we can do that using scp:

```
scp focal-server-cloudimg-amd64.img Proxmox_username@Proxmox_host:/path_on_Proxmox/focal-server-cloudimg-amd64.img
```

so now we should have the image configured and placed on our Proxmox server. let’s start creating the VM

```
qm create 9000 --name "ubuntu-focal-cloudinit-template" --memory 2048 --net0 virtio,bridge=vmbr0
```

rename the image suffix

```
mv focal-server-cloudimg-amd64.img focal-server-cloudimg-amd64.qcow2
```

import the disk to the VM

```
qm importdisk 9000 focal-server-cloudimg-amd64.qcow2 local-lvm
```

configure the VM to use the new image

```
qm set 9000 --scsihw virtio-scsi-pci --scsi0 local-lvm:vm-9000-disk-0
```

add cloud-init image to the VM

```
qm set 9000 --ide2 local-lvm:cloudinit
```

set the VM to boot from the cloud-init disk:

```
qm set 9000 --boot c --bootdisk scsi0
```

update the serial on the VM

```
qm set 9000 --serial0 socket --vga serial0
```

Good! so we are almost done with the image. now we can setup our base configuration for the image. connect to the Proxmox server and go to your VM, check the cloud-init tab, there you will find some more parameters that we will need to change.

![](https://miro.medium.com/max/1400/0*KBK80mz-NxXxWAF2)

you will need to change the user name, password, and add the ssh public key so we can connect to the VM later using Ansible and terraform. update the variables and click on Regenerate Image

you can also resize the main disk if you need in this stage. i usually resize them to 15 GB.

Great! so now we can convert the VM to a template and start working with terraform.

```
qm template 9000
```

## terraform setup

Clone the repo to get all the files and cd into the folder.

```
git clone https://github.com/NatiSayada/k3s-proxmox-terraform-ansible
```

our terraform also creates a dynamic host file for Ansible, so we need to create the files first

```
cp -R inventory/sample inventory/my-cluster
```

Rename the file terraform/vars.sample to terraform/vars.tf and update all the vars. there you can select how many nodes would you like to have on your cluster and configure the name of the base image.

to run the Terrafom, you will need to cd into terraform and run:

```
terraform initterraform planterraform apply
```

it can take some time to create the servers on Proxmox but you can monitor them over Proxmox. it should look like this now:

Add alt text

![](https://miro.medium.com/max/864/0*vnMepxEQgFND4dOw)

## Ansible setup

First, update the var file in inventory/my-cluster/group\_vars/all.yml and update the user name that you’re selected in the cloud-init setup.

after you run the Terrafom file, your host file should look like this:

```
[master]192.168.3.200 Ansible_ssh_private_key_file=~/.ssh/proxk3s[node]192.168.3.202 Ansible_ssh_private_key_file=~/.ssh/proxk3s192.168.3.201 Ansible_ssh_private_key_file=~/.ssh/proxk3s192.168.3.198 Ansible_ssh_private_key_file=~/.ssh/proxk3s192.168.3.203 Ansible_ssh_private_key_file=~/.ssh/proxk3s[k3s_cluster:children]masternode
```

Start provisioning of the cluster using the following command:

```
Ansible-playbook site.yml -i inventory/my-cluster/hosts.ini
```

this playbook will install k3s in 644 mode and helm.

the 644 mode is the permission needed for the /etc/rancher/k3s/k3s.yaml config file so it can be imported to rancher. so if you would also like to check out rancher.. you are good to go!

## Kubeconfig

To get access to your Kubernetes cluster just copy the k3s yaml file to your kube config file and change the ip address of the server

```
scp debian@master_ip:/etc/rancher/k3s/k3s.yaml ~/.kube/config
```

run _kubectl get nodes_ to check you cluster nodes status

![](https://miro.medium.com/max/1272/1*JgAE4EKXnCL-bEp7p0kOkg.png)

## Summary

Now you should have a full blown k3s cluster running on Proxmox! all you have left is to start running some deployments.

let me know how this was working for you.