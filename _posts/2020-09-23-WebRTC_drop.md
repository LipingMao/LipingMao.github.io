---
title: WebRTC -- drop packet
---

 1. 接收端维护两个计数器，每收到一个RTP包都更新：

* transmitted，接收到的RTP包的总数；
* retransmitted，接收到重传RTP包的数量；

2.某时刻收到的有序包的数量Count = transmitted-retransmitte ，当前时刻为Count2，上一时刻为Count1;

3.接收端以一定的频率发送RTCP包（RR、REMB、NACK等）时，会统计两次发送间隔之间(fraction)的接收包信息：
      //两次发送间隔之间理论上应该收到的包数量=当前接收到的最大包序号-上个时刻最大有序包序号
      uint16_t exp_since_last = (received_seq_max_ - last_report_seq_max_);
     
      //两次发送间隔之间实际接收到有序包的数量=当前时刻收到的有序包的数量-上一个时刻收到的有序包的数量
      uint32_t rec_since_last = Count2 - Count1
     
      //丢包数=理论上应收的包数-实际收到的包数
      int32_t missing = exp_since_last - rec_since_last

      missing即为两次发送间隔之间的丢包数量，会累加并通过RR包通知发送端。
