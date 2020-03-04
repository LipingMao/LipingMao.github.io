---
title: DevOps -- Terraform Import
---

> Terraform can import existed resource into state.

Here is an example:
```
[root@5b043787b854 WebSite]# terraform import module.WebSite.aws_route53_record.console Z34FROFM9IC0B2_console.pano.video_CNAME
module.WebSite.aws_route53_record.console: Importing from ID "Z34FROFM9IC0B2_console.pano.video_CNAME"...
module.WebSite.aws_route53_record.console: Import prepared!
  Prepared aws_route53_record for import
module.WebSite.aws_route53_record.console: Refreshing state... [id=Z34FROFM9IC0B2_console.pano.video_CNAME]
Import successful!
The resources that were imported are shown above. These resources are now in
your Terraform state and will henceforth be managed by Terraform.
```

you can use terraform import $[Module.Name] ${ID}, the ID is different for different resource, you need to check doc for the resource which you used.
