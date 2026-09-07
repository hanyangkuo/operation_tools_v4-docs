# 04: Frontend 資料互動 — 表單 + 表格 + Filter + 分頁 + Snackbar

**What to build:** 選擇 DB 後，主區域顯示兩張 Card。上方是 Form Card（左邊 4px 藍色邊框），三個欄位（群組名稱、帳號、密碼）水平排列加一個送出按鈕。下方是 Table Card，header 有標題 + 筆數 Chip + 搜尋框，body 是 MUI 基礎 Table（群組名稱用 Chip 樣式、帳號 monospace、創建者、創建時間），底部有分頁（每頁 20 筆）。

表單前端驗證：空值和長度超過 100 時欄位下方顯示紅字 inline error。送出時呼叫 `api/index.ts` 的 `createGroup`（仍為 mock）。成功：綠色 Snackbar「新增成功」+ 清空表單 + 自動 re-fetch 列表。失敗（模擬重複）：紅色 Snackbar 顯示錯誤訊息。搜尋框可即時 filter 群組名稱 + 帳號。

所有功能使用 Ticket 03 建立的 mock data 可獨立驗證，不需要後端服務。

**Blocked by:** 03-frontend-scaffold-sidebar

**Group:** FRONTEND

**Status:** ready-for-agent

- [ ] `AccountForm.tsx` — MUI Card + 左藍邊框，Stack direction="row"，3 個 TextField + Button
- [ ] 前端驗證：空值 → inline error，長度 > 100 → inline error
- [ ] 送出成功 → 綠色 Snackbar + 清空三個欄位 + 觸發父元件 re-fetch
- [ ] 送出失敗（伺服器錯誤）→ 紅色 Snackbar 顯示錯誤訊息
- [ ] `AccountTable.tsx` — MUI Card，header 有 Typography + Chip (筆數) + TextField (搜尋)
- [ ] Table 欄位：群組名稱（Chip variant="outlined"）、帳號（monospace）、創建者、創建時間
- [ ] Client-side filter：搜尋框輸入時即時 filter 群組名稱 + 帳號（case-insensitive）
- [ ] Client-side 分頁：MUI TablePagination，每頁 20 筆，顯示「X–Y / 共 Z 筆」
- [ ] 更新 `App.tsx` — 選 DB 後呈現 AccountForm + AccountTable，傳入 groups 資料與 callbacks
- [ ] 使用 mock data 驗證完整 UI 流程：選 DB → 表格顯示 → 搜尋 filter → 分頁切換 → 表單送出 → Snackbar → 表單清空
