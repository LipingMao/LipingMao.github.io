---
title: DevOps -- Gitlab Job Artifacts
---

> Gitlab 跑的时间久以后，会产生很多Artifacts文件占用大量磁盘。对于现有的Artifacts可以手动删除，并且可以配置定期清理。


### 手动清理artifacts

在gitlab-rails console中运行以下内容，可以清理一周之前的artifacts:
```
builds_with_artifacts = Ci::Build.with_artifacts_archive
builds_to_clear = builds_with_artifacts.where("finished_at < ?", 1.week.ago)
builds_to_clear.find_each do |build|
  build.artifacts_expire_at = Time.now
  build.erase_erasable_artifacts!
end
```


### 配置自动清理

配置文件中启用清理job：
```
gitlab_rails['expire_build_artifacts_worker_cron'] = "50 * * * *"
```

需要在.gitlab-ci.yml中配置artifacts超时：
```
job:
  artifacts:
    expire_in: 1 week
```


[1] https://docs.gitlab.com/ce/administration/job_artifacts.html#job-artifacts-using-too-much-disk-space
