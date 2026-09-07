# RUNBOOK — 把專案跑起來看結果

本文件是 **實際執行的 SOP**。對應 ticket `05-integration-e2e-verification.md` 的驗收項目。
最後驗證日期：2026-09-03（前後端整合完成，5/5 後端測試通過，端對端 curl 全綠）。

---

## 0. 前置需求

| 工具 | 版本 | 備註 |
|------|------|------|
| JDK  | 17   | `java -version` 需為 17.x |
| Node | 18+ (實測 24) | 附 npm |
| Maven | **不需要安裝** | 用 `backend/mvnw`（Maven Wrapper），第一次執行會自動下載 Maven 3.9.9 |
| 瀏覽器 | 桌面版 Chrome / Edge | 專案只支援桌面 |

> Windows 使用者：以下指令同時給 **PowerShell** 和 **Git Bash** 兩種寫法。擇一即可。

---

## 1. 模式 A — 單一 Spring Boot 進程（Production-like，推薦驗收用）

前端 build 成靜態檔塞進後端，只需啟動一個進程。對應 ticket 最後一項驗收。

### 1-1. Build 前端 → 輸出到 `backend/src/main/resources/static`

PowerShell：
```powershell
cd E:\claude\project_code\operation_tools_v4\frontend
npm install          # 第一次才需要
npm run build
```

Git Bash：
```bash
cd /e/claude/project_code/operation_tools_v4/frontend
npm install
npm run build
```

預期輸出：
```
../backend/src/main/resources/static/index.html   0.61 kB
../backend/src/main/resources/static/assets/index-*.js  ~389 kB
✓ built in ~2s
```

### 1-2. Build + 測試 + 啟動後端（h2 profile）

PowerShell：
```powershell
cd E:\claude\project_code\operation_tools_v4\backend
.\mvnw clean package                         # 會跑 5 個整合測試，需全綠
java -jar target\operation-tools-backend-0.0.1-SNAPSHOT.jar
```

Git Bash：
```bash
cd /e/claude/project_code/operation_tools_v4/backend
./mvnw clean package
java -jar target/operation-tools-backend-0.0.1-SNAPSHOT.jar
```

> `h2` 是預設 profile（`application.yaml` 的 `spring.profiles.active: h2`），不必額外指定。
> 想顯式指定：`java -jar ... --spring.profiles.active=h2`
> 想用 `mvn` 直接跑（免 package）：`.\mvnw spring-boot:run`

啟動成功會看到：
```
DataSourceRegistry ready with 2 database(s): [db1, db2]
Flyway migration complete for id='db1' ...
Flyway migration complete for id='db2' ...
Tomcat started on port 8080
```

### 1-3. 打開瀏覽器

```
http://localhost:8080/
```

前端頁面由 Spring Boot 直接提供，`/api/v1/*` 由同一進程處理，**不需要** Vite。

### 1-4. 關閉

在跑 `java -jar` 的視窗按 `Ctrl+C`。
H2 是 in-memory，**進程結束資料即清空**，下次啟動回到乾淨狀態。

---

## 2. 模式 B — 開發模式（前端 hot reload，兩個進程）

改前端程式碼會即時反映時使用。

### 視窗 1：後端
```powershell
cd E:\claude\project_code\operation_tools_v4\backend
.\mvnw spring-boot:run
```

### 視窗 2：前端 dev server
```powershell
cd E:\claude\project_code\operation_tools_v4\frontend
npm run dev
```

### 打開瀏覽器
```
http://localhost:5173/
```

Vite 會把 `/api` 的請求 proxy 到 `http://localhost:8080`（設定在 `vite.config.ts`）。
已驗證：`curl http://localhost:5173/api/v1/databases` 會回傳後端的 DB 清單。

---

## 3. 端對端驗收清單（對應 ticket 05）

啟動模式 A 後，在瀏覽器逐項確認：

| # | 操作 | 預期結果 |
|---|------|----------|
| 1 | 打開 `http://localhost:8080/` | 左側 Sidebar 顯示 **兩組 DB**：`H2 Simulated Oracle (Region A)` / `(Region B)`；Sidebar 底部顯示 `KUO HAN YANG` |
| 2 | 點左側 Region A | 主區出現「新增帳號」表單 + 「帳號列表」表格，表格 **空列表**（0 筆） |
| 3 | 填 群組=`Admin Group`、帳號=`admin01`、密碼=`P@ssw0rd`，按「送出」 | 上方跳 **綠色 Snackbar「新增成功」**；表單清空；表格 **自動刷新**，出現 1 列，欄位有 群組/帳號/創建者(`KUO HAN YANG`)/創建時間；**沒有密碼欄** |
| 4 | 再送出一次相同 群組+帳號 | **紅色 Snackbar**：`群組「Admin Group」+ 帳號「admin01」已存在`（後端回 409） |
| 5 | 清空任一欄位後送出（例如密碼留白） | 欄位下方 **紅字 inline error**（如「密碼為必填」），請求 **不送出** |
| 6 | 在表格搜尋框輸入 `admin` | 表格即時 filter；清空還原 |
| 7 | 連續新增 > 20 筆後看分頁 | 每頁 20 筆，`TablePagination` 可切下一頁 |
| 8 | 點左側 Region B | 表格為空（**跨 DB 隔離**）；在 Region B 新增相同的 `Admin Group` + `admin01` → **成功**（不同 DB 各自唯一） |
| 9 | 直接在網址列輸入 `http://localhost:8080/` 重新整理 | 前端頁面正常載入（靜態檔由 Spring Boot 提供） |

---

## 4. 純 API 冒煙測試（不開瀏覽器，快速驗證後端）

後端啟動後，Git Bash 執行：

```bash
B=http://localhost:8080

curl -s $B/api/v1/databases                                   # [{"id":"db1",...},{"id":"db2",...}]
curl -s $B/api/v1/databases/db1/groups                        # []

curl -s -i -X POST $B/api/v1/databases/db1/groups \
  -H 'Content-Type: application/json' \
  -d '{"groupName":"Admin Group","account":"admin01","password":"P@ssw0rd"}'
                                                               # HTTP 201  {"message":"新增成功","id":1}

curl -s $B/api/v1/databases/db1/groups
                                                               # 1 列，含 createdBy=KUO HAN YANG，無 password 欄

curl -s -o /dev/null -w '%{http_code}\n' -X POST $B/api/v1/databases/db1/groups \
  -H 'Content-Type: application/json' \
  -d '{"groupName":"Admin Group","account":"admin01","password":"x"}'
                                                               # 409

curl -s -X POST $B/api/v1/databases/db1/groups \
  -H 'Content-Type: application/json' -d '{"groupName":" ","account":"","password":""}'
                                                               # 400 + "errors":["群組名稱為必填","帳號為必填","密碼為必填"]

curl -s -o /dev/null -w '%{http_code}\n' $B/api/v1/databases/nope/groups
                                                               # 404
```

PowerShell 版用 `curl.exe`（不是 alias）或 `Invoke-RestMethod`。

已於 2026-09-03 全部實測通過，且後端 log 中密碼一律顯示為 `password=***`，無明文/密文外洩。

---

## 5. 疑難排解

| 症狀 | 原因 / 解法 |
|------|-------------|
| `mvnw` 第一次很慢 | 正在下載 Maven 3.9.9 到 `~/.m2/wrapper/dists`，只有第一次 |
| `Port 8080 already in use` | 有殘留進程。PowerShell：`Get-NetTCPConnection -LocalPort 8080` 找 PID 後 `Stop-Process -Id <pid>` |
| 前端頁面打開是 **舊版** | 模式 A 下沒重新 `npm run build` + 重新 `mvnw package`。build 產物在 jar 內 |
| Sidebar 空白、表格一直轉圈 | 後端沒起來或 profile 錯。確認 log 有 `DataSourceRegistry ready with 2 database(s)` |
| 重啟後資料不見了 | 正常。H2 in-memory，每次啟動重跑 Flyway，資料歸零 |
| `java -version` 不是 17 | 裝 JDK 17 並設 `JAVA_HOME` |
| 想連真的 Oracle | 用 `--spring.profiles.active=oracle`，並在 `application-oracle.yaml` 填連線資訊（密碼需 AES256 加密後填 `encrypted-password`）。**本次未驗證**，超出 prototype 範圍 |

---

## 6. 這次整合改了什麼（ticket 05）

- `frontend/src/api/index.ts`：從 mock in-memory store 全面改為真正的 `fetch` 呼叫
  - `fetchDatabases()` → `GET /api/v1/databases`
  - `fetchGroups(dbId)` → `GET /api/v1/databases/{dbId}/groups`（回傳後依時間新→舊排序、時間格式化為 `YYYY-MM-DD HH:mm:ss`）
  - `createGroup(dbId, data)` → `POST /api/v1/databases/{dbId}/groups`，body 為 JSON
  - 錯誤處理：非 2xx 時 parse 後端 `ApiError`，優先取 `errors[]`（驗證錯誤，以「、」串接）再取 `message`，丟出 `Error` 給 Snackbar 顯示
- `backend/mvnw`, `backend/mvnw.cmd`, `backend/.mvn/wrapper/`：新增 Maven Wrapper，免安裝 Maven
- `vite.config.ts`、build outDir：確認正確，未改動

### 已知限制
- 帳號列表載入失敗（後端關閉 / 500）時，前端靜默顯示空表格，不跳 Snackbar。表單送出的錯誤有 Snackbar。
- 專案目前 **非 git repo**，本次變更未提交版控。要保存請先 `git init`。
