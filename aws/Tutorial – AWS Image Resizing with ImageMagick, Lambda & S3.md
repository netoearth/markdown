This tutorial demonstrates how to use AWS Lambda for image resizing, image cropping and image processing – we'll use AWS S3 to store the images, and ImageMagick to resize them.

**Prerequisites:**

-   AWS CLI (See: [AWS CLI installation](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)).
-   Docker (See: [Docker installation](https://docs.docker.com/get-docker/)).
-   No other experience (we'll guide you through step-by-step).

**Full Working Example:**

-   [https://github.com/upload-io/aws-lambda-image-magick-resize-example](https://github.com/upload-io/aws-lambda-image-magick-resize-example)

**Programming Language & Frameworks:**

-   We'll use Node.js in this tutorial, but the examples can be adapted to any AWS Lambda-supported language, such as Python, Ruby, Java, C#, Go [and even PHP](https://aws.amazon.com/blogs/compute/introducing-the-new-serverless-lamp-stack/).
-   We'll avoid third-party ImageMagick wrappers to make the examples more portable.

## Part 1: Creating the AWS Lambda Boilerplate

If you already have an AWS Lambda function, you can skip forward to "part 2".

Otherwise, let's create our skeletal Lambda function...

### 1) Creating the AWS S3 Buckets

First, we'll create a CloudFormation stack for the S3 buckets we need:

**`buckets-cloudformation.yml:`**

```
Resources:

  # Contains our Lambda Function's code.
  LambdaFunctionCodeBucket: 
    Type: 'AWS::S3::Bucket'
    Properties:
      BucketName: my-lambda-function-code

  # Contains our images/photos.
  ImageBucket:
    Type: 'AWS::S3::Bucket'
    Properties:
      BucketName: my-images

```

___

**Remember:** replace `my-lambda-function-code` and `my-images` with unique bucket names.

___

Next, we'll create the stack:

```
aws cloudformation create-stack \
  --stack-name my-buckets \
  --template-body file://buckets-cloudformation.yml
```

### 2) Creating the AWS Lambda Function

To create a basic Lambda function, create the following files:

**`function.js:`**

```
module.exports.invoke = async () => {
  return "This will be your resized image...";
}
```

**`function-cloudformation.yml:`**

```
Resources:
  LambdaFunction:
    Type: 'AWS::Lambda::Function'
    Properties:
      FunctionName: AwsLambdaImageResizeExample
      Handler: function.invoke
      Runtime: nodejs14.x
      Role: !GetAtt LambdaFunctionRole.Arn
      MemorySize: 1024
      Code: 
        S3Bucket: my-lambda-function-code        # <--- CHANGE ME
        S3Key: AwsLambdaImageResizeExample.zip
  LambdaFunctionRole:
    Type: AWS::IAM::Role
    Properties:
      AssumeRolePolicyDocument:
        Version: '2012-10-17'
        Statement:
        - Effect: Allow
          Principal:
            Service:
            - lambda.amazonaws.com
          Action:
          - sts:AssumeRole
      Path: "/"
      Policies:
      - PolicyName: AppendToLogsPolicy
        PolicyDocument:
          Version: '2012-10-17'
          Statement:
          - Effect: Allow
            Action:
            - logs:CreateLogGroup
            - logs:CreateLogStream
            - logs:PutLogEvents
            Resource: "*"
      - PolicyName: S3FullAccessPolicy
        PolicyDocument:
          Version: '2012-10-17'
          Statement:
          - Effect: Allow
            Action: s3:*
            Resource:
            - arn:aws:s3:::my-images            # <--- CHANGE ME
            - arn:aws:s3:::my-images/*          # <--- CHANGE ME
```

___

**Remember:** replace `my-lambda-function-code` and `my-images` with your unique bucket names.

___

ZIP the function code:

```
zip -ry function-dist.zip .
```

Upload the ZIP file to S3:

```
aws s3 cp function-dist.zip s3://my-lambda-function-code/AwsLambdaImageResizeExample.zip
```

Deploy the stack:

```
aws cloudformation create-stack \
  --stack-name my-image-resize-lambda \
  --template-body file://function-cloudformation.yml \
  --capabilities CAPABILITY_NAMED_IAM
```

Check the status of the deployment:

```
aws cloudformation describe-stack-events \
  --stack-name my-image-resize-lambda 
```

___

**Important:** the latest event should say `"ResourceStatus": "CREATE_COMPLETE"`.

___

### 4) Invoking the AWS Lambda Function

Test the Lambda function we just created:

```
aws lambda invoke \
  --function-name AwsLambdaImageResizeExample \
  function-result.json
```

Now print the function's output:

```
cat function-result.json
```

It should read:

```
"This will be your resized image..."
```

_Note: we've used the AWS CLI to invoke our Lambda function. To invoke via HTTP, you'll need to create a  `AWS::ApiGatewayV2::Integration` – there are many tutorials for this online – just search around!_

## Part 2: Calling ImageMagick from AWS Lambda

If you've been following part 1, your working directory will now contain the following:

```
/
  buckets-cloudformation.yml
  function.js
  function-cloudformation.yml
  function-dist.zip
  function-result.json
```

Now we'll install `/imagemagick` as a subdirectory...

### 1) Installing ImageMagick on AWS Lambda

Firstly, please complete our [tutorial on building ImageMagick for Amazon Linux 2](https://upload.io/blog/installing-imagemagick-on-amazon-linux-2/).

On completing this tutorial, you'll have an ImageMagick directory containing the following:

```
bash-4.2# ls -la
total 28
drwxr-xr-x 7 root root 4096 May  2 11:04 .
dr-xr-x--- 1 root root 4096 May  2 11:04 ..
drwxr-xr-x 2 root root 4096 May  2 11:04 bin
drwxr-xr-x 3 root root 4096 May  2 11:04 etc
drwxr-xr-x 3 root root 4096 May  2 11:04 include
drwxr-xr-x 4 root root 4096 May  2 11:04 lib
drwxr-xr-x 3 root root 4096 May  2 11:04 share
```

Move this entire directory to `/imagemagick`, giving you:

```
/
  imagemagick/
    bin/
    etc/
    include/
    lib/
    share/
  buckets-cloudformation.yml
  function.js
  function-cloudformation.yml
  function-dist.zip
  function-result.json
```

### 2) Installing the S3 Client on AWS Lambda

Before we start resizing images, we'll need to install `@aws-sdk/client-s3` on our Lambda function.

This is so we can download the original photos from AWS S3 to generate image thumbnails for them on request:

```
npm install @aws-sdk/client-s3@3
```

### 3) Writing the AWS Lambda Image Resize Code

To start resizing images on our AWS Lambda function, we'll need update our code to the following:

**`function.js:`**

```
const { execFile } = require("child_process");
const { S3 }       = require("@aws-sdk/client-s3");
const fs           = require("fs");
const path         = require("path");
const localFile    = path.resolve("/tmp/original-image.jpg");
const s3           = new S3();
const s3Key        = "original-image.jpg";

//
// Remember to change this!
//
const imageBucketName = "my-images"

module.exports.invoke = async () => {
  await downloadFile(s3Key, localFile);
  await resizeImage(localFile);
  return await sendFileAsResponse(localFile);
}

async function downloadFile(s3Key, localFile) {
  const image = await s3.getObject({
    Bucket: imageBucketName,
    Key: s3Key
  });
  await new Promise((resolve, reject) => {
    const writer = fs.createWriteStream(localFile);
    writer.on("close", resolve);
    writer.on("error", reject);
    image.Body.pipe(writer);
  });
}

async function resizeImage(localFile) {
  await new Promise((resolve, reject) => {
    execFile(
      path.resolve("imagemagick/bin/magick"), 
      [
        localFile,
        "-resize",
        "100x100",
        localFile
      ], 
      (error, stdout, stderr) => {
        if (error !== null) {
          reject(error);
        } else {
          resolve();
        }
      }
    );
  });
}

async function sendFileAsResponse(localFile) {
  return {
    isBase64Encoded: true,
    statusCode: 200,
    headers: { "content-type": "image/jpg"},
    body: (await fs.promises.readFile(localFile)).toString('base64')
  }
}
```

Now redeploy the function:

```
zip -ry function-dist.zip .

aws s3 cp \
  function-dist.zip \
  s3://my-lambda-function-code/AwsLambdaImageResizeExample.zip

aws lambda update-function-code \
    --function-name AwsLambdaImageResizeExample \
    --s3-bucket my-lambda-function-code \
    --s3-key AwsLambdaImageResizeExample.zip
```

### 4) Try It Out!

Firstly, upload a test image:

```
aws s3 cp original-image.jpg s3://my-images/original-image.jpg
```

Finally, resize the image using the AWS Lambda function:

```
aws lambda invoke \
  --function-name AwsLambdaImageResizeExample \
  function-result.json
```

Now print the result:

```
cat function-result.json
```

You'll see your resized image, base64-encoded in the `body` field:

```
{
  "isBase64Encoded": true,
  "statusCode": 200,
  "headers": {
    "content-type": "image/jpg"
  },
  "body": "/9j/4AAQSkZJRg...9k="
}
```

This is because AWS Lambda only supports JSON responses: to return the raw image, you need to [put API Gateway in front of the Lambda function](https://upload.io/blog/cloudformation-lambda-examples/).

## AWS Lambda Image Resize Example – Code Repository

See a full working example of running ImageMagick on AWS Lambda to resize images below:

[https://github.com/upload-io/aws-lambda-image-magick-resize-example](https://github.com/upload-io/aws-lambda-image-magick-resize-example)

## Final Thoughts

To summarise, we've managed to build our own AWS image processing service by:

1.  Creating an AWS Lambda function.
2.  Installing ImageMagick on it.
3.  Uploading an image/photo to S3.
4.  Using ImageMagick to resize the image (within the Lambda function).
5.  Returning the resized image as a base64-encoded JSON response.

The crux of this tutorial has been building a standalone ImageMagick binary that's compatible with Amazon Linux, and uploading it as part of the Lambda function's ZIP file. Once you've done this, you can simply `exec` the ImageMagick binary from within your Lambda function.

We hope you've enjoyed this tutorial on AWS Lambda image processing, and look forward to having you back again soon!