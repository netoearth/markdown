[AWS Simple Storage Service](https://aws.amazon.com/s3/) or S3 is one of the most commonly used cloud-based storage across the globe. It provides infinite storage with high uptime, immediate access, and scales up as per your requirements.

A common use of AWS S3 storage is to store large files, like images. Images are often huge and have to be stored for really long times. Storing them in S3 takes away the pain of managing the storage on your own or continuously adding space on your server.

### Features missing for image optimization natively on AWS S3

While S3 is excellent for image storage, it does not provide the following features, which are now fundamental for every website that is serious about delivering an exceptional experience to its users -

1.  The ability to be able to [compress the images](https://imagekit.io/blog/image-optimization-its-importance/)
2.  The ability to be able to [convert the images to a newer, better formats](https://docs.imagekit.io/features/image-optimization/automatic-image-format-conversion) like WebP
3.  The ability to [resize and crop the images](https://docs.imagekit.io/features/image-transformations/resize-crop-and-other-transformations) for use on different pages of our application

And maybe a more advanced requirement could be to be able to these and more advanced manipulations like watermarking those images, creating smart thumbnails in real-time without having to write or maintain complex solutions for it. More often than not, managing images is not your core business, and you do not want to be spending a lot of time on it.

[

![](https://imagekit.io/blog/content/images/2020/06/bloated-s3-image.jpg)



](https://imagekit.io/blog/content/images/2020/06/bloated-s3-image.jpg)

A large and unoptimized image in S3 

### What is ImageKit, and how does it optimize images in S3?

[ImageKit.io](https://imagekit.io/) is a cloud-based [image CDN](https://imagekit.io/blog/what-is-image-cdn-guide/) with real-time image optimization and transformation features that help you deliver perfectly optimized images across all devices.

[

![](https://imagekit.io/blog/content/images/2020/06/optimized-with-ImageKit-1.jpg)



](https://imagekit.io/blog/content/images/2020/06/optimized-with-ImageKit-1.jpg)

Just by delivering through ImageKit, your images get optimized. You can then start resizing the images as well.

It comes integrated with AWS CloudFront for image delivery (and other non-image files too). Over 15,000 developers and companies already serve billions of optimized images via ImageKit and its CDN every day. So, you do not have to pay for a separate CDN if you are using ImageKit.

You can attach your AWS S3 bucket to ImageKit and start delivering optimized images in just a few minutes. Without writing a single line of code.

[

![](https://imagekit.io/blog/content/images/2020/06/s3-setup.jpg)



](https://imagekit.io/blog/content/images/2020/06/s3-setup.jpg)

Setup of your AWS S3 storage with ImageKit

###   
Image Optimization on S3 + ImageKit - How does it work?

Let's look at how a request gets handled when you optimize your S3 images via ImageKit.

1.  When a request comes to ImageKit, the image is delivered from the CDN cache.
2.  If the image is not there in the CDN cache, ImageKit tries to deliver the image from its internal caches. Usually, images that are transformed once will stay in ImageKit's caches for some time.
3.  If the image is not found in this internal cache as well, then ImageKit gets the image from your AWS S3 bucket attached to ImageKit, optimizes it, and transforms it in real-time and sends it back to the user.
4.  After this first time processing that takes about 200ms on an average, the image gets cached on the CDN and ImageKit's internal caches, bringing down the response time for almost 98% of image requests to under 50ms.

[

![](https://imagekit.io/blog/content/images/2020/06/User-Request--1-.png)



](https://imagekit.io/blog/content/images/2020/06/User-Request--1-.png)

The flow of request with ImageKit's CDN, Processing Server and your AWS S3 storage

### Setting up ImageKit and AWS S3 for image optimization and resizing

Let's set up AWS S3 with [ImageKit](https://imagekit.io/) and start delivering optimized images. It will take a few minutes if you follow the steps here.

### 1\. Creating a new ImageKit account

The first step is to [create a new ImageKit account](https://imagekit.io/registration) if you have not done so already. You get 20GB free output bandwidth every month forever without adding any credit card details.

[

![](https://imagekit.io/blog/content/images/2020/06/Screen-Shot-2020-06-30-at-12.55.22-PM.png)



](https://imagekit.io/blog/content/images/2020/06/Screen-Shot-2020-06-30-at-12.55.22-PM.png)

A new ImageKit account that has been setup for this demo

ImageKit has six regions for processing the images. Please select a region that is the same as your S3 bucket or is close to it.

### 2\. Creating a new user in AWS

ImageKit only needs a read-only or, more specifically, the GetObject access to your S3 bucket. It is recommended that you create a separate user in AWS for use in ImageKit.

You can [create a new user](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_users_create.html) by going to the AWS IAM console. For this demo, we have created a user called `imagekit-s3-demo-integration` with Programmatic access, i.e. access using keys, enabled.

[

![](https://imagekit.io/blog/content/images/2020/06/adding-new-user.png)



](https://imagekit.io/blog/content/images/2020/06/adding-new-user.png)

Creating a new user in AWS IAM console with Programmatic access enabled

You just need to create a new user as of now. Don't worry about the access you grant to this user. We will be doing that in the next section.

Once the user is created, copy its access key and secret key as we will use it later to integrate with ImageKit.

[

![](https://imagekit.io/blog/content/images/2020/06/new-user.jpg)



](https://imagekit.io/blog/content/images/2020/06/new-user.jpg)

Once the new user is created copy the access key ID and the Secret access key

### 3\. Assigning access to the bucket

We have a bucket named `imagekit-s3-demo-integration` for this demo with a couple of images in it.

[

![](https://imagekit.io/blog/content/images/2020/06/Screen-Shot-2020-06-30-at-12.57.16-PM.png)



](https://imagekit.io/blog/content/images/2020/06/Screen-Shot-2020-06-30-at-12.57.16-PM.png)

A demo S3 bucket that has been for this guide

The above bucket is private, but the steps here will be the same, even for a public bucket. We need to provide GetObject access on this bucket to the newly created user.

a. To do this, click on the "Add Permissions" button for this user. Select "Attach existing policies directly" and then click on "Create Policy."

[

![](https://imagekit.io/blog/content/images/2020/06/Screen-Shot-2020-06-30-at-1.04.51-PM.png)



](https://imagekit.io/blog/content/images/2020/06/Screen-Shot-2020-06-30-at-1.04.51-PM.png)

Click on "Add Permissions" for the newly created user to get started

[

![](https://imagekit.io/blog/content/images/2020/06/Screen-Shot-2020-06-30-at-6.02.28-PM.png)



](https://imagekit.io/blog/content/images/2020/06/Screen-Shot-2020-06-30-at-6.02.28-PM.png)

Create a new policy to be attached to this user

b. Select the JSON creator option for the policy and use the following JSON to create the policy that provides the GetObject access to our demo bucket. Note, that you need to change the bucket name to that of your bucket.

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "VisualEditor0",
            "Effect": "Allow",
            "Action": "s3:GetObject",
            "Resource": [
                "arn:aws:s3:::imagekit-s3-demo-integration",
                "arn:aws:s3:::imagekit-s3-demo-integration/*"
            ]
        }
    ]
}
```

Policy that provides GetObject Access to the bucket

We named our policy ImageKitS3DemoIntegration. Once you save the policy, it looks like this -

[

![](https://imagekit.io/blog/content/images/2020/06/Screen-Shot-2020-06-30-at-6.06.23-PM.png)



](https://imagekit.io/blog/content/images/2020/06/Screen-Shot-2020-06-30-at-6.06.23-PM.png)

c. Attach this newly created policy to the user and confirm that the user in IAM panel has this policy attached.

[

![](https://imagekit.io/blog/content/images/2020/06/user-with-policy.png)



](https://imagekit.io/blog/content/images/2020/06/user-with-policy.png)

Once the policy is attached, this is how the User in IAM panel looks like

### 4\. Adding access in ImageKit

We will now use the access and secret keys for the new user that we created in AWS to enable [ImageKit to access the S3 bucket](https://docs.imagekit.io/integration/configure-origin/amazon-s3-bucket-origin). The documentation for the same can also be found here - [Documentation](https://docs.imagekit.io/integration/configure-origin/amazon-s3-bucket-origin).

In ImageKit, go to the Origins section under External Storage and then click on the "Add Origin" button. Change the "Origin type" to "Amazon S3 Bucket" and add the values in the corresponding fields.

[

![](https://imagekit.io/blog/content/images/2020/06/values-in-s3.jpg)



](https://imagekit.io/blog/content/images/2020/06/values-in-s3.jpg)

Fill all the fields required for S3 bucket integration in ImageKit

Once we save this, if this is the first origin that you added in your ImageKit account, it will automatically get mapped against a URL endpoint. If this does not happen automatically, you can always go to the URL endpoints section and add this new S3 origin in the origin preference sequence.

[

![](https://imagekit.io/blog/content/images/2020/06/origin-added-to-endpoint.jpg)



](https://imagekit.io/blog/content/images/2020/06/origin-added-to-endpoint.jpg)

AWS S3 Bucket gets mapped to the URL Endpoint

### Let's access the optimized image via ImageKit

Our sample bucket has an image at the path `monuments/eiffel-tower.jpeg`. The image `eiffel-tower.jpeg` is 938kb initially in size.

[

![](https://imagekit.io/blog/content/images/2020/06/bloated-s3-image-1.png)



](https://imagekit.io/blog/content/images/2020/06/bloated-s3-image-1.png)

As indicated under the URL Endpoint inside your ImageKit dashboard, to access this image using ImageKit, you need to append the URL endpoint from your ImageKit dashboard with the path of the image in the bucket.

```
https://ik.imagekit.io/s3demoik/monuments/eiffel-tower.jpeg
```

Path for accessing the AWS S3 image. URL Endpoint is 

And voila!

Without adding any other parameter to the URL, the image is now down to 583KB in size and is also automatically converted to WebP image format in supported browsers.

[

![](https://imagekit.io/blog/content/images/2020/06/image-via-imagekit.jpg)



](https://imagekit.io/blog/content/images/2020/06/image-via-imagekit.jpg)

Accessing the image via ImageKit optimizes the image and converts it to the right format too

### Resizing and other transforms on AWS S3 images with ImageKit

Not just automatic optimization, you can also resize and transform the images in your S3 bucket in real-time.  You just need to specify the right transformation in your image URL

Here is the above image resized to a width of 300px -

```
https://ik.imagekit.io/s3demoik/monuments/eiffel-tower.jpeg?tr=w-300
```

[

![](https://imagekit.io/blog/content/images/2020/06/image.png)



](https://imagekit.io/blog/content/images/2020/06/image.png)

Image resized to 300px width - [URL](https://ik.imagekit.io/s3demoik/monuments/eiffel-tower.jpeg?tr=w-300)

We can also specify both height and width to get a square thumbnail of this image.

```
https://ik.imagekit.io/s3demoik/monuments/eiffel-tower.jpeg?tr=w-300,h-300
```

[

![](https://imagekit.io/blog/content/images/2020/06/image-1.png)



](https://imagekit.io/blog/content/images/2020/06/image-1.png)

A 300x300 thumbnail of the original image - [URL](https://ik.imagekit.io/s3demoik/monuments/eiffel-tower.jpeg?tr=w-300,h-300)

There are many [other transformations in ImageKit](https://docs.imagekit.io/features/image-transformations). All of them can be used with the images in the AWS S3 bucket. You can use the [smart crop feature](https://imagekit.io/blog/smart-crop-deliver-perfect-responsive-images/) to center the image's subject in the output thumbnail automatically.  Or you could use [overlays to create dynamic banners](https://imagekit.io/blog/how-to-create-a-photo-collage/), control how thee image aspect ratio and cropping are handled, and so on.

Here is a [video demonstrating the use of some of these transformations](https://www.youtube.com/watch?v=BlYfugkBfmc) on the image.

<iframe src="https://www.youtube.com/embed/BlYfugkBfmc?feature=oembed" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen="" name="fitvid0"></iframe>

### Real-time image optimization for AWS S3 without writing any code

And just like that, optimizing and resizing images in your AWS S3 bucket becomes really simple. Easy integration and you start delivering optimized images in minutes.

If you use some metrics like [Google Lighthouse](https://developers.google.com/web/tools/lighthouse) to monitor the images on your website, then you would be able to check off all the image-related points from that list using ImageKit with minimal effort.

### Get started with ImageKit now!

[ImageKit](https://imagekit.io/) comes with a forever free account that gives you access to all the image optimization and transformation related features for use with your S3 bucket. And you get 20GB free delivery bandwidth every month.

[Sign up now](https://imagekit.io/registration) and start delivering perfectly optimized images on your websites and apps.