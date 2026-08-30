# 阶段四：打包发布与生产

**目标**：掌握打包配置、代码签名、更新机制、移动端支持与性能优化，交付可分发、可升级的应用。

## 1. 打包基础

```bash
npm run tauri build
```

`tauri build` 先构建前端产物，再用 `cargo build --release` 编译 Rust，最后按 `bundle.targets` 生成安装包。产物在 `src-tauri/target/release/bundle/`。

## 2. bundle 配置

```json
{
  "bundle": {
    "active": true,
    "targets": ["deb", "appimage", "nsis", "dmg"],
    "icon": [
      "icons/32x32.png",
      "icons/128x128.png",
      "icons/128x128@2x.png",
      "icons/icon.icns",
      "icons/icon.ico"
    ],
    "category": "Productivity",
    "shortDescription": "跨平台文件管理器",
    "createUpdaterArtifacts": true
  }
}
```

- `icon` 按平台提供对应格式。
- `createUpdaterArtifacts` 生成更新签名所需产物。

## 3. 代码签名

macOS 与 Windows 需要签名才能无警告分发：

```json
{
  "bundle": {
    "macOS": {
      "signingIdentity": "-",
      "providerShortName": "your-team-id"
    },
    "windows": {
      "certificateThumbprint": "THUMBPRINT",
      "digestAlgorithm": "sha256"
    }
  }
}
```

- macOS 用 Developer ID 证书 + 公证（notarization）。
- Windows 用代码签名证书，凭据经 CI Secret 管理，不进仓库。

## 4. 更新机制

Tauri 2 提供基于静态服务器的更新机制，支持签名校验：

```json
"plugins": {
  "updater": {
    "pubkey": "你的公钥",
    "endpoints": ["https://example.com/updates/{{target}}/{{arch}}/{{current_version}}"]
  }
}
```

Rust 端注册：

```rust
.plugin(
    tauri_plugin_updater::Builder::new()
        .build()
)
```

前端检查更新：

```ts
import { check } from "@tauri-apps/plugin-updater";
import { relaunch } from "@tauri-apps/plugin-process";

const update = await check();
if (update) {
  await update.downloadAndInstall();
  await relaunch();
}
```

- 更新包签名，公钥校验防篡改。
- 发布新版本时生成签名产物并上传到 endpoints。

## 5. 移动端支持

Tauri 2 支持 iOS 与 Android。需要时用 `tauri mobile init` 初始化移动工程，Rust 代码与命令可复用，前端保持一致。

## 6. 性能优化

- 体积：Tauri 二进制已很小，控制前端产物与依赖。
- 启动：前端做代码分割与懒加载。
- 长任务：用异步命令 + 事件反馈进度。
- 内存：避免渲染进程持大对象，及时释放事件监听。

## 7. 发布检查清单

- [ ] 三平台产物打包成功并可安装。
- [ ] 签名与公证通过。
- [ ] 更新签名产物生成且公钥配置正确。
- [ ] capabilities 只含必要权限。
- [ ] CSP 已配置。
- [ ] 关键路径有错误处理与日志。
- [ ] 版本号递增，更新端点可访问。

## 8. 动手任务

1. 配置 bundle 图标与目标，打出当前平台安装包。
2. 配置更新公钥与端点，本地验证更新流程。
3. 审查 capabilities，移除未使用的权限。
4. 设置 CSP 并验证功能正常。
5. 编写发布检查清单并逐项确认。

## 阶段四验收

- 能打包出跨平台产物。
- 能配置签名与更新机制。
- 能按最小权限与安全基线审查应用。
- 能说明移动端支持与性能优化要点。

## 常见排错

| 现象 | 排查方向 |
|---|---|
| 打包缺少图标 | 按平台提供全部图标格式 |
| 更新失败 | 检查签名公钥、端点 URL 与签名产物 |
| 签名失败 | 证书、Thumbprint 与 CI 环境 |
| Linux 依赖缺失 | 安装打包所需系统库 |
| 权限被拒 | capabilities 未包含所需插件权限 |

## 进入下一阶段的条件

你能够打包、签名并接入更新机制。此时进入 [综合实战：跨平台文件管理器](./project-practice.md) 串起全流程。
