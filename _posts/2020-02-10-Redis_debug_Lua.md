---
title: DevOps -- Redis Lua Debug
---

> Redis Lua脚本写完后可以使用debug程序进行debug。

例如：
 redis-cli --ldb --eval ./a.lua

"--ldb" 就进入了debug模式，在debug模式默认是会回滚操作的，注意不要在线上使用debug模式，可能会卡死Redis。

关于ldb更多，不再造轮子，可以参考：
https://redis.io/topics/ldb
