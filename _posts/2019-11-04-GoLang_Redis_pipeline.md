---
title: Redis -- Pipeline
---



> Redis 支持MULT + EXEC实现批量（Pipeline）操作，对比了一下使用Pipeline和不使用Pipeline的性能。

## 性能对比

以下为对比测试的代码，使用了本地MAC容器化安装的Redis，进行10K Set，10K Get操作。分别用Pipeline和非Pipeline两种方式：

```go
package main

import (
	"fmt"
	"github.com/go-redis/redis"
	"strconv"
	"time"
)

func usedTime(what string) func() {
	start := time.Now()
	return func() {
		fmt.Printf("%v took %f ms\n", what, time.Since(start).Seconds()*1e3)
	}
}

func onebyone() {
	defer usedTime("onebyone")()
	rdb := redis.NewClient(&redis.Options{
		Addr: ":6379",
	})

		for i := 0; i <=10000; i++ {
			rdb.Set(strconv.Itoa(i), 100000+i, 0)
		}
		for i := 0; i<= 10000; i++ {
			rdb.Get(strconv.Itoa(i))
		}

	fmt.Printf("Get it done in onebyone!")
}

func pipleline() {
	defer usedTime("pipleline")()

	rdb := redis.NewClient(&redis.Options{
		Addr: ":6379",
	})

	_, err := rdb.Pipelined(func(pipe redis.Pipeliner) error {
		for i := 0; i <=10000; i++ {
			pipe.Set(strconv.Itoa(i), 100000+i, 0)
		}
		for i := 0; i<= 10000; i++ {
			pipe.Get(strconv.Itoa(i))
		}
		return nil
	})
	if err != nil {
		fmt.Printf("Error is %v\n", err)
		panic("Error")
	}
	fmt.Printf("Get it done in pipleline!")
}

func main() {
	pipleline()
	onebyone()
}
```

运行结果如下：

```
Get it done in pipleline!pipleline took 313.173553 ms
Get it done in onebyone!onebyone took 11404.180685 ms
```

可以看到运行结果相差巨大，使用Pipeline大概需要300+ms，不使用则需要11+s。



## 性能差距原因

1. Redis是TCP服务，每次请求有RTT延迟（Round Trip Time），即使使用的是local，时延一般也有0.04-0.08ms的时延。 另外即使走loopback，依然会进入网络协议栈处理，协议栈的中断，上下文切换，调度，等等会造成性能损失。

2. 每次请求会有read/write系统调用，会发生内核态用户态上下文切换，使用pipeline的话，仅发生一次。

   

## Refer

[1] https://redis.io/topics/pipelining

