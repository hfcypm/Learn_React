# 阶段二：数据、网络、持久化与并发

**目标**：用 NSURLSession 完成网络请求与 JSON 解析，用 Core Data / NSUserDefaults / Keychain 持久化，用 GCD 控制线程并保证 UI 回主线程。

**前置知识**：阶段一的 UIKit 与页面交互。

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

## 9. Keychain

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

## 10. 缓存策略

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

## 11. 验收

- 能用 NSURLSession 发起请求并解析 JSON。
- 能检查错误与 HTTP 状态码，并在主线程更新 UI。
- 能用 GCD 或 NSOperationQueue 处理并发任务链。
- 能用 Core Data 完成增删改查并驱动列表刷新。
- 能用 NSUserDefaults 存偏好、用 Keychain 存敏感信息。
- 能设计缓存优先的数据链路。

## 12. 排错

| 现象 | 排查 |
|---|---|
| 请求一直 pending | URL、超时、是否调用了 `resume` |
| JSON 解析崩溃 | 是否用 `isKindOfClass:` 校验类型，字段是否为 `null` |
| 主线程卡顿 | 网络/解析/数据库/解码是否在后台队列 |
| `Core Data` 崩溃 | 上下文是否同一队列使用，模型版本是否匹配 |
| 保存后列表不刷新 | 是否重新查询或使用 `NSFetchedResultsController` |
| Keychain 读写失败 | `SecItem` 返回码是否检查，签名与访问组是否匹配 |
| 缓存不更新 | TTL、ETag、是否覆盖旧数据 |
| 循环引用 | block 是否强引用 self，是否用 `__weak` |
