# 综合实战：Next.js + Tailwind CSS + Prisma + PostgreSQL 团队看板

**目标**：用一条完整的学习路径串联四条学习路线（[Next.js](../nextjs-learning-path/README.md)、[Tailwind CSS](../tailwindcss-learning-path/README.md)、[Prisma](../prisma-learning-path/README.md)、[PostgreSQL](../postgresql-learning-path/README.md)）的全部知识点，构建一个可运行、可部署的团队任务看板应用。

## 1. 项目是什么

一个多用户团队看板（Kanban），支持：

- 用户注册、登录与权限（管理员/成员）。
- 团队与项目，项目下多个看板。
- 任务卡片，支持状态流转、优先级、标签、指派与截止日期。
- 评论、活动日志、任务统计与全文搜索。
- 响应式布局与暗色模式。

```text
注册登录 -> 创建团队 -> 创建项目 -> 建看板 -> 建任务
    -> 拖动/流转任务状态 -> 添加标签与指派 -> 评论与活动日志
    -> 统计与搜索 -> 生产部署
```

## 2. 技术栈

| 技术 | 版本方向 | 用途 |
|---|---|---|
| Next.js | App Router | 路由、Server/Client 组件、Server Actions、缓存 |
| Tailwind CSS | v4 CSS-first | 样式、响应式、暗色模式、组件变体 |
| Prisma | v7 + driver adapter | ORM、Schema、迁移、类型安全查询 |
| PostgreSQL | 17 | 数据存储、索引、事务、全文搜索 |

## 3. 知识点映射

| 文档章节 | 覆盖知识点 | 对应学习路线 |
|---|---|---|
| [01 初始化](./01-setup.md) | create-next-app、Tailwind v4 接入、Prisma 初始化、driver adapter、连接串 | Next 阶段零、Tailwind 阶段零、Prisma 阶段零、PostgreSQL 阶段零 |
| [02 数据建模](./02-schema.md) | Prisma Schema、关系、枚举、约束、@db 原生类型、索引、迁移、seed | Prisma 阶段一、二、PostgreSQL 阶段二 |
| [03 鉴权](./03-auth.md) | Server Actions、Zod 校验、会话、中间件、角色权限 | Next 阶段三 |
| [04 数据获取](./04-data-fetching.md) | Server Component 数据获取、ISR、revalidatePath、Route Handler、缓存 | Next 阶段二 |
| [05 看板界面](./05-board-ui.md) | Tailwind 布局、响应式、暗色模式、组件变体、@theme、动效、无障碍 | Tailwind 阶段一至四 |
| [06 查询与事务](./06-advanced.md) | Prisma Client CRUD、关系查询、分页、事务、聚合、PostgreSQL 索引与 EXPLAIN | Prisma 阶段三、PostgreSQL 阶段一、三 |
| [07 生产交付](./07-deployment.md) | 生产迁移、连接池、日志、监控、备份、部署 | Prisma 阶段四、PostgreSQL 阶段四、Next 阶段四 |

## 4. 学习建议

- 每个章节先跑通代码，再对照「知识点映射」回到对应路线补深。
- 建议顺序：01 -> 02 -> 03 -> 04 -> 05 -> 06 -> 07，与四条路线的阶段递增一致。
- 每章结尾有验收与排错，全部通过后再进入下一章。

## 5. 目录结构

```text
docs/fullstack-kanban/
├── README.md              # 本文件：总览与映射
├── 01-setup.md            # 项目初始化
├── 02-schema.md           # 数据建模
├── 03-auth.md             # 鉴权
├── 04-data-fetching.md    # 数据获取与缓存
├── 05-board-ui.md         # 看板界面
├── 06-advanced.md         # 查询与事务
└── 07-deployment.md       # 生产交付
```

## 6. 跨端延伸：从 Web 到桌面

看板应用是 Web 全栈，完成后可向桌面端延伸，复用 React、TypeScript 与状态管理经验：

- **用 Electron 桌面化**：看板界面 + IPC 换成 [Electron 学习路线](../electron-learning-path/README.md) 的 preload/contextBridge 模式，数据层可在本地（SQLite/文件）或仍走远程 API。
- **用 Tauri 桌面化**：看板界面 + Rust 命令换成 [Tauri 学习路线](../tauri-learning-path/README.md)，把看板的数据库与业务逻辑迁入 Rust 命令，获得小体积与默认安全。
- **选型**：桌面化前先读 [桌面框架选型：Electron vs Tauri](../electron-vs-tauri.md)，按体积、性能、团队语言与移动端需求决策。

跨端桥接的共性：界面层（React + Tailwind）几乎零改动，主要迁移点是数据访问——Web 走 HTTP/Prisma，桌面走 IPC/命令。
