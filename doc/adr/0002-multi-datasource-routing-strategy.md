# ADR 0002: Multi-DataSource Routing Strategy

## Context
系統需要連線到定義在 `application.yaml` 中的多個 Oracle DB。前端會提供一個下拉選單讓使用者選擇目標 DB，之後所有的 API 請求都需要路由到該 DB 執行 SQL。Spring 處理多 DataSource 有幾種常見做法，例如使用 `AbstractRoutingDataSource` 搭配 `ThreadLocal` 動態切換，或在需要時才 Lazy Init。

## Decision
我們決定採用「啟動時全部初始化並以 Map 路由」的策略。在 Spring Boot 啟動時，遍歷 `DatabaseProperties` 中的所有 DB 設定，為每一組建立專屬的 `HikariDataSource` 與 `JdbcTemplate`，並存入一個 `Map<String, JdbcTemplate>` 中。API 設計上採用 RESTful 風格將 DB ID 放在 Path Parameter (`/api/v1/databases/{dbId}/groups`)，Controller 層接到請求後，直接從 Map 中取出對應的 `JdbcTemplate` 傳給 Service 使用。

## Consequences
* **Good:** 實作極度簡單，不需處理複雜的 `ThreadLocal` 狀態管理。
* **Good:** 啟動時 Fail-fast，如果某組 DB 設定錯誤或密碼解密失敗，會在啟動時立刻報錯，而不是等到使用者點選時才發生錯誤。
* **Bad:** 啟動時間會隨設定的 DB 數量增加而變慢。
* **Bad:** 就算某些 DB 很少被使用，仍會佔用 Connection Pool 的資源。考量到這是一個輕量級運維原型工具，此缺點可以接受。
