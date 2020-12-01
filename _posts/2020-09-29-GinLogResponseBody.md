---
title: Golang -- Log Gin Response Body in Middleware
---

Sample Middleware to log request/response info in middleware.

```
package middleware

import (
	"bytes"
	"github.com/gin-gonic/gin"
	"io/ioutil"
)

type bodyLogWriter struct {
	gin.ResponseWriter
	body *bytes.Buffer
}

func (w bodyLogWriter) Write(b []byte) (int, error) {
	w.body.Write(b)
	return w.ResponseWriter.Write(b)
}

func GinBodyLogMiddleware() gin.HandlerFunc {
	return func(c *gin.Context) {
		var bodyBytes []byte
		if c.Request.Body != nil {
			bodyBytes, _ = ioutil.ReadAll(c.Request.Body)
		}
		c.Request.Body = ioutil.NopCloser(bytes.NewBuffer(bodyBytes))
		blw := &bodyLogWriter{body: bytes.NewBufferString(""), ResponseWriter: c.Writer}
		c.Writer = blw

		c.Next()
                fmt.Printf("Send HTTP response, req uri: %v, method: %v, body: %v, resp code: %v, body: %v",
			c.Request.RequestURI, c.Request.Method, string(bodyBytes), c.Writer.Status(), blw.body.String())

}

```
