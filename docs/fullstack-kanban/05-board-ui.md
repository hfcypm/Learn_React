# 05 看板界面：Tailwind CSS v4 实战

**目标**：用 Tailwind CSS v4 搭建看板界面，覆盖布局、响应式、暗色模式、组件变体、@theme 设计变量、动效与无障碍。

## 1. 全局样式与主题变量

```css
/* app/globals.css */
@import "tailwindcss";

@theme {
  --color-brand-50:  oklch(0.97 0.02 259);
  --color-brand-100: oklch(0.94 0.05 259);
  --color-brand-500: oklch(0.62 0.2 259);
  --color-brand-600: oklch(0.55 0.22 259);
  --color-brand-700: oklch(0.48 0.22 259);

  --font-sans: "Inter", "PingFang SC", "Microsoft YaHei", sans-serif;

  --shadow-card: 0 1px 3px rgb(0 0 0 / 0.1), 0 1px 2px rgb(0 0 0 / 0.06);
}

/* 基于 class 的暗色模式（手动切换） */
@custom-variant dark (&:where(.dark, .dark *));
```

使用 `@theme` 定义品牌色、字体与阴影，页面中通过 `bg-brand-600`、`shadow-card` 引用，避免散落魔法值。

## 2. 布局骨架

顶栏 + 侧边栏 + 内容区，用 flex 与响应式控制：

```tsx
// components/dashboard-layout.tsx
export function DashboardLayout({ children }: { children: React.ReactNode }) {
  return (
    <div className="flex min-h-screen bg-gray-50 dark:bg-gray-950">
      <aside className="hidden w-64 flex-col border-r border-gray-200 bg-white lg:flex dark:border-gray-800 dark:bg-gray-900">
        <nav className="flex-1 px-3 py-4">
          <a className="mb-2 block rounded-md px-3 py-2 text-sm text-gray-700 hover:bg-gray-100 dark:text-gray-200 dark:hover:bg-gray-800" href="/dashboard">
            总览
          </a>
          <a className="mb-2 block rounded-md px-3 py-2 text-sm text-gray-700 hover:bg-gray-100 dark:text-gray-200 dark:hover:bg-gray-800" href="/teams">
            团队
          </a>
        </nav>
      </aside>
      <div className="flex flex-1 flex-col">
        <header className="flex items-center justify-between border-b border-gray-200 bg-white px-6 py-3 dark:border-gray-800 dark:bg-gray-900">
          <h1 className="text-base font-semibold text-gray-900 dark:text-gray-100">看板</h1>
          <ThemeToggle />
        </header>
        <main className="flex-1 p-6">{children}</main>
      </div>
    </div>
  );
}
```

知识点：`hidden lg:flex`（移动端隐藏、桌面显示）、flex 布局、`border`、语义化背景色、暗色变体成对书写。

## 3. 组件变体抽取

把可复用的按钮、徽章、标签抽成带变体的组件：

```tsx
// components/ui/button.tsx
type ButtonProps = {
  variant?: 'primary' | 'secondary' | 'danger' | 'ghost';
  children: React.ReactNode;
} & React.ButtonHTMLAttributes<HTMLButtonElement>;

const variants = {
  primary: 'bg-brand-600 text-white hover:bg-brand-700',
  secondary: 'bg-gray-100 text-gray-700 hover:bg-gray-200 dark:bg-gray-800 dark:text-gray-200',
  danger: 'bg-red-600 text-white hover:bg-red-700',
  ghost: 'text-gray-600 hover:bg-gray-100 dark:text-gray-300 dark:hover:bg-gray-800',
};

export function Button({ variant = 'primary', className = '', children, ...rest }: ButtonProps) {
  return (
    <button
      className={`inline-flex items-center justify-center rounded-md px-4 py-2 text-sm font-medium transition-colors focus-visible:outline-2 focus-visible:outline-offset-2 focus-visible:outline-brand-600 disabled:opacity-50 ${variants[variant]} ${className}`}
      {...rest}
    >
      {children}
    </button>
  );
}
```

知识点：变体映射表、`cn` 合并、`focus-visible` 键盘焦点、`disabled:opacity-50`、`transition-colors`。

## 4. 状态徽章与优先级

```tsx
// components/ui/badge.tsx
const statusStyles: Record<string, string> = {
  TODO: 'bg-gray-100 text-gray-700 dark:bg-gray-800 dark:text-gray-300',
  IN_PROGRESS: 'bg-amber-100 text-amber-700 dark:bg-amber-900/50 dark:text-amber-400',
  DONE: 'bg-green-100 text-green-700 dark:bg-green-900/50 dark:text-green-400',
};

const priorityStyles: Record<string, string> = {
  LOW: 'bg-gray-100 text-gray-600',
  MEDIUM: 'bg-blue-100 text-blue-700',
  HIGH: 'bg-orange-100 text-orange-700',
  URGENT: 'bg-red-100 text-red-700',
};

export function StatusBadge({ status }: { status: string }) {
  return (
    <span className={`inline-flex items-center rounded-full px-2.5 py-0.5 text-xs font-medium ${statusStyles[status] ?? ''}`}>
      {status}
    </span>
  );
}

export function PriorityBadge({ priority }: { priority: string }) {
  return (
    <span className={`inline-flex items-center rounded px-1.5 py-0.5 text-xs font-medium ${priorityStyles[priority] ?? ''}`}>
      {priority}
    </span>
  );
}
```

知识点：语义颜色、暗色模式、`rounded-full` 胶囊、`/50` 透明度变体。

## 5. 任务卡片

```tsx
// components/task-card.tsx
export function TaskCard({ task }: { task: any }) {
  return (
    <div className="group rounded-lg border border-gray-200 bg-white p-4 shadow-card transition-shadow hover:shadow-lg dark:border-gray-800 dark:bg-gray-900">
      <div className="flex items-start justify-between gap-2">
        <h3 className="text-sm font-medium text-gray-900 dark:text-gray-100">{task.title}</h3>
        <PriorityBadge priority={task.priority} />
      </div>

      {task.description && (
        <p className="mt-2 text-sm text-gray-600 line-clamp-2 dark:text-gray-400">{task.description}</p>
      )}

      <div className="mt-4 flex items-center justify-between">
        <div className="flex gap-1">
          {task.tags?.map(({ tag }: any) => (
            <span key={tag.id} className="rounded bg-brand-50 px-1.5 py-0.5 text-xs text-brand-700 dark:bg-brand-900/40 dark:text-brand-300">
              {tag.name}
            </span>
          ))}
        </div>
        {task.assignee && (
          <span className="flex h-6 w-6 items-center justify-center rounded-full bg-brand-600 text-xs text-white" title={task.assignee.name}>
            {task.assignee.name[0]}
          </span>
        )}
      </div>

      <div className="mt-3 opacity-0 transition-opacity group-hover:opacity-100">
        <Button variant="secondary" size="sm">
          移动
        </Button>
      </div>
    </div>
  );
}
```

知识点：`group` + `group-hover` 显示操作按钮、`line-clamp-2` 文本截断、`shadow-card` 主题阴影、头像圆形裁剪。

## 6. 看板列与响应式网格

```tsx
// components/board-columns.tsx
export function BoardColumns({ columns }: { columns: { name: string; tasks: any[] }[] }) {
  return (
    <div className="grid grid-cols-1 gap-4 sm:grid-cols-2 xl:grid-cols-3">
      {columns.map((col) => (
        <section key={col.name} className="rounded-lg bg-gray-100 p-3 dark:bg-gray-900">
          <header className="flex items-center justify-between px-1 pb-2">
            <h2 className="text-sm font-semibold text-gray-700 dark:text-gray-300">{col.name}</h2>
            <span className="rounded-full bg-gray-200 px-2 text-xs text-gray-600 dark:bg-gray-800 dark:text-gray-400">
              {col.tasks.length}
            </span>
          </header>
          <div className="space-y-3">
            {col.tasks.map((task) => (
              <TaskCard key={task.id} task={task} />
            ))}
          </div>
        </section>
      ))}
    </div>
  );
}
```

知识点：`grid-cols-1 -> sm:grid-cols-2 -> xl:grid-cols-3` 移动优先断点、`space-y` 间距、暗色模式。

## 7. 主题切换

```tsx
// components/theme-toggle.tsx
'use client';

import { useEffect, useState } from 'react';

export function ThemeToggle() {
  const [dark, setDark] = useState(false);

  useEffect(() => {
    document.documentElement.classList.toggle('dark', dark);
  }, [dark]);

  return (
    <button
      onClick={() => setDark((d) => !d)}
      className="rounded-md px-3 py-1.5 text-sm text-gray-600 hover:bg-gray-100 dark:text-gray-300 dark:hover:bg-gray-800"
    >
      {dark ? '亮色模式' : '暗色模式'}
    </button>
  );
}
```

要点：`@custom-variant dark` 让 `dark:` 变体响应 `.dark` class；切换逻辑保留在 Client Component。

## 8. 表单样式

```tsx
// components/task-form.tsx
export function TaskForm({ boardId }: { boardId: number }) {
  return (
    <form action={createTask.bind(null, boardId)} className="space-y-4">
      <div>
        <label htmlFor="title" className="block text-sm font-medium text-gray-700 dark:text-gray-300">
          标题
        </label>
        <input
          id="title"
          name="title"
          className="mt-1 block w-full rounded-md border border-gray-300 px-3 py-2 text-sm focus:border-brand-500 focus:ring-2 focus:ring-brand-500/40 dark:border-gray-700 dark:bg-gray-800 dark:text-gray-100"
        />
      </div>
      <Button type="submit">创建任务</Button>
    </form>
  );
}
```

知识点：`label` 关联 `htmlFor`、`focus:` 与 `focus:ring` 交互反馈、暗色表单。

## 9. 动效与无障碍

- 用 `transition-colors`、`transition-shadow`、`hover:` 提供轻量反馈。
- 尊重 `prefers-reduced-motion`：

```css
@media (prefers-reduced-motion: reduce) {
  * {
    animation-duration: 0.01ms !important;
    transition-duration: 0.01ms !important;
  }
}
```

- 所有可交互元素用 `focus-visible:` 显示焦点轮廓。
- 按钮用真实 `<button>`，导航用 `<a>`，输入框关联 `<label>`。

## 10. 本章验收

- [ ] 看板布局在手机、平板、桌面三档宽度下正确。
- [ ] 暗色模式完整覆盖卡片、表单、侧边栏。
- [ ] 按钮、徽章、标签均为带变体的复用组件。
- [ ] 无散落的魔法色值，主题变量统一。
- [ ] 键盘可遍历并可见焦点轮廓。
- [ ] 动效符合 reduced-motion 偏好。

## 11. 常见排错

| 现象 | 排查方向 |
|---|---|
| 暗色不生效 | 确认 `@custom-variant dark` 与 `.dark` class |
| 断点错乱 | 检查移动优先：默认写小屏，前缀覆盖大屏 |
| 类名不生效 | 确认类名被扫描（动态拼接会被跳过） |
| 主题色缺失 | 确认 `@theme` 变量名与 `--color-*` 规范 |
| focus 轮廓混乱 | 用 `focus-visible:` 而非 `focus:` |
| 组件类名过长 | 抽取为组件或 `@apply` 集中管理 |

## 进入下一章的条件

界面样式完整且交互可用。此时进入 [06 查询与事务](./06-advanced.md)。
