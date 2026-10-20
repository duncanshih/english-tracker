# English Tracker — 英文 Small Talk 訓練追蹤

Duncan 的英文口說練習追蹤網站。每次用 ChatGPT live 練完 small talk 後，把它產出的 lesson note (.md) 丟進指定資料夾，網站就會自動更新：痛點追蹤、錯誤計次、句型庫、複習卡、Replay 練習、筆記原文。

- **線上網址**：https://duncanshih.github.io/english-tracker/
- **配套教學站**：[Small Talk Pro 句型速查](https://duncanshih.github.io/small-talk-pro/Small_Talk_Pro_%E5%8F%A5%E5%9E%8B%E9%80%9F%E6%9F%A5.html)（網站內的「📖 教學」連結會深連到對應章節）

## 架構

```
english-tracker/
├── index.html   ← 網頁程式（module script，fetch data.json 渲染）。更新資料時不要動它
├── data.json    ← 全部結構化資料，唯一需要編輯的檔案（嚴格 JSON）
├── notes/       ← 每堂課 lesson note 的原文，「筆記」分頁會載入渲染
└── README.md
```

### data.json 主要欄位

| 欄位 | 內容 |
|---|---|
| `sessions` | 每堂課：id（S1、S2…）、date、label、topics |
| `patterns` | 結構性痛點 × 場次表現（history：bad/ok）、status（active/improving/resolved）、`learn`（教學站章節錨點） |
| `errors` | 文法/用法錯誤：wrong / right / rule / tag；**sessions 陣列長度＝出現次數**（重複犯的排最前） |
| `phrases` | 句型庫：sc（場景）/ en / zh / star |
| `replays` | 練習題：q / a / note（上限 14 題，舊的且痛點已解決者先淘汰） |
| `liveTargets`、`priorities`、`prep` | Live Target 演進、本週優先、上課前檢查清單（以最新一堂課為準） |
| `notes` | 筆記原文清單：sid ↔ notes/ 檔名 |
| `sourceFiles` | 已收錄的 lesson note 檔名（同步比對用） |
| `teach` | 教學站網址、章節表、場景→章節對照（sceneMap） |

## 同步機制（為什麼 repo 裡沒有 sync script / GitHub Action）

同步**不是**跑在 GitHub 上，而是本機 Claude Code 的排程任務 `english-lesson-note-sync`（每天 21:30，任務定義在 `C:\Users\Duncan Shih\.claude\scheduled-tasks\english-lesson-note-sync\SKILL.md`）：

1. 掃描 `OneDrive\桌面\英文\small talk 準備` 裡檔名含 **lesson** 的 .md 檔
2. 與 `data.json` 的 `sourceFiles` 比對，找出未收錄的筆記
3. 用 LLM 解析筆記內容，合併進 `data.json`（錯誤同規則要合併計次，不重複開條目）
4. 複製原文到 `notes/`，以 `node -e "JSON.parse(...)"` 驗證 JSON，然後 commit + push
5. GitHub Pages 自動重新部署

不用 GitHub Action 的原因：(a) 資料來源在本機 OneDrive，Action 讀不到；(b) 解析筆記（判斷痛點趨勢、錯誤是否同規則該合併計次）需要 LLM 判斷，不是確定性腳本能做的。repo 只存「結果」。

## Lesson note 命名慣例

- **規則**：檔名含 `lesson`（不分大小寫）的 .md 才會被收錄；其他檔案（prompt 設定檔、STATE 檔）一律忽略
- **建議格式（新筆記）**：`Lesson_Note_YYYY-MM-DD.md`，同一天多場加後綴：`Lesson_Note_YYYY-MM-DD_2.md` 或主題：`Lesson_Note_YYYY-MM-DD_Restaurant_Week.md`
- 歷史檔名不追溯更名：`sourceFiles` 以檔名比對，改名會造成重複收錄

## 本機開發注意

- `index.html` 以 fetch 載入 `data.json` 與 `notes/*.md`，**直接雙擊開檔案會被瀏覽器 CORS 擋掉**，請用線上網址或本機 HTTP server 檢視
- 改完 `data.json` 務必驗證：`node -e "JSON.parse(require('fs').readFileSync('data.json','utf8'))"`
- `.nojekyll` 不能刪：沒有它 GitHub Pages 的 Jekyll 會處理掉 `notes/*.md`，原文會 404
- 複習卡與檢查清單的勾選進度存在各裝置的 localStorage，不在 repo 裡
