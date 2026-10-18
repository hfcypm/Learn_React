# 阶段零：Objective-C 语言与 Xcode 工程

**目标**：掌握 Objective-C 的类、属性、内存管理与生命周期，能创建并读懂一个既有 Xcode 工程。

**前置知识**：C 语言基础（变量、函数、指针、结构体）与面向对象概念。不熟悉时先读第 0 节，再进入正式内容。

## 0. 预备知识

### 0.1 先看一个能跑的最小程序

```objc
// main.m
#import <Foundation/Foundation.h>

int main(int argc, const char * argv[]) {
  @autoreleasepool {
    NSString *name = @"Ada";
    NSLog(@"Hello, %@", name);
  }
  return 0;
}
```

- `main` 是程序入口，返回 0 表示正常结束。
- `#import <Foundation/Foundation.h>` 引入 Foundation 框架的声明，`NSString`、`NSLog`、`NSArray` 都来自这里。
- `@"..."` 是 OC 字符串字面量，类型为 `NSString *`；普通 `"..."` 是 C 字符串。
- `@autoreleasepool` 管理临时对象的释放，命令行程序需要显式写，App 里由主线程 run loop 维护。

`NSLog` 常用占位符：`%@` 对象、`%d` / `%ld` 整数、`%f` 浮点、`%p` 指针。

### 0.2 头文件、实现文件与编译流程

| 文件 | 作用 |
|---|---|
| `.h` | 声明：类、属性、方法签名，供其他文件引用 |
| `.m` | 实现：方法体与私有属性 |
| `.mm` | Objective-C++，需要混用 C++ 时使用 |
| `.pch` | 预编译头，老工程常见，用于全局导入 |

```text
.m / .h --预处理--> 展开 #import 与宏
        --编译--> 生成 .o 目标文件
        --链接--> 合并 Foundation 等库，生成可执行文件或 App
```

- `#import` 相当于“只包含一次的 `#include`”，避免重复声明。
- 编译由 `clang` 完成，Xcode 负责调度编译器与链接器。
- 报错分编译期（语法、类型、找不到声明）与链接期（找不到符号、重复符号），排查方向不同。

App 工程的 `main.m` 通常只有 `UIApplicationMain(...)`，把控制权交给 UIKit，所以业务代码从 `AppDelegate` 开始。

### 0.3 从 C 到 Objective-C

OC 是 C 的超集：变量、循环、函数、指针、结构体都能直接用。新增的核心是“对象”和“消息”：

```objc
// C 的函数调用：编译期确定调用谁
print_habit(habit);

// OC 的消息发送：运行时查找方法
[habit printTitle];
```

- 对象通常分配在堆上，`alloc` 负责分配并清零，`init` 负责初始化。
- 每个对象内部有 `isa` 指针指向它的类，方法查找、`isKindOfClass:`、KVO 都依赖它。
- 消息在运行时解析，因此 OC 支持动态替换方法、`respondsToSelector:`、KVC 等能力。
- 栈上的 C 结构体与堆上的 OC 对象生命周期不同，这是理解 ARC 的前提。

### 0.4 类、对象、实例、方法与属性

| 术语 | 含义 | 习惯打卡里的例子 |
|---|---|---|
| 类 | 模板、图纸 | `Habit` |
| 对象 / 实例 | 按模板创建出的具体个体 | “阅读”这条习惯 |
| 属性 | 对象保存的数据 | `title`、`streak` |
| 方法 | 对象能执行的动作 | `markCheckedIn` |
| 协议 | 一份方法清单，约定“能做什么” | `HabitCellDelegate` |

一个类可以创建多个实例，实例之间数据独立。`[[Habit alloc] init]` 就是“分配内存 + 初始化”两步。

### 0.5 先记住这几个词

- **框架（Framework）**：系统提供的一组类与函数，Foundation 管基础类型，UIKit 管界面。
- **SDK**：开发某平台所需的头文件、库与工具集合，iOS SDK 随 Xcode 安装。
- **Selector**：方法名在运行时的表示，如 `@selector(markCheckedIn)`。
- **Deployment Target**：App 支持的最低系统版本，决定能用哪些 API。
- **Bundle Identifier**：App 的唯一标识，签名与上架都依赖它。
- **ARC**：自动引用计数，编译器在合适位置插入引用计数操作。

如果 C 基础薄弱，先补：[附录：C 语言基础](./c-foundations.md) 覆盖变量与类型、分支与循环、函数与参数、指针与取地址、结构体、内存与预处理。这些在读老代码时每天都会遇到。

## 1. 类与对象

Objective-C 用 `.h` 声明接口、`.m` 实现：

```objc
// Habit.h
#import <Foundation/Foundation.h>

NS_ASSUME_NONNULL_BEGIN

@interface Habit : NSObject

@property (nonatomic, copy) NSString *title;
@property (nonatomic, assign) NSInteger streak;

- (instancetype)initWithTitle:(NSString *)title NS_DESIGNATED_INITIALIZER;
- (instancetype)init NS_UNAVAILABLE;

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
    _streak = 0;
  }
  return self;
}

@end
```

要点：

- `@interface` / `@implementation` 成对声明与实现。
- `instancetype` 表示“当前类实例”，优于 `id`。
- `NS_ASSUME_NONNULL_BEGIN` 内的参数默认非空，可空需显式 `nullable`。
- 指定初始化器用 `NS_DESIGNATED_INITIALIZER`，不可用初始化器用 `NS_UNAVAILABLE`。

## 2. 消息发送

Objective-C 的方法调用是“发消息”：

```objc
Habit *habit = [[Habit alloc] initWithTitle:@"阅读"];
[habit setStreak:1];
NSInteger streak = [habit streak];

// 点语法等价于调用 getter/setter
habit.title = @"跑步";
NSString *title = habit.title;
```

向 `nil` 发消息是安全的：返回 `nil`、`0` 或零值，不会崩溃。这既是容错也是隐藏 bug 的来源，关键路径仍需判空。

## 3. 属性与修饰符

```objc
@property (nonatomic, strong) NSMutableArray<Habit *> *habits;
@property (nonatomic, copy)   NSString *title;
@property (nonatomic, copy)   void (^onSaved)(Habit *habit);
@property (nonatomic, weak)   id<HabitCellDelegate> delegate;
@property (nonatomic, assign) NSInteger count;
```

| 修饰符 | 适用 | 说明 |
|---|---|---|
| `nonatomic` / `atomic` | 所有 | 默认 `atomic`，UI 与业务属性优先 `nonatomic` |
| `strong` | 对象 | 持有引用，ARC 下管理生命周期 |
| `weak` | 代理、避免循环引用 | 对象释放后自动置 `nil` |
| `copy` | NSString / 集合 / block | 防止外部可变对象被修改 |
| `assign` | 基本类型 | 数值、枚举、`NSInteger` |
| `readonly` | 只读属性 | 只生成 getter |

代理用 `weak`，block 用 `copy`，字符串用 `copy`，这是 OC 的三条硬规则。

## 4. 常用 Foundation 类型

```objc
NSString *name = @"Ada";
NSMutableString *mutable = [name mutableCopy];

NSArray<NSString *> *items = @[@"a", @"b"];
NSMutableArray<NSString *> *list = [items mutableCopy];
[list addObject:@"c"];

NSDictionary<NSString *, NSNumber *> *scores = @{@"math": @90};
NSMutableDictionary *dict = [scores mutableCopy];

NSSet<NSString *> *tags = [NSSet setWithObjects:@"work", @"life", nil];

NSNumber *num = @(42);
NSDate *now = [NSDate date];
NSDateFormatter *formatter = [[NSDateFormatter alloc] init];
formatter.dateFormat = @"yyyy-MM-dd";
NSString *text = [formatter stringFromDate:now];
```

## 5. Block 与循环引用

```objc
typedef void (^HabitCompletion)(NSArray<Habit *> *habits, NSError * _Nullable error);

HabitCompletion completion = ^(NSArray<Habit *> *habits, NSError *error) {
  if (error) {
    NSLog(@"失败：%@", error.localizedDescription);
    return;
  }
  NSLog(@"共 %lu 条", (unsigned long)habits.count);
};
```

block 捕获 `self` 时容易形成循环引用：

```objc
__weak typeof(self) weakSelf = self;
self.onSaved = ^(Habit *habit) {
  __strong typeof(weakSelf) self = weakSelf;
  if (!self) { return; }
  [self.habits addObject:habit];
};
```

## 6. 协议、分类与扩展

```objc
// 协议
@protocol HabitCellDelegate <NSObject>
- (void)habitCellDidTapCheckIn:(HabitCell *)cell;
@end

// 分类：给已有类加方法，不能加实例变量
@interface UIColor (AppTheme)
+ (UIColor *)appBackgroundColor;
@end

// 类扩展：在 .m 中声明私有属性与方法
@interface HabitListViewController ()
@property (nonatomic, strong) NSMutableArray<Habit *> *habits;
@end
```

## 7. KVC、KVO、通知与单例

```objc
// KVC
[habit setValue:@"阅读" forKey:@"title"];
NSString *title = [habit valueForKey:@"title"];

// KVO
[habit addObserver:self forKeyPath:@"streak" options:NSKeyValueObservingOptionNew context:nil];

// 通知
[[NSNotificationCenter defaultCenter] addObserver:self
                                         selector:@selector(handleHabitChanged:)
                                             name:@"HabitChangedNotification"
                                           object:nil];

// 单例
+ (instancetype)sharedStore {
  static HabitStore *instance;
  static dispatch_once_t onceToken;
  dispatch_once(&onceToken, ^{
    instance = [[HabitStore alloc] init];
  });
  return instance;
}
```

记得在 `dealloc` 中移除 KVO 与通知观察者（iOS 9+ 通知可自动移除，但显式移除更清晰）。

## 8. 内存管理（ARC）

ARC 自动插入 `retain` / `release`，你只需管好引用关系：

| 场景 | 写法 |
|---|---|
| 持有对象 | `strong` |
| 代理、避免环 | `weak` |
| 基本类型 | `assign` |
| 字符串/集合/block | `copy` |
| 释放时清理 | `dealloc` 移除观察者、取消网络任务 |

识别老代码中的 MRC 文件：出现 `retain` / `release` / `autorelease` / `dealloc` 中调用 `[super dealloc]`，通常该文件在 Build Phases 中设置了 `-fno-objc-arc`。改造前先确认编译标志，不要直接删 `release`。

### 8.1 常见循环引用

| 场景 | 问题 | 处理 |
|---|---|---|
| block 捕获 `self` | 对象互相强引用 | `__weak` + `__strong` 兜底 |
| delegate 用 `strong` | 控制器与代理互相持有 | 改为 `weak` |
| `NSTimer` 持有 target | 控制器无法释放 | 用 block 版本并 `invalidate` |
| 通知/KVO 观察者 | 未移除或重复添加 | `dealloc` 中移除 |

```objc
__weak typeof(self) weakSelf = self;
self.timer = [NSTimer scheduledTimerWithTimeInterval:1.0
                                             repeats:YES
                                               block:^(NSTimer *timer) {
  __strong typeof(weakSelf) self = weakSelf;
  if (!self) {
    [timer invalidate];
    return;
  }
  [self tick];
}];

- (void)dealloc {
  [_timer invalidate];
}
```

### 8.2 autoreleasepool

循环中产生大量临时对象时，用 `@autoreleasepool` 控制峰值内存：

```objc
for (NSDictionary *dict in largeArray) {
  @autoreleasepool {
    Habit *habit = [[Habit alloc] initWithTitle:dict[@"title"]];
    // 处理
  }
}
```

## 9. Xcode 工程结构

```text
LegacyHabit/
  LegacyHabit.xcodeproj
  LegacyHabit.xcworkspace        # 使用 CocoaPods 时打开这个
  Podfile / Podfile.lock
  LegacyHabit/
    AppDelegate.h / .m
    SceneDelegate.h / .m
    Info.plist
    Main.storyboard
    Assets.xcassets
    Controllers/
    Views/
    Models/
    Services/
  LegacyHabitTests/
```

- 有 CocoaPods 时，用 `.xcworkspace` 打开，`.xcodeproj` 会缺少 Pods 依赖。
- `Info.plist` 配置权限、ATS、启动配置。
- `Main.storyboard` 是入口界面（在 target 的 Main Interface 指定）。
- Deployment Target 决定可用的系统 API 下限。

## 10. 生命周期

### 10.1 AppDelegate（应用级）

```objc
- (BOOL)application:(UIApplication *)application
    didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
  return YES;
}

- (void)applicationDidEnterBackground:(UIApplication *)application {}
- (void)applicationWillEnterForeground:(UIApplication *)application {}
```

### 10.2 SceneDelegate（iOS 13+ 场景级）

```objc
- (void)scene:(UIScene *)scene
    willConnectToSession:(UISceneSession *)session
    options:(UISceneConnectionOptions *)connectionOptions {
  UIWindowScene *windowScene = (UIWindowScene *)scene;
  self.window = [[UIWindow alloc] initWithWindowScene:windowScene];
  self.window.rootViewController = [[UIStoryboard storyboardWithName:@"Main" bundle:nil]
                                    instantiateInitialViewController];
  [self.window makeKeyAndVisible];
}
```

### 10.3 UIViewController（页面级）

```text
loadView -> viewDidLoad -> viewWillAppear -> viewWillLayoutSubviews
  -> viewDidLayoutSubviews -> viewDidAppear -> viewWillDisappear
  -> viewDidDisappear -> dealloc
```

- `viewDidLoad`：只做一次初始化，不要在这里做重耗时任务。
- `viewWillAppear:`：每次出现前刷新数据与状态。
- `viewDidLayoutSubviews`：约束生效后调整 frame。
- `dealloc`：移除观察者、取消请求、释放资源。

## 11. 最小示例

Storyboard 中拖一个 `UILabel`，按住 Control 拖到代码里生成 IBOutlet：

```objc
#import "ViewController.h"

@interface ViewController ()
@property (weak, nonatomic) IBOutlet UILabel *titleLabel;
@end

@implementation ViewController

- (void)viewDidLoad {
  [super viewDidLoad];
  self.titleLabel.text = @"Hello, Objective-C";
}

@end
```

## 12. 构建与运行

| 操作 | 方式 |
|---|---|
| 安装依赖 | `pod install` |
| 打开工程 | 双击 `.xcworkspace` |
| 运行 | 选择模拟器，`Cmd + R` |
| 清理构建 | `Cmd + Shift + K` |
| 真机运行 | 选择设备，配置 Signing 后运行 |
| 命令行构建 | `xcodebuild -workspace LegacyHabit.xcworkspace -scheme LegacyHabit -destination 'platform=iOS Simulator,name=iPhone 16' build` |

## 13. 验收

- 能写出带属性、指定初始化器的 OC 类。
- 能解释 `strong` / `weak` / `copy` / `assign` 的选择依据。
- 能用 block 写回调，并用 `__weak` 避免循环引用。
- 能区分协议、分类与类扩展的用途。
- 能读懂 `AppDelegate` / `SceneDelegate` / `UIViewController` 的生命周期回调。
- 能用 `.xcworkspace` 打开并运行一个 CocoaPods 工程。

## 14. 排错

| 现象 | 排查 |
|---|---|
| `Property 'x' not found` | 头文件是否 import，属性是否声明在接口中 |
| 编译报 ARC 相关错误 | 文件是否设置了 `-fno-objc-arc`，混用了 MRC 语法 |
| 运行时 unrecognized selector | 方法未实现或对象类型错误，检查调用与 `respondsToSelector:` |
| 界面空白 | Main Interface 是否指向正确 Storyboard，入口控制器是否设置 |
| 崩溃在 `viewDidLoad` | IBOutlet 是否为 nil（连接丢失或视图未加载） |
| 依赖找不到 | 是否用 `.xcworkspace` 打开，是否执行 `pod install` |
| 内存持续增长 | block 是否强引用 self，观察者是否未移除，定时器是否未 invalidate |
