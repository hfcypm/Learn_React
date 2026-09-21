# 阶段一：录制与 Perfetto UI

**目标**：用 Android Studio 或命令行录 10–20 秒 System Trace，在 Perfetto UI 中找到目标进程、主线程和关键轨道。

## 1. 录制原则

- 时长 10–20 秒。太短抓不到完整操作，太长文件膨胀、UI 卡、环形缓冲会丢掉开头。
- 一次只复现一个动作：打开个股、切横屏、滑 F10、点买卖盘。
- 录之前先热身到目标页，点开始后再操作，避免把冷启动和业务卡顿混在一份里。
- 缓冲建议 32MB 起步；进程多、atrace 全开时再加大。

## 2. 方式 A：Android Studio System Trace（日常首选）

1. 用 USB 连接设备，打开目标 App。
2. View -> Tool Windows -> Profiler，选中进程。
3. CPU -> 录制配置选 **System Trace**（不要先选 Sample Java Methods）。
4. 点 Record，在设备上做一次操作，10–20 秒后 Stop。
5. Studio 时间线里先扫一眼，再点 Open in Perfetto / 导出 `.perfetto-trace`。

Studio Chipmunk+ 且 Android 12+ 时，CPU 记录里可以看到 Janky frames 提示。最终拆帧仍以 ui.perfetto.dev 的 FrameTimeline 为准。

导出路径因 Studio 版本而异，常见是 Profiler 会话的 Save / Export。文件扩展名是 `.perfetto-trace`。

## 3. 方式 B：官方脚本 `record_android_trace`

适合 CI、无法开 Studio、或要固定探针集合时复用。

```bash
curl -O https://raw.githubusercontent.com/google/perfetto/main/tools/record_android_trace

python3 record_android_trace \
    -o quote_detail.perfetto-trace \
    -t 15s \
    -b 32mb \
    -a com.tzt.android \
    sched freq idle am wm gfx view ss input binder_driver
```

- `-t` 时长，`-b` 主缓冲。
- `-a` 限定 atrace 应用；排查自己的包时写包名，避免 `*` 把系统注解挤爆缓冲。
- `sched freq` 看调度与频率；`gfx view` 看绘制与 View；`ss` 是 system_server；`input` 看触摸；`binder_driver` 看跨进程等待。

远程环境若不能自动弹 UI，加 `--no-open`。

## 4. 方式 C：完整 config（进阶预览）

需要 FrameTimeline、进程快照时，用文本 config：

```text
duration_ms: 15000

buffers {
    size_kb: 32768
    fill_policy: DISCARD
}

data_sources {
    config {
        name: "linux.ftrace"
        ftrace_config {
            ftrace_events: "sched/sched_switch"
            ftrace_events: "sched/sched_wakeup"
            ftrace_events: "power/suspend_resume"
            atrace_categories: "gfx"
            atrace_categories: "view"
            atrace_categories: "input"
            atrace_categories: "am"
            atrace_categories: "wm"
            atrace_categories: "dalvik"
            atrace_apps: "com.tzt.android"
        }
    }
}

data_sources {
    config {
        name: "android.surfaceflinger.frametimeline"
    }
}

data_sources {
    config {
        name: "linux.process_stats"
        process_stats_config {
            scan_all_processes_on_start: true
        }
    }
}
```

```bash
python3 record_android_trace -c config.pbtx -o quote_detail.perfetto-trace
```

`android.surfaceflinger.frametimeline` 是红帧轨道的数据源。Studio System Trace 默认会带上；手写 config 时不要漏。

## 5. 打开 UI

1. 浏览器打开 [https://ui.perfetto.dev/](https://ui.perfetto.dev/)。
2. Open trace file，选 `.perfetto-trace`。也可以直接把 bugreport.zip 丢进去，UI 会抽其中的 trace。
3. 左侧进程列表搜索包名，例如 `com.tzt.android`。
4. 展开该进程，置顶以下轨道：

| 轨道 | 用途 |
|---|---|
| Expected Timeline | 系统给这帧预留的时间窗 |
| Actual Timeline | 真正完成呈现的时间 |
| 主线程（常与包名同名或 `...main`） | `doFrame`、`draw`、animation |
| RenderThread | `DrawFrame`、GPU 提交 |
| Binder 线程 | 跨进程 RPC 等待 |

## 6. 导航手势

- `W` / `S`：时间轴放大 / 缩小。
- `A` / `D`：左右平移。
- 选中一段 slice，下方 Current Selection 给出 dur、线程、参数。
- 全局搜索（UI 顶部 Search）输入 `doFrame`、`draw`、`animation`、包名。
- 用时间框选用户操作前后 1–2 秒，避免在 15 秒全长里迷路。

## 7. 确认录制质量

打开后先做体检，再分析业务：

1. 进程在列表中，且主线程有连续 slice。
2. 有 FrameTimeline 两条轨道。没有则检查系统版本与 config。
3. `sched` 存在：主线程应能看到 Running（绿）与 Sleeping。
4. 文件未在开头被 DISCARD 切掉关键操作。若操作发生在录制最开始且缓冲小，可能只剩结尾。

## 8. 动手任务

1. 对目标 App 录一段 15 秒 System Trace，只做一次页面进入。
2. 在 ui.perfetto.dev 打开，找到包名、主线程、RenderThread。
3. 搜索 `Choreographer#doFrame`，点一条看 duration。
4. 把文件存到固定目录，后续阶段反复用同一份。

## 阶段一验收

- 能用 Studio 或脚本独立录到 `.perfetto-trace`。
- 能在 UI 中定位进程与主线程。
- 能说明 `-a` 包名、`gfx/view`、FrameTimeline 数据源各自的作用。

## 常见排错

| 现象 | 排查方向 |
|---|---|
| `adb` 无设备 | USB 调试、授权弹窗、换线 / 换端口 |
| 进程列表没有自己的包 | 录制时进程被杀；`-a` 写错；release 包未 profileable |
| 打开 UI 空白或极卡 | 文件过大，先缩短时长或减小 atrace 范围 |
| 只有系统进程 | 选错进程或 atrace 未包含应用 |
| Studio 能看、网页打不开 | 用最新 Chrome；或让 Studio 直接 Open in Perfetto |

## 进入下一阶段的条件

你手头有一份含目标进程与 `doFrame` 的 trace。此时进入 [阶段二：卡顿与 FrameTimeline](./stage-2-jank-and-frametimeline.md)。
