---
title: DevOps -- DNS Resolve
---


resolv.conf中的Options默认的配置是timeout 5s , 重试3次，这个值对于应用来说时间过长，通常可以使用以下配置减少时间：
```
options timeout:1 attempts:2 rotate
```

注意host上修改后，需要重启容器，容器的/etc/resolv.conf才会重新加载。
