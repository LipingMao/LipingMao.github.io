---
title: Golang -- ES API
---

> 当CDN日志进入ES之后，通常需要进行一些分析，一方面可以使用Kibana/Grafana作图，看趋势。另外也可以使用ES-Alert对结果进行报警。同时如果有些信息需要落库，也可以写周期性任务对ES进行查询然后落库。本文已Golang为例，调用ES接口进行数据聚合分析某个URL访问次数。


以下示例聚合了一段时间内CDN不同url的访问次数：
```
var reqTemplate = `{
  "aggs": {
    "result": {
      "terms": {
        "field": "metrics.cdn.url.keyword",
        "order": {
          "_count": "desc"
        },
        "size": 100000
      }
    }
  },
  "size": 0,
  "query": {
    "bool": {
      "must": [
        {
          "term": {
            "module": "cdn"
          }
        },
        {
          "range": {
            "@timestamp": {
              "format": "strict_date_optional_time",
              "gte": "%v",
              "lte": "%v"
            }
          }
        }
      ],
      "filter": [
        {
          "match_all": {}
        }
      ]
    }
  }
}`
type ESClient struct {
	api string
}
type ESSearchResult struct {
	Took    int  `json:"took"`
	Timeout bool `json:"timed_out"`
	Aggregations struct {
		Result struct {
			Buckets []struct {
				Key string `json:"key"`
				Count int  `json:"doc_count"`
			} `json:"buckets"`
		} `json:"result"`
	} `json:"aggregations"`
}
func (es *ESClient) Search (r string, url string) (ESSearchResult, error) {
	result := ESSearchResult{}
	client := &http.Client{Timeout: 30 * time.Second}
	req := bytes.NewBufferString(r)
	httpReq, err := http.NewRequest("GET", es.api+ url, req)
	if err != nil {
		return result, err
	}
	httpReq.Header.Set("Content-Type", "application/json")
	resp, err := client.Do(httpReq)
	if err != nil {
		return result, err
	}

	respbody, err := ioutil.ReadAll(resp.Body)
	if err != nil {
		return result,  err
	}
	json.Unmarshal(respbody, &result)
	return result, err
}
```
