# ADR 0001: Use H2 with Oracle Mode for Development and Demo

## Context
本專案是一個用來管理跨區域多台 Oracle DB 的運維工具。在開發與 Demo 階段，如果要求每位開發者都在本地安裝 Oracle DB，或是直接連線到遠端的開發/測試機，會大幅增加開發環境設定的門檻，也不利於快速展示（Demo）多資料庫切換的功能。

## Decision
我們決定在開發環境預設啟用 `h2` profile，並使用 H2 in-memory 資料庫的 `MODE=Oracle` 來模擬 Oracle DB。同時，我們會在 `application-h2.yaml` 中設定多組不同的 JDBC URL（例如 `jdbc:h2:mem:oracle_sim_1` 與 `oracle_sim_2`），以在本地完整模擬跨多台 DB 的情境。

## Consequences
* **Good:** 開發者 Clone 專案後無需任何外部相依服務即可直接啟動（Zero Setup）。
* **Good:** 方便在沒有真實網路環境的情況下進行 Demo，並能完美展示跨 DB 的隔離性。
* **Bad:** H2 的 `MODE=Oracle` 無法 100% 完美模擬 Oracle 的所有語法與行為。因此在 Flyway DDL 管理上，可能需要依賴 Vendor 分檔（例如 H2 使用 `BIGINT AUTO_INCREMENT`，Oracle 需使用 `NUMBER` 搭配 `SEQUENCE`）。
