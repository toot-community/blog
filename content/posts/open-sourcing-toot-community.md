---
title: "Open sourcing toot.community"
date: 2022-12-02T12:00:00+01:00
tags: []
draft: true
---

Open source software has been one of my very first introductions to computing. Way back in 1995 I got to install Red Hat Linux on a hand-me-down IBM PC and that led to me learn more about the inner working of computers than I could have ever imagined. A love for simple, open source software was born and I carried that forward by always open sourcing as much of my work as I can. To this day, even though my operating system is closed source, most software I'm using is open source.

<!--more-->

When [Jorijn](https://toot.community/@jorijn/) and I started toot.community we all had to figure it out on the fly - I don't think anyone else was doing Mastodon completely autoscaled and event-driven on top of Kubernetes. As we were going through a lot of challenges I put forward the idea to one day open source all of our efforts. 

> ⚠️ We're not open sourcing this because we want to provide a 1-click install and to replicate our setup, it's all simply too complex to wrap everything up nicely and make it fool proof. No, we open source it because we think it's a learning opportunity for others, a way for others to cherrypick some of our ideas and apply it to their infrastructure. No support will be provided either, buyer beware!

We're open sourcing both the platform (infrastructure) code as well as all Kubernetes manifests we're using to scale toot.community and have zero downtime upgrade paths. Let's talk about some challenges we faced and the who's who of our setup.

## Terraform

As we're hosting most of our infrastructure on Digitalocean I used [Terraform](https://www.terraform.io/) to describe it all. Having everything in code means it's self-documenting and setting up an environment to test an upgrade is as easy as copying a few files.

The infrastructure currently consists of:

- DOKS for the Kubernetes cluster
- Managed PostgreSQL
- Managed Redis
- Spaces for state management and backups

I'll be honest here, the Terraform [provider for Digitalocean](https://registry.terraform.io/providers/digitalocean/digitalocean/latest) isn't very good and that was a little bit frustrating to me at first. 

A few examples:

- There's no way to import a Digitalocean project into Terraform state. As I had to import already existing infrastructure, this means the project isn't in code and anyone using this Terraform code will not have their resources organized into one.
- Creating a Digitalocean Kubernetes cluster doesn't allow me to create an empty cluster without any node pools. Also, subsequent node pools need to be created using a separate resource called `digitalocean_kubernetes_node_pool`. This way changing anything about the primary node pool also has the potential to change some properties on the Kubernetes cluster. I'm not too psyched about that.
- Importing a Digitalocean Spaces bucket doesn't get you all properties associated with the resources like the CORS policy. This means you have to be extra careful to not overwrite something.
- Creating a read-replica of a database instance does indeed create a resource but then returns a 404 error. This was later fixed in [version 2.25.2](https://github.com/digitalocean/terraform-provider-digitalocean/issues/906) of the provider.

In the end we we're able to create a workaround for all of these issues and we now have a stable codebase for our infrastructure. I'm sure all of these issues in the provider will be ironed out soon a it's just lagging behind the capabilities of the API.

[The Terraform code can be found here](https://github.com/toot-community/platform).

## Kubernetes

The Kubernetes manifests needed a lot of cleanup and documentation before we could even think about open sourcing it. Everything is so interconnected and integrated with tools like 1Password for getting secrets during deployments, it's honestly not something anyone can just grab and execute against their Kubernetes cluster. This is why the README has a big banner telling the viewer the code is not be executed verbatim - it's simply there to share the concepts and provide a starting point.

Our current Kubernetes stack is quite different from where we started; we now have a lot more visibility into statistics and we've moved to a fully event-driven architecture. Among many other tidbits, it includes the following:

- [ArgoCD](https://argo-cd.readthedocs.io/en/stable/) for automated deployments
- Grafana
- HAproxy for tunneling Redis
- Nginx ingress
- [KEDA](https://keda.sh/) for event-driven scaling of the web and sidekiq pods
- Prometheus 
- [LibreTranslate](https://libretranslate.com/)

Jorijn did a wonderful job of going into detail on each of these applications in the README of [the kubernetes repository](https://github.com/toot-community/kubernetes) so I'll not be repeating that here.

## Moving forward

This blog will allow us to do deep dives into technical implementation details and things we learned along the way. Other than technical stuff, we also want to share more about moderating a Mastodon instance and I'm sure we will come up with even more tooling we can open source.

Written by [@mijndert](https://toot.community/@mijndert).