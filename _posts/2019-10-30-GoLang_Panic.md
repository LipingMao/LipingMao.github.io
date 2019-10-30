---
title: Golang -- Panic/Recover
---

> Golang Panic&Recover

## Panic&Recover

GoLang中并不支持try...catch的动作，通常需要使用Panic&Recovery&Defer组合使用达到效果。


在以下示例中，Recovery了一个panic：
```
func main() {
	defer func() {
		if r := recover(); r != nil {
			fmt.Println("[E] ", r)
		}
	}()
	panic("Kill in Main!")
	time.Sleep(10 * time.Second)
}

```

注意，如果在gorouting中panic，主gorouting并不能恢复，需要在启动gorouting时做recover动作，例如以下示例，是不能recover的。
```
func main() {
	defer func() {
		if r := recover(); r != nil {
			fmt.Println("[E] ", r)
		}
	}()
	go func() {
		fmt.Println("In Gorouting")
		panic("Kill in Gorouting!")
	}()
	time.Sleep(10 * time.Second)
}

```
