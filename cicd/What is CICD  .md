## **What is CI/CD?**  

Nowadays, the concept of the CI/CD is everywhere. CI stands for continuous integration and the CD stands for Continuous Deployment. CI/CD is the backbone of the DevOps environment. It helps organisations to speed up the release of small features in new software, and implement the feedback and changes quickly.  

**Gogs** is a free and open source Git service written in Go language. painless self hosted Git service. You can install and configure Gogs on your own servers. It supports all platforms including Linux, Mac and Windows. For more about Gogs, check their official document [https://gogs.io/docs](https://gogs.io/docs)  

**Jenkins** is a self-contained open source automation platform which can be used to automate all kinds of software development, testing, and distribution, or deployment tasks.  

In this tutorial, I will show you how to create a simple CI/CD Pipeline using Jenkins and Gogs In this scenario, we have hosted a [django](https://www.sparksupport.com/hire-python-developers-india) application on the linux server. As of now we are doing the following process if any changes needed for the application. Developer pulls the live code from the Gogs repository and pushes back to the repo after modifying the code. Then the SystemAdministrator will pull the latest code from the Gogs repo to the application server.  

We can automate the above processes with the help of **CI/CD.  
**  
**1**. Developer push/merge code into a branch in the repo.

**2**. Admin get a notification message in slack about the push

**3**. Gogs trigger the Jenkins job for the new build and initiate the build if the conditions satisfy that specified in the Jenkins.

**4**. Admin get a notification message after the build.

**5**. Once the build is completed, the jenkins will trigger the code deployment into the application server.

**6**. Finally the application server will pull the latest code from the Gogs repo and send a notificatification message to slack. 

![CI/CD Pipeline using Jenkins and Gogs img 1](https://lh6.googleusercontent.com/0isk4IT7U2rI10hyJQxD8dWNGVLyGAWHIlhEAdA85UHAs1Cwb0Sc7QdjU50CwxwM0xZNcLLogcL-HzobcECiKmqL8I84TRXrCtD5cCY3Va7OkhfpY9dVipD8w7KDG2_48tHNPmAN)

You may get a clear picture about this CI/CD Pipeline using Jenkins and Gogs from the above diagram.

### **Requirements**:  

SSH access to Jenkins server.  
Gogs admin access.  
SSH access to the application server.

### **1\. Install plugins on Jenkins.**  

Required plugins:  
Git  
Gogs  
Login to your Jenkins web interface.  
Go to Plugins Manage

![CI/CD Pipeline img2](https://lh5.googleusercontent.com/DGXOIspaOhrpZYPF45U3rpjrj_ZWiU31KbDENEOv2Iazwvh5Hp0e_j4Rds9QwR-ZaROI8EgO0AyM1QOSF6B-C1Koy7OY7NuYhZBoUGooOWbkGnxoci7k-ug_sW0Q7nt66ObeGLvR)

  
Search required plugins from there and click on “ Download now and install after restart”

### **2\. Configure webhook**  

  
We need a hook to trigger from Gogs to Jenkins job. I’m using a post-receive hook in the repository to poke Jenkins jobs when a new push occurs. In Gogs repo there is a Git Hook option to configure post-receive hook. This hook will help you to minimize the delay between a push and a build. CI/CD Pipeline using Jenkins and Gogs will do this, go to your Gogs repo >> Settings >> Git Hooks >> Post Receive Then add the following line in your hooks. curl http://yourjenkinsserver/git/notifyCommit?url=**<URL of the Gogs repo>**  

![ Jenkins and Gogsimg3](https://lh4.googleusercontent.com/7LBG2ltTDw-VRIXn_PleF7ZQNdSYZu2eV6n1tECYQydS0dXFmmkoZnAJT7iuzgUC4kRwYz2N6xhKFcA49-_KaJ_92BV8zpOM94JBp9V516d38TR21RoxRdiXQLrs8gWTNntminvy)

![using Jenkins and Gogsimg4](https://lh4.googleusercontent.com/7SOYuInoFC5dSFA3LPmfyUS1sbkABYaZ0Fyb0so3mOpQydhuLaFf1gXmm1QQ58vMZr9bsVUAJ1ISuopaeIEIzZawVeKEH04oxhWu4GuEnydUR9_eacM6D7CNMBuH0xXWzW6w7G-Z)

**3\. Configure access to Gogs from Jenkins.**

**SSH**

Login to your jenkins server through ssh and generate a ssh key.

\# ssh-keygen

The ssh private and public key will be generated in /var/jenkins\_home/.ssh/ Copy that public key from there.

\# cat /var/jenkins\_home/.ssh/id\_rsa.pub

Login to your Gogs and go to your Repository Settings > Deploy keys Add previously copied public key here

![ using Jenkins and mg 5](https://lh4.googleusercontent.com/5oTR-fuIvvnRj5siVO286V4WvZlmJBeUIIFmK6c8sSnOoORJmFQZ-tCALL1jDv0z02zQAFYtj14QmjPtmptA_QCfpr0uZHJkVZ7AUVyS2Ds_hmKG252INJsSrKJaunVm8u40xEfc)

Add this private key in Jenkins credentials.

Login to your jenkins server and go to:

Jenkins >> Credentials >> System >> Global credentials

Then select “SSH username with private key” from the Drop down menu. Enter Gogs ssh username and paste previously created private key there.

![Jenkins and Gogs img6](https://lh4.googleusercontent.com/Vsyh2JjSxJXyq03jjbpZqxwEDDMN6-Uzt3ZjxJF0Iztc36_0uSLtAxwJ4ezvSqm3HzJgd0-M0c4CdeRxVfLROR59ehu1bQcpwTBJtci84dGpvB4Xw9P_Wr_zpyWe9MJscnm4kqfo)

**Username and password  
**  
You can also configure this with your Gogs username and password.

Go to Jenkins >> Credentials >> System >> Global credentials

Enter your Gogs user username and password there.

![CI/CD Pipeline using](https://lh5.googleusercontent.com/QNyGOAVWQe9eXRn31ymoml_5-H6p-lqGJZ794sRq7FaSc0TtosRL0a0JG9QBpVYxCEt5rQyiDVLMRsfJkOj3i2G3oEZ8csgUZAyxaIeaOZb6jepfs_YDZrVidw66FPbTLbKkgHM_)

**4\. Configure SSH access to application server from jenkins.**

In this example I have a user called jenkins in the application server.

Login to your Jenkins server through SSH.

Copy previously created key to the application server

\# ssh-copy-id jenkins@192.168.1.35

Now configure this ssh details in jenkins.

Install “Publish over SSH” plugin

![Gogs img8](https://lh6.googleusercontent.com/Jn0ddk7x6Vu-MPg1xYCQa5x8w-WSUfi_3FFP-aQgbduxWU_f9XwnFv1hQK2UqKDppOLiZvTQX0He_5TawpXR-Ny7IuBk5na-TgTOCoRh7_83Pq13KHICKjDpXJo22-3IrZLrXciE)

  
Go to Jenkins > Manage Jenkins > Configure System > Publish Over SSH

![Jenkins and img9](https://lh4.googleusercontent.com/NeetobhJM5G-C63UPukCb46PeVW8nqSvbPr3fSWTpKSv0ZB8-aO32PJV0i8AUUt6mGiG0nsCx3dHxpqN5vkz4CDDb7hwaLxeHcfUnf-U7n1-TDlDbkpis429zXk1nHUv1O4S5138)

If you get any error as shown in the below screenshot, create ssh key with following options in the 3rd step.

\# ssh-keygen -t rsa -b 4096 -m PEM

  

![Pipeline using img10](https://lh5.googleusercontent.com/MyWG1MnnwAYhcHCOfbjNrpoKVib8qy7yCUxM5UJ4KlgFzt1P4V-ITsrLQDIBdKvKePiSHxAuB4sr9WoVWwNe5JZMIXLlg37O52SgKdE3HYPs4wSQIpRmFU--Et9BBZtoCY3YXeSx)

### **5**. **Configure Jenkins job**  

  
Login to your Jenkins dashboard.  
Go to Jenkins > New Item.  
Enter your project name and select “Free style project”

![ Jenkins and Gogs img11](https://lh4.googleusercontent.com/y6WxpgaBy5LG9RmsydCovfDolDp9RMIaM1U2ju6xJyUoQpLGBR48d3jRZZZH4nlMc8HIcctUq_3Y8ylUa0QR-ZHvYHQ8-y5Ea5zDXBuKPzZn_pPbqu_mmxElzAGS-IKSb_aO7bK1)

In this job, we are executing a bash script to deploy the code into the application server. So we need string parameters to pass into the bash script as a argument.

So select the option “This project is parameterised” under Gogs webhook

Then add the parameter “ GIT\_FOLDER”

Specify the directory name in the application server to which git pull happens for code deployment.

In my case the directory is /home/jenkins/[[cicd]]-dev

![ Pipeline   img12](https://lh4.googleusercontent.com/lsPv5OhueSrR4PfYggYJLIlF5WvIGMtnuU3V5P3zUnnGNGl6i4Ze6Bv6j0La2jrj0t7EN0NV4EFGcf3DCe5N_vZR6Rnv0aGSUadvPplm0y54GeLJ3GphSwM6oZeUCrxZJuQOuzKB)

Add next parameter is “GIT\_CHECKOUT\_DIR”

Specify the directory name in the application server to which will deploy the code.  

![CI/CD  img13](https://lh3.googleusercontent.com/yq8HlbCYx9CB0OBb8QBGbYPBeXTRz9bZVl6Tlv6NyN5yXNfTWXgZ43EaQMvB9AI02r60YJU8xfvKHEVRVm0-lYzHaihzHqkqIpXdGxcrD8o9Q4LY5Li5kDk3xGhni8QdAp3NsEJ0)

Next parameter is “SERVER\_ENV”  

![CI/ img14](https://lh5.googleusercontent.com/3-R_XG7l6MkX_74UcOU-YsFhm8hgpjQGoY9Z5TAqwuOCevKCi1Ca0GcSjmYhUl8U9uM6z6sie3AFmDxm2GsoNMwesLK7E7l4RPFDnNnTCkKaI4gcsnwzNeZ6Goe4KUvOs_OK5SSU)

Next parameter is “VENV”

Specify your python virtual environment path in the application server.  

![CI/CD Pipeline img15](https://lh3.googleusercontent.com/sBgSpbMsw9CT6ykhIAiXS5806CZgL988imrrrqZ2Z18Z3bfyAX9lxraqsj7gVxI7XMz6pUpRnJNUkZT0LZYYajxvrkqf5mU41HEzS_id-TFsGQe2LfjzrC25PnW0QikgL-L-qyTz)

Then add the parameter “DEPLOY\_SCRIPT”

This variable stores the path of the deploy script.

hsgfhsg

![CI/CD  using  and Gogs img16](https://lh4.googleusercontent.com/KJ6krPDhO4X82RRL9aii_F8OO-7TMF3YTV5U6iOtktzW2yDQMK7Ifotl9IGaY0NWOjkA-Hjzi1xz1JXORvMkLRScuNGmgg6FlcF9epvPOQEZYPJflCNjTfTtAQ0ixf_0du6oyAZO)

I will show you the content of the script at the end of the tutorial.

Now add the Gogs repo URL and credentials in the “Source Code Management” You can select previously added credentials from there.

![CI/CD Pipeline  Jenkins and  img17](https://lh4.googleusercontent.com/i2EboB6In9H44rPTCU9BCsOfr7L2fehXWTf_fXQ64cJMzK5E1UGVcspz87Bfgu9CFaMiQixscsRadDAex92pgB6dxfSZj7jrjYEuoxaAnA5PfKiVD7FYOXQnuSSbynhNMjXErGqg)

Finally go to the “Post-build Action” in the Jenkins job.

Select “Send build Artifacts over SSH” and paste the following command in the “Exec command box”

cd $GIT\_FOLDER && git fetch && if ! (git diff –exit-code –name-only $GIT\_BRANCH — $DEPLOY\_SCRIPT); then git pull;fi && bash $DEPLOY\_SCRIPT  $GIT\_FOLDER $GIT\_BRANCH $GIT\_CHECKOUT\_DIR $GIT\_URL $SERVER\_ENV $VENV

  

![CI/CD Pipeline Gogs img18](https://lh6.googleusercontent.com/vpPN3Zy9u18E7jGs4lFCEEvcpoAai1YBi8BdlD1w1bux6pTt88NfMbhFmK4JqlAeehrASRDdVTvu9WzG0edGdnVGp7BrWDpF8EkPHklySaMOVhDWBMD9g7e4PY1kXy3vVBT0PN5M)

If the conditions satisfy, jenkins will pull files from the gogs repo and execute the deploy script with the above mentioned arguments.  
Here is the deploy script written in the bash script.  

###Beginning##  
#!/bin/bash  
#Deployment script used by Jenkins  
#Author: Hans Emmanuel  
GIT\_FOLDER=$1  
GIT\_BRANCH=$2  
GIT\_CHECKOUT\_DIR=$3  
GIT\_URL=$4  
SERVER\_ENV=$5  
VENV=$6  
PIP\_INS=false  
DEPLOYED=false  
SLACK\_URL=  <specify your slak url if you are using slack>

cd $GIT\_FOLDER  
git fetch  
if (git cat-file -e $GIT\_BRANCH:DEPLOY)  
then  
echo “deploying the code”  
BRANCH=$(echo ${GIT\_BRANCH} | cut -d’/’ -f2)  
 if ! (git diff –exit-code –name-only $GIT\_BRANCH — requirements.txt)

then  
         PIP\_INS=true

fi  
git pull && GIT\_WORK\_TREE=$GIT\_CHECKOUT\_DIR git checkout $BRANCH  -f && DEPLOYED=true && echo “CODE Deployed”

 cd $GIT\_CHECKOUT\_DIR  
     if $PIP\_INS  
then  
cd $GIT\_CHECKOUT\_DIR  
echo “Installing requirements”  
source $VENV/bin/activate  
pip install -r requirements.txt

fi  
touch $GIT\_CHECKOUT\_DIR/uwsgi.ini  
url=$GIT\_URL  
REPO=$(basename $url)  
if $DEPLOYED  
then  
curl -X POST –data “payload={\\”text\\”: \\”:slack: Jenkins Repo _$REPO_ branch _${GIT\_BRANCH}_ Deployment is _SUCCESS_ on _${SERVER\_ENV}_ server\\”}” $SLACK\_URL

else  
curl -X POST –data “payload={\\”text\\”: \\”:slack:  Jenkins Repo \*$REPO\* branch \*${GIT\_BRANCH}\* Deployment is \*FAILED\* on \*${SERVER\_ENV}\* server\\”}” $SLACK\_URL

fi  
else echo “NOT Deploying Code”  
fi

###End of the Script###