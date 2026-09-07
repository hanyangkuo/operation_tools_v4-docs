# 01: Backend 基礎建設 — 專案骨架 + 多 DataSource + 加解密 + Flyway

**What to build:** Spring Boot 3 專案從零建起。啟動後自動連線到設定於 `application-h2.yaml` 的兩組 H2 in-memory 資料庫（模擬多個 Oracle），對每一組執行 Flyway migration 建立 `GROUP_ALL` table，並將所有 `JdbcTemplate` 存入一個 Map 供後續 API 使用。AES256 加解密服務可以正確解密 yaml 中的 DB 連線密碼。

啟動時的 log 應該可以看到：兩個 DataSource 初始化成功、兩次 Flyway migration 完成。密碼相關的值不可出現在 log 中。

**Blocked by:** None (can start immediately)

**Group:** BACKEND

**Status:** ready-for-agent

**參考資料：**
- ADR 0001 (H2 MODE=Oracle)、ADR 0002 (Map 路由)、ADR 0003 (統一 AES256)
- Schema（來自 prototype 驗證）：

```sql
CREATE TABLE GROUP_ALL (
    ID          BIGINT AUTO_INCREMENT PRIMARY KEY,
    GROUP_NAME  VARCHAR(100)  NOT NULL,
    ACCOUNT     VARCHAR(100)  NOT NULL,
    PASSWORD    VARCHAR(256)  NOT NULL,
    CREATED_BY  VARCHAR(100)  NOT NULL,
    CREATED_AT  TIMESTAMP     NOT NULL,
    CONSTRAINT UK_GROUP_ACCOUNT UNIQUE (GROUP_NAME, ACCOUNT)
);
```

- [ ] `pom.xml` 建立：Spring Boot 3.x parent、spring-boot-starter-web、spring-boot-starter-jdbc、flyway-core、h2 (runtime)、ojdbc11 (Oracle Maven Repo)
- [ ] `OperationToolsApplication.java` 標準入口
- [ ] `DatabaseProperties` 以 `@ConfigurationProperties(prefix = "app")` 讀取 `app.databases` 列表（id, name, url, username, encrypted-password）與 `app.encryption.key`
- [ ] `EncryptionService` 實作 AES/CBC/PKCS5Padding 加密與解密，金鑰來自 DatabaseProperties
- [ ] `MultiDataSourceConfig` 啟動時遍歷所有 DB config → 解密密碼 → 建 HikariDataSource + JdbcTemplate → 存入 Map → 對每個 DataSource 執行 Flyway migration
- [ ] `application.yaml` 共用設定（server.port: 8080, default profile: h2, encryption key）
- [ ] `application-h2.yaml` 設定兩組 H2 DB（oracle_sim_1, oracle_sim_2，MODE=Oracle）
- [ ] `application-oracle.yaml` 範例 Oracle 連線設定（註解說明）
- [ ] `V1__create_group_all_table.sql` Flyway DDL
- [ ] 執行 `mvn spring-boot:run -Dspring-boot.run.profiles=h2` 驗證啟動成功、兩組 DB 初始化、table 建立完成
