# 阶段一：Tailwind CSS 核心概念与布局

**目标**：掌握布局、间距、尺寸、颜色、排版和常用工具类，能够用工具类完成常见界面。

## 1. 布局与盒模型

### 1.1 盒模型工具类

| 工具类 | 作用 |
|---|---|
| `p-4`、`px-4`、`py-2` | padding（四边、水平、垂直） |
| `m-4`、`mx-auto`、`mt-8` | margin（四边、水平居中、上） |
| `w-64`、`w-full`、`max-w-3xl` | 宽度、100% 宽、最大宽度 |
| `h-16`、`h-screen`、`min-h-screen` | 高度、视口高、最小高度 |
| `border`、`border-2`、`border-gray-200` | 边框宽度与颜色 |
| `rounded`、`rounded-lg`、`rounded-full` | 圆角 |
| `box-border` | 统一盒模型（默认） |

```html
<div class="mx-auto max-w-3xl p-6">
  <div class="border border-gray-200 rounded-lg p-4">
    内容
  </div>
</div>
```

`mx-auto` 配合 `max-w-*` 实现水平居中内容，是中后台布局常用组合。

### 1.2 尺寸刻度

Tailwind 的间距和尺寸使用统一的刻度，4px 为基准：

| 类名 | 值 |
|---|---|
| `p-0` | 0 |
| `p-1` | 0.25rem (4px) |
| `p-2` | 0.5rem (8px) |
| `p-4` | 1rem (16px) |
| `p-6` | 1.5rem (24px) |
| `p-8` | 2rem (32px) |

统一刻度保证了全站间距和尺寸的一致性。

## 2. Flexbox 布局

### 2.1 基础

```html
<div class="flex items-center justify-between">
  <div>左侧</div>
  <div>右侧</div>
</div>
```

| 工具类 | 作用 |
|---|---|
| `flex` | `display: flex` |
| `flex-col` | 纵向排列 |
| `items-center` | 交叉轴居中 |
| `justify-between` | 主轴两端分布 |
| `gap-4` | 子元素间距 |
| `flex-1` | 平均分配剩余空间 |
| `flex-wrap` | 允许换行 |

### 2.2 导航栏示例

```html
<nav class="flex items-center justify-between border-b border-gray-200 bg-white px-6 py-4">
  <div class="text-lg font-bold text-gray-900">Logo</div>
  <ul class="flex gap-6">
    <li><a href="#" class="text-gray-700 hover:text-blue-600">首页</a></li>
    <li><a href="#" class="text-gray-700 hover:text-blue-600">文档</a></li>
    <li><a href="#" class="text-gray-700 hover:text-blue-600">关于</a></li>
  </ul>
  <button class="rounded-md bg-blue-600 px-4 py-2 text-white">登录</button>
</nav>
```

## 3. Grid 布局

### 3.1 基础

```html
<div class="grid grid-cols-3 gap-4">
  <div>列 1</div>
  <div>列 2</div>
  <div>列 3</div>
</div>
```

| 工具类 | 作用 |
|---|---|
| `grid` | `display: grid` |
| `grid-cols-3` | 三列等宽 |
| `grid-cols-[1fr_2fr]` | 自定义比例 |
| `gap-4` | 行和列间距 |
| `col-span-2` | 跨两列 |
| `row-span-2` | 跨两行 |
| `place-items-center` | 整体居中 |

### 3.2 卡片网格示例

```html
<div class="grid grid-cols-1 gap-4 md:grid-cols-2 lg:grid-cols-3">
  <article class="rounded-lg border border-gray-200 p-4 shadow-sm">
    <h3 class="font-semibold">卡片标题</h3>
    <p class="mt-2 text-sm text-gray-500">卡片描述文本。</p>
  </article>
  <article class="rounded-lg border border-gray-200 p-4 shadow-sm">
    <h3 class="font-semibold">卡片标题</h3>
    <p class="mt-2 text-sm text-gray-500">卡片描述文本。</p>
  </article>
</div>
```

`grid-cols-1 md:grid-cols-2 lg:grid-cols-3` 是响应式网格的经典写法：手机单列，平板两列，桌面三列。

## 4. 间距与排版

### 4.1 排版工具类

| 工具类 | 作用 |
|---|---|
| `text-sm`、`text-base`、`text-2xl` | 字体大小 |
| `font-bold`、`font-medium`、`font-semibold` | 字重 |
| `leading-6`、`leading-relaxed` | 行高 |
| `tracking-wide` | 字距 |
| `text-left`、`text-center` | 对齐 |
| `text-gray-900`、`text-white` | 文字颜色 |

### 4.2 文章示例

```html
<article class="prose mx-auto max-w-2xl p-6">
  <h1 class="text-3xl font-bold text-gray-900">文章标题</h1>
  <p class="mt-4 text-base leading-7 text-gray-600">
    这是正文内容，使用 text-base 和 leading-7 保证可读性。
  </p>
</article>
```

排版原则：标题用 `font-bold` 和较大字号，正文用 `leading-6` 或 `leading-7` 保证行高，次要信息用 `text-gray-500` 降低视觉权重。

## 5. 颜色系统

### 5.1 颜色刻度

Tailwind 的每种颜色有 50 到 950 的刻度：

```html
<div class="bg-blue-50">浅蓝背景</div>
<div class="bg-blue-500">主蓝色</div>
<div class="bg-blue-900">深蓝</div>
<div class="text-red-600">红色文字（错误提示）</div>
<div class="border-green-500">绿色边框（成功）</div>
```

### 5.2 常见语义

| 语义 | 常用颜色 |
|---|---|
| 主操作 | `blue-600` |
| 成功 | `green-500` / `emerald-500` |
| 警告 | `yellow-500` / `amber-500` |
| 错误 | `red-500` / `red-600` |
| 中性 | `gray-50` 到 `gray-900` |
| 强调 | `indigo-500`、`purple-500` |

### 5.3 状态颜色示例

```html
<button class="bg-blue-600 px-4 py-2 text-white rounded-md hover:bg-blue-700">
  保存
</button>
<button class="bg-red-50 px-4 py-2 text-red-600 rounded-md">
  删除
</button>
<span class="inline-block rounded-full bg-green-100 px-3 py-1 text-sm text-green-700">
  已上线
</span>
```

## 6. 常用组合模式

### 6.1 表单控件

```html
<label class="block text-sm font-medium text-gray-700">
  邮箱
  <input
    type="email"
    placeholder="you@example.com"
    class="mt-1 block w-full rounded-md border border-gray-300 px-3 py-2 text-sm shadow-sm focus:border-blue-500 focus:outline-none"
  />
</label>
```

### 6.2 徽章与提示

```html
<span class="inline-flex items-center rounded-full bg-blue-100 px-2.5 py-0.5 text-xs font-medium text-blue-700">
  新功能
</span>

<div class="rounded-md bg-red-50 p-4 text-sm text-red-700" role="alert">
  表单提交失败，请检查后重试。
</div>
```

### 6.3 表格

```html
<table class="min-w-full divide-y divide-gray-200 text-sm">
  <thead class="bg-gray-50">
    <tr>
      <th class="px-4 py-3 text-left font-medium text-gray-500">姓名</th>
      <th class="px-4 py-3 text-left font-medium text-gray-500">状态</th>
    </tr>
  </thead>
  <tbody class="divide-y divide-gray-100">
    <tr>
      <td class="px-4 py-3">Alice</td>
      <td class="px-4 py-3">
        <span class="rounded-full bg-green-100 px-2 py-0.5 text-green-700">在职</span>
      </td>
    </tr>
  </tbody>
</table>
```

## 7. 常见排错

| 现象 | 排查方向 |
|---|---|
| `mx-auto` 不居中 | 确认元素有明确宽度或 `max-w-*` |
| flex 子项不换行 | 加 `flex-wrap` |
| 间距不生效 | 确认 `gap-*` 用在 flex/grid 容器上 |
| 颜色太淡 | 尝试更高的刻度（600、700） |
| grid 列数不对 | 检查 `grid-cols-*` 和 `col-span-*` |

## 阶段一验收

- 能用 flex 和 grid 完成常见布局。
- 能用统一刻度控制间距和尺寸。
- 能用颜色系统表达语义（主操作、成功、错误）。
- 能完成导航、卡片、表单、表格等常见组件。

## 动手任务

1. 用 flex 实现一个左右布局的导航栏。
2. 用 grid 实现响应式卡片网格。
3. 实现一个登录表单，包含错误提示。
4. 实现带徽章和状态色的用户列表。
5. 用开发者工具测量并调整一个布局的间距。

## 进入下一阶段的条件

你能够用工具类独立完成常见页面布局与组件。此时进入 [阶段二：响应式、主题与交互](./stage-2-responsive-theme.md)。
