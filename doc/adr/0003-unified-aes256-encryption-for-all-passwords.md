# ADR 0003: Unified AES256 Encryption for All Passwords

## Context
系統中有兩種需要保護的密碼：
1. `application.yaml` 中連線 Oracle DB 的資料庫密碼（應用程式需要解密才能連線）。
2. 使用者透過 UI 表單新增的「群組帳號」密碼（儲存至 DB 的 `GROUP_ALL` table 中）。

通常，資料庫連線密碼使用對稱式加密（可逆），而使用者密碼使用雜湊（Hash，不可逆，如 bcrypt）以提升安全性。

## Decision
根據專案討論（Grill Session Q2 決策），我們決定為這兩種密碼統一使用 **同一個 AES256 金鑰與對稱式加密演算法**。此金鑰將設定於 `application.yaml` (`app.encryption.key`) 中。

## Consequences
* **Good:** 架構與實作大幅簡化，系統只需要維護單一的 `EncryptionService` 與一把金鑰。
* **Bad:** 使用者的密碼以可逆的方式儲存。如果 AES256 金鑰外洩，攻擊者可以直接解密出所有儲存在 DB 中的使用者密碼。由於此專案為 Prototype，在開發與易用性的考量下暫時接受此安全妥協，未來若進入 Production 可考慮分離機制（使用者密碼改用 bcrypt）。
