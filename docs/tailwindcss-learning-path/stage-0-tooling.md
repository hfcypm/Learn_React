# 阶段零：Tailwind CSS 工具链与基础

**目标**：理解 utility-first 思想、Tailwind 的安装配置与样式生成机制，能够创建并运行一个最小项目。

## 1. Tailwind CSS 是什么

Tailwind CSS 是一个 utility-first（工具类优先）的 CSS 框架。它不是提供预设组件，而是提供一组单一用途的工具类，例如 `p-4`、`flex`、`text-center`、`bg-blue-500`，直接在 HTML 中组合它们来构建界面。

### 1.1 utility-first 与组件 CSS 的对比

传统做法是为每个组件写一段 CSS：

```css
/* 传统 CSS */
.card {
  border-radius: 0.5rem;
  padding: 1rem;
  background-color: #fff;
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.1);
}
```

```html
<div class="card">...</div>
```

Tailwind 的做法是直接使用工具类组合：

```html
<div class="rounded-lg bg-white p-4 shadow-sm">...</div>
```

好处是样式与结构放在一起、容易看出布局、不会出现类名冲突、改动局部不影响全局。

### 1.2 Tailwind 如何工作

Tailwind 会扫描项目中的源文件（HTML、组件、模板），找出用到的类名，只为这些类生成 CSS。这与手写完整 CSS 文件完全不同：

```text
扫描源文件 -> 提取工具类 -> 生成对应 CSS -> 输出样式表
```

没有用到的类不会出现在产物中，这是控制体积的核心机制。

## 2. 安装与配置

### 2.1 与 Vite 集成

使用 Vite 官方插件（Tailwind CSS v4）：

```bash
# 创建 Vite 项目
npm create vite@latest my-app -- --template vanilla

# 进入项目
cd my-app

# 安装 Tailwind 与 Vite 插件
npm install tailwindcss @tailwindcss/vite
```

```js
// vite.config.js
import { defineConfig } from 'vite';
import tailwindcss from '@tailwindcss/vite';

export default defineConfig({
  plugins: [tailwindcss()],
});
```

在 CSS 入口文件顶部引入 Tailwind：

```css
/* src/style.css */
@import "tailwindcss";
```

```js
// src/main.js
import './style.css';
```

启动开发服务器：

```bash
npm run dev
```

### 2.2 与框架集成

| 框架 | 安装方式 |
|---|---|
| Vite | `@tailwindcss/vite` 插件 |
| Next.js | PostCSS 插件 `@tailwindcss/postcss` |
| 原生 HTML | PostCSS CLI 或 CDN（仅原型） |

Next.js 中配置 PostCSS：

```bash
npm install tailwindcss @tailwindcss/postcss
```

```js
// postcss.config.mjs
export default {
  plugins: {
    '@tailwindcss/postcss': {},
  },
};
```

在全局 CSS 引入：

```css
@import "tailwindcss";
```

## 3. 样式生成机制

### 3.1 扫描范围

v4 默认自动检测项目源文件，也可以通过 CSS 指定扫描路径：

```css
@import "tailwindcss";

@source "../components";
```

类名只有在源文件中以完整形式出现时才会生成。动态拼接类名会导致工具类丢失：

```html
<!-- 错误：build- 和 颜色名分开，Tailwind 扫描不到完整类名 -->
<div class="bg-{{ color }}">...</div>

<!-- 正确：颜色名作为完整类名写出来 -->
<div class="bg-red-500">...</div>
```

### 3.2 工具类结构

工具类通常遵循"属性-值"或"属性-修饰-值"的结构：

```html
<div class="p-4 text-lg font-bold bg-blue-500 rounded-lg">
  padding: 1rem; 字体大小: 1.125rem; 加粗; 蓝色背景; 圆角
</div>
```

| 类名 | 生成的 CSS |
|---|---|
| `p-4` | `padding: 1rem;` |
| `mt-2` | `margin-top: 0.5rem;` |
| `text-lg` | `font-size: 1.125rem;` |
| `font-bold` | `font-weight: 700;` |
| `bg-blue-500` | `background-color: var(--color-blue-500);` |
| `rounded-lg` | `border-radius: 0.5rem;` |

### 3.3 用开发者工具查看

打开浏览器开发者工具，检查元素可以看到 Tailwind 生成的 CSS 规则、类名对应的属性和值。这样能快速确认一个工具类实际产生什么效果。

## 4. 最小案例：卡片组件

```html
<div class="rounded-lg border border-gray-200 bg-white p-4 shadow-sm">
  <img
    src="/avatar.png"
    alt="用户头像"
    class="h-12 w-12 rounded-full"
  />
  <h2 class="mt-3 text-lg font-semibold text-gray-900">Alice</h2>
  <p class="text-sm text-gray-500">前端工程师</p>
  <button class="mt-4 rounded-md bg-blue-600 px-4 py-2 text-white">
    关注
  </button>
</div>
```

阅读顺序就是结构顺序：外层圆角边框和阴影 -> 内边距 -> 头像 -> 标题 -> 描述 -> 按钮。不需要为每个元素单独写 CSS。

## 5. 常见排错

| 现象 | 排查方向 |
|---|---|
| 样式完全没有生效 | 确认 `@import "tailwindcss"` 和插件配置 |
| 部分类不生效 | 确认类名被扫描、无拼写错误 |
| 动态拼接类不生效 | 改为完整类名或使用 safelist |
| 重启后样式丢失 | 确认插件在构建工具配置中注册 |
| 开发环境有样式、生产没有 | 检查生产构建是否包含扫描配置 |

## 阶段零验收

- 能安装 Tailwind 并接入一个框架。
- 能解释 utility-first 与组件 CSS 的区别。
- 能解释 Tailwind 扫描与按需生成的机制。
- 能创建一个使用工具类的页面。
- 能通过开发者工具确认生成的 CSS。

## 动手任务

1. 在 Vite 项目中接入 Tailwind。
2. 编写一个卡片和按钮组件。
3. 用开发者工具查看一个工具类的生成规则。
4. 尝试动态拼接类名，观察其为何失效。
5. 记录你常用的 15 个工具类及其作用。

## 进入下一阶段的条件

你能够解释 Tailwind 的生成机制并搭建可运行项目。此时进入 [阶段一：核心概念与布局](./stage-1-core-layout.md)。
