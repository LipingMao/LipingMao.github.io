---
title: DevOps -- AWX Launch with Extra Vars
---



>  AWX可以通过API控制Job Template的创建，我们可以为不同环境，不同的任务定义templates/inventories。在运行Job Template时，可以调用Post方法进行使用。



例如：

```
curl --location --request POST 'http://awx.pano.video/api/v2/job_templates/${ID}/launch/' --header 'Authorization: Basic XXXX' --header 'Content-Type: application/json' --data-raw '{"extra_vars":{"dc_name":"DC", "env":"Env", "provider":"TestP"}}'
```



注意的是，在创建Template时，需要勾选PROMPT ON LAUNCH：

![image-20200426214359919](https://raw.githubusercontent.com/LipingMao/LipingMao.github.io/master/_posts/picture/image-20200426214359919.png)



