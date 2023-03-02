**TL;DR** [SSH Port Forwarding as a printable cheat sheet](https://iximiuz.com/ssh-tunnels/ssh-tunnels.png).

SSH is [yet another example](https://iximiuz.com/en/posts/linux-pty-what-powers-docker-attach-functionality/) of an ancient technology that is still in wide use today. It may very well be that learning a couple of SSH tricks is more profitable in the long run than mastering a dozen Cloud Native tools destined to become deprecated next quarter.

One of my favorite parts of this technology is SSH Tunnels. With nothing but standard tools and often using just a single command, you can achieve the following:

-   Access internal VPC endpoints through a public-facing EC2 instance.
-   Open a port from the localhost of a development VM in the host's browser.
-   Expose any local server from a home/private network to the outside world.

And more 😍

But despite the fact that I use SSH Tunnels daily, it always takes me a while to figure out the right command. Should it be a Local or a Remote tunnel? What are the flags? Is it a _local\_port:remote\_port_ or the other way around? So, I decided to finally wrap my head around it, and it resulted in a series of labs and a visual cheat sheet 🙈

## Prerequisites

SSH Tunnels are about connecting hosts over the network, so every lab below expectedly involves multiple "machines". However, I'm too lazy to spin up full-blown instances, especially when containers can be used instead. That's why I ended up using just [a single vagrant VM with Docker on it](https://iximiuz.com/en/posts/how-to-setup-development-environment/).

In theory, any Linux box with Docker Engine on it should do. However, running the below examples as-is with Docker Desktop won't be possible because the ability to access the ~machines~ containers by their IPs is assumed.

Alternatively, the labs can be done with [Lima (QEMU + nerdctl + containerd + BuildKit)](https://github.com/lima-vm/lima), but don't forget to `limactl shell bash` first.

Every example requires a valid passphrase-less key pair on the host that is then mounted into the containers to simplify access management. If you don't have one, generating it is as simple as just `ssh-keygen` on the host.

**Important:** SSH daemons in the containers here are solely for educational purposes - containers in this post are meant to represent full-blown "machines" with SSH clients and servers on them. Beware that it's rarely a good idea to have SSH stuff in real-world containers!

Starting from the one that I use the most. Oftentimes, there might be a service listening on localhost or a private interface of a machine that I can only SSH to via its public IP. And I desperately need to access this port from the outside. A few typical examples:

-   Accessing a database (MySQL, Postgres, Redis, etc) using a fancy UI tool from your laptop.
-   Using your browser to access a web application exposed only to a private network.
-   Accessing a container's port from your laptop without publishing it on the server's public interface.

All of the above use cases can be solved with a single `ssh` command:

```
ssh -L [local_addr:]local_port:remote_addr:remote_port [user@]sshd_addr
```

The `-L` flag indicates we're starting a _local port forwarding_. What it actually means is:

-   On your machine, the SSH client will start listening on `local_port` (likely, on `localhost`, but _it depends_ - [check the `GatewayPorts` setting](https://linux.die.net/man/5/sshd_config#GatewayPorts)).
-   Any traffic to this port will be forwarded to the `remote_private_addr:remote_port` on the machine you SSH-ed to.

Here is how it looks on a diagram:

![SSH Tunnels visualized - local port forwarding.](https://iximiuz.com/ssh-tunnels/local-port-forwarding-2000-opt.png)

Lab 1: Using SSH Tunnels for Local Port Forwarding 👨🔬

The lab reproduces the setup from the diagram above. First, we need to prepare the server - a machine with the SSH daemon and a simple web service listening on `127.0.0.1:80`:

```
$ docker buildx build -t server:latest -<<'EOD'
# syntax=docker/dockerfile:1
FROM alpine:3

# Install the dependencies:
RUN apk add --no-cache openssh-server curl python3
RUN mkdir /root/.ssh && chmod 0700 /root/.ssh && ssh-keygen -A

# Prepare the entrypoint that starts the daemons:
COPY --chmod=755 <<'EOF' /entrypoint.sh
#!/bin/sh
set -euo pipefail

for file in /tmp/ssh/*.pub; do
  cat ${file} >> /root/.ssh/authorized_keys
done
chmod 600 /root/.ssh/authorized_keys

# Minimal config for the SSH server:
sed -i '/AllowTcpForwarding/d' /etc/ssh/sshd_config
sed -i '/PermitOpen/d' /etc/ssh/sshd_config
/usr/sbin/sshd -e -D &

python3 -m http.server --bind 127.0.0.1 ${PORT} &

sleep infinity
EOF

# Run it:
CMD ["/entrypoint.sh"]
EOD
```

Starting the server and noting its IP address:

```
$ docker run -d --rm \
   -e PORT=80 \
   -v $HOME/.ssh:/tmp/ssh \
   --name server \
   server:latest

SERVER_IP=$(
  docker inspect \
    -f '{{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}}' \
  server
)
```

Since the web service listens on the localhost, it won't be accessible from the outside (i.e., from the host system in this particular case):

```
$ curl ${SERVER_IP}
curl: (7) Failed to connect to 172.17.0.2 port 80: Connection refused
```

But from the inside of the "server", it works just fine:

```
$ ssh -o StrictHostKeyChecking=no root@${SERVER_IP}
7b3e49181769:$# curl localhost
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN" "http://www.w3.org/TR/html4/strict.dtd">
<html>
<head>
...
```

**And here is the trick:** bind the server's `localhost:80` to the host's `localhost:8080` using local port forwarding:

```
$ ssh -o StrictHostKeyChecking=no -f -N -L 8080:localhost:80 root@${SERVER_IP}
```

Now you should be able to access the web service on a local port of the host system:

```
$ curl localhost:8080
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN" "http://www.w3.org/TR/html4/strict.dtd">
<html>
<head>
...
```

A slightly more verbose (but more explicit and flexible) way to achieve the same goal - using the `local_addr:local_port:remote_addr:remote_port` form:

```
$ ssh -o StrictHostKeyChecking=no -f -N -L \
  localhost:8080:localhost:80 \
  root@${SERVER_IP}
```

## Local Port Forwarding with a Bastion Host

It might not be obvious at first, but the `ssh -L` command allows forwarding a local port to a remote port on _any machine_, not only on the SSH server itself. Notice how the `remote_addr` and `sshd_addr` may or may not have the same value:

```
ssh -L [local_addr:]local_port:remote_addr:remote_port [user@]sshd_addr
```

Not sure how legitimate the use of the term [_bastion host_](https://en.wikipedia.org/wiki/Bastion_host) is here, but that's how I visualize this scenario for myself:

![SSH Tunnels visualized - local port forwarding with a bastion host.](https://iximiuz.com/ssh-tunnels/local-port-forwarding-bastion-2000-opt.png)

I often use the above trick to call endpoints that are accessible from the _bastion host_ but not from my laptop (e.g., using an EC2 instance with private and public interfaces to connect to an OpenSearch cluster deployed fully within a VPC).

Lab 2: Local Port Forwarding with a Bastion Host 👨🔬

Again, the lab reproduces the setup from the diagram above. First, we need to prepare the bastion host - a machine with only the SSH daemon on it:

```
$ docker buildx build -t bastion:latest -<<'EOD'
# syntax=docker/dockerfile:1
FROM alpine:3

# Install the dependencies:
RUN apk add --no-cache openssh-server
RUN mkdir /root/.ssh && chmod 0700 /root/.ssh && ssh-keygen -A

# Prepare the entrypoint that starts the SSH daemon:
COPY --chmod=755 <<'EOF' /entrypoint.sh
#!/bin/sh
set -euo pipefail

for file in /tmp/ssh/*.pub; do
  cat ${file} >> /root/.ssh/authorized_keys
done
chmod 600 /root/.ssh/authorized_keys

# Minimal config for the SSH server:
sed -i '/AllowTcpForwarding/d' /etc/ssh/sshd_config
sed -i '/PermitOpen/d' /etc/ssh/sshd_config
/usr/sbin/sshd -e -D &

sleep infinity
EOF

# Run it:
CMD ["/entrypoint.sh"]
EOD
```

Starting the bastion host and noting its IP:

```
$ docker run -d --rm \
    -v $HOME/.ssh:/tmp/ssh \
    --name bastion \
    bastion:latest

BASTION_IP=$(
  docker inspect \
    -f '{{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}}' \
  bastion
)
```

Now, starting the target web service on a separate "machine":

```
$ docker run -d --rm \
    --name server \
    python:3-alpine \
    python3 -m http.server 80

SERVER_IP=$(
  docker inspect \
    -f '{{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}}' \
  server
)
```

Let's pretend that calling `curl ${SERVER_IP}` directly from the host is not an option for some reason (e.g., as if there were no route from the host to that IP address). So, we need to start the port-forwarding:

```
$ ssh -o StrictHostKeyChecking=no -f -N -L 8080:${SERVER_IP}:80 root@${BASTION_IP}
```

**Notice that the SERVER\_IP and the BASTION\_IP variables have different values in the above command.**

Checking that it works:

```
$ curl localhost:8080
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN" "http://www.w3.org/TR/html4/strict.dtd">
<html>
<head>
...
```

## Remote Port Forwarding

Another popular (but rather inverse) scenario is when you want to momentarily expose a local service to the outside world. Of course, for that, you'll need a _public-facing ingress gateway server_. But fear not! Any public-facing server with an SSH daemon on it can be used as such a gateway:

```
ssh -R [remote_addr:]remote_port:local_addr:local_port [user@]gateway_addr
```

The above command looks no more complicated than its `ssh -L` counterpart. But there is a pitfall...

**By default, the above SSH tunnel will allow using only the gateway's localhost as the remote address.** In other words, your local port will become accessible only from the inside of the gateway server itself, and highly likely it's not something you actually need. For instance, I typically want to use the gateway's public address as the remote address to expose my local services to the public Internet. For that, the SSH server needs to be configured with the [`GatewayPorts yes`](https://linux.die.net/man/5/sshd_config#GatewayPorts) setting.

Here is what remote port forwarding can be used for:

-   Exposing a dev service from your laptop to the public Internet for a demo.
-   Hmm... I can think of a few esoteric examples, but I doubt it's worth sharing them here. Curious to hear what other people may use remote port forwarding for!

Here is how the remote port forwarding looks on a diagram:

![SSH Tunnels visualized - remote port forwarding.](https://iximiuz.com/ssh-tunnels/remote-port-forwarding-2000-opt.png)

Lab 3: Using SSH Tunnels for Remote Port Forwarding 👨🔬

The lab reproduces the setup from the diagram above. First, we need to prepare a "dev machine" - a computer with an SSH client and a local web server:

```
$ docker buildx build -t devel:latest -<<'EOD'
# syntax=docker/dockerfile:1
FROM alpine:3

# Install dependencies:
RUN apk add --no-cache openssh-client curl python3
RUN mkdir /root/.ssh && chmod 0700 /root/.ssh

# Prepare the entrypoint that starts the web service:
COPY --chmod=755 <<'EOF' /entrypoint.sh
#!/bin/sh
set -euo pipefail

cp /tmp/ssh/* /root/.ssh
chmod 600 /root/.ssh/*

python3 -m http.server --bind 127.0.0.1 ${PORT} &

sleep infinity
EOF

# Run it:
CMD ["/entrypoint.sh"]
EOD
```

Starting the dev machine:

```
$ docker run -d --rm \
    -e PORT=80 \
    -v $HOME/.ssh:/tmp/ssh \
    --name devel \
    devel:latest
```

Preparing an ingress gateway server - a simple SSH server with `GatewayPorts` set to `yes` in `sshd_config`:

```
$ docker buildx build -t gateway:latest -<<'EOD'
# syntax=docker/dockerfile:1
FROM alpine:3

# Install the dependencies:
RUN apk add --no-cache openssh-server
RUN mkdir /root/.ssh && chmod 0700 /root/.ssh && ssh-keygen -A

# Prepare the entrypoint that starts the SSH server:
COPY --chmod=755 <<'EOF' /entrypoint.sh
#!/bin/sh
set -euo pipefail

for file in /tmp/ssh/*.pub; do
  cat ${file} >> /root/.ssh/authorized_keys
done
chmod 600 /root/.ssh/authorized_keys

sed -i '/AllowTcpForwarding/d' /etc/ssh/sshd_config
sed -i '/PermitOpen/d' /etc/ssh/sshd_config
sed -i '/GatewayPorts/d' /etc/ssh/sshd_config
echo 'GatewayPorts yes' >> /etc/ssh/sshd_config

/usr/sbin/sshd -e -D &

sleep infinity
EOF

# Run it:
CMD ["/entrypoint.sh"]
EOD
```

Starting the gateway server and noting its IP address:

```
$ docker run -d --rm \
    -v $HOME/.ssh:/tmp/ssh \
    --name gateway \
    gateway:latest

GATEWAY_IP=$(
  docker inspect \
    -f '{{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}}' \
  gateway
)
```

Now **from the inside of the dev machine, start the remote port forwarding**:

```
$ docker exec -it -e GATEWAY_IP=${GATEWAY_IP} devel sh
/ $# ssh -o StrictHostKeyChecking=no -f -N -R 0.0.0.0:8080:localhost:80 root@${GATEWAY_IP}
/ $# exit  # or detach with ctrl-p, ctrl-q
```

And validate that the dev machine's local port became exposed on the gateway's public interface (from the host system):

```
$ curl ${GATEWAY_IP}:8080
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN" "http://www.w3.org/TR/html4/strict.dtd">
<html>
<head>
...
```

## Remote Port Forwarding from a Home/Private Network

Much like local port forwarding, remote port forwarding has its own _bastion host_ mode. But this time, the machine with the SSH client (e.g., your dev laptop) plays the role of the bastion. In particular, it allows exposing ports from a home (or a private) network your laptop has the access to to the outside world through the ingress gateway:

```
ssh -R [remote_addr:]remote_port:local_addr:local_port [user@]gateway_addr
```

Looks almost identical to the simple remote SSH tunnel, but the `local_addr:local_port` pair becomes the address of a device in the home network. Here is how it can be depicted on a diagram:

![SSH Tunnels visualized - remote port forwarding to home network.](https://iximiuz.com/ssh-tunnels/remote-port-forwarding-home-network-2000-opt.png)

Since I typically use my laptop as a thin client and the actual development happens on a home server, I rely on such a remote port forwarding when I need to expose a dev service from a home server to the public Internet, and the only machine with the gateway access is my thin laptop.

Lab 4: Remote Port Forwarding from a Home/Private Network 👨🔬

As always, the lab reproduces the setup from the diagram above. First, we need to prepare the "thin dev machine":

```
$ docker buildx build -t devel:latest -<<'EOD'
# syntax=docker/dockerfile:1
FROM alpine:3

# Install the dependencies:
RUN apk add --no-cache openssh-client
RUN mkdir /root/.ssh && chmod 0700 /root/.ssh

# This time we run nothing (at first):
COPY --chmod=755 <<'EOF' /entrypoint.sh
#!/bin/sh
set -euo pipefail

cp /tmp/ssh/* /root/.ssh
chmod 600 /root/.ssh/*

sleep infinity
EOF

# Run it:
CMD ["/entrypoint.sh"]
EOD
```

Starting the "dev machine":

```
$ docker run -d --rm \
    -v $HOME/.ssh:/tmp/ssh \
    --name devel \
    devel:latest
```

Starting a private dev server using a separate "machine" and noting its IP address:

```
$ docker run -d --rm \
    --name server \
    python:3-alpine \
    python3 -m http.server 80

SERVER_IP=$(
  docker inspect -f '{{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}}' \
  server
)
```

Preparing the ingress gateway server:

```
$ docker buildx build -t gateway:latest -<<'EOD'
# syntax=docker/dockerfile:1
FROM alpine:3

# Install the dependencies:
RUN apk add --no-cache openssh-server
RUN mkdir /root/.ssh && chmod 0700 /root/.ssh && ssh-keygen -A

# Prepare the entrypoint that starts the SSH daemon:
COPY --chmod=755 <<'EOF' /entrypoint.sh
#!/bin/sh
set -euo pipefail

for file in /tmp/ssh/*.pub; do
  cat ${file} >> /root/.ssh/authorized_keys
done
chmod 600 /root/.ssh/authorized_keys

sed -i '/AllowTcpForwarding/d' /etc/ssh/sshd_config
sed -i '/PermitOpen/d' /etc/ssh/sshd_config
sed -i '/GatewayPorts/d' /etc/ssh/sshd_config
echo 'GatewayPorts yes' >> /etc/ssh/sshd_config

/usr/sbin/sshd -e -D &

sleep infinity
EOF

# Run it:
CMD ["/entrypoint.sh"]
EOD
```

And starting it:

```
$ docker run -d --rm \
    -v $HOME/.ssh:/tmp/ssh \
    --name gateway \
    gateway:latest

GATEWAY_IP=$(
  docker inspect \
    -f '{{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}}' \
  gateway
)
```

Now, **from the inside of the "dev machine", start the remote port forwarding SERVER-GATEWAY**:

```
$ docker exec -it -e GATEWAY_IP=${GATEWAY_IP} -e SERVER_IP=${SERVER_IP} devel sh
/ $# ssh -o StrictHostKeyChecking=no -f -N -R 0.0.0.0:8080:${SERVER_IP}:80 root@${GATEWAY_IP}
/ $# exit  # or detach with ctrl-p, ctrl-q
```

Finally, validate that the dev server became accessible on the gateway's public interface (from the host system):

```
$ curl ${GATEWAY_IP}:8080
<!DOCTYPE HTML>
<html lang="en">
<head>
...
```

## Summarizing

After doing all these labs and drawings, I noticed that:

-   The word **"local"** can mean either the **SSH client machine** or an upstream host accessible from this machine.
-   The word **"remote"** can mean either the **SSH server machine (sshd)** of an upstream host accessible from it.
-   Local port forwarding (`ssh -L`) implies it's the `ssh` client that starts listening on a new port.
-   Remote port forwarding (`ssh -R`) implies it's the `sshd` server that starts listening on an extra port.
-   The mnemonics are "ssh **\-L** **l**ocal:remote" and "ssh **\-R** **r**emote:local" and it's always the left-hand side that opens a new port.

## Instead of conclusion

Hope the above materials helped you a bit with becoming a master of SSH Tunnels 🧙 If you find the networking topics interesting but feel like most of the materials are written for bearded network engineers and almost indigestible for mere developers, I've got a few more articles that you may find useful:

-   [Computer Networking Introduction: Ethernet and IP](https://iximiuz.com/en/posts/computer-networking-101/)
-   [Illustrated introduction to Linux iptables](https://iximiuz.com/en/posts/laymans-iptables-101/)
-   [Bridge vs. Switch: What I Learned From a Data Center Tour](https://iximiuz.com/en/posts/bridge-vs-switch/)
-   [Container Networking Is Simple (not really)](https://iximiuz.com/en/posts/container-networking-is-simple/)

Happy reading!

### Resources

-   [SSH Tunneling Explained](https://goteleport.com/blog/ssh-tunneling-explained/) by networking gurus from Teleport.
-   [SSH Tunneling: Examples, Command, Server Config](https://www.ssh.com/academy/ssh/tunneling-example) by SSH Academy.