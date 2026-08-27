# Next.js 版本边界与迁移

**目标**：让学习内容、示例项目和生产项目拥有清晰的版本边界，降低 API 混用和升级风险。

## 1. 推荐版本分层

具体版本以项目锁定文件与官方发布说明为准。

| 技术 | 学习主线 | 维护重点 |
|---|---|---|
| Next.js | App Router、Server Component、Server Actions | Pages Router 旧项目维护 |
| React | React 19、`useActionState`、Suspense | 类组件和旧生命周期 |
| 数据层 | Server Component 获取、ISR、Route Handlers | 旧 `getServerSideProps` |
| 鉴权 | Cookie 会话、服务端权限校验 | 旧自定义 Token 方案 |
| 测试 | Vitest、Testing Library、Playwright | Jest 旧配置 |

## 2. 每个示例必须记录的元数据

```markdown
> 适用范围：Next.js 15+（App Router）
> 示例类型：服务端渲染 / 客户端渲染
> 需要的工具：Node.js、npm、TypeScript
> 验证命令：lint、typecheck、build
> 最后复核日期：YYYY-MM-DD
```

## 3. Pages Router 到 App Router 迁移

- 重新设计布局、loading、error 和 not-found 边界。
- 标记 Client Component 边界。
- 重新检查数据缓存、重新验证和请求位置。
- 将权限校验放入服务端数据访问流程。
- 用 Route Handlers 替代旧 API Routes。
- 用 Server Actions 替代旧表单提交逻辑。
- 用 `generateMetadata` 替代旧 `Head` 组件。

## 4. 升级前检查

1. 阅读官方 changelog、迁移指南和 breaking changes。
2. 记录当前依赖树、构建产物大小和测试基线。
3. 检查第三方组件库、ORM、测试工具兼容性。
4. 为关键用户流程补充回归测试。
5. 将升级拆成可独立回滚的小变更。
6. 在预发布环境运行真实 API、权限和构建验证。

## 5. 升级后检查

- 开发环境可以启动，热更新正常。
- 生产构建成功，静态资源路径正确。
- 路由、刷新、深链接和 404 行为正确。
- 登录、权限、Token 刷新和退出流程正常。
- 缓存失效、ISR 和动态渲染符合预期。
- LCP、INP、CLS 和 JavaScript 包体积没有超出预算。
- 错误监控能识别新版本并关联发布记录。
