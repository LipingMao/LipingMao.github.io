---
title: DevOps -- Blackbox exporter debug 
---

当blackbox exporter失败时，往往需要回溯当时出错的原因，blackbox exporter提供debug接口可以查询当时错误信息：
```
http://$IP:9115/debug
```

例如:
以下为blackbox监控OCSP失败时的动作：
```
Logs for the probe:
ts=2020-09-15T13:35:13.16009739Z caller=main.go:304 module=http_2xx target=http://status.rapidssl.com level=info msg="Beginning probe" probe=http timeout_seconds=5
ts=2020-09-15T13:35:13.160233349Z caller=http.go:274 module=http_2xx target=http://status.rapidssl.com level=info msg="Resolving target address" ip_protocol=ip4
ts=2020-09-15T13:35:13.161102726Z caller=http.go:274 module=http_2xx target=http://status.rapidssl.com level=info msg="Resolved target address" ip="unsupported value type"
ts=2020-09-15T13:35:13.161168297Z caller=main.go:119 module=http_2xx target=http://status.rapidssl.com level=info msg="Making HTTP request" url=http://117.18.237.29 host=status.rapidssl.com
ts=2020-09-15T13:35:18.160204847Z caller=main.go:119 module=http_2xx target=http://status.rapidssl.com level=error msg="Error for HTTP request" err="Get http://117.18.237.29: context deadline exceeded"
ts=2020-09-15T13:35:18.160238314Z caller=main.go:119 module=http_2xx target=http://status.rapidssl.com level=info msg="Response timings for roundtrip" roundtrip=0 start=2020-09-15T13:35:13.161246102Z dnsDone=2020-09-15T13:35:13.161246102Z connectDone=2020-09-15T13:35:18.16021687Z gotConn=0001-01-01T00:00:00Z responseStart=0001-01-01T00:00:00Z end=0001-01-01T00:00:00Z
ts=2020-09-15T13:35:18.160264656Z caller=main.go:304 module=http_2xx target=http://status.rapidssl.com level=error msg="Probe failed" duration_seconds=5.000111226



Metrics that would have been returned:
# HELP probe_dns_lookup_time_seconds Returns the time taken for probe dns lookup in seconds
# TYPE probe_dns_lookup_time_seconds gauge
probe_dns_lookup_time_seconds 0.000892783
# HELP probe_duration_seconds Returns how long the probe took to complete in seconds
# TYPE probe_duration_seconds gauge
probe_duration_seconds 5.000111226
# HELP probe_failed_due_to_regex Indicates if probe failed due to regex
# TYPE probe_failed_due_to_regex gauge
```
