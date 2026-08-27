# 阶段三：Server Actions、表单与鉴权

**目标**：掌握 Server Actions 的变更流程、表单处理、输入校验、鉴权和权限边界，能够构建安全的交互功能。

## 1. Server Actions

### 1.1 是什么

Server Actions 是运行在服务端的函数，客户端调用后执行服务端逻辑。主要用途是数据变更（创建、更新、删除），而不是数据获取——数据获取优先用 Server Component 或 Route Handlers。

创建方式：在文件顶部加 `"use server"`，或在函数内单独声明：

```ts
// app/actions/posts.ts —— 整个文件都是 Server Actions
'use server';

import { revalidatePath } from 'next/cache';
import { createPost } from '@/lib/posts';

export async function publishPost(formData: FormData) {
  const title = String(formData.get('title') ?? '');
  const body = String(formData.get('body') ?? '');

  // 输入校验
  if (!title || title.length < 3) {
    return { error: '标题至少 3 个字符' };
  }

  await createPost({ title, body });

  // 使列表缓存失效，让页面重新生成
  revalidatePath('/posts');

  return { success: true };
}
```

### 1.2 在表单中使用

```tsx
// app/posts/new/page.tsx
import { publishPost } from '@/app/actions/posts';

export default function NewPostPage() {
  return (
    <form action={publishPost}>
      <label>
        标题
        <input name="title" required minLength={3} />
      </label>
      <label>
        正文
        <textarea name="body" required />
      </label>
      <button type="submit">发布</button>
    </form>
  );
}
```

`<form action={serverAction}>` 是原生 HTML 表单的扩展：提交时把 FormData 传给服务端函数，不依赖 JavaScript 也能工作。

### 1.3 读取表单状态

```tsx
'use client';

import { useActionState } from 'react';
import { publishPost } from '@/app/actions/posts';

export default function NewPostForm() {
  const [state, formAction, pending] = useActionState(publishPost, null);

  return (
    <form action={formAction}>
      <input name="title" required />
      {state?.error && <p className="error">{state.error}</p>}
      <button type="submit" disabled={pending}>
        {pending ? '发布中...' : '发布'}
      </button>
    </form>
  );
}
```

`useActionState` 接收 Server Action，返回当前状态、包装后的 form action 和 pending 标记。适合需要显示错误和提交状态的表单。

## 2. 输入校验

### 2.1 在服务端校验

Server Action 必须对输入做服务端校验，不能依赖浏览器端的 `required`：

```ts
'use server';

import { revalidatePath } from 'next/cache';
import { createPost } from '@/lib/posts';

export async function publishPost(formData: FormData) {
  const title = String(formData.get('title') ?? '').trim();
  const body = String(formData.get('body') ?? '').trim();

  if (title.length < 3 || title.length > 120) {
    return { fieldErrors: { title: '标题长度需在 3 到 120 之间' } };
  }

  if (body.length < 10) {
    return { fieldErrors: { body: '正文至少 10 个字符' } };
  }

  await createPost({ title, body });
  revalidatePath('/posts');
  return { success: true };
}
```

### 2.2 使用 schema 校验（Zod）

```ts
// lib/schemas.ts
import { z } from 'zod';

export const PostSchema = z.object({
  title: z.string().min(3).max(120),
  body: z.string().min(10).max(5000),
});

export type PostInput = z.infer<typeof PostSchema>;
```

```ts
// app/actions/posts.ts
'use server';

import { revalidatePath } from 'next/cache';
import { PostSchema } from '@/lib/schemas';
import { createPost } from '@/lib/posts';

export async function publishPost(formData: FormData) {
  const parsed = PostSchema.safeParse({
    title: formData.get('title'),
    body: formData.get('body'),
  });

  if (!parsed.success) {
    return { errors: parsed.error.flatten().fieldErrors };
  }

  await createPost(parsed.data);
  revalidatePath('/posts');
  return { success: true };
}
```

`safeParse` 不抛异常，直接返回校验结果，便于把字段错误回传给表单。校验失败时 `revalidatePath` 不要执行。

## 3. 鉴权

### 3.1 在服务端读取会话

```ts
// lib/auth.ts
import { cookies } from 'next/headers';
import { verifySessionToken } from './session';

export async function getSession() {
  const cookieStore = await cookies();
  const token = cookieStore.get('session')?.value;

  if (!token) return null;

  return verifySessionToken(token);
}
```

```tsx
// app/posts/[id]/page.tsx
import { getSession } from '@/lib/auth';

export default async function PostPage({
  params,
}: {
  params: Promise<{ id: string }>;
}) {
  const { id } = await params;
  const session = await getSession();

  return (
    <article>
      <h1>文章 {id}</h1>
      {session ? <button>编辑</button> : <a href="/login">登录后编辑</a>}
    </article>
  );
}
```

### 3.2 在 Server Action 中鉴权

服务端变更必须二次检查权限，前端隐藏按钮只改善体验，不能作为安全边界：

```ts
'use server';

import { revalidatePath } from 'next/cache';
import { getSession } from '@/lib/auth';
import { updatePost } from '@/lib/posts';

export async function updatePostAction(formData: FormData) {
  const session = await getSession();

  if (!session?.user) {
    return { error: '请先登录' };
  }

  const postId = String(formData.get('id') ?? '');
  const post = await getPostById(postId);

  // 资源归属校验：只有作者或管理员能改
  if (post.authorId !== session.user.id && session.user.role !== 'admin') {
    return { error: '没有权限修改该文章' };
  }

  // 校验输入并更新...
  await updatePost({ id: postId, title, body });
  revalidatePath(`/posts/${postId}`);
  return { success: true };
}
```

### 3.3 中间件保护路由

`middleware.ts` 在请求到达页面之前运行，适合重定向未登录用户：

```ts
// middleware.ts
import { NextRequest, NextResponse } from 'next/server';

export function middleware(request: NextRequest) {
  const session = request.cookies.get('session')?.value;

  if (!session) {
    const loginUrl = new URL('/login', request.url);
    loginUrl.searchParams.set('from', request.nextUrl.pathname);
    return NextResponse.redirect(loginUrl);
  }

  return NextResponse.next();
}

export const config = {
  matcher: ['/dashboard/:path*', '/posts/new'],
};
```

中间件是轻量过滤层，适合重定向和基础判断；敏感操作仍要在 Server Action 中完整校验权限。

## 4. 登录与会话实践

### 4.1 登录表单

```tsx
'use client';

import { useActionState } from 'react';
import { loginAction } from '@/app/actions/auth';

export default function LoginForm() {
  const [state, formAction, pending] = useActionState(loginAction, null);

  return (
    <form action={formAction}>
      <input name="email" type="email" required />
      <input name="password" type="password" required />
      {state?.error && <p className="error">{state.error}</p>}
      <button type="submit" disabled={pending}>
        {pending ? '登录中...' : '登录'}
      </button>
    </form>
  );
}
```

### 4.2 登录 Server Action

```ts
// app/actions/auth.ts
'use server';

import { cookies } from 'next/headers';
import { redirect } from 'next/navigation';
import { verifyCredentials, createSession } from '@/lib/session';

export async function loginAction(formData: FormData) {
  const email = String(formData.get('email') ?? '').toLowerCase();
  const password = String(formData.get('password') ?? '');

  const user = await verifyCredentials(email, password);

  if (!user) {
    return { error: '邮箱或密码错误' };
  }

  const token = await createSession(user);
  const cookieStore = await cookies();
  cookieStore.set('session', token, {
    httpOnly: true,
    secure: process.env.NODE_ENV === 'production',
    sameSite: 'lax',
    path: '/',
  });

  redirect('/dashboard');
}
```

会话 Cookie 必须 `httpOnly`，防止浏览器脚本读取；生产环境加 `secure`。

## 5. 常见排错

| 现象 | 排查方向 |
|---|---|
| 表单提交不执行 Server Action | 确认函数是 `"use server"` 且返回可序列化值 |
| 校验不生效 | 确认校验在服务端而非只在浏览器 |
| 更新后页面不刷新 | 变更后调用 `revalidatePath` 或 `revalidateTag` |
| 权限检查被绕过 | 确认在 Server Action 内二次校验会话与资源归属 |
| 中间件不匹配 | 检查 `matcher` 配置 |
| 登录后无会话 | 检查 Cookie 设置：httpOnly、secure、sameSite |

## 阶段三验收

- 能使用 Server Actions 完成创建、更新和删除。
- 能在服务端校验输入并回传字段错误。
- 能实现登录、会话 Cookie 和退出。
- 能在 Server Action 中完成鉴权和资源归属校验。
- 能使用中间件做基础路由保护。

## 动手任务

1. 为内容平台添加发布文章表单，使用 Server Action。
2. 使用 Zod 校验输入并回显字段错误。
3. 实现登录和退出，会话使用 httpOnly Cookie。
4. 只有作者和管理员能编辑和删除文章。
5. 用中间件保护 `/dashboard` 路由。

## 进入下一阶段的条件

你能够通过 Server Actions 完成带校验和权限的完整交互。此时进入 [阶段四：性能、工程化与生产交付](./stage-4-production.md)。
