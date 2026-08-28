# 01 项目初始化：搭建四栈骨架

**目标**：用一条命令创建 Next.js 项目，接入 Tailwind CSS v4、Prisma 7 与 PostgreSQL 17，跑通「建表 -> 迁移 -> 查询」的最小闭环。

## 1. 前置条件

- Node.js 20+ 与 npm。
- 本机已安装并启动 PostgreSQL 17（参考 [PostgreSQL 阶段零](../postgresql-learning-path/stage-0-foundations.md)）。
- 了解 Next.js App Router 目录约定（参考 [Next.js 阶段零](../nextjs-learning-path/stage-0-foundations.md)）。

```bash
node --version
npm --version
psql --version
```

## 2. 创建项目

```bash
npx create-next-app@latest kanban --typescript --app --tailwind --eslint

cd kanban
```

创建完成后自带 `app/` 目录与 Tailwind 配置。

## 3. 接入 Tailwind CSS v4

`create-next-app` 已生成 `app/globals.css`，其中通过 `@import "tailwindcss"` 引入 Tailwind v4：

```css
/* app/globals.css */
@import "tailwindcss";
```

`create-next-app` 默认使用 PostCSS 集成，无需手动安装插件。

## 4. 安装 Prisma 与 driver adapter

```bash
npm install @prisma/client @prisma/adapter-pg pg
npm install -D prisma @types/pg

npx prisma init --datasource-provider postgresql
```

初始化生成 `prisma/schema.prisma` 与 `prisma.config.ts`。

## 5. 配置连接串

Prisma 7 通过 `prisma.config.ts` 显式加载环境变量：

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

```env
# .env
DATABASE_URL="postgresql://postgres:postgres@localhost:5432/kanban"
```

先创建数据库：

```bash
createdb kanban
```

## 6. 定义第一个模型并迁移

```prisma
// prisma/schema.prisma
generator client {
  provider = "prisma-client"
  output   = "./generated"
}

datasource db {
  provider = "postgresql"
}

model User {
  id       Int     @id @default(autoincrement())
  email    String  @unique
  name     String
  password String
}
```

```bash
npx prisma migrate dev --name init
npx prisma generate
```

## 7. 建立全局客户端单例

Prisma 7 通过 driver adapter 连接，PostgreSQL 使用 `@prisma/adapter-pg`：

```ts
// lib/db.ts
import { PrismaClient } from '@/generated/client';
import { PrismaPg } from '@prisma/adapter-pg';

const globalForPrisma = globalThis as unknown as { prisma?: PrismaClient };

function createPrisma() {
  const adapter = new PrismaPg({
    connectionString: process.env.DATABASE_URL,
  });
  return new PrismaClient({ adapter });
}

export const prisma = globalForPrisma.prisma ?? createPrisma();

if (process.env.NODE_ENV !== 'production') {
  globalForPrisma.prisma = prisma;
}
```

注意：`@/generated/client` 需在 `tsconfig.json` 的 `paths` 中配置别名，或使用相对路径导入。

## 8. 最小闭环验证

在首页读取一条数据：

```tsx
// app/page.tsx
import { prisma } from '@/lib/db';

export default async function Home() {
  const count = await prisma.user.count();
  return (
    <main>
      <h1>Kanban</h1>
      <p>用户数：{count}</p>
    </main>
  );
}
```

```bash
npm run dev
```

访问 `http://localhost:3000`，应显示用户数 0。

## 9. 本章验收

- [ ] `npm run dev` 启动无错误。
- [ ] Prisma 迁移成功，数据库中已生成 `User` 表。
- [ ] 首页通过 Server Component 读到数据库计数。
- [ ] 修改 Schema 后重新迁移，表结构能同步变化。

## 10. 常见排错

| 现象 | 排查方向 |
|---|---|
| `DATABASE_URL` 未找到 | 确认 `.env` 存在且 `prisma.config.ts` 加载了 dotenv |
| 连接被拒 | 数据库未启动、密码或端口错误 |
| 生成器报错 | 确认 Prisma 7 的 `prisma-client` 语法 |
| 客户端类型缺失 | 重新 `npx prisma generate` |
| 迁移失败 | 检查 Schema 语法与数据库权限 |
| `@/generated` 解析失败 | 配置 tsconfig `paths` 别名 |

## 进入下一章的条件

能初始化项目并通过 Server Component 读到数据库数据。此时进入 [02 数据建模](./02-schema.md)。
