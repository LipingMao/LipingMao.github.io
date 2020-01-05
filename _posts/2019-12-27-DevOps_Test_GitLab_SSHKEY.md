---
title: DevOps -- Tips Test Github ssh key
---


### Tips

以下命令可以测试github ssh key是否正常工作：
```
# ssh -T git@github.com
Hi LipingMao! You've successfully authenticated, but GitHub does not provide shell access.
```



使用以下命令可以替换github从https到git方式：

```
列出现有远程仓库以获取要更改的远程仓库的名称。

$ git remote -v
> origin  https://github.com/USERNAME/REPOSITORY.git (fetch)
> origin  https://github.com/USERNAME/REPOSITORY.git (push)
Change your remote's URL from HTTPS to SSH with the git remote set-url command.

$ git remote set-url origin git@github.com:USERNAME/REPOSITORY.git
验证远程 URL 是否已更改。

$ git remote -v
# Verify new remote URL
> origin  git@github.com:USERNAME/REPOSITORY.git (fetch)
> origin  git@github.com:USERNAME/REPOSITORY.git (push)
```

