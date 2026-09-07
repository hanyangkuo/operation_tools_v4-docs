# Operation Tools for Multi-Region System

## Problem Statement

運維團隊需要管理多個跨區域的 Oracle DB 中的群組帳號資料。目前沒有統一的工具，操作人員需要分別連入各 DB 手動執行 SQL，容易出錯且缺乏可追溯性（誰在什麼時候建了什麼帳號）。需要一個簡單的 GUI 工具，能夠切換不同 DB、新增帳號、瀏覽現有帳號，並自動記錄操作者與時間。

## Solution

建立一個 SPA prototype（React + Spring Boot），提供以下能力：

- 左側 Sidebar 列出所有已設定的 Oracle DB 連線，點選切換
- 主區域提供「新增帳號」表單和「帳號列表」表格
- 密碼統一使用 AES256 加密儲存
- 自動帶入操作者（目前 mock 為 "KUO HAN YANG"）和建立時間
- 開發/Demo 時使用 H2 in-memory DB 模擬多個 Oracle，無需安裝 Oracle 即可運作

## User Stories

1. As an 運維人員, I want to 在畫面左側看到所有已設定的 DB 連線清單, so that 我知道有哪些環境可以操作
2. As an 運維人員, I want to 點選左側某個 DB 後看到該 DB 的帳號列表, so that 我可以瀏覽該環境目前有哪些帳號
3. As an 運維人員, I want to 在帳號列表看到群組名稱、帳號、創建者、創建時間, so that 我能追溯每筆帳號的來源
4. As an 運維人員, I want to 帳號列表不顯示密碼, so that 敏感資訊不會在畫面上暴露
5. As an 運維人員, I want to 每頁最多顯示 20 筆帳號, so that 資料量大時畫面不會過長
6. As an 運維人員, I want to 用關鍵字搜尋群組名稱或帳號, so that 我能快速找到特定帳號
7. As an 運維人員, I want to 透過表單輸入群組名稱、帳號、密碼來新增一筆帳號, so that 我不需要手動寫 SQL
8. As an 運維人員, I want to 新增時系統自動檢查欄位是否為空, so that 不會意外送出不完整的資料
9. As an 運維人員, I want to 新增時系統自動檢查群組名稱 + 帳號是否已存在, so that 不會建立重複帳號
10. As an 運維人員, I want to 欄位驗證失敗時在欄位下方看到紅字錯誤訊息, so that 我知道哪個欄位有問題
11. As an 運維人員, I want to 伺服器端錯誤（重複、連線失敗）以 Snackbar 通知呈現, so that 我不會錯過錯誤訊息
12. As an 運維人員, I want to 新增成功後看到綠色 Snackbar 且表單自動清空, so that 我確認操作成功並能立刻輸入下一筆
13. As an 運維人員, I want to 新增成功後帳號列表自動刷新, so that 我不需要手動重整頁面就能看到新資料
14. As an 運維人員, I want to 在畫面右上角（Sidebar 底部）看到我的名字, so that 我知道目前是以哪個身份操作
15. As an 運維人員, I want to 新增帳號時系統自動帶入我的名字作為創建者, so that 操作紀錄可追溯
16. As an 運維人員, I want to 新增帳號時系統自動帶入當前時間作為創建時間, so that 有時間戳可查
17. As an 運維人員, I want to 密碼在儲存到 DB 前經過 AES256 加密, so that 即使 DB 被存取也不會直接洩漏明文密碼
18. As an 運維人員, I want to 切換不同 DB 時資料完全隔離, so that 我不會混淆不同環境的帳號
19. As a 開發者, I want to 在沒有 Oracle 的環境下用 H2 in-memory DB 執行專案, so that 我可以快速開發和 demo
20. As a 開發者, I want to H2 profile 自動建立多組模擬 DB, so that 我能測試跨 DB 切換的功能
21. As a 開發者, I want to 切換到 oracle profile 就能連接真正的 Oracle DB, so that 部署到正式環境時只需改 profile
22. As a 開發者, I want to DB 連線密碼在 application.yaml 中以 AES256 加密存放, so that 設定檔即使外洩也不會直接暴露密碼
23. As a 開發者, I want to 應用程式啟動時 Flyway 自動建表, so that 不需要手動執行 DDL
24. As a 開發者, I want to 前端 build 後輸出到 backend resources/static, so that 只需啟動 Spring Boot 就能同時服務前後端
25. As a 開發者, I want to 後端 log 中不出現任何密碼明文或密文, so that log 被存取時不會洩漏敏感資訊

## Implementation Decisions

### 前端架構

- **建置工具**：Vite + React + TypeScript
- **UI 元件庫**：MUI (Material UI)
- **佈局**：Variant B — Sidebar Dashboard 佈局（左側深色 Sidebar 導航 + 主區域 Card 排列）
- **表格**：MUI 基礎 `Table` 元件 + 自製 filter input + `TablePagination`，不使用 DataGrid
- **狀態管理**：純 `fetch` + `useState` / `useEffect`，不引入額外狀態管理庫
- **表單驗證**：前端做空值和長度檢查（inline error），伺服器錯誤用 Snackbar 呈現
- **Build 輸出**：`vite build` 的 `outDir` 指向 `../backend/src/main/resources/static`，開發時 proxy `/api` 到 `localhost:8080`

### 後端架構

- **技術棧**：Spring Boot 3.x + Maven + Java 17
- **多 DataSource 路由**：啟動時遍歷 `DatabaseProperties` 的所有 DB config，為每組建立 `HikariDataSource` + `JdbcTemplate`，存入 `Map<String, JdbcTemplate>`。Controller 依 path parameter `{dbId}` 查找對應 JdbcTemplate。
- **加密服務**：`EncryptionService` — AES/CBC/PKCS5Padding，key 寫在 `application.yaml`，用於：(1) 解密 yaml 中的 DB 連線密碼 (2) 加密使用者提交的帳號密碼存入 DB
- **Mock 登入**：`UserContext` 類別，`getCurrentUser()` 固定回傳 `"KUO HAN YANG"`。日後接入認證時只需替換此實作
- **DB Migration**：Flyway，`MultiDataSourceConfig` 初始化每個 DataSource 後立即執行 migration

### API 合約

```
GET  /api/v1/databases              → [{ id, name }]
GET  /api/v1/databases/{dbId}/groups → [{ id, groupName, account, createdBy, createdAt }]
POST /api/v1/databases/{dbId}/groups → { message, id }
     Body: { groupName, account, password }
     Errors: 400 (驗證失敗), 404 (DB 不存在), 409 (重複)
```

### DB Schema

來自 logic prototype 的驗證結果：

```sql
CREATE TABLE GROUP_ALL (
    ID          BIGINT AUTO_INCREMENT PRIMARY KEY,
    GROUP_NAME  VARCHAR(100)  NOT NULL,
    ACCOUNT     VARCHAR(100)  NOT NULL,
    PASSWORD    VARCHAR(256)  NOT NULL,   -- AES256 加密後
    CREATED_BY  VARCHAR(100)  NOT NULL,
    CREATED_AT  TIMESTAMP     NOT NULL,
    CONSTRAINT UK_GROUP_ACCOUNT UNIQUE (GROUP_NAME, ACCOUNT)
);
```

唯一性約束在 `(GROUP_NAME, ACCOUNT)` 組合上。相同的群組+帳號組合可以存在於不同 DB（跨 DB 隔離）。

### Profile 策略

- `h2` profile（預設）：多組 `jdbc:h2:mem:oracle_sim_N;MODE=Oracle` 模擬多個 Oracle
- `oracle` profile：連接真正的 Oracle DB，JDBC driver 從 Oracle Maven Repository 取得 `ojdbc11`

### 安全規則

- 後端所有 log 禁止記錄密碼明文或密文
- Controller log request 時遮蔽 password 欄位
- EncryptionService 不 log 輸入/輸出值
- GlobalExceptionHandler 錯誤訊息中不包含密碼內容

### 建置流程

```
cd frontend && npm run build     # 輸出到 backend/src/main/resources/static
cd backend && mvn spring-boot:run -Dspring-boot.run.profiles=h2
```

單一 Spring Boot 進程同時服務前端靜態資源和後端 API。

## Testing Decisions

### Testing Seam

唯一的 testing seam 設在 **API boundary（Controller 層）**。使用 Spring Boot 的 `MockMvc` 發送 HTTP request，驗證完整流程（路由 → 驗證 → 加密 → DB 操作 → response）。不單獨測試 Service 或 EncryptionService — 它們的行為透過 API 測試間接覆蓋。

### 好的測試特徵

- 只測外部行為（HTTP status + response body），不測內部實作細節
- 每個測試獨立，不依賴其他測試的執行順序
- 使用 H2 in-memory DB，測試自帶 Flyway migration，無外部依賴

### 測試案例（對應 logic prototype 的 4 個 walkthrough）

1. **Happy Path**：查詢 DB 列表 → 新增帳號 → 撈列表確認有資料且不含密碼
2. **重複新增**：新增一筆後再新增相同 (群組+帳號) → 409 Conflict
3. **驗證失敗**：空白欄位 → 400；不存在的 dbId → 404
4. **跨 DB 隔離**：db1 新增 → db2 查詢為空 → db2 新增相同組合應成功

## Out of Scope

- **認證與授權**：目前 mock 登入，不實作 login/logout、JWT、session management
- **修改與刪除（Update / Delete）**：只做 Create + Read
- **Server-side 分頁**：一次撈全部資料，前端切頁
- **密碼強度驗證**：不檢查密碼複雜度
- **Audit log**：不記錄操作歷史（超出 CREATED_BY/CREATED_AT 以外的）
- **多語系 (i18n)**：UI 固定中文
- **Responsive / Mobile**：只考慮桌面瀏覽器
- **CI/CD pipeline**
- **Docker / Kubernetes 部署設定**

## Further Notes

- 兩個 throwaway prototype 已提交於專案根目錄，用於紀錄設計決策的驗證過程：
  - `prototype-ui.html` — 3 個 UI variant，最終選定 Variant B (Sidebar Dashboard)
  - `prototype-backend-logic.html` — 後端邏輯模擬，驗證多 DataSource 路由、AES256 加解密、驗證 + 唯一性檢查、UserContext mock 的完整流程
- 這些 prototype 是 primary source，正式實作完成後應移至 throwaway branch 保存
- H2 的 `MODE=Oracle` 能模擬大部分 Oracle SQL 語法，但 Flyway DDL 在 H2 和 Oracle 之間有差異（`BIGINT AUTO_INCREMENT` vs `NUMBER` + `SEQUENCE + TRIGGER`），需要依 vendor 分檔
