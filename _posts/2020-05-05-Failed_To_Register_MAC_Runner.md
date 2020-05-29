---
title: DevOps -- Mac GitLab Runner register failed
---

### 问题：

Mac Gitlab Runner注册失败。显示“New runner, Has not connected yet.”


### 问题原因：

gitlab-runner 在mac版本register时，注意一定不要sudo，否则虽然可以注册上，但是不能正常工作。


### 完整步骤：

安装gitlab runner： 
```
download gitlab-runner
mv gitlab-runner /usr/local/bin/
chmod +x /usr/local/bin/gitlab-runner
```


注册gitlab runner：
```
/usr/local/bin/gitlab-runner register
gitlab-runner install
gitlab-runner start
```
