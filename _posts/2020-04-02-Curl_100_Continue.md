---
title: DevOps -- Curl Expect 100 HTTP Header
---



在使用curl做POST的时候, 当要POST的数据大于1024字节的时候, curl并不会直接就发起POST请求, 而是会分为俩步,



CURL在POST请求时，如果Request Body大于1024， curl不会直接发起POST请求，而是分成2步(See [1](http://www.w3.org/Protocols/rfc2616/rfc2616-sec8.html#sec8.2.3))：
* 发送一个请求，带了头Expect:100-continue，询问Server是否接受数据
* 接收到Server返回的100-continue应答以后, 才把数据POST给Server



如果不希望CURL做这个动作，可以在CURL进行POST时，手动添加-H "Expect:" ,禁用这个动作。



[1] http://www.w3.org/Protocols/rfc2616/rfc2616-sec8.html#sec8.2.3
