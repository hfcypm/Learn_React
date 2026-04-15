# 阶段一：React 基础阶段（入门必学）

**目标**：能独立开发简单页面、理解 React 核心思想

## 必须掌握的技能

### 1. 环境与基础工具

- [ ] 搭建 React 开发环境（Vite / Create React App）
- [ ] 熟悉 npm/yarn/pnpm 包管理器
- [ ] 理解 JSX 语法（HTML 与 JS 混合写法、规则、注意事项）

**详细概念：**

**1.1 搭建 React 开发环境**

搭建 React 开发环境是学习 React 的第一步。推荐使用 Vite 作为构建工具，它是一个基于 ES Module 的下一代前端构建工具，相比传统的 Create React App（CRA）有更快的启动速度和热更新体验。

Vite 的核心优势：
- **极速启动**：Vite 在开发环境下不需要打包，直接通过原生 ES Module 提供代码，实现秒级启动
- **热模块替换（HMR）**：修改代码后能快速看到变化，速度比传统 webpack 快 10-100 倍
- **按需编译**：只编译当前页面需要的模块，而不是整个项目

使用 Vite 创建项目的命令：
```bash
npm create vite@latest my-app -- --template react
cd my-app
npm install
npm run dev
```

**1.2 npm/yarn/pnpm 包管理器**

三大主流包管理器各有特点：

| 特性 | npm | yarn | pnpm |
|------|-----|------|------|
| 安装速度 | 较慢 | 快 | 最快 |
| 磁盘空间 | 占用较大 | 占用较大 | 共享依赖，节省空间 |
| 锁定文件 | package-lock.json | yarn.lock | pnpm-lock.yaml |
| 幽灵依赖 | 存在 | 存在 | 不存在，更安全 |

推荐在实际项目中使用 pnpm，它采用内容寻址存储，相同版本的包只会在磁盘上存储一份，大大节省空间且避免依赖混淆。

**1.3 JSX 语法详解**

JSX（JavaScript XML）是 React 的核心语法，它允许我们在 JavaScript 中编写类似 HTML 的代码。JSX 并不是一种新的语言，而是 JavaScript 的语法扩展。

JSX 的核心规则：
- **根元素限制**：JSX 表达式只能有一个根元素，可以使用 `<div>` 包裹，或使用 Fragment（`<>...</>`）避免额外 DOM 层级
- **className 替代 class**：由于 class 是 JavaScript 保留字，JSX 中使用 `className` 来设置 CSS 类名
- **单标签必须闭合**：如 `<img />`、`<input />` 必须自闭合
- **表达式嵌入**：使用 `{}` 在 JSX 中嵌入任何 JavaScript 表达式，如变量、函数调用、三元运算等
- **条件渲染**：使用三元表达式 `condition ? true : false` 或逻辑与 `condition && element`
- **列表渲染**：使用 `map()` 方法渲染列表，必须为每个元素提供唯一的 `key` 属性

JSX 编译原理：
JSX 会被 Babel 等编译器转换为 `React.createElement()` 调用，最终生成普通的 JavaScript 对象来描述 DOM 结构。

**经典案例：Vite + React 项目初始化**

```bash
# 使用 Vite 创建 React 项目
npm create vite@latest my-react-app -- --template react

# 进入项目目录
cd my-react-app

# 安装依赖
npm install

# 启动开发服务器
npm run dev
```

```jsx
// src/App.jsx - JSX 语法示例
function App() {
  const name = 'React';

  return (
    <div className="app">
      <h1>Hello, {name}!</h1>
      <p>JSX 规则：</p>
      <ul>
        <li>使用 className 而不是 class</li>
        <li>使用 {} 嵌入表达式</li>
        <li>必须有一个根元素</li>
      </ul>
    </div>
  );
}

export default App;
```

---

### 2. 组件基础

- [ ] 函数组件（优先掌握）
- [ ] 类组件（了解）
- [ ] 组件拆分、复用、嵌套、组合
- [ ] Props 传值
- [ ] Props 校验（PropTypes）

**详细概念：**

**2.1 函数组件**

函数组件是现代 React 开发的主流方式，它以 JavaScript 函数的形式定义组件。从 React 16.8 引入 Hooks 之后，函数组件已经完全支持状态管理和副作用处理，可以完成类组件的所有功能。

函数组件的特点：
- **简洁直观**：使用普通函数语法，易于理解和编写
- **Hooks 支持**：可以使用所有 React Hooks（useState、useEffect、useContext 等）
- **易于测试**：纯函数更容易进行单元测试
- **更好的性能**：React 团队持续优化函数组件的性能

函数组件的基本结构：
```jsx
function MyComponent(props) {
  // 使用 props 解构获取传递的值
  const { title, children } = props;
  
  // 可以添加状态管理
  const [count, setCount] = useState(0);
  
  // 返回 JSX
  return (
    <div>
      <h1>{title}</h1>
      {children}
    </div>
  );
}
```

**2.2 类组件（了解）**

类组件是 React 早期的组件定义方式，通过继承 `React.Component` 来创建。虽然现在推荐使用函数组件，但类组件仍在一些老项目中存在，了解其语法有助于维护旧代码。

类组件的核心概念：
- **constructor 构造函数**：初始化 state 和绑定事件处理器的 this 上下文
- **render 方法**：返回组件的 JSX 结构，必须是纯函数
- **this.setState()**：更新组件状态，触发重新渲染
- **生命周期方法**：componentDidMount、componentDidUpdate、componentWillUnmount 等

注意：类组件的状态更新可能是异步的，多个 setState 调用可能会被合并处理。

**2.3 组件拆分、复用、嵌套、组合**

组件化开发的核心思想是将 UI 拆分为独立可复用的部件。

**组件拆分原则**：
- **单一职责**：每个组件只负责一个功能或展示一个明确的 UI 区域
- **高内聚低耦合**：相关的逻辑和样式放在一起，组件之间依赖最小化
- **粒度适中**：不是越小越好，要在复用性和可维护性之间找到平衡

**组件复用方式**：
- **直接复用**：创建通用的 UI 组件（Button、Input、Modal 等）
- ** Props 抽象**：通过不同的 props 值实现组件的差异化
- **组合模式**：将组件作为 props 传递，实现灵活的内容分发

**组件嵌套**：
- 父组件可以包含一个或多个子组件
- 形成树形结构，根组件通常是 App 组件
- 数据和回调函数通过 props 从父组件传递给子组件

**2.4 Props 传值**

Props（Properties）是 React 组件之间传递数据的主要方式。

**Props 的特点**：
- **只读性**：Props 是不可变的，组件不应该修改自己的 props
- **单向数据流**：数据从父组件流向子组件，只能通过回调函数反向修改
- **类型灵活**：可以传递任何 JavaScript 支持的数据类型

**Props 传递方式**：
```jsx
// 传递静态值
<Component title="Hello" />

// 传递动态值
<Component title={variable} />

// 传递对象
<Component data={{ name: 'Tom', age: 20 }} />

// 传递函数
<Component onClick={handleClick} />

// 传递 JSX 作为子元素
<Component>
  <ChildComponent />
</Component>
```

**2.5 Props 校验（PropTypes）**

PropTypes 是一种运行时类型检查机制，可以在开发阶段捕获组件 props 的类型错误。

**常用 PropTypes 类型**：
```jsx
import PropTypes from 'prop-types';

Component.propTypes = {
  // 基本类型
  name: PropTypes.string,
  age: PropTypes.number,
  isActive: PropTypes.bool,
  
  // 数组和对象
  users: PropTypes.array,
  user: PropTypes.object,
  
  // 指定对象结构
  user: PropTypes.shape({
    name: PropTypes.string,
    age: PropTypes.number
  }),
  
  // 必填属性
  title: PropTypes.string.isRequired,
  
  // 函数类型
  onClick: PropTypes.func,
  
  // 自定义验证器
  customProp: function(props, propName, componentName) {
    if (!/^[A-Z]/.test(props[propName])) {
      return new Error('必须以大写字母开头');
    }
  }
};
```

注意：PropTypes 校验仅在开发环境生效，不会影响生产环境性能。如果使用 TypeScript，建议直接使用编译时类型检查。

**经典案例：函数组件与 Props**

```jsx
// Button.jsx - 可复用的按钮组件
function Button({ children, variant = 'primary', onClick, disabled = false }) {
  const className = `btn btn-${variant}`;

  return (
    <button className={className} onClick={onClick} disabled={disabled}>
      {children}
    </button>
  );
}

// Card.jsx - 组合组件
function Card({ title, children }) {
  return (
    <div className="card">
      <div className="card-header">
        <h3>{title}</h3>
      </div>
      <div className="card-body">
        {children}
      </div>
    </div>
  );
}

// UserCard.jsx - 使用 Props 传值
function UserCard({ user }) {
  return (
    <Card title="用户信息">
      <p>姓名：{user.name}</p>
      <p>邮箱：{user.email}</p>
      <Button variant="secondary" onClick={() => alert(user.email)}>
        发送邮件
      </Button>
    </Card>
  );
}

// App.jsx - 组合使用
function App() {
  const user = { name: '张三', email: 'zhang@example.com' };

  return (
    <UserCard user={user} />
  );
}
```

**经典案例：类组件（了解）**

```jsx
// ClassComponent.jsx - 类组件示例
import { Component } from 'react';

class Counter extends Component {
  constructor(props) {
    super(props);
    this.state = {
      count: props.initialCount || 0
    };
    this.handleIncrement = this.handleIncrement.bind(this);
  }

  handleIncrement() {
    this.setState({ count: this.state.count + 1 });
  }

  render() {
    return (
      <div>
        <p>计数：{this.state.count}</p>
        <button onClick={this.handleIncrement}>+1</button>
      </div>
    );
  }
}
```

---

### 3. 状态管理（基础）

- [ ] `useState`：组件内部状态管理
- [ ] `useEffect`：副作用处理（数据请求、定时器、事件监听）
- [ ] 理解 React 渲染机制
- [ ] 理解状态更新规则

**详细概念：**

**3.1 useState 详解**

useState 是 React 中最常用的 Hook，用于在函数组件中添加状态管理。

**useState 的基本用法**：
```jsx
const [state, setState] = useState(initialValue);
```

**参数说明**：
- `initialValue`：状态的初始值，可以是任意类型（数字、字符串、布尔值、对象、数组等）
- 如果初始值需要通过计算得到，可以传入一个函数：`useState(() => computeInitialState())`

**返回值**：
- 返回一个包含两个元素的数组：`[当前状态值, 更新状态的函数]`
- 通常使用数组解构赋值：`const [count, setCount] = useState(0);`

**setState 函数的两种形式**：
1. **直接传入新值**：`setCount(5)`
2. **传入一个函数**：`setCount(prev => prev + 1)`，这在基于前一个状态计算新状态时特别重要

**重要特性**：
- **批量更新**：在事件处理函数中多次 setState 会被合并为一次更新，提高性能
- **异步更新**：状态更新是异步的，获取更新后的值需要使用 useEffect 或 setState 的函数形式
- **函数式更新**：当新状态依赖前一个状态时，使用函数形式可以确保基于最新的状态值计算

**3.2 useEffect 详解**

useEffect 是用于处理副作用的 Hook，类似于类组件的生命周期方法，但概念上更统一。

**副作用的常见类型**：
- 数据获取（API 调用）
- 订阅事件（WebSocket、键盘事件）
- 修改 DOM（文档标题、滚动位置）
- 设置定时器（setTimeout、setInterval）

**useEffect 的基本用法**：
```jsx
useEffect(() => {
  // 副作用逻辑
  return () => {
    // 清理逻辑（可选）
  };
}, [dependencyArray]);
```

**第二个参数（依赖数组）的作用**：
- **省略依赖数组**：每次渲染后都会执行 effect
- **空数组 `[]`**：只在组件挂载时执行一次，类似于 componentDidMount
- **有依赖项**：`[dep1, dep2]` 当指定依赖变化时重新执行

**清理副作用的重要性**：
- 取消订阅，防止内存泄漏
- 清除定时器，避免多个定时器累积
- 取消未完成的请求，避免状态更新到已卸载的组件

**常见错误**：
- 忘记在依赖数组中包含使用的外部变量，导致闭包陷阱
- 在 effect 中直接使用 async 函数，需要在内部定义 async 函数并调用

**3.3 React 渲染机制**

理解 React 的渲染机制有助于编写高性能的组件。

**渲染流程**：
1. **触发渲染**：状态改变（setState）或 props 变化
2. **组件更新**：React 调用组件函数获取新的 JSX
3. **DOM 更新**：React 比较新旧虚拟 DOM，计算出最小更新方案
4. **实际 DOM 更新**：只更新变化的部分

**虚拟 DOM 的优势**：
- 减少直接 DOM 操作，提升性能
- 通过 diffing 算法优化更新策略
- 跨平台能力（React Native、React VR 等）

**3.4 状态更新规则**

React 对状态更新有一些重要的规则需要遵守：

**状态不可变性**：
- 不要直接修改状态对象，如 `state.count++`
- 使用 setState 创建新对象：`setState({ ...state, count: state.count + 1 })`

**状态更新的批处理**：
- React 17 及之前：只在事件处理函数中批量更新
- React 18+：所有状态更新（包括 async 函数中的）自动批处理

**状态更新是异步的**：
```jsx
// 错误：此时 count 的值还没有更新
setCount(count + 1);
console.log(count); // 输出的是旧值

// 正确：使用 useEffect 监听变化
useEffect(() => {
  console.log(count); // 这是更新后的值
}, [count]);

// 或者使用函数形式
setCount(prev => {
  console.log(prev); // prev 是最新的值
  return prev + 1;
});
```

**经典案例：useState 状态管理**

```jsx
// Counter.jsx - 基础 useState
import { useState } from 'react';

function Counter() {
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>计数：{count}</p>
      <button onClick={() => setCount(count + 1)}>+1</button>
      <button onClick={() => setCount(count - 1)}>-1</button>
      <button onClick={() => setCount(0)}>重置</button>
    </div>
  );
}

export default Counter;
```

```jsx
// FormWithState.jsx - 多个状态
import { useState } from 'react';

function FormWithState() {
  const [username, setUsername] = useState('');
  const [password, setPassword] = useState('');
  const [isLoading, setIsLoading] = useState(false);

  const handleSubmit = (e) => {
    e.preventDefault();
    setIsLoading(true);
    console.log({ username, password });
    setTimeout(() => setIsLoading(false), 1000);
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        value={username}
        onChange={(e) => setUsername(e.target.value)}
        placeholder="用户名"
      />
      <input
        type="password"
        value={password}
        onChange={(e) => setPassword(e.target.value)}
        placeholder="密码"
      />
      <button type="submit" disabled={isLoading}>
        {isLoading ? '登录中...' : '登录'}
      </button>
    </form>
  );
}
```

**经典案例：useEffect 副作用处理**

```jsx
// DataFetcher.jsx - 数据请求
import { useState, useEffect } from 'react';

function UserProfile({ userId }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    let cancelled = false;

    async function fetchUser() {
      try {
        setLoading(true);
        const response = await fetch(`/api/users/${userId}`);
        if (!response.ok) throw new Error('获取用户失败');
        const data = await response.json();
        if (!cancelled) setUser(data);
      } catch (err) {
        if (!cancelled) setError(err.message);
      } finally {
        if (!cancelled) setLoading(false);
      }
    }

    fetchUser();

    return () => {
      cancelled = true;
    };
  }, [userId]);

  if (loading) return <div>加载中...</div>;
  if (error) return <div>错误：{error}</div>;
  if (!user) return null;

  return (
    <div>
      <h1>{user.name}</h1>
      <p>{user.bio}</p>
    </div>
  );
}
```

```jsx
// Timer.jsx - 定时器
import { useState, useEffect } from 'react';

function Timer() {
  const [seconds, setSeconds] = useState(0);
  const [isRunning, setIsRunning] = useState(true);

  useEffect(() => {
    let interval = null;

    if (isRunning) {
      interval = setInterval(() => {
        setSeconds(prev => prev + 1);
      }, 1000);
    }

    return () => {
      if (interval) clearInterval(interval);
    };
  }, [isRunning]);

  const formatTime = (totalSeconds) => {
    const mins = Math.floor(totalSeconds / 60);
    const secs = totalSeconds % 60;
    return `${mins.toString().padStart(2, '0')}:${secs.toString().padStart(2, '0')}`;
  };

  return (
    <div>
      <p>计时器：{formatTime(seconds)}</p>
      <button onClick={() => setIsRunning(!isRunning)}>
        {isRunning ? '暂停' : '继续'}
      </button>
      <button onClick={() => setSeconds(0)}>重置</button>
    </div>
  );
}
```

---

### 4. 事件处理

- [ ] React 合成事件
- [ ] 事件绑定、传参
- [ ] 阻止默认行为、阻止冒泡

**详细概念：**

**4.1 React 合成事件（SyntheticEvent）**

React 为了保持一致性和跨浏览器兼容性，封装了自己的事件系统——合成事件（SyntheticEvent）。

**合成事件的特点**：
- **跨浏览器兼容**：封装了浏览器原生事件的差异，提供统一接口
- **自动绑定 this**：在 class 组件中事件处理器默认不会自动绑定 this，需要手动绑定或使用箭头函数
- **事件委托**：React 17 之前事件冒泡到 document 层面处理；React 17 之后改为在根元素上处理
- **池化事件对象**：React 17 之前事件对象会被重用，从池中获取；React 17 之后移除了池化，所有事件变为持久化

**支持的常见事件类型**：
- **Clipboard**：onCopy、onCut、onPaste
- **Keyboard**：onKeyDown、onKeyPress、onKeyUp
- **Focus**：onFocus、onBlur
- **Form**：onChange、onInput、onSubmit
- **Mouse**：onClick、onDoubleClick、onMouseEnter、onMouseLeave
- **Touch**：onTouchStart、onTouchMove、onTouchEnd
- **Wheel**：onWheel

**4.2 事件绑定与传参**

在 JSX 中绑定事件处理器有两种方式：

**方式一：直接传入函数引用**（推荐）
```jsx
<button onClick={handleClick}>点击</button>
```

**方式二：使用箭头函数包装**
```jsx
<button onClick={() => handleClick(param)}>点击</button>
```

注意：方式二会在每次渲染时创建新的函数引用，可能导致子组件不必要的重新渲染。

**向事件处理器传参的常见方式**：
```jsx
// 方式1：箭头函数包装
<button onClick={() => handleClick(id)}>删除</button>

// 方式2：bind 方法绑定
<button onClick={handleClick.bind(null, id)}>删除</button>

// 方式3：柯里化函数（常用于类组件）
handleClick = (id) => (e) => {
  // 可以同时访问 id 和事件对象 e
  console.log(id, e);
}

<button onClick={this.handleClick(id)}>删除</button>
```

**4.3 阻止默认行为与阻止冒泡**

**阻止默认行为**：
```jsx
// 使用 e.preventDefault()
function handleSubmit(e) {
  e.preventDefault();
  console.log('表单提交被阻止');
}

<form onSubmit={handleSubmit}>...</form>

// 阻止链接默认跳转
<a href="https://example.com" onClick={(e) => {
  e.preventDefault();
  // 自定义跳转逻辑
}}>链接</a>
```

**阻止事件冒泡**：
```jsx
function handleClick(e) {
  e.stopPropagation();
  console.log('点击事件不会冒泡');
}

<div onClick={() => console.log('父元素')}>
  <button onClick={handleClick}>子元素</button>
</div>
```

**与原生事件的区别**：
- 在 React 中所有事件名称使用驼峰命名（onClick 而不是 onclick）
- 在 React 中必须显式调用 `e.stopPropagation()` 阻止冒泡
- 在 React 中无法使用 `return false` 阻止默认行为，必须调用 `e.preventDefault()`

**经典案例：事件处理**

```jsx
// EventExamples.jsx
function EventExamples() {
  const handleClick = (e) => {
    console.log('按钮点击', e);
  };

  const handleClickWithParam = (name, e) => {
    console.log(`你好，${name}！`, e);
  };

  const handleSubmit = (e) => {
    e.preventDefault();
    console.log('表单提交');
  };

  const handleBubble = (label, e) => {
    console.log(`${label} 被点击`);
  };

  const handleStopBubble = (e) => {
    e.stopPropagation();
    console.log('停止冒泡');
  };

  return (
    <div onClick={() => handleBubble('父容器', e)}>
      <button onClick={handleClick}>基本点击</button>

      <button onClick={(e) => handleClickWithParam('张三', e)}>
        带参数点击
      </button>

      <form onSubmit={handleSubmit}>
        <button type="submit">提交表单（阻止默认行为）</button>
      </form>

      <div onClick={() => handleBubble('外层', event)}>
        <button onClick={handleStopBubble}>停止冒泡</button>
      </div>
    </div>
  );
}
```

---

### 5. 列表与条件渲染

- [ ] `map` 渲染列表
- [ ] `key` 属性的作用
- [ ] 三元运算条件渲染
- [ ] `&&` 逻辑实现条件渲染

**详细概念：**

**5.1 map 渲染列表**

在 React 中渲染列表使用 JavaScript 的 `map()` 方法，它接受一个函数，对数组每个元素执行并返回新的元素。

**基本语法**：
```jsx
const items = ['苹果', '香蕉', '橙子'];

return (
  <ul>
    {items.map(item => (
      <li>{item}</li>
    ))}
  </ul>
);
```

**渲染组件列表**：
```jsx
const components = [
  { id: 1, name: 'Header' },
  { id: 2, name: 'Content' },
  { id: 3, name: 'Footer' },
];

return (
  <div>
    {components.map(comp => (
      <ComponentItem key={comp.id} name={comp.name} />
    ))}
  </div>
);
```

**5.2 key 属性的作用**

key 是 React 用来识别列表中每个元素唯一性的特殊 prop，在渲染动态列表时是必需的。

**key 的重要性**：
- **高效更新**：React 使用 key 来判断哪些元素发生了变化，避免不必要的重新渲染
- **正确识别元素**：帮助 React 正确匹配新旧虚拟 DOM 树中的元素
- **列表稳定性**：保持列表项的识别性，即使顺序发生变化也能正确更新

**key 的选择原则**：
1. **使用唯一 ID**：优先使用数据中自然存在的唯一标识符
2. **避免使用索引**：不要使用数组索引作为 key，当列表顺序变化时会导致渲染错误
3. **保持稳定**：key 应该在列表重新排序时保持稳定对应到同一个数据项

**错误的 key 使用**：
```jsx
// 不好：使用索引作为 key，列表变化时会导致问题
{items.map((item, index) => (
  <li key={index}>{item.name}</li>
))}

// 好：使用唯一 ID
{items.map(item => (
  <li key={item.id}>{item.name}</li>
))}
```

**5.3 三元运算条件渲染**

使用 JavaScript 三元运算符 `condition ? trueValue : falseValue` 在 JSX 中实现条件渲染。

**基本用法**：
```jsx
{isLoggedIn ? (
  <UserProfile name={userName} />
) : (
  <LoginPrompt />
)}
```

**嵌套三元运算**（注意可读性）：
```jsx
{status === 'loading' ? (
  <Spinner />
) : status === 'error' ? (
  <ErrorMessage error={error} />
) : (
  <DataList data={data} />
)}
```

**5.4 && 逻辑实现条件渲染**

使用逻辑与 `&&` 运算符，当条件为 true 时渲染元素，为 false 时不渲染。

**基本用法**：
```jsx
{isOnline && <OnlineIndicator />}

// 等价于
{isOnline ? <OnlineIndicator /> : null}
```

**注意事项**：
- ** falsy 值问题**：如果 `isOnline` 是 `0`，React 仍会渲染 `0`。通常这不是问题，但需要留意
- **左侧不能是 JSX**：左侧必须是布尔值或能返回布尔值的表达式

```jsx
// 错误示例
{items.length && <ItemCount count={items.length} />}
// 当 items.length 为 0 时，会渲染 0 而不是什么都不渲染

// 正确示例
{items.length > 0 && <ItemCount count={items.length} />}
```

**经典案例：列表渲染**

```jsx
// UserList.jsx
function UserList() {
  const users = [
    { id: 1, name: '张三', age: 25, email: 'zhang@example.com' },
    { id: 2, name: '李四', age: 30, email: 'li@example.com' },
    { id: 3, name: '王五', age: 28, email: 'wang@example.com' },
  ];

  return (
    <ul>
      {users.map(user => (
        <li key={user.id}>
          <strong>{user.name}</strong> - {user.age}岁 - {user.email}
        </li>
      ))}
    </ul>
  );
}
```

**经典案例：条件渲染**

```jsx
// ConditionalRendering.jsx
function ConditionalRendering() {
  const [isLoggedIn, setIsLoggedIn] = useState(false);
  const [user, setUser] = useState({ name: '张三', role: 'admin' });

  return (
    <div>
      {/* 三元运算 */}
      {isLoggedIn ? (
        <p>欢迎，{user.name}！</p>
      ) : (
        <p>请登录</p>
      )}

      {/* && 逻辑 */}
      {user.role === 'admin' && (
        <button>管理后台</button>
      )}

      {/* 多种状态 */}
      {user.role === 'vip' ? (
        <span>VIP 用户</span>
      ) : user.role === 'admin' ? (
        <span>管理员</span>
      ) : (
        <span>普通用户</span>
      )}

      <button onClick={() => setIsLoggedIn(!isLoggedIn)}>
        {isLoggedIn ? '退出' : '登录'}
      </button>
    </div>
  );
}
```

**经典案例：综合列表与条件渲染**

```jsx
// FilterableUserList.jsx
function FilterableUserList() {
  const [users, setUsers] = useState([
    { id: 1, name: '张三', age: 25, vip: true },
    { id: 2, name: '李四', age: 30, vip: false },
    { id: 3, name: '王五', age: 35, vip: true },
    { id: 4, name: '赵六', age: 28, vip: false },
  ]);
  const [showVipOnly, setShowVipOnly] = useState(false);
  const [searchName, setSearchName] = useState('');

  const filteredUsers = users.filter(user => {
    const matchVip = showVipOnly ? user.vip : true;
    const matchName = user.name.includes(searchName);
    return matchVip && matchName;
  });

  return (
    <div>
      <label>
        <input
          type="checkbox"
          checked={showVipOnly}
          onChange={(e) => setShowVipOnly(e.target.checked)}
        />
        只看 VIP 用户
      </label>

      <input
        type="text"
        value={searchName}
        onChange={(e) => setSearchName(e.target.value)}
        placeholder="搜索用户名..."
      />

      <p>共 {filteredUsers.length} 位用户</p>

      <ul>
        {filteredUsers.map(user => (
          <li key={user.id}>
            {user.vip && <span>[VIP]</span>}
            {user.name} - {user.age}岁
          </li>
        ))}
      </ul>

      {filteredUsers.length === 0 && (
        <p>没有找到匹配的用户</p>
      )}
    </div>
  );
}
```

---

### 6. 表单处理

- [ ] 受控组件（输入框、单选、多选、下拉框）
- [ ] 非受控组件（useRef 获取 DOM 值）

**详细概念：**

**6.1 受控组件**

受控组件是指表单元素的值由 React 状态（state）控制的组件。用户的输入会触发状态更新，状态的更新又会驱动表单元素的显示。

**受控组件的核心**：
1. **value 属性绑定状态**：表单元素的值由 React 状态决定
2. **onChange 更新状态**：用户输入时通过 onChange 事件更新状态
3. **单向数据流**：表单状态由组件自己管理，父组件通过 props 控制和获取值

**输入框（text/password）**：
```jsx
const [value, setValue] = useState('');

<input 
  value={value} 
  onChange={(e) => setValue(e.target.value)} 
/>
```

**文本域（textarea）**：
```jsx
const [content, setContent] = useState('');

<textarea 
  value={content} 
  onChange={(e) => setContent(e.target.value)} 
/>
```

**下拉框（select）**：
```jsx
const [selected, setSelected] = useState('cn');

<select value={selected} onChange={(e) => setSelected(e.target.value)}>
  <option value="cn">中国</option>
  <option value="us">美国</option>
</select>
```

**单选框（radio）**：
```jsx
const [gender, setGender] = useState('male');

<input 
  type="radio" 
  name="gender" 
  value="male"
  checked={gender === 'male'}
  onChange={(e) => setGender(e.target.value)}
/>
<input 
  type="radio" 
  name="gender" 
  value="female"
  checked={gender === 'female'}
  onChange={(e) => setGender(e.target.value)}
/>
```

**复选框（checkbox）**：
```jsx
const [agree, setAgree] = useState(false);

<input 
  type="checkbox" 
  checked={agree} 
  onChange={(e) => setAgree(e.target.checked)} 
/>
```

**受控组件的优势**：
- 可以即时验证用户输入
- 可以动态禁用或启用表单元素
- 可以实时转换用户输入的格式（如自动添加千分位）
- 更容易实现复杂的表单逻辑

**6.2 非受控组件**

非受控组件通过 ref 直接访问 DOM 元素来获取表单值，而不是通过 state 管理。

**使用 useRef 获取 DOM**：
```jsx
const inputRef = useRef(null);

<input ref={inputRef} type="text" />

// 获取值
const value = inputRef.current.value;
```

**defaultValue 设置默认值**：
```jsx
<input 
  ref={inputRef} 
  type="text" 
  defaultValue="初始值" 
/>
```

**文件输入框（非受控组件的特殊情况）**：
```jsx
const fileRef = useRef(null);

<input type="file" ref={fileRef} />

// 获取文件
const file = fileRef.current.files[0];
```

**非受控组件的使用场景**：
- 表单简单，不需要复杂的验证或动态控制
- 需要集成不支持受控组件的第三方库
- 需要直接访问 DOM 元素（如 focus、select 等）
- 性能考虑，避免频繁的状态更新

**受控 vs 非受控对比**：

| 特性 | 受控组件 | 非受控组件 |
|------|----------|------------|
| 数据来源 | React state | DOM 本身 |
| 状态管理 | 组件内部 state | ref 获取 |
| 实时验证 | 支持 | 需要手动获取值后验证 |
| 动态控制 | 易于禁用/格式化 | 需要额外处理 |
| 适用场景 | 复杂表单 | 简单表单、第三方集成 |

**经典案例：受控组件**

```jsx
// ControlledForm.jsx - 受控表单
function ControlledForm() {
  const [form, setForm] = useState({
    username: '',
    password: '',
    gender: '',
    hobbies: [],
    country: ''
  });
  const [errors, setErrors] = useState({});

  const handleChange = (e) => {
    const { name, value, type, checked } = e.target;
    setForm(prev => ({
      ...prev,
      [name]: type === 'checkbox'
        ? checked
          ? [...prev.hobbies, value]
          : prev.hobbies.filter(h => h !== value)
        : value
    }));
  };

  const validate = () => {
    const newErrors = {};
    if (!form.username) newErrors.username = '用户名不能为空';
    if (!form.password) newErrors.password = '密码不能为空';
    if (form.password.length < 6) newErrors.password = '密码至少6位';
    if (!form.gender) newErrors.gender = '请选择性别';
    if (form.hobbies.length === 0) newErrors.hobbies = '请至少选择一个爱好';
    if (!form.country) newErrors.country = '请选择国家';
    return newErrors;
  };

  const handleSubmit = (e) => {
    e.preventDefault();
    const newErrors = validate();
    setErrors(newErrors);
    if (Object.keys(newErrors).length === 0) {
      console.log('表单数据：', form);
      alert('提交成功！');
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <div>
        <input
          name="username"
          value={form.username}
          onChange={handleChange}
          placeholder="用户名"
        />
        {errors.username && <span style={{ color: 'red' }}>{errors.username}</span>}
      </div>

      <div>
        <input
          name="password"
          type="password"
          value={form.password}
          onChange={handleChange}
          placeholder="密码"
        />
        {errors.password && <span style={{ color: 'red' }}>{errors.password}</span>}
      </div>

      <div>
        <label>
          <input
            type="radio"
            name="gender"
            value="male"
            checked={form.gender === 'male'}
            onChange={handleChange}
          />
          男
        </label>
        <label>
          <input
            type="radio"
            name="gender"
            value="female"
            checked={form.gender === 'female'}
            onChange={handleChange}
          />
          女
        </label>
        {errors.gender && <span style={{ color: 'red' }}>{errors.gender}</span>}
      </div>

      <div>
        <label>
          <input
            type="checkbox"
            name="hobbies"
            value="reading"
            checked={form.hobbies.includes('reading')}
            onChange={handleChange}
          />
          阅读
        </label>
        <label>
          <input
            type="checkbox"
            name="hobbies"
            value="sports"
            checked={form.hobbies.includes('sports')}
            onChange={handleChange}
          />
          运动
        </label>
        <label>
          <input
            type="checkbox"
            name="hobbies"
            value="music"
            checked={form.hobbies.includes('music')}
            onChange={handleChange}
          />
          音乐
        </label>
        {errors.hobbies && <span style={{ color: 'red' }}>{errors.hobbies}</span>}
      </div>

      <div>
        <select
          name="country"
          value={form.country}
          onChange={handleChange}
        >
          <option value="">请选择国家</option>
          <option value="cn">中国</option>
          <option value="us">美国</option>
          <option value="jp">日本</option>
        </select>
        {errors.country && <span style={{ color: 'red' }}>{errors.country}</span>}
      </div>

      <button type="submit">提交</button>
    </form>
  );
}
```

**经典案例：非受控组件**

```jsx
// UncontrolledForm.jsx - 非受控组件
import { useRef } from 'react';

function UncontrolledForm() {
  const usernameRef = useRef(null);
  const fileRef = useRef(null);

  const handleSubmit = (e) => {
    e.preventDefault();

    console.log('用户名：', usernameRef.current.value);
    console.log('文件：', fileRef.current.files[0]);

    usernameRef.current.focus();
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        ref={usernameRef}
        type="text"
        placeholder="用户名"
        defaultValue="张三"
      />

      <input
        ref={fileRef}
        type="file"
      />

      <button type="submit">提交</button>
    </form>
  );
}
```

---

## 了解即可

- 类组件的生命周期概念
- create-react-app vs Vite 的区别

## 阶段成果

能开发以下应用：
- 登录页
- 列表页
- 简单表单
- TodoList

## 练习项目建议

1. **TodoList**：实现添加、删除、完成待办事项
2. **登录页**：实现用户名密码验证、表单提交
3. **列表页**：展示数据列表，支持筛选和搜索

## 知识点速查

| 概念 | 核心要点 |
|------|----------|
| JSX | 1. 只能返回一个根元素 2. className 代替 class 3. 表达式用 {} 包裹 |
| useState | 1. 接收初始值 2. 返回 [state, setState] 3. 更新是异步的 |
| useEffect | 1. 第二个参数控制执行时机 2. [] 表示仅执行一次 3. 返回函数用于清理 |
| key | 1. 列表中唯一标识 2. 帮助 React 高效更新 3. 不推荐使用索引 |

## 下一阶段预告

完成基础阶段后，将进入 [React 进阶基础](./stage-2-advanced-basic.md)，你将学习：
- 高阶 Hooks（useMemo、useCallback、useReducer）
- 组件通信全方案
- React Router 路由管理
- 性能优化基础
