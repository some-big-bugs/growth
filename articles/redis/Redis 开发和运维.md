<h1> Redis 开发与运维</h1>

**Table of Contents**
- [初识Redis](#%e5%88%9d%e8%af%86redis)
- [常用命令](#%e5%b8%b8%e7%94%a8%e5%91%bd%e4%bb%a4)
- [缓存设计](#%e7%bc%93%e5%ad%98%e8%ae%be%e8%ae%a1)
  - [收益和风险](#%e6%94%b6%e7%9b%8a%e5%92%8c%e9%a3%8e%e9%99%a9)
  - [缓存更新策略](#%e7%bc%93%e5%ad%98%e6%9b%b4%e6%96%b0%e7%ad%96%e7%95%a5)
  - [缓存粒度](#%e7%bc%93%e5%ad%98%e7%b2%92%e5%ba%a6)
  - [缓存击穿](#%e7%bc%93%e5%ad%98%e5%87%bb%e7%a9%bf)
  - [缓存雪崩](#%e7%bc%93%e5%ad%98%e9%9b%aa%e5%b4%a9)

# 初识Redis

Redis 是一种基于键值的 NoSQL 数据库，Redis 提供了丰富的类型来满足不同的场景。

支持的数据类型：

1. string 字符串
1. hash 哈希
2. list 列表
3. set 集合 
4. zset 有序集合
5. bitmaps 位图
6. hyperLogLog
7. GEO 地理信息

附加功能：

1. 发布订阅
2. 事务
3. 流水线
4. Lua脚本

特性：

1. 速度快
2. 功能丰富
3. 简单稳定
4. 支持多种语言
5. 持久化
6. 主从复制
7. 高可用和分布式

# 常用命令

# 缓存设计

## 收益和风险

**收益**

1. 加速读写
2. 降低后端负载

**风险**

1. 数据不一致
2. 代码维护成本
3. 运维维护成本

## 缓存更新策略

**Redis的回收策略**

volatile-lru：从已设置过期时间的数据集（server.db[i].expires）中挑选最近最少使用的数据淘汰

volatile-ttl：从已设置过期时间的数据集（server.db[i].expires）中挑选将要过期的数据淘汰

volatile-random：从已设置过期时间的数据集（server.db[i].expires）中任意选择数据淘汰

allkeys-lru：从数据集（server.db[i].dict）中挑选最近最少使用的数据淘汰

allkeys-random：从数据集（server.db[i].dict）中任意选择数据淘汰

no-enviction（驱逐）：禁止驱逐数据

**使用场景**

## 缓存粒度

到底什么数据应该被缓存

## 缓存击穿

## 缓存雪崩

1. 保证缓存高可用
2. 应用限流
3. 提前预演
