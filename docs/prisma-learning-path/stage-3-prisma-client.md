# 阶段三：Prisma Client 查询

**目标**：掌握 Prisma Client 的 CRUD、关系查询、筛选、排序、分页、聚合和事务，写出类型安全且高效的查询。

## 1. 基础 CRUD

```ts
// 创建
const user = await prisma.user.create({
  data: { email: 'a@example.com', name: 'Alice' },
});

// 批量创建
await prisma.user.createMany({
  data: [
    { email: 'b@example.com', name: 'Bob' },
    { email: 'c@example.com', name: 'Cara' },
  ],
});

// 查询
const user = await prisma.user.findUnique({ where: { id: 1 } });
const users = await prisma.user.findMany({
  where: { name: { contains: 'A' } },
  orderBy: { createdAt: 'desc' },
  take: 10,
  skip: 20,
});

// 更新
await prisma.user.update({ where: { id: 1 }, data: { name: 'Alice2' } });

// 删除
await prisma.user.delete({ where: { id: 1 } });
```

## 2. 筛选条件

```ts
// 逻辑组合
await prisma.post.findMany({
  where: {
    AND: [{ published: true }, { authorId: 1 }],
    OR: [{ title: { contains: 'prisma' } }, { content: { contains: 'prisma' } }],
    NOT: { status: 'archived' },
  },
});

// 比较与空值
await prisma.post.findMany({
  where: {
    views: { gt: 100, lte: 1000 },
    publishedAt: { not: null },
  },
});
```

常用过滤符：`equals`、`in`、`notIn`、`lt`、`lte`、`gt`、`gte`、`contains`、`startsWith`、`endsWith`、`not`、`isNull`。

## 3. 关系查询

### 3.1 包含关系

```ts
const user = await prisma.user.findUnique({
  where: { id: 1 },
  include: {
    posts: true,
    profile: true,
  },
});
```

### 3.2 选择字段

```ts
await prisma.user.findMany({
  select: {
    id: true,
    email: true,
    posts: { select: { title: true, publishedAt: true } },
  },
});
```

`select` 只取需要的字段，减少传输与组装成本。`include` 与 `select` 不能混用。

### 3.3 嵌套过滤与排序

```ts
const users = await prisma.user.findMany({
  include: {
    posts: {
      where: { published: true },
      orderBy: { publishedAt: 'desc' },
      take: 3,
    },
  },
});
```

### 3.4 N+1 问题

循环里逐条查关联是 N+1：

```ts
// 避免：循环内查库
for (const post of posts) {
  const author = await prisma.user.findUnique({ where: { id: post.authorId } });
}

// 推荐：一次查询带出关系
const postsWithAuthor = await prisma.post.findMany({
  include: { author: true },
});
```

## 4. 分页

```ts
// offset 分页
await prisma.post.findMany({ skip: 40, take: 20 });

// keyset 分页（游标），大表性能更好
await prisma.post.findMany({
  take: 20,
  cursor: { id: 100 },
  orderBy: { id: 'asc' },
});
```

## 5. 聚合

```ts
const agg = await prisma.post.aggregate({
  _count: { _all: true },
  _sum: { views: true },
  _avg: { views: true },
  _min: { views: true },
  _max: { views: true },
  where: { published: true },
});

// 分组统计
const byAuthor = await prisma.post.groupBy({
  by: ['authorId'],
  _count: { _all: true },
  having: { views: { _avg: { gt: 100 } } },
});
```

## 6. 事务

### 6.1 顺序操作事务

```ts
const [user, count] = await prisma.$transaction([
  prisma.user.create({ data: { email: 'd@example.com', name: 'Dave' } }),
  prisma.user.count(),
]);
```

### 6.2 交互式事务

需要读后写和条件判断时使用：

```ts
await prisma.$transaction(async (tx) => {
  const user = await tx.user.findUniqueOrThrow({ where: { id: 1 } });
  await tx.post.create({
    data: { title: 'New', authorId: user.id },
  });
  await tx.post.update({
    where: { id: 2 },
    data: { views: { increment: 1 } },
  });
});
```

### 6.3 事务选项

```ts
await prisma.$transaction(
  [prisma.user.create({ data: {...} })],
  { timeout: 10000, isolationLevel: 'RepeatableRead' }
);
```

## 7. 原子更新

用运算符避免「读改写」竞态：

```ts
await prisma.post.update({
  where: { id: 1 },
  data: { views: { increment: 1 } },
});
```

## 8. 动手任务

1. 为博客 API 完成文章 CRUD。
2. 按分类与关键词筛选文章，支持排序与 keyset 分页。
3. 查询作者及已发布文章，验证 include 避免 N+1。
4. 用聚合统计总浏览量，用 groupBy 统计每作者文章数。
5. 编写发布文章的交互式事务，验证中途失败全部回滚。

## 阶段三验收

- 能完成 CRUD 和批量操作。
- 能用筛选、排序、分页查询。
- 能用 include/select 加载关系并避免 N+1。
- 能用聚合和分组统计。
- 能使用顺序与交互式事务。

## 常见排错

| 现象 | 排查方向 |
|---|---|
| `include` 与 `select` 冲突 | 二选一，或全用 select |
| 找不到记录抛错 | 用 findUniqueOrThrow 或先判空 |
| N+1 查询缓慢 | 改用 include 一次加载 |
| 事务内超时 | 缩短事务，提高 timeout 或分拆 |
| 更新期望多行失败 | update 单行用 updateMany  |
| 类型错误 | 重新 `prisma generate` |

## 进入下一阶段的条件

你能用 Prisma Client 完成类型安全的查询与事务。此时进入 [阶段四：工程化与生产交付](./stage-4-production.md)。
