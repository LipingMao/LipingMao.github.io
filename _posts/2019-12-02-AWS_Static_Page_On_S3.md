---
title: AWS -- Host Static WebSite with S3
---



> 当我们需要设立一个静态网页但又不想自己host EC2实例时，可以利用AWS S3提供的能力，快速构建一个静态网页。此wiki记录了搭建过程。



### Step 1: Register Domain

首先需要一个域名，AWS Route53提供了域名注册服务， $9起可以注册域名



### Step 2: Create an S3 Bucket and Configure It to Host a Website

创建一个S3 Bucket和你想host的域名同名，例如pano.video。

在Properties中选取**Static website hosting**. -> **Use this bucket to host a website**.

注意在设置permissions中，允许Public访问，创建一下Policy：

```
{
   "Version":"2012-10-17",
   "Statement":[{
      "Sid":"AddPerm",
      "Effect":"Allow",
      "Principal":"*",
      "Action":[
         "s3:GetObject"
      ],
      "Resource":[
         "arn:aws:s3:::pano.video/*"
      ]
    }]
}
```

注：将此处pano.video替换为相应域名。



### Step 3,  Another bucket for www.pano.video

类似Step2创建www.pano.video的bucket，不同之处在于**Properties**页面选择**Static website hosting**. -> **Redirect requests**. 到pano.video



 ### Step 4, Upload WebSite

将静态网页上传到S3中。



### Step5, Set up DNS

在Route53中配置pano.video -- A记录，选择Alias Yes，目标选择S3中的bucket。创建www.pano.video一条CNAME记录到pano.video 。



到此即可使用DNS访问静态网页。



[1] https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/getting-started.html#getting-started-find-domain-name