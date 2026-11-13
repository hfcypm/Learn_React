# iOS 版本边界与升级（Objective-C + UIKit）

**目标**：让学习内容与工程拥有清晰的版本边界，降低 IDE 升级、系统升级带来的风险。

本路线兼容 **iOS 12 及以上**；使用 SceneDelegate 的工程以 **iOS 13** 为基线。

## 1. 版本配套关系

iOS 工程由“系统版本 + Xcode + 依赖管理”共同决定可用能力。

| 层级 | 说明 |
|---|---|
| Deployment Target | 工程支持的最低系统版本，决定可用 API 下限 |
| 设备系统版本 | 用户设备上的 iOS 版本 |
| Xcode | IDE 与 SDK，版本过高可能无法打开过老工程 |
| CocoaPods | 依赖获取与版本锁定方式 |

查询方式：

- 工程：target 的 General -> Minimum Deployments。
- Xcode：`xcodebuild -version`。
- 依赖：`Podfile.lock`。

## 2. iOS 版本的代际差异

| 版本 | 关键变化 | 对 OC 工程的影响 |
|---|---|---|
| iOS 12 及更早 | 无 SceneDelegate，窗口在 AppDelegate | 多窗口适配需补 SceneDelegate |
| iOS 13 | 引入 SceneDelegate、深色模式 | 启动流程与配色需要适配 |
| iOS 14 | 隐私权限细化、Widget、App Clips | 权限文案与跟踪授权 |
| iOS 15 | 通知摘要、App 内活动 | 通知行为变化 |
| iOS 16 | 锁屏定制 | 系统能力适配 |
| iOS 17 | 隐私清单 | 工程需补 `PrivacyInfo.xcprivacy` |

存量 OC 工程升级时，优先关注：启动流程（AppDelegate -> SceneDelegate）、深色模式、权限与隐私声明、第三方库的最低系统版本。

## 3. 内存管理代际：MRC 与 ARC

| 维度 | MRC | ARC |
|---|---|---|
| 引用计数 | 手动 `retain` / `release` / `autorelease` | 编译器自动插入 |
| `dealloc` | 需调用 `[super dealloc]` | 不调用 `[super dealloc]` |
| 循环引用 | 手动打破 | 仍需 `weak` 打破 |
| 编译标志 | `-fno-objc-arc` | 默认开启 |

老工程中混有 MRC 文件时：

1. 先确认 Build Phases 中的 `-fno-objc-arc` 标记。
2. 不要直接删除 `release`，先在分支验证。
3. 优先把独立小文件迁移到 ARC，再逐步扩大范围。
4. 迁移后重点检查：野指针、过度释放、`dealloc` 中的 `[super dealloc]`。

## 4. 依赖管理：CocoaPods 要点

| 维度 | 说明 |
|---|---|
| 配置文件 | `Podfile` / `Podfile.lock` |
| 打开方式 | `.xcworkspace` |
| 版本锁定 | 提交 `Podfile.lock`，团队安装一致版本 |
| 常见坑 | 用 `.xcodeproj` 打开导致找不到头文件；`pod install` 与 `pod update` 混用导致版本漂移 |

存量工程继续用 CocoaPods；引入新库前先确认工程已有依赖，避免同类库重复引入。

## 5. Core Data 模型版本与迁移

- `.xcdatamodeld` 每次结构变更新建版本，并设置 Current Version，不要直接改旧版本。
- 轻量迁移覆盖：新增/删除属性、重命名属性（设置 renaming identifier）、新增实体。
- 开启自动迁移：

```objc
NSPersistentStoreDescription *desc = container.persistentStoreDescriptions.firstObject;
desc.shouldMigrateStoreAutomatically = YES;
desc.shouldInferMappingModelAutomatically = YES;
```

- 复杂迁移（拆分实体、字段类型转换）需要 Mapping Model 或自定义 `NSEntityMigrationPolicy`。
- 迁移前备份 store，测试必须覆盖“旧版本数据 -> 新版本”的升级路径。
- 大表迁移放后台上下文执行，并给用户展示进度，避免启动卡死。

## 6. 升级前检查

1. 备份或打 tag，确认可回滚。
2. 阅读 Xcode 与 iOS 版本说明，确认最低系统版本与弃用 API。
3. 检查依赖库对目标 Xcode / iOS 的兼容性。
4. 检查启动流程（AppDelegate / SceneDelegate）。
5. 检查权限、ATS、隐私清单。
6. 检查 MRC 文件与 ARC 混编情况。
7. 检查 Core Data 模型版本与迁移方案。
8. 在分支完整跑通 dev、测试与 Archive。

## 7. 升级后检查

- 冷启动、页面跳转、返回正常。
- 列表复用与刷新正常。
- 网络与 Core Data 读写正常，无数据迁移丢失。
- 深色模式与多语言显示正常。
- 权限申请与隐私声明一致。
- 崩溃与内存警告清零。
- 构建产物可签名、可安装、可上架。

## 8. 升级常见坑

- 只升级 Xcode 未调整 Deployment Target，导致 API 不存在。
- 用较新 Xcode 打开很老的工程，项目格式不兼容，直接无法编译。
- 用了 iOS 13+ 的 SceneDelegate 但工程仍按旧启动流程配置，出现黑屏。
- 新增隐私相关 API 未补 `PrivacyInfo.xcprivacy`，审核被拒。
- 把 MRC 文件的 `release` 直接删除，导致过度释放崩溃。
- 直接修改 Core Data 旧模型版本而非新建版本，导致老用户升级失败。
- 升级三方库后 API 行为变化，未做回归测试。
- Widget 与主应用未配置 App Group，timeline 读不到数据。
