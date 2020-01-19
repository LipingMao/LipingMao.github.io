---
title: DevOps -- Proxy Protocol
---

> 本文介绍代理协议(Proxy Protocol)相关内容。[1]

### 解决什么问题？

在当前服务架构中，负载均衡设备(或者代理设备)常被使用，当我们使用了负载均衡/代理设备之后，服务器端通常看到的是代理设备的IP地址。在HTTP协议下，通常在代理设备上会追加X-Forward-For头，用于存放原始客户端的IP地址，这样服务器端可以看到原始客户端IP。但是在UDP/TCP四层情况下，不存在可以追加Client IP信息的字段。Proxy Protocol是可以用来解决UDP/TCP四层情况下服务器端获取IP地址的问题。他工作在4层和7层之间，追加了一个Proxy Protocol头部，用于记录客户端地址信息。

### 什么是Proxy Protocol

目前Proxy-protocol支持两个版本，v1和v2。v1为方便人可读。v2加入了
```
12字节的固定signature。 \x0D\x0A\x0D\x0A\x00\x0D\x0A\x51\x55\x49\x54\x0A
4bits 协议版本号: \x2 : v2
4bits cmd: \x0 : LOCAL \x1 : PROXY
4bits 地址族 \x0 : AF_UNSPEC \x1 : AF_INET \x2 : AF_INET6 \x3 : AF_UNIX
4bits transport protocol \x0 : UNSPEC \x1 : STREAM \x2 : DGRAM
2字节地址长度字段(网络字节序)，指接下来剩余的报文长度
```

可使用以下方式描述：
```
    struct proxy_hdr_v2 {
        uint8_t sig[12];  /* hex 0D 0A 0D 0A 00 0D 0A 51 55 49 54 0A */
        uint8_t ver_cmd;  /* protocol version and command */
        uint8_t fam;      /* protocol family and address */
        uint16_t len;     /* number of following bytes part of the header */
    };
```

### Nginx中的相关实现

目前Ngnix支持了Proxy Protocol的V2版本，但是仅支持TCP的Proxy Protocol。

Proxy Protocol的协议头部定义如下：
```
src/core/ngx_proxy_protocol.c

typedef struct {
    u_char                                  signature[12];
    u_char                                  version_command;
    u_char                                  family_transport;
    u_char                                  len[2];
} ngx_proxy_protocol_header_t;
```

处理Proxy Protocol的主要函数是：
ngx_proxy_protocol_read   用于代理协议解析
ngx_proxy_protocol_write  用于代理协议组装

ngx_proxy_protocol_read代码如下：
```
u_char *
ngx_proxy_protocol_read(ngx_connection_t *c, u_char *buf, u_char *last)
{
    size_t                 len;
    u_char                *p;
    ngx_proxy_protocol_t  *pp;

    static const u_char signature[] = "\r\n\r\n\0\r\nQUIT\n";

    p = buf;
    len = last - buf;

    if (len >= sizeof(ngx_proxy_protocol_header_t)
        && memcmp(p, signature, sizeof(signature) - 1) == 0)
    {
        // 当读到Proxy Protocol v2头部时，进入ngx_proxy_protocol_v2_read处理：
        return ngx_proxy_protocol_v2_read(c, buf, last);
    }
    ...



static u_char *
ngx_proxy_protocol_v2_read(ngx_connection_t *c, u_char *buf, u_char *last)
{
    u_char                             *end;
    size_t                              len;
    socklen_t                           socklen;
    ngx_uint_t                          version, command, family, transport;
    ngx_sockaddr_t                      src_sockaddr, dst_sockaddr;
    ngx_proxy_protocol_t               *pp;
    ngx_proxy_protocol_header_t        *header;
    ngx_proxy_protocol_inet_addrs_t    *in;
#if (NGX_HAVE_INET6)
    ngx_proxy_protocol_inet6_addrs_t   *in6;
#endif

    header = (ngx_proxy_protocol_header_t *) buf;

    buf += sizeof(ngx_proxy_protocol_header_t);

    version = header->version_command >> 4;

    // 目前只有2
    if (version != 2) {
        ngx_log_error(NGX_LOG_ERR, c->log, 0,
                      "unknown PROXY protocol version: %ui", version);
        return NULL;
    }

    len = ngx_proxy_protocol_parse_uint16(header->len);

    if ((size_t) (last - buf) < len) {
        ngx_log_error(NGX_LOG_ERR, c->log, 0, "header is too large");
        return NULL;
    }

    end = buf + len;

    command = header->version_command & 0x0f;

    /* only PROXY is supported */
    if (command != 1) {
        ngx_log_debug1(NGX_LOG_DEBUG_CORE, c->log, 0,
                       "PROXY protocol v2 unsupported command %ui", command);
        return end;
    }

    transport = header->family_transport & 0x0f;

    // Nginx目前只实现了TCP，没有支持UDP的Proxy Protocol
    /* only STREAM is supported */
    if (transport != 1) {
        ngx_log_debug1(NGX_LOG_DEBUG_CORE, c->log, 0,
                       "PROXY protocol v2 unsupported transport %ui",
                       transport);
        return end;
    }

    pp = ngx_pcalloc(c->pool, sizeof(ngx_proxy_protocol_t));
    if (pp == NULL) {
        return NULL;
    }

    family = header->family_transport >> 4;

    switch (family) {

    // 解析IPv4地址
    case NGX_PROXY_PROTOCOL_AF_INET:

        if ((size_t) (end - buf) < sizeof(ngx_proxy_protocol_inet_addrs_t)) {
            return NULL;
        }

        ...

        break;

#if (NGX_HAVE_INET6)

        // IPv6
        ...

#endif

    default:
        ngx_log_debug1(NGX_LOG_DEBUG_CORE, c->log, 0,
                       "PROXY protocol v2 unsupported address family %ui",
                       family);
        return end;
    }

   ...
}
```

添加Proxy Protocol头部动作如下：
```
u_char *
ngx_proxy_protocol_write(ngx_connection_t *c, u_char *buf, u_char *last)
{
    ngx_uint_t  port, lport;

    if (last - buf < NGX_PROXY_PROTOCOL_MAX_HEADER) {
        return NULL;
    }

    if (ngx_connection_local_sockaddr(c, NULL, 0) != NGX_OK) {
        return NULL;
    }

    switch (c->sockaddr->sa_family) {

    case AF_INET:
        buf = ngx_cpymem(buf, "PROXY TCP4 ", sizeof("PROXY TCP4 ") - 1);
        break;

#if (NGX_HAVE_INET6)
    case AF_INET6:
        buf = ngx_cpymem(buf, "PROXY TCP6 ", sizeof("PROXY TCP6 ") - 1);
        break;
#endif

    default:
        return ngx_cpymem(buf, "PROXY UNKNOWN" CRLF,
                          sizeof("PROXY UNKNOWN" CRLF) - 1);
    }

    buf += ngx_sock_ntop(c->sockaddr, c->socklen, buf, last - buf, 0);

    *buf++ = ' ';

    buf += ngx_sock_ntop(c->local_sockaddr, c->local_socklen, buf, last - buf,
                         0);

    port = ngx_inet_get_port(c->sockaddr);
    lport = ngx_inet_get_port(c->local_sockaddr);

    return ngx_slprintf(buf, last, " %ui %ui" CRLF, port, lport);
}
```

### 总结

可以看到Proxy Protocol是用来在四层TCP/UDP头部后添加的一个头部，用于给Server端获取客户端IP。目前Nginx中仅支持TCP的Proxy Protocol，UDP目前不支持。

[1] https://www.haproxy.org/download/1.8/doc/proxy-protocol.txt

