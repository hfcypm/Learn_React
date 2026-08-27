# Next.js 综合实战：内容平台

## 1. 项目目标

构建一个带登录、文章发布、列表、详情、分类和缓存的完整内容平台。从静态骨架开始，按阶段逐步增加路由、数据、鉴权和部署能力，最终完成一个可运行的 Next.js 全栈应用。

```text
访问首页 -> 查看文章列表 -> 查看文章详情
    -> 登录/注册 -> 发布文章 -> 重新验证缓存
    -> 编辑/删除自己的文章 -> 生产构建与部署
```

前端用 App Router、Server Component、Server Actions 和 Tailwind CSS；数据先使用本地数据库或 Mock 数据层，之后替换为真实数据库。

## 2. 技术选择

| 技术 | 用途 |
|---|---|
| Next.js（App Router） | 路由、渲染、数据获取、API |
| React 19 | 组件、Server/Client 组件、useActionState |
| TypeScript | 类型安全 |
| Tailwind CSS | 样式 |
| Zod | 输入校验 |
| SQLite + Prisma | 数据库（可选） |
| Vitest + Playwright | 单元与 E2E 测试 |

## 3. 初始化项目

```bash
# 创建项目
npx create-next-app@latest content-platform --typescript --app --tailwind --eslint

# 进入项目
cd content-platform

# 安装校验与测试依赖（按需）
npm install zod
npm install -D vitest @testing-library/react @playwright/test
```

## 4. 目录结构

```text
app/
├── layout.tsx
├── page.tsx                 # 首页
├── login/
│   └── page.tsx             # 登录页
├── posts/
│   ├── page.tsx             # 文章列表
│   ├── new/
│   │   └── page.tsx         # 发布文章
│   └── [id]/
│       ├── page.tsx         # 文章详情
│       └── edit/page.tsx    # 编辑文章
├── actions/
│   ├── auth.ts              # 登录、退出
│   └── posts.ts             # 发布、编辑、删除
└── api/
    └── posts/
        └── route.ts         # 文章 API
components/
├── nav.tsx
├── post-card.tsx
└── post-form.tsx
lib/
├── db.ts                    # 数据访问
├── auth.ts                  # 会话
├── schemas.ts               # Zod 校验
└── env.ts                   # 环境变量校验
```

## 5. 数据模型

使用 Prisma 定义用户和文章：

```prisma
// prisma/schema.prisma
model User {
  id        String   @id @default(cuid())
  email     String   @unique
  name      String
  password  String
  role      String   @default("author")
  createdAt DateTime @default(now())
  posts     Post[]
}

model Post {
  id        String   @id @default(cuid())
  title     String
  body      String
  tag       String?
  published Boolean  @default(false)
  authorId  String
  author    User     @relation(fields: [authorId], references: [id])
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
}
```

## 6. 数据访问层

```ts
// lib/db.ts
import { PrismaClient } from '@prisma/client';

const globalForPrisma = globalThis as unknown as { prisma?: PrismaClient };

export const db = globalForPrisma.prisma ?? new PrismaClient();

if (process.env.NODE_ENV !== 'production') {
  globalForPrisma.prisma = db;
}
```

```ts
// lib/schemas.ts
import { z } from 'zod';

export const PostSchema = z.object({
  title: z.string().min(3).max(120),
  body: z.string().min(10).max(5000),
  tag: z.string().max(30).optional(),
});

export const LoginSchema = z.object({
  email: z.string().email(),
  password: z.string().min(8),
});
```

## 7. 认证与会话

```ts
// lib/auth.ts
import { cookies } from 'next/headers';
import { SignJWT, jwtVerify } from 'jose';
import { env } from './env';

export async function createSession(userId: string) {
  const secret = new TextEncoder().encode(env.AUTH_SECRET);

  return new SignJWT({ userId })
    .setProtectedHeader({ alg: 'HS256' })
    .setIssuedAt()
    .setExpirationTime('7d')
    .sign(secret);
}

export async function getSession() {
  const cookieStore = await cookies();
  const token = cookieStore.get('session')?.value;

  if (!token) return null;

  try {
    const secret = new TextEncoder().encode(env.AUTH_SECRET);
    const { payload } = await jwtVerify(token, secret);
    return { userId: payload.userId as string };
  } catch {
    return null;
  }
}
```

```ts
// app/actions/auth.ts
'use server';

import { cookies } from 'next/headers';
import { redirect } from 'next/navigation';
import { LoginSchema } from '@/lib/schemas';
import { db } from '@/lib/db';
import { verifyPassword } from '@/lib/password';
import { createSession } from '@/lib/auth';

export async function loginAction(formData: FormData) {
  const parsed = LoginSchema.safeParse({
    email: formData.get('email'),
    password: formData.get('password'),
  });

  if (!parsed.success) {
    return { error: '邮箱或密码格式不正确' };
  }

  const user = await db.user.findUnique({
    where: { email: parsed.data.email.toLowerCase() },
  });

  if (!user || !(await verifyPassword(parsed.data.password, user.password))) {
    return { error: '邮箱或密码错误' };
  }

  const token = await createSession(user.id);
  const cookieStore = await cookies();
  cookieStore.set('session', token, {
    httpOnly: true,
    secure: process.env.NODE_ENV === 'production',
    sameSite: 'lax',
    path: '/',
  });

  redirect('/');
}

export async function logoutAction() {
  const cookieStore = await cookies();
  cookieStore.delete('session');
  redirect('/login');
}
```

## 8. 文章发布与权限

```ts
// app/actions/posts.ts
'use server';

import { revalidatePath, revalidateTag } from 'next/cache';
import { redirect } from 'next/navigation';
import { PostSchema } from '@/lib/schemas';
import { db } from '@/lib/db';
import { getSession } from '@/lib/auth';

export async function publishPost(formData: FormData) {
  const session = await getSession();

  if (!session) {
    return { error: '请先登录' };
  }

  const parsed = PostSchema.safeParse({
    title: formData.get('title'),
    body: formData.get('body'),
    tag: formData.get('tag') || undefined,
  });

  if (!parsed.success) {
    return { errors: parsed.error.flatten().fieldErrors };
  }

  const post = await db.post.create({
    data: {
      ...parsed.data,
      authorId: session.userId,
      published: true,
    },
  });

  revalidatePath('/posts');
  revalidateTag('posts');
  redirect(`/posts/${post.id}`);
}

export async function deletePostAction(postId: string) {
  const session = await getSession();

  if (!session) {
    return { error: '请先登录' };
  }

  const post = await db.post.findUnique({ where: { id: postId } });

  if (!post) {
    return { error: '文章不存在' };
  }

  if (post.authorId !== session.userId) {
    return { error: '没有权限删除该文章' };
  }

  await db.post.delete({ where: { id: postId } });
  revalidatePath('/posts');
  revalidateTag('posts');
  return { success: true };
}
```

## 9. 页面示例

### 文章列表（服务端获取 + ISR）

```tsx
// app/posts/page.tsx
import { PostCard } from '@/components/post-card';
import { db } from '@/lib/db';
import { Suspense } from 'react';

export const revalidate = 60;

export default async function PostsPage() {
  const posts = await db.post.findMany({
    where: { published: true },
    orderBy: { createdAt: 'desc' },
    include: { author: true },
  });

  return (
    <main>
      <h1>全部文章</h1>
      <Suspense fallback={<div>加载中...</div>}>
        <div className="grid gap-4">
          {posts.map((post) => (
            <PostCard key={post.id} post={post} />
          ))}
        </div>
      </Suspense>
    </main>
  );
}
```

### 发布表单

```tsx
'use client';

import { useActionState } from 'react';
import { publishPost } from '@/app/actions/posts';

export function PostForm() {
  const [state, formAction, pending] = useActionState(publishPost, null);

  return (
    <form action={formAction} className="space-y-4">
      <input name="title" placeholder="标题" required className="w-full border p-2" />
      {state?.errors?.title && (
        <p className="text-red-600">{state.errors.title}</p>
      )}
      <textarea name="body" placeholder="正文" rows={10} required className="w-full border p-2" />
      {state?.errors?.body && (
        <p className="text-red-600">{state.errors.body}</p>
      )}
      <input name="tag" placeholder="分类（可选）" className="w-full border p-2" />
      <button type="submit" disabled={pending} className="bg-blue-600 text-white px-4 py-2">
        {pending ? '发布中...' : '发布'}
      </button>
    </form>
  );
}
```

## 10. 测试计划

| 层级 | 覆盖内容 |
|---|---|
| 单元测试 | Zod schema、数据过滤、密码哈希 |
| 组件测试 | PostCard、PostForm 的渲染与交互 |
| E2E 测试 | 注册登录、发布文章、编辑删除、缓存刷新 |
| 构建验收 | `next build` 成功，路由渲染标记正确 |

## 11. 部署

- 使用 Docker 或 Vercel 部署。
- `DATABASE_URL` 和 `AUTH_SECRET` 通过环境变量注入。
- 部署流程中执行 `prisma migrate deploy`。
- 健康检查确认数据库就绪后再接流量。
- 保留回滚版本和迁移降级方案。

## 12. 验收清单

- [ ] `npm run build` 成功，无类型错误和 lint 错误。
- [ ] 首页、列表、详情和登录页面路由正确。
- [ ] 未登录访问发布页被重定向到登录页。
- [ ] 只能编辑和删除自己的文章。
- [ ] 发布文章后列表通过 `revalidatePath` 刷新。
- [ ] 输入校验覆盖标题、正文和登录表单。
- [ ] 会话 Cookie 使用 httpOnly 和 secure。
- [ ] 单元、组件和 E2E 测试通过。
- [ ] 生产环境环境变量校验通过。
- [ ] Docker 或托管平台部署成功并可访问。

## 13. 实施顺序

1. 创建项目，完成根布局、导航和首页。
2. 完成文章列表、详情和动态路由。
3. 接入数据层，配置 ISR 和缓存失效。
4. 完成登录、会话和权限校验。
5. 完成发布、编辑和删除，配合 `revalidateTag`。
6. 补齐测试、性能优化和部署。
7. 运行验收清单并记录未实现的生产能力。

## 14. 按阶段学习卡片

| 阶段 | 项目增量 |
|---|---|
| 零 | 初始化项目、导航、静态页面 |
| 一 | 列表、详情、动态路由、布局边界 |
| 二 | 数据层、ISR、Route Handlers、缓存失效 |
| 三 | 登录、会话、发布、编辑、删除、权限 |
| 四 | 测试、性能、Docker、部署、监控 |
