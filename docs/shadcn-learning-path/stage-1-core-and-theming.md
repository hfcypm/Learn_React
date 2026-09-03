# 阶段一：核心组件与主题定制

**目标**：掌握 shadcn/ui 的基础组件用法与 CSS 变量主题体系，能定制主题并切换暗色模式。

## 1. 基础组件

### Button

```tsx
import { Button } from "@/components/ui/button";

<Button variant="primary">主要操作</Button>
<Button variant="secondary">次要操作</Button>
<Button variant="outline">描边</Button>
<Button variant="ghost">幽灵</Button>
<Button variant="destructive">危险操作</Button>
<Button size="sm" disabled>小号禁用</Button>
```

- `variant` 与 `size` 由组件内部变体表驱动。
- 组件基于原生按钮，可接收标准 HTML 属性。

### Card

```tsx
import {
  Card,
  CardHeader,
  CardTitle,
  CardDescription,
  CardContent,
  CardFooter,
} from "@/components/ui/card";

<Card>
  <CardHeader>
    <CardTitle>订阅概览</CardTitle>
    <CardDescription>本月订阅数据</CardDescription>
  </CardHeader>
  <CardContent>内容区域</CardContent>
  <CardFooter className="flex justify-between">页脚操作</CardFooter>
</Card>
```

### Input、Label、Badge、Separator

```tsx
import { Input } from "@/components/ui/input";
import { Label } from "@/components/ui/label";
import { Badge } from "@/components/ui/badge";

<Label htmlFor="email">邮箱</Label>
<Input id="email" type="email" placeholder="you@example.com" />
<Badge variant="destructive">已过期</Badge>
```

## 2. 主题即 CSS 变量

shadcn/ui 的主题不用类名覆盖，而是由一组 CSS 变量驱动。变量在全局样式里定义：

```css
@import "tailwindcss";
@import "tw-animate-css";
@import "shadcn/tailwind.css";

@custom-variant dark (&:is(.dark *));

@theme inline {
  --color-primary: var(--primary);
  --color-primary-foreground: var(--primary-foreground);
  --color-background: var(--background);
  --color-card: var(--card);
  --color-muted: var(--muted);
  --color-border: var(--border);
  --radius-lg: var(--radius);
}
```

`:root` 里放浅色变量，`.dark` 里放暗色变量。组件全部引用这些语义变量，因此改变量即整体换肤。

## 3. 语义色板

| 变量 | 用途 |
|---|---|
| `--background` / `--foreground` | 页面背景与正文 |
| `--card` / `--card-foreground` | 卡片表面 |
| `--primary` / `--primary-foreground` | 主操作色 |
| `--secondary` | 次操作色 |
| `--muted` | 弱化背景 |
| `--accent` | 悬停/强调背景 |
| `--destructive` | 危险/错误 |
| `--border` / `--input` / `--ring` | 边框、输入框与焦点环 |
| `--chart-1..5` | 图表系列色 |
| `--sidebar-*` | 侧边栏色 |

变量值使用 oklch 颜色空间，便于在不同色板间保持一致感知亮度。

## 4. 定制主题

改主色只动变量：

```css
:root {
  --primary: oklch(0.55 0.2 259);      /* 品牌蓝 */
  --primary-foreground: oklch(0.985 0 0);
}
.dark {
  --primary: oklch(0.62 0.2 259);
  --primary-foreground: oklch(0.145 0 0);
}
```

半径同理：`--radius` 控制全局圆角。改变量后所有引用组件自动更新。

## 5. 暗色模式切换

shadcn 的主题类名是 `.dark`，切换只需在根元素加/去 class：

```tsx
"use client";

export function ThemeToggle() {
  return (
    <Button
      variant="ghost"
      onClick={() => document.documentElement.classList.toggle("dark")}
    >
      切换主题
    </Button>
  );
}
```

`@custom-variant dark` 让 `dark:` 工具类与 CSS 变量在 `.dark` 下生效。

## 6. 动手任务

1. 添加 button、card、input、badge 组件并组合一个设置面板。
2. 在全局样式中把主色改为品牌色，观察全局变化。
3. 实现 ThemeToggle，验证暗色切换。
4. 调整 `--radius` 观察圆角联动。
5. 阅读 card 与 button 源码，找出它们引用的 CSS 变量。

## 阶段一验收

- 能使用基础组件并理解 variant。
- 能解释 CSS 变量主题机制。
- 能定制主题色与半径。
- 能实现暗色模式切换。

## 常见排错

| 现象 | 排查方向 |
|---|---|
| 改色不生效 | 确认变量写在 :root 且组件引用对应语义色 |
| 暗色不切换 | 确认 `.dark` 变量与 @custom-variant |
| 组件色错乱 | 混淆了 primary 与 accent 语义 |
| 变量失效 | 检查 @theme inline 与变量导入顺序 |
| 主题切换闪跳 | 用 next-themes 或初始化时读存储值 |

## 进入下一阶段的条件

你能够定制主题。此时进入 [阶段二：交互组件与表单](./stage-2-interactions-and-forms.md)。
