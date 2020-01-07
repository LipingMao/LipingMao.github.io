---
title: DevOps -- GeoIP with logstash and kibana
---



> 业务通常需要分析客户端的IP分布情况（地域情况，city，运营商等信息），由于我们使用了ELK套件管理Log，在ELK中Logstash可以使用GeoIP分析IP。



#### 日志的收集流程

目前我们使用Filebeat收集应用日志，filebeat会发送到kafka中作为中间缓冲存储，logstash从Kafka中将数据取出，存放在ElasticSearch中存放，交给Kibana做显示。

在filebeat收集应用产生的日志时，会有特定字段用于表示客户端IP地址，当Logstash从kafka中取出数据时，Logstash可以配置filter，我们需要在logstash中安装geoip的插件，安装步骤参考[1](https://yq.aliyun.com/articles/504364) 。

安装完成后，在配置类似如下filter：

```
if [module] == "gslb" {
        if [metrics.gslb][clientip] {
          geoip{
            source => "[metrics.gslb][clientip]"
            target => "[metrics.gslb][geoip]"
          }
        }
      }
```



通过filter后，会生撑geoip相关的内容：

```
...
    "clientip": "54.222.190.251",
      "geoip": {
        "continent_code": "AS",
        "country_code3": "CN",
        "timezone": "Asia/Shanghai",
        "country_code2": "CN",
        "latitude": 39.9288,
        "region_name": "Beijing",
        "location": {
          "lat": 39.9288,
          "lon": 116.3889
        },
        "region_code": "BJ",
        "longitude": 116.3889,
        "country_name": "China",
        "ip": "54.222.190.251",
        "city_name": "Beijing"
      }
...
```



在logstash中有这些信息后，Kibana上面可以配置Coordinate Map，可以看到client IP的地域热点图，效果如下：

![image-20200107143527362](_posts/picture/image-20200107143527362.png)





[1] https://yq.aliyun.com/articles/504364