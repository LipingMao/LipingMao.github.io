---
title: DevOps -- curl redirct
---



当用curl去call URL时，如果URL发生重定向，通常无法获得返回值。例如：

```
➜  _posts git:(master) ✗ curl http://www.pano.video
➜  _posts git:(master) ✗
```



可以使用curl -i参数打印http头部信息：

```
➜  _posts git:(master) ✗ curl -i http://www.pano.video
HTTP/1.1 301 Moved Permanently
x-amz-id-2: uTYV11KwjtHMp/vhLAQZgGVxnbziLMgTg2VMVn3dE9PZO23nGsDagjctoMdE5pUEeWHb/Xt5WVs=
x-amz-request-id: B662B326DDC347A6
Date: Wed, 04 Dec 2019 01:24:27 GMT
Location: http://pano.video/
Content-Length: 0
Server: AmazonS3
```



可以看到被301重定向到了http://pano.video/，curl如果使用-L命令，就可以直接访问301的地址，例如：

```
➜  _posts git:(master) ✗ curl -iL http://www.pano.video
HTTP/1.1 301 Moved Permanently
x-amz-id-2: pQ/58VPrkCNAvX7nxA6jrBeABPtYRuycNHkxQgjBV26XSNdCB/aS4M/JMTDFwY3Tn3mrRt/J8ns=
x-amz-request-id: CDFEAA56EBAB7586
Date: Wed, 04 Dec 2019 01:25:47 GMT
Location: http://pano.video/
Content-Length: 0
Server: AmazonS3

HTTP/1.1 200 OK
x-amz-id-2: z0A8jyXkFULLFH2RWkOFll6bMfj3NwhleVFvL4wb0IyHhfPOMUDJH6CPM/gg7fuDVNgr7cnI1h8=
x-amz-request-id: EDC941CF0D17AC23
Date: Wed, 04 Dec 2019 01:25:48 GMT
Last-Modified: Tue, 03 Dec 2019 13:47:22 GMT
ETag: "9fbc1060df4ff8589853318f45432774"
Content-Type: text/html
Content-Length: 2444
Server: AmazonS3

<!doctype html>
<html>

...
</html>
```

