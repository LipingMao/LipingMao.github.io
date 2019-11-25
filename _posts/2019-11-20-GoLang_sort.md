---
title: Golang -- Sorted Collection
---

> Golang中对Collection排序的，可以使用Sort库对Collection进行排序



以下为示例代码：

```go
package main

import (
	"fmt"
	"sort"
)

// Make sure MyList has implement the sort.Interface
var _ sort.Interface = MyList(nil)

type MyStruct struct {
	Load   int
	Data   string
}

type MyList []MyStruct

func(l MyList) Len() int {
	return len(l)
}

func(l MyList) Swap(i, j int) {
	l[i], l[j] = l[j], l[i]
}

// Compare by Load
func(l MyList) Less(i, j int) bool {
	return l[i].Load < l[j].Load
}

func main(){
	mList := MyList{
		MyStruct{
		Load: 100,
		Data: "test1",
		},
		MyStruct{
			Load: 80,
			Data: "test2",
		},
		MyStruct{
			Load: 110,
			Data: "test3",
		},
		MyStruct{
			Load: 70,
			Data: "test4",
		},
	}
	fmt.Println("Before Sort:")
	for i, m := range mList {
		fmt.Printf("[%v] %v\n", i, m)
	}
	sort.Sort(mList)
	fmt.Println("\nAfter Sort:")
	for i, m := range mList {
		fmt.Printf("[%v] %v\n", i, m)
	}
}
```



当对collection进行排序时，需要实现sort.Interface接口 （Len，Swap，Less）， 主要是Less可以进行实际类型的大小比较，此例中对比了Load大小。

