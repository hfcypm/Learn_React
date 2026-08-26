# Node.js 学习路线图（基础 → 生产）

**目标**：理解 Node.js 运行时，能够使用原生 API 和常见工程模式开发可测试、可观测、可部署的服务端应用。

## 先看这里

- [新手导读](./start-here.md)：前置条件、学习方法、排错顺序和完成标准。
- [综合实战：用户管理 API](./project-practice.md)：按阶段逐步扩展同一个项目。

## 学习阶段总览

| 阶段 | 名称 | 目标 |
|---|---|---|
| 零 | JavaScript 与运行时基础 | 理解 Node.js、模块、异步和命令行环境 |
| 一 | Node.js 核心 API | 掌握 HTTP、文件、路径、事件和 Streams |
| 二 | Web 服务与数据层 | 开发 API、处理校验、认证和数据库访问 |
| 三 | 工程化与并发 | 掌握测试、日志、配置、Worker 和性能分析 |
| 四 | 生产交付 | 完成安全、可观测性、部署和故障恢复 |

## 快速导航

- [阶段零：运行时与 JavaScript 基础](./stage-0-foundations.md)
- [阶段一：Node.js 核心 API](./stage-1-core-api.md)
- [阶段二：Web 服务与数据层](./stage-2-web-data.md)
- [阶段三：工程化与并发](./stage-3-engineering.md)
- [阶段四：生产交付](./stage-4-production.md)
- [综合实战：用户管理 API](./project-practice.md)

## 学习原则

- 使用 Node.js 官方 `node:` 前缀导入内置模块，例如 `node:fs/promises`。
- 明确同步 API、异步 API、Stream 和 Worker 的适用边界。
- 外部输入统一做校验、超时、取消和错误映射。
- 先使用单进程异步模型，再根据 CPU 瓶颈评估 Worker 或多进程方案。
- 每个服务具备测试、结构化日志、健康检查和优雅退出能力。
- 生产环境优先使用受支持的 LTS 版本，并在升级前验证原生模块、框架和部署镜像兼容性。

## 阶段成果

- 能写原生 HTTP JSON 服务和 CLI 工具。
- 能实现带认证、校验、分页、错误处理和数据库访问的 API。
- 能为异步流程、Streams、Worker 和关键业务编写测试。
- 能完成容器化部署、监控、限流、回滚和故障排查。
- 能独立完成一个带认证、数据库和测试的 Node.js API 项目。

## 每阶段固定学习模板

每个阶段按照“目标 -> 前置知识 -> 核心问题 -> 最小示例 -> 项目增量 -> 验收 -> 排错”执行。先完成阶段项目，再阅读对应的扩展主题；这样能够把 API 知识连接到可运行结果。

## Node.js 到 NestJS

完成阶段二后阅读 [从 Node.js 到 NestJS](../nestjs-learning-path/node-to-nest-bridge.md)，用同一组 HTTP、校验、错误、依赖和测试概念进入 NestJS。

## 官方资源

- [Node.js 官方文档](https://nodejs.org/docs/latest/api/)
- [Node.js 学习资源](https://nodejs.org/en/learn)
- [Node.js 发布计划](https://github.com/nodejs/release)
