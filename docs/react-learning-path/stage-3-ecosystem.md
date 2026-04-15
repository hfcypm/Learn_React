# 阶段三：React 生态与工程化（中级开发）

**目标**：适配企业开发流程、掌握主流工具链

## 必须掌握的技能

### 1. 状态管理（进阶）

- [ ] Redux Toolkit（现代 Redux，企业首选）
- [ ] Zustand/Jotai（轻量级状态库，简单易用）
- [ ] 状态持久化
- [ ] 异步状态处理

### 2. 样式方案

- [ ] CSS Modules
- [ ] Styled Components
- [ ] Tailwind CSS（企业主流）
- [ ] Ant Design / Material UI 组件库

### 3. 网络请求

- [ ] Axios 封装：请求拦截、响应拦截、错误统一处理
- [ ] React Query / SWR：数据请求、缓存、自动刷新（进阶必备）

### 4. TypeScript + React（必学）

- [ ] 为 Props/State/Hooks 定义类型
- [ ] 泛型组件
- [ ] 类型推导
- [ ] 避免 any 代码
- [ ] TS 常用类型：interface、type、泛型

### 5. 工程化与规范

- [ ] ESLint + Prettier 代码规范
- [ ] Git 版本管理、Commitlint 提交规范
- [ ] 环境变量配置
- [ ] 打包构建优化

## 了解即可

- MobX 响应式状态管理
- Emotion CSS-in-JS 方案
- React Query 的高级特性

## 阶段成果

能开发以下应用：
- 企业级中后台系统
- 带数据缓存的应用
- TypeScript 强类型规范项目

## 状态管理方案对比

| 方案 | 适用场景 | 学习成本 |
|------|----------|----------|
| Redux Toolkit | 大型复杂应用 | 中等 |
| Zustand | 中小型应用 | 低 |
| Jotai | 原子化状态 | 低 |
| Context + useReducer | 简单跨组件传值 | 低 |

## TypeScript 核心类型

```typescript
// Props 类型定义
interface Props {
  name: string;
  age?: number;
  onClick: (id: string) => void;
}

// 泛型组件
function GenericList<T>({ items, renderItem }: {
  items: T[];
  renderItem: (item: T) => React.ReactNode;
})

// 事件类型
onChange: (e: React.ChangeEvent<HTMLInputElement>) => void
onClick: (e: React.MouseEvent<HTMLButtonElement>) => void

// 状态类型
const [state, setState] = useState<T | null>(null);
```

## 经典学习案例

### 案例 1：Redux Toolkit 完整使用

```typescript
// store/userSlice.ts
import { createSlice, createAsyncThunk, PayloadAction } from '@reduxjs/toolkit';

interface User {
  id: string;
  name: string;
  email: string;
}

interface UserState {
  users: User[];
  selectedUser: User | null;
  loading: boolean;
  error: string | null;
}

const initialState: UserState = {
  users: [],
  selectedUser: null,
  loading: false,
  error: null,
};

export const fetchUsers = createAsyncThunk('users/fetchUsers', async () => {
  const response = await fetch('/api/users');
  if (!response.ok) throw new Error('获取用户失败');
  return response.json();
});

const userSlice = createSlice({
  name: 'users',
  initialState,
  reducers: {
    addUser: (state, action: PayloadAction<User>) => {
      state.users.push(action.payload);
    },
    removeUser: (state, action: PayloadAction<string>) => {
      state.users = state.users.filter(user => user.id !== action.payload);
    },
    setSelectedUser: (state, action: PayloadAction<User | null>) => {
      state.selectedUser = action.payload;
    },
  },
  extraReducers: (builder) => {
    builder
      .addCase(fetchUsers.pending, (state) => {
        state.loading = true;
        state.error = null;
      })
      .addCase(fetchUsers.fulfilled, (state, action) => {
        state.loading = false;
        state.users = action.payload;
      })
      .addCase(fetchUsers.rejected, (state, action) => {
        state.loading = false;
        state.error = action.error.message || '获取用户失败';
      });
  },
});

export const { addUser, removeUser, setSelectedUser } = userSlice.actions;
export default userSlice.reducer;
```

```typescript
// store/index.ts
import { configureStore } from '@reduxjs/toolkit';
import userReducer from './userSlice';

export const store = configureStore({
  reducer: {
    users: userReducer,
  },
});

export type RootState = ReturnType<typeof store.getState>;
export type AppDispatch = typeof store.dispatch;
```

```tsx
// components/UserList.tsx
import { useSelector, useDispatch } from 'react-redux';
import type { RootState, AppDispatch } from '../store';
import { fetchUsers, removeUser } from '../store/userSlice';

function UserList() {
  const dispatch = useDispatch<AppDispatch>();
  const { users, loading, error } = useSelector((state: RootState) => state.users);

  if (loading) return <div>加载中...</div>;
  if (error) return <div>错误：{error}</div>;

  return (
    <div>
      <button onClick={() => dispatch(fetchUsers())}>刷新用户</button>
      <ul>
        {users.map(user => (
          <li key={user.id}>
            {user.name} - {user.email}
            <button onClick={() => dispatch(removeUser(user.id))}>删除</button>
          </li>
        ))}
      </ul>
    </div>
  );
}
```

### 案例 2：Zustand 状态管理

```typescript
// store/useStore.ts
import { create } from 'zustand';
import { persist } from 'zustand/middleware';

interface CartItem {
  id: string;
  name: string;
  price: number;
  quantity: number;
}

interface CartStore {
  items: CartItem[];
  addItem: (item: CartItem) => void;
  removeItem: (id: string) => void;
  updateQuantity: (id: string, quantity: number) => void;
  clearCart: () => void;
  total: () => number;
}

export const useCartStore = create<CartStore>()(
  persist(
    (set, get) => ({
      items: [],

      addItem: (item) => {
        const existing = get().items.find(i => i.id === item.id);
        if (existing) {
          set((state) => ({
            items: state.items.map(i =>
              i.id === item.id
                ? { ...i, quantity: i.quantity + 1 }
                : i
            ),
          }));
        } else {
          set((state) => ({ items: [...state.items, { ...item, quantity: 1 }] }));
        }
      },

      removeItem: (id) => {
        set((state) => ({ items: state.items.filter(i => i.id !== id) }));
      },

      updateQuantity: (id, quantity) => {
        if (quantity <= 0) {
          get().removeItem(id);
          return;
        }
        set((state) => ({
          items: state.items.map(i =>
            i.id === id ? { ...i, quantity } : i
          ),
        }));
      },

      clearCart: () => set({ items: [] }),

      total: () => {
        return get().items.reduce((sum, item) => sum + item.price * item.quantity, 0);
      },
    }),
    { name: 'cart-storage' }
  )
);
```

```tsx
// components/Cart.tsx
import { useCartStore } from '../store/useStore';

function Cart() {
  const { items, addItem, removeItem, updateQuantity, total, clearCart } = useCartStore();

  return (
    <div>
      <h2>购物车</h2>
      {items.length === 0 ? (
        <p>购物车为空</p>
      ) : (
        <>
          <ul>
            {items.map(item => (
              <li key={item.id}>
                {item.name} - ¥{item.price} x {item.quantity}
                <button onClick={() => updateQuantity(item.id, item.quantity + 1)}>+</button>
                <button onClick={() => updateQuantity(item.id, item.quantity - 1)}>-</button>
                <button onClick={() => removeItem(item.id)}>删除</button>
              </li>
            ))}
          </ul>
          <p>总计：¥{total()}</p>
          <button onClick={clearCart}>清空购物车</button>
        </>
      )}
    </div>
  );
}
```

### 案例 3：TypeScript + React 泛型组件

```tsx
import { useState } from 'react';

interface Column<T> {
  key: keyof T;
  title: string;
  render?: (value: T[keyof T], record: T) => React.ReactNode;
}

interface DataTableProps<T> {
  data: T[];
  columns: Column<T>[];
  rowKey: keyof T;
  onRowClick?: (record: T) => void;
}

function DataTable<T extends Record<string, unknown>>({
  data,
  columns,
  rowKey,
  onRowClick,
}: DataTableProps<T>) {
  return (
    <table>
      <thead>
        <tr>
          {columns.map(col => (
            <th key={String(col.key)}>{col.title}</th>
          ))}
        </tr>
      </thead>
      <tbody>
        {data.map((record, index) => (
          <tr
            key={String(record[rowKey]) || index}
            onClick={() => onRowClick?.(record)}
            style={{ cursor: onRowClick ? 'pointer' : 'default' }}
          >
            {columns.map(col => (
              <td key={String(col.key)}>
                {col.render
                  ? col.render(record[col.key], record)
                  : String(record[col.key])}
              </td>
            ))}
          </tr>
        ))}
      </tbody>
    </table>
  );
}

// 使用示例
interface User {
  id: string;
  name: string;
  email: string;
  status: 'active' | 'inactive';
}

const columns: Column<User>[] = [
  { key: 'name', title: '姓名' },
  { key: 'email', title: '邮箱' },
  {
    key: 'status',
    title: '状态',
    render: (value) => (
      <span style={{ color: value === 'active' ? 'green' : 'gray' }}>
        {value === 'active' ? '活跃' : '不活跃'}
      </span>
    ),
  },
];

function App() {
  const [users] = useState<User[]>([
    { id: '1', name: '张三', email: 'zhang@example.com', status: 'active' },
    { id: '2', name: '李四', email: 'li@example.com', status: 'inactive' },
  ]);

  return (
    <DataTable
      data={users}
      columns={columns}
      rowKey="id"
      onRowClick={(user) => console.log('点击了', user.name)}
    />
  );
}
```

### 案例 4：Axios 封装

```typescript
// utils/api.ts
import axios, { AxiosInstance, AxiosError, InternalAxiosRequestConfig, AxiosResponse } from 'axios';

interface ApiResponse<T> {
  code: number;
  data: T;
  message: string;
}

interface RequestConfig extends InternalAxiosRequestConfig {
  _retry?: boolean;
}

const api: AxiosInstance = axios.create({
  baseURL: import.meta.env.VITE_API_BASE_URL || '/api',
  timeout: 10000,
  headers: {
    'Content-Type': 'application/json',
  },
});

api.interceptors.request.use(
  (config) => {
    const token = localStorage.getItem('token');
    if (token && config.headers) {
      config.headers.Authorization = `Bearer ${token}`;
    }
    return config;
  },
  (error) => Promise.reject(error)
);

api.interceptors.response.use(
  (response: AxiosResponse<ApiResponse<unknown>>) => {
    if (response.data.code !== 200) {
      return Promise.reject(new Error(response.data.message || '请求失败'));
    }
    return response;
  },
  async (error: AxiosError<ApiResponse<unknown>>) => {
    const config = error.config as RequestConfig;

    if (error.response?.status === 401 && !config._retry) {
      config._retry = true;

      try {
        const refreshToken = localStorage.getItem('refreshToken');
        const response = await axios.post('/api/auth/refresh', { refreshToken });

        localStorage.setItem('token', response.data.data.accessToken);
        if (config.headers) {
          config.headers.Authorization = `Bearer ${response.data.data.accessToken}`;
        }

        return api(config);
      } catch {
        localStorage.removeItem('token');
        localStorage.removeItem('refreshToken');
        window.location.href = '/login';
      }
    }

    const message = error.response?.data?.message || error.message || '网络错误';
    console.error('API Error:', message);

    return Promise.reject(error);
  }
);

export const request = {
  get: <T>(url: string, params?: Record<string, unknown>) =>
    api.get<ApiResponse<T>>(url, { params }).then(res => res.data.data),

  post: <T>(url: string, data?: unknown) =>
    api.post<ApiResponse<T>>(url, data).then(res => res.data.data),

  put: <T>(url: string, data?: unknown) =>
    api.put<ApiResponse<T>>(url, data).then(res => res.data.data),

  delete: <T>(url: string, params?: Record<string, unknown>) =>
    api.delete<ApiResponse<T>>(url, { params }).then(res => res.data.data),
};

export default api;
```

```tsx
// services/userService.ts
import { request } from '../utils/api';

interface User {
  id: string;
  name: string;
  email: string;
}

export const userService = {
  getUsers: () => request.get<User[]>('/users'),

  getUserById: (id: string) => request.get<User>(`/users/${id}`),

  createUser: (data: Omit<User, 'id'>) =>
    request.post<User>('/users', data),

  updateUser: (id: string, data: Partial<User>) =>
    request.put<User>(`/users/${id}`, data),

  deleteUser: (id: string) =>
    request.delete<void>(`/users/${id}`),
};
```

### 案例 5：React Query 数据请求与缓存

```tsx
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';
import { request } from '../utils/api';

interface Todo {
  id: string;
  title: string;
  completed: boolean;
}

async function fetchTodos(): Promise<Todo[]> {
  return request.get<Todo[]>('/todos');
}

async function createTodo(title: string): Promise<Todo> {
  return request.post<Todo>('/todos', { title });
}

async function toggleTodo(id: string, completed: boolean): Promise<Todo> {
  return request.put<Todo>(`/todos/${id}`, { completed });
}

function TodoList() {
  const queryClient = useQueryClient();

  const { data: todos, isLoading, error } = useQuery({
    queryKey: ['todos'],
    queryFn: fetchTodos,
    staleTime: 5 * 60 * 1000,
    refetchOnWindowFocus: false,
  });

  const createMutation = useMutation({
    mutationFn: createTodo,
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['todos'] });
    },
  });

  const toggleMutation = useMutation({
    mutationFn: ({ id, completed }: { id: string; completed: boolean }) =>
      toggleTodo(id, completed),
    onMutate: async ({ id, completed }) => {
      await queryClient.cancelQueries({ queryKey: ['todos'] });

      const previousTodos = queryClient.getQueryData<Todo[]>(['todos']);

      queryClient.setQueryData<Todo[]>(['todos'], (old) =>
        old?.map(todo =>
          todo.id === id ? { ...todo, completed } : todo
        )
      );

      return { previousTodos };
    },
    onError: (err, variables, context) => {
      if (context?.previousTodos) {
        queryClient.setQueryData(['todos'], context.previousTodos);
      }
    },
  });

  if (isLoading) return <div>加载中...</div>;
  if (error) return <div>错误：{error.message}</div>;

  return (
    <div>
      <form
        onSubmit={(e) => {
          e.preventDefault();
          const input = e.target as HTMLFormElement;
          const title = (input.elements.namedItem('title') as HTMLInputElement).value;
          if (title.trim()) {
            createMutation.mutate(title);
            input.reset();
          }
        }}
      >
        <input name="title" placeholder="添加待办..." />
        <button type="submit" disabled={createMutation.isPending}>
          {createMutation.isPending ? '添加中...' : '添加'}
        </button>
      </form>

      <ul>
        {todos?.map(todo => (
          <li key={todo.id}>
            <label>
              <input
                type="checkbox"
                checked={todo.completed}
                onChange={() =>
                  toggleMutation.mutate({ id: todo.id, completed: !todo.completed })
                }
              />
              <span style={{
                textDecoration: todo.completed ? 'line-through' : 'none'
              }}>
                {todo.title}
              </span>
            </label>
          </li>
        ))}
      </ul>
    </div>
  );
}
```

## 下一阶段预告

完成生态与工程化后，将进入 [React 高级进阶](./stage-4-advanced.md)，你将学习：
- React 渲染原理、Fiber 架构
- 深度性能优化
- Next.js SSR/SSG
- 测试与部署
- 架构设计
