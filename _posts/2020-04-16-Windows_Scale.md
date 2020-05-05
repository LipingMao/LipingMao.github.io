---
title: DevOps -- TCP Windows Scale
---



>本文记录和TCP Windows Scale Option相关内容。



TCP本身是一个可靠的协议，传输的所有内容都需要被对端ACK。然而，如果每传输一个字节就要求对端发送一个ACK，这样的动作显然是不高效的。因此，通常会有一个“窗口大小”决定了TCP发送端可以在对端ACK之前发送多少数据。今天我们来看一看和Windows Size相关的一些内容。



首先看一下TCP Window size 在TCP报文头中的样子：

![image-20200505112821489](https://raw.githubusercontent.com/LipingMao/LipingMao.github.io/master/_posts/picture/image-20200505112821489.png)

上图是一个Wireshark看到的TCP三次握手的SYN包，可以看到在TCP头部有一个2个Byte的字段表示了Windows Size。这两个Byte是“69 03”，对于16进制的“69 03“转化成十进制，就是26883。自然的，它表示了它自己的Windows Size是26883。这个windows size是告诉“对端的”，也就是说，对端在收到自己的ACK之前，最多可以发送26883个字节。



但是，这里有一个问题，TCP的windows size字段最多是2个字节，意味着它的最大值可以是65535。换句话说，对端在收到ACK之前，至多可以发送65535个字节。 显然，这样的窗口大小在延迟比较高时，会大大限制TCP的传输效率。假设在RTT为100ms的场景下，意味着，每100ms最多发送65535个字节对的数据，换句话说1s可以发送655350Byte的数据， 这个发送速度只能达到5mbps左右。显然对于当今动辄10G，百G的网络传输速度，这点速度未免太过龟速。



可能在TCP协议制定之初，没有考虑到网速会增长到如此之快，要解决这个问题最直接的办法自然是将Windows Size Value字段扩展到更多的位数，比如32位。但是修改TCP头部的定义会破坏TCP向下兼容性，因此TCP使用Option字段对其进行扩展，扩展Window Size的字段时Window Scale字段：

![image-20200505144433171](https://raw.githubusercontent.com/LipingMao/LipingMao.github.io/master/_posts/picture/image-20200505144433171.png)



可以看到这个例子中Option字段中的Window Scale是7，它表示了2^7=128。 换句话说，TCP后续的Window Size计算时扩大128倍。这个Option就起到了扩展Window Size上限的功能。在后续的TCP包中，可以看到Window Size会乘以Windows Size Scaling， 在下面的例子中，TCP包头中的Windows Size时253，实际的Window Size(Wireshark中叫做Caculated Window Size)是253 * 128 = 32384：

![image-20200505151420485](https://raw.githubusercontent.com/LipingMao/LipingMao.github.io/master/_posts/picture/image-20200505151420485.png)



关于Window Scale Option以下是需要注意的一些特性：

* Windows Scale的值是在建立TCP的SYN时进行通告的，在TCP三次握手之后，这个值是不能修改的。换句话说，如果使用Wireshark查看TCP流，如果没有抓到SYN包，那只看后续连接，是无法知道实际的Window Size的。

* 在SYN包本身中的Window Size是不计算Window Scale的，换句话说，TCP初始的Window Size不会超过65535 。后续包，可以通过改变TCP头中Window Size的值，来修改Window Size。

* Window Scale本身最大是14。也就是说Window Size理论最大值为2^30 = 1073741824 Byte。如果RTT时延是100ms的话，那单个TCP理论最快传输速度可以达到1073741824 * 8 * 10 / 1024 /1024 / 1024 =  80Gbps。



