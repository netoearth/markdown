The procedure to install Docker on AMI 2 (Amazon Linux 2)

1.  Login into the remote AWS server using the ssh command or connect EC2 Instance Connect (browser-based SSH connection.
2.  Apply pending updates using the [yum command](https://www.cyberciti.biz/faq/rhel-centos-fedora-linux-yum-command-howto/?utm_source=Linux_Unix_Command&utm_medium=faq&utm_campaign=nixcmd):

`$ sudo yum update`

3\. Install docker, run:

`$ sudo yum install docker`

![](https://miro.medium.com/max/700/1*iZwaFYN7h5rFWQGYv15neQ.png)

Amazon Linux 2: Install the docker command

4\. let’s check the version and info of the docker

![](https://miro.medium.com/max/700/1*mbDvoKWNmPs09y4ije1JSA.png)

This tells us that, it’s not started let’s start the docker service and make it enable during boot time as well.

`$ sudo service docker start ; $ sudo systemctl enable docker.service`

![](https://miro.medium.com/max/700/1*dXCoMaZFW8ASc-WJ863W5A.png)

## Verification

Now that the required software is installed, we need to ensure it is working. Hence, type the following commands.  
`$ sudo systemctl status docker.service`

![](https://miro.medium.com/max/700/1*SK3NATLQ0QDg6dHhjtAGzw.png)

## To control the docker service

Use these command

```
sudo systemctl start docker.service #<-- start the servicesudo systemctl stop docker.service #<-- stop the servicesudo systemctl restart docker.service #<-- restart the servicesudo systemctl status docker.service #<-- get the service status
```

## Getting docker info on Amazon Linux

![](https://miro.medium.com/max/700/1*YDKvBHphmOPgPz-zAj6QhQ.png)

That is all. You learned how to install Docker on AMI 2.