# Tailwind CSS 版本边界与迁移

**目标**：让学习内容、示例项目和生产项目拥有清晰的版本边界，降低 API 混用和升级风险。

## 1. 版本差异

| 能力 | v3 | v4 |
|---|---|---|
| 配置方式 | `tailwind.config.js` | CSS-first：`@theme`、`@custom-variant` |
| 引入方式 | `@tailwind base; @tailwind components; @tailwind utilities;` | `@import "tailwindcss";` |
| 构建集成 | PostCSS | `@tailwindcss/vite` 或 PostCSS |
| 深色模式 | 需配置 `darkMode: 'class'` | 默认系统跟随，class 用 `@custom-variant` |
| 自定义工具类 | `@layer utilities` | `@utility` |

## 2. 每个示例必须记录的元数据

```markdown
> 适用范围：Tailwind CSS v4
> 集成方式：Vite 插件 / PostCSS
> 需要的工具：Node.js、npm
> 验证命令：build、视觉回归
> 最后复核日期：YYYY-MM-DD
```

## 3. v3 到 v4 迁移

- 将 `@tailwind base/components/utilities` 替换为 `@import "tailwindcss";`。
- 将 `tailwind.config.js` 的 theme 配置迁移到 CSS `@theme` 变量。
- 将自定义颜色、字体、断点迁移为 `--color-*`、`--font-*`、`--breakpoint-*`。
- 将深色模式 `darkMode: 'class'` 迁移为 `@custom-variant dark`。
- 将 `@layer utilities` 中的自定义类迁移为 `@utility`。
- 检查移除的旧工具类和默认值变化。
- 阅读官方 [升级指南](https://tailwindcss.com/docs/upgrade-guide)。

## 4. 升级前检查

1. 阅读官方 changelog 和升级指南。
2. 记录当前 CSS 体积、构建时间和视觉基线。
3. 检查第三方组件库和主题的兼容性。
4. 为关键页面补充视觉回归测试。
5. 将升级拆成可独立回滚的小变更。

## 5. 升级后检查

- 开发与生产构建产物一致。
- 主题颜色、字体、断点全部生效。
- 暗色模式切换正确。
- 工具类未被移除的类覆盖。
- 构建体积没有异常增长。
- 无障碍和视觉回归测试通过。
