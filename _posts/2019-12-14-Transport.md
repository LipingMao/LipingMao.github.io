---
title: DevOps -- Transport
---


使用httpstat可以轻松构建一个函数，用于检测用户输入的网址的DNSLookup，TCPConnection，TLSHandshake，ServerProcess，ContentTransfer等指标，函数具体实现如下例：
```
package main

import (
	"context"
	"encoding/json"
	"fmt"
	"github.com/tcnksm/go-httpstat"
	"github.com/tencentyun/scf-go-lib/cloudfunction"
	"io"
	"io/ioutil"
	"net/http"
	"time"
)

// Request Body used by this Function
type Body struct {
	URL string `json:"url"`
}

// Tencent Cloud require read event from body
type Event struct {
	Body string `json:"body"`
}

// Response Body
type Response struct {
	DNSLookup        int `json:"DNSLookup"`
	TCPConnection    int `json:"TCPConnection"`
	TLSHandshake     int `json:"TLSHandshake"`
	ServerProcessing int `json:"ServerProcessing"`
	ContentTransfer  int `json:"ContentTransfer"`
	NameLookup       int `json:"NameLookup"`
	Connect          int `json:"Connect"`
	Pretransfer      int `json:"Pretransfer"`
	StartTransfer    int `json:"StartTransfer"`
	Total            int `json:"Total"`
}

// Generate JSON response
func NewResponse(r httpstat.Result) Response {
	end := time.Now()
	return Response{
		DNSLookup:        int(r.DNSLookup / time.Millisecond),
		TCPConnection:    int(r.TCPConnection / time.Millisecond),
		TLSHandshake:     int(r.TLSHandshake / time.Millisecond),
		ServerProcessing: int(r.ServerProcessing / time.Millisecond),
		ContentTransfer:  int(r.ContentTransfer(end) / time.Millisecond),
		NameLookup:       int(r.NameLookup / time.Millisecond),
		Connect:          int(r.Connect / time.Millisecond),
		Pretransfer:      int(r.Pretransfer / time.Millisecond),
		StartTransfer:    int(r.StartTransfer / time.Millisecond),
		Total:            int(r.Total(end) / time.Millisecond),
	}
}


func httpStats(ctx context.Context, event Event) (interface{}, error) {
	var body Body
	if err := json.Unmarshal([]byte(event.Body), &body); err != nil {
		return fmt.Sprintf("invalid request(req: %v, err: %v)", event.Body, err), nil
	}

	req, err := http.NewRequest("GET", body.URL, nil)
	if err != nil {
		return fmt.Sprintf("invalid request(req: %v, err: %v)", event.Body, err), nil
	}

	var result httpstat.Result
	c := httpstat.WithHTTPStat(req.Context(), &result)
	req = req.WithContext(c)

	tr := &http.Transport{
		DisableKeepAlives:      true,
	}
	client := &http.Client{
		Transport:     tr,
		Timeout:       10*time.Second,
	}
	res, err := client.Do(req)
	if err != nil {
		return fmt.Sprintf("fail to request url(req: %v, err: %v)", event.Body, err), nil
	}

	if _, err := io.Copy(ioutil.Discard, res.Body); err != nil {
		return fmt.Sprintf("fail to get response body(req: %v, err: %v)", event.Body, err), nil
	}
	if err := res.Body.Close(); err != nil {
		return fmt.Sprintf("fail to close body(req: %v, err: %v)", event.Body, err), nil
	}
	return NewResponse(result), nil
}

func main() {
	// Make the handler available for Remote Procedure Call by Cloud Function
	cloudfunction.Start(httpStats)
}

```

此处碰到了一个小坑值得一提，在构建http Client时，如果使用默认的Transport，则会启用KeepAlive功能，并且默认情况下会保持TCP Connection一段时间，这意味着，如果连续两次监控同一个网址，DNS，TCP建立连接的时间会是0，因此在函数中将Keep Alive Disable。
另外从此例可以说明，云厂商并不是在函数执行完成后，立刻销毁函数，因为函数本身启动有一个过程（有冷启动问题，之后再谈），启动函数后，会保持一段时间才会销毁，此处发现tcp连接已经建立就是最好的证明（如果每次都销毁，应该每次需要建立TCP连接）。

在使用Go的HTTP Client时，Transport的一些参数值得注意，可以控制TCP传输层的一些指标，需要根据业务场景调整。
