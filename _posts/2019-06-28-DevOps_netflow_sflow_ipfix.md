---
title: DevOps -- netFlow/sFlow/IPFIX/Conntrack
---

## Background

> We want to monitor and do analysis 5-Tuple flow level network. We have throughput monitor, but when some network issue happen, we just know some peak happened and do not have too much infomation could help us to understand what happened at that time. So the idea is to monitor 5-Tuple level things to help us do trouble shooting and undestand what happened for the network.

## netFlow v.s. sFlow v.s. IPFIX v.s. conntrack

The first thing jump into my mind is netFlow or sFlow could help us to do this. When I want to choose from netFlow or sFlow, I find this [blog](https://www.plixer.com/blog/netFlow-vs-sFlow-2/netFlow-vs-sFlow-which-is-better/) compared them. In short:

> if you ask any customer that has a network with equipment supporting both sFlow and NetFlow, they will tell you this about sFlow when NetFlow isn’t available: "I’d rather collect NetFlow but, sFlow is better than nothing". 

sFlow is doing sampling "packet based" but not "flow based". netFlow has v5 and v9, IPFIX is based on netFlow v9(sometimes people call it netFlow v10). Basically, if your device support all, then you can choose like this: IPFIX > netFlow(v9) > netFlow(v5) > sFlow. And we can use conntrack to collect 5-Tuple info as well, but conntrack only have flow info, it do not have traffic info.

## Collect

OVS support sFlow, netFlow(v5), IPFIX. But not support netFlow(v9). It can send to logstash agent to transfer to ELK cluster to do analysis.
Conntrack can be collect by prometheus exporter, and pulled by prometheus to show in grafana.

So, in OVS, we support netFlow(v5) but it do not support sampling, sFlow is packet based sampling. So , we should choose IPFIX in OVS.

For Conntrack, we should notice the collect performance and the items if prometheus can support.
