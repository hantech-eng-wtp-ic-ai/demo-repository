# CLAUDE.md — demo-repository

## 這個 repo 是什麼

`hantech-eng-wtp-ic-ai` 組織的**治理範本 repo**。表面上是 GitHub 的 demo 網頁專案
（`index.html` + `package.json`），實際用途是驗證組織層級的審查流程：
CODEOWNERS 路由、branch protection、CI 關卡。

改動時請意識到：這裡的 `.github/` 內容是要被複製到其他正式 repo 的範本，
不是只服務這個 repo 自己。

## 五角色 RACI 審查路由

`.github/CODEOWNERS` 依路徑指派審查者。修改任何檔案前，先確認會觸發哪個角色：

| 路徑 | 審查者 Team |
|---|---|
| `*`（fallback） | `tech-lead` |
| `/rules/`, `/rule-tables/`, `*.xlsx` | `ontology-owner` |
| `/golden-master/` | `ontology-owner` **+** `pe-signoff`（設計上為雙審） |
| `/approvals/`, `/sign-off/` | `pe-signoff` |
| `/src/`, `/api/`, `/tests/` | `tech-lead` |
| `/workflows/`, `/n8n/` | `process-operator` |
| `/.github/workflows/`, `/dags/` | `tech-lead` |
| `/docs/architecture/`, `/DOC-2026-WTP-AI-002*` | `eng-manager` |
| `/.github/CODEOWNERS`, `/.github/PULL_REQUEST_TEMPLATE.md` | `eng-manager` |

比對邏輯同 `.gitignore`：**由上而下比對，最後一條命中的規則生效**。
新增規則時放在檔案下方才會覆蓋前面的規則。

五個 Team 均已在組織中建立（`eng-manager` / `ontology-owner` / `pe-signoff` /
`process-operator` / `tech-lead`），CODEOWNERS 的 Team 參照有效。

### ⚠️ 目前 RACI 路由「未被強制執行」

ruleset `main-protection-standard`（id 21277118）的 pull_request 規則實際值為：

```
require_code_owner_review: false          ← CODEOWNERS 不是必要審查
required_approving_review_count: 1        ← 任何一人核准即可合併
```

也就是說：CODEOWNERS 目前只會**自動把對應 Team 加進 reviewer 清單（提示性質）**，
但不會擋合併。任何一位有寫入權的成員核准就能 merge，
`/golden-master/` 的「Ontology Owner + PE Sign-off 雙審」在技術上**尚未生效**。

要讓 RACI 真正落地，需由組織管理者（org owner / repo admin）調整 ruleset：
1. 開啟 `Require review from Code Owners`
2. `/golden-master/` 的雙審需把 `required_approving_review_count` 提高到 2，
   或改用 branch ruleset 的 required_reviewers 指定 Team

在這項調整完成前，**不要假設路徑對應的角色一定看過該變更**。

### 職責分離的紅線

- **Ontology Owner ≠ PE Sign-off**。內容擁有者不能自己簽核自己的變更，
  `/golden-master/` 的雙審設計就是為此。不要為了「加快流程」把這兩個角色合併或改成單審。
- **治理檔案自我保護**。`CODEOWNERS` 本身指定由 `eng-manager` 審查，
  避免有人改掉審查規則卻不用被審查。這是刻意設計，不是設定錯誤。

## CI 關卡

| Workflow | 觸發 | 內容 |
|---|---|---|
| `Proof HTML` | 每次 push、手動 | `anishathalye/proof-html@v1.1.0` 檢查根目錄 HTML |
| `CI Regression Test` | 對 `main` 的 PR | job 名 `ci/regression-test`，目前是 placeholder |

### `ci/regression-test` 的名稱不可更動

已確認 ruleset 的 required status check 就是綁 `ci/regression-test`：

```
required_status_checks: [{ context: "ci/regression-test" }]
strict_required_status_checks_policy: true
```

這個 job 現在只 `echo` 一行，看起來像可以隨便改的 placeholder，
但 **job 的 `name:` 被 branch ruleset 當作 required check 引用**。
改掉名字會讓所有 PR 找不到對應 check 而永久卡在 pending。

**步驟內容可以整個換掉（要接的是 Golden Master test runner），但 `name: ci/regression-test` 必須保留。**

`strict` 為 true 代表：分支必須與 `main` 同步到最新才能合併，落後就要先 rebase/merge。

### `Proof HTML` 會驗證連結

每次 push 都跑，且會檢查 HTML 內的連結可達性。
在 `README.md` 或 `index.html` 加外部連結時，死連結會直接讓 CI 紅燈。

## main 分支的其他限制

ruleset `main-protection-standard` 另外啟用了：

- `deletion` / `non_fast_forward` — 禁止刪除 `main`、禁止 force push
- `dismiss_stale_reviews_on_push: true` — 新 push 會作廢既有核准，需重新審查
- `require_last_push_approval: true` — **最後一次 push 的人不能是唯一核准者**，
  自己 push 後必須由他人核准，無法自審自合
- `required_review_thread_resolution: true` — 所有 review comment 必須 resolve 才能合併
- `require_extra_approval_for_unattributed_changes: true`

實務影響：**你無法獨自完成一個 PR 的合併**，一定需要另一位成員核准。

## 慣例

- **Commit 訊息**：Conventional Commits（現有紀錄為 `chore: ...`）。
- **分支**：從 `main` 開功能分支，經 PR 合併。`main` 有 branch protection，不要直接 push。
- **語言**：治理文件與註解使用繁體中文，程式碼識別字用英文。

## 環境

開發在 `alice@HE08-PC`（Ubuntu 22.04 WSL2）上，透過 VS Code remote-ssh 連入。
repo 路徑 `~/projects/demo-repository`。
