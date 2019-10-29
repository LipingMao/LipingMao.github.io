---
title: Golang -- Test
---

> Golang基本UT/IT测试用例

## UT功能测试用例

用例以Test开头，示例中使用GoLang自带的测试框架，测试cid在并发情况下的生成情况，验证cid不重复：
Sample:
```
func TestGenerateCid2(t *testing.T) {
	var wg sync.WaitGroup
	wg.Add(100)                            // using 100 goroutine to generate 1000000 ids
	results := make(chan string, 10000000) //store result
	for i := 0; i < 100; i++ {
		go func() {
			for i := 0; i < 100000; i++ {
				id := GenerateCid()
				results <- id
			}
			wg.Done()
		}()
	}
	wg.Wait()

	m := make(map[string]bool)
	for i := 0; i < 10000000; i++ {
		select {
		case id := <-results:
			if _, ok := m[id]; ok {
				t.Errorf("Found duplicated id: %v", id)
				return
			} else {
				m[id] = true
			}
		case <-time.After(2 * time.Second):
			t.Errorf("Expect 10000000000 ids in results, but got %v", i)
			return
		}
	}
}
```

## UT性能测试用例

以Benchmark开头，可以测试和统计并发能力：

Sample:
```
func BenchmarkGenerateCid(b *testing.B) {
	for n := 0; n < b.N; n++ {
		GenerateCid()
	}
}
```

Note：
goconvey[1]也是不错的UT框架，之后可以考虑取代原UT库


## IT测试用例

IT测试没有什么好的办法，[2]介绍了IT的基本概念，GSLB采用写MockServer实现模拟其他组件的动作，完成功能测试和性能测试要求。


## Refer

[1] https://github.com/smartystreets/goconvey

[2] https://www.guru99.com/integration-testing.html
