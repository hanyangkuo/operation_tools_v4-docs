# A Operation tool for Multi-Region system: Project Specification
- 一個簡單的 prototype 專案, 能夠 demo GUI 加上實際的按鈕功能，可以實際接到 DB 塞入資料
- 先不實做認證授權, 直接 mock 登入者是 "KUO HAN YANG" 並顯示於 UI 右上方
- 這是一個 SPA Spring Boot 專案, 前端使用 React 搭配 Typescript 開發, 後端使用 SpringBoot 搭配 Java 17
- 專案的第一層直接分出 frontend, backend 的資料夾, 起專案的流程會是 frontend 先 pre-build 出靜態網站, 放到 backend resources 資料夾後再把 springboot 跑起來

## Global Config (Application.yaml)
- 可以 config 多組 Oracle DB (ID, db url, account, encrypted password)
- application.yaml 上的 encrypted password 是事先用 AES256 對稱式加密過後的密碼


## UI/UX
- 左上角呈現一個下拉式選單, 可以選擇 config 在 application.yaml 的 Oracle DB
- 選擇 DB 後會出現兩個 block
- block 1 表單, 表單包含三個欄位 1.群組名稱 2.帳號  3. 密碼。送出後, 會打一發 api 送到後端, 經過處理檢查後將資料塞到 Oracle DB
- block 2 從該 Oracle DB 撈出所有的群組名稱, 帳號, 創建者, 創建時間, (不顯示密碼) 用表格呈現,每頁最多呈現20筆資料, 並包含一個 filter 視窗, 可以用關鍵字搜尋

## 後段
- api 1. 可以詢問 application.yaml config 了幾組 Oracle DB 並顯示於前端
- api 2. 可以撈 Oracle DB 的 Group_ALL table, 並回傳群組名稱, 帳號, 創建者, 創建時間
- api 3. 可以輸入 1.群組名稱 2.帳號  3. 密碼 將資料塞入 Oracle DB 的 Group_ALL table, 並根據 session 與時間自動帶入創建者,創建時間。