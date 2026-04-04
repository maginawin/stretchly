# macOS 托盘菜单重构设计

## 概述

重构 macOS 系统托盘菜单的菜单项布局，将原有的子菜单快捷操作提升为独立顶层菜单项，减少操作层级。

## 当前菜单结构（正常运行状态）

```
┌─────────────────────────────┐
│ Next Mini break in 5:00     │  ← 状态信息
│ Next Long break after 2...  │
├─────────────────────────────┤
│ Skip to the next        ▸   │  ← 子菜单: Mini break / Long break
│ Pause breaks            ▸   │  ← 子菜单: 30min / 1h / 2h / 5h / Until morning / --- / Indefinitely
│ Reset breaks                │
├─────────────────────────────┤
│ Preferences                 │
│ Quit Stretchly              │
└─────────────────────────────┘
```

## 目标菜单结构

```
┌─────────────────────────────┐
│ Next Mini break in 5:00     │  ← 状态信息
│ Next Long break after 2...  │
├─────────────────────────────┤
│ Pause indefinitely          │  ← 新增：快捷暂停无限期
│ Pause breaks            ▸   │  ← 上移（子菜单不变）
│ Long break                  │  ← 新增：替代原 Skip to next > Long break
│ Mini break                  │  ← 新增：替代原 Skip to next > Mini break
│ Reset breaks                │
├─────────────────────────────┤
│ Preferences                 │
│ Quit Stretchly              │
└─────────────────────────────┘
```

## 变更详情

### 1. 删除 "Skip to the next" 子菜单

移除整个 "Skip to the next" 菜单项及其子菜单，替换为两个独立菜单项。

### 2. 新增 "Pause indefinitely" 独立菜单项

- 位置：在 "Pause breaks" 上方
- 功能：调用 `pauseBreaks(1)`，与原 Pause breaks 子菜单中 "Indefinitely" 行为一致
- 显示条件：`!isPaused && !isOnDnd && !appExclusionPause`（与原 "Pause breaks" 相同）

### 3. 新增 "Long break" 独立菜单项

- 位置：在 "Pause breaks" 下方、"Mini break" 上方
- 功能：调用 `skipToBreak()`，与原 "Skip to next > Long break" 行为一致
- 显示条件：继承原 "Skip to the next" 的完整条件逻辑（外层 `!isPaused && !isOnDnd && !appExclusionPause`，内层检查 `break` 设置是否启用）

### 4. 新增 "Mini break" 独立菜单项

- 位置：在 "Long break" 下方、"Reset breaks" 上方
- 功能：调用 `skipToMicrobreak()`，与原 "Skip to next > Mini break" 行为一致
- 显示条件：继承原 "Skip to the next" 的完整条件逻辑（外层 `!isPaused && !isOnDnd && !appExclusionPause`，内层检查 `microbreak` 设置是否启用）

### 5. "Pause breaks" 子菜单上移

- 位置从原来 "Skip to the next" 下方移到 "Pause indefinitely" 下方
- 子菜单内容不变（30min / 1h / 2h / 5h / Until morning / --- / Indefinitely）

## i18n 变更

在所有 53 个语言文件中新增键 `main.pauseIndefinitely`，翻译为对应语言。

复用现有键：
- `main.toBreak`（"Long break"）
- `main.toMicrobreak`（"Mini break"）

## 影响范围

- **`app/main.js`**：修改 `getTrayMenuTemplate()` 函数（约 1366-1440 行）
- **`app/locales/*.json`**：53 个语言文件各增加一个键
- **不涉及**：设置（defaultSettings.js）、快捷键、测试文件
- **不影响**：暂停状态菜单（"Resume breaks"）、DND/App Exclusion 状态菜单
