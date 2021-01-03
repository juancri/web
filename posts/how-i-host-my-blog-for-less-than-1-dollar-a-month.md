---
title: How I host my blog for less than 1 dollar a month
date: 2021-03-28
tags:
  - english
  - cloud
  - aws
layout: layouts/post.njk
---

## Introduction

I have had multiple websites over the time. I can remember a website I had back around the year 2000. Back then, [GeoCities](https://en.wikipedia.org/wiki/Yahoo!_GeoCities) was a popular option. I'm not sure I ever had a website there but I remember to have used similar services. These services were free of charge but usually inserted advertisement into the websites hosted there, so not very professional but good enough for a teenager.

After a few years, I paid for a co-located server. After a few years, virtualization was a thing so during the last years I have had multiple virtual server providers, including [DigitalOcean](https://www.digitalocean.com/) and [Amazon Web Services](https://en.wikipedia.org/wiki/Yahoo!_GeoCities). About the cost, now days you can get a small virtual server for as cheap as $5 a month. One downside of this approach is all the administration work you have to do. OS updates, security patches, certificates renewal and other tasks are not easily avoidable when you have your own server, even if it's virtual.

If you just want something that works, services like [Squarespace](https://www.squarespace.com/) or [WiX](https://www.wix.com/) are an easy way to get a website up and running. There are no administration tasks but one downside is their pricing. The *Combo* plan from WiX is USD 14 / month, which is not particularly cheap.

Can we do better than this? Is there any way to get a professional looking website for a low price and not having to use our valuable time managing a server?

Well, a few months ago I had the same question and decided to experiment with  a static website generator and some services.

## Requirements

For my particular use case, which is a personal blog, I have these requirements:

- **Own domain name:** I want to be able to use my own domain name *juancri.com*. This is part of making it look a little bit professional, even when it's a personal blog.
- **Cheap:** For this particular service, I don't want to pay more than $5 a month, which is the baseline of a virtual server.
- **No admin tasks:** I don't want to spend time managing a server. That way, I can instead use my valuable time watching YouTube.
- **Simple process to add posts:** This is important to me, but I understand it's a very subjective matter. What I find a *simple process* could be a hard task for you. I'm sure some of you can find the current process not that simple enough, but it takes me one minute to create a new post. More about this later.
- **No ads:** Just to be clear, I don't want a free service that will inject ads to my website. As I said before, I want to keep it clean and professional.

## Static vs dynamic

Blogs and similar websites are usually viewed as *"dynamic websites"*. At the end of the day, we have content we will be adding or modifying from time to time, right? Well, it depends. Historically, there have been these two kind of websites: *static* and *dynamic* and static ones were super hard to modify -at least for someone who has no experience with HTML, CSS and other web technologies. Dynamic websites, on the other hand, usually require a database and a server that can generate the pages on the fly so it can render the new content if the database has changed.

There's a third kind, though. It is possible to generate static websites. This works for sites like blogs, where content is added from time to time, but it does not need to react to changes every second or minute. I find this is usually true for most small companies websites. It's common for these small companies to have their own design and hire a programmer who can connect the website to a database and being able to modify products, posts and other information, even when these changes rarely occur.

This third case is my use case. I create posts once every a few weeks or even months, so it's not really necessary for this website to be able to render new content every second. Besides content rendering, my website would not have any dynamic content, so I decided to look for alternatives.

There are multiple alternatives available and [this list](https://jamstack.org/generators/) is good as a starting point. Each one of these options has been written in a specific programming language and uses a template language for its pages or posts. Since I have been developing with [Node.js](https://nodejs.org) for a while, I can filter these options to use JavaScript as the programming language. Also, I have been writing all my documents in [Markdown](https://en.wikipedia.org/wiki/Markdown) for a while (topic for another post), so I can filter for that as well.

I didn't try all of them, but I finally chose [eleventy](https://www.11ty.dev/). It's written in JavaScript, supports markdown and has blog templates that I can use.

## Hosting

Now that I know my website will be static (and generated), I need at least a hosting platform that can serve these static files and also point the DNS to the right server.

My choice for DNS server is DigitalOcean. Even when they are more famous for their virtual servers, they offer [free DNS hosting](https://www.digitalocean.com/docs/networking/dns/), which is perfect for my goal to have a cheap solution.

For the hosting itself, AWS offers a very good bundle. [S3](https://aws.amazon.com/s3/) is their object storage, which also has a [static website feature](https://docs.aws.amazon.com/AmazonS3/latest/userguide/WebsiteHosting.html). Of course, this alone will not handle the right address, but we can mix this with [CloudFront](https://aws.amazon.com/cloudfront/), their CDN, and now I'm good to have my own website with my own domain. [This post from the AWS blog](https://aws.amazon.com/blogs/networking-and-content-delivery/amazon-s3-amazon-cloudfront-a-match-made-in-the-cloud/) has more details about the use of S3 + CloudFront. CloudFront even offers a free [TLS](https://en.wikipedia.org/wiki/Transport_Layer_Security) (AKA SSL) certificate, so, more *"free"* stuff!

## Deploying

Assuming we have successfully configured the DNS in DigitalOcean, storage in AWS S3 and our endpoint in CloudFront (which is out of the scope of this post), we should be ready to deploy our website.

### Generate

The first step is to generate the website itself. We can achieve this by moving to the directory where our eleventy website is and run `npx @11ty/eleventy`.

### Sync to S3

Now that the website is generated, we can use the [AWS CLI](https://aws.amazon.com/cli/) to sync our output directory (`_site`) to S3 (Configuring AWS CLI is out of the scope of this post as well... just google it).

In order to sync, we can run:

```
aws s3 cp \
    --recursive \
    --acl public-read \
    --region us-east-1 \
    ./_site/ \
    s3://BUCKETNAME
```

Here's the explanation:

- `--recursive` ensures that all the directories and files inside `_site` will be synced to S3
- `--acl public-read` allows external users (including CloudFront) to read these files
- `--region us-east-1` is just the region where I created the S3 bucket
- `./_site/` is my local directory
- `s3://BUCKETNAME` is the S3 URL. In my example, the bucket is called *`BUCKETNAME`* but of course you will have to replace it with the right S3 bucket name.

This process is necessary every time the content changes. In my case, that's every a few weeks or months. In your case, it could be daily or yearly.

In order to make sure CloudFront invalidates its cache and the new content is served, I run this command after the sync process is completed:

```
aws cloudfront create-invalidation \
    --distribution-id MYDISTRIBUTIONID \
    --paths '/*'
```

You can find your distribution ID in the CloudFront section of the AWS console.

That's it. Now we have a website up and running!

## Automate the deployment process

I don't want to remember running these commands every time. I also want to save my content somewhere else, so I'm using GitHub for this.

GitHub offers the hosting for the source code but also, by using [GitHub Actions](https://github.com/features/actions), I can trigger the deployment process to S3 and CloudFront.

You can see my full website code here: [https://github.com/juancri/web](https://github.com/juancri/web)

The GitHub action workflow is [here](https://github.com/juancri/web/blob/master/.github/workflows/main.yml).

Now all I have to do is to push to master and GitHub will automatically generate the website, sync with S3 and invalidate the CDN cache.

***NOTE:*** I have had a few problem with this last cache invalidation step for a few weeks. Two options now are to wait for the cache to invalidate by its TTL or to run the invalidate command on my own computer.

## Summary

Now, I have a website that I can easily modify, it has its own address and TLS certificate.

Here is a summary of the services I am using:

|Service        |Feature        |Monthly cost
|---------------|---------------|------------
|GitHub         |Code hosting   |$0.00
|AWS S3         |Object storage |$0.20
|AWS CloudFront |CDN            |$0.00
|**TOTAL**      |               |$0.20

***NOTE:*** These are the current and maximum costs, since I have more buckets and other services on AWS. Also, not sure why CloudFront is not charging me since I think I don't have access to the free tier anymore, but you can see the pricing [here](https://aws.amazon.com/cloudfront/pricing/).

