---
title: Golang -- Print Struct
---



需要打印Struct时，可以使用%+v显示更多信息：


```
package main

import "fmt"

type info struct {
    Name string
    id int
}

func main()  {
    v := info{"limao", 123}
    fmt.Printf("%v\n", v)
    fmt.Printf("%+v\n", v)
}
```



打印结果：

```
{limao 123}
{Name:limao id:123}
```

