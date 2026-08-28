# 阶段三：事务、锁与性能优化

**目标**：掌握事务与隔离级别、锁机制、EXPLAIN 执行计划和索引优化，写出正确、高并发、高性能的查询。

## 1. 事务

事务是一组要么全部成功、要么全部回滚的操作。

```sql
BEGIN;
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;
COMMIT;
-- 出错时
ROLLBACK;
```

事务属性（ACID）：

- 原子性：全部或全不。
- 一致性：约束始终成立。
- 隔离性：并发事务相互隔离。
- 持久性：提交后数据不丢失。

## 2. 隔离级别

| 隔离级别 | 脏读 | 不可重复读 | 幻读 |
|---|---|---|---|
| Read Uncommitted | 可能 | 可能 | 可能 |
| Read Committed（默认） | 否 | 可能 | 可能 |
| Repeatable Read | 否 | 否 | 可能 |
| Serializable | 否 | 否 | 否 |

```sql
BEGIN ISOLATION LEVEL REPEATABLE READ;
SELECT sum(total) FROM orders;
COMMIT;
```

- Read Committed 默认，适合大多数场景。
- Repeatable Read 保证快照内数据一致，适合报表统计。
- Serializable 用于需要严格串行化的场景，冲突时可能报 `40001`。

## 3. 锁与并发控制

### 3.1 行锁

```sql
-- 显式行锁，防止并发修改
SELECT * FROM accounts WHERE id = 1 FOR UPDATE;
```

`FOR UPDATE` 用于「先查后改」的竞态场景，如扣库存、转账。

### 3.2 更新冲突

```sql
-- 原子更新避免竞态
UPDATE accounts SET balance = balance - 100 WHERE id = 1;

-- 原子计数
UPDATE products SET stock = stock - 1 WHERE id = 10 AND stock > 0;
```

### 3.3 死锁

两个事务互相等对方持有的锁时死锁，PostgreSQL 检测后回滚其中一个事务并报 `40P01`。减少死锁：统一锁顺序、缩短事务、避免长事务。

### 3.4 观察锁

```sql
SELECT pid, state, wait_event_type, query
FROM pg_stat_activity
WHERE state = 'active';
```

## 4. 索引优化

### 4.1 复合索引列顺序

把等值列放在前面，范围列放在后面：

```sql
-- 高效：user_id 等值，created_at 范围
CREATE INDEX idx_orders_user_created ON orders (user_id, created_at);
```

### 4.2 覆盖索引

索引包含查询需要的所有列，可避免回表：

```sql
CREATE INDEX idx_orders_user_total ON orders (user_id, total);
```

### 4.3 分析统计

```sql
ANALYZE orders;
```

让优化器获得最新统计信息，选择合适的执行计划。

## 5. EXPLAIN 执行计划

```sql
EXPLAIN SELECT * FROM orders WHERE user_id = 100;

EXPLAIN ANALYZE SELECT * FROM orders WHERE user_id = 100;

EXPLAIN (ANALYZE, BUFFERS) SELECT * FROM orders WHERE user_id = 100;
```

`EXPLAIN ANALYZE` 会真正执行查询，分析数据修改型语句时建议放在事务里并回滚：

```sql
BEGIN;
EXPLAIN ANALYZE UPDATE orders SET status = 'paid' WHERE id = 1;
ROLLBACK;
```

### 5.1 阅读执行计划

| 关键点 | 含义 |
|---|---|
| Seq Scan | 全表扫描，数据量大时需索引 |
| Index Scan / Index Only Scan | 走索引 |
| Bitmap Heap Scan | 位图扫描，用于多个过滤条件 |
| Nested Loop | 嵌套循环连接 |
| Hash Join | 哈希连接，适合大表等值连接 |
| Merge Join | 排序合并连接 |
| rows / actual time | 估计值与实际值，差异大需 ANALYZE |

### 5.2 优化流程

1. `EXPLAIN ANALYZE` 找到慢节点。
2. 确认 `Seq Scan` 的大表是否该走索引。
3. 检查索引是否被使用，是否需要复合或覆盖索引。
4. 检查过滤列类型是否匹配（隐式转换会阻止索引）。
5. `ANALYZE` 后重试。

## 6. 常见优化手段

- 用 `EXISTS` 替代 `IN` 子查询判断存在性（多数时候优化器等价）。
- 避免 `SELECT *`，只取需要的列。
- 分页用 keyset 替代大 OFFSET。
- 减少事务内不必要的行锁持有时间。
- 批量插入用多行 VALUES 或 `COPY`。
- 大量更新分批执行，避免长事务和膨胀。

## 7. 项目增量

为电商数据库的订单查询建立复合索引，用 EXPLAIN ANALYZE 对比加索引前后的执行计划，编写一个带事务的扣库存流程。

## 阶段三验收

- 能解释四种隔离级别及各自解决的问题。
- 能使用事务和 `FOR UPDATE` 处理并发场景。
- 能用 EXPLAIN ANALYZE 定位慢查询。
- 能根据执行计划选择并验证索引。

## 常见排错

| 现象 | 排查方向 |
|---|---|
| 查询阻塞 | `pg_stat_activity` 查锁等待 |
| 死锁 40P01 | 统一锁顺序、缩短事务 |
| 隔离级别冲突 40001 | 事务重试机制 |
| 索引没走 | 类型转换、列顺序、统计过旧 |
| 内存排序慢 | 优化 ORDER BY 与索引匹配 |

## 进入下一阶段的条件

你能用事务保证正确性并用执行计划优化性能。此时进入 [阶段四：生产运维](./stage-4-operations.md)。
