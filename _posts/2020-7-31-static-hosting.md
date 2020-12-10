---
layout: post
title: Route53, S3, Cloudfront, Github Pages and the Struggles of Hosting
---

In this project blog I'll be exploring how to utilize AWS Free tier and Github Pages to create a fast,
maintainable website using Route53, \*~~Cloudfront~~, ~~AWS Certificate Manager~~, Github Pages and
Jekyll.

It seems only fitting that the first post is about the site you're currently on, so I guess we can call this project a success.
This post will explain the technologies used to deploy this static site, some tips on getting it working and the endless struggles of routing and DNS caches.

If you're just after the tutorial and don't want to read about a backend developer attempting to build front end (understandable), skip to the [tutorial](#tutorial)

\* There were issues with routing from Cloudfront to Github pages, where Github pages was our origin. Without access to the origin servers, it's very hard to debug network issues, and for this reason I advise against it. See [this](#cautionary-tale) section for more information

# Contents

- [Introduction](#introduction)
- [Tutorial](#tutorial)
- [Closing Thoughts](#Closing-thoughts)

# Introduction

**Goal:** A inexepensive, visually simple, easiliy maintainable, static website that utilizes the free tier AWS
ecosystem.

The full static website tech stack is as follows:

- [Github Pages](https://pages.github.com/) for the hosting
- [Route53](https://aws.amazon.com/route53/) for custom domain registration and DNS resolution
- [Cloudfront](https://aws.amazon.com/cloudfront/) for the CDN service
- [AWS Certificate Manager](https://aws.amazon.com/certificate-manager/) for the public SSL/TLS certificates
- [Jekyll](https://jekyllrb.com/) + [Jekyll Now Theme](https://jekyllthemes.io/theme/jekyll-now) for the static site generation

**NOTE:** All costs are correct as of 2020.08.01 and are subject to change.

**Why Github Pages and not S3?**

Originally, the design utilied AWS's [S3](https://aws.amazon.com/s3/) service to keep with the theme
of everything hosted within one space, in this case, an AWS account. S3 has additional benifits, namely
routing from Route53 to S3 [does not incur any charge](https://aws.amazon.com/route53/pricing/), therefore,
if you chose not to use the CDN, the only cost of this site is the S3 storage and access charges
(Free up to 5GB and 20,000 respectively), the DNS ($0.59 PCM) and domain registration ($12 PA for .com).

However, when introduction Cloudfront as your CDN, the reduced costs of S3 are no longer applicable.
Route 53 does not charge for routing to a Cloudfront distribution also and as such the only additional
costs are from Cloudfront, making it indifferent if the site if on Github Pages or within S3 Free tier.

In addition, Github Pages has excellent Jeykll and theme integration. Using jekyll locally and uploading
the files to S3 requires knoweledge of Ruby gems and dependency management and even after 2 days, I was
unable to get a theme to apply correctly to a locally hosted Jekyll site. In comparison, I was able to apply
the Jekyll Now theme to my Github Page within 5 minutes. Although, it's recognized this point may be moot to a more experienced
Ruby developer.

Finally, using Github Pages gives you possibly the simpliest CI/CD pipeline ever made. If you're
planning on using Github as your source version control (SVC), pushing to your master branch will
update your website automcatically - No need for Github Web hooks to trigger deployment to S3.

**Why did you choose Jekyll over similar frameworks, or even writing the markup yourself?**

To begin with, I was adament about writing the markup myself. This quickly dissapeared
after a few painful hours of misbehaving CSS. Without the Github intergration, I may have reconsidered this
usage, but the combination just makes it too easy to pass up. The same point applies to
why I didn't use alteranatives such as [Hugo](https://gohugo.io/) and [Hexo](https://hexo.io/)

**Why did you focus on the AWS Ecosystem over GCP or Azure?**

Good question. I'm most familiar was AWS. Sometimes the right tool depends on which tool
you're most comfortable with (Not always mind).

# Tutorial

## Prequisites

- An AWS Account
- A Github Account

The below image shows the final architecture after following this tutorial:

![_config.yml]({{ site.baseurl }}/images/final-arch.png)

_Fig 1: Final Architecture_

## Setting up our custom domain name in Route53

First, we'll set up the custom domain name as this can take some time to be registered in the DNS.
Github Pages very kindly give us the domain name `${YOUR_GITHUB_USENAME}.github.io` for our site, however something more personal might be
good (Really it's up you).

To register a domain name in the AWS Console, head to Route53 -> Register Domain Name and follow the steps to pick your custom
domain name. Bear in mind the prices differ slightly between the different top level domains, for example `.com` is $12 whereas `.me` is $17.

![_config.yml]({{ site.baseurl }}/images/register-domain-name.png)
_Fig 2: Registering a domain on AWS_

Amazon will create a route53 hosted zone with the same name as your domain name, we'll come back to this later to create some aliases.

## Add SSL/TLS Certificate to our custom domain name

:warning: [Why you may not want to do this](#cautionary-tale)

Next, we're going to request a SSL/TLS certicate for our domain. Since our domain is resolved in Route53, Amazaon can verify the domain for us. To request a certificate, head to
AWS certicate Manager -> Request a certicate. Follow the process to request a public certicate. A good suggestion is to request a certificate that includes both your domain (e.g mattjnewberry.com)
and using a wildcard to any lower level domains (e.g \*.mattjnewberry.com).

Amazon will usually validate the domain name in a couple of minutes.

## Creating your Github Page

The actual hosting of the website is using Github Pages, which offers one personal static website for free. Jekyll is a powerful static site generator with
pre-existing easy-to-configure themes. But more importently, has built-in integration with Github Pages. This site was build using the jekyll-now theme, and can be replicated
by following the steps in this repository (Yes, it's as easy as forking a repository).

Once Github has realised it's a repository for our personal static website, additional options will appeaer in the repository settings. Make sure to add your custom domain whilst we're here!

![_config.yml]({{ site.baseurl }}/images/github-pages-repo-settings.png)
_Fig 3: Github Pages repository setttings_

## Adding Cloudfront

:warning: [Why you may not want to do this](#cautionary-tale)

Cloudfront is a CDN (Content Delivery Network), which means we can cache our origin resources closer to clients and improve our network performance.

Since I never managed to get Cloudfront working consistently with Github Pages, I can't offer much advice. For serving an S3 site using Cloudfront, this [AWS blog](https://aws.amazon.com/blogs/networking-and-content-delivery/amazon-s3-amazon-cloudfront-a-match-made-in-the-cloud/) is excellent

## Adding Route53 aliases

Aliases allow us to route traffic from one domain name to another, allowing us to make use of subdomains.

In this example we'll configure our aliases to point directly to our Github Pages, however if you're using Cloudfront, it should be a similar process, just make sure to select `Alias to CloudFront distribution` when creating the aliases.

We'll create two aliases:

- `mattjnewberry.com` - Domain name
- `www.mattjnewberry.com` - Subdomain

To create an our domain alias:

1. Select _Create Record_
2. Set the _Record type_ `A - Route traffic to an IPv4 address..` and paste the following IP's into the value box:

```
185.199.108.153
185.199.109.153
185.199.110.153
185.199.111.153
```

3. Set any TTL value you like (I selected 1hr)
4. Select _Create record_ to save it

To create our subdomain alias:

1. Select _Create Record_
2. Type `www` into _Record name_
3. Set the _Record type_ to `A - Route traffic to an IPv4 address..`
4. Tick _Alias_ checkbox
5. Set the _Route traffic_ to `Alias to another record...` and select your domain alias
6. Set any TTL value you like (I selected 1hr).
7. Select _Create record_ to save it

## Cautionary Tale

The original plan for this blog post was to include a CloudFront setup, and it did for a month or so. But then something terrible happened. Cloudfront could no longer serve the Github Pages origin. No changes in configuration, it just stopped working - such is software I suppose :man_shrugging:.

It's with regret that when using Github pages as our origin, I advise against using a CDN - it's a classic case of over engineering. If your site is hosted on Github Pages, it's unlikely you need a CDN. If you want to use a CDN, I highly suggest hosting yourself (e.g S3) where you have access to the origin server.

# Closing Thoughts

Creating this website and writing about my experiences has been a suprisinly difficult endevour. Not becuase anything is particulary challenging (except perhaps the spelling), but rather the opposite. There is an abundance of free hosting options and finding the one that fits your use case is tricky. Some may want more freedom, others want to abstract as much of deployment as possible and it's not always easy to find the balance.

Using AWS for the networking but letting Github Pages deal with the hosting felt just about right for me at this point in time but that doesn't mean it's right for you, or right for me in 10 years or 10 weeks and I think this is a constant challenge when build something new. It's important to take the time, try each solution as a prototype and finally commit to whichever solution you pick whilst recognizing nothing in software is permenant.

Thank you for reading :smile:

If you're interested in more, do reach out to me via the <a href="../about">contact section</a>)
