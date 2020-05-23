---
title: DevOps -- DNS Resolv
---

resolv.conf中的Options默认的配置是timeout 5s , 重试3次，这个值对于应用来说时间过长，通常可以使用以下优化配置：
```
options timeout:1 rotate
```

阿里这边的默认配置并不是特别好：
```
options timeout:2 attempts:3 rotate single-request-reopen
```

他会等待6s左右才会尝试其他ns。rotate代表“随机”性选取一个ns作为第一个尝试的服务器，而不是每次都从上到下。
