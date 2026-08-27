# Next.js 学习路线图（基础 → 生产）

**目标**：掌握 Next.js 的 App Router、渲染模式、数据获取与缓存、Server Actions、鉴权和生产部署，能够构建可缓存、可测试、可部署的全栈 React 应用。

## 先看这里

- [新手导读](./start-here.md)：前置条件、学习顺序、排错顺序和完成标准。
- [综合实战：内容平台](./project-practice.md)：按阶段逐步扩展同一个项目。
- [学习评估与项目评分](./learning-assessment.md)：阶段考核、排错题、项目评分和评审问题。

## 学习阶段总览

| 阶段 | 名称 | 目标 |
|---|---|---|
| 零 | Next.js 基础与项目初始化 | 理解框架定位、目录约定和开发流程 |
| 一 | App Router 路由与渲染 | 掌握文件系统路由、Server/Client 组件和布局 |
| 二 | 数据获取、缓存与重新验证 | 掌握服务端数据获取和缓存失效策略 |
| 三 | Server Actions、表单与鉴权 | 掌握变更、表单处理、鉴权和权限边界 |
| 四 | 性能、工程化与生产交付 | 掌握性能优化、测试、部署和可观测性 |

## 快速导航

- [阶段零：Next.js 基础与项目初始化](./stage-0-foundations.md)
- [阶段一：App Router 路由与渲染](./stage-1-app-router.md)
- [阶段二：数据获取、缓存与重新验证](./stage-2-data-fetching.md)
- [阶段三：Server Actions、表单与鉴权](./stage-3-forms-auth.md)
- [阶段四：性能、工程化与生产交付](./stage-4-production.md)
- [综合实战：内容平台](./project-practice.md)

## 学习原则

- 默认使用 App Router，旧项目中的 Pages Router 只在维护场景使用。
- 尽量让组件默认在服务端渲染，只在需要交互时标记 `"use client"`。
- 数据请求放在服务端，减少客户端往返和重复请求。
- 理解缓存默认行为，再决定何时使用 `revalidatePath`、`revalidateTag` 和动态渲染。
- 变更通过 Server Actions 或 Route Handlers 完成，并校验输入与权限。
- 生产环境优先使用受支持的 Node.js LTS 与 Next.js 稳定版本。

## 阶段成果

- 能使用 App Router 组织页面、布局、嵌套路由和动态路由。
- 能区分 Server Component 与 Client Component，并正确选择渲染模式。
- 能设计数据获取、缓存、重新验证和 ISR 策略。
- 能通过 Server Actions 完成表单提交和数据变更，并校验输入与权限。
- 能完成性能优化、测试、部署、监控和回滚。

## 每阶段固定学习模板

每个阶段按照“目标 -> 前置知识 -> 核心问题 -> 最小示例 -> 项目增量 -> 验收 -> 排错”执行。先完成阶段项目，再阅读扩展主题；这样能够把框架 API 连接到可运行结果。

## 版本边界

学习主线对应 Next.js App Router 与 React 19。示例默认使用 `next@latest` 安装的稳定版本，具体版本以项目 `package.json` 锁定文件为准。Pages Router 旧项目维护相关主题参见 [版本边界与迁移](./version-governance.md)。

## 官方资源

- [Next.js 官方文档](https://nextjs.org/docs)
- [Next.js Learn](https://nextjs.org/learn)
- [Next.js GitHub](https://github.com/vercel/next.js)
