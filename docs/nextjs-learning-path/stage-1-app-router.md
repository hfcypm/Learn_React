# 阶段一：App Router 路由与渲染

**目标**：掌握 App Router 的文件系统路由、Server/Client 组件边界、布局、加载态和错误边界。

## 1. 路由基础

App Router 中 URL 由 `app/` 目录下的文件夹结构决定。文件夹创建路由，特殊文件定义路由行为。

### 1.1 路由与文件约定

| 路径 | URL |
|---|---|
| `app/page.tsx` | `/` |
| `app/about/page.tsx` | `/about` |
| `app/posts/[id]/page.tsx` | `/posts/:id` |
| `app/posts/[id]/comments/page.tsx` | `/posts/:id/comments` |
| `app/blog/[...slug]/page.tsx` | `/blog/...`（捕获多段） |

```tsx
// app/posts/[id]/page.tsx —— Server Component 直接 await params
export default async function PostPage({
  params,
}: {
  params: Promise<{ id: string }>;
}) {
  const { id } = await params;
  return <h1>文章 {id}</h1>;
}
```

### 1.2 Link 与客户端导航

使用 `next/link` 做客户端导航，预取相邻路由，避免整页刷新：

```tsx
import Link from 'next/link';

export default function Nav() {
  return (
    <nav>
      <Link href="/" prefetch={false}>首页</Link>
      <Link href="/posts">文章</Link>
      <Link href="/posts/1">第一篇</Link>
    </nav>
  );
}
```

需要编程式导航时使用 `useRouter`：

```tsx
'use client';

import { useRouter } from 'next/navigation';

export default function BackButton() {
  const router = useRouter();
  return <button onClick={() => router.push('/posts')}>返回列表</button>;
}
```

`<Link>` 默认预取视口内的链接。不需要预取时设置 `prefetch={false}`，减少不必要的请求。

## 2. Server Component 与 Client Component

### 2.1 默认是 Server Component

App Router 中所有组件默认在服务端执行，没有 `"use client"` 指令时即为 Server Component。Server Component 可以读取文件系统、数据库和环境变量，不把代码发送到浏览器。

```tsx
// Server Component：直接读取数据
import { getPosts } from '@/lib/posts';

export default async function PostsPage() {
  const posts = await getPosts();
  return <ul>{posts.map((p) => <li key={p.id}>{p.title}</li>)}</ul>;
}
```

### 2.2 需要交互时标记 Client Component

文件顶部加 `"use client"`，该文件及其子组件在客户端执行：

```tsx
'use client';

import { useState } from 'react';

export default function Counter() {
  const [count, setCount] = useState(0);
  return <button onClick={() => setCount(count + 1)}>计数 {count}</button>;
}
```

`"use client"` 是文件边界而非组件边界。一个文件标记后，文件内所有组件都会进入客户端渲染。

### 2.3 选择规则

| 场景 | 使用 |
|---|---|
| 读取数据、调用数据库、读环境变量 | Server Component |
| 处理 onClick、useState、useEffect | Client Component |
| 使用浏览器 API（window、document） | Client Component |
| 无交互的展示组件 | Server Component（默认） |

避免在 Client Component 中发起大量数据请求，优先把数据获取放在 Server Component 并向下传递。

## 3. 布局、嵌套与并行

### 3.1 嵌套布局

`layout.tsx` 包裹同一目录下所有页面，并在导航时保持状态：

```tsx
// app/posts/layout.tsx
import Link from 'next/link';

export default function PostsLayout({ children }: { children: React.ReactNode }) {
  return (
    <section>
      <aside>
        <Link href="/posts?tag=all">全部</Link>
        <Link href="/posts?tag=react">React</Link>
      </aside>
      {children}
    </section>
  );
}
```

布局是嵌套的：访问 `/posts/1` 时，根布局、posts 布局和页面会一起渲染。布局不会在切换子路由时重新渲染，因此适合放导航和持久 UI。

### 3.2 模板

`template.tsx` 与布局相似，但在每次导航时重新创建实例。适合需要在切换时重置状态的 UI，例如表单和滚动容器。

### 3.3 并行路由

`@slot` 文件夹定义并行路由，多个区域可独立加载：

```text
app/
├── layout.tsx
├── @analytics/page.tsx      # 分析面板
├── @feed/page.tsx           # 信息流
└── page.tsx                 # 主内容
```

```tsx
// app/layout.tsx
export default function DashboardLayout({
  children,
  analytics,
  feed,
}: {
  children: React.ReactNode;
  analytics: React.ReactNode;
  feed: React.ReactNode;
}) {
  return (
    <div>
      {children}
      <aside>{analytics}</aside>
      <aside>{feed}</aside>
    </div>
  );
}
```

并行路由常用于仪表盘、多栏布局和独立加载区域。

## 4. loading、error 与 not-found

### 4.1 加载态

`loading.tsx` 在页面等待异步数据时展示，自动包裹在 Suspense 中：

```tsx
// app/posts/loading.tsx
export default function PostsLoading() {
  return <div>正在加载文章...</div>;
}
```

`loading.tsx` 是路由级的，也可以直接用 Suspense 控制更细粒度的加载边界：

```tsx
import { Suspense } from 'react';
import { PostList } from './PostList';

export default function Page() {
  return (
    <Suspense fallback={<div>加载列表中...</div>}>
      <PostList />
    </Suspense>
  );
}
```

### 4.2 错误边界

`error.tsx` 是 Client Component，捕获渲染过程中的错误并展示恢复 UI：

```tsx
'use client';

export default function ErrorBoundary({
  error,
  reset,
}: {
  error: Error & { digest?: string };
  reset: () => void;
}) {
  return (
    <div>
      <h2>出错了</h2>
      <p>{error.message}</p>
      <button onClick={reset}>重试</button>
    </div>
  );
}
```

### 4.3 404

`not-found.tsx` 在页面调用 `notFound()` 或路由不存在时展示：

```tsx
// app/not-found.tsx
import Link from 'next/link';

export default function NotFound() {
  return (
    <div>
      <h2>页面不存在</h2>
      <Link href="/">返回首页</Link>
    </div>
  );
}
```

```tsx
// 页面内主动触发 404
import { notFound } from 'next/navigation';

export default async function PostPage({ params }) {
  const { id } = await params;
  const post = await getPost(id);

  if (!post) {
    notFound();
  }

  return <article>{post.title}</article>;
}
```

## 5. 动态路由进阶

### 5.1 捕获多段路由

```text
app/
└── blog/
    └── [...slug]/page.tsx   # /blog/a、/blog/a/b 都能匹配
```

```tsx
export default async function BlogPage({
  params,
}: {
  params: Promise<{ slug: string[] }>;
}) {
  const { slug } = await params;
  return <h1>博客路径：{slug.join(' / ')}</h1>;
}
```

### 5.2 可选捕获

`[[...slug]]` 表示可选捕获，`/blog` 也能匹配。

### 5.3 查询参数

```tsx
export default async function SearchPage({
  searchParams,
}: {
  searchParams: Promise<{ q?: string; page?: string }>;
}) {
  const { q = '', page = '1' } = await searchParams;
  return <h1>搜索 {q}，第 {page} 页</h1>;
}
```

在 Server Component 中读取 `searchParams` 需要 `await`；在 Client Component 中使用 `useSearchParams`。

## 6. 元数据与 SEO

### 6.1 静态元数据

```tsx
// app/posts/[id]/page.tsx
import type { Metadata } from 'next';

export const metadata: Metadata = {
  title: '文章详情',
  description: '单篇文章页面',
  openGraph: {
    title: '文章详情',
    description: '分享给社交平台',
  },
};
```

### 6.2 动态元数据

数据依赖动态值时使用 `generateMetadata`：

```tsx
import type { Metadata } from 'next';

export async function generateMetadata({
  params,
}: {
  params: Promise<{ id: string }>;
}): Promise<Metadata> {
  const { id } = await params;
  const post = await getPost(id);

  return {
    title: post?.title ?? '文章不存在',
    description: post?.excerpt,
  };
}
```

## 7. 常见排错

| 现象 | 排查方向 |
|---|---|
| 点击链接整页刷新 | 使用 `Link` 而非 `<a>` |
| Server Component 用了事件处理 | 拆成 `"use client"` 文件 |
| 页面渲染两次或状态丢失 | 检查是否误标记 `"use client"` |
| 布局随路由变化重置 | 布局切换时用 `template.tsx` |
| `params` 为 undefined | 确认 Next.js 15+ 用 `await` 读取 |
| 404 页面不显示 | 确认 `notFound()` 在渲染期间调用 |

## 阶段一验收

- 能创建嵌套路由、动态路由和捕获路由。
- 能区分 Server Component 与 Client Component，并正确标记边界。
- 能配置布局、loading、error 和 not-found 边界。
- 能使用 `generateMetadata` 生成动态 SEO 元数据。

## 动手任务

1. 为内容站点添加文章详情、分类和搜索路由。
2. 拆分一个交互组件为 Client Component。
3. 为列表页添加 loading 和 error 边界。
4. 使用嵌套布局组织侧边导航。
5. 为文章详情页添加动态元数据。

## 进入下一阶段的条件

你能够解释 App Router 的路由、布局和渲染边界，并能独立组织多页面站点的路由。此时进入 [阶段二：数据获取、缓存与重新验证](./stage-2-data-fetching.md)。
