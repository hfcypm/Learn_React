# 阶段四：内存、APA 与生产流程

**目标**：分清 Perfetto 在内存问题上的边界，了解 Android GPU Inspector / APA 与 Perfetto 的关系，把排查沉淀成可重复的团队流程。

## 1. 内存：先选对工具

| 症状 | 首选 | Perfetto 是否主路径 |
|---|---|---|
| Activity / View / Listener 泄漏，要跳源码 | LeakCanary + Studio Memory Profiler | 否 |
| Native `malloc` 持续涨 | heapprofd | 是 |
| 看 GC、堆大小随时间变化 | Studio Memory / `proc_stats` | 辅助 |
| 怀疑图形缓冲 / Surface 占用 | 系统内存轨道 + dumpsys | 辅助 |

Java 泄漏的核心产物是 hprof 引用链。Perfetto 可以在同一 config 里触发 `java_hprof` dump，但 UI 不会像 Studio 那样点进 `MinuteView.java:120`。日常 Java 泄漏不要把 Perfetto 当主查工具。

## 2. heapprofd（Native 堆）

Android 10+ 的 out-of-process native heap profiler。对 `malloc` 采样，异步 unwind，把调用栈写进同一份 trace，便于和卡顿时间线对齐。

开销明显高于普通 System Trace。只在已经怀疑 native 分配（解码、行情 native SDK、bitmap）时打开。

```sql
INCLUDE PERFETTO MODULE android.memory.heap_profile.summary_tree;

SELECT
  name,
  mapping_name,
  self_size,
  cumulative_size
FROM android_heap_profile_summary_tree
ORDER BY cumulative_size DESC
LIMIT 30;
```

目标进程必须是 profileable 或 debuggable。

## 3. `java_hprof` 与 Studio 的关系

Perfetto config 可请求 Java 堆转储并塞进 trace。用途：

- 和同一时间窗的 CPU / 帧数据对齐：「OOM 前 2 秒主线程在干什么」。
- 系统服务、多进程一次性抓。

要回答「谁引用了这个 Fragment」，用 Studio Capture Heap Dump 或 LeakCanary。

## 4. APA（Android Performance Analyzer）

APA 是 2026 左右推出的官方性能分析桌面工具（独立安装，或挂在较新的 Android Studio Panda 线）。它能打开同一份 `.perfetto-trace`。

能力边界：

- 可以录、可以打开 Perfetto 格式。
- 没有「一键 AI 扫描全片红帧并给出修复补丁」的按钮。
- 和 AI 的衔接走 Android 的 skill：`android skills add perfetto-trace-analysis` / `perfetto-sql`，由 Agent 调 Trace Processor，不是 APA 窗口里的魔法扫描。

股票卡顿主因多在 CPU / 主线程。APA 若强调 GPU / Vulkan 计数器，对行情详情页优先级低于 FrameTimeline。

## 5. 其它周边工具

| 工具 | 用途 |
|---|---|
| Android GPU Inspector (AGI) | GPU 帧调试，System Trace 不够用时 |
| Macrobenchmark / JankStats | 自动化量 jank，CI 回归 |
| `dumpsys gfxinfo` | 粗看 janky frames 计数 |
| Journeys（Studio Labs） | 预览级用户旅程测试，不适合当生产 CI |
| `android` CLI + Perfetto skill | 给 Agent 封装 adb/sdk；`android run` 不构建 |

生产回归用 Macrobenchmark 录 Perfetto，比纯手工点 UI 稳定。

## 6. 生产级排查清单

把下面清单当成每次卡顿单的必填项。

1. 复现步骤、机型、刷新率、系统版本、App 版本、debuggable 与否。
2. 录 10–20 秒 System Trace，文件名含日期与场景：`20260921-quote-detail-f10.perfetto-trace`。
3. 在 ui.perfetto.dev 标注红帧时间窗。
4. 填写阶段二的「帧记录」至少 3 条。
5. SQL 导出 `doFrame` Top N。
6. 若根因是动画 / 自定义 View：加埋点后复录，对比 dur。
7. 若根因疑似泄漏：另开 LeakCanary / Memory Profiler，不混在同一结论里。
8. 修复后用同一操作再录一份，红帧数与 p99 `doFrame` 对比入单。

## 7. 与业务架构的交界

行情详情多层自定义 View + Fragment + ViewPager 时，性能结论要写回生命周期约定：

- 主推 / WebView 归当前 Fragment 的 ViewModel；`RESUMED` 订阅，`onPause` 退订。
- 子 View 只 `bind(Quote)`，不自己持有长连接。
- 控件资源跟 attach/detach 或 `viewLifecycleOwner`，不在自定义 View 里层层手写 `release()`。
- F10 滑到底再 `new WebView`，避免离屏页提前创建。

这些约定减少「邻页仍在动画」造成的红帧。

## 8. 动手任务

1. 写一份本团队的「卡顿提单模板」（含文件命名与帧记录）。
2. 用 LeakCanary 或 Memory Profiler 做一次对照，书面说明它和 Perfetto 的分工。
3. 若有 APA / 新版 Studio，用同一份 trace 打开，确认轨道一致。

## 阶段四验收

- 能正确拒绝「用 Perfetto 当 Java 泄漏主工具」这种路径。
- 能说明 heapprofd 的适用场景与开销。
- 能独立走完从复现、录制、拆帧、复录对比的闭环。
- 能向同事讲清 APA 与 Perfetto UI 的关系。

## 常见排错

| 现象 | 排查方向 |
|---|---|
| heapprofd 无数据 | 非 profileable；未采样到 malloc；进程名过滤错误 |
| hprof 很大、UI 卡死 | 不要把全量 Java dump 当日常卡顿附件 |
| APA 里找不到红帧 | 打开的不是同一文件，或未加载 FrameTimeline 数据源 |
| 修复后仍红 | 刷新率变了；邻页仍 attach；对比的是不同操作路径 |

## 学完之后

回到 [综合实战：个股详情页卡顿排查](./project-practice.md) 按阶段把同一场景做完，并用 [学习评估](./learning-assessment.md) 打分。
