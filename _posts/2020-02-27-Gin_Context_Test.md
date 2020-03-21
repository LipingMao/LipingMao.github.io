---
title: Golang -- Gin Context for test
---



在写UT测试用例时需要单独生成Gin的Context，如果直接new struct会有段错误。可以类似以下方式生成：

```
	w := httptest.NewRecorder()
	c ,_ := gin.CreateTestContext(w)
	req, _ := http.NewRequest("POST", "/gslb/inner/rule?current=5&size=7", nil)
	c.Request = req
	...
```



此时可以单独使用gin的Context。