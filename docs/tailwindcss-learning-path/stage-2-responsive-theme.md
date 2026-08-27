# 阶段二：Tailwind CSS 响应式、主题与交互

**目标**：掌握断点、暗色模式、状态变体、动效和任意值，能够构建适配多种屏幕和交互状态的界面。

## 1. 响应式设计

### 1.1 断点

Tailwind 内置五个断点，全部是移动优先的 `min-width` 媒体查询：

| 前缀 | 最小宽度 | CSS |
|---|---|---|
| `sm` | 40rem (640px) | `@media (width >= 40rem)` |
| `md` | 48rem (768px) | `@media (width >= 48rem)` |
| `lg` | 64rem (1024px) | `@media (width >= 64rem)` |
| `xl` | 80rem (1280px) | `@media (width >= 80rem)` |
| `2xl` | 96rem (1536px) | `@media (width >= 96rem)` |

移动优先意味着默认样式面向小屏，断点前缀只在达到最小宽度时覆盖：

```html
<div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-4">
  <!-- 手机 1 列，平板 2 列，桌面 4 列 -->
</div>
```

### 1.2 响应式常用模式

```html
<!-- 文字大小随屏幕变化 -->
<h1 class="text-2xl md:text-4xl">响应式标题</h1>

<!-- 间距随屏幕变化 -->
<section class="p-4 md:p-8 lg:p-12">内容</section>

<!-- 侧边栏只在桌面显示 -->
<aside class="hidden lg:block w-64">侧边栏</aside>

<!-- 按钮在小屏占满、大屏自适应 -->
<button class="w-full md:w-auto px-4 py-2">操作</button>
```

### 1.3 任意断点值

需要非标准断点时使用任意值：

```html
<div class="min-[720px]:grid-cols-3">...</div>
<div class="max-md:flex-col">...</div>
```

## 2. 暗色模式

### 2.1 默认策略

v4 中 `dark:` 变体默认跟随系统配色（`prefers-color-scheme`）：

```html
<div class="bg-white dark:bg-gray-900">
  <h1 class="text-gray-900 dark:text-gray-100">标题</h1>
  <p class="text-gray-600 dark:text-gray-400">正文</p>
</div>
```

### 2.2 手动切换

需要手动切换主题（如网站内的明暗切换按钮）时，在 CSS 中配置基于 class 的暗色模式：

```css
@import "tailwindcss";
@custom-variant dark (&:where(.dark, .dark *));
```

```html
<html class="dark">
  <body class="bg-white dark:bg-gray-900">
    <!-- dark: 变体在 html.dark 下生效 -->
  </body>
</html>
```

```js
// 切换逻辑（放在头部脚本）
document.documentElement.classList.toggle('dark');
```

### 2.3 组合变体

变体可以组合使用，顺序从外到内：

```html
<button class="dark:lg:hover:bg-white">...</button>
```

含义：在暗色模式、大屏、hover 状态下应用该背景。

## 3. 状态变体

### 3.1 交互状态

| 变体 | 触发时机 |
|---|---|
| `hover:` | 鼠标悬停 |
| `focus:` | 获得焦点 |
| `focus-visible:` | 键盘聚焦（推荐用于可访问性） |
| `active:` | 按下 |
| `disabled:` | 禁用状态 |

```html
<button class="bg-blue-600 text-white rounded-md px-4 py-2
               hover:bg-blue-700 focus-visible:outline-2
               focus-visible:outline-offset-2 focus-visible:outline-blue-500
               active:bg-blue-800 disabled:opacity-50">
  保存
</button>
```

### 3.2 表单状态

```html
<input
  class="w-full rounded-md border border-gray-300 px-3 py-2
         focus:border-blue-500 focus:outline-none
         invalid:border-red-500
         disabled:bg-gray-100"
/>
```

### 3.3 group 变体

父元素加 `group`，子元素用 `group-hover:` 响应父元素状态：

```html
<div class="group rounded-lg border border-gray-200 p-4 hover:border-blue-300">
  <h3 class="text-gray-900">项目卡片</h3>
  <p class="text-gray-500">鼠标悬停在卡片上时，下面的文字变蓝：</p>
  <span class="text-gray-400 group-hover:text-blue-600 group-hover:underline">
    查看详情
  </span>
</div>
```

`group-focus-within:` 在组内任一元素聚焦时生效，常用于搜索框整体高亮。

## 4. 动效与过渡

### 4.1 过渡

| 工具类 | 作用 |
|---|---|
| `transition` | 过渡常用属性 |
| `transition-colors` | 只过渡颜色 |
| `duration-200` | 时长 200ms |
| `ease-out` | 缓动函数 |

```html
<button class="transition-colors duration-200 bg-blue-600 hover:bg-blue-700">
  悬停时颜色过渡
</button>

<a class="transition-transform duration-150 hover:scale-105">
  缩放效果
</a>
```

### 4.2 动画

Tailwind 内置常用动画，也可自定义：

```css
@theme {
  --animate-fade-in: fade-in 0.3s ease-out;
}

@keyframes fade-in {
  from { opacity: 0; }
  to { opacity: 1; }
}
```

```html
<div class="animate-fade-in">内容淡入</div>
```

动效原则：过渡时长控制在 150-300ms，减少动画数量，尊重 `prefers-reduced-motion`。

## 5. 任意值与主题变量

### 5.1 任意值

标准刻度不够时使用任意值（方括号内写任意 CSS 值）：

```html
<div class="w-[17rem]">固定宽度</div>
<div class="mt-[2px]">任意间距</div>
<div class="bg-[#123456]">任意颜色</div>
<div class="grid-cols-[1fr_2fr]">任意列比例</div>
<div class="p-[calc(1rem+2px)]">任意表达式</div>
```

优先使用主题变量和标准刻度，任意值用于一次性特殊场景。

### 5.2 CSS 变量

工具类也可以直接引用 CSS 变量：

```html
<div style="--brand-color: oklch(0.62 0.2 259)" class="bg-(--brand-color)">
  使用自定义变量
</div>
```

## 6. 常见排错

| 现象 | 排查方向 |
|---|---|
| 断点不生效 | 确认移动优先：默认写小屏，前缀只覆盖大屏 |
| `dark:` 不生效 | 确认是跟随系统还是基于 class 的自定义变体 |
| hover 效果在触屏上触发 | 触屏没有 hover，为触屏设计不同交互 |
| 过渡不生效 | 确认有起始状态、`transition-*` 和触发状态 |
| group-hover 不生效 | 确认父元素有 `group` 类 |
| 任意值不生效 | 检查方括号内语法和空格 |

## 阶段二验收

- 能使用断点前缀构建响应式布局。
- 能配置并实现暗色模式。
- 能使用状态变体表达交互状态。
- 能使用过渡和动画提升交互体验。
- 能使用任意值处理特殊场景。

## 动手任务

1. 将阶段一的项目改为响应式布局。
2. 为界面添加暗色模式，并支持系统跟随与手动切换。
3. 为按钮、输入框添加 hover、focus 和 disabled 状态。
4. 使用 group-hover 实现卡片悬停交互。
5. 为关键交互添加过渡动画，并验证 reduced-motion。

## 进入下一阶段的条件

你能够构建响应式、支持暗色模式且交互完整的界面。此时进入 [阶段三：组件化与进阶模式](./stage-3-components-advanced.md)。
