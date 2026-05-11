# Copilot PR Review — 雄獅內部使用說明

本 repo 配置了 GitHub Copilot 自動 code review,當有人開 PR 時 Copilot 會掃 diff 並留 inline comments。本文件說明這套配置在做什麼、會怎麼影響你、以及怎麼維護。

## 這是什麼

一組給 GitHub Copilot 的審查指引。檔案結構:

```
.github/
├── copilot-instructions.md            # 主檔,每個 PR 都會吃
└── instructions/
    ├── pcm.instructions.md            # 動到 PCM 路徑才疊加
    ├── schema-migration.instructions.md  # 動到 SQL/migrations 才疊加
    └── config.instructions.md         # 動到 appsettings/web.config 才疊加
docs/
└── lion-pr-review-full-spec.md        # 完整 spec,給 Copilot Chat 深審用
```

設計目的:讓 Copilot 在 PR 上**輔助 reviewer 發現可疑訊號**,不取代 reviewer 的判斷。Copilot 不會說「這個 PR 不能合」,只會標「這裡值得看一眼」。

## 你會看到什麼

### PR author 的視角

開 PR 後,Copilot 會被自動 (或手動) request review,在 Files changed 頁面留 inline comments。每條 comment 會帶嚴重度標記:

- 🔴 **Blocker** — 功能 bug / 資安風險 / 命中 🔴 級反模式。建議在合入前處理
- 🟡 **Should-fix** — 架構合規 / 跨系統影響 / 命中 🟡 級反模式。建議考慮處理
- 🔵 **Consider** — 設計選擇題,可討論
- 💡 **Nit** — 吹毛求疵,可忽略

**語氣紀律**:每條 comment 都是「建議」「可考慮」「值得確認」這類軟語氣,不會出現「必修」「不能合」。**最終合入決定是 reviewer 的,不是 Copilot 的**。

如果你不同意某條 comment,正常 reply / dismiss 即可,不必把 Copilot 的話當聖旨。

### Reviewer 的視角

Copilot 已經做完第一輪「機械式掃描」(SQL injection / 硬編碼密碼 / 跨 DB 直寫 / 高頻字典直寫等),你可以把時間放在:

1. 業務邏輯正確性 (Copilot 不太懂雄獅領域)
2. 跨系統影響評估 (Copilot 只標 cross-check 提醒,不代讀其他系統)
3. 判斷 Copilot 標的訊號是否屬實 (Copilot 約 30-50% 命中率,會誤標)

當 Copilot 的某條 comment 提到「reviewer may want to cross-check ERP schema」這類提醒時,意思是「動到了某張 table,但我 (Copilot) 不知道下游影響,你自己去確認」。

### 想要深審某個 PR

預設 Copilot 只審 D1 正確性 + D2 雄獅架構合規 + D4 安全。想要全面審 (含可讀性 / 測試 / 影響範圍) 時,在 Copilot Chat 中說:

```
依照 docs/lion-pr-review-full-spec.md,深審 PR #<N>
```

Chat 場景會載入完整 spec (6 個維度 + 結構化輸出格式),輸出一段總結式 review。

## 涵蓋的反模式

主檔覆蓋四條 🔴/🔵 級反模式,跨 PR 全域生效:

| ID | 嚴重度 | 內容 |
|---|---|---|
| AP-01 | 🔴 | 跨 DB SQL 繞過 API (Controller/Service 直寫 LionGroup* / MRP) |
| AP-02 | 🔴 | 硬編碼 credentials (literal password / token / apikey) |
| AP-03 | 🔴 | 直寫高頻 dictionary table (MLLAN00 / tptbm00) |
| AP-07 | 🔵 | Aspirational layer 增厚無 caller (新增抽象但沒人用) |

Path-scoped 細節檔在特定路徑命中時才疊加:

| 檔案 | 觸發路徑 | 反模式 |
|---|---|---|
| `pcm.instructions.md` | `**/gitpcm*/**` 等 PCM 路徑 | AP-04 Template-Override |
| `schema-migration.instructions.md` | `**/*.sql`, `**/Migrations/**` | AP-06 Schema 變更無同步文件 |
| `config.instructions.md` | `**/appsettings*.json`, `**/web.config` | AP-05 隱性開關被改沒交代 |

完整反模式定義 (含偵測訊號 / 合理例外 / 評論範本) 見 `docs/lion-pr-review-full-spec.md`。

## 維護指引

### 想新增一條反模式

1. 評估屬於通用 (每個 PR 都該掃) 還是路徑特定
2. **通用**:加進 `.github/copilot-instructions.md`。注意 4,000 字元上限——超過會被截掉
3. **路徑特定**:在 `.github/instructions/` 新增 `<scope>.instructions.md`,frontmatter 指定 `applyTo`
4. 同步更新 `docs/lion-pr-review-full-spec.md` 的完整定義
5. **改在 base branch (`main`)**:Copilot code review 讀 base branch 的 instructions,不讀 feature branch 的改動

### 想調整語氣或嚴重度紀律

主檔的 "Role and Tone" 與 "Severity Discipline" 段是行為的總開關。改這兩段會影響所有 PR 上 Copilot 的輸出風格,改動幅度大時建議先在測試 repo 驗證。

### 想關掉 Copilot review

Repo Settings → Copilot → Code review → 關閉「Use custom instructions when reviewing pull requests」(只關 instructions,Copilot 還是會用預設規則審),或在 Reviewers 移除 Copilot 並關閉自動 review。

## 已知限制

讀完比較不會失望:

1. **4,000 字元上限**:Copilot code review 只讀主檔前 4,000 字元,目前主檔卡在 3,997。新增規則必須先砍舊規則
2. **不是 100% 遵守**:Copilot 是非確定性的,同一個 PR 跑兩次可能標不同東西。每次 review 命中率不固定
3. **Low-risk 檔可能被略過**:Copilot 會自動跳過它認為低風險的檔 (config、generated code),這代表 `config.instructions.md` 可能不總是生效
4. **快取問題**:改 instructions 後,自動觸發的 review 可能仍用舊的快取版本。手動 request 重審才比較容易吃到新版
5. **輸出形態固定**:Copilot 自動 review 一定是逐行 inline comments,**不會給一段總結報告**。想要總結式輸出只能在 Chat 場景觸發
6. **不懂雄獅領域**:Copilot 不懂團旅成行邏輯 / PCM Template-Override / 搜尋 ES 字典維運。這些必須靠 reviewer 自己看,或用 cross-check 提醒交給對應 squad 確認
7. **沒有業務上下文**:Copilot 看不到 PR 對應的 ADO ticket / Sprint 規劃。是否符合需求、是否在 sprint 範圍內,必須由人類判斷

## 參考

- `docs/lion-pr-review-full-spec.md` — 完整 spec (給 Chat 深審用)
- [GitHub Copilot code review 官方文件](https://docs.github.com/copilot/using-github-copilot/code-review/using-copilot-code-review)
- [Custom instructions 最佳實踐](https://docs.github.com/en/copilot/tutorials/customize-code-review)
