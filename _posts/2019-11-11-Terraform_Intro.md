---
title: Terraform -- Intro
---

> 本文介绍Terraform基础。


## Terraform解决什么问题？

对于使用云基础设施的公司来说，基础设施的构建和变更是不得不面对的问题。人们总是希望更加快速的部署基础设施，并且能够控制基础设施的变更。否则时间一长，就没人知道基础设施发生了什么事，久而久之，没人知道这条ACL代表什么意思（对于上千条ACL的防火墙，要删除去除一条规则，简直是。。。）。基础设施的变更需要能够追溯什么时候，做了什么事情。以便后期debug时，可以理解变更的意义。因此，将基础设施本身作为代码（Infra as code）如果在配置基础设施的第一天可以做到，对于后期维护，是非常有帮助的。Terraform就是业界比较好的基础设施即代码的工具。

## Terraform带来的好处

以下优势：
* 基础设施由代码描述，可以使用git等代码跟踪工具记录基础设施变更过程。
* 基础设施变更本身（代码），可在类似环境进行验证，有助于对于变更的验证。
* Terraform可以在分钟级别完成对基础设施的变更。（新建一个DC的基础设施仅需要2-3分钟）
* Terraform可以进行Plan（try run）看到所有变更对现有环境的影响。
* 所有的变更都通过Terraform完成，自然的，会对所有的资源进行标准化（例如：命名规则等，要进行代码化必须先标准化）
* Terraform几乎支持所有主流云厂商，进行多云支持时，可以相对简单的支持。比如支持国内的阿里云，腾讯云，华为云。国际的AWS，微软，谷歌，甲骨文。

## 使用Terraform的不足

以下不足需要注意：
* 使用了Terraform控制基础设施，那所有对其变更都需要通过Terraform，这点不能说是不足，但是这就要求Terraform能描述几乎所有需要的操作。实践过程中发现，比如腾讯云，并不是所有服务都支持Terraform，例如HAVIP，DNS等。
* 同样是腾讯云为例，腾讯云对于虚拟主机的计费分为按需和包月，对于包月的情况，是跟着虚拟机走的，比如这台虚拟机是包月的，这会造成如果Terraform对基础设施变更时，当我们想实现应用替换部署，例如应用A版本1，升级为版本2时，先部署版本2的VM，再删除版本1的VM，这种情况下，会删除包月的VM，腾讯云仅支持一定的VM退费功能，这点不太灵活。相比之下AWS在这块计费非常灵活。对于RI instance，可以绑定到任何符合条件的VM，并且可以对大小进行灵活拆分。这样对于虚拟机级别“不可变基础设施”的推进非常有帮助。
* Terraform本身升级，Provider需要云厂商维护。当然，目前看社区还是比较active，支持没有太大问题。

## 腾讯云上Terraform

不重复造轮子，官方推出的系列简单操作足以：

Packer ： https://cloud.tencent.com/developer/article/1474736

概念： https://cloud.tencent.com/developer/article/1473838

腾讯云上使用TF：

https://cloud.tencent.com/developer/article/1473713

https://cloud.tencent.com/developer/article/1478955

https://cloud.tencent.com/developer/article/1482560

https://cloud.tencent.com/developer/article/1487537
