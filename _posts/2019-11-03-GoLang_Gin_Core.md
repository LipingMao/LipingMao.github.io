---
title: Golang -- Gin Middleware&Handler
---

> Gin中实现Middleware框架的主要逻辑。

Gin中实现Middleware利用了函数调用栈，其中最核心的步骤是利用c.next()实现middleware的压栈动作。
如下所示:
```go
func MiddlewareSample() HandlerFunc {
	return func(c *Context) {
		// Handler执行前
		c.Next()
		// Handler执行后
	}
}
```

在调用Use(Middleware1, Middleware2...)，注册Middleware时，实际将Middleware注册到了Handlers。

当调用c.Next()时，则从数组顺序调用下一个Middleware，这样利用函数调用天然实现了压栈动作：
以下为Next的实现。
```go
// Next should be used only inside middleware.
// It executes the pending handlers in the chain inside the calling handler.
// See example in GitHub.
func (c *Context) Next() {
	c.index++
	for c.index < int8(len(c.handlers)) {
		c.handlers[c.index](c)
		c.index++
	}
}

```

最终达到的效果即：
```go
Middleware1:
	// Before Handler Actions in Middleware1
	// ...
	Middleware2:
		// Before Handler Actions in Middleware2
		// ...
		Middleware3:
			// Before Handler Actions in Middleware3
			// ...
			Handler
			// Before Handler Actions in Middleware3
			// ...
		// After Handler Actions in Middleware3
		// ...

	// After Handler Actions in Middleware2
	// ...
// After Handler Actions in Middleware1
// ...
			
```
