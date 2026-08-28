# 06 查询与事务：Prisma Client 与 PostgreSQL 优化

**目标**：用 Prisma Client 完成复杂查询、关系加载、分页与事务，并结合 PostgreSQL 的索引、聚合、窗口函数与全文搜索优化数据访问。

## 1. 关系查询与 N+1 规避

按条件筛选任务并一次带出所有关联：

```ts
// lib/queries.ts
import { prisma } from '@/lib/db';
import { Prisma } from '@/generated/client';

export type TaskFilter = {
  boardId?: number;
  assigneeId?: number;
  status?: 'TODO' | 'IN_PROGRESS' | 'DONE';
  priority?: 'LOW' | 'MEDIUM' | 'HIGH' | 'URGENT';
  tagIds?: number[];
};

export async function listTasks(filter: TaskFilter, cursor?: number, take = 20) {
  const where: Prisma.TaskWhereInput = {
    ...(filter.boardId ? { boardId: filter.boardId } : {}),
    ...(filter.assigneeId ? { assigneeId: filter.assigneeId } : {}),
    ...(filter.status ? { status: filter.status } : {}),
    ...(filter.priority ? { priority: filter.priority } : {}),
    ...(filter.tagIds?.length
      ? { tags: { some: { tagId: { in: filter.tagIds } } } }
      : {}),
  };

  return prisma.task.findMany({
    where,
    include: {
      assignee: { select: { id: true, name: true } },
      tags: { include: { tag: true } },
      comments: { select: { id: true, content: true, createdAt: true }, take: 5 },
    },
    orderBy: { updatedAt: 'desc' },
    take,
    ...(cursor ? { cursor: { id: cursor }, skip: 1 } : {}),
  });
}
```

知识点：`Prisma.TaskWhereInput` 动态组合条件、`select` 限定字段、keyset 分页（游标）。

## 2. 筛选与排序

```ts
// 我的任务（筛选指派 + 状态 + 日期）
const myTasks = await prisma.task.findMany({
  where: {
    assigneeId: session.userId,
    status: { in: ['TODO', 'IN_PROGRESS'] },
    dueDate: { lte: new Date() },
  },
  orderBy: [{ priority: 'desc' }, { dueDate: 'asc' }],
});
```

组合索引 `@@index([assigneeId])` 与 `@@index([priority])` 支撑这类查询。

## 3. 原子更新

避免「读改写」竞态，用 `increment` 等运算符：

```ts
// 评论数（若用计数器列）
await prisma.task.update({
  where: { id: taskId },
  data: { commentCount: { increment: 1 } },
});
```

## 4. 交互式事务：移动任务并写日志

拖动任务跨列时，状态更新与活动日志必须同时成功：

```ts
// lib/services/move-task.ts
import { prisma } from '@/lib/db';

export async function moveTask(taskId: number, toStatus: string, actorId: number) {
  await prisma.$transaction(async (tx) => {
    const task = await tx.task.findUniqueOrThrow({
      where: { id: taskId },
      include: { board: { include: { project: true } } },
    });

    if (task.status === toStatus) return;

    await tx.task.update({
      where: { id: taskId },
      data: { status: toStatus as 'TODO' | 'IN_PROGRESS' | 'DONE' },
    });

    await tx.activityLog.create({
      data: {
        teamId: task.board.project.teamId,
        actorId,
        action: 'MOVE',
        entityType: 'TASK',
        entityId: taskId,
        meta: { from: task.status, to: toStatus },
      },
    });
  });
}
```

知识点：交互式事务处理「读后写」依赖、`findUniqueOrThrow`、事务内所有操作原子提交。

## 5. 防止重复操作

给任务打标签用 upsert 加唯一约束兜底：

```ts
await prisma.taskTag.upsert({
  where: { taskId_tagId: { taskId, tagId } },
  update: {},
  create: { taskId, tagId },
});
```

## 6. 聚合统计

```ts
// 按状态统计任务
const byStatus = await prisma.task.groupBy({
  by: ['status'],
  where: { board: { project: { teamId } } },
  _count: { _all: true },
});

// 计算看板总数与平均任务数
const agg = await prisma.board.aggregate({
  _count: { _all: true },
  _avg: { id: true },
});
```

## 7. 原生 SQL：全文搜索与窗口函数

### 7.1 全文搜索

Schema 中不声明 `tsvector` 列，直接调用原生 SQL：

```ts
// lib/services/search.ts
import { prisma } from '@/lib/db';

export async function searchTasks(teamId: number, query: string) {
  const rows = await prisma.$queryRaw<{ id: number; title: string; rank: number }[]>`
    SELECT "id", "title", ts_rank("searchVector", to_tsquery('simple', ${query})) AS rank
    FROM "Task"
    JOIN "Board" ON "Board"."id" = "Task"."boardId"
    JOIN "Project" ON "Project"."id" = "Board"."projectId"
    WHERE "Project"."teamId" = ${teamId}
      AND "searchVector" @@ to_tsquery('simple', ${query})
    ORDER BY rank DESC
    LIMIT 20
  `;
  return rows;
}
```

前置：按 [02 数据建模](./02-schema.md) 中的 SQL 为 `Task` 添加 `searchVector` 生成列与 GIN 索引。

### 7.2 窗口函数：每列最新任务

```ts
const latestPerStatus = await prisma.$queryRaw`
  SELECT * FROM (
    SELECT "id", "title", "status",
      row_number() OVER (PARTITION BY "status" ORDER BY "updatedAt" DESC) AS rn
    FROM "Task"
    WHERE "boardId" = ${boardId}
  ) t
  WHERE rn = 1
`;
```

## 8. EXPLAIN 验证索引

慢查询用 EXPLAIN 分析是否走索引：

```bash
psql -d kanban

EXPLAIN ANALYZE
SELECT * FROM "Task"
WHERE "boardId" = 1 AND "status" = 'TODO';
```

`@@index([boardId, status])` 应命中 Index Scan。对比移除索引前后的执行计划，理解组合索引列顺序的价值。

## 9. 常见优化手段

- 列表查询用 `select` 限字段，避免拉取整行大字段。
- 关系查询一次 `include`，杜绝循环查询。
- 大分页用 keyset 游标，不用大 `OFFSET`。
- 写操作尽量原子化，少用「先查后改」。
- 需要事务读后写时用 `$transaction`，缩短事务时长。
- 全文搜索、排名用原生 SQL，数据库内完成。

## 10. 本章验收

- [ ] 组合条件筛选任务无 N+1。
- [ ] 移动任务用交互式事务，状态与日志原子更新。
- [ ] upsert 配合唯一约束防止重复打标签。
- [ ] 聚合统计返回正确。
- [ ] 全文搜索命中 GIN 索引，结果按相关度排序。
- [ ] 窗口函数取回每组最新任务。
- [ ] EXPLAIN 验证关键查询走索引。

## 11. 常见排错

| 现象 | 排查方向 |
|---|---|
| 事务超时 | 缩短事务、减少锁持有时间 |
| 并发移动冲突 | 事务重试或加锁策略 |
| upsert 报唯一冲突 | 确认复合唯一键命名 `taskId_tagId` |
| 全文搜索不匹配 | tsvector 与查询用同一配置（simple） |
| $queryRaw 类型错误 | 声明返回行类型 |
| 索引未生效 | 隐式类型转换、列顺序、统计过旧（ANALYZE） |

## 进入下一章的条件

查询正确、事务可靠、搜索与统计可用。此时进入 [07 生产交付](./07-deployment.md)。
