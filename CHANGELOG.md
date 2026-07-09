# CHANGELOG

VoidNotch 的版本與里程碑紀錄。此檔只記錄已完成或已落地的公開變更。

格式參考 Keep a Changelog，版本號在正式 tag 前先以 `dev` 標示。

## [0.5.0-dev] - 2026-07-09

### Added

- **系統 status 監控擴充**（Mole `mo status` 啟發；Stats MIT 移植主線，禁 GPL 抄碼）：
  - 資料層：`DiskReader` / `DiskIOReader` / `NetworkReader` / `BatteryReader` / `HostInfoReader` / `ProcessReader` / `GPUReader` / `HealthScorer`；`SystemSnapshot` 擴充 disk/net/battery/gpu/host/topProcesses/health；CPU 補 load average；Manager 分頻輪詢。
  - Compact 四格：`CPU% · MEM% · DISK% · NET↓`（`ViewThatFits` 降級）。
  - Expanded 分區卡：Health / CPU / Memory / Disk / Network / Power / Thermal·GPU / Host / Top processes（EN-first）。
  - GPU util best-effort（IOKit PerformanceStatistics）；無資料顯示 `—`，不假造。
  - `vn-probe` / `vn-selftest` 覆蓋新指標。
- **Notch compact 佈局設定**：Settings 可為左右兩側各自選擇要顯示的 widget、開關 pinned、調整 max width（48–240pt）與 content height（16–36pt）；`NotchCompactLayoutStore` + `NotchCompactLayout` clamp；Agent compact 改為活動 pill。
- **System metrics 可選**：Settings 可開關 CPU / Memory / Disk / Network / Battery / Temp / Health / GPU / Host / Processes；compact 與 expanded 同步；至少保留一個 compact 指標。
- 預設 compact 寬度：**左 150 / 右 110**。

### Verified

- `swift test`：見最新 commit 輸出（含 layout clamp / system metric prefs）。
- `swift run vn-selftest`：33 通過。
- `swift build --product VoidNotch`：SUCCEEDED。
- 本機 M4 Pro probe：Disk/Net/Battery/Health/Top processes 有值；GPU util 偶發可讀。

---

## [0.4.0-dev] - 2026-07-09

### Added

- **Grok 用量監控**：`TokenProviderKind.grok` 從 pending 升為 live adapter；`CodexBarTokenUsageProvider` 映射 CodexBarCore `UsageProvider.grok`（`grok agent` billing RPC → grok.com web billing → 本機 auth 身份）。主視窗標題對齊 CodexBar 動態 Credits/Weekly/Monthly；預設可見清單已含 Grok。

### Added (earlier 0.4.0-dev)

- **免 Xcode CLI 建置/打包**：`Package.swift` 新增 executable target `VoidNotch`（源碼 `App/`，依賴 DynamicNotchKit + CodexBarCore，revision 對齊原 xcodeproj pin）；新增 `scripts/make_app.sh`（`swift build -c release` → 組 `build/VoidNotch.app` + Info.plist + ad-hoc 簽名，`--run` / `--install` 直接啟動或掛到 /Applications）。全程不需開 Xcode GUI；仍需完整 Xcode 工具鏈（DynamicNotchKit 的 SwiftUI `@Entry` 巨集，CLT-only 編不過，見 dynamicnotchkit-spike §5）。
- **語言切換 zh-TW / EN**：`VoidNotchKit/AppLanguage`（預設 zh-TW，未知值 fallback，附單元測試）+ `App/Theme/L10n.swift` UI chrome 字串表；設定工具列語言 segmented picker 即時切換（`@AppStorage`），狀態列選單與視窗標題經 UserDefaults 觀察者重建，展開面板/分頁/系統與 Agent 面板全數本地化。provider 回傳的資料細節字串維持英文（住 Kit）。
- **共用元件庫 `App/Components/`**：`ProviderIcon` / `ProviderStatusDot` / `ProviderStatusBadge` / `UsageWindowRow` / `UsageBar`（provider 視覺）與 `NotchCardModifier`（`.notchCard()`）/ `SummaryPill` / `NotchEmptyState`（通用面板元素），供 Token / Agent / Settings 三處共用。
- `App/Theme/ProviderAppearance.swift`：provider 圖示/色票與狀態色的單一真相。
- **Gemini (Agy) 多帳號管理**：新增 provider-neutral `ProviderAccount` / `ProviderAccountUsage` / `ProviderAccountImport` / `ProviderAccountExport` / `TokenAccountManaging`；App 層 `CodexBarTokenAccountManager` 沿用 CodexBar token-account store，支援匯入目前 Antigravity OAuth credentials、貼上 refresh token/OAuth JSON/Antigravity-Manager export JSON、依 Google email 去重、切換 active account、移除帳號、匯出 refresh-token JSON。VoidNotch Settings 的 AGY 詳細頁新增 Accounts 區塊；selected account 會注入 `ANTIGRAVITY_OAUTH_CREDENTIALS_JSON` 並傳 `selectedTokenAccountID` 給 CodexBarCore fetch context，使 AGY quota fetch 對齊選定帳號；切換/刪除 active account 也會同步 CodexBar shared AGY credentials。帳號列會逐一掃描 live quota、顯示可用狀態/主要剩餘額度，並依剩餘配額標記 Best account；手動或 403/ban 類錯誤標記為 skipped 的帳號會排除於背景用量刷新與 Best 推薦之外。

### Changed

- **依賴單一真相**：xcodeproj 移除 DynamicNotchKit / CodexBar 遠端 package 引用（先前 branch vs revision 需求與本地 Package.swift 衝突，xcodebuild 無法解析），產品依賴改由本地 package 依賴圖解析；兩條建置路徑用同一組 pin（根目錄 `Package.resolved` 入版控）。
- **代碼瘦身 + 元件化**：TokenWidget / ProviderSettingsView / AgentActivityWidget 內約 12 個私有元件（provider 圖示、狀態徽章、配額列、進度條、卡片底、統計 pill、空狀態，多為兩三份逐字重複）收斂到 `App/Components/` 與 `ProviderAppearance`；刪除死碼 `AgentActivityCompactView`（compact 狀態已併入 AI 摘要膠囊）與對應 preview；`src/` 空佔位移除。
- **UI 簡約化**：Token 展開卡移除統計 pills 列（Enabled/Available/Needs check，與書籤列狀態點重複）、header 副標語、capability chips + identity 資訊列（完整資訊仍在 Settings 的 Capability Matrix / Info Grid）；修 NotchShell deprecated `onChange` 與多餘 `await` 警告，`swift build` 零警告。

### Verified

- `swift test`：71 tests, 0 failures（2026-07-09，含 Grok live adapter 測試）。
- `swift run vn-selftest`：24 通過，0 失敗（2026-07-06）。
- `xcodebuild -project VoidNotch.xcodeproj -scheme VoidNotch -configuration Debug -destination platform=macOS build`：BUILD SUCCEEDED（2026-07-06）。
- `swift build --product VoidNotch`：BUILD SUCCEEDED（2026-07-09，含 Grok mapping）。
- `scripts/make_app.sh --run`：打包啟動成功；展開面板 zh-TW（系統監控 / 模型配額 / 系統 / 記憶體 / CPU 溫度）與 EN（System / Model Usage / CPU Temp）雙語真機目視通過（Codex/Claude 即時配額正常顯示）。
- `swift test --filter TokenStoreTests`：15 tests, 0 failures（2026-07-06，含 AGY account catalog / active switch / current import / token JSON import / quota recommendation / skipped account recommendation / account export）。

### Pending

- Grok 真機 notch 刷新目視（需瀏覽器 grok.com session 或 `grok` CLI billing 可用）。
- 本機（macOS 26）IOHID 溫度讀不到值（`vn-probe` 同樣為空），CPU/GPU 溫度顯示 `—`；待溫度感測 key 對照調查。
- 30 分鐘常駐穩定性浸泡仍待人工確認；`v0.3.0` / `v0.4.0` tag 由契約者驗收後打。

---

## [0.3.0] - 2026-06-23

### Added

- 新增 `NotchMetrics` 幾何 helper（`VoidNotchKit`），封裝 `NSScreen` 推導的 `expandedTopInset`、`expandedMaxHeight`、`compactPanelRect`；附單元測試，測試總數由 49 升至 50。

### Changed

- **元件 A — 展開防截斷**：展開面板改用 `ScrollView`，頂部留白改由 `NotchMetrics.expandedTopInset` 動態計算（依 `NSScreen` 瀏海高度推導），取代寫死 `.padding(.top, 42)`；內容超過面板高度上限（`expandedMaxHeight`）時改為可滾動，不被實體瀏海截斷。
- **元件 B — compact 全左移**：compact 內容全部收到瀏海左側；左側顯示 System 指標 + 單一 AI 摘要膠囊（Token 主 provider 用量百分比 + Token/Agent 合併狀態點）；`AgentActivityWidget` compact 改為 0 寬不佔空間；Token/Agent 顯式設定 `preferredSide = .leading`；設定頁 Notch Layout 由雙側佈局改為單側，右側視覺淨空。
- **元件 C — 選單列共存（路徑 F）**：根因為 DynamicNotchKit panel 寫死 `width/2 × height/2` 且未設 `ignoresMouseEvents`，整塊攔截右側選單列點擊。收窄 panel 的 spike 在真機 FAIL（`setFrame` 破壞 DynamicNotchKit 內容置中，無可行修法）；最終採路徑 F：保留原生 panel 尺寸，在 140ms 輪詢中依滑鼠位置動態切換 `ignoresMouseEvents`（滑鼠在「左側內容 + 瀏海」命中區外時 panel 穿透，右側選單列圖示與 Ice 圖示可點）；展開/點擊改用自算 hitRect 命中，不依賴 `.onHover`。

### Verified

- `swift test`：50 tests, 0 failures（2026-06-23）。
- `swift run vn-selftest`：24 通過，0 失敗（2026-06-23）。
- `xcodebuild -project VoidNotch.xcodeproj -scheme VoidNotch -configuration Debug -destination platform=macOS build`：BUILD SUCCEEDED（2026-06-23）。

### Pending

- 真機 30 分鐘穩定性浸泡尚在進行中；版本 tag `v0.3.0` 待浸泡完成 + 最終 review 後由契約者打。

---

## [0.2.0] - 2026-06-23

### Added

- 新增 `VoidNotchKit` SPM library target，將 token/agent 純邏輯從 App 層抽離，使其在無 UI 的 CLI 環境可測試。
- 新增 `mapConcurrentlyPreservingOrder` 並行輔助函式，CodexBar provider 抓取改為並行且保留順序。
- 新增 `TokenStore`、`TokenProviderKind`、`ProviderUsage`、`UnavailableTokenUsageProvider` 等 token 純邏輯型別移入 `VoidNotchKit`；App 層注入 `CodexBarTokenUsageProvider`。
- 新增 `AgentEventLogParser` 移入 `VoidNotchKit`，agent 模型與 store 同步移入。
- 新增 `NotchSide` enum 與 `NotchWidget.preferredSide`，compact 左/右放置改用明確語意取代 array-index 假設。
- 新增 `App/Theme/Theme.swift`，集中視覺常數；`SystemWidget` 與 `TokenWidget` 改由 Theme tokens 取值。
- 新增 `CPULoad.coreSplitIsHeuristic` 旗標，標記 P/E 核心分割為未驗證的啟發式推算。
- 新增 `VoidNotchKitTests`（token 格式化/縮寫/成本/視窗排序/provider 選擇/持久化/刷新）與 `AgentEventLogParserTests`，測試總數從 23 升至 45。

### Changed

- `SystemMonitorManager` 輪詢改為自適應節奏：前景 1s / 背景 3s / idle 10s，切換節奏不重置 CPU diff baseline。
- `PeonPingAgentActivityProvider` 的 JSONL 檔案 IO 改為 `Task.detached(.utility)` 背景執行，解除 MainActor 阻塞。

### Verified

- `swift test`：45 tests, 0 failures（2026-06-23）。
- `swift run vn-selftest`：24 通過，0 失敗（2026-06-23）。
- `xcodebuild -project VoidNotch.xcodeproj -scheme VoidNotch -configuration Debug -destination platform=macOS build`：BUILD SUCCEEDED（2026-06-23）。

---

## [0.2.0-dev] - 2026-06-22

狀態：已升版為 [0.2.0]，此節保留原始開發期紀錄。

### Added

- 建立 `VoidNotch.xcodeproj` macOS App target，使用 SwiftUI、macOS 14+、`LSUIElement = YES`。
- Xcode app target 已接入本 repo 的 `SystemMonitor` library 與 DynamicNotchKit。
- App target 已加入 CodexBarCore SPM dependency，revision 釘選於 Xcode project / `Package.resolved`。
- 新增 `CodexBarTokenUsageProvider` adapter，將 CodexBarCore cost / quota snapshot 映射成 VoidNotch 的 `ProviderUsage`。
- `TokenStore` 擴充為多 provider 狀態容器，產品面目前先開放 Codex、Claude、Gemini (Agy)。
- Token provider 設定入口已接到狀態列選單：`VoidNotch Settings...`。
- Token 展開 UI 新增 provider card、quota window、剩餘/已用顯示模式。
- Gemini (Agy) 已接 quota pipeline，可顯示 Gemini Models quota window。
- 新增 `AgentActivityStore` 與 `AgentActivityWidget` 初版，先承接 Codex / Claude / Gemini (Agy) lifecycle 事件模型。
- 新增 `PeonPingAgentActivityProvider`，可讀取本機 JSONL relay 事件並映射為 Agent Activity timeline。
- 新增 `resources/hooks/peonping-voidnotch-relay.sh`，供 PeonPing / agent hooks append VoidNotch agent lifecycle JSONL。
- 新增 PeonPing → VoidNotch relay 設定文件。
- 新增 Widget Visibility 設定，可在設定視窗切換 Token / Agent Activity 顯示；System Metrics 作為 v1 核心 widget 保持常駐。
- 新增 `SystemMonitorManagerTests`。
- 新增 `ThermalFailure` 與 `Logger`，保留溫度讀取失敗原因與診斷紀錄。
- 新增 Xcode 測試教學、Xcode app target 接線清單與程式碼審查紀錄。

### Changed

- README 與 `docs/PROGRESS.md` 更新為 2026-06-22 狀態：資料層、Xcode build、Notch compact 第一輪真機驗收皆已通過。
- `TokenWidget` 從佔位展示升級為 provider 狀態與 quota / cost 展示。
- Provider 展示 UI/UX 初版升級：expanded widget 新增 provider health summary、capability chips、資料覆蓋摘要、身份資訊、空狀態提示；provider settings 視窗新增健康狀態、能力矩陣與更完整的明細面板。
- Provider UI 文案移除內部 `CodexBarCore` 名稱，改以使用者可理解的資料來源描述顯示。
- Provider 預設展示範圍限縮為 Codex、Claude、Gemini (Agy)，其餘 provider case 暫不出現在設定與 notch 預設 UI。
- Notch compact 右側從單 widget 擴為雙 widget，讓 token 與 agent activity 可以同時顯示。
- App 啟動時 Agent Activity store 改為讀取 PeonPing/VoidNotch relay JSONL，預設路徑為 `~/Library/Application Support/VoidNotch/agent-events.jsonl`。
- Notch compact 改為左右兩側固定寬度區塊，Token / Agent idle 文案預設自動收縮，避免右側過長。
- Notch compact 的外露長顯示勾選框已移除；長顯示控制改放在 provider 設定視窗的 Notch Layout 區塊。
- Notch compact 預設短版寬度再收窄，右側 AI / Agent 狀態避免在收合時往 MacBook 瀏海右側外突。
- `NotchShell` 改為依 `WidgetRegistry.visibleSortedByPriority` 動態渲染 compact / expanded，讓設定中的 widget 顯示狀態可立即反映在 Notch UI。
- `NotchShell` 參考 MacNotch gallery 的頂邊固定收合方向調整轉場節奏；compact 左/右 slot 改為由 Notch Layout 獨立控制 `Open` / `Collapsed`。
- App 啟動後進入 compact，預設左側系統資訊常開、右側 AI 縮入劉海；狀態列新增 `Show Dashboard` / `Collapse to Notch` 以手動展開與收合。
- Collapsed 側 compact slot 不渲染 widget，避免該側在實體瀏海外側殘留或向右突出。
- `NotchShell` 改為 click-to-expand：compact 狀態滑鼠移入 notch 後需左鍵點擊才展開，避免 hover 過於敏感；展開後滑鼠離開仍會延遲自動收回 compact。
- Expanded dashboard 頂部新增額外留白，避免內容直接貼齊螢幕頂邊而被實體劉海或 mask 遮住。
- Notch 右鍵點擊新增設定入口，可直接開啟 `VoidNotch Settings`。
- `TokenStore.refresh()` 新增重入合併，避免啟動輪詢、手動刷新或設定切換同時觸發重複 provider 抓取。
- VoidNotch process 內強制停用 CodexBarCore real keychain cache 存取，避免 macOS 跳出 `CodexBar Cache` 錯品牌鑰匙圈提示。
- 狀態列設定入口由 `Token Providers...` 改名為 `VoidNotch Settings...`，同一視窗集中 provider、layout 與 widget 顯示控制。
- `SystemMonitorManager` 輪詢間隔加上下限保護，避免過密輪詢。
- `ThermalReader` 改為 best-effort 讀取時同步保留失敗原因。
- `Package.swift` 註明資料層與 Xcode app target 的分工，CodexBarCore 由 app target 連結。

### Verified

- `swift run vn-selftest`：24 通過，0 失敗。
- `swift test`：23 tests, 0 failures。
- `xcodebuild -project VoidNotch.xcodeproj -scheme VoidNotch -configuration Debug -destination platform=macOS build`：通過。
- 2026-06-22 provider UI/UX 改版後重跑 `swift test`：23 tests, 0 failures；重跑 Xcode Debug build：通過。
- 2026-06-22 provider 範圍收斂與 Agent Activity scaffold 後重跑 `swift test`：23 tests, 0 failures；重跑 Xcode Debug build：通過。
- 2026-06-22 PeonPing relay 接線後：relay script smoke test 產生可解析 JSONL；`swift test` 23 tests, 0 failures；Xcode Debug build 通過。
- 2026-06-23 compact 左右收縮與長顯示勾選控制後：`swift test` 23 tests, 0 failures；Xcode Debug build 通過。
- 2026-06-23 compact 外露勾選框移除、長顯示控制移入設定後：`swift test` 23 tests, 0 failures；Xcode Debug build 通過。
- 2026-06-23 Widget Visibility 設定接入後：`swift run vn-selftest` 24 通過，0 失敗；`swift test` 23 tests, 0 failures；Xcode Debug build 通過。
- 2026-06-23 MacNotch-style 收合方向接入後：Xcode Debug build 通過。
- 2026-06-23 左/右 compact slot 改為 `Open` / `Collapsed` 可選後：Xcode Debug build 通過。
- 2026-06-23 左鍵點擊展開、右鍵開設定、expanded 頂部留白、token refresh 合併、CodexBarCore keychain prompt 停用後：Xcode Debug build 通過。
- 歷史真機驗收：有瀏海 MacBook 曾成功顯示 compact 條，左側 CPU/RAM/溫度、右側 token 區塊可視；現行策略改為左/右可個別常開或縮入劉海。

### Pending

- 仍需完成 App target live update 觀察、10-30 分鐘穩定性、Activity Monitor 對照。
- 仍需在有瀏海 MacBook 上重新啟動 App，確認左/右任一側設為 `Collapsed` 後，該側實體瀏海外側沒有任何殘留 widget、mask 或陰影。
- 仍需真機驗收 click-to-expand 行為：compact 時滑鼠移入 notch 後左鍵點擊才展開；右鍵點擊 notch 會開啟設定；滑鼠離開 dashboard 後會自動收回 compact。
- 仍需真機點選 VoidNotch Settings 的 Widget Visibility，確認 Token / Agent Activity 開關會即時反映到 dashboard expanded。
- 仍需對照 Claude / Codex / Gemini (Agy) 等 provider 的真實 token / quota 資料。
- 仍需把 PeonPing relay 實際註冊到 Claude / Codex / Gemini (Agy) hook 設定並做真機事件驗收。
- 仍需決定 v1 是否顯示 Token widget，或將 Token 明確標為 v2 beta。

## [0.1.0-dev] - 2026-06-22

狀態：`dev` 分支目前 HEAD `822ec1c`。

### Added

- 建立 `SystemMonitor` Swift library target。
- 建立 `CSensors` Obj-C bridge target，用於 Apple Silicon IOHID 溫度讀取。
- 建立 `vn-probe` 即時探針。
- 建立 `vn-selftest` CommandLineTools 自測。
- 建立 `Tests/SystemMonitorTests` XCTest 測試。
- 建立 `App/` SwiftUI 層骨架：`NotchShell`、`NotchWidget`、`WidgetRegistry`、`SystemWidget`、`TokenWidget`。
- 完成 CodexBarCore 引入性 spike 重驗，確認外部 SPM consumer 可讀本機 Claude token 用量。
- 補齊 LICENSE 與第三方授權聲明。

### Changed

- 測試從 Swift `Testing` 轉為 XCTest，讓完整 Xcode toolchain 下的 `swift test` 可跑。
- v1 範圍當時收斂為 Notch compact 外殼 + CPU/RAM/溫度；Token 追蹤延為 v2，但保留可行性證據。
- 發佈軌道確認為 Developer ID + Hardened Runtime + Notarization，不走 Mac App Store。

### Verified

- 系統監控資料層可建置。
- `vn-selftest` 與 `swift test` 已可作為資料層驗證 gate。

## [0.0.1-dev] - 2026-06-18

### Added

- 完成 v1 架構設計文件。
- 完成 GitHub 生態勘查，評估 boring.notch、DynamicNotchKit、Stats、CodexBar 等來源。
- 完成 DynamicNotchKit spike，確認有瀏海 MacBook 的 compact API 路徑可行。
- 完成 CodexBar 架構分析與 CodexBarCore 可行性分析。
- 確認 boring.notch 因 GPL-3.0 僅作架構參考，不 vendoring。

## [0.0.0] - 2026-06-07

### Added

- 初始化 VoidNotch repo。
- 專案方向定為 macOS Notch 系統監控 + AI Token 追蹤工具。
- 初始需求：CPU、RAM、溫度、AI provider token 流量。
