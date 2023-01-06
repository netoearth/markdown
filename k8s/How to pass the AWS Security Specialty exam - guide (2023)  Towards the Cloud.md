This guide contains the notes that I created during the preparation for the AWS Certified Security Specialty exam. I’ve mostly used the content that was provided for free by AWS using their [AWS Skillbuilder program](https://explore.skillbuilder.aws/), [AWS Whitepapers](https://aws.amazon.com/whitepapers/?whitepapers-main.sort-by=item.additionalFields.sortDate&whitepapers-main.sort-order=desc&awsf.whitepapers-content-type=*all&awsf.whitepapers-global-methodology=*all&awsf.whitepapers-tech-category=*all&awsf.whitepapers-industries=*all&awsf.whitepapers-business-category=*all), and the [official AWS documentation](https://docs.aws.amazon.com/?nc2=h_ql_doc_do).

I’ve curated the things that you should know for the exam, which means that the technical notes in this blog post are very dense and to the point. If you wish to dive deeper, then you can always read further in the links that I’ve provided above.

So let’s get started! **Here are the detailed steps to help you pass the AWS Certified Security Specialty exam**.

Table of Contents

1

-   [Who should take the AWS Certified Security Specialty exam?](https://towardsthecloud.com/aws-security-specialty-exam-guide#Who_should_take_the_AWS_Certified_Security_Specialty_exam "Who should take the AWS Certified Security Specialty exam?")
-   [How to prepare for the AWS Certified Security Specialty exam](https://towardsthecloud.com/aws-security-specialty-exam-guide#How_to_prepare_for_the_AWS_Certified_Security_Specialty_exam "How to prepare for the AWS Certified Security Specialty exam")
    -   [Exam overview](https://towardsthecloud.com/aws-security-specialty-exam-guide#Exam_overview "Exam overview")
    -   [Content outline](https://towardsthecloud.com/aws-security-specialty-exam-guide#Content_outline "Content outline")
    -   [Technical preparation notes](https://towardsthecloud.com/aws-security-specialty-exam-guide#Technical_preparation_notes "Technical preparation notes")
        -   [Domain 1: Incident response](https://towardsthecloud.com/aws-security-specialty-exam-guide#Domain_1_Incident_response "Domain 1: Incident response")
            -   [AWS Shield](https://towardsthecloud.com/aws-security-specialty-exam-guide#AWS_Shield "AWS Shield")
            -   [API Gateway Throttling](https://towardsthecloud.com/aws-security-specialty-exam-guide#API_Gateway_Throttling "API Gateway Throttling")
            -   [AWS Systems Manager parameter store](https://towardsthecloud.com/aws-security-specialty-exam-guide#AWS_Systems_Manager_parameter_store "AWS Systems Manager parameter store")
            -   [AWS Systems manager run command](https://towardsthecloud.com/aws-security-specialty-exam-guide#AWS_Systems_manager_run_command "AWS Systems manager run command")
        -   [Domain 2: Logging and monitoring](https://towardsthecloud.com/aws-security-specialty-exam-guide#Domain_2_Logging_and_monitoring "Domain 2: Logging and monitoring")
            -   [AWS CloudWatch](https://towardsthecloud.com/aws-security-specialty-exam-guide#AWS_CloudWatch "AWS CloudWatch")
            -   [AWS GuardDuty](https://towardsthecloud.com/aws-security-specialty-exam-guide#AWS_GuardDuty "AWS GuardDuty")
            -   [AWS Config](https://towardsthecloud.com/aws-security-specialty-exam-guide#AWS_Config "AWS Config ")
            -   [AWS Artifact](https://towardsthecloud.com/aws-security-specialty-exam-guide#AWS_Artifact "AWS Artifact")
            -   [Network packet inspection](https://towardsthecloud.com/aws-security-specialty-exam-guide#Network_packet_inspection "Network packet inspection")
            -   [AWS CloudTrail](https://towardsthecloud.com/aws-security-specialty-exam-guide#AWS_CloudTrail "AWS CloudTrail")
            -   [AWS VPC Flow Logs](https://towardsthecloud.com/aws-security-specialty-exam-guide#AWS_VPC_Flow_Logs "AWS VPC Flow Logs")
            -   [AWS Inspector](https://towardsthecloud.com/aws-security-specialty-exam-guide#AWS_Inspector "AWS Inspector")
            -   [Amazon Kinesis](https://towardsthecloud.com/aws-security-specialty-exam-guide#Amazon_Kinesis "Amazon Kinesis")
            -   [Amazon Athena](https://towardsthecloud.com/aws-security-specialty-exam-guide#Amazon_Athena "Amazon Athena")
        -   [Domain 3: Infrastructure security](https://towardsthecloud.com/aws-security-specialty-exam-guide#Domain_3_Infrastructure_security "Domain 3: Infrastructure security")
            -   [AWS WAF](https://towardsthecloud.com/aws-security-specialty-exam-guide#AWS_WAF "AWS WAF")
            -   [Network ACL (NACL) vs Security Group](https://towardsthecloud.com/aws-security-specialty-exam-guide#Network_ACL_NACL_vs_Security_Group "Network ACL (NACL) vs Security Group")
            -   [AWS AMI](https://towardsthecloud.com/aws-security-specialty-exam-guide#AWS_AMI "AWS AMI")
        -   [Domain 4: Identity and Access Management (IAM)](https://towardsthecloud.com/aws-security-specialty-exam-guide#Domain_4_Identity_and_Access_Management_IAM "Domain 4: Identity and Access Management (IAM)")
            -   [AWS IAM](https://towardsthecloud.com/aws-security-specialty-exam-guide#AWS_IAM "AWS IAM")
            -   [Amazon S3](https://towardsthecloud.com/aws-security-specialty-exam-guide#Amazon_S3 "Amazon S3")
            -   [Amazon S3 Block Public Access feature](https://towardsthecloud.com/aws-security-specialty-exam-guide#Amazon_S3_Block_Public_Access_feature "Amazon S3 Block Public Access feature")
            -   [Amazon Glacier](https://towardsthecloud.com/aws-security-specialty-exam-guide#Amazon_Glacier "Amazon Glacier")
            -   [AWS Lambda](https://towardsthecloud.com/aws-security-specialty-exam-guide#AWS_Lambda "AWS Lambda")
            -   [AWS STS with Active Directory (AD)](https://towardsthecloud.com/aws-security-specialty-exam-guide#AWS_STS_with_Active_Directory_AD "AWS STS with Active Directory (AD)")
            -   [Amazon Cognito](https://towardsthecloud.com/aws-security-specialty-exam-guide#Amazon_Cognito "Amazon Cognito")
        -   [Domain 5: Data protection](https://towardsthecloud.com/aws-security-specialty-exam-guide#Domain_5_Data_protection "Domain 5: Data protection")
            -   [Application Load Balancer (ALB)](https://towardsthecloud.com/aws-security-specialty-exam-guide#Application_Load_Balancer_ALB "Application Load Balancer (ALB)")
            -   [AWS Key Management Service (KMS)](https://towardsthecloud.com/aws-security-specialty-exam-guide#AWS_Key_Management_Service_KMS "AWS Key Management Service (KMS)")
            -   [AWS KMS access control](https://towardsthecloud.com/aws-security-specialty-exam-guide#AWS_KMS_access_control "AWS KMS access control")
            -   [Client-side encryption vs. Server-side encryption](https://towardsthecloud.com/aws-security-specialty-exam-guide#Client-side_encryption_vs_Server-side_encryption "Client-side encryption vs. Server-side encryption")
-   [AWS Certified Security Specialty Study material](https://towardsthecloud.com/aws-security-specialty-exam-guide#AWS_Certified_Security_Specialty_Study_material "AWS Certified Security Specialty Study material")
    -   [AWS Documentation](https://towardsthecloud.com/aws-security-specialty-exam-guide#AWS_Documentation "AWS Documentation")
    -   [AWS Whitepapers](https://towardsthecloud.com/aws-security-specialty-exam-guide#AWS_Whitepapers "AWS Whitepapers")
-   [Conclusion](https://towardsthecloud.com/aws-security-specialty-exam-guide#Conclusion "Conclusion")

## **Who should take the** AWS Certified Security Specialty exam?

According to AWS, they recommend you have the following experience and skills prior to taking the exam:

-   5 years of IT security experience in designing and implementing security solutions
-   2 years of hands-on experience in securing AWS workloads
-   Patch management and security automation
-   Know how encryption at rest and transit works within AWS
-   Access control on network and data layers
-   Data retention
-   Logging and monitoring strategies
-   Know the AWS shared responsibility model

## How to prepare for the AWS Certified Security Specialty exam

In this guide, we’ll follow the domains and topics that are provided in the content outline of the official [AWS Certified Security – Specialty (SCS-C01) Exam Guide](https://d1.awsstatic.com/training-and-certification/docs-security-spec/AWS-Certified-Security-Specialty_Exam-Guide.pdf).

For each domain, we’ll let you know what AWS expects from you (knowledge-wise) and then I provide the technical notes that help you prepare and meet up these expectations.

### Exam overview

This is what you can expect when you schedule the AWS Certified Security Specialty exam:

-   Consists of 65 multiple-choice, multiple-answer questions.
-   The exam needs to be completed within 170 min (**Note**: follow this [advice](https://towardsthecloud.com/7-tips-aws-certification-exam) to permanently receive 30 minutes extra time for your AWS exams)
-   Costs $300,-
-   The minimum passing score is 750 points
-   The exam is available in English, Japanese, Korean, and Simplified Chinese.

### Content outline

The content outline of the exam consists of 6 separate domains, each with its own weighting.

The table below lists the domains with their weightings:

![AWS Certified Security Specialty exam content outline](https://towardsthecloud.com/wp-content/uploads/aws-security-specialt-content-outline-1024x321.webp)

AWS Certified Security Specialty exam content outline

Further on in the guide, a more detailed explanation is added to each domain to give a rough idea of what you should know.

![AWS Certified Security Specialty exam detailed content outline](https://towardsthecloud.com/wp-content/uploads/aws-security-specialty-detailed-content-outline-331x1024.webp)

AWS Certified Security Specialty exam detailed content outline

### Technical preparation notes

In this section, I’ve bundled up my notes which you can use when you’re preparing for the AWS Certified Security Specialty exam.

Prior to this Blogpost, I also released a guide for the [AWS Cloud Practitioner exam technical preparation notes](https://towardsthecloud.com/aws-cloud-practitioner-exam-guide/#technical-preparation-notes). This contains the foundational information which also helps for this exam, so I highly recommend reading the notes from there as well.

Moving on to the preparation, I’ve written technical notes which highlight all the important details regarding security that are worth remembering for the exam.

Since Security is such a broad topic it covers a broad variety of AWS Services. **This means you don’t need to know the ins- and outs of each specific service**, but it’s good to know the basics prior to learning about their specific security properties.

This guide focuses mainly on the security properties of the AWS services. To simplify the learning process, I’ve categorized my technical notes into the domain sections as it’s displayed in the content outline.

#### Domain 1: Incident response

You should be comfortable being prepared for the following in this domain:

-   When an AWS abuse notice lands in your inbox, you’ll need to swiftly assess the situation and determine if a compromised instance or exposed access keys are to blame.
-   Don’t let a security incident catch you off guard! Make sure your incident response plan covers all the necessary AWS services.
-   Stay on top of potential threats by regularly evaluating your automated alerting configuration. And if an issue does arise, be ready to jump into action and implement any necessary remediation measures.

##### AWS Shield

AWS Shield protects against SUN/UDP floods, reflection attacks, and other layers 3/4 attacks.

AWS Shield Advanced provides enhanced protections for your applications running on ELB, CloudFront, WAF, ASG, Cloudwatch, and R53 against larger and more sophisticated attacks. Costs 3000 dollars per month.

##### API Gateway Throttling

-   AWS Throttles API GW when too many requests
-   When request submissions exceed the steady-state request rate and burst limits, API GW fails the limit exceeding requests and returns 429 too many requests errors.
-   By default API GW limits the steady-state requests to 10 000 requests per sec
-   The account level and burst limit can be increased
-   You can enable API caching (default 300 TTL)

##### AWS Systems Manager parameter store

-   Passwords and database connection strings can be stored in the SSM Parameter store
-   You can store it as plain text or you can encrypt it
-   You can reference these values by using their names
-   You can use it with various AWS services such as CloudFormation, EC2, AWS Lambda, etc.

##### AWS Systems manager run command

-   Commands can be applied to a group of systems based on tags or manually
-   SSM agent needs to be installed
-   The commands and parameters are defined in a Systems Manager Document
-   Commands can be issued using AWS Console, AWS CLI, AWS Tools for Windows
-   You can use it with on-premise systems as well as EC2 instances

#### Domain 2: Logging and monitoring

You should be comfortable being prepared for the following in this domain:

-   Keep your systems safe by designing and implementing security monitoring and alerting systems.
-   When something goes wrong, you’ll need to be able to troubleshoot and fix any issues with your security monitoring and alerting systems.
-   Being able to design and implement a logging solution that gives you a clear picture of what’s going on.

##### AWS CloudWatch

Provides real-time monitoring of your AWS resources and applications you run on AWS.

-   Supports custom metrics next to the built-in ones
-   Allows you to collect and process log files from AWS Services and your applications on EC2 and containers
-   Detects events and responds with notifications or automated responses/actions

##### AWS GuardDuty

AWS GuardDuty is a threat detection service that uses machine learning to continuously monitor your AWS account for malicious behavior. Some of the features it provides:

-   Unusual API calls e.g. call from a malicious or unknown IP address
-   Attempts to disable AWS CloudTrail
-   Unauthorized deployments
-   Compromised instances
-   Port scanning and failed logins on EC2 instances
-   Monitors CloudTrail logs, VPC Flow Logs, and DNS Logs
-   Centralize detection across multiple AWS accounts

##### AWS Config

AWS Config is a managed service that provides resource inventory, configuration history, and change notifications.

-   Continuously captures configuration changes on your AWS resources
-   Enables compliance monitoring 
-   Can identify and mitigate configuration changes using automated remediations

##### AWS Artifact

AWS Artifact provides on-demand access to security and compliance reports from AWS and ISVs who sell their products on AWS Marketplace.

-   Central resource for compliance and security-related information
-   you can download copies of the documents
-   demonstrate compliance with regulators
-   evaluate your own cloud architecture
-   Cover industry standards like ISO 27001

##### **Network packet inspection**

Network packet inspection is typically provided via 3rd party tools, make sure to know that when you receive questions about securing your network with IDS/IPS packet inspection.

The following AWS Services **don’t** provide network packet inspection:

-   AWS VPC Flow logs
-   AWS WAF
-   AWS Shield
-   AWS Security groups
-   Host-based firewalls like `iptables` on Amazon EC2

##### AWS CloudTrail

Allows you to track user activity and API usage within AWS. A couple of best practices include:

-   Enable in all regions
-   Enable log file validation
-   Encrypted logs
-   Integrate with CloudWatch logs
-   Centralize logs from all accounts
-   Create additional trails as needed

##### AWS VPC Flow Logs

VPC Flow Logs allow you to capture information about the IP traffic that goes through your network interfaces in your VPC. The logs are then captured in AWS CloudWatch Logs for analysis and reporting.

Flow logs can be created at 3 levels:

-   AWS VPC
-   Subnet
-   Network interface

You can choose to filter the type of traffic that will be logged, either accepted-, rejected- or all-traffic.

##### AWS Inspector

Amazon Inspector checks for security exposures and vulnerabilities in your EC2 instances. Assessments include:

-   Check for ports reachable from outside the VPC (No Inspector agent needed)
-   CIS benchmarks
-   Vulnerable software (CVE)
-   Security best practices
-   Runtime behavioral analysis
-   Assessment reports are generated with findings and include remediation steps

##### Amazon Kinesis

Collect, process, and analyze real-time streaming data for quick incident response

-   Log ingestion: use services such as Kinesis data streams or data firehose
-   Log analysis: Kinesis data analytics

##### Amazon Athena

Provides an interactive query service to analyze data in Amazon S3 using standard SQL.

-   Allows to query data located in S3 using standard SQL
-   Serverless Commonly used to analyze log data stored in S3
-   Cross-region queries are supported
-   No need to load or aggregate data
    -   Uses Presto and Apache Hive

#### Domain 3: Infrastructure security

To be proficient in this domain, you should have knowledge of the following topics:

-   Designing edge security on AWS
-   Designing and implementing a secure network infrastructure
-   Troubleshooting issues with a secure network infrastructure
-   Designing and implementing host-based security

##### AWS WAF

AWS Web Application Firewall (WAF) that helps protect your web applications:

-   Increase protection from web attacks
-   Integrated security with other AWS Services
-   Improved web traffic visibility
-   Enhanced security with managed rules

AWS WAF available conditions to secure your applications:

| **Condition** | **Allow or Block Requests Based On…** |
| --- | --- |
| Cross-site scripting match | Whether the requests appear to contain malicious scripts |
| IP match | The IP addresses that they originate from |
| Geographic match | The country that they originate from |
| Size constraint | Whether the requests exceed a specified length |
| SQL injection match | Whether the requests appear to contain malicious SQL code |
| String match | Strings that appear in the requests |
| Regex match | A regex pattern that appears in the requests |

##### Network ACL (NACL) vs Security Group

**ACL** – Network ACLs are applied to subnets within a VPC. Any resource that is within that subnet will inherit the security rules that are applied in that list. ACLs are also stateless meaning that you need to explicitly allow connections from incoming and outgoing traffic.

The rules within an NACL are sequenced based meaning that when the first rule matches the request it stops processing any additional rules with lower priority. For NACLs you need to configure ingress and egress rules for incoming and outgoing traffic.

**Security group** – Security groups are applied to AWS resources such as EC2 instances, RDS, ECS, etc.. meaning that you have finer control of what goes in and out of your AWS resource.

Security groups are stateful meaning any changes applied to an incoming rule will be automatically applied to the outgoing rule in security groups. By default, it denies all ingress traffic and allows all egress traffic. You can also whitelist other security groups instead of having to whitelist addresses.

##### AWS AMI

Amazon Machine Image (AMI) is the image or snapshot that contains the operating system and applications for your Amazon EC2 instances.

To improve the security of an AMI, consider taking the following actions:

-   Disabling services that use clear text authentication
-   Disabling non-essential services during startup
-   Disabling services such as file sharing and the print spooler if they are not needed
-   Deleting all user SSH public and private keys
-   Removing and disabling passwords for all accounts
-   Deleting all AWS credentials from the disk and configuration files

#### Domain 4: Identity and Access Management (IAM)

To be proficient in this domain, you should have knowledge of the following topics:

-   Designing and implementing an authorization and authentication system to access AWS resources that can scale as needed
-   Troubleshooting issues with an authorization and authentication system to access AWS resources

##### AWS IAM

There are 3 IAM access policy types:

-   **AWS managed** – AWS managed policies are pre-defined policies that are created and managed by AWS. They cover a wide range of AWS services and are designed to provide a secure and flexible way to manage permissions.
-   **Customer-managed** – Customer-managed policies are policies that are created and managed by the customer. These policies are useful when the customer wants to have more control over their policies and when the available AWS-managed policies do not meet their needs.
-   **Inline** – Inline policies are embedded directly into an IAM user, group, or role and cannot be shared with other IAM entities

There are 5 available policy elements that can be used in an IAM policy document:

| **Element type** | **Description** | **Required** |
| --- | --- | --- |
| Effect | Specifies whether the statement results in an allow or an explicit deny. | YES |
| Action | Describes the specific action or actions that will be allowed or denied. | YES |
| Resource | Specifies the object or objects that the statement covers. | YES |
| Condition | Specifies conditions for when a policy is in effect. | NO |
| Principle | Specifies the entity that is allowed or denied access to a resource. Used for resource-based or trust policies. | NO |

##### Amazon S3

**S3 Bucket policy** – S3 Bucket policies are a great way to grant cross-account access to your s3 bucket without IAM roles.

Use condition `"aws:SecureTransport": true` in your S3 bucket policy, to force SSL encryption in transit.

**Troubleshooting tip S3 IAM Policies** – For cross-account access to S3 check whether the external AWS account is trusted. The IAM policy in the external account needs to allow the user to call `STS:AssumeRole`. The IAM policy in the trusting account needs to allow the action.

##### Amazon S3 Block Public Access feature

The **Block Public Access** feature was introduced to help prevent accidental exposure to a bucket.

**Available “block public access” options for S3 Buckets:**

-   **Block** public access to buckets and objects granted through **any access control lists** (ACLs).
-   **Block** public access to buckets and objects granted through **new access control lists** (ACLs)
-   **Block** public and cross-account access to buckets and objects through **any public bucket policies.**
-   **Block** public access to buckets and objects granted through **new public bucket policies.**

Things you should and shouldn’t do when blocking public access on S3:

| **Do this** | **Don’t do this** |
| --- | --- |
| Use S3 Block Public Access at the account level to make sure that buckets and accounts reject public access to your objects. | Don’t allow public access unless you have an explicit reason that it must be public (e.g., static content). |
| Audit existing bucket settings. | Don’t open up a bucket to public access as a means of troubleshooting a software issue as a quick fix. |
|   
 | Don’t grant unrestrained admin rights to users who require some form of account management. |

##### Amazon Glacier

**Glacier vault lock** – Set up compliance controls for individual Glacier vaults using a Vault Lock Policy. You have 24 hours to validate the lock policy, after that, you can’t change the policy since it’s immutable.

##### AWS Lambda

**Lambda execution role** – allows you to specify how your lambda function can access different AWS services within the boundaries that you define.

**Lambda function policy** – This is a resource-based policy that allows you to define what event can trigger your lambda function (similar to an S3 bucket policy).

##### AWS STS with Active Directory (AD)

-   **Federation** – combining a list of users in one domain such as IAM with a list of users in another domain such as AD
-   **Identity broker** –  A service that allows you to take identity from one place and federate it to another.
-   **Identity store** – Services like Active directory, Facebook, Google, etc.
-   **Identities** – A user of a service from an identity store.

##### Amazon Cognito

Amazon Cognito provides Web identity federation with the following features:

-   Sign up and sign in to your apps
-   Mostly recommended for mobile applications
-   Access for guest users
-   Acts as an identity broker between your application and web ID providers so you don’t need to write your own custom broker.

#### Domain 5: Data protection

To be proficient in data protection, you should have knowledge of the following topics:

-   Designing and implementing key management
-   Troubleshooting key management issues
-   Designing and implementing data encryption solutions for data at rest and in transit

##### Application Load Balancer (ALB)

The ALB only supports TLS/SSL termination, if you want to terminate TLS/SSL on the EC2 instance then you need to use a Network Load Balancer (NLB) or a Classic Load Balancer.

##### AWS Key Management Service (KMS)

AWS KMS is a huge part of the AWS Certified Security specialty exam, so please pay attention to this part of the guide!

AWS KMS is a fully managed service that makes it easy to create and control the encryption keys used to encrypt your data. With KMS, you can create, rotate, and manage the encryption keys used to encrypt your data, as well as audit the use of your keys, and detect and respond to any unauthorized use.

There are two types of customer-managed KMS keys:

-   **Symmetric** – Single encryption key that is used for both encrypt and decrypt operations. This is the most common key for AWS services
-   **Asymmetric** – A public and private keypair that can be used for encrypt/decrypt and sign/verify operations. This is used mostly for SSH and EC2 instances.  

**AWS KMS key deletion** – Scheduling a key deletion requires a minimum waiting period of 7 days before it will be deleted (this waiting period doesn’t apply to your own key material that was imported for CMK)

The **Customer Managed Key** (CMK) contains the following properties:

-   Add an alias that can be used to reference keys easily
-   Creation date
-   Description
-   All KMS keys can be in the Enabled, Disabled, and PendingDeletion states.
-   AWS or the customer can import the key material to create a KMS key
-   Can’t be exported out of AWS once it’s created
-   Can’t move or copy the key from one AWS Region to another AWS Region
-   Keys can be deleted
-   Keys can be rotated automatically every year (doesn’t apply to CMK with imported key material)

The difference between the key rotation for the managed KMS keys:

| **AWS Managed** | **Customer Managed** | **Customer Managed (imported key material)** |
| --- | --- | --- |
| Rotates automatically every year | Rotates automatically every year (disabled by default) | No automatic rotation |
| Can’t rotate manually | You **can** rotate it manually | You **must** rotate it manually |
| AWS manages and keeps the old key | Creates a new CMK and updates your applications or key alias to use the new CMK. The old key is kept for older encrypted files. | Creates a new CMK and updates your applications or key alias to use the new CMK. The old key is kept for older encrypted files. |

Next to AWS IAM to protect key usage, there are two other security mechanisms available to protect your AWS KMS keys:

| **Policies** | **Grants** |
| --- | --- |
| Resource-based permissions | Temporary or more granular permissions |
| Similar syntax to IAM policies | Programmatically delegate CMKs to a user in your own or another AWS account. |
| Specifies who can manage a key and who can encrypt/decrypt | Are for temporary use of the CMK |

##### AWS KMS access control

Access to KMS CMK’s is controlled using:

-   The key policy – add the root user, not individual IAM users
-   IAM Policies – define the allowed actions and the CMK ARN
-   If you want to enable cross-account access:
    -   Enable trust to the external account in the key policy for the account which owns the CMK
    -   Enable access to KMS in the IAM Policy for the external account (specify the KMS key in the resource section combined with the actions that are allowed)
-   Both steps are necessary otherwise it will not work

An important distinction between KMS Key policy and KMS IAM Policy:

| **KMS Key policy** | **KMS IAM Policy** |
| --- | --- |
|   
Resource-based policy attached to the CMK defines key users, key administrators, and trusted external accounts |   
Assigned to User, Group, or Role. Defines the allowed actions e.g. `KMS:ListKeys`, `kms:Encrypt`, `kms:Decrypt` |

**KMS policy conditions** – The `kms:ViaService` condition key limits the use of an AWS KMS CMK to requests from specified AWS services.

For example, the following statement from a key policy uses the `kms:ViaService` condition key to allow a CMK to be used for the specified actions only when the request comes from Amazon EC2 or Amazon RDS in the US West (Oregon) region on behalf of `ExampleUser`.

```
{
  "Effect": "Allow",
  "Principal": {
    "AWS": "arn:aws:iam::111122223333:user/ExampleUser"
  },
  "Action": [
    "kms:Encrypt",
    "kms:Decrypt",
    "kms:ReEncrypt*",
    "kms:GenerateDataKey*",
    "kms:CreateGrant",
    "kms:ListGrants",
    "kms:DescribeKey"
  ],
  "Resource": "*",
  "Condition": {
    "StringEquals": {
      "kms:ViaService": [
        "ec2.us-west-2.amazonaws.com",
        "rds.us-west-2.amazonaws.com"
      ]
    }
  }
}
```

##### **Client-side encryption vs. Server-side encryption**

When choosing an encryption solution for your data in the AWS Cloud, consider the following three questions:

1.  How will the keys be stored? Will you use your own hardware or hardware provided by AWS?
2.  Where will the keys be used? Will they be used by client software or on AWS?
3.  Who will manage the keys? Do you want to manage permissions at the user or application level, or have AWS handle it?

| **Client-side encryption** | **Server-side encryption** |
| --- | --- |
| The user encrypts data before sending it to AWS.  
 | AWS encrypts data on your behalf after it has been received. |
| Data is stored in an encrypted state. | Transparent to end user. |
| Keys and algorithms are known only to the user. | AWS rotates the CMK on regular basis. |

## AWS Certified Security Specialty Study material

On the internet, you’ll find a lot of study material for the AWS Certified Security Specialty exam. It can be really overwhelming if you need to search for great quality material.

Lucky for you, I’ve spent some time curating the available study material and highlighting some of the stuff worth reading.

### AWS Documentation

The notes that I’ve written in the previous chapter contain keywords and summaries, don’t solely depend on that! If a concept or keyword is unknown to you then see it as an incentive to dive deeper into that topic.

Based on my experience with the exam I would recommend reading the official documentation on the following services:

-   [AWS CloudTrail](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-user-guide.html)
-   [Amazon CloudWatch](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/WhatIsCloudWatch.html)
-   [AWS Config](https://docs.aws.amazon.com/config/latest/developerguide/WhatIsConfig.html)
-   [AWS Organizations](https://docs.aws.amazon.com/organizations/latest/userguide/orgs_introduction.html)
-   [AWS Systems Manager](https://docs.aws.amazon.com/systems-manager/latest/userguide/what-is-systems-manager.html)
-   [AWS Security Hub](https://docs.aws.amazon.com/securityhub/latest/userguide/what-is-securityhub.html)
-   [AWS Shield](https://docs.aws.amazon.com/waf/latest/developerguide/shield-chapter.html)
-   [Amazon VPC](https://docs.aws.amazon.com/vpc/latest/userguide/what-is-amazon-vpc.html)
    -   VPC Endpoints
    -   Network ACLs
    -   Security groups
-   [AWS WAF](https://docs.aws.amazon.com/waf/latest/developerguide/waf-chapter.html)
-   [AWS CloudHSM](https://docs.aws.amazon.com/cloudhsm/latest/userguide/introduction.html)
-   [AWS Directory Service](https://docs.aws.amazon.com/directoryservice/latest/admin-guide/what_is.html)
-   [AWS Certificate Manager](https://docs.aws.amazon.com/acm/latest/userguide/acm-overview.html)
-   [Amazon GuardDuty](https://docs.aws.amazon.com/guardduty/latest/ug/what-is-guardduty.html)
-   [Amazon Inspector](https://docs.aws.amazon.com/inspector/latest/user/what-is-inspector.html)
-   [AWS IAM](https://docs.aws.amazon.com/iam/?icmpid=docs_homepage_security)
-   [AWS KMS](https://docs.aws.amazon.com/kms/latest/developerguide/overview.html)
-   [Amazon Macie](https://docs.aws.amazon.com/macie/latest/user/what-is-macie.html)

### AWS Whitepapers

There are a couple of whitepapers from AWS that are worth reading in regard to the preparation for this exam:

-   [AWS Well-Architected Framework – Security Pillar](https://docs.aws.amazon.com/wellarchitected/latest/security-pillar/wellarchitected-security-pillar.pdf)
-   [AWS Security best practices](https://d1.awsstatic.com/whitepapers/aws-security-best-practices.pdf)
-   [AWS Key Management best practices](https://d1.awsstatic.com/whitepapers/aws-kms-best-practices.pdf)
-   [Building a scalable and secure multi-VPC AWS network architecture](https://docs.aws.amazon.com/whitepapers/latest/building-scalable-secure-multi-vpc-network-infrastructure/building-scalable-secure-multi-vpc-network-infrastructure.pdf)

## Conclusion

In conclusion, this guide provided the technical notes that I created during the preparation for the AWS Certified Security Specialty exam. The exam covers a range of topics like incident response, data protection, infrastructure security, and identity and access management.

AWS recommends having at least five years of IT security experience and two years of hands-on experience in securing AWS workloads, as well as a familiarity with patch management, security automation, encryption at rest and in transit, access control, data retention, logging, and monitoring strategies.