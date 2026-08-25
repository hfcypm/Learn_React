# 阶段零：JavaScript 到 TypeScript 的基础准备

**目标**：理解 TypeScript 解决的问题、编译与运行边界，并具备阅读和改写 TypeScript 代码所需的 JavaScript 基础。

## 1. TypeScript 到底解决什么问题

TypeScript 是 JavaScript 的静态类型检查工具和语言扩展。它在开发和构建阶段分析代码，帮助发现参数错误、属性不存在、返回值不匹配和模块导入错误。浏览器和 Node.js 最终执行 JavaScript，因此类型信息通常会在编译过程中被移除。

TypeScript 提供的是开发阶段保证，运行时数据仍然需要校验：

```ts
type User = { name: string };

function greet(user: User) {
  return `你好，${user.name}`;
}

const response: unknown = JSON.parse('{"name":"Tom"}');
```

`response` 声明为 `unknown` 后，代码必须先确认它符合 `User`，才能安全传入 `greet`。

## 2. 必备 JavaScript 基础

- 变量、作用域、闭包和执行上下文
- 基本类型、对象、数组、Map、Set
- 函数、箭头函数、参数和返回值
- 解构、展开、可选链和空值合并
- Promise、`async/await` 和错误处理
- ES Module 的 `import` 与 `export`
- `this`、class、原型和继承的基本行为
- DOM、事件和浏览器 API 的类型来源

## 3. ES6/ES2015 核心语法

ES6 是 ECMAScript 2015 的常用简称。它为 JavaScript 带来了块级变量、箭头函数、解构赋值、模板字符串、默认参数、剩余参数、展开语法、class、Promise、Map、Set 和 ES Module 等能力。

### 3.1 常用语法对照

| 能力 | 作用 | 使用建议 |
|---|---|---|
| `let` / `const` | 块级作用域变量 | 默认使用 `const`，需要重新赋值时使用 `let` |
| 箭头函数 | 简洁函数表达式，并保留外层 `this` | 回调和无独立 `this` 的函数优先使用 |
| 解构与展开 | 提取、复制和组合数组或对象 | 配合不可变更新使用 |
| 模板字符串 | 插入表达式并处理多行文本 | 用于动态文本和 URL |
| 默认参数与剩余参数 | 声明默认值、接收可变数量参数 | 减少参数判断和 `arguments` 使用 |
| `class` | 使用原型方法组织对象行为 | 适合需要实例和继承的模型 |
| `Promise` | 表示异步操作结果 | 配合 `then/catch` 使用；`async/await` 属于 ES2017 |
| `Map` / `Set` | 管理键值集合和唯一值集合 | 需要非字符串键或去重时使用 |
| `import` / `export` | 拆分模块和声明依赖 | 每个模块保持清晰的输入和输出 |

可选链 `?.` 和空值合并 `??` 属于后续 ECMAScript 版本，TypeScript 项目可以使用它们，但应根据目标运行环境配置转译和兼容策略。

### 3.2 综合案例：统计用户角色

下面的案例同时使用解构、默认参数、箭头函数、模板字符串、`Map`、`Set`、展开语法和 ES Module：

```ts
// user-stats.ts
export interface User {
  id: number;
  name: string;
  role?: string;
}

export function summarizeUsers(users: User[], defaultRole = 'viewer') {
  const roleCounts = new Map<string, number>();
  const roleNames = new Set<string>();

  users.forEach(({ role = defaultRole }) => {
    roleCounts.set(role, (roleCounts.get(role) || 0) + 1);
    roleNames.add(role);
  });

  return {
    roles: [...roleNames],
    summary: [...roleCounts].map(([role, count]) => `${role}: ${count}`),
  };
}

const users: User[] = [
  { id: 1, name: 'Lin', role: 'admin' },
  { id: 2, name: 'Ming', role: 'editor' },
  { id: 3, name: 'Jia' },
];

console.log(summarizeUsers(users));
```

案例中的 `role = defaultRole` 是解构默认值，`[...roleNames]` 将 `Set` 展开为数组，`[role, count]` 是对 `Map` 条目的解构。TypeScript 负责约束 `User` 和函数返回值，ES6 语法负责表达数据处理逻辑。

### 3.3 `target` 与 ES6 的关系

`target` 决定 TypeScript 输出的 JavaScript 语法版本，`lib` 决定类型检查时可使用的标准库 API。两者职责不同：

```json
{
  "compilerOptions": {
    "target": "ES2015",
    "lib": ["ES2015", "DOM"]
  }
}
```

- `target: "ES2015"` 会保留大部分 ES6 语法，让运行环境负责执行。
- 较低的 `target` 可能把箭头函数、class 等语法转换为旧语法。
- `lib: ["ES2015"]` 提供 `Map`、`Set`、`Promise` 等 API 的类型定义。
- `target` 不会自动提供旧浏览器缺少的运行时 API；必要时仍需 polyfill 或选择兼容实现。

## 4. 编译、类型检查与运行

TypeScript 项目通常包含三个相互关联的过程：

1. `tsc` 解析 `.ts` 和 `.tsx` 文件并执行类型检查。
2. 构建工具将 TypeScript 转换或交给转译器处理。
3. 浏览器或 Node.js 执行生成的 JavaScript。

类型检查与代码转译可以由不同工具完成。Vite、esbuild、SWC 等工具负责快速转译时，仍应使用 `tsc --noEmit` 单独执行完整类型检查。

## 5. 学习环境

建议项目级安装 TypeScript，并使用锁定文件保证团队环境一致：

```bash
npm install -D typescript
npx tsc --init
npx tsc --noEmit
```

基础 `tsconfig.json`：

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "ESNext",
    "moduleResolution": "Bundler",
    "strict": true,
    "noEmit": true,
    "skipLibCheck": true,
    "noUncheckedIndexedAccess": true,
    "exactOptionalPropertyTypes": true
  },
  "include": ["src", "tests"]
}
```

## 6. 第一组练习

将一组 JavaScript 工具函数迁移为 TypeScript：

- `formatCurrency(value, currency)`
- `groupBy(items, key)`
- `parseQuery(search)`
- `requestJson(url, options)`
- `createValidator(schema)`

每个函数需要定义参数和返回值，处理 `null`、`undefined`、空数组和异常输入，并运行 `tsc --noEmit` 验证。

## 阶段零验收

- 能解释类型检查、转译和运行时的区别。
- 能使用 ES6/ES2015 语法完成集合统计、模块拆分和不可变数据处理。
- 能区分 `any`、`unknown`、`never` 和 `void` 的用途。
- 能为函数、对象、数组、Promise 和错误结果建立类型。
- 能使用严格模式修复至少 10 个类型错误。
