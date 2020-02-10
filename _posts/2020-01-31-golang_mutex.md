---
title: DevOps -- Map protected by mutex caused panic: concurrent map writes
---

在对map进行加锁时，碰到了和以下一样的问题：
https://github.com/golang/go/issues/20060

应该使用指针receiver，否则会拷贝锁。
对receiver类型需要进一步加深理解。

另外，也可以使用sync包中提供的sync.Map，这是可以开箱即用，线程安全的。
