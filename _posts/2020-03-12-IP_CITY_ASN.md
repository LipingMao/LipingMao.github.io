---
title: DevOps -- IP City and ASN
---

> 目前免费的IP地址库中，maxmind[1]提供了IP地址的地域信息，ASN信息的查询。

maxmind提供了GeoLite2-City.mmdb ，GeoLite2-ASN.mmdb。这两个分别提供了City的查询，ASN查询。

例如：
访问ASN可以查询到运营商信息：
```
func TestGeoInfo_TestASN(t *testing.T) {
	r, _ := geoip2.Open("../../../GeoLite2-ASN.mmdb")
	{
		a, err := r.ASN(net.ParseIP("223.220.0.123"))
		fmt.Printf("%+v, %v\n", a, err)
	}

	{
		a, err := r.ASN(net.ParseIP("116.243.165.157"))
		fmt.Printf("%+v, %v\n", a, err)
	}

}
```

类似以下结果：
```
=== RUN   TestGeoInfo_TestASN
&{AutonomousSystemNumber:4134 AutonomousSystemOrganization:Chinanet}, <nil>
&{AutonomousSystemNumber:4808 AutonomousSystemOrganization:China Unicom Beijing Province Network}, <nil>
--- PASS: TestGeoInfo_TestASN (0.00s)
```

访问City信息可以查询城市信息：
```
func TestGeoInfo_TestCity(t *testing.T) {
	r, _ := geoip2.Open("../../../GeoLite2-City.mmdb")
	{
		a, err := r.City(net.ParseIP("223.220.0.123"))
		fmt.Printf("%+v, %v\n", a, err)
	}

	{
		a, err := r.City(net.ParseIP("116.243.165.157"))
		fmt.Printf("%+v, %v\n", a, err)
	}

}
```

类似以下结果：
```
=== RUN   TestGeoInfo_TestCity
&{City:{GeoNameID:0 Names:map[]} Continent:{Code:AS GeoNameID:6255147 Names:map[de:Asien en:Asia es:Asia fr:Asie ja:アジア pt-BR:Ásia ru:Азия zh-CN:亚洲]} Country:{GeoNameID:1814991 IsInEuropeanUnion:false IsoCode:CN Names:map[de:China en:China es:China fr:Chine ja:中国 pt-BR:China ru:Китай zh-CN:中国]} Location:{AccuracyRadius:50 Latitude:36.6167 Longitude:101.7667 MetroCode:0 TimeZone:Asia/Shanghai} Postal:{Code:} RegisteredCountry:{GeoNameID:1814991 IsInEuropeanUnion:false IsoCode:CN Names:map[de:China en:China es:China fr:Chine ja:中国 pt-BR:China ru:Китай zh-CN:中国]} RepresentedCountry:{GeoNameID:0 IsInEuropeanUnion:false IsoCode: Names:map[] Type:} Subdivisions:[{GeoNameID:1280239 IsoCode:QH Names:map[en:Qinghai fr:Province de Qinghai zh-CN:青海省]}] Traits:{IsAnonymousProxy:false IsSatelliteProvider:false}}, <nil>
&{City:{GeoNameID:0 Names:map[]} Continent:{Code:AS GeoNameID:6255147 Names:map[de:Asien en:Asia es:Asia fr:Asie ja:アジア pt-BR:Ásia ru:Азия zh-CN:亚洲]} Country:{GeoNameID:1814991 IsInEuropeanUnion:false IsoCode:CN Names:map[de:China en:China es:China fr:Chine ja:中国 pt-BR:China ru:Китай zh-CN:中国]} Location:{AccuracyRadius:50 Latitude:39.9289 Longitude:116.3883 MetroCode:0 TimeZone:Asia/Shanghai} Postal:{Code:} RegisteredCountry:{GeoNameID:1814991 IsInEuropeanUnion:false IsoCode:CN Names:map[de:China en:China es:China fr:Chine ja:中国 pt-BR:China ru:Китай zh-CN:中国]} RepresentedCountry:{GeoNameID:0 IsInEuropeanUnion:false IsoCode: Names:map[] Type:} Subdivisions:[{GeoNameID:2038349 IsoCode:BJ Names:map[en:Beijing fr:Municipalité de Pékin zh-CN:北京市]}] Traits:{IsAnonymousProxy:false IsSatelliteProvider:false}}, <nil>
--- PASS: TestGeoInfo_TestCity (0.01s)
PASS
```

Refer:
[1] https://dev.maxmind.com/geoip/geoip2/geolite2/
