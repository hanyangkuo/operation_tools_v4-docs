# Operation Tools for Multi-Region System

這是一個用來管理多個跨區域 Oracle DB 群組帳號資料的單頁應用程式 (SPA) Prototype。提供簡單的 GUI 工具，讓運維人員能夠切換不同 DB、新增帳號、瀏覽現有帳號，並自動記錄操作者與時間。

---

## 🚀 快速上手 (Quick Start)

專案分為 `frontend` (React + Vite) 與 `backend` (Spring Boot 3)。在開發與測試階段，我們預設使用 `H2` in-memory 資料庫模擬 Oracle，因此**不需安裝任何資料庫即可直接啟動**。

### 1. 啟動開發環境 (Frontend + Backend 分離)

如果你要修改程式碼，建議分別啟動前後端：

**啟動 Backend (API)**
```powershell
cd backend
# 使用 Spring Boot 內建的 Maven Wrapper 啟動，預設載入 h2 profile
.\mvnw spring-boot:run -Dspring-boot.run.profiles=h2
```
*後端會運行在 `http://localhost:8080`*

**啟動 Frontend (UI)**
```powershell
cd frontend
npm install
npm run dev
```
*前端會運行在 `http://localhost:5173` (Vite 預設)，所有 `/api` 請求會自動 Proxy 到後端 8080 port。*

### 2. 打包與整合測試 (Single Jar 部署)

正式部署或端對端測試時，可以將前端編譯並打包進後端：

```powershell
# 1. 編譯前端 (產出物會自動放進 backend/src/main/resources/static)
cd frontend
npm run build

# 2. 啟動後端 (Spring Boot 會同時 Serve API 與靜態網頁)
cd ../backend
.\mvnw spring-boot:run -Dspring-boot.run.profiles=h2
```
*打開瀏覽器訪問 `http://localhost:8080` 即可看到完整的應用程式。*

---

## 🔐 設定真實的 Oracle DB

當準備好要連線到真實的 Oracle DB 時，請使用 `oracle` profile，設定檔位於：`backend/src/main/resources/application-oracle.yaml`。

### 產生加密的資料庫密碼

基於安全規範，`application-oracle.yaml` 中**絕對不可明碼儲存資料庫連線密碼**。
我們提供了一支小工具 `PasswordEncryptor` 幫助你加密密碼：

1. 開啟終端機，切換到 `backend` 目錄。
2. 執行以下指令編譯並執行加密工具（請將 `你的真實密碼` 替換為實際密碼）：
```powershell
cd backend
.\mvnw compile
java -cp target\classes com.example.operationtools.crypto.PasswordEncryptor "你的真實密碼"
```
3. 工具會輸出一段 Base64 格式的加密字串（例如：`+j/wT35xcgvIps9RFLd6ME7br/n8ct4NZjA4d6CgmkY=`）。
4. 開啟 `application-oracle.yaml`，找到 `REPLACE_WITH_AES256_ENCRYPTED_PASSWORD`，將其替換為你剛剛產出的加密字串。

### 使用 Oracle Profile 啟動
設定完成後，使用以下指令啟動專案（切換為 `oracle` profile）：
```powershell
cd backend
.\mvnw spring-boot:run -Dspring-boot.run.profiles=oracle
```

---

## 🛠 後續開發指南 (Development Guide)

以下為專案的架構決策與設計細節，接手開發時請遵守。

### 前端架構
- **建置工具**：Vite + React + TypeScript
- **UI 元件庫**：MUI (Material UI)
- **佈局**：Sidebar Dashboard 佈局（左側深色 Sidebar 導航 + 主區域 Card 排列）
- **表格**：MUI 基礎 `Table` 元件 + 自製 filter input + `TablePagination`，不使用 DataGrid
- **狀態管理**：純 `fetch` + `useState` / `useEffect`，不引入額外狀態管理庫
- **表單驗證**：前端做空值和長度檢查（inline error），伺服器錯誤用 Snackbar 呈現

### 後端架構
- **技術棧**：Spring Boot 3.x + Maven + Java 17
- **多 DataSource 路由**：啟動時遍歷 `DatabaseProperties` 的所有 DB config，為每組建立 `HikariDataSource` + `JdbcTemplate`，存入 `Map<String, JdbcTemplate>`。Controller 依 path parameter `{dbId}` 查找對應 JdbcTemplate。
- **加密服務**：`EncryptionService` — AES/CBC/PKCS5Padding，key 寫在 `application.yaml`，用於：(1) 解密 yaml 中的 DB 連線密碼 (2) 加密使用者提交的帳號密碼存入 DB
- **Mock 登入**：`UserContext` 類別，`getCurrentUser()` 固定回傳 `"KUO HAN YANG"`。日後接入真實認證系統時只需替換此實作
- **DB Migration**：Flyway，`MultiDataSourceConfig` 初始化每個 DataSource 後立即自動執行 migration

### API 合約
```http
GET  /api/v1/databases                → [{ id, name }]
GET  /api/v1/databases/{dbId}/groups  → [{ id, groupName, account, createdBy, createdAt }]
POST /api/v1/databases/{dbId}/groups  → { message, id }
     Body: { groupName, account, password }
     Errors: 400 (驗證失敗), 404 (DB 不存在), 409 (重複)
```

### DB Schema (GROUP_ALL)
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
*(註: H2 的 `MODE=Oracle` 使用上方語法。真實 Oracle DB 需依 Flyway Vendor 機制使用 `NUMBER` + `SEQUENCE + TRIGGER`，詳見 ADR 0001)*

### ⚠️ 安全規範 (Security Rules)
- **後端所有 log 禁止記錄密碼明文或密文。**
- Controller log request 時需遮蔽 password 欄位。
- `EncryptionService` 不 log 輸入/輸出值。
- `GlobalExceptionHandler` 錯誤訊息中不可包含密碼內容。

### 自動化測試 (Testing Decisions)
唯一的 testing seam 設在 **API boundary（Controller 層）**。
使用 Spring Boot 的 `MockMvc` 發送 HTTP request，驗證完整流程（路由 → 驗證 → 加密 → DB 操作 → response）。不單獨為 Service 或 EncryptionService 寫孤立測試。

主要測試案例包含：
1. **Happy Path**：查詢 DB 列表 → 新增帳號 → 撈列表確認有資料且不含密碼
2. **重複新增**：新增一筆後再新增相同 (群組+帳號) → 409 Conflict
3. **驗證失敗**：空白欄位 → 400；不存在的 dbId → 404
4. **跨 DB 隔離**：db1 新增 → db2 查詢應為空 → db2 新增相同組合應成功

### 專案範圍外 (Out of Scope)
- 認證與授權（Login/Logout, JWT, Session management）
- 帳號的修改與刪除（Update / Delete）
- Server-side 分頁（目前為一次撈全部資料，前端切頁）
- 密碼強度複雜度驗證
- 額外的 Audit log 表（目前僅依賴 CREATED_BY / CREATED_AT）
- 多語系 (i18n) 與 Mobile RWD
- CI/CD pipeline 與 Docker / K8s 部署設定

---

> 此文件整理自專案初期的設計討論與 Prototype 驗證。詳細架構決策請參考 `doc/adr/` 目錄下的 ADR 文件。
