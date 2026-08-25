# TypeScript 工程质量与类型设计

**目标**：让类型系统服务于业务建模、接口约束、重构安全和运行时可靠性。

## 1. 类型设计原则

- 类型表达业务规则，命名体现领域含义。
- 优先使用联合类型表达有限状态。
- 使用 `unknown` 接收外部不可信数据。
- 使用类型守卫或 schema 校验缩小类型。
- 通过泛型复用结构，通过联合类型表达差异。
- 保持类型与运行时行为同步。
- 避免用类型断言掩盖未解决的数据问题。

## 2. `any`、`unknown`、`never`

| 类型 | 含义 | 适用场景 |
|---|---|---|
| `any` | 关闭该值的类型检查 | 迁移遗留代码，必须逐步收敛 |
| `unknown` | 不确定但需要验证的值 | API 响应、JSON、异常值 |
| `never` | 不可能出现的值 | 穷尽检查、永不返回函数 |
| `void` | 函数没有有用返回值 | 事件处理器、无返回副作用函数 |

穷尽检查示例：

```ts
type Status = 'idle' | 'loading' | 'success' | 'error';

function assertNever(value: never): never {
  throw new Error(`未处理的状态：${String(value)}`);
}

function getLabel(status: Status) {
  switch (status) {
    case 'idle': return '未开始';
    case 'loading': return '加载中';
    case 'success': return '成功';
    case 'error': return '失败';
    default: return assertNever(status);
  }
}
```

## 3. 异步状态建模

```ts
type AsyncState<T> =
  | { status: 'idle' }
  | { status: 'loading' }
  | { status: 'success'; data: T }
  | { status: 'error'; error: Error };
```

这种模型保证组件显式处理加载、成功和失败状态，避免 `data`、`loading`、`error` 之间出现互相矛盾的组合。

## 4. API 类型安全

API 层建议分为：

```text
组件 -> 业务查询/命令 -> API client -> HTTP
```

- API client 统一超时、取消、Headers 和错误转换。
- DTO 类型描述网络传输格式。
- Domain 类型描述业务内部模型。
- Mapper 负责 DTO 与 Domain 之间的转换。
- 外部响应使用运行时 schema 校验。

```ts
type ApiError = {
  code: string;
  message: string;
  requestId?: string;
  details?: unknown;
};

type PageResult<T> = {
  items: T[];
  page: number;
  pageSize: number;
  total: number;
};
```

TypeScript 类型无法验证服务器真实返回值。API 响应、用户输入、LocalStorage 和 URL 参数都属于运行时边界，需要 schema 校验或类型守卫。

## 5. React 类型实践

- Props 使用明确的对象类型。
- 事件处理器使用 React 提供的事件类型。
- `children` 使用 React 节点类型或更精确的组件结构。
- `useState` 在初始值无法推导时显式指定泛型。
- `useRef` 根据 DOM 元素或可空状态标注类型。
- 表单字段使用 schema 推导，避免重复声明。
- 组件泛型只在调用方确实需要多种数据类型时使用。

## 6. 编译配置分层

大型项目建议拆分配置：

```text
tsconfig.json       # 基础公共配置
tsconfig.app.json   # 浏览器应用
tsconfig.node.json  # 构建配置和脚本
tsconfig.test.json  # 测试环境
```

重点配置：

- `strict`
- `noUncheckedIndexedAccess`
- `exactOptionalPropertyTypes`
- `noUnusedLocals`
- `noUnusedParameters`
- `noFallthroughCasesInSwitch`
- `forceConsistentCasingInFileNames`
- `moduleResolution`
- `paths` 与构建工具 alias 的一致性

## 7. 声明文件与第三方库

- 优先安装官方或社区维护的 `@types/*`。
- 没有类型声明时，建立最小的模块声明并记录风险。
- 避免把整个模块声明为 `any`，优先为实际使用的 API 建立类型。
- 发布库时区分运行时代码、类型声明和 source map。
- 检查 `exports`、`types`、`typesVersions` 和 ESM/CJS 兼容性。

## 8. 测试与质量门禁

推荐 CI 顺序：

```bash
npm run lint
npm run typecheck
npm run test
npm run build
```

类型检查不能替代测试。测试验证运行时行为，类型检查验证开发时契约，两者需要共同覆盖边界。
