# 阶段二：数据获取、缓存与重新验证

**目标**：掌握服务端数据获取、静态与动态渲染、ISR、缓存失效，能够为不同页面选择正确的渲染与缓存策略。

## 1. 在 Server Component 中获取数据

Server Component 可以直接读取数据库、文件系统或调用外部 API，不需要在浏览器发起请求。

```tsx
// app/posts/page.tsx
import { getPosts } from '@/lib/posts';

export default async function PostsPage() {
  const posts = await getPosts();
  return <ul>{posts.map((p) => <li key={p.id}>{p.title}</li>)}</ul>;
}
```

```ts
// lib/posts.ts —— 数据库访问示例
import { db } from './db';

export async function getPosts() {
  return db.post.findMany({ orderBy: { createdAt: 'desc' } });
}
```

数据请求放服务端的收益：

- 减少浏览器到 API 的往返和暴露的 Token。
- 一次请求即可渲染完整 HTML，首屏更快。
- 数据库连接、凭据等敏感能力不进入浏览器。

## 2. 静态渲染、动态渲染与 ISR

### 2.1 渲染模式对比

| 模式 | 渲染时机 | 数据新鲜度 | 适用场景 |
|---|---|---|---|
| 静态渲染（Static） | 构建时 | 构建后不变 | 文档、公告、营销页 |
| 动态渲染（Dynamic） | 每次请求 | 实时 | 个性化、登录态、实时数据 |
| ISR | 构建时 + 按间隔更新 | 可接受延迟 | 内容频繁但不要求实时 |

### 2.2 静态渲染

没有动态 API 的页面默认静态渲染，构建时生成 HTML，请求时直接返回，速度快、可缓存：

```tsx
// 默认静态渲染
export default async function AboutPage() {
  return <h1>关于我们</h1>;
}
```

### 2.3 动态渲染

使用 `cookies()`、`headers()` 或读取请求时动态数据时，页面自动变为动态渲染：

```tsx
// 读取 cookie 判断登录态，页面动态渲染
import { cookies } from 'next/headers';

export default async function ProfilePage() {
  const cookieStore = await cookies();
  const theme = cookieStore.get('theme')?.value ?? 'light';
  return <div>当前主题：{theme}</div>;
}
```

动态渲染在每次请求时重新执行，数据始终最新，但缓存效率低。

### 2.4 ISR：按间隔重新验证

使用 `revalidate` 配置，构建后按秒数间隔重新生成页面：

```tsx
// app/posts/page.tsx
export const revalidate = 60; // 60 秒重新验证一次

export default async function PostsPage() {
  const posts = await getPosts();
  return <ul>{posts.map((p) => <li key={p.id}>{p.title}</li>)}</ul>;
}
```

首次请求渲染并缓存，之后 60 秒内的请求返回缓存，60 秒后触发后台重新生成。适合内容发布不频繁但访问量大的场景。

### 2.5 主动重新验证

使用 `revalidatePath` 在数据变更后立即失效缓存：

```ts
// 在 Server Action 或 Route Handler 中调用
import { revalidatePath } from 'next/cache';

export async function publishPost(formData: FormData) {
  // 创建文章...
  await createPost(formData);
  // 让 /posts 路径在下一次请求时重新生成
  revalidatePath('/posts');
}
```

使用 `revalidateTag` 失效一组相关数据：

```tsx
// 数据获取时打标签
import { unstable_cache } from 'next/cache';

export const getPosts = unstable_cache(
  async () => db.post.findMany(),
  ['posts'],
  { tags: ['posts'] },
);
```

```ts
// 变更后失效所有带 posts 标签的数据
import { revalidateTag } from 'next/cache';

export async function deletePost(id: string) {
  await db.post.delete({ where: { id } });
  revalidateTag('posts');
}
```

## 3. Route Handlers

Route Handlers 用 `route.ts` 定义 API 端点，适合表单提交之外的通用接口和 webhook。

```ts
// app/api/posts/route.ts
import { NextResponse } from 'next/server';
import { getPosts, createPost } from '@/lib/posts';

export async function GET(request: Request) {
  const { searchParams } = new URL(request.url);
  const tag = searchParams.get('tag') ?? '';
  const posts = await getPosts(tag);
  return NextResponse.json(posts);
}

export async function POST(request: Request) {
  const body = await request.json();
  const post = await createPost(body);
  return NextResponse.json(post, { status: 201 });
}
```

```ts
// app/api/posts/[id]/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { getPost, deletePost } from '@/lib/posts';

export async function GET(
  request: NextRequest,
  { params }: { params: Promise<{ id: string }> },
) {
  const { id } = await params;
  const post = await getPost(id);

  if (!post) {
    return NextResponse.json({ error: '文章不存在' }, { status: 404 });
  }

  return NextResponse.json(post);
}

export async function DELETE(
  request: NextRequest,
  { params }: { params: Promise<{ id: string }> },
) {
  const { id } = await params;
  await deletePost(id);
  return new NextResponse(null, { status: 204 });
}
```

Route Handler 配置缓存与 ISR：

```ts
export const revalidate = 60;

export async function GET() {
  const data = await fetch('https://api.example.com/posts');
  return Response.json(await data.json());
}
```

Route Handlers 适合数据接口和 webhook；表单提交和数据变更优先使用 Server Actions。

## 4. fetch 的缓存行为

在 Server Component 中使用 fetch 时，Next.js 默认缓存 GET 请求：

```tsx
// 默认缓存，除非请求包含动态参数
const data = await fetch('https://api.example.com/posts');

// 禁用该请求缓存
const data = await fetch('https://api.example.com/posts', {
  cache: 'no-store',
});
```

```tsx
// 按间隔重新验证
const data = await fetch('https://api.example.com/posts', {
  next: { revalidate: 60 },
});

// 打标签，配合 revalidateTag
const data = await fetch('https://api.example.com/posts', {
  next: { tags: ['posts'] },
});
```

## 5. 客户端数据获取

需要频繁交互、实时更新或分页加载时，使用 TanStack Query 等客户端数据层：

```tsx
'use client';

import { useQuery } from '@tanstack/react-query';
import { getPosts } from '@/lib/client-posts';

export default function PostsFeed() {
  const { data, isLoading, error } = useQuery({
    queryKey: ['posts'],
    queryFn: getPosts,
  });

  if (isLoading) return <div>加载中...</div>;
  if (error) return <div>加载失败</div>;

  return <ul>{data.map((p) => <li key={p.id}>{p.title}</li>)}</ul>;
}
```

选择原则：初始渲染用 Server Component 获取；交互高频、分页、轮询、缓存失效用客户端数据层。

## 6. 常见排错

| 现象 | 排查方向 |
|---|---|
| 页面数据不更新 | 检查缓存与 `revalidate` 配置 |
| 变更后列表不刷新 | 在变更后调用 `revalidatePath` 或 `revalidateTag` |
| fetch 一直返回旧数据 | 确认没有默认缓存且按需配置 `no-store` |
| 页面变成动态但不需要 | 检查是否误用了 `cookies()`、`headers()` |
| Route Handler 404 | 确认 `route.ts` 文件路径和方法命名 |

## 阶段二验收

- 能区分静态渲染、动态渲染和 ISR 并选择场景。
- 能使用 `revalidatePath` 和 `revalidateTag` 失效缓存。
- 能编写 Route Handlers 并配置缓存与重新验证。
- 能理解 fetch 默认缓存行为并正确覆盖。
- 能判断何时使用 Server Component 获取、何时使用客户端数据层。

## 动手任务

1. 为文章列表添加 ISR，并验证缓存更新时间。
2. 在发布文章后调用 `revalidatePath` 使列表立即刷新。
3. 为文章详情编写 Route Handler，包含 404 和参数校验。
4. 使用 `revalidateTag` 让详情页和列表页共享失效。
5. 为实时模块接入 TanStack Query 并处理加载和错误状态。

## 进入下一阶段的条件

你能够为不同页面选择渲染模式并设计缓存失效策略。此时进入 [阶段三：Server Actions、表单与鉴权](./stage-3-forms-auth.md)。
