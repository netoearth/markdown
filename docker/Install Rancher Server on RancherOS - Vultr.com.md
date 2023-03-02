## Overview

RancherOS is an incredibly lightweight operating system (only about 60 MB) that runs a "system" Docker daemon as **PID 0** for running system services, (networking, console access and so on), as well as a "user" Docker daemon for running non-system containers, (MySQL, Rancher and more).

Rancher is a container-run orchestration platform for managing containers, as well as broader aspects of infrastructure such as hosts, environments and more. A Rancher server controls the orchestration, and a Rancher agent is deployed to each host that is managed by Rancher.

In this article, we'll go through the following steps to deploy a Rancher server:

-   **Cloud-init script** - Publish a cloud-init file to install and run the Rancher server.
    
-   **PXE script** - Write a PXE script to retrieve the cloud-init file and boot up the host for the first time.
    
-   **Firewall** - Create a firewall group, because security is paramount.
    
-   **Start it up** - Provision the host and install Rancher.
    

## Requirements

-   VPS with a minimum of 1 GB RAM - We'll install Rancher server on this host.
    
-   Block storage - To persistently store Rancher server's data, configurations, users and more.
    
-   1 Reserved IP address - To give Rancher agents a consistent IP to use to join the Rancher environment.
    

## Cloud-init script

Save the following script to a location reachable by your host via **HTTP/HTTPS** so it can reference it from its PXE script.

Replace the **ssh-...** parts with your SSH public key so you can SSH into the host.

```
#cloud-config

ssh_authorized_keys:

  - ssh-...



write_files:

  - path: /cloud-config.yml

    permissions: "0700"

    owner: root

    content: |

      #cloud-config

      ssh_authorized_keys:

        - ssh-...



      mounts:

       - ["/dev/vdb1", "/mnt", "ext4", ""] 



      rancher:

        services:

          rancher-server:

            image: rancher/server:stable

            ports:

              - 8080:8080

            restart: always

            volumes:

              - /mnt/rancher-server-mysql:/var/lib/mysql

  - path: /opt/rancher/bin/start.sh

    permissions: "0700"

    owner: root

    content: |

      #!/bin/bash

      echo y | ros install -f -c /cloud-config.yml -d /dev/vda
```

Note that this is actually planting a **cloud-config.yml** inside another **cloud-config.yml**. The outer one is loaded when iPXE boots the host for the first time, and it installs Rancher to the host's drive **/dev/vda**. The inner config is for subsequent boots and will actually start the Rancher server.

The MySQL data is stored on the block storage **/dev/vdb**, so the critical Rancher server data and configurations can survive replacement of the VPS host.

You can upload the script to any number of free locations that are reachable as a URL publicly, or you can host it on a different VPS so it's only accessible by your hosts via a private network.

## PXE script

Copy the following as a PXE startup script called "**Rancher Server**" while replacing **CLOUD\_CONFIG\_URL** with the URL of your **cloud-config.yml** file (something like `https://example.com/cloud-config.yml`).

```
#!ipxe



# Location of Kernel/Initrd images

set base-url https://releases.rancher.com/os/latest



kernel ${base-url}/vmlinuz rancher.state.dev=LABEL=RANCHER_STATE -- rancher.cloud_init.datasources=[url:CLOUD_CONFIG_URL]



initrd ${base-url}/initrd



boot
```

This will pull the latest RancherOS ISO and boot it to memory using your cloud-init script. Your cloud-init script will then proceed to install RancherOS to disk, and the second boot will run the Rancher server container.

## Firewall

When Rancher first becomes available, anyone who hits the endpoint will immediately have admin privileges.

To prevent outsiders from hijacking your Rancher server, create a firewall called "Rancher Server" with the following rules:

-   **TCP 22** on your IP, so you can SSH into the host.
    
-   **TCP 8080** on your IP, so you can load the Rancher server webpage.
    
-   **TCP 8080** for any Rancher agent hosts, so they can register with the Rancher server.
    

## Start it up

Provision your 1+ GB host in the same region as your block storage, and set its **Server Type** to the "Rancher Server" iPXE custom startup script.

Once it's booted, be sure to convert its IP to a reserved IP so your Rancher agents have an endpoint they can register with consistently.

It will take ~4 minutes for iPXE to pull the RancherOS ISO, the first boot to install RancherOS to **/dev/vda**, and for the second boot to pull the **rancher/server:stable** Docker image and start up its containers.

Once it's up, you will be able to reach it at **[http://YOUR\_RESERVED\_IP:8080](http://your_reserved_ip:8080/)**.

Congratulations, you've just set up Rancher server on RancherOS.

You can restart your instance or even destroy / re-install it, and the block storage will preserve your data and configurations while your reserved IP will allow new Rancher agents to know where to find your server.

A few next steps:

-   **Set up access control** - at the very least, create a local admin user with a secure password.
    
-   **Add hosts** - in the **Add Hosts -> Custom** section, copy the URL that contains a long token specific to your Rancher server. You'll need this to register Rancher agents with your server.
    
-   **Explore** the [latest Rancher server documentation](http://rancher.com/docs/rancher/latest/en/).