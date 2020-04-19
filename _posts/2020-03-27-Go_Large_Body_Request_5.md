---
title: Golang -- Large Request Body 5
---

继续看serve，再调用“w, err := c.readRequest(ctx)”:
```
		// HTTP cannot have multiple simultaneous active requests.[*]
		// Until the server replies to this request, it can't read another,
		// so we might as well run the handler in this goroutine.
		// [*] Not strictly true: HTTP pipelining. We could let them all process
		// in parallel even if their responses need to be serialized.
		// But we're not going to implement HTTP pipelining because it
		// was never deployed in the wild and the answer is HTTP/2.
		serverHandler{c.server}.ServeHTTP(w, w.req)
```

从这里看到交给Handler，从这里看到就进入了Handler流程；
```
func (sh serverHandler) ServeHTTP(rw ResponseWriter, req *Request) {
	handler := sh.srv.Handler
	if handler == nil {
		handler = DefaultServeMux
	}
	if req.RequestURI == "*" && req.Method == "OPTIONS" {
		handler = globalOptionsHandler{}
	}
	handler.ServeHTTP(rw, req)
}
```

回忆一下测试服务器源码：
```
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
注意此时，并没有完整的http body（此时，未读取的数据还在还在Linux系统的队列中，以及customer收到了windows满）。
在示例中，调用了“ShouldBindJSON” ，而在ShouldBindJSON中才会读取整个Body。
