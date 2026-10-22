# 阶段一：ArkUI 声明式 UI 与状态管理

**目标**：掌握 ArkUI 的组件、布局与全部核心装饰器，理解每个装饰器的数据流方向与刷新粒度，能构建数据驱动刷新的可交互界面。本阶段以状态管理 V2 为主线，并给出 V1 的对应关系与迁移口径。

## 1. 声明式 UI 范式

ArkUI 中 UI 是程序状态的运行结果。你描述“界面应该长什么样”，状态变化时框架自动刷新相关部分。这与命令式地在代码里手动改控件属性完全不同。

```text
状态(state) 变化 -> 框架标记依赖该状态的组件 -> 重新执行相关 build 片段 -> 差量更新
```

三个基本概念：

- 组件（Component）：UI 的构建块，如 `Text`、`Button`、`Column`。
- 状态（State）：驱动刷新的数据源，用装饰器标注。
- 构建函数（build）：声明组件树的方法。

关键理解：**装饰器决定三件事**——这个数据从哪来、谁能改它、它的变化会刷新谁。后文每个装饰器都围绕这三点说明。

## 2. 装饰器完整清单

装饰器分三类：V2 状态管理、V1 状态管理、通用装饰器。**新项目优先使用 V2，单组件内不混用 V1 与 V2。**

### 2.1 V2 状态管理装饰器

| 装饰器 | 作用 | 可否外部初始化 | 观测粒度 |
|---|---|---|---|
| `@ComponentV2` | 声明 V2 自定义组件 | - | - |
| `@Local` | 组件内部状态 | 否 | 变量本身；对象深层需 `@Trace` |
| `@Param` | 组件外部输入，子组件只读 | 是（父传入） | 引用变化 |
| `@Once` | 初始化同步一次，之后本地拥有 | 是（仅一次） | 变量本身 |
| `@Event` | 组件输出回调（子传父） | 是（父传入） | - |
| `@Provider` | 向后代提供数据 | 是 | key 级 |
| `@Consumer` | 消费祖先数据，双向同步 | 是（自动关联） | key 级 |
| `@Monitor` | 状态变量异步监听（可深层） | - | 指定路径 |
| `@SyncMonitor` | 状态变量同步监听 | - | 指定路径 |
| `@Computed` | 计算属性，依赖变化才重算 | - | 依赖项 |
| `@ObservedV2` | 标记类可观察 | - | 类实例 |
| `@Trace` | 标记类属性可观察（深层） | - | 属性级 |
| `@Type` | 标记类属性类型 | - | 序列化类型 |
| `@ReusableV2` | 标记组件可复用（复用池） | - | 组件级 |

### 2.2 通用装饰器

| 装饰器 | 作用 |
|---|---|
| `@Entry` | 标记页面入口 |
| `@Builder` | 自定义构建函数（可复用 UI 片段） |
| `@LocalBuilder` | 维持组件关系的构建函数（保留 `this`） |
| `@BuilderParam` | 引用 `@Builder`，实现插槽 |
| `@Styles` | 定义通用可复用样式（仅通用属性） |
| `@Extend` | 扩展指定组件的样式 |
| `@AnimatableExtend` | 定义可动画的自定义属性 |
| `@Require` | 校验构造传参必填 |
| `@Env` | 读取系统环境变量（如深浅色） |

### 2.3 V1 状态管理装饰器（了解与迁移用）

| 装饰器 | 作用 |
|---|---|
| `@Component` | 声明 V1 自定义组件 |
| `@State` | 组件内部状态（可外部初始化） |
| `@Prop` | 父子单向同步（深拷贝） |
| `@Link` | 父子双向同步 |
| `@Provide` / `@Consume` | 跨层双向同步 |
| `@Watch` | 状态变量变化监听（顶层属性） |
| `@Observed` / `@ObjectLink` | 类对象嵌套属性观察 |
| `@Track` | 类对象属性级精确更新 |
| `@StorageLink` / `@StorageProp` | 与 AppStorage 双向 / 单向同步 |
| `@LocalStorageLink` / `@LocalStorageProp` | 与 LocalStorage 双向 / 单向同步 |
| `@Reusable` | 标记组件可复用（V1） |

### 2.4 V1 与 V2 对应关系

| V1 | V2 | 迁移口径 |
|---|---|---|
| `@Component` | `@ComponentV2` | 组件基类 |
| `@State` | `@Local` / `@Param`+`@Once` | `@State` 可外部初始化；需外部传参时用 `@Param`+`@Once` |
| `@Prop` | `@Param` | `@Prop` 深拷贝，`@Param` 引用 |
| `@Link` | `@Param`+`@Event`（或 `!!`） | V2 用输入+事件显式实现双向绑定 |
| `@Provide` / `@Consume` | `@Provider` / `@Consumer` | 跨层双向同步 |
| `@Watch` | `@Monitor` / `@SyncMonitor` | `@Monitor` 支持深层、可拿新旧值 |
| `@Observed` / `@ObjectLink` | `@ObservedV2` / `@Trace` | V2 原生支持深层观测 |
| `@Track` | `@Trace` | 属性级精确更新 |
| `@StorageLink` / `@StorageProp` | `AppStorageV2` | 应用级内存状态 |
| `PersistentStorage` | `PersistenceV2` | 应用级持久化 |
| `LocalStorage` | `LocalStorage` + `@ObservedV2`/`@Trace` | 页面/能力级状态 |

## 3. 装饰器语义对照（数据流与刷新粒度）

理解 V2 的关键是“数据自带可观察性、数据流显式、刷新按需”。下面这张表概括了最常用的组合：

| 场景 | 推荐装饰器 | 数据流 | 刷新行为 |
|---|---|---|---|
| 组件自己的状态 | `@Local` | 组件内闭环 | 赋值即刷新 |
| 父传子只读 | `@Param` | 父 -> 子（单向） | 父值变化刷新子 |
| 父传子且子可改回父 | `@Param` + `@Event` / `!!` | 双向 | 双方变化互相同步 |
| 只在初始化接收一次 | `@Param` + `@Once` | 一次性 | 之后父变化不刷新 |
| 子通知父 | `@Event` | 子 -> 父 | 触发父逻辑 |
| 跨多层共享 | `@Provider` + `@Consumer` | 双向 | 按 key 关联刷新 |
| 嵌套对象深层变化 | `@ObservedV2` + `@Trace` | 数据自可观察 | 属性级精准刷新 |
| 派生数据 | `@Computed` | 只读派生 | 依赖变化才重算 |
| 变化监听 | `@Monitor` / `@SyncMonitor` | 只读监听 | 命中路径触发回调 |
| 应用级内存状态 | `AppStorageV2` | 全局 | 关联组件刷新 |
| 应用级持久化 | `PersistenceV2` | 全局且落盘 | 关联组件刷新 |
| 长列表复用 | `@ReusableV2` | 复用池 | 复用实例减少创建 |

## 4. 核心组件

```ts
@Entry
@ComponentV2
struct ComponentsDemo {
  @Local clicked: boolean = false;

  build() {
    Column({ space: 16 }) {
      Text('标题').fontSize(20).fontWeight(FontWeight.Bold)
      Image($r('app.media.logo')).width(64).height(64)
      TextInput({ placeholder: '请输入金额' })
        .type(InputType.Number)
        .onChange((value: string) => {})
      Button('提交')
        .onClick(() => { this.clicked = true; })
      Toggle({ type: ToggleType.Switch, isOn: this.clicked })
      Progress({ value: 60, total: 100, type: ProgressType.Linear })
    }
    .padding(16)
    .width('100%')
  }
}
```

## 5. 布局

| 容器 | 用途 |
|---|---|
| `Column` / `Row` | 纵向 / 横向线性布局 |
| `Stack` | 层叠布局，控制 Z 序 |
| `Flex` | 弹性布局，支持换行 |
| `Grid` / `GridItem` | 网格布局 |
| `List` / `ListItem` | 列表，配合 `LazyForEach` 性能优化 |
| `RelativeContainer` | 相对定位，适合复杂自适应 |
| `GridRow` / `GridCol` | 响应式栅格，适配多设备断点 |

```ts
Column({ space: 8 }) {
  Row({ space: 8 }) {
    Text('设置').layoutWeight(1)
    Toggle({ type: ToggleType.Switch, isOn: false })
  }
  .width('100%')
  .padding(16)
  .backgroundColor($r('app.color.card_bg'))
  .borderRadius(12)
}
.width('100%')
.padding(16)
```

## 6. 状态管理 V2 详解

### 6.1 `@Local`：组件内部状态

```ts
@ComponentV2
struct Counter {
  @Local count: number = 0;

  build() {
    Row({ space: 12 }) {
      Button('-').onClick(() => { this.count -= 1; })
      Text(`${this.count}`).fontSize(20)
      Button('+').onClick(() => { this.count += 1; })
    }
  }
}
```

`@Local` 只能被本组件修改，不能被外部初始化。需要外部传初始值时用 `@Param` + `@Once`。

### 6.2 `@Param` / `@Event`：父子单向与事件

```ts
@ComponentV2
struct AmountRow {
  @Param amount: number = 0;                 // 外部输入，子组件只读
  @Event onAdd: (delta: number) => void = () => {};  // 组件输出

  build() {
    Row({ space: 12 }) {
      Text(`¥${this.amount.toFixed(2)}`).fontSize(18)
      Button('+10').onClick(() => this.onAdd(10))      // 不能直接改 @Param
    }
  }
}

@Entry
@ComponentV2
struct LedgerPage {
  @Local total: number = 0;

  build() {
    Column({ space: 16 }) {
      AmountRow({ amount: this.total, onAdd: (d: number) => { this.total += d; } })
    }
    .padding(16)
  }
}
```

要点：

- `@Param` 是单向输入，子组件**不能直接赋值**，修改必须通过 `@Event` 回调通知父组件。
- `@Param` 传复杂类型用引用，`@Prop` 是深拷贝；如果子组件需要隔离，可在子组件内部拷贝一份到 `@Local`。

### 6.3 `!!` 双向绑定语法糖

当子组件的 `@Event` 命名为 `$` + `@Param` 变量名时，父组件可用 `!!` 简化双向绑定（API 12+）。

```ts
@ComponentV2
struct Star {
  @Param value: number = 0;
  @Event $value: (val: number) => void = (val: number) => {};

  build() {
    Column() {
      Text(`${this.value}`)
      Button('子组件设为 10').onClick(() => this.$value(10))
    }
  }
}
```

```ts
// 不用语法糖：显式传 @Param 与 @Event
Star({ value: this.value, $value: (val: number) => { this.value = val; } })

// 用语法糖：自动初始化 @Param value 与 @Event $value
Star({ value: this.value!! })
```

`!!` 只是写法简化，本质仍是 `@Param` + `@Event`，命名规则（`$value` 对应 `value`）不能写错。

### 6.4 `@Once`：只初始化同步一次

`@Once` 必须与 `@Param` 连用，父组件值只在初始化时同步一次，之后父组件变化不再覆盖，子组件可本地修改。

```ts
@ComponentV2
struct CountInput {
  @Once
  @Param initial: number = 0;   // 初始化接收一次，之后归子组件所有

  build() {
    Row({ space: 8 }) {
      Button('-').onClick(() => { this.initial -= 1; })
      Text(`${this.initial}`)
      Button('+').onClick(() => { this.initial += 1; })
    }
  }
}
```

适用场景：编辑表单的初始值、一次性配置项。等价于 V1 中“`@State` 由外部初始化一次”的意图。

### 6.5 `@ObservedV2` + `@Trace`：深层观测与属性级刷新

```ts
@ObservedV2
class Transaction {
  @Trace id: string = '';
  @Trace amount: number = 0;
  @Trace category: string = '';
}

@ComponentV2
struct TransactionItem {
  @Param item: Transaction = new Transaction();

  build() {
    Row({ space: 8 }) {
      Text(this.item.category)
      Text(`¥${this.item.amount}`)
    }
  }
}
```

- `@ObservedV2` 只标记类“可观察”，**必须配合 `@Trace`** 标注需要观测的属性。
- 修改 `transaction.amount` 时，只有读取了该属性的组件刷新，实现属性级精准更新。
- V1 的 `@Observed` + `@ObjectLink` 只能观察顶层属性，嵌套层级受限；V2 无此限制。

### 6.6 `@Monitor` 与 `@SyncMonitor`：状态监听

```ts
@ComponentV2
struct Summary {
  @Local transactions: Transaction[] = [];
  @Local keyword: string = '';

  @Monitor('transactions', 'keyword')
  onChange(monitor: IMonitor): void {
    monitor.dirty.forEach((path: string) => {
      const info = monitor.value(path);
      console.info(`${path}: ${info?.before} -> ${info?.now}`);
    });
  }
}
```

- `@Monitor` 是异步监听，支持多个路径，可获取变化前/后值。
- `@SyncMonitor` 是同步监听，在变量变更时立即回调。
- 深层属性监听需配合 `@Trace`；一次事件中同一变量多次变化只触发一次，以最终值为准。

### 6.7 `@Computed`：计算属性

```ts
@ComponentV2
struct Ledger {
  @Local list: Transaction[] = [];

  @Computed
  get total(): number {
    return this.list.reduce((sum, t) => sum + t.amount, 0);
  }

  build() {
    // total 只依赖 list，list 不变则不重算，避免 build 中重复计算
    Text(`合计：${this.total}`)
  }
}
```

在 `build` 中多次使用 `this.total` 也不会重复计算，适合金额汇总、过滤结果这类派生数据。

### 6.8 `@Provider` / `@Consumer`：跨层共享

```ts
@ComponentV2
struct AppRoot {
  @Provider('theme') theme: string = 'light';
  build() { Column() { DeepChild() } }
}

@ComponentV2
struct DeepChild {
  @Consumer('theme') theme: string = 'light';
  build() { Text(`当前主题：${this.theme}`) }
}
```

- `@Provider` 在祖先声明，`@Consumer` 在后代按相同 key 关联，双向同步。
- 别名（字符串 key）必须一致，中间层组件无需感知。
- 适合路由栈（`NavPathStack`）、主题、断点等“子树全局”状态。

### 6.9 `@Type`：标记类属性类型

`@Type` 用于标注类属性的类型，主要用于嵌套类对象，保证类型信息在构造与序列化场景下正确。

```ts
class Address {
  city: string = '';
  detail: string = '';
}

@ObservedV2
class Profile {
  @Trace name: string = '';
  @Trace @Type(Address) address: Address = new Address();
}
```

### 6.10 `@ReusableV2`：组件复用

```ts
@ReusableV2
@ComponentV2
struct TransactionRow {
  @Param item: Transaction = new Transaction();
  build() {
    Row() { Text(this.item.category) }
  }
}
```

在 `LazyForEach` / `Repeat` 长列表中使用时，框架从复用池取实例，减少反复创建销毁的开销。配合 `aboutToReuse` 处理复用时的数据刷新。

### 6.11 应用级与持久化：`AppStorageV2` / `PersistenceV2` / `LocalStorage`

```ts
import { AppStorageV2, PersistenceV2 } from '@kit.ArkUI';

@ObservedV2
class UserSession {
  @Trace token: string = '';
  @Trace nickname: string = '';
}

// 应用级内存状态：进程存活期间有效
AppStorageV2.connect(UserSession, 'session', () => new UserSession());

// 应用级持久化：重启后仍在
PersistenceV2.connect(UserSession, 'settings', () => new UserSession());
```

| 作用域 | V1 方案 | V2 方案 |
|---|---|---|
| 应用级内存 | `AppStorage` + `@StorageLink` / `@StorageProp` | `AppStorageV2.connect` |
| 应用级持久化 | `PersistentStorage` | `PersistenceV2.connect` |
| 页面 / 能力级 | `LocalStorage` + `@LocalStorageLink` / `@LocalStorageProp` | `LocalStorage` 配合 `@ObservedV2` / `@Trace` |

实践建议：把频繁变化的字段拆成独立 key，避免一个对象变化带整棵依赖树刷新。

## 7. 状态管理 V1 简述（理解与迁移）

新代码用 V2，但读旧代码时需要认识 V1 装饰器：

```ts
@Entry
@Component
struct Legacy {
  @State count: number = 0;          // 可外部初始化
  @Prop title: string = '';           // 父 -> 子，深拷贝
  @Link shared: number;               // 双向同步
  @Provide('theme') theme: string = 'light';
  @Watch('count')
  onCountChange(): void {}            // 只监听顶层属性

  build() {
    Column() {
      Text(`${this.count}`)
      Child({ title: this.title, shared: $shared })
    }
  }
}

@Component
struct Child {
  @Prop title: string = '';
  @Link shared: number;
  @Consume('theme') theme: string = 'light';
  build() { Text(`${this.title}-${this.shared}-${this.theme}`) }
}
```

V1 局限（也是 V2 要解决的问题）：

- `@Watch` 只能监听变量本身及其第一层属性变化。
- `@State` / `@Observed` 只能观察顶层属性，嵌套属性变化不刷新。
- `@Link` 自动双向绑定，可控性弱；V2 改为显式 `@Param` + `@Event`。
- 输入输出未在组件上显式声明，不利于组件化。

## 8. 条件与循环渲染

```ts
Column() {
  if (this.state === 'loading') {
    LoadingProgress().width(48).height(48)
  } else if (this.state === 'error') {
    Text('加载失败').fontColor(Color.Red)
  } else {
    ForEach(this.list, (item: Transaction) => {
      TransactionItem({ item })
    }, (item: Transaction) => item.id)   // 第三个参数是稳定键
  }
}
```

| 渲染控制 | 用途 |
|---|---|
| `if / else if / else` | 条件渲染 |
| `ForEach` | 数组渲染，适合中小列表 |
| `LazyForEach` | 按需构建，适合长列表 |
| `Repeat` | 循环渲染（支持虚拟滚动与复用） |

`ForEach` 的键生成函数必须稳定且唯一，否则会出现列表项错乱与重复创建。

## 9. 通用装饰器详解

### 9.1 `@Builder` / `@LocalBuilder` / `@BuilderParam`

```ts
// 全局 @Builder：跨组件复用，数据只能靠参数传入（按值传递）
@Builder
function cardHeader(title: string) {
  Row() {
    Text(title).fontSize(16).fontWeight(FontWeight.Bold)
  }.width('100%')
}

@ComponentV2
struct Demo {
  build() {
    Column() {
      cardHeader('账户总览')   // 全局 @Builder 按名直接调用
    }
  }
}

// @BuilderParam：插槽，内容由父组件注入
@ComponentV2
struct Card {
  @BuilderParam header: () => void = this.defaultHeader;  // 插槽，带默认内容

  @Builder
  defaultHeader() {
    Text('默认标题')
  }

  build() {
    Column() {
      this.header()
      // 内容
    }
  }
}

@ComponentV2
struct DemoPage {
  @Builder
  customHeader() {
    Text('自定义标题').fontSize(18)
  }

  build() {
    Column() {
      Card({ header: this.customHeader })   // 注入自定义插槽内容
    }
  }
}
```

- `@Builder`：定义可复用 UI 片段，可在全局或组件内声明。
- `@BuilderParam`：在组件内引用一个 `@Builder`，实现类似“插槽”的能力。
- `@LocalBuilder`：在组件内声明并使用，**保留 `this` 指向当前组件**，适合需要访问组件状态的构建函数；`@Builder` 内的 `this` 需要额外处理。
- 传参方式决定刷新行为：按值传递（普通参数）时，状态变化不会刷新 Builder 内 UI；需要联动刷新时改用对象字面量按引用传递，如 `function cardHeader($$: { title: string })`，调用 `cardHeader({ title: this.title })`。
- 需要把全局 `@Builder` 跨文件当参数传递（存变量、进数组）时，用 `wrapBuilder()` 封装。

```ts
@ComponentV2
struct ListPanel {
  @Local items: number[] = [1, 2, 3];

  @LocalBuilder
  itemView(item: number) {
    Text(`${item}`).onClick(() => { this.items.push(item); })  // this 指向 ListPanel
  }

  build() {
    Column() {
      ForEach(this.items, (it: number) => { this.itemView(it) })
    }
  }
}
```

### 9.2 `@Styles` / `@Extend` / `@AnimatableExtend`

```ts
// @Styles：复用通用属性（仅通用属性，不能指定具体组件）
@Styles function cardStyle() {
  .width('100%')
  .padding(16)
  .backgroundColor($r('app.color.card_bg'))
  .borderRadius(12)
}

// @Extend：扩展指定组件的属性
@Extend(Text) function titleStyle() {
  .fontSize(18)
  .fontWeight(FontWeight.Bold)
  .fontColor($r('app.color.text_primary'))
}

// @AnimatableExtend：定义可参与动画的自定义属性
@AnimatableExtend(Text) function progressWidth(w: number) {
  .width(w)
}
```

```ts
// 使用
Column() { Text('标题').titleStyle() }.cardStyle()
Text('动画宽度').progressWidth(this.width).animation({ duration: 300, curve: Curve.EaseInOut })
```

### 9.3 `@Require` 与 `@Env`

```ts
@ComponentV2
struct TitleBar {
  @Require @Param title: string;   // 构造时必须传 title，否则编译报错
  build() { Text(this.title) }
}

// 使用：TitleBar({ title: '首页' }) 合法；TitleBar({}) 报错
```

`@Env` 用于读取系统环境变量（如深浅色模式）：

```ts
@ComponentV2
struct ThemedText {
  @Env('colorMode') colorMode: ColorMode = ColorMode.LIGHT;
  build() {
    Text('自适应文本')
      .fontColor(this.colorMode === ColorMode.DARK ? Color.white : Color.black)
  }
}
```

支持的环境变量以官方文档为准。

## 10. 动画入门

```ts
@Local scale: number = 1;

// 属性动画：状态变化时自动过渡
Text('卡片')
  .scale({ x: this.scale, y: this.scale })
  .animation({ duration: 300, curve: Curve.EaseInOut })
```

更复杂的可用 `animateTo` 显式驱动：`animateTo({ duration: 300 }, () => { this.scale = 1.2; })`。配合 `@AnimatableExtend` 可让自定义属性参与动画。

## 阶段验收

- 能解释声明式 UI“状态驱动刷新”的原理，并说出装饰器决定的三件事（来源、可写性、刷新对象）。
- 能区分并正确选择 V1 与 V2 装饰器，单组件内不混用。
- 能用 `@Local` / `@Param` / `@Event` / `!!` 组织单向与双向数据流。
- 能用 `@Once` 表达“只初始化一次”的语义。
- 能用 `@ObservedV2` + `@Trace` 实现嵌套对象属性级刷新。
- 能用 `@Monitor` / `@SyncMonitor` / `@Computed` 处理监听与派生数据。
- 能用 `@Provider` / `@Consumer` / `AppStorageV2` / `PersistenceV2` 管理跨层与全局状态。
- 能用 `@Type` / `@ReusableV2` 处理类型标注与组件复用。
- 能用 `@Builder` / `@LocalBuilder` / `@BuilderParam` / `@Styles` / `@Extend` / `@AnimatableExtend` / `@Require` / `@Env`。
- 能用 `ForEach` / `LazyForEach` / `Repeat` 与动画构建列表与可复用 UI。

## 动手任务

1. 把阶段零的页面改造成 V2 组件，拆分出至少一个 `@Param` 子组件。
2. 用 `@Param` + `@Event` 实现一个双向计数，再用 `!!` 语法糖改写一次。
3. 用 `@Once` + `@Param` 实现“编辑表单初始值只接收一次”。
4. 用 `@ObservedV2` / `@Trace` 建模 `Transaction`，验证深层属性修改只刷新对应组件。
5. 用 `@Computed` 计算总金额，用 `@Monitor` 打印多路径变化前后值。
6. 用 `@Provider` / `@Consumer` 共享主题，用 `AppStorageV2` / `PersistenceV2` 保存登录态。
7. 用 `@ReusableV2` + `LazyForEach` 渲染长列表并观察复用效果。
8. 用 `@Extend` / `@Styles` 抽取样式，用 `@AnimatableExtend` 让一个自定义宽度参与动画。

## 常见排错

| 现象 | 排查方向 |
|---|---|
| 界面不刷新 | 状态变量缺少装饰器；嵌套属性未用 `@Trace` |
| 报混用 V1/V2 错误 | 单组件内统一使用一套装饰器 |
| `@Param` 修改无效 | 子组件无权改 `@Param`，应用 `@Event` 或 `!!` 通知父组件 |
| `!!` 不生效 | `@Event` 是否命名为 `$` + `@Param` 名，二者是否配对 |
| `@Once` 仍被父组件覆盖 | `@Once` 是否与 `@Param` 连用，书写顺序是否正确 |
| `@Monitor` 深层不触发 | 目标属性是否加了 `@Trace` |
| `@Computed` 频繁重算 | 依赖状态是否被高频写入，考虑拆分状态 |
| 跨层状态不更新 | `@Provider` / `@Consumer` 的 key 是否一致 |
| 列表错乱 | `ForEach` 键不稳定或不唯一 |
| 复用后数据错乱 | `@ReusableV2` 组件是否在 `aboutToReuse` 中正确刷新 |

## 进入下一阶段的条件

你能用 V2 状态管理独立构建一个多组件、数据驱动的页面，能针对“来源 / 可写性 / 刷新对象”三点为每个状态选择正确装饰器，并能解释每种装饰器的适用场景与性能代价。此时进入 [阶段二：Stage 模型、Ability 与路由导航](./stage-2-ability-navigation.md)。
