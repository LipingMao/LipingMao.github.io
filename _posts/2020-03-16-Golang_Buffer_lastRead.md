---
title: Golang -- Buffer lastRead & UnReadRune
---

> Buffer结构中有lastRead字段，这个字段是用于UnReadRune的。


回顾一下Buffer的定义：
```
type Buffer struct {
	buf      []byte // contents are the bytes buf[off : len(buf)]
	off      int    // read at &buf[off], write at &buf[len(buf)]
	lastRead readOp // last read operation, so that Unread* can work correctly.
}
```

lastRead这个字段主要记录了Read rune的size。可以看到readOp定义如下：
```
// Don't use iota for these, as the values need to correspond with the
// names and comments, which is easier to see when being explicit.
const (
	opRead      readOp = -1 // Any other read operation.
	opInvalid   readOp = 0  // Non-read operation.
	opReadRune1 readOp = 1  // Read rune of size 1.
	opReadRune2 readOp = 2  // Read rune of size 2.
	opReadRune3 readOp = 3  // Read rune of size 3.
	opReadRune4 readOp = 4  // Read rune of size 4.
)
```

从ReadRune中可以看到，记录上次读取的Rune的Size：
```
func (b *Buffer) ReadRune() (r rune, size int, err error) {
	if b.empty() {
		// Buffer is empty, reset to recover space.
		b.Reset()
		return 0, 0, io.EOF
	}
	c := b.buf[b.off]
	if c < utf8.RuneSelf {
		b.off++
		b.lastRead = opReadRune1
		return rune(c), 1, nil
	}
	r, n := utf8.DecodeRune(b.buf[b.off:])
	b.off += n
	b.lastRead = readOp(n)
	return r, n, nil
}
```

在UnreadRune时，可以看到回退了一个Rune，并且此时将上次lastRead字段置为opInvalid:
```
func (b *Buffer) UnreadRune() error {
	if b.lastRead <= opInvalid {
		return errors.New("bytes.Buffer: UnreadRune: previous operation was not a successful ReadRune")
	}
	if b.off >= int(b.lastRead) {
		b.off -= int(b.lastRead)
	}
	b.lastRead = opInvalid
	return nil
}
```

以下为测试代码：
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
	err = b.UnreadRune()
	fmt.Printf("First UnreadRune: err: %v, b.String(): %v , b.Len(): %v, b.Cap(): %v\n", err, b.String(), b.Len(), b.Cap())
	err = b.UnreadRune()
	fmt.Printf("Second UnreadRune: err: %v, b.String(): %v , b.Len(): %v, b.Cap(): %v\n", err, b.String(), b.Len(), b.Cap())
}
```

结果如下：
```
First ReadRune: 你, n: 3, err: <nil>, b.String(): 好World , b.Len(): 8, b.Cap(): 32
First UnreadRune: err: <nil>, b.String(): 你好World , b.Len(): 11, b.Cap(): 32
Second UnreadRune: err: bytes.Buffer: UnreadRune: previous operation was not a successful ReadRune, b.String(): 你好World , b.Len(): 11, b.Cap(): 32

```
