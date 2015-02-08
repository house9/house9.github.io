---
layout: post
title: "AWS IAM policy for uploading SSL certificates"
date: 2015-02-08 10:04
comments: true
categories: AWS
---

The following [IAM](http://aws.amazon.com/iam/) policy can be used to upload SSL certificates to [AWS](http://aws.amazon.com/) for use by [cloudfront](http://aws.amazon.com/cloudfront/) or [ELB](http://aws.amazon.com/elasticloadbalancing/).

```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "iam:DeleteServerCertificate",
        "iam:UploadServerCertificate",
        "iam:ListServerCertificates",
        "iam:GetServerCertificate"
      ],
      "Resource": "*"
    }
  ]
}
```

This policy gives full access to certificate management and nothing else.

## Resources

* [https://bryce.fisher-fleig.org/blog/setting-up-ssl-on-aws-cloudfront-and-s3/](https://bryce.fisher-fleig.org/blog/setting-up-ssl-on-aws-cloudfront-and-s3/)
* [https://www.tiredpixel.com/2014/09/01/secure-static-websites-using-aws-s3-ssl-aws-cloudfront-and-aws-route-53/](https://www.tiredpixel.com/2014/09/01/secure-static-websites-using-aws-s3-ssl-aws-cloudfront-and-aws-route-53/)