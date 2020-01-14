---
title: Golang -- Gin TrackingId
---



> 不同系统间相互调用时，希望能有一个Tracking ID(或者叫Request ID)定位是哪个请求的。业界比较有名的是zipkin，目前我们并没有使用zipkin，当前实现时，仅使用在Header中带一个Tracking Id的简单方式来定位请求。



以Gin为例，这意味着需要在接收请求的时候判断一下Tracking ID是否已经存在，如果存在，则此调用，以及后续调用都是用这个tracking id，如果不存在，则生成一个tracking id放入头部中。使用Gin Middleware非常容易实现。[trackingId](https://github.com/LipingMao/trackingId)



```
package trackingId

import (
	"github.com/gin-gonic/gin"
	"github.com/google/uuid"
)

const DefaultHeader = "Tracking-Id"

func NewTrackingId() string {
	return uuid.New().String()
}

// Gin Middleware with default header
func TrackingId() gin.HandlerFunc {
	return TrackingIdWithCustomizedHeader(DefaultHeader)
}

// Gin Middleware with cusomized header
func TrackingIdWithCustomizedHeader(head string) gin.HandlerFunc {
	return func(c *gin.Context) {
		tId := c.GetHeader(head)
		// Generate TrackingID if not exist
		if tId == "" {
			tId = NewTrackingId()
			c.Header(head, tId)
		}

		// Set in Context
		c.Set(head, tId)
		c.Next()
	}
}
```



在middleware中查看tracking Id的header是否存在，如果不存在就添加一个header。后续在日志可以从gin的context获取tracking Id的值，例如c.GetString(trackingId.DefaultHeader)。如果想自定义Header，在初始化加载middleware时，使用TrackingIdWithCustomizedHeader("X-My-Tracking-ID-Header")。

