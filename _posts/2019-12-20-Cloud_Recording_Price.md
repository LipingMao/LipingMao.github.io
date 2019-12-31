---
title: Cloud Recording Price
---



> 最近在看云端录制的收费，不同厂家不同玩法。



首先什么是云端录制？云端录制通常来说是指将音频/视频存档，可用于回放，质检，归档等用途。云端录制通常会将录制文件存放在云端的对象存储中，有播放/下载需求时，使用CDN加速下载。
可以看到云端录制主要的成本在于：

* 录制文件转码费用（主要云主机计算能力）
* 对象存储的存放费用 （一般在0.1-0.15元每月每GB）
* 对象存储/CDN下载的费用（CDN回流一般在0.15元每GB，CDN下载费用大概在0.1-0.3元每GB之间）
* 存入对象存储时，云主机网络流出费用（如果是按量计费: 腾讯云为例0.8元每GB，AWS中国大约时0.9+元每GB，阿里是0.8元每GB）

可以看到，存储文件大小，下载次数等因素有关。



### 声网

[声网计费]([https://docs.agora.io/cn/cloud-recording/billing_cloud_recording?platform=All%20Platforms#billing](https://docs.agora.io/cn/cloud-recording/billing_cloud_recording?platform=All Platforms#billing))如下：

| 服务             | 报价（元/每 1000 分钟） |
| ---------------- | ----------------------- |
| 录制音频         | 9.00                    |
| 录制高清视频 HD  | 36.00                   |
| 录制超清视频 HD+ | 135.00                  |



总的来说费用还是很高的，他们的玩法是客户提供第三方存储（目前支持七牛，阿里，腾讯，aws）的账户，他们直接通过API发送过去。可以参考[API文档]([https://docs.agora.io/cn/cloud-recording/cloud_recording_api_rest?platform=All%20Platforms#%E5%8F%82%E6%95%B0-1](https://docs.agora.io/cn/cloud-recording/cloud_recording_api_rest?platform=All Platforms#参数-1))。 对于播放来说，则使用第三方存储生成播放链接，见[文档]([https://docs.agora.io/cn/cloud-recording/cloud_recording_onlineplay?platform=All%20Platforms](https://docs.agora.io/cn/cloud-recording/cloud_recording_onlineplay?platform=All Platforms)) 。



注：

不知道为什么声网对云端录制收费这么高，整体成本应该是云端转码（主要计算费用），和存储费用。他们使用的是客户的第三方存储，应该不存在存储费用一说。当然他们自己推送文件到第三方存储时，会产生一次从云端主机网络带宽流出的费用（通常0.8元每GB）。





### UCloud

[计费](https://docs.ucloud.cn/video/ulive/charge)如下：

| 收费项   |          价格 |
| -------- | ------------: |
| 录制时长 | 0.02元/10分钟 |

可以看到UCloud录制大约2元/1000分钟，他们对接的自家的对象存储，对象存储费用另计。单纯从录制费用上看，要比声网价格便宜。





### Zoom

[计费]([https://support.zoom.us/hc/zh-cn/articles/203741855-%E4%BA%91%E5%BD%95%E5%88%B6](https://support.zoom.us/hc/zh-cn/articles/203741855-云录制))如下：

**存储和回放/下载**

- 下方列出了为每种计划类型帐户提供的免费存储容量

| **计划类型** | **帐户免费存储容量** |
| ------------ | -------------------- |
| Pro          | 1 GB/专业版用户      |
| 商业         | 1 GB/专业版用户      |
| 教育         | 0.5 GB/专业版用户    |
| Zoom Rooms   | 1 GB/Zoom Room       |

- 使用的存储容量达到订购容量限值的80%后，帐户所有者将收到电子邮件提醒。
- 额外存储容量 定价如下：

| **计划** | **存储空间** | **额外增加1 GB** |
| -------- | ------------ | ---------------- |
| $40/月   | 100 GB       | $1.5/GB          |
| $100/月  | 500 GB       | $0.5/GB          |
| $500/月  | 3 TB         | $0.1/GB          |



注：

Zoom是会议产品，计费模式是按照存储空间大小收费，和视频PaaS产品玩法不一样。

