---
title: Golang -- fsnotify
---

> 一个常见的需求是监控某一文件的动态变化，触发更新动作，例如：定期刷新密钥。

Golang中可以使用fsnotify监控文件的变化，以下为一个例子：

```
package main

import (
	"fmt"
	"github.com/fsnotify/fsnotify"
	"time"
)

func monitorFile(w *fsnotify.Watcher, done chan string){
	for {
		select {
			case ev := <-w.Events:
				fmt.Printf("OP is %v, Name is %v\n", ev.Op, ev.Name)
				// just add it again for rename case
				w.Add(ev.Name)
			case er := <-w.Errors:
				done<- fmt.Sprintf("Failed(%v)", er)
				return
			case <-time.After(30 * time.Second):
				done<- fmt.Sprintf("timeout")
				return
		}
	}
}

func main() {
	watcher, err := fsnotify.NewWatcher()
	if err != nil {
		panic(err)
	}
	defer watcher.Close()
	err = watcher.Add("/Users/limao/go/src/HelloGo/chan/test.txt")
	if err != nil {
		panic(err)
	}

	done := make(chan string)
	go monitorFile(watcher, done)
	fmt.Printf("Done is %v", <-done)
}

```


[fsnotify](https://github.com/fsnotify/fsnotify)支持了不同平台的文件监控，不同平台利用的机制不同，以Linux为例，使用的是inotify，在调用Add方法时，监听了文件变化，见http://man7.org/linux/man-pages/man2/inotify_add_watch.2.html
