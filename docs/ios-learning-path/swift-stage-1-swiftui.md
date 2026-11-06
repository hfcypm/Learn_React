# 阶段一：SwiftUI 声明式 UI 与状态

**目标**：掌握 SwiftUI 的视图、布局与状态来源，理解每个属性包装器的数据流方向与刷新范围，能构建数据驱动刷新的可交互界面。

**前置知识**：阶段零的 Swift 语法、`App` / `View` 与值类型。

## 1. 声明式 UI 范式

SwiftUI 中界面是状态的函数。你描述“界面应该长什么样”，状态变化时框架对依赖该状态的视图做差量更新。

```text
状态变化 -> 框架标记依赖该状态的 View -> 重新计算 body -> 差量提交到渲染树
```

三个基本概念：

- 视图（View）：界面构建块，如 `Text`、`Button`、`List`。
- 状态（State）：驱动刷新的数据，用属性包装器标注。
- `body`：声明视图树的计算属性。

属性包装器决定三件事：数据从哪来、谁能改它、变化会刷新谁。

## 2. 状态来源清单

iOS 17+ 主线使用 Observation。单视图内不要混用两套观察体系。

| 包装器 | 作用 | 谁拥有 | 谁能改 |
|---|---|---|---|
| `@State` | 视图私有状态 | 当前 View | 当前 View |
| `@Binding` | 读写外部状态的入口 | 父级 | 子级通过绑定改父级 |
| `@Bindable` | 把 `@Observable` 对象拆成绑定 | 可观察对象 | 子级改对象属性 |
| `@Observable` | 标记类的属性可观察 | 类实例 | 持有者 |
| `@Environment` | 从环境读取依赖 | 祖先注入 | 视具体类型 |
| `@Query` | SwiftData 查询结果 | 模型容器 | 通过 context 改模型 |
| `@AppStorage` | UserDefaults 绑定 | 系统偏好 | 当前视图 |

对照（iOS 16 及更早，仅作阅读存量代码）：

| 旧写法 | 新写法 |
|---|---|
| `ObservableObject` + `@Published` | `@Observable` |
| `@StateObject` / `@ObservedObject` | `@State` 持有 `@Observable` 实例 |
| `@EnvironmentObject` | `@Environment` |

## 3. 最小可交互页面

```swift
import SwiftUI

struct HabitListView: View {
  @State private var habits: [Habit] = [
    Habit(title: "阅读"),
    Habit(title: "跑步")
  ]
  @State private var draft = ""

  var body: some View {
    NavigationStack {
      List {
        ForEach(habits) { habit in
          HStack {
            Text(habit.title)
            Spacer()
            Text("连续 \(habit.streak) 天")
              .foregroundStyle(.secondary)
          }
        }
        .onDelete { indexSet in
          habits.remove(atOffsets: indexSet)
        }
      }
      .navigationTitle("习惯")
      .safeAreaInset(edge: .bottom) {
        HStack {
          TextField("新习惯", text: $draft)
            .textFieldStyle(.roundedBorder)
          Button("添加") {
            let title = draft.trimmingCharacters(in: .whitespaces)
            guard !title.isEmpty else { return }
            habits.append(Habit(title: title))
            draft = ""
          }
        }
        .padding()
      }
    }
  }
}
```

要点：

- `$draft` 是 `Binding<String>`，`TextField` 通过绑定双向更新。
- `ForEach` 依赖 `Identifiable`，用稳定 `id`，不用数组下标。
- 列表编辑用 `.onDelete`，不要手写 index 运算散落各处。

## 4. 布局

常用容器：

| 容器 | 用途 |
|---|---|
| `VStack` / `HStack` / `ZStack` | 垂直 / 水平 / 层叠 |
| `List` | 系统列表，自带分割与滑动删除 |
| `Form` | 设置页与输入分组 |
| `ScrollView` | 自定义滚动内容 |
| `LazyVStack` / `LazyHGrid` | 长列表懒加载 |
| `Grid` | 规则二维布局 |

```swift
VStack(alignment: .leading, spacing: 12) {
  Text(habit.title)
    .font(.headline)
  Text("连续 \(habit.streak) 天")
    .font(.subheadline)
    .foregroundStyle(.secondary)
}
.padding()
.frame(maxWidth: .infinity, alignment: .leading)
```

布局原则：

- 用 `padding` / `frame` / `alignment` 表达约束，避免写死像素坐标。
- `Spacer` 吃掉剩余空间；`fixedSize` 阻止压缩。
- 安全区用 `.safeAreaInset` 或忽略安全区时明确说明原因。
- 动态字体用 `.font(.headline)` 这类文本样式，便于辅助功能缩放。

## 5. 组件拆分与单向数据流

```swift
struct HabitRow: View {
  let habit: Habit
  var onCheckIn: () -> Void

  var body: some View {
    HStack {
      VStack(alignment: .leading) {
        Text(habit.title)
        Text("连续 \(habit.streak) 天")
          .font(.caption)
          .foregroundStyle(.secondary)
      }
      Spacer()
      Button("打卡", action: onCheckIn)
        .buttonStyle(.borderedProminent)
    }
  }
}
```

父视图持有数组，子视图只读 `habit`，通过闭包把事件传回父级。子视图不直接改父级数组。

需要子视图改值类型字段时用 `@Binding`：

```swift
struct HabitEditor: View {
  @Binding var habit: Habit

  var body: some View {
    TextField("名称", text: $habit.title)
  }
}
```

需要子视图改 `@Observable` 对象时用 `@Bindable`：

```swift
@Observable
final class Draft {
  var title = ""
}

struct TitleField: View {
  @Bindable var draft: Draft

  var body: some View {
    TextField("名称", text: $draft.title)
  }
}
```

## 6. 列表、空态与加载态

```swift
enum LoadState {
  case idle
  case loading
  case empty
  case loaded
  case failed(String)
}

struct HabitListContainer: View {
  let state: LoadState
  let habits: [Habit]

  var body: some View {
    switch state {
    case .idle, .loading:
      ProgressView("加载中")
    case .empty:
      ContentUnavailableView("还没有习惯", systemImage: "checkmark.circle", description: Text("点右上角添加第一条"))
    case .loaded:
      List(habits) { habit in
        Text(habit.title)
      }
    case .failed(let message):
      ContentUnavailableView("加载失败", systemImage: "wifi.slash", description: Text(message))
    }
  }
}
```

空态、加载、失败都要有界面，不要只留空白 `List`。

## 7. 系统控件

```swift
Toggle("提醒", isOn: $enabled)
DatePicker("提醒时间", selection: $time, displayedComponents: .hourAndMinute)
Picker("分组", selection: $group) {
  Text("全部").tag("all")
  Text("晨间").tag("morning")
}
.pickerStyle(.segmented)
```

表单页用 `Form` 分组；操作页用 `List` 或自定义 `ScrollView`。按钮要有明确的角色：`.borderedProminent` 表示主操作。

## 8. 深色模式、动态字体与无障碍

```swift
Text(habit.title)
  .font(.headline)
  .foregroundStyle(.primary)

Image(systemName: "checkmark.circle.fill")
  .accessibilityLabel("已打卡")
```

- 颜色优先语义色：`.primary` / `.secondary` / `Color.accentColor`。
- 自定义色放进 Assets 的 Color Set，勾选 Any/Dark。
- 图标按钮必须有 `accessibilityLabel`。
- 点击区域不小于 44pt，用 `.frame(minHeight: 44)` 保证。

## 阶段验收

- 能解释“状态驱动刷新”，并说清 `@State` / `@Binding` / `@Observable` 的职责。
- 能用 `List` / `ForEach` 展示数据，稳定 `id`，处理删除。
- 能拆分子视图，用闭包或 `@Binding` 保持单向数据流。
- 能处理空态、加载态与失败态。
- 界面在深色模式与动态字体下可用。

## 动手任务

1. 把阶段零的 `ContentView` 改成习惯列表，支持添加与滑动删除。
2. 抽出 `HabitRow`，打卡按钮通过闭包把事件传回列表。
3. 增加空态：数组为空时显示 `ContentUnavailableView`。
4. 用 `#Preview` 分别预览空态与两条数据。
5. 打开深色模式与更大字体，检查截断与对比度。

## 常见排错

| 现象 | 排查方向 |
|---|---|
| 输入框改了界面不更新 | 是否用了 `$` 绑定，属性是否 `@State` |
| 改了结构体副本没反应 | 改的是拷贝，应改 `@State` 数组中的元素 |
| 列表行错乱 | `id` 是否稳定，是否用了下标当 id |
| 子视图改值无效 | 值类型需要 `@Binding`，可观察对象需要 `@Bindable` |
| Preview 崩溃 | Preview 是否构造了完整状态 |

## 进入下一阶段的条件

你能独立写出带列表、表单、空态的页面，并能说清每个状态包装器的数据流方向。此时进入 [阶段二：导航、生命周期与数据流](./swift-stage-2-navigation.md)。
