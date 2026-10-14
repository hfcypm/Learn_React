# 阶段二：Stage 模型、Ability 与路由导航

**目标**：理解 Stage 模型的运行时结构，掌握 UIAbility 生命周期与窗口管理，用 `Navigation` + `NavPathStack` 组织多页面导航与传参。

## 1. Stage 模型

Stage 模型是鸿蒙当前的推荐应用模型，围绕组件化的 Ability 组织运行时：

```text
应用(APP)
  └── Module(HAP)
        ├── UIAbility          带 UI 的入口能力（每个应用至少一个）
        ├── ExtensionAbility   无 UI 或特定场景的扩展能力
        └── AbilityStage       模块级运行时入口
```

对比旧 FA 模型：Stage 模型以 `UIAbility` 替代 `PageAbility`，窗口与页面解耦，支持多窗口、多实例与更好的进程模型。

| 概念 | 职责 |
|---|---|
| UIAbility | 承载一个 UI 实例，管理窗口与生命周期 |
| ExtensionAbility | 后台任务、卡片、输入法等扩展（如 `ServiceExtensionAbility`） |
| AbilityStage | 模块被加载时的入口，可做进程级初始化 |
| WindowStage | 窗口舞台，管理窗口与页面内容 |

## 2. UIAbility 生命周期

```text
onCreate -> onWindowStageCreate -> onForeground -> (运行)
    -> onBackground -> onWindowStageDestroy -> onDestroy
```

```ts
// entry/src/main/ets/entryability/EntryAbility.ets
import { AbilityConstant, UIAbility, Want } from '@kit.AbilityKit';
import { window } from '@kit.ArkUI';
import { hilog } from '@kit.PerformanceAnalysisKit';

export default class EntryAbility extends UIAbility {
  onCreate(want: Want, launchParam: AbilityConstant.LaunchParam): void {
    // 进程内初始化；只做轻量、必要的工作
    hilog.info(0x0000, 'EntryAbility', 'onCreate');
  }

  onWindowStageCreate(windowStage: window.WindowStage): void {
    // 加载入口页面
    windowStage.loadContent('pages/Index', (err) => {
      if (err.code) {
        hilog.error(0x0000, 'EntryAbility', 'loadContent failed: %{public}s', JSON.stringify(err));
      }
    });
  }

  onForeground(): void {
    // 回到前台：恢复刷新、重连
  }

  onBackground(): void {
    // 退到后台：在这里持久化关键状态（onDestroy 不保证被调用）
  }

  onWindowStageDestroy(): void {
    // 释放与窗口相关资源
  }

  onDestroy(): void {
    // 释放资源
  }
}
```

要点：

- 重初始化放 `onCreate`，避免堆在 `onWindowStageCreate` 拖慢冷启动。
- 关键状态在 `onBackground` 保存，系统可能在资源紧张时直接杀进程，`onDestroy` 不保证执行。
- `onWindowStageCreate` 是窗口内容加载的关键节点。

## 3. Context 与 Want

- **Context**：能力上下文，用于获取资源、启动其他能力、访问数据库等。
  ```ts
  import { common } from '@kit.AbilityKit';
  const context = getContext(this) as common.UIAbilityContext;
  ```
- **Want**：能力启动意图，描述启动哪个 Ability、带什么参数。
  ```ts
  const want: Want = {
    bundleName: 'com.example.myapp',
    abilityName: 'EntryAbility',
    parameters: { from: 'detail' }
  };
  context.startAbility(want);
  ```

## 4. 路由方案：Navigation 优先

鸿蒙有两套路由：

| 方案 | 定位 | 建议 |
|---|---|---|
| `Router` | 页面级路由，基于 `@Entry` 页面 URL | 不推荐在新项目使用 |
| `Navigation` + `NavPathStack` | 组件级路由，栈式导航 | 推荐，支持分栏、动态加载、精细生命周期 |

### 4.1 Navigation 基本用法

```ts
// pages/Index.ets
@Entry
@ComponentV2
struct Index {
  @Provider('navStack') navStack: NavPathStack = new NavPathStack();

  build() {
    Navigation(this.navStack) {
      Column({ space: 12 }) {
        Text('首页').fontSize(20)
        Button('查看详情')
          .onClick(() => {
            this.navStack.pushPathByName('DetailPage', 'from-index');
          })
      }
      .padding(16)
    }
    .title('记账')
    .mode(NavigationMode.Stack)
  }
}
```

### 4.2 声明 NavDestination 与路由映射

```ts
// pages/DetailPage.ets
@ComponentV2
export struct DetailPage {
  @Consume('navStack') navStack: NavPathStack;

  build() {
    NavDestination() {
      Column({ space: 12 }) {
        Text(`参数：${this.navStack.getParamByName('DetailPage') ?? ''}`)
        Button('返回').onClick(() => this.navStack.pop())
      }
      .padding(16)
    }
    .title('详情')
    .onShown(() => { /* 页面可见 */ })
    .onHidden(() => { /* 页面隐藏 */ })
    .onReady((ctx: NavDestinationContext) => {
      // 可在此安全访问 pathStack 与参数
    })
  }
}
```

用 `@Builder` 注册路由表并挂到 `Navigation`：

```ts
@Builder
export function PageMap(name: string) {
  if (name === 'DetailPage') {
    DetailPage()
  }
}

// 使用
Navigation(this.navStack) {
  // 首页内容
}
.navDestination(PageMap)
```

### 4.3 传参与返回值

```ts
// 传基础类型
this.navStack.pushPathByName('DetailPage', 'hello');

// 传复杂对象：先序列化，避免可观察对象的内部字段被污染
this.navStack.pushPathByName('DetailPage', JSON.stringify({ id: 'T1', amount: 99 }));

// 接收方解析
const raw = this.navStack.getParamByName('DetailPage') as string;
const payload = JSON.parse(raw) as Record<string, Object>;

// 带返回值
this.navStack.pushPathByName('EditPage', null, (popInfo: PopInfo) => {
  console.info(`返回结果：${JSON.stringify(popInfo.result)}`);
});
// 子页面
this.navStack.pop({ result: true });
```

注意：`Navigation` 传参底层是引用传递，`Router` 是深拷贝。被 `@ObservedV2` / `@Trace` 装饰的对象序列化后属性名可能被加前缀，跨页面传复杂对象优先 `JSON.stringify`。

### 4.4 页面栈管理

```ts
this.navStack.pushPathByName('DetailPage', null); // 入栈
this.navStack.pop();                               // 出栈
this.navStack.popToName('HomePage');               // 回退到指定页
this.navStack.clear();                             // 清空栈
const size: number = this.navStack.size();         // 栈深度
```

## 5. 页面与组件生命周期

| 回调 | 触发时机 |
|---|---|
| `aboutToAppear` | 组件即将出现，可做数据初始化 |
| `aboutToDisappear` | 组件即将销毁，释放资源 |
| `onPageShow` | 页面每次显示（含从后台返回） |
| `onPageHide` | 页面每次隐藏 |
| `onBackPress` | 物理返回键，返回 `true` 拦截 |
| `onShown` / `onHidden` | NavDestination 页面可见 / 隐藏 |

```ts
@Entry
@ComponentV2
struct Ledger {
  @Local list: Transaction[] = [];

  aboutToAppear(): void {
    // 首次加载数据
  }

  onPageShow(): void {
    // 每次可见刷新（例如从编辑页返回）
  }

  onBackPress(): boolean {
    // 返回 true 表示自行处理返回逻辑
    return false;
  }
}
```

## 阶段验收

- 能画出 Stage 模型的运行时结构与 UIAbility 生命周期。
- 能解释 `onCreate` / `onWindowStageCreate` / `onForeground` / `onBackground` 各自的职责。
- 能用 `Want` 启动另一个能力，能获取 `Context`。
- 能用 `Navigation` + `NavPathStack` 完成多页面跳转、传参与返回。
- 能用 `@Provider` / `@Consumer` 在页面树中共享 `NavPathStack`。
- 能区分页面生命周期与组件生命周期。

## 动手任务

1. 在 `EntryAbility` 的每个生命周期回调打印日志，观察前台/后台切换顺序。
2. 在 `onBackground` 保存一个标记，`onForeground` 读取并打印。
3. 用 `Navigation` 实现“首页 -> 详情 -> 编辑”三级导航。
4. 传入一个复杂对象（序列化），并在详情页解析显示。
5. 从编辑页 `pop` 返回结果，在首页接收并更新状态。
6. 用 `onPageShow` 实现从后台返回时刷新列表。

## 常见排错

| 现象 | 排查方向 |
|---|---|
| 冷启动慢 | 初始化是否堆在 `onWindowStageCreate`；改到 `onCreate` 并异步化 |
| 数据丢失 | 是否只在 `onDestroy` 保存；改到 `onBackground` |
| 路由找不到 | `navDestination` 路由表名称与 `pushPathByName` 是否一致 |
| 传参丢失 | 复杂对象是否序列化；可观察对象字段是否被改写 |
| 页面不刷新 | 是否用了 `onPageShow` 而非仅 `aboutToAppear` |
| 返回后状态错乱 | `pop` 返回值回调是否处理；栈是否越堆越深 |

## 进入下一阶段的条件

你能独立组织一个多页面应用的能力生命周期与导航流，并能解释页面与能力各自的边界。此时进入 [阶段三：网络、数据持久化与工程化](./stage-3-engineering.md)。
