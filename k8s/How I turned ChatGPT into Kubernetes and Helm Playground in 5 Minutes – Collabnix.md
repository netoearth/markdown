24th January 2023 48 sec read

![](https://collabnix.com/wp-content/uploads/2023/01/Screenshot-2023-01-24-at-3.53.10-PM.png)

As a highly advanced language model, ChatGPT can be fine-tuned on specific tasks, or on specific data, to improve its accuracy and performance even more. It can be fine-tuned on a dataset of Linux commands and corresponding outputs, or on a dataset of Helm commands and corresponding outputs, to simulate a Linux terminal or Helm playground respectively.

Here’s how I turned ChatGPT into a Kubernetes and Helm Playground.

_**Ajeet: Hello GPT. Good Morning. I want you to act as a Mac terminal. Kubernetes is already installed. I will type some commands and you’ll reply with what the terminal should show. Please show the result using the Mac terminal. When I tell you something, I will do so by putting text inside curly brackets {like this}. Please don’t write an explanation just the command result is sufficient. My first command is the kubectl version.**_

[![Image1](https://res.cloudinary.com/practicaldev/image/fetch/s--fe_9KtaU--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/a7utv8dx86ogv78zuap5.png)](https://res.cloudinary.com/practicaldev/image/fetch/s--fe_9KtaU--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/a7utv8dx86ogv78zuap5.png)

**_Ajeet: {kubectl get po,deploy,svc}_**

[![Image2](https://res.cloudinary.com/practicaldev/image/fetch/s--JQCL4z0c--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/n4pidekqh2uqq4b8f7vu.png)](https://res.cloudinary.com/practicaldev/image/fetch/s--JQCL4z0c--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/n4pidekqh2uqq4b8f7vu.png)

_**Ajeet: {kubectl run –image=nginx nginx-app –port=80 –env=”DOMAIN=cluster”}**_

[![Image3](https://res.cloudinary.com/practicaldev/image/fetch/s--DBy_gMy---/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/qwyiph1fv9rj1oxm92ol.png)](https://res.cloudinary.com/practicaldev/image/fetch/s--DBy_gMy---/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/qwyiph1fv9rj1oxm92ol.png)

_**Ajeet:{kubectl expose deployment nginx-app –port=80 –name=nginx-http}**_

[![Image4](https://res.cloudinary.com/practicaldev/image/fetch/s--7r6Dr-tp--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/hhfly8kpo3e7t89fahk4.png)](https://res.cloudinary.com/practicaldev/image/fetch/s--7r6Dr-tp--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/hhfly8kpo3e7t89fahk4.png)

_**Ajeet: {kubectl get po,svc,deploy}**_

[![Image5](https://res.cloudinary.com/practicaldev/image/fetch/s--7Fuanu_7--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/zcv7o8svv5zifz0eyge4.png)](https://res.cloudinary.com/practicaldev/image/fetch/s--7Fuanu_7--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/zcv7o8svv5zifz0eyge4.png)

_**Ajeet: {curl 10.100.67.94:80}**_

[![Image6](https://res.cloudinary.com/practicaldev/image/fetch/s--yVzZ-n3l--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/av19gif4sh2ihj8cr37x.png)](https://res.cloudinary.com/practicaldev/image/fetch/s--yVzZ-n3l--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/av19gif4sh2ihj8cr37x.png)

_**Ajeet: {helm repo add bitnami [https://charts.bitnami.com/bitnami](https://charts.bitnami.com/bitnami)}**_

[![Image7](https://res.cloudinary.com/practicaldev/image/fetch/s--KsUNEV5S--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/922ewg0dh11w0hrdtgqq.png)](https://res.cloudinary.com/practicaldev/image/fetch/s--KsUNEV5S--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/922ewg0dh11w0hrdtgqq.png)

_**Ajeet: {helm repo update}**_

[![Image8](https://res.cloudinary.com/practicaldev/image/fetch/s--Zr-ltMar--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/p9fwb7hbckslhhttab3x.png)](https://res.cloudinary.com/practicaldev/image/fetch/s--Zr-ltMar--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/p9fwb7hbckslhhttab3x.png)

_**Ajeet: {helm install bitnami/mysql –generate-name}**_

[![Image9](https://res.cloudinary.com/practicaldev/image/fetch/s--bbBKb2xq--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/01t18mrl2wihi6d364pd.png)](https://res.cloudinary.com/practicaldev/image/fetch/s--bbBKb2xq--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/01t18mrl2wihi6d364pd.png)

_**Ajeet: {helm show chart bitnami/mysql}**_

[![Image9](https://res.cloudinary.com/practicaldev/image/fetch/s--klBjk_IR--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/96vjw06imk5y3qkprvxv.png)](https://res.cloudinary.com/practicaldev/image/fetch/s--klBjk_IR--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/96vjw06imk5y3qkprvxv.png)

## Additional Resources:

-   [Turn ChatGPT into a Docker Playground in 5 Minutes](https://collabnix.com/turn-chatgpt-into-a-docker-playground-in-5-minutes/)
    
-   [Get Started with ChatGPT](https://collabnix.com/getting-started-with-chatgpt/)
    
-   [Using ChatGPT to Build an Optimised Docker Image using Docker Multi-Stage Build](https://collabnix.com/when-chatgpt-meet-docker-for-the-first-time/)
    

Please follow and like us:

Views 127

Have Queries? Join https://launchpass.com/collabnix