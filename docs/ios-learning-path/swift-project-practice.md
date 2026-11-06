# 综合实战：习惯打卡 App

## 1. 项目目标

用 Swift + SwiftUI + SwiftData 从零构建 `HabitTracker`，覆盖入门到进阶的全部能力：工程搭建、声明式 UI 与状态、导航与数据流、网络与持久化、分层与测试、小组件与上架。项目按阶段扩展，最终交付一款可在 iPhone 运行、可上 TestFlight 的应用。

```text
初始化工程 -> 界面与状态 -> 多页面导航
    -> 网络与本地缓存 -> 分层测试 -> 小组件 / 通知 / 隐私
    -> 打包发布
```

选择“习惯打卡”的原因：数据结构简单（习惯 + 打卡日期），天然覆盖列表、表单、导航、网络同步、本地数据库、并发与发布，能串起本路线全部知识点。与 OC 支线的 [维护老版习惯打卡 App](./project-practice.md) 是同一产品主题、两套技术栈，方便对照。

## 2. 需求

**必须实现（MVP）**

- 习惯列表：展示标题与连续打卡天数，空态提示。
- 新增习惯：输入名称，保存后列表刷新。
- 打卡：点击后连续天数加一，重启后不丢失。
- 页面跳转：列表 -> 详情 -> 编辑，能返回并刷新。
- 本地持久化：SwiftData 存储习惯与打卡日期。

**进阶**

- 网络同步：从服务端拉取习惯，弱网时用本地缓存。
- 本地通知：每天固定时间提醒打卡。
- 数据迁移：为模型新增字段，验证老数据升级不丢失。
- 分层：Store / Service 与视图分离，控制器式逻辑不进 `body`。
- Widget：主屏幕展示今日已打卡数量。
- App Intent：快捷指令给指定习惯打卡。
- 发布：签名并上传 TestFlight。

## 3. 技术选型

| 技术 | 用途 |
|---|---|
| Swift 5.9+ | 开发语言 |
| SwiftUI | 声明式界面与导航 |
| `@Observable` | 共享状态 |
| SwiftData | 本地持久化 |
| URLSession + Codable | 网络与解析 |
| UserNotifications | 打卡提醒 |
| AppStorage / Keychain | 偏好与敏感信息 |
| Swift Concurrency | 异步与主线程纪律 |
| SPM | 三方依赖 |
| XCTest | 单元与 UI 测试 |
| WidgetKit / App Intents | 小组件与快捷指令 |
| Xcode Organizer | Archive 与 TestFlight |

## 4. 项目结构

```text
HabitTracker/
  HabitTrackerApp.swift
  Features/
    Habits/
      HabitListView.swift
      HabitDetailView.swift
      HabitEditorView.swift
      HabitRow.swift
    Settings/
      SettingsView.swift
    Stats/
      StatsView.swift
  Models/
    Habit.swift
  Services/
    HabitClient.swift
    HabitRepository.swift
    ReminderScheduler.swift
    KeychainStore.swift
  Support/
    Formatters.swift
    HabitTracker.entitlements
  Assets.xcassets
  Localizable.xcstrings
  PrivacyInfo.xcprivacy
  HabitTrackerTests/
  HabitTrackerUITests/
  StreakWidget/                 # 进阶
```

## 5. 阶段零：初始化（工程与 Swift）

新建 iOS App，Interface 选 SwiftUI，Language 选 Swift。Deployment Target 设为 iOS 17。

```swift
import Foundation

struct HabitDraft: Identifiable, Hashable {
  let id: UUID
  var title: String
  var streak: Int

  init(title: String, streak: Int = 0) {
    self.id = UUID()
    self.title = title
    self.streak = streak
  }
}

enum HabitError: LocalizedError {
  case emptyTitle

  var errorDescription: String? {
    switch self {
    case .emptyTitle: "名称不能为空"
    }
  }
}
```

```swift
@main
struct HabitTrackerApp: App {
  var body: some Scene {
    WindowGroup {
      ContentView()
    }
  }
}
```

验收：工程可运行，预览正常，模型类型安全。

## 6. 阶段一：界面与状态

首页展示习惯列表，支持空态、添加与滑动删除。

```swift
struct HabitListView: View {
  @State private var habits: [HabitDraft] = []
  @State private var draft = ""

  var body: some View {
    NavigationStack {
      Group {
        if habits.isEmpty {
          ContentUnavailableView(
            "还没有习惯",
            systemImage: "checkmark.circle",
            description: Text("在下方输入名称并添加")
          )
        } else {
          List {
            ForEach(habits) { habit in
              HabitRow(habit: habit) {
                checkIn(habit.id)
              }
            }
            .onDelete { habits.remove(atOffsets: $0) }
          }
        }
      }
      .navigationTitle("习惯")
      .safeAreaInset(edge: .bottom) {
        HStack {
          TextField("新习惯", text: $draft)
            .textFieldStyle(.roundedBorder)
          Button("添加", action: add)
            .disabled(draft.trimmingCharacters(in: .whitespaces).isEmpty)
        }
        .padding()
      }
    }
  }

  private func add() {
    let title = draft.trimmingCharacters(in: .whitespaces)
    guard !title.isEmpty else { return }
    habits.append(HabitDraft(title: title))
    draft = ""
  }

  private func checkIn(_ id: UUID) {
    guard let index = habits.firstIndex(where: { $0.id == id }) else { return }
    habits[index].streak += 1
  }
}
```

验收：状态驱动刷新正常，空态与列表切换正确，删除与打卡立即反映。

## 7. 阶段二：多页面导航

用 `NavigationStack` 实现列表 -> 详情，用 Sheet 实现新增 / 编辑。

```swift
enum Route: Hashable {
  case detail(UUID)
}

struct HabitListView: View {
  @Environment(HabitStore.self) private var store
  @State private var path: [Route] = []
  @State private var showEditor = false

  var body: some View {
    NavigationStack(path: $path) {
      List(store.habits) { habit in
        NavigationLink(value: Route.detail(habit.id)) {
          HabitRow(habit: habit) { store.checkIn(habit.id) }
        }
      }
      .navigationDestination(for: Route.self) { route in
        switch route {
        case .detail(let id):
          HabitDetailView(habitID: id)
        }
      }
      .sheet(isPresented: $showEditor) {
        HabitEditorView()
      }
      .toolbar {
        Button("新增", systemImage: "plus") { showEditor = true }
      }
      .navigationTitle("习惯")
    }
  }
}
```

```swift
@Observable
final class HabitStore {
  var habits: [HabitDraft] = []

  func checkIn(_ id: UUID) {
    guard let index = habits.firstIndex(where: { $0.id == id }) else { return }
    habits[index].streak += 1
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

验收：详情打卡后返回列表天数已更新；Sheet 保存后关闭并出现新行。

## 8. 阶段三：网络、SwiftData、缓存与通知

### 8.1 模型迁到 SwiftData

```swift
import SwiftData

@Model
final class Habit {
  var title: String
  var streak: Int
  var checkIns: [Date]
  var createdAt: Date

  init(title: String) {
    self.title = title
    self.streak = 0
    self.checkIns = []
    self.createdAt = .now
  }

  func markCheckedIn(on date: Date = .now) {
    let calendar = Calendar.current
    if let last = checkIns.max(), calendar.isDate(last, inSameDayAs: date) {
      return
    }
    checkIns.append(date)
    streak += 1
  }
}
```

```swift
.modelContainer(for: Habit.self)
```

列表改用 `@Query`。同一天重复打卡不叠加。

### 8.2 网络层

```swift
struct HabitDTO: Codable, Sendable {
  let id: UUID
  let title: String
  let streak: Int
}

struct HabitClient {
  func fetchHabits() async throws -> [HabitDTO] {
    let url = URL(string: "https://example.com/api/habits")!
    let (data, response) = try await URLSession.shared.data(from: url)
    let http = response as! HTTPURLResponse
    guard (200..<300).contains(http.statusCode) else {
      throw URLError(.badServerResponse)
    }
    return try JSONDecoder().decode([HabitDTO].self, from: data)
  }
}
```

### 8.3 缓存优先

```text
1. 启动先读 SwiftData，立即渲染列表
2. 后台请求服务端
3. 成功 -> 合并写入 SwiftData -> 界面自动刷新
4. 失败 -> 保留本地数据 -> 顶部提示“同步失败，可重试”
```

### 8.4 本地通知

设置页用 `@AppStorage("dailyReminderEnabled")`。打开时申请授权并注册每天 8:00 的重复通知；关闭时删除 identifier `habit.daily`。

验收：断网仍能查看与打卡；联网后能同步；授权后能收到提醒。

## 9. 阶段四：分层、测试与发布

### 9.1 Store 变薄视图

```swift
@MainActor
@Observable
final class HabitStore {
  var isLoading = false
  var errorMessage: String?
  private let client: any HabitFetching

  init(client: any HabitFetching = HabitClient()) {
    self.client = client
  }

  func load(context: ModelContext) async { /* 缓存优先 */ }
}
```

视图只绑定状态，不拼 URL。

### 9.2 单元测试

```swift
func testSameDayCheckInDoesNotIncreaseStreak() {
  let habit = Habit(title: "阅读")
  habit.markCheckedIn(on: Date())
  habit.markCheckedIn(on: Date())
  XCTAssertEqual(habit.streak, 1)
}
```

### 9.3 签名与 TestFlight

```text
1. Signing & Capabilities 选择 Team
2. 设置唯一 Bundle Identifier
3. 填写 PrivacyInfo.xcprivacy
4. Product -> Archive
5. Organizer -> Distribute App -> App Store Connect
6. TestFlight 内部测试
```

发布前检查：

- 无调试日志与硬编码测试地址。
- 权限与隐私声明一致。
- 深色模式、离线、异常网络都验证过。
- 崩溃与警告清零。

## 10. 阶段五：小组件与系统能力（进阶）

- Widget：展示今日已打卡数量，数据来自 App Group。
- App Intent：`CheckInIntent` 给指定习惯打卡，同一天幂等。
- 深链：`habit://detail/<uuid>` 打开详情。
- 本地化：String Catalog 增加 English。

这些项不阻塞 MVP。

## 11. 实施顺序

1. 新建 SwiftUI 工程，跑通 Hello World。
2. 写出 `HabitDraft` 与列表空态。
3. 抽出 `HabitRow`，完成添加、删除、打卡。
4. 加上 `NavigationStack`、详情与 Sheet 编辑。
5. 提升为 `HabitStore` 并注入环境。
6. 迁到 SwiftData，实现同一天幂等打卡。
7. 接入 `HabitClient`，缓存优先刷新。
8. 接入本地通知，处理授权与取消。
9. 补 XCTest 与关键路径 UI 测试。
10. 配置签名，Archive 并上传 TestFlight。
11. 可选：Widget、App Intent、深链、英文文案。

## 12. 验收清单

- [ ] 工程能在模拟器构建并运行。
- [ ] 列表展示标题与连续天数，空态正确。
- [ ] 新增习惯后列表立即刷新，重启后数据仍在。
- [ ] 同一天重复打卡天数不叠加。
- [ ] 详情打卡后返回列表已更新。
- [ ] 网络失败时回退本地缓存并给出重试提示。
- [ ] 异步工作不阻塞主线程。
- [ ] 通知授权后能按时提醒，关闭后能取消。
- [ ] 视图代码明显变薄，业务逻辑在 Store / Service。
- [ ] 单元测试覆盖模型计算与失败分支。
- [ ] 签名完整，可 Archive 并上传 TestFlight。
- [ ] 权限与隐私声明和实际使用一致。

## 13. 按阶段学习卡片

| 阶段 | 项目增量 | 关键能力 |
|---|---|---|
| 入门零 | 跑通工程、读懂模型 | Swift 语法、可选值、App 入口 |
| 入门一 | 列表、空态、添加、打卡 | SwiftUI 状态、List、单向数据流 |
| 入门二 | 详情、Sheet、环境对象 | NavigationStack、@Observable |
| 进阶三 | 网络、SwiftData、缓存、通知 | URLSession、Concurrency、持久化 |
| 进阶四 | 分层、测试、Archive | SPM、XCTest、隐私清单 |
| 进阶五 | Widget、Intent、深链 | WidgetKit、App Intents |
