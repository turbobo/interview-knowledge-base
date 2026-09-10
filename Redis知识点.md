# Redis 知识全景

> 来源：[小林 coding — Redis 系列](https://xiaolincoding.com/redis/)
> 配套可视化图：[Redis全景图.html](./Redis全景图.html)（浏览器打开）

---

## 一、基础篇

### Redis 是什么

- 开源内存键值数据库，更准确说是"数据结构服务器"
- 单线程命令执行（6.0+ I/O 多线程，命令仍单线程）
- 常作为 MySQL/PostgreSQL 前的缓存层

### Redis vs Memcached

| 维度 | Redis | Memcached |
|------|-------|-----------|
| 数据类型 | 9 种丰富类型 | 仅 String |
| 持久化 | RDB / AOF / 混合 | 不支持 |
| 集群 | 原生 Cluster | 客户端分片 |
| 线程模型 | 单线程命令 + 多线程 I/O | 多线程 |

### 为什么快

- 纯内存操作，读写纳秒级
- 单线程避免锁竞争和上下文切换
- 高效数据结构（SDS、跳表、哈希表）
- I/O 多路复用

### 线程模型

- **命令执行**：单线程（避免竞争/死锁/切换）
- **后台 BIO 线程**：close-file、AOF fsync、lazy-free
- **6.0+ I/O 线程池**：`io-threads N` 开启 N-1 个读写线程

---

## 二、数据类型篇（9 种）

| 类型 | 底层结构 | 典型场景 |
|------|----------|----------|
| **String** | SDS / int / embstr | 缓存、计数器、分布式锁、Session |
| **List** | quicklist（7.0 listpack） | 消息队列（LPUSH+RPOP） |
| **Hash** | 压缩列表/哈希表（7.0 listpack） | 缓存对象、购物车 |
| **Set** | intset / 哈希表 | 去重、交集（共同关注）、抽奖 |
| **Zset** | 跳表 + 哈希表（7.0 listpack） | 排行榜、延迟队列 |
| **BitMap** | String 类型 | 签到、登录状态、BITOP |
| **HyperLogLog** | 基数算法 | UV 统计（误差 0.81%，12KB/键） |
| **GEO** | ZSet + GeoHash | 附近搜索、叫车 |
| **Stream** | 消息队列专用 | 消费组、ACK、自动生成 ID |

### 底层数据结构演进

```
ziplist（连锁更新问题）→ quicklist（限制大小缓解）→ listpack（7.0 彻底解决，无 prevlen）
```

- **SDS**：O(1) 长度、二进制安全、自动扩容（<1MB 翻倍，≥1MB 加 1MB）
- **跳表**：平均 O(logN) 查询，概率分层（0.25），Zset 范围查询用跳表、精确查找用哈希表
- **哈希表**：渐进式 rehash，负载因子 ≥1 触发（未执行 bgsave 时）

---

## 三、持久化篇

### AOF 持久化

| 写回策略 | 机制 | 安全性 | 性能 |
|----------|------|--------|------|
| Always | 每次写后 fsync | 最高 | 最差 |
| Everysec | 主线程 write + 后台每秒 fsync | 最多丢 1 秒 | 折中 |
| No | 只写 page cache，OS 决定 | 最差 | 最好 |

**AOF 重写**：`bgrewriteaof` 子进程执行，fork + COW，压缩膨胀的 AOF 文件

**Redis 7.0 Multi Part AOF**：manifest（清单）+ base（RDB 全量）+ incr（AOF 增量），无需重写缓冲区

### RDB 快照

- **save**：主线程同步执行（阻塞）
- **bgsave**：fork 子进程（非阻塞，推荐）
- **自动触发**：`save 900 1` / `save 300 10` / `save 60 10000`
- **COW 写时复制**：fork 共享页表，写入时才复制物理页

### 混合持久化（4.0+）

AOF 重写时先写 RDB 全量，再追加 AOF 增量。重启优先用 AOF 恢复（RDB 前缀快速加载）。

### 大 Key 影响

- fork 大页表阻塞主线程（监控 `latest_fork_usec`）
- COW 复制大 Key 内存翻倍
- 优化：单实例 < 10GB、纯缓存关闭 AOF、删除用 `unlink`、关闭 Linux 透明大页

---

## 四、功能篇

### 过期删除策略

| 策略 | 说明 |
|------|------|
| 惰性删除 | 访问时检查是否过期 |
| 定期删除 | 每秒 10 次，随机抽查 20 个 key，过期 >25% 继续，单次上限 25ms |

Redis 同时使用惰性删除 + 定期删除。从节点不主动扫描过期，等主节点传播 DEL。

### 内存淘汰策略（8 种）

| 策略 | 说明 |
|------|------|
| noeviction | 默认，禁止写入 |
| allkeys-lru | 所有 key 最久未访问 |
| allkeys-lfu | 所有 key 最低频（4.0+） |
| volatile-lru | 有过期时间的 LRU |
| volatile-lfu | 有过期时间的 LFU |
| volatile-ttl | 过期时间最短的 |
| allkeys-random | 随机淘汰 |
| volatile-random | 随机淘汰有过期的 |

LRU 近似采样 5 个 key，存在缓存污染问题；LFU 用 logc 计数更精确。

### 分布式锁

**加锁**：`SET lock_key unique_value NX EX 30`

**释放锁**：Lua 脚本原子 GET+DEL（防误删他人锁）

**看门狗**：自动续期（Redisson 内置）

**Redlock**：至少 3 个独立节点，多数加锁成功且总耗时 < TTL。Martin Kleppmann 有时钟可靠性争议。

**常见陷阱**：看门狗无法续期宕机节点、Redlock 性能开销大、锁粒度需适中

---

## 五、高可用篇

### 主从复制

**全量同步三阶段**：
1. psync 协商（FULLRESYNC + runID + offset）
2. bgsave 生成 RDB 发送 + 写命令入 repl buffer
3. RDB 加载完成后发送缓冲写命令

**增量复制（2.8+）**：repl_backlog_buffer 循环缓冲，断线重连比较 offset

**关键特性**：异步复制、级联复制分散压力、心跳（主 10s / 从 1s）

### 哨兵 Sentinel

**三大职责**：监控（PING）、选主（自动故障转移）、通知（发布订阅）

**故障判定**：主观下线 → 投票达 quorum → 客观下线

**故障转移四步**：选新主（priority → offset → runID）→ 从节点复制新主 → 通知客户端 → 旧主降级

**部署**：至少 3 个奇数节点（3→quorum=2，5→quorum=3）

### Cluster 集群

**哈希槽**：16384 个 slot，`CRC16(key) % 16384`，slot → node 映射

**客户端路由**：Smart Client 本地计算、MOVED 永久重定向、ASK 临时重定向

**Gossip 协议**：ping/pong/meet/fail，端口 = 业务端口 + 10000，主节点 ≤ 1000

**故障转移**：pfail 主观下线（15s）→ fail 超半数确认 → 从节点选举新主

**部署**：最小 3 主 3 从，偏向 AP 可能丢数据

---

## 六、缓存篇

### 缓存雪崩

- **原因**：大量 key 同时过期 / Redis 宕机
- **方案**：过期加随机值、互斥锁重建、后台更新+预热、熔断限流、高可用集群

### 缓存击穿

- **原因**：热 key 过期
- **方案**：互斥锁、热 key 不过期、后台异步刷新

### 缓存穿透

- **原因**：缓存和 DB 都不存在（恶意攻击 / 误操作）
- **方案**：API 参数校验、缓存空值、布隆过滤器

### 数据库与缓存一致性

**Cache Aside 策略（推荐）**：
- 读：缓存命中 → 返回；未命中 → 读 DB → 写缓存
- 写：**更新 DB → 删除缓存**（不是更新缓存）

**先更新 DB 再删缓存 vs 先删缓存再更新 DB**：

| 方案 | 问题 |
|------|------|
| 先删缓存再更新 DB ❌ | 读+写竞态，旧值被重新加载到缓存 |
| 先更新 DB 再删缓存 ✅ | 理论有竞态但概率低（缓存写远快于 DB 写） |

**保证删除成功**：消息队列重试 / Canal 订阅 binlog 异步删除 / 缓存过期兜底

**为什么删除而非更新缓存**：删除更轻量、缓存常聚合多表重建成本高、Lazy Loading 更省资源

---

## 七、面试高频追问

**Q1：Redis 为什么用单线程？**
> CPU 不是瓶颈（内存操作极快），单线程避免锁竞争、上下文切换、死锁。6.0+ I/O 多线程仅加速网络读写，命令执行仍单线程。

**Q2：Redis 持久化选 AOF 还是 RDB？**
> 对数据安全要求高用 AOF（Everysec 最多丢 1 秒）；恢复速度优先用 RDB；最佳实践是混合持久化（RDB 全量 + AOF 增量）。

**Q3：缓存和数据库一致性怎么做？**
> Cache Aside：更新 DB 后删除缓存。用消息队列或 Canal 订阅 binlog 保证删除成功。给缓存设过期时间兜底。

**Q4：Redis Cluster 为什么用 16384 个槽？**
> bitmap 2KB（65536 则 8KB），Gossip 每秒交换更省带宽。官方建议 ≤ 1000 主节点，16384 足以均衡。2^14 可用位运算优化。

**Q5：分布式锁用 Redis 还是 ZooKeeper？**
> Redis 性能好但 AP 模型可能丢锁（Redlock 有时钟争议）；ZooKeeper CP 模型强一致但性能低。简单场景用 Redis + 看门狗，核心场景考虑 ZK + fencing token。
