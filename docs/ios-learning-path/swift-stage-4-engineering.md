# 阶段四：架构、测试、性能与发布

**目标**：把项目整理成可测试的分层结构，接入 SPM，补测试与性能排查，完成签名、隐私清单与 TestFlight。

**前置知识**：阶段三的网络、SwiftData 与并发。

## 1. 分层

```text
HabitTracker/
  App/
    HabitTrackerApp.swift
  Features/
    Habits/
      HabitListView.swift
      HabitDetailView.swift
      HabitEditorView.swift
    Settings/
      SettingsView.swift
  Models/
    Habit.swift
  Services/
    HabitClient.swift
    HabitRepository.swift
    KeychainStore.swift
    ReminderScheduler.swift
  Support/
    Formatters.swift
```

| 层 | 职责 | 不做什么 |
|---|---|---|
| View | 布局、绑定、导航 | 拼 URL、写查询细节 |
| Store / ViewModel | 组织状态、调用服务、错误文案 | 持有具体 `UIView` |
| Service | 网络、SwiftData、系统 API | 引用 SwiftUI 视图 |
| Model | 数据结构与领域规则 | 依赖界面 |

视图保持薄：

```swift
struct HabitListView: View {
  @Environment(HabitStore.self) private var store

  var body: some View {
    List(store.habits) { habit in
      HabitRow(habit: habit) {
        store.checkIn(habit.id)
      }
    }
    .task { await store.load() }
    .overlay {
      if store.isLoading { ProgressView() }
    }
  }
}
```

## 2. 依赖注入

用协议替换具体客户端，测试时注入假实现：

```swift
protocol HabitFetching: Sendable {
  func fetchHabits() async throws -> [HabitDTO]
}

struct HabitClient: HabitFetching { /* URLSession */ }

struct PreviewClient: HabitFetching {
  func fetchHabits() async throws -> [HabitDTO] {
    [HabitDTO(id: UUID(), title: "阅读", streak: 3)]
  }
}
```

`App` 只在入口组装真实依赖；Preview 与测试组装假依赖。

## 3. SPM

Xcode：File -> Add Package Dependencies。命令行包：

```swift
// Package.swift
dependencies: [
  .package(url: "https://github.com/apple/swift-http-types", from: "1.0.0")
]
```

规则：

- 锁定版本，提交 `Package.resolved`。
- 同一库不要同时用 CocoaPods 与 SPM。
- 二进制依赖确认隐私清单与签名。

## 4. 测试

```swift
import XCTest
@testable import HabitTracker

final class HabitStoreTests: XCTestCase {
  @MainActor
  func testCheckInIncreasesStreak() async {
    let store = HabitStore(client: PreviewClient())
    await store.load()
    let id = store.habits[0].id
    store.checkIn(id)
    XCTAssertEqual(store.habits[0].streak, 4)
  }

  func testEmptyTitleRejected() {
    XCTAssertThrowsError(try Habit.validate(title: "  "))
  }
}
```

UI 测试覆盖主路径：启动 -> 添加 -> 打卡 -> 列表出现新连续天数。

测试原则：

- 测领域规则与失败分支，不测 SwiftUI 布局细节。
- 网络用假客户端，SwiftData 用内存容器。
- 时间相关逻辑注入 `Clock` / 固定 `Date`，避免依赖系统时钟。

命令行：

```bash
xcodebuild test -scheme HabitTracker \
  -destination 'platform=iOS Simulator,name=iPhone 16'
```

## 5. 调试与性能

| 工具 | 用途 |
|---|---|
| LLDB / 断点 | 逻辑错误 |
| View Hierarchy | 约束与叠加 |
| Instruments Time Profiler | 主线程卡顿 |
| Instruments Allocations / Leaks | 内存增长 |
| SwiftUI Instrument | body 过度刷新 |
| Console / os.Logger | 结构化日志 |

```swift
import os

private let logger = Logger(subsystem: "com.example.habit", category: "network")
logger.error("fetch failed: \(error.localizedDescription, privacy: .public)")
```

常见卡顿：

- `body` 里做解码、排序大量数据。
- 图片同步解码。
- SwiftData 查询无谓全表。
- 列表没有稳定 `id` 导致整表重建。

## 6. 无障碍、本地化与深色模式

- 所有图标按钮有 `accessibilityLabel`。
- 字符串进 `Localizable.xcstrings`，不要在视图里写死中文以外的拼接逻辑。
- Color Set 提供 Any 与 Dark。
- 动态字体下检查截断，标题用 `.minimumScaleFactor` 要谨慎。

## 7. 签名、隐私与发布

发布清单：

1. Bundle Identifier、版本号、构建号。
2. 签名 Team、证书、描述文件。
3. `PrivacyInfo.xcprivacy` 完整。
4. 权限用途字符串与真实功能一致。
5. Archive -> Organizer -> Distribute -> App Store Connect。
6. TestFlight 内部测试通过后再提交审核。

```bash
xcodebuild -scheme HabitTracker \
  -destination 'generic/platform=iOS' \
  -configuration Release \
  archive -archivePath build/HabitTracker.xcarchive
```

审核常见拒因：权限文案空洞、隐私清单缺失、崩溃、登录演示账号缺失。

## 阶段验收

- 能按 Feature / Model / Service 拆分，视图不直接打网络。
- 能用协议注入假客户端并写出 XCTest。
- 能用 SPM 添加并锁定依赖。
- 能用 Instruments 指出一次卡顿或一次泄漏的根因。
- 能完成 Archive 并上传 TestFlight。

## 动手任务

1. 把网络与 SwiftData 从视图挪到 `HabitStore` / `HabitRepository`。
2. 为打卡与空标题写出单元测试。
3. 接入 `os.Logger`，失败请求打日志。
4. 用 Time Profiler 录一次滑动列表。
5. 填写隐私清单，Archive 出包。

## 常见排错

| 现象 | 排查方向 |
|---|---|
| 测试找不到内部类型 | 是否 `@testable import`，类型是否 `internal` 以上 |
| Preview 与真机数据不一致 | 是否共用了真实磁盘容器 |
| Archive 签名失败 | Team、Bundle ID、证书是否匹配 |
| 审核被拒隐私 | `PrivacyInfo.xcprivacy` 与用途字符串 |
| body 频繁刷新 | 可观察对象是否在 `body` 里创建新实例 |

## 进入下一阶段的条件

你能在不破坏现有功能的前提下完成分层与测试，并能独立 Archive。此时进入 [阶段五：系统能力专题](./swift-stage-5-advanced.md)，或直接做 [综合实战](./swift-project-practice.md) 的进阶项。
