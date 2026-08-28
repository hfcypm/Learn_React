# 阶段一：核心 SQL 与查询

**目标**：掌握 SELECT 查询的完整能力，包括过滤、排序、分页、聚合、JOIN、子查询和窗口函数。

## 1. 基础查询

```sql
-- 全列
SELECT * FROM users;

-- 指定列与别名
SELECT id, email AS 邮箱 FROM users;

-- 过滤
SELECT * FROM users WHERE created_at >= '2026-01-01';

-- 排序
SELECT * FROM users ORDER BY created_at DESC;

-- 分页
SELECT * FROM users ORDER BY id LIMIT 20 OFFSET 40;
```

## 2. 过滤与运算符

```sql
-- 逻辑组合
SELECT * FROM orders
WHERE status = 'paid' AND total >= 100 OR status = 'refunded';

-- IN
SELECT * FROM users WHERE id IN (1, 2, 3);

-- 范围
SELECT * FROM orders WHERE total BETWEEN 100 AND 500;

-- 模糊匹配
SELECT * FROM users WHERE email ILIKE '%example.com';

-- 空值判断
SELECT * FROM users WHERE phone IS NULL;

-- 日期过滤
SELECT * FROM orders WHERE created_at >= now() - interval '7 days';
```

## 3. 聚合与分组

```sql
-- 常用聚合
SELECT
  count(*) AS total,
  sum(total) AS revenue,
  avg(total) AS avg_order,
  min(total) AS min_order,
  max(total) AS max_order
FROM orders;

-- 分组统计
SELECT status, count(*) AS cnt
FROM orders
GROUP BY status
ORDER BY cnt DESC;

-- 过滤分组
SELECT user_id, sum(total) AS spend
FROM orders
GROUP BY user_id
HAVING sum(total) > 1000;
```

`WHERE` 过滤行，`HAVING` 过滤分组后的结果。

## 4. JOIN

```sql
-- 内连接：只返回两边都匹配的行
SELECT o.id, u.name
FROM orders o
JOIN users u ON o.user_id = u.id;

-- 左连接：保留左表全部行
SELECT u.name, o.id AS order_id
FROM users u
LEFT JOIN orders o ON o.user_id = u.id;

-- 右连接：保留右表全部行
SELECT u.name, o.id
FROM orders o
RIGHT JOIN users u ON o.user_id = u.id;

-- 全连接：两边都保留
SELECT u.name, o.id
FROM users u
FULL JOIN orders o ON o.user_id = u.id;
```

自连接：同一张表连接自身，常用于层级数据：

```sql
SELECT e.name AS employee, m.name AS manager
FROM employees e
LEFT JOIN employees m ON e.manager_id = m.id;
```

## 5. 子查询

```sql
-- WHERE 子查询
SELECT name FROM users
WHERE id IN (SELECT user_id FROM orders WHERE total > 500);

-- FROM 子查询（派生表）
SELECT user_id, avg_total
FROM (
  SELECT user_id, avg(total) AS avg_total
  FROM orders
  GROUP BY user_id
) t
WHERE avg_total > 300;

-- EXISTS 相关子查询
SELECT name FROM users u
WHERE EXISTS (
  SELECT 1 FROM orders o WHERE o.user_id = u.id AND o.total > 1000
);
```

## 6. 窗口函数

窗口函数在行的分组（窗口）内计算，但不合并行，适合排名、累计和与上一行比较。

```sql
-- 排名
SELECT name, score,
  row_number() OVER (ORDER BY score DESC) AS row_num,
  rank()       OVER (ORDER BY score DESC) AS rank,
  dense_rank() OVER (ORDER BY score DESC) AS dense_rank
FROM students;

-- 分组内排名
SELECT user_id, total,
  rank() OVER (PARTITION BY user_id ORDER BY total DESC) AS order_rank
FROM orders;

-- 累计与滚动
SELECT created_at, total,
  sum(total) OVER (ORDER BY created_at) AS running_total,
  avg(total) OVER (ORDER BY created_at ROWS BETWEEN 6 PRECEDING AND CURRENT ROW) AS ma7
FROM orders;

-- 与上一行比较
SELECT created_at, total,
  lag(total) OVER (ORDER BY created_at) AS prev_total,
  total - lag(total) OVER (ORDER BY created_at) AS diff
FROM orders;
```

## 7. 常见模式

### 7.1 每类取 N 条（Top-N per Group）

```sql
SELECT user_id, total
FROM (
  SELECT user_id, total,
    row_number() OVER (PARTITION BY user_id ORDER BY total DESC) AS rn
  FROM orders
) t
WHERE rn <= 3;
```

### 7.2 去重取最新

```sql
SELECT DISTINCT ON (user_id) user_id, created_at, status
FROM order_events
ORDER BY user_id, created_at DESC;
```

### 7.3 去重计数

```sql
SELECT count(DISTINCT user_id) FROM orders;
```

## 8. 项目增量

在你的电商数据库中加入订单表，用 JOIN 关联用户，写聚合统计总收入、各状态订单数，用窗口函数给用户订单排名。

## 阶段一验收

- 能写过滤、排序、分页查询。
- 能正确使用 JOIN 的四种连接类型。
- 能用聚合与 GROUP BY、HAVING 统计。
- 能使用子查询和窗口函数解决排名、累计问题。

## 常见排错

| 现象 | 排查方向 |
|---|---|
| `column must appear in the GROUP BY clause` | 被 SELECT 的非聚合列加入 GROUP BY |
| JOIN 后行数变多 | 一对多 JOIN 产生重复，先确认基数 |
| 聚合与普通列混用 | 拆成子查询或用窗口函数 |
| 空值参与聚合 | count(列) 忽略 NULL，count(*) 统计所有行 |
| 分页偏移过大性能差 | 用 keyset 分页替代 OFFSET |

## 进入下一阶段的条件

你能用 SQL 完成多表关联和统计查询。此时进入 [阶段二：数据建模与约束](./stage-2-data-modeling.md)。
