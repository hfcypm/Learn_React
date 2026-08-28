# Prisma 版本边界与迁移

**目标**：让学习内容、示例项目和生产项目拥有清晰的版本边界，降低大版本升级和 API 混用风险。

## 1. 版本差异

| 能力 | Prisma 5 / 6 | Prisma 7 |
|---|---|---|
| 客户端生成器 | `prisma-client-js` | `prisma-client`（新默认） |
| 生成目录 | `node_modules/@prisma/client` | `output` 指定的项目目录 |
| 数据库连接 | 直接使用连接串 | 必须通过 driver adapter |
| 连接池 | 客户端内部管理 | 由驱动管理，adapter 配置超时 |
| 环境变量 | 默认读 `.env` | 通过 `prisma.config.ts` 显式加载 |

## 2. 每个示例必须记录的元数据

```markdown
> 适用范围：Prisma 7 + PostgreSQL 17
> 需要的工具：Node.js、npm、psql
> 验证命令：prisma validate、migrate status、构建与测试
> 最后复核日期：YYYY-MM-DD
```

## 3. Prisma 7 迁移要点

- 生成器改用 `prisma-client`，并设置 `output` 目录。
- 客户端从生成目录导入：`import { PrismaClient } from './generated/client'`。
- 安装并配置 driver adapter：`@prisma/adapter-pg` + `PrismaPg`，客户端创建时传入 `{ adapter }`。
- 连接串通过 `prisma.config.ts` 显式加载，不隐式读 `.env`。
- 连接池由底层数据库驱动管理，通过 adapter 配置超时与空闲行为。
- 检查新版本的默认行为变化与移除的预览特性。

## 4. 升级前检查

1. 阅读官方 [升级指南](https://www.prisma.io/docs/orm/more/upgrade-guides)。
2. 阅读目标版本 release notes 与 breaking changes。
3. 检查生成器、客户端导入路径和配置语法。
4. 检查项目中的废弃 API 与预览特性。
5. 在分支环境完整验证迁移、查询与构建。

## 5. 升级后检查

- Schema 通过 `prisma validate`。
- 迁移文件一致，`migrate status` 正常。
- 客户端导入路径与类型正确。
- 查询与事务行为一致。
- 构建与生产部署通过。
- 连接池与日志配置生效。
