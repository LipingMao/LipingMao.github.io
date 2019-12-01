---
title: GoLang -- Gin UT
---



>  对于Handler/Route也需要进行UT，使用的框架是Gin，可以使用httptest写UT。



 以下为示例：

```
	w := httptest.NewRecorder()
	body := fmt.Sprintf("{\"dcName\":\"dc1\", \"address\":\"dc1.pano.video\"}")
	enc, _ := crypt.EncryptInternalData(body)
	req, _ := http.NewRequest("POST", "/gslb/node/dc", strings.NewReader(enc))
	req.Header.Set(crypt.AUTHORISATIONHEADER, crypt.PANOSTOKEN+" "+token)
	r.Engine.ServeHTTP(w, req)
	assert.Equal(t, 200, w.Code)
```



测试前需要加载好路由，以及初始化Gin Engine，通过生成httptest.NewRecorder，可以获取返回的Code，body等返回值。其他部分仅需要正常生成request即可。