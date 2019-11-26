---
title: DevOps  -- Gitlab Runner problem on CentOS7
---


当在CentOS7上部署Gitlab runner自动化测试时，有时会发生以下错误：
```
Fetching changes with git depth set to 50...
Reinitialized existing Git repository in /home/gitlab-runner/builds/fjmPh7Js/0/galaxy/gslb/.git/
fatal: git fetch-pack: expected shallow list
fatal: The remote end hung up unexpectedly
ERROR: Job failed: exit status 1
```

经过调查，发现这是CentOS7上的git版本为1.8+，需要安装git2以上版本。具体安装步骤很简单，参考[2]

升级Git版本后可以解决此问题。




[1] https://stackoverflow.com/questions/56663096/gitlab-runner-doesnt-work-on-a-specific-project

[2] https://computingforgeeks.com/how-to-install-latest-version-of-git-git-2-x-on-centos-7/
