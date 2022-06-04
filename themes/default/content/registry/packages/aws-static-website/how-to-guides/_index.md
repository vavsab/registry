---
title: AWS Static Website How-to Guides
meta_desc: |
  Tutorials for using the Pulumi AWS Static Website component
layout: how-to
---

## About

This component makes it easy to deploy a static website to s3 along with an optional CloudFront distribution using any of the supported Pulumi programming languages including markup languages like YAML and JSON.

## Resources

This component deploys and configures the following types of infrastructure resources in AWS depending on the inputs provided.

- S3 Bucket
- CloudFront Distribution
- Route53 DNS records
- ACM Certificate

## Simple Example

This is an example of how to use this component in its simplest form. Here, we will just provision an S3 bucket to house the website's contents. This will only provision the bucket and upload the website files to the bucket that are located in the `sitePath` directory.

{{< chooser language "typescript,yaml" >}}
{{% choosable language typescript %}}

```typescript
import * as website from "@pulumi/aws-static-website"

const args =  {
    sitePath: "./website/build",
} as website.WebsiteArgs

const site = new website.Website("website", args);
export const websiteURL = site.websiteURL;
```

{{% /choosable %}}
{{% choosable language yaml %}}

```yaml
name: static-website
runtime: yaml
resources:
  web:
    type: "aws-static-website:index:Website"
    properties:
      sitePath: "./website/build"
outputs:
  websiteURL: ${web.websiteURL}
```

{{% /choosable %}}
{{< /chooser >}}

In the example above we simply created a new static website resource and specified the path to the directory that contains the website files. Running `pulumi up`, will create an S3 bucket and upload the website content to that bucket. Once `pulumi up` completes, the bucket website URL is output to the console.

## S3 + CloudFront Example

This examples shows how to use this component to provision a static website using S3 along with a CloudFront CDN to serve the content. Here, we will provision the S3 bucket to house website content, a CloudFront distribution to serve the content, an ACM certificate to enable serving over HTTP, an s3 bucket to house access logs, and a Route53 record to associate the domain name with the CloudFront distribution. Note: in this example we will be specifying a target domain. If public access is intended, it is assumed you own this domain as well as have a hosted zone configured in Route53 for the domain. For information on how to configure a public hosted zone see this AWS [doc](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/CreatingHostedZone.html).

{{< chooser language "typescript,yaml" >}}
{{% choosable language typescript %}}

```typescript
import * as website from "@pulumi/aws-static-website"

const args =  {
    withCDN: true,
    withLogs: true,
    targetDomain: "dev.my-site.com",
    sitePath: "./website/build",
} as website.WebsiteArgs

const site = new website.Website("website", args);
export const websiteURL = site.websiteURL;
```

{{% /choosable %}}
{{% choosable language yaml %}}

```yaml
name: static-website
runtime: yaml
resources:
  web:
    type: "aws-static-website:index:Website"
    properties:
      withCDN: true
      withLogs: true
      targetDomain: "dev.my-site.com"
      sitePath: "./website/build"
outputs:
  websiteURL: ${web.websiteURL}
```

{{% /choosable %}}
{{< /chooser >}}

In the above example we set the `withCDN` property to true in order to provision the CloudFront distribution. We also set `withLogs` to true in order to provision a bucket to house the access logs. The `targetDomain` specifies the domain we want to use to access the website. This will enable our component to provision an ACM certificate allowing us to serve over HTTPS as well as configure a Route53 DNS record to associate the CloudFront Distribution with the domain we specified. Once `pulumi up` successfully completes, the website can be accessed using at the target domain specified.
