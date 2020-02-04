---
title: Cloud -- Peering Connection
---



## 对等连接的定义：

对等连接（Peering Connection）可以提供跨区域的内网连接功能。主要提供以下能力：

1） 跨VPC的内网通信能力。

2）使用对等连接时，内网间相互通信走的是腾讯云内部网络，不走公网，相当于专线。



举例来说：

假设VPC1和VPC2分别在上海和香港，VPC1和VPC2建立了对等连接。

其中：

VPC1中：

VM1内网地址为10.0.0.1，外网为66.0.0.1

VM2内网地址为10.0.0.1，外网为66.0.0.2

VPC2中：

VM3内网地址为10.1.0.3，外网为77.0.0.3

VM4内网地址为10.1.0.4，外网为77.0.0.4

![image-20200204231356947](https://raw.githubusercontent.com/LipingMao/LipingMao.github.io/master/_posts/picture/image-20200204231356947.png)

则，

以下是走公网的场景：

当VM1 访问 77.0.0.3（VM3的外网地址时）， VM1会走公网网络访问到77.0.0.3，VM3看到的VM1的地址是VM1的外网地址66.0.0.1 。VM3回包时会走公网返回给66.0.0.1。



以下是走对等连接的场景：

当VM2访问10.1.0.3（VM3的内网地址时），由于VPC1和VPC2建立了对等连接，VPC1会使用腾讯云内网最终转发到VPC2，VM3看到的时VM1的内网地址10.0.0.1， 因此VM3回包时，直接返回的时VM1的内网地址10.0.0.1。同样会走VPC2的对等连接线路。





## 对等连接的限制：

对等连接互通性不传递

对等连接能使 VPC 两两建立互联，但是这种互通关系不发生传递。
例如，如下图所示，VPC 1 与 VPC 2 建立了对等连接，VPC 1 和 VPC 3 也建立了对等连接。然而由于对等连接的不传递性，VPC 2 和 VPC 3 的流量不能互通。

![image-20200204231411364](https://raw.githubusercontent.com/LipingMao/LipingMao.github.io/master/_posts/picture/image-20200204231411364.png)

因此：

当有多个VPC时，必须要Fullmash的建立对等连接。如果有N个VPC，需要建立N*(N-1)/2条链路，两两互联。



## 费用：

![image-20200204231544264](https://raw.githubusercontent.com/LipingMao/LipingMao.github.io/master/_posts/picture/image-20200204231544264.png)

注意：

月95带宽峰值：每5分钟采集一次，取5分钟内两地域互通带宽峰值作为一个统计点，当月所有统计点从高到低排序，去掉最高5%的统计点后，取剩下的点中的最大值，记作该月的月95带宽峰值。

此处带宽取出入带宽中高的那个。



## Refer:

[1] https://cloud.tencent.com/document/product/553

[2] https://cloud.tencent.com/document/product/553/18829