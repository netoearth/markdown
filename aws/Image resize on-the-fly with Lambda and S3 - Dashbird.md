Handling large images has always been a pain in my side since I started writing code. Lately, it has started to have a huge impact on page speed and SEO ranking. If your website has poorly optimized images it won’t score well on [Google Lighthouse](https://developers.google.com/web/tools/lighthouse/). If it doesn’t score well, it won’t be on the first page of Google. That sucks.

![](https://dashbird.io/wp-content/uploads/2020/10/googlepage2.gif)

#### TL;DR

I’ve built and open-sourced a snippet of code that automates the process of creating and deploying an image resize function and an S3 bucket with one simple command. Check out [the code here](https://github.com/adnanrahic/serverless-docker-image-resize).

But if you want to follow along and learn how to do it yourself, keep reading.

### Where do we start?

Luckily enough, there is a way to solve the issue of bad image optimization with little to no hassle. Today we’ll build an [AWS Lambda](https://aws.amazon.com/lambda/) function to resize images on-the-fly.

The images will be stored in an [S3](https://aws.amazon.com/s3/) bucket and, once requested, will be served from it. If you need a resized version, you’ll request the image and supply the height and width. This will trigger a function. It’ll grab the existing image, resize it, return it back to the bucket and then serve it from the bucket.

This scenario will only resize an image for a given set of dimensions once. Every subsequent request to the image of that size will be served from the bucket. Pretty cool huh? Here’s a diagram, because who doesn’t love diagrams.

![](https://dashbird.io/wp-content/uploads/2020/10/diagram.png)

Because I already assume you know how to use the Serverless Framework and have already been introduced to the basics of [serverless](https://sls.dashbird.io/en/serverless-best-practices), [Docker](https://www.docker.com/) and [AWS](https://aws.amazon.com/), I’ll immediately jump into the gist of things. Here’s an overview of what we’ll be doing.

-   Create the project structure
-   Create a secrets file
-   Write the AWS Lambda function configuration
-   Write AWS Lambda function source code
-   Write S3 Bucket configuration
-   Deploy with Docker
-   Test with [Dashbird](https://dashbird.io/)

_**Note**: Please install [Docker](https://docs.docker.com/install/) and [Docker Compose](https://docs.docker.com/compose/install/) before continuing with this tutorial._

The funny thing here is that we need to use Docker to deploy this service because of [Sharp](https://github.com/lovell/sharp). This image-resize module has binaries that need to be built on the same operating system as it will run on. Because AWS Lambda is running on Amazon Linux, we need to `npm install` the packages on an Amazon Linux instance before running `sls deploy`.

With that all out of the way let’s build some stuff.

### Create the project structure

The best way of explaining this complex structure will be with an image.

![project structure](https://dashbird.io/wp-content/uploads/2020/10/image-resize-on-the-fly-diagram.png)

Folders in green and files in blue. Check out the [repo](https://github.com/adnanrahic/serverless-docker-image-resize) for further insight.

### Create a secrets file

We’ll create a `secrets.env` file in the **secrets** folder to hold our secrets and inject them into our Docker container once it spins up.

```
SLS_KEY=XXX # replace with your IAM User key
SLS_SECRET=YYY # replace with your IAM User secret
STAGE=dev
REGION=us-east-1
BUCKET=images.yourdomain.com
```

_**Note**: The `deploy.sh` script will create a secondary `secrets.json` file only to hold the domain name of our API Gateway endpoint._

### Write the AWS Lambda function configuration

Open up the **functions** folder, and start by creating the `package.json` file. Paste this in.

```
{
  "name": "image-resize",
  "version": "1.0.0",
  "description": "Serverless image resize on-the-fly.",
  "main": "resize.js",
  "dependencies": {
    "serverless-plugin-tracing": "^2.0.0",
    "sharp": "^0.18.4"
  }
}
```

No need to run any install, because we need to run the Docker container first. Note that we’re adding a tracing plugin to enable [X-Ray](https://aws.amazon.com/xray/) because it’s awesome.

Moving on, let’s create the `serverless.yml` file to configure our function.

```
service: image-resize-on-the-fly-functions

plugins:
  - serverless-plugin-tracing

provider:
  name: aws
  runtime: nodejs8.10
  stage: ${env:STAGE, 'dev'}
  profile: serverless-admin
  tracing: true
  region: ${env:REGION, 'us-east-1'}
  environment:
    BUCKET: ${env:BUCKET}
    REGION: ${env:REGION}
  iamRoleStatements:
    - Effect: "Allow"
      Action:
        - "s3:ListBucket"
      Resource: "arn:aws:s3:::${env:BUCKET}"
    - Effect: "Allow"
      Action:
        - "s3:PutObject"
      Resource: "arn:aws:s3:::${env:BUCKET}"
    - Effect: "Allow"
      Action:
        - "xray:PutTraceSegments"
        - "xray:PutTelemetryRecords"
      Resource:
        - "*"

functions:
  resize:
    handler: resize.handler
    events:
      - http:
          path: resize
          method: get
```

As you can see we’re grabbing a bunch of values from the `env` and also allowing the function to access the `BUCKET` we specified and some X-Ray telemetry.

### Write AWS Lambda function source code

Only the code left now. Here’s what we’ll do. Because images can get quite large and we don’t want to risk loading several megabytes into the lambda function’s memory, I’ll show you how to use [Node.js streams](https://nodejs.org/dist/latest-v8.x/docs/api/stream.html) to read the image from an S3 stream, pipe it to Sharp, and then write it back to S3, once again, as a stream.

Ready? Let’s do it in two steps. First, define the helper functions we need in order to create the streams, then create the handler function itself which our lambda will invoke.

```
// resize.js

// require modules
const stream = require('stream')
const AWS = require('aws-sdk')
const S3 = new AWS.S3({
  signatureVersion: 'v4'
})
const sharp = require('sharp')

// create constants
const BUCKET = process.env.BUCKET
const URL = `http://${process.env.BUCKET}.s3-website.${process.env.REGION}.amazonaws.com`

// create the read stream abstraction for downloading data from S3
const readStreamFromS3 = ({ Bucket, Key }) => {
  return S3.getObject({ Bucket, Key }).createReadStream()
}
// create the write stream abstraction for uploading data to S3
const writeStreamToS3 = ({ Bucket, Key }) => {
  const pass = new stream.PassThrough()
  return {
    writeStream: pass,
    uploadFinished: S3.upload({
      Body: pass,
      Bucket,
      ContentType: 'image/png',
      Key
    }).promise()
  }
}
// sharp resize stream
const streamToSharp = ({ width, height }) => {
  return sharp()
    .resize(width, height)
    .toFormat('png')
}
```

Here we’re defining constants and the constructor functions for our streams. What’s important to note is that S3 doesn’t have a default method for writing to storage with a stream. So you need to create one. It’s doable with the `stream.PassThrough()` helper. The abstraction above will write the data with a stream and resolve a promise once it’s done.

Moving on, let’s check out the handler. Just underneath the code above, paste this in.

```
exports.handler = async (event) => {
  const key = event.queryStringParameters.key
  // match a string like: '/1280x720/image.jpg'
  const match = key.match(/(\d+)x(\d+)\/(.*)/)
  // get the dimensions of the new image
  const width = parseInt(match[1], 10)
  const height = parseInt(match[2], 10)
  const originalKey = match[3]
  // create the new name of the image, note this has a '/' - S3 will create a directory
  const newKey = '' + width + 'x' + height + '/' + originalKey
  const imageLocation = `${URL}/${newKey}`

  try {
    // create the read and write streams from and to S3 and the Sharp resize stream
    const readStream = readStreamFromS3({ Bucket: BUCKET, Key: originalKey })
    const resizeStream = streamToSharp({ width, height })
    const { 
      writeStream, 
      uploadFinished 
    } = writeStreamToS3({ Bucket: BUCKET, Key: newKey })

    // trigger the stream
    readStream
      .pipe(resizeStream)
      .pipe(writeStream)

    // wait for the stream to finish
    const uploadedData = await uploadFinished

    // log data to Dashbird
    console.log('Data: ', {
      ...uploadedData,
      BucketEndpoint: URL,
      ImageURL: imageLocation
    })

    // return a 301 redirect to the newly created resource in S3
    return {
      statusCode: '301',
      headers: { 'location': imageLocation },
      body: ''
    }
  } catch (err) {
    console.error(err)
    return {
      statusCode: '500',
      body: err.message
    }
  }
}
```

The query string parameters will look like this `/1280x720/image.jpg`. Which means we grab the image dimensions from the parameters by using a regex match. The `newKey` value is set to the dimensions followed by a `/` and the original name of the image. This will create a new folder named `1280x720` with the image `image.jpg` in it. Pretty cool.

Once we trigger the streams and await the promise to resolve we can log out the data and return a 301 redirect to the location of the image in the bucket. Note that we will enable static website hosting on our bucket in the next paragraph.

##Write S3 Bucket configuration

The bucket configuration will mostly consist of default [CloudFormation](https://aws.amazon.com/cloudformation/) templates, so if you’re a bit rusty, I’d advise you brush up a bit. But, the gist of it is simple enough to understand. Open up the **bucket** folder and create a `serverless.yml` file.

```
service: image-resize-on-the-fly-bucket

custom:
  secrets: ${file(../secrets/secrets.json)} # will be created in our deploy.sh script

provider:
  name: aws
  runtime: nodejs8.10
  stage: ${env:STAGE, 'dev'}
  profile: serverless-admin
  region: us-east-1
  environment:
    BUCKET: ${env:BUCKET}

resources:
  Resources:
    ImageResizeOnTheFly:
      Type: AWS::S3::Bucket # creating an S3 Bucket
      Properties:
        AccessControl: PublicReadWrite
        BucketName: ${env:BUCKET}
        WebsiteConfiguration: # enabling the static website option
          ErrorDocument: error.html
          IndexDocument: index.html
          RoutingRules: # telling it to redirect for every 404 error
            - 
              RedirectRule:
                HostName: ${self:custom.secrets.DOMAIN} # API Gateway domain
                HttpRedirectCode: "307" # temporary redirect HTTP code
                Protocol: "https"
                ReplaceKeyPrefixWith: "${self:provider.stage}/resize?key=" # route
              RoutingRuleCondition:
                HttpErrorCodeReturnedEquals: "404"
                KeyPrefixEquals: ""
    ImageResizeOnTheFlyPolicy:
      Type: AWS::S3::BucketPolicy # add policy for public read access
      Properties: 
        Bucket: 
          Ref: ImageResizeOnTheFly
        PolicyDocument: 
          Statement: 
            - 
              Action: 
                - "s3:*"
              Effect: "Allow"
              Resource:
                Fn::Join: 
                  - ""
                  - 
                    - "arn:aws:s3:::"
                    - 
                      Ref: ImageResizeOnTheFly
                    - "/*"
              Principal: "*"
```

Once we enable the static website hosting option on our bucket, it will behave like any website. This lets us serve images from it and utilize the 404 error redirect rule.

If no image is found, the bucket will trigger the 404, which redirects to our lambda function. Then resize the original image to the requested dimensions, and put it back to the bucket at the exact route it was requested from.

Just like magic!

### Deploy with Docker

Here comes the fun part! We’ll create a `Dockerfile` and `docker-compose.yml` file to create our Amazon Linux container and load it with `.env` values. That’s easy. The hard part will be writing the bash script to run all commands and deploy our function and bucket.

Starting with the `Dockerfile`, here’s what you need to add.

```
FROM amazonlinux

# Create deploy directory
WORKDIR /deploy

# Install system dependencies
RUN yum -y install make gcc*
RUN curl --silent --location https://rpm.nodesource.com/setup_8.x | bash -
RUN yum -y install nodejs

# Install serverless
RUN npm install -g serverless

# Copy source
COPY . .

# Install app dependencies
RUN cd /deploy/functions && npm i --production && cd /deploy

#  Run deploy script
CMD ./deploy.sh ; sleep 5m
```

Because Amazon Linux is rather basic, we need to install both gcc and Node.js at the get-go. Then it’s as simple as any Dockerfile you’ve seen. Install the Serverless Framework globally, copy over the source code, install the npm modules in the **functions** directory and run the `deploy.sh` script.

The `docker-compose.yml` file is literally only used to load the `.env` values.

```
version: "3"
services:
  image-resize-on-the-fly:
    build: .
    volumes:
      - ./secrets:/deploy/secrets
    env_file:
      - ./secrets/secrets.env
```

That’s it. The Docker part is done. Let’s write some bash.

Starting out simple, we’ll define the initial variables, configure our serverless installation and deploy our function.

```
# deploy.sh

# variables
stage=${STAGE}
region=${REGION}
bucket=${BUCKET}
secrets='/deploy/secrets/secrets.json'

# Configure your Serverless installation to talk to your AWS account
sls config credentials \
  --provider aws \
  --key ${SLS_KEY} \
  --secret ${SLS_SECRET} \
  --profile serverless-admin

# cd into functions dir
cd /deploy/functions

# Deploy function
echo "------------------"
echo 'Deploying function...'
echo "------------------"
sls deploy
```

Once we have deployed it, we need to grab the domain name of our API Gateway endpoint and put it in a `secrets.json` file which will be loaded into our `serverless.yml` file from the **bucket** directory. To do this I just did some regex magic. Add this at the bottom of your `deploy.sh`.

```
# find and replace the service endpoint
if [ -z ${stage+dev} ]; then echo "Stage is unset."; else echo "Stage is set to '$stage'."; fi

sls info -v | grep ServiceEndpoint > domain.txt
sed -i 's@ServiceEndpoint:\ https:\/\/@@g' domain.txt
sed -i "s@/$stage@@g" domain.txt
domain=$(cat domain.txt)
sed "s@.execute-api.$region.amazonaws.com@@g" domain.txt > id.txt
id=$(cat id.txt)

echo "------------------"
echo "Domain:"
echo "  $domain"
echo "------------------"
echo "API ID:"
echo "  $id"

rm domain.txt
rm id.txt

echo "{\"DOMAIN\":\"$domain\"}" > $secrets
```

Now you have a `secrets.json` file in the **secrets** directory. All that’s left is to run the bucket deployment. Paste this final snippet at the bottom of the `deploy.sh` script.

```
cd /deploy/bucket

# Deploy bucket config
echo "------------------"
echo 'Deploying bucket...'
sls deploy

echo "------------------"
echo 'Bucket endpoint:'
echo "  http://$bucket.s3-website.$region.amazonaws.com/"

echo "------------------"
echo "Service deployed. Press CTRL+C to exit."
```

There we have it! The coding part is done. Would you believe me we only need to run one command now? Open up a terminal window in the root of your project and run:

```
$ docker-compose up --build
```

Let it do its magic, and you’ll see everything get created automagically! Please note that the terminal will show you the bucket endpoint which you’ll use to access the images.

### Test with Dashbird

The final step is to check if it all works. Let’s upload an image to the bucket, so we have something to resize. Go ahead and find an image you like and want to resize, or just take [this one](https://github.com/adnanrahic/cdn/raw/master/image-resize-on-the-fly/the-earth.jpg).

It’s rather huge and around 6 MB in size. Here’s the command you want to run to upload it.

```
$ aws s3 cp --acl public-read IMAGE s3://BUCKET
```

So, if your bucket name is **images** and your image name is **the-earth.jpg**, then it should look like this if you run the command from the directory where the image is located.

```
$ aws s3 cp --acl public-read the-earth.jpg s3://images
```

Keep in mind you need to have the [AWS CLI](https://aws.amazon.com/cli/) installed on your machine, or upload the image through the AWS Console.

Now, try out to request the image through the bucket. Type into your browser the S3 bucket URL.

```
http://BUCKET.s3-website.REGION.amazonaws.com/the-earth.jpg
```

You’ll see the original image appear. But, now add dimensions to the URL.

```
http://BUCKET.s3-website.REGION.amazonaws.com/400x400/the-earth.jpg
```

This will trigger the resize function and create a 400×400 pixel version of the image. It’ll take a few hundred milliseconds to create and once it’s done, the browser will redirect you to the newly created image. When you try to refresh the URL you’ll see that it’s now fetching the new image, without calling the resize function.

Let’s check the logs in [Dashbird](https://dashbird.io/features/) and make sure it all works fine under the hood.

![invocation log and traces](https://dashbird.io/wp-content/uploads/2020/10/dashbird-invocation.gif)

Looks fine, but I did make some beginner mistakes when I first tried setting up the function. One of them was forgetting to require a module before using it. Luckily I immediately got an alert that explained what was wrong. [Slack alerts](https://dashbird.io/docs/user-guide/alerting/) are life-savers.

![slack alert](https://dashbird.io/wp-content/uploads/2020/10/slack-alert.png)

The way I fixed the issue was with the [live-tailing](https://dashbird.io/docs/user-guide/debugging/#live-tailing) feature. It let me check the invocation logs with a few seconds of latency so I could debug the issue. Pretty cool.

![live debugging](https://dashbird.io/wp-content/uploads/2020/10/dashbird-live-tailing.gif)

### Wrapping up

To conclude this little _showdev_ session, I’d like to point out using serverless as a helper to support your existing infrastructure is incredible. It’s language agnostic and easy to use.

[We at Dashbird](https://dashbird.io/about-us/) use clusters of containers for our core features that interact heavily with our databases while offloading all the rest to lambda functions, queues, streams and other serverless services on AWS.

And, of course, [here’s the repo](https://github.com/adnanrahic/serverless-docker-image-resize) once again, give it a star if you want more people to see it on GitHub. If you follow along with the instructions there, you’ll be able to have this image resize on-the-fly microservice up and running in no time at all.

Or, take a look at a few of my articles right away:

-   [Containers vs. Serverless from a DevOps standpoint](https://dashbird.io/blog/containers-vs-serverless-from-a-devops-standpoint/)
-   [A crash course on serverless-side rendering with Vue.js, Nuxt.js and AWS Lambda](https://dashbird.io/blog/a-crash-course-on-serverless-side-rendering-with-vuejs-nuxtjs-and-aws-lambda)
-   [Building a serverless contact form with AWS Lambda and AWS SES](https://dashbird.io/blog/building-a-serverless-contact-form-with-aws-lambda-and-aws-ses)
-   [A crash course on Serverless APIs with Express and MongoDB](https://dashbird.io/blog/a-crash-course-on-serverless-apis-with-express-and-mongodb/)
-   [Solving invisible scaling issues with Serverless and MongoDB](https://dashbird.io/blog/solving-invisible-scaling-issues-with-serverless-and-mongodb/)
-   [How to deploy a Node.js application to AWS Lambda using Serverless](https://dashbird.io/blog/how-to-deploy-a-nodejs-application-to-aws-lambda-using-serverless/)
-   [Getting started with AWS Lambda and Node.js](https://dashbird.io/blog/getting-started-with-aws-lambda-and-nodejs)

_I had an absolute blast writing this open-source code snippet. Writing the article wasn’t that bad either! Hope you guys and girls enjoyed reading it as much as I enjoyed writing it. If you liked it, drop a comment below. Until next time, be curious and have fun._

___

_We aim to improve [Dashbird](https://dashbird.io/features/) every day and user feedback is extremely important for that, so [please let us know](mailto:support@dashbird.io) if you have any feedback about these improvements and new features! We would really appreciate it!_

## Read our blog

![](https://dashbird.io/wp-content/uploads/2022/09/how_to_monitor_AWS_opensearch.png)

### Why and How to Monitor Amazon OpenSearch Service

One of the most vital aspects to monitor is the metrics. You should know how your cluster performs and if it can keep up with the traffic. Learn more about monitoring Amazon OpenSearch Service.

![](https://dashbird.io/wp-content/uploads/2022/08/aws-lambda-metrics.png)

### Why and How to Monitor AWS Elastic Load Balancing

Dashbird recently added support for ELB, so now you can keep track of your load balancers in one central place. It comes with all the information you expect from AWS monitoring services and more!

![](https://dashbird.io/wp-content/uploads/2022/08/aws-lambda-metrics-3.png)

### Monitor Your AWS AppSync GraphQL APIs with Simplicity

Dashbird has just added support for AppSync to help you monitor all of your AppSync endpoints without needing to browse dozens of logs or stumble through traces in the X-Ray UI.

[More articles](https://dashbird.io/category/lambda/)