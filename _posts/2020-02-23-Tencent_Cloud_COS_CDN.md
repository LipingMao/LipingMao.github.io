---
title: DevOps -- COS CDN CI/CD on Tencent Cloud
---

> 有一些静态网站利用腾讯COS和CDN部署，在gitlab中可以配置CICD实现自动化部署。


可以配置类似以下CI job，仅当有tag生成时，进行一次Release部署：
```
stages:
  - Release
  
Upload to COS and Refresh CDN:
  stage: Release
  script:
    - sh /opt/release.sh
  tags:
    - build
  only:
    - tags

```

假设这个Repo中的内容是静态文件，需要同步到cos，然后刷新cdn进行发布，则当你需要release时，仅需要添加一个tag。
在gitlab runner上，配置好/opt/release.sh脚本，这个脚本的作用是上传文件到cos，刷新cdn。

上传文件到cos，可以利用coscmd命令行, 例如：
```
coscmd -c /opt/cos.conf upload -rs ./ /
```
在cos.conf中定义了密钥，bucket，region信息。 -rs会同步整个目录。最后“./”是指同步当前目录，到前缀"/", 即bucket的根。


上传完文件后，需要刷新cdn，可以使用QcloudCdnTools_V2.py工具刷新cdn，例如：
```
python ./QcloudCdnTools_V2.py RefreshCdnDir -u XXX -p XXX --dirs=http://www.pano.video
```

这样，就可以通过打tag的方式控制release，基本实现自动化发布到cos/cdn中。 
