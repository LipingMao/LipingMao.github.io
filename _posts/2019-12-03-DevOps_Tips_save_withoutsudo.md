---
title: DevOps -- Save without sudo
---

有时用非root权限vim打开后，保存时没有root权限无法保存，可以使用以下命令在退出时获取sudo权限。
```
:w !sudo tee %
```

PS：就是这么短。 
