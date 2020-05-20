---
title: DevOps -- OverSea AIA Speed up
---



### 如果使用AIA加速的方案：

![image-20200520224843194](https://raw.githubusercontent.com/LipingMao/LipingMao.github.io/master/_posts/picture/image-20200520224843194.png)

这种方案直接在Media DC部署带AIA加速的RN节点。 利用Anycast加速过的RN节点可以直接部署到Media DC，海外客户访问RN时，已经被AIA加速，RN和MS/RTMS在同一个DC，所以这种情况下可以直接走内网访问。



这个方案利用了AIA的加速能力，相对Pop节点的方案来说可以减少节点部署。

AIA本身利用了腾讯云的加速能力，会比从HK固定的入口效果要好。



如果某些情况下，云上没有专线能力，利用AIA也可以跨云加速。（例如，如果有物理数据中心，没有买国际专线，或者在有的云上，规划专线实在太贵/容量不好规划的情况）可以使用以下方式进行加速：



![image-20200520224923368](https://raw.githubusercontent.com/LipingMao/LipingMao.github.io/master/_posts/picture/image-20200520224923368.png)

