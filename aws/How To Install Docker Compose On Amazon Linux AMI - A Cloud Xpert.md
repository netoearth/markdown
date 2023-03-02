[Docker](https://acloudxpert.com/docker/)[How To](https://acloudxpert.com/docker/how-to-docker/)

[By **admin**](https://acloudxpert.com/author/admin/ "Browse Author Articles") Last updated **Jan 3, 2019**

1.  Create EC2 with Amazon Linux AMI
2.  Install Docker over it, [click here](https://acloudxpert.com/how-toinstall-docker-on-amazon-linux-ami/) for help
3.  Now to install docker-compose, run
    
    ```
    sudo curl -L https://github.com/docker/compose/releases/download/1.21.0/docker-compose-`uname -s`-`uname -m` | sudo tee /usr/local/bin/docker-compose > /dev/null
    ```
    
4.  For permission
    
    ```
    sudo chmod +x /usr/local/bin/docker-compose
    ```
    
5.  Create a symbolic link
    
    ```
    ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
    ```
    
6.  Check docker-compose version:
    
    ```
    docker-compose --version
    ```
    

[](https://acloudxpert.com/author/admin/ "Browse Author Articles")

[admin](https://acloudxpert.com/author/admin/)

Prev Post

[How To Install Docker On Amazon Linux AMI](https://acloudxpert.com/how-toinstall-docker-on-amazon-linux-ami/)

Next Post

[Creating ToDo Application with Serverless Application Repository](https://acloudxpert.com/creating-todo-application-with-serverless-application-repository/)

[You might also like](https://acloudxpert.com/how-to-install-docker-compose-on-amazon-linux-ami/#relatedposts_479807231_1) [More from author](https://acloudxpert.com/how-to-install-docker-compose-on-amazon-linux-ami/#relatedposts_479807231_2)

[Docker](https://acloudxpert.com/docker/)

[Docker Swarm Lab Guide](https://acloudxpert.com/docker-swarm-lab-guide/ "Docker Swarm Lab Guide")

[Docker](https://acloudxpert.com/docker/)

[Running Grafana as a Container](https://acloudxpert.com/running-grafana-as-a-container/ "Running Grafana as a Container")

[Prometheus](https://acloudxpert.com/prometheus/)

[Running Prometheus Alert Manager as a Container](https://acloudxpert.com/running-prometheus-alert-manager-as-a-container/ "Running Prometheus Alert Manager as a Container")

[Docker](https://acloudxpert.com/docker/)

[Running Prometheus as a Container](https://acloudxpert.com/running-prometheus-as-a-container/ "Running Prometheus as a Container")

[Prev](https://acloudxpert.com/how-to-install-docker-compose-on-amazon-linux-ami/#prev-2016884107 "Previous") [Next](https://acloudxpert.com/how-to-install-docker-compose-on-amazon-linux-ami/#next-2016884107 "Next")

Leave A Reply

Your email address will not be published.

Save my name, email, and website in this browser for the next time I comment.