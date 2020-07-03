---
title: Golang -- update mod
---

有时需要升级特定的go mod, 可以先使用 go list -m all 查询所有的go mod。
然后挑选要升级的模块，例如:
> go get github.com/LipingMao/trackingId

然后使用go mod vendor更新到vendor中即可。
