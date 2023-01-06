## Video

<iframe src="https://www.youtube.com/embed/T4lp6wtS--4" height="423" width="752" frameborder="0" allowfullscreen=""><a href="https://www.youtube.com/watch?v=T4lp6wtS--4" target="_blank">View YouTube video</a></iframe>

## About the Talk

You may have heard the buzz around Kubernetes and noticed that many companies have been rapidly adopting it. Due to its many components and vast ecosystem it can be quite confusing to find where the path starts to learn it.

In this session, you will learn the basics of containers and Kubernetes. Step by step, we will go through the entire process of packaging a Node.js application into a Docker container image and then deploying it on Kubernetes. We will demonstrate scaling to multiple replicas for better performance. The end result will be a resilient and scalable Node.js deployment.

You will leave this session with sufficient knowledge of containerization, Kubernetes basics, and the ability to deploy highly available, performant, and scalable Node.js applications on Kubernetes.

💰 Use this [free **$100 credit**](https://try.digitalocean.com/open-source/?utm_campaign=kamal_200521&utm_medium=webinar) to try out Kubernetes on DigitalOcean for free!

## About the Presenter

Kamal Nasser is a Developer Advocate at DigitalOcean. If not automating and playing with modern software and technologies, you’ll likely find him penning early 17th-century calligraphy. You can find Kamal [on Twitter at @kamaln7](https://twitter.com/kamaln7) or [on GitHub at @kamaln7](https://github.com/kamaln7).

## Resources

[View the slides for this talk](https://do.co/nodejs-k8s-talk-slides), or [watch the recording on YouTube](https://www.youtube.com/watch?v=T4lp6wtS--4).

### Transcript of The Commands and Manifests Used

Be sure to follow along with the recording for an explanation and replace `kamaln7` with your own [DockerHub](https://hub.docker.com/) username.

#### Node App

1.  Create an empty node package: `npm init -y`
    
2.  Install express as a dependency: `npm install express`
    
3.  index.js
    

#### Docker

1.  Dockerfile
    
2.  Build the image: `docker build -t kamaln7/node-hello-app .`
    
3.  Edit `index.js` and replace the word `Hi` with `Hello`.
    
4.  Re-build the image and notice Docker re-using previous layers: `docker build -t kamaln7/node-hello-app .`
    
5.  Run a container to test it: `docker run --rm -d -p 3000:3000 kamaln7/node-hello-app`
    
6.  Look at the running containers: `docker ps`
    
7.  Stop the container: `docker stop CONTAINER_ID`
    
8.  Push the image to DockerHub: `docker push kamaln7/node-hello-app`
    

#### Kubernetes

1.  Get worker nodes: `kubectl get nodes`
2.  Create a deployment: `kubectl create deployment --image kamaln7/node-hello-app node-app`
3.  Scale up to 3 replicas: `kubectl scale deployment node-app --replicas 3`
4.  Expose the deployment as a NodePort replica: `kubectl expose deployment node-app --type=NodePort --port 3000`
5.  Look at the newly created service (and the assigned port): `kubectl get services`
6.  Grab the public IP of one of the worker nodes: `kubectl get nodes -o wide`
7.  Browse to `IP:port` to test the service
8.  Edit the service: `kubectl edit service node-app`
    1.  Replace `port: 3000` with `port: 80`
    2.  Replace `type: NodePort` with `type: LoadBalancer`
9.  Verify that the service was updated: `kubectl get service`
10.  Run the above command every few seconds until you get the external IP address of the Load Balancer
11.  Browse to the IP of the Load Balancer

### Want to learn more? Join the DigitalOcean Community!

Join our DigitalOcean community of over a million developers for free! Get help and share knowledge in our Questions & Answers section, find tutorials and tools that will help you grow as a developer and scale your project or business, and subscribe to topics of interest.

[Sign up](https://www.digitalocean.com/api/dynamic-content/v1/login?success_redirect=https%3A%2F%2Fwww.digitalocean.com%2Fcommunity%2Ftech_talks%2Fhow-to-deploy-a-resilient-node-js-application-on-kubernetes-from-scratch&error_redirect=https%3A%2F%2Fwww.digitalocean.com%2Fauth_error)