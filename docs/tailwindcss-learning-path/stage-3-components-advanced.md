# 阶段三：Tailwind CSS 组件化与进阶模式

**目标**：掌握组件抽取、@apply、CSS-first 主题配置、插件和设计一致性，能够组织可维护的样式体系。

## 1. 组件抽取

### 1.1 为什么抽取

工具类写在模板里会让类名很长，重复多次后难以维护。当一组工具类组合在多个地方重复时，应该抽取为组件。

### 1.2 框架组件抽取

在 React 中抽取为组件：

```tsx
// components/Button.tsx
type ButtonProps = {
  variant?: 'primary' | 'secondary' | 'danger';
  children: React.ReactNode;
} & React.ButtonHTMLAttributes<HTMLButtonElement>;

const variantClasses: Record<NonNullable<ButtonProps['variant']>, string> = {
  primary: 'bg-blue-600 text-white hover:bg-blue-700',
  secondary: 'bg-gray-100 text-gray-700 hover:bg-gray-200',
  danger: 'bg-red-600 text-white hover:bg-red-700',
};

export function Button({ variant = 'primary', className = '', children, ...rest }: ButtonProps) {
  return (
    <button
      className={`inline-flex items-center justify-center rounded-md px-4 py-2 text-sm font-medium transition-colors ${variantClasses[variant]} ${className}`}
      {...rest}
    >
      {children}
    </button>
  );
}
```

```tsx
// 使用
<Button variant="primary">保存</Button>
<Button variant="danger">删除</Button>
```

### 1.3 变体管理

组件变体是 Tailwind 组件的核心。用变体映射表集中管理：

```tsx
const badgeStyles = {
  success: 'bg-green-100 text-green-700',
  warning: 'bg-amber-100 text-amber-700',
  error: 'bg-red-100 text-red-700',
  neutral: 'bg-gray-100 text-gray-700',
} as const;

export function Badge({ status }: { status: keyof typeof badgeStyles }) {
  return (
    <span className={`inline-flex items-center rounded-full px-2.5 py-0.5 text-xs font-medium ${badgeStyles[status]}`}>
      {status}
    </span>
  );
}
```

变体映射表让类名集中、类型安全，避免散落的条件拼接。

## 2. @apply 与 CSS 组织

### 2.1 @apply 的用途

需要在 CSS 中复用工具类样式时使用 `@apply`：

```css
@import "tailwindcss";

.btn {
  @apply inline-flex items-center justify-center rounded-md px-4 py-2 text-sm font-medium transition-colors;
}

.btn-primary {
  @apply bg-blue-600 text-white hover:bg-blue-700;
}

.btn-danger {
  @apply bg-red-600 text-white hover:bg-red-700;
}
```

```html
<button class="btn btn-primary">保存</button>
<button class="btn btn-danger">删除</button>
```

### 2.2 @apply 的使用边界

- 适合把"重复出现的工具类组合"命名成语义化类。
- 适合与第三方 CSS、旧的全局样式共存。
- 不要滥用：如果项目是纯 React/Vue 组件，优先用组件抽取而不是 @apply。
- @apply 内的类必须是 Tailwind 能识别的工具类。

```css
/* 错误：@apply 使用了不存在的类 */
.card {
  @apply my-custom-class;
}
```

## 3. CSS-first 主题配置

### 3.1 @theme 定义设计变量

v4 使用 CSS 变量定义主题，在 CSS 文件中完成配置：

```css
@import "tailwindcss";

@theme {
  /* 颜色 */
  --color-brand-50: oklch(0.97 0.02 240);
  --color-brand-100: oklch(0.94 0.05 240);
  --color-brand-500: oklch(0.62 0.2 259);
  --color-brand-600: oklch(0.55 0.22 259);
  --color-brand-700: oklch(0.48 0.22 259);

  /* 字体 */
  --font-sans: "Inter", "PingFang SC", sans-serif;

  /* 断点 */
  --breakpoint-3xl: 120rem;

  /* 动画 */
  --animate-fade-in: fade-in 0.3s ease-out;

  /* 阴影 */
  --shadow-card: 0 1px 3px rgb(0 0 0 / 0.1);
}
```

定义后自动生成工具类：

```html
<div class="bg-brand-500 text-white">品牌色</div>
<button class="hover:bg-brand-600">悬停变深</button>
<div class="shadow-card">卡片阴影</div>
<div class="animate-fade-in">淡入</div>
```

### 3.2 覆盖默认主题

在 `@theme` 中定义与默认同名的变量即可覆盖：

```css
@theme {
  /* 覆盖默认主色为品牌色 */
  --color-blue-500: oklch(0.62 0.2 259);
}
```

### 3.3 主题组织建议

```text
styles/
├── globals.css        # @import tailwindcss 与重置
├── theme.css          # @theme 设计变量
└── components.css     # @apply 组件类（可选）
```

```css
/* globals.css */
@import "tailwindcss";
@import "./theme.css";
@import "./components.css";
```

设计变量集中在 theme 文件，业务样式尽量用工具类直接表达。

## 4. 自定义变体与函数

### 4.1 自定义变体

用 `@custom-variant` 定义自己的变体：

```css
@custom-variant hocus (&:hover, &:focus-visible);
```

```html
<button class="hocus:bg-blue-700">悬停或键盘聚焦时变色</button>
```

### 4.2 @utility 自定义工具类

用 `@utility` 定义可复用、支持变体的工具类：

```css
@utility tabular-nums {
  font-variant-numeric: tabular-nums;
}
```

```html
<table class="tabular-nums">数字等宽对齐</table>
```

### 4.3 官方插件

Tailwind 官方插件扩展能力：

| 插件 | 用途 |
|---|---|
| `@tailwindcss/typography` | 文章排版（prose） |
| `@tailwindcss/forms` | 表单控件统一样式 |
| `@tailwindcss/aspect-ratio` | 宽高比 |

```css
@plugin "@tailwindcss/typography";
```

```html
<article class="prose prose-gray mx-auto max-w-2xl">
  <h1>文章标题</h1>
  <p>自动获得良好排版。</p>
</article>
```

## 5. 设计一致性与规范

### 5.1 建立设计变量

- 颜色：主色、语义色、中性色统一命名。
- 字体：正文字体、标题字体、等宽字体。
- 间距：沿用默认刻度或定义业务刻度。
- 圆角：sm、md、lg、full。
- 阴影：卡片、弹出层、悬浮。

### 5.2 审查类名

- 组件通过 props 控制变体，避免在调用处散落重复类名。
- 类名顺序保持一致（布局 -> 尺寸 -> 颜色 -> 文本 -> 状态）。
- 使用 Prettier 的 `prettier-plugin-tailwindcss` 自动排序类名。

```bash
npm install -D prettier prettier-plugin-tailwindcss
```

```json
// .prettierrc
{
  "plugins": ["prettier-plugin-tailwindcss"]
}
```

## 6. 常见排错

| 现象 | 排查方向 |
|---|---|
| @apply 报错 | 确认使用的是工具类且类名完整 |
| 自定义颜色不生成工具类 | 确认变量在 `@theme` 中定义 |
| 插件不生效 | 确认 `@plugin` 与安装的包名一致 |
| 变体不生效 | 确认 `@custom-variant` 语法正确 |
| 类名顺序不一致 | 接入 prettier 插件自动排序 |

## 阶段三验收

- 能将重复工具类组合抽取为组件并管理变体。
- 能使用 @apply 组织语义化 CSS 类。
- 能通过 @theme 定义和覆盖主题变量。
- 能使用插件、自定义变体和工具类扩展能力。
- 能建立并维护设计一致性。

## 动手任务

1. 将按钮、徽章、卡片抽取为组件并支持变体。
2. 使用 @theme 定义品牌色和业务变量。
3. 接入 typography 插件排版一篇文章。
4. 定义并应用一个自定义变体。
5. 配置 prettier 插件统一类名顺序。

## 进入下一阶段的条件

你能够组织可维护的组件样式和主题体系。此时进入 [阶段四：工程化与生产交付](./stage-4-production.md)。
