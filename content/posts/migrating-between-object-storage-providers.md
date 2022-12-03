---
title: "A zero-downtime migration from DigitalOcean Spaces to Cloudflare R2"
date: 2022-12-02T12:00:00+01:00
tags: []
draft: true
---

Tomorrow marks the first month of toot.community's existence. We grew very fast, and with us, our storage did so too.
Currently, we store 935GB spread out over 3,191,702 files. Our data consists of avatars, header images, individual
account exports, and media attachments of posts.

At first, the easy choice was to set up object storage at DigitalOcean, where the rest of our infrastructure is. Yet, if
we're being honest here: I underestimated the rate of growth, and as it turns out, DigitalOcean performs just fine, but
when crossing their free tier, it can get expensive. Not for the amount of data stored but for the bandwidth for
outgoing traffic.

We will migrate our data to Cloudflare R2, API-compatible with S3, just like DigitalOcean, but cheaper for Egress fees.

<!--more-->

## Taking inventory: what do we use?

![](/images/20221202-screenshot-cache-overview.png)

This chart describes the request summary of the past week for all static content served from our CDN,
files.toot.community. If we extrapolate, we can conclude that our CDN serves 47,2 million requests each month, of which
±5,6 million by the edge endpoint of the storage bucket.

![](/images/20221202-screenshot-bandwidth-cdn.png)

Here, we're looking at the amount of traffic our CDN has served. As you can see, Cloudflare eats up quite a big chunk of
this bandwidth, and since their Egress fees are free, we're left to deal with the costs for traffic served from our
origin servers. Extrapolating again: ± 7,5 TB of outgoing traffic, only 920 GB from our origin servers.

## How does this translate to costs?

The DigitalOcean pricing plan is as follows:

* The base fee is $5 per month
* 250GB storage is free, and additional storage for $0.02/GB
* 1TB of outbound transfer included, and more additional bandwidth for $0.01/GB

**For us, this translates to**:

* 250GB free, 685GB overage (935GB total) is $13.70 per month
* For now, no additional Egress fees
* **Total: $18.70/month**

## Comparing to CloudFlare R2

CloudFlare R2 is a bit more complicated to calculate, but we can use the following formula:

* 10GB free, $0.015 for each additional GB
* Class A operations: 1 million/month free, $4.50 for each additional million
* Class B operations: 10 million/month free, $0.36 for each additional million

Since we don't know yet, we can only guesstimate how our operations will divide between type A and B. Class A operations
are the ones that mutate the state, such as creating new directories and uploading new files. Class B operations don't;
these are reading files. We have about 3,2 million files in our storage bucket and these have been requested 47,2
million times.

**If we'd assume a fresh start**:

* 3,2 million uploads (Class A) is 2,2 million billed: $9/month
* 5,6 million downloads (Class B), falls below the free-tier threshold of 10 million, free
* $0.015 times 935GB = $13.70/month
* **Total: $22.70/month**

Granted, Cloudflare would be a bit more expensive at this point but looking at the future; I still think it's a good
choice. Class A operations should be less costly when running regular operations since the data is already there. When
looking at it plainly:

|              | DigitalOcean | Cloudflare R2 |
|--------------|--------------|---------------|
| Price per GB | $ 0.02       | $ 0.015       |
| Bandwidth    | $ 0.01       | Free          |

## Migration Strategy: Intermediate Webserver

How do you migrate data from one storage provider to another without downtime? The answer is: you don't. But, we can set
up a intermediate webserver that is configured for automatic failover between the two storage providers; completely the
effects. This is how we're going to do it:

We will spin up a temporary VPS for serving all the static content. To do that, we will configure NGINX to proxy the
static content. The idea is to have NGINX query Cloudflare R2 to see if the file is there, and if it's not, it will
fetch it from DigitalOcean. Since it'll check at Cloudflare R2 first, the more we reach the end of the migration, the
faster it'll be able to return the content. That way, we can lean on two different (possibly incomplete) backends during
the migration. At first, it'll fetch 99% of the content from DigitalOcean, but towards the end, it might look more like
80% from Cloudflare R2 and the remainder from DigitalOcean.

We'll use this configuration snippet:

```nginx
location / {
        try_files /$uri @cloudflare;
}

location @cloudflare {
        proxy_ssl_server_name on;
        proxy_pass https://<our bucket url>;
        add_header X-Upstream cloudflare always;
        proxy_intercept_errors on;
        recursive_error_pages on;
        error_page 404 = @digitalocean;
}

location @digitalocean {
        proxy_ssl_server_name on;
        add_header X-Upstream digitalocean always;
        proxy_pass <cdn location of bucket at digitalocean>;
}
```

Written by [@jorijn](https://toot.community/@jorijn).