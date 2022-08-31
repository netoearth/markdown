![](https://miro.medium.com/max/1400/0*pqpvJS87OlFIUf2W)

Photo by [Todd Cravens](https://unsplash.com/@toddcravens?utm_source=medium&utm_medium=referral) on [Unsplash](https://unsplash.com/?utm_source=medium&utm_medium=referral)

It’s important for developers to have a development environment that is easy to use, and Docker is really a good tool to simplify many things, from having databases and redis containers up and running with same version for everyone, to having application servers running with proper dependencies, and ability to trash and spawn again everything when the state of the environment is becoming dirty.

Docker Desktop, however, is a VM containing Docker for Mac or Windows workstations. And as such, comes with many drawbacks:

-   The FileSystem bindings are a terrible performance culprit. Even with playing with mount options such as cached, or using the new gRPC-fuse file system, the performances of server reload or running a tool that requires scanning the repository, such as Pytest, are terribly slower than one would expect. (I was already mentioning this issue in a p[revious article about speeding up our tests](https://medium.com/partoo/speeding-up-tests-with-pytest-and-postgresql-a308b28228fe))
-   Despite the resource constraints configured for the VM, somehow the VM gets to generously overflow it and block a lot of RAM (e.g. I configured it at 6GB which is enough for our application, and found Docker Desktop feeding off of 15GB out of my 16GB)
-   There is no documented way to get inside the VM, which means there's no possibility to store some files locally except for named volumes to be mounted across several volumes

## Vagrant + VirtualBox + Ansible = 💖

When onboarding a new developer, the README would contain around 20 steps, but thanks to automation provided by our beloved DevOps tools, we could shrink it down to 8 steps.

VirtualBox is the new VM manager that we use to replace Docker Desktop. Vagrant is a nice VM as Code tool that will create the VM in VirtualBox and interact with it. And it will provision the inside of the VM by calling Ansible.

## Getting your VM started

First, let's [init](https://www.vagrantup.com/docs/cli/init) a Vagrantfile that will be used by Vagrant to bootstrap:

```
vagrant init generic/ubuntu2004
```

Then, in your Vagrantfile, you can configure how you want your VM:

And now time for provisioning with Ansible with common things. Although I cannot share the entire playbook, I've selected useful pieces that might give away some fancy tips for a good development experience 😉

## Editing your code inside the VM

We've identified 2 main communities inside Partoo for code editors, Visual Studio Code on one hand, JetBrains' PyCharm on the other end (and VIM users were granted to know how it'd work anyway 😝). So before kicking off the project, we made sure that remote edition was something available for both communities.  
For reference, here's the link to set-up remote development [on PyCharm](https://www.jetbrains.com/help/pycharm/creating-a-remote-server-configuration.html) & [on Visual Studio Code](https://code.visualstudio.com/docs/remote/ssh)

However, my team is only working with Visual Studio Code and I must admit that we didn't look enough into PyCharm's configuration and user experience, and that resulted into a biased experience where Visual Studio Code ends up in a much shinier position, I myself had made the transition specifically because of the remote development experience that was provided by Visual Studio Code, both with SSH and Containers.

## Some performance measures

Some of the big activities such as running our unit tests or running yarn serve for our front-end resources went faster by around 30% in average, so that's definitely a win given our original objective.

However, a downside is that VM still steams to consume RAM heavily, up to 13GB, so even VirtualBox isn't behaving nicely with just the resources originally allocated. For those extremely curious about it, there is a [forum thread on virtualbox](https://www.virtualbox.org/ticket/19726) discussing the issue.  
It might be just an issue of how Mac OS is reporting the memory usage of a Virtual Machine, maybe it invalidates the original hypothesis about Docker Desktop.

## Right about time

Just as if we had forecast it, we started working and deploying our new Development Environment, just before Docker announced its new policy to have Docker Desktop chargeable for companies like [Partoo](https://www.partoo.co/). And it felt good!

## Conclusion

Starting from a simple goal of improving the performances of our high CPU consuming tasks, we've reached several achievements: getting better performances, unifying the developers' environments (while still letting room for choice), speeding up the onboarding of new developers, and avoiding the vendor lock-in that would have increased our expenses .