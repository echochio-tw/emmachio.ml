---
layout: post
title: hosting in aws s3
date: 2021-11-12
tags: 圖
---

客戶的 AWS 的 S3 要綁換自己的域名
```
http://hosting.s3.ap-southeast-1.amazonaws.com/
chang to 
my.domain.com
```

上網找到方法有三個 客戶選第一個 測試有成功
```
1. https://havecamerawilltravel.com/amazon-s3-bucket-cname/
  https://docs.aws.amazon.com/zh_tw/AmazonS3/latest/userguide/VirtualHosting.html
  每個bucket都有一個 endpoint ，若 bucket 修改，調整 cname 對應的值即可
  所以改 bucket 可對應域名 這方式只能用 http 不能用 https

2.利用傳統方式安裝 nginx proxy 對 S3 的域名作反向代理, 可用 http 及 https

3.用 CDN 回源方式作代理 可用 http 及 https 
  AWS 推薦  把 S3 來當作靜態網站存取使用 是使用 S3、CloudFront、ACM
  https://docs.aws.amazon.com/zh_tw/AmazonS3/latest/userguide/WebsiteHosting.html
```
