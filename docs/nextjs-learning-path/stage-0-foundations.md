# 阶段零：Next.js 基础与项目初始化

**目标**：理解 Next.js 的框架定位、目录约定和开发流程，能够创建并运行一个基础项目。

## 1. Next.js 解决什么问题

React 只负责组件渲染，路由、数据获取、代码分割、构建优化和部署都需要自己搭建。Next.js 是一个 React 框架，内置文件系统路由、渲染模式、数据获取与缓存、优化能力和构建工具，让你专注于产品功能。

### 1.1 框架提供的能力

| 能力 | React 单独做 | Next.js 内置 |
|---|---|---|
| 路由 | React Router 或手写 | App Router 文件系统路由 |
| 渲染 | CSR，需要自己配 | SSR、SSG、ISR、RSC |
| 数据获取 | 自己封装 | Server Component、Route Handler |
| 图片优化 | 手写 lazy、srcset | `next/image` |
| 字体优化 | 手动接入 | `next/font` |
| 代码分割 | 配置动态导入 | 路由级自动分割 |
| 构建部署 | 自己配工具链 | `next build`、`next start` |

### 1.2 与纯 React 项目的区别

纯 React 项目（Vite）产出一份静态 HTML，浏览器运行 JavaScript 后渲染内容。Next.js 可以在服务器上完成渲染，生成完整的 HTML 交给浏览器，同时保留客户端交互能力。这带来的核心价值是更快的首屏、更好的 SEO 和更少的前端数据请求。

## 2. 创建项目

### 2.1 create-next-app

使用官方脚手架创建 TypeScript + App Router 项目：

```bash
# 创建项目（交互式选择配置）
npx create-next-app@latest my-app

# 指定核心配置（TypeScript + App Router + Tailwind + ESLint）
npx create-next-app@latest my-app --typescript --app --tailwind --eslint
```

```bash
# 进入项目并启动开发服务器
cd my-app
npm run dev
```

访问 http://localhost:3000 查看默认页面。`npm run dev` 启动开发服务器，支持热更新和错误提示。

### 2.2 常用脚本

| 命令 | 作用 |
|---|---|
| `npm run dev` | 开发服务器，带热更新 |
| `npm run build` | 生产构建 |
| `npm start` | 运行生产构建产物 |
| `npm run lint` | 运行 ESLint |
| `npm run typecheck` | 运行 TypeScript 检查（`tsc --noEmit`） |

## 3. 项目目录结构

```
my-app/
├── app/
│   ├── layout.tsx          # 根布局，包裹所有页面
│   ├── page.tsx            # 首页路由
│   └── globals.css         # 全局样式
├── public/                 # 静态资源
├── components/             # 业务组件
├── lib/                    # 工具与数据访问
├── next.config.ts          # Next.js 配置
├── package.json
└── tsconfig.json
```

### 3.1 App Router 目录约定

App Router 中，`app/` 目录下的文件和文件夹映射到 URL。文件名决定路由行为：

| 文件约定 | 作用 |
|---|---|
| `layout.tsx` | 共享布局，嵌套路由时保持渲染 |
| `page.tsx` | 页面组件，定义路由内容 |
| `loading.tsx` | 加载态 UI（配合 Suspense） |
| `error.tsx` | 错误边界 UI |
| `not-found.tsx` | 404 UI |
| `route.ts` | Route Handler，定义 API 端点 |
| `[param]/` | 动态路由段 |

## 4. 根布局与首页

### 4.1 根布局

`app/layout.tsx` 是所有页面的外壳，必须包含 `<html>` 和 `<body>`：

```tsx
// app/layout.tsx
import type { Metadata } from 'next';
import './globals.css';

export const metadata: Metadata = {
  title: '内容平台',
  description: 'Next.js 学习项目',
};

export default function RootLayout({
  children,
}: Readonly<{ children: React.ReactNode }>) {
  return (
    <html lang="zh-CN">
      <body>{children}</body>
    </html>
  );
}
```

### 4.2 首页

`app/page.tsx` 映射到 `/` 路径：

```tsx
// app/page.tsx
export default function HomePage() {
  return (
    <main>
      <h1>欢迎使用 Next.js</h1>
      <p>这是服务端渲染的页面。</p>
    </main>
  );
}
```

保存文件后，浏览器会立即显示更新后的内容，这是开发服务器的热更新能力。

## 5. 开发与构建流程

### 5.1 生产构建

```bash
# 构建生产产物
npm run build
```

构建输出会显示每条路由的渲染方式（Static 或 Dynamic），这是理解 Next.js 缓存行为的重要入口：

```text
Route (app)                              Size     First Load JS
┌ ○ /                                    1.2 kB   89.3 kB
└ ○ /_not-found                          875 B    87.2 kB
○  (Static)  prerendered as static content
```

### 5.2 环境变量

在项目根目录创建 `.env.local`（git 忽略），通过 `NEXT_PUBLIC_` 前缀决定变量是否暴露给浏览器：

```bash
# .env.local（仅服务端可见）
DATABASE_URL=postgres://user:password@localhost:5432/blog

# 暴露给浏览器
NEXT_PUBLIC_API_BASE=http://localhost:3000
```

```ts
// 服务端读取
const databaseUrl = process.env.DATABASE_URL;

// 浏览器读取（带 NEXT_PUBLIC_ 前缀）
const apiBase = process.env.NEXT_PUBLIC_API_BASE;
```

不要把真实密钥提交到仓库，使用 `.env.example` 保存占位符。

## 6. 最小案例：多页面站点骨架

创建一个包含首页、关于页和动态文章页的最小站点：

```text
app/
├── layout.tsx
├── page.tsx                # /
├── about/page.tsx          # /about
└── posts/
    ├── page.tsx            # /posts 文章列表
    └── [id]/page.tsx       # /posts/:id 文章详情
```

```tsx
// app/layout.tsx
import Link from 'next/link';

export const metadata = {
  title: '内容平台',
};

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="zh-CN">
      <body>
        <nav>
          <Link href="/">首页</Link>
          <Link href="/about">关于</Link>
          <Link href="/posts">文章</Link>
        </nav>
        {children}
      </body>
    </html>
  );
}
```

```tsx
// app/about/page.tsx
export default function AboutPage() {
  return (
    <main>
      <h1>关于我们</h1>
      <p>这个站点用于学习 Next.js 的路由与渲染。</p>
    </main>
  );
}
```

```tsx
// app/posts/page.tsx
import Link from 'next/link';

const posts = [
  { id: '1', title: '理解 App Router' },
  { id: '2', title: 'Server Component 入门' },
];

export default function PostsPage() {
  return (
    <main>
      <h1>文章列表</h1>
      <ul>
        {posts.map((post) => (
          <li key={post.id}>
            <Link href={`/posts/${post.id}`}>{post.title}</Link>
          </li>
        ))}
      </ul>
    </main>
  );
}
```

```tsx
// app/posts/[id]/page.tsx
import { notFound } from 'next/navigation';

const posts = [
  { id: '1', title: '理解 App Router', body: 'App Router 基于文件系统路由。' },
  { id: '2', title: 'Server Component 入门', body: 'Server Component 在服务端执行。' },
];

export default async function PostPage({ params }: { params: Promise<{ id: string }> }) {
  const { id } = await params;
  const post = posts.find((item) => item.id === id);

  if (!post) {
    notFound();
  }

  return (
    <main>
      <h1>{post.title}</h1>
      <p>{post.body}</p>
    </main>
  );
}
```

> 说明：Next.js 15+ 中 `params` 和 `searchParams` 是 Promise。Server Component 中直接 `await params`；Client Component 中通过 `use(params)` 读取。

## 7. 常见排错

| 现象 | 排查方向 |
|---|---|
| 页面 404 | 确认 `page.tsx` 文件名和目录路径 |
| 热更新不生效 | 确认开发服务器运行，浏览器访问 3000 端口 |
| 样式不生效 | 确认 `globals.css` 已导入 |
| 路由不更新 | 确认新页面文件已保存，必要时重启 dev |
| 环境变量 undefined | 确认变量前缀和 `.env.local` 位置 |

## 阶段零验收

- 能使用 `create-next-app` 创建项目并启动开发服务器。
- 能解释 App Router 中 `layout.tsx` 和 `page.tsx` 的作用。
- 能创建静态多页面站点并正确链接。
- 能区分服务端与浏览器端环境变量的读取方式。
- 能理解 `npm run build` 输出的路由渲染标记。

## 动手任务

1. 创建项目并启动开发服务器。
2. 添加 `/about` 和 `/contact` 页面。
3. 添加一个包含多个链接的导航布局。
4. 添加一个动态路由页面，访问不存在的 id 时返回 404。
5. 在 `.env.local` 配置一个 `NEXT_PUBLIC_` 变量并在页面显示。

## 进入下一阶段的条件

你能够解释 App Router 目录到 URL 的映射关系，并能独立创建多页面站点。此时进入 [阶段一：App Router 路由与渲染](./stage-1-app-router.md)。
