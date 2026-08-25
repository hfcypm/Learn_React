# 现代 React 与版本边界

**目标**：建立 React 18、React 19、客户端渲染、服务端渲染和 Server Components 的准确认知，减少 API 版本混用。

## 1. React 18 核心能力

### 根节点 API

新项目使用 `createRoot` 创建客户端根节点：

```tsx
import { createRoot } from 'react-dom/client';
import App from './App';

createRoot(document.getElementById('root')!).render(<App />);
```

服务端生成 HTML 后使用 `hydrateRoot` 接管已有 DOM：

```tsx
import { hydrateRoot } from 'react-dom/client';
import App from './App';

hydrateRoot(document.getElementById('root')!, <App />);
```

### 并发相关 API

- `startTransition`：将更新标记为非紧急更新。
- `useTransition`：读取 transition 的 pending 状态。
- `useDeferredValue`：延迟低优先级值，保持输入响应。
- `Suspense`：声明异步边界和 fallback 展示。
- 自动批处理：React 18 对更多异步场景中的状态更新进行批处理。

并发渲染强调可中断和优先级调度。它不会自动让所有计算变快，昂贵计算仍需要数据结构、分页、虚拟化和代码分割配合。

## 2. React 19 及现代 React

学习 React 19 时应结合具体框架和运行环境验证 API。重点主题包括：

- `useActionState`：管理 action 的返回状态、错误和 pending 状态。
- `useFormStatus`：在表单子树中读取提交状态。
- `useOptimistic`：在请求完成前展示乐观 UI。
- `use`：读取 Promise 或 Context，通常与 Suspense 和服务端能力一起使用。
- Server Actions：由框架提供服务端函数调用能力，需明确序列化、鉴权和部署边界。
- React Compiler：通过编译阶段优化减少部分手动 memo 需求，使用前需要确认项目工具链支持情况。

### 学习要求

- 标明示例所需 React 版本。
- 说明 API 是 React 核心能力还是框架能力。
- 为 pending、成功、失败、取消和重复提交设计状态。
- 不把实验性 API 当作通用浏览器能力。

## 3. Suspense 与错误边界

Suspense 负责加载边界，Error Boundary 负责渲染错误。生产页面通常需要同时设计两种边界：

```tsx
<ErrorBoundary fallback={<ErrorPage />}>
  <Suspense fallback={<PageSkeleton />}>
    <UserPage />
  </Suspense>
</ErrorBoundary>
```

需要覆盖：

- 路由级 fallback
- 组件级 fallback
- 数据请求失败后的重试
- 错误上报与 request ID
- 用户可理解的错误提示
- 错误边界的恢复和重新挂载

## 4. SSR、SSG、ISR 与 RSC

| 模式 | HTML 生成时间 | 适用场景 |
|---|---|---|
| CSR | 浏览器运行时 | 交互密集、SEO 要求低的应用 |
| SSR | 每次请求 | 个性化、实时性较强的页面 |
| SSG | 构建时 | 内容稳定、访问量高的页面 |
| ISR | 构建后按策略更新 | 内容变化可控且需要缓存的页面 |
| RSC | 服务端执行组件逻辑 | 减少客户端 JS、靠近数据源 |

### RSC 边界

- Server Component 可以在服务端读取数据和组合 UI。
- Client Component 通过 `use client` 声明客户端边界。
- 客户端组件适合事件处理、浏览器 API 和交互状态。
- 服务端与客户端之间传递的数据需要可序列化。
- 权限校验必须放在服务端数据访问层。
- RSC 属于框架运行模型，不能简单等同于传统 SSR。

### Next.js 学习顺序

1. App Router、布局、页面、动态路由和 loading/error 文件。
2. Server Component 与 Client Component 边界。
3. 服务端数据获取、缓存和重新验证。
4. 表单 action、鉴权、metadata 和部署。
5. 旧项目中的 Pages Router 与 `getStaticProps`、`getServerSideProps` 维护。

## 5. 现代 React 验收项目

实现一个服务端渲染的内容站点：

- 首页使用静态生成或增量更新。
- 详情页使用动态路由和 metadata。
- 搜索页展示 pending、empty、error 三种状态。
- 点赞操作使用乐观更新并支持失败回滚。
- 服务端完成权限校验，客户端只负责交互。
- 使用 Lighthouse 和 Playwright 验证性能与关键流程。
