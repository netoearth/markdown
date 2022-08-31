Although it’s possible to deploy [Docker](https://www.docker.io/) containers without root privileges, that doesn’t necessarily mean it’s rootless throughout. That’s is because there are other components within the stack (such as runc, containerd, and dockerd) that do require root privileges to run. That can equate to a security issue by way of heightened privilege attacks.

Sure, you can add your user to the docker group and run the docker deploy command without the help of `sudo`, but that really doesn’t solve the problem. There are other ways to run docker that seem like a good idea but, in the end, they’re just as dangerous as running docker with `sudo` privileges.

So, what do you do? You can always go rootless.

## How Rootless Works

Effectively, running rootless Docker takes advantage of user namespaces. This subsystem provides both privilege isolation and user identification segregation across processes. This feature has been available to the Linux kernel since version 3.8 and can be used with docker to map a range of user IDs so the root user within the innermost namespace maps to an unprivileged range in a parent namespace.

Docker has been able to take advantage of the user namespace feature for some time. This is done using the `--userns-remap` option. The only problem with this is the runtime engine is still run as root, so it doesn’t solve our problem.

That’s where rootless docker comes into play.

## Limitations

Unfortunately, rootless mode isn’t perfect. The first issue is that rootless docker will not have access to privileged ports, which are any port below 1024. That means you’ll need to remember to expose your containers to ports above 1024, otherwise, they will fail to run.

Another issue is that limiting resources with options such as –cpus, –memory, and –pids-limit are only supported when running with cgroup v2 and systemd.

Other limitations you might run into include:

-   No support for AppArmor, checkpoint, overlay network, and SCTP port exposure.
-   Limited storage driver support (only the overlay2, fuse-overlayfs, and vfs storage drivers are supported).
-   Doesn’t support –net-host.

With all of that said, how do we install docker such that it can be run in rootless mode? It’s actually quite simple. Let me show you how.

I’ll be demonstrating on my go-to server of choice, Ubuntu Server 20.04, but you can do this on nearly any Linux distribution. The only difference will be the installation command to be run for the one dependency.

## Installing the Lone Dependency

The first thing we must do is install the sole dependency for this setup. That dependency is uidmap, which handles the user namespace mapping for the system. To install uidmap, log into your server and issue the command:

`sudo apt-get install uidmap -y`

That’s all there is for the dependencies.

## Installing Docker

Next, we install Docker. We don’t want to go with the version found in the standard repository, as that won’t successfully run in rootless mode. Instead, we need to download a special installation script that will install rootless Docker. We can download and install the rootless version of docker with a single command:

`curl -fsSL https://get.docker.com/rootless | sh`

When that installation finishes, you then need to add a pair of environment variables to .bashrc. Open the file with:

`nano ~/.bashrc`

In that file, add the following lines to the bottom:

<table><tbody><tr><td data-settings="show"></td><td><div><p><span>export </span><span>PATH</span>=/<span>home</span>/<span>jack</span>/<span>bin</span>:$<span>PATH</span></p><p><span>export </span><span>DOCKER_HOST</span>=<span>unix</span>:<span>///run/user/1000/docker.sock</span></p></div></td></tr></tbody></table>

NOTE: Make sure to add your particular user ID. In the above code, my ID was 1000. To find your user ID, issue the command:

`id`

You’ll want to add the number after _uid=_ in the line:

<table><tbody><tr><td data-settings="show"></td><td><div><p><span>export </span><span>DOCKER_HOST</span>=<span>unix</span>:<span>///run/user/ID/docker.sock</span></p></div></td></tr></tbody></table>

Where _ID_ is your user ID number.

Save and close the file.

Log out and log back into the server (so the changes will take effect) and you’re ready to test out rootless docker.

## Testing Rootless Docker

We’ll deploy our trusty NGINX container as a test. Remember, we’ve not added our user to the docker group. If this were a standard Docker installation, we wouldn’t be able to successfully deploy the NGINX container without either adding our user to the docker group or running the deploy command with `sudo` privileges.

To test rootless mode (deploying NGINX in detached mode), issue the command:

`docker run --name docker-nginx -p 8080:80 -d nginx`

Open a web browser and point it to http://SERVER:8080 (Where SERVER is the IP address of your Docker server) and you should see the NGINX welcome page.

This container was deployed without using root, so the entire stack is without those elevated privileges. You can even deploy a full Linux container and access it’s bash shell with a command like:

`docker run -it ubuntu bash`

All of this done without touching root privileges.

## Conclusion

This is obviously not a perfect solution to solve all of the security issues surrounding Docker containers. And you might even find [Podman](https://thenewstack.io/deploy-a-pod-on-centos-with-podman/) a better solution, as it can run rootless out of the box. But for those who are already invested in Docker, but are looking to gain as much security as possible, running Docker in rootless mode is certainly a viable option.

Give rootless Docker a try and see if it doesn’t ease your security headaches a bit.

The New Stack is a wholly owned subsidiary of Insight Partners, an investor in the following companies mentioned in this article: Docker.

Image by ykaiavu from Pixabay.