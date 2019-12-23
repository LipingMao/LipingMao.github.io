---
title: Golang -- JSON Number Tips
---


有时需要兼容数字和字符串类型，例如：
```
{
    "age": 10
}

{
    "age": "10"
}
```

此时可以按下面方式定义struct兼容：
```
type Info struct {
    Age json.Number  `json:"age"`
}
```
