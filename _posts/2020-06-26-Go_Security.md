---
title: Golang -- Protect code
---

私有化场景需要防护反编译，对于Go语言可以删除调试符号等方式，加大反编译难度：

> go build -ldflags ”-s -w“ <package>


[1] https://zhuanlan.zhihu.com/p/26733683
[2] https://www.anquanke.com/post/id/170332
