# ADR 0005: Use API Boundary as Sole Testing Seam

## Context
對於後端包含多 DB 路由、驗證、AES加解密、DB Insert 等邏輯，我們需要決定自動化化測試的粒度與接縫（Testing Seam）。傳統做法可能會為 Controller、Service、EncryptionService 分別撰寫 Unit Tests。

## Decision
我們決定將測試的最高接縫設定於 **API Boundary（Controller 層）**。
使用 Spring Boot 的 `MockMvc` 來發起 HTTP 請求，並驗證最終的回應狀態碼（Status）與內容（Body）。我們不對 `GroupService` 或 `EncryptionService` 撰寫孤立的單元測試。測試執行時將使用 H2 in-memory DB 並自動執行 Flyway migration。

## Consequences
* **Good:** 測試直接驗證了使用者的真實行為（HTTP IN -> HTTP OUT + DB State），減少了測試與實作細節（Implementation Detail）的耦合。
* **Good:** 未來內部邏輯（例如把加解密抽出、或更改 Map 路由實作）重構時，測試代碼不需修改。
* **Good:** 大幅減少 Mock 數量。
* **Bad:** 執行速度相較於純粹的 Unit Test 稍慢，因為需要啟動 Spring Context 與 H2 資料庫。
