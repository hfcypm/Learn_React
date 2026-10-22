# 附录：C 语言基础（面向 Objective-C 老代码阅读）

**目标**：用 2-3 天补齐读 Objective-C / iOS 老代码所需的 C 基础，覆盖变量与类型、分支与循环、函数与参数、指针、结构体、内存与未定义行为、存储类与多文件工程、标准库、C 与 OC 的边界。

**用法**：这是 [阶段零](./stage-0-foundations.md) 的预备读物，不追求完整覆盖 C 语言。建议边读边在 Xcode 里建一个 macOS Command Line Tool（语言选 C）跑示例，不要只看不写。

## 1. 怎么练

```bash
# 单文件编译运行
clang main.c -o main
./main

# 打开所有警告，接近 Xcode 的严格程度
clang -Wall -Wextra main.c -o main
```

Xcode 方式：`File -> New -> Project -> macOS -> Command Line Tool`，Language 选 C，然后把示例贴进 `main.c`。Xcode 的断点、变量查看、LLDB 在 C 工程里同样可用，练指针时比 `printf` 更直观。

## 2. 变量与类型

```c
#include <stdio.h>
#include <stdbool.h>
#include <stdint.h>

int main(void) {
  int count = 3;              // 32 位整数，最常用
  long total = 100L;          // arm64 上是 64 位，armv7 上是 32 位
  unsigned int mask = 0xFFu;  // 无符号，回绕而不是溢出
  float ratio = 0.5f;         // 单精度
  double precise = 0.5;       // 双精度
  bool done = false;          // 需要 stdbool.h
  int32_t fixed = -1;         // 固定宽度，跨平台稳定
  size_t length = 10;         // 用于长度、下标，无符号

  printf("count=%d total=%ld ratio=%f length=%zu\n",
         count, total, ratio, length);
  printf("sizeof(int)=%zu sizeof(long)=%zu\n", sizeof(int), sizeof(long));
  return 0;
}
```

要点：

- `sizeof` 返回 `size_t`，用 `%zu` 打印。
- 有符号与无符号混算时，有符号会被转成无符号，`-1 < 1u` 可能为假。
- `long` 的宽度随架构变化，跨平台数据用 `<stdint.h>` 的 `int32_t` / `uint64_t`。
- OC 里的 `NSInteger` / `NSUInteger` / `CGFloat` 就是按平台切换宽度的类型别名。

老代码里常见的类型对照：

| C | Objective-C / iOS | 注意 |
|---|---|---|
| `int` | `int`、`NSInteger`（64 位） | 混用会截断 |
| `unsigned int` | `NSUInteger` | 做减法可能回绕成巨大值 |
| `float` / `double` | `CGFloat` | 32 位设备是 `float`，64 位是 `double` |
| `char *` | `const char *` | OC 字符串转 C 字符串要管理编码与生命周期 |
| `bool` | `BOOL` | `BOOL` 在老 SDK 里是 `signed char`，只有 0/1 语义 |

## 3. 运算符、分支与循环

```c
int score = 72;

if (score >= 90) {
  printf("A\n");
} else if (score >= 60) {
  printf("Pass\n");
} else {
  printf("Fail\n");
}

switch (score / 10) {
  case 9:
  case 10:
    printf("A\n");
    break;
  case 6:
  case 7:
  case 8:
    printf("Pass\n");
    break;
  default:
    printf("Fail\n");
    break;
}

int sum = 0;
for (int i = 0; i < 10; i++) {
  if (i == 5) { continue; }
  sum += i;
}

int n = 3;
while (n > 0) {
  n--;
}
```

要点：

- `=` 是赋值，`==` 是比较；`if (x = 1)` 在 C 里合法，编译器只会给警告。
- `switch` 只能用于整型与枚举，漏写 `break` 会继续执行下一个分支。
- 无符号变量做 `for (size_t i = count - 1; i >= 0; i--)` 会死循环，下界判断要改写。
- 位运算在老代码里很常见：`option & Flag` 判断、`option | Flag` 组合、`option & ~Flag` 清除。

老代码场景：`UIViewAutoresizing`、`NS_OPTIONS`、权限位、状态机，底层都是整数与位掩码。

## 4. 函数与参数

```c
// 声明：放在头文件里给其他文件用
int add(int a, int b);

// 定义
int add(int a, int b) {
  return a + b;
}

// 指针作为输出参数，函数返回是否成功
bool divide(int a, int b, int *outResult) {
  if (b == 0) {
    return false;
  }
  *outResult = a / b;
  return true;
}

int main(void) {
  int result = 0;
  if (divide(10, 2, &result)) {
    printf("%d\n", result);
  }
  return 0;
}
```

要点：

- C 只有值传递。想通过参数把结果带出来，就传指针。
- 数组作为参数会退化成指针，函数内部拿不到长度，必须额外传 `count`。
- 函数要先声明后使用；跨文件调用靠头文件声明 + 链接。
- `static` 修饰函数表示只在当前文件可见；`extern` 表示定义在别处。

函数指针是 block 出现之前的老回调方式：

```c
typedef void (*HabitCallback)(int habitId, bool success);

void loadHabit(int habitId, HabitCallback callback) {
  // 模拟异步完成
  callback(habitId, true);
}

void onLoaded(int habitId, bool success) {
  printf("habit %d: %d\n", habitId, success);
}

int main(void) {
  loadHabit(1, onLoaded);
  return 0;
}
```

老代码场景：C 函数回调、`qsort` 比较器、`target-action` 之前的观察者、`dispatch_apply` 的 C 函数版本。

## 5. 指针与取地址（重点）

```c
int value = 42;
int *pointer = &value;   // & 取地址，pointer 保存 value 的地址
printf("%d\n", *pointer); // * 解引用，读取地址上的值
*pointer = 43;            // 通过指针修改原变量

int *empty = NULL;        // 空指针，解引用会崩溃
if (pointer != NULL) {
  printf("%d\n", *pointer);
}
```

要点：

- 指针保存地址，类型决定解引用时读多少字节、如何解释。
- 未初始化或指向已释放内存的指针是野指针，解引用通常得到 `EXC_BAD_ACCESS`。
- `free` 之后指针不会自动变 `NULL`，要手动置空，避免重复释放。
- `const int *` 表示不能通过该指针改数据；`int * const` 表示指针本身不能改。

`void *` 是通用指针，可以指向任意类型，解引用前必须转回具体类型：

```c
int value = 42;
void *raw = &value;
int *typed = (int *)raw;
printf("%d\n", *typed);
```

老代码里的回调 `context`、`dispatch_set_context`、`memcpy` 的目标参数都是 `void *`。

指针数组与数组指针容易混淆：

```c
int a = 1, b = 2;
int *arrayOfPointers[2] = { &a, &b };  // 数组，元素是 int *

int row[2] = { 3, 4 };
int (*pointerToArray)[2] = &row;       // 指针，指向 int[2]

printf("%d\n", *arrayOfPointers[0]);
printf("%d\n", (*pointerToArray)[1]);
```

读声明时从变量名往外看：`*arrayOfPointers[2]` 先结合 `[]`，所以是数组；`(*pointerToArray)[2]` 先结合 `*`，所以是指针。

指针与数组：

```c
int values[3] = {10, 20, 30};
int *p = values;          // 数组名退化为首元素地址
printf("%d\n", p[0]);
printf("%d\n", *(p + 1)); // 指针算术，等价于 p[1]
```

- 数组下标本质是指针算术，C 不检查越界，越界会踩坏其他内存。
- 数组长度用 `sizeof(values) / sizeof(values[0])`，只对当前作用域内的真数组有效。

字符串：

```c
char name[] = "Ada";           // 可修改，长度 4（含 '\0'）
const char *literal = "Ada";   // 只读字符串字面量
size_t length = strlen(name);  // 3，不包含 '\0'

for (size_t i = 0; name[i] != '\0'; i++) {
  printf("%c", name[i]);
}
```

- C 字符串以 `'\0'` 结尾，缺了终止符，`strlen` 与 `printf("%s")` 会一路读到越界。
- `strcpy` / `strcat` 不检查目标缓冲区大小，老代码里的缓冲区溢出多来自这里。
- 更安全的替代：`snprintf`、`strncpy`（注意手动补 `'\0'`）、`strlcpy`。

指向指针的指针：

```c
bool parse(const char *text, int *outValue) {
  // 省略解析
  *outValue = 0;
  return true;
}

void load(void) {
  int value = 0;
  if (parse("42", &value)) {
    printf("%d\n", value);
  }
}
```

OC 里 `NSError **` 是同一个模式：函数返回 `BOOL` 表示成功与否，错误对象通过指针带回。看到 `&error` 就应该想到“输出参数”。

老代码场景：`NSError **`、`id` 与 `void *` 互转、Core Foundation 与 OC 桥接、图像像素缓冲区、`memcpy` / `memset`。

## 6. 结构体、枚举与联合

```c
typedef struct {
  int x;
  int y;
} Point;

typedef struct {
  Point origin;
  int width;
  int height;
} Rect;

int main(void) {
  Point p = { .x = 10, .y = 20 };
  Rect r = { .origin = p, .width = 100, .height = 50 };
  printf("%d %d\n", r.origin.x, r.width);
  printf("size=%zu\n", sizeof(Rect));
  return 0;
}
```

要点：

- 结构体按值传递会整块拷贝，大结构体用指针传递。
- 结构体成员之间可能有填充字节，`sizeof` 不一定等于成员大小之和。
- `typedef` 让类型名更短，OC 的 `CGRect` / `CGPoint` / `CGSize` / `NSRange` 都是结构体。

结构体指针用 `->` 访问成员：

```c
void move(Point *point, int dx, int dy) {
  point->x += dx;   // 等价于 (*point).x
  point->y += dy;
}

int main(void) {
  Point p = { .x = 0, .y = 0 };
  move(&p, 10, 20);
  printf("%d %d\n", p.x, p.y);
  return 0;
}
```

- 拿到结构体值用 `.`，拿到结构体指针用 `->`。
- 函数要修改结构体时必须传指针，按值传参改的是副本。
- 位域在老协议结构里常见：`unsigned int enabled : 1;` 表示只占 1 位，读到能算清含义即可。

枚举：

```c
typedef enum {
  HabitStateIdle = 0,
  HabitStateDone = 1
} HabitState;
```

OC 用 `NS_ENUM` / `NS_OPTIONS` 声明，保证与底层类型和 Swift 的兼容：

```objc
typedef NS_ENUM(NSInteger, HabitState) {
  HabitStateIdle = 0,
  HabitStateDone = 1
};

typedef NS_OPTIONS(NSUInteger, HabitOptions) {
  HabitOptionNone   = 0,
  HabitOptionRemind = 1 << 0,
  HabitOptionShare  = 1 << 1
};
```

`union` 让多个成员共享同一块内存，老代码里用于协议解析、类型双关，日常业务代码很少用，看到能读懂即可。

## 7. 内存布局与生命周期

| 区域 | 存放内容 | 生命周期 |
|---|---|---|
| 栈 | 局部变量、函数参数 | 函数返回即失效 |
| 堆 | `malloc` / `calloc` 分配的内存、OC 对象 | 手动 `free` 或引用计数归零 |
| 全局 / 静态区 | 全局变量、`static` 变量 | 整个进程 |
| 常量区 | 字符串字面量、`const` 数据 | 整个进程 |

```c
#include <stdlib.h>
#include <string.h>

int *create_values(size_t count) {
  int *values = calloc(count, sizeof(int)); // 分配并清零
  return values;                            // 返回堆内存，调用方负责 free
}

int main(void) {
  size_t capacity = 4;
  int *values = create_values(capacity);
  if (values == NULL) {
    return 1;
  }

  values[0] = 1;
  free(values);
  values = NULL; // 避免悬空指针
  return 0;
}
```

动态扩容：

```c
int *grown = realloc(values, newCapacity * sizeof(int));
if (grown == NULL) {
  free(values);   // realloc 失败时原内存仍然有效
  return -1;
}
values = grown;
```

常见错误：

- 忘记 `free`：内存泄漏。
- `free` 后继续使用：悬空指针，随机崩溃。
- 重复 `free`：堆破坏。
- 在函数里返回局部数组的地址：函数返回后栈内存失效。
- 在栈上开超大数组：栈溢出，尤其在递归或主线程里。

### 未定义行为

下面这些写法编译器不报错，但结果不可预测，排查崩溃时要优先怀疑：

- 有符号整数溢出（`INT_MAX + 1`），无符号是回绕、有符号是未定义。
- 读取未初始化的局部变量。
- 数组越界读或写。
- 使用已 `free` 的内存、重复 `free`、`free` 非堆指针。
- 用不兼容的类型指针读写同一块内存（strict aliasing）。
- 在同一表达式里多次修改同一变量，如 `i = i++`。

Xcode 的 Scheme -> Diagnostics 里开启 Address Sanitizer 与 Undefined Behavior Sanitizer，可以更快复现这类问题。

与 OC 的关系：OC 对象也在堆上，`alloc` 分配、`init` 初始化、引用计数归零后释放。ARC 只是把引用计数操作交给编译器，理解 C 的栈/堆划分后，`strong` / `weak` / 循环引用会更容易想明白。

## 8. 预处理与宏

OC 工程的头文件里常见这种写法：

```c
#ifndef Habit_h
#define Habit_h

#define kMaxHabitCount 20
#define SQUARE(x) ((x) * (x))

#ifdef DEBUG
#define HabitLog(fmt, ...) NSLog((@"[Habit] " fmt), ##__VA_ARGS__)
#else
#define HabitLog(fmt, ...)
#endif

#endif /* Habit_h */
```

要点：

- 头文件用 `#ifndef` / `#define` / `#endif` 防止重复包含；OC 用 `#import` 自动处理。
- 宏是文本替换，参数必须加括号：`#define SQUARE(x) x * x` 在 `SQUARE(a + 1)` 时会算错。
- 宏没有类型检查，能用 `static const` 或函数替代时优先替代。
- `#ifdef DEBUG` 常用于日志、断言与测试开关。

老代码场景：`DEBUG` 日志、颜色与尺寸常量、多环境接口地址、`#pragma mark` 分节。

## 9. 存储类、链接与多文件工程

`static` 在不同位置含义不同：

- 修饰局部变量：只初始化一次，函数返回后值仍保留。
- 修饰全局变量或函数：只在当前 `.c` / `.m` 文件可见。
- 头文件里只放 `extern` 声明，不要放全局变量定义。

```c
// Habit.h
#ifndef Habit_h
#define Habit_h

extern const int kMaxHabitCount;

int habit_add(int a, int b);

#endif /* Habit_h */
```

```c
// Habit.c
#include "Habit.h"

const int kMaxHabitCount = 20;

static int callCount = 0;

static void log_call(void) {
  callCount++;
}

int habit_add(int a, int b) {
  log_call();
  return a + b;
}
```

- `extern` 是声明，告诉编译器“定义在别处”；定义全工程只能有一处。
- `static` 函数与全局变量是文件私有，适合放内部辅助逻辑。
- `inline` 只是建议内联；头文件里的内联函数写成 `static inline`。
- 多文件工程靠头文件声明加链接器解析符号：缺定义报 `Undefined symbol`，重复定义报 `Duplicate symbol`。
- `.m`（OC）与 `.c`（C）可以混合编译。公共 C 头文件要被 `.mm`（OC++）引用时，必须用 `extern "C"` 防止 C++ 名称修饰：

```c
#ifdef __cplusplus
extern "C" {
#endif

int habit_count(void);

#ifdef __cplusplus
}
#endif
```

## 10. 常用标准库速查

| 头文件 | 常用函数 | 用途 |
|---|---|---|
| `<string.h>` | `memcpy` `memmove` `memset` `memcmp` | 内存块拷贝、填充、比较 |
| `<string.h>` | `strlen` `strcmp` `strncmp` `strstr` `strchr` | 字符串长度、比较、查找 |
| `<stdlib.h>` | `malloc` `calloc` `realloc` `free` | 堆内存管理 |
| `<stdlib.h>` | `atoi` `strtol` `strtod` | 字符串转数字 |
| `<stdlib.h>` | `qsort` `bsearch` `abs` `labs` | 排序、查找、绝对值 |
| `<stdio.h>` | `printf` `snprintf` `fopen` `fread` `fwrite` `fclose` | 格式化输出与文件读写 |
| `<stdint.h>` | `int8_t` `int32_t` `uint64_t` `intptr_t` | 固定宽度整数 |
| `<stdbool.h>` | `bool` `true` `false` | 布尔类型 |
| `<math.h>` | `floor` `ceil` `round` `fabs` `sqrt` `pow` | 数学计算 |

几个容易踩坑的细节：

- `memcpy` 不处理内存重叠，源和目标重叠时用 `memmove`。
- `snprintf` 返回“本应写入的长度”，据此判断是否被截断。

```c
char buffer[16];
int written = snprintf(buffer, sizeof(buffer), "%s-%d", "habit", 42);
if (written < 0 || (size_t)written >= sizeof(buffer)) {
  // 发生截断或编码错误
}
```

- `strtol` 用 `endptr` 判断是否完整解析，`atoi` 无法区分错误与 0。

```c
char *end = NULL;
long value = strtol("42abc", &end, 10);
if (end == NULL || *end != '\0') {
  // 没有完整解析，end 指向第一个非法字符
}
```

- `qsort` 的比较器返回负数、0、正数，分别表示小于、等于、大于。

## 11. C 与 Objective-C 的边界

接手老工程时，C 代码与 OC 代码经常贴在一起，下面几组概念要能分清：

**字符串转换**

```objc
NSString *text = @"阅读";
const char *utf8 = [text UTF8String];   // 指针生命周期跟随 text，不要长期保存
NSString *back = [NSString stringWithUTF8String:"阅读"];
```

`UTF8String` 返回的内部缓冲区由 `NSString` 管理，原字符串释放后指针失效。需要保留时用 `strdup` 复制，用完 `free`。

**空值**

| 写法 | 含义 |
|---|---|
| `NULL` | C 空指针 |
| `nil` | OC 对象空指针 |
| `Nil` | OC 类空指针 |
| `NSNull` | 集合里占位的“空对象” |

`nil` 可以安全地接收消息，`NSNull` 是一个真实对象，向它发未实现的消息会崩溃。集合里取出的 `NSNull` 要先判断再转换。

**`id` 与 `void *`**

```objc
id object = [[NSObject alloc] init];
void *raw = (__bridge void *)object;   // 不转移所有权
id back = (__bridge id)raw;
```

`__bridge` 只做类型转换；Core Foundation 对象转 OC 并移交所有权用 `__bridge_transfer`，反向用 `__bridge_retained`。ARC 下混用 C 指针与 OC 对象时，桥接关键字写错会造成过度释放或泄漏。

**`BOOL` 与 `bool`**

`BOOL` 是 OC 的布尔类型，取值 `YES` / `NO`；`bool` 来自 `<stdbool.h>`。判断时统一写 `if (flag)`，不要写 `if (flag == YES)`，因为老代码里 `BOOL` 可能是 `signed char`，非 0 即真。

**结构体进集合**

`CGRect`、`NSRange` 不是对象，不能直接放进 `NSArray` / `NSDictionary`，需要装箱：

```objc
NSValue *value = [NSValue valueWithBytes:&rect objCType:@encode(CGRect)];
CGRect decoded;
[value getValue:&decoded];
```

## 12. 常见崩溃对照表

| 现象 | C 视角的常见原因 |
|---|---|
| `EXC_BAD_ACCESS` | 野指针、已释放对象、`NULL` 解引用 |
| 随机数据错乱 | 数组越界写坏相邻内存 |
| `strlen` / `%s` 崩溃 | 字符串缺少 `'\0'` 或缓冲区未初始化 |
| 尺寸计算异常 | 无符号回绕、`int` 溢出后转 `size_t` |
| 栈溢出 | 递归过深、函数内大数组 |
| 堆破坏（`malloc` 报错） | 越界写、重复 `free`、释放非堆指针 |

排查时先用 Xcode 的 Address Sanitizer（Scheme -> Diagnostics）复现，再回到指针与下标的计算上找越界。

## 13. 练习清单

在命令行工程里完成，每题都要求能编译、能解释结果：

1. 用指针交换两个整数。
2. 自己实现 `strlen`，不调用标准库。
3. 用 `malloc` / `realloc` / `free` 实现一个可扩容的整数数组。
4. 用 `qsort` 与函数指针对结构体数组按字段排序。
5. 用位掩码实现三个开关的组合、判断与清除。
6. 写一个返回 `bool`、通过指针输出结果的函数，模拟 `NSError **` 模式。
7. 故意制造一次数组越界与一次悬空指针，用 Address Sanitizer 观察报错。
8. 用 `strtol` 解析字符串并判断是否完整，用 `snprintf` 安全拼接。
9. 把结构体指针传入函数，用 `->` 修改成员。
10. 写一个 `.h` + `.c` 的小模块，用 `static` 隐藏内部函数，并在 `.mm` 里通过 `extern "C"` 调用。

## 14. 验收

- 能解释 `&`、`*`、`NULL` 与指针类型的关系。
- 能用 `->` 操作结构体指针，区分指针数组与数组指针。
- 能读写 `char *` 字符串，知道 `'\0'` 的作用。
- 能看懂 `CGRect`、`NSRange` 这类结构体的定义与传值方式。
- 能识别野指针、数组越界、未终止字符串与无符号回绕。
- 能说明 `static` / `extern` 的作用，并让 C 头文件在 `.mm` 里正确链接。
- 能区分 `NULL` / `nil` / `NSNull`，安全完成 `NSString` 与 `const char *` 转换。
- 能用函数指针写回调，并理解它与 block 的对应关系。
- 能说清栈与堆的区别，以及 OC 对象为什么需要引用计数。

完成后回到 [阶段零](./stage-0-foundations.md#0-预备知识)，继续类与对象、属性与 ARC。
