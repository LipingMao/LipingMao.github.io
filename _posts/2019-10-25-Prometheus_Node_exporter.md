---
title: Prometheus Node exporter
---



> Prometheus Node Exporter是Node监控基础，简单走读Node Exporter源码[1]。



## 源码目录结构

Node_exporter的代码结构非常简单，Node Exporter启动了一个http server，对/metrics作出响应，每次GET /metrics就收集Node当时数据。主要代码仅需关注以下两个：

```
collector/
node_exporter.go
```



node_exporter完成了HTTP的启动，加载handler等动作，collector完成了各个collect的实际动作。



## 主要流程

找到main入口如下：

```go
func main() {
## 启动参数初始化
	var (
		listenAddress = kingpin.Flag(
			"web.listen-address",
			"Address on which to expose metrics and web interface.",
		).Default(":9100").String()
...

## 加载 ”/“ 和 “metrics”
	http.Handle(*metricsPath, newHandler(!*disableExporterMetrics, *maxRequests))
	http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
		w.Write([]byte(`<html>
			<head><title>Node Exporter</title></head>
			<body>
			<h1>Node Exporter</h1>
			<p><a href="` + *metricsPath + `">Metrics</a></p>
			</body>
			</html>`))
	})

## 启动HTTP Server
	log.Infoln("Listening on", *listenAddress)
	if err := http.ListenAndServe(*listenAddress, nil); err != nil {
		log.Fatal(err)
	}
}
```



初始化handler时，会调用到innerHandler，这里会进行注册collector到Registry中。 关于Registry的主要动作是在prometheus/client_golang/prometheus/registry.go中实现的，简单看一下数据结构：



```go
type Registry struct {
   mtx                   sync.RWMutex
   collectorsByID        map[uint64]Collector // ID is a hash of the descIDs.
   descIDs               map[uint64]struct{}
   dimHashesByName       map[string]uint64
   uncheckedCollectors   []Collector
   pedanticChecksEnabled bool
}

## Register实现了注册Collector的工作
func (r *Registry) Register(c Collector) error {}
func (r *Registry) Unregister(c Collector) bool {}
func (r *Registry) MustRegister(cs ...Collector) {}
## Gather方法实现了收集metrics的功能
func (r *Registry) Gather() ([]*dto.MetricFamily, error) {}
```



每一个Collector仅需要实现一下Update接口：

```go
// Collector is the interface a collector has to implement.
type Collector interface {
   // Get new metrics and expose them via prometheus registry.
   Update(ch chan<- prometheus.Metric) error
}
```



以conntrack_linux Collector为例：

```go
func init() {
   registerCollector("conntrack", defaultEnabled, NewConntrackCollector)
}
```

在初始化时，注册到了一个Collector list中。 并且将工厂方法NewConntrackCollector同时记录。NewConntrackCollector返回了conntrackCollector。 而conntrackCollector是collector接口的具体实现，在其Update方法中做了需要收集的metrics动作， 在conntrack这个例子中，仅仅是读取了sys下的内核参数获取conntrack的最大值以及当前conntrack的值。

```go
func (c *conntrackCollector) Update(ch chan<- prometheus.Metric) error {
   value, err := readUintFromFile(procFilePath("sys/net/netfilter/nf_conntrack_count"))
   if err != nil {
      // Conntrack probably not loaded into the kernel.
      return nil
   }
   ch <- prometheus.MustNewConstMetric(
      c.current, prometheus.GaugeValue, float64(value))

   value, err = readUintFromFile(procFilePath("sys/net/netfilter/nf_conntrack_max"))
   if err != nil {
      return nil
   }
   ch <- prometheus.MustNewConstMetric(
      c.limit, prometheus.GaugeValue, float64(value))

   return nil
}
```



其他collector也以类似方式注册，最终由prometheus周期调用收集存储metrics数据。



## Refer

[1] https://github.com/prometheus/node_exporter