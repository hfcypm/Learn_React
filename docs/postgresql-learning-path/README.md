# PostgreSQL 学习路线图（基础 → 生产）

**目标**：掌握 PostgreSQL 的关系模型、SQL 查询、数据建模、事务、索引、性能优化和生产运维，能够设计并维护可靠、可扩展的数据库。

## 先看这里

- [新手导读](./start-here.md)：前置条件、学习顺序、排错顺序和完成标准。
- [综合实战：电商订单数据库](./project-practice.md)：按阶段逐步扩展同一个数据库项目。
- [学习评估与项目评分](./learning-assessment.md)：阶段考核、排错题、项目评分和评审问题。

## 学习阶段总览

| 阶段 | 名称 | 目标 |
|---|---|---|
| 零 | 数据库基础与工具链 | 理解关系模型，能安装并使用 psql 完成基本操作 |
| 一 | 核心 SQL 与查询 | 掌握 SELECT、JOIN、聚合、窗口函数和子查询 |
| 二 | 数据建模与约束 | 掌握 DDL、范式、约束、索引和 JSONB |
| 三 | 事务、锁与性能优化 | 掌握事务隔离、锁、EXPLAIN 和索引优化 |
| 四 | 生产运维 | 掌握备份恢复、复制、监控和扩展 |

## 快速导航

- [阶段零：数据库基础与工具链](./stage-0-foundations.md)
- [阶段一：核心 SQL 与查询](./stage-1-core-sql.md)
- [阶段二：数据建模与约束](./stage-2-data-modeling.md)
- [阶段三：事务、锁与性能优化](./stage-3-performance.md)
- [阶段四：生产运维](./stage-4-operations.md)
- [综合实战：电商订单数据库](./project-practice.md)

## 学习原则

- 每个 SQL 概念先在 psql 里写出并运行，再结合数据模型理解。
- 查询优先看执行计划，再谈写法优化。
- 数据建模先设计再建表，约束在入口处防止脏数据。
- 事务与隔离级别决定并发正确性，锁决定并发吞吐。
- 索引是有代价的，为查询建立索引而非为表建立索引。
- 生产环境任何变更都要有备份和回滚路径。

## 阶段成果

- 能安装 PostgreSQL 并用 psql 完成建库、建表和查询。
- 能写出多表 JOIN、聚合、窗口函数和子查询。
- 能设计范式合理、约束完整的数据模型。
- 能用 EXPLAIN 定位慢查询并通过索引优化。
- 能理解事务隔离、锁和备份恢复机制。

## 每阶段固定学习模板

每个阶段按照“目标 -> 前置知识 -> 核心问题 -> 最小示例 -> 项目增量 -> 验收 -> 排错”执行。先完成阶段项目，再阅读扩展主题；这样能够把数据库概念连接到可运行结果。

## 版本边界

学习主线对应 PostgreSQL 17。特性以官方文档为准，PostgreSQL 大版本每年发布一次，具体差异参见 [版本边界与迁移](./version-governance.md)。

## 配套路线

在 TypeScript 项目中用 ORM 访问 PostgreSQL 时，阅读 [Prisma 学习路线](../prisma-learning-path/README.md)；在服务端应用中接入数据库时，参考 [Node.js 学习路线](../nodejs-learning-path/README.md) 与 [NestJS 学习路线](../nestjs-learning-path/README.md) 的数据层部分。

## 官方资源

- [PostgreSQL 官方文档](https://www.postgresql.org/docs/)
- [PostgreSQL 17 文档](https://www.postgresql.org/docs/17/)
- [PostgreSQL EXPLAIN 文档](https://www.postgresql.org/docs/current/sql-explain.html)
