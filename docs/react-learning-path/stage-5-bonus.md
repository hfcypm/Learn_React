# 阶段五：附加技能（加分项）

**目标**：扩展技术视野，提升竞争力

## 可选技能列表

### 1. 状态管理进阶

- [ ] Redux 源码思想
- [ ] MobX 响应式状态管理

**详细概念：**

**1.1 Redux 源码思想**

深入理解 Redux 的核心思想，即使使用 Redux Toolkit，了解源码也有助于更好地使用。

Redux 核心原则：
- **单一数据源**：整个应用的 state 存储在唯一的 store 中
- **State 是只读的**：只能通过触发 action 来改变 state
- **使用纯函数Reducer**：Reducer 根据 action 计算新 state，必须是纯函数

单向数据流：
```
用户交互 → dispatch(action) → Reducer 计算新 state → 通知订阅者 → UI 更新
```

中间件机制：
- 中间件扩展了 dispatch 的能力
- 经典的 applyMiddleware 实现了 middleware chain
- 常用中间件：redux-thunk（异步）、redux-saga（复杂异步）、redux-logger（日志）

MobX 与 Redux 对比：
- **Redux**：更规范，适合大型团队；学习曲线陡峭
- **MobX**：更灵活，适合快速开发；响应式自动追踪依赖

**1.2 MobX 响应式原理**

MobX 基于响应式编程思想，核心是自动追踪依赖并响应变化。

核心概念：
- **Observable（可观察对象）**：被标记的状态可以自动追踪
- **Action（动作）**：修改状态的方法
- **Computed（计算属性）**：基于 observable 计算的派生值
- **Reaction（反应）**：当依赖变化时自动执行的副作用

makeAutoObservable 原理：
- 自动将属性变为 observable
- 自动将方法变为 action
- 自动将 getter 变为 computed

响应式追踪机制：
- 当访问 observable 属性时自动收集依赖
- 当这些依赖变化时自动触发更新
- 不需要手动声明依赖关系

**1.3 MobX 与 Redux 的选择**

两种状态管理方案各有适用场景，需要根据项目特点选择。

| 场景 | 推荐方案 |
|------|----------|
| 大型团队项目 | Redux（更规范可控） |
| 快速原型开发 | MobX（更少代码） |
| 需要 DevTools 调试 | 两者皆可 |
| 复杂异步流 | Redux + redux-saga |
| 简单状态 | Context + useReducer |

最佳实践：
- 中小型项目可以使用 Zustand 或 Context
- 大型复杂应用使用 Redux Toolkit
- 需要响应式编程体验可选 MobX

**经典案例：MobX 完整使用**

```typescript
// stores/UserStore.ts
import { makeAutoObservable, runInAction } from 'mobx';

interface User {
  id: string;
  name: string;
  email: string;
  avatar?: string;
  role: 'admin' | 'user';
}

class UserStore {
  users: User[] = [];
  selectedUser: User | null = null;
  loading = false;
  error: string | null = null;

  constructor() {
    makeAutoObservable(this, {
      error: false,
    });
  }

  get userCount() {
    return this.users.length;
  }

  get adminCount() {
    return this.users.filter(user => user.role === 'admin').length;
  }

  get sortedUsers() {
    return [...this.users].sort((a, b) => a.name.localeCompare(b.name));
  }

  get selectedUserName() {
    return this.selectedUser?.name ?? '(未选择)';
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

  async createUser(userData: Omit<User, 'id'>) {
    this.loading = true;
    this.error = null;

    try {
      const response = await fetch('/api/users', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(userData),
      });

      const newUser = await response.json();

      runInAction(() => {
        this.users.push(newUser);
        this.loading = false;
      });

      return newUser;
    } catch (err) {
      runInAction(() => {
        this.error = err instanceof Error ? err.message : '创建用户失败';
        this.loading = false;
      });
      throw err;
    }
  }

  removeUser(id: string) {
    this.users = this.users.filter(u => u.id !== id);
    if (this.selectedUser?.id === id) {
      this.selectedUser = null;
    }
  }
}

export const userStore = new UserStore();
```

```typescript
// stores/CartStore.ts
import { makeAutoObservable } from 'mobx';

interface CartItem {
  id: string;
  name: string;
  price: number;
  quantity: number;
}

class CartStore {
  items: CartItem[] = [];
  taxRate = 0.1;
  shippingFee = 10;

  constructor() {
    makeAutoObservable(this, {
      shippingFee: false,
    });
  }

  get subtotal() {
    return this.items.reduce(
      (sum, item) => sum + item.price * item.quantity,
      0
    );
  }

  get tax() {
    return this.subtotal * this.taxRate;
  }

  get total() {
    return this.subtotal + this.tax + this.shippingFee;
  }

  get itemCount() {
    return this.items.reduce((sum, item) => sum + item.quantity, 0);
  }

  get isEmpty() {
    return this.items.length === 0;
  }

  addItem(item: Omit<CartItem, 'quantity'>) {
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

  updateQuantity(id: string, quantity: number) {
    if (quantity <= 0) {
      this.removeItem(id);
      return;
    }

    const item = this.items.find(i => i.id === id);
    if (item) {
      item.quantity = quantity;
    }
  }

  clearCart() {
    this.items = [];
  }

  setTaxRate(rate: number) {
    if (rate >= 0 && rate <= 1) {
      this.taxRate = rate;
    }
  }

  applyDiscount(code: string): boolean {
    if (code === 'SAVE10') {
      this.shippingFee = 0;
      return true;
    }
    return false;
  }
}

export const cartStore = new CartStore();
```

```typescript
// stores/RootStore.ts
import { createContext, useContext } from 'react';
import { userStore } from './UserStore';
import { cartStore } from './CartStore';

class RootStore {
  userStore = userStore;
  cartStore = cartStore;
}

export const rootStore = new RootStore();
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
import { useUserStore } from '../stores/RootStore';

export const UserList = observer(function UserList() {
  const userStore = useUserStore();

  if (userStore.loading && userStore.users.length === 0) {
    return <div>加载中...</div>;
  }

  if (userStore.error) {
    return <div>错误：{userStore.error}</div>;
  }

  return (
    <div>
      <div>
        <span>总用户数：{userStore.userCount}</span>
        <span>管理员数：{userStore.adminCount}</span>
      </div>

      <button onClick={() => userStore.fetchUsers()} disabled={userStore.loading}>
        刷新
      </button>

      <ul>
        {userStore.sortedUsers.map(user => (
          <li
            key={user.id}
            onClick={() => userStore.setSelectedUser(user)}
            style={{
              cursor: 'pointer',
              fontWeight: userStore.selectedUser?.id === user.id ? 'bold' : 'normal',
            }}
          >
            <img src={user.avatar || '/default-avatar.png'} alt={user.name} />
            <span>{user.name}</span>
            <span>{user.email}</span>
            <span>[{user.role}]</span>
            <button onClick={(e) => { e.stopPropagation(); userStore.removeUser(user.id); }}>
              删除
            </button>
          </li>
        ))}
      </ul>

      <div>
        <p>选中的用户：{userStore.selectedUserName}</p>
      </div>
    </div>
  );
});
```

```tsx
// components/Cart.tsx
import { observer } from 'mobx-react';
import { useCartStore } from '../stores/RootStore';

export const Cart = observer(function Cart() {
  const cartStore = useCartStore();

  if (cartStore.isEmpty) {
    return <div>购物车为空</div>;
  }

  return (
    <div>
      <h2>购物车</h2>

      <ul>
        {cartStore.items.map(item => (
          <li key={item.id}>
            <span>{item.name}</span>
            <span>¥{item.price} x {item.quantity} = ¥{item.price * item.quantity}</span>
            <button onClick={() => cartStore.updateQuantity(item.id, item.quantity - 1)}>
              -
            </button>
            <button onClick={() => cartStore.updateQuantity(item.id, item.quantity + 1)}>
              +
            </button>
            <button onClick={() => cartStore.removeItem(item.id)}>
              删除
            </button>
          </li>
        ))}
      </ul>

      <div>
        <p>小计：¥{cartStore.subtotal.toFixed(2)}</p>
        <p>税费（{cartStore.taxRate * 100}%）：¥{cartStore.tax.toFixed(2)}</p>
        <p>运费：¥{cartStore.shippingFee}</p>
        <p>总计：¥{cartStore.total.toFixed(2)}</p>
      </div>

      <button onClick={() => cartStore.clearCart()}>
        清空购物车
      </button>
    </div>
  );
});
```

---

### 2. 跨端技术

- [ ] React Native（移动端）
- [ ] Taro（小程序）

**详细概念：**

**2.1 React Native 核心原理**

React Native 允许使用 React 语法开发原生移动应用，一套代码可以运行在 iOS 和 Android 平台。

与 Web React 的区别：
- **渲染目标不同**：Web React 渲染到 DOM，RN 渲染到原生组件
- **组件不同**：`<div>` 变成 `<View>`，`<text>` 代替 span/div
- **样式系统**：RN 使用类 CSS 的 Flexbox 布局，但属性名有差异
- **无 DOM API**：不能使用 window、document 等浏览器 API

React Native 工作原理：
- JavaScript 代码运行在 JavaScriptCore 引擎中
- 通过 Bridge 与原生平台通信
- 原生模块提供设备能力（如相机、地理位置）
- 新架构（Fabric/TurboModules）正在改进性能和互操作性

**2.2 React Native vs Flutter**

跨端框架的两大选择：React Native 和 Flutter。

| 维度 | React Native | Flutter |
|------|--------------|---------|
| 语言 | JavaScript/TypeScript | Dart |
| 渲染 | 原生组件 | Skia 自绘引擎 | 
| 性能 | 接近原生 | 非常接近原生 |
| 生态 | NPM 生态丰富 | Dart 包管理器 |
| 学习曲线 | 低（有 React 基础） | 中（Dart 语言） |
| 热更新 | 支持 JS 热更新 | 支持 Dart 热更新 |
| 社区 | 更大更成熟 | 增长迅速 |

**2.3 Taro 多端统一开发**

Taro 是京东开源的多端统一开发框架，支持编译到 React Native、微信小程序、H5 等。

核心优势：
- **一套代码多端运行**：减少开发和维护成本
- **React 语法支持**：可以使用 Hooks、TypeScript
- **丰富组件库**：京东团队维护的组件库
- **插件机制**：支持自定义转换逻辑

小程序开发特点：
- 体积限制严格，需要代码优化
- API 有白名单，不能使用所有浏览器 API
- 分包加载优化首屏性能

**经典案例：React Native 基础组件**

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
  Alert,
  Image,
  ScrollView,
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
    Alert.alert(
      '确认删除',
      '确定要删除这个待办事项吗？',
      [
        { text: '取消', style: 'cancel' },
        {
          text: '删除',
          style: 'destructive',
          onPress: () => setTodos(todos.filter(todo => todo.id !== id)),
        },
      ]
    );
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

      <Text
        style={[
          styles.todoText,
          item.completed && styles.completedText,
        ]}
      >
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
        <Text style={styles.subtitle}>共 {todos.length} 项</Text>
      </View>

      <View style={styles.inputContainer}>
        <TextInput
          style={styles.input}
          value={inputText}
          onChangeText={setInputText}
          placeholder="添加新待办..."
          placeholderTextColor="#999"
          onSubmitEditing={addTodo}
          returnKeyType="done"
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
        ListEmptyComponent={
          <View style={styles.emptyContainer}>
            <Text style={styles.emptyText}>暂无待办事项</Text>
          </View>
        }
      />

      <View style={styles.statsContainer}>
        <Text style={styles.statsText}>
          已完成：{todos.filter(t => t.completed).length} / {todos.length}
        </Text>
      </View>
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
    color: '#333',
  },
  subtitle: {
    fontSize: 14,
    color: '#666',
    marginTop: 4,
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
    fontSize: 16,
  },
  addButton: {
    backgroundColor: '#007AFF',
    paddingHorizontal: 20,
    borderRadius: 8,
    justifyContent: 'center',
    alignItems: 'center',
  },
  addButtonText: {
    color: '#fff',
    fontSize: 16,
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
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 1 },
    shadowOpacity: 0.1,
    shadowRadius: 2,
    elevation: 2,
  },
  checkbox: {
    width: 28,
    height: 28,
    borderRadius: 14,
    borderWidth: 2,
    borderColor: '#007AFF',
    justifyContent: 'center',
    alignItems: 'center',
    marginRight: 12,
  },
  checkboxText: {
    fontSize: 16,
    color: '#007AFF',
    fontWeight: 'bold',
  },
  todoText: {
    flex: 1,
    fontSize: 16,
    color: '#333',
  },
  completedText: {
    textDecorationLine: 'line-through',
    color: '#999',
  },
  deleteButton: {
    width: 28,
    height: 28,
    justifyContent: 'center',
    alignItems: 'center',
  },
  deleteText: {
    fontSize: 24,
    color: '#FF3B30',
    fontWeight: 'bold',
  },
  emptyContainer: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    paddingVertical: 40,
  },
  emptyText: {
    fontSize: 16,
    color: '#999',
  },
  statsContainer: {
    padding: 16,
    backgroundColor: '#fff',
    borderTopWidth: 1,
    borderTopColor: '#e0e0e0',
  },
  statsText: {
    fontSize: 14,
    color: '#666',
    textAlign: 'center',
  },
});
```

---

### 3. 构建工具深度配置

- [ ] Webpack 深度配置
- [ ] Vite 深度配置

**详细概念：**

**3.1 Webpack 核心概念**

Webpack 是最成熟的模块打包工具，功能强大但配置复杂。

核心概念：
- **Entry**：入口文件，Webpack 从这里开始构建依赖图
- **Output**：输出配置，告诉 Webpack 如何输出 bundle
- **Loaders**：处理非 JS 文件，将它们转换为有效模块
- **Plugins**：执行范围更广的任务（打包优化、资源管理、注入环境变量）
- **Mode**：环境模式（development/production），自动启用优化

常见的 Loader：
- `ts-loader` / `babel-loader`：处理 TypeScript/JavaScript
- `css-loader` / `style-loader`：处理 CSS
- `file-loader` / `asset/resource`：处理静态资源
- `html-webpack-plugin`：生成 HTML 文件

常见的 Plugin：
- `mini-css-extract-plugin`：将 CSS 提取为单独文件
- `terser-webpack-plugin`：压缩 JS
- `html-webpack-plugin`：生成 HTML 入口
- `ModuleFederationPlugin`：微前端支持

**3.2 Vite 现代构建工具**

Vite 是下一代前端构建工具，利用浏览器原生 ES Module 实现极快的开发体验。

Vite vs Webpack：
- **开发体验**：Vite 启动快（无需打包），HMR 几乎即时
- **生产构建**：都使用 Rollup，产出质量相近
- **配置复杂度**：Vite 配置更简洁
- **生态**：Webpack 插件更丰富，Vite 正在追赶

Vite 核心原理：
- **Dev Server**：不打包，利用浏览器 ES Module 按需加载
- **HMR**：基于 ESM，只需精确更新变化的模块
- **预构建**：使用 esbuild 预构建依赖（比 webpack 快 10-100 倍）
- **Rollup 打包**：生产环境使用 Rollup 打包，优化产出

**3.3 代码分割与 Tree Shaking**

代码分割是将代码拆分成多个 chunk，用户只下载需要的部分。

代码分割策略：
- **路由分割**：每个路由页面单独打包
- **组件分割**：按需加载大型组件
- **Vendor 分割**：第三方库单独打包，利用缓存

Tree Shaking 原理：
- 基于 ES Module 静态分析
- 移除未使用的导出
- 需要开启 minifier 才能完全移除 dead code

动态导入：
```tsx
const LazyComponent = lazy(() => import('./LazyComponent'));
```

**经典案例：Vite 深度配置**

```typescript
// vite.config.ts
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import { vitePluginAutoImport } from 'vite-plugin-auto-import';
import { vitePluginAutoComponent } from 'vite-plugin-auto-component';
import path from 'path';

export default defineConfig({
  plugins: [
    react(),
    vitePluginAutoImport({
      imports: [
        'react',
        'react-dom',
        'react-router-dom',
        {
          'react-redux': ['useDispatch', 'useSelector'],
          '@tanstack/react-query': ['useQuery', 'useMutation'],
        },
      ],
      dts: 'src/auto-imports.d.ts',
    }),
  ],

  resolve: {
    alias: {
      '@': path.resolve(__dirname, 'src'),
      '@components': path.resolve(__dirname, 'src/components'),
      '@hooks': path.resolve(__dirname, 'src/hooks'),
      '@utils': path.resolve(__dirname, 'src/utils'),
      '@store': path.resolve(__dirname, 'src/store'),
    },
  },

  server: {
    port: 3000,
    host: true,
    proxy: {
      '/api': {
        target: 'http://localhost:8080',
        changeOrigin: true,
        rewrite: path => path.replace(/^\/api/, ''),
      },
      '/ws': {
        target: 'ws://localhost:8080',
        ws: true,
      },
    },
  },

  build: {
    target: 'es2015',
    cssTarget: 'chrome80',
    sourcemap: false,
    rollupOptions: {
      output: {
        manualChunks: {
          'react-vendor': ['react', 'react-dom', 'react-router-dom'],
          'ui-vendor': ['antd', '@ant-design/icons'],
        },
        chunkFileNames: 'assets/js/[name]-[hash].js',
        entryFileNames: 'assets/js/[name]-[hash].js',
        assetFileNames: 'assets/[ext]/[name]-[hash].[ext]',
      },
    },
    chunkSizeWarningLimit: 1000,
  },

  css: {
    modules: {
      localsConvention: 'camelCase',
      generateScopedName: '[name]__[local]___[hash:base64:5]',
    },
    preprocessorOptions: {
      less: {
        javascriptEnabled: true,
        modifyVars: {
          'primary-color': '#1890ff',
        },
      },
    },
  },

  optimizeDeps: {
    include: ['react', 'react-dom', 'react-router-dom', 'antd'],
    exclude: [],
  },

  esbuild: {
    drop: process.env.NODE_ENV === 'production' ? ['console', 'debugger'] : [],
  },
});
```

**经典案例：Webpack 配置**

```javascript
// webpack.config.js
const path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');
const MiniCssExtractPlugin = require('mini-css-extract-plugin');
const CssMinimizerPlugin = require('css-minimizer-webpack-plugin');
const TerserPlugin = require('terser-webpack-plugin');
const { ModuleFederationPlugin } = require('webpack').container;

module.exports = (env, argv) => {
  const isProduction = argv.mode === 'production';

  return {
    entry: './src/index.tsx',

    output: {
      path: path.resolve(__dirname, 'dist'),
      filename: isProduction ? '[name].[contenthash].js' : '[name].js',
      chunkFilename: isProduction ? '[name].[contenthash].chunk.js' : '[name].chunk.js',
      publicPath: 'auto',
      clean: true,
    },

    resolve: {
      extensions: ['.tsx', '.ts', '.jsx', '.js'],
      alias: {
        '@': path.resolve(__dirname, 'src'),
        '@components': path.resolve(__dirname, 'src/components'),
        '@hooks': path.resolve(__dirname, 'src/hooks'),
      },
    },

    module: {
      rules: [
        {
          test: /\.(ts|tsx)$/,
          use: {
            loader: 'ts-loader',
            options: {
              transpileOnly: true,
            },
          },
          exclude: /node_modules/,
        },
        {
          test: /\.css$/,
          use: [
            isProduction ? MiniCssExtractPlugin.loader : 'style-loader',
            'css-loader',
            {
              loader: 'postcss-loader',
              options: {
                postcssOptions: {
                  plugins: ['autoprefixer'],
                },
              },
            },
          ],
        },
        {
          test: /\.(png|svg|jpg|jpeg|gif)$/i,
          type: 'asset/resource',
        },
        {
          test: /\.(woff|woff2|eot|ttf|otf)$/i,
          type: 'asset/resource',
        },
      ],
    },

    plugins: [
      new HtmlWebpackPlugin({
        template: './public/index.html',
        inject: 'body',
        minify: isProduction
          ? {
              removeComments: true,
              collapseWhitespace: true,
              removeAttributeQuotes: true,
            }
          : false,
      }),

      new MiniCssExtractPlugin({
        filename: isProduction ? '[name].[contenthash].css' : '[name].css',
      }),

      new ModuleFederationPlugin({
        name: 'host',
        remotes: {
          remoteApp: 'remote@http://localhost:3001/remoteEntry.js',
        },
        shared: {
          react: { singleton: true, eager: true },
          'react-dom': { singleton: true, eager: true },
        },
      }),
    ],

    optimization: {
      minimize: isProduction,
      minimizer: [
        new TerserPlugin({
          terserOptions: {
            compress: {
              drop_console: isProduction,
              drop_debugger: isProduction,
            },
          },
        }),
        new CssMinimizerPlugin(),
      ],
      splitChunks: {
        chunks: 'all',
        cacheGroups: {
          vendor: {
            test: /[\\/]node_modules[\\/]/,
            name: 'vendors',
            chunks: 'all',
          },
          react: {
            test: /[\\/]node_modules[\\/](react|react-dom|react-router-dom)[\\/]/,
            name: 'react-vendor',
            chunks: 'all',
            priority: 10,
          },
        },
      },
    },

    devServer: {
      port: 3000,
      hot: true,
      historyApiFallback: true,
      proxy: {
        '/api': {
          target: 'http://localhost:8080',
          changeOrigin: true,
        },
      },
    },

    devtool: isProduction ? 'source-map' : 'eval-source-map',

    performance: {
      hints: isProduction ? 'warning' : false,
      maxEntrypointSize: 512000,
      maxAssetSize: 512000,
    },
  };
};
```

---

### 4. React 18 新特性

- [ ] 并发渲染（Concurrent Rendering）
- [ ] Suspense 深入理解
- [ ] Server Components（服务端组件）

**详细概念：**

**4.1 并发渲染（Concurrent Rendering）**

React 18 引入并发模式，这是 React 架构的重大升级，让 React 可以同时准备多个版本的 UI。

并发模式的核心能力：
- **可中断渲染**：高优先级更新可以打断低优先级更新
- **Transition**：`useTransition` 标记非紧急更新
- **Deferred Value**：`useDeferredValue` 延迟值更新
- **Suspense**：更强大的加载状态处理

useTransition 的使用场景：
- 搜索输入时，搜索结果更新是非紧急的
- 标签页切换时，内容渲染可以延迟
- 大列表排序/筛选时，UI 保持响应

并发渲染的优势：
- 用户输入始终保持响应（紧急更新优先）
- 后台可以准备新版本的 UI
- 更可预测的性能表现

**4.2 Suspense 深入理解**

Suspense 不仅仅是一个 loading 组件，它是 React 异步数据获取的核心基础设施。

Suspense 的工作原理：
- 当子组件抛出 Promise（或者说进入"pending"状态），Suspense 显示 fallback
- Promise resolved 后，Suspense 显示实际内容
- 多个 Suspense 可以并行加载各自的内容

Suspense 与数据获取库集成：
- React Router v6 原生支持 Suspense
- Relay/Apollo 客户端支持 Suspense
- React Query/SWR 通过第三方集成支持

Suspense 边界问题：
- 一个 Suspense 只处理直接子树的加载状态
- 深层嵌套的加载需要多个 Suspense
- Error Boundaries 处理错误，不处理 loading

**4.3 Server Components 服务端组件**

React Server Components（RSC）是 React 18 的革命性特性，允许组件在服务端渲染。

与 SSR 的区别：
- **SSR**：整个页面在服务端渲染成 HTML，然后 hydrate
- **Server Components**：部分组件在服务端渲染，不增加客户端 JS 体积

Server Components 优势：
- **零客户端 JS**：服务端组件不会发送到客户端
- **直接访问后端**：可以直接调用数据库、文件系统
- **自动代码分割**：客户端组件自动从 bundle 中排除

限制：
- 不能使用 hooks（因为 hooks 是客户端概念）
- 不能使用浏览器 API
- 不能有事件处理

**经典案例：并发特性**

```tsx
// ConcurrentFeatures.tsx
import { useState, useTransition, useDeferredValue, Suspense, use } from 'react';

function SlowList({ text }: { text: string }) {
  const items = Array.from({ length: 500 }, (_, i) => (
    <div key={i}>
      {text} - 项目 {i + 1}
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
      <label>
        搜索：
        <input value={query} onChange={handleChange} />
      </label>

      {isPending && <p>更新中...</p>}

      <SlowList text={deferredQuery} />
    </div>
  );
}
```

```tsx
// use.ts hook (React 18.3+)
interface User {
  id: string;
  name: string;
  email: string;
}

function UserProfile({ userPromise }: { userPromise: Promise<User> }) {
  const user = use(userPromise);

  return (
    <div>
      <h1>{user.name}</h1>
      <p>{user.email}</p>
    </div>
  );
}

function UserPage({ userId }: { userId: string }) {
  const userPromise = fetch(`/api/users/${userId}`).then(res => res.json());

  return (
    <Suspense fallback={<div>加载中...</div>}>
      <UserProfile userPromise={userPromise} />
    </Suspense>
  );
}
```

```tsx
// Suspense最佳实践
import { Suspense } from 'react';

function Resource({ promise }: { promise: Promise<any> }) {
  const data = use(promise);
  return <div>{JSON.stringify(data)}</div>;
}

function ParallelSuspense() {
  const userPromise = fetch('/api/user').then(r => r.json());
  const postsPromise = fetch('/api/posts').then(r => r.json());

  return (
    <div>
      <h1>并行加载</h1>

      <Suspense fallback={<div>加载用户...</div>}>
        <Resource promise={userPromise} />
      </Suspense>

      <Suspense fallback={<div>加载帖子...</div>}>
        <Resource promise={postsPromise} />
      </Suspense>
    </div>
  );
}
```

```tsx
// 错误边界与 Suspense
import { Component, ReactNode } from 'react';

class ErrorBoundary extends Component<
  { children: ReactNode; fallback?: ReactNode },
  { hasError: boolean; error?: Error }
> {
  constructor(props: any) {
    super(props);
    this.state = { hasError: false };
  }

  static getDerivedStateFromError(error: Error) {
    return { hasError: true, error };
  }

  componentDidCatch(error: Error, errorInfo: any) {
    console.error('Error:', error, errorInfo);
  }

  render() {
    if (this.state.hasError) {
      return (
        this.props.fallback || (
          <div>
            <h2>出错了</h2>
            <p>{this.state.error?.message}</p>
          </div>
        )
      );
    }

    return this.props.children;
  }
}

function App() {
  return (
    <ErrorBoundary fallback={<div>加载失败</div>}>
      <Suspense fallback={<div>加载中...</div>}>
        <SlowComponent />
      </Suspense>
    </ErrorBoundary>
  );
}
```

---

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
