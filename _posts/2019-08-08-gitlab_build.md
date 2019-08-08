---
title: DevOps -- Build Gitlab in AWS
---

## Background

> We need to have a private repo for project. Although gitlab has private, but only 3 co-contributor can be add in the free tier, so we decide to build our own private repo. Gitlab-ee without license(functionally the same as gitlab-ce) is a good choice in our use case.

## Steps

Here is step by step install guide:
1. Install dependency RPMs
```
yum install -y curl policycoreutils-python openssh-server
curl https://packages.gitlab.com/install/repositories/gitlab/gitlab-ee/script.rpm.sh | sudo bash
```

2. Install gitlab-ee(replace 10.0.4.89 to your ip in your use case)
```
EXTERNAL_URL="http://10.0.4.89" yum install -y gitlab-ee
```
