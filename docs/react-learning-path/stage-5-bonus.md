# 阶段五：附加技能（加分项）

**目标**：扩展技术视野，提升竞争力

## 可选技能列表

### 1. 状态管理进阶

- [ ] Redux 源码思想
- [ ] MobX 响应式状态管理

### 2. 跨端技术

- [ ] React Native（移动端）
- [ ] Taro（小程序）

### 3. 构建工具深度配置

- [ ] Webpack 深度配置
- [ ] Vite 深度配置

### 4. React 18 新特性

- [ ] 并发渲染（Concurrent Rendering）
- [ ] Suspense 深入理解
- [ ] Server Components（服务端组件）

## 技能说明

### Redux 源码思想

理解 Redux 的核心概念：
- Store：单一数据源
- Action：描述操作的 plain object
- Reducer：纯函数，根据 action 计算新状态
- Middleware：扩展 Redux 能力（异步、日志等）

### MobX

```typescript
import { makeAutoObservable } from 'mobx';

class Counter {
  count = 0;

  constructor() {
    makeAutoObservable(this);
  }

  increment() {
    this.count++;
  }
}
```

### React Native

- 使用 React 语法开发原生 iOS/Android 应用
- 复用 React 组件化和状态管理思想
- 学习原生组件和布局系统

### Taro

- 一套代码，多端运行（微信小程序、H5、React Native）
- 遵循 React 语法规范

### 构建工具

**Webpack 重点**：
- Loader 和 Plugin 机制
- 模块联邦（Module Federation）
- 构建优化

**Vite 重点**：
- ESM 原理
- 依赖预构建
- 插件系统

## 经典学习案例

### 案例 1：MobX 完整使用

```typescript
// stores/rootStore.ts
import { createContext, useContext } from 'react';
import { makeAutoObservable, runInAction } from 'mobx';

interface User {
  id: string;
  name: string;
  email: string;
}

class UserStore {
  users: User[] = [];
  selectedUser: User | null = null;
  loading = false;
  error: string | null = null;

  constructor() {
    makeAutoObservable(this);
  }

  get activeUserCount() {
    return this.users.length;
  }

  get sortedUsers() {
    return [...this.users].sort((a, b) => a.name.localeCompare(b.name));
  }

  setSelectedUser(user: User | null) {
    this.selectedUser = user;
  }

  async fetchUsers() {
    this.loading = true;
    this.error = null;

    try {
      const response = await fetch('/api/users');
      const data = await response.json();

      runInAction(() => {
        this.users = data;
        this.loading = false;
      });
    } catch (err) {
      runInAction(() => {
        this.error = err instanceof Error ? err.message : '获取用户失败';
        this.loading = false;
      });
    }
  }

  addUser(user: User) {
    this.users.push(user);
  }

  removeUser(id: string) {
    this.users = this.users.filter(u => u.id !== id);
    if (this.selectedUser?.id === id) {
      this.selectedUser = null;
    }
  }
}

class CartStore {
  items: { id: string; name: string; price: number; quantity: number }[] = [];
  taxRate = 0.1;

  constructor() {
    makeAutoObservable(this, {
      taxRate: false,
    });
  }

  get subtotal() {
    return this.items.reduce((sum, item) => sum + item.price * item.quantity, 0);
  }

  get tax() {
    return this.subtotal * this.taxRate;
  }

  get total() {
    return this.subtotal + this.tax;
  }

  get itemCount() {
    return this.items.reduce((sum, item) => sum + item.quantity, 0);
  }

  addItem(item: typeof this.items[0]) {
    const existing = this.items.find(i => i.id === item.id);
    if (existing) {
      existing.quantity += 1;
    } else {
      this.items.push({ ...item, quantity: 1 });
    }
  }

  removeItem(id: string) {
    this.items = this.items.filter(i => i.id !== id);
  }

  setTaxRate(rate: number) {
    this.taxRate = rate;
  }
}

class RootStore {
  userStore: UserStore;
  cartStore: CartStore;

  constructor() {
    this.userStore = new UserStore();
    this.cartStore = new CartStore();
  }
}

const rootStore = new RootStore();
const StoreContext = createContext<RootStore>(rootStore);

export function useStore() {
  return useContext(StoreContext);
}

export function useUserStore() {
  const { userStore } = useStore();
  return userStore;
}

export function useCartStore() {
  const { cartStore } = useStore();
  return cartStore;
}
```

```tsx
// components/UserList.tsx
import { observer } from 'mobx-react';
import { useUserStore } from '../stores/rootStore';

export const UserList = observer(function UserList() {
  const userStore = useUserStore();

  if (userStore.loading) return <div>加载中...</div>;
  if (userStore.error) return <div>错误：{userStore.error}</div>;

  return (
    <div>
      <h2>用户列表（共 {userStore.activeUserCount} 人）</h2>

      <ul>
        {userStore.sortedUsers.map(user => (
          <li
            key={user.id}
            onClick={() => userStore.setSelectedUser(user)}
            style={{
              fontWeight: userStore.selectedUser?.id === user.id ? 'bold' : 'normal'
            }}
          >
            {user.name} - {user.email}
            <button onClick={() => userStore.removeUser(user.id)}>删除</button>
          </li>
        ))}
      </ul>

      <button onClick={() => userStore.fetchUsers()}>刷新</button>
    </div>
  );
});
```

### 案例 2：Vite 插件开发

```typescript
// plugins/vite-plugin-auto-mpa-html.ts
import { Plugin } from 'vite';
import { glob } from 'glob';
import { resolve } from 'path';
import { readFile } from 'fs/promises';

export function vitePluginAutoMpaHtml(): Plugin {
  return {
    name: 'vite-plugin-auto-mpa-html',
    enforce: 'pre',

    async buildStart() {
      const pages = await glob('src/pages/**/index.html');

      for (const page of pages) {
        const relativePath = page.replace('src/pages/', '').replace('/index.html', '');
        const routePath = '/' + relativePath.replace(/\//g, '/');

        this.emitFile({
          type: 'chunk',
          id: page,
          name: `page-${relativePath.replace(/\//g, '-')}`,
        });
      }
    },

    resolveId(source, importer) {
      if (source === 'virtual:mpa-routes') {
        return source;
      }
    },

    async load(id) {
      if (id === 'virtual:mpa-routes') {
        const pages = await glob('src/pages/**/index.html');
        const routes = pages.map(page => {
          const relativePath = page.replace('src/pages/', '').replace('/index.html', '');
          const routePath = '/' + relativePath.replace(/\//g, '/');
          return {
            path: routePath,
            file: page,
          };
        });

        return `export default ${JSON.stringify(routes, null, 2)}`;
      }
    },
  };
}
```

```typescript
// vite.config.ts
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import { vitePluginAutoMpaHtml } from './plugins/vite-plugin-auto-mpa-html';

export default defineConfig({
  plugins: [
    react(),
    vitePluginAutoMpaHtml(),
  ],
  build: {
    rollupOptions: {
      input: {
        main: resolve(__dirname, 'index.html'),
      },
    },
  },
  server: {
    port: 3000,
    proxy: {
      '/api': {
        target: 'http://localhost:8080',
        changeOrigin: true,
      },
    },
  },
  css: {
    modules: {
      localsConvention: 'camelCase',
    },
  },
  optimizeDeps: {
    include: ['react', 'react-dom', 'react-router-dom'],
  },
});
```

### 案例 3：React 18 并发特性

```tsx
import { useState, useTransition, useDeferredValue, Suspense } from 'react';

function SlowList({ text }: { text: string }) {
  const items = Array.from({ length: 500 }, (_, i) => (
    <div key={i}>
      {text} - 项目 {i}
    </div>
  ));

  return <div>{items}</div>;
}

function SearchResults() {
  const [query, setQuery] = useState('');
  const [isPending, startTransition] = useTransition();
  const deferredQuery = useDeferredValue(query);

  const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    startTransition(() => {
      setQuery(e.target.value);
    });
  };

  return (
    <div>
      <input value={query} onChange={handleChange} placeholder="搜索..." />

      {isPending && <div>更新中...</div>}

      <Suspense fallback={<div>加载中...</div>}>
        <SlowList text={deferredQuery} />
      </Suspense>
    </div>
  );
}
```

```tsx
import { useState, Suspense } from 'react';
import { fetchUserProfile, fetchUserRecommendations } from './api';

function UserProfile({ userId }: { userId: string }) {
  return (
    <Suspense fallback={<div>加载用户信息...</div>}>
      <ProfileDetails userId={userId} />
    </Suspense>
  );
}

function UserRecommendations({ userId }: { userId: string }) {
  return (
    <Suspense fallback={<div>加载推荐...</div>}>
      <Recommendations userId={userId} />
    </Suspense>
  );
}

function UserPage({ userId }: { userId: string }) {
  return (
    <div>
      <UserProfile userId={userId} />
      <UserRecommendations userId={userId} />
    </div>
  );
}
```

### 案例 4：React Native 基础组件

```tsx
// App.tsx
import React, { useState } from 'react';
import {
  StyleSheet,
  Text,
  View,
  TextInput,
  TouchableOpacity,
  FlatList,
  SafeAreaView,
  StatusBar,
} from 'react-native';

interface TodoItem {
  id: string;
  title: string;
  completed: boolean;
}

export default function App() {
  const [todos, setTodos] = useState<TodoItem[]>([]);
  const [inputText, setInputText] = useState('');

  const addTodo = () => {
    if (inputText.trim() === '') return;

    const newTodo: TodoItem = {
      id: Date.now().toString(),
      title: inputText.trim(),
      completed: false,
    };

    setTodos([...todos, newTodo]);
    setInputText('');
  };

  const toggleTodo = (id: string) => {
    setTodos(
      todos.map(todo =>
        todo.id === id ? { ...todo, completed: !todo.completed } : todo
      )
    );
  };

  const deleteTodo = (id: string) => {
    setTodos(todos.filter(todo => todo.id !== id));
  };

  const renderItem = ({ item }: { item: TodoItem }) => (
    <View style={styles.todoItem}>
      <TouchableOpacity
        style={styles.checkbox}
        onPress={() => toggleTodo(item.id)}
      >
        <Text style={styles.checkboxText}>
          {item.completed ? '✓' : '○'}
        </Text>
      </TouchableOpacity>

      <Text style={[
        styles.todoText,
        item.completed && styles.completedText
      ]}>
        {item.title}
      </Text>

      <TouchableOpacity
        style={styles.deleteButton}
        onPress={() => deleteTodo(item.id)}
      >
        <Text style={styles.deleteText}>×</Text>
      </TouchableOpacity>
    </View>
  );

  return (
    <SafeAreaView style={styles.container}>
      <StatusBar barStyle="dark-content" />

      <View style={styles.header}>
        <Text style={styles.title}>待办事项</Text>
      </View>

      <View style={styles.inputContainer}>
        <TextInput
          style={styles.input}
          value={inputText}
          onChangeText={setInputText}
          placeholder="添加新待办..."
          onSubmitEditing={addTodo}
        />
        <TouchableOpacity style={styles.addButton} onPress={addTodo}>
          <Text style={styles.addButtonText}>添加</Text>
        </TouchableOpacity>
      </View>

      <FlatList
        data={todos}
        renderItem={renderItem}
        keyExtractor={item => item.id}
        style={styles.list}
        contentContainerStyle={styles.listContent}
      />
    </SafeAreaView>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#f5f5f5',
  },
  header: {
    padding: 20,
    backgroundColor: '#fff',
    borderBottomWidth: 1,
    borderBottomColor: '#e0e0e0',
  },
  title: {
    fontSize: 24,
    fontWeight: 'bold',
  },
  inputContainer: {
    flexDirection: 'row',
    padding: 16,
    backgroundColor: '#fff',
  },
  input: {
    flex: 1,
    height: 44,
    borderWidth: 1,
    borderColor: '#e0e0e0',
    borderRadius: 8,
    paddingHorizontal: 12,
    marginRight: 12,
  },
  addButton: {
    backgroundColor: '#007AFF',
    paddingHorizontal: 20,
    borderRadius: 8,
    justifyContent: 'center',
  },
  addButtonText: {
    color: '#fff',
    fontWeight: '600',
  },
  list: {
    flex: 1,
  },
  listContent: {
    padding: 16,
  },
  todoItem: {
    flexDirection: 'row',
    alignItems: 'center',
    backgroundColor: '#fff',
    padding: 16,
    borderRadius: 8,
    marginBottom: 8,
  },
  checkbox: {
    width: 24,
    height: 24,
    borderRadius: 12,
    borderWidth: 2,
    borderColor: '#007AFF',
    justifyContent: 'center',
    alignItems: 'center',
    marginRight: 12,
  },
  checkboxText: {
    fontSize: 14,
    color: '#007AFF',
  },
  todoText: {
    flex: 1,
    fontSize: 16,
  },
  completedText: {
    textDecorationLine: 'line-through',
    color: '#999',
  },
  deleteButton: {
    width: 24,
    height: 24,
    justifyContent: 'center',
    alignItems: 'center',
  },
  deleteText: {
    fontSize: 20,
    color: '#FF3B30',
  },
});
```

## 学习建议

附加技能无需全部掌握，根据实际工作需求选择：
- 想做移动端 App → React Native
- 想做小程序 → Taro
- 想深入理解原理 → Redux 源码
- 想提升构建能力 → Webpack/Vite 深入配置

## 持续学习资源

- [React 官方文档](https://react.dev)
- [React Native 官方文档](https://reactnative.dev)
- [Next.js 官方文档](https://nextjs.org)
- [TypeScript 官方文档](https://www.typescriptlang.org)
- [MobX 官方文档](https://mobx.js.org)
- [Vite 官方文档](https://vitejs.dev)
