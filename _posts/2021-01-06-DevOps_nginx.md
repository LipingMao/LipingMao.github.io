---
title: DevOps -- Nginx proxy_pass
---


```
location ^~ /abc/
{
	proxy_pass http://test.p.v/;
}
```

curl http://test.p.v/abc/test.html
后端： http://test.p.v/test.html



```
location ^~ /abc/
{
        proxy_pass http://test.p.v;
}
```

curl http://test.p.v/abc/test.html
后端： http://test.p.v/abc/test.html
