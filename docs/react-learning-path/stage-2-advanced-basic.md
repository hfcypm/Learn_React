# 阶段二：React 进阶基础（核心拔高）

**目标**：解决复杂场景问题、优化代码、掌握高频 Hooks

## 必须掌握的技能

### 1. Hooks 进阶

- [ ] `useRef`：获取 DOM、存储可变值、跨渲染周期存数据
- [ ] `useMemo`：缓存计算结果，优化性能
- [ ] `useCallback`：缓存函数，避免子组件重复渲染
- [ ] `useContext` + `createContext`：跨组件传值（替代多层 Props）
- [ ] `useReducer`：复杂状态逻辑管理（类似 Redux 精简版）

**详细概念：**

**1.1 useRef 详解**

useRef 是 React 中一个重要的 Hook，它返回一个 ref 对象，该对象的 `.current` 属性可以存储可变的值。

**useRef 的三个核心用途**：

**1. 获取 DOM 引用**：
```jsx
const inputRef = useRef(null);

<input ref={inputRef} />

// 调用 DOM 方法
inputRef.current.focus();
inputRef.current.select();
inputRef.current.value;
```

**2. 存储跨渲染周期存数据**：
- ref 的值存储在组件外部，不会因组件重新渲染而丢失
- 更新 ref 不会触发组件重新渲染
- 适合存储定时器 ID、订阅取消函数等

```jsx
const timerRef = useRef(null);

useEffect(() => {
  timerRef.current = setInterval(() => {
    setCount(c => c + 1);
  }, 1000);
  
  return () => clearInterval(timerRef.current);
}, []);
```

**3. 保存上一个渲染周期的值**：
```jsx
const prevValueRef = useRef(null);

useEffect(() => {
  prevValueRef.current = currentValue;
}, [currentValue]);

// prevValueRef.current 保存的是上一个周期的 currentValue 值
```

**useRef vs useState 对比**：

| 特性 | useRef | useState |
|------|--------|----------|
| 更新触发渲染 | 否 | 是 |
| 返回值 | `{ current: value }` | `[value, setValue]` |
| 用途 | DOM 引用、存储副作用相关数据 | UI 相关状态 |
| 可变性 | current 可变 | 通过 setValue 更新 |

**1.2 useMemo 详解**

useMemo 用于缓存计算结果，避免在每次渲染时重新计算昂贵的值。

**基本语法**：
```jsx
const memoizedValue = useMemo(() => computeExpensiveValue(a, b), [a, b]);
```

**使用场景**：
- **昂贵计算**：如排序、过滤、大数据处理等
- **稳定对象引用**：避免每次渲染创建新对象
- **稳定函数引用**：配合 useCallback 使用

**依赖数组规则**：
- 省略依赖数组：每次渲染都重新计算
- 空数组：只计算一次，后续渲染使用缓存值
- 有依赖项：依赖变化时重新计算

**注意事项**：
- 不要过度使用 useMemo，它也有性能开销
- 对于简单计算，直接计算可能更快
- 确保计算函数是纯函数，没有副作用

**1.3 useCallback 详解**

useCallback 用于缓存函数引用，避免因函数引用变化导致子组件不必要的重新渲染。

**基本语法**：
```jsx
const memoizedCallback = useCallback(
  () => doSomething(a, b),
  [a, b]
);
```

**与 useMemo 的关系**：
```jsx
// 这两等价
const memoizedCallback = useCallback(() => doSomething(a, b), [a, b]);
const memoizedCallback = useMemo(() => () => doSomething(a, b), [a, b]);
```

**使用场景**：
- 传递给子组件的回调函数
- 作为其他 Hooks 的依赖项
- 传递给需要稳定引用的第三方库

**常见误区**：
- 不是所有函数都需要 useCallback，过度使用反而增加开销
- 只有当函数作为 props 传递给使用 memo 包装的子组件时才有意义
- 配合 React.memo 使用才能发挥最大效果

**1.4 useContext + createContext 详解**

Context 提供了一种在组件树中传递数据的方式，避免层层传递 props（prop drilling）。

**创建 Context**：
```jsx
const MyContext = createContext(defaultValue);
```

** Provider 供应数据**：
```jsx
<MyContext.Provider value={sharedValue}>
  {children}
</MyContext.Provider>
```

**消费 Context**：
```jsx
// 方式1：useContext Hook（推荐）
const value = useContext(MyContext);

// 方式2：Consumer 组件
<MyContext.Consumer>
  {value => <Component value={value} />}
</MyContext.Consumer>
```

**最佳实践**：
- 将 Context 按功能拆分为多个小 Context，避免不必要的重新渲染
- 考虑使用 useMemo 优化 Context value
- 提供默认值时要注意默认为 undefined 时的处理

**1.5 useReducer 详解**

useReducer 是 useState 的替代方案，适用于复杂的状态逻辑。

**基本语法**：
```jsx
const [state, dispatch] = useReducer(reducer, initialState);
```

**reducer 函数结构**：
```jsx
function reducer(state, action) {
  switch (action.type) {
    case 'INCREMENT':
      return { ...state, count: state.count + 1 };
    case 'DECREMENT':
      return { ...state, count: state.count - 1 };
    default:
      return state;
  }
}
```

**与 useState 的选择**：
- useState：简单状态，状态更新逻辑不复杂
- useReducer：相关的一组状态，状态更新逻辑复杂，状态更新逻辑类似或有规律可循

**useReducer 的优势**：
- 集中状态逻辑，便于管理和调试
- 可以传递 dispatch 给子组件，而不需要传递回调
- 更容易实现撤销/重做等高级功能

**经典案例：useRef 使用**

```jsx
// FocusInput.jsx - 获取 DOM
import { useRef } from 'react';

function FocusInput() {
  const inputRef = useRef(null);

  const handleClick = () => {
    inputRef.current.focus();
  };

  return (
    <div>
      <input ref={inputRef} type="text" placeholder="输入内容..." />
      <button onClick={handleClick}>聚焦输入框</button>
    </div>
  );
}

// TimerWithRef.jsx - 跨渲染周期存储值
import { useRef, useState, useEffect } from 'react';

function TimerWithRef() {
  const [count, setCount] = useState(0);
  const intervalRef = useRef(null);

  useEffect(() => {
    intervalRef.current = setInterval(() => {
      setCount(prev => prev + 1);
    }, 1000);

    return () => clearInterval(intervalRef.current);
  }, []);

  const stopTimer = () => {
    if (intervalRef.current) {
      clearInterval(intervalRef.current);
    }
  };

  return (
    <div>
      <p>计数：{count}</p>
      <button onClick={stopTimer}>停止</button>
    </div>
  );
}

// PreviousValue.jsx - 存储上一个值
import { useRef, useState, useEffect } from 'react';

function PreviousValue() {
  const [value, setValue] = useState('');
  const previousValueRef = useRef('');

  useEffect(() => {
    previousValueRef.current = value;
  }, [value]);

  return (
    <div>
      <input
        value={value}
        onChange={(e) => setValue(e.target.value)}
        placeholder="输入内容..."
      />
      <p>当前值：{value}</p>
      <p>上一个值：{previousValueRef.current}</p>
    </div>
  );
}
```

**经典案例：useMemo 缓存计算**

```jsx
// ExpensiveCalculation.jsx
import { useState, useMemo } from 'react';

function ExpensiveCalculation() {
  const [number, setNumber] = useState(5);
  const [multiplier, setMultiplier] = useState(2);

  const expensiveResult = useMemo(() => {
    console.log('计算中...');
    let result = 0;
    for (let i = 0; i < 1000000000; i++) {
      result += number * multiplier;
    }
    return result;
  }, [number, multiplier]);

  return (
    <div>
      <p>数字：{number}</p>
      <p>倍数：{multiplier}</p>
      <p>计算结果：{expensiveResult}</p>
      <button onClick={() => setNumber(n => n + 1)}>+1 数字</button>
      <button onClick={() => setMultiplier(m => m + 1)}>+1 倍数</button>
    </div>
  );
}
```

**经典案例：useCallback 缓存函数**

```jsx
// MemoizedCallbacks.jsx
import { useState, useCallback } from 'react';

function ParentComponent() {
  const [count, setCount] = useState(0);
  const [name, setName] = useState('');

  const handleClick = useCallback(() => {
    console.log('按钮点击');
  }, []);

  const handleSubmit = useCallback((data) => {
    console.log('提交数据：', data);
  }, []);

  return (
    <div>
      <ChildComponent onClick={handleClick} />
      <AnotherChild onSubmit={handleSubmit} />
      <p>计数：{count}</p>
      <button onClick={() => setCount(c => c + 1)}>增加</button>
      <input
        value={name}
        onChange={(e) => setName(e.target.value)}
        placeholder="输入名字"
      />
    </div>
  );
}

function ChildComponent({ onClick }) {
  console.log('ChildComponent 渲染');

  return <button onClick={onClick}>点击</button>;
}

function AnotherChild({ onSubmit }) {
  console.log('AnotherChild 渲染');

  return <button onClick={() => onSubmit({ name: 'test' })}>提交</button>;
}
```

**经典案例：useReducer 复杂状态**

```jsx
// ShoppingCart.jsx
import { useReducer } from 'react';

const initialState = {
  items: [],
  total: 0,
  discount: 0
};

function cartReducer(state, action) {
  switch (action.type) {
    case 'ADD_ITEM':
      const existingItem = state.items.find(item => item.id === action.payload.id);
      if (existingItem) {
        return {
          ...state,
          items: state.items.map(item =>
            item.id === action.payload.id
              ? { ...item, quantity: item.quantity + 1 }
              : item
          ),
          total: state.total + action.payload.price
        };
      }
      return {
        ...state,
        items: [...state.items, { ...action.payload, quantity: 1 }],
        total: state.total + action.payload.price
      };

    case 'REMOVE_ITEM':
      const item = state.items.find(item => item.id === action.payload);
      return {
        ...state,
        items: state.items.filter(item => item.id !== action.payload),
        total: state.total - item.price * item.quantity
      };

    case 'UPDATE_QUANTITY':
      const targetItem = state.items.find(item => item.id === action.payload.id);
      const quantityDiff = action.payload.quantity - targetItem.quantity;
      return {
        ...state,
        items: state.items.map(item =>
          item.id === action.payload.id
            ? { ...item, quantity: action.payload.quantity }
            : item
        ),
        total: state.total + targetItem.price * quantityDiff
      };

    case 'APPLY_DISCOUNT':
      return {
        ...state,
        discount: action.payload
      };

    case 'CLEAR_CART':
      return initialState;

    default:
      return state;
  }
}

function ShoppingCart() {
  const [state, dispatch] = useReducer(cartReducer, initialState);

  const products = [
    { id: 1, name: '苹果', price: 5 },
    { id: 2, name: '香蕉', price: 3 },
    { id: 3, name: '橙子', price: 4 },
  ];

  const finalTotal = state.total * (1 - state.discount / 100);

  return (
    <div>
      <h2>购物车</h2>

      <div>
        <h3>商品列表</h3>
        {products.map(product => (
          <div key={product.id}>
            <span>{product.name} - ¥{product.price}</span>
            <button onClick={() => dispatch({ type: 'ADD_ITEM', payload: product })}>
              添加
            </button>
          </div>
        ))}
      </div>

      <div>
        <h3>购物车内容</h3>
        {state.items.length === 0 ? (
          <p>购物车为空</p>
        ) : (
          state.items.map(item => (
            <div key={item.id}>
              <span>{item.name} x {item.quantity} = ¥{item.price * item.quantity}</span>
              <button onClick={() => dispatch({
                type: 'UPDATE_QUANTITY',
                payload: { id: item.id, quantity: item.quantity - 1 }
              })}>-</button>
              <button onClick={() => dispatch({
                type: 'UPDATE_QUANTITY',
                payload: { id: item.id, quantity: item.quantity + 1 }
              })}>+</button>
              <button onClick={() => dispatch({ type: 'REMOVE_ITEM', payload: item.id })}>
                删除
              </button>
            </div>
          ))
        )}
      </div>

      <div>
        <p>总价：¥{state.total}</p>
        <p>折扣：{state.discount}%</p>
        <p>最终价格：¥{finalTotal.toFixed(2)}</p>
        <button onClick={() => dispatch({ type: 'APPLY_DISCOUNT', payload: 10 })}>
          使用 10% 折扣
        </button>
        <button onClick={() => dispatch({ type: 'CLEAR_CART' })}>
          清空购物车
        </button>
      </div>
    </div>
  );
}
```

**经典案例：useContext 跨组件传值**

```jsx
// ThemeContext.jsx
import { createContext, useContext, useState } from 'react';

const ThemeContext = createContext();

function ThemeProvider({ children }) {
  const [theme, setTheme] = useState('light');
  const [fontSize, setFontSize] = useState(16);

  const toggleTheme = () => {
    setTheme(prev => prev === 'light' ? 'dark' : 'light');
  };

  const value = {
    theme,
    fontSize,
    setFontSize,
    toggleTheme,
    isDark: theme === 'dark'
  };

  return (
    <ThemeContext.Provider value={value}>
      {children}
    </ThemeContext.Provider>
  );
}

function useTheme() {
  const context = useContext(ThemeContext);
  if (!context) {
    throw new Error('useTheme 必须在 ThemeProvider 内使用');
  }
  return context;
}

export { ThemeProvider, useTheme };
```

```jsx
// Header.jsx
import { useTheme } from './ThemeContext';

function Header() {
  const { theme, toggleTheme } = useTheme();

  return (
    <header style={{
      background: theme === 'dark' ? '#333' : '#fff',
      color: theme === 'dark' ? '#fff' : '#333',
      padding: '1rem'
    }}>
      <button onClick={toggleTheme}>切换主题</button>
    </header>
  );
}

export default Header;
```

```jsx
// Content.jsx
import { useTheme } from './ThemeContext';

function Content() {
  const { theme, fontSize, setFontSize } = useTheme();

  return (
    <div style={{
      background: theme === 'dark' ? '#222' : '#f5f5f5',
      color: theme === 'dark' ? '#fff' : '#333',
      padding: '1rem',
      fontSize: `${fontSize}px`
    }}>
      <p>当前主题：{theme}</p>
      <p>字体大小：{fontSize}px</p>
      <input
        type="range"
        min="12"
        max="24"
        value={fontSize}
        onChange={(e) => setFontSize(Number(e.target.value))}
      />
    </div>
  );
}

export default Content;
```

```jsx
// App.jsx
import { ThemeProvider } from './ThemeContext';
import Header from './Header';
import Content from './Content';

function App() {
  return (
    <ThemeProvider>
      <Header />
      <Content />
    </ThemeProvider>
  );
}

export default App;
```

---

### 2. 组件进阶

- [ ] 组件通信：父子/兄弟/跨层级传值全方案
- [ ] 高阶组件（HOC）- 了解思想
- [ ] Render Props - 了解思想
- [ ] 自定义 Hooks：封装复用逻辑（表单、请求、定时器等）

**详细概念：**

**2.1 组件通信方案全解**

React 组件之间的通信有多种方式，根据组件关系选择合适的方案：

**父子通信（Parent → Child）**：
- 通过 props 传递数据
- 父组件传递函数作为 props，子组件调用通知父组件

```jsx
// 父组件
<ChildComponent 
  data={parentData} 
  onChildEvent={handleChildEvent}
/>
```

**子父通信（Child → Parent）**：
- 父组件传递回调函数给子组件
- 子组件通过调用回调传递数据

```jsx
// 子组件
function Child({ onDataChange }) {
  return <button onClick={() => onDataChange('新数据')}>通知父组件</button>;
}
```

**兄弟通信（Sibling → Sibling）**：
- 通过共同的父组件中转
- 组件 A 更新父组件状态，父组件将数据传给组件 B

**跨多层通信**：
- **层层传递**：不推荐，导致代码耦合
- **Context**：适合全局数据（主题、用户信息）
- **状态管理库**：适合复杂应用（Redux、Zustand）

**2.2 高阶组件（HOC）**

高阶组件是一个函数，接收一个组件并返回一个新组件。它是一种代码复用模式，用于共享逻辑。

**HOC 的基本结构**：
```jsx
function withHOC(WrappedComponent) {
  return function EnhancedComponent(props) {
    // 添加额外 props 或逻辑
    const extraProp = '额外数据';
    return <WrappedComponent {...props} extraProp={extraProp} />;
  }
}
```

**常见用途**：
- 属性代理：添加 props、访问 DOM
- 状态提升：添加状态管理
- 条件渲染：基于条件决定渲染
- 样式增强：添加样式或类名

**注意事项**：
- 不要在 HOC 内部修改原组件
- HOC 应该传递不相关的 props
- 使用 displayName 便于调试
- 不推荐多层 HOC 嵌套

**2.3 Render Props**

Render Props 是一种在组件之间共享代码的技术，使用一个 prop 值为函数，这个函数返回一个 React 元素。

**基本模式**：
```jsx
<DataProvider render={data => <Display data={data} />} />

// 或
<DataProvider>
  {data => <Display data={data} />}
</DataProvider>
```

**与 HOC 的对比**：

| 特性 | HOC | Render Props |
|------|-----|-------------|
| 代码组织 | 包装组件 | 作为 children 或 prop 传递 |
| 灵活性 | 静态组合 | 动态组合 |
| 学习成本 | 较高 | 较低 |
| Props 冲突 | 可能冲突 | 不冲突 |

**经典场景**：
- 订阅外部数据源
- 共享行为逻辑
- 延迟加载

**2.4 自定义 Hooks**

自定义 Hooks 是以 `use` 开头的函数，可以在其中使用其他 Hooks，用于封装和复用有状态的逻辑。

**创建自定义 Hooks 的规则**：
- 名称必须以 `use` 开头
- 可以在内部使用其他 Hooks
- 每个组件使用独立的 Hook 状态

**常见自定义 Hooks**：
- `useLocalStorage`：持久化状态到本地存储
- `useDebounce`：防抖处理
- `useFetch`：数据请求封装
- `usePrevious`：获取上一个值
- `useClickOutside`：检测点击外部

**经典案例：组件通信方案**

```jsx
// Parent.jsx - 父传子
function Parent() {
  const [message, setMessage] = useState('来自父组件的消息');

  return (
    <div>
      <h2>父组件</h2>
      <Child message={message} onUpdate={setMessage} />
    </div>
  );
}

function Child({ message, onUpdate }) {
  return (
    <div>
      <h3>子组件</h3>
      <p>收到消息：{message}</p>
      <button onClick={() => onUpdate('子组件回复的消息')}>
        回复父组件
      </button>
    </div>
  );
}
```

```jsx
// Sibling通信 - 通过父组件中转
function SiblingCommunication() {
  const [messageToB, setMessageToB] = useState('');

  return (
    <div>
      <ComponentA onSendMessage={setMessageToB} />
      <ComponentB message={messageToB} />
    </div>
  );
}

function ComponentA({ onSendMessage }) {
  const [input, setInput] = useState('');
  return (
    <div>
      <h3>组件 A</h3>
      <input
        value={input}
        onChange={(e) => setInput(e.target.value)}
        placeholder="发送给 B..."
      />
      <button onClick={() => onSendMessage(input)}>发送</button>
    </div>
  );
}

function ComponentB({ message }) {
  return (
    <div>
      <h3>组件 B</h3>
      <p>收到消息：{message || '(无)'}</p>
    </div>
  );
}
```

**经典案例：自定义 Hooks - useLocalStorage**

```jsx
// useLocalStorage.js
import { useState, useEffect } from 'react';

function useLocalStorage(key, initialValue) {
  const [storedValue, setStoredValue] = useState(() => {
    try {
      const item = window.localStorage.getItem(key);
      return item ? JSON.parse(item) : initialValue;
    } catch (error) {
      console.error('读取 localStorage 失败:', error);
      return initialValue;
    }
  });

  useEffect(() => {
    try {
      window.localStorage.setItem(key, JSON.stringify(storedValue));
    } catch (error) {
      console.error('写入 localStorage 失败:', error);
    }
  }, [key, storedValue]);

  return [storedValue, setStoredValue];
}

export default useLocalStorage;
```

```jsx
// useDebounce.js
import { useState, useEffect } from 'react';

function useDebounce(value, delay = 500) {
  const [debouncedValue, setDebouncedValue] = useState(value);

  useEffect(() => {
    const timer = setTimeout(() => {
      setDebouncedValue(value);
    }, delay);

    return () => clearTimeout(timer);
  }, [value, delay]);

  return debouncedValue;
}

export default useDebounce;
```

```jsx
// useFetch.js
import { useState, useEffect } from 'react';

function useFetch(url) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    const controller = new AbortController();

    async function fetchData() {
      try {
        setLoading(true);
        const response = await fetch(url, {
          signal: controller.signal
        });

        if (!response.ok) {
          throw new Error(`HTTP 错误：${response.status}`);
        }

        const json = await response.json();
        setData(json);
        setError(null);
      } catch (err) {
        if (err.name !== 'AbortError') {
          setError(err.message);
        }
      } finally {
        setLoading(false);
      }
    }

    fetchData();

    return () => controller.abort();
  }, [url]);

  return { data, loading, error };
}

export default useFetch;
```

**经典案例：高阶组件（HOC 思想）**

```jsx
// withLoading.js - HOC 示例
function withLoading(Component) {
  return function WithLoadingComponent({ isLoading, ...props }) {
    if (isLoading) {
      return <div>加载中...</div>;
    }
    return <Component {...props} />;
  };
}

// 使用
const UserListWithLoading = withLoading(UserList);

function App() {
  const [loading, setLoading] = useState(true);

  return (
    <UserListWithLoading
      isLoading={loading}
      users={users}
    />
  );
}
```

**经典案例：Render Props**

```jsx
// MouseTracker.jsx - Render Props 模式
function MouseTracker({ render }) {
  const [position, setPosition] = useState({ x: 0, y: 0 });

  const handleMouseMove = (e) => {
    setPosition({ x: e.clientX, y: e.clientY });
  };

  return (
    <div onMouseMove={handleMouseMove}>
      {render(position)}
    </div>
  );
}

// 使用
function App() {
  return (
    <MouseTracker
      render={({ x, y }) => (
        <div>
          鼠标位置：({x}, {y})
        </div>
      )}
    />
  );
}
```

---

### 3. 路由管理（React Router）

- [ ] 路由配置、嵌套路由
- [ ] 动态路由
- [ ] 编程式导航
- [ ] 路由守卫
- [ ] 路由懒加载
- [ ] 404 页面、路由参数获取

**详细概念：**

**3.1 路由配置与嵌套路由**

React Router 使用声明式的方式定义路由，通过 `<Routes>` 和 `<Route>` 组件来配置。

**基础路由配置**：
```jsx
<Routes>
  <Route path="/" element={<Home />} />
  <Route path="/about" element={<About />} />
</Routes>
```

**嵌套路由**：嵌套路由允许在父路由中渲染子路由，使用 `<Outlet>` 组件作为子路由的渲染位置。

```jsx
<Route path="/users" element={<UsersLayout />}>
  <Route index element={<UserList />} />
  <Route path=":id" element={<UserDetail />} />
</Route>
```

**3.2 动态路由**

动态路由允许 URL 包含可变部分，使用冒号 `:` 标记参数。

**路由参数**：
```jsx
<Route path="/users/:id" element={<UserProfile />} />

// 获取参数
const { id } = useParams();
```

**查询参数**：
```jsx
// URL: /search?keyword=react&page=1
const [searchParams, setSearchParams] = useSearchParams();

const keyword = searchParams.get('keyword');
const page = searchParams.get('page');
```

**3.3 编程式导航**

编程式导航允许在代码中控制页面跳转，不依赖用户点击链接。

**useNavigate Hook**：
```jsx
const navigate = useNavigate();

// 跳转到指定路径
navigate('/dashboard');

// 带状态跳转
navigate('/user', { state: { fromHome: true } });

// 替换当前历史记录
navigate('/home', { replace: true });

// 前进或后退
navigate(1);  // 前进
navigate(-1); // 后退
```

**3.4 路由守卫**

路由守卫用于保护需要权限的页面，未授权用户重定向到登录页。

```jsx
function ProtectedRoute({ children, isAuthenticated }) {
  if (!isAuthenticated) {
    return <Navigate to="/login" replace />;
  }
  return children;
}

// 使用
<Route
  path="/dashboard"
  element={
    <ProtectedRoute isAuthenticated={isAuth}>
      <Dashboard />
    </ProtectedRoute>
  }
/>
```

**3.5 路由懒加载**

路由懒加载使用 React.lazy 和 Suspense 实现代码分割，减少首屏加载时间。

```jsx
const Dashboard = lazy(() => import('./pages/Dashboard'));

<Suspense fallback={<Loading />}>
  <Routes>
    <Route path="/" element={<Dashboard />} />
  </Routes>
</Suspense>
```

**3.6 404 页面与通配路由**

使用 `*` 作为通配符匹配所有未匹配的路由，实现 404 页面。

```jsx
<Route path="*" element={<NotFound />} />
```

**经典案例：基础路由配置**

```jsx
// App.jsx
import { BrowserRouter, Routes, Route, Link } from 'react-router-dom';

function Home() {
  return <h1>首页</h1>;
}

function About() {
  return <h1>关于我们</h1>;
}

function App() {
  return (
    <BrowserRouter>
      <nav>
        <Link to="/">首页</Link> | <Link to="/about">关于</Link>
      </nav>

      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/about" element={<About />} />
      </Routes>
    </BrowserRouter>
  );
}
```

**经典案例：嵌套路由**

```jsx
// Layout.jsx
import { Outlet, Link } from 'react-router-dom';

function Layout() {
  return (
    <div>
      <header>
        <nav>
          <Link to="/">首页</Link> | <Link to="/users">用户</Link>
        </nav>
      </header>

      <main>
        <Outlet />
      </main>

      <footer>页脚</footer>
    </div>
  );
}

function App() {
  return (
    <BrowserRouter>
      <Routes>
        <Route path="/" element={<Layout />}>
          <Route index element={<Home />} />
          <Route path="users" element={<UserLayout />}>
            <Route index element={<UserList />} />
            <Route path=":id" element={<UserProfile />} />
          </Route>
        </Route>
        <Route path="*" element={<NotFound />} />
      </Routes>
    </BrowserRouter>
  );
}
```

**经典案例：动态路由与参数获取**

```jsx
// UserProfile.jsx
import { useParams, useSearchParams } from 'react-router-dom';

function UserProfile() {
  const { id } = useParams();
  const [searchParams, setSearchParams] = useSearchParams();

  const tab = searchParams.get('tab') || 'info';

  return (
    <div>
      <h1>用户 ID：{id}</h1>
      <p>当前标签：{tab}</p>

      <nav>
        <button onClick={() => setSearchParams({ tab: 'info' })}>基本信息</button>
        <button onClick={() => setSearchParams({ tab: 'posts' })}>帖子</button>
        <button onClick={() => setSearchParams({ tab: 'settings' })}>设置</button>
      </nav>
    </div>
  );
}
```

**经典案例：编程式导航**

```jsx
// Login.jsx
import { useNavigate, useLocation } from 'react-router-dom';

function Login() {
  const navigate = useNavigate();
  const location = useLocation();

  const from = location.state?.from || '/';

  const handleLogin = () => {
    localStorage.setItem('token', 'fake-token');
    navigate(from, { replace: true });
  };

  return (
    <div>
      <h1>登录</h1>
      <button onClick={handleLogin}>登录并跳转</button>
    </div>
  );
}
```

**经典案例：路由守卫**

```jsx
// ProtectedRoute.jsx
import { Navigate, useLocation } from 'react-router-dom';

function ProtectedRoute({ children, isAuthenticated }) {
  const location = useLocation();

  if (!isAuthenticated) {
    return (
      <Navigate
        to="/login"
        state={{ from: location }}
        replace
      />
    );
  }

  return children;
}

// 使用
function App() {
  const isAuthenticated = !!localStorage.getItem('token');

  return (
    <Routes>
      <Route path="/login" element={<Login />} />
      <Route
        path="/dashboard"
        element={
          <ProtectedRoute isAuthenticated={isAuthenticated}>
            <Dashboard />
          </ProtectedRoute>
        }
      />
    </Routes>
  );
}
```

---

### 4. 性能优化基础

- [ ] 避免不必要渲染（React.memo、useMemo、useCallback）
- [ ] 代码分割、懒加载（React.lazy + Suspense）
- [ ] 列表渲染优化（虚拟列表初步了解）

**详细概念：**

**4.1 React.memo 与性能优化**

React.memo 是一个高阶组件，用于包装函数组件，缓存其渲染结果。

**基本用法**：
```jsx
const MemoizedComponent = memo(function MyComponent(props) {
  return <div>{props.name}</div>;
});
```

**工作原理**：
- React.memo 会对组件的 props 进行浅比较（shallow compare）
- 如果 props 没有变化，跳过渲染，直接使用缓存结果
- 如果 props 发生变化，才会重新渲染

**自定义比较函数**：
```jsx
const MemoizedComponent = memo(
  function MyComponent({ data, onClick }) {
    return <div onClick={onClick}>{data.name}</div>;
  },
  (prevProps, nextProps) => {
    // 返回 true 表示 props 相等，不重新渲染
    return prevProps.data.id === nextProps.data.id;
  }
);
```

**适用场景**：
- 组件频繁渲染，但输出相同的概率高
- 组件接收大量 props，更新频繁
- 组件是纯展示组件，不依赖外部状态

**4.2 useMemo 与 useCallback**

这两个 Hook 都用于避免不必要的计算或渲染：

**useMemo 缓存计算结果**：
```jsx
const sortedList = useMemo(() => {
  return items.slice().sort((a, b) => a.name.localeCompare(b.name));
}, [items]);
```

**useCallback 缓存函数引用**：
```jsx
const handleClick = useCallback(() => {
  doSomething(id);
}, [id]);
```

**何时使用**：
- 传递给子组件的函数使用 useCallback
- 派生状态（计算属性）使用 useMemo
- 不要过度使用，会增加代码复杂度

**4.3 代码分割与懒加载**

代码分割是将应用拆分成多个小块，按需加载，减少首屏加载时间。

**React.lazy + Suspense**：
```jsx
const OtherComponent = lazy(() => import('./OtherComponent'));

function MyComponent() {
  return (
    <Suspense fallback={<Loading />}>
      <OtherComponent />
    </Suspense>
  );
}
```

**路由级别的代码分割**：
```jsx
const Dashboard = lazy(() => import('./pages/Dashboard'));
const Settings = lazy(() => import('./pages/Settings'));

<Suspense fallback={<Loading />}>
  <Routes>
    <Route path="/dashboard" element={<Dashboard />} />
    <Route path="/settings" element={<Settings />} />
  </Routes>
</Suspense>
```

**4.4 虚拟列表**

虚拟列表是一种优化大量数据渲染的技术，只渲染可视区域内的元素。

**核心原理**：
- 计算可视区域能显示多少个元素
- 只渲染这些可见元素
- 使用绝对定位或 transform 调整位置
- 滚动时动态计算并渲染新的可见元素

**适用场景**：
- 列表项数量巨大（成千上万）
- 每个列表项高度固定或可预估
- 需要保持列表滚动流畅

**经典案例：React.memo 避免重复渲染**

```jsx
// MemoizedComponent.jsx
import { memo, useState } from 'react';

const ExpensiveList = memo(function ExpensiveList({ items }) {
  console.log('ExpensiveList 渲染');

  return (
    <ul>
      {items.map(item => (
        <li key={item.id}>{item.name}</li>
      ))}
    </ul>
  );
});

function Parent() {
  const [count, setCount] = useState(0);
  const [name, setName] = useState('');

  const items = [
    { id: 1, name: '苹果' },
    { id: 2, name: '香蕉' },
    { id: 3, name: '橙子' },
  ];

  return (
    <div>
      <ExpensiveList items={items} />
      <p>计数：{count}</p>
      <button onClick={() => setCount(c => c + 1)}>增加</button>
      <input
        value={name}
        onChange={(e) => setName(e.target.value)}
        placeholder="输入名字"
      />
    </div>
  );
}
```

**经典案例：懒加载**

```jsx
// LazyLoading.jsx
import { lazy, Suspense } from 'react';
import { BrowserRouter, Routes, Route } from 'react-router-dom';

const Home = lazy(() => import('./pages/Home'));
const About = lazy(() => import('./pages/About'));
const Dashboard = lazy(() => import('./pages/Dashboard'));

function Loading() {
  return <div>加载中...</div>;
}

function App() {
  return (
    <BrowserRouter>
      <Suspense fallback={<Loading />}>
        <Routes>
          <Route path="/" element={<Home />} />
          <Route path="/about" element={<About />} />
          <Route path="/dashboard" element={<Dashboard />} />
        </Routes>
      </Suspense>
    </BrowserRouter>
  );
}
```

**经典案例：虚拟滚动列表**

```jsx
// VirtualList.jsx
import { useState, useRef, useCallback } from 'react';

function VirtualList({ items, height = 400, itemHeight = 50 }) {
  const [scrollTop, setScrollTop] = useState(0);
  const containerRef = useRef(null);

  const visibleCount = Math.ceil(height / itemHeight);
  const startIndex = Math.floor(scrollTop / itemHeight);
  const endIndex = Math.min(startIndex + visibleCount + 2, items.length);

  const visibleItems = [];
  for (let i = startIndex; i < endIndex; i++) {
    visibleItems.push(
      <div
        key={items[i].id}
        style={{
          position: 'absolute',
          top: `${i * itemHeight}px`,
          height: `${itemHeight}px`,
          width: '100%',
          borderBottom: '1px solid #eee',
          display: 'flex',
          alignItems: 'center',
          paddingLeft: 10
        }}
      >
        {items[i].name}
      </div>
    );
  }

  return (
    <div
      ref={containerRef}
      style={{ height, overflow: 'auto', position: 'relative' }}
      onScroll={(e) => setScrollTop(e.target.scrollTop)}
    >
      <div style={{ height: items.length * itemHeight, position: 'relative' }}>
        {visibleItems}
      </div>
    </div>
  );
}

function App() {
  const items = Array.from({ length: 1000 }, (_, i) => ({
    id: i,
    name: `项目 ${i + 1}`
  }));

  return <VirtualList items={items} />;
}
```

---

## 了解即可

- 高阶组件的具体实现方式
- Render Props 的具体实现方式
- 虚拟列表的原理

## 阶段成果

能开发以下应用：
- 多页面应用
- 复杂表单
- 带权限的页面
- 优化渲染性能的项目

## Hooks 对比速查

| Hook | 用途 | 何时使用 |
|------|------|----------|
| useState | 状态管理 | 简单状态、组件内部使用 |
| useReducer | 状态管理 | 复杂状态逻辑 |
| useRef | DOM/值引用 | 获取 DOM、存储跨渲染周期的值 |
| useMemo | 性能优化 | 缓存计算结果 |
| useCallback | 性能优化 | 缓存函数，避免重复渲染 |
| useContext | 跨组件传值 | 避免多层 props 传递 |

## 路由基础对照

| 功能 | 实现方式 |
|------|----------|
| 声明式导航 | `<Link to="/path">` |
| 编程式导航 | `useNavigate()` 或 `history.push()` |
| 路由传参 | `/path/:id` 或 `?id=1` |
| 嵌套路由 | 父路由配置 children |

## 自定义 Hooks 模式

常见自定义 Hooks：
- `useForm`：表单状态管理
- `useFetch`：数据请求
- `useInterval`：定时器
- `useDebounce`：防抖
- `useLocalStorage`：本地存储

## 下一阶段预告

完成进阶基础后，将进入 [React 生态与工程化](./stage-3-ecosystem.md)，你将学习：
- 状态管理（Redux Toolkit、Zustand）
- 样式方案（Tailwind CSS、组件库）
- 网络请求封装
- TypeScript + React
- 工程化与规范
