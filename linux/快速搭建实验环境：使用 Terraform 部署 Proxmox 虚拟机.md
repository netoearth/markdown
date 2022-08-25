自从用上 m1 的电脑，本地开发环境偶尔会遇到兼容性的问题。比如之前尝试[用 Colima 在虚拟机中运行容器运行时和 Kubernetes](https://mp.weixin.qq.com/s/V9dTD3R6ApnW3b1DebzdEg)，其实际使用的还是 aarch64 虚拟机，实际使用还是会有些差异。

手上有台之前用的黑苹果小主机，吃灰几个月了，实属浪费。正好元旦假期，有时间折腾一下。

```
CPU: Intel 8700 6C12T
MEM: 64G DDR4
DISK: 1T SSD
```

折腾的目的：

-   将平台虚拟化
-   提供多套实验环境
-   快速创建销毁实验环境
-   体验基础设施即代码 IaaS

主要用到的工具：

-   虚拟化工具 [Proxmox VE](https://www.proxmox.com/en/proxmox-ve)
-   [Terraform](https://www.terraform.io/)：开源的基础设施即代码工具
-   [terraform-provider-proxmox](https://github.com/Telmate/terraform-provider-proxmox)：Terraform Proxmox Provider，通过 Proxmox VE 的 REST API 在创建虚拟机。

## 安装 Proxmox 虚拟化工具

从[官网](https://www.proxmox.com/en/downloads/category/proxmox-virtual-environment) 下载 ISO 镜像，写入到 U 盘中。macOS上推荐使用 balenaEtcher 写盘。

电脑上插入 U 盘并从 U 盘启动，按照步骤一步步完成设置。

官方的 wiki [安装步骤](https://pve.proxmox.com/wiki/Installation)很详细。

安装完成之后就可以创建虚拟机了，可以用命令行 `qm create` 或者 `https://localhost:8006` Web UI来创建。

这样毕竟还是有点麻烦，每次都要执行很多操作。虽说可以编写脚本，但是通用型不够好。因此我们选择 Terraform，实现基础设施即代码。

## 创建 Ubuntu Cloud-Init Template

这里选用 Cloud-Init 的方式，从 cloud-init template 来克隆虚拟机。cloud-init 的虚拟机可以完成一些高级定制的初始化工作，有兴趣的参考 [Cloud Init 文档](https://cloudinit.readthedocs.io/en/latest/topics/examples.html)。

**登陆到 Proxmox VE 宿主机**，使用 Ubuntu 20.04 cloud init image 来创建模板，从官网下载：

```
wget https://cloud-images.ubuntu.com/releases/focal/release/ubuntu-20.04-server-cloudimg-amd64.img
```

执行下面的命令创建一个虚拟机：

```
qm create 9000 --name "ubuntu-2004-cloudinit-template" --memory 1024 --cores 1 --net0 virtio,bridge=vmbr0
qm importdisk 9000 ubuntu-20.04-server-cloudimg-amd64.img local-lvm
qm set 9000 --scsihw virtio-scsi-pci --scsi0 local-lvm:vm-9000-disk-0
qm set 9000 --boot c --bootdisk scsi0
qm set 9000 --ide2 local-lvm:cloudinit
qm set 9000 --serial0 socket --vga serial0
qm set 9000 --agent enabled=1
```

将刚创建好的虚拟机转换成模板：

模板与普通的虚拟机会有些许的不同，使用模板我们可以快速创建虚拟机。这里我们不会用 UI来创建。

![Ubuntu Cloud-Init](https://atbug.oss-cn-hangzhou.aliyuncs.com/2022/01/03/20220102-at-104523.png)

![Clone from template](https://atbug.oss-cn-hangzhou.aliyuncs.com/2022/01/03/20220102-at-104604.png)

## 创建 Proxmox 用户及 API Token

使用 Proxmox VE 的 REST API 需要权限校验，有用户名密码或者 API Token 两种方式。我们选用后者，登陆到 Proxmox 宿主机，执行如下命令创建角色、用户以及 API Token：

```
pveum role add TerraformProv -privs "VM.Allocate VM.Clone VM.Config.CDROM VM.Config.CPU VM.Config.Cloudinit VM.Config.Disk VM.Config.HWType VM.Config.Memory VM.Config.Network VM.Config.Options VM.Monitor VM.Audit VM.PowerMgmt Datastore.AllocateSpace Datastore.Audit"
pveum user add terraform-prov@pve
pveum aclmod / -user terraform-prov@pve -role TerraformProv
pveum user token add terraform-prov@pve terraform-token --privsep=0

┌──────────────┬──────────────────────────────────────┐
│ key          │ value                                │
╞══════════════╪══════════════════════════════════════╡
│ full-tokenid │ terraform-prov@pve!terraform-token   │
├──────────────┼──────────────────────────────────────┤
│ info         │ {"privsep":"0"}                      │
├──────────────┼──────────────────────────────────────┤
│ value        │ 9748c040-a283-4c72-a48b-9ce784778eed │
└──────────────┴──────────────────────────────────────┘
```

这里我们会用到 token 的_full-tokenid_ 和 _value_。

## Terraform

有了 token 和 cloud-init 模板后，就是定义虚拟机了。

安装最新版本的 terraform。

在空目录中创建 `ubuntu.tf` 文件，并按步骤进行配置：

### 配置要使用的 provider：

```
terraform {
  required_providers {
    proxmox = {
      source = "telmate/proxmox"
    }
  }
}
```

### 配置 provider

需要提供 `pm_api_url`、`pm_api_token_id` 和 `pm_api_token_secret`：

```
provider "proxmox" {
  pm_tls_insecure     = true
  pm_api_url          = "https://192.168.1.4:8006/api2/json"
  pm_api_token_id     = "terraform-prov@pve!terraform-token"
  pm_api_token_secret = "9748c040-a283-4c72-a48b-9ce784778eed"
}
```

### 配置虚拟机资源

可以参考[provider 的配置说明](https://github.com/Telmate/terraform-provider-proxmox/blob/master/docs/resources/vm_qemu.md#provision-through-cloud-init)：

```
resource "proxmox_vm_qemu" "proxmox-ubuntu" {
  count = 1
  name  = "ubuntu-${count.index + 1}"
  desc  = "Ubuntu develop environment"

  # 节点名
  target_node = "pve"

  # cloud-init template
  clone = "ubuntu-2004-cloudinit-template"

  # 关机 guest agent
  agent   = 0
  os_type = "ubuntu"
  onboot  = true
  # CPU
  cores    = 4
  sockets  = 1
  cpu      = "host"
  # 内存
  memory   = 16384
  scsihw   = "virtio-scsi-pci"
  bootdisk = "scsi0"

  # 硬盘设置，因计算的方式 101580M 代替 100G
  disk {
    slot     = 0
    size     = "101580M"
    type     = "scsi"
    storage  = "local-lvm"
    iothread = 1
  }

  # 网络
  network {
    model  = "virtio"
    bridge = "vmbr0"
  }

  lifecycle {
    ignore_changes = [
      network,
    ]
  }
  # 记住这里要使用IP CIDR。因为只创建一个虚拟机，虚拟机的 IP 是 192.168.1.91。如果要创建多个虚拟机的话，IP 将会是 .91、.92、.93 。
  ipconfig0 = "ip=192.168.1.9${count.index + 1}/24,gw=192.168.1.2"

  # 用户名和 SSH key
  ciuser  = "addo"
  sshkeys = <<EOF
  SSH KEYS HERE
  EOF
}
```

### 创建虚拟机

第一次需要先执行 `init` 命令进行初始化：

可以使用 `terraform fmt` 和 `terraform validate` 对配置文件进行格式化和校验。

然后执行 `terraform apply` 并输入 `yes` 开始创建虚拟机，

```
proxmox_vm_qemu.proxmox-ubuntu[0]: Creating...
proxmox_vm_qemu.proxmox-ubuntu[0]: Still creating... [10s elapsed]
proxmox_vm_qemu.proxmox-ubuntu[0]: Still creating... [20s elapsed]
proxmox_vm_qemu.proxmox-ubuntu[0]: Still creating... [30s elapsed]
proxmox_vm_qemu.proxmox-ubuntu[0]: Still creating... [40s elapsed]
proxmox_vm_qemu.proxmox-ubuntu[0]: Creation complete after 42s [id=pve/qemu/100]
```

这样虚拟机就创建成功了，使用前面配置的私钥和 IP 地址就可以 ssh 到虚拟机中。

### 销毁虚拟机

虚拟机的销毁也很简单，执行 `terraform destory` 并输入 `yes` 即可。

## 总结

有了 Terraform 和 Proxmox VE 后，就可以愉快的使用干净的实验环境了。但是干净到一些开发中常用软件都没有，使用起来也不方便。

后续考虑通过 cloud-init 来对虚拟机进行高级定制，比如容器环境和 K3s 等等。