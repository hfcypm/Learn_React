# Perfetto 新手导读

## 适合谁

这套路线适合已经能独立编译 Android 应用、正在排查卡顿 / ANR / 掉帧的客户端工程师。学习重点是：录一份能复现问题的 System Trace，在 Perfetto UI 里把红帧拆到主线程 slice，再用 SQL 与自定义埋点把根因钉死。

证券行情类 App（多层自定义 View + Fragment + ViewPager、主线程动画、行情推送）是本路线的默认场景；其它 Android 应用同样适用。

## 开始前准备

- 一台 Android 12+ 真机或模拟器，已开 USB 调试。Android 12 起 FrameTimeline 与 Studio Janky frames 更完整。
- 本机已装 Android Studio（Chipmunk / 2021.2.1 及以上；查 jank 建议更新到较新稳定版）。
- 能用 `adb devices` 看到设备。
- 目标 App 是 debuggable 或已声明 `android:profileable="true"`，否则部分内存 / Java 方法采样不可用。

```bash
adb devices
adb shell getprop ro.build.version.release
```

## 核心观念

Perfetto 记录的是「时间线上发生了什么」，不是「哪一行 Java 源码」。

- System Trace：CPU 调度、atrace 切片、帧时间线、系统服务。
- Studio CPU Profiler 的 Sample / Trace Java Methods：把 slice 对到 Java 方法。
- Memory Profiler / LeakCanary：跳源码看引用链。

卡顿先用 Perfetto 定位「哪一帧、哪条线程、哪个 slice 超时」；需要对到方法名时再开 Java 采样。

## 学习顺序

| 阶段 | 先回答的问题 | 阶段产出 |
| --- | --- | --- |
| 零 | Perfetto 解决什么，和 Profiler 如何分工？ | 一张工具分工表 |
| 一 | 如何录 10–20 秒并在 UI 里找到自己的进程？ | 一份可打开的 `.perfetto-trace` |
| 二 | 如何确认红帧，并拆开 `doFrame` / `draw`？ | 一帧的根因草图 |
| 三 | 如何用 SQL 量化，如何解释 Sleeping 与动画长切片？ | 可复现的查询与埋点 |
| 四 | 内存该不该用 Perfetto？APA 能做什么？ | 团队可复用的排查清单 |

## 每阶段学习方法

1. 每个概念都对着一份真实 trace 操作，不要只读截图。
2. 录制窗口固定 10–20 秒，只做一次可复现操作（例如打开个股详情再滑 F10）。
3. 先看 FrameTimeline 颜色，再点进主线程，最后才看 RenderThread / Binder / 系统进程。
4. 主线程出现大段 Sleeping 时，点该段看 waker，不要先猜 GPU。
5. 动画、自定义 `onDraw`、行情回调里打 `Trace.beginSection`，下一次录制就能对上业务名。

## 常见排错顺序

```text
设备连不上 / 录不到
  -> 进程轨道找不到（包名、profileable、atrace 未包含本应用）
  -> 没有 FrameTimeline（系统版本、未开 gfx/view、录制时长太短）
  -> 红帧很多但 doFrame 很短（GPU、SF、vsync 错过）
  -> 主线程 Sleeping 很长（等锁、Binder、RenderThread、休眠）
  -> 有 animation 长切片但对不上代码（缺 Java 采样或业务埋点）
  -> 内存结论含糊（误用 Perfetto hprof，应改 Studio / LeakCanary）
```

## 阶段项目路线

从 [综合实战：个股详情页卡顿排查](./project-practice.md) 开始，每完成一个阶段就加深同一份问题：先录到，再读帧，再定量，最后形成团队规范。

## 完成标准

- 能独立录制并打开 System Trace。
- 能指出一帧为何变红，并说出是 App 侧还是 SurfaceFlinger 侧。
- 能把超时 `doFrame` 拆成 `traversal` / `draw` / animation / 调度等待。
- 能写至少两条 Trace Processor SQL。
- 能说明 Java 泄漏为什么主用 Studio + LeakCanary。
- 完成 [学习评估](./learning-assessment.md) 中的阶段考核与项目评分。
