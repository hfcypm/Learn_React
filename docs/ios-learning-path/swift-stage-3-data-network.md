# 阶段三：网络、持久化与并发

**目标**：用 URLSession 完成请求与 JSON 解码，用 SwiftData / AppStorage / Keychain 持久化，用 Swift Concurrency 保证 UI 在主线程更新。

**前置知识**：阶段二的导航与 `@Observable` 数据源。

## 1. 网络请求

```swift
struct HabitDTO: Codable, Sendable {
  let id: UUID
  let title: String
  let streak: Int
}

enum APIError: Error {
  case status(Int)
  case decoding
  case transport(Error)
}

struct HabitClient {
  var session: URLSession = .shared
  var baseURL = URL(string: "https://example.com/api")!

  func fetchHabits() async throws -> [HabitDTO] {
    var request = URLRequest(url: baseURL.appending(path: "habits"))
    request.timeoutInterval = 10
    request.setValue("application/json", forHTTPHeaderField: "Accept")

    let (data, response) = try await session.data(for: request)
    guard let http = response as? HTTPURLResponse else {
      throw APIError.status(-1)
    }
    guard (200..<300).contains(http.statusCode) else {
      throw APIError.status(http.statusCode)
    }
    do {
      return try JSONDecoder().decode([HabitDTO].self, from: data)
    } catch {
      throw APIError.decoding
    }
  }
}
```

要点：

- `async/await` 替代 completion handler。
- 必须检查 `HTTPURLResponse.statusCode`。
- `Codable` 解码失败要映射成业务错误，避免把底层 `DecodingError` 直接丢给用户。
- 生产使用 https；http 需在 ATS 配置例外。

视图中调用：

```swift
.task {
  do {
    items = try await client.fetchHabits()
  } catch {
    message = error.localizedDescription
  }
}
```

`.task` 在视图消失时取消，对应 URLSession 任务会被取消。

## 2. 缓存优先链路

```swift
func loadHabits() async {
  isLoading = true
  defer { isLoading = false }
  do {
    let remote = try await client.fetchHabits()
    try await repository.replaceAll(remote)
  } catch {
    errorMessage = "网络不可用，已显示本地数据"
  }
}
```

顺序：先展示 SwiftData 本地结果，再后台拉网，成功后覆盖。弱网时列表仍可用。

## 3. SwiftData

```swift
import SwiftData

@Model
final class Habit {
  var title: String
  var streak: Int
  var checkIns: [Date]
  var createdAt: Date

  init(title: String, streak: Int = 0) {
    self.title = title
    self.streak = streak
    self.checkIns = []
    self.createdAt = .now
  }
}
```

注入容器：

```swift
@main
struct HabitTrackerApp: App {
  var body: some Scene {
    WindowGroup {
      HabitListView()
    }
    .modelContainer(for: Habit.self)
  }
}
```

查询与写入：

```swift
struct HabitListView: View {
  @Environment(\.modelContext) private var context
  @Query(sort: \Habit.createdAt, order: .reverse) private var habits: [Habit]

  var body: some View {
    List(habits) { habit in
      Text(habit.title)
    }
    .toolbar {
      Button("添加") {
        context.insert(Habit(title: "阅读"))
      }
    }
  }
}
```

| 能力 | 做法 |
|---|---|
| 查询 | `@Query` 带 sort / filter |
| 插入 | `context.insert` |
| 删除 | `context.delete` |
| 保存 | 默认自动保存；批量导入可 `try context.save()` |
| Preview | `.modelContainer(for: Habit.self, inMemory: true)` |

模型字段变更时新建 Schema 版本，不要直接改已经上线的属性类型。轻量变更（加可选字段）由 SwiftData 推断；复杂迁移写 `VersionedSchema` + `SchemaMigrationPlan`。

## 4. AppStorage、文件与 Keychain

| 存储 | 适用 | 不适用 |
|---|---|---|
| `AppStorage` / UserDefaults | 开关、上次打开时间、轻量偏好 | 列表、密钥 |
| 文件 / JSON | 导出备份、日志 | 需要查询的关系数据 |
| SwiftData | 习惯、打卡记录 | 令牌 |
| Keychain | token、密码 | 普通配置 |

```swift
@AppStorage("dailyReminderEnabled") var reminderEnabled = false

struct KeychainStore {
  func saveToken(_ token: String) throws {
    let data = Data(token.utf8)
    let query: [String: Any] = [
      kSecClass as String: kSecClassGenericPassword,
      kSecAttrAccount as String: "habit.token",
      kSecValueData as String: data
    ]
    SecItemDelete(query as CFDictionary)
    let status = SecItemAdd(query as CFDictionary, nil)
    guard status == errSecSuccess else {
      throw APIError.status(Int(status))
    }
  }
}
```

## 5. 并发

```swift
@MainActor
@Observable
final class HabitStore {
  var habits: [Habit] = []
  var isLoading = false
  private let client = HabitClient()

  func load() async {
    isLoading = true
    defer { isLoading = false }
    do {
      let dtos = try await client.fetchHabits()
      habits = dtos.map { Habit(title: $0.title, streak: $0.streak) }
    } catch {
      // 保留本地数据，记录错误
    }
  }
}
```

规则：

- UI 状态标 `@MainActor`。
- CPU 重活（统计、导入）用 `Task.detached` 或非主 actor，结果再回到主 actor。
- 共享可变缓存用 `actor`，不要在多个 Task 里无隔离地改同一个 `class`。
- 用 `TaskGroup` 做有限并发，避免一次性打爆接口。

```swift
actor StatsCalculator {
  func streakTotals(_ habits: [Habit]) -> Int {
    habits.reduce(0) { $0 + $1.streak }
  }
}
```

对照老代码：GCD 的 `DispatchQueue.main.async` 对应 `@MainActor`；`DispatchQueue.global` 对应脱离主 actor 的 `await` 计算。

## 6. 本地通知

```swift
import UserNotifications

func requestReminder() async {
  let center = UNUserNotificationCenter.current()
  let granted = try? await center.requestAuthorization(options: [.alert, .sound])
  guard granted == true else { return }

  let content = UNMutableNotificationContent()
  content.title = "该打卡了"
  content.body = "保持今天的连续记录"

  var date = DateComponents()
  date.hour = 8
  date.minute = 0
  let trigger = UNCalendarNotificationTrigger(dateMatching: date, repeats: true)
  let request = UNNotificationRequest(identifier: "daily-habit", content: content, trigger: trigger)
  try? await center.add(request)
}
```

在 `Info.plist` 说明通知用途。用户拒绝后走设置页引导，不要反复弹系统框。

## 7. 权限与 ATS

每引入一种能力就补声明：

- 网络：默认 https。
- 通知：授权文案与关闭路径。
- 相册 / 相机 / 追踪：用途字符串必须真实。
- `PrivacyInfo.xcprivacy`：列出使用的需要声明的 API（UserDefaults、文件时间戳等）。

## 阶段验收

- 能用 URLSession + Codable 拉取并解码列表，处理状态码与解码失败。
- 能实现本地优先、网络刷新的数据链路。
- 能用 SwiftData 完成增删改查，Preview 使用内存容器。
- 能区分 AppStorage、SwiftData、Keychain 的适用场景。
- 能用 `async/await` 与 `@MainActor` 更新 UI，不阻塞主线程。
- 能申请通知权限并安排、取消本地通知。

## 动手任务

1. 给 `Habit` 加上 `@Model`，列表改用 `@Query`。
2. 写 `HabitClient`，失败时仍显示本地数据。
3. 设置页用 `@AppStorage` 保存提醒开关。
4. 打开提醒后申请通知权限并注册 8:00 重复通知。
5. 用 `#Preview` 注入 `inMemory` 容器，预置两条习惯。

## 常见排错

| 现象 | 排查方向 |
|---|---|
| 请求成功 UI 不变 | 是否在 `@MainActor` 上赋值 |
| JSON 解码失败 | 字段名、可选、日期策略是否与接口一致 |
| `@Query` 为空 | `modelContainer` 是否注入到当前场景 |
| Preview 崩溃在 SwiftData | 是否使用 `inMemory: true` |
| 通知收不到 | 授权状态、identifier 是否被覆盖、触发时间是否在过去 |
| 主线程卡顿 | 是否在视图 `body` 里做网络或重计算 |

## 进入下一阶段的条件

你能把网络、SwiftData、通知接到同一个列表流程上，并能说清哪一层允许失败降级。此时进入 [阶段四：架构、测试、性能与发布](./swift-stage-4-engineering.md)。
