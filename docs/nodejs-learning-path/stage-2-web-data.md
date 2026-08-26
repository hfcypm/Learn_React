# 阶段二：Web 服务与数据层

**目标**：把 Node.js 核心能力组织成可维护的 API 服务。

## 1. 服务分层

推荐使用以下边界：

```text
HTTP Controller -> Application Service -> Repository -> Database
```

- Controller 负责协议解析、状态码和响应格式。
- Application Service 负责业务用例和事务边界。
- Repository 负责数据库读写，不向 Controller 暴露 ORM 细节。
- Schema/Validator 负责运行时校验外部数据。

## 2. 请求校验与错误模型

TypeScript 类型无法替代运行时校验。请求体、查询参数、环境变量和数据库返回值都属于不可信边界。

统一错误结构可以包含 `code`、`message`、`requestId` 和可选的字段错误。内部堆栈写入日志，面向用户的响应只暴露必要信息。

```js
function parseCreateUser(input) {
  if (!input || typeof input !== 'object') {
    throw new Error('请求体格式错误');
  }

  const { name, email } = input;
  if (typeof name !== 'string' || typeof email !== 'string') {
    throw new Error('name 和 email 必须是字符串');
  }

  return { name: name.trim(), email: email.trim().toLowerCase() };
}
```

生产项目可使用 Zod、Valibot 或框架内置 Schema 工具集中维护规则。

## 3. 认证与授权

认证回答“用户是谁”，授权回答“用户能做什么”。权限判断必须在服务端完成。密码使用适合密码的哈希算法，Token、Cookie、CSRF、刷新和注销策略需要整体设计。

建议：

- 会话或 Token 设置过期时间和撤销策略。
- 只记录用户 ID、角色和权限，不把敏感凭证写入日志。
- 每个资源操作检查资源归属和租户边界。
- 登录、验证码、密码重置和高风险操作增加速率限制。

## 4. 数据库与事务

数据库访问需要连接池、超时、参数化查询和迁移管理。事务覆盖一个完整业务用例，并根据隔离级别处理并发更新。

分页优先使用稳定排序和游标分页；列表接口限制最大 page size。缓存需要明确 key、TTL、失效和击穿保护。

## 5. Web 服务案例结构

```text
src/
├── app/
│   ├── routes.js
│   └── error-handler.js
├── modules/
│   └── users/
│       ├── user.controller.js
│       ├── user.service.js
│       ├── user.repository.js
│       └── user.schema.js
├── infrastructure/
│   ├── database.js
│   └── logger.js
└── server.js
```

## 阶段验收

- 能实现 CRUD、分页、校验、统一错误和认证。
- 能解释事务、连接池、缓存失效和幂等键。
- 能把业务规则从 HTTP 层和数据库层隔离出来。
- 能为成功、校验失败、未认证、无权限和服务错误设计响应。

## 动手任务

1. 将用户功能拆成 Controller、Service、Repository 和 Schema 四个模块。
2. 为创建用户增加字段校验、重复邮箱冲突和统一错误响应。
3. 为列表接口增加稳定排序、最大 page size 和分页元数据。
4. 为一个完整业务用例划定事务边界，并写出成功和回滚场景。

## 进入下一阶段的条件

你能够画出一次创建用户请求的完整数据流，能够在不修改 Controller 的情况下替换 Repository，并能为每类错误写出状态码和错误码。
