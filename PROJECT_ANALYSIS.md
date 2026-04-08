# Gemini Voyager 项目功能模块分析报告

## 1. 内容脚本 (`src/pages/content/`)

| 模块 | 功能 |
|------|------|
| `folder` | 文件夹增删移动、排序 |
| `export` | 导出图片/Markdown/PDF |
| `deepResearch` | 深度研究报告导出 |
| `draftSave` | 输入框草稿自动保存/恢复 |
| `timeline` | 时间线预览面板 |
| `prompt` | Prompt 管理器 |
| `watermarkRemover` | 去除水印 |
| `gemsHider`/`recentsHider` | 侧边栏隐藏 |
| `visualEffects` | 雪花/樱花/雨特效 |

**实现思路**：MutationObserver 监听 DOM + `chrome.storage.sync` 响应式配置 + `beforeunload` 清理资源

---

## 2. Features 功能模块 (`src/features/`)

| 模块 | 功能 | 实现方式 |
|------|------|---------|
| `backup` | 本地文件备份 | File System Access API，纯数据层 |
| `export` | 对话导出 | Strategy 模式，DOM 提取内容 |
| `contextSync` | 上下文同步 IDE | `chrome.runtime.sendMessage`，端口 3030 |
| `formulaCopy` | 公式复制 | DOM click 事件委托，识别 `data-math` |

**架构边界**：Content Scripts 是**启动器**（判断页面环境、分发任务），Features 是**执行器**（具体业务逻辑）

---

## 3. 核心服务 (`src/core/services/`)

| Service | 职责 |
|---------|------|
| `StorageService` | 三层存储抽象（sync/local/localStorage） |
| `GoogleDriveSyncService` | Drive OAuth2 双向同步 |
| `AccountIsolationService` | 多账户数据隔离 |
| `DataBackupService` | 多层备份 + 7天过期 |
| `KeyboardShortcutService` | 键盘快捷键管理 |
| `StorageMonitor` | 存储配额监控（80%/90%/95%） |

**依赖链**：`LoggerService` → `StorageService` → `GoogleDriveSyncService` / `AccountIsolationService`

---

## 4. Popup UI (`src/pages/popup/`)

- **自包含大组件**（`Popup.tsx` ~1800行）
- **直接调用** `chrome.storage.sync`（未使用 StorageService 封装）
- **通信模式**：
  - Popup ↔ Background：`chrome.runtime.sendMessage`
  - Popup ↔ Content Script：`chrome.tabs.sendMessage`（实时查询）

---

## 整体架构图

```
┌─────────────────────────────────────────────────────────┐
│                   Content Script                         │
│              (index.tsx 入口 + 调度器)                   │
│  ┌─────────────────────────────────────────────────────┐│
│  │ Content Scripts 子模块 (30+)                         ││
│  │ draftSave / folder / export / timeline / ...       ││
│  └─────────────────────────────────────────────────────┘│
│                           ↕ chrome.storage.sync          │
└─────────────────────────────────────────────────────────┘
                              ↕ chrome.runtime.sendMessage
┌─────────────────────────────────────────────────────────┐
│  Popup (UI 组件)    │    Background Script             │
│  StorageService     │    GoogleDriveSyncService         │
│  直接调 chrome.* API │    消息路由                       │
└─────────────────────────────────────────────────────────┘
                              ↕ chrome.identity (OAuth2)
                         Google Drive
```
