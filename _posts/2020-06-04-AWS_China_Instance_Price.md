---
title: DevOps -- AWS China Instance Price
---



> 本文是总结国内AWS主机价格。



AWS的虚拟机的实例分类如下：

| 实例类型 | 付费                                               | 备注                                               |
| -------- | -------------------------------------------------- | -------------------------------------------------- |
| 按需实例 | 按小时收费                                         | 随时启动，删除，但价格贵。                         |
| 预留实例 | commit使用1年或者3年，获得按需实例的60-80%的折扣。 | 价格相比按需实例便宜，但需要Commit 1年/3年的使用。 |
| Spot实例 | 最高90%的折扣                                      | 竞价启动，随时删除。                               |
| 专用主机 | 相比按需实例单价更高，有更好的隔离性。             | 价格高，除非在需要Dedicated主机场景                |

其中：

Spot实例是按照竞价方式启动，专用主机是Dedicate的机器，都不适合目前生产环境。 主要关注按需实例和预留实例。





AWS宁夏区域c5.xlarge（4cpu/8G）RI实例为例：

| 实例类型  | 期限 | 产品类型     | 预付价格（人民币） | 使用价格（人民币） | 月度成本（人民币） | 有效 RI 率 | 与 OD 相比的成本节省 | OD 价格（人民币） |
| --------- | ---- | ------------ | ------------------ | ------------------ | ------------------ | ---------- | -------------------- | ----------------- |
| c5.xlarge | 1 年 | 无预付费用   | 0                  | 0.35               | 255.5              | 0.35       | 65%                  | 0.986             |
| c5.xlarge | 1 年 | 预付部分费用 | 1458               | 0.166              | 121.18             | 0.332      | 66%                  | 0.986             |
| c5.xlarge | 1 年 | 预付全部费用 | 2858               | 0                  | 0                  | 0.326      | 67%                  | 0.986             |
| c5.xlarge | 3 年 | 无预付费用   | 0                  | 0.232              | 169.36             | 0.232      | 76%                  | 0.986             |
| c5.xlarge | 3 年 | 预付部分费用 | 2824               | 0.107              | 78.11              | 0.214      | 78%                  | 0.986             |
| c5.xlarge | 3 年 | 预付全部费用 | 5309               | 0                  | 0                  | 0.202      | 80%                  | 0.986             |



**从以上对比可以得出：**

1） c5.xlarge的1年期的折扣率在按需实例的3.5折左右，3年期的折扣在2.5折左右。

2）无预付费/部分预付费/全部预付费相差1-2%。





理解国内AWS不同区域（北京，宁夏）之间计算资源的成本：

| 实例类型   | 按需实例（北京） | 按需实例（宁夏） | 宁夏价格/北京价格 |
| ---------- | ---------------- | ---------------- | ----------------- |
| c5.large   | 0.739            | 0.493            | 66.7%             |
| c5.xlarge  | 1.479            | 0.986            | 66.7%             |
| c5.2xlarge | 2.957            | 1.972            | 66.7%             |

注：宁夏价格是北京的2/3。





**Refer：**

https://www.amazonaws.cn/ec2/pricing/ec2-linux-pricing/