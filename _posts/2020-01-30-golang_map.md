---
title: DevOps -- golang map concurrent save
---

性能测试时发现以下错误：
```
time="2020-02-08T03:57:58Z" level=info msg="Success to schedule(cid: 39109890077150976, servers: [{dc2 MEDIA_SERVER_dc2_18 [] []}], docShow: {\"audio\":{\"aecType\":3,\"audioMode\":3,\"capType\":5,\"farNG\":300,\"nearNG\":150,\"nearPreG\":1.0,\"nsType\":0,\"plyType\":0},\"video\":{}})"
time="2020-02-08T03:57:58Z" level=info msg="Send HTTP response, Tracking-Id: eb32bae0-56de-40f4-8b58-62beb212381c, Status code: 200"
time="2020-02-08T03:57:58Z" level=info msg="Event Process Success. (cid: 39109886239362816, event:{Close dc1 MEDIA_SERVER_dc1_41})"

fatal error: concurrent map writes

goroutine 1115 [running]:
runtime.throw(0xbf645f, 0x15)
/usr/local/go/src/runtime/panic.go:774 +0x72 fp=0xc0008d52f8 sp=0xc0008d52c8 pc=0x42f7a2
runtime.mapassign_faststr(0xb06b80, 0xc0001fd5f0, 0xc000474080, 0x12, 0xc00071d388)
/usr/local/go/src/runtime/map_faststr.go:211 +0x417 fp=0xc0008d5360 sp=0xc0008d52f8 pc=0x414af7
gslb/pkg/gslb/metrics.GSLBLoad.AddNodeLoad(0xc0001fd5f0, 0x0, 0x0, 0x0, 0xc00083a0c8, 0x3, 0xc000474080, 0x12, 0xc00083a0e0, 0xc, ...)
/go/src/gslb/pkg/gslb/metrics/metrics.go:183 +0x1d3 fp=0xc0008d54d8 sp=0xc0008d5360 pc=0xa49ed3
gslb/pkg/gslb/metrics.RecordNodeLoad(0xc00083a0c8, 0x3, 0xc000474080, 0x12, 0xc00083a0e0, 0xc, 0x1182170, 0x1, 0xc00083a0ec, 0x3)
/go/src/gslb/pkg/gslb/metrics/metrics.go:163 +0xfa fp=0xc0008d5550 sp=0xc0008d54d8 pc=0xa49cea
gslb/pkg/gslb/handlers.NodeHandlers.NodeHeartBeat(0xc00020fe10)
/go/src/gslb/pkg/gslb/handlers/handler_node.go:340 +0x8ee fp=0xc0008d5740 sp=0xc0008d5550 pc=0xa523de
```

是“concurrent map writes”。 这是因为自定义了一个记录metrics的map，在gin中发生了并发写入的动作。
go语言中，map并不是一个开箱即用的结构，他并不是线程安全的。因此可以在访问map时加入读写锁即可。
