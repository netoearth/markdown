When building an instance in the cloud you may find yourself using the graphical user interface of your cloud provider. Clicking on buttons and paying for products just like you would on an e-commerce site. The drawback of this approach is that it’s a complete and total chaos and you might click a thousand different buttons to get your Virtual Machine configured properly. Moreover, what happens when you need to reproduce the same VM in the future? You’d have to go through the same exhaustive process once again.

## **Terraform: What it is and how does it solve the problem?**

Terraform is infrastructure as code tool that allows you to create and provision resources in cloud using HCL (HASHICORP Configuration Language) or JSON. In addition to HCL, the CDK for Terraform (CDKTF) allows you to define your infrastructure in a familiar programming language such as TypeScript, Python, Go, C#, or Java.

Terraform allows you to _automate_ and _manage_ cloud infrastructure, services and platform all from a single file which you can share and reuse. This saves a lot of time as you don’t have to handle cloud infrastructure manually by logging into your cloud services and creating each component of your infrastructure. Need to change something in your infrastructure? No problem, you can just update your code and changes will be applied.

![](https://miro.medium.com/max/842/1*BKqMHYB2yvEP4BtAqzp2QQ.png)

Visualization of Terraform Automation

**How to automate your Cloud Infrastructure?**

**Step 1: Setting Up Terraform.**

_I will be using windows to set up terraform and will be using AWS as cloud provider for this tutorial._

Setting up terraform is pretty straight forward, here are the steps you need to follow (I’ve used windows in this setup):

1.  Download Terraform: [Downloads | Terraform by HashiCorp](https://www.terraform.io/downloads)
2.  Extract terraform.exe
3.  Add terraform.exe path to path variable _(System properties →environment variables →path → \[Add path to terraform.exe\])_

You can verify your installation using CMD:

```
terraform --version
```

![](https://miro.medium.com/max/277/1*RmQ1ozA6f2EPRH5glOKDug.png)

Verifying Installation

**Step 2: Setting Up AWS CLI.**

Setting up AWS CLI is just downloading and running an exe file. You can get your installation file from here: [Installing or updating the latest version of the AWS CLI — AWS Command Line Interface (amazon.com)](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)

Once you’ve installed AWS CLI you can verify your installation by running the following command on CMD:

```
terraform configure
```

You’ll have to configure AWS by entering AWS Access Key ID  
AWS Secret Access Key and Default Region. (You can access your credentials from _IAM →Users →User\_Name →Security Credentials →Create Access Key →Download CSV(Credentials)._

Once you’ve configured AWS, you can start writing you first Terraform script. In this tutorial I’ll be creating a _T2. micro EC2_ instance with _Amazon Linux2 AMI_ using terraform_._

**Step 3: Writing Terraform Script.**

_Pre-requisites:_ **_Visual Studio Code & HashiCorp Terraform Extension._**

After installing Terraform extension, create a folder for main.tf (terraform script) with terraform.exe in it. Here’s the script for our configuration of EC2 instance:

```
provider "aws" {region = "us-east-1" #select your default region}
```

```
resource "aws_instance" "LinuxVM" {ami = "ami-047a51fa27710816e"instance_type = "t2.micro"}
```

![](https://miro.medium.com/max/842/1*d0TRoOdGjOkZZM3dw-T2YQ.png)

TF script

Simple enough, right? Now we just have to execute the script but first we’ll need to initialize terraform by using the following command:

```
terraform init
```

After initializing, you should have a similar output to one shown in picture above. Our next command is to validate our terraform script which is:

```
terraform validate
```

This should return _“_**_Success_**_! The configuration is valid.”_ if your script had no issues. The next step is to check whether the execution plan for a set of changes matches your expectations without making any changes to real resources or to the state.

```
terraform plan
```

![](https://miro.medium.com/max/842/1*70dvajNnPqfXltsf9y6rSA.png)

Output from terraform plan command

Our configuration file is ready and just needs to create resources in AWS. To create resources in AWS, use the following command:

```
terraform apply
```

![](https://miro.medium.com/max/842/1*vJCkDZedIlmnQKEmseQbQQ.png)

Verification

You’ll be asked to verify the execution, type _“yes”_ to approve. You should have an output like this if your instance was created: _Creation Complete._

Congrats, you’ve just created an EC2 instance using an infrastructure as code tool only.

**_Thanks for Reading!_** If you liked my tutorial let me know in the comments and please feel free to give feedback if there’s something to improve.

**Learning Resources:**

[The CLOUDNATIVEFM With SAIM — YouTube](https://www.youtube.com/channel/UC7B9fl8jQ8TEdOCypF4g3Wg)

[CNCF Islamabad](https://community.cncf.io/islamabad/)

[CNCF Twitter](https://twitter.com/CloudIslamabad)

**Addtional Learning Resources:**

[12 Terraform Best Practices to Improve your TF workflow (spacelift.io)](https://spacelift.io/blog/terraform-best-practices)

**References:**

[Terraform by HashiCorp](https://www.terraform.io/)