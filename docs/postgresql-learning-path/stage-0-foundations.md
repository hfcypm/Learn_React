# 阶段零：数据库基础与工具链

**目标**：理解关系数据库和 PostgreSQL 的基本概念，安装并连接数据库，掌握 psql 的基础操作。

## 1. 关系模型

PostgreSQL 是对象关系型数据库。数据存储在表（table）中，表由行（row）和列（column）组成，列有类型约束，行通过主键唯一标识。

```sql
CREATE TABLE users (
  id         BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  email      TEXT NOT NULL UNIQUE,
  name       TEXT NOT NULL,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);
```

关键概念：

- 主键：唯一标识一行。
- 外键：引用另一张表的行，保证引用完整性。
- 索引：加速查询的数据结构。
- 约束：保证数据合法性的规则。

## 2. 安装与连接

### 2.1 安装

macOS 使用 Homebrew，Ubuntu 使用 apt，或使用 Docker：

```bash
docker run -d \
  --name pg17 \
  -e POSTGRES_PASSWORD=postgres \
  -p 5432:5432 \
  postgres:17
```

### 2.2 连接串

```text
postgresql://USER:PASSWORD@HOST:PORT/DBNAME
```

示例：

```bash
psql "postgresql://postgres:postgres@localhost:5432/postgres"
```

## 3. psql 基础

```bash
# 列出数据库
\l

# 切换数据库
\c mydb

# 列出表
\dt

# 查看表结构
\d users

# 执行 SQL 文件
psql -f schema.sql

# 退出
\q
```

## 4. 库与表管理

```sql
-- 创建数据库
CREATE DATABASE mydb;

-- 创建表
CREATE TABLE users (
  id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  email TEXT NOT NULL UNIQUE,
  name TEXT NOT NULL,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

-- 修改表
ALTER TABLE users ADD COLUMN age INT;
ALTER TABLE users DROP COLUMN age;

-- 删除表
DROP TABLE users;
```

## 5. 数据操作基础

```sql
-- 插入
INSERT INTO users (email, name) VALUES ('a@example.com', 'Alice');

-- 查询
SELECT id, email, name FROM users;

-- 更新
UPDATE users SET name = 'Alice2' WHERE id = 1;

-- 删除
DELETE FROM users WHERE id = 1;
```

## 6. 数据类型

| 类型 | 用途 | 示例 |
|---|---|---|
| `TEXT` | 变长字符串 | `'hello'` |
| `VARCHAR(n)` | 限长字符串 | `'ab'` |
| `INTEGER` / `BIGINT` | 整数 | `42` |
| `NUMERIC(p,s)` | 精确小数 | `19.99` |
| `REAL` / `DOUBLE PRECISION` | 浮点数 | `3.14` |
| `BOOLEAN` | 布尔 | `true` |
| `DATE` | 日期 | `2026-08-28` |
| `TIMESTAMPTZ` | 带时区时间 | `2026-08-28T10:00:00Z` |
| `UUID` | 通用唯一标识 | 需 `gen_random_uuid()` |
| `JSONB` | 二进制 JSON | `'{"a": 1}'::jsonb` |
| `ARRAY` | 数组 | `ARRAY[1,2,3]` |

## 7. 项目增量

在你的本地数据库创建 `users`、`products` 两张表，插入几条数据，并用 `\d` 检查表结构。

## 阶段零验收

- 能安装并连接 PostgreSQL。
- 能创建库、表并执行增删改查。
- 能读懂连接串并解释各部分含义。
- 能使用 psql 查看表结构和数据。

## 常见排错

| 现象 | 排查方向 |
|---|---|
| 连接被拒 | 端口、密码、pg_hba.conf 权限 |
| `psql: command not found` | 未安装 psql 客户端 |
| 表已存在报错 | 使用 `IF NOT EXISTS` 或先检查 `\dt` |
| 字符串比较大小写敏感 | 确认引号与 `ILIKE` 使用 |
| 时区显示偏差 | 检查 `TIMESTAMPTZ` 与会话时区 |

## 进入下一阶段的条件

你能够连接数据库并完成基础建表与增删改查。此时进入 [阶段一：核心 SQL 与查询](./stage-1-core-sql.md)。
