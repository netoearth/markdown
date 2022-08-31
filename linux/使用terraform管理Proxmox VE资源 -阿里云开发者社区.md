## terraform-proxmox

使用terraform管理Proxmox VE资源  
Using terraform to manage proxmox resources

env:  
Proxmox VE v6.1  
Terraform v0.11.14  
provider.proxmox v0.1.0

By Elvin ,2020-8-23, [](http://blog.elvin.vip/)[http://blog.elvin.vip](http://blog.elvin.vip/)  
git source [](https://gitee.com/alivv/terraform-proxmox.git)[https://gitee.com/alivv/terraform-proxmox.git](https://gitee.com/alivv/terraform-proxmox.git)

___

##### install terraform v0.11.14

```
wget http://files.elvin.vip/cli/terraform_0.11.14_linux_amd64.zip
unzip -oq terraform_0.11.14_linux_amd64.zip -d /usr/local/bin/
rm -f terraform_0.11.14_linux_amd64.zip
```

##### install terraform-provider-proxmox v0.1.0

```
wget http://files.elvin.vip/cli/terraform-provider-proxmox_0.1.0_linux_amd64.tar.gz
mkdir -p ~/.terraform.d/plugins
tar -zxf terraform-provider-proxmox_0.1.0_linux_amd64.tar.gz -C ~/.terraform.d/plugins/
rm -f    terraform-provider-proxmox_0.1.0_linux_amd64.tar.gz
```

##### clone demo

```
git clone https://gitee.com/alivv/terraform-proxmox.git /opt/
```

##### test

```
cd  /opt/terraform-proxmox/test

#config pm_api_url,pm_password,clone,storage ...
#vim vm.tf 

#test
terraform version
terraform init
terraform plan

#创建VM, 从模板ubuntu克隆vm
#Create VM, Cloning VM from template Ubuntu

terraform apply

#删除VM, Delete VM
terraform destroy
```

##### Terraform template

```
#使用terraform模板,批量创建VM
#Create VM in batch using terraform template

cd /opt/terraform-proxmox/vm

#mv list
cat > vm.list.txt << EOF
# name,count,cpu,ram,disk,vid,os(default ubuntu),notes(Optional)

vm-ubuntu,2,2,4,20,161,ubuntu
vm-centos,2,4,8,20,163,centos,centos for test
EOF

#make vm.tf and outputs.tf from vm.list.txt
./make.vm.tf.sh

#run check
terraform init
terraform plan

#Create VM
terraform apply
```

##### Run the demo results

```
# terraform version

Terraform v0.11.14
+ provider.proxmox v0.1.0

# terraform plan

Refreshing Terraform state in-memory prior to plan...
The refreshed state will be used to calculate this plan, but will not be
persisted to local or remote state storage.

------------------------------------------------------------------------

An execution plan has been generated and is shown below.
Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  + module.vm-centos.proxmox_vm_qemu.cloudinit-test[0]
      id:                           <computed>
      agent:                        "1"
      balloon:                      "0"
      bios:                         "seabios"
      boot:                         "cdn"
      clone:                        "centos"
      clone_wait:                   "15"
      cores:                        "4"
      cpu:                          "host"
      desc:                         "centos for test"
      disk.#:                       "1"
      disk.1057333298.backup:       "false"
      disk.1057333298.cache:        "none"
      disk.1057333298.discard:      "on"
      disk.1057333298.format:       "qcow2"
      disk.1057333298.id:           "0"
      disk.1057333298.iothread:     "true"
      disk.1057333298.mbps:         "0"
      disk.1057333298.mbps_rd:      "0"
      disk.1057333298.mbps_rd_max:  "0"
      disk.1057333298.mbps_wr:      "0"
      disk.1057333298.mbps_wr_max:  "0"
      disk.1057333298.replicate:    "false"
      disk.1057333298.size:         "20G"
      disk.1057333298.ssd:          ""
      disk.1057333298.storage:      "data"
      disk.1057333298.storage_type: "dir"
      disk.1057333298.type:         "scsi"
      force_create:                 "false"
      full_clone:                   "true"
      hotplug:                      "network,disk,usb"
      ipconfig0:                    "ip=192.168.21.163/24,gw=192.168.21.1"
      kvm:                          "true"
      memory:                       "8192"
      name:                         "vm-centos-1"
      numa:                         "false"
      onboot:                       "true"
      os_type:                      "cloud-init"
      preprovision:                 "true"
      sockets:                      "1"
      ssh_host:                     <computed>
      ssh_port:                     <computed>
      target_node:                  "n11"
      vcpus:                        "0"
      vlan:                         "-1"
      vmid:                         "163"

  + module.vm-centos.proxmox_vm_qemu.cloudinit-test[1]
...
```

___

provider.proxmox source  
[](https://github.com/Telmate/terraform-provider-proxmox/)[https://github.com/Telmate/terraform-provider-proxmox/](https://github.com/Telmate/terraform-provider-proxmox/)