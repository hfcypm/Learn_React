# 阶段一：UIKit 与 Storyboard/XIB

**目标**：用 UIKit 的控制器、Auto Layout、UITableView 与导航控制器改造老工程界面，掌握页面传值与回调。

**前置知识**：阶段零的 Objective-C 语法、属性与生命周期。

## 1. UIViewController

控制器是页面的核心，负责生命周期、布局与交互：

```objc
#import "HabitListViewController.h"

@interface HabitListViewController ()
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

- (void)dealloc {
  [[NSNotificationCenter defaultCenter] removeObserver:self];
}

@end
```

## 2. Storyboard 与 XIB

| 方式 | 适用 | 连接方式 |
|---|---|---|
| Storyboard | 多页面、有 segue 跳转 | IBOutlet / IBAction / Segue |
| XIB | 单个视图或 Cell 复用 | IBOutlet / IBAction |
| 纯代码 | 动态布局、易冲突的复杂界面 | 手写约束 |

IBOutlet / IBAction：

```objc
@property (weak, nonatomic) IBOutlet UILabel *titleLabel;

- (IBAction)checkInTapped:(id)sender {
  self.habit.streak += 1;
  [self refreshUI];
}
```

注意：

- IBOutlet 用 `weak`（视图层级已强持有）。
- 删除控件前先断开连接，否则运行时可能崩溃或收到野指针消息。
- Storyboard 的 Initial View Controller 决定启动页。

## 3. Auto Layout

优先用约束而不是写死 frame：

```objc
self.tableView.translatesAutoresizingMaskIntoConstraints = NO;
[NSLayoutConstraint activateConstraints:@[
  [self.tableView.topAnchor constraintEqualToAnchor:self.view.safeAreaLayoutGuide.topAnchor],
  [self.tableView.leadingAnchor constraintEqualToAnchor:self.view.leadingAnchor],
  [self.tableView.trailingAnchor constraintEqualToAnchor:self.view.trailingAnchor],
  [self.tableView.bottomAnchor constraintEqualToAnchor:self.view.bottomAnchor],
]];
```

要点：

- `translatesAutoresizingMaskIntoConstraints = NO` 后才使用约束。
- 贴边用 `safeAreaLayoutGuide`，避开刘海与 Home 指示条。
- 复杂表单用 `UIStackView` 组织，减少约束数量。
- Storyboard 中检查约束是否冲突（黄色/红色警告）。

## 4. 导航与模态

```objc
// push（需要外层是 UINavigationController）
HabitDetailViewController *detail =
    [[HabitDetailViewController alloc] initWithNibName:@"HabitDetailViewController" bundle:nil];
detail.habit = habit;
[self.navigationController pushViewController:detail animated:YES];

// modal
HabitEditorViewController *editor = [[HabitEditorViewController alloc] init];
editor.onSaved = ^(Habit *habit) { /* 回调 */ };
UINavigationController *nav =
    [[UINavigationController alloc] initWithRootViewController:editor];
[self presentViewController:nav animated:YES completion:nil];

// 关闭模态
[self dismissViewControllerAnimated:YES completion:nil];
```

- `UINavigationController` 管理系统页面栈，`push` / `pop` 控制前进与返回。
- `UITabBarController` 管理底部标签页。
- 模态用 `present` / `dismiss`，适合编辑、登录等中断式流程。

### 4.1 Storyboard Segue 传值

```objc
- (void)prepareForSegue:(UIStoryboardSegue *)segue sender:(id)sender {
  if ([segue.identifier isEqualToString:@"ShowDetail"]) {
    HabitDetailViewController *detail = segue.destinationViewController;
    detail.habit = sender;
  }
}
```

## 5. UITableView

```objc
@interface HabitListViewController () <UITableViewDataSource, UITableViewDelegate>
@property (nonatomic, strong) UITableView *tableView;
@property (nonatomic, strong) NSMutableArray<Habit *> *habits;
@end

@implementation HabitListViewController

- (void)setupTableView {
  self.tableView = [[UITableView alloc] initWithFrame:CGRectZero style:UITableViewStyleInsetGrouped];
  self.tableView.dataSource = self;
  self.tableView.delegate = self;
  self.tableView.translatesAutoresizingMaskIntoConstraints = NO;
  [self.tableView registerClass:[UITableViewCell class] forCellReuseIdentifier:@"HabitCell"];
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
  UITableViewCell *cell = [tableView dequeueReusableCellWithIdentifier:@"HabitCell"
                                                          forIndexPath:indexPath];
  Habit *habit = self.habits[indexPath.row];
  cell.textLabel.text = habit.title;
  cell.detailTextLabel.text = [NSString stringWithFormat:@"%ld 天", (long)habit.streak];
  return cell;
}

- (void)tableView:(UITableView *)tableView didSelectRowAtIndexPath:(NSIndexPath *)indexPath {
  [tableView deselectRowAtIndexPath:indexPath animated:YES];
  Habit *habit = self.habits[indexPath.row];
  // 跳转或打卡
}

@end
```

要点：

- `dequeueReusableCellWithIdentifier:forIndexPath:` 复用 cell，必须注册复用标识符。
- 数据源方法在主线程调用，数组越界会直接崩溃。
- 自定义 cell 在 `prepareForReuse` 中重置状态，避免复用串数据。
- 大量数据用分页与增量刷新，避免一次性 `reloadData`。

### 5.1 自定义 Cell

```objc
@interface HabitCell : UITableViewCell
@property (weak, nonatomic) IBOutlet UILabel *titleLabel;
@property (weak, nonatomic) IBOutlet UILabel *streakLabel;
- (void)configureWithHabit:(Habit *)habit;
@end

@implementation HabitCell

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

## 6. UICollectionView 速览

网格布局用 `UICollectionView`：

```objc
UICollectionViewFlowLayout *layout = [[UICollectionViewFlowLayout alloc] init];
layout.itemSize = CGSizeMake(100, 100);
layout.minimumInteritemSpacing = 8;
UICollectionView *collectionView =
    [[UICollectionView alloc] initWithFrame:CGRectZero collectionViewLayout:layout];
[collectionView registerClass:[UICollectionViewCell class]
   forCellWithReuseIdentifier:@"Cell"];
```

数据源遵循 `UICollectionViewDataSource`，布局细节用 `UICollectionViewDelegateFlowLayout`。

## 7. 回调与页面传值

| 方向 | 方式 |
|---|---|
| 正向传值（A -> B） | B 暴露 property，A 在 push 前赋值 |
| 反向传值（B -> A） | delegate、block、通知 |
| 跨层解耦 | NSNotificationCenter |
| 一对多状态 | KVO |

### 7.1 Delegate

```objc
@protocol HabitEditorDelegate <NSObject>
- (void)habitEditor:(HabitEditorViewController *)editor didSaveHabit:(Habit *)habit;
@end

// 编辑器内
[self.delegate habitEditor:self didSaveHabit:habit];

// 列表页
@interface HabitListViewController () <HabitEditorDelegate>
@end

- (void)habitEditor:(HabitEditorViewController *)editor didSaveHabit:(Habit *)habit {
  [self.habits addObject:habit];
  [self.tableView reloadData];
}
```

### 7.2 Block

```objc
typedef void (^HabitSavedBlock)(Habit *habit);
@property (nonatomic, copy) HabitSavedBlock onSaved;

// 使用方
__weak typeof(self) weakSelf = self;
editor.onSaved = ^(Habit *habit) {
  __strong typeof(weakSelf) self = weakSelf;
  if (!self) { return; }
  [self.habits addObject:habit];
  [self.tableView reloadData];
};
```

## 8. 外观与适配

- 颜色用语义色或 `Assets.xcassets` 的 Any / Dark 配置，适配深色模式。
- 文本用动态字体 `UIFont.preferredFontForTextStyle:`，支持系统字号。
- 多语言用 `Localizable.strings` 与 `NSLocalizedString`。
- 屏幕适配用 Auto Layout + `safeAreaLayoutGuide`，避免写死尺寸。

## 9. 验收

- 能新建控制器并在 Storyboard / XIB 中连线 IBOutlet 与 IBAction。
- 能用 Auto Layout 约束完成贴边、居中与自适应高度。
- 能用 `UINavigationController` 完成 push / pop 与模态展示。
- 能用 UITableView 展示数据并处理点击与复用。
- 能用 delegate 或 block 完成反向传值，且无循环引用。
- 界面在深色模式与不同屏幕尺寸下可用。

## 10. 排错

| 现象 | 排查 |
|---|---|
| 界面空白 | Main Interface 是否设置，Initial View Controller 是否存在 |
| `IBOutlet` 为 nil | 连接是否断开，`viewDidLoad` 时视图是否已加载 |
| `IBAction` 不触发 | 连接是否成对，控件 `userInteractionEnabled` 是否为 YES |
| 列表崩溃在 `cellForRow` | 数据源数组是否越界，`numberOfRows` 与数据是否一致 |
| cell 内容串行 | 是否在 `prepareForReuse` 重置，复用标识符是否正确 |
| 约束冲突 | 是否少/多约束，是否设置 `translatesAutoresizingMaskIntoConstraints = NO` |
| push 无效 | 外层是否有 `UINavigationController` |
| 点击区域错位 | 约束在 `viewDidLayoutSubviews` 前未生效，检查 frame 使用时机 |
