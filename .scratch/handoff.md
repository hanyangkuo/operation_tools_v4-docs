# Handoff Context: AI-Driven Development Presentation

## 1. 專案背景與核心故事 (The Story)
我們正在準備一份向工程團隊分享的技術簡報，主題是**「重塑開發工作流：掌握邊界控制與架構思維」**。
故事的主軸是我們在開發 `Operation Tools` 專案時，歷經了 v1 到 v3 的慘痛失敗，最後在 v4 取得空前成功（將原本 >2 小時且充滿 bug 的流程，縮短到 20 分鐘極速交付）。

## 2. 破局的兩大核心洞察 (The Core Insights)
這份簡報的靈魂在於向團隊傳達這兩個 Agentic Coding 的進階心法：
1. **殘酷的減法 (Ruthless Scope Reduction)：** AI 很容易過度設計 (Over-engineering，例如偷偷幫你加微服務架構、分散式交易等)。我們必須透過 `/grill-me` 提問與設立 `CLAUDE.md` 開發鐵律，嚴格限制 MVP 範圍。連最複雜的 OIDC 認證，都必須定下「禁止發明，只能套用現有範本」的規矩。
2. **物理隔離切票 (Horizontal/Physical Isolation)：** 傳統 `/to-tickets` 會切出橫跨前後端的「垂直切片」。這會導致單一 AI Context 肥大，或多個 AI 平行開發時引發嚴重的檔案衝突 (Merge Conflicts)。v4 成功的關鍵是人類介入，強迫將票依據目錄 (`frontend/` vs `backend/`) 徹底切開，實現完美的平行開發。

## 3. 目前狀態 (Current State)
- `presentation.html`: 已使用 Reveal.js 寫出 9 頁的簡報骨架，包含了上述的核心故事。
- `報告流程.md`: 講稿與 Demo 備忘錄，包含何時該切換出去展示 `prototype.html` 等互動環節。
- `project-description-v5-hardened.md`: 作為教材的附加文件，展示如何寫出一份帶有「絕對禁止清單」的防彈規格書。

## 4. 目標 (Next Steps in Claude Desktop)
使用者希望在 Claude Desktop 中繼續優化這份簡報。可能的方向包含：
- 潤飾 `presentation.html` 的文案與排版。
- 增加更生動的比喻或視覺化特效。
- 完善演講者的講稿細節。
