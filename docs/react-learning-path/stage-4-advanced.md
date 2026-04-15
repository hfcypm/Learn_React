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

**详细概念：**

**1.1 React 渲染原理**

React 的渲染过程分为两个阶段：Render 阶段和 Commit 阶段。

Render 阶段（可中断）：
- 计算需要 DOM 更新的部分
- 创建新的 React Element 树
- 比较新旧 Element 树（Reconciliation/调和）
- 确定需要更新的最小集合
- 此阶段可能被高优先级任务中断

Commit 阶段（不可中断）：
- 将 Render 阶段的变更应用到 DOM
- 调用 componentDidMount/DidUpdate
- 执行 ref 的 attach/detach
- 此阶段必须完整执行

React 17 的并发模式改进：
- 使用 `startTransition` 标记非紧急更新
- 使用 `useDeferredValue` 延迟值更新
- 允许在渲染过程中被打断

**1.2 Fiber 架构**

Fiber 是 React 16 引入的核心架构改造，将之前的同步递归渲染改为链表结构的可中断渲染。

Fiber 的核心目标：
- **可中断**：将渲染工作拆分成小单元，可以暂停、恢复
- **优先级调度**：高优先级（如用户输入）可以打断低优先级（如数据列表渲染）
- **时间切片**：将工作分散到多帧，避免阻塞主线程

Fiber 节点结构：
- `type`：元素类型
- `key`：元素标识
- `child`：第一个子元素
- `sibling`：下一个兄弟元素
- `return`：父元素
- `memoizedProps`/`pendingProps`：属性状态

**1.3 调和（Reconciliation）机制**

调和是 React 比较新旧 Virtual DOM 树，决定如何高效更新真实 DOM 的算法过程。

Diffing 算法原则：
- **不同类型元素产生不同树**：`<div>` 变成 `<span>` 会触发完整重建
- **同类型元素只更新属性**：只更新变化的部分
- **子元素列表 Diff**：使用 key 作为标识，支持高效移动

Key 的重要性：
- 列表渲染时 key 帮助 React 识别元素身份
- 错误的 key（如 index）会导致性能问题和状态错乱
- 推荐使用唯一 ID 作为 key，避免使用数组索引

**1.4 性能分析与优化策略**

性能优化的核心是找到瓶颈并针对性解决。

React DevTools Profiler 使用：
- 录制渲染过程，分析每个组件的渲染时间
- 识别不必要的重渲染
- 分析组件树的结构

常见性能问题：
- **父组件重渲染导致子组件不必要重渲染**
- **组件内创建新的对象/函数导致子组件 props 变化**
- **大列表渲染没有优化，卡顿明显**
- **不必要的订阅和副作用清理**

优化工具：
- `React.memo`：缓存组件渲染结果
- `useMemo`：缓存计算结果
- `useCallback`：缓存回调函数
- `useTransition`/`useDeferredValue`：标记非紧急更新

**1.5 内存泄漏排查**

内存泄漏会导致应用越用越卡、占用内存持续增长。

常见内存泄漏场景：
- **未清理的定时器**：setInterval/setTimeout 未 clear
- **未取消的事件监听**：addEventListener 后未 removeEventListener
- **未取消的订阅**：useEffect 中订阅后未 unsubscribe
- **闭包引用**：长时间持有外部变量引用

排查工具：
- Chrome DevTools Memory 面板
- Heap Snapshot 分析
- Allocation Timeline
- Performance Monitor 实时监控

**1.6 首屏加载优化**

首屏性能直接影响用户体验和 SEO 效果。

核心指标：
- **FCP (First Contentful Paint)**：首次内容绘制
- **LCP (Largest Contentful Paint)**：最大内容绘制
- **TTI (Time to Interactive)**：可交互时间

优化策略：
- **代码分割**：按路由分割，首屏只加载必要代码
- **预加载/预取**：使用 `<link rel="preload">` 预加载关键资源
- **图片优化**：使用 WebP/AVIF、响应式图片、懒加载
- **骨架屏**：loading 状态使用骨架屏替代白屏
- **CDN 加速**：静态资源使用 CDN 分发
- **Gzip/Brotli 压缩**：减少传输体积

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

**详细概念：**

**2.1 表单高级方案**

复杂表单场景（动态字段、多步表单、实时校验）需要专业的表单解决方案。

React Hook Form 核心优势：
- **性能极佳**：不基于 uncontrolled inputs，重渲染次数最少
- **Typed Form**：完整的 TypeScript 支持
- **小巧轻量**：核心约 3KB
- **Yup/Zod 集成**：方便的定义复杂校验规则
- **Field Array**：支持动态增加/删除字段

Formik 特点：
- 最早的 React 表单库之一，社区成熟
- 学习曲线平缓，文档丰富
- 适合简单到中等复杂度表单

校验库选择：
- **Yup**：最流行的 schema 校验库，链式语法
- **Zod**：TypeScript 优先，类型推导更强
- ** Joi**：老牌校验库，功能全面

**2.2 表格虚拟化原理**

表格虚拟化是渲染大数据量列表的核心技术，只渲染可视区域内的行，大幅减少 DOM 节点数量。

核心原理：
- **可视区域计算**：知道容器高度，计算可见行范围
- **固定行高**：如果行高一致，计算更简单高效
- **滚动位置追踪**：滚动时实时计算可见范围
- **内容定位**：使用 absolute 或 transform 定位可见行

实现要点：
- 维护 scrollTop 状态
- 计算 startIndex = Math.floor(scrollTop / rowHeight)
- 计算 endIndex = startIndex + visibleCount + buffer
- 只渲染 [startIndex, endIndex] 范围内的数据
- 用 padding 或占位元素维持总高度

**2.3 无限滚动实现**

无限滚动允许用户持续滚动加载数据，无需分页切换。

实现方式：
- **滚动事件监听**：`onScroll` 事件判断是否滚动到底部
- **Intersection Observer**：观察底部加载指示器是否进入视口
- **阈值控制**：快到到底时提前触发加载，避免白屏

性能考虑：
- 数据量持续增长可能导致内存问题，需要实现数据淘汰策略
- 使用防抖处理滚动事件，避免过度触发
- 加载状态和没有更多数据的提示要明确

**2.4 微前端架构**

微前端是将微服务思想应用于前端的技术方案，实现大型应用的独立开发、部署和技术栈无关。

核心价值：
- **独立部署**：各子应用独立部署，不影响主应用
- **技术无关**：可以用 React/Vue/Angular 混搭
- **团队自治**：各团队负责自己的应用，独立开发
- **增量升级**：可以逐步迁移旧项目

qiankun 核心 API：
- `registerMicroApps`：注册子应用
- `start`：启动 qiankun
- `loadMicroApp`：手动加载子应用
- `initGlobalState`：主子应用通信

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

**详细概念：**

**3.1 Next.js 核心优势**

Next.js 是 React 官方推荐的元框架，提供了完整的 SSR/SSG/ISR 解决方案，是企业级 React 应用的首选。

核心渲染模式：
- **SSG（Static Site Generation）**：构建时生成静态 HTML，适合内容固定不变的页面，性能最优
- **SSR（Server-Side Rendering）**：请求时动态生成 HTML，适合个性化内容
- **ISR（Incremental Static Regeneration）**：定期重新生成静态页面，适合内容频繁变化但不需要实时的场景

App Router vs Pages Router：
- **Pages Router**：更成熟，兼容性更好，约定式路由
- **App Router**：更现代，React Server Components 支持，嵌套布局更灵活

**3.2 服务端数据获取策略**

服务端数据获取是 SSR/SSG 的核心，不同场景需要不同策略。

Pages Router 数据获取：
- `getStaticProps`：构建时获取数据，用于 SSG
- `getStaticPaths`：生成静态路径列表，用于 SSG 动态路由
- `getServerSideProps`：每个请求时获取数据，用于 SSR

App Router 数据获取：
- `fetch()` API 直接使用，配置 cache 和 revalidate
- React 的 `use()` hook 支持在组件中调用异步数据
- Route Handlers（app/api/*）处理 API 请求

数据获取的最佳实践：
- 需要 SEO 的页面使用 SSG
- 个性化内容使用 SSR
- 频繁更新但不要求实时的内容使用 ISR

**3.3 SEO 优化核心要素**

SEO（Search Engine Optimization）是提升网站在搜索引擎中排名的技术。

Next.js SEO 支持：
- `generateMetadata`：生成动态 meta 标签
- 自动生成 sitemap.xml
- 自动生成 robots.txt
- Open Graph 和 Twitter Card 支持

核心 SEO 要素：
- **Title 和 Description**：每个页面有独特、准确的描述
- **结构化数据**：JSON-LD 格式帮助搜索引擎理解内容
- **语义化 HTML**：使用正确的标签（header、main、article 等）
- **图片 Alt 文本**：所有图片有描述性 alt 属性
- **页面性能**：Core Web Vitals 影响搜索排名

**3.4 路由方案**

Next.js 提供了灵活的文件系统路由。

Pages Router 约定：
- `pages/index.tsx` → `/`
- `pages/about.tsx` → `/about`
- `pages/posts/[id].tsx` → `/posts/1`, `/posts/2` ...
- `pages/api/users.ts` → `/api/users`

App Router 约定：
- `app/page.tsx` → `/`
- `app/about/page.tsx` → `/about`
- `app/posts/[id]/page.tsx` → `/posts/1` ...
- `app/api/users/route.ts` → `/api/users`

路由组和布局：
- `(group)` 语法用于组织代码但不影响 URL
- `layout.tsx` 定义嵌套布局
- `template.tsx` 每次路由切换重新创建

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

**详细概念：**

**4.1 测试策略分层**

测试是保障代码质量的重要手段，需要分层构建测试策略。

测试金字塔：
- **单元测试（Unit Tests）**：测试最小单位（函数、组件），数量最多，执行最快
- **集成测试（Integration Tests）**：测试模块间的协作，如组件与服务
- **端到端测试（E2E Tests）**：模拟真实用户操作，覆盖核心流程

测试覆盖率：
- 覆盖率不是越高越好，重点测试核心逻辑和边界情况
- 业务复杂度高的地方需要更多测试
- 通用组件库需要高覆盖率的单元测试

**4.2 React Testing Library 设计理念**

React Testing Library（RTL）是 React 官方推荐的测试库，它的设计理念是"以用户的方式测试 UI"。

核心理念：
- **不测试实现细节**：不直接测试组件的 state、props、生命周期
- **通过 DOM 交互测试**：模拟用户如何与页面交互
- **Queries 优先级**：`getByRole` > `getByLabelText` > `getByText` > `getByTestId`

常用 Queries：
- `getByRole`：最接近用户视角，推荐优先使用
- `getByLabelText`：测试表单输入
- `getByText`：查找文本内容
- `getByTestId`：最后手段，不推荐用于业务逻辑测试

**4.3 Jest 测试框架**

Jest 是 Facebook 开发的测试框架，零配置、内置 Mock、快照测试等功能。

核心概念：
- **describe**：组织测试用例的块
- **it/test**：单个测试用例
- **expect**：断言函数
- **beforeEach/afterEach**：每个测试前后的钩子

Mock 机制：
- `jest.fn()`：创建模拟函数
- `jest.mock()`：模拟整个模块
- `jest.spyOn()`：监视对象方法

**4.4 自动化部署**

自动化部署是现代开发流程的标配，确保代码经过 CI/CD 流程自动部署到目标环境。

CI/CD 流程：
- **持续集成（CI）**：代码提交后自动运行测试、构建
- **持续部署（CD）**：CI 通过后自动部署到环境

主流部署平台：
- **Vercel**：Next.js 官方托管平台，零配置
- **Netlify**：静态站点和 SSR 都能托管
- **Railway/Render**：现代 PaaS 平台
- **阿里云/腾讯云**：国内企业常用

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

**详细概念：**

**5.1 项目目录结构设计**

良好的目录结构是大型项目可维护性的基础，需要考虑团队规模、技术栈、业务复杂度。

目录组织原则：
- **按职责分**：组件、hooks、工具、类型等分开
- **按特性/业务分**：大型项目按功能模块组织
- **就近原则**：组件的样式、测试文件与组件放一起
- **扁平优先**：避免过深的嵌套

常见目录模式：
- **Feature-Based**：按业务功能组织（user/、product/、order/）
- **Layer-Based**：按技术层级组织（components/、hooks/、services/）
- **混合模式**：团队项目常用，核心代码按层级，业务代码按特性

**5.2 公共逻辑封装**

公共逻辑封装减少重复代码，提高代码复用性。

封装层次：
- **工具函数**：纯函数，如日期格式化、金额格式化
- **自定义 Hooks**：有状态逻辑的封装，如 useDebounce、useLocalStorage
- **高阶组件（HOC）**：逻辑复用，已逐渐被 Hooks 替代
- **Render Props**：逻辑分发模式

自定义 Hooks 设计原则：
- 单一职责：一个 Hook 只做一件事
- 命名规范：`use` 前缀
- 返回值明确：类型安全，语义清晰
- 文档完善：说明参数、返回值、使用场景

**5.3 权限系统设计**

权限系统控制用户能访问哪些功能和数据，是企业应用的核心系统。

权限模型：
- **RBAC（Role-Based Access Control）**：基于角色的权限控制
- **ABAC（Attribute-Based Access Control）**：基于属性的权限控制

常见权限设计：
- **页面级权限**：用户是否能看到某个页面
- **操作级权限**：用户是否能进行某个操作（新增、编辑、删除）
- **数据级权限**：用户只能看到自己有权限的数据

实现方案：
- **权限组件**：`PermissionGate` 根据权限显示/隐藏组件
- **权限指令**：路由守卫检查权限
- **API 级权限**：后端也要做权限校验

**5.4 国际化（i18n）方案**

国际化让应用支持多语言，服务全球用户。

i18n 核心概念：
- **Key-Value 映射**：使用 key 代替硬编码文本
- **复数处理**：不同语言的复数规则不同
- **日期/货币格式化**：不同地区格式不同
- **RTL 支持**：阿拉伯语等从右到左的语言

主流 i18n 库：
- **react-i18next**：最流行的 React i18n 方案，功能完善
- **react-intl**：FormatJS 的一部分，早期方案
- **lingui**：编译时优化，性能好

国际化最佳实践：
- 所有用户可见文本都要国际化
- 避免字符串拼接，使用模板语法
- 保持 key 的语义化，如 `user.login.button`

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
