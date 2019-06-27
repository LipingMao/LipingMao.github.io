---
title: DevOps -- Cilium route mode datapath
---

## Background

> Researched Cilium work in route mode.

## Datapath

Here is what the data path looks like:

![Datapath1](https://raw.githubusercontent.com/LipingMao/LipingMao.github.io/master/_posts/picture/2019_06_26_1.png)
![Datapath2](https://raw.githubusercontent.com/LipingMao/LipingMao.github.io/master/_posts/picture/2019_06_26_2.png)
![Datapath3](https://raw.githubusercontent.com/LipingMao/LipingMao.github.io/master/_posts/picture/2019_06_26_3.png)

Note:
1) Runs rib_XXX to sync route between kubernetes nodes and route gateway nodes.
2) all traffic goes with route directly.
3) Technically, Gateway nodes can do VPN tunnel to other sites, and when Pod/Service CIDR does not overlap, K8S can talk with each other with pods IP directly.
