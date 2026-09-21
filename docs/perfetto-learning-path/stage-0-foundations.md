# 阶段零：Perfetto 基础与工具链

**目标**：理解 Perfetto 在 Android 性能工具里的位置，分清 System Trace、atrace、ftrace、Studio Profiler 各自回答什么问题。

## 1. Perfetto 是什么

Perfetto 是 Android 与 Chrome 共用的 tracing 套件：设备上的 traced 守护进程采集事件，桌面端的 Perfetto UI / Trace Processor 负责可视化与 SQL 查询。

| 组件 | 职责 |
|---|---|
| traced / traced_probes | 设备上采集 ftrace、atrace、统计信息 |
| 数据源 | FrameTimeline、heapprofd、process_stats、logcat 等 |
| `.perfetto-trace` | protobuf 二进制时间线，可互开 |
| Perfetto UI | 时间线、切片、SQL、火焰图 |
| Trace Processor | 把 trace 解析成表，供 UI 与命令行 SQL 使用 |

```text
设备(traced 采集)
   -> .perfetto-trace / bugreport.zip
   -> Perfetto UI 或 Trace Processor SQL
```

## 2. 一帧是怎么被画出来的

Android 界面按 vsync 节拍推进。60Hz 时每帧预算约 16.67ms。

```text
vsync
  -> Choreographer#doFrame（主线程）
       -> input / animation / traversal / draw
  -> RenderThread DrawFrame
  -> GPU / SurfaceFlinger 合成
  -> 上屏（Actual Timeline 的终点）
```

- `doFrame`：主线程这一帧的工作总包。
- `draw`：遍历 View 树并记录绘制命令，常出现在主线程。
- RenderThread：真正把 DisplayList 交给 GPU。
- Actual Timeline：这一帧真正完成呈现的时间，取 CPU 提交与 GPU 完成中较晚者。

主线程在 `doFrame` 里超过预算，用户就会感到卡。主线程在 Sleeping，CPU 却被别的线程或锁占着，同样会错过 vsync。

## 3. 工具分工

| 问题 | 首选工具 | Perfetto 角色 |
|---|---|---|
| 哪一帧红了、主线程在干什么 | System Trace + FrameTimeline | 主工具 |
| 这个 slice 对应哪段 Java | Studio Sample / Trace Java Methods | 互补 |
| 对象泄漏、引用链跳源码 | LeakCanary + Memory Profiler | Perfetto hprof 只作系统侧补充 |
| Native 堆分配 | heapprofd | Perfetto 数据源 |
| 方法级插桩、Debug.startMethodTracing | 旧 CPU 记录 | 开销大，日常卡顿不用 |
| GPU / Vulkan 长帧 | GPU 计数器、AGI | 股票类卡顿多数先在 CPU / 主线程 |

Systrace 是旧前端。现在 Android Studio 的 System Trace 底层已经是 Perfetto；导出的文件用 ui.perfetto.dev 打开即可。

## 4. 关键名词

| 名词 | 含义 |
|---|---|
| slice | 时间线上一段有名字的工作，如 `Choreographer#doFrame` |
| track | 一条轨道，通常对应进程、线程或计数器 |
| sched | 内核调度：线程 Running / Runnable / Sleeping |
| atrace | 用户态注解，`Trace.beginSection` 会出现在这里 |
| ftrace | 内核事件，sched_switch、cpu_frequency 等 |
| FrameTimeline | SurfaceFlinger 提供的预期帧 vs 实际帧 |
| waker | 把 Sleeping 线程唤醒的那个线程 / 中断 |

## 5. 最小心智模型：先问三个问题

1. 用户操作发生在哪几秒？（用录制窗口框住）
2. 那几秒里哪些帧是红的？（FrameTimeline Actual）
3. 红帧对应的主线程 `doFrame` 把时间花在哪？（slice 展开）

答完这三问，再决定要不要 Java 采样、锁分析或内存工具。

## 6. 动手任务

1. 打开 [ui.perfetto.dev](https://ui.perfetto.dev/)，用官方示例 trace 或任意 `.perfetto-trace` 熟悉缩放与搜索。
2. 在 Android Studio 里找到 Profiler -> CPU -> System Trace 入口，先不要急着录业务，确认设备能连上。
3. 用自己的话写出：System Trace、Java Methods、Memory Profiler 各解决哪类问题。

## 阶段零验收

- 能解释 Perfetto 采集端与分析端的分工。
- 能默写一帧从 vsync 到上屏的路径。
- 能正确选择卡顿 / 泄漏 / native 堆该用哪套工具。

## 常见排错

| 现象 | 排查方向 |
|---|---|
| 把 systrace HTML 和 Perfetto 当成两套互斥工具 | Studio System Trace 已是 Perfetto |
| 一上来就录 Java Methods | 先确认红帧，再对方法 |
| 认为 Perfetto 能当 IDE 跳源码 | 时间线工具，源码跳转走 Studio |
| 只看 CPU 占用百分比 | 卡顿看帧与主线程 slice，百分比会误导 |

## 进入下一阶段的条件

你能够说明「先录 System Trace 看帧，再决定要不要采样」。此时进入 [阶段一：录制与 Perfetto UI](./stage-1-record-and-ui.md)。
