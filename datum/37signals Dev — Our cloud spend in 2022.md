Since we published why [we’re leaving the cloud](https://world.hey.com/dhh/why-we-re-leaving-the-cloud-654b47e0) last October, we’ve received a lot of questions about how we are using Amazon Web Services, what our current bills looked like, and if the costs savings are likely to justify our decision. We’re happy to share, both where we currently are and where we’re going, so let’s dive in.

To begin, let’s review our collection of applications, and where they are running. We run two main products, [Basecamp](https://basecamp.com/) and [HEY](https://hey.com/), which are actively being sold to new customers. These are our largest products by far, with the most customers, and the most revenue. Additionally, we run a lot of legacy services, like [Basecamp Classic](https://signalvnoise.com/archives/000542.php), [Basecamp 2](https://signalvnoise.com/posts/3129-launch-the-all-new-basecamp), [Highrise](https://signalvnoise.com/posts/329-launch-highrise), [Backpack](https://signalvnoise.com/archives2/backpack_launches_a_new_breed_of_personal_and_business_information_manager.php), [Campfire](https://signalvnoise.com/archives2/launch_campfire_easy_group_chat_for_business.php), [Writeboard](https://signalvnoise.com/archives2/writeboard_is_live.php), and even [Ta-da List](https://signalvnoise.com/archives/001021.php). These services are no longer sold to new customers, but combined have many tens of thousands of customers, hundreds of thousands of users, and continue to contribute millions of dollars to our bottom line. We intend to support all of this [until the end of the internet](https://basecamp.com/about/policies/until-the-end-of-the-internet).

Not all of these applications run in the cloud. The current Basecamp, which is our biggest application by far, as well as the older Basecamp 2, both run almost entirely on our own servers, including application, database, and caching servers. Only search (OpenSearch), file storage (S3), and CDN services (CloudFront) come from the cloud for these services.

But HEY runs almost entirely in the cloud (with the exception of certain email and image processing services, which run on our own hardware). Our usage of AWS for HEY includes running the full Rails application on Kubernetes clusters via EKS, MySQL database servers via Aurora RDS, Redis via Elasticache, and search via OpenSearch. Additionally, our other legacy apps also run on EKS, and use of RDS for the databases.

In total, we spent **$3,201,564** on all these cloud services in 2022. That comes out to **$266,797** per month. Whew!

For HEY, the yearly bill was **$1,066,150** ($88,846/month) for production workloads only. That one service breaks down into big buckets as follows:

![HEY Costs](https://dev.37signals.com/assets/images/our-cloud-spend-in-2022/hey-costs.png)

In terms of individual services more generally, we spent **$473,196** ($39,433/month) on RDS in 2022 for all the application databases. Remember, this excludes the current Basecamp and Basecamp 2, which use our own hardware for this. So the bulk of that was spent on HEY, which was responsible for **$355,950** of the total amount for the year (or $29,662/month), the rest on the other legacy services.

For OpenSearch, which we use to host our application search clusters, and indexed storage for logging pipelines, we spent **$519,959** in 2022 ($43,330/month). This is one of those cloud services we use even for the current Basecamp and Basecamp 2, and the big spend is spread between those two otherwise-on-our-own-hardware services, as well as HEY and Basecamp Classic.

For EC2 and EKS, Amazon’s Kubernetes services, we spent a total of **$759,983** ($63,331/month) in 2022. The lion’s share of that is for HEY, both for production and staging environments, with a total of **$272,359** ($22,696/month), and the rest for the remaining legacy applications. But, again, none of it for the current Basecamp or Basecamp 2.

For Elasticache, we spent **$123,852** ($10,321/month) in 2022. The lion’s share of that is once more held by HEY, which uses this service to get Redis-backed caching.

And finally, S3, where we are storing around 8 petabytes of files, we spent a whopping **$907,838** ($75,653/month) in 2022. It’s worth noting that this setup uses a dual-region replication strategy, so we’re resilient against an entire AWS region disappearing, including all the availability zones. To serve those files, and other static assets, we spent **$66,742** ($5,562/month) in 2022 on CloudFront CDN services.

Getting this massive spend down to just $3.2 million has taken a ton of work. The ops team runs a vigilant cost-inspection program, with monthly reporting and tracking, and we’ve entered into long-term agreements on Reserved Instances and committed usage, as part of a Private Pricing Agreement. This is a highly-optimized budget.

So that’s how we spent millions on the cloud in 2022!

In 2023, we hope to dramatically cut that bill by moving a lot of services and dependencies out of the cloud and onto our own hardware. We don’t operate our own data centers, but work with our friends at [Deft](https://deft.com/) to lease rackspace, bandwidth, power, and white glove service. That isn’t cheap either at our scale, but it’s far, far less than what we spend on the cloud.

It also means that nobody from 37signals is actually roaming around a data center, racking equipment, or pulling cables. We order from Dell, have it delivered straight to the data center, and then we see the servers appear online, and then we get to work.

Anyway, when the year is over, we’ll be back with another report on how much we managed to save in 2023 by getting a lot of stuff out of the cloud, and what the cost is of building an alternative using our own hardware in the two physical data centers we currently use for Basecamp and Basecamp 2.

Hope you enjoyed the accounting!