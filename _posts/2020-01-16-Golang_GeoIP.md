---
title: Golang -- GeoIP
---

> 有些场景下我们需要知道Client IP地址的地域信息，我们可以使用GeoIP库获得IP的信息，本文使用Golang查询IP地址信息。


Golang可以使用geoip2-golang来读取GeoIP:
```
package geoip

import (
	"fmt"
	"github.com/oschwald/geoip2-golang"
	"github.com/oschwald/maxminddb-golang"
	"net"
)

func Address(db *maxminddb.Reader) {
	// If you are using strings that may be invalid, check that ip is not nil
	ip := net.ParseIP("106.2.176.22")
	var record struct {
		Country struct {
			ISOCode string `maxminddb:"iso_code"`
		} `maxminddb:"country"`
	} // Or any appropriate struct

	db.Lookup(ip, &record)
}

func Address2(db *geoip2.Reader) {
	// If you are using strings that may be invalid, check that ip is not nil
	ip := net.ParseIP("106.2.176.22")
	c, err := db.City(ip)
	fmt.Printf("Country: %v, err: %v \n", c.Country.IsoCode, err)
}
```

可以获取City级别的信息：
```
Country: &{{0 map[]} {AS 6255147 map[de:Asien en:Asia es:Asia fr:Asie ja:アジア pt-BR:Ásia ru:Азия zh-CN:亚洲]} {1814991 false CN map[de:China en:China es:China fr:Chine ja:中国 pt-BR:China ru:Китай zh-CN:中国]} {50 39.9289 116.3883 0 Asia/Shanghai} {} {1814991 false CN map[de:China en:China es:China fr:Chine ja:中国 pt-BR:China ru:Китай zh-CN:中国]} {0 false  map[] } [{2038349 BJ map[en:Beijing fr:Municipalité de Pékin zh-CN:北京市]}] {false false}}, err: <nil> 

```

如果仅仅需要个别信息，比如只要国家信息，可以使用Lookup接口，性能比City的要更好。例如Lookup性能大约1167 ns/op， City大约7895 ns/op。
