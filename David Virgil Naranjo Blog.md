![](https://dvirgiln.github.io/assets/images/aws_logo.png)

This article is an introduction to the deployment service provided by AWS. After the introduction, we will discuss a real AWS CodeDeploy deployment issue and how to troubleshoot it.

AWS CodeDeploy is a fantastic deployment service provided by AWS. Before digging into the CodeDeploy problem I want to include a short introduction about CodeDeploy.

CodeDeploy has two parts: application and deployment.

First it is required to create an **Application**, that only requires a name and compute type: ecs, lambda or ec2.

![](https://dvirgiln.github.io/assets/images/codedeploy_troubleshoot/create_application.png)

Once the application is created, then you can add a **Deployment Group**.

![](https://dvirgiln.github.io/assets/images/codedeploy_troubleshoot/application_details.png)

The Deployment Group consists on different attributes that describe how to deploy your application and where to deploy it:

1.  Define from where your application can be download by the AWS CodeDeploy agent. You need to point to a S3 bucket object.
2.  Define the type of deployment:
    -   **In place**: whenever a new revision of your application is created, then CodeDeploy automatically uploads it to the compute option selected (EC2, Lambda or ECS).
    -   **BlueGreen**: when a change in the revision is detected AWS automatically creates a new set of EC2 instances, install the application on them, deregister the old instances and register the new instances into the ASG.
3.  How to detect the EC2 instances:
    -   They could be part of an ASG.
    -   You can specify the EC2 Tag to associate with the CodeDeploy Deployment Group.

The following CloudFormation code describes perfectly the different parts of a CodeDeploy app:

```
12345678910111213141516171819202122CodeDeployApplication:
  Type: "AWS::CodeDeploy::Application"
  Properties:
    ApplicationName: !Sub ${Function}-${Environment}
DeploymentGroup:
  Type: "AWS::CodeDeploy::DeploymentGroup"
  DependsOn:
    - CodeDeployApplication
  Properties:
    ApplicationName: !Sub ${Function}-${Environment}
    DeploymentGroupName: !Sub ${Function}-${Environment}
    AutoScalingGroups:
      - !Ref ASGName
    DeploymentConfigName: !Ref DeploymentConfigName
    ServiceRoleArn: !Sub arn:aws:iam::${AWS::AccountId}:role/CodeDeploy
    Deployment:
      Revision:
        RevisionType: S3
        S3Location:
          Bucket: !Ref RevisionBucket
          Key: !Ref RevisionKey
          BundleType: zip

```

Next thing that needs to be setup: installing in our EC2 instances a **CodeDeploy agent**. To do that you can include a script like this in the EC2 **User-Data**:

```
1234567891011UserData:
  "Fn::Base64":
    |
       #!/bin/bash
       set -x
       source /etc/profile.d/aws_region.sh
       curl --noproxy aws-codedeploy-${AWS_DEFAULT_REGION}.s3.amazonaws.com -v -sLO  https://aws-codedeploy-${AWS_DEFAULT_REGION}.s3.amazonaws.com/latest/install
       chmod 0755 install
       ./install auto
       systemctl enable codedeploy-agent
       systemctl start codedeploy-agent

```

This script installs on the EC2 startup a CodeDeploy agent that polls from CodeDeploy trying to find a revision to install.

### CodeDeploy is not being invoked. Why?

Once of the most common problems with CodeDeploy is a deployment failure that happens not to have any trace/logging in the AWS CodeDeploy console. It is difficult to know what happened, you just see that all the CodeDeploy steps have been skipped.

![](https://dvirgiln.github.io/assets/images/codedeploy_troubleshoot/problem.png)

The problem is because CodeDeploy has timeout as there is no communication from the CodeDeploy agent that initially should have been installed in the EC2 instance.

The problem is inside of the EC2 instance. We need to SSH into it and check what it is going on.

### TroubleShooting

The first thing to check is the cloud init log file:

`cat /var/log/cloud-init.log`

```
123456Sep 12 15:08:14 ip-10-18-34-8 cloud-init: + chmod 0755 install
Sep 12 15:08:14 ip-10-18-34-8 cloud-init: + ./install auto
Sep 12 15:08:19 ip-10-18-34-8 cloud-init: + systemctl enable codedeploy-agent
Sep 12 15:08:19 ip-10-18-34-8 cloud-init: Failed to execute operation: No such file or directory
Sep 12 15:08:19 ip-10-18-34-8 cloud-init: + systemctl start codedeploy-agent
Sep 12 15:08:19 ip-10-18-34-8 cloud-init: Failed to start codedeploy-agent.service: Unit not foun

```

In line 2 we could see that it failed to enable the code deploy agent.

The next file to check is the **/tmp/codedeploy-agent.update.log**:

```
123456789101112I, [2019-09-12T15:08:15.037264 #3284]  INFO -- : Downloading version file from bucket aws-codedeploy-us-west-2 and key latest/VERSION...
I, [2019-09-12T15:08:15.097876 #3284]  INFO -- : Downloading version file from bucket aws-codedeploy-us-west-2 and key latest/VERSION...
I, [2019-09-12T15:08:15.144471 #3284]  INFO -- : Downloading package from bucket aws-codedeploy-us-west-2 and key releases/codedeploy-agent-1.0-1.1597.noarch.rpm...
I, [2019-09-12T15:08:15.225273 #3284]  INFO -- : Executing `/usr/bin/yum -y localinstall /tmp/codedeploy-agent-1.0-1.1597.noarch.tmp-20190912-3284-xheme.rpm`...
Loaded plugins: fastestmirror
Examining /tmp/codedeploy-agent-1.0-1.1597.noarch.tmp-20190912-3284-xheme.rpm: codedeploy-agent-1.0-1.1597.noarch
Marking /tmp/codedeploy-agent-1.0-1.1597.noarch.tmp-20190912-3284-xheme.rpm to be installed
Resolving Dependencies
--> Running transaction check
---> Package codedeploy-agent.noarch 0:1.0-1.1597 will be installed
Error: Rpmdb changed underneath us
E, [2019-09-12T15:08:19.845655 #3284] ERROR -- : Error installing /tmp/codedeploy-agent-1.0-1.1597.noarch.tmp-20190912-3284-xheme.rpm

```

From this log, we can see that it failed installing the CodeDeploy agent rpm with a cryptic message “**Error: Rpmdb changed underneath us**”.

The next thing that we need to check is the yum log:

`cat /var/log/yum.log`

```
123456789101112131415161718192021222324...
Mar 19 11:12:28 Installed: python36-3.6.6-2.el7.x86_64
Mar 19 11:12:45 Installed: amazon-ssm-agent-2.3.479.0-1.x86_64
Mar 19 11:13:16 Installed: unzip-6.0-19.el7.x86_64
Mar 19 11:13:32 Installed: inspec-2.1.43-1.el7.x86_64
Mar 20 12:30:33 Installed: python2-pyasn1-0.1.9-7.el7.noarch
Mar 20 12:30:33 Installed: python-enum34-1.0.4-1.el7.noarch
Mar 20 12:30:33 Installed: python-httplib2-0.9.2-1.el7.noarch
Mar 20 12:30:33 Installed: sshpass-1.06-2.el7.x86_64
Mar 20 12:30:33 Installed: libtommath-0.42.0-6.el7.x86_64
Mar 20 12:30:34 Installed: libtomcrypt-1.17-26.el7.x86_64
Mar 20 12:30:34 Installed: python2-crypto-2.6.1-15.el7.x86_64
Mar 20 12:30:34 Installed: python-keyczar-0.71c-2.el7.noarch
Mar 20 12:30:34 Installed: python2-jmespath-0.9.0-3.el7.noarch
Mar 20 12:30:34 Installed: python-ply-3.4-11.el7.noarch
Mar 20 12:30:35 Installed: python-pycparser-2.14-1.el7.noarch
Mar 20 12:30:35 Installed: python-cffi-1.6.0-5.el7.x86_64
Mar 20 12:30:35 Installed: python-idna-2.4-1.el7.noarch
Mar 20 12:30:35 Installed: python2-cryptography-1.7.2-2.el7.x86_64
Mar 20 12:30:35 Installed: python-paramiko-2.1.1-9.el7.noarch
Mar 20 12:30:46 Installed: ansible-2.7.7-1.el7.noarch
Mar 20 12:31:01 Installed: ruby-2.3.0-1.el7.centos.x86_64
Sep 12 15:08:33 Installed: bzip2-1.0.6-13.el7.x86_64
Sep 12 15:08:44 Installed: CylancePROTECT-2.0.1530-705.x86_64

```

The last line of this file indicates the last package that was installed.

It appears that the Cylance service installed 2 additional packages. It makes sense that these 2 packages being installed by a parallel process has created a race, whereby the Cylance service got to lock the RPMDB before the CodeDeploy RPM could, so the CodeDeploy failed. In order to avoid this race condition going forward, the easiest thing to try is to just add a ‘sleep 60s’ to the CloudInit script, just before the ‘./install auto’, to ensure that it pauses for 60 seconds, giving Cylance a chance to complete first.

Once the timeout is being added, CodeDeploy works fine. You can check all the output of CodeDeploy in the directory:

`/opt/codedeploy-agent/deployment-root/`

### Conclusions

TroubleShooting CodeDeploy it is not easy. The official documentation lacks a bit on these corner cases. And the CodeDeploy AWS Console does not show all the information to guide the developer to the error.

Remember to ssh into the machine and have a look into all the different log files that can help you narrowing down the error.