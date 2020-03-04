---
title: DevOps -- Route53 Query Log
---



> Route53 可以将Query Log存放起来，后期用于对DNS请求进行分析。



Route53可以简单的使用[1]去配置，将route53发送到cloudwatch中。然后从CloudWatch中使用以下示例查看最近所有responseCode有错的数据：

```
fields @timestamp, queryName, resolverIp, responseCode
| sort @timestamp desc
| filter responseCode!="NOERROR"
| limit 200
```



可以得到类似以下返回用于分析客户请求DNS情况：

![image-20200304235808863](https://raw.githubusercontent.com/LipingMao/LipingMao.github.io/master/_posts/picture/image-20200304235808863.png)






[1] https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/query-logs.html
