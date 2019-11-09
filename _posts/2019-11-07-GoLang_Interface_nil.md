---
title: Golang -- define unused nil var with Interface
---

在看go-redis代码时，看到以下代码起初不能理解：
> var _ Pipeliner = (*Pipeline)(nil)

Pipeliner是Interface类型，Pipeline是一个实现。这个代码使用"_"定义了一个不会被访问的变量，通过这种方式约束了Pipeline必须实现Pipeliner所有接口，否则在编译阶段就会出错。这是检查实现所有接口的不错的小技巧。

