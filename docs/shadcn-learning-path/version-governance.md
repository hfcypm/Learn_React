# shadcn/ui 版本边界与迁移

**目标**：让学习内容、示例项目和生产项目拥有清晰的版本边界，降低大版本升级和 API 混用风险。

## 1. 版本差异

| 能力 | 旧版本（v2/v3，Tailwind v3） | 当前主线（Tailwind CSS v4） |
|---|---|---|
| 样式配置 | `tailwind.config.js` 扩展 | CSS-first：`@import "tailwindcss"` |
| 主题文件 | `globals.css` 手动维护全部变量 | `@import "shadcn/tailwind.css"` 提供基础 |
| 暗色变体 | `dark:` 需配置 darkMode: class | `@custom-variant dark` |
| 颜色空间 | HSL 为主 | oklch 为主 |
| 安装风格 | `style` 字段（default/new-york） | 新风格名（如 base-nova） |
| 新组件 | 组件持续新增 | sidebar、chart 等已稳定 |
| CLI | `npx shadcn@latest` | 同前，支持 `init -t` 脚手架 |

## 2. 每个示例必须记录的元数据

```markdown
> 适用范围：shadcn/ui 当前稳定版 + Tailwind CSS v4 + React
> 需要的工具：Node.js、npm、Vite 或 Next.js
> 验证命令：npm run dev、npx shadcn@latest add <component>
> 最后复核日期：YYYY-MM-DD
```

## 3. 迁移要点（v3 -> v4 方向）

- Tailwind 接入改为 Vite 插件 + CSS 导入，移除 `tailwind.config.js` 主题扩展。
- 全局样式按官方 v4 模板重写：保留 `:root` 与 `.dark` 变量块，新增 `@theme inline` 映射。
- 添加 `@custom-variant dark` 使暗色工具类生效。
- 颜色变量可保留 HSL，但建议迁移到 oklch 以获得一致的视觉。
- 检查 `components.json` 的 `style` 与组件风格是否匹配。
- 重新 `npx shadcn@latest add` 覆盖到新基线，再合并自定义改动。

## 4. 升级前检查

1. 阅读官方文档的 installation 与 theming 章节。
2. 确认项目 Tailwind 主版本与构建方式。
3. 检查全局样式的变量与导入结构。
4. 检查被深度定制的组件，评估重新 add 的合并成本。
5. 在分支环境完整验证 init、add、dev 与 build。

## 5. 升级后检查

- init 与 add 正常，主题在浅色/暗色下正确。
- 组件导入路径与别名未破坏。
- 表单、表格、图表行为一致。
- 自定义变体与源码改动保留完整。
- 构建与生产部署通过。
