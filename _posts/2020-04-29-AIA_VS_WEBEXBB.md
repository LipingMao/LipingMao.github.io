---
title: DevOps -- AIA V.S. Webex BB
---

Webex在全球部署自己的数据中心，拥有自己的Backbone。如果客户站点Site建在北美，例如cisco.webex.com。 则在中国加入这个站点的人员，大致会走国内到日本/新加坡两条专线，然后走Webex自己的backbone到SJC。今年日本/新加坡数据中心和国内建立了两条专线连接，Webex在这两条专线上，将所有Webex的公网地址，都宣告给了这两条专线（日本是电信，新加坡是联通）。这样当客户从国内访问SJC站点时，就会先到日本/新加坡，然后再到SJC。



从国内Bestrace cisco.webex.com（北美的站点），可以看到国内流量是通过走到上海电信，上海电信到日本，从日本的数据中心进入webex的backbone，再到美国数据中心的。
```
[root@bj01vpn001 ~]# ./besttrace 66.114.168.212
traceroute to 66.114.168.212 (66.114.168.212), 30 hops max, 60 byte packets
 1  *
    *
    *
 2  *
    *
    *
 3  *
    *
    *
 4  *
    *
    *
 5  *
    *
    *
 6  100.65.11.193  0.35 ms  *  Shared Address
    100.65.11.193  0.37 ms  *  Shared Address
    100.65.11.193  0.28 ms  *  Shared Address
 7  54.222.25.6  1.46 ms  AS55960  China Beijing ChinaTelecom
    54.222.25.6  1.48 ms  AS55960  China Beijing ChinaTelecom
    54.222.25.6  1.47 ms  AS55960  China Beijing ChinaTelecom
 8  54.222.0.213  3.11 ms  AS55960  China Beijing ChinaTelecom
    54.222.0.213  3.04 ms  AS55960  China Beijing ChinaTelecom
    54.222.0.213  3.51 ms  AS55960  China Beijing ChinaTelecom
 9  54.222.0.148  2.69 ms  AS55960  China Beijing ChinaTelecom
    54.222.0.148  1.65 ms  AS55960  China Beijing ChinaTelecom
    54.222.0.148  2.23 ms  AS55960  China Beijing ChinaTelecom
10  *
    97.31.110.36.static.bjtelecom.net (36.110.31.97)  3.31 ms  AS4847  China Beijing ChinaTelecom
    97.31.110.36.static.bjtelecom.net (36.110.31.97)  3.11 ms  AS4847  China Beijing ChinaTelecom
11  bj141-142-118.bjtelecom.net (219.141.142.118)  2.84 ms  AS4847  China Beijing ChinaTelecom
    bj141-142-118.bjtelecom.net (219.141.142.118)  3.86 ms  AS4847  China Beijing ChinaTelecom
    bj141-142-118.bjtelecom.net (219.141.142.118)  2.77 ms  AS4847  China Beijing ChinaTelecom
12  59.43.80.5  10.29 ms  *  China Beijing ChinaTelecom
    59.43.80.5  7.83 ms  *  China Beijing ChinaTelecom
    59.43.80.5  5.42 ms  *  China Beijing ChinaTelecom
13  59.43.46.86  23.12 ms  *  China Shanghai ChinaTelecom
    59.43.46.86  23.13 ms  *  China Shanghai ChinaTelecom
    59.43.46.86  23.15 ms  *  China Shanghai ChinaTelecom
14  59.43.130.198  24.10 ms  *  China Shanghai ChinaTelecom
    *
    *
15  59.43.247.62  23.22 ms  *  China Shanghai ChinaTelecom
    59.43.247.62  23.01 ms  *  China Shanghai ChinaTelecom
    59.43.247.62  23.01 ms  *  China Shanghai ChinaTelecom
16  59.43.183.54  59.30 ms  *  Japan Tokyo ChinaTelecom
    59.43.183.54  58.84 ms  *  Japan Tokyo ChinaTelecom
    59.43.183.54  58.85 ms  *  Japan Tokyo ChinaTelecom
17  202.55.27.98  58.72 ms  AS4809  Japan Tokyo ChinaTelecom
    202.55.27.98  58.73 ms  AS4809  Japan Tokyo ChinaTelecom
    202.55.27.98  58.54 ms  AS4809  Japan Tokyo ChinaTelecom
18  nrt02-wxbb-crt01-bu10.webex.com (114.29.223.129)  60.09 ms  AS13445  Japan Tokyo webex.com
    nrt02-wxbb-crt01-bu10.webex.com (114.29.223.129)  60.00 ms  AS13445  Japan Tokyo webex.com
    nrt02-wxbb-crt01-bu10.webex.com (114.29.223.129)  60.31 ms  AS13445  Japan Tokyo webex.com
19  sjc10-wxbb-pe02-be101.webex.com (114.29.223.29)  160.82 ms  AS13445  United States California San Jose webex.com
    sjc10-wxbb-pe02-be101.webex.com (114.29.223.29)  160.90 ms  AS13445  United States California San Jose webex.com
    sjc10-wxbb-pe02-be101.webex.com (114.29.223.29)  160.82 ms  AS13445  United States California San Jose webex.com
20  sjc02-wxbb-crt02-te0-1-0-4.webex.com (64.68.116.133)  159.29 ms  AS13445  United States California San Jose webex.com
    sjc02-wxbb-crt02-te0-1-0-4.webex.com (64.68.116.133)  159.32 ms  AS13445  United States California San Jose webex.com
    sjc02-wxbb-crt02-te0-1-0-4.webex.com (64.68.116.133)  159.63 ms  AS13445  United States California San Jose webex.com
21  sjc02-wxbb-pe01-bu21.webex.com (64.68.117.101)  164.63 ms  AS13445  United States California San Jose webex.com
    sjc02-wxbb-pe01-bu21.webex.com (64.68.117.101)  164.41 ms  AS13445  United States California San Jose webex.com
    sjc02-wxbb-pe01-bu21.webex.com (64.68.117.101)  164.77 ms  AS13445  United States California San Jose webex.com
22  sjc02-wxp00-gw01-po511-100.webex.com (209.197.223.161)  159.29 ms  AS13445  United States California San Jose webex.com
    sjc02-wxp00-gw01-po511-100.webex.com (209.197.223.161)  159.34 ms  AS13445  United States California San Jose webex.com
    sjc02-wxp00-gw01-po511-100.webex.com (209.197.223.161)  159.54 ms  AS13445  United States California San Jose webex.com
23  *
    *
    *
24  global-nebulaac.webex.com (66.114.168.212)  159.22 ms  AS13445  United States California San Jose webex.com
    global-nebulaac.webex.com (66.114.168.212)  159.14 ms  AS13445  United States California San Jose webex.com
    global-nebulaac.webex.com (66.114.168.212)  159.20 ms  AS13445  United States California San Jose webex.com
```





对比各数据中心访问AIA 和 访问Webex 北美站点的速度（[cisco.webex.com](http://cisco.webex.com/))

![image-20200521090750430](https://raw.githubusercontent.com/LipingMao/LipingMao.github.io/master/_posts/picture/image-20200521090750430.png)



从此处可以看到，单单就时延/丢包来说，硅谷AIA的效果，比Webex SJC的效果还要好。这是因为腾讯在国内有直接的入口，Webex必须要从日本/新加坡才能进入。而Webex在日本/新加坡到SJC的专线本身时延大概有100ms，因此北美的国内反向加速AIA效果应该不错。