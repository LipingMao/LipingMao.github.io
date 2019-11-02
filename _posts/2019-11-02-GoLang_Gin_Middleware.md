---
title: Golang -- Gin Middleware
---



> Gin 中支持Middleware系统，本文简单介绍了如何自定义一个Middleware，并对默认的两个Middleware简单介绍。



## Gin 自定义 Middleware

以下为Gin中自定一个Middleware的例子：

```go
package main

import (
	"fmt"
	"github.com/gin-gonic/gin"
	"net/http"
	"time"
)

// 自定义Middleware，返回类型为gin.HandlerFunc
func Logger() gin.HandlerFunc {
	// 输入参数为 *gin.Context
	return func(c *gin.Context) {
		// 实际请求处理函数之前的动作
		t := time.Now()

		// 上下文中可以存贮KV，后续处理函数中使用
		c.Set("example", "12345")

		c.Next()

		// 请求处理函数之后的动作
		latency := time.Since(t)
		fmt.Println(latency)

		// 读取Handler处理的结果
		status := c.Writer.Status()
		fmt.Println(status)
	}
}

func main() {
	r := gin.New()
	r.Use(Logger())

	r.GET("/ping", func(c *gin.Context) {
		example := c.MustGet("example").(string)
		// 读取中间件存储的内容
		fmt.Println(example)
		// 返回
		c.JSON(http.StatusOK, gin.H{"Response":"OK"})
	})

	r.Run(":8082")
}
```


## Gin Default Middleware

Gin在Default中默认加载了两个Middleware，代码如下：

```go
// Default returns an Engine instance with the Logger and Recovery middleware already attached.
func Default() *Engine {
	debugPrintWARNINGDefault()
	engine := New()
	// 加载middleware
	engine.Use(Logger(), Recovery())
	return engine
}
```

可以看到两个Middleware分别是Logger和Recovery，分别用于记录请求的信息的logger，和从Panic中恢复的Recovery。

### Gin Middleware -- Logger

Logger 可以支持记录一个API请求发生时间，返回Status Code，Latency，远端IP，方法和URL Path，代码见[0]。
例如：

```
[GIN] 2019/11/02 - 14:15:38 | 500 |    1.068623ms |       127.0.0.1 | GET      /panic
```

Logger可以支持自定义formater输出，以及自定义文件输出，在加载Logger Middleware时，可以使用以下Config方式加载，指定formater和Log文件：

```
// LoggerWithConfig instance a Logger middleware with config.
func LoggerWithConfig(conf LoggerConfig) HandlerFunc {
...
}

// 可以看到通过自定义，Formatter和Output的方式定义格式和输出文件。
// LoggerConfig defines the config for Logger middleware.
type LoggerConfig struct {
	// Optional. Default value is gin.defaultLogFormatter
	Formatter LogFormatter

	// Output is a writer where logs are written.
	// Optional. Default value is gin.DefaultWriter.
	Output io.Writer

	// SkipPaths is a url path array which logs are not written.
	// Optional.
	SkipPaths []string
}
```

Logger Middleware可以用于ELK收集metrics，例如，如果有ELK系统处理日志，则可以将Logger中间件中的Metrics以特定格式存放至特定Metrics日志文件中，后期给ELK分析。

示例如下：

```go
// 初始化Metrics日志
func InitMetrics(m Metric) error {
	if mlog == nil {
		mlog = logrus.New()
	}

	// 设置Rotate
	writer, err := rotatelogs.New(
		m.GetMetricPath()+".%Y%m%d%H%M",
		rotatelogs.WithLinkName(m.GetMetricPath()),
		rotatelogs.WithRotationTime(METRICSROTATETIME),
		rotatelogs.WithRotationCount(METRICSROTATECOUNT),
	)
	if err != nil {
		return err
	}

	// 设置logger的Writer
	mlog.SetOutput(writer)
	// 设置日志格式为JSON，方便ELK分析
	mlog.SetFormatter(&logrus.JSONFormatter{
		TimestampFormat:  "",
		DisableTimestamp: false,
		DataKey:          "",
		FieldMap:         nil,
		CallerPrettyfier: nil,
		PrettyPrint:      false,
	})

	return nil
}

// 自定义Metrics格式
type GinLogger struct {
	// StatusCode is HTTP response code.
	StatusCode int `json:"statuscode"`
	// Latency is how much time the server cost to process a certain request. (ms)
	Latency float64 `json:"latencyms"`
	// ClientIP equals Context's ClientIP method.
	ClientIP string `json:"clientip"`
	// Method is the HTTP method given to the request.
	Method string `json:"method"`
	// Path is a path the client requests.
	Path string `json:"path"`
	// BodySize is the size of the Response Body
	BodySize int `json:"bodysize"`

	// Common Fields
	// level
	Level string `json:"level"`
	// msg
	Msg string `json:"msg"`
	// time
	Time string `json:"time"`
	// Metrics Type
	MetricsType string `json:"metricstype"`
}

// 自定义给Gin的Formatter
var GinLoggerFormatter = func(param gin.LogFormatterParams) string {
  // 按需求定义哪些Metrics需要收集
	g := GinLogger{
		StatusCode:  param.StatusCode,
		Latency:     param.Latency.Seconds() * 1e3,
		ClientIP:    param.ClientIP,
		Method:      param.Method,
		Path:        param.Path,
		BodySize:    param.BodySize,
		Level:       "Info",
		Msg:         "RestRequest",
		Time:        time.Now().Format(time.RFC3339),
		MetricsType: TYPE_RESTHTTP_PERFORMANCE,
	}
	gjson, _ := json.Marshal(g)
	return string(gjson) + "\n"
}

// 设置给Gin Middleware的Output
func GetMetricsOutput() io.Writer {
	return mlog.Out
}
```



### Gin Middleware -- Recovery

Gin 支持自动从Panic中恢复的功能，这是因为在Recovery Middleware中对Panic做了recovery()动作。代码见[2]。

处理逻辑非常简单，核心是利用了defer在Recovery Middleware中调用了recovery动作。

```go
// RecoveryWithWriter returns a middleware for a given writer that recovers from any panics and writes a 500 if there was one.
func RecoveryWithWriter(out io.Writer) HandlerFunc {
...
	return func(c *Context) {
		defer func() {
			if err := recover(); err != nil {
...
			}
		}()
		c.Next()
	}
}

```





以下为一个典型的Recovery Middleware在panic后的日志。

```go
2019/11/02 14:15:38 [Recovery] 2019/11/02 - 14:15:38 panic recovered:
GET /panic HTTP/1.1
Host: 127.0.0.1:8082
Accept: */*
Accept-Encoding: gzip, deflate
Cache-Control: no-cache
Connection: keep-alive
Content-Length: 47
Content-Type: application/json
Postman-Token: 19d3447d-1189-4105-9bec-0d376a5324b0
User-Agent: PostmanRuntime/7.18.0


Panic in Handler!
/Users/limao/go/src/HelloGo/HelloGin/TryGin.go:17 (0x1505868)
        main.func1: panic("Panic in Handler!")
/Users/limao/go/src/github.com/gin-gonic/gin/context.go:147 (0x14f071a)
        (*Context).Next: c.handlers[c.index](c)
/Users/limao/go/src/github.com/gin-gonic/gin/recovery.go:83 (0x1503f53)
        RecoveryWithWriter.func1: c.Next()
/Users/limao/go/src/github.com/gin-gonic/gin/context.go:147 (0x14f071a)
        (*Context).Next: c.handlers[c.index](c)
/Users/limao/go/src/github.com/gin-gonic/gin/logger.go:241 (0x1503080)
        LoggerWithConfig.func1: c.Next()
/Users/limao/go/src/github.com/gin-gonic/gin/context.go:147 (0x14f071a)
        (*Context).Next: c.handlers[c.index](c)
/Users/limao/go/src/github.com/gin-gonic/gin/gin.go:391 (0x14fa5a9)
        (*Engine).handleHTTPRequest: c.Next()
/Users/limao/go/src/github.com/gin-gonic/gin/gin.go:352 (0x14f9c9d)
        (*Engine).ServeHTTP: engine.handleHTTPRequest(c)
/usr/local/Cellar/go/1.13/libexec/src/net/http/server.go:2802 (0x12cad33)
        serverHandler.ServeHTTP: handler.ServeHTTP(rw, req)
/usr/local/Cellar/go/1.13/libexec/src/net/http/server.go:1890 (0x12c65d4)
        (*conn).serve: serverHandler{c.server}.ServeHTTP(w, w.req)
/usr/local/Cellar/go/1.13/libexec/src/runtime/asm_amd64.s:1357 (0x105bdd0)
        goexit: BYTE    $0x90   // NOP

[GIN] 2019/11/02 - 14:15:38 | 500 |    1.068623ms |       127.0.0.1 | GET      /panic
```

可以看到记载了Panic相关的上下文。

但是值得注意的是，虽然Gin仅能帮你Recover handler中的错误，不能recover Gorouting中的Panic，例如下例:

```go
func main() {
	r := gin.Default()

	r.GET("/panic", func(c *gin.Context) {
		// Gin无法恢复gorouting中的panic
		go func () {
			panic("Panic in Gorouting")
		}()
		// Gin可以恢复Handler中的panic
		//panic("Panic in Handler!")
		c.JSON(http.StatusOK, gin.H{"Response":"OK"})
	})

	r.Run(":8082")
}
```

这是因为GoLang中，任何gorouting中发生了panic，都会panic整个程序。每个gorouting需要自己处理panic。



## Refer

[0]  https://github.com/gin-gonic/gin/blob/master/logger.go

[2] https://github.com/gin-gonic/gin/blob/master/recovery.go