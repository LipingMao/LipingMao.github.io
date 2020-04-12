---
title: Golang -- Buffer
---

> read through byte.Buffer

here is the defination of struct:
```
type Buffer struct {
	buf      []byte // contents are the bytes buf[off : len(buf)]
	off      int    // read at &buf[off], write at &buf[len(buf)]
	lastRead readOp // last read operation, so that Unread* can work correctly.
}
```

Here is a sample test code:
```
package main

import (
"bytes"
"fmt"
)

func main() {
	b := bytes.NewBufferString("你好World")
	r, n, err := b.ReadRune()
	fmt.Printf("First ReadRune: %v, n: %v, err: %v, b.String(): %v , b.Len(): %v, b.Cap(): %v\n", string(r), n, err, b.String(), b.Len(), b.Cap())
	r, n, err = b.ReadRune()
	fmt.Printf("Second ReadRune: %v, n: %v, err: %v, b.String(): %v , b.Len(): %v, b.Cap(): %v\n", string(r), n, err, b.String(), b.Len(), b.Cap())
	n, err = b.WriteString("123")
	fmt.Printf("First Write , n: %v, err: %v, b.String(): %v , b.Len(): %v, b.Cap(): %v\n", n, err, b.String(), b.Len(), b.Cap())
	b.Grow(40)
	fmt.Printf("Grow Size 40, b.String(): %v , b.Len(): %v, b.Cap(): %v\n", b.String(), b.Len(), b.Cap())
}
```

Result:
```
First ReadRune: 你, n: 3, err: <nil>, b.String(): 好World , b.Len(): 8, b.Cap(): 32
Second ReadRune: 好, n: 3, err: <nil>, b.String(): World , b.Len(): 5, b.Cap(): 32
First Write , n: 3, err: <nil>, b.String(): World123 , b.Len(): 8, b.Cap(): 32
Grow Size 40, b.String(): World123 , b.Len(): 8, b.Cap(): 104
```

Understand how it grow, here is core logic:
```
	} else {
		// Not enough space anywhere, we need to allocate.
		buf := makeSlice(2*c + n)
		copy(buf, b.buf[b.off:])
		b.buf = buf
	}
```

c is the cap of buf, in this case, it is 32, so 2*32+40 = 104.
