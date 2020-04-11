---
title: Golang -- rune
---

> string, []byte, []rune is difference in golang, here is a sample to show the difference


Example:
```
package main

import "fmt"

func main() {
	s := "Hello世界"
	fmt.Printf("Length of string: %v\n", len(s))
	fmt.Printf("Length of []byte: %v\n", len([]byte(s)))
	fmt.Printf("Length of []rune: %v\n", len([]rune(s)))
	fmt.Printf("Length of the string: %v\n", utf8.RuneCountInString(s))
}
```

result:
```
Length of string: 11
Length of []byte: 11
Length of []rune: 7
Length of the string: 7
```

here is the defination of rune:
```
// rune is an alias for int32 and is equivalent to int32 in all ways. It is
// used, by convention, to distinguish character values from integer values.
type rune = int32
```

In golang, one Chinese word is 3 byte. If you want to get the "length" of a string which has Chinese word, you can use []rune.
