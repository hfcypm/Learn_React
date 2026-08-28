# 阶段一：Schema 数据建模

**目标**：掌握 Prisma Schema 的模型、字段类型、约束、关系、枚举和原生类型，能够设计完整的数据模型。

## 1. 模型与字段

```prisma
model User {
  id        Int      @id @default(autoincrement())
  email     String   @unique
  name      String
  age       Int?
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
}
```

- `@id`：主键。
- `@default(autoincrement())`：自增。
- `@unique`：唯一约束。
- 类型后加 `?`：可空。
- `@updatedAt`：自动更新为当前时间。

## 2. 标量类型映射

| Prisma 类型 | PostgreSQL 类型 | 说明 |
|---|---|---|
| `Int` | `INTEGER` | 32 位整数 |
| `BigInt` | `BIGINT` | 64 位整数 |
| `Float` | `DOUBLE PRECISION` | 浮点 |
| `Decimal` | `NUMERIC` | 精确小数，适合金额 |
| `Boolean` | `BOOLEAN` | 布尔 |
| `String` | `TEXT` | 字符串 |
| `DateTime` | `TIMESTAMP(3)` | 时间 |
| `Json` | `JSONB` | JSON |
| `Bytes` | `BYTEA` | 字节 |

用原生类型微调映射：

```prisma
model Product {
  price Decimal @db.Decimal(10, 2)
  title String  @db.VarChar(100)
}
```

## 3. 关系

### 3.1 一对多

```prisma
model User {
  id     Int     @id @default(autoincrement())
  posts  Post[]
}

model Post {
  id       Int  @id @default(autoincrement())
  author   User @relation(fields: [authorId], references: [id])
  authorId Int
}
```

关系字段成对出现：父模型侧是列表 `Post[]`，子模型侧是标量 + 对象字段。

### 3.2 一对一

```prisma
model User {
  id      Int           @id @default(autoincrement())
  profile Profile?
}

model Profile {
  id     Int  @id @default(autoincrement())
  user   User @relation(fields: [userId], references: [id])
  userId Int  @unique
}
```

`@unique` 使关系成为一对一。

### 3.3 多对多

```prisma
model Post {
  id         Int      @id @default(autoincrement())
  categories Category[]
}

model Category {
  id    Int    @id @default(autoincrement())
  posts Post[]
}
```

隐式多对多由 Prisma 自动生成关联表。

### 3.4 自引用

```prisma
model User {
  id        Int     @id @default(autoincrement())
  name      String
  manager   User?   @relation("Employee", fields: [managerId], references: [id])
  managerId Int?
  reports   User[]  @relation("Employee")
}
```

## 4. 枚举

```prisma
enum Role {
  USER
  ADMIN
}

model User {
  id   Int  @id @default(autoincrement())
  role Role @default(USER)
}
```

## 5. 复合主键与唯一

```prisma
model OrderItem {
  orderId   Int
  productId Int
  quantity  Int

  @@id([orderId, productId])
  @@unique([orderId, productId])
}
```

## 6. 命名与约定

- 模型名用 PascalCase 单数（`User`）。
- 字段名用 camelCase。
- 关系字段语义化命名，需要歧义消除时给关系起名。
- 时间字段统一 `DateTime`，创建与更新时间用默认值自动维护。

## 7. 动手任务

1. 为博客项目定义 `User`、`Post`、`Category` 模型。
2. 加入 `User` 一对多 `Post`、多对多 `Category` 关系。
3. 添加 `Role` 枚举并设为默认值。
4. 添加一个自引用关系（如用户互相关注），用 `@relation` 命名。
5. 用 `@db.` 原生类型限定一个字段，完成迁移并检查生成 SQL。

## 阶段一验收

- 能用 Schema 定义模型和标量类型。
- 能实现一对一、一对多、多对多和自引用关系。
- 能使用枚举、唯一和复合主键。
- 能解释关系字段的成对结构。

## 常见排错

| 现象 | 排查方向 |
|---|---|
| 关系字段未成对 | 两侧都要声明对应字段 |
| 一对一提示歧义 | 子模型加 `@unique` 或命名关系 |
| 字段类型映射不符 | 用 `@db.` 原生类型指定 |
| 迁移报错缺失关系 | 确认 `@relation` 指定外键字段 |
| 枚举无法用 | 确认 Prisma 版本支持枚举 |

## 进入下一阶段的条件

你能用 Schema 设计完整的数据模型。此时进入 [阶段二：迁移工作流](./stage-2-migrations.md)。
