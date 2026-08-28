# Prisma 综合实战：博客 API

## 1. 项目目标

用 TypeScript 构建一个博客 API，覆盖 Schema 建模、迁移、类型安全查询、事务和工程化部署。项目按阶段扩展，最终交付生产可用的 API 服务。

```text
初始化 -> 数据建模 -> 迁移与种子 -> 查询 API
    -> 事务与鉴权 -> 优化与部署
```

## 2. 需求

- 用户：注册登录、角色（USER/ADMIN）、资料。
- 文章：标题、内容、状态、浏览量、分类。
- 分类：文章多对多关联。
- 评论：文章一对多，用户一对多。
- 操作：文章 CRUD、评论、发布流程、统计。

## 3. 技术选择

| 技术 | 用途 |
|---|---|
| TypeScript | 语言 |
| Prisma 7 | ORM |
| PostgreSQL 17 | 数据库 |
| Express | HTTP 框架 |
| Prisma Studio | 数据查看 |

## 4. 初始化

```bash
npm install @prisma/client @prisma/adapter-pg pg
npm install -D prisma typescript @types/node @types/pg tsx

npx prisma init --datasource-provider postgresql
```

## 5. Schema 定义

```prisma
generator client {
  provider = "prisma-client"
  output   = "./generated"
}

datasource db {
  provider = "postgresql"
}

enum Role {
  USER
  ADMIN
}

enum PostStatus {
  DRAFT
  PUBLISHED
  ARCHIVED
}

model User {
  id        Int       @id @default(autoincrement())
  email     String    @unique
  name      String
  role      Role      @default(USER)
  posts     Post[]
  comments  Comment[]
  profile   Profile?
  createdAt DateTime  @default(now())
}

model Profile {
  id     Int    @id @default(autoincrement())
  bio    String?
  userId Int    @unique
  user   User   @relation(fields: [userId], references: [id])
}

model Post {
  id          Int        @id @default(autoincrement())
  title       String
  content     String
  status      PostStatus @default(DRAFT)
  views       Int        @default(0)
  authorId    Int
  author      User       @relation(fields: [authorId], references: [id])
  categories  Category[]
  comments    Comment[]
  createdAt   DateTime   @default(now())
  publishedAt DateTime?

  @@index([authorId, status])
  @@index([status, publishedAt])
}

model Category {
  id    Int    @id @default(autoincrement())
  name  String @unique
  posts Post[]
}

model Comment {
  id        Int     @id @default(autoincrement())
  content   String
  postId    Int
  post      Post    @relation(fields: [postId], references: [id])
  authorId  Int
  author    User    @relation(fields: [authorId], references: [id])
  createdAt DateTime @default(now())
}
```

## 6. 迁移与种子

```bash
npx prisma migrate dev --name init
npx prisma db seed
```

```ts
// prisma/seed.ts
import { PrismaClient } from '../src/generated/client';
import { PrismaPg } from '@prisma/adapter-pg';

const prisma = new PrismaClient({
  adapter: new PrismaPg({ connectionString: process.env.DATABASE_URL }),
});

async function main() {
  const admin = await prisma.user.upsert({
    where: { email: 'admin@example.com' },
    update: {},
    create: { email: 'admin@example.com', name: 'Admin', role: 'ADMIN' },
  });

  const js = await prisma.category.upsert({
    where: { name: 'TypeScript' },
    update: {},
    create: { name: 'TypeScript' },
  });

  await prisma.post.create({
    data: {
      title: 'Prisma 入门',
      content: '正文...',
      status: 'PUBLISHED',
      publishedAt: new Date(),
      authorId: admin.id,
      categories: { connect: [{ id: js.id }] },
    },
  });
}

main()
  .catch((e) => { throw e })
  .finally(() => prisma.$disconnect());
```

## 7. 查询 API

### 6.1 文章列表

```ts
// routes/posts.ts
import { Router } from 'express';
import { prisma } from '../db';

const router = Router();

// 已发布文章列表，带作者与分类，keyset 分页
router.get('/posts', async (req, res) => {
  const cursor = req.query.cursor ? Number(req.query.cursor) : undefined;

  const posts = await prisma.post.findMany({
    where: { status: 'PUBLISHED' },
    include: {
      author: { select: { id: true, name: true } },
      categories: { select: { id: true, name: true } },
      _count: { select: { comments: true } },
    },
    orderBy: { publishedAt: 'desc' },
    take: 20,
    ...(cursor ? { cursor: { id: cursor }, skip: 1 } : {}),
  });

  res.json({
    data: posts,
    nextCursor: posts.length === 20 ? posts[posts.length - 1].id : null,
  });
});
```

### 6.2 文章详情

```ts
router.get('/posts/:id', async (req, res) => {
  const id = Number(req.params.id);

  // 浏览量原子递增
  const post = await prisma.post.update({
    where: { id },
    data: { views: { increment: 1 } },
    include: { author: true, categories: true, comments: { include: { author: true } } },
  });

  res.json(post);
});
```

### 6.3 创建文章

```ts
router.post('/posts', async (req, res) => {
  const { title, content, authorId, categoryIds } = req.body;

  const post = await prisma.post.create({
    data: {
      title,
      content,
      authorId,
      categories: categoryIds?.length ? { connect: categoryIds.map((id: number) => ({ id })) } : undefined,
    },
  });

  res.status(201).json(post);
});
```

### 6.4 发布流程（事务）

```ts
router.post('/posts/:id/publish', async (req, res) => {
  const id = Number(req.params.id);

  const post = await prisma.$transaction(async (tx) => {
    const existing = await tx.post.findUniqueOrThrow({
      where: { id },
      include: { _count: { select: { comments: true } } },
    });

    if (existing.status === 'ARCHIVED') {
      throw new Error('Archived posts cannot be published');
    }

    return tx.post.update({
      where: { id },
      data: { status: 'PUBLISHED', publishedAt: new Date() },
    });
  });

  res.json(post);
});
```

## 8. 统计

```ts
// 发布文章数与总浏览量
const stats = await prisma.post.aggregate({
  where: { status: 'PUBLISHED' },
  _count: { _all: true },
  _sum: { views: true },
});

// 每个作者的发布文章数
const byAuthor = await prisma.post.groupBy({
  by: ['authorId'],
  _count: { _all: true },
  where: { status: 'PUBLISHED' },
});
```

## 9. 实施顺序

1. 初始化项目与连接。
2. 定义 Schema 并完成首次迁移。
3. 编写 seed 数据。
4. 实现文章 CRUD 与列表分页。
5. 实现评论与发布事务。
6. 添加统计接口。
7. 配置日志、索引与部署脚本。

## 10. 验收清单

- [ ] Schema 模型完整，关系正确。
- [ ] 迁移可生成、可重置、可部署。
- [ ] 列表接口分页正确且无 N+1。
- [ ] 详情接口原子增加浏览量。
- [ ] 发布流程事务正确且能拦截非法状态。
- [ ] 统计接口返回正确聚合。
- [ ] 客户端单例与连接池配置正确。
- [ ] 生产部署流程可执行。

## 11. 按阶段学习卡片

| 阶段 | 项目增量 |
|---|---|
| 零 | 初始化、连接、最小客户端 |
| 一 | 完整数据模型与关系 |
| 二 | 迁移、种子与冲突处理 |
| 三 | 类型安全查询 API 与事务 |
| 四 | 框架集成、日志、部署 |
