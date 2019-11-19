---
title: Golang -- Tips no tab in struct tag
---

在使用struct tag时，要注意，不要在tag中使用tab...
```
type abc struct {
    Name string `json:"name"  validate:"required,min=1,max=255"`
}
```

如上图，如果validate前使用了tab，将不 work。
如果你使用的是goland，可以设置Editor->Code Style->Go, enable Smart tabs

PS:
Debug 2 hours on this ... ...
