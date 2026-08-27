# Tailwind CSS 综合实战：后台管理界面

## 1. 项目目标

构建一个带响应式布局、暗色模式、数据卡片、表单和表格的后台管理界面。从布局骨架开始，按阶段逐步增加主题、交互、组件抽取和体积优化，最终完成一个可交付的界面。

```text
侧边栏导航 -> 顶栏与搜索 -> 数据卡片
    -> 数据表格 -> 表单与校验状态 -> 暗色模式
    -> 组件抽取 -> 体积与无障碍检查
```

前端使用 Vite + Tailwind CSS v4，组件用 React 实现，数据使用静态 Mock 数据。

## 2. 技术选择

| 技术 | 用途 |
|---|---|
| Vite | 构建工具 |
| Tailwind CSS v4 | 样式系统 |
| React | 组件化 |
| TypeScript | 类型安全 |
| prettier-plugin-tailwindcss | 类名排序 |

## 3. 初始化项目

```bash
# 创建项目
npm create vite@latest admin-dashboard -- --template react-ts

# 进入项目
cd admin-dashboard

# 安装依赖
npm install tailwindcss @tailwindcss/vite
npm install -D prettier prettier-plugin-tailwindcss
```

```js
// vite.config.ts
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import tailwindcss from '@tailwindcss/vite';

export default defineConfig({
  plugins: [react(), tailwindcss()],
});
```

```css
/* src/index.css */
@import "tailwindcss";
```

## 4. 目录结构

```text
src/
├── main.tsx
├── index.css          # @import tailwindcss 与 @theme
├── components/
│   ├── layout/
│   │   ├── sidebar.tsx
│   │   ├── header.tsx
│   │   └── dashboard-layout.tsx
│   ├── ui/
│   │   ├── button.tsx
│   │   ├── badge.tsx
│   │   ├── card.tsx
│   │   └── input.tsx
│   ├── stat-card.tsx
│   └── data-table.tsx
├── pages/
│   ├── overview.tsx
│   └── users.tsx
└── data/
    └── users.ts        # Mock 数据
```

## 5. 主题配置

```css
/* src/index.css */
@import "tailwindcss";

@theme {
  --color-brand-50: oklch(0.97 0.02 259);
  --color-brand-100: oklch(0.94 0.05 259);
  --color-brand-500: oklch(0.62 0.2 259);
  --color-brand-600: oklch(0.55 0.22 259);
  --color-brand-700: oklch(0.48 0.22 259);

  --font-sans: "Inter", "PingFang SC", "Microsoft YaHei", sans-serif;

  --shadow-card: 0 1px 3px rgb(0 0 0 / 0.1), 0 1px 2px rgb(0 0 0 / 0.06);
}

/* 基于 class 的暗色模式 */
@custom-variant dark (&:where(.dark, .dark *));
```

## 6. 布局骨架

### 侧边栏

```tsx
// components/layout/sidebar.tsx
const navItems = [
  { href: '/', label: '概览', icon: '📊' },
  { href: '/users', label: '用户', icon: '👥' },
  { href: '/settings', label: '设置', icon: '⚙️' },
];

export function Sidebar() {
  return (
    <aside className="hidden lg:flex w-64 flex-col border-r border-gray-200 bg-white dark:border-gray-800 dark:bg-gray-900">
      <div className="px-6 py-4 text-lg font-bold text-gray-900 dark:text-gray-100">
        Admin
      </div>
      <nav className="flex-1 px-3">
        <ul className="space-y-1">
          {navItems.map((item) => (
            <li key={item.href}>
              <a
                href={item.href}
                className="flex items-center gap-3 rounded-md px-3 py-2 text-sm text-gray-600 hover:bg-gray-100 hover:text-gray-900 dark:text-gray-400 dark:hover:bg-gray-800 dark:hover:text-gray-100"
              >
                <span>{item.icon}</span>
                {item.label}
              </a>
            </li>
          ))}
        </ul>
      </nav>
    </aside>
  );
}
```

### 布局容器

```tsx
// components/layout/dashboard-layout.tsx
import { Sidebar } from './sidebar';
import { Header } from './header';

export function DashboardLayout({ children }: { children: React.ReactNode }) {
  return (
    <div className="flex min-h-screen bg-gray-50 dark:bg-gray-950">
      <Sidebar />
      <div className="flex flex-1 flex-col">
        <Header />
        <main className="flex-1 p-6">{children}</main>
      </div>
    </div>
  );
}
```

### 顶栏与主题切换

```tsx
// components/layout/header.tsx
'use client';

import { useState } from 'react';

export function Header() {
  const [dark, setDark] = useState(false);

  function toggleTheme() {
    const next = !dark;
    setDark(next);
    document.documentElement.classList.toggle('dark', next);
  }

  return (
    <header className="flex items-center justify-between border-b border-gray-200 bg-white px-6 py-3 dark:border-gray-800 dark:bg-gray-900">
      <input
        placeholder="搜索..."
        className="w-64 rounded-md border border-gray-300 px-3 py-1.5 text-sm dark:border-gray-700 dark:bg-gray-800 dark:text-gray-100"
      />
      <button
        onClick={toggleTheme}
        className="rounded-md px-3 py-1.5 text-sm text-gray-600 hover:bg-gray-100 dark:text-gray-300 dark:hover:bg-gray-800"
      >
        {dark ? '切换到亮色' : '切换到暗色'}
      </button>
    </header>
  );
}
```

## 7. 数据卡片

```tsx
// components/stat-card.tsx
type StatCardProps = {
  label: string;
  value: string;
  trend: string;
  trendDirection: 'up' | 'down';
};

export function StatCard({ label, value, trend, trendDirection }: StatCardProps) {
  const trendColor =
    trendDirection === 'up'
      ? 'bg-green-100 text-green-700 dark:bg-green-900/50 dark:text-green-400'
      : 'bg-red-100 text-red-700 dark:bg-red-900/50 dark:text-red-400';

  return (
    <div className="rounded-lg border border-gray-200 bg-white p-5 shadow-sm dark:border-gray-800 dark:bg-gray-900">
      <p className="text-sm text-gray-500 dark:text-gray-400">{label}</p>
      <p className="mt-2 text-2xl font-bold text-gray-900 dark:text-gray-100">{value}</p>
      <span className={`mt-3 inline-block rounded-full px-2 py-0.5 text-xs font-medium ${trendColor}`}>
        {trend}
      </span>
    </div>
  );
}
```

```tsx
// pages/overview.tsx
import { StatCard } from '../components/stat-card';

export function OverviewPage() {
  return (
    <div className="grid grid-cols-1 gap-4 sm:grid-cols-2 xl:grid-cols-4">
      <StatCard label="总用户" value="12,480" trend="+12%" trendDirection="up" />
      <StatCard label="活跃用户" value="3,210" trend="+5%" trendDirection="up" />
      <StatCard label="本月订单" value="892" trend="-3%" trendDirection="down" />
      <StatCard label="收入" value="¥48,300" trend="+18%" trendDirection="up" />
    </div>
  );
}
```

## 8. 数据表格

```tsx
// components/data-table.tsx
import { Badge } from './ui/badge';
import { users } from '../data/users';

const statusMap = {
  active: { label: '正常', color: 'success' },
  pending: { label: '待审核', color: 'warning' },
  suspended: { label: '已停用', color: 'error' },
} as const;

export function DataTable() {
  return (
    <div className="overflow-hidden rounded-lg border border-gray-200 bg-white shadow-sm dark:border-gray-800 dark:bg-gray-900">
      <table className="min-w-full divide-y divide-gray-200 text-sm dark:divide-gray-800">
        <thead className="bg-gray-50 dark:bg-gray-800/50">
          <tr>
            <th className="px-4 py-3 text-left font-medium text-gray-500 dark:text-gray-400">姓名</th>
            <th className="px-4 py-3 text-left font-medium text-gray-500 dark:text-gray-400">邮箱</th>
            <th className="px-4 py-3 text-left font-medium text-gray-500 dark:text-gray-400">角色</th>
            <th className="px-4 py-3 text-left font-medium text-gray-500 dark:text-gray-400">状态</th>
          </tr>
        </thead>
        <tbody className="divide-y divide-gray-100 dark:divide-gray-800">
          {users.map((user) => (
            <tr key={user.id} className="hover:bg-gray-50 dark:hover:bg-gray-800/50">
              <td className="px-4 py-3 font-medium text-gray-900 dark:text-gray-100">{user.name}</td>
              <td className="px-4 py-3 text-gray-600 dark:text-gray-400">{user.email}</td>
              <td className="px-4 py-3 text-gray-600 dark:text-gray-400">{user.role}</td>
              <td className="px-4 py-3">
                <Badge status={statusMap[user.status].color}>
                  {statusMap[user.status].label}
                </Badge>
              </td>
            </tr>
          ))}
        </tbody>
      </table>
    </div>
  );
}
```

## 9. UI 组件抽取

```tsx
// components/ui/button.tsx
type ButtonProps = {
  variant?: 'primary' | 'secondary' | 'danger';
  children: React.ReactNode;
} & React.ButtonHTMLAttributes<HTMLButtonElement>;

const variants = {
  primary: 'bg-brand-600 text-white hover:bg-brand-700',
  secondary: 'bg-gray-100 text-gray-700 hover:bg-gray-200 dark:bg-gray-800 dark:text-gray-200',
  danger: 'bg-red-600 text-white hover:bg-red-700',
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

```tsx
// components/ui/badge.tsx
type BadgeStatus = 'success' | 'warning' | 'error' | 'neutral';

const badgeStyles: Record<BadgeStatus, string> = {
  success: 'bg-green-100 text-green-700 dark:bg-green-900/50 dark:text-green-400',
  warning: 'bg-amber-100 text-amber-700 dark:bg-amber-900/50 dark:text-amber-400',
  error: 'bg-red-100 text-red-700 dark:bg-red-900/50 dark:text-red-400',
  neutral: 'bg-gray-100 text-gray-700 dark:bg-gray-800 dark:text-gray-400',
};

export function Badge({ status, children }: { status: BadgeStatus; children: React.ReactNode }) {
  return (
    <span className={`inline-flex items-center rounded-full px-2.5 py-0.5 text-xs font-medium ${badgeStyles[status]}`}>
      {children}
    </span>
  );
}
```

## 10. 测试计划

| 层级 | 覆盖内容 |
|---|---|
| 组件测试 | Button 变体、Badge 状态、StatCard 渲染 |
| 视觉回归 | 概览页、暗色模式、不同断点截图 |
| 手动检查 | 键盘焦点、对比度、reduced-motion、构建体积 |

## 11. 验收清单

- [ ] 布局响应式：手机单列，桌面侧边栏加内容区。
- [ ] 暗色模式可切换，组件和颜色全部覆盖。
- [ ] 数据卡片、表格、表单样式一致。
- [ ] 按钮、徽章、卡片抽取为组件并支持变体。
- [ ] 主题变量统一，无散落的魔法值。
- [ ] 所有可交互元素有 focus-visible 轮廓。
- [ ] 文字对比度满足可读性。
- [ ] 构建产物 CSS 体积在合理范围。
- [ ] prettier 插件统一类名顺序。
- [ ] 生产构建成功并可部署。

## 12. 实施顺序

1. 创建项目，接入 Tailwind，完成全局布局。
2. 完成侧边栏、顶栏和内容区。
3. 完成数据卡片和数据表格。
4. 完成暗色模式与主题切换。
5. 抽取 UI 组件并管理变体。
6. 完成表单与校验状态。
7. 补齐测试、体积和无障碍检查。
8. 运行验收清单并记录待改进项。

## 13. 按阶段学习卡片

| 阶段 | 项目增量 |
|---|---|
| 零 | 初始化、接入 Tailwind、全局样式 |
| 一 | 布局骨架、导航、卡片、表格 |
| 二 | 响应式网格、暗色模式、状态与动效 |
| 三 | 组件抽取、@theme、变体、插件 |
| 四 | 体积优化、无障碍、测试、部署 |
