---
title: DevOps -- Sonar
---



> 编写代码过程中，通常需要对代码进行静态扫描，用以确保代码质量。目前我们使用Sonar对代码进行分析，并集成在CI过程中。



安装Sonar Server没什么好说的，follow 官方文档安装即可，安装完后，以GoLang为例使用Sonar作为代码静态扫描工具时可以配置以下配置：

```
sonar.host.url=${YOUR_SONAR_SERVER}
sonar.projectKey=gslb
sonar.login=${YOUR_USERNAME}
sonar.password=${YOUR_PASSEORD}

sonar.sources=.
sonar.exclusions=**/*_test.go,**/vendor/**,**/test/**  > 
sonar.exclusions=**/vendor/**,**/test/**
sonar.tests=.
sonar.test.inclusions=**/*_test.go
sonar.test.exclusions=**/vendor/*,**/test/***
sonar.go.coverage.reportPaths=/tmp/coverage.out
```

其中Sonar可以定义，哪些代码是不需要包含在扫描范围的，例如vendor，test等。



如果要使用Sonar进行coverage分析，Golang需要实现生成好coverage文件，可以在CI中类似以下设置：

```
Unit Test:
  stage: UT
  script:
    - go test ./pkg/... -coverprofile=coverage.out
    - mv ./coverage.out /tmp/coverage.out
  before_script:
    - cd /home/gitlab-runner/go/src
    - rm -f gslb
    - ln -s $CI_PROJECT_DIR
    - cd $CI_PROJECT_NAME
  tags:
    - build
  only:
    - merge_requests

Scan Code:
  stage: SCAN
  script:
    - sonar-scanner -Dproject.settings=./sonar-scanner.properties
  tags:
    - build
  only:
    - merge_requests
```

此例中，第一步UT将测试结果保存，第二步，Sonar使用第一步的UT结果进行分析。



当然集成了Sonar后，最好再集成一下Sonar回调Gitlab，让Sonar将结果直接Comments到Gitlab中，遗憾的是最新版本的Sonar与Gitlab集成有bug，不过此处写一个简单的服务回调Gitlab Comments的API也是非常容易做到的的。至此，当提交merge_requests后，就会直接进行UT测试，Sonar扫描代码并将扫描结果Comments到merge requests里，方便后续review代码时参考。