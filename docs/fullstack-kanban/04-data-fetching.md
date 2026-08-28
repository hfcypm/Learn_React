# 04 数据获取：Server Component、缓存与 Route Handler

**目标**：用 Server Component 直连数据库读取看板数据，用 ISR 与 revalidatePath 控制缓存，用 Route Handler 提供 API，并处理加载态、错误态与 404。

## 1. Server Component 直接取数

看板页作为 Server Component，直接调用 Prisma：

```tsx
// app/teams/[teamId]/boards/[boardId]/page.tsx
import { notFound } from 'next/navigation';
import { prisma } from '@/lib/db';
import { requireUser } from '@/lib/auth';
import { assertTeamAccess } from '@/lib/permissions';
import { BoardColumns } from '@/components/board-columns';

export default async function BoardPage({
  params,
}: {
  params: Promise<{ teamId: string; boardId: string }>;
}) {
  const { teamId, boardId } = await params;
  const session = await requireUser();
  const teamIdNum = Number(teamId);
  const boardIdNum = Number(boardId);

  await assertTeamAccess(session, teamIdNum);

  const board = await prisma.board.findFirst({
    where: { id: boardIdNum, project: { teamId: teamIdNum } },
    include: {
      tasks: {
        include: {
          assignee: { select: { id: true, name: true } },
          tags: { include: { tag: true } },
        },
        orderBy: { updatedAt: 'asc' },
      },
    },
  });

  if (!board) notFound();

  return (
    <main>
      <h1>{board.name}</h1>
      <BoardColumns boardId={board.id} tasks={board.tasks} />
    </main>
  );
}
```

要点：

- `params` 在 Next.js 15 是 `Promise`，需 `await`。
- `findFirst` + 团队条件校验归属，避免跨团队访问。
- `include` 一次带出指派人与标签，避免 N+1。
- `notFound()` 渲染最近 `not-found.tsx`。

## 2. 客户端组件接收服务端数据

```tsx
// components/board-columns.tsx
'use client';

import type { Board } from '@/generated/client';

type Props = {
  boardId: number;
  tasks: Awaited<ReturnType<typeof loadTasks>>;
};

export function BoardColumns({ boardId, tasks }: Props) {
  return (
    <div className="grid grid-cols-1 gap-4 md:grid-cols-3">
      {tasks.map((task) => (
        <TaskCard key={task.id} task={task} />
      ))}
    </div>
  );
}
```

服务端渲染的数据以 props 传给 Client Component，交互逻辑留在客户端。

## 3. 加载态与错误态

```tsx
// app/teams/[teamId]/boards/[boardId]/loading.tsx
export default function Loading() {
  return <p className="p-6">加载看板…</p>;
}
```

```tsx
// app/teams/[teamId]/boards/[boardId]/error.tsx
'use client';

export default function Error({ reset }: { reset: () => void }) {
  return (
    <div className="p-6">
      <h2>加载失败</h2>
      <button onClick={reset}>重试</button>
    </div>
  );
}
```

```tsx
// app/teams/[teamId]/boards/[boardId]/not-found.tsx
export default function NotFound() {
  return <h1>看板不存在</h1>;
}
```

## 4. 静态渲染、动态渲染与 ISR

看板数据实时性强，使用动态渲染。统计概览等低频页面用 ISR：

```tsx
// app/teams/[teamId]/stats/page.tsx
import { prisma } from '@/lib/db';

export const revalidate = 300; // 5 分钟

export default async function TeamStats({ params }: { params: Promise<{ teamId: string }> }) {
  const { teamId } = await params;

  const counts = await prisma.task.groupBy({
    by: ['status'],
    where: { board: { project: { teamId: Number(teamId) } } },
    _count: { _all: true },
  });

  return (
    <ul>
      {counts.map((row) => (
        <li key={row.status}>
          {row.status}: {row._count._all}
        </li>
      ))}
    </ul>
  );
}
```

动态页面会用到请求时数据；`export const revalidate` 让整页按间隔重新验证（ISR）。

## 5. 主动重新验证

写操作后调用 `revalidatePath` 使相关路由缓存失效：

```ts
// app/actions/tasks.ts
'use server';

import { revalidatePath } from 'next/cache';
import { prisma } from '@/lib/db';
import { requireUser } from '@/lib/auth';

export async function createTask(boardId: number, formData: FormData) {
  const session = await requireUser();

  const title = String(formData.get('title')).trim();
  if (!title) throw new Error('任务标题不能为空');

  await prisma.task.create({
    data: {
      boardId,
      title,
      priority: String(formData.get('priority') ?? 'MEDIUM') as 'LOW' | 'MEDIUM' | 'HIGH' | 'URGENT',
    },
  });

  revalidatePath(`/boards/${boardId}`);
}
```

需要团队归属校验时，先查出看板所属团队再判断。

## 6. Route Handler 提供 API

移动任务的交互场景适合用 API：

```ts
// app/api/boards/[boardId]/tasks/[taskId]/move/route.ts
import { NextResponse } from 'next/server';
import { prisma } from '@/lib/db';
import { requireUser } from '@/lib/auth';

export async function POST(
  request: Request,
  { params }: { params: Promise<{ boardId: string; taskId: string }> }
) {
  const session = await requireUser();
  const { taskId } = await params;
  const { status } = (await request.json()) as { status: string };

  const valid = ['TODO', 'IN_PROGRESS', 'DONE'];
  if (!valid.includes(status)) {
    return NextResponse.json({ error: '非法的状态' }, { status: 400 });
  }

  const updated = await prisma.task.update({
    where: { id: Number(taskId) },
    data: { status: status as 'TODO' | 'IN_PROGRESS' | 'DONE' },
  });

  return NextResponse.json(updated);
}
```

Route Handler 需要自行校验鉴权、入参与错误映射。

## 7. 客户端数据获取（需要实时交互时）

看板拖动可使用客户端获取 + 乐观更新，配合 Route Handler：

```tsx
'use client';

import { useEffect, useState } from 'react';

export function useTaskMove(taskId: number) {
  const [moving, setMoving] = useState(false);
  const [error, setError] = useState<string | null>(null);

  async function moveTo(status: string) {
    setMoving(true);
    setError(null);
    try {
      const res = await fetch(`/api/boards/1/tasks/${taskId}/move`, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ status }),
      });
      if (!res.ok) throw new Error('移动失败');
    } catch (e) {
      setError((e as Error).message);
    } finally {
      setMoving(false);
    }
  }

  return { moving, error, moveTo };
}
```

原则：默认用 Server Component 取数；只有需要即时交互或客户端状态时，才用客户端数据获取。

## 8. 本章验收

- [ ] 看板页用 Server Component 直连数据库渲染。
- [ ] 任务列表一次 include 出指派人与标签，无 N+1。
- [ ] 加载态、错误态、404 各就各位。
- [ ] 写操作后 revalidatePath，页面即时更新。
- [ ] Route Handler 能移动任务状态并校验入参。
- [ ] ISR 页面按间隔重新验证。

## 9. 常见排错

| 现象 | 排查方向 |
|---|---|
| params 是 Promise 报错 | Next 15 需 `await params` |
| 页面一直动态渲染 | 依赖了请求期函数（cookies、searchParams） |
| 写后不刷新 | 漏掉 `revalidatePath` 或路径不匹配 |
| Route Handler 403 | 未在 Handler 内做鉴权 |
| include 数据太多 | 用 `select` 只取所需字段 |
| 跨团队访问到数据 | 查询条件加团队归属校验 |

## 进入下一章的条件

数据正确渲染且缓存与 API 正常工作。此时进入 [05 看板界面](./05-board-ui.md)。
