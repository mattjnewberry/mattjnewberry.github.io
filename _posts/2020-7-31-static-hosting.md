---
layout: post
title: Route53, S3, Cloudfront, Github Pages and the Struggles of Hosting
---
In this project blog I'll be exploring how to utilize AWS Free tier and Github Pages to create a fast,
maintainable website using Route53, Cloudfront, AWS Certificate Manager, Github Pages and
Jekyll

It seems only fitting that the first post is about the site you're currently on, so I guess we can call this project a success.
This post will explain the technologies used to deploy this static site, some tips on getting it working and the endless struggles of routing and DNS caches.

If you're just after the tutorial and don't want to read about a backend developer attempting to build front end (understandable), skip to the [tutorial](#tutorial)

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

*Fig 1: Final Architecture*

## Setting up your domain name

First, we'll set up the custom domain name as this can take some time to be registered in the DNS. 
Github Pages very kindly give us the domain name `${YOUR_GITHUB_USENAME}.github.io` for our site, however something more personal might be
good (Really it's up you). 

To register a domain name in the AWS Console, head to Route53 -> Register Domain Name and follow the steps to pick your custom
domain name. Bear in mind the prices differ slightly between the different top level domains, for example `.com` is $12 whereas `.me` is $17.

![_config.yml]({{ site.baseurl }}/images/register-domain-name.png)
*Fig 2: Registering a domain on AWS*

Amazon will create a route53 hosted zone with the same name as your domain name, we'll come back to this later to create some aliases.

## Add SSL/TLS Certificate

*To be continued...*

# Closing Thoughts

