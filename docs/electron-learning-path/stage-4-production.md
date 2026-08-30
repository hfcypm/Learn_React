# 阶段四：打包发布与生产

**目标**：掌握应用打包、代码签名、自动更新、安全基线与应用性能，交付可分发、可升级的桌面应用。

## 1. 打包工具选择

| 工具 | 特点 |
|---|---|
| electron-builder | 跨平台打包，配置丰富，内置自动更新 |
| Electron Forge | 官方推荐，集成打包与插件 |
| electron-vite + builder | 与前端构建一体化 |

本路线以 electron-builder 为主线。

## 2. electron-builder 配置

```json
{
  "build": {
    "appId": "com.example.mynotes",
    "productName": "MyNotes",
    "directories": {
      "output": "dist"
    },
    "files": [
      "dist/**",
      "main.js",
      "preload.js"
    ],
    "win": {
      "target": "nsis"
    },
    "mac": {
      "target": "dmg",
      "category": "public.app-category.productivity"
    },
    "linux": {
      "target": "AppImage",
      "category": "Utility"
    }
  },
  "scripts": {
    "build": "vite build",
    "dist": "npm run build && electron-builder"
  }
}
```

```bash
npm run dist
```

产物输出到 `dist/`，生成对应平台的安装包。跨平台打包建议在各目标平台运行，或用 CI 矩阵。

## 3. 应用图标与元数据

- `build/icon.png`（512x512 起）作为基础图标。
- `appId` 与应用身份关联，自动更新与签名都需要。
- 各平台图标规格由 builder 自动生成。

## 4. 代码签名

macOS 与 Windows 要求签名才能无警告分发与更新：

```json
"mac": {
  "identity": "Developer ID Application: ..."
},
"win": {
  "certificateFile": "win-cert.pfx",
  "certificatePassword": "change-me"
}
```

- macOS 用 Apple Developer 证书，签名后公证（notarize）。
- Windows 用代码签名证书。
- 签名证书属于敏感凭据，用 CI 的 Secret 管理，不进仓库。

## 5. 自动更新

electron-builder 提供自动更新能力：

```js
// main.js
const { autoUpdater } = require("electron-updater");

autoUpdater.on("update-downloaded", () => {
  autoUpdater.quitAndInstall();
});

app.whenReady().then(() => {
  autoUpdater.checkForUpdates();
});
```

- 更新源指向发布服务器（GitHub Releases、自建静态服务器等）。
- 更新包与 latest.yml 一起发布。
- 强制更新时先提示用户再安装。

## 6. 安全基线复查

- `contextIsolation: true`、`sandbox: true`、`nodeIntegration: false`。
- 只通过 contextBridge 暴露窄接口，不整对象暴露 `ipcRenderer`。
- 主进程校验所有 IPC 入参（类型、范围、路径白名单）。
- 不加载不可信的远程内容；必须加载时设 `webSecurity` 与 CSP。
- 升级前审查依赖与 Chromium 版本。

## 7. 性能与体验

- 用 `win.webContents.setBackgroundThrottling(false)` 控制后台节流。
- 打开窗口后延迟加载非关键模块。
- 使用 `app.setLoginItemSettings` 可选开机自启。
- 崩溃与错误用 `process.on('uncaughtException')` 记录并上报。

## 8. 发布检查清单

- [ ] 打包产物在三个平台可安装、可运行。
- [ ] 签名与公证通过。
- [ ] 自动更新链路验证（发布新版本后应用能升级）。
- [ ] IPC 通道全部有入参校验。
- [ ] 数据读写路径使用系统目录。
- [ ] 生产环境关闭 DevTools 与调试输出。
- [ ] 版本号一致递增，更新源与安装包同步。

## 9. 动手任务

1. 配置 electron-builder，打出当前平台的安装包。
2. 配置应用图标与 `appId`。
3. 接入自动更新并本地验证更新流程。
4. 复查安全基线，确认无 `nodeIntegration` 与全局 `ipcRenderer` 暴露。
5. 写一份发布检查清单并逐项确认。

## 阶段四验收

- 能打包出可安装产物。
- 能配置签名与自动更新。
- 能按安全基线审查应用。
- 能执行完整的发布流程。

## 常见排错

| 现象 | 排查方向 |
|---|---|
| 打包缺少文件 | 检查 `files` 白名单与构建输出 |
| 安装包无法启动 | 确认入口文件与资源路径在产物内 |
| 更新不生效 | 检查 latest.yml 与安装包一致性、发布源可达 |
| 签名失败 | 证书路径、密码与密钥链配置 |
| 打包体积大 | 排除多余依赖与调试资源 |
| Windows 防御拦截 | 未签名或证书不被信任 |

## 进入下一阶段的条件

你能够打包、签名并发布可自动更新的应用。此时进入 [综合实战：桌面笔记应用](./project-practice.md) 串起全流程。
