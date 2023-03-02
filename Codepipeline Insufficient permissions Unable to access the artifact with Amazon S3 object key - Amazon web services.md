![](https://www.gravatar.com/avatar/eff5fe91b5e3889b2642df1da0c46ee8?s=64&d=identicon&r=PG&f=1) Author: Arleen Jimenez Date: 2022-07-28

Uploading a file to S3 bucket using low-level API: Uploading a file to S3 bucket using high-level API: Solution 4: @mejiamanuel57's solution works fine for small files under 15MB. This is a great solution using the sync feature, because it compares all your local files against the S3 bucket and only copies files that have changed (by comparing file sizes and checksums).

-   [Codepipeline: Insufficient permissions Unable to access the artifact with Amazon S3 object key](https://copyprogramming.com/howto/codepipeline-insufficient-permissions-unable-to-access-the-artifact-with-amazon-s3-object-key#codepipeline-insufficient-permissions-unable-to-access-the-artifact-with-amazon-s3-object-key)
-   [How to upload a file to amazon S3 super easy using c#](https://copyprogramming.com/howto/codepipeline-insufficient-permissions-unable-to-access-the-artifact-with-amazon-s3-object-key#how-to-upload-a-file-to-amazon-s3-super-easy-using-c)
-   [Jenkins Continuous Integration with Amazon S3](https://copyprogramming.com/howto/codepipeline-insufficient-permissions-unable-to-access-the-artifact-with-amazon-s3-object-key#jenkins-continuous-integration-with-amazon-s3)

## Codepipeline: Insufficient permissions Unable to access the artifact with Amazon S3 object key

**Question:**

Hello I created a codepipeline project with the following configuration:

-   Source Code in S3 pulled from Bitbucket.
-   Build with CodeBuild, generating an [docker image](https://copyprogramming.com/howto/can-i-put-my-docker-repository-image-on-github-bitbucket "Can I put my docker repository/image on GitHub/Bitbucket?") and storing it into a [Amazon ECS](https://copyprogramming.com/howto/run-docker-image-on-amazon-ecs "Run docker image on amazon ecs") repository.
-   Deployment provider Amazon ECS.

All the process works ok until when it tries to deploy, for some reason I am getting the following error during deployment:

> Insufficient permissions Unable to access the artifact with Amazon S3 [object key](https://copyprogramming.com/howto/unable-to-access-the-object-key "Unable to access the object key") 'FailedScanSubscriber/MyAppBuild/Wmu5kFy' located in the Amazon S3 artifact bucket 'codepipeline-us-west-2-913731893217'. The provided role does not have sufficient permissions.

During the building phase, it is even able to create a new docker image in the ECS repository.

I tried everything, changed IAM roles and policies, add full access to S3, I have even setted the [S3 bucket](https://copyprogramming.com/howto/run-shell-command-on-s3-bucket "Run shell command on S3 bucket?") as public, nothing worked. I am without options, if someone could help me that would be wonderful, I have poor experience with AWS, so any help is appreciated.

**Solution 1:**

I was able to find a solution. The true issue is that when the deployment provider is set as Amazon ECS, we need to generate an output artifact indicating the name of the task definition and the image uri, for example:

```
post_build:
    commands:
      - printf '[{"name":"your.task.definition.name","imageUri":"%s"}]' $AWS_ACCOUNT_ID.dkr.ecr.$AWS_DEFAULT_REGION.amazonaws.com/$IMAGE_REPO_NAME:$IMAGE_TAG > imagedefinitions.json
artifacts:
    files: imagedefinitions.json
```

**Solution 2:**

This happens when AWS CodeDeploy cannot find the build artifact from AWS CodeBuild. If you go into the S3 bucket and check the path you would actually see that the artifact object is NOT THERE!

Even though the error says about a permission issue. This can happen due the absent of the artifact object.

**Solution:** Properly configure `artifacts` section in `buildspec.yml` and configure AWS [codepipeline stages](https://copyprogramming.com/howto/how-can-i-skip-a-codepipeline-stage "How can I skip a codepipeline stage?") properly specifying input and output [artifact names](https://copyprogramming.com/howto/gitlab-ci-dynamic-artifact-names "Gitlab-ci: Dynamic artifact names").

```
artifacts:
  files:
    - '**/*'
  base-directory: base_dir
  name: build-artifact-name
  discard-paths: no
```

**Refer this article -** https://medium.com/@shanikae/insufficient-permissions-unable-to-access-the-artifact-with-[amazon-s3](https://copyprogramming.com/howto/how-to-access-private-objects-from-amazon-s3-server-in-laravel "How to access private objects from amazon s3 server in laravel")\-247f27e6cdc3

**Solution 3:**

For me the issue was that my CodeBuild step was encrypting the artifacts using the Default AWS Managed S3 key.

My Deploy step uses a [Cross-Account](https://copyprogramming.com/howto/s3-to-s3-cross-account-data-replication "S3 to S3 Cross account data replication") role, and so it couldn't retrieve the artifact. Once I changed the Codebuild encryption key to my CMK as it should've been originally, my deploy step succeeded.

What is the difference between Amazon ECS and, To simplify it further, if you have launched an Amazon ECS with no EC2 instances added to it, it's good for nothing i.e. you can't do anything about … Code sample#!/bin/bashecho ECS\_CLUSTER=your\_cluster\_name >> /etc/ecs/ecs.configFeedback

### Inverted Color Amazon Fire Tablet Fix (Washed Out

Fix the Inverted Color With Amazon Fire Tablet where the colors of the display becomes washed out, looks faded or like a negative film with black becoming wh

<iframe loading="lazy" width="560" height="315" data-src="https://www.youtube.com/embed/7FSQPLdxJSA" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen=""></iframe>

### 2021 New Tucson 1.6 T-GDI Hybrid 230 hp Silky

This is the new Hyundai Tucson 2021 first look. 360 exterior shot and interior closeups. Bit shaky because I didn't have the gimbal with me. Had to take some

<iframe loading="lazy" width="560" height="315" data-src="https://www.youtube.com/embed/TjUV_omxAtQ" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen=""></iframe>

### The best trick to CLEAN BRONZE

Easy with only 3 cooking ingredients, IN FEW MINUTES !!! how to clean bronze without chemicals, household cleaning paste for copper, metals, bronze , brass. L

<iframe loading="lazy" width="560" height="315" data-src="https://www.youtube.com/embed/MaQ1Vr2zHk0" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen=""></iframe>

## How to upload a file to amazon S3 super easy using c#

**Question:**

I am tired of all these "upload to S3" examples and tutorials that don't work , can someone just show me an example that simply works and is super easy?

**Solution 1:**

well here are the instruction that you have to follow to get a fully working demo program ...

1-Download and install the Amazon web services SDK for .NET which you can find in (http://aws.amazon.com/sdk-for-net/). because I have visual studio 2010 I choose to install the [3.5](https://copyprogramming.com/howto/where-is-the-net-3-5-sdk "Where is the .net 3.5 SDK?") .NET SDK.

2- open visual studio and make a new project , I have visual studio 2010 and I am using a console application project.

3- add reference to AWSSDK.dll , it is installed with the Amazon web service SDK mentioned above , in my system the dll is located in "C:\\Program Files (x86)\\AWS SDK for .NET\\bin\\Net35\\AWSSDK.dll".

4- make a new class file ,call it "AmazonUploader" here the complete code of the class:

```
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using Amazon;
using Amazon.S3;
using Amazon.S3.Transfer;
namespace UploadToS3Demo
{
    public class AmazonUploader
    {
        public bool sendMyFileToS3(string localFilePath, string bucketName, string subDirectoryInBucket, string fileNameInS3)
        {
        // input explained :
        // localFilePath = the full local file path e.g. "c:\mydir\mysubdir\myfilename.zip"
        // bucketName : the name of the bucket in S3 ,the bucket should be alreadt created
        // subDirectoryInBucket : if this string is not empty the file will be uploaded to
            // a subdirectory with this name
        // fileNameInS3 = the file name in the S3
        // create an instance of IAmazonS3 class ,in my case i choose RegionEndpoint.EUWest1
        // you can change that to APNortheast1 , APSoutheast1 , APSoutheast2 , CNNorth1
        // SAEast1 , USEast1 , USGovCloudWest1 , USWest1 , USWest2 . this choice will not
        // store your file in a different cloud storage but (i think) it differ in performance
        // depending on your location
        IAmazonS3 client = Amazon.AWSClientFactory.CreateAmazonS3Client(RegionEndpoint.EUWest1);
        // create a TransferUtility instance passing it the IAmazonS3 created in the first step
        TransferUtility utility = new TransferUtility(client);
        // making a TransferUtilityUploadRequest instance
        TransferUtilityUploadRequest request = new TransferUtilityUploadRequest();
        if (subDirectoryInBucket == "" || subDirectoryInBucket == null)
        {
            request.BucketName = bucketName; //no subdirectory just bucket name
        }
        else
        {   // subdirectory and bucket name
            request.BucketName = bucketName + @"/" + subDirectoryInBucket;
        }
        request.Key = fileNameInS3 ; //file name up in S3
        request.FilePath = localFilePath; //local file name
        utility.Upload(request); //commensing the transfer
        return true; //indicate that the file was sent
    }
  }
}
```

5- add a configuration file : right click on your project in the solution explorer and choose "add" -> "new item" then from the list choose the type "Application configuration file" and click the "add" button. a file called "App.config" is added to the solution.

6- edit the app.config file : double click the "app.config" file in the solution explorer the edit menu will appear . replace all the text with the following text :

```
<?xml version="1.0" encoding="utf-8" ?>
<configuration>
  <appSettings>
    <add key="AWSProfileName" value="profile1"/>
    <add key="AWSAccessKey" value="your Access Key goes here"/>
    <add key="AWSSecretKey" value="your Secret Key goes here"/>
  </appSettings>
</configuration>
```

you have to modify the above text to reflect your Amazon Access Key Id and Secret Access Key.

7- now in the program.cs file (remember this is a console application) write the following code :

```
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
namespace UploadToS3Demo
{
    class Program
    {
        static void Main(string[] args)
        {
            // preparing our file and directory names
            string fileToBackup = @"d:\mybackupFile.zip" ; // test file
            string myBucketName = "mys3bucketname"; //your s3 bucket name goes here
            string s3DirectoryName = "justdemodirectory";
            string s3FileName = @"mybackupFile uploaded in 12-9-2014.zip";
            AmazonUploader myUploader = new AmazonUploader();
            myUploader.sendMyFileToS3(fileToBackup, myBucketName, s3DirectoryName, s3FileName);
        }
    }
}
```

8- replace the strings in the code above with your own data

9- add error correction and your program is ready

**Solution 2:**

The solution of @docesam is for an old version of AWSSDK. **Here is an example with the latest documentation of AmazonS3:**

1.  First open Visual Studio (I'm using VS2015) and create a New Project **\->** ASP.NET Web Application **\->** MVC.
    
2.  Browse in Manage Nuget Package , the package **AWSSDK.S3** and install it.
    
3.  Now create a class named `AmazonS3Uploader` , then copy and paste this code:
    
    ```
    using System;
    using Amazon.S3;
    using Amazon.S3.Model;
    namespace AmazonS3Demo
    {
     public class AmazonS3Uploader
     {
         private string bucketName = "your-amazon-s3-bucket";
         private string keyName = "the-name-of-your-file";
         private string filePath = "C:\\Users\\yourUserName\\Desktop\\myImageToUpload.jpg";
         public async void UploadFile()
         {
             var client = new AmazonS3Client(Amazon.RegionEndpoint.USEast1);
             try
             {
                 PutObjectRequest putRequest = new PutObjectRequest
                 {
                     BucketName = bucketName,
                     Key = keyName,
                     FilePath = filePath,
                     ContentType = "text/plain"
                 };
                 PutObjectResponse response = await client.PutObjectAsync(putRequest);
             }
             catch (AmazonS3Exception amazonS3Exception)
             {
                 if (amazonS3Exception.ErrorCode != null &&
                     (amazonS3Exception.ErrorCode.Equals("InvalidAccessKeyId")
                     ||
                     amazonS3Exception.ErrorCode.Equals("InvalidSecurity")))
                 {
                     throw new Exception("Check the provided AWS Credentials.");
                 }
                 else
                 {
                     throw new Exception("Error occurred: " + amazonS3Exception.Message);
                 }
             }
         }
     }
    }
    ```
    
4.  Edit your Web.config file adding the next lines inside of `<appSettings></appSettings>` :
    
5.  Now call your method `UploadFile` from **HomeController.cs** to test it:
    
    ```
     public class HomeController : Controller
     {
         public ActionResult Index()
         {
             AmazonS3Uploader amazonS3 = new AmazonS3Uploader();
             amazonS3.UploadFile();
             return View();
         }
     ....
    ```
    
6.  Find your file in your Amazon S3 bucket and that's all.
    

Download my Demo Project

**Solution 3:**

I have written a tutorial about this.

**Uploading a file to S3 bucket using low-level API:**

```
IAmazonS3 client = new AmazonS3Client("AKI...access-key...", "+8Bo...secrey-key...", RegionEndpoint.APSoutheast2);  
FileInfo file = new FileInfo(@"c:\test.txt");  
string destPath = "folder/sub-folder/test.txt"; // <-- low-level s3 path uses /
PutObjectRequest request = new PutObjectRequest()  
{  
    InputStream = file.OpenRead(),  
    BucketName = "my-bucket-name",  
    Key = destPath // <-- in S3 key represents a path  
};  
  
PutObjectResponse response = client.PutObject(request); 
```

**Uploading a file to S3 bucket using high-level API:**

```
IAmazonS3 client = new AmazonS3Client("AKI...access-key...", "+8Bo...secrey-key...", RegionEndpoint.APSoutheast2);  
FileInfo localFile = new FileInfo(@"c:\test.txt");  
string destPath = @"folder\sub-folder\test.txt"; // <-- high-level s3 path uses \
 
S3FileInfo s3File = new S3FileInfo(client, "my-bucket-name", destPath);  
if (!s3File.Exists)  
{  
    using (var s3Stream = s3File.Create()) // <-- create file in S3  
    {  
        localFile.OpenRead().CopyTo(s3Stream); // <-- copy the content to S3  
    }  
}  
```

**Solution 4:**

@mejiamanuel57's solution works fine for small files under 15MB. For larger files, I was getting `System.Net.Sockets.SocketException: The I/O operation has been aborted because of either a thread exit or an application request` . Following improved solution works for larger files (tested with 50MB file):

```
...
public void UploadFile()
{
    var client = new AmazonS3Client(Amazon.RegionEndpoint.USEast1);
    var transferUtility = new TransferUtility(client);
    try
    {
        TransferUtilityUploadRequest transferUtilityUploadRequest = new TransferUtilityUploadRequest
        {
            BucketName = bucketName,
            Key = keyName,
            FilePath = filePath,
            ContentType = "text/plain"
        };
        transferUtility.Upload(transferUtilityUploadRequest); // use UploadAsync if possible
    }
...
```

More info here.

Installing docker-compose on Amazon EC2 Linux 2. 9kb, 1,976 5 5 gold badges 29 29 silver badges 32 32 bronze badges. 1. thanks! this works on linux 2 arm as well – fauxxi. Sep 1, 2021 at 19:36. sudo …

## Jenkins Continuous Integration with Amazon S3

**Question:**

I'm running Jenkins and I have it successfully working with my Github account, but I can't get it working correctly with Amazon S3.

I installed the S3 plugin and when I run a build it successfully uploads to the S3 bucket I specify, but all of the files uploaded end up in the root of the bucket. I have a bunch of folders (such as /css /js and so on), but all of the files in those folders from hithub end up in the root of my [S3 account](https://copyprogramming.com/howto/transfer-1000-files-at-a-time-from-one-s3-bucket-to-another-using-python "Transfer 1000 files at a time from one s3 bucket to another using python").

Is it possible to get the S3 plugin to upload and retain the folder structure?

**Solution 1:**

It doesn't look like this is possible. Instead, I'm using s3cmd to do this. You must first install it on your server, and then in one of the bash scripts within a Jenkins job you can use:

```
s3cmd sync -r -P $WORKSPACE/ s3://YOUR_BUCKET_NAME
```

That will copy all of the files to your S3 account maintaining the folder structure. The -P keeps read permissions for everyone (needed if you're using your bucket as a web server). This is a great solution using the sync feature, because it compares all your local files against the S3 bucket and only copies files that have changed (by comparing file sizes and checksums).

**Solution 2:**

I have never worked with the S3 plugin for Jenkins (but now that I know it exists, I might give it a try), though, looking at the code, it seems you can only do what you want using a workaround.

Here's what the actual plugin code does (taken from github) --I removed the parts of the code that are not relevant for the sake of readability:

class `hudson.plugins.s3.S3Profile` , method `upload` :

```
final Destination dest = new Destination(bucketName,filePath.getName());
getClient().putObject(dest.bucketName, dest.objectName, filePath.read(), metadata);
```

Now if you take a look into `hudson.FilePath.getName()` 's JavaDoc:

> Gets just the file name portion without directories.

Now, take a look into the `hudson.plugins.s3.Destination` 's constructor:

```
public Destination(final String userBucketName, final String fileName) {
    if (userBucketName == null || fileName == null) 
        throw new IllegalArgumentException("Not defined for null parameters: "+userBucketName+","+fileName);
    final String[] bucketNameArray = userBucketName.split("/", 2);
    bucketName = bucketNameArray[0];
    if (bucketNameArray.length > 1) {
        objectName = bucketNameArray[1] + "/" + fileName;
    } else {
        objectName = fileName;
    }
}
```

The `Destination` class JavaDoc says:

> The convention implemented here is that a / in a bucket name is used to construct a structure in the object name. That is, a put of file.txt to bucket name of "mybucket/v1" will cause the object "v1/file.txt" to be created in the mybucket.

Conclusion: the `filePath.getName()` call strips off any prefix (S3 does not have any directory, but rather prefixes, see this and this threads for more info) you add to the file. If you really need to put your files into a "folder" (i.e. having a specific prefix that contains a slash ( `/` )), I suggest you to add this prefix to the end of your bucket name, as explicited in the `Destination` class JavaDoc.

**Solution 3:**

Yes this is possible.

It looks like for each folder destination, you'll need a separate instance of the S3 plugin however.

"Source" is the file you're uploading.

"Destination bucket" is where you place your path.

**Solution 4:**

Using Jenkins 1.532.2 and S3 Publisher Plug-In 0.5, the UI configure Job screen rejects additional S3 publish entries. There would also be a significant maintenance benefit to us if the plugin recreated the workspace directory structure as we'll have many directories to create.

How to get Amazon MWS Developer ID, 3,775 1 1 gold badge 18 18 silver badges 22 22 bronze badges. 6. joshubrown : On signing up for mws, I got only merchant ID, Marketplace ID, AWS …

[![Ezoic](https://copyprogramming.com/howto/undefined)](https://www.ezoic.com/what-is-ezoic/)