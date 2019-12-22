---
title: DevOps -- HttpTrace
---



> 既然ICMP Ping不能在函数计算中使用，想在函数中尝试一下Http相关的测试，使用函数计算达到httpstat类似的效果。



先看一下命令行版本httpstat [1]可以获取到什么样的metrics：

```
➜  bin ./httpstat www.webex.com

Connected to 223.119.152.185:443

HTTP/2.0 200 OK
Server: Apache
Accept-Ranges: bytes
Access-Control-Allow-Credentials: false
Access-Control-Allow-Headers: *
Access-Control-Allow-Methods: POST, GET, OPTIONS, DELETE, PUT
Access-Control-Max-Age: 86400
Cache-Control: public, max-age=85508
Cdchost: wbxxweb-pub-prd2-01
Content-Type: text/html
Date: Sun, 22 Dec 2019 13:27:50 GMT
Expires: Mon, 23 Dec 2019 13:12:58 GMT
Last-Modified: Sat, 21 Dec 2019 00:38:00 GMT
Server-Timing: cdn-cache; desc=HIT,edge; dur=2
Strict-Transport-Security: max-age=31536000; includeSubDomains; preload
Vary: Accept-Encoding
X-Akamai-Transformed: 9 - 0 pmb=mRUM,3
X-Content-Type-Options: nosniff
X-Frame-Options: SAMEORIGIN
X-Wbx-About: CN, 112.0.150.93
X-Xss-Protection: 1; mode=block

Body discarded

  DNS Lookup   TCP Connection   TLS Handshake   Server Processing   Content Transfer
[    221ms  |          80ms  |        301ms  |            128ms  |           551ms  ]
            |                |               |                   |                  |
   namelookup:221ms          |               |                   |                  |
                       connect:302ms         |                   |                  |
                                   pretransfer:604ms             |                  |
                                                     starttransfer:732ms            |
                                                                                total:1284ms
```



这里可以看到，httpstat可以返回DNS查询时间，TCP建立连接，TLS握手，Server处理，内容传输各个阶段的时间，对于HTTP Server来说这些Metrics有一定的参考价值。

自然的一个问题是，这些数据httpstats是怎么获取到的呢？扫一眼httpstats的源码，发现本身利用了golang的HTTP Tracing。HTTP Tracing是golang在1.7版本引入的功能，原理非常简单，httptrace提供了很多Hook点，可以在请求不同阶段调用hook函数。



查看ClientTrace struct可以看到提供了以下Hock点：

```
type ClientTrace struct {
    // GetConn is called before a connection is created or
    // retrieved from an idle pool. The hostPort is the
    // "host:port" of the target or proxy. GetConn is called even
    // if there's already an idle cached connection available.
    GetConn func(hostPort string)

    // GotConn is called after a successful connection is
    // obtained. There is no hook for failure to obtain a
    // connection; instead, use the error from
    // Transport.RoundTrip.
    GotConn func(GotConnInfo)

    // PutIdleConn is called when the connection is returned to
    // the idle pool. If err is nil, the connection was
    // successfully returned to the idle pool. If err is non-nil,
    // it describes why not. PutIdleConn is not called if
    // connection reuse is disabled via Transport.DisableKeepAlives.
    // PutIdleConn is called before the caller's Response.Body.Close
    // call returns.
    // For HTTP/2, this hook is not currently used.
    PutIdleConn func(err error)

    // GotFirstResponseByte is called when the first byte of the response
    // headers is available.
    GotFirstResponseByte func()

    // Got100Continue is called if the server replies with a "100
    // Continue" response.
    Got100Continue func()

    // Got1xxResponse is called for each 1xx informational response header
    // returned before the final non-1xx response. Got1xxResponse is called
    // for "100 Continue" responses, even if Got100Continue is also defined.
    // If it returns an error, the client request is aborted with that error value.
    Got1xxResponse func(code int, header textproto.MIMEHeader) error // Go 1.11

    // DNSStart is called when a DNS lookup begins.
    DNSStart func(DNSStartInfo)

    // DNSDone is called when a DNS lookup ends.
    DNSDone func(DNSDoneInfo)

    // ConnectStart is called when a new connection's Dial begins.
    // If net.Dialer.DualStack (IPv6 "Happy Eyeballs") support is
    // enabled, this may be called multiple times.
    ConnectStart func(network, addr string)

    // ConnectDone is called when a new connection's Dial
    // completes. The provided err indicates whether the
    // connection completedly successfully.
    // If net.Dialer.DualStack ("Happy Eyeballs") support is
    // enabled, this may be called multiple times.
    ConnectDone func(network, addr string, err error)

    // TLSHandshakeStart is called when the TLS handshake is started. When
    // connecting to a HTTPS site via a HTTP proxy, the handshake happens after
    // the CONNECT request is processed by the proxy.
    TLSHandshakeStart func() // Go 1.8

    // TLSHandshakeDone is called after the TLS handshake with either the
    // successful handshake's connection state, or a non-nil error on handshake
    // failure.
    TLSHandshakeDone func(tls.ConnectionState, error) // Go 1.8

    // WroteHeaderField is called after the Transport has written
    // each request header. At the time of this call the values
    // might be buffered and not yet written to the network.
    WroteHeaderField func(key string, value []string) // Go 1.11

    // WroteHeaders is called after the Transport has written
    // all request headers.
    WroteHeaders func()

    // Wait100Continue is called if the Request specified
    // "Expect: 100-continue" and the Transport has written the
    // request headers but is waiting for "100 Continue" from the
    // server before writing the request body.
    Wait100Continue func()

    // WroteRequest is called with the result of writing the
    // request and any body. It may be called multiple times
    // in the case of retried requests.
    WroteRequest func(WroteRequestInfo)
}
```



httpstat就是利用这些hock点记录时间，从而计算出不同阶段花费的时间。

[3] 是一个实现，实现了在上述阶段（DNS查询时间，TCP建立连接，TLS握手，Server处理，内容传输）的回调函数。因此可以获取上述cli示例中的信息。



### Refer

[1] https://github.com/davecheney/httpstat

[2] https://blog.golang.org/http-tracing

[3] https://github.com/tcnksm/go-httpstat