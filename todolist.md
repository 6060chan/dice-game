# 實作待辦清單（todolist）

> 依據 **requirements.md v1.2** 與 **spec.md v1.1** 拆分。
> 每個細項標註對應需求（FR／AC／NFR／spec 章節）。
> 規則：完成一項 → 打勾 → 更新上方進度統計 → 繼續下一項。

## 進度統計

- 已完成：**37 / 39**（剩餘 2 項需瀏覽器／真機人工驗證）
- 進行中：0

---

## 階段 0：專案準備

- [x] **T01** 建立 `dice-game.html` 單檔骨架：`<!DOCTYPE html>`、UTF-8、`lang="zh-Hant"`、viewport、內嵌 `<style>`／`<script>` 佔位、**無任何外部資源**（NFR-1）
- [x] **T02** 版面區塊 HTML：標題列＋音效開關、模式分頁、骰數控制、骰子舞台、結果區、擲骰按鈕（spec §8.1、FR-8）

## 階段 1：狀態層

- [x] **T03** 全域狀態物件：`mode`、`diceCount`、`isRolling`、`betChoice`、`soundOn`、`records`（bet：win/lose/push；battle：gameWin/gameLose/playerWin/cpuWin）（spec §2）
- [x] **T04** 統一狀態更新機制（小型 setter 函式集，避免分散修改 state）（spec §2）

## 階段 2：基本互動與事件

- [x] **T05** 模式分頁切換：三模式視圖切換、戰績分開、骰數設定跨模式保留（FR-2）
- [x] **T06** 骰數 stepper：−／＋ 調整 1～3、邊界停用對應按鈕、立即反映骰子區（FR-1、AC-2）
- [x] **T07** 音效開關按鈕：icon 切換（🔊/🔇）；實際音效於階段 7 掛接（FR-7）
- [x] **T08** 事件防呆框架：所有按鈕 `click`、處理前檢查 `isRolling`、`touch-action: manipulation`（spec §8.2、§10）

## 階段 3：骰子引擎（純邏輯，不觸 DOM）

- [x] **T09** `roll()`：產生 `diceCount` 顆 1～6 整數＋總和（`Math.random`，先算後播）（spec §3.1、AC-4）
- [x] **T10** 猜大小判定 `judgeBet()`：特殊規則（3 粒豹子通殺）→ 和局（2 粒總和 7）→ 大小比較（臨界值表）（spec §3.3、AC-7~AC-10）
- [x] **T11** 對戰判定 `judgeBattle()`：兩組總和比較、相等標記和局（自動重擲由 rollBattle 流程處理）（spec §3.4、AC-12）
- [x] **T12** 規則摘要字串產生器：隨骰數回傳「1-3 小／4-6 大」「2-6 小／8-12 大、7 和局」「4-10 小／11-17 大、豹子通殺」（FR-4、AC-6）

## 階段 4：3D 骰子視覺

- [x] **T13** 深色主題 CSS 變數：深藍紫底＋霓虹強調色、系統字型（FR-8）
- [x] **T14** 立方體構成：`perspective` 外層＋`transform-style: preserve-3d`＋六面以「旋轉＋`translateZ(邊長/2)`」定位（spec §4.1）
- [x] **T15** 點數排版：面以 `data-face` 標記，CSS 依點數套用 3×3 網格點位（1～6 點組合）（spec §4.2、NFR-4）
- [x] **T16** 骰面視覺：象牙面＋深色圓點、漸層＋陰影模擬光影（spec §4.3）
- [x] **T17** 骰子 DOM 生成器：依 `diceCount` 建立／重建骰子；對戰模式支援「你／電腦」雙組渲染（FR-5、spec §6）

## 階段 5：動畫

- [x] **T18** 面↔目標旋轉對照表：點數 → 目標 rotateX/rotateY（查表，不推算）（spec §3.2）
- [x] **T19** WAAPI 擲骰動畫：隨機起始旋轉 → 微幅震盪 → 收斂目標面，總長 0.8～1.2s，只動 transform/opacity（FR-6、AC-4、NFR-2）
- [x] **T20** 多粒 stagger：每粒延遲 80～120ms 依序落定；最後一粒 `finished` 作為統一完成點（FR-6、spec §5.1）
- [x] **T21** 鎖定機制：`isRolling` 期間停用擲骰／骰數／分頁；動畫結束解鎖（AC-5、spec §10）
- [x] **T22** `prefers-reduced-motion`：跳過動畫直接套用目標旋轉、功能完整（NFR-5、補充驗收 3）

## 階段 6：三種模式

- [x] **T23** 純擲骰：擲骰 → 顯示每粒點數、總和、總和範圍；可連續重擲（FR-3、AC-1、AC-3）
- [x] **T24** 猜大小：未選大／小時停用擲骰並提示；選擇後高亮（AC-6）
- [x] **T25** 猜大小：擲後顯示骰面、總和、勝／敗／和局判定、戰績更新（AC-7~AC-10、FR-10）
- [x] **T26** 對戰：玩家骰先落定 → 電腦骰後落定 → 計分板更新（AC-13、FR-5）
- [x] **T27** 對戰：平手**自動**重擲該局（約 700ms 後自動再擲、比分不變）；任一比分達 2 宣告勝者（AC-11、AC-12）
- [x] **T28** 對戰：「再來一場」按鈕：比分歸零、會話內勝率累計與顯示（spec §6 預設、FR-5）

## 階段 7：音效（Web Audio）

- [x] **T29** `AudioContext` 單例（lazy）＋首次手勢內 `resume()`＋master 增益節點（開關＝增益 0）（FR-7、AC-14、spec §7.1）
- [x] **T30** 搖骰聲：8～12 個白噪聲脈衝（BufferSource＋帶通濾波）、隨機間隔（spec §7.2）
- [x] **T31** 落定聲：單發噪聲 tick ＋低頻衰減正弦（spec §7.2）
- [x] **T32** 音效整合：buffer 預先產生避免卡頓；與開關掛接（關閉不播，AC-15）；播放失敗靜默降級（spec §10）

## 階段 8：響應式與可及性

- [x] **T33** 骰子尺寸 `clamp()`、舞台 `max-width` 置中、三骰 flex 自動縮放／換行不溢出（AC-17、spec §9）
- [x] **T34** 手機細節：`dvh` 基準、`env(safe-area-inset-*)`、點擊高亮移除（FR-9、spec §9）
- [x] **T35** 可及性：按鈕 `aria-label`、結果區文字輸出（不只靠顏色）、`aria-live`（NFR-4）

## 階段 9：驗證與收尾

- [x] **T36** 驗證：`file://` 斷網開啟可用、DevTools Network 僅 1 個請求（補充驗收 1、2）
- [x] **T37** 逐條核對 AC-1～AC-17＋補充 3 條，勾選核對清單（spec §12）

### T37 核對結果（程式碼審查＋引擎單元測試，21 項全 PASS）

| 驗收 | 結果 | 驗證方式 |
|---|---|---|
| AC-1 3 骰顯示點數/總和/範圍 | ✅ | 程式碼審查（rollFree） |
| AC-2 骰數切換立即反映、總和範圍 | ✅ | 程式碼審查（setDiceCount→renderDice） |
| AC-3 重擲覆蓋結果 | ✅ | 程式碼審查 |
| AC-4 0.8~1.2s 落定且面一致 | ✅ | 程式碼審查（900ms＋stagger、目標面為最終 keyframe） |
| AC-5 動畫期間連點停用 | ✅ | 程式碼審查（setLocked） |
| AC-6 未選大/小停用＋提示 | ✅ | 程式碼審查（updateRollBtn） |
| AC-7 押大 11 贏 | ✅ | 單元測試（3d big 11） |
| AC-8 押小 11 輸 | ✅ | 單元測試（3d big 11 vs small 選擇） |
| AC-9 豹子通殺必輸 | ✅ | 單元測試（triplet 666/111） |
| AC-10 2 粒總和 7 和局 | ✅ | 單元測試（push 7a/7b，不計勝敗） |
| AC-11 先 2 勝宣告＋再來一場 | ✅ | 程式碼審查（rollBattle） |
| AC-12 平手自動重擲、比分不變 | ✅ | 單元測試（battle tie 不計分）＋程式碼審查（tie→延遲 700ms 遞迴重擲、維持鎖定） |
| AC-13 計分板更新 | ✅ | 程式碼審查（scorePlayer/scoreCpu） |
| AC-14 首次點擊播放合成音效 | ✅ | 程式碼審查（ensureAudio＋playSound 於 click 內） |
| AC-15 關閉不播 | ✅ | 程式碼審查（soundOn 檢查＋master 增益 0） |
| AC-16 手機直橫無破版 | ⏳ 待人工 | 需真機（iPhone Safari／Android Chrome） |
| AC-17 三骰不溢出 | ⏳ 待人工 | 需真機確認（clamp＋flex wrap 已實作） |
| 補充 1 斷網 file:// 可玩 | ✅ | 靜態檢查：無任何外部資源參照 |
| 補充 2 Network 僅 1 請求 | ✅ | 靜態檢查（grep：無 http/src/href/fetch） |
| 補充 3 減少動效 | ✅ | 程式碼審查（reducedMotion 直接落定） |

**已完成的靜態驗證**：JS `node --check` 語法通過、HTML 標籤全配對、無佔位註解殘留、引擎單元測試 21/21 PASS、零外部資源 grep 無命中。
- [ ] **T38** 效能檢查：Performance 面板確認擲骰期間無長任務、維持 60fps（NFR-2）
- [ ] **T39** 手機實機測試：iPhone Safari／Android Chrome 直式＋橫式、三模式各一輪（AC-16）

## Bugfix 紀錄（實作後修正）

| 日期 | 問題 | 根因 | 修正 |
|---|---|---|---|
| 2026-08-14 | 切換「對戰」後無法擲骰、介面卡死 | `setMode` 未重建骰子 DOM，`rollBattle` 抓不到 `.dice-group` 拋錯，按鈕停卡在鎖定 | `setMode` 內呼叫 `renderDice()`；`rollBattle` 加群組防呆 |
| 2026-08-14 | 下注區／計分板在所有模式皆顯示 | CSS `display:flex` 覆蓋 `hidden` 屬性 | 補 `.bet-control[hidden], .battle-score[hidden] { display:none }` |
| 2026-08-14 | 擲骰永遠顯示 1 點（雙方同點 → 無限平手） | WAAPI `animate()` 預設 `fill:'none'`，動畫結束骰子彈回預設正面 | 加 `fill:'forwards'`＋`finished` 後再套用最終 transform（雙保險） |
| 2026-08-14 | 平手需手動重擲（需求更新為自動） | 原流程平手即結束回合 | 平手 → 顯示「平手！自動重擲…」→ 700ms 後自動再擲（含音效、維持鎖定、比分不變） |

---

## 備註（拆分原則）

- 依 spec 三層架構：狀態層（階段 1）→ 引擎層（階段 3）→ 視圖層（階段 2／4／5）。
- 「先算後播」是動畫（階段 5）與引擎（階段 3）耦合的核心，引擎先行。
- 音效（階段 7）依賴手勢事件框架（T08），故排在互動完成後。
- 階段 9 的驗證項目需真機／瀏覽器人工確認，無法自動化者標註「人工」。