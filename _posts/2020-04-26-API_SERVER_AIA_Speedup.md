---
title: DevOps -- API Server OverSea Speed up
---



**目前方案：**

GEO DNS控制，国内使用上海公网SLB，海外用户走HK pop DC的SLB。（HK pop DC和SH Central DC之间走专线）



**如果使用AIA：**

GEO DNS控制，国内使用上海公网SLB，海外用户走带AIA IP的SLB。 这样他们使用同一组Nginx API GW。





**方案对比：**

a) AIA加速海外比HK的好。

b) 对于api.pano.video来说，可以去掉Pop DC，减少一跳。

