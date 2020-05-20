---
title: DevOps -- OverSea Speed up
---



### 目前海外加速方案：



![image-20200520224553248](https://raw.githubusercontent.com/LipingMao/LipingMao.github.io/master/_posts/picture/image-20200520224553248.png)

在海外部署RN节点，海外客户通过RN接入PanoCloud，RN和国内的DC走内网专线通讯。达到海外加速的效果。

这里的假设是：

1) 海外客户接入海外Pop DC的速度都不错。目前我们使用HK作为我们的海外Pop DC

2) 使用专线打通Pop DC 和国内Media DC，这是为了稳定的走过GFW。



这个方案在腾讯执行时，Pop DC使用的是HK，Media 使用的是SH。 其中海外专线使用的是腾讯提供的CCN。

腾讯的CCN还是比较灵活，他可以任意连接腾讯云的Region。并且带宽费用支持95后付费， 你可以按需付费，并且不需要预付费和规划好网络带宽容量。



###  这个方案存在以下限制：

1） 我们假设了海外Pop DC是离客户比较近的。这点并不是总是成立的，当然可以通过增加多个Pop节点的方式，提高覆盖率，然后在海外客户接入时，GSLB考虑就近分配RN接入节点。这样做带来的Trade off是，我们需要部署多个Pop DC，在腾讯带宽是95后付费的，可能还可以，如果在其他云，预付费包专线带宽的话，就很难规划了。但是多个Pop DC会增加成本。

2） 当跨云时，每个云上都需要有专线服务，如果没有专线，就需要使用RN跳多条（例如腾讯云的HK的RN代理到腾讯云国内的RN，再转发到其他云的DC）。