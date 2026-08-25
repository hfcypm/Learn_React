# React 工程质量与生产交付

**目标**：把 React 知识转化为可维护、可测试、可上线和可观测的团队工程。

## 1. 状态分层

先判断状态的归属，再选择工具：

| 状态 | 示例 | 推荐位置 |
|---|---|---|
| 组件状态 | 弹窗、输入框、当前 Tab | `useState`、`useReducer` |
| 共享 UI 状态 | 主题、语言、布局 | Context 或轻量 store |
| 服务端状态 | 用户、订单、列表 | TanStack Query 或 RTK Query |
| URL 状态 | 搜索、分页、筛选 | Router search params |
| 表单状态 | 值、校验、提交状态 | React Hook Form |

服务端状态需要统一处理缓存、失效、重试、分页、预取、乐观更新和错误恢复。Redux 或 Zustand 主要解决客户端状态组织，服务端数据应使用专门的数据获取方案。

## 2. 类型安全 API 层

推荐分为三层：

```text
组件 -> feature 查询/命令 -> API client -> HTTP
```

- API client：统一 base URL、Headers、超时、取消和错误转换。
- feature 层：定义 query key、缓存策略和业务操作。
- 组件层：只消费类型化数据和明确的 loading/error 状态。

外部数据进入系统时需要运行时校验。TypeScript 只在编译期提供保证，API 响应仍然可能不符合声明类型。

```ts
type Result<T> =
  | { ok: true; data: T }
  | { ok: false; error: ApiError };
```

## 3. 表单与复杂交互

复杂表单应覆盖：

- React Hook Form 的字段注册与数组字段
- Zod 等 schema 校验
- 同步校验和异步校验
- 服务端校验错误回填
- 防止重复提交
- 草稿保存和恢复
- 文件上传进度与失败重试
- 键盘操作和错误 focus

## 4. 错误处理

统一错误模型：

| 类型 | 例子 | 用户行为 |
|---|---|---|
| 网络错误 | 断网、超时 | 重试、离线提示 |
| 鉴权错误 | 401、会话过期 | 刷新会话或重新登录 |
| 权限错误 | 403 | 展示无权限页面 |
| 资源错误 | 404 | 展示资源不存在 |
| 业务错误 | 库存不足 | 展示具体业务提示 |
| 服务错误 | 5xx | 降级、重试、上报 |
| 渲染错误 | 组件抛异常 | Error Boundary 恢复 |

要求每个异步页面都有 loading、success、empty、error 状态，核心操作还要有 pending、success、failure 和 retry 状态。

## 5. 可访问性

- 优先使用语义化 HTML。
- 所有交互元素支持键盘操作。
- 弹窗打开后管理 focus，关闭后恢复原 focus。
- 表单错误通过文本和 `aria-describedby` 关联输入框。
- 图片提供有意义的 `alt`，装饰图片使用空 `alt`。
- 颜色之外提供文字、图标或结构信息。
- 检查焦点可见性、色彩对比度和 reduced motion。
- 使用 axe 或同类工具进行自动化检查。

## 6. 安全

- React 默认转义文本，插入 HTML 时必须审查来源和内容。
- 严格限制 `dangerouslySetInnerHTML`，必要时使用可信 HTML 清洗库。
- Token、Cookie、Refresh Token 的存储方案需要结合 XSS 和 CSRF 风险选择。
- 权限校验必须在服务端完成，前端路由守卫只负责用户体验。
- 外部跳转校验协议和允许域名。
- 上传文件校验类型、大小、名称和服务端内容。
- 生产环境配置 CSP、HTTPS、依赖漏洞扫描和敏感信息脱敏。

## 7. 测试金字塔

### 单元测试

测试纯函数、reducer、格式化函数、权限判断和数据转换。

### 组件测试

从用户行为出发测试输入、点击、提交、异步加载、错误提示和键盘操作。避免依赖内部 state 或实现细节。

### 集成测试

使用 MSW 模拟 API，验证页面、请求层、缓存和错误处理之间的协作。

### E2E 测试

使用 Playwright 覆盖登录、权限、核心 CRUD、异常恢复和关键支付或提交流程。

推荐检查命令：

```bash
npm run lint
npm run typecheck
npm run test
npm run test:e2e
npm run build
```

## 8. 目录与依赖边界

```text
src/
├── app/          # 应用入口、全局 Provider
├── routes/       # 路由配置和鉴权边界
├── pages/        # 页面组合
├── features/     # 业务功能
├── entities/     # 领域模型和基础数据组件
├── shared/       # 通用 UI、工具、类型
├── services/     # API client 和外部服务
└── tests/        # 测试工具和测试数据
```

推荐依赖方向：`pages -> features -> entities/shared`，API 请求经过 `services`，通用层不依赖具体业务模块。

## 9. CI/CD 与环境

环境至少分为 development、test、staging 和 production。每个环境需要明确 API 地址、日志级别、资源配置和数据权限。

CI 流程建议：

1. 安装锁定版本依赖。
2. 执行 lint 和格式检查。
3. 执行 TypeScript 检查。
4. 执行单元测试和集成测试。
5. 构建生产包。
6. 执行 E2E 或冒烟测试。
7. 发布带版本号的制品。
8. 记录变更并支持回滚。

环境变量只保存非敏感配置和由部署平台注入的密钥引用，仓库中使用 `.env.example` 提供变量名和占位符。

## 10. 性能与可观测性

### 性能检查

- 使用 React DevTools Profiler 定位重渲染。
- 使用 Lighthouse 检查 FCP、LCP、CLS 和交互性能。
- 使用代码分割、懒加载、图片压缩和缓存控制降低首屏成本。
- 大列表使用分页、虚拟化或增量渲染。
- 通过 Bundle Analyzer 定位依赖体积。

### 运行监控

- 记录错误类型、版本、路由和 request ID。
- 监控 API 延迟、失败率和前端白屏。
- 记录关键业务事件，过滤个人隐私数据。
- 将发布版本与错误监控平台关联。
- 设计告警阈值、负责人和恢复流程。
