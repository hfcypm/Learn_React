# React 技术版本治理与升级指南

**目标**：让学习内容、示例项目和生产项目拥有清晰的版本边界，降低 API 混用和升级风险。

## 1. 推荐版本分层

版本文档应同时记录“学习主线”和“旧项目维护线”。具体版本以项目锁定文件和官方发布说明为准。

| 技术 | 学习主线 | 维护重点 |
|---|---|---|
| React | React 18+ 并发能力、现代 React API | 类组件、旧生命周期和旧根节点 API |
| React DOM | `createRoot`、`hydrateRoot` | `ReactDOM.render` 旧项目迁移 |
| React Router | 当前主版本的路由、数据加载和错误边界 | 旧版 `<Switch>`、旧 history API |
| Next.js | App Router、RSC、服务端数据和缓存 | Pages Router、旧数据获取 API |
| TypeScript | 严格模式、`unknown`、泛型和运行时校验 | 宽松配置、隐式 `any` |
| 数据请求 | TanStack Query 或 RTK Query | 手写全局 loading 和重复缓存逻辑 |
| 测试 | Vitest、Testing Library、Playwright、MSW | Jest 旧配置和实现细节测试 |
| 构建 | Vite 或框架内置构建系统 | CRA 旧项目维护和迁移 |

## 2. 每个示例必须记录的元数据

每个教程或项目示例顶部增加以下信息：

```markdown
> 适用范围：React 18+
> 示例类型：客户端渲染
> 需要的工具：Node.js、包管理器、TypeScript
> 验证命令：lint、typecheck、test、build
> 最后复核日期：YYYY-MM-DD
```

版本信息需要覆盖：

- Node.js 运行时范围
- 包管理器和锁定文件
- React 与 React DOM
- 构建工具或应用框架
- 测试工具
- 浏览器支持范围

## 3. 升级前检查

1. 阅读官方 changelog、迁移指南和 breaking changes。
2. 记录当前依赖树、构建产物大小和测试基线。
3. 检查第三方组件库、路由库、状态库和测试工具兼容性。
4. 为关键用户流程补充回归测试。
5. 将升级拆成可独立回滚的小变更。
6. 在预发布环境运行真实 API、权限和构建验证。

## 4. 升级后检查

- 开发环境可以启动，热更新正常。
- 生产构建成功，静态资源路径正确。
- 路由、刷新、深链接和 404 行为正确。
- 表单、弹窗、focus 和键盘操作正常。
- 登录、权限、Token 刷新和退出流程正常。
- 错误边界、Suspense 和异步状态符合预期。
- LCP、INP、CLS 和 JavaScript 包体积没有超出预算。
- 错误监控能识别新版本并关联发布记录。

## 5. 常见迁移主题

### CRA 到 Vite

- 将脚本和入口迁移到 Vite 约定。
- 检查环境变量前缀和读取方式。
- 检查静态资源路径、代理和 `base` 配置。
- 迁移 Jest、Webpack alias 和 CSS 处理方式。
- 使用生产构建验证路由 fallback。

### JavaScript 到 TypeScript

- 先为 API 响应、Props 和表单模型建立类型。
- 使用 `unknown` 接收不可信输入。
- 通过类型守卫或 schema 校验缩小类型。
- 分阶段打开 `strict`、`noUncheckedIndexedAccess` 等检查。
- 禁止用大量类型断言掩盖真实数据问题。

### Pages Router 到 App Router

- 重新设计布局、loading、error 和 not-found 边界。
- 标记 Client Component 边界。
- 重新检查数据缓存、重新验证和请求位置。
- 将权限校验放入服务端数据访问流程。
- 为浏览器 API 和事件处理保留客户端组件。
