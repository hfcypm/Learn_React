# TypeScript 版本治理与迁移指南

**目标**：稳定管理 TypeScript、运行时、构建工具和类型声明之间的兼容关系。

## 1. 版本边界

| 层级 | 需要锁定的内容 |
|---|---|
| 运行时 | Node.js、浏览器目标、服务端运行时 |
| 编译器 | TypeScript 版本和 `tsconfig` 继承关系 |
| 构建 | Vite、Webpack、Next.js、SWC 或 esbuild |
| 类型声明 | `@types/node`、`@types/react` 等 |
| 测试 | 测试运行器、DOM 环境和测试类型 |
| 依赖 | 包管理器、lockfile、workspace 配置 |

每个示例项目需要在 README 中说明适用版本，并以 lockfile 固定实际安装结果。

## 2. 升级前检查

1. 阅读 TypeScript release notes 和 breaking changes。
2. 保存当前 `tsc --noEmit`、测试、构建和包体积基线。
3. 检查构建工具、测试工具、Lint 插件和编辑器支持。
4. 检查第三方类型声明是否使用了新的 TypeScript 特性。
5. 先在独立分支升级并修复错误分类。

错误通常分为：

- 编译器行为变化
- 标准库类型变化
- 第三方声明变严格
- 模块解析变化
- JSX 或构建工具处理变化
- 测试环境类型变化

## 3. JavaScript 到 TypeScript 迁移

建议顺序：

1. 增加 `allowJs`，让 JavaScript 与 TypeScript 共存。
2. 先迁移公共类型、API 模型和工具函数。
3. 再迁移状态、组件 Props 和表单。
4. 收敛 `any`，将外部输入改为 `unknown`。
5. 开启 `strictNullChecks` 和 `noImplicitAny`。
6. 最后开启完整 strict 检查并删除迁移兼容配置。

## 4. TypeScript 版本升级验收

- `tsc --noEmit` 通过。
- Lint、单元测试、E2E 和生产构建通过。
- 类型声明生成结果符合预期。
- ESM/CJS、Node.js 和浏览器入口均可加载。
- API 响应、表单、路由和错误模型没有退化。
- 记录新增的类型断言和暂时保留的 `any`。
