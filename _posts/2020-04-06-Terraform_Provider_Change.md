---
title: DevOps -- Terraform Provider Change
---

> 修改Terraform Provider的源码后，需要编译一下，然后替换掉module以生效。


通过以下命令可以在mac上面对编译：
```
export GOOS=linux
export GOARCH=amd64
make build
```

替换掉Terraform目录下的.terraform下plugin的文件。重新terraform init即可。
