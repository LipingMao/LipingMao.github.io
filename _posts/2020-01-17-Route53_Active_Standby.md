---
title: DevOps -- Route53 Active-Standby
---



在Route53上可以配置Active-Standby的DNS用于服务的failover。



首先配置healthcheck, 可以配置TCP/HTTP/HTTPS的健康检查条目，以HTTP为例：



![image-20200123171531880](https://raw.githubusercontent.com/LipingMao/LipingMao.github.io/master/_posts/picture/image-20200123171531880.png)

可以看到目前支持： 配置URL ， 配置监控周期，Failure周期， 返回String等



在等待一段时间后，可以看到Health check的状态变为Healthy：

![image-20200123172002102](https://raw.githubusercontent.com/LipingMao/LipingMao.github.io/master/_posts/picture/image-20200123172002102.png)



创建一个Failover类型的DNS记录，首先创建Primary类型的记录，绑定刚刚创建的健康检查：

![image-20200130114204209](https://raw.githubusercontent.com/LipingMao/LipingMao.github.io/master/_posts/picture/image-20200130114204209.png)

创建一个Secondary的记录：

![image-20200130114702398](https://raw.githubusercontent.com/LipingMao/LipingMao.github.io/master/_posts/picture/image-20200130114702398.png)



当健康检查正常时，可以看到返回的结果是1.2.3.4（Primary）：

```
nslookup limao-global.pano.video
Server:		192.168.0.1
Address:	192.168.0.1#53

Non-authoritative answer:
Name:	limao-global.pano.video
Address: 1.2.3.4
```



将健康检查异常，大约1.5分钟后（这是由于设置的检查周期是30s，错误次数是3次），dns failover到了Secondary：

```
➜  ~ curl -v http://13.52.219.197:80
* Rebuilt URL to: http://13.52.219.197:80/
*   Trying 13.52.219.197...
* TCP_NODELAY set
* Connection failed
* connect to 13.52.219.197 port 80 failed: Connection refused
* Failed to connect to 13.52.219.197 port 80: Connection refused
* Closing connection 0
curl: (7) Failed to connect to 13.52.219.197 port 80: Connection refused

➜  ~ nslookup limao-global.pano.video
Server:		192.168.0.1
Address:	192.168.0.1#53

Non-authoritative answer:
Name:	limao-global.pano.video
Address: 5.6.7.8
```



可以看到当Health check失败的时候，DNS自动failover到了Secondary。

这个检查周期是30s * 3次=90s，目前AWS支持最短的检查周期是10s，最短可以设置检查2次失败后转移，可以将时间下降到20s左右。实际应用时，更低的failover也没有太大帮助，因为在很多地方运营商会将DNS的最小TTL改为60s（即使你设置更低的TTL也没有左右）。

