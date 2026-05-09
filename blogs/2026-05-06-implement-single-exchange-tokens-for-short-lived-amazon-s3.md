---
title: "Implement single-exchange tokens for short-lived Amazon S3 presigned URLs with Terraform"
url: "https://aws.amazon.com/blogs/storage/implement-single-exchange-tokens-for-short-lived-amazon-s3-presigned-urls-with-terraform/"
date: "2026-05-06"
author: "Giorgio Pessot"
feed_url: "https://aws.amazon.com/blogs/storage/feed/"
---
Organizations across industries use signed URLs to grant temporary, credential-less access to private resources such as receipts, medical or financial records, legal files, or confidential reports. However, signed URLs can be reused by anyone until they expire, creating security risks if a URL is shared or inadvertently disclosed. This risk can be mitigated by vending the signed URL only when needed and making it as short-lived as possible. Amazon S3 offers presigned URLs for granting time-limited access to private objects without requiring AWS credentials.
