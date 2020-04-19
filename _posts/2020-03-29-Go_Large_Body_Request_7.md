---
title: Golang -- Large Request Body 7
---



 从tcpdump可以看到在1262-1263之间有相差了30+ms，这是因为发送端1263收到了1262的ACK之后才继续发送的数据：

![image-20200419205334083](https://raw.githubusercontent.com/LipingMao/LipingMao.github.io/master/_posts/picture/image-20200419205334083.png)



HTTP完整的Request是由1258，1259，1261，1263，1265组成：

![image-20200419205618110](https://raw.githubusercontent.com/LipingMao/LipingMao.github.io/master/_posts/picture/image-20200419205618110.png)



就此，大致了解了为什么大Request Body会造成这样的延迟。 后续，需要对TCP Windows机制，TCP快速发送，linux Send/Receive buffer梳理一下。