# 阶段三：React 生态与工程化（中级开发）

**目标**：适配企业开发流程、掌握主流工具链

## 必须掌握的技能

### 1. 状态管理（进阶）

- [ ] Redux Toolkit（现代 Redux，企业首选）
- [ ] Zustand/Jotai（轻量级状态库，简单易用）
- [ ] 状态持久化
- [ ] 异步状态处理

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
