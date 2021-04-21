---
title: DevOps -- Nginx Large File
---

> nginx上传大文件时，一些参数需要修改。


提升max body size，默认为1m:

> client_max_body_size 300m;

一般不需要提升client_body_timeout, 两次读操作之间一般不太可能到60s以上,注意这个值并不是总耗时:

> client_body_timeout : Defines a timeout for reading client request body. The timeout is set only for a period between two successive read operations, not for the transmission of the whole request body. If a client does not transmit anything within this time, the request is terminated with the 408 (Request Time-out) error.
