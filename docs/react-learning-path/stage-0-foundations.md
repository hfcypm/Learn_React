# 阶段零：前端基础与开发底座

**目标**：在学习 React 前建立 JavaScript、浏览器、网络和协作基础，能够解释代码运行过程并独立排查常见问题。

## 1. JavaScript 核心

### 必须掌握

- 变量、作用域、闭包和执行上下文
- 基本类型、引用类型、相等性和类型转换
- 数组、对象、Map、Set 的常用操作
- 函数、箭头函数、参数解构和展开语法
- 模块系统：`import`、`export`、默认导出和命名导出
- `this`、原型链、class 和继承的基本行为
- Promise、`async/await`、错误传播和并发控制
- 事件循环、宏任务、微任务和渲染时机

### React 关联

React 组件本质上是 JavaScript 函数。Props、state、事件处理器和 Hooks 都依赖闭包与引用关系。学习 `useEffect` 时，需要能够解释依赖数组、闭包捕获和异步竞态。

```js
async function loadUser(id, signal) {
  const response = await fetch(`/api/users/${id}`, { signal });

  if (!response.ok) {
    throw new Error(`请求失败：${response.status}`);
  }

  return response.json();
}
```

### 验收标准

- 能说明 `let`、`const` 和闭包的作用域差异。
- 能写出带超时、取消和错误处理的异步函数。
- 能解释为什么循环中的异步回调会产生闭包问题。
- 能使用浏览器断点、Network 和 Console 面板定位简单问题。

## 2. 浏览器与 DOM

需要理解浏览器从 URL 到页面的主要过程：解析 URL、建立连接、获取资源、解析 HTML/CSS、执行 JavaScript、布局、绘制和合成。

### 核心主题

- DOM 树、CSSOM、渲染树和布局计算
- 事件捕获、目标阶段、冒泡和事件委托
- 默认行为与 `preventDefault`
- `localStorage`、`sessionStorage`、Cookie 和 IndexedDB
- URL、History API、前进后退和刷新恢复
- 浏览器缓存、强缓存、协商缓存和资源版本化
- 浏览器安全边界、同源策略和 CORS

### 与 React 的连接

React 负责声明 UI 状态与视图关系，开发者仍然需要理解 DOM 才能正确处理 focus、滚动、测量、键盘交互和第三方 DOM 库集成。`useRef` 适用于与 DOM 或外部系统同步，常规 UI 状态应使用 state。

## 3. HTTP 与 API 基础

### 必须掌握

| 主题 | 需要理解的内容 |
|---|---|
| 方法 | GET、POST、PUT、PATCH、DELETE 的语义 |
| 状态码 | 2xx、3xx、4xx、5xx 的处理策略 |
| Headers | Content-Type、Authorization、Cache-Control、ETag |
| 数据格式 | JSON、文件上传、分页和错误响应 |
| 跨域 | CORS 预检、凭证请求和开发代理 |
| 可靠性 | 超时、重试、取消、幂等和竞态 |

建议统一 API 响应模型：

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

## 4. Git 与协作

### 学习清单

- 创建分支、提交、合并和解决冲突
- 使用 `git diff`、`git log`、`git blame` 追踪变更
- Pull Request 描述变更背景、方案、测试和风险
- 使用 Conventional Commits 统一提交信息
- 变更前确认影响范围，变更后运行格式检查、类型检查、测试和构建

### 阶段项目

使用原生 JavaScript 完成一个“用户列表”页面：

- 从本地 Mock API 获取列表
- 支持搜索、分页、加载、空数据和错误状态
- 使用 URL 参数保存筛选条件
- 实现请求取消和重复请求保护
- 编写 API 模块和至少 5 个单元测试

## 阶段零验收

- 能从 Network 面板解释一次请求的完整生命周期。
- 能区分 UI 状态、服务端数据和 URL 状态。
- 能定位一次 CORS、404、401、500 和超时问题。
- 能创建一个可复现、可测试、可提交的最小前端项目。
