---
title: Golang -- update go dep to go mod
---

在1.11版本以后的Golang官方已开始支持Go mod，由于之前的项目使用的是go dep，因此需要将其convert到go mod。大致以下过程：

```
// 初始化go mod，go mod会从go dep的依赖中导入初始化的依赖版本
go mod init

// 更新依赖版本
go mod tidy

// 删除vender
rm -fr vendor

// 使用go mod管理vendor
go mod vendor

// 使用vendor模式测试
go build -mod=vendor ...
go test  -mod=vendor ...

// 删除Gopkg相关
rm -f Gopkg.lock Gopkg.toml
```

注意：
1. 由于众所周知的原因，国内下载依赖时可能会有网络问题，在go build时还是使用vendor比较好，因此我使用了vendor模式。
2. 使用vendor模式后，go build/test等命令时，使用-mod=vendor。

