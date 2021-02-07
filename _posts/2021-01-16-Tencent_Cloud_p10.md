---
title: DevOps -- Tencent Cloud (Part 10)
---

> 浏览腾讯云当前组件及功能

### NoSQL 数据库

#### 云数据库 Redis

云数据库 Redis（TencentDB for Redis）是由腾讯云提供的兼容 Redis 协议的缓存数据库，具备高可用、高可靠、高弹性等特征。云数据库 Redis 服务兼容 Redis 2.8、Redis 4.0、Redis 5.0 版本协议，提供标准和集群两大架构版本。最大支持4TB的存储容量，千万级的并发请求，可满足业务在缓存、存储、计算等不同场景中的需求。

#### 云数据库 MongoDB

云数据库 MongoDB（TencentDB for MongoDB）是腾讯云基于开源非关系型数据库 MongoDB 专业打造的高性能分布式数据存储服务，完全兼容 MongoDB 协议，适用于面向非关系型数据库的场景。

#### 云数据库 Memcached

云数据库 Memcached（TencentDB for Memcached）是腾讯自主研发的极高性能、内存级、持久化、分布式 Key-Value 存储服务。适用于高速缓存的场景，兼容 Memcached 协议，为您提供主从热备、自动容灾切换、数据备份、故障迁移、实例监控全套服务，无需您关注以上服务的底层细节。

#### 时序数据库 CTSDB

时序数据库 CTSDB（TencentDB for CTSDB）是腾讯云推出的一款分布式、可扩展、支持近实时数据搜索与分析的时序数据库。该数据库为非关系型数据库，提供高效读写、低成本存储、强大的聚合分析能力、实例监控以及数据查询结果可视化等功能。整个系统采用多节点多副本的部署方式，有效保证了数据的高可用性和安全性。

#### 游戏数据库 TcaplusD

游戏数据库 TcaplusDB 是专为游戏设计的 NoSQL 分布式数据存储服务，支持 Protobuf 接口访问，Tcaplus 将 Cache 与硬盘结合，追求高性能的同时，也节省成本，很好地支持全区全服和分区分服，并针对游戏爆发增长和长尾运维特点提供不停机扩缩容、备份容灾、快速回档等全套解决方案，安全可信赖。

#### 云数据库 Tendis

开源 Redis 使用内存存储介质，能够在计算和缓存场景提供超高并发和超低延迟，但是将 Redis 作为存储数据库面临着高成本和低可靠性的缺点，为弥补 Redis 在存储场景的空缺，腾讯云研发了兼容 Redis 协议，且使用磁盘作为存储介质的 KV（key-value）数据库 Tendis。

云数据库 Tendis（TencentDB for Tendis，Tendis）是兼容 Redis 协议的 KV 存储数据库，Tendis 兼容 Redis 4.0 版本协议，并提供存储版和混合存储版两个产品系列，支持千万级的并发请求，可满足业务在 KV 存储场景中的多种需求。
