# 02 数据建模：Prisma Schema 与 PostgreSQL

**目标**：为看板应用设计完整数据模型，用 Prisma Schema 表达关系、枚举、约束与索引，通过迁移和 seed 落地到 PostgreSQL。

## 1. 数据模型总览

```text
User
 ├── TeamMember (membership) 多对多：User <-> Team
 ├── Project 一对多：Team -> Project
 │    └── Board 一对多：Project -> Board
 │         └── Task 一对多：Board -> Task
 │              ├── Tag 多对多：Task <-> Tag
 │              ├── Comment 一对多：Task -> Comment
 │              └── Assignee（User 引用）
 └── ActivityLog 一对多：任意实体 -> 操作日志
```

## 2. Prisma Schema

```prisma
// prisma/schema.prisma
generator client {
  provider = "prisma-client"
  output   = "./generated"
}

datasource db {
  provider = "postgresql"
}

enum Role {
  ADMIN
  MEMBER
}

enum TaskStatus {
  TODO
  IN_PROGRESS
  DONE
}

enum Priority {
  LOW
  MEDIUM
  HIGH
  URGENT
}

model User {
  id        Int       @id @default(autoincrement())
  email     String    @unique
  name      String
  password  String
  memberships TeamMember[]
  assignedTasks Task[]
  comments  Comment[]
  activities ActivityLog[]
  createdAt DateTime  @default(now())
}

model Team {
  id        Int          @id @default(autoincrement())
  name      String
  slug      String       @unique
  members   TeamMember[]
  projects  Project[]
  createdAt DateTime     @default(now())
}

model TeamMember {
  id     Int    @id @default(autoincrement())
  teamId Int
  team   Team   @relation(fields: [teamId], references: [id])
  userId Int
  user   User   @relation(fields: [userId], references: [id])
  role   Role   @default(MEMBER)

  @@unique([teamId, userId])
  @@index([userId])
}

model Project {
  id        Int       @id @default(autoincrement())
  teamId    Int
  team      Team      @relation(fields: [teamId], references: [id])
  name      String
  boards    Board[]
  createdAt DateTime  @default(now())

  @@index([teamId])
}

model Board {
  id        Int       @id @default(autoincrement())
  projectId Int
  project   Project   @relation(fields: [projectId], references: [id])
  name      String
  tasks     Task[]
  createdAt DateTime  @default(now())

  @@index([projectId])
}

model Task {
  id          Int       @id @default(autoincrement())
  boardId     Int
  board       Board     @relation(fields: [boardId], references: [id])
  title       String
  description String?
  status      TaskStatus @default(TODO)
  priority    Priority  @default(MEDIUM)
  assigneeId  Int?
  assignee    User?     @relation(fields: [assigneeId], references: [id])
  dueDate     DateTime?
  tags        TaskTag[]
  comments    Comment[]
  activities  ActivityLog[]
  createdAt   DateTime  @default(now())
  updatedAt   DateTime  @updatedAt

  @@index([boardId, status])
  @@index([assigneeId])
  @@index([priority])
}

model Tag {
  id    Int       @id @default(autoincrement())
  name  String
  tasks TaskTag[]

  @@unique([name])
}

model TaskTag {
  taskId Int
  task   Task @relation(fields: [taskId], references: [id])
  tagId  Int
  tag    Tag  @relation(fields: [tagId], references: [id])

  @@id([taskId, tagId])
}

model Comment {
  id      Int      @id @default(autoincrement())
  taskId  Int
  task    Task     @relation(fields: [taskId], references: [id])
  authorId Int
  author  User     @relation(fields: [authorId], references: [id])
  content String
  createdAt DateTime @default(now())

  @@index([taskId])
}

model ActivityLog {
  id        Int       @id @default(autoincrement())
  teamId    Int
  actorId   Int
  action    String
  entityType String
  entityId  Int
  meta      Json?
  createdAt DateTime  @default(now())

  @@index([teamId, createdAt])
}
```

## 3. 建模要点解析

### 3.1 关系类型

- 一对多：`Team -> Project`、`Project -> Board`、`Board -> Task`。
- 多对多（显式关联表）：`Task <-> Tag` 通过 `TaskTag`，因需要附带信息与联合主键。
- 多对多（隐式）：`User <-> Team` 也可用隐式写法，这里用 `TeamMember` 显式承载角色字段，扩展性更好。

### 3.2 复合主键

`TaskTag` 使用 `@@id([taskId, tagId])`，防止同一任务重复打同一标签。

### 3.3 枚举

状态与优先级用 Prisma `enum`，落到 PostgreSQL 为 `ENUM` 类型，从数据库层约束取值。

### 3.4 原生类型

金额、评分等精确场景可用 `@db.` 指定原生类型；这里 `ActivityLog.meta` 用 `Json` 映射 `JSONB`，灵活存储操作详情。

### 3.5 索引

- `@@index([boardId, status])`：按看板过滤状态的列表查询。
- `@@index([assigneeId])`：按成员过滤我的任务。
- `@@index([teamId, createdAt])`：活动日志按团队与时间倒序。
- `@@unique([teamId, userId])`：同一用户在团队中唯一，天然去重。

## 4. 迁移

```bash
npx prisma migrate dev --name init-schema
```

Prisma 会对比 Schema 与数据库，生成 SQL 迁移文件。可查看生成内容：

```sql
-- prisma/migrations/*_init-schema/migration.sql（节选）
CREATE TABLE "TeamMember" (
    "id" SERIAL NOT NULL,
    "teamId" INTEGER NOT NULL,
    "userId" INTEGER NOT NULL,
    "role" "Role" NOT NULL DEFAULT 'MEMBER',
    CONSTRAINT "TeamMember_pkey" PRIMARY KEY ("id")
);

CREATE UNIQUE INDEX "TeamMember_teamId_userId_key" ON "TeamMember"("teamId", "userId");
CREATE INDEX "Task_boardId_status_idx" ON "Task"("boardId", "status");
```

## 5. 用原生 SQL 补充搜索能力

全文搜索需要生成 `tsvector` 列与 GIN 索引，用迁移后的追加 SQL 或 `prisma migrate dev --create-only` 后手写：

```sql
ALTER TABLE "Task" ADD COLUMN "searchVector" tsvector
  GENERATED ALWAYS AS (to_tsvector('simple', coalesce(title, '') || ' ' || coalesce(description, ''))) STORED;

CREATE INDEX "Task_searchVector_idx" ON "Task" USING GIN ("searchVector");
```

然后在 Schema 中声明该列（否则迁移检测会认为有漂移）：

```prisma
model Task {
  // ...原有字段
  searchVector Unsupported("tsvector")?
}
```

或者更简单：不在 Schema 声明，检索时直接调 `$queryRaw`（见 [06 查询与事务](./06-advanced.md)）。

## 6. Seed 数据

```ts
// prisma/seed.ts
import { PrismaClient } from '../src/generated/client';
import { PrismaPg } from '@prisma/adapter-pg';

const prisma = new PrismaClient({
  adapter: new PrismaPg({ connectionString: process.env.DATABASE_URL! }),
});

async function main() {
  const user = await prisma.user.upsert({
    where: { email: 'admin@kanban.dev' },
    update: {},
    create: {
      email: 'admin@kanban.dev',
      name: 'Admin',
      password: 'hashed-placeholder',
    },
  });

  const team = await prisma.team.create({
    data: {
      name: '产品团队',
      slug: 'product',
      members: {
        create: [{ userId: user.id, role: 'ADMIN' }],
      },
      projects: {
        create: [
          {
            name: '看板应用',
            boards: {
              create: [
                { name: '待办', tasks: { create: [{ title: '设计数据模型', priority: 'HIGH' }] } },
                { name: '进行中' },
                { name: '完成' },
              ],
            },
          },
        ],
      },
    },
    include: { projects: { include: { boards: true } } },
  });

  console.log('seed 完成', team.name);
}

main()
  .catch((e) => { throw e })
  .finally(() => prisma.$disconnect());
```

在 `package.json` 配置 seed 命令：

```json
{
  "prisma": { "seed": "tsx prisma/seed.ts" }
}
```

```bash
npm install -D tsx
npx prisma db seed
```

## 7. 数据验证

用 psql 验证表结构与数据：

```bash
psql -d kanban -c '\dt'
psql -d kanban -c 'SELECT "name" FROM "Task";'
```

用 Prisma Studio 可视化检查：

```bash
npx prisma studio
```

## 8. 本章验收

- [ ] 所有模型、关系、枚举迁移成功。
- [ ] `TaskTag` 复合主键生效，重复关联被拒绝。
- [ ] seed 后数据库有用户、团队、项目、看板、任务。
- [ ] `\dt` 能看到全部表，索引已建立。
- [ ] 全文搜索 GIN 索引可用。

## 9. 常见排错

| 现象 | 排查方向 |
|---|---|
| 关系字段未成对 | Schema 两侧都要声明对应字段 |
| 多对多歧义 | 显式关联表 + 命名 `@relation` |
| 迁移检测漂移 | `Unsupported("tsvector")` 或改用 `$queryRaw` |
| seed 连接失败 | 确认 `DATABASE_URL` 与 adapter 配置 |
| 枚举修改失败 | 枚举变更走迁移，生产环境谨慎处理 |
| 索引未生效 | EXPLAIN 验证，确认列顺序与查询匹配 |

## 进入下一章的条件

数据模型完整落地并能用 Studio 查看。此时进入 [03 鉴权](./03-auth.md)。
