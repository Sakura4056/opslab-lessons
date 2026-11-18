---
类型: 学习讲义
阶段: 4
标签: [学习讲义, Redis, 缓存]
---

# Redis 入门与持久化

> 阶段笔记：[[阶段4_服务实战]] ｜ 上一讲：[[阶段4_服务实战/02_MySQL安装与日常运维|MySQL安装与日常运维]] ｜ 下一讲：[[阶段4_服务实战/04_LNMP串联实战|LNMP串联实战]]
> Redis = 内存键值数据库。运维关心：**装得起、连得上、持久化策略、别把内存撑爆**。

## 学习目标

- 安装 Redis 并用 redis-cli 完成五大数据类型基本操作
- 理解 RDB 与 AOF 两种持久化的取舍（面试高频）
- 会配置内存上限与淘汰策略，会看 INFO 关键指标

## 核心概念

| 持久化 | 机制 | 优点 | 缺点 |
|---|---|---|---|
| RDB | 定时把内存**快照**落盘（dump.rdb） | 文件小、恢复快 | 两次快照间的数据可能丢 |
| AOF | **追加**每条写命令（appendonly.aof） | 丢失少（每秒/每命令刷盘） | 文件大、恢复慢 |

生产常见组合：**RDB + AOF 同时开**（AOF 保数据，RDB 加速恢复）。

**内存淘汰策略**（maxmemory 满了怎么办）：`noeviction`(默认，写报错) / `allkeys-lru`(最常用) / `volatile-lru`(只淘汰有过期时间的) 等。

## 命令实操

```bash
sudo apt install -y redis-server
sudo systemctl enable --now redis-server
redis-cli                          # 进入交互
```

```redis
PING                               # → PONG 连通性
SET greeting "hello" EX 60         # 写入并 60 秒过期
GET greeting                       # 读
DEL greeting

-- 五大数据类型速览
RPUSH queue a b c                  # List：队列
SADD tags "linux" "ops"            # Set：去重集合
HSET user:1 name li age 28         # Hash：对象
ZADD rank 100 alice 90 bob         # ZSet：带分数排序（排行榜）
TYPE rank

KEYS *                             # ⚠ 生产禁用（阻塞）；用 SCAN
TTL greeting                       # 剩余过期秒数（-1 永不过期，-2 不存在）
INFO memory | grep -E "used_memory_human|maxmemory"
INFO stats | grep -i keyspace
CONFIG GET maxmemory               # 查配置（CONFIG SET 可动态改）
```

### 持久化与内存配置（/etc/redis/redis.conf）

```conf
maxmemory 256mb
maxmemory-policy allkeys-lru
appendonly yes
save "900 1"                       # RDB：900 秒内 1 次修改触发快照
```

改完 `sudo systemctl restart redis-server`；数据落盘位置 `/var/lib/redis/`。

## 动手实验

- [ ] 五大类型各完成一次写入/读取/删除，用 TYPE 确认类型
- [ ] SET 一个 60 秒过期的 key，两次 TTL 观察倒数
- [ ] 开启 AOF，写入数据，`systemctl restart redis-server` 后数据仍在（持久化生效）
- [ ] 设 `maxmemory 10mb` 并灌数据，观察淘汰策略行为后改回
- [ ] `redis-cli INFO memory` 记录 used_memory_human

## 常见坑

- **KEYS * 在生产**：百万 key 时阻塞 Redis 数秒——用 SCAN 迭代；
- **没设 maxmemory**：内存吃满触发 OOM；缓存场景必须配淘汰策略；
- **RDB 写入失败**：磁盘满/权限会导致快照失败，monitor 里看 "BGSAVE" 报错；
- **缓存三兄弟**（雪崩/击穿/穿透）：面试题，原理见 [[小林coding_快照索引|《图解Redis》缓存问题篇]]。

## 自测

1. RDB 和 AOF 的取舍？生产怎么组合？
2. allkeys-lru 和 volatile-lru 区别？纯缓存选哪个？
3. 为什么生产禁用 KEYS *？
4. 如何验证持久化真的生效？

## 延伸

- 快照：[[小林coding_快照索引|《图解Redis》持久化/缓存问题]]
- 下一讲：串起 Nginx+MySQL+Redis+应用 → [[阶段4_服务实战/04_LNMP串联实战|LNMP串联实战]]

---
*相邻：[[阶段4_服务实战/02_MySQL安装与日常运维|← 上一讲]] ｜ [[阶段4_服务实战/04_LNMP串联实战|下一讲 →]]*
