---
title: DevOps -- NTP Jitter / Offset drafting on Tencent Cloud
---



> 在腾讯云上，我们发现NTP有时有Jitter / Offset增大的情况。



Prometheus发现NTPD的offset/Jitter忽然增大：

![image-20200310112546667](https://raw.githubusercontent.com/LipingMao/LipingMao.github.io/master/_posts/picture/image-20200310112546667.png)



最后和腾讯云发现，这是由于腾讯云后台对VM进行Live Migration造成的。 Live Migration过程中，会对VM进行Pause，会造成ntpd的offset和jitter。



大概6-8小时后，ntp趋于稳定：

![image-20200310114326103](https://raw.githubusercontent.com/LipingMao/LipingMao.github.io/master/_posts/picture/image-20200310114326103.png)



最后调整了报警阈值：

当offset > 128ms时，发出报警。

当Jitter > 50ms && 持续超过12小时，发出报警。

