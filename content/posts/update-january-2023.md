---
title: "January 2023 update"
date: 2023-01-10T07:00:00+01:00
tags: []
draft: false
---

Hey everyone. 👋

I hope everyone is doing well in the new year. I'd like to provide a quick update of our current financial situation and
provide some insight into the recently made changes.

<!--more-->

## Changes made to toot.community

* We're now sending emails through Amazon SES instead of Mailgun. They're significantly cheaper.
* We've moved from DigitalOcean Spaces to Cloudflare R2. Although they're more expensive, they provide a better
  experience. We've seen people experiencing issues with uploading their media. This should be fixed now.
* We've started an ActivityPub relay over at https://relay.toot.community. It's private and invite-only. I've reached
  out to some admins, and there are currently about ±175.000 active people filling this relay. This is great for
  improving content discovery for all instances involved.

## Traffic statistics

- 38.4 million requests handled in the last seven days (46% less to last update)
- 2 TB data transferred in the last seven days (13% less to last update)
- 238k 365k pages viewed in the last seven days (34% less to last update)
- 13,228,5 API requests served in the last seven days (53% less to last update)

# Financial overview, so far

| **Category**                    | **Nov 2022** | **Dec 2022** | **Jan 2023** | **Average** |    **Total** |
|:--------------------------------|-------------:|-------------:|-------------:|------------:|-------------:|
| PayPal / Patreon                |       761,97 |      1038,09 |       690,49 |      830,18 |      2490,55 |
| Stripe                          |         0,00 |       905,32 |       127,23 |      344,18 |      1032,55 |   
| **Total Income**                |   **761,97** |  **1943,41** |   **817,72** | **1174,37** |  **3523,10** |
| Kubernetes                      |         0,00 |      -353,50 |      -377,97 |     -243,82 |      -731,47 |
| Caching Proxy / Anti-DDoS*      |       -20,42 |      -236,97 |         0,00 |      -85,80 |      -257,39 |
| PostgreSQL (Persistent Storage) |         0,00 |      -137,64 |      -180,26 |     -105,97 |      -317,90 |
| Redis (Queues / Cache)          |         0,00 |       -32,11 |       -60,00 |      -30,70 |       -92,11 |
| Email                           |       -31,67 |       -45,18 |       -40,05 |      -38,97 |      -116,90 |
| Storage                         |         0,00 |       -57,92 |      -132,83 |      -63,58 |      -190,75 |  
| **Total Expenses**              |   **-52,09** |  **-863,32** |  **-791,11** | **-568,84** | **-1706,52** |
| **Net Income**                  |   **709,88** |  **1080,09** |    **26,61** |  **605,53** |  **1816,58** |

\* We've paid for Cloudflare (Caching Proxy / Anti-DDoS) in advance for the next 12 months. This is why the expenses
appear higher in December.

## Donation links

- Ko-Fi: https://ko-fi.com/jorijn
- Patreon: https://patreon.com/tootcommunity

If any of you have any questions, please let me know. Thanks for reading and until the next time. 🙏

Written by [@jorijn](https://toot.community/@jorijn).
