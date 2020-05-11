---
title: DevOps -- Small GoLang Image
---

如果使用golang:alpine, 制作的golang容器也需要大概200-300MB。可以先build好Binary，使用scratch制作，则镜像可以控制到20-30MB。例如：
```
// command
CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o main .


// Dockerfile
FROM scratch
MAINTAINER Liping Mao (limao455@163.com)

ADD main ./
CMD ["/main"]
```
