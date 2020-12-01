---
title: Golang -- Gin Middleware abort
---

Gin的中间件通过调用next方法/abort方法控制是否进入下一个handler。代码非常简单，当调用abort方法时，就设置index为最大整数的一半，middleware是不可能这么多的。当调用next时，则读取下一个已注册的handler：
```
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


// Abort prevents pending handlers from being called. Note that this will not stop the current handler.
// Let's say you have an authorization middleware that validates that the current request is authorized.
// If the authorization fails (ex: the password does not match), call Abort to ensure the remaining handlers
// for this request are not called.
func (c *Context) Abort() {
	c.index = abortIndex
}
```
