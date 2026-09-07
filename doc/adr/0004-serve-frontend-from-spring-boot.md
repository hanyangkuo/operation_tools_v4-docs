# ADR 0004: Serve Frontend Static Files from Spring Boot

## Context
專案分為前端（React + Vite）與後端（Spring Boot 3）。我們需要決定應用程式的部署與打包策略。選項包含前後端分離部署（例如前端放 Nginx、後端跑 Java），或是將前端靜態檔案打包進 Spring Boot 中一併提供服務。

## Decision
我們決定將前端打包為靜態檔案，交由 Spring Boot 統一 Serve。
實作方式：前端的 `vite.config.ts` 中的 `build.outDir` 指向 `../backend/src/main/resources/static`。透過執行 `npm run build`，將打包結果直接放入 Spring Boot 的靜態資源目錄下。

## Consequences
* **Good:** 部署極簡化，最終產出只有一個 `.jar` 檔，隨處可跑。
* **Good:** 解決跨網域（CORS）問題，前端 API 請求不需額外處理 Origin。
* **Bad:** 前後端發布生命週期綁定，前端任何小修改都需要重新打包整個 Java 應用程式。
* **Bad:** Spring Boot 在 Serve 靜態檔案的效能與進階快取設定上，不如專業的 Nginx。但在內部運維工具的場景下，效能差異可忽略。
