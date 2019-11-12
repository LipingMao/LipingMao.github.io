---
title: Golang -- 拾遗 Part2
---

> Golang语言小白学习基础知识

* String是unsafe.Pointe和len的复合struct.
* String空为“”不是nil, 跨行时+ 必须在末尾
* 遍历字符串时byte/rune不同，rune会直接返回Unicode字符。
*  每次字符串拼接都必须重新分配内存，应该避免大字符串通过拼接构建，会有性能问题。利用strings.Join或者bytes.Buffer可以改善性能。
* Unicode时使用rune处理。它是int32别名，相当于UCS-4/UTF-32编码格式。
* 数组长度和数组元素类型一起构成数组类型，两者之一不同即类型不同。多维数组仅第一维可以为… len等操作也是对第一维的。
* 与C语言不同的是，go语言中数组是值类型，赋值和传参都会复制数组，除非使用切片或指针。
* slice本身是数组，len，cap的引用类型，需要make初始化。
* 访问不存在的键值，默认返回零值，不会错误，但推荐使用v，ok ：= m[“d”]; ok 判断key是否存在
* 不能对nil字典写，但是能读，内容为空与nil不同。
