---
title: Golang -- channel
---

> 记录了一些channel的基本用法。

### 使用前make

chan在使用前需要make，如果没有make会发生错误
```
func main() {
    var x chan int
    go func() {
        x <- 1
    }()
    <-x
}
```

error chan send(nil chan):
```
goroutine 17 [chan send (nil chan)]:
main.main.func1(0x0)
        /Users/limao/go/src/HelloGo/chan/main.go:30 +0x37
created by main.main
        /Users/limao/go/src/HelloGo/chan/main.go:29 +0x3e

```

因为chan需要初始化底层数据结构，因此使用前需要make。


### 实现gorouting间同步

由于x的长度为1，主进程会阻塞在<-x，直到有数据被放入chan x：
```
func main() {
	x := make(chan int)
	go func() {
		for i := 3; i > 0; i-- {
			x<- i
			time.Sleep(1 * time.Second)
		}
		close(x)
	}()
	for {
		n, ok:= <-x
		if !ok {
			break
		}
		fmt.Println(n)
	}
	fmt.Println("Go!")
}
```

### 实现buffer chan

chan中可以缓存一定数据：
```
func main() {
	x := make(chan int, 3)
	go func() {
		for i := 10; i > 0; i-- {
			x<- i
			fmt.Printf("Push %v\n", i)
		}
		close(x)
	}()
	for {
		n, ok:= <-x
		if !ok {
			break
		}
		fmt.Printf("Pull %v\n", n)
		time.Sleep(1 * time.Second)
	}
	fmt.Println("Go!")
}
```

可以看到最多push 3个元素到chan buffer中，x<-动作会阻塞:
```
Push 10
Push 9
Pull 10
Push 8
Push 7
Pull 9
Push 6
Pull 8
Push 5
Pull 7
Push 4
Pull 6
Push 3
Pull 5
Push 2
Pull 4
Push 1
Pull 3
Pull 2
Pull 1
Go!
```

通常用于发送和处理速度不匹配的场景中。


### 使用select

select可以用于统一管理多个chan，一般会在select中添加一个timeout用于没有收到任何信息超时的情况：
```
func sender(n chan int, name string) {
	for i := 5; i > 0; i-- {
		n<- i
		fmt.Printf("%s Push %v\n", name, i)
	}
}

func main() {
	x := make(chan int, 3)
	y := make(chan int, 3)
	go sender(x, "x")
	go sender(y, "y")
	for {
		select {
		case n, ok := <-x:
			if !ok {
				panic("Not OK!")
			}
			fmt.Printf("x Pull %v\n", n)

		case n, ok := <-y:
			if !ok {
				panic("Not OK!")
			}
			fmt.Printf("y Pull %v\n", n)
		case <- time.After(2 *time.Second):
			fmt.Printf("Timeout to pull, Done!")
			return
		}
	}
}

```

输出结果：
```
x Pull 5
y Push 5
y Push 4
y Push 3
y Push 2
y Pull 5
x Push 5
y Pull 4
x Pull 4
x Push 4
y Pull 3
x Pull 3
x Push 3
x Push 2
x Push 1
y Push 1
x Pull 2
y Pull 2
y Pull 1
x Pull 1
Timeout to pull, Done!
```

### 单向chan

可以定义单向chan，例如x := make(<-chan int)，一般用于函数传参，约定优于配置。


