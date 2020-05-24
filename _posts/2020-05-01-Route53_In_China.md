---
title: DevOps -- Route53 in China
---



> AWS国内区域最近上线了Route53服务，简单测试了一下。



创建了公网托管域之后，在国内的Route53生成了6个NS服务器：

```
ns-1047.awsdns-cn-01.net. 
ns-intl-3268.awsdns-cn-12.cn. 
ns-intl-189.awsdns-cn-11.com. 
ns-2511.awsdns-cn-28.biz. 
ns-3268.awsdns-cn-12.cn. 
ns-189.awsdns-cn-11.com.
```



从IP地址上看是2个北京Region的地址，2个宁夏Region的地址，2个海外分别是北美/欧洲：

| 域名/IP                      | 获取的IP地址                                                 | IP的物理位置                      |
| ---------------------------- | ------------------------------------------------------------ | --------------------------------- |
| ns-intl-3268.awsdns-cn-12.cn | [52.46.180.196](http://ip.tool.chinaz.com/?ip=52.46.180.196) | 北美， 美国Oregon                 |
| ns-intl-189.awsdns-cn-11.com | [52.46.184.189](http://ip.tool.chinaz.com/?ip=52.46.184.189) | 欧洲， 德国Frankfurt              |
| ns-189.awsdns-cn-11.com      | [52.82.176.189](http://ip.tool.chinaz.com/?ip=52.82.176.189) | 宁夏中卫 Amazon数据中心           |
| ns-3268.awsdns-cn-12.cn      | [54.222.36.196](http://ip.tool.chinaz.com/?ip=54.222.36.196) | 北京市 亚马逊(Amazon)公司数据中心 |
| ns-1047.awsdns-cn-01.net     | [52.82.180.23](http://ip.tool.chinaz.com/?ip=52.82.180.23)   | 宁夏中卫 Amazon数据中心           |
| ns-2511.awsdns-cn-28.biz     | [54.222.33.207](http://ip.tool.chinaz.com/?ip=54.222.33.207) | 北京市 亚马逊(Amazon)公司数据中心 |



这些权威服务器的地址在国内访问时时延在十几ms到几十ms之间。而国内访问两个海外权威服务器国内访问都需要200-300ms。





