---
title: Golang -- Package
---

> Golang的包结构。

## 路径

GOROOT为Golang默认库所在路径，GOPATH为工作空间目录，一般开发代码放在GOPATH下。

## 工作空间

工作空间由三个目录组成：
```
src/
bin/
pkg/
```

其中src为放置源码目录的路径，bin放置编译好的可执行文件，pkg是编译的库文件（用于增量编译）。

## 导入包

总结起来有以下几种不同的导入包方式：
```
import    "github.com/lipingmao/test"     <-- 默认方式，通过test.A访问
import X  "github.com/lipingmao/test"     <-- 别名方式，通过X.A访问
import .  "github.com/lipingmao/test"     <-- 简便方式，A访问
import _  "github.com/lipingmao/test"     <-- 仅初始化 
```

如果导入了包，但没有使用，编译器在编译阶段就会出错。可以使用"_"仅进行初始化。

## 初始化

在init函数是在全局变量初始化完成之后，main函数执行之前进行的。同一个包可以定义多个初始化函数，但是这些初始化函数执行顺序是不保证的（事实上有顺序，但不可维护，不应该依赖于初始化顺序）
```
var  x=100

func init() {
    println("init :  x = %v", x)
    x++
}

func main() {
    println("main :  x = %v", x)
}
```

输出：
```
init :  x = 100
main :  x = 101
```

## 访问控制

Golang中通过首字母大小写控制访问权限，大写为Public，小写为Private。还有第三种internal，可以让上层包及其子包可访问，例如：

```
src/
 |
 +-- main.go
 |
 +-- lib/
      |
      +-- internal/
      |      |
      |      +-- a/
      |      |
      |      +-- b/
      |
      +-- x/
          |
          +-- y/
 
```

x, y中可以直接访问internal a/b中小写的内容，但main.go中不能直接访问internal a/b中的内容。

## 依赖管理

Go中依赖包放在vendor包中，专门存放第三放包，对于第三方包可以使用go dep或者go mod。
