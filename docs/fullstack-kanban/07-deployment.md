# 07 生产交付：迁移、部署、监控与备份

**目标**：掌握生产环境的数据迁移、连接池、日志与监控、备份恢复，以及完整部署流程，交付可上线、可回滚、可运维的看板应用。

## 1. 生产环境迁移

开发用 `migrate dev`，生产一律用 `migrate deploy`：

```bash
# 生产部署流程
npm ci
npx prisma generate
npx prisma migrate deploy
npm run build
npm run start
```

生产禁止使用 `migrate dev` 或 `migrate reset`。迁移文件随代码提交，部署时按序应用。

### 迁移失败处理

- 检查 `prisma/migrations/` 与 `_prisma_migrations` 表。
- 手动修复数据库到预期状态后，用 `prisma migrate resolve --applied <name>` 标记。
- 确认迁移一致性后再继续部署。

## 2. 环境变量

```env
# 生产环境（占位符示例）
DATABASE_URL="postgresql://user:password@host:5432/kanban?sslmode=require"
SESSION_SECRET=<random-long-secret>
```

- `.env` 不进版本控制。
- 用平台 Secret 管理注入真实值。
- 连接串区分开发/测试/生产。

## 3. 连接池

Prisma 7 通过 driver adapter 连接，连接池由 pg 驱动管理。适配生产规模：

```ts
// lib/db.ts
import { PrismaClient } from '@/generated/client';
import { PrismaPg } from '@prisma/adapter-pg';

function createPrisma() {
  const adapter = new PrismaPg({
    connectionString: process.env.DATABASE_URL,
    connectionTimeoutMillis: 5_000,
    idleTimeoutMillis: 300_000,
  });
  return new PrismaClient({
    adapter,
    log: process.env.NODE_ENV === 'development' ? ['query', 'warn', 'error'] : ['error'],
  });
}

const globalForPrisma = globalThis as unknown as { prisma?: PrismaClient };
export const prisma = globalForPrisma.prisma ?? createPrisma();

if (process.env.NODE_ENV !== 'production') {
  globalForPrisma.prisma = prisma;
}
```

客户端单例避免每次请求新建连接；生产关闭 query 日志。

## 4. 日志与可观测性

结构化日志便于聚合查询：

```ts
// lib/logger.ts
export function log(level: 'info' | 'warn' | 'error', msg: string, fields?: Record<string, unknown>) {
  console[level](JSON.stringify({ level, msg, time: new Date().toISOString(), ...fields }));
}
```

在关键路径埋点：

```ts
log('info', 'task moved', { taskId, from, to, actorId });
```

## 5. 数据库监控

启用 `pg_stat_statements` 定位慢查询：

```bash
psql -d kanban -c 'CREATE EXTENSION IF NOT EXISTS pg_stat_statements;'
```

```sql
-- 最慢的十条查询
SELECT query, calls, mean_exec_time, max_exec_time
FROM pg_stat_statements
ORDER BY mean_exec_time DESC
LIMIT 10;

-- 连接数
SELECT count(*) FROM pg_stat_activity;

-- 死元组（膨胀）
SELECT relname, n_dead_tup
FROM pg_stat_user_tables
WHERE n_dead_tup > 1000;
```

持续膨胀时手动维护：

```sql
VACUUM ANALYZE "Task";
```

## 6. 备份与恢复

```bash
# 逻辑备份
pg_dump kanban > kanban-$(date +%Y%m%d).sql

# 恢复演练
createdb kanban_restore
psql -d kanban_restore -f kanban-$(date +%Y%m%d).sql
```

生产环境使用 WAL 归档 + 基础备份实现时间点恢复：

```text
postgresql.conf
  wal_level = replica
  archive_mode = on
  archive_command = 'cp %p /backup/wal/%f'
```

定期演练恢复，确认 RTO/RPO 达标。

## 7. Dockerfile

```dockerfile
# Dockerfile
FROM node:22-alpine AS deps
WORKDIR /app
COPY package.json package-lock.json ./
RUN npm ci

FROM node:22-alpine AS build
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .
RUN npx prisma generate \
  && npm run build

FROM node:22-alpine AS runner
WORKDIR /app
ENV NODE_ENV=production
COPY --from=build /app/.next ./.next
COPY --from=build /app/node_modules ./node_modules
COPY --from=build /app/public ./public
COPY --from=build /app/prisma ./prisma
COPY --from=build /app/package.json ./package.json
EXPOSE 3000
CMD ["sh", "-c", "npx prisma migrate deploy && npm run start"]
```

## 8. CI 检查

```text
lint
typecheck
prisma validate      # 校验 Schema
prisma migrate status  # 确认迁移一致
test
build                # 确认产物含生成客户端
```

## 9. 部署检查清单

- [ ] `prisma migrate deploy` 在部署流程内执行。
- [ ] 生产连接串启用 SSL。
- [ ] 客户端单例、连接池超时配置正确。
- [ ] 生产不打印 query 日志。
- [ ] 监控覆盖慢查询、连接数、死元组。
- [ ] 备份可恢复，恢复演练通过。
- [ ] 环境变量用 Secret 管理，无明文。
- [ ] 部署可回滚，镜像标签可追溯。

## 10. 常见排错

| 现象 | 排查方向 |
|---|---|
| 生产连接数耗尽 | 检查连接池与 max_connections |
| 部署后客户端未生成 | 构建流程缺 `prisma generate` |
| 生产迁移未应用 | 确认 `migrate deploy` 在启动前执行 |
| 慢查询 | pg_stat_statements 定位后加索引 |
| 表膨胀 | VACUUM 与 autovacuum 配置 |
| 备份恢复失败 | 校验备份完整性、权限与路径 |
| 会话失效 | SESSION_SECRET 变化或过期 |

## 11. 本章验收

- [ ] 能完成一次生产部署（构建 + 迁移 + 启动）。
- [ ] 连接池与日志配置符合生产要求。
- [ ] 慢查询、连接数、死元组可监控。
- [ ] 备份恢复演练通过。
- [ ] Docker 镜像构建成功并可运行。
- [ ] CI 包含 Schema 与迁移校验。

## 项目完成标准

完成本章后，你已用一条路径跑通 Next.js + Tailwind CSS + Prisma + PostgreSQL 的完整开发与交付循环。回到 [README](./README.md) 核对「知识点映射」，对照四条学习路线的评估与排错题自测，补齐薄弱环节。
