# 可运行示例项目与 Mock API 规范

**目标**：把学习路线中的代码片段组织成可启动、可测试、可复现的示例项目。

## 1. 项目分层

建议建立一个独立示例目录，按学习阶段拆分：

```text
examples/
├── 01-todolist/
├── 02-form-and-list/
├── 03-router-and-auth/
├── 04-query-and-cache/
├── 05-typesafe-dashboard/
└── 06-next-rendering/
```

每个示例至少包含：

- `README.md`：目标、前置知识、启动和验证命令
- `package.json`：明确脚本和依赖
- 锁定文件：保证安装结果可复现
- `src/`：最小完整源码
- `tests/`：关键行为测试
- `.env.example`：变量名和占位符
- `CHANGELOG.md`：示例随技术版本变化的记录

## 2. 统一脚本

所有示例使用一致的脚本名称：

```json
{
  "scripts": {
    "dev": "启动开发服务",
    "build": "构建生产产物",
    "lint": "执行代码检查",
    "typecheck": "执行类型检查",
    "test": "执行单元与组件测试",
    "test:e2e": "执行端到端测试"
  }
}
```

示例项目应在 README 中明确命令的实际实现，并在 CI 中按 `lint -> typecheck -> test -> build` 顺序执行。

## 3. Mock API 统一模型

### 用户列表

```http
GET /api/users?page=1&pageSize=20&keyword=tom&role=member
```

成功响应：

```json
{
  "items": [],
  "page": 1,
  "pageSize": 20,
  "total": 0
}
```

### 用户详情

```http
GET /api/users/:id
```

### 创建用户

```http
POST /api/users
Content-Type: application/json
```

```json
{
  "name": "Tom",
  "email": "tom@example.com",
  "role": "member"
}
```

### 更新和删除

```http
PATCH /api/users/:id
DELETE /api/users/:id
```

### 统一错误

```json
{
  "code": "USER_EMAIL_EXISTS",
  "message": "邮箱已经存在",
  "requestId": "request-id-placeholder",
  "details": {}
}
```

## 4. Mock 场景

每个示例至少支持以下可控场景：

- 成功返回
- 空列表
- 401 未登录
- 403 无权限
- 404 资源不存在
- 422 字段校验失败
- 500 服务错误
- 延迟响应
- 网络失败
- 重复提交
- 删除失败后的回滚

Mock 场景通过请求参数、测试 handler 或本地开关控制，确保测试可以稳定复现。

## 5. 示例推进顺序

### 示例一：TodoList

覆盖组件、Props、state、事件、列表、条件渲染和基础测试。

### 示例二：表单与列表

覆盖受控组件、表单校验、分页、loading、empty 和 error 状态。

### 示例三：路由与认证

覆盖登录、路由守卫、嵌套路由、权限按钮和会话失效。

### 示例四：服务端状态

覆盖 query key、缓存、失效、预取、乐观更新和错误重试。

### 示例五：类型安全中后台

覆盖严格 TypeScript、schema 校验、权限矩阵、组件测试和 E2E。

### 示例六：现代渲染

覆盖 CSR、SSR、SSG、RSC、Suspense、错误边界和缓存策略。

## 6. 示例完成标准

- 新用户按照 README 可以启动项目。
- 首次安装使用锁定依赖并可重复完成。
- 关键 API 有成功、失败和延迟场景。
- 关键用户流程具备组件测试和 E2E 测试。
- 生产构建成功且可预览。
- 示例说明适用版本和已知限制。
