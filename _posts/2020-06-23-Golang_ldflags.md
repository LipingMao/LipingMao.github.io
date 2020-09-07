---
title: DevOps -- Golang ldflags
---

> Golang程序通常需要携带版本信息，代码Git提交版本号等信息，此类信息可以通过Golang在编译时传递参数实现。

例如：
```
Code:
var version string
fmt.Print("Version: %v", version)


Build:
flags="-X 'main.version=v0.0.1'"
go build -ldflags "$flags" ...
```

编译后，程序中version是v0.0.1 。这些参数可以在CI过程中通过变量传递。
