# 阶段零：前端基础与开发底座

**目标**：在学习 React 前建立 JavaScript、浏览器、网络和协作基础，能够解释代码运行过程并独立排查常见问题。

## 1. JavaScript 核心

### 1.1 作用域、闭包与执行上下文

**详细概念：**

作用域决定变量在何处可见。`let` 和 `const` 是块级作用域，`var` 是函数作用域。执行上下文是代码运行时的环境，包含变量绑定、作用域链和 `this`。

**1.1.1 变量声明对比**

```js
// var：函数作用域，会提升，可重复声明
function varExample() {
  console.log(flag); // undefined，声明被提升但未赋值
  var flag = true;
}

// let/const：块级作用域，存在暂时性死区
{
  let a = 1;
  const b = 2;
}
// console.log(a); // ReferenceError：块外不可见

// const 绑定的是引用，不是值
const user = { name: 'Alice' };
user.name = 'Bob';   // 允许：修改对象属性
// user = {};        // TypeError：不能重新赋值
```

**1.1.2 闭包**

闭包是函数与其词法作用域的组合。内层函数可以访问外层函数的变量，即使外层函数已经返回：

```js
function createCounter() {
  let count = 0; // 私有变量
  return {
    increment() {
      count += 1;
      return count;
    },
    reset() {
      count = 0;
      return count;
    },
  };
}

const counter = createCounter();
counter.increment(); // 1
counter.increment(); // 2
counter.reset();     // 0
```

React 中函数组件每次渲染都会形成新的闭包，`useEffect` 的依赖数组正是依赖这一机制。理解闭包捕获的时机是排查 Hooks 过期值问题的前提。

### 1.2 相等性与类型转换

`==` 会做隐式转换，`===` 不转换。优先使用 `===`。

```js
0 === false;            // false
0 == false;             // true
'' == 0;                // true
null == undefined;      // true
null === undefined;     // false
NaN === NaN;            // false，应使用 Number.isNaN 判断

// 对象比较的是引用，不是结构
{} === {};              // false
```

数组和对象用结构比较时不要直接 `===`：

```js
function sameItems(a, b) {
  return (
    Array.isArray(a) &&
    Array.isArray(b) &&
    a.length === b.length &&
    a.every((item, i) => item === b[i])
  );
}
```

### 1.3 Promise 与 async/await

**1.3.1 错误传播**

`async` 函数抛出的错误会变为 rejected Promise，`try/catch` 捕获不到跨 `await` 的边界时用 `.catch` 兜底：

```js
async function loadUser(id) {
  const response = await fetch(`/api/users/${id}`);

  if (!response.ok) {
    throw new Error(`请求失败：${response.status}`);
  }

  return response.json();
}

try {
  const user = await loadUser(1);
  renderUser(user);
} catch (error) {
  renderError(error);
}
```

**1.3.2 并发控制**

多个互不依赖的请求用 `Promise.all` 并发执行，而不是串行等待：

```js
// 错误写法：串行，慢一倍
const roles = await fetchRoles();
const teams = await fetchTeams();

// 正确写法：并发执行
const [roles, teams] = await Promise.all([fetchRoles(), fetchTeams()]);
```

`Promise.all` 一个失败就整体失败。需要"部分成功也要返回"时用 `Promise.allSettled`：

```js
const results = await Promise.allSettled([fetchA(), fetchB()]);
for (const result of results) {
  if (result.status === 'fulfilled') {
    // result.value
  } else {
    // result.reason
  }
}
```

**1.3.3 带超时和取消的异步函数**

```js
async function loadUser(id, signal) {
  const response = await fetch(`/api/users/${id}`, { signal });

  if (!response.ok) {
    throw new Error(`请求失败：${response.status}`);
  }

  return response.json();
}

// 超时封装：与调用方的 AbortSignal 组合
function withTimeout(promise, ms) {
  const controller = new AbortController();
  const timer = setTimeout(() => controller.abort(), ms);
  return Promise.race([promise(controller.signal), timeoutReject(ms)]);
}

function timeoutReject(ms) {
  return new Promise((_, reject) =>
    setTimeout(() => reject(new Error(`请求超时（${ms}ms）`)), ms),
  );
}
```

### 1.4 事件循环、宏任务与微任务

JavaScript 单线程，通过事件循环调度任务。宏任务包含 `setTimeout`、I/O 和事件回调；微任务包含 Promise 回调和 `queueMicrotask`。每个宏任务结束后会清空所有微任务。

```js
console.log('start');

setTimeout(() => console.log('timeout'), 0);

Promise.resolve().then(() => console.log('microtask 1'));
Promise.resolve().then(() => console.log('microtask 2'));

console.log('end');

// 输出顺序：
// start
// end
// microtask 1
// microtask 2
// timeout
```

这个顺序是面试与调试的常见考点，也是理解 React 渲染时机和并发特性的基础。

### 1.5 模块系统

```js
// utils/format.js
export function formatDate(date) {
  return new Intl.DateTimeFormat('zh-CN').format(date);
}

// 默认导出
export default function greeting(name) {
  return `你好，${name}`;
}
```

```js
// main.js
import greeting, { formatDate } from './utils/format.js';
```

命名导出用于多个工具函数，默认导出用于单一主要功能。工具模块保持无副作用，导入多次只执行一次。

### 1.6 原型链与 class

```js
class User {
  constructor(name, role) {
    this.name = name;
    this.role = role;
  }

  can(action) {
    return ['create', 'update'].includes(action);
  }
}

const admin = new User('admin', 'admin');
console.log(admin.can('delete')); // false
```

class 是原型链的语法糖。方法定义在 `prototype` 上，实例通过 `__proto__` 找到。排查"方法不是函数"错误时，先确认方法是否绑定到实例。

## 2. 浏览器与 DOM

### 2.1 从 URL 到页面

一次访问的主要步骤：解析 URL、建立连接、获取资源、解析 HTML/CSS、执行 JavaScript、构建 DOM/CSSOM、布局、绘制与合成。

**核心主题：**

- DOM 树、CSSOM、渲染树和布局计算
- 事件捕获、目标阶段、冒泡和事件委托
- 默认行为与 `preventDefault`
- `localStorage`、`sessionStorage`、Cookie 和 IndexedDB
- URL、History API、前进后退和刷新恢复
- 浏览器缓存、强缓存、协商缓存和资源版本化
- 浏览器安全边界、同源策略和 CORS

### 2.2 事件传播与委托

事件先捕获（从根到目标），再冒泡（从目标到根）。事件委托利用冒泡，用父节点统一处理子节点事件：

```html
<ul id="list">
  <li data-id="1">Alice</li>
  <li data-id="2">Bob</li>
</ul>
```

```js
const list = document.getElementById('list');

list.addEventListener('click', (event) => {
  const item = event.target.closest('li');

  if (!item) return;

  const id = item.dataset.id;
  console.log(`选中用户 ${id}`);
});
```

对比给每个 `<li>` 单独绑定监听器，委托只绑定一次，动态新增的项也自动生效。需要用 `event.preventDefault()` 阻止表单提交、链接跳转等默认行为。

### 2.3 存储选型

| 存储 | 容量 | 生命周期 | 适用场景 |
|---|---|---|---|
| `localStorage` | 约 5MB | 永久 | 主题偏好、非敏感缓存 |
| `sessionStorage` | 约 5MB | 标签页关闭 | 会话内草稿 |
| Cookie | 约 4KB | 可设置过期 | 认证凭证（HttpOnly） |
| IndexedDB | 大 | 永久 | 离线数据、大对象 |

```js
// localStorage 统一封装，避免散落键名
const storage = {
  get(key, fallback) {
    try {
      const raw = localStorage.getItem(key);
      return raw === null ? fallback : JSON.parse(raw);
    } catch {
      return fallback;
    }
  },
  set(key, value) {
    localStorage.setItem(key, JSON.stringify(value));
  },
  remove(key) {
    localStorage.removeItem(key);
  },
};
```

不要把 Token 等敏感信息放入 `localStorage`，应使用 `HttpOnly` Cookie 交给服务器处理。

### 2.4 缓存策略

- 强缓存：`Cache-Control: max-age=31536000` 配合文件名哈希（`app.abc123.js`），内容变化时文件名变化，天然失效。
- 协商缓存：`ETag`/`Last-Modified`，每次请求带回条件头，服务器决定 304。
- 开发调试时可在 Network 面板勾选 Disable cache，避免强缓存干扰。

### 2.5 同源策略与 CORS

浏览器限制跨源读取。CORS 预检（OPTIONS）发生在带自定义 Header 或非简单方法的请求前。前端开发通常用代理解决开发期跨域：

```js
// vite.config.ts 开发代理
export default {
  server: {
    proxy: {
      '/api': {
        target: 'http://localhost:3001',
        changeOrigin: true,
      },
    },
  },
};
```

排查 CORS 问题时按顺序检查：是否同源、响应是否带 `Access-Control-Allow-Origin`、是否包含凭证、预检是否通过。

## 3. HTTP 与 API 基础

### 3.1 必须掌握的 HTTP 知识

| 主题 | 需要理解的内容 |
|---|---|
| 方法 | GET、POST、PUT、PATCH、DELETE 的语义 |
| 状态码 | 2xx、3xx、4xx、5xx 的处理策略 |
| Headers | Content-Type、Authorization、Cache-Control、ETag |
| 数据格式 | JSON、文件上传、分页和错误响应 |
| 跨域 | CORS 预检、凭证请求和开发代理 |
| 可靠性 | 超时、重试、取消、幂等和竞态 |

统一 API 响应模型，前后端共用语义：

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

### 3.2 fetch 封装：超时、取消与错误归一化

```js
// api/client.js
export class ApiError extends Error {
  constructor(code, message, requestId) {
    super(message);
    this.name = 'ApiError';
    this.code = code;
    this.requestId = requestId;
  }
}

export async function request(path, { method = 'GET', body, signal } = {}) {
  const response = await fetch(path, {
    method,
    headers: { 'Content-Type': 'application/json' },
    body: body === undefined ? undefined : JSON.stringify(body),
    signal,
  });

  const data = await response.json().catch(() => null);

  if (!response.ok) {
    throw new ApiError(
      data?.code ?? 'UNKNOWN_ERROR',
      data?.message ?? `请求失败：${response.status}`,
      data?.requestId,
    );
  }

  return data;
}
```

```js
// 使用时组合取消信号与超时
const controller = new AbortController();
const timer = setTimeout(() => controller.abort(), 8000);

try {
  const users = await request('/api/users?page=1', { signal: controller.signal });
  renderList(users);
} catch (error) {
  if (error.name === 'AbortError') {
    // 用户主动取消或超时，静默处理
  } else {
    renderError(error);
  }
} finally {
  clearTimeout(timer);
}
```

### 3.3 竞态防护

连续请求同一接口时，只有最后一次的结果应当生效。用请求序号或 AbortController 丢弃过期响应：

```js
// 序号方案：简单可靠
let latestRequestId = 0;

async function searchUsers(keyword) {
  const current = ++latestRequestId;
  const result = await fetchUsers(keyword);

  if (current !== latestRequestId) {
    return; // 已有更新的请求，丢弃本次结果
  }

  renderUsers(result);
}
```

分页、搜索框输入、Tab 切换都是竞态高频场景。把竞态防护放在可复用的请求层，而不是每个组件各自处理。

## 4. Git 与协作

### 4.1 常用命令流程

```bash
# 创建并切换到新分支
git checkout -b 260827-feat-user-list

# 查看变更
git status
git diff

# 暂存并提交（Conventional Commits）
git add src/api/user.js
git commit -m "feat(user-list): 新增用户列表 API 封装"

# 推送到远程并创建合并请求
git push -u origin 260827-feat-user-list
```

### 4.2 提交信息规范

```text
<type>(<scope>): <subject>

feat(login): 实现登录会话持久化
fix(list): 修复分页页码越界问题
docs(readme): 补充环境变量说明
test(user-list): 增加请求取消测试
refactor(api): 抽取统一错误处理
```

每次提交只包含一个逻辑变更。`feat` 与 `fix` 会影响版本号，其他类型不触发。

### 4.3 变更前与变更后

- 变更前确认影响范围，检查目标分支是否最新。
- 变更后运行格式检查、类型检查、测试和构建。
- 合并前用 `git diff` 复核，避免把调试代码带入主干。

## 5. 阶段项目：原生 JavaScript 用户列表

用原生 JavaScript 完成一个"用户列表"页面，验证本阶段全部知识点。

### 5.1 项目目录

```text
user-list/
├── index.html
├── styles.css
├── src/
│   ├── api/
│   │   ├── client.js      # fetch 封装：错误归一化
│   │   └── users.js       # 用户接口：列表、搜索、分页
│   ├── state.js           # 应用状态：关键字、页码、数据
│   ├── render.js          # DOM 渲染：列表、分页、状态提示
│   └── main.js            # 入口：绑定事件、URL 同步、竞态防护
└── tests/
    └── users.test.js      # 单元测试
```

### 5.2 API 模块

```js
// src/api/users.js
import { request } from './client.js';

export function fetchUsers({ page, pageSize, keyword }, signal) {
  const params = new URLSearchParams({ page, pageSize });

  if (keyword) {
    params.set('keyword', keyword);
  }

  return request(`/api/users?${params.toString()}`, { signal });
}
```

### 5.3 状态与渲染

```js
// src/state.js
export const state = {
  page: 1,
  pageSize: 10,
  keyword: '',
  items: [],
  total: 0,
  loading: false,
  error: null,
};
```

```js
// src/render.js
export function renderList(items) {
  const container = document.querySelector('#user-list');
  container.innerHTML = items
    .map(
      (user) => `
        <li class="user-item">
          <span class="user-name">${escapeHtml(user.name)}</span>
          <span class="user-email">${escapeHtml(user.email)}</span>
        </li>`,
    )
    .join('');
}

function escapeHtml(value) {
  return String(value)
    .replaceAll('&', '&amp;')
    .replaceAll('<', '&lt;')
    .replaceAll('>', '&gt;')
    .replaceAll('"', '&quot;');
}
```

### 5.4 入口：防抖、取消与 URL 状态

```js
// src/main.js
import { fetchUsers } from './api/users.js';
import { state } from './state.js';
import { renderList, renderPagination, renderStatus } from './render.js';

let latestRequestId = 0;
let searchTimer;

const keywordInput = document.querySelector('#keyword');
const prevButton = document.querySelector('#prev');
const nextButton = document.querySelector('#next');

// 防抖：停止输入 300ms 后才请求
keywordInput.addEventListener('input', (event) => {
  clearTimeout(searchTimer);
  searchTimer = setTimeout(() => {
    state.keyword = event.target.value.trim();
    state.page = 1;
    syncUrl();
    load();
  }, 300);
});

prevButton.addEventListener('click', () => {
  if (state.page > 1) {
    state.page -= 1;
    syncUrl();
    load();
  }
});

nextButton.addEventListener('click', () => {
  if (state.page * state.pageSize < state.total) {
    state.page += 1;
    syncUrl();
    load();
  }
});

async function load() {
  const current = ++latestRequestId;
  state.loading = true;
  renderStatus(state);

  try {
    const result = await fetchUsers({
      page: state.page,
      pageSize: state.pageSize,
      keyword: state.keyword,
    });

    if (current !== latestRequestId) return; // 过期响应，丢弃

    state.items = result.items;
    state.total = result.total;
    renderList(state.items);
    renderPagination(state);
  } catch (error) {
    if (current === latestRequestId) renderStatus(error);
  } finally {
    if (current === latestRequestId) {
      state.loading = false;
      renderStatus(state);
    }
  }
}

// 将筛选条件写入 URL，刷新后恢复
function syncUrl() {
  const url = new URL(window.location.href);
  url.searchParams.set('page', String(state.page));

  if (state.keyword) {
    url.searchParams.set('keyword', state.keyword);
  } else {
    url.searchParams.delete('keyword');
  }

  history.replaceState({}, '', url);
}

// 页面加载时从 URL 恢复状态
function initFromUrl() {
  const params = new URLSearchParams(window.location.search);
  state.page = Number(params.get('page')) || 1;
  state.keyword = params.get('keyword') ?? '';
  keywordInput.value = state.keyword;
}

initFromUrl();
load();
```

### 5.5 单元测试

```js
// tests/users.test.js
import { describe, it, expect } from './harness.js';

describe('fetchUsers 参数拼接', () => {
  it('无关键字时不带 keyword 参数', async () => {
    const called = await captureParams((signal) =>
      fetchUsers({ page: 1, pageSize: 10, keyword: '' }, signal),
    );
    expect(called).toContain('page=1');
    expect(called).toContain('pageSize=10');
    expect(called).not.toContain('keyword');
  });

  it('有关键字时包含 keyword 参数', async () => {
    const called = await captureParams((signal) =>
      fetchUsers({ page: 2, pageSize: 10, keyword: 'alice' }, signal),
    );
    expect(called).toContain('keyword=alice');
  });
});
```

测试需要覆盖：API 模块参数拼接、防抖后只触发一次请求、过期响应被丢弃、空数据与错误状态渲染。

## 阶段零验收

- 能从 Network 面板解释一次请求的完整生命周期。
- 能写出带超时、取消和错误处理的异步函数。
- 能解释事件循环中宏任务与微任务的执行顺序。
- 能说明 `let`、`const` 和闭包的作用域差异。
- 能区分 UI 状态、服务端数据和 URL 状态。
- 能定位一次 CORS、404、401、500 和超时问题。
- 能创建一个可复现、可测试、可提交的最小前端项目。
- 项目包含：原生 JavaScript 用户列表、请求取消与重复请求保护、URL 参数保存筛选条件、至少 5 个单元测试。

## 进入下一阶段的条件

你能够独立解释一次请求从输入 URL 到渲染完成的完整链路，并能在不依赖框架的情况下完成带状态、竞态防护和错误处理的前端页面。此时进入 [阶段一：React 基础](./stage-1-basic.md)。
