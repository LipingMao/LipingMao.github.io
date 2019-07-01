---
title: AWS -- CloudTrail
---

## What it is

People need to do audit for who do what at when, CloudTrial is helping to do this. It supports to record the audit for users in all Region. It store recently 90 days data by default, if you need more, it can be configured to store in S3. Here is [user guide](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-user-guide.html)


## How to configure

It support to configure in console, cli. It can use Terraform to configure as well, here is [terraform doc](https://www.terraform.io/docs/providers/aws/r/cloudtrail.html)


## Price

It is free to have recently 90 days record, it needs to pay if you store it longer in S3 typicaly. Here is the [Price](https://aws.amazon.com/cloudtrail/pricing/)


