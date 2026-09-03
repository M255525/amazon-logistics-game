# CLAUDE.md — amazon-logistics-game（跨境物流大挑戰）

依使用者提供的兩張 Amazon 跨境電商「物流費用」教材截圖（來源：亞馬遜全球開店 Facebook）改編的兩關互動教學遊戲。單檔前端，無建置步驟、無框架、零外部資源，直接開啟 `index.html`（`file://`）或以靜態伺服器託管即可。此資料夾本身是獨立 git 儲存庫。

## 原始教材內容（忠實還原，未竄改）

- **流程圖**：賣家 →（出口報關）→ 分岔「空運／海運」→ 匯流「進口清關關稅」→「貨運」→ 分岔「FBA倉／FBM」→ 匯流「買家」。出口報關～貨運段框在「頭程物流」；FBA倉／買家段框在「美國」。另有一條獨立紅色箭頭直接從「賣家」繞到「買家」、箭頭上貼著「FBM」標籤，代表 FBM 可不經頭程物流直接送達——遊戲中以第一關全對後解鎖的紅色虛線 SVG 曲線呈現，曲線中點放一個「FBM」文字標籤（`#fbmLabelGroup`），視覺上讀成「賣家 → FBM → 買家」，忠於原圖箭頭貼標籤的畫法（2026-09-03 應使用者要求把 FBM 直接標在路徑上，不再只靠旁邊的文字說明）。
- **運輸方式定義**（三選一情境題的判分依據與回饋文字）：國際商業快遞（UPS/DHL/FedEx，到達目的國後仍需當地配送）、海運（整箱/拼箱，20 立方米為門檻）、空運（飛機運輸）。

## 遊戲設計

**第一關「物流流程拼圖」**：9 張可拖曳卡片（賣家/出口報關/空運/海運/進口清關關稅/貨運/FBA倉/FBM/買家），DOM 用「主軸列＋並列分岔列」重現分岔匯流結構（`.zone-headhaul` 橘框、`.zone-us` 紫框、`.flow-branch` 讓空運/海運與FBA倉/FBM並列）。9/9 全對後用 `drawFbmShortcutPath()` 即時算出賣家/買家 slot 座標，動態畫一條紅色虛線 SVG 貝茲曲線＋提示文字說明 FBM 捷徑。

**空格不顯示文字提示（2026-09-03 應使用者要求移除）**：原版每個空格內有答案文字（如「賣家」「出口報關」），使用者指出這樣卡片文字達不到練習效果，玩家只需比對文字就能過關。已移除所有 `.slot` 內的文字，空格只剩虛線框；玩家必須理解流程邏輯才能作答，卡片本身的文字才是唯一資訊來源。**連帶影響判分邏輯**：拿掉文字後，橘框內的空運/海運兩個並列空格、紫框內的 FBA倉/FBM 兩個並列空格，視覺上左右對稱、玩家無從得知哪個空格對應哪張卡（現實中這兩者本來就沒有固定順序），因此改用 `BRANCH_GROUPS = [['air','sea'], ['fba','fbm']]` 做**群組判分**——只要求這對空格裡裝的是「這兩張卡」的正確集合，不比對誰在左誰在右；其餘 5 個單一空格（賣家/出口報關/進口清關關稅/貨運/買家）仍要求精確對應位置的 exact match，因為線性流程順序才是實際的學習目標。

**FBM 捷徑曲線的 RWD 修法（2026-09-03）**：`drawFbmShortcutPath()` 原本只有「往下方 bow」一種畫法，desktop 版賣家/買家水平相鄰所以沒問題；但 `@media(max-width:760px)` 斷點把 `.flow-board` 改成直排（`flex-direction:column`），賣家在最上、買家在最下，兩者垂直距離可能長達整個流程圖高度，同一套公式會畫出一條貫穿整頁的近乎直線並蓋住底下手牌區，加上新增的 FBM 標籤文字直接跑到版面外看不到。已改成偵測 `getComputedStyle(flowBoard).flexDirection`：直排時改成「往右側 bow」，整條曲線與標籤都畫在 `flowBoard` 自身的寬度範圍內（`svg` 改成 `left:0;top:0` 覆蓋整個 flow-board，而非 `top:100%` 往下延伸），賣家→買家的路徑沿右側繞過中間所有 slot；橫排時維持原本「往下方 bow」的畫法。兩種模式共用 `bezierMid()`／`placeFbmLabel()` 算 FBM 標籤在曲線 t=0.5 的位置。`.flow-wrap.has-shortcut` 的多餘 `padding-bottom`（原本是為了容納往下延伸的曲線）改成只在橫排模式才加上，直排模式不需要、加了只是浪費空間。`window resize` 監聽也一併判斷模式切換 `has-shortcut` class，避免跨斷點縮放視窗時版面卡住舊狀態。

**第二關「運輸方式情境選擇」**：`LEVEL2_QUESTIONS` 6 組情境題（三選一：快遞/海運/空運），其中 q3/q4 專測海運 20 立方米整箱/拼箱門檻。每題作答後立即鎖定並顯示忠於原教材定義文字的回饋卡，不論對錯都顯示完整定義（教學導向，不只是判對錯）。

**計分（2026-09-03 應使用者要求改為 100 分制）**：`computeScore()` 把第一關按 9 格中答對比例換算成 60 分（`Math.round(level1Correct/9*60)`），第二關按 6 題中答對比例換算成 40 分（`Math.round(level2Correct/6*40)`），加總為總分。兩者權重取 60/40 是因為第一關題目數（9）多於第二關（6），四捨五入在兩關都拿滿分時精確等於 100（其餘組合不見得整除，但這是常見計分慣例可接受），因此「總分 100」與「兩關都全對」互為充要條件，恭賀畫面/彩帶觸發條件維持 `total===100` 不受影響。結算頁同時顯示每關的「原始格數/題數」與換算後的「分數」兩種資訊。**不做「暫停」功能**（比照 `bowling-game`／`boxing-cam` 的教學小遊戲從簡定位，非 `crispe-game` 的多主題重複挑戰模式，暫停/凍結計時的需求較低）。

## 與 crispe-game 的沿用與差異

拖放引擎（Pointer Events、`makeDraggable()`/`forceCancelDrag()`/`slotUnderPoint()`、mobile 觸控自我修復）、WebAudio 音效合成（`sfx.place/good/bad/perfect/fanfare`）、本機排行榜骨架（`localStorage` 前 10 名，`lbAddEntry`/`renderLeaderboard`）、彩帶、跑馬燈 IIFE、PWA 安裝 IIFE、成績卡 Canvas 繪製/下載/分享（`buildResultCanvas()`）皆逐字/近似複製 `互動遊戲/crispe-game/index.html` 的已驗證骨架。

**差異**：本遊戲是**單線性兩關**（非 crispe-game 的「5選20主題可任意重挑戰」），因此**不做狀態持久化**（`S` 只存在記憶體，重新整理頁面即重置整場遊戲進度；只有排行榜走 `localStorage`）——比 crispe-game 簡單，因為沒有跨關卡導覽或中途離開再回來的需求。第一關 slot 佈局改為分岔匯流（crispe-game 是單列 5 格），判定邏輯改用 `card.dataset.card === slot.dataset.key` 對應 `#flowBoard .slot`（非 `.slot-row`）。

## 身分欄位、排行榜與成績卡（2026-09-03 應使用者要求新增）

結算畫面新增 `#identityPanel`（姓名/學號/科系三個選填文字欄位，取代原本單純的「暱稱」），存讀走 `IDENTITY_KEY='logisticsGameIdentity'` 的 `localStorage` 草稿（`loadIdentityDraft()`/`saveIdentityDraft()`），填過一次下次自動帶入。三個按鈕互相獨立、不互相依賴：

- **🏆 記錄成績到排行榜**：`lbAddEntry(name, studentId, dept, score, timeMs)` 把身分欄位一併存進 `LB_STORAGE_KEY='logisticsGameLeaderboard'`；改用「先加入陣列排序、再檢查該筆是否還在前 10 名內」（`madeCut`）取代原本的 `lbQualifies()` 事先判斷，因此按鈕**一律可按**（原本只有預估擠得進前 10 名才顯示輸入框的邏輯已移除，使用者要求能隨時輸入身分資訊）；已記錄過會鎖住按鈕改顯示「✅ 已記錄成績」避免重複計入。排行榜表格（`renderLeaderboard()`）新增「學號」「科系」欄，7 欄較寬，外層加 `.table-wrap{overflow-x:auto}` 因應窄螢幕。
- **💾 下載成績卡**：`buildResultCanvas()`（Canvas API 手繪，不依賴外部庫，符合工作區「零外部資源」原則）畫一張深色卡片，含姓名/學號/科系、兩關個別得分（X/60、Y/40）、總分（100 分制，滿分變琥珀色）、總時間、創作者字樣；`canvas.toBlob()` + `<a download>` 觸發下載。
- **📤 分享成績卡**：`initShareButton()` 偵測 `navigator.share`/`navigator.canShare` 才顯示按鈕（不支援的瀏覽器維持隱藏，只能下載），呼叫方式與 `crispe-game` 的 `buildResultCanvas`/分享邏輯同構但改成本遊戲自己的畫面內容。
- 下載與分享**不要求**先記錄到排行榜，三者互不阻擋——manual.html 已明確說明這點。

## 視覺主題

深色底＋Amazon 琥珀金 amber 強調色（沿用 `行銷內容工具/amazon-listing-generator` 的 `:root` 變數：`--bg:#0b0d16;--accent:#f59e0b`），疊加 `--zone-us:#8b7cf6` 紫框呼應原簡報「美國」框色，三種運輸方式卡片配色呼應圖2原色（快遞綠 `--express:#34d399`／海運藍 `--sea:#38bdf8`／空運橘 `--air:#fb923c`）。PWA icons 用 PIL 手繪（深色圓角方塊＋琥珀色貨櫃船剪影，呼應「跨境物流」主題，非外部素材）。

## 功能配套範圍

- ✅ 頂部跑馬燈（沿用工作區共用 Google Sheet Apps Script 端點，`localStorage` key `logisticsGameMarquee`）
- ✅ PWA 加入主畫面（`manifest.json` + `service-worker.js` + `icons/`，九專案共用已驗證版本逐字複製；`manifest.json` 的 `SHELL_FILES` 已補上 `manual.html`）
- ✅ `manual.html`（2026-09-03 應使用者要求補上，比照 `crispe-game` 版型改琥珀色系；topbar 新增「📖 操作手冊」`<a class="ctl-btn">` 連結，`target="_blank"` 避免跳走弄丟進行中的計時/拖曳狀態）
- ✅ 訪客計數器（2026-09-03 應使用者要求補上，`visitor-badge.laobi.icu`，`page_id=m255525.amazonlogisticsgame`，放在開場畫面 `.footnote` 使用警語＋創作者名字下方，逐字比照工作區既有慣例）
- ❌ **仍未做**：序號授權、exe 打包——教學小遊戲定位（比照 `bowling-game`／`boxing-cam`），非對外發布的商業工具；也**未部署 GitHub Pages**（本次任務範圍僅限本機建置，未經使用者要求上線）。

## Port

固定用 **8810**（工作區下一個可用埠，8765-8809 已被其他專案佔用）。

## 指令

無建置/測試指令。修改 `index.html` 後直接用瀏覽器開啟驗證，或用 Preview MCP／`python -m http.server 8810 --directory 互動遊戲/amazon-logistics-game` 暫起伺服器測完關閉。自動化驗證建議用 Playwright `browser_evaluate` 派發 PointerEvent 模擬拖曳（比照 crispe-game 慣例，關鍵流程放在單一 evaluate 內完成避免真人操作干擾狀態斷言）。
