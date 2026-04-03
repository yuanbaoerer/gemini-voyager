# CLAUDE.md - Gemini Voyager

## 命令

```bash
bun install                # 初始化
bun run dev:chrome         # 开发模式（其他：dev:firefox, dev:safari）
bun run build:chrome       # 构建（其他：build:firefox, build:safari, build:edge, build:all）
bun run test               # 测试（其他：test:watch, test:ui, test:coverage）
bun run typecheck          # 类型检查
bun run lint               # 代码检查
bun run format             # 格式化
bun run bump               # 版本号递增（patch）
bun run docs:dev           # 文档开发服务器
```

## 核心规则

1. **禁止使用 `any` 类型。** 使用 `unknown` + 类型收窄。使用Branded Types处理ID。
2. **UI组件中禁止直接使用 `chrome.storage`。** 使用 `StorageService`。内容脚本（`src/pages/content/`）是例外——它们通过 ExtGlobal 直接使用 `chrome.storage`。
3. **生产环境禁止使用 `console.log`。** 使用 `LoggerService`。
4. **禁止使用全局变量**（在定义的Services外部）。
5. **禁止使用魔法字符串。** 使用常量/枚举处理存储键和CSS类名。
6. **所有注入到Gemini DOM的CSS类必须以 `gv-` 为前缀。**
7. **所有翻译必须同步更新到全部10个语言包**（`en`, `ar`, `es`, `fr`, `ja`, `ko`, `pt`, `ru`, `zh`, `zh_TW`），添加/修改i18n键时必须全部更新。
8. **禁止直接修改 `dist_*` 文件夹。**
9. **禁止提交 `.env` 或密钥文件。**
10. **添加Material Symbol图标时**，将图标名称添加到 `src/pages/popup/index.html` 中Google Fonts URL的 `icon_names=` 参数里。

## 验证（完成前必做）

1. `bun run typecheck` — 任何 `.ts`/`.tsx` 修改后运行
2. `bun run lint` — 结束前运行
3. `bun run test` — 所有测试通过
4. `bun run build:chrome` — 构建无错误
5. 新功能/修复必须包含测试

## 提交格式

Conventional Commits：`<type>(<scope>): <祈使语气摘要>`

- 类型：`feat`, `fix`, `refactor`, `chore`, `docs`, `test`, `build`, `ci`, `perf`, `style`
- 范围：简短，聚焦功能（如 `copy`, `export`, `popup`）
- 摘要：小写，祈使语气，句末不加句号
- 如果提交与GitHub issue或讨论相关，在提交**正文**中包含 `Closes #xxx` 或 `Fixes #xxx`

## 版本递增与发布

```bash
bun run bump    # 自动更新 package.json, manifest.json, manifest.dev.json
```

**变更日志必须：** 递增版本后，确保 `src/pages/content/changelog/notes/` 下有新版本的 `.md` 文件后再推送。不要跳过此步骤。

然后：提交 `chore: bump to v{VERSION}` → `git tag v{VERSION}` → `git push && git push --tags`

## 设计原则

1. **KISS原则。** 按需求的最小化理解实现。未经明确确认，不要合并正交功能（如"淡入"和"细体"）。
2. **向后兼容性是铁律。** 对用户数据零破坏（尤其是 `localStorage`）。
3. **数据结构优先。** 通过重新设计数据消除特殊场景，而非添加分支。
4. **视觉/CSS变更：** 描述预期渲染效果，验证浅色和深色主题下的对齐/居中/间距，检查外部资源（图标字体、CDN链接）。
5. **需求不明确时：** 先实现最小化版本。添加范围前先确认。

## 架构

- **Services**：单例，位于 `src/core/services/`。 `StorageService` 是持久化的单一数据源。
- **内容脚本**：`src/pages/content/`。每个子模块是自包含的。
- **UI**：函数式React组件 + hooks。业务逻辑放在 `features/*/services/` 或自定义hooks中，不放在UI文件里。
- **类型**：`src/core/types/common.ts` 用于存储键和共享类型。
- **翻译**：`src/locales/*/messages.json`（10种语言）。
- **注入的CSS**：`public/contentStyle.css`。

## 任务地图

| 任务 | 位置 |
|------|------|
| 添加存储键 | `src/core/types/common.ts` → `StorageService.ts` → 全部10个语言包 |
| 更新翻译 | `src/locales/*/messages.json`（全部10个） |
| 修改DOM注入 | `src/pages/content/` |
| 修改弹窗设置 | `src/pages/popup/components/` |
| 修复云同步 | `src/core/services/GoogleDriveSyncService.ts` |
| 添加键盘快捷键 | `src/core/services/KeyboardShortcutService.ts` + 类型定义 |
