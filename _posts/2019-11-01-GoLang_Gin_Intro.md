---
title: Golang -- Gin框架简介
---

> 记录Gin框架简介。

## 什么是Gin？

> Gin is a HTTP web framework written in Go (Golang). It features a Martini-like API with much better performance – up to 40 times faster. If you need smashing performance, get yourself some Gin. [1]

Gin是一个Golang的轻量http web框架，Gin可以方便支持Middleware，路由Group，内置一些Middleware支持Catch Panic，错误管理，内置常用Render等特性。

## Hello Gin

以下是最简单的Gin Hello World：
```
package main

import (
	"github.com/gin-gonic/gin"
	"net/http"
)

func main() {
	// 使用Default生成默认Engine，会带上默认的Middleware，logger和Recovery
	// logger中间件会记录每次API请求的基本信息
	// Recovery会恢复handler中的panic，并产生记录
	r := gin.Default()

	// 定义ping路由，以及处理函数
	r.GET("/ping", func(c *gin.Context){
		c.JSON(http.StatusOK, gin.H{
			"Message": "Pong!",
		})
	})

	// 启动默认http Server开始提供服务
	r.Run()
}
```

## 使用Middleware以及定义Group

以下示例自定义加载了Middleware以及定义了API Group:
```
func main() {
	// Creates a router without any middleware by default
	r := gin.New()

	// Global middleware
	// Logger middleware will write the logs to gin.DefaultWriter even if you set with GIN_MODE=release.
	// By default gin.DefaultWriter = os.Stdout
	r.Use(gin.Logger())

	// Recovery middleware recovers from any panics and writes a 500 if there was one.
	r.Use(gin.Recovery())

	// Per route middleware, you can add as many as you desire.
	r.GET("/benchmark", MyBenchLogger(), benchEndpoint)

	// Authorization group
	// authorized := r.Group("/", AuthRequired())
	// exactly the same as:
	authorized := r.Group("/")
	// per group middleware! in this case we use the custom created
	// AuthRequired() middleware just in the "authorized" group.
	authorized.Use(AuthRequired())
	{
		authorized.POST("/login", loginEndpoint)
		authorized.POST("/submit", submitEndpoint)
		authorized.POST("/read", readEndpoint)

		// nested group
		testing := authorized.Group("testing")
		testing.GET("/analytics", analyticsEndpoint)
	}

	// Listen and serve on 0.0.0.0:8080
	r.Run(":8080")
}
```

## Bind Json

以下示例Bind Json:
```
package main

import (
	"github.com/gin-gonic/gin"
	"net/http"
)

type Login struct {
	Name string `json:"name"`
	Pass string `json:"pass"`
}

func main() {
	r := gin.Default()
	r.POST("/login", func(c *gin.Context){
		var login Login
		if err := c.ShouldBindJSON(&login); err != nil {
			c.JSON(http.StatusBadRequest, gin.H{"Message": "Error"})
		}
		c.JSON(http.StatusOK, gin.H{
			"Response Name": login.Name,
			"Response Pass": login.Pass,
		})
	})
	r.Run(":8081")
}
```




## Refer

[1] https://github.com/gin-gonic/gin

[2] https://gin-gonic.com/docs/

[3] https://github.com/gin-gonic/gin#quick-start
