# Node.js 与 NestJS 新手学习路线重构需求

## Introduction

现有 Node.js 与 NestJS 文档覆盖面较完整，学习入口、实践节奏和知识之间的依赖关系需要进一步明确。本次重构面向已经具备基础 JavaScript、TypeScript 和前端经验、服务端经验较少的学习者，建立从环境准备到生产交付的连续学习路径。

## Glossary

- **学习阶段**：围绕一个能力目标组织的知识、练习、产出和验收内容。
- **阶段产出**：学习者完成阶段任务后能够运行或展示的最小成果。
- **综合项目**：贯穿多阶段、逐步增加能力的可运行项目。
- **桥接章节**：解释 Node.js 原生能力如何映射到 NestJS 抽象的内容。

## Requirements

### Requirement 1: 新手导读

**User Story:** AS a 初学者, I want to know the prerequisites and learning order, so that I can begin without guessing the next topic.

#### Acceptance Criteria

1. WHEN a learner opens either learning package, THE documentation SHALL show prerequisites, environment setup, recommended order, expected time investment, and completion criteria.
2. WHEN a learner completes a stage, THE documentation SHALL identify the stage output and measurable self-check items.
3. IF a learner encounters a common setup or conceptual error, THE documentation SHALL provide a troubleshooting path and a corrective example.

### Requirement 2: 渐进式实践

**User Story:** AS a 初学者, I want each stage to produce a small result, so that I can connect concepts with working code.

#### Acceptance Criteria

1. THE Node.js route SHALL progress from script and HTTP primitives to a layered API, tests, and production delivery.
2. THE NestJS route SHALL progress from framework bootstrap to modules, request lifecycle, data access, authentication, testing, and deployment.
3. EACH stage SHALL define prerequisites, core questions, practice tasks, expected files or behavior, and acceptance checks.

### Requirement 3: Node.js 到 NestJS 桥接

**User Story:** AS a Node.js learner, I want to understand why NestJS abstractions exist, so that I can learn the framework instead of memorizing decorators.

#### Acceptance Criteria

1. THE documentation SHALL map Node.js HTTP, routing, middleware, validation, errors, dependency wiring, and testing concepts to NestJS equivalents.
2. WHEN a framework concept introduces an abstraction, THE documentation SHALL explain the underlying runtime behavior and the responsibility boundary.
3. THE documentation SHALL identify cases where a plain Node.js implementation is sufficient and cases where NestJS structure provides value.

### Requirement 4: 综合项目

**User Story:** AS a learner, I want one project to evolve across stages, so that I can practice an end-to-end service.

#### Acceptance Criteria

1. THE Node.js project SHALL evolve from a health endpoint to a users API with validation, persistence, authentication, tests, logging, and deployment checks.
2. THE NestJS project SHALL evolve from a health module to a modular users and roles API with DTO validation, guards, persistence, tests, and deployment checks.
3. EACH project increment SHALL state the purpose, files to create, request examples, expected result, and verification command.

### Requirement 5: 文档可维护性

**User Story:** AS a maintainer, I want consistent document structure, so that future lessons can be added without breaking the learning route.

#### Acceptance Criteria

1. EACH learning package README SHALL provide a stage table, navigation, project milestones, and official references.
2. THE documentation SHALL distinguish conceptual examples from complete runnable project code.
3. THE documentation SHALL use placeholders for credentials and include no real secrets.
