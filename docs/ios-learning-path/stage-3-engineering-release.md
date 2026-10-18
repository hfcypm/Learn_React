# 阶段三：老工程维护、测试与发布

**目标**：把老工程整理成可维护的分层结构，接入 CocoaPods 与 OC/Swift 混编，补测试、定位崩溃，并完成签名发布。

**前置知识**：阶段二的网络、持久化与并发。

## 1. 老工程的典型问题

| 问题 | 表现 | 改造方向 |
|---|---|---|
| Massive ViewController | 单文件上千行，网络、解析、布局、跳转混在一起 | 拆 Service / Model / View |
| 全局单例滥用 | 到处 `[Manager sharedManager]` | 依赖注入、限定职责 |
| 魔法数字与硬编码 | 字符串、颜色、尺寸散落 | 常量、Assets、配置 |
| 循环引用 | block / delegate / timer 导致内存不释放 | `weak`、及时移除 |
| 无测试 | 改一处坏一片 | 先补关键逻辑测试 |
| 依赖混乱 | Pod 版本漂移、重复库 | 锁定版本、统一依赖 |

## 2. 分层改造

目标结构：

```text
LegacyHabit/
  App/
    AppDelegate.h/.m
    SceneDelegate.h/.m
  Features/
    Habits/
      HabitListViewController.h/.m
      HabitEditorViewController.h/.m
      HabitListViewModel.h/.m
    Settings/
      SettingsViewController.h/.m
  Models/
    Habit.h/.m
  Services/
    HabitService.h/.m
    HabitRepository.h/.m
    KeychainStore.h/.m
  Views/
    HabitCell.h/.m
  Categories/
    UIColor+AppTheme.h/.m
```

职责边界：

| 层 | 职责 | 不做什么 |
|---|---|---|
| ViewController | 生命周期、布局、转发事件 | 直接拼 URL、写 SQL |
| ViewModel | 组织状态、调用服务、错误转换 | 持有 UIKit 视图 |
| Service | 网络、持久化、系统能力封装 | 引用 UIKit |
| Model | 数据结构与领域规则 | 依赖 UI |

### 2.1 Service 示例

```objc
// HabitService.h
typedef void (^HabitListCompletion)(NSArray<Habit *> * _Nullable habits,
                                    NSError * _Nullable error);

@interface HabitService : NSObject
- (void)fetchHabitsWithCompletion:(HabitListCompletion)completion;
@end
```

### 2.2 ViewModel 示例

```objc
// HabitListViewModel.h
@interface HabitListViewModel : NSObject
@property (nonatomic, copy, readonly) NSArray<Habit *> *habits;
@property (nonatomic, assign, readonly) BOOL isLoading;
@property (nonatomic, copy, readonly) NSString * _Nullable errorMessage;
- (void)loadWithCompletion:(void (^)(void))completion;
@end
```

控制器只做绑定：

```objc
- (void)viewDidLoad {
  [super viewDidLoad];
  self.viewModel = [[HabitListViewModel alloc] initWithService:[HabitService new]];
}

- (void)reloadData {
  __weak typeof(self) weakSelf = self;
  [self.viewModel loadWithCompletion:^{
    __strong typeof(weakSelf) self = weakSelf;
    if (!self) { return; }
    [self.tableView reloadData];
    if (self.viewModel.errorMessage) {
      [self showError:self.viewModel.errorMessage];
    }
  }];
}
```

## 3. CocoaPods

### 3.1 Podfile

```ruby
platform :ios, '12.0'
use_frameworks!

target 'LegacyHabit' do
  pod 'AFNetworking', '~> 4.0'
  pod 'SDWebImage', '~> 5.0'
  pod 'Masonry', '~> 1.1'
end
```

```bash
# 安装依赖
pod install

# 只在需要时更新
pod update AFNetworking
```

要点：

- 安装后用 `.xcworkspace` 打开。
- `Podfile.lock` 必须提交，保证团队依赖一致。
- 升级 Pod 前先看 CHANGELOG 与最低系统版本。
- 老工程可能用 `use_frameworks!` 或静态库，注意模块化与命名冲突。

### 3.2 常见三方库

| 库 | 用途 |
|---|---|
| AFNetworking | 网络请求 |
| SDWebImage | 图片加载与缓存 |
| Masonry | 代码写 Auto Layout |
| YYModel | JSON 转模型 |
| MJRefresh | 下拉刷新 |
| FMDB | SQLite 封装 |
| Realm | 移动端数据库 |
| CocoaLumberjack | 日志 |

引入前先确认工程现有技术栈，避免重复造轮子或引入第二套同类库。

## 4. Objective-C 与 Swift 混编

### 4.1 OC 调 Swift

使用自动生成的 `ProductModuleName-Swift.h`：

```objc
#import "LegacyHabit-Swift.h"

SwiftFeatureViewController *vc = [[SwiftFeatureViewController alloc] init];
[self.navigationController pushViewController:vc animated:YES];
```

Swift 侧需要 `@objc` 或继承 `NSObject` 才能暴露给 OC：

```swift
@objcMembers
final class SwiftFeatureViewController: UIViewController { }
```

### 4.2 Swift 调 OC

在 Bridging Header 中导入 OC 头文件：

```objc
// LegacyHabit-Bridging-Header.h
#import "Habit.h"
#import "HabitService.h"
```

在 Build Settings 的 `Objective-C Bridging Header` 指向该文件。

要点：

- 混编会增加编译时间与心智负担，改造时按模块逐步迁移。
- 迁移顺序：先 Model，再 Service，最后 ViewController。
- 避免同一功能同时存在 OC 与 Swift 两套实现。

## 5. 测试

### 5.1 单元测试（XCTest）

```objc
#import <XCTest/XCTest.h>
#import "Habit.h"

@interface HabitTests : XCTestCase
@end

@implementation HabitTests

- (void)testInitWithTitleSetsDefaultStreak {
  Habit *habit = [[Habit alloc] initWithTitle:@"阅读"];
  XCTAssertEqualObjects(habit.title, @"阅读");
  XCTAssertEqual(habit.streak, 0);
}

- (void)testTitleIsCopied {
  NSMutableString *title = [@"阅读" mutableCopy];
  Habit *habit = [[Habit alloc] initWithTitle:title];
  [title appendString:@"（改）"];
  XCTAssertEqualObjects(habit.title, @"阅读");
}

@end
```

测试重点：

- Model 的初始化、校验、计算、格式化。
- Service 的错误分支（超时、非 2xx、解析失败）。
- ViewModel 的状态流转（加载、成功、失败）。
- 网络层用协议或 block 注入 Mock，避免真实请求。

### 5.2 UI 测试

```objc
- (void)testAddHabit {
  XCUIApplication *app = [[XCUIApplication alloc] init];
  [app launch];
  [app.navigationBars[@"习惯"].buttons[@"新增"] tap];
  [app.textFields[@"习惯名称"] tap];
  [app.textFields[@"习惯名称"] typeText:@"阅读"];
  [app.buttons[@"保存"] tap];
  XCTAssertTrue([app.staticTexts[@"阅读"] exists]);
}
```

关键路径用 UI 测试兜底，避免回归。

## 6. 调试与崩溃排查

### 6.1 LLDB

```text
po self.viewModel.habits
p (NSInteger)self.habits.count
bt                 # 查看调用栈
breakpoint set -n "-[HabitListViewController viewDidLoad]"
```

### 6.2 Instruments

| 工具 | 用途 |
|---|---|
| Time Profiler | CPU 热点、主线程卡顿 |
| Allocations | 内存分配与增长 |
| Leaks | 内存泄漏 |
| Zombies | 释放后访问（EXC_BAD_ACCESS） |
| Network | 请求耗时与流量 |

### 6.3 崩溃日志

- 从 Xcode Organizer 或设备“分析与改进”导出崩溃日志。
- 必须有对应版本的 `.dSYM` 才能符号化。
- 先看异常类型（`EXC_BAD_ACCESS`、`EXC_CRASH`、`SIGABRT`），再看调用栈定位到具体方法。
- 保留 `dSYM` 并纳入归档，避免线上无法符号化。

### 6.4 常见崩溃

| 崩溃 | 原因 |
|---|---|
| `unrecognized selector sent to instance` | 方法未实现、对象类型错误、代理未赋值 |
| `EXC_BAD_ACCESS` | 野指针、weak 对象已释放、Zombies |
| 数组越界 | 数据源与 UI 不一致、并发修改数组 |
| 约束冲突 | Auto Layout 约束不完整 |
| `NSInternalInconsistencyException` | Storyboard 连接失效、cell 复用标识符未注册 |

### 6.5 View Debugger 与静态分析

- **View Debugger**：运行后点 Xcode 的 “Debug View Hierarchy”，检查视图层级、遮挡、约束与 `hidden` 状态，定位“看不到/点不到”的问题。
- **静态分析**：`Product -> Analyze`（或 `xcodebuild analyze`）能发现内存泄漏、未使用变量、空指针路径等问题，提交前跑一遍。
- **警告清零**：把编译器警告当错误处理，尤其是 ARC 相关、弃用 API 与格式字符串警告。

## 7. 性能

- 启动：减少 `didFinishLaunchingWithOptions` 中的同步 IO 与大批量初始化，懒加载非首屏模块。
- 列表：cell 复用、`prepareForReuse`、避免在 `cellForRow` 做重计算与同步网络。
- 图片：按显示尺寸降采样，使用异步解码与缓存。
- 主线程：耗时任务移出主线程，避免 `dispatch_sync` 到主队列造成死锁。
- 内存：及时释放大对象，检查循环引用与未取消的定时器/请求。

## 8. 签名与发布

### 8.1 配置

1. 加入 Apple Developer Program。
2. Xcode target 的 `Signing & Capabilities` 选择 Team，勾选自动签名。
3. 设置唯一 Bundle Identifier。
4. 配置 App 图标、启动图、版本号（Version）与构建号（Build）。

### 8.2 隐私清单

使用特定 API（如 UserDefaults、文件时间戳）需要 `PrivacyInfo.xcprivacy`：

```xml
<key>NSPrivacyAccessedAPITypes</key>
<array>
  <dict>
    <key>NSPrivacyAccessedAPIType</key>
    <string>NSPrivacyAccessedAPICategoryUserDefaults</string>
    <key>NSPrivacyAccessedAPITypeReasons</key>
    <array><string>CA92.1</string></array>
  </dict>
</array>
```

权限用途（相机、通知、定位）在 `Info.plist` 写清 `UsageDescription`，缺失会被审核拒绝。

### 8.3 TestFlight 与 App Store

```text
1. Xcode -> Product -> Archive
2. Organizer -> Distribute App -> App Store Connect
3. App Store Connect 等待构建处理完成
4. TestFlight 分发给内部测试，验证真机表现
5. 补充截图、描述、隐私问卷，提交审核
```

发布前检查：

- 版本号递增，Build 号唯一。
- 无调试代码、无硬编码测试密钥。
- 权限与隐私声明和实际使用一致。
- 真机关键路径跑通（登录、网络异常、深色模式、离线）。
- 崩溃与警告清零。

### 8.4 命令行构建与持续集成

```bash
# 构建并归档
xcodebuild -workspace LegacyHabit.xcworkspace \
  -scheme LegacyHabit \
  -configuration Release \
  -destination 'generic/platform=iOS' \
  -archivePath build/LegacyHabit.xcarchive \
  archive

# 导出 ipa（需要 ExportOptions.plist）
xcodebuild -exportArchive \
  -archivePath build/LegacyHabit.xcarchive \
  -exportPath build/ipa \
  -exportOptionsPlist ExportOptions.plist
```

- CI 中用 `xcodebuild` 或 `fastlane`（`gym` + `pilot`）自动化构建与上传。
- 签名凭据放 CI 的密钥管理，不提交到仓库。
- 每次合并前至少跑单元测试与静态分析。

## 9. 验收

- 能把 Massive ViewController 拆成 Controller / ViewModel / Service / Model。
- 能用 CocoaPods 管理依赖并用 `.xcworkspace` 构建。
- 能完成 OC 与 Swift 的双向调用配置。
- 能写出 XCTest 单元测试与关键路径 UI 测试。
- 能用 LLDB、Instruments 与 View Debugger 定位崩溃、泄漏与卡顿。
- 能用静态分析与命令行构建接入持续集成。
- 能配置签名，Archive 后上传 TestFlight。

## 10. 排错

| 现象 | 排查 |
|---|---|
| `pod install` 失败 | Ruby 环境、CocoaPods 版本、Podfile 平台版本 |
| 混编找不到头文件 | Bridging Header 路径、`-Swift.h` 是否生成 |
| Archive 失败 | 签名 Team、证书、Deployment Target |
| 上传被拒（隐私） | 是否缺少 `PrivacyInfo.xcprivacy` 或用途说明 |
| 崩溃无法符号化 | 是否保留对应 `.dSYM` |
| 测试找不到类 | target 是否开启可测试性、`@testable` 是否适用 |
| 真机安装失败 | Bundle Identifier 冲突、设备 UDID、证书过期 |
