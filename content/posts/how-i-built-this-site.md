---
title: "How I Built This Site"
date: 2023-11-19T20:38:34-05:00
draft: false
---

# Introduction

This post is one half documentation for my own purposes, and one half tutorial for someone who wants a similar site. This is a pretty simple project, so it's likely that what follows is more than what's necessary, but I'm doing it anyway. I intend to update this document at about the same rate as the system it describes (which I don't expect to change much).

## Update

About a year after writing this I decided to move the website to GitHub Pages. This architecture worked fine and cost barely any money ($2/month).

But, it *did* cost money, and there's very little reason to pay for a low-traffic static website these days. I've already gotten what I wanted out of this endeavor, so the cost had no benefit.

Moving it to GitHub Pages took all of about an hour since I was already writing the content with [Hugo.](https://gohugo.io/hosting-and-deployment/hosting-on-github/)

# Guiding Principles

1. **Focus on learning one thing at a time.** Focus is very important to productivity, and that includes not spreading myself thin. For example, I've never run a site on EC2 using [Phoenix](https://www.phoenixframework.org/) and Elixir. I've also never run a personal website before. I chose to learn the latter here. I can always revisit technologies for future projects.
1. **Keep it simple, but not too bad.** This is not a flagship product, it's a blog. It would be unusual if it gets more than a handful of visitors per week or month (in fact I'm guessing most traffic will be from bots). In other words, I don't need to hold it to the same standards that I hold [my professional work](/about/) to. I don't need stringent alarming, high-availability architectures, etc. That said, it has my name on it, so it shouldn't be total junk.

# Requirements

The main reason I wanted this site was because I had never created and hosted a website end-to-end by myself, so it seemed like a good start. I learned to code in college, and didn't start doing web development until I had a job at mature companies with their own platforms for managing and hosting sites. In general, my AWS experience is mostly using it "on the job," so it's good to see what customers see.

So I started this site with basically one customer in mind: myself.

I wanted a site that:

1. Is easy to understand
1. Is painless to manage, meaning it has a near-zero maintenance cost (either time or money) and it's trivial to update content
1. Gave me an easy venue to mess around and learn stuff about web development (not that I'd learn much from a static website, but nonetheless)
1. Cost under \$20/month. I expected to make no money off of this site, so affordability is important. This also includes being easy to shut it down, in case bot traffic starts racking up my hosting bills.

I had a pretty good idea of what technology I would use to build this, but these requirements rule out a lot of technology. For example, I knew I wouldn't learn much from using managed services like GitHub Pages. These services would be a good start if I just wanted to post content ASAP, but that isn't the case.

The site would host static content (until/unless I decide otherwise); would not require authentication or authorization; not support multiple authors; not support internationalization (if you want to read this content in a language other than English, you probably don't want me to translate it for you); and generally not require much features at all. Simplicity is better than flexibility here, so hosting and running the site on a server would be unnecessary.

# Architecture

![](../../img/billdawsdotcom-arch.drawio.png)

This is the architecture for my website. That's pretty much it.

The only things missing here are things that I consider implementation details to support it, like a CloudFront Function I had to write for accessing folders/relative paths (e.g. [/posts/](/posts/)) to work with S3's ACL configuration.

## Choices

I landed on using AWS over any other hosting provider mainly because I already knew it pretty well and I wanted to focus on learning how to build and run a static site at all, not how to do it using a particular technology. So to minimize how much I was learning at once, I chose the platform I'm most familiar with.

That choice makes the other ones pretty obvious. I needed a place to store static assets like HTML files and images, for which [S3](https://docs.aws.amazon.com/AmazonS3/latest/userguide/Welcome.html) is the most obvious choice. [S3 is capable of hosting a static website on its own](https://docs.aws.amazon.com/AmazonS3/latest/userguide/WebsiteHosting.html), and that covers most of my requirements really.

One thing S3's hosting doesn't support is TLS (i.e. hosting via HTTPS rather HTTP). Technically, nothing I have planned for this site really _requires_ TLS, like handling financial transactions. But I plan to send this site to non-technical folks, and I don't want them to turn away when their browser gives them the scary "This site is not secure" warning. The next logical choice is to introduce [CloudFront](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/Introduction.html) as a place to attach my custom domain and then certify it. Certification is done via [AWS Certificate Manager.](https://docs.aws.amazon.com/acm/latest/userguide/acm-overview.html)

One thing missing here is my domain registrar. I use [Porkbun](https://porkbun.com/) for this. I have nothing against [Route53,](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/Welcome.html) in fact I think it's great. I chose not to use Route53 for two reasons: I bought the domain before I had any plan about how to build the site; and if I ever wanted to migrate off of AWS, I would want to do that without messing with domain registration. Being able to do that reduces the likelihood that my site becomes a dead link while I wait to do some work, since I'll often go weeks without updating this site.

Lastly, I hate managing infrastructure manually, so I did everything using the [Cloud Development Kit (CDK)](https://docs.aws.amazon.com/cdk/v2/guide/home.html).

This site is a [Hugo site](https://gohugo.io/getting-started/usage/). The source code is managed privately.

### Alternatives Considered

Admittedly this architecture is simple enough that there's not much difference between its dependencies and their competitors, especially not within AWS.

I didn't spend much time considering other cloud providers like Google Cloud or Azure. From the perspective of my functional requirements, I'm sure they're just as viable. The main benefit of using a different cloud provider would be to learn something, but my focus is on learning how to make a website at all.

There's many ways within AWS to host a website. I could have used EC2 and simply ran the site myself. I could have used Lightsail or Amplify. These services are more complex, but also more flexible, than what I need. If my requirements included anything more than serving text, I would have went with one of them (or really, I would have had to, since S3 can only serve static content). Even if I used a compute service, I would probably rely on S3 and CloudFront for storage and distribution.

An alternative to CDK is to use [OpenTofu](https://opentofu.org/) (or Terraform), but again, I'm avoiding learning too many things at once. An advantage to using OpenTofu is that I'd probably have an easier time switching clouds if I ever wanted to, but I'll cross that bridge when I get to it.

I thought about using a different static site generator than Hugo, but decided against it because the differences between generators seem mostly negligible, and the cost of switching seems low. Hugo is pretty fast, which is nice, too.

# Learnings

It's really easy to set up a static website. I think it took me about a weekend at maximum. It would have taken me less time if I had lower standards, like not requiring infrastructure as code, or not requiring TLS, etc. My next static website will probably use a more hands-off approach, because I don't see much benefit in building one from scratch now that I've done it. That said, I have no regrets about building this site this way.

In the process I was exposed to what it's like to use CDK from the perspective of someone who isn't working at AWS. The internal experience of using CDK is really nice, because CDK integrates with internal account management tools natively. The external experience of creating an AWS account, setting up roles for access using [IAM Identity Center](https://aws.amazon.com/iam/identity-center/), and then using those roles is not as nice as the internal, but it's not bad either. My future projects will involve some more sophisticated infrastructure, and I'm glad I learned the "overhead" stuff on a simple project like this one.
