# Perfetto 学习路线图（入门 → 进阶）

**目标**：掌握 Android 系统级 tracing 的录制、阅读与根因定位，能用 Perfetto UI 和 Trace Processor 把卡顿、调度、主线程阻塞和内存问题落到具体线程、slice 与代码埋点。

## 先看这里

- [新手导读](./start-here.md)：前置条件、学习顺序、排错顺序和完成标准。
- [综合实战：个股详情页卡顿排查](./project-practice.md)：按阶段逐步把同一份 trace 查透。
- [学习评估与项目评分](./learning-assessment.md)：阶段考核、排错题、项目评分和评审问题。

## 学习阶段总览

| 阶段 | 名称 | 目标 |
|---|---|---|
| 零 | Perfetto 基础与工具链 | 理解 trace 是什么，分清 System Trace、systrace、Profiler 的职责 |
| 一 | 录制与 Perfetto UI | 用 Android Studio / 命令行录 10–20 秒，在 ui.perfetto.dev 打开并导航 |
| 二 | 卡顿与 FrameTimeline | 用 Expected / Actual Timeline 判定红帧，拆 `doFrame` / `draw` / RenderThread |
| 三 | SQL、调度与自定义埋点 | 用 Trace Processor SQL 量化 jank，对上 CPU 调度、锁与 `Trace.beginSection` |
| 四 | 内存、APA 与生产流程 | 理解 heapprofd / java_hprof 边界，建立可重复的排查与交付清单 |

## 快速导航

- [阶段零：Perfetto 基础与工具链](./stage-0-foundations.md)
- [阶段一：录制与 Perfetto UI](./stage-1-record-and-ui.md)
- [阶段二：卡顿与 FrameTimeline](./stage-2-jank-and-frametimeline.md)
- [阶段三：SQL、调度与自定义埋点](./stage-3-sql-and-cpu.md)
- [阶段四：内存、APA 与生产流程](./stage-4-memory-and-production.md)
- [综合实战：个股详情页卡顿排查](./project-practice.md)

## 学习原则

- 先录一份能复现问题的短 trace，再打开 UI；空谈概念解决不了红帧。
- 卡顿主线永远是：红帧 -> Actual Timeline -> `Choreographer#doFrame` -> 主线程 slice -> 调度 / 锁 / 回调。
- 60Hz 帧预算约 `1000/60≈16.67ms`；Actual Timeline 是真正交帧时间（CPU post 与 GPU 较晚者）。
- Perfetto 擅长系统时间线；要跳 Java 源码用 Android Studio CPU / Memory Profiler。
- 自定义 View 的业务埋点用 `Trace.beginSection` / `endSection`，把「动画 233ms」对到具体 listener。
- 同一份 `.perfetto-trace` 可在 Studio、ui.perfetto.dev、APA 之间互开。

## 阶段成果

- 能解释 Perfetto、atrace、ftrace、FrameTimeline 各自提供什么信息。
- 能用 Android Studio System Trace 或 `record_android_trace` 录到可分析文件。
- 能在 Perfetto UI 找到进程、主线程、RenderThread 和 FrameTimeline 轨道。
- 能判定一帧是 App jank、SurfaceFlinger jank 还是丢帧。
- 能写 SQL 列出超时 `doFrame`，并能解释主线程 Sleeping 的 waker。
- 能判断内存问题该走 LeakCanary / Studio hprof，还是 Perfetto heapprofd。

## 每阶段固定学习模板

每个阶段按照「目标 -> 前置知识 -> 核心问题 -> 最小示例 -> 项目增量 -> 验收 -> 排错」执行。先完成阶段项目，再阅读扩展主题；这样能够把 Perfetto 概念连接到一份真实 trace。

## 版本边界

学习主线面向 Android 12+ 设备上的 System Tracing 与当前 Perfetto UI（ui.perfetto.dev）。旧版 systrace HTML、Android 11 及以下 FrameTimeline 完整度差异参见 [版本边界与迁移](./version-governance.md)。

## 前置知识

这是 Android 客户端性能路线，和仓库里的 Web 主线并行。建议已能独立编译并运行一个 Android 应用，熟悉主线程、Choreographer、自定义 View 的 `onDraw` / `onMeasure`。若你同时做 Web / 桌面，可对照 [React 学习路线](../react-learning-path/README.md) 理解「一帧要在预算内画完」这一共同约束。

## 官方资源

- [Perfetto 官方文档](https://perfetto.dev/docs/)
- [Perfetto UI](https://ui.perfetto.dev/)
- [System Tracing 入门](https://perfetto.dev/docs/getting-started/system-tracing)
- [FrameTimeline 数据源](https://perfetto.dev/docs/data-sources/frametimeline)
- [Trace Processor SQL](https://perfetto.dev/docs/analysis/trace-processor)
- [Android 官方：使用 Perfetto 捕获跟踪记录](https://developer.android.com/topic/performance/tracing)
