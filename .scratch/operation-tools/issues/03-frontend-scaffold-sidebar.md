# 03: Frontend 基礎建設 — 專案骨架 + App Shell + Sidebar + Empty State

**What to build:** React + Vite + TypeScript 專案從零建起。`npm run dev` 啟動後，畫面呈現 Variant B (Sidebar Dashboard) 的基本骨架：左側深色 Sidebar（#1a237e）列出 mock DB 清單，使用者可以點選切換（高亮 selected），Sidebar 底部顯示 Avatar + "KUO HAN YANG"。未選擇任何 DB 時，主區域顯示置中的 Empty State 提示（icon + 文字「請從左側選擇一個 Database」）。

此 ticket 的 `api/index.ts` 定義好所有 function signature，但暫時回傳 hardcoded mock data（與 Ticket 04 共用），不呼叫後端。

**Blocked by:** None (can start immediately)

**Group:** FRONTEND

**Status:** ready-for-agent

**UI 參考：** `prototype-ui.html` 中的 Variant B (Sidebar Dashboard) 佈局。

**Types（來自 spec）：**
```typescript
interface DatabaseInfo { id: string; name: string; }
interface GroupAccount {
  id: number; groupName: string; account: string;
  createdBy: string; createdAt: string;
}
interface GroupAccountRequest {
  groupName: string; account: string; password: string;
}
```

- [ ] `package.json` — react, react-dom, @mui/material, @emotion/react, @emotion/styled, @mui/icons-material；dev: vite, @vitejs/plugin-react, typescript
- [ ] `vite.config.ts` — dev proxy `/api` → `localhost:8080`，build outDir 指向 `../backend/src/main/resources/static`
- [ ] `tsconfig.json` 基本設定
- [ ] `index.html` 入口 HTML
- [ ] `main.tsx` — React root + MUI ThemeProvider
- [ ] `theme.ts` — MUI theme（primary: #1976d2，sidebar 深色 #1a237e）
- [ ] `types/index.ts` — DatabaseInfo, GroupAccount, GroupAccountRequest 型別定義
- [ ] `api/index.ts` — `fetchDatabases()`, `fetchGroups(dbId)`, `createGroup(dbId, data)` 三個 function，暫時回傳 hardcoded mock data
- [ ] `App.tsx` — 頂層 state（selectedDb, databases），啟動時呼叫 fetchDatabases，傳 props 給子元件
- [ ] `Sidebar.tsx` — MUI Drawer variant="permanent"，寬 260px，深色背景，DB 列表用 ListItemButton（selected 高亮），底部 Avatar + user name
- [ ] `EmptyState.tsx` — 置中 icon + 文字提示
- [ ] `npm run dev` 啟動後畫面正確呈現 Sidebar + Empty State
- [ ] 點選 DB 後 Sidebar 高亮切換正確
