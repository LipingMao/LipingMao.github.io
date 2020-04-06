---
title: Golang -- POST Large Body
---

> 如果一个HTTP请求的Body比较长，那这个请求的Body是全部收好了，在触发handler还是什么时机触发的呢？

这个问题是因为在做性能测试时，发现如果两台机器之间有较大的时延，当request body比较大时，handler在read request body时，会需要很久。这说明handler处理时，request body并不是事先读好的。
使用以下测试用例测试了一下：
```
package main

import (
	"encoding/json"
	"fmt"
	"github.com/gin-gonic/gin"
	"net/http"
	"time"
)

func Now() string {
	return time.Now().UTC().Format(time.RFC3339Nano)
}

type request struct {
	Data json.RawMessage `json:"data"`
}

func main() {
	router := gin.Default()
	router.POST("/data", func(c *gin.Context) {
		var d request
		fmt.Printf("%s: before bind\n", Now())
		if err := c.ShouldBindJSON(&d); err != nil {
			c.String(http.StatusInternalServerError, "Bind JSON Failed")
			return
		}
		fmt.Printf("%s: after bind\n", Now())
		fmt.Printf("Size of Data: %v\n", len(d.Data))
		c.String(http.StatusOK, "hello gin")
	})
	router.Run()
}

```

可以看到，这个测试里面，仅开了一个POST接口，进行bindJSON操作，这个操作内会读取request body，然后print一下data的lenth。测试发现以下结果：
```
[root@officeci001 lipingmao]# ./Test
[GIN-debug] [WARNING] Creating an Engine instance with the Logger and Recovery middleware already attached.

[GIN-debug] [WARNING] Running in "debug" mode. Switch to "release" mode in production.
 - using env:  export GIN_MODE=release
 - using code:  gin.SetMode(gin.ReleaseMode)

[GIN-debug] POST   /data                     --> main.main.func1 (3 handlers)
[GIN-debug] Environment variable PORT is undefined. Using port :8080 by default
[GIN-debug] Listening and serving HTTP on :8080
2020-04-06T12:45:47.799847595Z: before bind
2020-04-06T12:45:47.800047407Z: after bind
Size of Data: 12
[GIN] 2020/04/06 - 12:45:47 | 200 |     229.372µs |       10.0.0.83 | POST     "/data"
2020-04-06T12:45:58.90173365Z: before bind
2020-04-06T12:45:58.901814988Z: after bind
Size of Data: 102
[GIN] 2020/04/06 - 12:45:58 | 200 |     101.835µs |       10.0.0.83 | POST     "/data"
2020-04-06T12:46:19.241241365Z: before bind
2020-04-06T12:46:19.241323473Z: after bind
Size of Data: 902
[GIN] 2020/04/06 - 12:46:19 | 200 |       97.93µs |       10.0.0.83 | POST     "/data"
2020-04-06T12:46:36.936273254Z: before bind
2020-04-06T12:46:36.936515591Z: after bind
Size of Data: 1802
[GIN] 2020/04/06 - 12:46:36 | 200 |     258.585µs |       10.0.0.83 | POST     "/data"
2020-04-06T12:46:49.557171997Z: before bind
2020-04-06T12:46:49.590656011Z: after bind
Size of Data: 3602
[GIN] 2020/04/06 - 12:46:49 | 200 |   33.512743ms |       10.0.0.83 | POST     "/data"
2020-04-06T12:47:10.191230672Z: before bind
2020-04-06T12:47:10.285231561Z: after bind
Size of Data: 14402
[GIN] 2020/04/06 - 12:47:10 | 200 |   94.038166ms |       10.0.0.83 | POST     "/data"
```

Request body len 比较小时，处理速度很快。
Request body len 大概在1802-3602之间时，引入了一个RTT的时延。
到了14402时，引入了大概3个RTT的时延。

因此说明：
1） request body是在read的时候实际获取的。
2） 这个len应该和tcp的滑动窗口，tcp send/receive buffer有关系。

