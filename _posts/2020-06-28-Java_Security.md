---
title: DevOps -- Protect Java code
---

私有化场景需要防护反编译，对于Java语言可以使用以下两种方式，加大反编译难度：

* ProGuard : 代码混淆
* xjar : 将jar包加密，启动时使用go语言程序启动加密后的jar，内存中解密。

[1] https://blog.csdn.net/mengzhengjie/article/details/50420130
[2] https://github.com/core-lib/xjar
