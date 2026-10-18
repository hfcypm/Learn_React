# 综合实战：维护老版习惯打卡 App

## 1. 项目目标

接手一个既有的 Objective-C 工程 `LegacyHabit`：它用 UIKit + Storyboard 构建，依赖 CocoaPods，数据存 Core Data。目标是在不推倒重写的前提下，按阶段完成“跑通 -> 修界面 -> 接数据 -> 分层改造 -> 上架”的维护闭环。

```text
跑通工程 -> 修复列表与导航 -> 接入网络与持久化
    -> 拆分 Massive ViewController -> 补测试 -> 签名发布
```

选择“习惯打卡”的原因：数据结构简单（习惯 + 打卡日期），但天然覆盖列表、表单、导航、网络同步、本地数据库、并发与发布，能串起本路线全部知识点。

## 2. 接手现状

老工程常见问题（也是本项目的起步状态）：

- `HabitListViewController` 超过 800 行，网络、解析、数据库、布局、跳转混在一起。
- 网络用回调嵌套，错误分支没有统一处理。
- Core Data 写入后直接 `reloadData`，没有监听变化。
- 无单元测试，改一处容易坏一片。
- 部分文件设置了 `-fno-objc-arc`，与新代码风格不一致。

## 3. 需求

**必须实现（MVP）**

- 习惯列表：展示标题与连续打卡天数，空态提示。
- 新增习惯：输入名称，保存后列表刷新。
- 打卡：点击后连续天数加一，重启后不丢失。
- 页面跳转：列表 -> 详情 -> 编辑，能返回并刷新。
- 本地持久化：Core Data 存储习惯与打卡日期。

**进阶**

- 网络同步：从服务端拉取习惯，弱网时用本地缓存。
- 本地通知：每天固定时间提醒打卡（需要通知权限）。
- 数据迁移：为模型新增字段，验证老数据升级不丢失。
- 分层改造：拆出 ViewModel 与 Service，控制器只做绑定。
- 混编：新增一个 Swift 设置页，通过 Bridging Header 调用 OC 模型。
- 可观测性：接入日志与崩溃上报（可选）。
- 发布：签名并上传 TestFlight。

## 4. 技术选型

| 技术 | 用途 |
|---|---|
| Objective-C | 开发语言 |
| UIKit + Storyboard | 界面与导航 |
| Auto Layout | 布局与适配 |
| NSURLSession + NSJSONSerialization | 网络与解析 |
| Core Data | 本地持久化 |
| UserNotifications | 打卡提醒 |
| NSUserDefaults / Keychain | 偏好与敏感信息 |
| GCD / NSOperationQueue | 并发与线程切换 |
| CocoaPods | 三方依赖 |
| XCTest | 单元与 UI 测试 |
| Xcode Organizer | Archive 与 TestFlight |

## 5. 项目结构

改造前：

```text
LegacyHabit/
  AppDelegate.h/.m
  SceneDelegate.h/.m
  Main.storyboard
  HabitListViewController.h/.m
  HabitCell.h/.m
  Habit.h/.m
  Habit+CoreDataClass.h/.m
  Habit+CoreDataProperties.h/.m
  LegacyHabit.xcdatamodeld
  Podfile
```

改造后：

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
    Habit+CoreDataClass.h/.m
    Habit+CoreDataProperties.h/.m
  Services/
    HabitService.h/.m
    HabitRepository.h/.m
    KeychainStore.h/.m
  Views/
    HabitCell.h/.m
  Categories/
    UIColor+AppTheme.h/.m
  Resources/
    Main.storyboard
    Assets.xcassets
  LegacyHabit.xcdatamodeld
  Podfile
```

## 6. 阶段零：跑通与读懂

### 6.1 让工程跑起来

```bash
# 安装依赖
pod install

# 打开 workspace，而不是 xcodeproj
open LegacyHabit.xcworkspace
```

检查清单：

- [ ] Deployment Target 与本机 Xcode 匹配。
- [ ] Signing Team 与 Bundle Identifier 可构建。
- [ ] `AppDelegate` / `SceneDelegate` 的启动入口正确。
- [ ] `Main.storyboard` 是 Initial View Controller。
- [ ] `LegacyHabit.xcdatamodeld` 能正常加载。

### 6.2 核心模型

```objc
// Habit.h
#import <Foundation/Foundation.h>

NS_ASSUME_NONNULL_BEGIN

@interface Habit : NSObject

@property (nonatomic, copy) NSString *title;
@property (nonatomic, strong) NSMutableArray<NSDate *> *checkIns;

- (instancetype)initWithTitle:(NSString *)title NS_DESIGNATED_INITIALIZER;
- (instancetype)init NS_UNAVAILABLE;

- (NSInteger)streak;

@end

NS_ASSUME_NONNULL_END
```

```objc
// Habit.m
#import "Habit.h"

@implementation Habit

- (instancetype)initWithTitle:(NSString *)title {
  self = [super init];
  if (self) {
    _title = [title copy];
    _checkIns = [NSMutableArray array];
  }
  return self;
}

- (NSInteger)streak {
  // 简化实现：按日期去重后计算连续天数
  NSMutableSet<NSString *> *days = [NSMutableSet set];
  NSDateFormatter *formatter = [[NSDateFormatter alloc] init];
  formatter.dateFormat = @"yyyy-MM-dd";
  for (NSDate *date in self.checkIns) {
    [days addObject:[formatter stringFromDate:date]];
  }
  return (NSInteger)days.count;
}

@end
```

验收：工程可运行，能读懂启动链路与模型字段。

## 7. 阶段一：修复列表与导航

### 7.1 列表控制器

```objc
#import "HabitListViewController.h"
#import "HabitCell.h"
#import "Habit.h"

@interface HabitListViewController () <UITableViewDataSource, UITableViewDelegate>
@property (nonatomic, strong) UITableView *tableView;
@property (nonatomic, strong) NSMutableArray<Habit *> *habits;
@end

@implementation HabitListViewController

- (void)viewDidLoad {
  [super viewDidLoad];
  self.title = @"习惯";
  self.habits = [NSMutableArray array];
  [self setupTableView];
}

- (void)viewWillAppear:(BOOL)animated {
  [super viewWillAppear:animated];
  [self.tableView reloadData];
}

- (void)setupTableView {
  self.tableView = [[UITableView alloc] initWithFrame:CGRectZero style:UITableViewStyleInsetGrouped];
  self.tableView.dataSource = self;
  self.tableView.delegate = self;
  self.tableView.translatesAutoresizingMaskIntoConstraints = NO;
  [self.tableView registerClass:[HabitCell class] forCellReuseIdentifier:@"HabitCell"];
  [self.view addSubview:self.tableView];

  [NSLayoutConstraint activateConstraints:@[
    [self.tableView.topAnchor constraintEqualToAnchor:self.view.safeAreaLayoutGuide.topAnchor],
    [self.tableView.leadingAnchor constraintEqualToAnchor:self.view.leadingAnchor],
    [self.tableView.trailingAnchor constraintEqualToAnchor:self.view.trailingAnchor],
    [self.tableView.bottomAnchor constraintEqualToAnchor:self.view.bottomAnchor],
  ]];
}

- (NSInteger)tableView:(UITableView *)tableView numberOfRowsInSection:(NSInteger)section {
  return self.habits.count;
}

- (UITableViewCell *)tableView:(UITableView *)tableView
         cellForRowAtIndexPath:(NSIndexPath *)indexPath {
  HabitCell *cell = [tableView dequeueReusableCellWithIdentifier:@"HabitCell"
                                                   forIndexPath:indexPath];
  [cell configureWithHabit:self.habits[indexPath.row]];
  return cell;
}

@end
```

### 7.2 自定义 Cell

```objc
// HabitCell.h
#import <UIKit/UIKit.h>
@class Habit;

@interface HabitCell : UITableViewCell
- (void)configureWithHabit:(Habit *)habit;
@end
```

```objc
// HabitCell.m
#import "HabitCell.h"
#import "Habit.h"

@interface HabitCell ()
@property (nonatomic, strong) UILabel *titleLabel;
@property (nonatomic, strong) UILabel *streakLabel;
@end

@implementation HabitCell

- (instancetype)initWithStyle:(UITableViewCellStyle)style
              reuseIdentifier:(NSString *)reuseIdentifier {
  self = [super initWithStyle:style reuseIdentifier:reuseIdentifier];
  if (self) {
    _titleLabel = [[UILabel alloc] init];
    _streakLabel = [[UILabel alloc] init];
    _streakLabel.textColor = [UIColor secondaryLabelColor];
    UIStackView *stack = [[UIStackView alloc] initWithArrangedSubviews:@[_titleLabel, _streakLabel]];
    stack.axis = UILayoutConstraintAxisHorizontal;
    stack.spacing = 8;
    stack.translatesAutoresizingMaskIntoConstraints = NO;
    [self.contentView addSubview:stack];
    [NSLayoutConstraint activateConstraints:@[
      [stack.leadingAnchor constraintEqualToAnchor:self.contentView.leadingAnchor constant:16],
      [stack.trailingAnchor constraintEqualToAnchor:self.contentView.trailingAnchor constant:-16],
      [stack.centerYAnchor constraintEqualToAnchor:self.contentView.centerYAnchor],
    ]];
  }
  return self;
}

- (void)prepareForReuse {
  [super prepareForReuse];
  self.titleLabel.text = nil;
  self.streakLabel.text = nil;
}

- (void)configureWithHabit:(Habit *)habit {
  self.titleLabel.text = habit.title;
  self.streakLabel.text = [NSString stringWithFormat:@"%ld 天", (long)habit.streak];
}

@end
```

### 7.3 新增与反向传值

```objc
typedef void (^HabitSavedBlock)(Habit *habit);

// 编辑页保存
- (void)saveTapped {
  Habit *habit = [[Habit alloc] initWithTitle:self.textField.text];
  if (self.onSaved) {
    self.onSaved(habit);
  }
  [self dismissViewControllerAnimated:YES completion:nil];
}

// 列表页打开编辑
- (void)addTapped {
  HabitEditorViewController *editor = [[HabitEditorViewController alloc] init];
  __weak typeof(self) weakSelf = self;
  editor.onSaved = ^(Habit *habit) {
    __strong typeof(weakSelf) self = weakSelf;
    if (!self) { return; }
    [self.habits addObject:habit];
    [self.tableView reloadData];
  };
  UINavigationController *nav = [[UINavigationController alloc] initWithRootViewController:editor];
  [self presentViewController:nav animated:YES completion:nil];
}
```

验收：列表可展示、可新增、可跳转，cell 复用不串数据。

## 8. 阶段二：网络与持久化

### 8.1 Service

```objc
// HabitService.h
typedef void (^HabitListCompletion)(NSArray<Habit *> * _Nullable habits,
                                    NSError * _Nullable error);

@interface HabitService : NSObject
- (void)fetchHabitsWithCompletion:(HabitListCompletion)completion;
@end
```

```objc
// HabitService.m
- (void)fetchHabitsWithCompletion:(HabitListCompletion)completion {
  NSURL *url = [NSURL URLWithString:@"https://example.com/api/habits"];
  NSURLSessionDataTask *task =
  [[NSURLSession sharedSession] dataTaskWithURL:url
                               completionHandler:^(NSData *data,
                                                   NSURLResponse *response,
                                                   NSError *error) {
    if (error) {
      dispatch_async(dispatch_get_main_queue(), ^{ completion(nil, error); });
      return;
    }
    NSError *jsonError = nil;
    NSDictionary *json = [NSJSONSerialization JSONObjectWithData:data options:0 error:&jsonError];
    if (![json isKindOfClass:[NSDictionary class]]) {
      dispatch_async(dispatch_get_main_queue(), ^{ completion(nil, jsonError); });
      return;
    }
    NSMutableArray<Habit *> *habits = [NSMutableArray array];
    for (NSDictionary *dict in json[@"items"]) {
      Habit *habit = [[Habit alloc] initWithTitle:dict[@"title"] ?: @"未命名"];
      [habits addObject:habit];
    }
    dispatch_async(dispatch_get_main_queue(), ^{ completion(habits, nil); });
  }];
  [task resume];
}
```

### 8.2 Core Data 持久化

```objc
// 保存
NSManagedObjectContext *context = self.persistentContainer.viewContext;
Habit *managed = [NSEntityDescription insertNewObjectForEntityForName:@"Habit"
                                               inManagedObjectContext:context];
managed.title = habit.title;
managed.createdAt = [NSDate date];
NSError *error = nil;
if (![context save:&error]) {
  NSLog(@"保存失败：%@", error);
}

// 查询
NSFetchRequest<Habit *> *request = [Habit fetchRequest];
request.sortDescriptors = @[[NSSortDescriptor sortDescriptorWithKey:@"createdAt" ascending:NO]];
NSArray<Habit *> *habits = [context executeFetchRequest:request error:&error];
```

模型变更时开启轻量迁移，避免老用户升级后数据丢失：

```objc
NSPersistentStoreDescription *desc = container.persistentStoreDescriptions.firstObject;
desc.shouldMigrateStoreAutomatically = YES;
desc.shouldInferMappingModelAutomatically = YES;
```

### 8.3 缓存优先

```text
1. 启动先读 Core Data，立即渲染列表
2. 后台请求服务端
3. 成功 -> 合并写入 Core Data -> 主线程刷新
4. 失败 -> 保留本地数据 -> 顶部提示“同步失败，可重试”
```

验收：断网时仍能查看与打卡，联网后能同步。

### 8.4 本地通知提醒

```objc
#import <UserNotifications/UserNotifications.h>

- (void)enableDailyReminderAtHour:(NSInteger)hour {
  UNUserNotificationCenter *center = [UNUserNotificationCenter currentNotificationCenter];
  [center requestAuthorizationWithOptions:(UNAuthorizationOptionAlert | UNAuthorizationOptionSound)
                        completionHandler:^(BOOL granted, NSError *error) {
    if (!granted) { return; }

    UNMutableNotificationContent *content = [UNMutableNotificationContent new];
    content.title = @"该打卡了";
    content.body = @"完成今天的习惯";
    content.sound = [UNNotificationSound defaultSound];

    NSDateComponents *components = [NSDateComponents new];
    components.hour = hour;
    UNCalendarNotificationTrigger *trigger =
        [UNCalendarNotificationTrigger triggerWithDateMatchingComponents:components repeats:YES];

    UNNotificationRequest *request =
        [UNNotificationRequest requestWithIdentifier:@"habit.daily"
                                             content:content
                                             trigger:trigger];
    [center addNotificationRequest:request error:nil];
  }];
}
```

验收：授权后每天固定时间收到提醒；关闭开关后取消待发通知。

## 9. 阶段三：分层、测试与发布

### 9.1 拆分 ViewModel

```objc
// HabitListViewModel.h
@interface HabitListViewModel : NSObject
@property (nonatomic, copy, readonly) NSArray<Habit *> *habits;
@property (nonatomic, assign, readonly) BOOL isLoading;
@property (nonatomic, copy, readonly) NSString * _Nullable errorMessage;
- (instancetype)initWithService:(HabitService *)service;
- (void)loadWithCompletion:(void (^)(void))completion;
@end
```

控制器不再关心数据来自网络还是缓存，只绑定状态。

### 9.2 单元测试

```objc
- (void)testStreakCountsUniqueDays {
  Habit *habit = [[Habit alloc] initWithTitle:@"阅读"];
  NSDate *now = [NSDate date];
  [habit.checkIns addObject:now];
  [habit.checkIns addObject:now];
  XCTAssertEqual(habit.streak, 1);
}
```

### 9.3 签名与 TestFlight

```text
1. Signing & Capabilities 选择 Team
2. 设置唯一 Bundle Identifier
3. Product -> Archive
4. Organizer -> Distribute App -> App Store Connect
5. TestFlight 内部测试
```

发布前检查：

- [ ] 无调试日志与硬编码测试地址。
- [ ] 权限与隐私声明一致。
- [ ] 深色模式、离线、异常网络都验证过。
- [ ] 崩溃与警告清零。

## 10. 实施顺序

1. `pod install` 并用 `.xcworkspace` 打开，跑通工程。
2. 修复 Storyboard 入口与 IBOutlet 连接，列表能显示。
3. 抽出 `HabitCell`，修复复用串数据。
4. 用 block 完成新增习惯的反向传值。
5. 接入 `HabitService`，补错误与状态码处理。
6. 接入 Core Data，实现缓存优先刷新，并开启轻量迁移。
7. 拆出 `HabitListViewModel`，控制器只做绑定。
8. 接入本地通知，处理授权与取消。
9. 补 XCTest 单元测试与关键路径 UI 测试。
10. 配置签名，Archive 并上传 TestFlight。

## 11. 验收清单

- [ ] 工程能用 `.xcworkspace` 构建并运行。
- [ ] 列表展示标题与连续天数，空态正确。
- [ ] 新增习惯后列表立即刷新，重启后数据仍在。
- [ ] cell 复用不串数据。
- [ ] 网络失败时回退本地缓存并给出重试提示。
- [ ] 网络回调不阻塞主线程。
- [ ] Core Data 轻量迁移后老数据仍可读。
- [ ] 通知授权后能按时提醒，关闭后能取消。
- [ ] 控制器代码明显变薄，业务逻辑在 Service / ViewModel。
- [ ] 单元测试覆盖模型计算与失败分支。
- [ ] 签名完整，可 Archive 并上传 TestFlight。
- [ ] 权限与隐私声明和实际使用一致。

## 12. 按阶段学习卡片

| 阶段 | 项目增量 | 关键能力 |
|---|---|---|
| 零 | 跑通工程、读懂模型 | OC 语法、属性、ARC、生命周期 |
| 一 | 列表、Cell、新增、跳转 | UIKit、Auto Layout、UITableView、回调 |
| 二 | 网络、Core Data、缓存、通知 | NSURLSession、GCD、持久化、迁移 |
| 三 | 分层、测试、混编、发布 | 架构改造、XCTest、签名上架 |
