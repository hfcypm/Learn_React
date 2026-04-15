# 阶段四：React 高级进阶（高级开发）

**目标**：解决大型项目难题、掌握性能调优、架构设计

## 必须掌握的技能

### 1. 深度性能优化

- [ ] React 渲染原理、Fiber 架构
- [ ] 调和（Reconciliation）机制
- [ ] 生产环境性能分析
- [ ] 内存泄漏排查
- [ ] 图片优化、打包体积优化
- [ ] 首屏加载优化

### 2. 复杂场景解决方案

- [ ] 表单高阶方案（React Hook Form、Formik）
- [ ] 表格虚拟化
- [ ] 无限滚动
- [ ] 大数据渲染
- [ ] 微前端（qiankun）
- [ ] 跨应用集成

### 3. 服务端渲染/静态站点（SSR/SSG）

- [ ] Next.js（React 官方推荐框架）核心用法
- [ ] 服务端数据获取
- [ ] SEO 优化
- [ ] 路由方案

### 4. 测试与部署

- [ ] React 组件单元测试（Jest + React Testing Library）
- [ ] E2E 测试
- [ ] 自动化部署

### 5. 架构设计

- [ ] 项目目录结构设计、模块化拆分
- [ ] 公共逻辑封装
- [ ] 通用组件/Hooks 抽离
- [ ] 权限系统
- [ ] 国际化（i18n）方案

## 了解即可

- Fiber 架构的具体实现细节
- 微前端的深入原理
- React 18 并发特性

## 阶段成果

能主导以下工作：
- 大型 React 项目架构
- 性能优化
- 团队代码规范
- 技术方案选型

## 性能优化清单

### 渲染优化
- [ ] 合理使用 React.memo
- [ ] 合理使用 useMemo/useCallback
- [ ] 避免匿名函数和对象字面量
- [ ] 列表使用虚拟滚动

### 加载优化
- [ ] 代码分割（React.lazy）
- [ ] 路由懒加载
- [ ] 图片懒加载
- [ ] 预加载关键资源

### 包体积优化
- [ ] Tree Shaking
- [ ] 按需加载组件库
- [ ] 外部依赖优化
- [ ] gzip 压缩

## Next.js 核心概念

| 概念 | 说明 |
|------|------|
| SSG | 静态站点生成，构建时生成 HTML |
| SSR | 服务端渲染，请求时生成 HTML |
| ISR | 增量静态再生成，定期自动重新生成 |
| API Routes | Next.js API 接口 |

## 经典学习案例

### 案例 1：React Hook Form + Yup 表单验证

```tsx
import { useForm } from 'react-hook-form';
import { yupResolver } from '@hookform/resolvers/yup';
import * as yup from 'yup';

interface FormData {
  username: string;
  email: string;
  password: string;
  confirmPassword: string;
  age: number;
}

const schema = yup.object({
  username: yup
    .string()
    .required('用户名不能为空')
    .min(3, '用户名至少3个字符')
    .max(20, '用户名最多20个字符'),
  email: yup
    .string()
    .required('邮箱不能为空')
    .email('请输入有效的邮箱地址'),
  password: yup
    .string()
    .required('密码不能为空')
    .min(8, '密码至少8个字符')
    .matches(/[A-Z]/, '密码必须包含大写字母')
    .matches(/[a-z]/, '密码必须包含小写字母')
    .matches(/[0-9]/, '密码必须包含数字'),
  confirmPassword: yup
    .string()
    .required('请确认密码')
    .oneOf([yup.ref('password')], '两次输入的密码不一致'),
  age: yup
    .number()
    .required('年龄不能为空')
    .min(18, '必须年满18岁')
    .max(100, '年龄不能超过100岁'),
});

function RegisterForm() {
  const {
    register,
    handleSubmit,
    formState: { errors, isSubmitting, isDirty, isValid },
  } = useForm<FormData>({
    resolver: yupResolver(schema),
    mode: 'onChange',
  });

  const onSubmit = async (data: FormData) => {
    await new Promise(resolve => setTimeout(resolve, 1000));
    console.log('表单数据：', data);
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <div>
        <label>用户名</label>
        <input {...register('username')} />
        {errors.username && <p>{errors.username.message}</p>}
      </div>

      <div>
        <label>邮箱</label>
        <input {...register('email')} type="email" />
        {errors.email && <p>{errors.email.message}</p>}
      </div>

      <div>
        <label>密码</label>
        <input {...register('password')} type="password" />
        {errors.password && <p>{errors.password.message}</p>}
      </div>

      <div>
        <label>确认密码</label>
        <input {...register('confirmPassword')} type="password" />
        {errors.confirmPassword && <p>{errors.confirmPassword.message}</p>}
      </div>

      <div>
        <label>年龄</label>
        <input {...register('age', { valueAsNumber: true })} type="number" />
        {errors.age && <p>{errors.age.message}</p>}
      </div>

      <button type="submit" disabled={!isDirty || !isValid || isSubmitting}>
        {isSubmitting ? '提交中...' : '注册'}
      </button>
    </form>
  );
}
```

### 案例 2：React.lazy + Suspense 路由懒加载

```tsx
import { lazy, Suspense } from 'react';
import { BrowserRouter, Routes, Route } from 'react-router-dom';

const Home = lazy(() => import('./pages/Home'));
const About = lazy(() => import('./pages/About'));
const Dashboard = lazy(() => import('./pages/Dashboard'));
const UserList = lazy(() => import('./pages/UserList'));
const UserProfile = lazy(() => import('./pages/UserProfile'));
const NotFound = lazy(() => import('./pages/NotFound'));

function Loading() {
  return (
    <div style={{
      display: 'flex',
      justifyContent: 'center',
      alignItems: 'center',
      height: '100vh'
    }}>
      <div className="spinner" />
      <p>加载中...</p>
    </div>
  );
}

function ErrorBoundary({ children }: { children: React.ReactNode }) {
  return (
    <ErrorFallback>
      <Suspense fallback={<Loading />}>{children}</Suspense>
    </ErrorFallback>
  );
}

function App() {
  return (
    <BrowserRouter>
      <ErrorBoundary>
        <Routes>
          <Route path="/" element={<Home />} />
          <Route path="/about" element={<About />} />
          <Route path="/dashboard" element={<Dashboard />} />
          <Route path="/users">
            <Route index element={<UserList />} />
            <Route path=":id" element={<UserProfile />} />
          </Route>
          <Route path="*" element={<NotFound />} />
        </Routes>
      </ErrorBoundary>
    </BrowserRouter>
  );
}
```

### 案例 3：虚拟列表实现

```tsx
import { useState, useRef, useCallback } from 'react';

interface VirtualListProps<T> {
  items: T[];
  height: number;
  itemHeight: number;
  renderItem: (item: T, index: number) => React.ReactNode;
}

function VirtualList<T>({ items, height, itemHeight, renderItem }: VirtualListProps<T>) {
  const [scrollTop, setScrollTop] = useState(0);
  const containerRef = useRef<HTMLDivElement>(null);

  const visibleCount = Math.ceil(height / itemHeight);
  const startIndex = Math.floor(scrollTop / itemHeight);
  const endIndex = Math.min(startIndex + visibleCount + 2, items.length);

  const visibleItems = [];
  for (let i = startIndex; i < endIndex; i++) {
    visibleItems.push(
      <div
        key={i}
        style={{
          position: 'absolute',
          top: `${i * itemHeight}px`,
          height: `${itemHeight}px`,
          width: '100%',
        }}
      >
        {renderItem(items[i], i)}
      </div>
    );
  }

  const handleScroll = useCallback((e: React.UIEvent<HTMLDivElement>) => {
    setScrollTop(e.currentTarget.scrollTop);
  }, []);

  return (
    <div
      ref={containerRef}
      style={{ height, overflow: 'auto', position: 'relative' }}
      onScroll={handleScroll}
    >
      <div style={{ height: items.length * itemHeight, position: 'relative' }}>
        {visibleItems}
      </div>
    </div>
  );
}

// 使用示例
function LargeListDemo() {
  const items = Array.from({ length: 10000 }, (_, i) => ({
    id: i,
    title: `项目 ${i + 1}`,
    description: `这是第 ${i + 1} 项的描述内容`,
  }));

  return (
    <VirtualList
      items={items}
      height={400}
      itemHeight={60}
      renderItem={(item) => (
        <div style={{ padding: 10, borderBottom: '1px solid #eee' }}>
          <h3>{item.title}</h3>
          <p>{item.description}</p>
        </div>
      )}
    />
  );
}
```

### 案例 4：Next.js SSR + SSG 混合使用

```tsx
// pages/posts/[id].tsx
import { GetStaticPaths, GetStaticProps, InferGetStaticPropsType } from 'next';
import { ParsedUrlQuery } from 'querystring';

interface Post {
  id: string;
  title: string;
  content: string;
  publishDate: string;
  author: Author;
}

interface Author {
  id: string;
  name: string;
  avatar: string;
}

interface Params extends ParsedUrlQuery {
  id: string;
}

type Props = {
  post: Post;
};

export const getStaticPaths: GetStaticPaths<Params> = async () => {
  const response = await fetch('https://api.example.com/posts');
  const posts: Post[] = await response.json();

  return {
    paths: posts.map(post => ({ params: { id: post.id } })),
    fallback: 'blocking',
  };
};

export const getStaticProps: GetStaticProps<Props, Params> = async ({ params }) => {
  const response = await fetch(`https://api.example.com/posts/${params?.id}`);

  if (!response.ok) {
    return { notFound: true };
  }

  const post = await response.json();

  return {
    props: { post },
    revalidate: 60,
  };
};

export default function PostPage({ post }: InferGetStaticPropsType<typeof getStaticProps>) {
  return (
    <article>
      <header>
        <h1>{post.title}</h1>
        <div className="author">
          <img src={post.author.avatar} alt={post.author.name} />
          <span>{post.author.name}</span>
          <time>{post.publishDate}</time>
        </div>
      </header>
      <div className="content">
        {post.content}
      </div>
    </article>
  );
}
```

```tsx
// pages/index.tsx
import { GetStaticProps, InferGetStaticPropsType } from 'next';

interface FeaturedPost {
  id: string;
  title: string;
  excerpt: string;
}

type Props = {
  featuredPosts: FeaturedPost[];
};

export const getStaticProps: GetStaticProps<Props> = async () => {
  const response = await fetch('https://api.example.com/posts/featured');
  const featuredPosts = await response.json();

  return {
    props: { featuredPosts },
  };
};

export default function HomePage({ featuredPosts }: InferGetStaticPropsType<typeof getStaticProps>) {
  return (
    <div>
      <h1>精选文章</h1>
      <ul>
        {featuredPosts.map(post => (
          <li key={post.id}>
            <h2>{post.title}</h2>
            <p>{post.excerpt}</p>
          </li>
        ))}
      </ul>
    </div>
  );
}
```

### 案例 5：Jest + Testing Library 单元测试

```tsx
// components/Counter.tsx
import { useState } from 'react';

interface CounterProps {
  initialCount?: number;
  onIncrement?: () => void;
}

export function Counter({ initialCount = 0, onIncrement }: CounterProps) {
  const [count, setCount] = useState(initialCount);

  const handleIncrement = () => {
    setCount(prev => prev + 1);
    onIncrement?.();
  };

  const handleDecrement = () => {
    setCount(prev => prev - 1);
  };

  return (
    <div>
      <button data-testid="decrement-btn" onClick={handleDecrement}>-</button>
      <span data-testid="count-display">{count}</span>
      <button data-testid="increment-btn" onClick={handleIncrement}>+</button>
    </div>
  );
}
```

```tsx
// components/Counter.test.tsx
import { render, screen, fireEvent } from '@testing-library/react';
import { Counter } from './Counter';

describe('Counter', () => {
  it('renders with initial count', () => {
    render(<Counter initialCount={5} />);
    expect(screen.getByTestId('count-display')).toHaveTextContent('5');
  });

  it('increments count when increment button is clicked', () => {
    render(<Counter initialCount={0} />);
    
    fireEvent.click(screen.getByTestId('increment-btn'));
    
    expect(screen.getByTestId('count-display')).toHaveTextContent('1');
  });

  it('decrements count when decrement button is clicked', () => {
    render(<Counter initialCount={5} />);
    
    fireEvent.click(screen.getByTestId('decrement-btn'));
    
    expect(screen.getByTestId('count-display')).toHaveTextContent('4');
  });

  it('calls onIncrement callback when incrementing', () => {
    const mockCallback = jest.fn();
    render(<Counter onIncrement={mockCallback} />);
    
    fireEvent.click(screen.getByTestId('increment-btn'));
    
    expect(mockCallback).toHaveBeenCalledTimes(1);
  });
});
```

```tsx
// hooks/useCounter.test.ts
import { renderHook, act } from '@testing-library/react';
import { useCounter } from './useCounter';

describe('useCounter', () => {
  it('initializes with default value', () => {
    const { result } = renderHook(() => useCounter());
    expect(result.current.count).toBe(0);
  });

  it('initializes with custom value', () => {
    const { result } = renderHook(() => useCounter(10));
    expect(result.current.count).toBe(10);
  });

  it('increments', () => {
    const { result } = renderHook(() => useCounter());
    
    act(() => {
      result.current.increment();
    });
    
    expect(result.current.count).toBe(1);
  });

  it('decrements', () => {
    const { result } = renderHook(() => useCounter(5));
    
    act(() => {
      result.current.decrement();
    });
    
    expect(result.current.count).toBe(4);
  });

  it('resets to initial value', () => {
    const { result } = renderHook(() => useCounter(5));
    
    act(() => {
      result.current.increment();
      result.current.increment();
      result.current.reset();
    });
    
    expect(result.current.count).toBe(5);
  });
});
```

### 案例 6：项目目录结构设计

```
src/
├── assets/                 # 静态资源
│   ├── images/
│   └── fonts/
├── components/              # 通用组件
│   ├── common/             # 基础组件（Button、Input、Modal）
│   ├── layout/             # 布局组件（Header、Footer、Sidebar）
│   └── business/           # 业务组件（UserCard、ProductList）
├── composables/            # 组合式函数（Vue风格的Hooks）
├── config/                 # 配置文件
│   ├── routes.ts
│   └── constants.ts
├── contexts/               # React Context
│   ├── AuthContext.tsx
│   └── ThemeContext.tsx
├── hooks/                  # 自定义 Hooks
│   ├── useAuth.ts
│   ├── usePagination.ts
│   └── useDebounce.ts
├── pages/                  # 页面组件
│   ├── Home/
│   │   ├── index.tsx
│   │   └── components/
│   ├── About/
│   └── Users/
├── services/               # API 服务层
│   ├── api.ts
│   ├── userService.ts
│   └── productService.ts
├── store/                  # 状态管理
│   ├── index.ts
│   └── slices/
├── styles/                 # 全局样式
│   ├── variables.css
│   └── global.css
├── types/                  # TypeScript 类型定义
│   ├── user.ts
│   └── api.ts
├── utils/                  # 工具函数
│   ├── format.ts
│   ├── validation.ts
│   └── storage.ts
└── App.tsx
```

### 案例 7：权限系统设计

```tsx
// contexts/PermissionContext.tsx
import { createContext, useContext, useState, useCallback, ReactNode } from 'react';

interface Permission {
  resource: string;
  actions: ('create' | 'read' | 'update' | 'delete')[];
}

interface User {
  id: string;
  name: string;
  roles: string[];
  permissions: Permission[];
}

interface PermissionContextValue {
  user: User | null;
  setUser: (user: User | null) => void;
  hasPermission: (resource: string, action: string) => boolean;
  hasRole: (role: string) => boolean;
}

const PermissionContext = createContext<PermissionContextValue | null>(null);

export function PermissionProvider({ children }: { children: ReactNode }) {
  const [user, setUser] = useState<User | null>(null);

  const hasPermission = useCallback((resource: string, action: string) => {
    if (!user) return false;
    if (user.roles.includes('admin')) return true;

    const permission = user.permissions.find(p => p.resource === resource);
    return permission?.actions.includes(action as any) ?? false;
  }, [user]);

  const hasRole = useCallback((role: string) => {
    return user?.roles.includes(role) ?? false;
  }, [user]);

  return (
    <PermissionContext.Provider value={{ user, setUser, hasPermission, hasRole }}>
      {children}
    </PermissionContext.Provider>
  );
}

export function usePermission() {
  const context = useContext(PermissionContext);
  if (!context) {
    throw new Error('usePermission 必须在 PermissionProvider 内使用');
  }
  return context;
}
```

```tsx
// components/PermissionGate.tsx
import { ReactNode } from 'react';
import { usePermission } from '../contexts/PermissionContext';

interface PermissionGateProps {
  resource: string;
  action: string;
  children: ReactNode;
  fallback?: ReactNode;
}

export function PermissionGate({ resource, action, children, fallback = null }: PermissionGateProps) {
  const { hasPermission } = usePermission();

  return hasPermission(resource, action) ? <>{children}</> : <>{fallback}</>;
}
```

```tsx
// 使用示例
function UserManagement() {
  const { hasPermission, hasRole } = usePermission();

  return (
    <div>
      <h1>用户管理</h1>

      {hasRole('admin') && <button>导出数据</button>}

      <PermissionGate resource="users" action="create">
        <button>添加用户</button>
      </PermissionGate>

      <PermissionGate resource="users" action="delete">
        <button>删除用户</button>
      </PermissionGate>

      <ul>
        {/* 用户列表 */}
      </ul>
    </div>
  );
}
```

## 下一阶段

完成高级进阶后，可以继续学习 [附加技能](./stage-5-bonus.md) 来扩展你的技术栈。
