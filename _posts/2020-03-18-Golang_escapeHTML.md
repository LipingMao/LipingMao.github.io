---
title: Golang -- json escapeHTML 
---

> Golang在marshal json时会默认会进行escapeHTML的处理。


示例代码：
```
package main

import (
	"bytes"
	"encoding/json"
	"fmt"
)

type Test struct {
	Content string
}

func main() {
	t := new(Test)
	t.Content = "http://www.baidu.com?id=123&test=1"
	jsonByte, _ := json.Marshal(t)
	fmt.Println(string(jsonByte))
	var n Test
	json.Unmarshal(jsonByte, &n)
	fmt.Printf("t: %+v, n: %+v\n", t, n)

	bf := bytes.NewBufferString("")
	jsonEncoder := json.NewEncoder(bf)
	jsonEncoder.SetEscapeHTML(false)
	jsonEncoder.Encode(t)
	fmt.Println(bf.String())
}
```

结果：
```
{"Content":"http://www.baidu.com?id=123\u0026test=1"}
t: &{Content:http://www.baidu.com?id=123&test=1}, n: {Content:http://www.baidu.com?id=123&test=1}
{"Content":"http://www.baidu.com?id=123&test=1"}
```

可以看到marshl到string时，对“&”进行了转义动作。从marshal的源码可以看到escapeHTML操作可以配置：
```
func Marshal(v interface{}) ([]byte, error) {
	e := newEncodeState()

	err := e.marshal(v, encOpts{escapeHTML: true})
	if err != nil {
		return nil, err
	}
	buf := append([]byte(nil), e.Bytes()...)

	e.Reset()
	encodeStatePool.Put(e)

	return buf, nil
}

type encOpts struct {
	// quoted causes primitive fields to be encoded inside JSON strings.
	quoted bool
	// escapeHTML causes '<', '>', and '&' to be escaped in JSON strings.
	escapeHTML bool
}
```

'<', '>', and '&'会被默认转义。可以使用Json Encoder手动SetEscapeHTML到false。
```
	bf := bytes.NewBufferString("")
	jsonEncoder := json.NewEncoder(bf)
	jsonEncoder.SetEscapeHTML(false)
	jsonEncoder.Encode(t)
```
