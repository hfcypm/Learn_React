# 阶段二：数据、网络、持久化与并发

> 本页属于 **Objective-C + UIKit 学习路线**：从零学习与接手存量工程都适用。总览见 [学习路线 README](./README.md)。

**目标**：用 NSURLSession 完成网络请求与 JSON 解析，用 Core Data / NSUserDefaults / Keychain 持久化，用 GCD 控制线程并保证 UI 回主线程。

**前置知识**：阶段一的 UIKit 与页面交互。

## Java 对照速查（数据与并发篇）

| iOS | Java | 关键差异 |
|---|---|---|
| `NSURLSession` | `HttpURLConnection` / OkHttp | 回调默认在后台线程，“回主线程刷 UI”的纪律与 OkHttp 一致 |
| `NSJSONSerialization` | Jackson / Gson | OC 无注解绑定：`NSDictionary` 手工取值映射模型，防御性判空比 Java 多 |
| `NSNumber` | `Integer` / `Double` 装箱 | JSON 数字统一成 `NSNumber`，用 `intValue` / `doubleValue` 取 |
| GCD `dispatch_async` | `executor.submit` | 串行队列 ≈ 单线程池；`dispatch_get_main_queue` ≈ UI 线程 |
| `dispatch_once` | 静态初始化 / DCL 单例 | GCD 原子保证，比手写 double-checked locking 简单 |
| `NSOperationQueue` | `ExecutorService` | 面向任务对象、可取消、可设依赖 |
| 主线程刷新 UI | Swing EDT / Android 主线程 | `dispatch_async(main_queue)` ≈ `SwingUtilities.invokeLater` |
| Core Data 上下文 | JPA `EntityManager` / Hibernate Session | 托管对象、脏检查、save 落库；对象离开上下文 ≈ detached entity |
| Core Data 模型迁移 | Flyway / Liquibase | 轻量迁移自动推断，复杂迁移要 Mapping Model |
| `NSFetchedResultsController` | 无直接对应 | 盯住数据库并驱动列表增量刷新，≈ Spring Data 分页 + 事件刷新的组合 |
| `NSUserDefaults` | `Properties` / `SharedPreferences` | 键值持久化，别存大对象与敏感数据 |
| Keychain | 无标准对应 | 系统级加密存储，密码与 token 放这里 |

三条提醒：

1. **回调线程**：`NSURLSession` 的 completion 不在主线程，等价 OkHttp 异步回调；“后台取数、主线程刷 UI”两边纪律一致。
2. **Core Data 的托管对象有归属**：跨上下文传递会失效，思路像 detached entity；后台上下文写、主上下文读、用通知合并是 iOS 标准打法。
3. **没有注解式 JSON 绑定**：字段拼错、类型不符不会在编译期暴露，解析层要集中判空判型，把崩溃挡在边界。

## 1. 网络请求

```objc
- (void)fetchHabitsWithCompletion:(void (^)(NSArray<Habit *> * _Nullable habits,
                                            NSError * _Nullable error))completion {
  NSURL *url = [NSURL URLWithString:@"https://example.com/api/habits"];
  NSMutableURLRequest *request = [NSMutableURLRequest requestWithURL:url];
  request.timeoutInterval = 10;
  [request setValue:@"application/json" forHTTPHeaderField:@"Accept"];

  NSURLSessionDataTask *task =
  [[NSURLSession sharedSession] dataTaskWithRequest:request
                                  completionHandler:^(NSData *data,
                                                      NSURLResponse *response,
                                                      NSError *error) {
    if (error) {
      completion(nil, error);
      return;
    }
    NSHTTPURLResponse *http = (NSHTTPURLResponse *)response;
    if (http.statusCode < 200 || http.statusCode >= 300) {
      NSError *statusError = [NSError errorWithDomain:@"HTTP"
                                                 code:http.statusCode
                                             userInfo:@{NSLocalizedDescriptionKey: @"服务异常"}];
      completion(nil, statusError);
      return;
    }
    NSError *jsonError = nil;
    id json = [NSJSONSerialization JSONObjectWithData:data options:0 error:&jsonError];
    if (![json isKindOfClass:[NSDictionary class]]) {
      completion(nil, jsonError ?: [NSError errorWithDomain:@"JSON" code:-1 userInfo:nil]);
      return;
    }
    NSMutableArray<Habit *> *habits = [NSMutableArray array];
    for (NSDictionary *dict in json[@"items"]) {
      if (![dict isKindOfClass:[NSDictionary class]]) { continue; }
      Habit *habit = [[Habit alloc] initWithTitle:dict[@"title"] ?: @"未命名"];
      habit.streak = [dict[@"streak"] integerValue];
      [habits addObject:habit];
    }
    completion(habits, nil);
  }];
  [task resume];
}
```

要点：

- `NSURLSession` 的回调在后台线程，UI 更新前必须切主线程。
- 必须检查 `NSError` 与 `statusCode`。
- `NSJSONSerialization` 返回 `id`，用 `isKindOfClass:` 做类型校验，避免崩溃。
- 生产环境使用 https；http 需在 `Info.plist` 配置 ATS 例外。

## 2. JSON 与模型映射

```objc
NSDictionary *json = [NSJSONSerialization JSONObjectWithData:data options:0 error:&error];
NSString *title = json[@"title"];
NSInteger streak = [json[@"streak"] integerValue];
BOOL enabled = [json[@"enabled"] boolValue];
```

老工程常用映射库：

| 库 | 用途 |
|---|---|
| YYModel | 字典转模型，性能好 |
| Mantle | 模型与 JSON 双向映射 |
| JSONModel | 声明式映射 |

新写代码也可以手写 `initWithDictionary:`，减少依赖。

## 3. 并发：GCD

```objc
// 后台执行耗时任务，回主线程更新 UI
dispatch_async(dispatch_get_global_queue(QOS_CLASS_UTILITY, 0), ^{
  NSData *data = [NSData dataWithContentsOfURL:url];
  dispatch_async(dispatch_get_main_queue(), ^{
    self.imageView.image = [UIImage imageWithData:data];
  });
});

// 只执行一次
static dispatch_once_t onceToken;
dispatch_once(&onceToken, ^{
  // 初始化单例或全局配置
});

// 并发任务聚合
dispatch_group_t group = dispatch_group_create();
dispatch_group_enter(group);
[self fetchProfile:^{ dispatch_group_leave(group); }];
dispatch_group_enter(group);
[self fetchHabits:^{ dispatch_group_leave(group); }];
dispatch_group_notify(group, dispatch_get_main_queue(), ^{
  [self reloadUI];
});
```

原则：主线程只做 UI；网络、解析、数据库、图片解码放后台队列。

## 4. 并发：NSOperationQueue

```objc
NSOperationQueue *queue = [[NSOperationQueue alloc] init];
queue.maxConcurrentOperationCount = 3;

NSBlockOperation *download = [NSBlockOperation blockOperationWithBlock:^{
  // 下载
}];
NSBlockOperation *parse = [NSBlockOperation blockOperationWithBlock:^{
  // 解析
}];
[parse addDependency:download];
[queue addOperation:download];
[queue addOperation:parse];
```

适合有先后依赖、需要限制并发数的任务链。取消用 `cancelAllOperations`，操作内部检查 `isCancelled`。

## 5. 持久化方案选型

| 方案 | 适用 | 说明 |
|---|---|---|
| `NSUserDefaults` | 轻量偏好、开关、上次打开时间 | 不适合业务数据 |
| 文件 / plist | 小规模结构化数据 | 用 `NSFileManager` 管理路径 |
| `NSKeyedArchiver` | 模型归档 | 模型需实现 `NSCoding` / `NSSecureCoding` |
| Core Data | 复杂对象图、查询、关系 | 老工程最常见数据库方案 |
| SQLite / FMDB | 已有 SQL、需要精细控制 | 直接写 SQL |
| Keychain | 令牌、密码等敏感信息 | 不放 UserDefaults |

## 6. NSUserDefaults

```objc
NSUserDefaults *defaults = [NSUserDefaults standardUserDefaults];
[defaults setBool:YES forKey:@"reminderEnabled"];
[defaults setObject:[NSDate date] forKey:@"lastOpenAt"];
[defaults synchronize];   // 现代系统一般无需手动调用

BOOL enabled = [defaults boolForKey:@"reminderEnabled"];
NSDate *lastOpen = [defaults objectForKey:@"lastOpenAt"];
```

## 7. 文件与归档

```objc
NSString *directory = NSSearchPathForDirectoriesInDomains(NSDocumentDirectory,
                                                          NSUserDomainMask, YES).firstObject;
NSString *path = [directory stringByAppendingPathComponent:@"habits.plist"];

// 写入 plist
[habitsArray writeToFile:path atomically:YES];

// 归档模型（模型需遵循 NSSecureCoding）
NSError *error = nil;
NSData *data = [NSKeyedArchiver archivedDataWithRootObject:habits
                                     requiringSecureCoding:YES
                                                     error:&error];
[data writeToFile:path atomically:YES];
```

## 8. Core Data

### 8.1 容器初始化（AppDelegate）

```objc
@synthesize persistentContainer = _persistentContainer;

- (NSPersistentContainer *)persistentContainer {
  @synchronized (self) {
    if (_persistentContainer == nil) {
      _persistentContainer = [[NSPersistentContainer alloc] initWithName:@"LegacyHabit"];
      [_persistentContainer loadPersistentStoresWithCompletionHandler:
       ^(NSPersistentStoreDescription *description, NSError *error) {
        if (error) {
          NSLog(@"Core Data 加载失败：%@", error);
          abort();
        }
      }];
    }
  }
  return _persistentContainer;
}
```

### 8.2 查询

```objc
NSManagedObjectContext *context = self.persistentContainer.viewContext;
NSFetchRequest<Habit *> *request = [Habit fetchRequest];
request.sortDescriptors = @[[NSSortDescriptor sortDescriptorWithKey:@"createdAt"
                                                          ascending:NO]];
request.predicate = [NSPredicate predicateWithFormat:@"title != nil"];

NSError *error = nil;
NSArray<Habit *> *habits = [context executeFetchRequest:request error:&error];
if (error) {
  NSLog(@"查询失败：%@", error);
}
```

### 8.3 写入与删除

```objc
Habit *habit = [NSEntityDescription insertNewObjectForEntityForName:@"Habit"
                                             inManagedObjectContext:context];
habit.title = @"阅读";
habit.createdAt = [NSDate date];

[context delete:habit];

if ([context hasChanges]) {
  NSError *error = nil;
  if (![context save:&error]) {
    NSLog(@"保存失败：%@", error);
  }
}
```

### 8.4 列表联动

用 `NSFetchedResultsController` 监听数据变化，配合 `UITableView` 增量刷新；简单场景也可在写操作后重新查询并 `reloadData`。

老工程也可能使用 `MagicalRecord`、`Realm` 或 `FMDB`。先确认现有方案，再决定是否迁移，避免同一工程混用多套数据库。

### 8.5 轻量迁移与并发

模型加字段、改字段名时优先用轻量迁移：

```objc
NSPersistentStoreDescription *desc = container.persistentStoreDescriptions.firstObject;
desc.shouldMigrateStoreAutomatically = YES;
desc.shouldInferMappingModelAutomatically = YES;
[container loadPersistentStoresWithCompletionHandler:^(NSPersistentStoreDescription *d, NSError *error) {
  if (error) { NSLog(@"迁移失败：%@", error); }
}];
```

- 轻量迁移支持新增/删除属性、重命名（配合 `renamingIdentifier`）、新增实体。
- 复杂变更（拆分实体、数据转换）需要 Mapping Model 或自定义迁移。
- 迁移前备份数据库，并在旧版本数据上验证。

并发访问 Core Data：

```objc
NSManagedObjectContext *background = [container newBackgroundContext];
[background performBlock:^{
  // 在 background 队列查询/写入
  NSError *error = nil;
  if (![background save:&error]) {
    NSLog(@"后台保存失败：%@", error);
  }
}];
```

`NSManagedObject` 不是线程安全的。跨上下文传递用 `NSManagedObjectID`，在目标上下文重新 fetch，不要直接传对象。

## 9. 本地通知与权限

习惯提醒、超时提醒等场景用 `UNUserNotificationCenter`：

```objc
#import <UserNotifications/UserNotifications.h>

UNMutableNotificationContent *content = [[UNMutableNotificationContent alloc] init];
content.title = @"该打卡了";
content.body = @"完成今天的习惯";
content.sound = [UNNotificationSound defaultSound];

NSDateComponents *components = [[NSDateComponents alloc] init];
components.hour = 20;
components.minute = 0;
UNCalendarNotificationTrigger *trigger =
    [UNCalendarNotificationTrigger triggerWithDateMatchingComponents:components repeats:YES];

UNNotificationRequest *request =
    [UNNotificationRequest requestWithIdentifier:@"habit.reminder"
                                          content:content
                                          trigger:trigger];

UNUserNotificationCenter *center = [UNUserNotificationCenter currentNotificationCenter];
[center requestAuthorizationWithOptions:(UNAuthorizationOptionAlert | UNAuthorizationOptionSound)
                      completionHandler:^(BOOL granted, NSError *error) {
  if (granted) {
    [center addNotificationRequest:request error:nil];
  }
}];
```

要点：

- 先申请授权，用户拒绝后不要反复弹窗，引导到系统设置。
- 前台展示通知需实现 `UNUserNotificationCenterDelegate` 的 `willPresentNotification:`。
- 通知标识符保持稳定，重复添加会覆盖；取消用 `removePendingNotificationRequestsWithIdentifiers:`。
- 权限用途在 `Info.plist` 与隐私材料中保持一致。

## 10. Keychain

```objc
#import <Security/Security.h>

+ (void)saveData:(NSData *)data forKey:(NSString *)key {
  NSDictionary *query = @{
    (__bridge id)kSecClass: (__bridge id)kSecClassGenericPassword,
    (__bridge id)kSecAttrAccount: key
  };
  SecItemDelete((__bridge CFDictionaryRef)query);

  NSMutableDictionary *attrs = [query mutableCopy];
  attrs[(__bridge id)kSecValueData] = data;
  SecItemAdd((__bridge CFDictionaryRef)attrs, NULL);
}

+ (NSData *)dataForKey:(NSString *)key {
  NSDictionary *query = @{
    (__bridge id)kSecClass: (__bridge id)kSecClassGenericPassword,
    (__bridge id)kSecAttrAccount: key,
    (__bridge id)kSecReturnData: @YES,
    (__bridge id)kSecMatchLimit: (__bridge id)kSecMatchLimitOne
  };
  CFTypeRef result = NULL;
  SecItemCopyMatching((__bridge CFDictionaryRef)query, &result);
  return (__bridge_transfer NSData *)result;
}
```

## 11. 缓存策略

```text
1. 读本地缓存（Core Data / NSCache）-> 立即渲染
2. 发起网络请求
3. 成功 -> 更新缓存 -> 主线程刷新 UI
4. 失败 -> 保留缓存 -> 展示可重试提示
```

- 内存缓存用 `NSCache`，避免自己实现淘汰。
- HTTP 缓存用 `NSURLCache`、`ETag`、`Cache-Control`。
- 图片用 `SDWebImage` 或自定义磁盘缓存。
- 为缓存加 TTL，避免展示过期数据。

## 12. 验收

- 能用 NSURLSession 发起请求并解析 JSON。
- 能检查错误与 HTTP 状态码，并在主线程更新 UI。
- 能用 GCD 或 NSOperationQueue 处理并发任务链。
- 能用 Core Data 完成增删改查并驱动列表刷新。
- 能处理 Core Data 轻量迁移与后台上下文写入。
- 能申请通知权限并添加本地通知。
- 能用 NSUserDefaults 存偏好、用 Keychain 存敏感信息。
- 能设计缓存优先的数据链路。

## 13. 排错

| 现象 | 排查 |
|---|---|
| 请求一直 pending | URL、超时、是否调用了 `resume` |
| JSON 解析崩溃 | 是否用 `isKindOfClass:` 校验类型，字段是否为 `null` |
| 主线程卡顿 | 网络/解析/数据库/解码是否在后台队列 |
| `Core Data` 崩溃 | 上下文是否同一队列使用，模型版本是否匹配 |
| 升级后数据丢失 | 是否启用轻量迁移，模型版本与 Mapping Model 是否正确 |
| 保存后列表不刷新 | 是否重新查询或使用 `NSFetchedResultsController` |
| 通知不触发 | 是否申请授权、触发条件是否满足、是否被前台代理拦截 |
| Keychain 读写失败 | `SecItem` 返回码是否检查，签名与访问组是否匹配 |
| 缓存不更新 | TTL、ETag、是否覆盖旧数据 |
| 循环引用 | block 是否强引用 self，是否用 `__weak` |
