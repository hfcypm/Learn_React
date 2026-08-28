# PostgreSQL 综合实战：电商订单数据库

## 1. 项目目标

从零设计并维护一个电商订单数据库，覆盖数据建模、查询统计、事务并发、索引优化和备份运维。项目按阶段扩展，最终交付一个可运行的完整数据库方案。

```text
建库建表 -> 多表查询与统计 -> 规范化与索引
    -> 事务与并发 -> 性能优化 -> 备份与监控
```

## 2. 需求

- 用户：注册、资料、收货地址。
- 商品：分类、SKU、价格、库存。
- 订单：用户、状态、总额、创建时间。
- 订单项：商品、数量、单价。
- 支付记录：订单、金额、方式、状态。

## 3. 技术选择

| 技术 | 用途 |
|---|---|
| PostgreSQL 17 | 数据库 |
| psql | 交互查询 |
| pg_dump | 备份 |
| EXPLAIN ANALYZE | 性能分析 |

## 4. 建库与建表

```sql
CREATE DATABASE shop;

\c shop
```

```sql
CREATE TABLE users (
  id         BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  email      TEXT NOT NULL UNIQUE,
  name       TEXT NOT NULL,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE TABLE addresses (
  id      BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  user_id BIGINT NOT NULL REFERENCES users(id),
  city    TEXT NOT NULL,
  detail  TEXT NOT NULL
);

CREATE TABLE categories (
  id   BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  name TEXT NOT NULL UNIQUE
);

CREATE TABLE products (
  id          BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  category_id BIGINT NOT NULL REFERENCES categories(id),
  sku         TEXT NOT NULL UNIQUE,
  name        TEXT NOT NULL,
  price       NUMERIC(10,2) NOT NULL CHECK (price >= 0),
  stock       INTEGER NOT NULL DEFAULT 0 CHECK (stock >= 0)
);

CREATE TABLE orders (
  id         BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  user_id    BIGINT NOT NULL REFERENCES users(id),
  address_id BIGINT REFERENCES addresses(id),
  status     TEXT NOT NULL DEFAULT 'pending'
             CHECK (status IN ('pending', 'paid', 'shipped', 'completed', 'cancelled')),
  total      NUMERIC(12,2) NOT NULL DEFAULT 0,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE TABLE order_items (
  order_id   BIGINT NOT NULL REFERENCES orders(id),
  product_id BIGINT NOT NULL REFERENCES products(id),
  quantity   INTEGER NOT NULL CHECK (quantity > 0),
  unit_price NUMERIC(10,2) NOT NULL,
  PRIMARY KEY (order_id, product_id)
);

CREATE TABLE payments (
  id         BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  order_id   BIGINT NOT NULL REFERENCES orders(id),
  amount     NUMERIC(12,2) NOT NULL CHECK (amount >= 0),
  method     TEXT NOT NULL,
  status     TEXT NOT NULL DEFAULT 'pending'
             CHECK (status IN ('pending', 'success', 'failed')),
  paid_at    TIMESTAMPTZ
);
```

## 5. 查询统计

```sql
-- 各状态订单数
SELECT status, count(*) FROM orders GROUP BY status;

-- 用户消费 TOP 10
SELECT u.name, sum(o.total) AS spend
FROM users u
JOIN orders o ON o.user_id = u.id
WHERE o.status IN ('paid', 'shipped', 'completed')
GROUP BY u.id, u.name
ORDER BY spend DESC
LIMIT 10;

-- 商品销量
SELECT p.name, sum(oi.quantity) AS sold
FROM order_items oi
JOIN products p ON p.id = oi.product_id
GROUP BY p.id, p.name
ORDER BY sold DESC
LIMIT 10;

-- 每月收入
SELECT date_trunc('month', created_at) AS month, sum(total) AS revenue
FROM orders
WHERE status IN ('paid', 'shipped', 'completed')
GROUP BY month
ORDER BY month;

-- 每用户最新订单
SELECT DISTINCT ON (user_id) user_id, id, created_at
FROM orders
ORDER BY user_id, created_at DESC;
```

## 6. 索引设计

```sql
-- 订单按用户查询
CREATE INDEX idx_orders_user_id ON orders (user_id);
CREATE INDEX idx_orders_user_created ON orders (user_id, created_at);

-- 状态过滤的部分索引
CREATE INDEX idx_orders_pending ON orders (status) WHERE status = 'pending';

-- 订单项
CREATE INDEX idx_order_items_product ON order_items (product_id);

-- 支付
CREATE INDEX idx_payments_order ON payments (order_id);
```

验证：

```sql
EXPLAIN ANALYZE
SELECT * FROM orders WHERE user_id = 1 ORDER BY created_at DESC LIMIT 20;
```

## 7. 事务与并发

### 7.1 下订单流程

```sql
BEGIN;

-- 锁定商品行，防止超卖
SELECT id, stock FROM products WHERE id = 1 FOR UPDATE;
UPDATE products SET stock = stock - 1 WHERE id = 1 AND stock > 0;

INSERT INTO orders (user_id, status, total)
VALUES (1, 'pending', 99.00) RETURNING id;

INSERT INTO order_items (order_id, product_id, quantity, unit_price)
VALUES (1, 1, 1, 99.00);

COMMIT;
```

### 7.2 支付流程

```sql
BEGIN;

UPDATE payments SET status = 'success', paid_at = now() WHERE order_id = 1 AND status = 'pending';
UPDATE orders SET status = 'paid' WHERE id = 1 AND status = 'pending';

COMMIT;
```

`WHERE status = 'pending'` 保证支付幂等，避免重复扣款。

### 7.3 隔离级别测试

```sql
BEGIN ISOLATION LEVEL REPEATABLE READ;
SELECT sum(total) FROM orders;
COMMIT;
```

## 8. 性能优化

- 用 `EXPLAIN ANALYZE` 检查慢查询是否走索引。
- 分页用 keyset 替代大 OFFSET：

```sql
SELECT * FROM orders
WHERE (created_at, id) < ('2026-08-01', 5000)
ORDER BY created_at DESC, id DESC
LIMIT 20;
```

- 定期 `ANALYZE`，让统计保持新鲜。

## 9. 备份与监控

```bash
# 备份
pg_dump shop > shop.sql

# 恢复演练
createdb shop_restore
psql -d shop_restore -f shop.sql
```

```sql
-- 启用慢查询统计（一次即可）
CREATE EXTENSION IF NOT EXISTS pg_stat_statements;

-- 慢查询
SELECT query, calls, mean_exec_time
FROM pg_stat_statements
ORDER BY mean_exec_time DESC LIMIT 10;

-- 死元组
SELECT relname, n_dead_tup
FROM pg_stat_user_tables
WHERE n_dead_tup > 1000;
```

## 10. 实施顺序

1. 建库建表，插入示例数据。
2. 完成查询统计与报表 SQL。
3. 设计并验证索引。
4. 编写下单与支付事务。
5. 压测并优化慢查询。
6. 完成备份脚本与监控查询。

## 11. 验收清单

- [ ] 所有约束生效，能拦截非法数据。
- [ ] 报表查询正确返回。
- [ ] 慢查询走索引并可用 EXPLAIN 验证。
- [ ] 并发扣库存不超卖。
- [ ] 支付流程幂等，重复执行不重复扣款。
- [ ] 备份与恢复演练成功。
- [ ] 有慢查询与死元组监控。

## 12. 按阶段学习卡片

| 阶段 | 项目增量 |
|---|---|
| 零 | 建库建表，连接操作 |
| 一 | 多表查询与统计报表 |
| 二 | 约束、索引与数据建模完善 |
| 三 | 事务、并发与性能优化 |
| 四 | 备份、监控与扩展 |
