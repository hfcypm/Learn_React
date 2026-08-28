# 阶段四：生产运维

**目标**：掌握备份与恢复、复制、监控、连接池和扩展策略，能够安全地运营生产数据库。

## 1. 备份与恢复

### 1.1 逻辑备份

```bash
# 备份单个数据库
pg_dump mydb > mydb.sql

# 备份全部数据库
pg_dumpall > all.sql

# 恢复
createdb mydb_new
psql -d mydb_new -f mydb.sql
```

### 1.2 物理备份（连续归档）

逻辑备份有备份窗口，物理备份结合 WAL 归档可以实现时间点恢复。

```bash
# 基础备份
pg_basebackup -D /backup/base -Fp -Xs -P

# WAL 归档（postgresql.conf）
wal_level = replica
archive_mode = on
archive_command = 'cp %p /backup/wal/%f'
```

恢复流程：还原基础备份，重放归档 WAL 到目标时间点，配置 `recovery_target_time` 即可实现 PITR。

## 2. 复制与高可用

### 2.1 流复制

主库开启归档，从库通过 `pg_basebackup` 建立并持续追赶主库 WAL，作为只读副本。

```bash
# 主库
wal_level = replica
max_wal_senders = 10

# 从库
primary_conninfo = 'host=primary port=5432 user=replica'
```

### 2.2 高可用方案

| 方案 | 特点 |
|---|---|
| 流复制 + 自动故障转移 | 用 Patroni / repmgr 管理 |
| 读写分离 | 从库承担读流量 |
| 备份 + 快速恢复 | 简单可靠，RTO 较长 |

## 3. 监控

### 3.1 系统视图

`pg_stat_statements` 是慢查询统计扩展，需要先启用：

```sql
-- 需要超级用户权限，一次即可
CREATE EXTENSION IF NOT EXISTS pg_stat_statements;
```

启用后，统计从下一次查询开始累积。核心查询：

```sql
-- 连接数
SELECT count(*) FROM pg_stat_activity;

-- 慢查询（统计最近执行时间，需先启用 pg_stat_statements）
SELECT query, calls, mean_exec_time, max_exec_time
FROM pg_stat_statements
ORDER BY mean_exec_time DESC LIMIT 10;

-- 表膨胀
SELECT relname, n_dead_tup, n_live_tup
FROM pg_stat_user_tables
WHERE n_dead_tup > 1000;
```

### 3.2 关键指标

- 连接数（达到 max_connections 会拒绝新连接）。
- 慢查询数量与分布。
- 死元组数量（需 VACUUM）。
- 复制延迟。
- 缓存命中率（应接近 99%）。

## 4. 连接与池化

每个连接都有内存开销，应用应使用连接池：

- PgBouncer：事务级连接池，适合短连接应用。
- 应用内连接池：Node.js 用 `pg.Pool`，Go 用 `database/sql` 连接池。

```js
const { Pool } = require('pg');
const pool = new Pool({ connectionString: process.env.DATABASE_URL });
```

## 5. 维护任务

### 5.1 VACUUM

清理死元组，防止表膨胀：

```bash
# 手动清理
VACUUM mytable;

# 分析统计
ANALYZE mytable;

# 组合
VACUUM ANALYZE mytable;
```

PostgreSQL 自动 VACUUM 通常足够，大表维护窗口内可手动执行。

### 5.2 索引维护

```sql
REINDEX INDEX idx_orders_user_id;
```

## 6. 扩展

### 6.1 读写分离

主库处理写，从库处理读，应用层路由。

### 6.2 分区表

大表按时间或键分区，查询只扫描相关分区：

```sql
CREATE TABLE orders (
  id BIGINT,
  created_at TIMESTAMPTZ NOT NULL
) PARTITION BY RANGE (created_at);

CREATE TABLE orders_2026 PARTITION OF orders
FOR VALUES FROM ('2026-01-01') TO ('2027-01-01');
```

### 6.3 缓存层

热点读用 Redis 缓存，数据库作为最终一致的数据源。

### 6.4 上云

托管服务（RDS、Cloud SQL、Supabase）提供自动备份、监控和扩容。

## 7. 部署检查清单

- 修改默认密码，限制访问来源。
- 开启备份与 WAL 归档，定期演练恢复。
- 配置监控与告警。
- 限制最大连接数，应用侧配置连接池。
- 生产环境避免手动 DDL 与大表全表更新。
- 变更前备份，变更可回滚。

## 8. 动手任务

1. 编写 `pg_dump` 备份脚本，并在一张空库上完成恢复演练。
2. 配置 WAL 归档，用 `pg_basebackup` 建立一台从库。
3. 从从库执行只读查询，验证流复制生效。
4. 用 `pg_stat_statements` 找出最慢的十条查询。
5. 观察死元组数量并执行 `VACUUM ANALYZE`，对比前后数据。
6. 编写一份包含备份、恢复演练、监控与告警的部署检查清单。

## 阶段四验收

- 能用 pg_dump 与 WAL 归档完成备份和恢复。
- 能配置流复制并理解高可用方案。
- 能使用系统视图监控连接、慢查询和膨胀。
- 能配置连接池并理解扩展策略。

## 常见排错

| 现象 | 排查方向 |
|---|---|
| 连接数耗尽 | 检查连接池与 max_connections |
| 复制延迟增长 | 检查从库负载与 WAL 发送 |
| 表膨胀严重 | VACUUM 检查与 autovacuum 配置 |
| 备份恢复失败 | 校验备份完整性，注意权限与路径 |
| 时间点恢复偏差 | 确认 recovery_target_time 与 WAL 覆盖 |

## 进入下一阶段的条件

你能安全地备份、监控和扩展生产数据库。此时进入 [综合实战：电商订单数据库](./project-practice.md)。
