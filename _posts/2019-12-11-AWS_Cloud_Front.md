---
title: AWS -- Enable HTTPS for S3 Static Website
---



直接使用AWS S3 Host Static Website时有两个问题：

* 没有enable https
* 国内访问慢



解决第一个问题需要有证书，AWS的Cert Manager可以免费申请证书（但不提供下载，因此仅能在AWS上使用）。解决第二个问题可以利用cloudfront。 Cloudfront是AWS提供的CDN功能，可以将S3设置为源站，推送到网络边缘，加速访问。配置的过程非常简单，不重复造轮参考[1]即可。



#### 测试效果：

以pano.video为例，enable 后可以通过https://pano.video 访问，查看访问速度首页打开速度比之前快了，之前下载build.js大约20-30KB/s，需要30-60s，加速后速度可以到300KB+， 需要2-3s

```
➜  _posts git:(master) ✗ wget http://pano.video/build.js
--2019-12-17 10:12:48--  http://pano.video/build.js
正在解析主机 pano.video (pano.video)... 99.86.144.8, 99.86.144.27, 99.86.144.31, ...
正在连接 pano.video (pano.video)|99.86.144.8|:80... 已连接。
已发出 HTTP 请求，正在等待回应... 301 Moved Permanently
位置：https://pano.video/build.js [跟随至新的 URL]
--2019-12-17 10:12:49--  https://pano.video/build.js
正在连接 pano.video (pano.video)|99.86.144.8|:443... 已连接。
已发出 HTTP 请求，正在等待回应... 200 OK
长度：1012871 (989K) [application/javascript]
正在保存至: “build.js”

build.js               100%[=========================>] 989.13K   379KB/s  用时 2.6s

2019-12-17 10:12:53 (379 KB/s) - 已保存 “build.js” [1012871/1012871])
```



查看使用的Cloudfront IP：

```
➜  _posts git:(master) ✗ nslookup pano.video
Server:		10.0.128.173
Address:	10.0.128.173#53

Non-authoritative answer:
Name:	pano.video
Address: 99.86.144.8
Name:	pano.video
Address: 99.86.144.31
Name:	pano.video
Address: 99.86.144.27
Name:	pano.video
Address: 99.86.144.101
```



可以看到国内使用的CloudFront时延大概70-80ms，ipip查询地址是在韩国：

```
➜  _posts git:(master) ✗ ping 99.86.144.8
PING 99.86.144.8 (99.86.144.8): 56 data bytes
64 bytes from 99.86.144.8: icmp_seq=0 ttl=238 time=77.702 ms
64 bytes from 99.86.144.8: icmp_seq=1 ttl=238 time=74.804 ms
64 bytes from 99.86.144.8: icmp_seq=2 ttl=238 time=72.534 ms
64 bytes from 99.86.144.8: icmp_seq=3 ttl=238 time=70.732 ms
```



从亚洲访问主要访问AWS的亚洲区：

```
pano.video服务器iP：
当前解析：
日本 东京13.32.52.38
印度 马哈拉施特拉 孟买13.33.169.48
日本 东京54.192.109.248
美国13.224.161.68
日本 东京99.84.143.71
新加坡54.192.151.11
新加坡54.192.151.3
日本 东京54.192.109.12
新加坡13.35.19.119
```



### Tips：

* 国内直接访问美国速度带宽和速度惊人(20k+)
* 海外加速必要性



[1] https://www.freecodecamp.org/news/simple-site-hosting-with-amazon-s3-and-https-5e78017f482a/