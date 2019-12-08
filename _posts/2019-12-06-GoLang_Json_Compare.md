---
title: GoLang -- JSON String Compare
---

UT时比较两个JSON String结果是否相同，可以将其unmarshal出来然后用DeepEqual比较是否相同：
```
func AreEqualJSON(s1, s2 string) (bool, error) {
	var o1 interface{}
	var o2 interface{}

	var err error
	err = json.Unmarshal([]byte(s1), &o1)
	if err != nil {
		return false, fmt.Errorf("Error mashalling string 1 :: %s", err.Error())
	}
	err = json.Unmarshal([]byte(s2), &o2)
	if err != nil {
		return false, fmt.Errorf("Error mashalling string 2 :: %s", err.Error())
	}

	return reflect.DeepEqual(o1, o2), nil
}

```

注意此处用interface{}，好于map[string]interface{}, 因为"123"也是合法的JSON。
