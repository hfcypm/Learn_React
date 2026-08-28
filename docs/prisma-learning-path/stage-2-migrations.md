# 阶段二：迁移工作流

**目标**：掌握 Prisma Migrate 的完整工作流，理解迁移文件的生成机制，能够解决迁移冲突并正确处理生产环境迁移。

## 1. 迁移命令

| 命令 | 用途 |
|---|---|
| `prisma migrate dev` | 开发环境：生成迁移并应用 |
| `prisma migrate deploy` | 生产环境：应用未执行的迁移 |
| `prisma migrate reset` | 重置数据库并应用全部迁移 |
| `prisma migrate status` | 检查迁移状态 |
| `prisma migrate diff` | 生成 Schema 与数据库的差异 SQL |
| `prisma migrate resolve` | 标记迁移为已应用或回滚 |

## 2. 开发迁移流程

修改 Schema 后创建迁移：

```bash
npx prisma migrate dev --name add-user-profile
```

流程：

1. 对比 Schema 与当前数据库结构。
2. 生成 SQL 迁移文件到 `prisma/migrations/`。
3. 应用到开发数据库。
4. 触发 `prisma generate` 重新生成客户端。

迁移文件示例（`prisma/migrations/xxxx_add_user_profile/migration.sql`）：

```sql
CREATE TABLE "User" (
    "id" SERIAL NOT NULL,
    "email" TEXT NOT NULL,
    "name" TEXT,
    CONSTRAINT "User_pkey" PRIMARY KEY ("id")
);
```

## 3. 迁移与客户端的一致性

Schema 变更后必须同步：

1. 修改 Schema。
2. `npx prisma migrate dev`（应用迁移并生成客户端）。
3. 提交 Schema 与迁移文件。

重要：迁移文件与 Schema 一起提交，团队成员靠这些文件同步数据库结构。

## 4. 处理迁移冲突

多人协作时，本地数据库可能已经应用了别人新加的迁移。冲突时的处理：

```bash
# 查看状态
npx prisma migrate status

# 重置并重放全部迁移（开发环境）
npx prisma migrate reset
```

开发环境可以 `migrate reset`，但会清空数据。生产环境禁止 reset。

## 5. 用开发脚本初始化数据

```ts
// prisma/seed.ts
import { PrismaClient } from '../src/generated/client';
import { PrismaPg } from '@prisma/adapter-pg';

const prisma = new PrismaClient({
  adapter: new PrismaPg({ connectionString: process.env.DATABASE_URL }),
});

async function main() {
  await prisma.user.create({
    data: { email: 'a@example.com', name: 'Alice' },
  });
}

main()
  .catch((e) => { throw e })
  .finally(() => prisma.$disconnect());
```

```json
{
  "prisma": {
    "seed": "tsx prisma/seed.ts"
  }
}
```

```bash
npx prisma db seed
```

## 6. 生产环境迁移

生产环境绝不使用 `migrate dev`：

```bash
npx prisma migrate deploy
```

部署流程：

1. 构建应用。
2. 应用迁移（`migrate deploy`）。
3. 生成客户端（若未在构建时完成）。
4. 启动服务。

### 6.1 迁移失败处理

- 部分迁移失败后，先看 `prisma/migrations/migration_lock.toml` 与失败文件。
- 手动修复数据库到预期状态后，用 `prisma migrate resolve --applied <name>` 标记完成。
- 确认迁移在迁移表中一致性后再继续部署。

## 7. 迁移最佳实践

- 每次 Schema 变更独立成一次迁移，避免一次大变更。
- 迁移文件提交到版本控制。
- 生产迁移前先在 staging 环境演练。
- 破坏性变更（删列、删表）先评估影响，分步执行。
- 不手动修改已应用过的迁移文件。

## 8. 动手任务

1. 为博客模型添加 `Profile` 一对一关系，生成一次迁移。
2. 再添加 `Post.status` 枚举字段，生成第二次独立迁移。
3. 检查两次迁移的 SQL 文件，确认内容独立完整。
4. 编写 seed 脚本插入示例数据并执行 `prisma db seed`。
5. 故意在分支上制造一次迁移冲突，练习 reset 重放。

## 阶段二验收

- 能解释 migrate dev 与 deploy 的区别。
- 能生成并理解迁移 SQL 文件。
- 能处理开发环境的迁移冲突。
- 能编写 seed 脚本。
- 能正确执行生产迁移流程。

## 常见排错

| 现象 | 排查方向 |
|---|---|
| `migrate dev` 数据库非空 | 需重新 baseline 或用 reset |
| 迁移冲突提示 | 更新主分支后 reset 重放 |
| 生产迁移失败 | 手动修复后 resolve 标记 |
| 客户端类型过旧 | 重新 `prisma generate` |
| seed 不执行 | 检查 package.json prisma.seed 配置 |

## 进入下一阶段的条件

你能用迁移管理数据库结构变更。此时进入 [阶段三：Prisma Client 查询](./stage-3-prisma-client.md)。
