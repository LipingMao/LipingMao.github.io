---
title: DevOps -- Terraform Create Route53 Record Slow
---

在用Terraform创建Route53记录时，一条记录往往需要40+s。
```
module.officeSystem.aws_route53_record.tftest: Modifying... [id=Z34FROFM9IC0B2_tftest.pano.video_A]
module.officeSystem.aws_route53_record.tftest: Still modifying... [id=Z34FROFM9IC0B2_tftest.pano.video_A, 10s elapsed]
module.officeSystem.aws_route53_record.tftest: Still modifying... [id=Z34FROFM9IC0B2_tftest.pano.video_A, 20s elapsed]
module.officeSystem.aws_route53_record.tftest: Still modifying... [id=Z34FROFM9IC0B2_tftest.pano.video_A, 30s elapsed]
module.officeSystem.aws_route53_record.tftest: Modifications complete after 39s [id=Z34FROFM9IC0B2_tftest.pano.video_A]

Apply complete! Resources: 0 added, 1 changed, 0 destroyed.
```

这是因为Terraform在创建Route53记录时，判断了记录处于“INSYNC”状态才算完成。一条DNS请求的API发出后，AWS需要一段时间才能将配置完全生效，在配置完全生效前，他的状态是PENDING，完全一致后，状态才为INSYNC。
```
问：对资源记录集的更改是否是事务性的？

是。事务性更改有助于确保更改是一致、可靠的，并且与其他更改独立。Amazon Route 53 已经过了设计，使得更改可在任何个体 DNS 服务器上彻底完成，或者完全不更改。这有助于确保 DNS 查询始终都能获得一致的回答，这在进行目标服务器之间翻转等更改时非常重要。在使用 API 时，对 ChangeResourceRecordSets 的每个调用将返回一个能跟踪相关更改的状态的识别符。一旦状态报告为 INSYNC 时，您的更改就已在所有 Route 53 DNS 服务器上执行完毕。
```

Terraform处理时，等待状态转换为INSYNC：
```
func waitForRoute53RecordSetToSync(conn *route53.Route53, requestId string) error {
        wait := resource.StateChangeConf{
                Delay:      30 * time.Second,
                Pending:    []string{"PENDING"},
                Target:     []string{"INSYNC"},
                Timeout:    30 * time.Minute,
                MinTimeout: 5 * time.Second,
                Refresh: func() (result interface{}, state string, err error) {
                        changeRequest := &route53.GetChangeInput{
                                Id: aws.String(requestId),
                        }
                        return resourceAwsGoRoute53Wait(conn, changeRequest)
                },
        }
        _, err := wait.WaitForState()
        return err
}
```

这个动作导致了需要长时间等待。
