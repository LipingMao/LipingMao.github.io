---
title: GoLang -- JSON Parse & Merge
---



GoLang是静态类型语言，在处理JSON时，通常先定义struct，用json Marshal / Unmarshal做序列化操作。但有时，json格式及字段不确定，例如通过API参数传入确定获取JSON中某些字段。以下两个库是目前被使用较多的库：

```
github.com/tidwall/gjson
github.com/buger/jsonparser
```



以gjson为例：

```
j := `{"android":{
        "common":{
            "videoHW": 1,
            "VOIP":{
                "detail_common_key1": "default_value",
                "detail_common_key2": "default_value"
            },
            "MUSIC":{
                "detail_common_key1": "default_value",
                "detail_common_key2": "default_value"
            },
            "Others":{
 
            }
        },
        "MANUFACTURER#MODEL#BOARD#OSVERSION":{
            "videoHW": 0,
            "VOIP":{
                "diff_key": ""
            },
            "MUSIC": {
                "diff_key": ""
            },
            "Others":{
 
            }
        }
    }}`
result:= gjson.Get(j, "android.common")
```



此时result可以获得以下内容：

```
{
            "videoHW": 1,
            "VOIP":{
                "detail_common_key1": "default_value",
                "detail_common_key2": "default_value"
            },
            "MUSIC":{
                "detail_common_key1": "default_value",
                "detail_common_key2": "default_value"
            },
            "Others":{
 
            }
        }
```



另外项目中，遇到合并Json的需求，RFC7386定义了Json merge的具体行为[1], 以下可以进行合并JSON：

```
import (
	jpatch "github.com/evanphx/json-patch"
)

func TryMergeJson() {
	original := []byte(`{"VOIP": {
                "detail_common_key1": "default_value",
                "detail_common_key2": "default_value"
            },
            "MUSIC": {
                "detail_common_key1": "default_value",
                "detail_common_key2": "default_value"
            },
            "Others": {
 
            }}`)
	target := []byte(`{
            "VOIP": {
                "detail_common_key2": "new",
                "diff_key": "diff1"
            },
            "MUSIC": {
                "diff_key": "diff2"
            },
            "Others": {
 
            }
        }`)

	_, err := jpatch.MergePatch(original, target)
	if err != nil {
		panic(err)
	}
}
```







[1] https://tools.ietf.org/html/rfc7386





