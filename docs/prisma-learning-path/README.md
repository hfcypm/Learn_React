# Prisma 学习路线图（基础 → 生产）

**目标**：掌握 Prisma ORM 的 schema 建模、迁移工作流、类型安全查询 API 和生产工程化，能够在 TypeScript/Node.js 项目中高效、安全地访问数据库。

## 先看这里

- [新手导读](./start-here.md)：前置条件、学习顺序、排错顺序和完成标准。
- [综合实战：博客 API](./project-practice.md)：按阶段逐步扩展同一个项目。
- [学习评估与项目评分](./learning-assessment.md)：阶段考核、排错题、项目评分和评审问题。

## 学习阶段总览

| 阶段 | 名称 | 目标 |
|---|---|---|
| 零 | Prisma 基础与工具链 | 理解 ORM 定位，能初始化项目并生成客户端 |
| 一 | Schema 数据建模 | 掌握模型、关系、枚举、约束和原生类型 |
| 二 | 迁移工作流 | 掌握 migrate dev、deploy、reset 与冲突处理 |
| 三 | Prisma Client 查询 | 掌握 CRUD、关系查询、筛选分页和事务 |
| 四 | 工程化与生产交付 | 掌握类型安全、框架集成、连接池和部署 |

## 快速导航

- [阶段零：Prisma 基础与工具链](./stage-0-foundations.md)
- [阶段一：Schema 数据建模](./stage-1-schema-modeling.md)
- [阶段二：迁移工作流](./stage-2-migrations.md)
- [阶段三：Prisma Client 查询](./stage-3-prisma-client.md)
- [阶段四：工程化与生产交付](./stage-4-production.md)
- [综合实战：博客 API](./project-practice.md)

## 学习原则

- Schema 是唯一事实来源，模型与数据库结构以 Schema 为准。
- 迁移驱动开发：每次模型变更生成一次迁移，保持可追溯。
- 查询优先用 Prisma Client 的类型安全 API，少写手写 SQL。
- 关系查询注意 N+1 问题，用 include 和 select 控制加载。
- 写操作用事务保证一致性，交互式事务用于复杂流程。
- 生产环境迁移用 `migrate deploy`，连接用连接池。

## 阶段成果

- 能初始化 Prisma 项目并连接 PostgreSQL。
- 能用 Schema 定义模型、关系和约束。
- 能用迁移管理数据库结构变更。
- 能用 Prisma Client 完成增删改查与关系查询。
- 能用事务保证数据一致并优化查询性能。

## 每阶段固定学习模板

每个阶段按照“目标 -> 前置知识 -> 核心问题 -> 最小示例 -> 项目增量 -> 验收 -> 排错”执行。先完成阶段项目，再阅读扩展主题；这样能够把 Prisma 概念连接到可运行结果。

## 版本边界

学习主线对应 Prisma 7 与 PostgreSQL 17，Prisma Client 生成器使用 `prisma-client`。历史版本用 `prisma-client-js`，具体差异参见 [版本边界与迁移](./version-governance.md)。

## 官方资源

- [Prisma 官方文档](https://www.prisma.io/docs)
- [Prisma 数据建模文档](https://www.prisma.io/docs/orm/prisma-schema/data-model/models)
- [Prisma Client 文档](https://www.prisma.io/docs/orm/prisma-client/queries/crud)
- [Prisma GitHub](https://github.com/prisma/prisma)
