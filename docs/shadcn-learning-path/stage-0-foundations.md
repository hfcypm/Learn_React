# 阶段零：shadcn/ui 基础与工具链

**目标**：理解 shadcn/ui 的组件分发模式，初始化项目，添加第一个组件并理解项目结构。

## 1. shadcn/ui 是什么

shadcn/ui 是组件源码集合，通过 CLI 把组件源码复制进你的项目。它依赖 React、Tailwind CSS 与 Radix UI（无头组件），但你不需要直接管理这三者的组件层。

```text
CLI 添加组件 -> 源码进入 components/ui -> 基于 Tailwind + CSS 变量 + Radix
   -> 可读、可改、随项目主题
```

- 组件基于 Radix UI 构建，交互与可访问性来自 Radix。
- 样式用 Tailwind 工具类与 CSS 变量，随项目主题变化。
- 图标默认使用 lucide-react。

## 2. 前置环境

```bash
# 一个 Vite + React + TS 项目（或 Next.js 项目）
npm create vite@latest my-admin -- --template react-ts
cd my-admin
npm install

# 初始化 Tailwind CSS v4
npm install tailwindcss @tailwindcss/vite
```

Tailwind v4 用 Vite 插件与 CSS 导入接入，无需 `tailwind.config.js`。

## 3. 初始化 shadcn/ui

```bash
npx shadcn@latest init
```

CLI 会询问基础色与风格，随后生成：

- `components.json`：shadcn/ui 配置。
- `lib/utils.ts`：`cn` 类名合并工具。
- `components/ui/`：组件目录。
- 全局样式：把主题变量写入 CSS。

也可直接用脚手架创建带 shadcn 的 Next.js 或 Vite 项目：

```bash
npx shadcn@latest init -t next
npx shadcn@latest init -t vite
```

## 4. components.json

```json
{
  "$schema": "https://ui.shadcn.com/schema.json",
  "style": "base-nova",
  "rsc": false,
  "tsx": true,
  "tailwind": {
    "css": "src/styles/globals.css",
    "baseColor": "neutral",
    "cssVariables": true,
    "prefix": ""
  },
  "aliases": {
    "components": "@/components",
    "utils": "@/lib/utils",
    "ui": "@/components/ui",
    "lib": "@/lib",
    "hooks": "@/hooks"
  },
  "iconLibrary": "lucide"
}
```

- `style`：组件风格（决定默认视觉）。
- `aliases`：组件与工具函数的导入路径。
- `cssVariables`：是否用 CSS 变量驱动主题（默认开启）。
- `rsc`：是否面向 React Server Components（Next.js 中开启）。

## 5. 添加组件

```bash
npx shadcn@latest add button
npx shadcn@latest add card input label
```

一次添加多个组件：

```bash
npx shadcn@latest add button card input label badge dialog
```

命令把源码复制到 `components/ui/`，并自动安装 Radix 等依赖。

## 6. 使用组件

```tsx
import { Button } from "@/components/ui/button";

export function Hello() {
  return <Button variant="primary">你好，shadcn</Button>;
}
```

- 组件是源码，可直接在 `components/ui/button.tsx` 查看实现。
- 导入路径来自 `components.json` 的别名。

## 7. 动手任务

1. 创建 Vite + React + TS 项目并接入 Tailwind CSS v4。
2. 运行 `npx shadcn@latest init` 并查看生成的 components.json。
3. 添加 `button`、`card` 组件。
4. 用 Button 与 Card 拼一个简单卡片。
5. 阅读 `components/ui/button.tsx` 源码，找出 variant 的写法。

## 阶段零验收

- 能解释组件源码分发模式。
- 能初始化并配置 components.json。
- 能添加组件并正确导入使用。

## 常见排错

| 现象 | 排查方向 |
|---|---|
| init 报框架不支持 | 确认项目已接入 React + Tailwind |
| 组件导入失败 | 组件未 add，或别名路径不匹配 |
| Tailwind 样式缺失 | 确认 Tailwind v4 插件与 CSS 导入 |
| 别名 `@/` 报错 | 配置 tsconfig paths 与构建别名 |
| init 交互卡住 | 用脚手架 `-t` 参数或先手动配置 |

## 进入下一阶段的条件

你能够初始化并添加组件。此时进入 [阶段一：核心组件与主题定制](./stage-1-core-and-theming.md)。
