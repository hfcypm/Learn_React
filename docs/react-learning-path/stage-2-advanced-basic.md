# 阶段二：React 进阶基础（核心拔高）

**目标**：解决复杂场景问题、优化代码、掌握高频 Hooks

## 必须掌握的技能

### 1. Hooks 进阶

- [ ] `useRef`：获取 DOM、存储可变值、跨渲染周期存数据
- [ ] `useMemo`：缓存计算结果，优化性能
- [ ] `useCallback`：缓存函数，避免子组件重复渲染
- [ ] `useContext` + `createContext`：跨组件传值（替代多层 Props）
- [ ] `useReducer`：复杂状态逻辑管理（类似 Redux 精简版）

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
