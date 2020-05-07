---
title: DevOps -- net.ipv4.tcp_workaround_signed_windows
---

There is a Kernel Option : net.ipv4.tcp_workaround_signed_windows

It is disabled by default:
```
tcp_workaround_signed_windows  (Boolean; default: disabled; since Linux
   2.6.26)
          If enabled, assume that no receipt of  a  window-scaling  option
          means  that  the remote TCP is broken and treats the window as a
          signed quantity.  If disabled, assume that the remote TCP is not
          broken  even  if  we do not receive a window scaling option from
          it.
```

This Option is used as workaround for some System which use TCP window as "signed int16". It should be "unsigned int16".

From the code, when sysctl_tcp_workaround_signed_windows the max of TCP window will be MAX_TCP_WINDOW. 
```
#define MAX_TCP_WINDOW		32767U


	/* NOTE: offering an initial window larger than 32767
	 * will break some buggy TCP stacks. If the admin tells us
	 * it is likely we could be speaking with such a buggy stack
	 * we will truncate our initial window offering to 32K-1
	 * unless the remote has sent us a window scaling option,
	 * which we interpret as a sign the remote TCP is not
	 * misinterpreting the window field as a signed quantity.
	 */
	if (sock_net(sk)->ipv4.sysctl_tcp_workaround_signed_windows)
		(*rcv_wnd) = min(space, MAX_TCP_WINDOW);
	else
		(*rcv_wnd) = min_t(u32, space, U16_MAX);
```


[1] https://github.com/torvalds/linux/blob/1b649e0bcae71c118c1333e02249a7510ba7f70a/net/ipv4/tcp_output.c#L208
