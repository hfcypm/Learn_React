# 阶段二：数据建模与约束

**目标**：掌握表结构设计、约束、范式、索引、JSONB 和全文搜索，能够设计规范、完整、可查询的数据模型。

## 1. 约束

约束在数据库层保证数据合法性，不依赖应用代码。

```sql
CREATE TABLE products (
  id          BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  sku         TEXT NOT NULL UNIQUE,
  name        TEXT NOT NULL,
  price       NUMERIC(10,2) NOT NULL CHECK (price >= 0),
  stock       INTEGER NOT NULL DEFAULT 0 CHECK (stock >= 0),
  category_id BIGINT REFERENCES categories(id),
  deleted_at  TIMESTAMPTZ
);
```

| 约束 | 作用 |
|---|---|
| `PRIMARY KEY` | 主键，非空且唯一 |
| `UNIQUE` | 唯一，允许一个 NULL |
| `NOT NULL` | 非空 |
| `CHECK` | 自定义条件 |
| `REFERENCES` / `FOREIGN KEY` | 引用完整性 |
| `DEFAULT` | 默认值 |

## 2. 外键与关系

### 2.1 一对一

```sql
CREATE TABLE user_profiles (
  user_id BIGINT PRIMARY KEY REFERENCES users(id),
  bio     TEXT
);
```

### 2.2 一对多

```sql
CREATE TABLE orders (
  id      BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  user_id BIGINT NOT NULL REFERENCES users(id)
);
```

### 2.3 多对多（关联表）

```sql
CREATE TABLE order_items (
  order_id   BIGINT NOT NULL REFERENCES orders(id),
  product_id BIGINT NOT NULL REFERENCES products(id),
  quantity   INTEGER NOT NULL CHECK (quantity > 0),
  PRIMARY KEY (order_id, product_id)
);
```

## 3. 范式

- 第一范式：列不可再分。
- 第二范式：非主键列完全依赖主键，没有部分依赖。
- 第三范式：非主键列不依赖其他非主键列（消除传递依赖）。

反范式：在需要读性能时，适度冗余（如预聚合统计、缓存列），配合应用维护一致性。

## 4. 索引

索引加速查询，但增加写入成本和存储。

### 4.1 常用索引

```sql
-- B-tree：默认，适合等值和范围
CREATE INDEX idx_orders_user_id ON orders (user_id);
CREATE INDEX idx_orders_created_at ON orders (created_at);

-- 唯一索引
CREATE UNIQUE INDEX idx_users_email ON users (email);

-- 部分索引：只索引满足条件的行
CREATE INDEX idx_orders_active ON orders (status) WHERE status = 'pending';

-- 表达式索引：函数结果
CREATE INDEX idx_users_email_lower ON users (lower(email));

-- 组合索引：列顺序影响查询
CREATE INDEX idx_orders_user_created ON orders (user_id, created_at);
```

### 4.2 何时需要索引

- WHERE、JOIN、ORDER BY、GROUP BY 中频繁使用的列。
- 高基数列（很多不同值）适合索引。
- 低基数列（如布尔状态）单独索引收益低，考虑部分索引。

### 4.3 索引验证

```sql
EXPLAIN SELECT * FROM orders WHERE user_id = 100;
```

## 5. JSONB

JSONB 是二进制 JSON，支持索引和丰富操作符，适合存储结构灵活的数据。

```sql
CREATE TABLE events (
  id      BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  payload JSONB NOT NULL
);

-- 写入
INSERT INTO events (payload)
VALUES ('{"type": "click", "page": "/home", "meta": {"count": 3}}'::jsonb);

-- 查询键值
SELECT payload->>'type' AS type FROM events;

-- 包含查询（用 GIN 索引加速）
CREATE INDEX idx_events_payload ON events USING GIN (payload jsonb_path_ops);

SELECT * FROM events WHERE payload @> '{"type": "click"}';
```

使用要点：

- 用 `->>` 取文本值，用 `->` 取 JSON 值。
- `@>` 判断包含关系，可走 GIN 索引。
- 结构固定的数据用普通列，结构多变的数据用 JSONB。

## 6. 全文搜索

```sql
-- 创建 tsvector 生成列并加 GIN 索引
CREATE TABLE articles (
  id      BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  title   TEXT NOT NULL,
  body    TEXT NOT NULL,
  document tsvector GENERATED ALWAYS AS
    (to_tsvector('simple', title || ' ' || body)) STORED
);

CREATE INDEX idx_articles_document ON articles USING GIN (document);

-- 查询
SELECT title FROM articles
WHERE document @@ to_tsquery('simple', 'postgresql & index');
```

## 7. 设计实践

- 命名：表名复数或单数一致，列名小写下划线。
- 时间统一 `TIMESTAMPTZ`。
- 金额用 `NUMERIC(p,s)`，不用浮点数。
- 软删除加 `deleted_at`，配合部分索引过滤。
- 为唯一约束（邮箱、SKU）建唯一索引并处理冲突。

## 8. 动手任务

1. 为电商数据库建立商品分类、商品、订单、订单项、用户资料表。
2. 为所有表定义主键、外键、NOT NULL 与 CHECK 约束。
3. 为邮箱、SKU 建立唯一索引，为订单用户与时间建立组合索引。
4. 用 EXPLAIN 验证新增索引被查询使用。
5. 用 JSONB 列存储订单的扩展属性，验证 `@>` 查询。

## 阶段二验收

- 能设计规范且约束完整的数据模型。
- 能区分三种表关系并用外键实现。
- 能解释范式并说明反范式场景。
- 能合理选择 B-tree、部分索引、GIN 索引。

## 常见排错

| 现象 | 排查方向 |
|---|---|
| 外键插入失败 | 引用行不存在或类型不匹配 |
| 唯一约束冲突 | `ON CONFLICT` 处理或先查后写 |
| 索引未生效 | 检查列顺序、表达式匹配、类型转换 |
| JSONB 查询慢 | 确认走 GIN 索引而非 `?` 路径操作 |
| 全文搜索不匹配 | 确认 tsvector 与查询用同一配置 |

## 进入下一阶段的条件

你能设计规范的数据模型并配置合理索引。此时进入 [阶段三：事务、锁与性能优化](./stage-3-performance.md)。
