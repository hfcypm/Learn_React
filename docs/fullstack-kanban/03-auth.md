# 03 鉴权：Server Actions、会话与权限

**目标**：用 Server Actions 实现注册、登录、退出，用 jose 管理会话 Cookie，用中间件保护路由，并用角色判断控制团队与项目操作权限。

## 1. 依赖安装

```bash
npm install zod bcryptjs jose
npm install -D @types/bcryptjs
```

## 2. 密码哈希

```ts
// lib/password.ts
import bcrypt from 'bcryptjs';

const SALT_ROUNDS = 10;

export async function hashPassword(plain: string) {
  return bcrypt.hash(plain, SALT_ROUNDS);
}

export async function verifyPassword(plain: string, hashed: string) {
  return bcrypt.compare(plain, hashed);
}
```

## 3. 会话管理（jose）

```ts
// lib/session.ts
import { SignJWT, jwtVerify } from 'jose';

const secret = new TextEncoder().encode(process.env.SESSION_SECRET);

export type Session = {
  userId: number;
  teamId?: number;
  role: 'ADMIN' | 'MEMBER';
};

export async function createSession(payload: Session) {
  return new SignJWT(payload)
    .setProtectedHeader({ alg: 'HS256' })
    .setIssuedAt()
    .setExpirationTime('7d')
    .sign(secret);
}

export async function verifySession(token: string): Promise<Session | null> {
  try {
    const { payload } = await jwtVerify(token, secret);
    return payload as unknown as Session;
  } catch {
    return null;
  }
}
```

`.env` 增加密钥占位符（不提交真实值）：

```env
SESSION_SECRET=change-me-to-a-long-random-string
```

## 4. 登录表单（Client Component）

```tsx
// app/login/login-form.tsx
'use client';

import { useActionState } from 'react';
import { login } from '@/app/actions/auth';

export function LoginForm() {
  const [state, action, pending] = useActionState(login, { error: '' });

  return (
    <form action={action}>
      <input name="email" type="email" placeholder="邮箱" required />
      <input name="password" type="password" placeholder="密码" required />
      <button disabled={pending}>{pending ? '登录中…' : '登录'}</button>
      {state.error && <p>{state.error}</p>}
    </form>
  );
}
```

## 5. 登录 Server Action

```ts
// app/actions/auth.ts
'use server';

import { redirect } from 'next/navigation';
import { cookies } from 'next/headers';
import { z } from 'zod';
import { prisma } from '@/lib/db';
import { verifyPassword } from '@/lib/password';
import { createSession } from '@/lib/session';

const LoginSchema = z.object({
  email: z.string().email(),
  password: z.string().min(8),
});

export type LoginState = { error?: string };

export async function login(_prev: LoginState, formData: FormData) {
  const parsed = LoginSchema.safeParse({
    email: formData.get('email'),
    password: formData.get('password'),
  });

  if (!parsed.success) {
    return { error: '邮箱或密码格式不正确' };
  }

  const user = await prisma.user.findUnique({
    where: { email: parsed.data.email },
    include: { memberships: true },
  });

  if (!user || !(await verifyPassword(parsed.data.password, user.password))) {
    return { error: '邮箱或密码错误' };
  }

  const membership = user.memberships[0];
  const token = await createSession({
    userId: user.id,
    teamId: membership?.teamId,
    role: membership?.role ?? 'MEMBER',
  });

  const cookieStore = await cookies();
  cookieStore.set('session', token, {
    httpOnly: true,
    sameSite: 'lax',
    secure: process.env.NODE_ENV === 'production',
    path: '/',
    maxAge: 60 * 60 * 24 * 7,
  });

  redirect('/dashboard');
}
```

要点：

- `'use server'` 声明 Server Action，可被表单直接调用。
- 用 Zod 在服务端校验输入（客户端校验可被绕过）。
- 查询失败不暴露细节，统一返回「邮箱或密码错误」。
- 登录成功写 `httpOnly` Cookie，前端 JavaScript 不可读。

## 6. 服务端读取会话

```ts
// lib/auth.ts
import { cookies } from 'next/headers';
import { redirect } from 'next/navigation';
import { verifySession, type Session } from '@/lib/session';

export async function getSession(): Promise<Session | null> {
  const cookieStore = await cookies();
  const token = cookieStore.get('session')?.value;
  if (!token) return null;
  return verifySession(token);
}

export async function requireUser() {
  const session = await getSession();
  if (!session) redirect('/login');
  return session;
}
```

## 7. 中间件保护路由

```ts
// middleware.ts
import { NextRequest, NextResponse } from 'next/server';
import { verifySession } from '@/lib/session';

const PUBLIC_PATHS = ['/login', '/register', '/'];

export async function middleware(request: NextRequest) {
  const token = request.cookies.get('session')?.value;
  const { pathname } = request.nextUrl;

  const isPublic = PUBLIC_PATHS.some(
    (p) => pathname === p || pathname.startsWith(`${p}/`)
  );

  const session = token ? await verifySession(token) : null;

  if (!isPublic && !session) {
    const loginUrl = new URL('/login', request.url);
    loginUrl.searchParams.set('next', pathname);
    return NextResponse.redirect(loginUrl);
  }

  return NextResponse.next();
}

export const config = {
  matcher: ['/((?!_next/static|_next/image|favicon.ico).*)'],
};
```

## 8. 角色与团队权限

团队成员可操作自己团队的项目，管理员可管理成员：

```ts
// lib/permissions.ts
import { prisma } from '@/lib/db';
import type { Session } from '@/lib/session';

export async function assertTeamAccess(session: Session, teamId: number) {
  if (session.teamId !== teamId) {
    throw new Error('无权访问该团队');
  }
}

export async function assertTeamAdmin(session: Session, teamId: number) {
  if (session.role !== 'ADMIN' || session.teamId !== teamId) {
    throw new Error('需要团队管理员权限');
  }
}

export async function getProject(projectId: number) {
  return prisma.project.findUnique({
    where: { id: projectId },
    include: { team: true },
  });
}
```

在 Server Action 中做鉴权：

```ts
// app/actions/projects.ts
'use server';

import { revalidatePath } from 'next/cache';
import { requireUser } from '@/lib/auth';
import { assertTeamAdmin } from '@/lib/permissions';
import { prisma } from '@/lib/db';

export async function createProject(teamId: number, formData: FormData) {
  const session = await requireUser();
  await assertTeamAdmin(session, teamId);

  const name = String(formData.get('name')).trim();
  if (!name) throw new Error('项目名不能为空');

  await prisma.project.create({
    data: { teamId, name },
  });

  revalidatePath(`/teams/${teamId}`);
}
```

## 9. 注册

注册动作复用登录的会话写入逻辑：

```ts
export async function register(formData: FormData) {
  const parsed = RegisterSchema.safeParse({
    email: formData.get('email'),
    name: formData.get('name'),
    password: formData.get('password'),
  });

  if (!parsed.success) return { error: '输入不合法' };

  const exists = await prisma.user.findUnique({
    where: { email: parsed.data.email },
  });
  if (exists) return { error: '邮箱已被注册' };

  const user = await prisma.user.create({
    data: {
      email: parsed.data.email,
      name: parsed.data.name,
      password: await hashPassword(parsed.data.password),
    },
  });

  // 写入会话并跳转创建团队页
}
```

## 10. 退出登录

```ts
export async function logout() {
  const cookieStore = await cookies();
  cookieStore.delete('session');
  redirect('/');
}
```

## 11. 本章验收

- [ ] 注册、登录、退出流程完整。
- [ ] Cookie 为 httpOnly，前端无法读取。
- [ ] 未登录访问受保护路由会被重定向到登录页。
- [ ] 非管理员调用管理 Action 被拒绝。
- [ ] 密码以 bcrypt 哈希存储，数据库无明文。

## 12. 常见排错

| 现象 | 排查方向 |
|---|---|
| 登录后仍跳回登录页 | Cookie 写入失败、`secure` 与协议不匹配 |
| 中间件未生效 | 检查 `matcher` 排除静态资源 |
| 会话无效 | SESSION_SECRET 变化导致旧签名失效 |
| useActionState 不更新 | Server Action 需返回新状态对象 |
| 权限校验被绕过 | 鉴权必须在服务端 Action 内完成 |
| bcrypt 类型报错 | 安装 `@types/bcryptjs` |

## 进入下一章的条件

能注册登录并访问受保护页面。此时进入 [04 数据获取](./04-data-fetching.md)。
