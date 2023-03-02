![Cover image for How to build a CI/CD Pipeline in AWS using CodeCommit, CodeDeploy, CodePipeline: Hands-on!](https://res.cloudinary.com/practicaldev/image/fetch/s--sRQBoYP---/c_imagga_scale,f_auto,fl_progressive,h_420,q_auto,w_1000/https://dev-to-uploads.s3.amazonaws.com/i/ppm0fk9mh2atsbtab0n5.png)

This tutorial helps to continuous integration and continuous deployment of your application from the local system to your QA or Staging or Production server. So please follow the below steps to get it done.

**Step-1: Install GIT and configure in your local system**

Go to git-scm.com and download it to your system then install it. Then after you need to configure the GIT on the local system using below command.  

```
$ git config - -global user.name “Trilochan Parida”
$ git config - -global user.email “tri***@gmail.com”
```

Enter fullscreen mode Exit fullscreen mode

**Step-2: Create CodeCommit repository ( [YouTube](https://youtu.be/-tkfm8a7L1U?t=54s) )**

1.  Go to CodeCommit service in the AWS console.
2.  Create a new repository with your project name.
3.  Copy the clone URL → Clone HTTPS.

Before clone to your local system, you need the credentials to access this repository, so let’s create that first.

**Step-3: Create a new AWS IAM user and generate GIT credentials ([YouTube](https://youtu.be/-tkfm8a7L1U?t=1m53s))**

1.  Add the user and create a group.
2.  Attach IAM policy “AWSCodeCommitFullAccess” and “AWSCodePipelineFullAccess”.
3.  Security Credentials → HTTPS GIT Credentials → Generate.

**Step-4: GIT Clone ([YouTube](https://youtu.be/-tkfm8a7L1U?t=3m40s))**

Using the above credentials, now we can clone the repository from AWS CodeCommit to the local system. And the command to clone is  

```
$ git clone <URL>
```

Enter fullscreen mode Exit fullscreen mode

As we are in AWS IAM service console so let’s create two Roles for future use in CodeDeploy and EC2 to access S3 artifacts.

**Step-5: Create service Role for EC2 and CodeDeploy ( [YouTube](https://youtu.be/-tkfm8a7L1U?t=4m53s) )**

1.  Create an IAM role for CodeDeploy service and attach existing IAM policy “AWSCodeDeployRole”
2.  Create an IAM role for EC2 service and attach existing IAM policy “AmazonS3ReadOnlyAccess”

**Step-6: Code push and understanding of appspec.yml ( [YouTube](https://youtu.be/-tkfm8a7L1U?t=7m) )**

Now we will put our application to that same folder which cloned from AWS. And here you have remembered to create an appspec.yml file to automate the code deployment. Here I will recommend watching my video to better understanding.

appspec.yml  

```
version: 0.0
os: linux
files:
  - source: /
    destination: /var/www/html/
```

Enter fullscreen mode Exit fullscreen mode

**Step-7: Set up an EC2 instance, install Nginx, and install CodeDeploy agent ( [YouTube](https://youtu.be/-tkfm8a7L1U?t=8m50s) )**

1.  Launch EC2 instance with default VPC, instance type would be t2.micro, select IAM Role that we had created in step-5 and add Tags with Key and value which will be used in CodeDeploy.
2.  Install Nginx

```
$ apt update
$ apt install nginx
```

Enter fullscreen mode Exit fullscreen mode

1.  Install CodeDeploy agent

```
$ apt install ruby
$ apt install wget
$ wget https://aws-codedeploy-ap-south-1.s3.ap-south-1.amazonaws.com/latest/install //For Mumbai Region
$ chmod +x ./install
$./ install auto
```

Enter fullscreen mode Exit fullscreen mode

Now you need to code deploy agent service for that use below command  

```
$ service codedeploy-agent start
```

Enter fullscreen mode Exit fullscreen mode

**Step-8: Create an application in CodeDeploy ( [YouTube](https://youtu.be/-tkfm8a7L1U?t=16m15s) )**

1.  Go to CodeDeploy service console
2.  Create a new application with your application name and select compute platform to EC2/On-Premises
3.  Create deployment groups with name and deployment type is “in-place” and select environment configuration as Amazon EC2 instance with Key=Name and Value= “tag value of EC2”

**Step-9: Create CodePipeline ( [YouTube](https://youtu.be/-tkfm8a7L1U?t=17m27s) )**

1.  Go to AWS CodePipeline service console
2.  Create a pipeline with a name and keep the default set up.
3.  Add Source Stage → Source provider: AWS CodeCommit → Select your repository → Select your branch → Detection option: CloudWatch Event
4.  Add Build Stage → Skip
5.  Add Deploy Stage → Deploy Provider: AWS CodeDeploy → Enter application name → EnterDeployment Group

**Step-10: Test your application and watch the complete video**

Go to your browser and type your IP. Congratulations!!! You have successfully deployed your code.

<iframe width="710" height="399" src="https://www.youtube.com/embed/-tkfm8a7L1U" allowfullscreen="" loading="lazy"></iframe>