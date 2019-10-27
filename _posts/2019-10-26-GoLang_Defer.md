---
title: Golang - defer and measure time
tags: Golang
---

> 利用defer实现计时功能，以及defer的基础知识。

## 一些注意点

Golang中可以利用defer在函数返回前执行一些动作，常用的比如close文件等操作，这样并不需要在所有返回处都重复close的代码。理解defer需要理解他发生在return的时机，首先需要理解return操作不是原子操作。
例如:
```
func TryDefer() (ret int){
	defer func(){ret++}()
	return 1
}
```

此操作实际执行时:
1) ret=1 （return 1）
2) ret++  (此操作后ret为2)
3) 实际返回为2

另外需要注意的点是, 多个defer操作时，是先进后出的堆栈动作，例如:

```
func TryDefer() {
	defer fmt.Println(1)
	defer fmt.Println(2)
	defer fmt.Println(3)
}
```

此处为3，2，1的调用顺序。

defer会使用调用defer时的值，如下例：
```
func TryDefer() {
	i := 0
	defer fmt.Println(i)
	i++
}
```
结果为0


## 利用defer实现计时

以下示例利用defer的特性，实现了log函数用时:

```
// Measure and log the function duration.
// defer LogDuration("Function 1")()
func LogDuration(what string) func() {
	start := time.Now()
	return func() {
		logger.Infof("%v took %v", what, time.Since(start))
	}
}

// Caller
func Func1() {
	defer LogDuration("Func1")()
}
```

当调用LogDuration时，记录了当前时间，并在return前计算了时间差值，将其记录在log中。


## Refer

[1] https://deepzz.com/post/how-to-use-defer-in-golang.html

[2] https://xiaozhou.net/something-about-defer-2014-05-25.html

[3] https://draveness.me/golang/keyword/golang-defer.html

[4] https://yourbasic.org/golang/measure-execution-time/
