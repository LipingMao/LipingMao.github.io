---
title: Cloud -- Object Storage
---



## 对比：

| 厂商        | 标准存储（GB/月） | 外网下行流量（GB） | CDN回源流出流量 | API/工具支持 |
| :---------- | :---------------- | :----------------- | :-------------- | :----------- |
| 七牛云      | 0.148             | 0.29               | 0.15            | 较好         |
| 腾讯云      | 0.118             | 0.5                | 0.15            | 较好         |
| 阿里云      | 0.12              | 0.5（0-8AM, 0.25） | 0.15            | 较好         |
| AWS（中国） | 0.1755            | 0.933              | N/A             | 最好         |



## 备注：

1. AWS网络下行费用最大，七牛云网络最低。
2. AWS API生态最好。
3. 存储费用基本差不多。
4. 腾讯和阿里各方面感觉差不多。



综合情况下，AWS在国内没有费用优势，CDN国内也支持的一般。腾讯和阿里两家差不多。七牛综合费用，API，CDN能力要好一些，因此优先考虑七牛云。

## Refer:

AWS(中国): https://www.amazonaws.cn/s3/pricing/

腾讯云：https://buy.cloud.tencent.com/price/cos

阿里云： https://www.aliyun.com/price/product?spm=5176.8030368.1058477.26.b4573aa4DVFZLz&aly_as=5rdMK2-m#/oss/detail

七牛云：https://portal.qiniu.com/financial/price