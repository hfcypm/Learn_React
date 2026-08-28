# 阶段四：工程化与生产交付

**目标**：掌握 Prisma 的工程化实践，包括类型安全、与 Web 框架集成、连接池、日志、监控、缓存和部署。

## 1. 客户端单例

避免每次请求创建新客户端，防止连接耗尽：

```ts
// src/db.ts
import { PrismaClient } from './generated/client';

const globalForPrisma = globalThis as unknown as { prisma?: PrismaClient };

export const prisma =
  globalForPrisma.prisma ??
  new PrismaClient({
    log: process.env.NODE_ENV === 'development' ? ['query', 'warn', 'error'] : ['error'],
  });

if (process.env.NODE_ENV !== 'production') {
  globalForPrisma.prisma = prisma;
}
```

## 2. 类型安全

### 2.1 从模型生成类型

```ts
import type { User, Post } from './generated/client';
```

### 2.2 请求 DTO 类型

用 Prisma 的输入类型约束接口入参：

```ts
import type { Prisma } from './generated/client';

type CreatePostInput = Prisma.PostCreateInput;

function createPost(data: CreatePostInput) {
  return prisma.post.create({ data });
}
```

## 3. 与 Web 框架集成

### 3.1 Express

```ts
import express from 'express';
import { prisma } from './db';

const app = express();
app.use(express.json());

app.get('/posts', async (req, res) => {
  const posts = await prisma.post.findMany({
    include: { author: true },
  });
  res.json(posts);
});

app.post('/posts', async (req, res) => {
  const post = await prisma.post.create({
    data: {
      title: req.body.title,
      content: req.body.content,
      authorId: req.body.authorId,
    },
  });
  res.status(201).json(post);
});

app.listen(3000);
```

### 3.2 NestJS

使用 `PrismaService` 封装客户端并注入：

```ts
import { Injectable, OnModuleInit, OnModuleDestroy } from '@nestjs/common';
import { PrismaClient } from '../generated/client';

@Injectable()
export class PrismaService
  extends PrismaClient
  implements OnModuleInit, OnModuleDestroy
{
  async onModuleInit() {
    await this.$connect();
  }

  async onModuleDestroy() {
    await this.$disconnect();
  }
}
```

## 4. 连接管理

### 4.1 连接池

数据库直接连接在连接数上受限。Prisma 支持连接池扩展：

```bash
npm install @prisma/extension-pool
```

```ts
import { PrismaPool } from '@prisma/extension-pool';

const prisma = new PrismaClient().$extends(
  PrismaPool({ poolSize: 5, maxQueue: 100 })
);
```

### 4.2 超时与重试

- 连接池排队超时：池满时请求等待，超过 `maxQueue` 返回错误。
- 事务重试：`SerializationFailure`（`P2034`）时重试事务。

## 5. 日志与监控

```ts
const prisma = new PrismaClient({
  log: [
    { emit: 'event', level: 'query' },
    { emit: 'stdout', level: 'error' },
  ],
});

prisma.$on('query', (e) => {
  console.log(`Query: ${e.query}`);
  console.log(`Duration: ${e.duration}ms`);
});
```

监控要点：

- 慢查询与执行时间。
- 连接池排队与超时。
- 事务失败与重试率。
- 数据库层配合 `pg_stat_statements` 排查慢 SQL。

## 6. 查询性能

- 用 `select` 只取需要的字段。
- 关系查询一次 `include`，避免 N+1。
- 大分页用 keyset 游标。
- 复杂聚合尽量在数据库完成，不搬数据到应用层。
- 常用条件建索引（Prisma 支持 `@@index`）：

```prisma
model Post {
  id       Int    @id @default(autoincrement())
  authorId Int
  status   String @default("draft")

  @@index([authorId, status])
}
```

## 7. 缓存

热点查询可加 Redis 缓存，注意缓存失效策略：

```ts
const key = `post:${id}`;
const cached = await redis.get(key);
if (cached) return JSON.parse(cached);

const post = await prisma.post.findUnique({ where: { id } });
await redis.set(key, JSON.stringify(post), 'EX', 60);
return post;
```

写操作时使对应缓存失效。

## 8. 部署

### 8.1 部署流程

```text
npm ci
npx prisma generate
npx prisma migrate deploy
npm run build
npm run start
```

### 8.2 CI 检查

- `prisma validate`：校验 Schema。
- `prisma migrate status`：确认迁移一致。
- `prisma generate`：生成客户端。
- lint、typecheck、test。
- 构建产物包含生成目录。

### 8.3 环境配置

- `.env` 不进版本控制，用环境变量注入。
- 连接串区分开发/测试/生产。
- 生产不打印 query 日志。

## 9. 项目增量

将博客 API 接入 Express 或 NestJS，客户端单例化，配置查询日志，添加文章列表分页和缓存，编写生产部署脚本。

## 阶段四验收

- 能实现客户端单例并控制连接。
- 能接入 Web 框架并暴露查询 API。
- 能配置日志、监控与缓存。
- 能执行正确的生产迁移与部署流程。
- 能在 CI 中校验 Schema 与迁移。

## 常见排错

| 现象 | 排查方向 |
|---|---|
| 连接数耗尽 | 客户端单例、连接池配置 |
| 部署后客户端未生成 | 构建流程加入 prisma generate |
| 生产迁移未应用 | 部署前执行 migrate deploy |
| 慢查询 | 索引、select、include 优化 |
| 事务冲突 | 重试与隔离级别配置 |
| 查询日志过多 | 生产关闭 query 日志 |

## 进入下一阶段的条件

你能将 Prisma 应用到生产级项目。此时进入 [综合实战：博客 API](./project-practice.md)。
