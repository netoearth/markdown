-   Note: All servers must have a valid SUSE SLES license and must connect to SUSE customer center or a local Repository Mirroring Tool (RMT) server. To update the servers to the latest updates, run \- zypper update. These steps have been adapted from the [quick start guide](https://docs.rke2.io/install/quickstart/) and serve as an example.
    
    1.  Disable firewalld and apparmor in all three servers as root (if not already disabled):
    
    $ sudo -i
    
    $ systemctl disable apparmor.service
    
    $ systemctl disable firewalld.service
    
    $ systemctl stop apparmor.service
    
    $ systemctl stop firewalld.service
    
    2.  Ensure that SWAP is disabled and not running. If swap has not been removed from the disk proposal on the install, do the following:
    
    $ systemctl disable swap.target
    
    $ swapoff -a
    
    3.  Download and install RKE2 in all three nodes. In SLES, this script will download the latest RKE2 tarballs, extract in a /tmp directory, and then copy them to their final locations:
    
    $ curl -sfL https://get.rke2.io | sh -
    
    Use the following command to download a specific version of RKE2 if needed
    
    $ curl -sfL https://get.rke2.io |\\ INSTALL\_RKE2\_VERSION=${RKE2\_VERSION} sh -
    
    4.  Create a round-robin DNS for the first RKE2 node.
    5.  Create the folder /etc/rancher/rke2 on the first node.
    
    $ mkdir -p /etc/rancher/rke2
    
    6.  On the first node, navigate to the newly created folder and create a config.yaml file.
    
    $ vi config.yaml
    
    token: my-shared-secret
    
    tls-san:
    
      \- rancher.apex.dell.com
    
      \- rke2node1.apex.dell.com
    
      \- rke2node2.apex.dell.com
    
      \- rke2node3.apex.dell.com
    
    Note: RKE2 setup on SLES15 SP4 requires unique hostnames or the node-name parameter set in /etc/rancher/rke2/config.yaml. Multiple nodes require several TCP/UDP ports to be configured properly. See the [quick start guide](https://docs.rke2.io/install/quickstart/) for details.
    
    7.  Enable the service for the first RKE2 node. This will enable the service installed using the shell script run previously.
    
    $ systemctl enable --now rke2-server.service
    
    8.  Create the folder /etc/rancher/rke2 on the second and third nodes.
    
    $ mkdir -p /etc/rancher/rke2
    
    9.  On the second and third nodes navigate to the newly created folder and create a config.yaml file with the following content:
    
    $ vi config.yaml
    
    server: https://rancher.apex.dell.com:9345
    
    token: my-shared-secret
    
    tls-san:
    
      \- rancher.apex.dell.com
    
      \- rke2node1.apex.dell.com
    
      \- rke2node2.apex.dell.com
    
      \- rke2node3.apex.dell.com
    
    10.  Enable the service for the second and third RKE2 nodes. Run this command on the first node and wait a few minutes for the other two nodes to join. This will enable the service installed using the shell script run previously.
    
    $ systemctl enable --now rke2-server.service
    
    11.  Create a round-robin DNS for the remaining RKE2 nodes.
    12.  Copy kubectl to the local user bin folder.
    
    $ cp /var/lib/rancher/rke2/bin/kubectl /usr/local/bin
    
    13.  Add kubectl to the PATH on the first server.
    
    $ export PATH=$PATH:/opt/rke2/bin:/var/lib/rancher/rke2/bin
    
    14.  Export the kubeconfig file on the first server.
    
    $ export KUBECONFIG=/etc/rancher/rke2/rke2.yaml
    
    15.  Check the health of the deployment by running a status command.
    
    $ kubectl get componentstatuses
    
    $ kubectl get nodes