# 02: Backend API 層 — 3 支 API + 驗證 + UserContext + 錯誤處理 + MockMvc 測試

**What to build:** 在 Ticket 01 的骨架上建立完整的 API 層。使用者可以透過 HTTP 呼叫查詢已設定的 DB 列表、撈取特定 DB 的群組帳號（回傳中不含密碼欄位）、以及新增一筆群組帳號（經過欄位驗證、唯一性檢查、AES256 加密密碼、自動帶入操作者與時間後存入 DB）。整組 MockMvc 整合測試覆蓋所有 edge case。

**Blocked by:** 01-backend-scaffold-datasource-flyway

**Group:** BACKEND

**Status:** ready-for-agent

**API 合約：**
```
GET  /api/v1/databases                → 200 [{ id, name }]
GET  /api/v1/databases/{dbId}/groups  → 200 [{ id, groupName, account, createdBy, createdAt }]
                                        404 if dbId not found
POST /api/v1/databases/{dbId}/groups  → 201 { message, id }
     Body: { groupName, account, password }
     400 if validation fails
     404 if dbId not found
     409 if (groupName + account) already exists
```

- [ ] `UserContext` — `@Component`，`getCurrentUser()` 回傳 `"KUO HAN YANG"`
- [ ] `GroupAccount` — Response DTO（id, groupName, account, createdBy, createdAt，不含 password）
- [ ] `GroupAccountRequest` — Request DTO（groupName, account, password）
- [ ] `GroupService.listGroups(dbId)` — 從 Map 取 JdbcTemplate，SELECT 所有帳號（不含密碼）
- [ ] `GroupService.createGroup(dbId, request)` — 驗證欄位（非空、長度 ≤ 100）→ 唯一性查詢 → 加密密碼 → INSERT（帶入 UserContext.getCurrentUser + now）
- [ ] `GroupController` — 3 支 REST endpoint，path parameter `{dbId}` 路由
- [ ] `DuplicateEntryException` — RuntimeException 子類
- [ ] `GlobalExceptionHandler` — `@RestControllerAdvice`，400/404/409/500 對應處理，錯誤訊息不含密碼
- [ ] Controller log request 時 password 欄位顯示為 `***`
- [ ] MockMvc 測試 — Happy Path：查 DB 列表 → 新增 → 撈列表確認（不含密碼）
- [ ] MockMvc 測試 — 重複新增：新增後再新增相同 (groupName + account) → 409
- [ ] MockMvc 測試 — 驗證失敗：空白欄位 → 400（多個錯誤訊息）
- [ ] MockMvc 測試 — DB 不存在：dbId 不在 config 中 → 404
- [ ] MockMvc 測試 — 跨 DB 隔離：db1 新增 → db2 查為空 → db2 新增相同組合成功
