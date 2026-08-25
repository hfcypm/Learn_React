# TypeScript 综合实战：类型安全的团队管理 API

**目标**：用一个完整项目串联基础类型、泛型、联合类型、运行时校验、React 类型实践和工程质量。

## 1. 项目范围

构建一个团队成员管理模块，包含成员列表、成员详情、创建编辑、角色权限、分页筛选和审计记录。

## 2. 核心类型

```ts
type Role = 'admin' | 'manager' | 'member';
type UserStatus = 'active' | 'disabled';

type User = {
  id: string;
  name: string;
  email: string;
  role: Role;
  status: UserStatus;
  createdAt: string;
};

type CreateUserInput = Omit<User, 'id' | 'createdAt'>;
type UpdateUserInput = Partial<Pick<User, 'name' | 'email' | 'role' | 'status'>>;
```

## 3. 功能要求

- 使用 URL 参数表达关键词、角色、状态和分页。
- API 响应使用 `PageResult<User>` 泛型。
- 创建和编辑使用不同的输入类型。
- 角色使用联合类型，禁止任意字符串。
- API 错误使用统一 `ApiError` 类型。
- 外部 JSON 进入业务层前完成运行时校验。
- 服务端权限检查和前端权限展示分别建模。

## 4. 推荐目录

```text
src/
├── domain/
│   ├── user.ts
│   └── permissions.ts
├── api/
│   ├── client.ts
│   ├── user-api.ts
│   └── schemas.ts
├── features/
│   └── user-management/
├── shared/
│   ├── result.ts
│   ├── pagination.ts
│   └── errors.ts
└── tests/
```

## 5. 实战任务

### 任务一：基础建模

- 定义成员、角色、状态和输入模型。
- 为列表、详情、创建、更新定义独立类型。
- 使用 `Pick`、`Omit`、`Partial` 生成合理的派生类型。

### 任务二：结果类型

```ts
type Result<T, E = ApiError> =
  | { ok: true; data: T }
  | { ok: false; error: E };
```

使用 `Result` 处理请求成功和失败，调用方必须通过 `ok` 判断后才能访问对应字段。

### 任务三：运行时校验

- 校验服务端成员列表。
- 校验表单输入。
- 处理缺失字段、错误类型和未知字段。
- 测试校验失败时的错误信息。

### 任务四：React 集成

- Props 使用成员和输入类型。
- 列表组件使用泛型表格或明确的 `User` 类型。
- 表单从 schema 推导输入类型。
- 事件、ref、异步状态和错误边界均通过类型检查。

## 6. 测试要求

- 类型测试：验证公共类型和泛型 API 的预期使用方式。
- 单元测试：验证 mapper、权限判断、分页转换和错误转换。
- API 测试：验证成功、401、403、404、422、500。
- 组件测试：验证表单、列表、空状态和错误状态。
- E2E 测试：验证登录、创建、编辑和权限流程。
- 构建检查：执行 `tsc --noEmit` 和生产构建。

## 7. 完成标准

- 业务核心代码无隐式 `any`。
- 外部输入经过运行时校验。
- 状态模型能够表达 idle、loading、success 和 error。
- 修改领域类型后，相关调用方能被编译器准确提示。
- API、表单、权限和组件测试全部通过。
- README 记录安装、检查、测试和构建命令。
