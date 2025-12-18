# 1. Electron
> 适用版本：Electron ≥ 20  
> 技术栈参考：**vite + vue3 + electron-builder**

---

## 一、preload + IPC

Electron 是**多进程架构**：

```

Main Process   （Node 环境，系统权限）
↑ IPC
Preload        （安全桥接层）
↑ API
Renderer       （浏览器环境，Vue/React）

````

### 核心原因
- Renderer **不安全**（可能被 XSS 注入）
- Main **权限过高**（文件、系统、进程）
- **preload 是唯一安全桥梁**

> preload = 白名单 + 最小权限原则

---

## 二、preload 的核心作用

### 1️⃣ 隔离 Node 与 Renderer

```ts
// ❌ 错误
window.require('fs')
````

```ts
// ✅ 正确（preload）
contextBridge.exposeInMainWorld(...)
```

---

### 2️⃣ 向前端暴露「可控 API」

Renderer 只能使用你暴露的能力：

```ts
window.updater.check()
window.fs.readConfig()
```

而不是：

```ts
fs.readFile('/etc/passwd') // ❌
```

---

### 3️⃣ 统一 IPC 通信规范

* 所有 channel 集中管理
* Renderer 不能随意发消息

### 注意: 
> 1. preload中的windows与渲染层(页面)的windows并不相关
> 2. ipc不能传递 DOM / 自定义Class / electron原生API / 函数function 这几类对象

---

## 三、IPC 通信模型（3 种）

| 类型   | API             | 使用场景      |
| ---- | --------------- | --------- |
| 单向通知 | send / on       | 事件、日志     |
| 请求响应 | invoke / handle | 查询、读写（推荐） |
| 推送订阅 | on + 回调         | 进度、状态     |

---

## 四、推荐项目结构

```
src/
├─ main/
│  ├─ ipc/
│  │   └─ updater.ts
│  └─ index.ts
├─ preload/
│  └─ index.ts
├─ renderer/
│  └─ src/
```

---

## 五、preload 标准写法（模板）

```ts
// src/preload/index.ts
import { contextBridge, ipcRenderer } from 'electron'

contextBridge.exposeInMainWorld('api', {
  checkUpdate: () => ipcRenderer.send('update:check'),

  readConfig: () => ipcRenderer.invoke('config:read'),

  onUpdateProgress: (cb: (p: number) => void) => {
    ipcRenderer.on('update:progress', (_, p) => cb(p))
  }
})
```

**原则**

* preload 不写业务逻辑
* 只做参数校验 + 转发
* 不操作 DOM

---

## 六、Main 进程 IPC 实现

### 1️⃣ 单向通信（send / on）

```ts
ipcMain.on('update:check', () => {
  autoUpdater.checkForUpdates()
})
```

---

### 2️⃣ 请求响应（invoke / handle）✅ 推荐

```ts
ipcMain.handle('config:read', async () => {
  return readConfigFromDisk()
})
```

Renderer：

```ts
const config = await window.api.readConfig()
```

优点：

* Promise 化
* 无回调地狱
* 易维护

---

### 3️⃣ 事件推送（进度）

```ts
autoUpdater.on('download-progress', p => {
  mainWindow.webContents.send('update:progress', p.percent)
})
```

---

## 七、Renderer（Vue3）中使用

### 1️⃣ 类型声明（非常重要）

```ts
// src/types/preload.d.ts
declare global {
  interface Window {
    api: {
      checkUpdate(): void
      readConfig(): Promise<any>
      onUpdateProgress(cb: (p: number) => void): void
    }
  }
}
```

---

### 2️⃣ Vue3 示例

```ts
onMounted(() => {
  window.api.checkUpdate()

  window.api.onUpdateProgress(p => {
    console.log('下载进度', p)
  })
})
```

---

## 八、安全配置（必须）

### ❌ 禁止

```ts
nodeIntegration: true
contextIsolation: false
```

### ✅ 正确

```ts
new BrowserWindow({
  webPreferences: {
    preload,
    contextIsolation: true,
    nodeIntegration: false
  }
})
```

---

## 九、IPC Channel 命名规范

```
模块:动作

update:check
update:progress
config:read
file:select
```

优点：

* 可读性强
* 避免冲突
* 易维护

---

## 十、常见反模式

### ❌ preload 写业务逻辑

> preload 是桥，不是 service

### ❌ Renderer 直接 import electron

> renderer 永远不应接触 Node API

### ❌ channel 随意定义

> 会导致权限失控

---

## 十一、推荐的模块化暴露方式

```ts
contextBridge.exposeInMainWorld('updater', updaterApi)
contextBridge.exposeInMainWorld('fs', fsApi)
```

使用：

```ts
window.updater.check()
window.fs.read()
```

---

# 2. Electron Builder 

## 一. 完整配置示例（package.json）
```jsonc
{
  "name": "my-app",
  "version": "1.0.0",
  "main": "dist/main/index.js",

  // =====================================================
  // electron-builder 配置（位于 package.json -> build）
  // =====================================================
  "build": {
    // =========================
    // 应用基础信息（必填）
    // =========================
    "appId": "com.example.myapp",          // 应用唯一 ID（发布后不要随意修改）
    "productName": "MyApp",                // 应用名称（安装包 / 桌面显示名）

    // =========================
    // 目录配置
    // =========================
    "directories": {
      "output": "release",                 // 打包输出目录
      "buildResources": "build"             // 图标、签名等资源目录
    },

    // =========================
    // 打包文件范围（非常重要）
    // =========================
    "files": [
      "dist/**",                            // vite 构建产物（main / preload / renderer）
      "package.json"                        // 运行时需要
    ],

    // =========================
    // ASAR 打包
    // =========================
    "asar": true,                           // 打包为 asar，提升性能 & 防止源码暴露

    // =========================
    // 不进 asar 的额外资源,二进制文件等,如 exe / dll 等
    // =========================
    "extraResources": [
      {
        "from": "resources/",               // 项目内资源目录
        "to": "resources/",                 // 打包后位置
        "filter": ["**/*"]
      }
    ],
    
    // =========================
    // 应用图标（必填）
    // =========================
    icon: "build/icon",                 // 兜底应用图标（不带扩展名，electron-builder 会自动识别 .ico/.icns/.png）
    
    // =========================
    // 压缩配置
    // =========================
    
    compression: "maximum",               // 最大压缩率，减小安装包体积
    
    // =========================
    // Windows 配置
    // =========================
    "win": {
      "target": "nsis",                     // Windows 安装器
      "icon": "build/icon.ico"
    },

    "nsis": {
      "oneClick": false,                    // false = 可选择安装目录
      "allowToChangeInstallationDirectory": true,
      "perMachine": false,                  // 当前用户安装
      "createDesktopShortcut": true,
      "createStartMenuShortcut": true
    },

    // =========================
    // macOS 配置
    // =========================
    "mac": {
      "target": "dmg",
      "icon": "build/icon.icns",
      "hardenedRuntime": true,
      "entitlements": "build/entitlements.mac.plist",
      "entitlementsInherit": "build/entitlements.mac.plist"
    },

    "dmg": {
      "sign": false
    },

    // =========================
    // Linux 配置
    // =========================
    "linux": {
      "target": ["AppImage"],
      "icon": "build/icon.png",
      "category": "Utility"
    }
  }

  /*
  ======================================================
  说明：
  1. electron-builder 默认会读取 package.json 中的 build 字段
  2. 也可以将以上 build 内容独立成：
     - electron-builder.json
     - electron-builder.yml
  3. 若使用独立配置文件，可从 package.json 中移除 build 字段
  ======================================================
  */
}
```

# 3. macOS 应用签名 & 公证（Notarization）

> 适用场景：  
> - Electron + electron-builder  
> - **配置写在 package.json → build**  
> - 面向 **macOS 正式分发**

---

## 一、项目中涉及的文件位置总览

**推荐目录结构：**

```

project-root/
├─ package.json                  # electron-builder build 配置
├─ build/
│  ├─ icon.icns                  # mac 应用图标
│  ├─ entitlements.mac.plist     # mac 权限描述文件（必须）
│  └─ notarize.js                # 公证脚本（必须）
├─ dist/                          # vite + electron 构建产物
└─ release/                       # electron-builder 输出目录

```

> 📌 **mac 签名 / 公证相关文件统一放在 `build/` 目录**  
> 这是 electron-builder 的官方推荐做法

---

## 二、为什么 mac 必须签名和公证

macOS Gatekeeper 要求：

1. ✅ Code Signing（代码签名）
2. ✅ Notarization（Apple 公证）
3. ✅ Hardened Runtime（强化运行时）

否则用户会看到：
- “应用已损坏，无法打开”
- “无法验证开发者”

---

## 三、证书准备（系统层）

### 1️⃣ Apple Developer 账号
- Apple Developer Program（$99/年）

---

### 2️⃣ 创建证书（一次性）

证书类型选择：

```

Developer ID Application

```

安装后位置：

```

钥匙串访问.app → 登录 → 证书

```

你应该能看到：

```

Developer ID Application: Your Name (TEAMID)

````

---

## 四、package.json → build 中的 mac 配置

> 文件位置：**project-root/package.json**

```jsonc
{
  "build": {
    "mac": {
      "target": "dmg",                           // ← 生成 dmg 安装包
      "icon": "build/icon.icns",                 // ← build/icon.icns, mac 图标
      "hardenedRuntime": true,                   // 开启强化运行时
      "entitlements": "build/entitlements.mac.plist", // ← build/entitlements.mac.plist, 主进程需要
      "entitlementsInherit": "build/entitlements.mac.plist", // ← build/entitlements.mac.plist, 子进程也需要
      "identity": "Developer ID Application"     // ← 证书名称（可选，默认自动选择）
    },

    // 签名完成后执行公证
    "afterSign": "build/notarize.js"
  }
}
````

---

## 五、entitlements.mac.plist（权限文件）

### 文件放置位置

```
build/entitlements.mac.plist
```

### 完整示例（Electron 通用）

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN"
  "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
  <dict>
    <!-- Electron 必需 -->
    <key>com.apple.security.cs.allow-jit</key>
    <true/>

    <key>com.apple.security.cs.allow-unsigned-executable-memory</key>
    <true/>

    <key>com.apple.security.cs.disable-library-validation</key>
    <true/>

    <!-- 用户选择的文件访问 -->
    <key>com.apple.security.files.user-selected.read-write</key>
    <true/>

    <!-- 网络访问 -->
    <key>com.apple.security.network.client</key>
    <true/>
  </dict>
</plist>
```

---

## 六、公证（Notarization）所需配置

### 1️⃣ Apple App 专用密码（系统层）

路径：

```
Apple ID → 登录与安全 → App 专用密码
```

生成后保存（只显示一次）

---

### 2️⃣ 环境变量（本地 / CI）

推荐写入：

```bash
# ~/.zshrc 或 CI 环境变量
export APPLE_ID="your@email.com"
export APPLE_APP_SPECIFIC_PASSWORD="xxxx-xxxx-xxxx-xxxx"
export APPLE_TEAM_ID="ABCDE12345"
```

---

## 七、notarize.js（公证脚本）

### 文件放置位置

```
build/notarize.js
```

### 完整示例

```js
// build/notarize.js
const { notarize } = require('@electron/notarize')

exports.default = async function notarizing(context) {
  const { electronPlatformName, appOutDir } = context
  if (electronPlatformName !== 'darwin') return

  const appName = context.packager.appInfo.productFilename

  await notarize({
    appBundleId: 'com.example.myapp',     // 必须与 appId 一致
    appPath: `${appOutDir}/${appName}.app`,
    appleId: process.env.APPLE_ID,
    appleIdPassword: process.env.APPLE_APP_SPECIFIC_PASSWORD,
    teamId: process.env.APPLE_TEAM_ID
  })
}
```

---

## 八、构建流程（标准）

> 需要在 macOS 环境下才可执行构建/签名/公证：

```bash
pnpm vite build
pnpm electron-builder --mac
```

electron-builder 会自动：

1. 构建 `.app`
2. 使用证书签名
3. 执行 `build/notarize.js`
4. 生成 `.dmg`

---

## 九、验证结果（强烈建议）

### 1️⃣ 验证签名

```bash
codesign --verify --deep --strict release/mac/MyApp.app
```

---

### 2️⃣ 验证公证

```bash
spctl -a -vv release/mac/MyApp.app
```

应显示：

```
accepted
source=Notarized Developer ID
```

---

## 十、常见问题 & 对应文件

| 问题    | 检查文件                   |
| ----- | ---------------------- |
| 已损坏   | notarize.js / 环境变量     |
| 打不开   | entitlements.mac.plist |
| 找不到证书 | 钥匙串访问                  |
| 公证失败  | Apple ID / 专用密码        |
| CI 失败 | 环境变量未注入                |

---
