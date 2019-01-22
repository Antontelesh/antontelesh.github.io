---
layout: post
title: "Serve Multiple Websites From One Domain on AWS"
date: 2019-01-22 09:00:00 +0300
categories: aws
---

## Preamble

This post summarizes the spike I had to do for my customer.
We met a requirement to serve multiple static websites from the same domain.
The sites were acting like page-level [microfrontends](https://micro-frontends.org/) that combined together founded a big platform delivering a plethora of features around the same business domain.

Each website is developed by a separate team.
A team should be capable of deploying their part of the platform independently of the others.

Amazon Web Services cloud provider was chosen as a base for all the services under the platform.

## Technical Requirements

Given a domain name `my-platform.com`:

- serve website `A` when navigated to `https://my-platform.com/site/a/`
- serve website `B` when navigated to `https://my-platform.com/site/b/`

## Hosting Websites Separately

Keeping in mind that each website should have it's own release cycle, we quickly decided to start with serving each of them separately, on their own domains.

The idea behind this is that each team is responsible for its own resources and we do not introduce any resource-specific dependencies. We will combine websites later, using HTTP protocol. This should free the teams in terms of choosing technology stack.

The simplest solution to host a static website in Amazon is to leverage the [website hosting](https://aws.amazon.com/getting-started/projects/host-static-website/) feature from the S3 service.

It simply requires two things:

1. Configure S3 bucket to serve its contents as a simple http server.
   <br/>
   This should be done once.
2. Upload assets into this S3 bucket.
   <br />
   This is to be performed on each release (deployment).

When we've done everything we ended up having two static websites at their own domains:

- https://website-a.s3-website-eu-west-1.amazonaws.com
- https://website-b.s3-website-eu-west-1.amazonaws.com

## Merging Websites

During the research we have identified three main approaches each behaving at a different level of abstraction.
Let me describe them one by one.

### Custom Proxy Server

In theory it's possible to start a custom Nginx server in a Fargate container or EC2 instance.
Nginx should be configured to proxy different resources to the respective origins.

We did not choose to use this approach as it looks to low level.
Let's see what other options do we have.

### CloudFront Cache Behaviors

CloudFront allows to route requests to a specific origin by path pattern.

Any website can act as an origin. Path pattern is a glob-like pattern. Each request to CloudFront distribution is validated against the list or path patterns configured to determine the right origin. The origin then is used as a source for the response. It works like a simple routing. Let's configure:

| Path Pattern | Origin                                               |
| ------------ | ---------------------------------------------------- |
| `/site/a/*`  | https://website-a.s3-website-eu-west-1.amazonaws.com |
| `/site/b/*`  | https://website-b.s3-website-eu-west-1.amazonaws.com |

But here's the problem. When we try issuing a request to
<code>https://my-portal.cloudfront.net/<b>site/a/index.html</b></code> it will proxy the request to <code>https://website-a.s3-website-eu-west-1.amazonaws.com/<b>site/a/index.html</b></code>. Our S3 Buckets do not have `/site/a` or `/site/b` folders. Instead, they have `index.html` at the root level.

There are two solutions to this problem:

1. Move files to `/site/a` and `/site/b` folders within S3 Buckets. The downside is that now each website is aware of the final routing, while we want them to be decoupled.
2. Use Lambda@Edge to modify requests before delegating them to CloudFront. This solution prevents coupling of websites to the routing but it's more complex. It requires to register a lambda function, deploy it to its own S3 Bucket, configure it within the CloudFront distribution. [Serverless](https://serverless.com) framework simplifies the work with lambda functions, but still it [does not support](https://github.com/serverless/serverless/issues/3944) Lambda@Edge natively.

### API Gateway

API Gateway is intended to be used mostly for the purposes of JSON API. But this service can be used to proxy requests to your static websites.

With API Gateway it's possible to configure separate resources to act as HTTP proxies. Additional benefit is that each resource can be maintained in a separate CloudFormation stack, thus decoupled from the API Gateway setup itself.

#### Issue with Binary Files

There's one downside with API Gateway that we faced — it fails at handling binary files. Each time we wanted to serve a font file from our website, API Gateway messed with response body and headers and we ended up with corrupted files in the browser.

API Gateway has a special configuration for binary files. It allows you to list all binary mimetypes and will treat all responses that contain `Content-Type` header matching one of the mimetypes from the list as a binary file. But it requires the request to also contain the `Accept` header with the mimetype from the list.

Unfortunately, all requests issued by the browser from a CSS file (`@font-face`, `@import`, `url`, etc.) contain the `Accept` header with the value of `*/*`. It means that if we wanted these requests to be treated as binary we would need to configure `*/*` as binary mimetype. This setting would break all non-binary responses. While this would still work for html/css/js — browsers still can parse these files — it breaks all JSON API hosted on the same API Gateway.

We have identified two ways to fix this.

#### **Use separate API Gateway for JSON API**

Extracting JSON API to a separate domain allows us to configure `*/*` as a binary mimetype. While this approach works it has a hacky smell and does not look like a proper way of solving the problem.

#### **Serve all binary files from a separate domain or even CDN**

This requires maintaining a separate stack for serving binary files. But agreed with having to maintain a separate stack there is a way to take some benefits from it.

For example, creating a separate CDN for serving all binary files together with other assets would positively influence the performance because of advanced caching applied.

## Conclusion

We decided to use the API Gateway approach, because it has more advantages compared to other solutions identified, keeping im mind our requirements. Other teams might choose other options depending on their needs.
