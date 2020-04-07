---
title: DevOps -- Memory Buffer&Cache
---


在查看内存时，我们会看到有buff & cache，两个会有点迷惑：
```
procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
 1  0      0 854644      0 3205588    0    0     0     0  440  715  0  1 99  0  0
 0  0      0 854644      0 3205588    0    0     0     0  408  637  0  0 100  0  0
 0  0      0 854644      0 3205588    0    0     0   180  619  934  1  1 98  0  1
```

简单来说：
Buffer 是对磁盘数据的缓存，而 Cache 是文件数据的缓存，它们既会用在读请求中，也会用在写请求中。

如果直接对磁盘操作的读写，就会记录在Buffer中，对文件系统进行的读写，就会记录在Cache中。
