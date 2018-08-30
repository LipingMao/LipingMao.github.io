---
title: DevOps Memo(1) -- Ansible with ladvdc issue
---

## Background

> 在产线上，我们的计算节点启用了ladvd发现上联交换机接口信息。DevOps可以使用Ansible调用ladvdc抓取汇总上联交换机信息。


## What is the problem?

手动调用ladvdc命令显示的内容没有问题，Ansible在调用ladvdc的时候会发生截断，超过80字节的部分被截断了。这导致Ansible分析ladvdc返回结果不准确。


## What is the root cause?

从ladvd的code看到，ladvdc会获取terminal width，在被ansible调用时，使用的时default值80。


```
src/cli.c

272 void cli_header() {
273     int twidth = TERM_DEFAULT;
274     struct winsize ws = {};
275
276     // Try to fetch the terminal width
277     if (ioctl(0, TIOCGWINSZ, &ws) == 0)
278         twidth = ws.ws_col;
279
280     twidth -= TERM_DEFAULT;
281     if (twidth > 0) {
282         if (twidth > 20) {
283             host_width += 10;
284             port_width = twidth;
285         } else {
286             twidth /= 2;
287             host_width += twidth;
288             port_width += twidth;
289         }
290     }
291
292     printf("Capability Codes:\n"
293         "\tr - Repeater, B - Bridge, H - Host, R - Router, S - Switch,\n"
294         "\tW - WLAN Access Point, C - DOCSIS Device, T - Telephone, "
295         "O - Other\n\n");
296     printf("%-*s Local Intf    Proto   "
297         "Hold-time    Capability    Port ID\n", host_width, "Device ID");
298 }
```

一般情况下80的长度也够显示了，但是我们使用的环境中使用了N5K和N2K的环境，port名称会比一般显示的更长。例如:  
在启用N2K FEX的情况下：Eth105/0/20 <-- 101代表fex id  
一般的port名称：       Eth1/20     <-- 长度比较短  
因此，在ansible使用调用ladvdc的时候会发生截断。


## What is the solution?

  1）修改ladvdc的code，增加width长度。
  2）使用ladvdc -b按照K-V的方式输出，正常一行不会超过80字符。

