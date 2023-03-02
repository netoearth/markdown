![](https://expeditedsecurity.com/img/plain-english/aws-logo.png) Hey, have you heard of the new AWS services: ContainerCache, ElastiCast and QR72? Of course not, I just made those up.

But with 50 plus opaquely named services, we decided that enough was enough and that some plain english descriptions were needed.

![](https://expeditedsecurity.com/img/shirt-on-white-background.png)

### Use Heroku? Try Expedited WAF. Get a Free Tee.

Either [Book a Demo](https://app.harmonizely.com/expedited/30-min) or [Install Expedited WAF](https://elements.heroku.com/addons/expeditedwaf?utm_campaign=t-shirt&utm_source=expeditedsecurity.com&utm_medium=marketingsite) from the Heroku Marketplace - after a week we'll ask for your US or Canda mailing address and some feedback.

## Run an App Services

No matter what you do with AWS you'll probably end up using these services as everything else interacts with them.

### EC2

**Should have been called**  
Amazon Virtual Servers **Use this to**  
Host the bits of things you think of as a computer. **It's like**  
It's handwavy, but EC2 instances are similar to the virtual private servers you'd get at Linode, DigitalOcean or Rackspace.

### IAM

**Should have been called**  
Users, Keys and Certs **Use this to**  
Set up additional users, set up new AWS Keys and policies.

### S3

**Should have been called**  
Amazon Unlimited FTP Server **Use this to**  
Store images and other assets for websites. Keep backups and share files between services. Host static websites. Also, many of the other AWS services write and read from S3.

[S3 in Plain English](https://expeditedsecurity.com/aws-in-plain-english/s3)

### VPC

**Should have been called**  
Amazon Virtual Colocated Rack **Use this to**  
Overcome objections that "all our stuff is on the internet!" by adding an additional layer of security. Makes it appear as if all of your AWS services are on the same little network instead of being small pieces in a much bigger network. **It's like**  
If you're familar with networking: VLANs

### Lambda

**Should have been called**  
AWS App Scripts **Use this to**  
Run little self contained snippets of JS, Java or Python to do discrete tasks. Sort of a combination of a queue and execution in one. Used for storing and then executing changes to your AWS setup or responding to events in S3 or DynamoDB.

[Lambda in Plain English](https://expeditedsecurity.com/aws-in-plain-english/lambda)

## Web Developer Services

If you're setting up a web app, these are mostly what you'd end up using. These are similar to what you'd find in Heroku's Addon Marketplace.

### API Gateway

**Should have been called**  
API Proxy **Use this to**  
Proxy your apps API through this so you can throttle bad client traffic, test new versions, and present methods more cleanly. **It's like**  
3Scale

### RDS

**Should have been called**  
Amazon SQL **Use this to**  
Be your app's Mysql, Postgres, and Oracle database. **It's like**  
Heroku Postgres

### Route53

**Should have been called**  
Amazon DNS + Domains **Use this to**  
Buy a new domain and set up the DNS records for that domain. **It's like**  
DNSimple, GoDaddy, Gandi

### SES

**Should have been called**  
Amazon Transactional Email **Use this to**  
Send one-off emails like password resets, notifications, etc. You could use it to send a newsletter if you wrote all the code, but that's not a great idea. **It's like**  
SendGrid, Mandrill, Postmark

### Cloudfront

**Should have been called**  
Amazon CDN **Use this to**  
Make your websites load faster by spreading out static file delivery to be closer to where your users are. **It's like**  
MaxCDN, Akamai

### CloudSearch

**Should have been called**  
Amazon Fulltext Search **Use this to**  
Pull in data on S3 or in RDS and then search it for every instance of 'Jimmy.' **It's like**  
Sphinx, Solr, ElasticSearch

### DynamoDB

**Should have been called**  
Amazon NoSQL **Use this to**  
Be your app's massively scalable key valueish store. **It's like**  
MongoLab

### Elasticache

**Should have been called**  
Amazon Memcached **Use this to**  
Be your app's Memcached or Redis. **It's like**  
Redis to Go, Memcachier

### Elastic Transcoder

**Should have been called**  
Amazon Beginning Cut Pro **Use this to**  
Deal with video weirdness (change formats, compress, etc.).

### SQS

**Should have been called**  
Amazon Queue **Use this to**  
Store data for future processing in a queue. The lingo for this is storing "messages" but it doesn't have anything to do with email or SMS. SQS doesn't have any logic, it's just a place to put things and take things out. **It's like**  
RabbitMQ, Sidekiq

### WAF

**Should have been called**  
AWS Firewall **Use this to**  
Block bad requests to Cloudfront protected sites (aka stop people trying 10,000 passwords against /wp-admin) **It's like**  
Sophos, Kapersky

## Mobile App Developer Services

These are the services that only work for mobile developers.

### Cognito

**Should have been called**  
Amazon OAuth as a Service **Use this to**  
Give end users - (non AWS) - the ability to log in with Google, Facebook, etc. **It's like**  
OAuth.io

### Device Farm

**Should have been called**  
Amazon Drawer of Old Android Devices **Use this to**  
Test your app on a bunch of different IOS and Android devices simultaneously. **It's like**  
MobileTest, iOS emulator

### Mobile Analytics

**Should have been called**  
Spot on Name, Amazon Product Managers take note **Use this to**  
Track what people are doing inside of your app. **It's like**  
Flurry

### SNS

**Should have been called**  
Amazon Messenger **Use this to**  
Send mobile notifications, emails and/or SMS messages **It's like**  
UrbanAirship, Twilio

## Ops and Code Deployment Services

These are for automating how you manage and deploy your code onto other services.

### CodeCommit

**Should have been called**  
Amazon GitHub **Use this to**  
Version control your code - hosted Git. **It's like**  
Github, BitBucket

### Code Deploy

**Should have been called**  
Not bad **Use this to**  
Get your code from your CodeCommit repo (or Github) onto a bunch of EC2 instances in a sane way. **It's like**  
Heroku, Capistrano

### CodePipeline

**Should have been called**  
Amazon Continuous Integration **Use this to**  
Run automated tests on your code and then do stuff with it depending on if it passes those tests. **It's like**  
CircleCI, Travis

### EC2 Container Service

**Should have been called**  
Amazon Docker as a Service **Use this to**  
Put a Dockerfile into an EC2 instance so you can run a website.

### Elastic Beanstalk

**Should have been called**  
Amazon Platform as a Service **Use this to**  
Move your app hosted on Heroku to AWS when it gets too expensive. **It's like**  
Heroku, BlueMix, Modulus

## Enterprise / Corporate Services

Services for business and networks.

### AppStream

**Should have been called**  
Amazon Citrix **Use this to**  
Put a copy of a Windows application on a Windows machine that people get remote access to. **It's like**  
Citrix, RDP

### Direct Connect

**Should have been called**  
Pretty spot on actually **Use this to**  
Pay your Telco + AWS to get a dedicated leased line from your data center or network to AWS. Cheaper than Internet out for Data. **It's like**  
A toll road turnpike bypassing the crowded side streets.

### Directory Service

**Should have been called**  
Pretty spot on actually **Use this to**  
Tie together other apps that need a Microsoft Active Directory to control them.

### WorkDocs

**Should have been called**  
Amazon Unstructured Files **Use this to**  
Share Word Docs with your colleagues. **It's like**  
Dropbox, DataAnywhere

### WorkMail

**Should have been called**  
Amazon Company Email **Use this to**  
Give everyone in your company the same email system and calendar. **It's like**  
Google Apps for Domains

### Workspaces

**Should have been called**  
Amazon Remote Computer **Use this to**  
Gives you a standard windows desktop that you're remotely controlling.

### Service Catalog

**Should have been called**  
Amazon Setup Already **Use this to**  
Give other AWS users in your group access to preset apps you've built so they don't have to read guides like this.

### Storage Gateway

**Should have been called**  
S3 pretending it's part of your corporate network **Use this to**  
Stop buying more storage to keep Word Docs on. Make automating getting files into S3 from your corporate network easier.

## Big Data Services

Services to ingest, manipulate and massage data to do your will.

### Data Pipeline

**Should have been called**  
Amazon ETL **Use this to**  
Extract, Transform and Load data from elsewhere in AWS. Schedule when it happens and get alerts when they fail.

### Elastic Map Reduce

**Should have been called**  
Amazon Hadooper **Use this to**  
Iterate over massive text files of raw data that you're keeping in S3. **It's like**  
Treasure Data

### Glacier

**Should have been called**  
Really slow Amazon S3 **Use this to**  
Make backups of your backups that you keep on S3. Also, beware the cost of getting data back out in a hurry. For long term archiving.

### Kinesis

**Should have been called**  
Amazon High Throughput **Use this to**  
Ingest lots of data very quickly (for things like analytics or people retweeting Kanye) that you then later use other AWS services to analyze. **It's like**  
Kafka

### RedShift

**Should have been called**  
Amazon Data Warehouse **Use this to**  
Store a whole bunch of analytics data, do some processing, and dump it out.

### Machine Learning

**Should have been called**  
Skynet **Use this to**  
Predict future behavior from existing data for problems like fraud detection or "people that bought x also bought y."

### SWF

**Should have been called**  
Amazon EC2 Queue **Use this to**  
Build a service of "deciders" and "workers" on top of EC2 to accomplish a set task. Unlike SQS - logic is set up inside the service to determine how and what should happen. **It's like**  
IronWorker

### Snowball

**Should have been called**  
AWS Big Old Portable Storage **Use this to**  
Get a bunch of hard drives you can attach to your network to make getting large amounts (Terabytes of Data) into and out of AWS **It's like**  
Shipping a Network Attached Storage device to AWS

## AWS Management Services

AWS can get so difficult to manage that they invented a bunch of services to sell you to make it easier to manage.

### CloudFormation

**Should have been called**  
Amazon Services Setup **Use this to**  
Set up a bunch of connected AWS services in one go.

### CloudTrail

**Should have been called**  
Amazon Logging **Use this to**  
Log who is doing what in your AWS stack (API calls).

### CloudWatch

**Should have been called**  
Amazon Status Pager **Use this to**  
Get alerts about AWS services messing up or disconnecting. **It's like**  
PagerDuty, Statuspage

### Config

**Should have been called**  
Amazon Configuration Management **Use this to**  
Keep from going insane if you have a large AWS setup and changes are happening that you want to track.

### OpsWorks

**Should have been called**  
Amazon Chef **Use this to**  
Handle running your application with things like auto-scaling.

### Trusted Advisor

**Should have been called**  
Amazon Pennypincher **Use this to**  
Find out where you're paying too much in your AWS setup (unused EC2 instances, etc.).

### Inspector

**Should have been called**  
Amazon Auditor **Use this to**  
Scans your AWS setup to determine if you've setup it up in an insecure way **It's like**  
Alert Logic