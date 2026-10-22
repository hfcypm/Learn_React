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

### 4.2 Unwind Segue 返回

Storyboard 中从编辑页返回列表并带回数据，可以用 Unwind Segue：

```objc
// 在返回后停留的控制器（列表页）中声明
- (IBAction)unwindToHabitList:(UIStoryboardSegue *)segue {
  HabitEditorViewController *editor = segue.sourceViewController;
  if (editor.savedHabit) {
    [self.habits addObject:editor.savedHabit];
    [self.tableView reloadData];
  }
}
```

在 Storyboard 中把编辑页的按钮拖到顶部 Exit 图标，选择 `unwindToHabitList:` 即可。返回逻辑集中在目标控制器，避免跨页面反注册回调。

### 4.3 弹窗：UIAlertController

```objc
UIAlertController *alert =
    [UIAlertController alertControllerWithTitle:@"确认删除"
                                        message:@"删除后不可恢复"
                                 preferredStyle:UIAlertControllerStyleAlert];
[alert addAction:[UIAlertAction actionWithTitle:@"取消"
                                          style:UIAlertActionStyleCancel
                                        handler:nil]];
[alert addAction:[UIAlertAction actionWithTitle:@"删除"
                                          style:UIAlertActionStyleDestructive
                                        handler:^(UIAlertAction *action) {
  [self deleteHabit:habit];
}]];
[self presentViewController:alert animated:YES completion:nil];
```

- iOS 8 起统一用 `UIAlertController`；老代码里的 `UIAlertView` / `UIActionSheet` 已废弃，维护时优先替换。
- 弹窗也是模态：上一个 present 未完成就再 present 会报 "Attempt to present ... while a presentation is in progress"，先检查 `presentedViewController` 或等 completion。

## 5. UITableView

```objc
@interface HabitListViewController () <UITableViewDataSource, UITableViewDelegate>
@property (nonatomic, strong) UITableView *tableView;
@property (nonatomic, strong) NSMutableArray<Habit *> *habits;
@end

@implementation HabitListViewController

- (void)setupTableView {
  self.tableView = [[UITableView alloc] initWithFrame:CGRectZero
                                                style:UITableViewStyleInsetGrouped]; // iOS 13+，低版本用 UITableViewStyleGrouped
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

`HabitCell` 用 XIB 实现时改用 nib 注册，与 `registerClass:` 二选一：

```objc
[self.tableView registerNib:[UINib nibWithNibName:@"HabitCell" bundle:nil]
     forCellReuseIdentifier:@"HabitCell"];
```

### 5.2 分组、自适应高度与滑动操作

```objc
// 分组的行数
- (NSInteger)tableView:(UITableView *)tableView numberOfRowsInSection:(NSInteger)section {
  return [self.habitsBySection[section] count];
}

// 自适应高度
self.tableView.rowHeight = UITableViewAutomaticDimension;
self.tableView.estimatedRowHeight = 60;

// iOS 11+ 滑动删除
- (UISwipeActionsConfiguration *)tableView:(UITableView *)tableView
    trailingSwipeActionsConfigurationForRowAtIndexPath:(NSIndexPath *)indexPath {
  UIContextualAction *delete =
  [UIContextualAction contextualActionWithStyle:UIContextualActionStyleDestructive
                                          title:@"删除"
                                        handler:^(UIContextualAction *action,
                                                  UIView *sourceView,
                                                  void (^completionHandler)(BOOL)) {
    [self.habits removeObjectAtIndex:indexPath.row];
    [tableView deleteRowsAtIndexPaths:@[indexPath]
                     withRowAnimation:UITableViewRowAnimationAutomatic];
    completionHandler(YES);
  }];
  return [UISwipeActionsConfiguration configurationWithActions:@[delete]];
}
```

要点：删除时先改数据源再更新 UI，数量必须与 `numberOfRows` 一致；复杂列表可用 `UITableViewDiffableDataSource` 降低一致性风险。

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

数据源遵循 `UICollectionViewDataSource`，布局细节用 `UICollectionViewDelegateFlowLayout`。cell 注册方式与 UITableView 相同，支持 `registerClass:` 与 `registerNib:`；老代码常见的自定义流水布局，重点看 `prepareLayout` 与 `layoutAttributesForElementsInRect:`。

## 7. 控件事件与键盘

### 7.1 target-action 与 IBAction

```objc
[self.checkInButton addTarget:self
                       action:@selector(checkInTapped:)
             forControlEvents:UIControlEventTouchUpInside];
```

- `IBAction` 是 target-action 在 Storyboard 里的连线形式，两者等价。
- 常用事件：`UIControlEventTouchUpInside`（按钮）、`UIControlEventValueChanged`（开关、滑块、日期选择器）、`UIControlEventEditingDidEnd`（输入框结束编辑）。
- 按钮无反应的排查顺序：是否 `addTarget`、事件类型是否匹配、`enabled` 与 `userInteractionEnabled` 是否为 YES、控件是否被别的视图盖住。

### 7.2 键盘处理

```objc
// 收起键盘：页面任意位置
[self.view endEditing:YES];

// 滚动列表时自动收起
self.tableView.keyboardDismissMode = UIScrollViewKeyboardDismissModeInteractive;

// 点击空白处收起
UITapGestureRecognizer *tap =
    [[UITapGestureRecognizer alloc] initWithTarget:self action:@selector(dismissKeyboard)];
[self.view addGestureRecognizer:tap];
```

- 输入框被键盘遮挡时：监听 `UIKeyboardWillChangeFrameNotification`，调整底部约束或改 scrollView 的 `contentInset`。
- 老工程普遍集成 IQKeyboardManager 自动避让，维护时先确认是否已启用，避免自己再监听一遍造成双重调整。
- `UITextFieldDelegate` 的 `textFieldShouldReturn:` 处理回车；输入框之间跳转用 `becomeFirstResponder` 与 `resignFirstResponder`。

## 8. 回调与页面传值

| 方向 | 方式 |
|---|---|
| 正向传值（A -> B） | B 暴露 property，A 在 push 前赋值 |
| 反向传值（B -> A） | delegate、block、通知 |
| 跨层解耦 | NSNotificationCenter |
| 一对多状态 | KVO |

### 8.1 Delegate

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

### 8.2 Block

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

## 9. 外观与适配

- 颜色用语义色或 `Assets.xcassets` 的 Any / Dark 配置，适配深色模式。
- 文本用动态字体 `UIFont.preferredFontForTextStyle:`，支持系统字号。
- 多语言用 `Localizable.strings` 与 `NSLocalizedString`。
- 屏幕适配用 Auto Layout + `safeAreaLayoutGuide`，避免写死尺寸。
- 网络图片用工程现有的图片库加载并给占位图，如 `[imageView sd_setImageWithURL:url placeholderImage:placeholder]`（SDWebImage），避免自行管理下载与缓存。

```objc
// 动态字体
self.titleLabel.font = [UIFont preferredFontForTextStyle:UIFontTextStyleHeadline];
self.titleLabel.adjustsFontForContentSizeCategory = YES;

// 无障碍描述，VoiceOver 会朗读
self.checkInButton.accessibilityLabel = @"打卡";
self.checkInButton.accessibilityHint = @"将连续天数加一";
self.checkInButton.accessibilityTraits = UIAccessibilityTraitButton;
```

无障碍检查：按钮点击区域不小于 44x44pt，文本对比度足够，图标按钮都有 `accessibilityLabel`，用 VoiceOver 走一遍主流程。

## 10. 验收

- 能新建控制器并在 Storyboard / XIB 中连线 IBOutlet 与 IBAction。
- 能用 Auto Layout 约束完成贴边、居中与自适应高度。
- 能用 `UINavigationController` 完成 push / pop 与模态展示。
- 能用 `UIAlertController` 完成确认弹窗，并理解弹窗的模态本质。
- 能处理键盘收起与遮挡避让。
- 能用 UITableView 展示数据并处理点击与复用。
- 能用 delegate 或 block 完成反向传值，且无循环引用。
- 界面在深色模式与不同屏幕尺寸下可用。

## 11. 排错

| 现象 | 排查 |
|---|---|
| 界面空白 | Main Interface 是否设置，Initial View Controller 是否存在 |
| `IBOutlet` 为 nil | 连接是否断开，`viewDidLoad` 时视图是否已加载 |
| `IBAction` 不触发 | 连接是否成对，控件 `userInteractionEnabled` 是否为 YES |
| 弹窗报 "presentation is in progress" | 是否连续 present，检查 `presentedViewController`，等上一个 completion |
| 输入框被键盘遮挡 | 是否监听键盘通知调整约束或 inset，是否与 IQKeyboardManager 重复处理 |
| 列表崩溃在 `cellForRow` | 数据源数组是否越界，`numberOfRows` 与数据是否一致 |
| cell 内容串行 | 是否在 `prepareForReuse` 重置，复用标识符是否正确 |
| 约束冲突 | 是否少/多约束，是否设置 `translatesAutoresizingMaskIntoConstraints = NO` |
| push 无效 | 外层是否有 `UINavigationController` |
| 点击区域错位 | 约束在 `viewDidLayoutSubviews` 前未生效，检查 frame 使用时机 |
