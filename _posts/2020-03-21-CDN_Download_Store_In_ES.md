---
title: DevOps -- CDN Log get and filter in ES
---

> CDN的日志往往会有1-3小时的延迟才能产生，因此需要有一个周期性任务下载日志，日志下载后，通过filebeat发送给ES集群，在logstash中需要使用grok分析一下日志内容。

### 下载日志


腾讯云提供CDNLog List接口，可以获取到CDN日志下载地址：
```
class CdnHelper(object):
    SecretId='YOUR_ID'
    SecretKey='YOUR_KEY'
    requestHost='cdn.api.qcloud.com'
    requestUri='/v2/index.php'

    def __init__(self, host, startDate, endDate):
        self.host = host
        self.startDate = startDate
        self.endDate = endDate
        self.params = {
            'Timestamp': int(time.time()),
            'Action': 'GetCdnLogList',
            'SecretId': CdnHelper.SecretId,
            'Nonce': random.randint(10000000,99999999),
            'host': self.host,
            'startDate': self.startDate,
            'endDate': self.endDate
        }
        self.params['Signature'] =  Sign(CdnHelper.SecretId, CdnHelper.SecretKey).make(CdnHelper.requestHost, CdnHelper.requestUri, self.params)
        self.url = 'https://%s%s' % (CdnHelper.requestHost, CdnHelper.requestUri)


    def GetCdnLogList(self):
        ret = requests.get(self.url, params=self.params)
        return ret.json()
```


### Logstash Filter

日志下载后，发送到Logstash时，需要处理一下：
```
if [module] and [module] == "cdn" and [typeSwitch] and [typeSwitch] == "metrics" {
        grok {
          match => {"message" => ["%{DATA:[metrics][cdn][@timestamp]} %{IPORHOST:[metrics][cdn][client_ip]} %{IPORHOST:[metrics][cdn][domain]} (?:%{NOTSPACE:[metrics][cdn][url]}\?%{NOTSPACE:[metrics][cdn][url_sign]}) %{NUMBER:[metrics][cdn][request_bytes]} %{NUMBER:[metrics][cdn][province]} %{NUMBER:[metrics][cdn][ISP]} %{NUMBER:[metrics][cdn][response_code]} (?:%{URI:[metrics][cdn][referrer]}|%{WORD:[metrics][cdn][referrer]}) %{NUMBER:[metrics][cdn][response_time]} %{QS:[metrics][cdn][agent]} %{QS:[metrics][cdn][range]} %{WORD:[metrics][cdn][request_method]} %{NOTSPACE:[metrics][cdn][protocol]} %{WORD:[metrics][cdn][cache_status]}"]}
          remove_field => ["message","[metrics][cdn][url_sign]", "[typeSwitch]"]
        }
        if "_grokparsefailure" in [tags] {
          drop { }
        }
        mutate {
          replace => { "eventtype" => "metrics" }
        }

        date {
          match => ["[metrics][cdn][@timestamp]", "yyyyMMddHHmmss"]
          target => "[metrics][cdn][@timestamp]"  
        }

        if [metrics][cdn][province] == "22" {
          mutate {
            replace => { "[metrics][cdn][province]" => "北京" }
          }
        } else if [metrics][cdn][province] == "86" {
          mutate {
            replace => { "[metrics][cdn][province]" => "内蒙古" }
          }
        } else if [metrics][cdn][province] == "146" {
          mutate {
            replace => { "[metrics][cdn][province]" => "山西" }
          }
        } else if [metrics][cdn][province] == "1069" {
          mutate {
            replace => { "[metrics][cdn][province]" => "河北" }
          }
        } else if [metrics][cdn][province] == "1177" {
          mutate {
            replace => { "[metrics][cdn][province]" => "天津" }
          }
        } else if [metrics][cdn][province] == "119" {
          mutate {
            replace => { "[metrics][cdn][province]" => "宁夏" }
          }
        } else if [metrics][cdn][province] == "152" {
          mutate {
            replace => { "[metrics][cdn][province]" => "陕西" }
          }
        } else if [metrics][cdn][province] == "1208" {
          mutate {
            replace => { "[metrics][cdn][province]" => "甘肃" }
          }
        } else if [metrics][cdn][province] == "1467" {
          mutate {
            replace => { "[metrics][cdn][province]" => "青海" }
          }
        } else if [metrics][cdn][province] == "1468" {
          mutate {
            replace => { "[metrics][cdn][province]" => "新疆" }
          }
        } else if [metrics][cdn][province] == "145" {
          mutate {
            replace => { "[metrics][cdn][province]" => "黑龙江" }
          }
        } else if [metrics][cdn][province] == "1445" {
          mutate {
            replace => { "[metrics][cdn][province]" => "吉林" }
          }
        } else if [metrics][cdn][province] == "1464" {
          mutate {
            replace => { "[metrics][cdn][province]" => "辽宁" }
          }
        } else if [metrics][cdn][province] == "2" {
          mutate {
            replace => { "[metrics][cdn][province]" => "福建" }
          }
        } else if [metrics][cdn][province] == "120" {
          mutate {
            replace => { "[metrics][cdn][province]" => "江苏" }
          }
        } else if [metrics][cdn][province] == "121" {
          mutate {
            replace => { "[metrics][cdn][province]" => "安徽" }
          }
        } else if [metrics][cdn][province] == "122" {
          mutate {
            replace => { "[metrics][cdn][province]" => "山东" }
          }
        } else if [metrics][cdn][province] == "1050" {
          mutate {
            replace => { "[metrics][cdn][province]" => "上海" }
          }
        } else if [metrics][cdn][province] == "1442" {
          mutate {
            replace => { "[metrics][cdn][province]" => "浙江" }
          }
        } else if [metrics][cdn][province] == "182" {
          mutate {
            replace => { "[metrics][cdn][province]" => "河南" }
          }
        } else if [metrics][cdn][province] == "1135" {
          mutate {
            replace => { "[metrics][cdn][province]" => "湖北" }
          }
        } else if [metrics][cdn][province] == "1465" {
          mutate {
            replace => { "[metrics][cdn][province]" => "江西" }
          }
        } else if [metrics][cdn][province] == "1466" {
          mutate {
            replace => { "[metrics][cdn][province]" => "湖南" }
          }
        } else if [metrics][cdn][province] == "118" {
          mutate {
            replace => { "[metrics][cdn][province]" => "贵州" }
          }
        } else if [metrics][cdn][province] == "153" {
          mutate {
            replace => { "[metrics][cdn][province]" => "云南" }
          }
        } else if [metrics][cdn][province] == "1051" {
          mutate {
            replace => { "[metrics][cdn][province]" => "重庆" }
          }
        } else if [metrics][cdn][province] == "1068" {
          mutate {
            replace => { "[metrics][cdn][province]" => "四川" }
          }
        } else if [metrics][cdn][province] == "1155" {
          mutate {
            replace => { "[metrics][cdn][province]" => "西藏" }
          }
        } else if [metrics][cdn][province] == "4" {
          mutate {
            replace => { "[metrics][cdn][province]" => "广东" }
          }
        } else if [metrics][cdn][province] == "173" {
          mutate {
            replace => { "[metrics][cdn][province]" => "广西" }
          }
        } else if [metrics][cdn][province] == "1441" {
          mutate {
            replace => { "[metrics][cdn][province]" => "海南" }
          }
        } else if [metrics][cdn][province] == "0" {
          mutate {
            replace => { "[metrics][cdn][province]" => "其他" }
          }
        } else if [metrics][cdn][province] == "1" {
          mutate {
            replace => { "[metrics][cdn][province]" => "港澳台" }
          }
        } else if [metrics][cdn][province] == "-1" {
          mutate {
            replace => { "[metrics][cdn][province]" => "海外" }
          }
        }

        if [metrics][cdn][ISP] == "2" {
          mutate {
            replace => { "[metrics][cdn][ISP]" => "中国电信" }
          }
        } else if [metrics][cdn][ISP] == "26" {
          mutate {
            replace => { "[metrics][cdn][ISP]" => "中国联通" }
          }
        } else if [metrics][cdn][ISP] == "38" {
          mutate {
            replace => { "[metrics][cdn][ISP]" => "教育网" }
          }
        } else if [metrics][cdn][ISP] == "43" {
          mutate {
            replace => { "[metrics][cdn][ISP]" => "长城宽带" }
          }
        } else if [metrics][cdn][ISP] == "1046" {
          mutate {
            replace => { "[metrics][cdn][ISP]" => "中国移动" }
          }
        } else if [metrics][cdn][ISP] == "3947" {
          mutate {
            replace => { "[metrics][cdn][ISP]" => "中国铁通" }
          }
        } else if [metrics][cdn][ISP] == "-1" {
          mutate {
            replace => { "[metrics][cdn][ISP]" => "海外运营商" }
          }
        } else if [metrics][cdn][ISP] == "0" {
          mutate {
            replace => { "[metrics][cdn][ISP]" => "其他运营商" }
          }
        } 

        if [metrics][cdn][client_ip] {
          geoip{
            source => "[metrics][cdn][client_ip]"
            target => "[metrics][cdn][geoip]"
          }
          geoip{
            source => "[metrics][cdn][client_ip]"
            target => "[metrics][cdn][geoip]"
            default_database_type => "ASN"
          }
        }

```

注：
处理时，大概解析了一下日志的格式，然后对省份运营商进行了翻译。并且对IP地址使用Geo DNS查询了一下地址和运营商。
