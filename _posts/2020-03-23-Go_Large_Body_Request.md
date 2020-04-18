---
title: Golang -- Large Request Body
---

> 测试时发现当Http Request Body请求比较大时，Gin在handler中处理的时间会急剧加大，但看起来并不是单纯处理请求的时延，本文对于这个问题进行初步分析。


问题简化如下：
* 服务器端使用Golang Gin框架做Restful API Server
* Client在远端发送一个Request Body比较大的Request

现象：
> 哪怕Gin的Handler中只是echo hello world，Handler的响应时间也会随着Request body变大而变大，并且处理时间会从0.X ms急剧增长到几十ms-几百ms。客户端和服务器端并不在一起，相互之间rtt大概是几十ms。

这个现象与之前理解不太一致，之前一直以为服务器会完整获取到HTTP Request之后再进入handler响应，先说结论，最后发现golang的net/http并不是这么处理的，Gin在收到http请求之后就进入了handler，gin会读取request body的前4096字节。之后就会进入handler处理流程，此时服务器的TCP缓存队列会满，服务器端TCP发送给Client端的TCP Window降为0，Client端无法继续发送body，等待服务器端将数据从tcp队列中取出。Gin的Handler中bind request时，会将整个request body读出，此时服务器端的TCP缓存队列会有空余空间，client端才会继续发送body的数据。而Client与Server端之间有RTT时间，根据Request body的大小，会产生若干次RTT才能将Request Body中的内容发送完毕。因此如果看Handler的处理时间，就会有若干个RTT的时间+实际数据处理时间。由于我测试的RTT为几十ms，最终handler的处理时间也就变成了几十ms到几百ms。
