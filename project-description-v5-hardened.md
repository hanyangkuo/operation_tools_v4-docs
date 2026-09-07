# Operation Tool V5: Hardened Specification & Team Guidelines

> **Architectural Note:** 
> 這份文件是從原始 v2 需求經過「殘酷減法」與「邊界控制」淬鍊出的最終規格。
> 它不僅描述需求，更定義了**不該做什麼 (Out of Scope)** 以及**開發鐵律 (Iron Rules)**，
> 用來確保 Agent 在開發過程中不會過度設計或發散。

---

## 1. 核心需求與邊界 (Core Requirements & Boundaries)

我們正在建立一個用於維運的 SPA Web 應用程式 (React + Spring Boot)，作為一個**輕量級任務編排器 (Task Orchestrator)**，允許管理員在多個資料庫上執行定義在設定檔中的 SQL 任務。

### 🚨 絕對禁止的過度設計 (Out of Scope)
- **🚫 禁止自行發明認證架構：** 不准手寫 Keycloak BFF 或複雜的 Token 交換。直接使用 Spring Boot OAuth2 Client 或套用公司既有的 OIDC Filter 範本。
- **🚫 禁止動態 DB 連線池：** 不要在 Runtime 動態建立 DB 連線，啟動時全部讀取 `application.yaml` 初始化完畢。
- **🚫 禁止複雜的前端狀態庫：** 不使用 Redux / Zustand / TanStack Query。只允許使用原生的 `fetch` + `useState`。
- **🚫 禁止超重型自動化測試：** 整合測試中 **禁止使用 Testcontainers (Oracle/Keycloak)**。測試請一律使用 `H2 (MODE=Oracle/MySQL)` 以及 MockMvc + MockUser 來確保能在 10 秒內跑完。
- **🚫 禁止建立任務排程器：** 任務是由 UI「手動」觸發並等待結果，不實作 CronJob、背景 Queue 或非同步對帳機制。

---

## 2. 精確的 API 合約 (Strict API Contracts)

為了防止 Agent 隨意擴增後端範圍，本系統**僅允許存在以下 4 支 API**。不准新增任何其他 API。

1. **`GET /api/v1/auth/me`**
   - 用途：前端取得當前登入者資訊，檢查是否為 `application.yaml` 中的 admin whitelist。
2. **`GET /api/v1/databases`**
   - 用途：取得可用的資料庫清單 (從 yaml 讀取)。
3. **`GET /api/v1/tasks`**
   - 用途：取得定義好的 Task 列表與其需要的 UI Parameters (從 yaml 讀取)。
4. **`POST /api/v1/tasks/{taskId}/execute`**
   - 用途：執行特定任務。
   - Body: `{ dbName: "Oracle-A", params: { "userId": "123" } }`
   - Response: `200 OK` (成功執行) / `400` (參數錯誤) / `403` (非管理員)

---

## 3. 架構實作鐵律 (Team Iron Rules)

如果未來的 Agent (或 Claude Code) 要讀取此專案進行開發，必須遵守以下 `CLAUDE.md` 級別的指示：

### Phase 1: MVP 核心實作 (不含資安與真實 DB)
- **Backend:** 
  - 實作 Task Orchestrator 核心邏輯（解析 YAML 中的 Steps，支援帶入參數）。
  - 用 H2 in-memory DB 模擬目標資料庫。
  - 使用 `@MockUser` 或簡單的 `UserContext` 假裝已經登入且是 Admin。
- **Frontend:** 
  - 實作 Sidebar (選 DB 與 Task) 與主畫面 (動態產生 Parameter 表單)。
  - 串接 Phase 1 假 API，驗證完整邏輯。

### Phase 2: 外圍整合 (Infrastructure & Security)
- **Security (OIDC):** 在確認 MVP 邏輯 100% 正確後，才允許引入 Keycloak OIDC。將 `reference-code/` 中的標準 Filter 掛載到 Spring Security 中。
- **Database:** 將 H2 切換為真實的 MariaDB & Oracle JDBC Driver，並在本地使用 `docker-compose.yml` 進行端對端人工測試。

---

## 4. 目錄實體隔離原則
本專案嚴格分為 `frontend/` 與 `backend/` 兩個獨立目錄。
在建立 Ticket 或進行實作時，**一張 Ticket / 一個 Session 只能修改其中一個目錄**。絕對不允許跨目錄的 Full-stack 修改。
