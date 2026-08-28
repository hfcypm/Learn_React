# 阶段零：Prisma 基础与工具链

**目标**：理解 Prisma 的架构与定位，初始化项目，定义第一个 Schema，生成客户端并完成连接。

## 1. Prisma 是什么

Prisma 是 Node.js 和 TypeScript 的 ORM，用 Schema 定义数据模型，自动生成类型安全的查询客户端。典型流程：

```text
定义 Schema -> prisma migrate dev 建表 -> prisma generate 生成客户端
   -> 用 PrismaClient 查询
```

核心组件：

- Prisma Schema：模型、关系、数据源和生成器的声明。
- Prisma Client：类型安全的查询 API。
- Prisma Migrate：迁移工作流。
- Prisma Studio：可视化数据查看与编辑。

## 2. 初始化项目

```bash
# 创建项目
mkdir blog-api && cd blog-api
npm init -y

# 安装依赖
npm install @prisma/client @prisma/adapter-pg pg
npm install -D prisma typescript @types/node @types/pg tsx

# 初始化 TypeScript
npx tsc --init

# 初始化 Prisma
npx prisma init --datasource-provider postgresql
```

初始化后生成 `prisma/schema.prisma` 和 `prisma.config.ts`。Prisma 7 通过 `prisma.config.ts` 显式加载环境变量：

```ts
// prisma.config.ts
import 'dotenv/config';
import { defineConfig } from 'prisma/config';

export default defineConfig({
  schema: 'prisma/schema.prisma',
  datasource: {
    url: process.env.DATABASE_URL,
  },
});
```

## 3. Schema 结构

```prisma
generator client {
  provider = "prisma-client"
  output   = "./generated"
}

datasource db {
  provider = "postgresql"
}

model User {
  id    Int    @id @default(autoincrement())
  email String @unique
  name  String
}
```

- `generator client`：Prisma 7 使用 `prisma-client`，生成到 `output` 指定目录。
- `datasource`：声明数据库类型。
- `model`：映射数据库表。

## 4. 连接串

`prisma.config.ts` 中的 `datasource.url` 从环境变量读取，`.env` 存放连接串：

```env
DATABASE_URL="postgresql://postgres:postgres@localhost:5432/blog_api"
```

## 5. 首次迁移与生成
```bash
# 生成迁移并创建表
npx prisma migrate dev --name init

# 生成 Prisma Client
npx prisma generate
```

## 6. 使用 PrismaClient

Prisma 7 通过 driver adapter 连接数据库，PostgreSQL 使用 `@prisma/adapter-pg`：

```bash
npm install @prisma/adapter-pg pg
npm install -D @types/pg
```

```ts
// src/client.ts
import { PrismaClient } from './generated/client';
import { PrismaPg } from '@prisma/adapter-pg';

const adapter = new PrismaPg({
  connectionString: process.env.DATABASE_URL,
});

export const prisma = new PrismaClient({ adapter });
```

```ts
// src/index.ts
import { prisma } from './client';

async function main() {
  const user = await prisma.user.create({
    data: { email: 'a@example.com', name: 'Alice' },
  });
  console.log(user);
}

main()
  .catch((e) => console.error(e))
  .finally(() => prisma.$disconnect());
```

## 7. Prisma Studio

```bash
npx prisma studio
```

可视化查看和编辑数据，适合调试模型。

## 8. 动手任务

1. 初始化博客项目，安装 Prisma 与 driver adapter。
2. 定义 `User` 模型，生成首次迁移并创建表。
3. 用 PrismaClient 插入并查询一条数据。
4. 用 Prisma Studio 查看并编辑数据。
5. 修改 Schema 添加一个字段，重新迁移并观察生成的 SQL。

## 阶段零验收

- 能解释 Schema、Client、Migrate 的角色。
- 能初始化项目并正确配置连接串。
- 能生成迁移并创建表。
- 能用 PrismaClient 完成一次写入和读取。

## 常见排错

| 现象 | 排查方向 |
|---|---|
| `DATABASE_URL` 未找到 | 确认 `.env` 存在且未被忽略 |
| 连接被拒 | 数据库未启动、密码或端口错误 |
| `prisma-client` 生成器报错 | 确认使用 Prisma 7 语法 |
| 客户端类型缺失 | 重新 `npx prisma generate` |
| 迁移失败 | 检查 Schema 语法与数据库权限 |

## 进入下一阶段的条件

你能够连接数据库并用客户端完成基本操作。此时进入 [阶段一：Schema 数据建模](./stage-1-schema-modeling.md)。
