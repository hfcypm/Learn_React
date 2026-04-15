# 阶段三：React 生态与工程化（中级开发）

**目标**：适配企业开发流程、掌握主流工具链

## 必须掌握的技能

### 1. 状态管理（进阶）

- [ ] Redux Toolkit（现代 Redux，企业首选）
- [ ] Zustand/Jotai（轻量级状态库，简单易用）
- [ ] 状态持久化
- [ ] 异步状态处理

**详细概念：**

**1.1 Redux Toolkit 核心思想**

Redux Toolkit 是 Redux 官方推荐的状态管理方案，它大幅简化了传统 Redux 的使用方式，解决了样板代码过多、配置复杂等痛点。

Redux Toolkit 的核心 API：
- **configureStore**：简化 store 配置，自动组合 reducers，自动添加中间件
- **createSlice**：自动生成 action creators 和 action types，基于 immer 允许"可变"式更新
- **createAsyncThunk**：处理异步逻辑，自动生成 pending/fulfilled/rejected action types
- **createEntityAdapter**：高效管理实体集合，提供标准化的 CRUD reducers

为什么企业首选 Redux Toolkit：
- **极简的配置**：几行代码即可完成复杂的状态管理配置
- **Immer 集成**：可以直接修改 state 对象，无需手动返回新状态
- **RTK Query**：内置的数据请求解决方案，支持缓存、自动刷新等高级特性
- **DevTools 集成**：开箱即用的 Redux DevTools 支持

**1.2 Zustand 轻量方案**

Zustand 是一个极度轻量级的状态管理库，核心只有约 1kb，却提供了完整的状态管理能力。它的 API 设计非常直观，深受 Hooks 开发者喜爱。

Zustand 的核心优势：
- **极简 API**：使用 `create` 函数即可创建 store，无需 Provider 包裹
- **订阅机制灵活**：可以在组件外订阅状态变化
- **中间件支持**：支持 persist、devtools、immer 等中间件扩展
- **TypeScript 友好**：完整的类型推断，几乎不需要手动标注类型

Zustand vs Redux Toolkit：
- Zustand 更轻量，适合中小型项目
- Redux Toolkit 功能更全面，适合大型复杂应用
- 两者都基于 immer，允许"可变"式更新

**1.3 状态持久化**

状态持久化是将内存中的状态保存到存储介质（localStorage、sessionStorage、IndexedDB 等），在页面刷新或重新打开时恢复状态。

常见的持久化策略：
- **全量持久化**：整个 store 整体保存，适合配置类、用户偏好设置
- **增量持久化**：只保存必要字段，减少存储空间和序列化开销
- **混合持久化**：部分状态持久化，部分状态不持久化（如登录状态）

实现方式：
- Zustand/Redux 的 persist 中间件
- 自定义 localStorage/sessionStorage 读写逻辑
- IndexedDB 用于存储大量结构化数据

**1.4 异步状态处理**

异步状态是指从服务器获取的、随时间可能变化的数据。处理异步状态需要考虑：加载中、加载成功、加载失败三种状态。

常见的异步状态管理方案：
- **createAsyncThunk（Redux Toolkit）**：将异步逻辑封装为可复用的 thunk
- **RTK Query**：基于 Redux Toolkit 的数据获取和缓存方案
- **React Query/SWR**：独立于状态管理库的数据获取方案

**经典案例：Redux Toolkit 完整使用**

```typescript
// store/userSlice.ts
import { createSlice, createAsyncThunk, PayloadAction } from '@reduxjs/toolkit';

interface User {
  id: string;
  name: string;
  email: string;
  role: 'admin' | 'user';
}

interface UserState {
  users: User[];
  selectedUser: User | null;
  loading: boolean;
  error: string | null;
  filter: 'all' | 'admin';
}

const initialState: UserState = {
  users: [],
  selectedUser: null,
  loading: false,
  error: null,
  filter: 'all',
};

export const fetchUsers = createAsyncThunk('users/fetchUsers', async () => {
  const response = await fetch('/api/users');
  if (!response.ok) throw new Error('获取用户失败');
  return response.json();
});

export const createUser = createAsyncThunk(
  'users/createUser',
  async (userData: Omit<User, 'id'>) => {
    const response = await fetch('/api/users', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(userData),
    });
    if (!response.ok) throw new Error('创建用户失败');
    return response.json();
  }
);

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
    setFilter: (state, action: PayloadAction<'all' | 'admin'>) => {
      state.filter = action.payload;
    },
    updateUser: (state, action: PayloadAction<User>) => {
      const index = state.users.findIndex(u => u.id === action.payload.id);
      if (index !== -1) {
        state.users[index] = action.payload;
      }
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
      })
      .addCase(createUser.fulfilled, (state, action) => {
        state.users.push(action.payload);
      });
  },
});

export const { addUser, removeUser, setSelectedUser, setFilter, updateUser } = userSlice.actions;
export default userSlice.reducer;
```

```typescript
// store/index.ts
import { configureStore } from '@reduxjs/toolkit';
import userReducer from './userSlice';
import cartReducer from './cartSlice';

export const store = configureStore({
  reducer: {
    users: userReducer,
    cart: cartReducer,
  },
  middleware: (getDefaultMiddleware) =>
    getDefaultMiddleware({
      serializableCheck: false,
    }),
});

export type RootState = ReturnType<typeof store.getState>;
export type AppDispatch = typeof store.dispatch;
```

```typescript
// store/hooks.ts
import { TypedUseSelectorHook, useDispatch, useSelector } from 'react-redux';
import type { RootState, AppDispatch } from './index';

export const useAppDispatch = () => useDispatch<AppDispatch>();
export const useAppSelector: TypedUseSelectorHook<RootState> = useSelector;
```

```tsx
// components/UserList.tsx
import { useAppDispatch, useAppSelector } from '../store/hooks';
import { fetchUsers, removeUser, setFilter } from '../store/userSlice';

function UserList() {
  const dispatch = useAppDispatch();
  const { users, loading, error, filter } = useAppSelector((state) => state.users);

  const filteredUsers = filter === 'admin'
    ? users.filter(user => user.role === 'admin')
    : users;

  const handleFetch = () => {
    dispatch(fetchUsers());
  };

  const handleDelete = (id: string) => {
    dispatch(removeUser(id));
  };

  return (
    <div>
      <h2>用户列表</h2>

      <button onClick={handleFetch} disabled={loading}>
        {loading ? '加载中...' : '刷新用户'}
      </button>

      <select
        value={filter}
        onChange={(e) => dispatch(setFilter(e.target.value as 'all' | 'admin'))}
      >
        <option value="all">全部用户</option>
        <option value="admin">仅管理员</option>
      </select>

      {error && <p style={{ color: 'red' }}>错误：{error}</p>}

      <ul>
        {filteredUsers.map(user => (
          <li key={user.id}>
            <span>{user.name}</span>
            <span>{user.email}</span>
            <span>[{user.role}]</span>
            <button onClick={() => handleDelete(user.id)}>删除</button>
          </li>
        ))}
      </ul>

      {filteredUsers.length === 0 && !loading && (
        <p>暂无用户</p>
      )}
    </div>
  );
}

export default UserList;
```

**经典案例：Zustand 状态管理**

```typescript
// store/useCartStore.ts
import { create } from 'zustand';
import { persist } from 'zustand/middleware';

interface CartItem {
  id: string;
  name: string;
  price: number;
  quantity: number;
}

interface CartState {
  items: CartItem[];
  addItem: (item: Omit<CartItem, 'quantity'>) => void;
  removeItem: (id: string) => void;
  updateQuantity: (id: string, quantity: number) => void;
  clearCart: () => void;
  getTotal: () => number;
}

export const useCartStore = create<CartState>()(
  persist(
    (set, get) => ({
      items: [],

      addItem: (item) => {
        const existing = get().items.find(i => i.id === item.id);
        if (existing) {
          set((state) => ({
            items: state.items.map(i =>
              i.id === item.id ? { ...i, quantity: i.quantity + 1 } : i
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

      getTotal: () => {
        return get().items.reduce((sum, item) => sum + item.price * item.quantity, 0);
      },
    }),
    { name: 'cart-storage' }
  )
);
```

```tsx
// components/Cart.tsx
import { useCartStore } from '../store/useCartStore';

function Cart() {
  const { items, addItem, removeItem, updateQuantity, clearCart, getTotal } = useCartStore();

  const products = [
    { id: '1', name: '苹果', price: 5 },
    { id: '2', name: '香蕉', price: 3 },
    { id: '3', name: '橙子', price: 4 },
  ];

  return (
    <div>
      <h2>购物车</h2>

      <div>
        <h3>添加商品</h3>
        {products.map(product => (
          <button key={product.id} onClick={() => addItem(product)}>
            添加 {product.name} - ¥{product.price}
          </button>
        ))}
      </div>

      <div>
        <h3>购物车内容</h3>
        {items.length === 0 ? (
          <p>购物车为空</p>
        ) : (
          items.map(item => (
            <div key={item.id} style={{ marginBottom: 10 }}>
              <span>{item.name}</span>
              <span>¥{item.price} x {item.quantity}</span>
              <button onClick={() => updateQuantity(item.id, item.quantity - 1)}>-</button>
              <button onClick={() => updateQuantity(item.id, item.quantity + 1)}>+</button>
              <button onClick={() => removeItem(item.id)}>删除</button>
            </div>
          ))
        )}
      </div>

      {items.length > 0 && (
        <div>
          <p>总计：¥{getTotal()}</p>
          <button onClick={clearCart}>清空购物车</button>
        </div>
      )}
    </div>
  );
}

export default Cart;
```

---

### 2. 样式方案

- [ ] CSS Modules
- [ ] Styled Components
- [ ] Tailwind CSS（企业主流）
- [ ] Ant Design / Material UI 组件库

**详细概念：**

**2.1 CSS Modules 原理与应用**

CSS Modules 是一种 CSS  scoping（作用域）解决方案，通过将类名编译为唯一哈希值来实现样式隔离，避免全局污染。

核心特性：
- **局部作用域**：类名自动转换为唯一哈希，天然避免冲突
- **组合式类名**：支持 `.button.primary` 组合 `.button` 和 `.primary` 的样式
- **引用导出**：通过 `styles.xxx` 方式引用类名，IDE 自动补全支持
- **零运行时开销**：编译时处理，不影响运行时性能

使用场景：
- 适合需要组件级样式隔离的项目
- 适合团队协作，避免样式冲突
- 适合已有 CSS 资产的项目渐进迁移

**2.2 Styled Components 动态样式**

Styled Components 是 CSS-in-JS 方案的代表，它使用模板字符串定义样式，生成带唯一类名的 React 组件。

核心优势：
- **组件化**：样式本身就是组件，可复用、可嵌套
- **Props 驱动**：样式可根据 props 动态变化
- **主题系统**：内置 ThemeProvider，支持深色/浅色主题切换
- **自动关键CSS**：只会注入页面使用到的样式

使用建议：
- 适合需要大量动态样式的场景（如主题切换、数据可视化）
- 适合追求极致开发体验的项目
- 注意运行时性能，大规模使用可能影响首屏性能

**2.3 Tailwind CSS 实用主义美学**

Tailwind CSS 是一个以"实用类"为核心的 CSS 框架，通过组合小粒度类名（如 `flex p-4 text-center`）来实现样式，颠覆了传统 CSS 开发方式。

核心理念：
- **原子化设计**：每个类只做一个样式属性，高度可组合
- **直观的命名**：语义化的类名，如 `text-lg`（大文本）、`bg-blue-500`（蓝色背景）
- **响应式设计**：前缀支持（`md:`、`lg:`），轻松适配多端
- **暗色模式**：`dark:` 前缀支持主题切换

企业主流原因：
- **开发效率极高**：无需切换文件，组件和样式同处一地
- **一致性保证**：预定义的设计令牌保证视觉统一
- **体积优化**：支持 Tree-shaking，未使用的类不打包
- **生态系统成熟**：配套工具完善（VSCode 插件、预ttier 插件等）

**2.4 UI 组件库选型**

UI 组件库是现成的组件集合，可以快速构建功能完整、视觉统一的应用界面。

主流 React 组件库对比：

| 组件库 | 特点 | 适用场景 |
|--------|------|----------|
| Ant Design | 阿里出品，企业级，中后台首选 | 中后台系统、数据密集型应用 |
| Material UI | Google Material Design 实现 | 追求国际化、通用型应用 |
| Chakra UI | 现代化设计，高度可定制 | 快速原型、需要品牌定制化 |
| Radix UI | 无样式、语义化、可访问性好 | 需要深度定制设计系统 |

选型建议：
- 中后台系统优先考虑 Ant Design（组件丰富、生态完善）
- 追求现代感 UI 可考虑 Chakra UI 或 Radix UI
- 需要快速交付可先用组件库，再按需定制

**经典案例：CSS Modules**

```css
/* Button.module.css */
.button {
  padding: 10px 20px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 16px;
  transition: background-color 0.3s;
}

.primary {
  background-color: #007bff;
  color: white;
}

.secondary {
  background-color: #6c757d;
  color: white;
}

.button:hover {
  opacity: 0.9;
}

.button:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}
```

```tsx
// Button.tsx
import styles from './Button.module.css';

interface ButtonProps {
  children: React.ReactNode;
  variant?: 'primary' | 'secondary';
  disabled?: boolean;
  onClick?: () => void;
}

function Button({ children, variant = 'primary', disabled = false, onClick }: ButtonProps) {
  const className = `${styles.button} ${styles[variant]}`;

  return (
    <button className={className} disabled={disabled} onClick={onClick}>
      {children}
    </button>
  );
}

export default Button;
```

**经典案例：Tailwind CSS**

```tsx
// TailwindComponents.tsx
function TailwindComponents() {
  return (
    <div className="p-6 max-w-md mx-auto bg-white rounded-xl shadow-md">
      <h2 className="text-2xl font-bold text-gray-900 mb-4">
        Tailwind CSS 示例
      </h2>

      <div className="space-y-4">
        <input
          type="text"
          className="w-full px-4 py-2 border border-gray-300 rounded-lg focus:outline-none focus:ring-2 focus:ring-blue-500"
          placeholder="输入内容..."
        />

        <div className="flex space-x-2">
          <button className="px-4 py-2 bg-blue-500 text-white rounded-lg hover:bg-blue-600 transition-colors">
            主要按钮
          </button>
          <button className="px-4 py-2 bg-gray-200 text-gray-700 rounded-lg hover:bg-gray-300 transition-colors">
            次要按钮
          </button>
        </div>

        <div className="grid grid-cols-3 gap-2">
          {[1, 2, 3, 4, 5, 6].map(i => (
            <div
              key={i}
              className="aspect-square bg-gradient-to-br from-blue-400 to-purple-500 rounded-lg flex items-center justify-center text-white font-bold"
            >
              {i}
            </div>
          ))}
        </div>
      </div>
    </div>
  );
}

export default TailwindComponents;
```

**经典案例：Ant Design 组件库**

```tsx
// AntDesignExample.tsx
import { Button, Input, Select, Table, Form, Modal, message } from 'antd';

const { Column } = Table;

interface User {
  id: string;
  name: string;
  email: string;
  status: 'active' | 'inactive';
}

function AntDesignExample() {
  const [form] = Form.useForm();
  const [isModalOpen, setIsModalOpen] = useState(false);
  const [users, setUsers] = useState<User[]>([
    { id: '1', name: '张三', email: 'zhang@example.com', status: 'active' },
    { id: '2', name: '李四', email: 'li@example.com', status: 'inactive' },
  ]);

  const handleFinish = (values: any) => {
    const newUser = {
      id: Date.now().toString(),
      ...values,
      status: 'active' as const,
    };
    setUsers([...users, newUser]);
    setIsModalOpen(false);
    form.resetFields();
    message.success('用户添加成功');
  };

  return (
    <div style={{ padding: 24 }}>
      <h2>Ant Design 示例</h2>

      <Button type="primary" onClick={() => setIsModalOpen(true)}>
        添加用户
      </Button>

      <Table dataSource={users} style={{ marginTop: 16 }} rowKey="id">
        <Column title="姓名" dataIndex="name" key="name" />
        <Column title="邮箱" dataIndex="email" key="email" />
        <Column
          title="状态"
          dataIndex="status"
          key="status"
          render={(status: string) => (
            <Tag color={status === 'active' ? 'green' : 'red'}>
              {status === 'active' ? '活跃' : '不活跃'}
            </Tag>
          )}
        />
      </Table>

      <Modal
        title="添加用户"
        open={isModalOpen}
        onCancel={() => setIsModalOpen(false)}
        footer={null}
      >
        <Form form={form} layout="vertical" onFinish={handleFinish}>
          <Form.Item
            name="name"
            label="姓名"
            rules={[{ required: true, message: '请输入姓名' }]}
          >
            <Input />
          </Form.Item>
          <Form.Item
            name="email"
            label="邮箱"
            rules={[
              { required: true, message: '请输入邮箱' },
              { type: 'email', message: '请输入有效邮箱' },
            ]}
          >
            <Input />
          </Form.Item>
          <Form.Item>
            <Button type="primary" htmlType="submit">
              提交
            </Button>
          </Form.Item>
        </Form>
      </Modal>
    </div>
  );
}

export default AntDesignExample;
```

---

### 3. 网络请求

- [ ] Axios 封装：请求拦截、响应拦截、错误统一处理
- [ ] React Query / SWR：数据请求、缓存、自动刷新（进阶必备）

**详细概念：**

**3.1 Axios 封装最佳实践**

Axios 是最流行的 HTTP 请求库，提供 Promise API，支持请求/响应拦截、自动转换 JSON、取消请求等功能。

封装的核心价值：
- **统一配置**：集中管理 baseURL、超时时间、请求头等公共配置
- **请求拦截**：自动添加认证 token、loading 状态
- **响应拦截**：统一处理错误、token 过期自动刷新
- **类型支持**：完整的 TypeScript 类型提示

企业级封装要点：
- **统一的错误处理**：区分网络错误、服务器错误、业务错误
- **Token 管理**：自动刷新 token 机制
- **请求取消**：防止竞态条件（如搜索联想）
- **日志记录**：方便调试和问题排查

**3.2 React Query 数据获取革命**

React Query（现为 TanStack Query）是服务端状态管理的革命性方案，它将服务器数据（API 响应）视为独立的状态进行管理。

核心概念：
- **声明式数据获取**：只需声明数据来源，React Query 自动管理获取、缓存、更新
- **自动缓存**：数据会被缓存，重复请求直接使用缓存，减少 API 调用
- **后台刷新**：页面重新聚焦时自动更新数据，保证数据新鲜
- **乐观更新**：先更新 UI，再发送请求，提供即时反馈

对比 Redux 处理 API：
- Redux：需要手动管理 loading/error/data 状态，代码量大
- React Query：自动管理，代码简洁，专注业务逻辑

**3.3 SWR 轻量选择**

SWR（Stale-While-Revalidate）是 Vercel 开发的轻量级数据请求库，理念与 React Query 类似，但 API 更简洁。

SWR 的优势：
- **极小的体积**：比 React Query 小很多
- **内置轮询**：简单的 `refreshInterval` 配置即可实现轮询
- **聚焦简单**：没有 React Query 那么多功能，但足够日常使用

**经典案例：Axios 封装**

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

  patch: <T>(url: string, data?: unknown) =>
    api.patch<ApiResponse<T>>(url, data).then(res => res.data.data),

  delete: <T>(url: string, params?: Record<string, unknown>) =>
    api.delete<ApiResponse<T>>(url, { params }).then(res => res.data.data),
};

export default api;
```

```typescript
// services/userService.ts
import { request } from '../utils/api';

interface User {
  id: string;
  name: string;
  email: string;
  avatar?: string;
  role: 'admin' | 'user';
  createdAt: string;
}

export const userService = {
  getUsers: (params?: { page?: number; pageSize?: number }) =>
    request.get<{ list: User[]; total: number }>('/users', params),

  getUserById: (id: string) =>
    request.get<User>(`/users/${id}`),

  createUser: (data: Omit<User, 'id' | 'createdAt'>) =>
    request.post<User>('/users', data),

  updateUser: (id: string, data: Partial<Omit<User, 'id' | 'createdAt'>>) =>
    request.put<User>(`/users/${id}`, data),

  deleteUser: (id: string) =>
    request.delete<void>(`/users/${id}`),

  updateAvatar: (id: string, formData: FormData) =>
    request.patch<User>(`/users/${id}/avatar`, formData),
};
```

**经典案例：React Query 数据请求**

```tsx
// hooks/useUsers.ts
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';
import { userService } from '../services/userService';

export function useUsers(page = 1, pageSize = 10) {
  return useQuery({
    queryKey: ['users', { page, pageSize }],
    queryFn: () => userService.getUsers({ page, pageSize }),
    keepPreviousData: true,
  });
}

export function useUser(id: string) {
  return useQuery({
    queryKey: ['user', id],
    queryFn: () => userService.getUserById(id),
    enabled: !!id,
  });
}

export function useCreateUser() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: userService.createUser,
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['users'] });
    },
  });
}

export function useUpdateUser() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: ({ id, data }: { id: string; data: any }) =>
      userService.updateUser(id, data),
    onSuccess: (_, variables) => {
      queryClient.invalidateQueries({ queryKey: ['users'] });
      queryClient.invalidateQueries({ queryKey: ['user', variables.id] });
    },
  });
}

export function useDeleteUser() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: userService.deleteUser,
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['users'] });
    },
  });
}
```

```tsx
// components/UserList.tsx
import { useState } from 'react';
import { useUsers, useDeleteUser } from '../hooks/useUsers';

function UserList() {
  const [page, setPage] = useState(1);
  const { data, isLoading, isFetching, error } = useUsers(page);
  const deleteUser = useDeleteUser();

  const handleDelete = (id: string) => {
    if (confirm('确定要删除该用户吗？')) {
      deleteUser.mutate(id);
    }
  };

  if (isLoading) return <div>加载中...</div>;
  if (error) return <div>错误：{error.message}</div>;

  return (
    <div>
      <h2>用户列表</h2>

      {isFetching && <div>刷新中...</div>}

      <ul>
        {data?.list.map(user => (
          <li key={user.id}>
            <span>{user.name}</span>
            <span>{user.email}</span>
            <button onClick={() => handleDelete(user.id)}>删除</button>
          </li>
        ))}
      </ul>

      <div>
        <button disabled={page === 1} onClick={() => setPage(p => p - 1)}>
          上一页
        </button>
        <span>第 {page} 页</span>
        <button
          disabled={data?.list.length === data?.total}
          onClick={() => setPage(p => p + 1)}
        >
          下一页
        </button>
      </div>
    </div>
  );
}

export default UserList;
```

---

### 4. TypeScript + React（必学）

- [ ] 为 Props/State/Hooks 定义类型
- [ ] 泛型组件
- [ ] 类型推导
- [ ] 避免 any 代码
- [ ] TS 常用类型：interface、type、泛型

**详细概念：**

**4.1 TypeScript 在 React 中的价值**

TypeScript 是 JavaScript 的超集，添加了可选的静态类型系统。React 项目使用 TypeScript 可以获得：
- **编译时检查**：很多错误在编译时就能发现，而不是等到运行时
- **智能提示**：IDE 提供准确的属性、方法提示，提高开发效率
- **代码文档**：类型本身就是最好的文档
- **重构支持**：类型系统让重构更安全、更自信

为什么要避免 any：
- `any` 类型会绕过所有类型检查，失去 TypeScript 的保护
- 使用 `any` 意味着放弃 IDE 的智能提示
- 正确做法是使用 `unknown` 或具体类型

**4.2 Props 类型定义规范**

Props 是组件的输入，良好的类型定义让组件接口清晰可预测。

常见 Props 类型模式：
- **基础类型**：`string`、`number`、`boolean` 直接标注
- **可选属性**：使用 `?` 标记可选 props
- **函数类型**：明确函数签名，包括参数和返回值类型
- **联合类型**：限制 props 只能是特定值之一
- **默认Props**：使用默认值或 `?` 配合默认值处理

最佳实践：
```tsx
interface ButtonProps {
  variant: 'primary' | 'secondary' | 'danger';
  size?: 'small' | 'medium' | 'large';  // 可选
  disabled?: boolean;
  onClick: () => void;
  children: React.ReactNode;  // 支持 JSX 子元素
}
```

**4.3 泛型组件设计**

泛型组件是能够处理多种数据类型的组件，在保持类型安全的同时提供最大的灵活性。

常见泛型场景：
- **列表组件**：可以展示任何类型的列表
- **表格组件**：可以渲染任何数据结构
- **表单组件**：可以处理各种表单数据
- **选择组件**：支持各种选项类型

泛型约束：
- 使用 `extends` 约束泛型必须具有特定属性
- 如 `<T extends Record<string, unknown>>` 确保 T 是对象类型

**4.4 Hooks 类型标注**

React Hooks 都有对应的类型定义，正确标注可以让 Hooks 的返回值类型准确。

常用 Hooks 类型：
- `useState<T>`：指定 state 类型
- `useRef<T>`：指定 ref 绑定的元素或值的类型
- `useCallback<T>`：函数类型由参数和返回值自动推断
- `useMemo<T>`：指定计算结果的类型

**经典案例：Props 与 State 类型定义**

```tsx
// types.ts
export interface User {
  id: string;
  name: string;
  email: string;
  avatar?: string;
  role: 'admin' | 'user' | 'guest';
  status: 'active' | 'inactive' | 'banned';
  createdAt: Date;
  metadata?: Record<string, unknown>;
}

export interface UserFormData {
  name: string;
  email: string;
  role: User['role'];
}

export type UserStatus = User['status'];
export type UserRole = User['role'];
```

```tsx
// UserCard.tsx
import { User } from '../types';

interface UserCardProps {
  user: User;
  onEdit?: (user: User) => void;
  onDelete?: (id: string) => void;
  isSelected?: boolean;
}

function UserCard({ user, onEdit, onDelete, isSelected = false }: UserCardProps) {
  const handleEdit = () => {
    onEdit?.(user);
  };

  const handleDelete = () => {
    onDelete?.(user.id);
  };

  return (
    <div
      style={{
        padding: 16,
        border: '1px solid #ddd',
        borderRadius: 8,
        backgroundColor: isSelected ? '#f5f5f5' : '#fff',
      }}
    >
      <img
        src={user.avatar || '/default-avatar.png'}
        alt={user.name}
        style={{ width: 50, height: 50, borderRadius: '50%' }}
      />
      <h3>{user.name}</h3>
      <p>{user.email}</p>
      <p>角色：{user.role}</p>
      <p>状态：{user.status}</p>
      <button onClick={handleEdit}>编辑</button>
      <button onClick={handleDelete}>删除</button>
    </div>
  );
}

export default UserCard;
```

**经典案例：泛型组件**

```tsx
// GenericList.tsx
import { ReactNode } from 'react';

interface Column<T> {
  key: keyof T;
  title: string;
  width?: string;
  render?: (value: T[keyof T], record: T, index: number) => ReactNode;
}

interface GenericListProps<T> {
  data: T[];
  columns: Column<T>[];
  rowKey: keyof T;
  emptyText?: string;
  onRowClick?: (record: T, index: number) => void;
  loading?: boolean;
}

function GenericList<T extends Record<string, unknown>>({
  data,
  columns,
  rowKey,
  emptyText = '暂无数据',
  onRowClick,
  loading = false,
}: GenericListProps<T>) {
  if (loading) {
    return <div>加载中...</div>;
  }

  if (data.length === 0) {
    return <div>{emptyText}</div>;
  }

  return (
    <table style={{ width: '100%', borderCollapse: 'collapse' }}>
      <thead>
        <tr>
          {columns.map(col => (
            <th
              key={String(col.key)}
              style={{ padding: 12, borderBottom: '2px solid #ddd', width: col.width }}
            >
              {col.title}
            </th>
          ))}
        </tr>
      </thead>
      <tbody>
        {data.map((record, index) => (
          <tr
            key={String(record[rowKey]) || index}
            onClick={() => onRowClick?.(record, index)}
            style={{ cursor: onRowClick ? 'pointer' : 'default' }}
          >
            {columns.map(col => (
              <td
                key={String(col.key)}
                style={{ padding: 12, borderBottom: '1px solid #eee' }}
              >
                {col.render
                  ? col.render(record[col.key], record, index)
                  : String(record[col.key])}
              </td>
            ))}
          </tr>
        ))}
      </tbody>
    </table>
  );
}

export default GenericList;
```

```tsx
// 使用示例
interface Product {
  id: string;
  name: string;
  price: number;
  stock: number;
  status: 'in_stock' | 'out_of_stock';
}

const columns: Column<Product>[] = [
  { key: 'name', title: '商品名称' },
  { key: 'price', title: '价格', render: (value) => `¥${value}` },
  {
    key: 'status',
    title: '状态',
    render: (value) => (
      <span style={{ color: value === 'in_stock' ? 'green' : 'red' }}>
        {value === 'in_stock' ? '有货' : '缺货'}
      </span>
    ),
  },
];

function ProductList() {
  const [products] = useState<Product[]>([
    { id: '1', name: '苹果', price: 5, stock: 100, status: 'in_stock' },
    { id: '2', name: '香蕉', price: 3, stock: 0, status: 'out_of_stock' },
  ]);

  return (
    <GenericList
      data={products}
      columns={columns}
      rowKey="id"
      onRowClick={(product) => console.log('点击了', product.name)}
    />
  );
}
```

**经典案例：事件处理类型**

```tsx
// EventHandling.tsx
import { useState, ChangeEvent, MouseEvent, KeyboardEvent, FormEvent } from 'react';

function EventHandling() {
  const [inputValue, setInputValue] = useState('');
  const [selected, setSelected] = useState<string | null>(null);

  const handleChange = (e: ChangeEvent<HTMLInputElement>) => {
    setInputValue(e.target.value);
  };

  const handleSelect = (e: ChangeEvent<HTMLSelectElement>) => {
    setSelected(e.target.value);
  };

  const handleClick = (e: MouseEvent<HTMLButtonElement>) => {
    console.log('点击位置:', e.clientX, e.clientY);
  };

  const handleKeyDown = (e: KeyboardEvent<HTMLInputElement>) => {
    if (e.key === 'Enter') {
      console.log('按下回车:', inputValue);
    }
  };

  const handleSubmit = (e: FormEvent<HTMLFormElement>) => {
    e.preventDefault();
    console.log('表单提交');
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        value={inputValue}
        onChange={handleChange}
        onKeyDown={handleKeyDown}
        placeholder="输入内容..."
      />

      <select value={selected || ''} onChange={handleSelect}>
        <option value="">请选择</option>
        <option value="a">选项 A</option>
        <option value="b">选项 B</option>
      </select>

      <button type="submit" onClick={handleClick}>
        提交
      </button>
    </form>
  );
}

export default EventHandling;
```

---

### 5. 工程化与规范

- [ ] ESLint + Prettier 代码规范
- [ ] Git 版本管理、Commitlint 提交规范
- [ ] 环境变量配置
- [ ] 打包构建优化

**详细概念：**

**5.1 ESLint + Prettier 代码规范**

代码规范是团队协作的基础。ESLint 负责代码质量检查（潜在错误、坏味道），Prettier 负责代码格式统一（风格一致）。

分工协作：
- **ESLint**：检查逻辑问题、强制最佳实践（如禁止 `var`、要求 `const`）
- **Prettier**：处理格式化（缩进、分号、引号、换行）
- **两者互补**：ESLint 的 --cache 选项和 Prettier 的 --write 选项配合使用

企业配置建议：
- 使用主流共享配置（如 eslint-config-airbnb、eslint-config-standard）
- 根据项目需求自定义规则
- CI/CD 中集成 lint 检查

**5.2 Git 提交规范**

统一的提交信息规范让团队协作更高效，便于生成 CHANGELOG 和追溯问题。

Angular 提交规范格式：
```
<type>(<scope>): <subject>

<body>

<footer>
```

常用 Type：
- `feat`：新功能
- `fix`：Bug 修复
- `docs`：文档更新
- `style`：格式调整（不影响代码运行）
- `refactor`：重构
- `test`：测试相关
- `chore`：构建/工具/依赖更新

Commitlint 作用：
- 校验提交信息是否符合规范
- 防止不规范提交进入仓库
- 配合 Conventional Commits 生成版本日志

**5.3 环境变量配置**

不同环境（开发、测试、生产）通常需要不同的配置，环境变量是管理这些配置的标准方式。

前端环境变量规则：
- 以 `VITE_` 开头的变量可在客户端代码中使用
- 以 `VITE_` 开头的变量会在构建时被嵌入到输出代码中
- 敏感信息不要暴露在前端代码中

多环境配置策略：
- `.env`：默认/共享配置
- `.env.development`：开发环境覆盖
- `.env.production`：生产环境覆盖
- `.env.local`：本地覆盖（通常加入 .gitignore）

**5.4 打包构建优化**

生产环境打包优化直接影响应用性能和用户体验。

核心优化方向：
- **代码分割**：按路由或组件分割，减少首屏加载体积
- **Tree Shaking**：移除未使用的代码
- **压缩优化**：JS/CSS 压缩、混淆
- **资源优化**：图片压缩、字体子集化
- **缓存策略**：长期缓存（hash 命名）

构建工具选择：
- **Vite**：开发体验好，Rollup 打包，生态成熟
- **Webpack**：功能全面，配置灵活，大型项目首选
- **esbuild**：极快的构建速度，适合对速度敏感的场景

**经典案例：ESLint + Prettier 配置**

```json
// .eslintrc.json
{
  "extends": [
    "eslint:recommended",
    "plugin:react/recommended",
    "plugin:@typescript-eslint/recommended",
    "plugin:react-hooks/recommended",
    "prettier"
  ],
  "plugins": ["react", "@typescript-eslint", "react-hooks"],
  "parser": "@typescript-eslint/parser",
  "parserOptions": {
    "ecmaVersion": 2022,
    "sourceType": "module",
    "ecmaFeatures": {
      "jsx": true
    }
  },
  "settings": {
    "react": {
      "version": "detect"
    }
  },
  "rules": {
    "@typescript-eslint/no-unused-vars": ["error", { "argsIgnorePattern": "^_" }],
    "@typescript-eslint/explicit-function-return-type": "off",
    "react/prop-types": "off",
    "react/react-in-jsx-scope": "off",
    "no-console": ["warn", { "allow": ["warn", "error"] }]
  }
}
```

```json
// .prettierrc
{
  "semi": true,
  "singleQuote": true,
  "tabWidth": 2,
  "trailingComma": "es5",
  "printWidth": 100,
  "arrowParens": "always",
  "endOfLine": "lf"
}
```

**经典案例：环境变量配置**

```bash
# .env.development
VITE_API_BASE_URL=http://localhost:8080/api
VITE_APP_TITLE=开发环境
VITE_ENABLE_DEBUG=true

# .env.production
VITE_API_BASE_URL=https://api.example.com
VITE_APP_TITLE=生产环境
VITE_ENABLE_DEBUG=false
```

```typescript
// config.ts
interface EnvConfig {
  apiBaseUrl: string;
  appTitle: string;
  enableDebug: boolean;
}

const env: EnvConfig = {
  apiBaseUrl: import.meta.env.VITE_API_BASE_URL as string,
  appTitle: import.meta.env.VITE_APP_TITLE as string,
  enableDebug: import.meta.env.VITE_ENABLE_DEBUG === 'true',
};

export default env;
```

**经典案例：Git 提交规范**

```bash
# 提交信息格式
# <type>(<scope>): <subject>
#
# type: feat | fix | docs | style | refactor | test | chore
# scope: 涉及模块
# subject: 简短描述

# 示例
git commit -m "feat(user): 添加用户登录功能"
git commit -m "fix(cart): 修复购物车数量计算错误"
git commit -m "docs(readme): 更新项目文档"
git commit -m "style(button): 调整按钮样式"
git commit -m "refactor(api): 重构 API 请求模块"
git commit -m "test(user): 添加用户模块单元测试"
git commit -m "chore(deps): 更新项目依赖"
```

---

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

## 下一阶段预告

完成生态与工程化后，将进入 [React 高级进阶](./stage-4-advanced.md)，你将学习：
- React 渲染原理、Fiber 架构
- 深度性能优化
- Next.js SSR/SSG
- 测试与部署
- 架构设计
