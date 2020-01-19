---
title: DevOps -- Route53 Geo DNS
---



Route53可以配置基于地域的DNS解析。比如，我们希望在海外的客户可以走海外的入口进入我们的服务，在国内的客户从国内进入，就可以利用Geo DNS给客户返回不同的值。



比如设置默认的地理位置返回2.3.4.5:

![image-20200119183142077](https://raw.githubusercontent.com/LipingMao/LipingMao.github.io/master/_posts/image-20200119183142077.png)





给国内的返回为1.2.3.4:

![image-20200119183119985](https://raw.githubusercontent.com/LipingMao/LipingMao.github.io/master/_posts/image-20200119183119985.png)