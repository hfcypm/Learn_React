# Perfetto 版本边界与迁移

**目标**：让学习内容、示例命令和生产排查拥有清晰的版本边界，避免把旧 systrace HTML 流程和当前 Perfetto UI 混用。

## 1. 版本差异

| 能力 | 旧写法 / 旧环境 | 当前主线 |
|---|---|---|
| 采集前端 | `systrace.py` 出 HTML | Perfetto `record_android_trace` / Studio System Trace |
| 分析 UI | 浏览器打开 systrace HTML | [ui.perfetto.dev](https://ui.perfetto.dev/) |
| 帧时间线 | 靠 `doFrame` 手工对齐 vsync | `android.surfaceflinger.frametimeline` |
| Janky frames 提示 | 依赖 gfxinfo 计数 | Studio Chipmunk+ 且 Android 12+ |
| atrace 名字长度 | 旧内核常截断到 16 字符 | Perfetto 宽松，仍建议短而稳 |
| 内存 | 独立 hprof / Allocation Tracker | 可选 heapprofd / java_hprof 进同一 trace |
| AI 辅助 | 无 | Android skill：`perfetto-trace-analysis` / `perfetto-sql` |

## 2. 每个示例必须记录的元数据

```markdown
> 适用范围：Android 12+，Perfetto UI 当前稳定版（ui.perfetto.dev）
> 需要的工具：adb、Android Studio、Chrome、python3
> 验证命令：adb devices；能打开 .perfetto-trace
> 包名：com.tzt.android（替换为实际应用）
> 最后复核日期：YYYY-MM-DD
```

## 3. 系统版本要点

- Android 9 及以下：可录 Perfetto，FrameTimeline 与部分数据源不完整。
- Android 10+：heapprofd。
- Android 12+：FrameTimeline、Studio Janky frames 作为学习主线。
- 刷新率 90/120Hz 时帧预算按 `1000/Hz` 计算，SQL 阈值跟着改。

应用侧：

- debug 包默认可采集。
- release 要在 Manifest 声明 `android:profileable="true"` 才能做部分分析。
- 未 profileable 时仍可能有系统级 sched，但应用 atrace / 堆采样会缺。

## 4. 迁移要点

- 旧 HTML systrace 用 Perfetto UI 打开通常无意义，重新用 System Trace 录一份。
- bugreport.zip 可直接丢进 Perfetto UI，不必先手工拆文件。
- Studio 导出与 `record_android_trace` 产出都是 protobuf，可互开。
- APA / 新版 Performance Analyzer 打开同一格式，分析步骤仍按本路线的红帧主线。

## 5. 升级前检查

1. 阅读 [Perfetto 文档](https://perfetto.dev/docs/) 对应数据源页。
2. 确认设备 Android 版本与 profileable。
3. 确认 config 含 `android.surfaceflinger.frametimeline`。
4. 确认 `-a` 包名与进程列表里的 `process.name` 一致（注意 `:push` 等多进程）。
5. 大文件先缩短时长，再升级缓冲。

## 6. 升级后检查

- 目标进程出现在 UI 进程列表。
- 主线程能搜到 `Choreographer#doFrame`。
- Expected / Actual Timeline 可见。
- SQL `slice` 查询返回业务切片。
- 自定义 `Trace.beginSection` 名字可搜索。
