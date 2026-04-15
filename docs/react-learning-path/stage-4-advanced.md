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

**经典案例：性能分析与优化**

```tsx
// PerformanceMonitor.tsx - 性能监控组件
import { useEffect, useRef } from 'react';

function PerformanceMonitor({ children }: { children: React.ReactNode }) {
  const lastRenderTime = useRef<number>(Date.now());
  const renderCount = useRef<number>(0);

  useEffect(() => {
    renderCount.current += 1;
    const now = Date.now();
    const diff = now - lastRenderTime.current;
    lastRenderTime.current = now;

    if (diff > 16) {
      console.warn(`渲染延迟: ${diff}ms, 渲染次数: ${renderCount.current}`);
    }

    if (performance && performance.memory) {
      console.log(`内存使用: ${(performance.memory.usedJSHeapSize / 1024 / 1024).toFixed(2)} MB`);
    }
  });

  return <>{children}</>;
}
```

```tsx
// OptimizedList.tsx - 列表优化
import { memo, useMemo } from 'react';

interface ListItem {
  id: string;
  title: string;
  description: string;
}

interface OptimizedListProps {
  items: ListItem[];
  onItemClick?: (item: ListItem) => void;
}

const ListItemComponent = memo(function ListItemComponent({
  item,
  onClick,
}: {
  item: ListItem;
  onClick?: () => void;
}) {
  return (
    <div onClick={onClick} style={{ padding: 16, borderBottom: '1px solid #eee' }}>
      <h3>{item.title}</h3>
      <p>{item.description}</p>
    </div>
  );
});

function OptimizedList({ items, onItemClick }: OptimizedListProps) {
  const sortedItems = useMemo(() => {
    return [...items].sort((a, b) => a.title.localeCompare(b.title));
  }, [items]);

  const handleClick = useMemo(() => {
    return (item: ListItem) => () => onItemClick?.(item);
  }, [onItemClick]);

  return (
    <div>
      {sortedItems.map(item => (
        <ListItemComponent
          key={item.id}
          item={item}
          onClick={handleClick(item)}
        />
      ))}
    </div>
  );
}
```

```tsx
// ImageOptimization.tsx - 图片优化
import { lazy, Suspense } from 'react';

function ImageWithLazyLoad({ src, alt }: { src: string; alt: string }) {
  return (
    <Suspense fallback={<div>图片加载中...</div>}>
      <img
        src={src}
        alt={alt}
        loading="lazy"
        decoding="async"
        style={{ maxWidth: '100%', height: 'auto' }}
      />
    </Suspense>
  );
}

function Avatar({ src, size = 48 }: { src?: string; size?: number }) {
  const placeholder = `data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' width='${size}' height='${size}'%3E%3Ccircle cx='50%25' cy='50%25' r='50%25' fill='%23ddd'/%3E%3C/svg%3E`;

  return (
    <img
      src={src || placeholder}
      alt="avatar"
      width={size}
      height={size}
      style={{
        borderRadius: '50%',
        objectFit: 'cover',
      }}
    />
  );
}
```

---

### 2. 复杂场景解决方案

- [ ] 表单高阶方案（React Hook Form、Formik）
- [ ] 表格虚拟化
- [ ] 无限滚动
- [ ] 大数据渲染
- [ ] 微前端（qiankun）
- [ ] 跨应用集成

**经典案例：React Hook Form**

```tsx
// AdvancedForm.tsx
import { useForm, useFieldArray, Controller } from 'react-hook-form';
import { yupResolver } from '@hookform/resolvers/yup';
import * as yup from 'yup';

interface FormData {
  name: string;
  email: string;
  passwords: {
    password: string;
    confirmPassword: string;
  }[];
  hobbies: string[];
  country: string;
}

const schema = yup.object({
  name: yup
    .string()
    .required('姓名不能为空')
    .min(2, '姓名至少2个字符'),
  email: yup
    .string()
    .required('邮箱不能为空')
    .email('请输入有效邮箱'),
  passwords: yup.array().of(
    yup.object({
      password: yup
        .string()
        .required('密码不能为空')
        .min(8, '密码至少8位'),
      confirmPassword: yup
        .string()
        .required('请确认密码')
        .oneOf([yup.ref('password')], '两次密码不一致'),
    })
  ),
  hobbies: yup
    .array()
    .of(yup.string())
    .min(1, '请至少选择一个爱好'),
  country: yup.string().required('请选择国家'),
});

function AdvancedForm() {
  const {
    register,
    handleSubmit,
    control,
    formState: { errors, isSubmitting, isDirty },
    watch,
  } = useForm<FormData>({
    resolver: yupResolver(schema),
    defaultValues: {
      name: '',
      email: '',
      passwords: [{ password: '', confirmPassword: '' }],
      hobbies: [],
      country: '',
    },
    mode: 'onChange',
  });

  const { fields, append, remove } = useFieldArray({
    control,
    name: 'passwords',
  });

  const watchPasswords = watch('passwords');

  const onSubmit = async (data: FormData) => {
    await new Promise(resolve => setTimeout(resolve, 1000));
    console.log('表单数据：', data);
    alert('提交成功！');
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <div>
        <label>姓名</label>
        <input {...register('name')} />
        {errors.name && <p>{errors.name.message}</p>}
      </div>

      <div>
        <label>邮箱</label>
        <input type="email" {...register('email')} />
        {errors.email && <p>{errors.email.message}</p>}
      </div>

      <div>
        <label>密码组</label>
        {fields.map((field, index) => (
          <div key={field.id}>
            <input
              type="password"
              placeholder="密码"
              {...register(`passwords.${index}.password`)}
            />
            {errors.passwords?.[index]?.password && (
              <p>{errors.passwords[index].password?.message}</p>
            )}
            <input
              type="password"
              placeholder="确认密码"
              {...register(`passwords.${index}.confirmPassword`)}
            />
            {errors.passwords?.[index]?.confirmPassword && (
              <p>{errors.passwords[index].confirmPassword?.message}</p>
            )}
            {index > 0 && (
              <button type="button" onClick={() => remove(index)}>
                删除
              </button>
            )}
          </div>
        ))}
        <button
          type="button"
          onClick={() => append({ password: '', confirmPassword: '' })}
        >
          添加密码
        </button>
      </div>

      <div>
        <label>爱好</label>
        <label>
          <input type="checkbox" value="reading" {...register('hobbies')} />
          阅读
        </label>
        <label>
          <input type="checkbox" value="sports" {...register('hobbies')} />
          运动
        </label>
        <label>
          <input type="checkbox" value="music" {...register('hobbies')} />
          音乐
        </label>
        {errors.hobbies && <p>{errors.hobbies.message}</p>}
      </div>

      <div>
        <label>国家</label>
        <select {...register('country')}>
          <option value="">请选择</option>
          <option value="cn">中国</option>
          <option value="us">美国</option>
          <option value="jp">日本</option>
        </select>
        {errors.country && <p>{errors.country.message}</p>}
      </div>

      <button type="submit" disabled={!isDirty || isSubmitting}>
        {isSubmitting ? '提交中...' : '提交'}
      </button>
    </form>
  );
}
```

**经典案例：无限滚动**

```tsx
// InfiniteScroll.tsx
import { useState, useEffect, useRef, useCallback } from 'react';

interface Post {
  id: string;
  title: string;
  body: string;
}

function InfiniteScroll() {
  const [posts, setPosts] = useState<Post[]>([]);
  const [page, setPage] = useState(1);
  const [loading, setLoading] = useState(false);
  const [hasMore, setHasMore] = useState(true);
  const observerRef = useRef<HTMLDivElement>(null);

  const loadMorePosts = useCallback(async () => {
    if (loading || !hasMore) return;

    setLoading(true);
    try {
      const response = await fetch(
        `https://jsonplaceholder.typicode.com/posts?_limit=10&_page=${page}`
      );
      const newPosts = await response.json();

      if (newPosts.length === 0) {
        setHasMore(false);
      } else {
        setPosts(prev => [...prev, ...newPosts]);
        setPage(prev => prev + 1);
      }
    } catch (error) {
      console.error('加载失败：', error);
    } finally {
      setLoading(false);
    }
  }, [page, loading, hasMore]);

  useEffect(() => {
    const observer = new IntersectionObserver(
      (entries) => {
        if (entries[0].isIntersecting) {
          loadMorePosts();
        }
      },
      { threshold: 0.1 }
    );

    if (observerRef.current) {
      observer.observe(observerRef.current);
    }

    return () => observer.disconnect();
  }, [loadMorePosts]);

  useEffect(() => {
    loadMorePosts();
  }, []);

  return (
    <div>
      <h1>无限滚动帖子列表</h1>

      <div>
        {posts.map(post => (
          <div
            key={post.id}
            style={{
              padding: 16,
              margin: 8,
              border: '1px solid #ddd',
              borderRadius: 8,
            }}
          >
            <h3>{post.title}</h3>
            <p>{post.body}</p>
          </div>
        ))}
      </div>

      <div ref={observerRef}>
        {loading && <p>加载中...</p>}
        {!hasMore && <p>没有更多了</p>}
      </div>
    </div>
  );
}
```

**经典案例：虚拟列表**

```tsx
// VirtualTable.tsx
import { useState, useRef, useMemo, useCallback } from 'react';

interface TableColumn<T> {
  key: keyof T;
  title: string;
  width: string;
  render?: (value: T[keyof T], record: T) => React.ReactNode;
}

interface VirtualTableProps<T> {
  data: T[];
  columns: TableColumn<T>[];
  rowKey: keyof T;
  height?: number;
  rowHeight?: number;
}

function VirtualTable<T extends Record<string, unknown>>({
  data,
  columns,
  rowKey,
  height = 400,
  rowHeight = 50,
}: VirtualTableProps<T>) {
  const [scrollTop, setScrollTop] = useState(0);
  const containerRef = useRef<HTMLDivElement>(null);

  const visibleCount = Math.ceil(height / rowHeight);
  const startIndex = Math.floor(scrollTop / rowHeight);
  const endIndex = Math.min(startIndex + visibleCount + 2, data.length);

  const visibleRows = useMemo(() => {
    return data.slice(startIndex, endIndex).map((record, index) => ({
      record,
      index: startIndex + index,
    }));
  }, [data, startIndex, endIndex]);

  const totalHeight = data.length * rowHeight;

  const handleScroll = useCallback((e: React.UIEvent<HTMLDivElement>) => {
    setScrollTop(e.target.scrollTop);
  }, []);

  return (
    <div
      ref={containerRef}
      style={{ height, overflow: 'auto' }}
      onScroll={handleScroll}
    >
      <div style={{ height: totalHeight, position: 'relative' }}>
        <table style={{ width: '100%', borderCollapse: 'collapse' }}>
          <thead>
            <tr>
              {columns.map(col => (
                <th
                  key={String(col.key)}
                  style={{
                    width: col.width,
                    position: 'sticky',
                    top: 0,
                    background: '#f5f5f5',
                    padding: 12,
                    borderBottom: '2px solid #ddd',
                  }}
                >
                  {col.title}
                </th>
              ))}
            </tr>
          </thead>
          <tbody>
            {visibleRows.map(({ record, index }) => (
              <tr
                key={String(record[rowKey]) || index}
                style={{ height: rowHeight }}
              >
                {columns.map(col => (
                  <td
                    key={String(col.key)}
                    style={{ padding: 12, borderBottom: '1px solid #eee' }}
                  >
                    {col.render
                      ? col.render(record[col.key], record)
                      : String(record[col.key])}
                  </td>
                ))}
              </tr>
            ))}
          </tbody>
        </table>
      </div>
    </div>
  );
}
```

---

### 3. 服务端渲染/静态站点（SSR/SSG）

- [ ] Next.js（React 官方推荐框架）核心用法
- [ ] 服务端数据获取
- [ ] SEO 优化
- [ ] 路由方案

**经典案例：Next.js Pages Router**

```tsx
// pages/index.tsx
import { GetStaticProps, InferGetStaticPropsType } from 'next';
import Link from 'next/link';

interface Post {
  id: string;
  title: string;
  excerpt: string;
  publishedAt: string;
}

interface Props {
  posts: Post[];
}

export const getStaticProps: GetStaticProps<Props> = async () => {
  const response = await fetch('https://api.example.com/posts');
  const posts = await response.json();

  return {
    props: { posts },
    revalidate: 60,
  };
};

export default function HomePage({ posts }: InferGetStaticPropsType<typeof getStaticProps>) {
  return (
    <div>
      <h1>博客首页</h1>

      <div>
        {posts.map(post => (
          <article key={post.id}>
            <Link href={`/posts/${post.id}`}>
              <h2>{post.title}</h2>
            </Link>
            <p>{post.excerpt}</p>
            <time>{post.publishedAt}</time>
          </article>
        ))}
      </div>
    </div>
  );
}
```

```tsx
// pages/posts/[id].tsx
import { GetStaticPaths, GetStaticProps, InferGetStaticPropsType } from 'next';
import { ParsedUrlQuery } from 'querystring';

interface Post {
  id: string;
  title: string;
  content: string;
  author: {
    name: string;
    avatar: string;
  };
  tags: string[];
  publishedAt: string;
}

interface Params extends ParsedUrlQuery {
  id: string;
}

type Props = {
  post: Post;
};

export const getStaticPaths: GetStaticPaths<Params> = async () => {
  const response = await fetch('https://api.example.com/posts');
  const posts = await response.json();

  return {
    paths: posts.map((post: Post) => ({ params: { id: post.id } })),
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
        <div>
          <img src={post.author.avatar} alt={post.author.name} />
          <span>{post.author.name}</span>
          <time>{post.publishedAt}</time>
        </div>
      </header>

      <div>
        {post.tags.map(tag => (
          <span key={tag}>{tag}</span>
        ))}
      </div>

      <main dangerouslySetInnerHTML={{ __html: post.content }} />

      <nav>
        <Link href="/">返回首页</Link>
      </nav>
    </article>
  );
}
```

**经典案例：Next.js App Router**

```tsx
// app/page.tsx
import { getPosts } from '@/lib/api';

export default async function HomePage() {
  const posts = await getPosts();

  return (
    <div>
      <h1>博客首页</h1>
      <div>
        {posts.map(post => (
          <article key={post.id}>
            <h2>{post.title}</h2>
            <p>{post.excerpt}</p>
          </article>
        ))}
      </div>
    </div>
  );
}
```

```tsx
// app/posts/[id]/page.tsx
import { getPostById, getAllPostIds } from '@/lib/api';

interface PageProps {
  params: Promise<{ id: string }>;
}

export async function generateStaticParams() {
  const ids = await getAllPostIds();
  return ids.map((id) => ({ id }));
}

export async function generateMetadata({ params }: PageProps) {
  const { id } = await params;
  const post = await getPostById(id);

  return {
    title: post.title,
    description: post.excerpt,
    openGraph: {
      title: post.title,
      images: [post.coverImage],
    },
  };
}

export default async function PostPage({ params }: PageProps) {
  const { id } = await params;
  const post = await getPostById(id);

  return (
    <article>
      <h1>{post.title}</h1>
      <div dangerouslySetInnerHTML={{ __html: post.content }} />
    </article>
  );
}
```

---

### 4. 测试与部署

- [ ] React 组件单元测试（Jest + React Testing Library）
- [ ] E2E 测试
- [ ] 自动化部署

**经典案例：组件测试**

```tsx
// components/LoginForm.tsx
import { useState } from 'react';

interface LoginFormProps {
  onSubmit: (username: string, password: string) => Promise<void>;
}

export function LoginForm({ onSubmit }: LoginFormProps) {
  const [username, setUsername] = useState('');
  const [password, setPassword] = useState('');
  const [error, setError] = useState('');
  const [isLoading, setIsLoading] = useState(false);

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    setError('');

    if (!username) {
      setError('用户名不能为空');
      return;
    }
    if (!password) {
      setError('密码不能为空');
      return;
    }

    setIsLoading(true);
    try {
      await onSubmit(username, password);
    } catch (err) {
      setError(err instanceof Error ? err.message : '登录失败');
    } finally {
      setIsLoading(false);
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <div>
        <label htmlFor="username">用户名</label>
        <input
          id="username"
          data-testid="username-input"
          value={username}
          onChange={(e) => setUsername(e.target.value)}
          disabled={isLoading}
        />
      </div>

      <div>
        <label htmlFor="password">密码</label>
        <input
          id="password"
          type="password"
          data-testid="password-input"
          value={password}
          onChange={(e) => setPassword(e.target.value)}
          disabled={isLoading}
        />
      </div>

      {error && (
        <p role="alert" data-testid="error-message">
          {error}
        </p>
      )}

      <button type="submit" disabled={isLoading} data-testid="submit-button">
        {isLoading ? '登录中...' : '登录'}
      </button>
    </form>
  );
}
```

```tsx
// components/LoginForm.test.tsx
import { render, screen, fireEvent, waitFor } from '@testing-library/react';
import { LoginForm } from './LoginForm';

describe('LoginForm', () => {
  it('renders login form correctly', () => {
    render(<LoginForm onSubmit={async () => {}} />);

    expect(screen.getByTestId('username-input')).toBeInTheDocument();
    expect(screen.getByTestId('password-input')).toBeInTheDocument();
    expect(screen.getByTestId('submit-button')).toBeInTheDocument();
  });

  it('shows error when username is empty', async () => {
    render(<LoginForm onSubmit={async () => {}} />);

    fireEvent.change(screen.getByTestId('password-input'), {
      target: { value: 'password123' },
    });

    fireEvent.click(screen.getByTestId('submit-button'));

    expect(await screen.findByTestId('error-message')).toHaveTextContent('用户名不能为空');
  });

  it('calls onSubmit with correct values', async () => {
    const mockSubmit = jest.fn().mockResolvedValue(undefined);

    render(<LoginForm onSubmit={mockSubmit} />);

    fireEvent.change(screen.getByTestId('username-input'), {
      target: { value: 'testuser' },
    });
    fireEvent.change(screen.getByTestId('password-input'), {
      target: { value: 'password123' },
    });

    fireEvent.click(screen.getByTestId('submit-button'));

    await waitFor(() => {
      expect(mockSubmit).toHaveBeenCalledWith('testuser', 'password123');
    });
  });

  it('displays loading state while submitting', async () => {
    const mockSubmit = jest.fn().mockImplementation(
      () => new Promise(resolve => setTimeout(resolve, 100))
    );

    render(<LoginForm onSubmit={mockSubmit} />);

    fireEvent.change(screen.getByTestId('username-input'), {
      target: { value: 'testuser' },
    });
    fireEvent.change(screen.getByTestId('password-input'), {
      target: { value: 'password123' },
    });

    fireEvent.click(screen.getByTestId('submit-button'));

    expect(screen.getByTestId('submit-button')).toBeDisabled();
    expect(screen.getByText('登录中...')).toBeInTheDocument();
  });

  it('displays error message when login fails', async () => {
    const mockSubmit = jest.fn().mockRejectedValue(new Error('用户名或密码错误'));

    render(<LoginForm onSubmit={mockSubmit} />);

    fireEvent.change(screen.getByTestId('username-input'), {
      target: { value: 'testuser' },
    });
    fireEvent.change(screen.getByTestId('password-input'), {
      target: { value: 'wrongpassword' },
    });

    fireEvent.click(screen.getByTestId('submit-button'));

    expect(await screen.findByTestId('error-message')).toHaveTextContent('用户名或密码错误');
  });
});
```

**经典案例：Hooks 测试**

```tsx
// hooks/useCounter.ts
import { useState, useCallback } from 'react';

interface UseCounterOptions {
  initialValue?: number;
  min?: number;
  max?: number;
}

interface UseCounterReturn {
  count: number;
  increment: () => void;
  decrement: () => void;
  reset: () => void;
  set: (value: number) => void;
}

export function useCounter({
  initialValue = 0,
  min = -Infinity,
  max = Infinity,
}: UseCounterOptions = {}): UseCounterReturn {
  const [count, setCount] = useState(initialValue);

  const increment = useCallback(() => {
    setCount(prev => Math.min(prev + 1, max));
  }, [max]);

  const decrement = useCallback(() => {
    setCount(prev => Math.max(prev - 1, min));
  }, [min]);

  const reset = useCallback(() => {
    setCount(initialValue);
  }, [initialValue]);

  const set = useCallback((value: number) => {
    setCount(Math.max(min, Math.min(max, value)));
  }, [min, max]);

  return { count, increment, decrement, reset, set };
}
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
    const { result } = renderHook(() => useCounter({ initialValue: 10 }));
    expect(result.current.count).toBe(10);
  });

  it('increments count', () => {
    const { result } = renderHook(() => useCounter());

    act(() => {
      result.current.increment();
    });

    expect(result.current.count).toBe(1);
  });

  it('decrements count', () => {
    const { result } = renderHook(() => useCounter({ initialValue: 5 }));

    act(() => {
      result.current.decrement();
    });

    expect(result.current.count).toBe(4);
  });

  it('resets count to initial value', () => {
    const { result } = renderHook(() => useCounter({ initialValue: 5 }));

    act(() => {
      result.current.increment();
      result.current.increment();
      result.current.reset();
    });

    expect(result.current.count).toBe(5);
  });

  it('respects min boundary', () => {
    const { result } = renderHook(() => useCounter({ initialValue: 0, min: 0 }));

    act(() => {
      result.current.decrement();
    });

    expect(result.current.count).toBe(0);
  });

  it('respects max boundary', () => {
    const { result } = renderHook(() => useCounter({ initialValue: 5, max: 5 }));

    act(() => {
      result.current.increment();
    });

    expect(result.current.count).toBe(5);
  });
});
```

---

### 5. 架构设计

- [ ] 项目目录结构设计、模块化拆分
- [ ] 公共逻辑封装
-  [ ] 通用组件/Hooks 抽离
- [ ] 权限系统
- [ ] 国际化（i18n）方案

**经典案例：项目目录结构**

```
src/
├── assets/                 # 静态资源
│   ├── images/
│   │   ├── logo.png
│   │   └── icons/
│   └── fonts/
├── components/             # 通用组件
│   ├── common/            # 基础组件
│   │   ├── Button/
│   │   │   ├── Button.tsx
│   │   │   ├── Button.module.css
│   │   │   └── Button.test.tsx
│   │   ├── Input/
│   │   ├── Modal/
│   │   └── Table/
│   ├── layout/            # 布局组件
│   │   ├── Header/
│   │   ├── Footer/
│   │   └── Sidebar/
│   └── business/           # 业务组件
│       ├── UserCard/
│       └── ProductList/
├── contexts/               # React Context
│   ├── AuthContext.tsx
│   ├── ThemeContext.tsx
│   └── PermissionContext.tsx
├── hooks/                  # 自定义 Hooks
│   ├── useAuth.ts
│   ├── usePermission.ts
│   ├── usePagination.ts
│   ├── useDebounce.ts
│   └── useLocalStorage.ts
├── pages/                  # 页面组件
│   ├── Home/
│   │   ├── index.tsx
│   │   └── components/
│   ├── About/
│   ├── Users/
│   │   ├── index.tsx
│   │   ├── UserList.tsx
│   │   ├── UserDetail.tsx
│   │   └── UserForm.tsx
│   └── Products/
├── services/               # API 服务层
│   ├── api.ts
│   ├── userService.ts
│   ├── productService.ts
│   └── orderService.ts
├── store/                  # 状态管理
│   ├── index.ts
│   ├── hooks.ts
│   └── slices/
│       ├── userSlice.ts
│       └── cartSlice.ts
├── styles/                 # 全局样式
│   ├── variables.css
│   ├── global.css
│   └── reset.css
├── types/                  # TypeScript 类型定义
│   ├── user.ts
│   ├── product.ts
│   └── api.ts
├── utils/                  # 工具函数
│   ├── format.ts
│   ├── validation.ts
│   ├── storage.ts
│   └── constants.ts
├── App.tsx
├── main.tsx
└── routes.tsx
```

**经典案例：权限系统**

```tsx
// contexts/PermissionContext.tsx
import { createContext, useContext, useState, useCallback, ReactNode } from 'react';

export type PermissionAction = 'create' | 'read' | 'update' | 'delete';
export type PermissionResource = 'users' | 'products' | 'orders' | 'settings';

interface Permission {
  resource: PermissionResource;
  actions: PermissionAction[];
}

interface User {
  id: string;
  name: string;
  email: string;
  roles: string[];
  permissions: Permission[];
}

interface PermissionContextValue {
  user: User | null;
  setUser: (user: User | null) => void;
  hasPermission: (resource: PermissionResource, action: PermissionAction) => boolean;
  hasRole: (role: string) => boolean;
  isLoading: boolean;
}

const PermissionContext = createContext<PermissionContextValue | null>(null);

export function PermissionProvider({ children }: { children: ReactNode }) {
  const [user, setUser] = useState<User | null>(null);
  const [isLoading, setIsLoading] = useState(false);

  const hasPermission = useCallback(
    (resource: PermissionResource, action: PermissionAction) => {
      if (!user) return false;
      if (user.roles.includes('admin')) return true;

      const permission = user.permissions.find(p => p.resource === resource);
      return permission?.actions.includes(action) ?? false;
    },
    [user]
  );

  const hasRole = useCallback(
    (role: string) => {
      return user?.roles.includes(role) ?? false;
    },
    [user]
  );

  const login = useCallback(async (email: string, password: string) => {
    setIsLoading(true);
    try {
      const response = await fetch('/api/auth/login', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ email, password }),
      });

      if (!response.ok) throw new Error('登录失败');

      const userData = await response.json();
      setUser(userData);
    } finally {
      setIsLoading(false);
    }
  }, []);

  const logout = useCallback(() => {
    setUser(null);
  }, []);

  return (
    <PermissionContext.Provider
      value={{ user, setUser, hasPermission, hasRole, isLoading }}
    >
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
import { usePermission, PermissionResource, PermissionAction } from '../contexts/PermissionContext';

interface PermissionGateProps {
  resource: PermissionResource;
  action: PermissionAction;
  children: ReactNode;
  fallback?: ReactNode;
}

export function PermissionGate({
  resource,
  action,
  children,
  fallback = null,
}: PermissionGateProps) {
  const { hasPermission } = usePermission();

  return hasPermission(resource, action) ? <>{children}</> : <>{fallback}</>;
}
```

```tsx
// components/RoleGate.tsx
import { ReactNode } from 'react';
import { usePermission } from '../contexts/PermissionContext';

interface RoleGateProps {
  role: string;
  children: ReactNode;
  fallback?: ReactNode;
}

export function RoleGate({ role, children, fallback = null }: RoleGateProps) {
  const { hasRole } = usePermission();

  return hasRole(role) ? <>{children}</> : <>{fallback}</>;
}
```

```tsx
// pages/UserManagement.tsx
import { usePermission } from '../contexts/PermissionContext';
import { PermissionGate } from '../components/PermissionGate';
import { RoleGate } from '../components/RoleGate';

function UserManagement() {
  const { hasPermission, hasRole } = usePermission();

  return (
    <div>
      <h1>用户管理</h1>

      <RoleGate role="admin" fallback={<p>仅管理员可见</p>}>
        <button>导出用户数据</button>
      </RoleGate>

      <PermissionGate
        resource="users"
        action="create"
        fallback={<p>您没有创建用户的权限</p>}
      >
        <button>添加用户</button>
      </PermissionGate>

      <PermissionGate resource="users" action="delete">
        <button>删除用户</button>
      </PermissionGate>

      <PermissionGate resource="users" action="update">
        <button>编辑用户</button>
      </PermissionGate>

      <ul>
        {/* 用户列表 */}
      </ul>
    </div>
  );
}

export default UserManagement;
```

**经典案例：国际化（i18n）**

```tsx
// i18n/index.ts
import i18n from 'i18next';
import { initReactI18next } from 'react-i18next';

const resources = {
  en: {
    translation: {
      welcome: 'Welcome',
      login: 'Login',
      logout: 'Logout',
      username: 'Username',
      password: 'Password',
      submit: 'Submit',
      cancel: 'Cancel',
      delete: 'Delete',
      edit: 'Edit',
      save: 'Save',
      loading: 'Loading...',
      error: 'Error',
      success: 'Success',
    },
  },
  zh: {
    translation: {
      welcome: '欢迎',
      login: '登录',
      logout: '退出登录',
      username: '用户名',
      password: '密码',
      submit: '提交',
      cancel: '取消',
      delete: '删除',
      edit: '编辑',
      save: '保存',
      loading: '加载中...',
      error: '错误',
      success: '成功',
    },
  },
};

i18n.use(initReactI18next).init({
  resources,
  lng: 'zh',
  fallbackLng: 'en',
  interpolation: {
    escapeValue: false,
  },
});

export default i18n;
```

```tsx
// components/LanguageSwitcher.tsx
import { useTranslation } from 'react-i18next';

function LanguageSwitcher() {
  const { i18n } = useTranslation();

  const languages = [
    { code: 'zh', name: '中文' },
    { code: 'en', name: 'English' },
  ];

  return (
    <select
      value={i18n.language}
      onChange={(e) => i18n.changeLanguage(e.target.value)}
    >
      {languages.map((lang) => (
        <option key={lang.code} value={lang.code}>
          {lang.name}
        </option>
      ))}
    </select>
  );
}
```

```tsx
// pages/Login.tsx
import { useTranslation } from 'react-i18next';
import { LanguageSwitcher } from '../components/LanguageSwitcher';

function Login() {
  const { t } = useTranslation();

  return (
    <div>
      <h1>{t('welcome')}</h1>

      <LanguageSwitcher />

      <form>
        <div>
          <label>{t('username')}</label>
          <input type="text" />
        </div>

        <div>
          <label>{t('password')}</label>
          <input type="password" />
        </div>

        <button type="submit">{t('login')}</button>
        <button type="button">{t('cancel')}</button>
      </form>
    </div>
  );
}
```

---

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
- 列表使用虚拟滚动

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

## 下一阶段

完成高级进阶后，可以继续学习 [附加技能](./stage-5-bonus.md) 来扩展你的技术栈。
