# 阶段零：Swift 语言与 Xcode 工程

**目标**：建立对 Swift、工程结构和工具链的正确认知，掌握类型系统、可选值与值类型，跑通并读懂第一个 SwiftUI 应用。

**前置知识**：一门静态类型语言的变量、函数、结构体/类基础。

## 1. Swift 与 iOS 工程是什么

Swift 是 Apple 平台的主开发语言。新 iOS 应用用 Swift 写逻辑，用 SwiftUI 描述界面，用 Xcode 构建与调试。

| 维度 | 新项目主线 | 存量工程常见形态 |
|---|---|---|
| 语言 | Swift | Objective-C |
| UI 框架 | SwiftUI | UIKit + Storyboard |
| 状态 | `@Observable` / `@State` | 属性 + 手动刷新 |
| 持久化 | SwiftData | Core Data |
| 依赖 | SPM | CocoaPods |
| 入口 | `@main` 的 `App` | `AppDelegate` / `SceneDelegate` |

本阶段只要求把工程跑起来，并写对基础 Swift。界面细节在阶段一展开。

## 2. 先看一个能跑的最小程序

```swift
import SwiftUI

@main
struct HabitTrackerApp: App {
  var body: some Scene {
    WindowGroup {
      ContentView()
    }
  }
}

struct ContentView: View {
  var body: some View {
    Text("Hello, Swift")
      .font(.title)
      .padding()
  }
}
```

- `@main` 标记程序入口，对应 UIKit 时代的 `UIApplicationMain`。
- `App` 的 `body` 返回场景；`WindowGroup` 是最常见的窗口场景。
- `View` 的 `body` 描述界面，类型是 `some View`（不透明返回类型）。
- 修饰符如 `.font` / `.padding` 返回新视图，不就地修改原视图。

命令行验证 Swift 语法：

```swift
// hello.swift
print("Hello, \(name)")
```

```bash
swift hello.swift
```

App 工程的业务代码从 `HabitTrackerApp` 开始，而不是手写 `main`。

## 3. 工程结构

用 Xcode 新建 `App`（SwiftUI）工程后，关键文件如下：

```text
HabitTracker/
├── HabitTracker.xcodeproj
├── HabitTracker/
│   ├── HabitTrackerApp.swift      # @main 入口
│   ├── ContentView.swift          # 根视图
│   ├── Assets.xcassets            # 颜色、图片、App 图标
│   └── Preview Content/           # 预览资源
├── HabitTrackerTests/
└── HabitTrackerUITests/
```

后续会补：

| 文件 | 作用 |
|---|---|
| `Info.plist` / target Info | 权限、ATS、场景配置 |
| `PrivacyInfo.xcprivacy` | 隐私清单，审核必填项 |
| `*.xcdatamodeld` 或 SwiftData schema | 本地模型 |
| `Package.swift` 或 Package Dependencies | SPM 依赖 |

### 3.1 Bundle Identifier 与 Deployment Target

- Bundle Identifier：应用唯一标识，签名与上架都依赖它，格式如 `com.example.habittracker`。
- Deployment Target：支持的最低系统版本。本路线以 **iOS 17** 为基线，才能直接使用 `@Observable` 与 SwiftData。
- 版本号：`MARKETING_VERSION`（用户可见，如 `1.0.0`）与 `CURRENT_PROJECT_VERSION`（构建号，整数递增）。

### 3.2 预览

```swift
#Preview {
  ContentView()
}
```

预览在同一进程渲染当前视图，依赖环境值时必须在 Preview 里提供，否则空白或崩溃。

## 4. Swift 核心语法

### 4.1 常量、变量与类型推断

```swift
let title = "阅读"
var streak = 0
let enabled: Bool = true
```

`let` 不可变，`var` 可变。优先 `let`。类型可推断，公开 API 仍建议显式标注。

### 4.2 可选值

```swift
var note: String? = nil
note = "晨间"

if let note {
  print(note)
}

let display = note ?? "无备注"
print(note?.count ?? 0)
```

- `?` 表示可空。
- `if let` / `guard let` 解包。
- `??` 提供默认值。
- `!` 强制解包只在你能证明非空时使用，学习阶段避免。

```swift
func loadTitle(_ raw: String?) throws -> String {
  guard let raw, !raw.isEmpty else {
    throw HabitError.emptyTitle
  }
  return raw
}
```

### 4.3 结构体、类与枚举

```swift
struct Habit: Identifiable, Hashable {
  let id: UUID
  var title: String
  var streak: Int

  init(title: String, streak: Int = 0) {
    self.id = UUID()
    self.title = title
    self.streak = streak
  }
}

final class HabitStore {
  var items: [Habit] = []
}

enum HabitError: Error {
  case emptyTitle
  case network(status: Int)
}
```

| 类型 | 语义 | 适用 |
|---|---|---|
| `struct` | 值类型，赋值拷贝 | 模型、视图状态 |
| `class` | 引用类型，共享实例 | 需要身份标识的服务对象 |
| `enum` | 有限状态 | 错误、页面、加载态 |
| `actor` | 隔离可变状态 | 并发下的共享资源 |

SwiftUI 视图本身是 `struct`。可观察的共享状态用 `@Observable class`。

### 4.4 协议、扩展与泛型

```swift
protocol Titlable {
  var title: String { get }
}

extension Habit: Titlable {}

func firstTitle<T: Titlable>(of items: [T]) -> String? {
  items.first?.title
}
```

协议描述能力，扩展给已有类型加方法。`Identifiable` 让 `ForEach` 能稳定识别行。

### 4.5 错误处理与 Result

```swift
func fetchCount() async throws -> Int {
  3
}

Task {
  do {
    let count = try await fetchCount()
    print(count)
  } catch {
    print(error.localizedDescription)
  }
}
```

异步函数用 `async throws`，调用处 `try await`。视图层把错误转成用户可读文案。

### 4.6 闭包与 trailing closure

```swift
let sorted = habits.sorted { $0.streak > $1.streak }
let titles = habits.map(\.title)
```

闭包作为最后一个参数时可写在括号外。捕获 `self` 的闭包在类中可能延长生命周期，阶段三会用 `[weak self]`。

## 5. 值类型与引用类型

```swift
var a = Habit(title: "阅读")
var b = a
b.title = "跑步"
print(a.title) // 阅读

let store = HabitStore()
let alias = store
alias.items.append(Habit(title: "冥想"))
print(store.items.count) // 1
```

结构体拷贝后独立；类共享同一实例。这是理解 `@State`（持有值）与 `@Observable`（共享引用）的前提。

## 6. 工具链

```bash
# 列出模拟器
xcrun simctl list devices available

# 命令行构建（把 scheme 换成你的工程名）
xcodebuild -scheme HabitTracker \
  -destination 'platform=iOS Simulator,name=iPhone 16' \
  build
```

| 操作 | 方式 |
|---|---|
| 运行 | 选择模拟器，`Cmd + R` |
| 预览 | Canvas 或 `#Preview` |
| 清理 | `Cmd + Shift + K` |
| 真机 | 选择设备，配置 Signing Team |
| 依赖 | File -> Add Package Dependencies |

## 阶段验收

- 能解释 Swift、SwiftUI、Xcode、Bundle Identifier、Deployment Target 的定位。
- 能读懂 `HabitTrackerApp.swift` 与 `ContentView.swift`。
- 能用 `let` / `var`、可选值、`struct` / `enum` / `protocol` 写出类型安全的代码。
- 能说明值类型与引用类型的拷贝差异。
- 能创建工程并跑通首个页面，使用 `#Preview`。

## 动手任务

1. 新建 SwiftUI App 工程并运行到模拟器。
2. 修改 Bundle Identifier 与 Display Name，观察安装后的图标名。
3. 写一个 `Habit` 结构体与 `HabitError` 枚举，在 `ContentView` 显示标题。
4. 用 `if let` 与 `guard let` 各写一次可选值解包。
5. 给工程加上 `#Preview`，确认 Canvas 能刷新。

## 常见排错

| 现象 | 排查方向 |
|---|---|
| 编译报 iOS 17 API 不可用 | Deployment Target 是否低于 17 |
| Preview 空白 | 是否缺少环境依赖，Canvas 是否选中当前文件 |
| `optional must be unwrapped` | 用 `if let` / `??`，避免 `!` |
| 改了 `struct` 副本界面没变 | 是否改的是拷贝而不是 `@State` 里的源数据 |
| 真机安装失败 | Signing Team、Bundle ID、设备信任 |

## 进入下一阶段的条件

你能独立创建并运行工程，能解释入口文件的作用，并能用 Swift 写出可选值与结构体都正确的基础代码。此时进入 [阶段一：SwiftUI 声明式 UI 与状态](./swift-stage-1-swiftui.md)。
