# CLAUDE.md — amazon-logistics-game（跨境物流大挑戰）

依使用者提供的兩張 Amazon 跨境電商「物流費用」教材截圖（來源：亞馬遜全球開店 Facebook）改編的兩關互動教學遊戲。單檔前端，無建置步驟、無框架、零外部資源，直接開啟 `index.html`（`file://`）或以靜態伺服器託管即可。此資料夾本身是獨立 git 儲存庫。

## 原始教材內容（忠實還原，未竄改）

- **流程圖**：賣家 →（出口報關）→ 分岔「空運／海運」→ 匯流「進口清關關稅」→「貨運」→ 分岔「FBA倉／FBM」→ 匯流「買家」。出口報關～貨運段框在「頭程物流」；FBA倉／買家段框在「美國」。另有一條獨立紅色箭頭直接從「賣家」繞到「買家」標示「FBM」，代表 FBM 可不經頭程物流直接送達——遊戲中以第一關全對後解鎖的紅色虛線 SVG 曲線＋提示文字呈現。
- **運輸方式定義**（三選一情境題的判分依據與回饋文字）：國際商業快遞（UPS/DHL/FedEx，到達目的國後仍需當地配送）、海運（整箱/拼箱，20 立方米為門檻）、空運（飛機運輸）。

## 遊戲設計

**第一關「物流流程拼圖」**：9 張可拖曳卡片（賣家/出口報關/空運/海運/進口清關關稅/貨運/FBA倉/FBM/買家），DOM 用「主軸列＋並列分岔列」重現分岔匯流結構（`.zone-headhaul` 橘框、`.zone-us` 紫框、`.flow-branch` 讓空運/海運與FBA倉/FBM並列）。9/9 全對後用 `drawFbmShortcutPath()` 即時算出賣家/買家 slot 座標，動態畫一條紅色虛線 SVG 貝茲曲線＋提示文字說明 FBM 捷徑。

**第二關「運輸方式情境選擇」**：`LEVEL2_QUESTIONS` 6 組情境題（三選一：快遞/海運/空運），其中 q3/q4 專測海運 20 立方米整箱/拼箱門檻。每題作答後立即鎖定並顯示忠於原教材定義文字的回饋卡，不論對錯都顯示完整定義（教學導向，不只是判對錯）。

**計分**：不硬湊 100 分（9+6=15 題不整除），結算頁顯示「第一關 X/9」「第二關 Y/6」「總計 X+Y/15」＋總時間，15/15 全對觸發彩帶 + `sfx.fanfare()`。**不做「暫停」功能**（比照 `bowling-game`／`boxing-cam` 的教學小遊戲從簡定位，非 `crispe-game` 的多主題重複挑戰模式，暫停/凍結計時的需求較低）。

## 與 crispe-game 的沿用與差異

拖放引擎（Pointer Events、`makeDraggable()`/`forceCancelDrag()`/`slotUnderPoint()`、mobile 觸控自我修復）、WebAudio 音效合成（`sfx.place/good/bad/perfect/fanfare`）、本機排行榜（`localStorage` 前 10 名，`lbQualifies`/`lbAddEntry`/`renderLeaderboard`）、彩帶、跑馬燈 IIFE、PWA 安裝 IIFE 皆逐字/近似複製 `互動遊戲/crispe-game/index.html` 的已驗證骨架。

**差異**：本遊戲是**單線性兩關**（非 crispe-game 的「5選20主題可任意重挑戰」），因此**不做狀態持久化**（`S` 只存在記憶體，重新整理頁面即重置整場遊戲進度；只有排行榜走 `localStorage`）——比 crispe-game 簡單，因為沒有跨關卡導覽或中途離開再回來的需求。第一關 slot 佈局改為分岔匯流（crispe-game 是單列 5 格），判定邏輯改用 `card.dataset.card === slot.dataset.key` 對應 `#flowBoard .slot`（非 `.slot-row`）。

## 視覺主題

深色底＋Amazon 琥珀金 amber 強調色（沿用 `行銷內容工具/amazon-listing-generator` 的 `:root` 變數：`--bg:#0b0d16;--accent:#f59e0b`），疊加 `--zone-us:#8b7cf6` 紫框呼應原簡報「美國」框色，三種運輸方式卡片配色呼應圖2原色（快遞綠 `--express:#34d399`／海運藍 `--sea:#38bdf8`／空運橘 `--air:#fb923c`）。PWA icons 用 PIL 手繪（深色圓角方塊＋琥珀色貨櫃船剪影，呼應「跨境物流」主題，非外部素材）。

## 功能配套範圍（刻意從簡，比照遊戲類別而非商業工具）

- ✅ 頂部跑馬燈（沿用工作區共用 Google Sheet Apps Script 端點，`localStorage` key `logisticsGameMarquee`）
- ✅ PWA 加入主畫面（`manifest.json` + `service-worker.js` + `icons/`，九專案共用已驗證版本逐字複製）
- ❌ **未做**：`manual.html`、訪客計數器、序號授權、exe 打包——教學小遊戲定位（比照 `bowling-game`／`boxing-cam`），非對外發布的商業工具；也**未部署 GitHub Pages**（本次任務範圍僅限本機建置，未經使用者要求上線）。

## Port

固定用 **8810**（工作區下一個可用埠，8765-8809 已被其他專案佔用）。

## 指令

無建置/測試指令。修改 `index.html` 後直接用瀏覽器開啟驗證，或用 Preview MCP／`python -m http.server 8810 --directory 互動遊戲/amazon-logistics-game` 暫起伺服器測完關閉。自動化驗證建議用 Playwright `browser_evaluate` 派發 PointerEvent 模擬拖曳（比照 crispe-game 慣例，關鍵流程放在單一 evaluate 內完成避免真人操作干擾狀態斷言）。
