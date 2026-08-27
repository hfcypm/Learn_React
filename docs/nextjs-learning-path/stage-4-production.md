# 阶段四：性能、工程化与生产交付

**目标**：掌握性能优化、测试、部署、监控和故障恢复，能够交付可维护、可观测的 Next.js 应用。

## 1. 性能优化

### 1.1 内置优化组件

**`next/image`** 自动处理尺寸、格式、懒加载和响应式：

```tsx
import Image from 'next/image';

export default function Hero() {
  return (
    <Image
      src="/images/hero.jpg"
      alt="横幅"
      width={1200}
      height={600}
      priority
      sizes="(max-width: 768px) 100vw, 50vw"
    />
  );
}
```

**`next/font`** 避免字体加载阻塞渲染：

```tsx
// app/layout.tsx
import { Inter } from 'next/font/google';

const inter = Inter({ subsets: ['latin'] });

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="zh-CN" className={inter.className}>
      <body>{children}</body>
    </html>
  );
}
```

### 1.2 核心 Web 指标

| 指标 | 含义 | 优化手段 |
|---|---|---|
| LCP | 最大内容绘制 | 图片优化、优先加载首屏、代码分割 |
| INP | 交互延迟 | 减少主线程阻塞、懒加载交互组件 |
| CLS | 布局偏移 | 给图片和广告占位、避免动态注入布局 |

### 1.3 减少客户端 JavaScript

- 默认使用 Server Component，减少发送到浏览器的代码。
- 交互组件使用 `next/dynamic` 懒加载：

```tsx
import dynamic from 'next/dynamic';

const Editor = dynamic(() => import('@/components/Editor'), {
  loading: () => <div>编辑器加载中...</div>,
});
```

- 大型图表、编辑器、富文本等按需加载，避免拖慢首屏。

### 1.4 使用 `useMemo` 与 `useCallback`

在 Client Component 中为昂贵计算和稳定引用使用 `useMemo`/`useCallback`，但不要过度使用：

```tsx
'use client';

import { useMemo } from 'react';

export default function PostList({ posts }: { posts: Post[] }) {
  const sortedPosts = useMemo(
    () => [...posts].sort((a, b) => b.createdAt - a.createdAt),
    [posts],
  );

  return <ul>{sortedPosts.map((p) => <li key={p.id}>{p.title}</li>)}</ul>;
}
```

### 1.5 性能测量

在开发环境使用 Lighthouse 和浏览器 Performance 面板测量 LCP、INP、CLS 和包体积。`next build` 输出每条路由的 First Load JS，关注大型路由的首屏体积。

## 2. 测试

### 2.1 单元测试（Vitest）

为纯逻辑和工具函数编写单元测试：

```ts
// lib/posts.test.ts
import { describe, it, expect } from 'vitest';
import { filterByTag } from './posts';

describe('filterByTag', () => {
  it('按标签过滤文章', () => {
    const posts = [
      { id: '1', title: 'A', tag: 'react' },
      { id: '2', title: 'B', tag: 'next' },
    ];

    expect(filterByTag(posts, 'react')).toHaveLength(1);
  });
});
```

### 2.2 组件测试（Testing Library）

```tsx
// components/PostCard.test.tsx
import { describe, it, expect } from 'vitest';
import { render, screen } from '@testing-library/react';
import { PostCard } from './PostCard';

describe('PostCard', () => {
  it('渲染标题和作者', () => {
    render(<PostCard title="理解缓存" author="Alice" />);
    expect(screen.getByText('理解缓存')).toBeDefined();
    expect(screen.getByText('Alice')).toBeDefined();
  });
});
```

### 2.3 E2E 测试（Playwright）

用 Playwright 覆盖关键用户流程：访问首页、查看文章、登录、发布文章：

```ts
// e2e/publish.spec.ts
import { test, expect } from '@playwright/test';

test('登录后可以发布文章', async ({ page }) => {
  await page.goto('/login');
  await page.getByLabel('邮箱').fill('alice@example.com');
  await page.getByLabel('密码').fill('password');
  await page.getByRole('button', { name: '登录' }).click();

  await page.goto('/posts/new');
  await page.getByLabel('标题').fill('新的文章');
  await page.getByLabel('正文').fill('这是一段足够长的正文内容。');
  await page.getByRole('button', { name: '发布' }).click();

  await expect(page).toHaveURL(/\/posts\/.+/);
});
```

### 2.4 测试策略

- 纯逻辑和校验：单元测试。
- 交互组件：组件测试。
- 登录、发布、权限等关键流程：E2E 测试。
- CI 中先跑 lint、typecheck、单测，再跑 E2E。

## 3. 配置与安全

### 3.1 安全响应头

```ts
// next.config.ts
import type { NextConfig } from 'next';

const nextConfig: NextConfig = {
  async headers() {
    return [
      {
        source: '/:path*',
        headers: [
          { key: 'X-Content-Type-Options', value: 'nosniff' },
          { key: 'X-Frame-Options', value: 'DENY' },
          { key: 'Referrer-Policy', value: 'strict-origin-when-cross-origin' },
        ],
      },
    ];
  },
};

export default nextConfig;
```

### 3.2 CORS（Route Handlers）

```ts
// app/api/posts/route.ts
export async function GET(request: Request) {
  const origin = request.headers.get('origin');
  const allowed = ['https://app.example.com'];

  if (origin && !allowed.includes(origin)) {
    return new Response('Forbidden', { status: 403 });
  }

  const response = Response.json(await getPosts());
  response.headers.set('Access-Control-Allow-Origin', allowed[0] ?? '*');
  return response;
}
```

### 3.3 环境变量校验

使用 Zod 校验必需的环境变量，缺失时在构建期报错：

```ts
// lib/env.ts
import { z } from 'zod';

const envSchema = z.object({
  DATABASE_URL: z.string().min(1),
  AUTH_SECRET: z.string().min(32),
  NEXT_PUBLIC_ANALYTICS_ID: z.string().optional(),
});

const env = envSchema.safeParse(process.env);

if (!env.success) {
  throw new Error(`环境变量缺失：${env.error.toString()}`);
}

export const env = env.data;
```

## 4. 部署

### 4.1 部署目标选择

| 平台 | 特点 |
|---|---|
| Vercel | 原生支持 App Router、ISR、缓存，自动部署 |
| 自有 Node.js 服务器 | 需要配置端口、环境变量、进程管理 |
| Docker | 适合已有容器化基础设施 |

### 4.2 Dockerfile

```dockerfile
FROM node:22-bookworm-slim AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM node:22-bookworm-slim
WORKDIR /app
ENV NODE_ENV=production
COPY package*.json ./
RUN npm ci --omit=dev
COPY --from=builder /app/.next ./.next
COPY --from=builder /app/public ./public
COPY --from=builder /app/next.config.ts ./next.config.ts
USER node
EXPOSE 3000
CMD ["npm", "start"]
```

### 4.3 部署检查清单

- `npm run build` 成功，路由静态/动态标记符合预期。
- 环境变量注入正确，敏感值不进入镜像。
- 数据库迁移在部署流程中执行。
- 健康检查就绪后才接流量。
- 回滚方案就绪，缓存与数据兼容。

## 5. 可观测性与故障恢复

### 5.1 结构化日志

Server Component 和 Server Actions 中的日志输出到服务端标准输出，接入日志采集：

```ts
export async function publishPost(formData: FormData) {
  const session = await getSession();

  console.info('post.publish.start', {
    userId: session?.user?.id,
    title: formData.get('title'),
    requestId: crypto.randomUUID(),
  });

  try {
    // 业务逻辑...
  } catch (error) {
    console.error('post.publish.failed', {
      error: error instanceof Error ? error.message : String(error),
    });
    return { error: '发布失败，请稍后重试' };
  }
}
```

### 5.2 错误监控

- `error.tsx` 捕获客户端渲染错误并上报。
- Server Action 返回结构化错误，不在响应中泄露堆栈。
- 接入错误监控平台，关联版本和 request ID。

### 5.3 回滚

- 保留可回滚的构建产物或镜像版本。
- 数据库迁移记录版本，支持降级脚本。
- 发布后先验证健康检查、核心指标和错误率，再放量。

## 6. 常见排错

| 现象 | 排查方向 |
|---|---|
| 生产页面与开发环境不一致 | 对比 `next build` 路由标记与缓存配置 |
| 首屏体积过大 | 检查 First Load JS 和 Client Component 数量 |
| 图片加载过慢 | 检查 `next/image` 的 sizes 与 priority |
| 构建失败 | 检查环境变量校验、类型错误和 ESLint |
| 部署后接口 404 | 确认 Route Handler 路径与方法 |
| 发布后看不到更新 | 检查缓存、ISR 和回滚配置 |

## 阶段四验收

- 能使用 `next/image`、`next/font` 和动态导入优化性能。
- 能编写单元、组件和 E2E 测试。
- 能配置安全响应头、CORS 和环境变量校验。
- 能完成生产构建、部署和回滚验收。
- 能设计结构化日志和错误监控。

## 动手任务

1. 优化首页图片和字体，测量 LCP 变化。
2. 为发布流程编写 E2E 测试。
3. 为敏感数据配置环境变量校验。
4. 完成生产构建并检查路由渲染标记。
5. 编写一份部署、监控和回滚检查清单。

## 进入下一阶段的条件

你能够完成生产构建、测试、部署和故障恢复方案。此时进入 [综合实战：内容平台](./project-practice.md)。
