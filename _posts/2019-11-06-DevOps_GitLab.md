---
title: DevOps -- Gitlab CI Memo
---

> 利用Gitlab进行CI

## Gitlab Runner

Gitlab支持注册多个Runner作为CI Job的Worker。用户可以灵活的定义不同Runner和不同Repo的绑定关系，同时定义CI Job的tag来绑定不同的Runner。配置注册一个Runner非常简单，只需要安装Runner的包，使用Token注册到Gitlab中即可。


## Job Define

定义gitlab的动作，仅需要在根目录添加.gitlab-ci.yml，以下为一个简单例子
```
stages:
  - build
  - test
  - deploy

build_gslb:
  stage: build
  variables:
    GIT_SUBMODULE_STRATEGY: recursive
  script:
    - cd ./build; sh ./build.sh
  tags:
    - aws

```

用户可以自定义不同的Stage，定义不同Job，使用tag和不同Runner绑定。
