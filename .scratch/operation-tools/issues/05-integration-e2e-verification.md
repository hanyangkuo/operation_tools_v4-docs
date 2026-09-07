# 05: 前後端整合 + Build Pipeline + 端對端驗證

**What to build:** 將前後端對接。Frontend 的 `api/index.ts` 從 mock data 切換為真正呼叫後端 REST API（使用 `fetch`）。確認 Vite dev proxy 與 build output path 設定正確。最終驗證：單獨啟動 Spring Boot 後，從瀏覽器存取完整應用，端對端完成所有 user story。

**Blocked by:** 02-backend-api-tests, 04-frontend-form-table-interaction

**Group:** INTEGRATION

**Status:** done (2026-09-03)

- [x] `api/index.ts` — 將 `fetchDatabases` 改為 `fetch('/api/v1/databases')`
- [x] `api/index.ts` — 將 `fetchGroups(dbId)` 改為 `fetch(`/api/v1/databases/${dbId}/groups`)`
- [x] `api/index.ts` — 將 `createGroup(dbId, data)` 改為 `POST /api/v1/databases/${dbId}/groups`，body 為 JSON
- [x] `api/index.ts` — 錯誤處理：非 2xx 回應時 parse error message 並 throw（優先 `errors[]`，次 `message`）
- [x] 確認 `vite.config.ts` 的 dev proxy（`/api` → `localhost:8080`）運作正常 — 實測 `curl :5173/api/v1/databases` 回傳後端資料
- [x] 確認 `npm run build` 輸出到 `backend/src/main/resources/static`
- [x] 端對端驗證 — 啟動後端（H2 profile）→ Sidebar 顯示兩組 DB（db1/db2）
- [x] 端對端驗證 — 選擇 DB → 表格顯示空列表
- [x] 端對端驗證 — 新增帳號 → 201 `{message,id}` → 列表含資料且無 password 欄
- [x] 端對端驗證 — 重複新增相同群組 + 帳號 → 409
- [x] 端對端驗證 — 表單空值送出 → 400 + `errors[]`（前端 inline error 阻擋）
- [x] 端對端驗證 — 搜尋 filter / 分頁（元件邏輯，ticket 04 已實作，整合後未回歸）
- [x] 端對端驗證 — 切換到另一組 DB → 資料隔離正確（db2 相同組合可新增）
- [x] 端對端驗證 — `npm run build` → 只啟動 Spring Boot → `GET /` 回 200 + `index.html`，`/assets/*.js` 200

**驗證方式：** 後端 `mvnw clean package` 5/5 測試通過；打包 jar 內含 SPA 靜態檔；`java -jar` 啟動後 curl 9 組情境全綠；後端 log 無密碼明文/密文。詳見 `doc/RUNBOOK.md`。
