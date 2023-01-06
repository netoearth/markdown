# How to setup a practically free CDN

I've been using [Backblaze][bbz] for a while now as my online backup service. I have used a few others in the past. None were particularly satisfactory until Backblaze came along.

It was - still is - keenly priced at a flat $5 (£4) per month for unlimited backup (I've currently got just under half a terabyte backed-up). It has a fast, reliable client. The company itself is [transparent about their operations][trans] and [generous with their knowledge sharing][blog]. To me, this says they understand their customers well. I've never had reliability problems and everything about the outfit exudes a sense of simple, quick, solid quality. The service has even saved the day on a couple of occasions where I've lost files.

Safe to say, I'm a happy customer. If you're not already using Backblaze, [I highly recommend you do][recommend].

## Taking on the big boys with B2

So when Backblaze [announced][announce] they were getting into the cloud storage business, taking on the likes of Amazon S3, Microsoft Azure, and Google Cloud, I paid attention. Even if the cost were the same, or a little bit more, I'd be interested because I like the company. I like their product, and I like their style.

What I wasn't expecting was for them to be cheaper. *Much* cheaper. How about 440% cheaper than S3 per GB? Don't believe me? [Take a look][comparison]. Remarkable.

What's more, they offer a [generous free tier][free] of 10 GB free storage and 1 GB free download per day.

If it were any other company, I might think they're a bunch of clowns trying it on. But I know from my own experience and following their journey, [they're genuine innovators][storagepod] and good people.

## Using B2

B2 is pretty simple. You can use their web UI, which is decent. Or you can use Cyberduck, which is what I use, is free, and of high quality. There is also a [command-line tool][cli] and [a number of other integrated tools][integrations]. There is also a [web API][api], of course.

## Setting up a vanity URL

You can set up a "vanity" URL for your public B2 files. Do it for free using CloudFlare. There's a [PDF [1.3 MB] documenting how][vanity].

## Using CloudFlare CDN to cache B2 hosted files

You can also configure CloudFlare to aggressively cache assets served by your B2 service. It is not immediately obvious how to do this, and took a bit of poking around to set up correctly.

By default, B2 serves with cache-invalidating headers: `cache-control:max-age=0, no-cache, no-store`, which causes CloudFlare to skip caching of assets. You can see this happening by looking for the `cf-cache-status:MISS` header.

To work around this problem, you can use CloudFlare's PageRules specifying an "Edge cache expire TTL". I won't explain what that means here as it is [covered in-depth on the CloudFlare blog][edgecache].

So, to cache your B2 assets, you need to create a PageRule that includes all files on your B2 domain. For example:

    files.silversuit.net/*

You then need to add your cache settings. I have **Cache Level** set to **Cache Everything**; **Browser Cache TTL** set to **a year**; **Edge Cache TTL** set to **7 days**. I'm caching aggressively here, but you can tweak these settings to suit. Here's a screenshot:

<figure>
    <img src="https://f000.backblazeb2.com/file/silversuit/2016-04-21-how-to-setup-a-practically-free-cdn/PageRules__screenshot__2016-04-21-124640.png">
    <figcaption>Screenshot showing PageRules settings</figcaption>
</figure>

To check that it's working correctly, use DevTools to look for the `cf-cache-status:HIT` header:

<figure style="max-width:50%">
    <img src="https://f000.backblazeb2.com/file/silversuit/2016-04-21-how-to-setup-a-practically-free-cdn/CacheHit__screenshot__2016-04-21-124707.png">
    <figcaption>Screenshot showing a CloudFlare cache hit</figcaption>
</figure>

## Wrapping up

So, with that, you're making use of already very inexpensive B2 storage coupled with CloudFlare's free CDN to serve your assets almost entirely for free. And it's not like these are rinky-dink services that are going to fall over regularly; these are both high-quality, reputable companies.

What a time to be alive, eh?

[recommend]: https://secure.backblaze.com/r/009oqf
[comparison]: https://www.backblaze.com/b2/cloud-storage-providers.html
[storagepod]: https://www.backblaze.com/b2/storage-pod.html
[cli]: https://www.backblaze.com/b2/docs/quick_command_line.html
[integrations]: https://www.backblaze.com/b2/docs/integrations.html
[api]: https://www.backblaze.com/b2/docs/calling.html
[vanity]: https://f001.backblaze.com/file/Backblaze_B2_Beta/Configuring+Cloudflare+and+B2.pdf
[free]: https://www.backblaze.com/b2/cloud-storage-pricing.html
[edgecache]: https://blog.cloudflare.com/edge-cache-expire-ttl-easiest-way-to-override/
[bbz]: https://www.backblaze.com/
[blog]: https://www.backblaze.com/blog/
[trans]: https://www.backblaze.com/hard-drive.html
[announce]: https://www.backblaze.com/blog/b2-cloud-storage-provider/
