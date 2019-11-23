---
title: Golang -- Tips Named Return value
---

> Golang中可以定义命名的返回值，一些注意事项。


先看如何定义： 
```go
func calc(a, b int) (mul int, div int) {
  mul = a*b
  div = a/b
  return
}
```

可以看到当定义了named return value, 在返回时，仅需要调用return。

注意的是，如果在函数中使用":=", 在编译阶段会出错, 因为mul，div已经被初始化过：
```go
func calc(a, b int) (mul int, div int) {
  // here, it will give an error
  // as parameters are already defined
  // in function signature
  mul := a*b
  div := a/b
  return
}
```



仅当函数比较简单且有多返回值时使用，如果函数比较复杂，最好还是显式return所有值。当然定义了named return value，显式return也是可以的。



[1] https://www.geeksforgeeks.org/named-return-parameters-in-golang/