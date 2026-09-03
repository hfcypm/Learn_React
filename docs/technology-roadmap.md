# 技术栈地图：按项目目标选择学习路线

**目标**：在 16 个学习包之间建立选择路径，根据你想构建的项目类型，确定学习顺序与组合，避免漫无目的地从头读起。

## 1. 三条主线

| 主线 | 目标项目 | 核心路线 |
|---|---|---|
| Web 应用 | 网站、SaaS、中后台、API 服务 | React -> Next.js -> Node.js/NestJS -> Prisma -> PostgreSQL |
| 桌面应用 | 本地工具、效率软件、跨平台客户端 | React -> TypeScript -> Electron 或 Tauri |
| AI 应用 | Chat、Agent、知识库问答 | Next.js -> AI 应用前端 -> LangChain |

## 2. Web 应用主线

```text
React（界面） -> TypeScript（类型） -> Next.js（全栈框架）
   -> Tailwind CSS + shadcn/ui（样式与组件）
   -> Node.js/NestJS（后端）
   -> Prisma + PostgreSQL（数据层）
```

- 前端入门：[React 学习路线](./react-learning-path/README.md) -> [TypeScript 学习路线](./typescript-learning-path/README.md) -> [Tailwind CSS 学习路线](./tailwindcss-learning-path/README.md)。
- 界面组件：[shadcn/ui 学习路线](./shadcn-learning-path/README.md) 基于 Tailwind CSS 的源码分发组件，快速搭出专业中后台界面。
- 全栈进阶：[Next.js 学习路线](./nextjs-learning-path/README.md) 覆盖 App Router、渲染、缓存与鉴权。
- 后端选型：小服务用 [Node.js 学习路线](./nodejs-learning-path/README.md)；企业级模块化用 [NestJS 学习路线](./nestjs-learning-path/README.md)。两条路线可对照 [服务端学习计划](./server-learning-study-plan.md) 的 8 周路径串行学习。
- 数据层：[PostgreSQL 学习路线](./postgresql-learning-path/README.md) 掌握 SQL 与数据库原理，再进入 [Prisma 学习路线](./prisma-learning-path/README.md) 用 ORM 建模与查询。
- 中后台界面：React + Tailwind CSS 之后进入 [shadcn/ui 学习路线](./shadcn-learning-path/README.md)，其综合实战 [订阅管理后台](./shadcn-learning-path/project-practice.md) 串起主题、表单、表格与图表。
- 综合验证：[综合实战：团队任务看板](./fullstack-kanban/README.md) 把 Web 主线四条路线串成一个应用。

## 3. 桌面应用主线

```text
React（界面） -> TypeScript（类型）-> Electron 或 Tauri（桌面壳）
```

- 前端基础与 Web 主线共用，先掌握 React 与 TypeScript。
- 框架选型：[桌面框架选型：Electron vs Tauri](./electron-vs-tauri.md) 根据体积、性能、安全、团队语言与移动端需求决策。
- 选 Electron：[Electron 学习路线](./electron-learning-path/README.md)（JS/TS 为主，生态成熟）。
- 选 Tauri：[Tauri 学习路线](./tauri-learning-path/README.md)（Rust 后端，默认安全，支持移动端）。

## 4. AI 应用主线

```text
Next.js（前端与路由）-> AI 应用前端（交互）
   -> LangChain（模型、RAG、Agent）
```

- [AI 应用前端学习路线](./ai-learning-path/README.md) 掌握 AI 界面的数据请求、流式响应与内容渲染。
- [LangChain 学习路线](./langchain-learning-path/README.md) 掌握服务端的模型接入、检索增强与 Agent。
- 两条路线互补：前端消费流式接口，LangChain 生成流式响应，综合在 [知识库客服机器人](./langchain-learning-path/project-practice.md) 中闭环。
- 前置建议：先完成 Next.js 与 TypeScript，再进入 AI 主线。

## 5. 交叉组合

| 你要做的产品 | 路线组合 | 综合实战 |
|---|---|---|
| 传统 Web 应用 | Web 主线全部 | [团队任务看板](./fullstack-kanban/README.md) |
| 桌面工具 | React + TypeScript + Electron/Tauri | [桌面笔记](./electron-learning-path/project-practice.md) 或 [文件管理器](./tauri-learning-path/project-practice.md) |
| AI 聊天/知识库 | Next.js + AI 前端 + LangChain | [知识库客服机器人](./langchain-learning-path/project-practice.md) |
| AI + 桌面客户端 | AI 主线 + Electron/Tauri | 可选：在桌面壳中嵌入 AI 前端 |

## 6. 通用建议

- 三条主线都以 React 与 TypeScript 为共同底座，先夯实这两项再进入分支。
- 数据库原理（PostgreSQL）建议在任何数据相关路线之前掌握，避免被 ORM 掩盖底层行为。
- 每个综合实战都是多路线知识的验收场，完成主线后再做对应实战。
- 不确定选型时，先看 [Electron vs Tauri](./electron-vs-tauri.md) 这类对比文档，再做最小原型验证。

## 7. 路线清单

| 领域 | 路线 |
|---|---|
| 语言与类型 | [React](./react-learning-path/README.md)、[TypeScript](./typescript-learning-path/README.md) |
| Web 框架 | [Next.js](./nextjs-learning-path/README.md)、[Tailwind CSS](./tailwindcss-learning-path/README.md)、[shadcn/ui](./shadcn-learning-path/README.md) |
| 服务端 | [Node.js](./nodejs-learning-path/README.md)、[NestJS](./nestjs-learning-path/README.md) |
| 数据层 | [PostgreSQL](./postgresql-learning-path/README.md)、[Prisma](./prisma-learning-path/README.md) |
| 桌面 | [Electron](./electron-learning-path/README.md)、[Tauri](./tauri-learning-path/README.md) |
| AI | [AI 应用前端](./ai-learning-path/README.md)、[LangChain](./langchain-learning-path/README.md) |
| 综合 | [团队任务看板](./fullstack-kanban/README.md)、[服务端学习计划](./server-learning-study-plan.md) |
