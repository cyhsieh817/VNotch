# CHANGELOG

VoidNotch 的版本與里程碑紀錄。只記錄已完成或已落地的公開變更。

- 格式： [Keep a Changelog](https://keepachangelog.com/)
- 版號規則：**權威見 [`VERSIONING.md`](VERSIONING.md)**（SemVer；0.x pre-stable）
- 進行中版本以 `-dev` 標示；正式 tag 去掉 `-dev` 並寫日期
- **2026-07-09 重基線**：自 `0.1.0` 起依產品能力重算；舊口語／舊 tag 對照見 VERSIONING §6

---

## [0.6.0] - 2026-07-09

> 正式公開發佈（GitHub Release）。對應舊稱 `0.5.0-dev`。含 App icon（logo candidate E 黑白 notch mark）。

### Added

- **系統 status 監控擴充**（Mole `mo status` 啟發；Stats MIT 移植主線，禁 GPL 抄碼）：
  - 資料層：`DiskReader` / `DiskIOReader` / `NetworkReader` / `BatteryReader` / `HostInfoReader` / `ProcessReader` / `GPUReader` / `HealthScorer`；`SystemSnapshot` 擴充 disk/net/battery/gpu/host/topProcesses/health；CPU 補 load average；Manager 分頻輪詢
  - Compact 四格：`CPU% · MEM% · DISK% · NET↓`（`ViewThatFits` 降級）
  - Expanded 分區卡：Health / CPU / Memory / Disk / Network / Power / Thermal·GPU / Host / Top processes（EN-first）
  - GPU util best-effort（IOKit PerformanceStatistics）；無資料顯示 `—`，不假造
  - `vn-probe` / `vn-selftest` 覆蓋新指標
- **Notch compact 佈局設定**：Settings 可為左右兩側各自選擇 widget、pinned、max width（48–240pt）、content height（16–36pt）；`NotchCompactLayoutStore` + clamp；Agent compact 改為活動 pill
- **System metrics 可選**：Settings 可開關 CPU / Memory / Disk / Network / Battery / Temp / Health / GPU / Host / Processes；compact 與 expanded 同步；至少保留一個 compact 指標
- 預設 compact 寬度：**左 150 / 右 110**

### Verified

- `swift test`：含 layout clamp / system metric prefs（見開發當日輸出；約 87 tests）
- `swift run vn-selftest`：33 通過
- `swift build --product VoidNotch`：SUCCEEDED
- 本機 M4 Pro probe：Disk/Net/Battery/Health/Top processes 有值；GPU util 偶發可讀

### Known issues / Pending

- Grok 真機 notch 刷新目視（需 grok.com session 或 `grok` CLI billing）
- 部分機型／系統 IOHID 溫度為空（顯示 `—`）
- 30 分鐘常駐穩定性浸泡待人工確認
- 已知限制：ad-hoc 簽名（首次開啟請右鍵→打開）；僅 Apple Silicon；溫度/Grok 真機視環境

---

## [0.5.0] - 2026-07-09

> 產品化與 Token 能力擴充。對應舊稱 `0.4.0-dev`（含 earlier 段）。

### Added

- **CLI 建置/打包**：`Package.swift` executable target `VoidNotch`（`App/`）；`scripts/make_app.sh`（`swift build -c release` → `build/VoidNotch.app` + ad-hoc 簽名；`--run` / `--install`）
- **語言切換 zh-TW / EN**：`AppLanguage` + `L10n`；設定工具列即時切換；provider 資料細節字串維持英文
- **共用元件庫 `App/Components/`** 與 `ProviderAppearance`（圖示／色票／狀態色單一真相）
- **Gemini (Agy) 多帳號管理**：匯入／貼上／匯出、切換 active、skip、Best account、per-account quota；與 CodexBarCore fetch context 對齊
- **Grok 用量監控**：`TokenProviderKind.grok` 升為 live adapter（CLI billing → web billing → 本機 auth）

### Changed

- xcodeproj 與 CLI 共用本地 package 依賴圖與 `Package.resolved`
- Token / Agent / Settings 私有 UI 收斂到 Components；UI 簡約化（展開卡去重資訊）
- `swift build` 零警告路徑收斂

### Verified

- `swift test`：71+ tests（含 Grok live adapter）
- `scripts/make_app.sh --run`：雙語真機目視；Codex/Claude 配額可顯示
- AGY 多帳相關 `TokenStoreTests` 通過

---

## [0.4.0] - 2026-06-23

> Notch UX 正確性。對應舊文 `0.3.0`。

### Added

- `NotchMetrics`（`expandedTopInset` / `expandedMaxHeight` / `compactPanelRect`）；測試 49 → 50

### Changed

- **展開防截斷**：`ScrollView` + 動態頂部留白，超高可滾動
- **Compact 內容策略**：系統指標 + AI 摘要膠囊；Agent compact 不佔寬；Token/Agent `preferredSide = .leading`
- **選單列共存（路徑 F）**：依滑鼠位置動態 `ignoresMouseEvents`，右側選單列／Ice 可點；點擊用自算 hitRect

### Verified

- `swift test` 50 綠；`vn-selftest` 24 通過；Xcode Debug build 通過（2026-06-23）

---

## [0.3.0] - 2026-06-23

> 架構韌性與可測性。對應舊 git tag **`v0.2.0`** 與舊 CHANGELOG `[0.2.0]`。

### Added

- `VoidNotchKit` SPM library：Token / Agent 純邏輯 CLI 可測
- `mapConcurrentlyPreservingOrder`；provider 抓取並行保序
- `NotchSide` / `preferredSide`；`Theme` tokens
- `CPULoad.coreSplitIsHeuristic`；`AgentEventLogParser`；測試 23 → 45

### Changed

- 自適應輪詢：前景 1s / 背景 3s / idle 10s（不重置 CPU baseline）
- Agent JSONL IO 移出 MainActor（`Task.detached`）
- `TokenStore` 預設 `UnavailableTokenUsageProvider`，App 注入 CodexBar adapter

### Verified

- `swift test` 45 綠；`vn-selftest` 24；Xcode Debug build 通過

---

## [0.2.0] - 2026-06-22

> 首個「有 Token／Agent 的產品面」。對應舊 `0.2.0-dev` 的 App 功能（不含後續 Kit 抽取）。

### Added

- `VoidNotch.xcodeproj` macOS App（SwiftUI、macOS 14+、`LSUIElement`）
- DynamicNotchKit + CodexBarCore；`CodexBarTokenUsageProvider`
- 多 provider 狀態：產品面 Codex / Claude / Gemini (Agy)
- Token 展開 UI（quota window、剩餘/已用模式）；VoidNotch Settings
- `AgentActivityStore` / `AgentActivityWidget`；PeonPing JSONL relay 與 hook script
- Widget Visibility；System Metrics 為核心常駐 widget
- `ThermalFailure` / `Logger`；SystemMonitor 測試補強

### Changed

- Compact 左／右 Open–Collapsed、click-to-expand、右鍵開設定
- Token refresh 重入合併；停用 CodexBarCore real keychain cache（避免錯品牌提示）
- Provider 預設範圍收斂；UI 文案去內部庫名

### Verified

- `swift test` 23 綠；`vn-selftest` 24；Xcode Debug build 多輪通過
- 有瀏海 MacBook compact 歷史真機驗收通過第一關

---

## [0.1.0] - 2026-06-22

> 首個可跑的資料層 + Notch 骨架。對應舊 `0.1.0-dev`。

### Added

- `SystemMonitor` library；`CSensors` IOHID 溫度 bridge（best-effort）
- `vn-probe`、`vn-selftest`；`Tests/SystemMonitorTests`（XCTest）
- `App/` 骨架：`NotchShell`、`NotchWidget`、`WidgetRegistry`、`SystemWidget`、`TokenWidget` 佔位
- CodexBarCore 引入性 spike 重驗（本機 Claude token 可讀）
- LICENSE 與第三方授權聲明

### Changed

- 測試改 XCTest，完整 Xcode toolchain 下 `swift test` 可跑
- v1 當時範圍：Notch + CPU/RAM/溫度；Token 列為後續（後於 0.2.0 產品化）
- 發佈軌道：Developer ID + Hardened Runtime + Notarization（不可 Mac App Store）

### Verified

- 資料層可建置；`vn-selftest` / `swift test` 可作 gate

---

## [0.0.1-dev] - 2026-06-18

### Added

- v1 架構設計、GitHub 生態勘查、DynamicNotchKit / CodexBarCore spike
- 確認 boring.notch 僅架構參考（GPL，不 vendoring）

---

## [0.0.0] - 2026-06-07

### Added

- 初始化 VoidNotch repo
- 方向：macOS Notch 系統監控 + AI Token 追蹤
