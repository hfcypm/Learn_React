# 阶段五：系统能力专题

**目标**：补齐从“会写应用”到“能交付系统级体验”的专题——Widget、App Intents、Live Activity、后台任务、本地化与深色模式、UIKit 互操作、深链与 Spotlight。

**前置知识**：阶段四的分层、签名与隐私清单。基线 iOS 17+。

## 1. Widget

小组件把核心信息放到主屏幕，由 Widget Extension 渲染。

| 类型 | 适用 |
|---|---|
| StaticConfiguration | 无用户配置，展示今日打卡数 |
| AppIntentConfiguration | 用户可选“显示哪条习惯” |
| Control Widget（iOS 18） | 控制中心开关 |

```swift
import WidgetKit
import SwiftUI

struct StreakEntry: TimelineEntry {
  let date: Date
  let title: String
  let streak: Int
}

struct StreakProvider: TimelineProvider {
  func placeholder(in context: Context) -> StreakEntry {
    StreakEntry(date: .now, title: "阅读", streak: 3)
  }

  func getSnapshot(in context: Context, completion: @escaping (StreakEntry) -> Void) {
    completion(placeholder(in: context))
  }

  func getTimeline(in context: Context, completion: @escaping (Timeline<StreakEntry>) -> Void) {
    let entry = StreakEntry(date: .now, title: "阅读", streak: 3)
    completion(Timeline(entries: [entry], policy: .after(.now.addingTimeInterval(3600))))
  }
}

struct StreakWidgetView: View {
  let entry: StreakEntry

  var body: some View {
    VStack(alignment: .leading) {
      Text(entry.title).font(.headline)
      Text("连续 \(entry.streak) 天").font(.title)
    }
    .containerBackground(.fill.tertiary, for: .widget)
  }
}
```

要点：

- Extension 有独立 target 与预算，`timeline` 要控制刷新频率。
- 与主应用共享数据用 App Group 容器，不要假设能直接打开主应用的 SwiftData 默认路径。
- 点击小组件用 `widgetURL` 或 App Intent 打开对应详情。

## 2. App Intents 与 Spotlight

```swift
import AppIntents

struct CheckInIntent: AppIntent {
  static var title: LocalizedStringResource = "打卡"

  @Parameter(title: "习惯")
  var title: String

  func perform() async throws -> some IntentResult {
    try await HabitIntentHandler.checkIn(title: title)
    return .result()
  }
}
```

用途：Siri、快捷指令、Spotlight、控制中心。参数要可发现，执行要幂等（同一天重复打卡不叠加）。

## 3. Live Activity 与灵动岛

适合进行中的短时任务（专注计时），不适合永久状态。

```swift
import ActivityKit

struct FocusAttributes: ActivityAttributes {
  public struct ContentState: Codable, Hashable {
    var remaining: Int
  }
  var habitTitle: String
}
```

真机才能看灵动岛。模拟器只能验证启动与结束 API。记得处理用户在设置里关闭 Live Activity 的情况。

## 4. 后台任务

| 机制 | 适用 |
|---|---|
| `BGAppRefreshTask` | 定时轻量刷新（拉习惯摘要） |
| `BGProcessingTask` | 数分钟级处理（导入备份） |
| 后台 URLSession | 大文件上传下载 |
| 静默推送 | 服务器触发的刷新 |

```swift
import BackgroundTasks

BGTaskScheduler.shared.register(forTaskWithIdentifier: "com.example.habit.refresh") { task in
  Task {
    await HabitRepository.shared.refreshIfNeeded()
    task.setTaskCompleted(success: true)
  }
}
```

在 Info 中声明 `BGTaskSchedulerPermittedIdentifiers`。系统决定真正执行时间，代码必须可跳过、可重试。

## 5. 本地化、深色模式与动态字体

```text
Resources/
  Localizable.xcstrings
  Assets.xcassets / AccentColor
```

```swift
Text("habit.empty.title")
Image(systemName: "checkmark.circle")
  .accessibilityLabel(Text("habit.checkin.done"))
```

- 复数、词序用 String Catalog，不要 `"共 " + count + " 天"`。
- 预览用 `.environment(\.locale, Locale(identifier: "en"))` 与 `.preferredColorScheme(.dark)`。
- 从右到左布局检查 `HStack` 是否误用绝对 leading 图像镜像。

## 6. 深链、Universal Link 与剪贴板

```swift
.onOpenURL { url in
  guard url.host == "habit", let id = UUID(uuidString: url.lastPathComponent) else { return }
  path.append(.detail(id))
}
```

Universal Link 需要 apple-app-site-association。自定义 scheme 仅用于调试。处理 URL 时校验 id，防止注入不存在的路径把导航栈弄乱。

## 7. UIKit 互操作

存量控件或地图、WebView 仍需 UIKit：

```swift
import UIKit
import SwiftUI

struct LegacyChartView: UIViewRepresentable {
  var values: [Int]

  func makeUIView(context: Context) -> UIView {
    let view = UIView()
    view.backgroundColor = .systemBackground
    return view
  }

  func updateUIView(_ uiView: UIView, context: Context) {}
}
```

反过来把 SwiftUI 嵌进 UIKit：

```swift
UIHostingController(rootView: HabitListView())
```

规则：同一页面只保留一套状态源。UIKit 回调进 SwiftUI 时切回 `@MainActor`。

## 8. 安全与隐私扩展

- App Transport Security：生产全站 https。
- 敏感字段进 Keychain，日志里用 `privacy: .private`。
- 跟踪 API 需要 ATT 弹窗与用途说明。
- 导出备份前二次确认，导入后校验 schema 版本。

## 阶段验收

- 能解释 Widget timeline 与 App Group 共享数据。
- 能写一个可被快捷指令调用的 App Intent。
- 能说明 BGTask 与推送刷新的适用边界。
- 能完成中英字符串目录与深色模式预览。
- 能用 `UIViewRepresentable` 或 `UIHostingController` 做单向互操作。

## 动手任务

1. 新增 Widget Extension，展示今日已打卡数量。
2. 实现“给指定习惯打卡”的 App Intent。
3. 字符串全部迁入 String Catalog，增加 English。
4. 处理 `habit://detail/<uuid>` 深链。
5. 可选：用 `UIViewRepresentable` 包一个旧的进度条视图。

## 常见排错

| 现象 | 排查方向 |
|---|---|
| 小组件不刷新 | timeline policy、App Group、主应用是否写入共享容器 |
| Intent 在快捷指令里看不到 | `AppShortcutsProvider` 是否声明，title 是否本地化 |
| 后台任务从不跑 | identifier 是否声明，是否在前台调度 `submit` |
| 深链没反应 | `onOpenURL` 是否挂在根视图，Associated Domains |
| Representable 不更新 | `updateUIView` 是否使用新的 `values` |

## 进入实战的条件

入门与进阶阶段验收通过后，按 [综合实战：习惯打卡 App](./swift-project-practice.md) 把能力串进同一个工程。Widget 与 Intent 作为实战进阶项，不阻塞 MVP。
