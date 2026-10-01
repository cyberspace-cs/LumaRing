# LumaRing 中文说明（fork 版）

> 本仓库为 [potatoQi/LumaRing](https://github.com/potatoQi/LumaRing) 的 fork，原项目 66 ⭐ · MIT 许可。
> 这是一个 **macOS 原生"轮盘"快速切换工具**：用一个小圆环切换应用、窗口、浏览器标签，还能运行 App 快捷操作。

---

## 🎯 一句话介绍

**LumaRing = macOS 上的"环形快速切换器"**，比 Cmd+Tab 更直观、比 Mission Control 更轻量：
- 呼出一个圆环 → 鼠标指向 App / 窗口 / 标签 → 松手即切换
- 仅 **2.9 MB**，原生 Swift 编写，启动和切换都很快
- 支持 **应用快捷操作**（每个 App 可配置自己的快捷键动作）
- 支持 **触控板手势**（三指/四指轻点唤出、三指捏合最小化）
- 支持 **浏览器标签切换**（Chrome / Edge）

---

## 🚀 安装

要求 **macOS 14+**，支持 Apple Silicon 和 Intel。

### 方式一：Homebrew

```sh
brew install --cask potatoQi/tap/lumaring
```

### 方式二：GitHub Releases

到 [Releases 页面](https://github.com/cyberspace-cs/LumaRing/releases) 下载最新 DMG，拖入 Applications 即可。

---

## 🖱️ 核心功能

| 功能 | 说明 |
| --- | --- |
| 🔘 **环形切换** | 默认 `Option + Tab` 呼出圆环，鼠标指向目标松手切换 |
| 🖥️ **窗口切换** | 每个 App 可展开窗口弧，悬停显示实时窗口预览 |
| 🌐 **浏览器标签** | 支持 Chrome / Edge 标签页切换、关闭 |
| ⚡ **App 快捷操作** | 每个 App 可录制专属快捷键动作（最多 48 个），环形模式下直接触发 |
| 🚀 **Option 快速启动** | 长按 Option 进入快速启动环，快速打开常用 App |
| ✋ **触控板手势** | 三指/四指轻点唤出；三指捏合最小化当前窗口（可选） |
| 🏷️ **窗口重命名** | 给窗口/标签设置自定义名字和颜色，方便辨认 |
| ❌ **快速关闭** | 悬停窗口/标签上的 × 按钮直接关闭，拖拽 App 图标可退出应用 |

---

## 🏗️ 架构速览

```
┌─────────────────────────────────────────────────┐
│  NSWorkspace 通知 → ApplicationCatalog           │  ← App 列表
│       ↓                                          │
│  RingController → RingView                       │  ← 环形 UI
│       ↓                                          │
│  WindowService（串行队列 + 批量化 AX 读取）        │  ← 窗口查询
│  BrowserTabService（Apple Events 直连 Chromium）  │  ← 浏览器标签
│  PreviewService（ScreenCaptureKit 截图）          │  ← 窗口预览
│  ActionExecutor（CGEvent.postToPid 按键注入）      │  ← App 快捷操作
│  TrackpadInput（MultitouchSupport C 识别器）      │  ← 触控板手势
└─────────────────────────────────────────────────┘
```

### 核心模块（Sources/）

| 模块 | 作用 |
| --- | --- |
| `LumaRing/` | 主应用：环形 UI、设置、控制器、预览、手势 |
| `LumaRingCore/` | 纯逻辑核心：环形几何计算（RingGeometry）、预览位置（PreviewPlacement） |
| `TrackpadInput/` | C 语言触控板手势识别（捏合/轻点，基于系统 MultitouchSupport） |

### 关键设计点

1. **数据路径**：`NSWorkspace 通知 → ApplicationCatalog → RingController → RingView`，单方向数据流
2. **串行队列 + 取消令牌**：窗口查询在单一串行队列上执行，带 180ms 列表超时、1.2s 迭代预算、128 窗口上限，防止卡 UI
3. **所有权与取消**：所有异步结果都带"generation"校验，过期结果无法更新 UI
4. **信任边界**：窗口/标签路径无网络访问；唯一第三方运行时依赖是 Sparkle（更新用）；不记录窗口标题和 URL
5. **可访问性**：两个圆环都复用原生辅助功能元素，屏幕阅读器可用

---

## ⚙️ 快捷键与手势

| 操作 | 默认 |
| --- | --- |
| 呼出环形切换器 | `Option + Tab` |
| 顺时针 / 逆时针选目标 | `Tab` / `Shift + Tab` |
| 主选择 ↔ 次选择 | 反引号键（`~`） |
| 确认选中 | `Return` |
| 模式切换（全局/窗口/操作） | 双击左 Option（环形打开时） |
| 快速启动 | 长按 Option（环形打开时） |
| 触控板轻点唤出（可选） | 三指或四指轻点 |
| 三指捏合最小化（可选） | 向内捏合 |

---

## 📦 为什么值得 fork 学习

- **小而精**：2.9MB 的完整 macOS 效率工具，代码量小、测试覆盖全（Tests/ 有 30+ 个测试文件）
- **原生体验**：SwiftUI + AppKit + ScreenCaptureKit，无 Electron 包袱
- **架构规范**：数据流单向、串行队列、取消令牌、信任边界划分清晰，是 macOS 桌面应用的优秀范例
- **工程化完整**：CI（GitHub Actions）、Homebrew tap、Sparkle 更新、DMG/ZIP 打包、签名验证全都有

---

## 🔗 链接

- 原仓库：https://github.com/potatoQi/LumaRing
- 我们的 fork：https://github.com/cyberspace-cs/LumaRing
- 架构文档：https://github.com/cyberspace-cs/LumaRing/blob/main/docs/ARCHITECTURE.md
- 演示视频：https://github.com/cyberspace-cs/LumaRing/blob/main/docs/assets/demo.mp4
