# CHANGELOG

VoidNotch 的版本與里程碑紀錄。只記錄已完成或已落地的公開變更。

- 格式： [Keep a Changelog](https://keepachangelog.com/)
- 版號規則：**權威見 [`VERSIONING.md`](VERSIONING.md)**（SemVer；0.x pre-stable）
- 進行中版本以 `-dev` 標示；正式 tag 去掉 `-dev` 並寫日期
- **2026-07-09 重基線**：自 `0.1.0` 起依產品能力重算；舊口語／舊 tag 對照見 VERSIONING §6

---

## [0.8.0]

### Fixed

- **Settings 視窗 0×0 隱形修復（右鍵叫不出設定）**：`NSHostingController` 掛上時 SwiftUI 初始 fitting size 可能為 0，若高度 preference 回呼因競態未觸發，視窗永久卡 0×0——右鍵其實每次都開了一扇看不見的窗。改為掛上後決定性 `setContentSize`（820×360 起步），preference 只負責精修高度；實機以 CGEvent 注入右鍵回歸驗證（視窗 820×545 置中現身）。
- **launchctl pipe 死結**：`runLaunchctlList` 改先 `readDataToEndOfFile` 排空 pipe 再 `waitUntilExit`，輸出超過 64KB 不再死結截斷。
- **更新橫幅不再靜默消失**：version.json 的 `url` 無效時仍顯示「新版本可用」文字，僅隱藏下載連結。
- **同 Label 多檔去重**：掃描去重鍵改為 launchd Label 本身（首見者勝，user 目錄優先），Finder 複製出的同 Label plist 不再顯示重複列。
- **compact 條空縫**：`NotchWidget` 新增 `hasCompactContent`（預設 true），排程 widget 回 false，不再於 compact HStack 佔位插 spacing。
- **對外連結單一真相源**：新增 `VoidNotchLinks`（site＋updateEndpoint），四處網域字面值收斂，杜絕死網域漂移再犯。
- **排程清單高度隨場景**：`LaunchdScheduleExpandedView` 清單限高參數化（notch 220pt／Settings 480pt）。
- **phase 判定單一權威**：plist parser 不再預判 phase（含重複的 `_DELETE_` 判定），三態邏輯唯一活在 scanner。

- **Xcode 專案漏收 Gauge 新檔**：`VoidNotch.xcodeproj` 以顯式檔案引用（非 filesystem-synchronized group），`GaugeMetrics`／`RingsSkin`／`GlassSkin`／`GaugeSkinPreview` 四個新檔未登記進 `project.pbxproj`。SPM（`make_app.sh` 走的 `swift build`，`path:"App"` 自動納入）不受影響，但 `xcodebuild` 會漏編這些檔。已補齊四處登記（PBXBuildFile／PBXFileReference／群組／Sources phase），`xcodebuild ... build` 與 `swift build` 皆成功。
- **provider icon 選擇器六個選項顯示同一張圖**：`scripts/make_app.sh` 從未把 `resources/provider-icons/` 複製進 app bundle，與 `VoidNotch.xcodeproj` 的 folder reference 形成兩條不同的建置路徑；`Bundle.main.url(...)` 因此一律回 nil，六個選項全數靜默 fallback 到同一個 SF Symbol。腳本補上資產複製與五個 svg 的建置期斷言，缺任一即 `exit 1`；選擇器預覽由 13pt 放大為 22pt。
- **`make_app.sh` 在中文全形字前觸發 `unbound variable`**：`set -euo pipefail` 下，`$VAR（中文全形括號` 會把全形字的第一個 UTF-8 位元組吃進變數名；全檔三處一律改為 `${VAR}`。
- **設定視窗固定高度造成內容少的分頁大片留白**：原本 `.frame(width: 960, height: 760)` 寫死，且零捲動版面以 Spacer 撐滿。改由 PreferenceKey 回報分頁內容高度後呼叫 `setContentSize`，加入 clamp、1pt 門檻擋抖動，以 `oldFrame.maxY - newHeight` 釘住上緣，並套用 0.22s easeOut；刻意不使用 `NSHostingController` 自動 sizing，避免尺寸回寫的約束迴圈。
- **設定視窗內縮、欄位格線與工具列資訊錯位**：新增 `SettingsMetrics` 作為 inset 20、windowWidth 820、sidebarWidth 260 等版面單一真相，刪除 providerDetail 用來抵銷 subTabBar 錯誤內縮的 `-18` 負 padding，Layout 的 Height 列改用與上方兩張卡片相同的兩欄格線；toolbar 只保留 Language 與當前分頁標題，Metric、Refresh 與健康度副標題移至 Providers 分頁頁首。
- **compact 列內容被硬裁切**：`.fixedSize(horizontal: true)` 會對子樹提出不受限的寬度建議，使各 widget 內的 `ViewThatFits` 永遠選中最寬的變體而無視側邊寬度上限，接著 `.clipped()` 把溢出的部分切掉。trailing 側靠右對齊，因此被切掉的是最左邊的元件。改為讓寬度上限真正傳遞到子樹，`ViewThatFits` 得以正常降級，寬度回報改量 HStack 內容本身。
- **移除展開面板內的巢狀 ScrollView**：外層 `NotchExpandedPanel` 已提供垂直捲動，`TokenExpandedView` 的 bookmark 列與 `SystemExpandedView` 各自又包了一層，屬多餘。此項為結構清理，並未修正面板空白，空白的根因是 Picker 的 label，見另一條。
- **展開面板多出 88pt 空白**：`TokenExpandedView` 的 Picker 漏加 `.labelsHidden()`，帶 label 的 Picker 被 `.frame(width: 116)` 夾住後將整列往縱向撐高。ImageRenderer 離線量測同一段 header 帶 label 為 174pt，加上 `.labelsHidden()` 後為 86pt。
- **compact 列 SF Symbol 左緣被切掉**：`ProviderGlyphArtwork` 把天生比字級寬的 SF Symbol 塞進 `size × size` 正方形，`AISummaryCapsule` 的 `.clipped()` 又貼齊 glyph 左緣。symbol 分支改為只約束高度、寬度依自然值，並移除 `.clipped()`。
- **同列系統卡片上緣不齊且高度不同**：`LazyVGrid` 的 `GridItem(.flexible())` 未指定 alignment 時會垂直置中，卡片又只取內容高度。改用 `alignment: .top`，新增 `fillsRowHeight` 撐滿列高，卡片內容改為 `.topLeading`；格線外的 host 與 processes 不套用。

- **顯示項目勾選框失效**（gauge／menubar 共用的「勾選並拖曳排序」）：
  - **勾了不會消失**：編輯器把「已選」與「未選」拆成兩個並排 `ForEach` 畫進同一個 `LazyVGrid` 並共用 id；項目跨 ForEach 遷移時 SwiftUI 重用了格位的 checkbox 卻不刷新狀態，於是設定明明已寫入、畫面上的勾卻不動。改為整份目錄只走一個 `ForEach`。
  - **空的勾選框按不動**：gauge 選滿 4 項（舊上限）後，所有未選項目被靜默停用且不解釋原因。上限提高到 6，並顯示 `n/6` 計數；達上限時明說「先取消一項才能再加」。

### Changed

- **Agent Activity 顯示時間窗**（`b86aa51`）：嚴格保留最近 30 秒，不再以舊事件回填；事件檔 snapshot 未變時仍依時間淘汰卡片；顯示最多 50 筆。
- **設定視窗全面零捲動重排**：本輪將視窗寬度調整為 820，並改為高度隨分頁內容自適應；移除所有 ScrollView。版面分頁下分 `Notch`／`Menu bar`／`Floating Gauge` 子頁，Providers 細節下分 `Usage`／`Details`／`Accounts` 子頁——所有內容都在同一視覺層完整顯示，靠頁籤切換而非捲動。

### Added

- **AGY 多帳號補完（進水口＋出水口）**：多帳號 UI 與後端原已全套，但在只裝 agy CLI、未裝 Antigravity IDE 的機器上整鏈斷頭——`Import Current` 只讀 CodexBar 專用的 `~/.codexbar/antigravity/oauth_creds.json`（本機永不存在），`setActiveAccount` 也不寫回 agy CLI 的 token 檔，切換名存實亡。新增 `AgyCLIOAuthBridge`（VoidNotchKit 純邏輯層）處理 agy CLI token 檔（`~/.gemini/antigravity-cli/antigravity-oauth-token`）的解析／序列化、RFC3339 expiry、compare-and-swap 寫回（SHA-256 防競態）、未知 JSON 欄位原樣保留、寫前備份 `.vn-prev`、0600 權限。`importAccount` 改以 agy token 檔為主來源並 best-effort 補 email、以 refresh_token 去重；新增每列「Apply to agy CLI」按鈕與 `applyAccountToAgyCLI`，切換時**只交付 refresh_token**（清空 access token、expiry 設為過期），逼 agy 下次啟動用它自己內嵌的 OAuth client 重新換發。**安全取捨**：切換依 agy 自身 client 重新換發憑證，VoidNotch 不持有任何 OAuth client secret。`AgyCLIOAuthBridgeTests` 13／13、`VoidNotchKitTests` 336／336。**已知限制**：CAS 為 TOCTOU（多次點擊未序列化）、寫入非完整交易、面板 per-account 即時用量尚未走 OAuth（依賴 agy CLI 當前登入帳號）。
- **Settings 可移除排程**：排程分頁每列垃圾桶鈕（僅使用者目錄 job；系統目錄需管理員權限、按鈕停用附提示）。確認對話框言明 VoidNotch 無法辨識排程由哪個 agent 建立；確認後 `launchctl bootout` 卸載並將 plist 改名 `_DELETE_<時間戳>_` 前綴封存——不刪檔、可於封存分頁回看。notch 面板維持唯讀。
- **軟體更新區塊常駐（Settings 的 System 分頁）**：有新版顯橫幅＋下載連結；無新版顯「已是最新版本（vX）」＋上次檢查相對時間；檢查中顯轉圈；永遠附「立即檢查」。dev 裸執行檔（無版號）自動停用檢查。
- **harness 分類器 token 化**：label 以非英數切 token 完整比對，第三方 label 含 mlx/lgd 子字串不再誤判徽章。
- **輪詢迴圈收斂 `PollingDriver`**：TokenStore／AgentActivityStore／LaunchdScheduleStore 三份手刻迴圈合一，公開 API 零改動。

- **Settings 視窗新增「排程」主分頁**：launchd 排程總覽不再只活在 notch dashboard——Settings 第三個主分頁完整顯示同一份 Run／Pause／Archive 清單（複用同一 view 與 store，零邏輯分歧）。
- **更新檢查模組**：啟動與每 24 小時向 `voidnotch.labgrimoire.com/downloads/version.json` 查詢最新版，semver 比對本機版號，有新版時在 Settings › System 分頁顯示提示與下載連結；網路失敗完全靜默、節流戳記成功後才寫（失敗下次啟動即重試）；端點可用 `VOIDNOTCH_UPDATE_API` 環境變數覆寫。純比對邏輯（前綴 v／`-dev` 後綴剝除、數字段比對、缺段補 0）落 `SemverCompare`，7 個單元測試。
- **修復死網域**：App 內 4 處 `voidnotch.com` 連結（含 notch 內開站連結）改指實際上線的 `voidnotch.labgrimoire.com`。
- **新增「Scheduled」dashboard 分頁：跨 agent harness 的 launchd 排程總覽**。展開面板第四個 widget（id `launchd-schedule`），以 Run／Pause／Archive 三個子 tab 區分運行中（已載入且未停用）、暫停（`Disabled` 或未載入）、封存（`_DELETE_` 前綴 plist，沿工作區退役慣例）的排程任務。掃描 `~/Library/LaunchAgents` 與 `/Library/LaunchAgents`，合併 `launchctl list` 的 PID／exit status，依 label 自動歸類 harness（Claude／Codex／Gemini／Grok／Hermes／VoidWeaver／oMLX／Other）並排序；每列顯示狀態圓點、label、harness 徽章、人讀排程摘要（`StartInterval`／`StartCalendarInterval` dict 與 array 雙型別／WatchPaths／KeepAlive／RunAtLoad／on-demand，中英雙語），懸停顯示完整指令與 plist 路徑。純邏輯（plist 解析、launchctl 解析、分類、排程格式化、掃描合併）落 `Sources/VoidNotchKit/Launchd/` 零 SwiftUI，15 個單元測試涵蓋 calendar 雙型別、weekday 0/7、封存三態、跨目錄去重等邊界；App 層 store 比照 `runTmux` 慣例跑 `/bin/launchctl`（固定絕對路徑＋2 秒 timeout），120 秒輪詢＋面板展開即時刷新。
- **浮動儀表可調大小**：新增 4 檔倍率（0.8／1.0／1.25／1.5，鍵 `VoidNotch.gauge.scale`，讀取時 clamp 0.5–2.0），右鍵選單「大小」submenu 與設定頁 Picker 皆可調。基準尺寸公式收斂到 `GaugeMetrics` 單一真相，content view 以基準尺寸渲染後 `scaleEffect`，skin 全體免改；panel 改尺寸與開機還原時 clamp 回所在螢幕 `visibleFrame`，放大不再被推出螢幕。
- **浮動儀表新增兩個外觀**：「圓環 Rings」（環形進度＋中央數值）與「玻璃 Glass」（`.ultraThinMaterial` 毛玻璃底＋細進度條），經 `GaugeSkinRegistry.register` 純加法接入。
- **外觀改真身預覽卡選擇器**：設定頁的文字下拉改為一排可點選的預覽卡，各卡以固定樣本資料（CPU 42%＋AI Claude）渲染該 skin 的真身縮小版，選中加高亮環——選外觀前即所見即所得，且預覽走 `makeView` 與 `GaugeSkinRegistry.all`，未來新增 skin 自動出現，零維護分歧。
- **AI 用量格顯示 provider 短名**：`TokenProviderKind.compactDisplayName`（≤7 字元）經新欄位 `DisplayReading.label` 傳至各 skin，標籤 `lineLimit(1)`＋`minimumScaleFactor` 保證不換行。
- **DisplayReading 明確 progress 語意**：新欄位 `progress: Double?`（0–1 正規化）只在真正百分比／score 指標（cpu／mem／disk／battery／gpu／health／AI 用量）設值；溫度、process 數、網路等非百分比指標為 nil，Rings／Glass 不再把 70°C 畫成 70% 進度環。
- **Agent Activity 來源導覽**（`b86aa51`）：事件可攜帶來源 surface 與 tmux socket／pane／window／session／client tty；點卡片可安全啟用 Ghostty／Terminal／iTerm／Claude Desktop／Codex App，tmux 來源會選取對應目標。只允許固定 app bundle ID、固定 tmux 路徑與格式驗證，不開 payload 任意 URL。
- **Agent event 語音**：可分別選擇朗讀 `completed`、`needsInput`、`failed`、`resourceLimit`；需要輸入時依題目實際文字選擇 zh-TW／en-US，且只朗讀受長度限制的安全文案。

- **Agent completed 事件中英 TTS 基線**（預設關閉，可於 Settings 開啟；後續已擴充為可選事件朗讀）：
  - **觸發範圍**：保留 Agent Activity 的 `completed` 朗讀相容行為；其他狀態是否朗讀由 Settings 個別選擇。
  - **語音選擇**：中文（zh-TW）與英文（en-US）voice **分開選擇**，各自具備自動 fallback（所選 voice 不可用時改用系統同語系可用 voice）。
  - **隱私邊界**：朗讀內容只含 **provider** 與 **title**；**不朗讀** `detail`／`workspace` 等可能含路徑或敏感上下文的欄位。
  - **資產邊界**：使用 macOS 內建 TTS（`AVSpeechSynthesizer`／系統 voice），**不打包**語音資產；與 Peon Audio「不重新散布第三方音檔」政策一致。

- **瀏海面板新增「接通狀態」診斷區塊**（Agent 活動分頁）：
  - 一眼看出每家 agent 的接通狀態。
  - **不隱瞞壞消息**：`conflict`（設定在、卻不會觸發）用橘色警示單獨標出，不混進「已接通」。標題旁的 badge 顯示還有幾家待處理。
  - 沒安裝的 agent 不列出——沒裝 codex 的人不需要看到一排紅字。

- **`HermesHookAdapter`**：偵測 hermes 的接通狀態。關鍵是能認出「config.yaml 掛好了、但沒進 allowlist」這個**看似接通、實則靜默不跑**的半死狀態（`hooks_auto_accept: false` 時未核准的 hook 會被略過）。只看 config 就判 installed 會謊報接通。hermes 無法一鍵接通（YAML 手工維護、重寫會沖掉註解），其 `plan()` 誠實拋錯而非用空 plan 假裝成功。

- **hermes 活動接入瀏海（只通知）**：
  - 新增 `hermes` provider（圖示 `wand.and.stars`／綠）。relay 認得 hermes 的 shell-hook 事件名並對應到五種狀態：`on_session_start`→started、`pre_llm_call`→running、`pre_approval_request`→needsInput、`api_request_error`→failed、`on_session_end`→completed。
  - 設定寫在 `~/.hermes/config.yaml` 的 `hooks:` 區塊，並在 `~/.hermes/shell-hooks-allowlist.json` 建立同意記錄（hermes 的 hook 需先進 allowlist 才會觸發，否則靜默不跑）。`hermes hooks doctor` 判定五個 hook 皆為 observer-only、每次約 30ms。

### Fixed

- **i18n 與語言設定**：
  - 建立以英文為預設語言的 i18n 架構，保留繁體中文切換。
  - Settings 的 Language 控制改為下拉選單。
- **PeonPing notch 提醒**：
  - Agent Activity 事件輪詢；首次載入只建立歷史基線，不彈出提醒。（間隔與短路策略見下方 Changed）
  - 後續事件以 UUID 去重；同一事件不會重複提醒。
  - `started`、`completed`、`needsInput`、`failed`、`resourceLimit` 會讓 notch 膨脹顯示提醒，約 3 秒後縮回 compact；`running`、`stopped` 僅保留於活動紀錄。
  - 色彩映射：開始為藍色、完成為綠色、需要輸入為橘色、失敗／資源限制為紅色。
- **本機 Peon 語音包**：
  - 執行時讀取本機 peon pack；預設位置為 `~/.claude/hooks/peon-ping/packs/peon`，亦可用 `VOIDNOTCH_PEON_PACK` 覆蓋。
  - Settings 新增 Peon Audio 開關。若原 PeonPing hook 已經播音，應關閉此開關以避免雙響。
  - VoidNotch 不重新散布 Warcraft／Blizzard 音檔；公開版不打包任何第三方語音。

### Changed

- Settings 的 widget 顯示與選單列項目改為自適應多欄格狀排列，保留切換與拖曳排序能力，提升寬視窗的空間使用效率。
- **Agent 事件輪詢改為「未變更即完全短路」**：新增 `AgentEventLogChangeDetector`，以 `(size, mtime)` 快照判斷事件檔是否變動；未變動時不讀檔、不解析、不觸碰 `isRefreshing`、不重指派 `events`。讀檔改為 `FileHandle` seek 尾端（不再 `Data(contentsOf:)` 整檔載入）。輪詢間隔 1s → 2s；`ProgressView` 僅於首次載入顯示。原本每秒全檔重讀＋逐行 JSON 重新解析（實測 169 KB／509 行），且每秒必觸發 `@Observable` 失效與一次 ProgressView 閃爍。
- **展開面板高度改為自然貼合**：移除「量測內容高度再回綁 ScrollView」的機制，改為 `.frame(maxHeight: scrollMaxHeight)` + `fixedSize`。原機制在切分頁時把量測值歸零，會退回半螢幕高度造成大片垂直死白，且量測迴圈可能永久鎖死在 0。
- **Agent 彈窗改用半透明材質**（`.ultraThinMaterial` + tint／contrast overlay）。註：真正的桌面穿透受限於 DynamicNotchKit 的不透明黑底。
- 模組化：`ProviderSettingsView` 1296→301 行、`NotchShell` 824→409 行、`CodexBarTokenAccountManager` 811→503 行（純機械搬移，逐行零改動）。`L10n`／`NotchCompactLayoutStore`／`PeonPingAgentActivityProvider`／`NotchPreferenceKeys`／`PeonSoundPack` 下沉至 `VoidNotchKit`，使其進入單元測試覆蓋範圍（`App/` 原本零測試覆蓋）。

### Fixed

- 補上 Apple Silicon `PMU tdie` 溫度感測器辨識，修復 M4 Pro 的 Temp 無法取得問題；不會誤收校正、NAND 或電池溫度。
- 修正 Xcode App target 的 Swift source membership，讓本次新增的 App 原始碼納入 target。
- **浮動儀表開啟後閃退**（約 60–90 秒必發）：`NSHostingView` 直接作為 borderless `NSPanel` 的 `contentView`，會依 `sizingOptions` 把 SwiftUI 內容尺寸回寫成視窗約束；儀表內容隨資料更新改變寬度，形成「內容改尺寸→改視窗約束→觸發 layout→內容再改尺寸」正回饋迴圈，AppKit 約束更新次數超過視窗內 view 數即擲例外 SIGTRAP。修法：`sizingOptions = []` 切斷回寫、host 以 autoresizing 單向填滿 panel、panel 寬度依項目數推導；並修掉 `UserDefaults.didChangeNotification → applyEnabledState() → show()` 的通知風暴（observer 累積、`persistFrame()` 無條件寫入）。
- **`Int(NaN)` 直接 trap**：`clampedPercent` 的 clamp 套用在 `Int()` 轉換之後，無配額帳號（0/0）算出 NaN 時直接 crash。
- **agent 事件在大 log 下全部消失**：位元組截斷落在多位元組 UTF-8 字元中間時，整批解碼失敗回傳空陣列。
- **`seenEventIDs` 無界成長**：以 `union` 累加，過期事件 UUID 永不修剪。
- **選集靜默重置**：`DisplaySelectionStore` encode 失敗時 `defaults.set(nil)` 會移除 key。
- **Menu bar items 勾選失效**（兩個獨立根因）：① 已選項目的 `.draggable` 包住整列、攔截 checkbox 點擊，取消勾選無效；② `statusItem` 寬度於建立當下凍結，選集變更不重算，新項目被裁切在按鈕外。
- **設定頁與儀表右鍵選單互相覆蓋**：`DisplayItemsEditor` 的 `@State` 為一次性快照。
- **agent 彈窗關閉時閃現完整 dashboard**：收合前先清 `agentAlertState.event`，畫面立刻 fallback 到展開面板才播動畫。
- **agent 彈窗擋住滑鼠點擊且可能永久卡住**：`.expanded` 時整個半螢幕視窗關閉穿透（命中區已縮至彈窗本體）；`transition(to:)` 遇 `isTransitioning` 會靜默丟棄請求，導致收合遺失、穿透永不恢復。改為 pending 排隊消耗最新目標。
- **HookInstaller 的回滾承諾是假的**：`.writeJSON` 覆寫既有檔時未登記 undo，回滾僅靠 adapter 自律加 `.backup` 兜住。新增 `.restoreContents` undo。
- **帳號停用狀態持久化失敗被 `try?` 吞掉**：導致下次啟動該帳號再被拿去打 API、再吃一次 ban。改走 OSLog（token／憑證標 `privacy: .private`）。
- **i18n 缺口**：英文介面下 Appearance 下拉顯示「數碼管」、Alerts 分頁整區為中文。`GaugeSkin.displayName` 改為吃語言；HookWiringSettingsView／GaugeController 右鍵選單／MenubarSummaryView／NotchShell banner／VoidNotchApp 選單全面收攏進 `L10n`。反向補齊 `SystemMetricKind.label(language:)`（繁中介面原本全是英文）。
- **pi 事件全部重複寫入兩次**：`~/.pi/agent/extensions/_DELETE_voidnotch.ts` 這種「加 `_DELETE_` 前綴軟刪除」的慣例對 pi 無效——pi 的 loader（`isExtensionFile`）只看副檔名 `.ts`/`.js`，不看檔名前綴，殘檔照樣被載入並重複註冊 handler。實測 210 筆 pi 事件全是 105 組 ×2。停用 pi extension 必須改副檔名或移出該目錄。

### Verification

- 2026-07-16：`b86aa51`（`feat(agent): improve activity prompts and navigation`，12 個檔案，+1510/−88）結案證據——`relay_test` ALL PASS；Agent display 5/5、new-event 7/7、speech 34/34；`swift test` **335 tests、0 failures**；`vn-selftest` **33/33**；SwiftPM 與 Xcode build 成功。
- 2026-07-16：以本提交內容執行 `scripts/make_app.sh --install`，建置 Release、ad-hoc 簽章並安裝 `/Applications/VoidNotch.app`（版本 0.7.0、build 74、PID 75768）；簽章驗證通過、麥克風用途說明存在。**此安裝非正式發布。** 提交前主線重跑 `swift test` **335 tests、0 failures**；先前 `vn-selftest` **33/33**、Xcode Debug `BUILD SUCCEEDED`。
- 2026-07-15：Agent completed 中英 TTS 落地後驗證——`swift test` **304 tests、0 failures**；`vn-selftest` **33/33**；`swift build --product VoidNotch` 成功；Xcode Debug `BUILD SUCCEEDED`。**未**做真機聽感驗收（語音可懂度、延遲、與 Peon Audio 並存手感仍待有瀏海 MacBook 人工確認）。
- 2026-07-11：`swift test` 159 tests、0 failures；`xcodebuild -quiet -project VoidNotch.xcodeproj -scheme VoidNotch -configuration Debug -destination platform=macOS build` exit 0；App bundle 的 `Resources/hooks` 路徑已確認。`vn-selftest` 既有 33 項驗證通過；有瀏海 MacBook 真機提醒動畫／音訊仍待驗收。
- 2026-07-12：`swift test` **201 tests、0 failures**（自 159 起新增 42 項，含 path-traversal 拒絕、UTF-8 截斷邊界、HookInstaller 回滾還原原始內容、`.en` 語系不得含 CJK 字元等回歸鎖）；`swift build` 與 `xcodebuild ... build` 皆 `BUILD SUCCEEDED`；`Sources/VoidNotchKit` 無 SwiftUI／AppKit import（純邏輯層）。
- 2026-07-12：閃退修復以實跑取證——修復前約 80 秒必死、4 分鐘內 2528 次 AppKit `DisplayCycle` 迴圈警告；修復後連續存活 240 秒、`DisplayCycle` 警告 0 次、約束例外 0 次、RSS 平穩。
- **尚未驗收**：menubar 勾選、面板留白、彈窗閃現與透明度等視覺行為僅通過建置與單元測試，未經真機截圖逐項確認。

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

### Changed

- 測試改 XCTest，完整 Xcode toolchain 下 `swift test` 可跑
- v1 當時範圍：Notch + CPU/RAM/溫度；Token 列為後續（後於 0.2.0 產品化）

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
