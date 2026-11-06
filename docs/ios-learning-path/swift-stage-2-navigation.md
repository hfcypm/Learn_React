# 阶段二：导航、生命周期与数据流

**目标**：掌握 `NavigationStack`、Sheet、Tab 与页面传值，理清 `App` / `Scene` / `View` 生命周期，能把列表、详情、编辑连成完整流程。

**前置知识**：阶段一的 SwiftUI 状态与列表。

## 1. 场景与生命周期

```swift
@main
struct HabitTrackerApp: App {
  @Environment(\.scenePhase) private var scenePhase

  var body: some Scene {
    WindowGroup {
      HabitListView()
    }
    .onChange(of: scenePhase) { _, phase in
      if phase == .background {
        // 持久化未保存草稿、暂停计时器
      }
    }
  }
}
```

| 层级 | 职责 |
|---|---|
| `App` | 入口、依赖注入、`modelContainer` |
| `Scene` / `scenePhase` | 前台、后台、非活跃 |
| `View.task` / `onAppear` | 页面出现时加载 |
| `onDisappear` | 取消轻量工作；网络任务用 `task` 的自动取消 |

```swift
.task {
  await store.load()
}
.onAppear {
  // 只做轻量 UI 同步
}
```

优先 `.task`：视图消失时任务自动取消，避免 `onAppear` 里启动的请求泄漏。

## 2. NavigationStack

```swift
enum Route: Hashable {
  case detail(UUID)
  case settings
}

struct HabitListView: View {
  @State private var path: [Route] = []
  let habits: [Habit]

  var body: some View {
    NavigationStack(path: $path) {
      List(habits) { habit in
        NavigationLink(value: Route.detail(habit.id)) {
          Text(habit.title)
        }
      }
      .navigationTitle("习惯")
      .navigationDestination(for: Route.self) { route in
        switch route {
        case .detail(let id):
          HabitDetailView(habitID: id)
        case .settings:
          SettingsView()
        }
      }
    }
  }
}
```

要点：

- 用类型化 `value` + `navigationDestination`，避免 Storyboard segue 式的隐式跳转。
- `path` 可编码时能做状态恢复。
- 返回用 `path.removeLast()` 或系统返回手势，不要同时维护两套导航状态。

旧的 `NavigationView` 仅用于阅读 iOS 15/16 代码，新项目不用。

## 3. 传值与返回刷新

正向传值：传 `id` 或只读模型，详情页自己查数据。

```swift
struct HabitDetailView: View {
  let habitID: UUID
  let habits: [Habit]
  var onCheckIn: (UUID) -> Void

  var habit: Habit? {
    habits.first { $0.id == habitID }
  }

  var body: some View {
    if let habit {
      VStack(spacing: 16) {
        Text(habit.title).font(.largeTitle)
        Text("连续 \(habit.streak) 天")
        Button("打卡") { onCheckIn(habit.id) }
      }
    } else {
      ContentUnavailableView("习惯不存在", systemImage: "questionmark")
    }
  }
}
```

反向刷新：数据源放在栈外的 `@Observable` 或 SwiftData 容器里，详情修改后列表自动刷新。不要靠 `viewWillAppear` 手动 `reloadData` 的心智。

## 4. Sheet、全屏覆盖与弹窗

```swift
struct HabitListView: View {
  @State private var showEditor = false
  @State private var pendingDelete: Habit?

  var body: some View {
    List { /* ... */ }
      .toolbar {
        Button("新增") { showEditor = true }
      }
      .sheet(isPresented: $showEditor) {
        HabitEditorView()
      }
      .alert(
        "删除习惯",
        isPresented: Binding(
          get: { pendingDelete != nil },
          set: { if !$0 { pendingDelete = nil } }
        )
      ) {
        Button("删除", role: .destructive) {
          if let pendingDelete { delete(pendingDelete) }
        }
        Button("取消", role: .cancel) {}
      }
  }
}
```

| 呈现方式 | 适用 |
|---|---|
| `sheet` | 新增、筛选、短表单 |
| `fullScreenCover` | 引导页、沉浸流程 |
| `alert` | 确认删除、错误提示 |
| `confirmationDialog` | 多项操作选择 |

同一时间只呈现一个模态。连续弹窗要等上一个关闭。

## 5. TabView

```swift
TabView {
  HabitListView()
    .tabItem { Label("习惯", systemImage: "checkmark.circle") }
  StatsView()
    .tabItem { Label("统计", systemImage: "chart.bar") }
  SettingsView()
    .tabItem { Label("设置", systemImage: "gear") }
}
```

每个 Tab 内部再放 `NavigationStack`，不要把 `TabView` 塞进导航栈里。Tab 之间共享的数据放进 `App` 注入的环境。

## 6. 工具栏与编辑模式

```swift
.toolbar {
  ToolbarItem(placement: .topBarTrailing) {
    EditButton()
  }
  ToolbarItem(placement: .primaryAction) {
    Button("新增", systemImage: "plus") { showEditor = true }
  }
}
```

列表进入编辑模式后，`onDelete` / `onMove` 才会显示系统控件。自定义滑动操作：

```swift
.swipeActions(edge: .trailing) {
  Button("删除", role: .destructive) { delete(habit) }
  Button("置顶") { pin(habit) }
}
```

## 7. 可观察模型作为单一数据源

```swift
@Observable
final class HabitStore {
  var habits: [Habit] = []

  func checkIn(_ id: UUID) {
    guard let index = habits.firstIndex(where: { $0.id == id }) else { return }
    habits[index].streak += 1
  }

  func add(title: String) {
    habits.append(Habit(title: title))
  }
}

@main
struct HabitTrackerApp: App {
  @State private var store = HabitStore()

  var body: some Scene {
    WindowGroup {
      HabitListView()
        .environment(store)
    }
  }
}
```

页面读取：

```swift
struct HabitListView: View {
  @Environment(HabitStore.self) private var store

  var body: some View {
    List(store.habits) { habit in
      NavigationLink(value: habit.id) {
        Text(habit.title)
      }
    }
  }
}
```

`HabitStore` 是引用类型，列表与详情共享同一实例，打卡后两侧同时刷新。阶段三会把内存数组换成 SwiftData。

## 8. 键盘与输入

```swift
TextField("名称", text: $title)
  .submitLabel(.done)
  .onSubmit { save() }

.scrollDismissesKeyboard(.interactively)
```

多输入表单用 `Form` + `@FocusState` 管理焦点。避免在 `onAppear` 里抢焦点导致预览抖动。

## 阶段验收

- 能画出 `App` -> `scenePhase` -> `View.task` 的生命周期。
- 能用 `NavigationStack` + `navigationDestination` 完成列表到详情。
- 能用 Sheet 完成新增，用 alert 完成删除确认。
- 能用 `@Observable` 环境对象作为单一数据源，返回后列表自动刷新。
- 能组织 Tab + 各 Tab 内导航栈。

## 动手任务

1. 给习惯列表加上详情页，展示连续天数与打卡按钮。
2. 用 Sheet 实现新增习惯，保存后关闭并出现在列表。
3. 删除前弹出确认。
4. 增加设置 Tab，暂时放一个提醒开关。
5. 把数组提升到 `HabitStore`，用 `.environment` 注入。

## 常见排错

| 现象 | 排查方向 |
|---|---|
| 点了行不跳转 | 是否包在 `NavigationStack`，`value` 类型是否与 destination 一致 |
| 详情打卡列表不刷新 | 数据是否在共享的 `@Observable` / SwiftData 中 |
| Sheet 立刻关掉 | `isPresented` 是否被 body 重算重置 |
| 连续 alert 崩溃或被忽略 | 是否同时 present 两个模态 |
| Tab 里 push 后切 Tab 状态丢失 | 是否把导航状态放在 Tab 子树内并保持身份稳定 |

## 进入下一阶段的条件

你能独立完成列表 -> 详情 -> 编辑 -> 返回刷新，并能说清数据放在哪一层。此时进入 [阶段三：网络、持久化与并发](./swift-stage-3-data-network.md)。
