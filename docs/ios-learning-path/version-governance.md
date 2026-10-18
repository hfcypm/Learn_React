# iOS 版本边界与迁移

**目标**：让学习内容与存量工程拥有清晰的版本边界，降低 IDE 升级、系统升级与技术栈迁移的风险。

## 1. 版本配套关系

iOS 工程由“系统版本 + Xcode + Swift/OC 编译器 + 依赖管理”共同决定可用能力。

| 层级 | 说明 |
|---|---|
| Deployment Target | 工程支持的最低系统版本，决定可用 API 下限 |
| 设备系统版本 | 用户设备上的 iOS 版本 |
| Xcode | IDE 与 SDK，版本过高可能无法打开过老工程 |
| Swift 版本 | 混编时决定 Swift 语法与并发检查强度 |
| CocoaPods / SPM | 依赖获取与版本锁定方式 |

查询方式：

- 工程：target 的 General -> Minimum Deployments。
- Xcode：`xcodebuild -version`。
- 依赖：`Podfile.lock` / `Package.resolved`。

## 2. iOS 版本的代际差异

| 版本 | 关键变化 | 对老工程的影响 |
|---|---|---|
| iOS 12 及更早 | 无 SceneDelegate，窗口在 AppDelegate | 多窗口适配需补 SceneDelegate |
| iOS 13 | 引入 SceneDelegate、深色模式、SwiftUI | 启动流程与配色需要适配 |
| iOS 14 | 隐私权限细化、Widget、App Clips | 权限文案与跟踪授权 |
| iOS 15 | 通知摘要、App 内活动 | 通知行为变化 |
| iOS 16 | 锁屏、`NavigationStack`（SwiftUI） | 主要影响 SwiftUI |
| iOS 17 | Observation、SwiftData、隐私清单 | 新项目能力增强，老工程需补隐私清单 |

存量 OC 工程升级时，优先关注：启动流程（AppDelegate -> SceneDelegate）、深色模式、权限与隐私声明、第三方库的最低系统版本。

## 3. 内存管理代际：MRC 与 ARC

| 维度 | MRC | ARC |
|---|---|---|
| 引用计数 | 手动 `retain` / `release` / `autorelease` | 编译器自动插入 |
| `dealloc` | 需调用 `[super dealloc]` | 不调用 `[super dealloc]` |
| 循环引用 | 手动打破 | 仍需 `weak` / `unowned` 打破 |
| 编译标志 | `-fno-objc-arc` | 默认开启 |

老工程中混有 MRC 文件时：

1. 先确认 Build Phases 中的 `-fno-objc-arc` 标记。
2. 不要直接删除 `release`，先在分支验证。
3. 优先把独立小文件迁移到 ARC，再逐步扩大范围。
4. 迁移后重点检查：野指针、过度释放、`dealloc` 中的 `[super dealloc]`。

## 4. UI 技术栈代际：UIKit 与 SwiftUI

| 维度 | UIKit | SwiftUI |
|---|---|---|
| 范式 | 命令式，控制器 + 视图 | 声明式，视图组合 |
| 布局 | Auto Layout / frame | 声明式修饰符 |
| 导航 | UINavigationController | NavigationStack |
| 状态 | 属性 + 手动刷新 | `@State` / `@Observable` |
| 成熟度 | 生态成熟、老工程主流 | 新项目推荐，持续演进 |

存量工程的策略：

- 以 UIKit 为主，新页面可评估 SwiftUI。
- 混用通过 `UIHostingController` 承载 SwiftUI 视图。
- 不在同一页面同时维护两套状态来源。

## 5. 语言代际：Objective-C 与 Swift

| 维度 | Objective-C | Swift |
|---|---|---|
| 类型安全 | 弱，运行时消息 | 强，编译期检查 |
| 内存 | ARC（或 MRC） | ARC |
| 并发 | GCD / NSOperation | async/await / actor |
| 互操作 | 通过 Bridging Header 与 `-Swift.h` | 通过 `@objc` 暴露 |

迁移顺序建议：Model -> Service -> ViewModel -> ViewController。每迁移一层都补测试，避免一次性重写。

## 6. 依赖管理代际：CocoaPods 与 SPM

| 维度 | CocoaPods | SPM |
|---|---|---|
| 配置文件 | `Podfile` / `Podfile.lock` | `Package.swift` / `Package.resolved` |
| 打开方式 | `.xcworkspace` | `.xcodeproj` / `.xcworkspace` |
| 生态 | 存量库多 | 新库增长快 |
| 二进制依赖 | 支持较好 | 支持中 |

存量工程继续用 CocoaPods 更稳；新引入的库优先评估 SPM，避免同一工程两套依赖体系混用导致冲突。

## 7. 升级前检查

1. 备份或打 tag，确认可回滚。
2. 阅读 Xcode 与 iOS 版本说明，确认最低系统版本与弃用 API。
3. 检查依赖库对目标 Xcode / iOS 的兼容性。
4. 检查启动流程（AppDelegate / SceneDelegate）。
5. 检查权限、ATS、隐私清单。
6. 检查 MRC 文件与 ARC 混编情况。
7. 在分支完整跑通 dev、测试与 Archive。

## 8. 升级后检查

- 冷启动、页面跳转、返回正常。
- 列表复用与刷新正常。
- 网络与 Core Data 读写正常，无数据迁移丢失。
- 深色模式与多语言显示正常。
- 权限申请与隐私声明一致。
- 崩溃与内存警告清零。
- 构建产物可签名、可安装、可上架。

## 9. 迁移常见坑

- 只升级 Xcode 未调整 Deployment Target，导致 API 不存在。
- 用了 iOS 13+ 的 SceneDelegate 但工程仍按旧启动流程配置，出现黑屏。
- 新增隐私相关 API 未补 `PrivacyInfo.xcprivacy`，审核被拒。
- 把 MRC 文件的 `release` 直接删除，导致过度释放崩溃。
- OC/Swift 混编时 Bridging Header 路径错误或 `@objc` 缺失。
- CocoaPods 与 SPM 同时引入同一库，产生重复符号。
- 升级三方库后 API 行为变化，未做回归测试。
