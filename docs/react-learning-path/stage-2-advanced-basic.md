# 阶段二：React 进阶基础（核心拔高）

**目标**：解决复杂场景问题、优化代码、掌握高频 Hooks

## 必须掌握的技能

### 1. Hooks 进阶

- [ ] `useRef`：获取 DOM、存储可变值、跨渲染周期存数据
- [ ] `useMemo`：缓存计算结果，优化性能
- [ ] `useCallback`：缓存函数，避免子组件重复渲染
- [ ] `useContext` + `createContext`：跨组件传值（替代多层 Props）
- [ ] `useReducer`：复杂状态逻辑管理（类似 Redux 精简版）

### 2. 组件进阶

- [ ] 组件通信：父子/兄弟/跨层级传值全方案
- [ ] 高阶组件（HOC）- 了解思想
- [ ] Render Props - 了解思想
- [ ] 自定义 Hooks：封装复用逻辑（表单、请求、定时器等）

### 3. 路由管理（React Router）

- [ ] 路由配置、嵌套路由
- [ ] 动态路由
- [ ] 编程式导航
- [ ] 路由守卫
- [ ] 路由懒加载
- [ ] 404 页面、路由参数获取

### 4. 性能优化基础

- [ ] 避免不必要渲染（React.memo、useMemo、useCallback）
- [ ] 代码分割、懒加载（React.lazy + Suspense）
- [ ] 列表渲染优化（虚拟列表初步了解）

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

## 经典学习案例

### 案例 1：useReducer 复杂状态管理

```jsx
import { useReducer } from 'react';

const initialState = { count: 0, step: 1 };

function reducer(state, action) {
  switch (action.type) {
    case 'increment':
      return { ...state, count: state.count + state.step };
    case 'decrement':
      return { ...state, count: state.count - state.step };
    case 'setStep':
      return { ...state, step: action.payload };
    case 'reset':
      return initialState;
    default:
      return state;
  }
}

function Counter() {
  const [state, dispatch] = useReducer(reducer, initialState);

  return (
    <div>
      <p>计数：{state.count}（步长：{state.step}）</p>
      <button onClick={() => dispatch({ type: 'increment' })}>+1</button>
      <button onClick={() => dispatch({ type: 'decrement' })}>-1</button>
      <button onClick={() => dispatch({ type: 'reset' })}>重置</button>
      <hr />
      <input
        type="number"
        value={state.step}
        onChange={(e) => dispatch({
          type: 'setStep',
          payload: Number(e.target.value)
        })}
      />
    </div>
  );
}
```

### 案例 2：useContext 跨组件传值

```jsx
import { createContext, useContext, useState } from 'react';

const ThemeContext = createContext();

function ThemeProvider({ children }) {
  const [theme, setTheme] = useState('light');

  const toggleTheme = () => {
    setTheme(prev => prev === 'light' ? 'dark' : 'light');
  };

  return (
    <ThemeContext.Provider value={{ theme, toggleTheme }}>
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

function Header() {
  const { theme, toggleTheme } = useTheme();

  return (
    <header style={{
      background: theme === 'dark' ? '#333' : '#fff',
      color: theme === 'dark' ? '#fff' : '#333'
    }}>
      <button onClick={toggleTheme}>切换主题</button>
    </header>
  );
}

function Content() {
  const { theme } = useTheme();

  return (
    <div style={{
      background: theme === 'dark' ? '#222' : '#f5f5f5',
      color: theme === 'dark' ? '#fff' : '#333',
      padding: 20
    }}>
      <p>当前主题：{theme}</p>
    </div>
  );
}

function App() {
  return (
    <ThemeProvider>
      <Header />
      <Content />
    </ThemeProvider>
  );
}
```

### 案例 3：自定义 Hooks - useLocalStorage

```jsx
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

function App() {
  const [name, setName] = useLocalStorage('name', '');

  return (
    <div>
      <input
        value={name}
        onChange={(e) => setName(e.target.value)}
        placeholder="输入你的名字"
      />
      <p>你好，{name || '陌生人'}！刷新页面后名字会保留。</p>
    </div>
  );
}
```

### 案例 4：React Router 路由配置

```jsx
import { BrowserRouter, Routes, Route, Link, useParams, Navigate } from 'react-router-dom';

function Home() {
  return <h1>首页</h1>;
}

function UserList() {
  const users = [
    { id: 1, name: '张三' },
    { id: 2, name: '李四' },
    { id: 3, name: '王五' },
  ];

  return (
    <div>
      <h1>用户列表</h1>
      <ul>
        {users.map(user => (
          <li key={user.id}>
            <Link to={`/users/${user.id}`}>{user.name}</Link>
          </li>
        ))}
      </ul>
    </div>
  );
}

function UserProfile() {
  const { id } = useParams();

  return (
    <div>
      <h1>用户详情</h1>
      <p>用户ID：{id}</p>
      <Link to="/users">返回列表</Link>
    </div>
  );
}

function NotFound() {
  return <h1>404 - 页面不存在</h1>;
}

function ProtectedRoute({ children }) {
  const isAuthenticated = false;

  if (!isAuthenticated) {
    return <Navigate to="/login" replace />;
  }

  return children;
}

function App() {
  return (
    <BrowserRouter>
      <nav>
        <Link to="/">首页</Link> | <Link to="/users">用户</Link>
      </nav>

      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/users" element={<UserList />} />
        <Route path="/users/:id" element={<UserProfile />} />
        <Route path="*" element={<NotFound />} />
      </Routes>
    </BrowserRouter>
  );
}
```

### 案例 5：useMemo 与 useCallback 性能优化

```jsx
import { useState, useMemo, useCallback, memo } from 'react';

const ChildComponent = memo(function ChildComponent({ onClick, data }) {
  console.log('ChildComponent 渲染');
  return (
    <div>
      <p>数据：{data}</p>
      <button onClick={onClick}>点击</button>
    </div>
  );
});

function ParentComponent() {
  const [count, setCount] = useState(0);
  const [name, setName] = useState('');

  const expensiveCalc = useMemo(() => {
    console.log('计算中...');
    return Array.from({ length: 1000 }, (_, i) => i * 2);
  }, []);

  const handleClick = useCallback(() => {
    console.log('按钮点击');
  }, []);

  return (
    <div>
      <p>计数：{count}</p>
      <button onClick={() => setCount(c => c + 1)}>增加</button>

      <input
        value={name}
        onChange={(e) => setName(e.target.value)}
        placeholder="输入名字"
      />

      <ChildComponent onClick={handleClick} data={expensiveCalc.length} />
    </div>
  );
}
```

### 案例 6：自定义 Hooks - useFetch

```jsx
import { useState, useEffect } from 'react';

function useFetch(url, options = {}) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    const controller = new AbortController();

    async function fetchData() {
      try {
        setLoading(true);
        const response = await fetch(url, {
          ...options,
          signal: controller.signal,
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

function UserList() {
  const { data: users, loading, error } = useFetch('/api/users');

  if (loading) return <div>加载中...</div>;
  if (error) return <div>错误：{error}</div>;

  return (
    <ul>
      {users?.map(user => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  );
}
```

## 下一阶段预告

完成进阶基础后，将进入 [React 生态与工程化](./stage-3-ecosystem.md)，你将学习：
- 状态管理（Redux Toolkit、Zustand）
- 样式方案（Tailwind CSS、组件库）
- 网络请求封装
- TypeScript + React
- 工程化与规范
