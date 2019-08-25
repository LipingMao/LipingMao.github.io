---
title: DevOps -- Container start on boot
---

Here is a quick memo:
https://docs.docker.com/config/containers/start-containers-automatically/

Sample code to update a running container:

```
docker update --restart always $CONTAINER_ID
```
