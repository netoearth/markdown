For about 2 years now, I’ve been very interested in learning Kubernetes and potentially using it in my homelab, but there’s always been one main thing holding me back: the complexity. To get Kubernetes up the first time, I had to spin up a bunch of VMs manually and then put tons of binaries on each one of them. Even if I could get a cluster spun up, there was no chance I could reproduce my work or deploy anything useful on the cluster.

All of this changed when I found out about [k3s](https://github.com/rancher/k3s). k3s allows you to run a full Kubernetes-compliant cluster without “a PhD in k8s clusterology”. Its main innovation is packaging all the normal Kubernetes components into a single binary and refactoring some authentication to make it much easier for nodes to join the cluster.

Combined with some Ansible and Terraform knowledge, I decided to get cooking on some automation to make spinning up a new cluster a simple command.

## Prerequisites

There are a few things I’m not going to cover in this guide that you need to get started:

-   Install Terraform
    -   Just download Terraform from [the downloads page](https://www.terraform.io/downloads.html) and drop it somewhere in your `$PATH`
-   Install the [Terraform Proxmox provider](https://github.com/Telmate/terraform-provider-proxmox)
    -   Use the `go install` command found in the README
    -   Drop the binaries into `~/.terraform.d/plugins/`
-   Have a Proxmox host
-   Install [Ansible](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html)

## Building a cloud-init Ubuntu template

In order for Terraform to work smoothly with new VMs, we need to make a template VM that has [cloud-init](https://cloud-init.io/) on it. Cloud-init adds some packages to the VM that makes automatic provisioning possible. Thankfully, Proxmox has pretty good support for it.

SSH into your Proxmox host, and enter these commmands to create an Ubuntu VM with cloud-init:

```
$ wget https://cloud-images.ubuntu.com/bionic/current/bionic-server-cloudimg-amd64.img
# Export whatever storage pool you want to use to hold your VM images. Mine just happens to be named hermes_data
$ export STORAGE_POOL=hermes_data
$ qm create 8000 --memory 2048 --net0 virtio,bridge=vmbr0
$ qm importdisk 8000 bionic-server-cloudimg-amd64.img $STORAGE_POOL
Formatting '/data/images/8000/vm-8000-disk-0.raw', fmt=raw size=2361393152
    (100.00/100%)
$ qm set 8000 --scsihw virtio-scsi-pci --scsi0 $STORAGE_POOL:8000/vm-8000-disk-0.raw
update VM 8000: -scsi0 hermes_data:8000/vm-8000-disk-0.raw -scsihw virtio-scsi-pci
$ qm set 8000 --name ubuntu-ci
update VM 8000: -name ubuntu-ci
```

Now we have a VM with all the appropriate cloud-init packages installed. Now we just have to attach the hardware that cloud-init requires, and we can template the VM.

```
$ qm set 8000 --ide2 $STORAGE_POOL:cloudinit
update VM 8000: -ide2 hermes_data:cloudinit
Formatting '/data/images/8000/vm-8000-cloudinit.qcow2', fmt=qcow2 size=4194304 cluster_size=65536 preallocation=metadata lazy_refcounts=off refcount_bits=16
$ qm set 8000 --boot c --bootdisk scsi0
update VM 8000: -boot c -bootdisk scsi0
$ qm set 8000 --serial0 socket --vga serial0
update VM 8000: -serial0 socket -vga serial0
$ qm template 8000
```

There we go! We now have a template VM that we can build our k3s nodes off of.

## Deploying the VMs

Now, on whatever machine you have Ansible and Terraform installed on, clone down my `proxmox-k3s` repo:

```
$ git clone https://github.com/Pwpon500/proxmox-k3s
Cloning into 'proxmox-k3s'...
remote: Enumerating objects: 64, done.
remote: Counting objects: 100% (64/64), done.
remote: Compressing objects: 100% (38/38), done.
remote: Total 64 (delta 3), reused 64 (delta 3), pack-reused 0
Unpacking objects: 100% (64/64), done.
$ cd proxmox-k3s/proxmox-tf/prod
```

Now, edit the `main.tf` file to reflect the IPs of all your nodes as well as what SSH keys you want to use. Next, export the appropriate variables so that the Proxmox Terraform provider can connect to the Proxmox API:

```
$ export PM_API_URL="https://<node_ip>:8006/api2/json"<Paste>
$ export PM_USER=root@pam
$ export PM_PASS=<your_pass_here>
```

If you don’t set any of these, you’ll be prompted for them whenever you do a `terraform plan` or a `terraform apply`.

Without any further ado, let’s create some VMs!

```
$ terraform init
$ terraform plan
$ terraform apply
```

These commands will have a lot of output, but I don’t want to paste it all here. It should be clear if things have worked.

Wait a few minutes for the VMs to finish doing their cloud-init inital configuration, and continue to the next step.

## Applying k3s Configs

Now, go into the `ansible-roles` directory. All you need to edit here is the `inventory.toml` file. Change the IPs to whatever yours are, and add as many hosts as you created. The only important thing to remember is to have exactly 1 master node and to make the rest workers.

Once you’ve entered in the appropriate IPs, apply your playbook:

```
$ ansible-playbook -i inventory.toml playbook.yml -u ubuntu
```

This command will take a few minutes to complete, but if it finishes successfully, you have a fully working k3s cluster!

## Testing Out the Cluster

To test out the cluster, we’re going to go into the master node and use the in-built `k3s kubectl`:

```
$ ssh ubuntu@172.30.100.80
ubuntu@k3s-node-0:~$ sudo k3s kubectl get nodes
NAME         STATUS   ROLES    AGE   VERSION
k3s-node-0   Ready    master   2d    v1.14.4-k3s.1
k3s-node-1   Ready    worker   2d    v1.14.4-k3s.1
k3s-node-2   Ready    worker   2d    v1.14.4-k3s.1
k3s-node-3   Ready    worker   2d    v1.14.4-k3s.1
```

Looks pretty good. We have all the nodes properly joined up. Let’s start a pod just to test things out:

```
$ sudo k3s kubectl run lab-nginx --image=nginx --port=80
kubectl run --generator=deployment/apps.v1 is DEPRECATED and will be removed in a future version. Use kubectl run --generator=run-pod/v1 or kubectl create instead.
deployment.apps/lab-nginx created
$ sudo k3s kubectl expose deployment lab-nginx --type=NodePort
service/lab-nginx exposed
$ sudo k3s kubectl port-forward svc/lab-nginx 8080:80 &
$ curl localhost:8080
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

Woohoo! Looks to me like this thing’s working. If you want to get out the kubeconfig to run `kubectl` from your local machine, it’s in `/etc/rancher/k3s/k3s.yaml`.

Now if you don’t want it anymore, you can do a simple `terraform destroy` in the `proxmox-tf` directory, and you’re back to where you started.

## Conclusion

Obviously, what we’ve done here is exciting because we get a Kubernetes cluster with a pretty simple process. It’s also crazy useful that we can now spin up entire VM clusters with just a simple command. Before, I had to do this through the Proxmox Web UI, but not anymore.

I hope all this was helpful! I had a ton of fun with this, and there’s more of this kind of material coming in the future.