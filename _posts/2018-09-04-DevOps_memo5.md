---
title: OpenStack  -- AIO mode for disk device
---

## Background

> 在调优OpenStack Disk IO时，AIO(asynchronous IO)提供了两个参数，"native"和"threads"。 本文测试了两者参数对性能影响。


## How to set up

nova的默认参数使用了threads模式，在nova的配置文件中需要配置以下参数，则会使用native模式：
```
/etc/nova/nova.conf

preallocate_images=space
```

需要重启nova-compute服务，对于新建(或者hard reboot的)的vm，配置会生效。配置生效后，可以从xml中看到io='native'的下配置：
```
# virsh dumpxml 44 | grep native
      <driver name='qemu' type='qcow2' cache='none' io='native'/>
```


## Performance Test

主要测试了小块的写操作，测试命令如下：
```
4K write:
for i in {1..10}; do dd if=/dev/zero of=./test bs=4K count=10240 oflag=direct; done

16K write:
for i in {1..10}; do dd if=/dev/zero of=./test bs=16K count=10240 oflag=direct; done

64K write:
for i in {1..10}; do dd if=/dev/zero of=./test bs=64K count=10240 oflag=direct; done
```
测试结果如下：
```
                 	4K	    16K	    64K
Prod Phy	        26.97	  54.58	  94.24
Prod VM(threads)	18.46	  32.47	  75.93
Prod VM(native)	  22.46	  49.45	  81.57
```
在我们目前的使用场景中，使用native会比threads更接近物理机器性能。
