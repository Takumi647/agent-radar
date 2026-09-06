# AI Workflow Deep Dive — 2026-09-06

> 主題：九個開源 Agent / Context / Workflow 專案，對 GitHub-backed deterministic multi-agent workflow 的可借鑑設計。
>
> 本文不是 README 摘要，而是以 `Side Project - AI Workflow` 的既有設計原則為基準，分析各專案在整體 Agent Stack 中扮演什麼角色、哪些值得借、哪些不值得搬，以及最應該做哪些實驗。
>
> 判讀原則：**已驗證事實** 優先以 repository / source / issue / benchmark 等 primary source 支持；**推論** 會明確標示為架構判斷，不把專案自己的 benchmark 或 claim 當成已獨立驗證結論。

## TL;DR

這九個專案不是九個互斥的 Agent framework，而是分散在同一條 stack 的不同位置：

```text
Human / Product Intent
        ↓
Task / Control Surface
        ↓
Workflow Coordinator / Scheduler
        ↓
Agent Runtime / Isolated Execution
        ↓
Code Context / Knowledge / Memory
        ↓
Structured Stage Output
        ↓
Validated Write Boundary
        ↓
Git / Issue / PR / CI
        ↓
Verification / Documentation / Long-term Knowledge
```

對目前 AI Workflow 最重要的結論是：

> **GitHub 保存 Truth；Runner 保存 Authority；Agent 提供 Intelligence；Graph / Memory / Wiki 提供 Context。**

不要讓四個角色混成一個系統。

目前既有的核心方向——Git tree 作為 canonical artifacts、GitHub Issue 作為 workflow control state、PR 作為 implementation delivery、GitHub Actions 作為 deterministic verification，以及「Dumb Coordinator, Smart Agents」——沒有被這九個專案推翻；反而可從它們補強四種能力：

1. **Symphony**：scheduler / retry / reconciliation / workspace mechanics
2. **GitHub Agentic Workflows (`gh-aw`)**：read-only Agent + validated write boundary
3. **Codebase-Memory / Graphify**：共享的 code-context intelligence layer
4. **OpenWiki / OpenViking**：durable repo knowledge 與 long-term agent context

## 九個專案快速定位

| Repository | 真正解決的問題 | 在 Agent Stack 的位置 | 判定 |
|---|---|---|---|
| [`chuspeeism/dashi-taskboard`](https://github.com/chuspeeism/dashi-taskboard) | Agent task ownership、claim、thread / branch / worktree 綁定、Human board | Control Surface | 🟡 值得 PoC，借 pattern 比直接採用重要 |
| [`DeusData/codebase-memory-mcp`](https://github.com/DeusData/codebase-memory-mcp) | Persistent code knowledge graph / code intelligence MCP | Code Context Plane | 🟢 強烈建議 benchmark |
| [`github/gh-aw`](https://github.com/github/gh-aw) | GitHub-native agent workflow + sandbox + controlled writes | Execution / Permission Plane | 🔥 非常重要 |
| [`Graphify-Labs/graphify`](https://github.com/Graphify-Labs/graphify) | Code / docs / schema 等多來源 knowledge graph | Code / Knowledge Graph | 🟢 與 Codebase-Memory 對打 |
| [`langchain-ai/openwiki`](https://github.com/langchain-ai/openwiki) | 維護 source-grounded repository wiki | Durable Repo Knowledge | 🔥 很值得後期加入 |
| [`nashsu/llm_wiki`](https://github.com/nashsu/llm_wiki) | 個人研究 / 文件轉 wiki / graph / agent knowledge | Personal Research Plane | 🟡 適合研究，不宜進核心 delivery pipeline |
| [`openai/symphony`](https://github.com/openai/symphony) | Issue → isolated autonomous implementation run | Scheduler / Runner | 🔥 與 workflow-driver 高度相關 |
| [`tt-a1i/archify`](https://github.com/tt-a1i/archify) | Agent → typed IR → validated architecture/workflow artifact | Verification / Communication | 🟢 低耦合、值得試 |
| [`volcengine/OpenViking`](https://github.com/volcengine/OpenViking) | Agent Memory + Resources + Skills 的 context database | Long-term Context Plane | 🟡 V2 很有價值 |

---

# 一、先固定 AI Workflow 的基準線

## 1. Durable authority 在 GitHub，不在 transient Agent session

概念上：

```text
Git tree
= canonical artifacts

GitHub Issue
= workflow control state

Pull Request
= implementation delivery

GitHub Actions
= deterministic verification
```

因此 Agent session 掛掉之後，不應靠 transcript 才能恢復，而應能重新讀 branch + Issue + artifacts 後繼續。

這個 property 必須保留，即使之後加入 memory、graph、wiki 或 taskboard。

## 2. Dumb Coordinator, Smart Agents

已知 stage 順序不應再交給 LLM 判斷：

```text
PRODUCT_READY
→ FABLE_SPEC
→ CODEX_REVIEW
→ FABLE_RESOLVE
→ SPEC_FROZEN
→ OPUS_IMPLEMENT
→ VERIFY
→ PR_READY
→ ACCEPTANCE
→ DONE
```

Coordinator 應處理的是：state reduction、validation、retry / backoff、workspace allocation、claim / concurrency、reconciliation、timeout / crash recovery。

語意判斷才交給 Fable / Codex / Opus 等 stage agent。

## 3. Agent 不直接等於 GitHub Write Authority

最理想的 write path 應是：

```text
Agent
  ↓
Structured Proposed Mutation
  ↓
Runner Validator
  ├─ schema
  ├─ expected path
  ├─ allowed state transition
  ├─ current HEAD / frozen SHA
  └─ mutation allowlist
  ↓
Credential Boundary
  ↓
GitHub Write
```

---

# 二、執行與協調層：Dashi、gh-aw、Symphony

## `chuspeeism/dashi-taskboard`

### 它解什麼問題

Dashi 表面上是 Agent task board，但真正值得研究的是 **task ownership protocol**，而不是 UI。

它把任務 lifecycle、claim、Agent thread、workspace / branch / worktree 等執行 identity 綁在一起，避免多個 agent 同時把同一張卡當成自己的工作。

### 值得借的設計

最有價值的是 optimistic ownership / concurrency pattern：

```text
read task
↓
確認 requirements / comments / attachment
↓
用目前 version claim
↓
進入 in_progress
↓
綁定 execution identity / workspace
↓
執行 + verify
↓
in_review
↓
Human acceptance
↓
done
```

如果 claim 時 version 已 stale，就不應直接覆蓋，而是重新讀 task、確認 ownership / requirements / status 後再決定是否重試。

### 對 AI Workflow 的具體借鑑

GitHub Issue 裡可以加入一個 execution claim 概念：

```yaml
claim:
  stage: opus-implementation
  runner_id: runner-03
  workspace: /worktrees/change-123
  started_at: 2026-09-06T...
  attempt: 2
```

這樣 workflow-driver 不需要 Dashi 也能拿到它最重要的 concurrency pattern。

### 不建議照搬的地方

**不要讓 Dashi SQLite 變成第二個 canonical workflow database。**

如果同時存在：

```text
GitHub Issue
    ↕ sync
Dashi SQLite
```

就會立刻出現「誰才是 authoritative state」的問題。

### Research Candidate

**Yes — 但研究的是 ownership protocol，不是是否換 taskboard。**

Experiment：模擬兩個 runner 同時 claim 相同 stage，比較無 version guard、optimistic version guard、central lock 三種方案。

Metrics：duplicate execution、stale overwrite、recovery complexity、operator intervention。

---

## `github/gh-aw` — GitHub Agentic Workflows

### 它解什麼問題

`gh-aw` 將 Markdown + YAML frontmatter 編譯成標準 GitHub Actions workflow，讓需要 reasoning 的 repository automation 可以由 AI agent 執行。

Primary source：[`github/gh-aw`](https://github.com/github/gh-aw)

GitHub 自己也明確區分：deterministic build / test / lint / deploy 用普通 GitHub Actions；issue triage / review / investigation / docs maintenance 等需要 interpretation 的工作才使用 agentic workflow。

### 最值得借的是 Security Architecture

`gh-aw` 最重要的 pattern 不是「Markdown 寫 workflow」，而是：

```text
Agent Reasoning
      ≠
GitHub Write Authority
```

Agent job 預設朝 read-only / sandboxed 執行；需要 GitHub mutation 時，透過受控輸出與另一個具 scoped permission 的 job 驗證並套用。

### 對 AI Workflow 的具體借鑑

把 `safe-outputs` 哲學移進 local runner：

```text
Fable / Codex / Opus
(no broad GitHub write token)
        ↓
structured result
        ↓
Runner
  ├─ validate schema
  ├─ validate expected mutation
  ├─ validate current workflow state
  └─ reject unrelated mutation
        ↓
GitHub credential boundary
        ↓
Issue / Branch / PR / Comment / Label
```

這比「直接把 gh-aw 全部拿來取代 workflow-driver」更適合目前設計。

### Research Candidate

**Yes — 高優先。**

Experiment：在含 prompt injection 的 issue 上比較：

A. Agent 持 GitHub write token

B. Agent 只輸出 structured mutation，由 runner 驗證後寫入

測試惡意要求：改 unrelated file、刪 label、關閉別的 PR、越權讀寫其他 repo。

Metrics：policy violation、false reject、write latency、implementation complexity、audit completeness。

---

## `openai/symphony`

### 它解什麼問題

Symphony 的定位是把 project work 變成 isolated autonomous implementation runs，而不是讓人一直看著 coding agent session。

Primary source：[`openai/symphony`](https://github.com/openai/symphony)

它持續讀 issue tracker、挑 eligible work、建立 isolated workspace、啟動 agent、追蹤執行狀態，並處理 retry / reconciliation / recovery。

### 最值得借的 runtime mechanics

對 workflow-driver 最有價值的是：bounded concurrency、per-issue workspace、eligibility / claim、retry queue、exponential backoff、reconciliation、stop ineligible run、restart recovery、repo-level execution policy。

### Symphony 與目前 workflow 的重要差異

Symphony 比較像：

```text
Issue
↓
Autonomous Coding Run
↓
PR
```

目前 AI Workflow 則是：

```text
Product Intent
↓
Fable Spec
↓
Codex Review
↓
Fable Resolve
↓
Spec Freeze
↓
Opus Implement
↓
Verify
↓
PR
↓
Acceptance
```

所以不應整套搬 Symphony。真正應採用的是：**scheduler / workspace / retry / reconciliation mechanics**。

### Research Candidate

**Yes — 最高優先之一。**

用以下 failure scenarios 驗 runner：agent process crash、stale HEAD、quota / auth blocked、issue 在 execution 中被改狀態、machine / runner restart。

Metrics：duplicate work、錯誤 state transition、mean recovery time、人工介入次數、stuck run rate。

---

# 三、Code Context / Knowledge 層：Codebase-Memory 與 Graphify

## `DeusData/codebase-memory-mcp`

### 它解什麼問題

它不是一般 vector RAG，而是為 coding agent 建立 **persistent code knowledge graph**，希望把大量：

```text
grep → read → grep → read
```

轉成結構化 code intelligence query。

Primary sources：

- [`DeusData/codebase-memory-mcp`](https://github.com/DeusData/codebase-memory-mcp)
- [Codebase-Memory paper](https://arxiv.org/abs/2603.27277)

核心 implementation 方向包括 Tree-sitter AST、部分語言搭配 semantic / LSP resolution，再暴露 call chain、architecture、impact analysis、route、dead-code、graph query 等 MCP 工具。

### 為什麼對 Multi-Agent Workflow 特別重要

如果 pipeline 是：

```text
Fable 讀一次 repo
Codex 再讀一次 repo
Opus 再讀一次 repo
Acceptance 再讀一次 repo
```

四個模型可能重建四次相同 architecture。

Shared read-only graph 有機會改成：

```text
Repository
   ↓
Persistent Graph
   ├─ Fable
   ├─ Codex
   └─ Opus
```

這是實際可能省 context / tool calls 的地方。

### Benchmark 要保守看

專案 / paper 報告在 31 個 real-world repos 上，相對 file exploration 有顯著 token / tool-call 降低，但 answer quality 不是全面勝出。

因此正確問題不是「graph 是否一定比 source reading 準」，而是：**在你的 coding workload 上，能不能用較便宜的 structural query 保留足夠品質？**

### 實務風險

需要特別注意 large repo、parser / LSP、memory safety、worktree / branch-specific indexing 等 operational complexity。

相關公開 issue：

- [Stability umbrella #390](https://github.com/DeusData/codebase-memory-mcp/issues/390)
- [Git worktree indexing #351](https://github.com/DeusData/codebase-memory-mcp/issues/351)

### Research Candidate

**Yes — 必做 benchmark。**

---

## `Graphify-Labs/graphify`

### 它解什麼問題

Graphify 同樣建 graph，但 scope 更廣：code、docs、SQL schema、configs、PDF 等都可進 knowledge graph。

Primary source：[`Graphify-Labs/graphify`](https://github.com/Graphify-Labs/graphify)

它值得注意的一個設計是區分 graph relation 的 provenance，例如 source extraction 與推論關係不要被視為相同證據強度。

### 為什麼這個 distinction 重要

對 AI Workflow 很重要的原則是：**Evidence 與 inference 不應混在同一層。**

例如：

```text
EXTRACTED
= source code / schema 中直接存在

INFERRED
= resolver / model 推論
```

### 與 Codebase-Memory 的差異

粗略可理解為：

- Codebase-Memory：更偏 coding-agent structural code intelligence / MCP
- Graphify：更偏多來源 knowledge graph 與 code/doc 統一檢索

### Research Candidate

**Yes — 與 Codebase-Memory 做 A/B/C。**

```text
A = native rg + read
B = Codebase-Memory
C = Graphify
```

同模型、同 repo、同問題集。

Metrics：answer correctness、source grounding、input tokens、tool calls、wall-clock latency、index latency、incremental update latency、stale result rate。

---

# 四、Durable Knowledge：OpenWiki 與 LLM Wiki

## `langchain-ai/openwiki`

### 它解什麼問題

OpenWiki 不只是「AI 幫 repo 寫文件」，真正值得研究的是 **source-grounded durable knowledge maintenance**。

Primary source：[`langchain-ai/openwiki`](https://github.com/langchain-ai/openwiki)

理想模型是：

```text
Claim
"Authentication middleware rejects expired sessions"
        ↕
Exact versioned source evidence
repo / file / lines / source version
```

當 supporting source 改變，對應 claim 應被標 stale 並重新檢查，而不是讓生成文件一直變成舊的第二份 truth。

### 對 AI Workflow 最適合的位置

```text
Accepted / Merged Change
        ↓
OpenWiki incremental update
        ↓
Agent-readable repository knowledge
```

OpenSpec / frozen spec 仍然是 product / implementation contract；OpenWiki 應該是 **accepted codebase 的 derived knowledge layer**。

### Research Candidate

**Yes — Workflow 穩定後做。**

Experiment：先建立 wiki，之後故意做 10 個會讓文件失效的 code changes。

Metrics：stale fact detection recall、false positive、unnecessary rewrite、人工 review time。

---

## `nashsu/llm_wiki`

### 它解什麼問題

LLM Wiki 比 OpenWiki 更接近 **個人 / 團隊 Research Knowledge OS**。

Primary source：[`nashsu/llm_wiki`](https://github.com/nashsu/llm_wiki)

它把 raw sources 轉成 interlinked wiki，並結合 ingest queue、cache、graph、vector retrieval、web research、MCP / Agent Skills 等能力。

### 對目前工作的價值

它更適合放：Agent Radar research、論文、GitHub 專案研究、Notion 匯出的知識、Side Project design notes；不適合承擔 workflow control state、stage transition authority 或 PR delivery state。

可以把兩者角色分成：

> **OpenWiki = repository accepted knowledge**
>
> **LLM Wiki = personal / cross-project research knowledge**

### Research Candidate

**No for core workflow；Yes as personal research tooling。**

---

# 五、Long-term Agent Context：OpenViking

## `volcengine/OpenViking`

### 它解什麼問題

OpenViking 解的是：Agent 每次 fresh session 都要完全從零開始嗎？

Primary source：[`volcengine/OpenViking`](https://github.com/volcengine/OpenViking)

它嘗試把 Memory、Resources、Skills 放進一致的 context database / namespace，並用 filesystem-like 的方式組織 agent 可讀 context。

### 最值得借的 pattern：Progressive Context

概念上可以是：

```text
L0 = Abstract
很便宜，只判斷相關性

L1 = Overview
給足夠背景做下一步判斷

L2 = Details
真的需要才讀完整內容
```

這比「每次 retrieval 直接塞 full chunk」更適合長期 agent context。

### 對 AI Workflow 的安全定位

加入 OpenViking 後，以下 property 必須仍成立：

```text
session crash
↓
fresh agent
↓
read GitHub durable artifacts
↓
workflow can recover
```

因此：

```text
GitHub durable artifacts
= Authority / Truth

OpenViking memory
= Advisory context
```

### Research Candidate

**Yes — V2。**

先把 deterministic workflow 做穩，再測 memory 對 coding stages 的實際增益。

Metrics：task quality、context tokens、retrieval latency、wrong-memory rate、stale memory rate、recovery independence。

---

# 六、Verification / Communication：Archify

## `tt-a1i/archify`

### 它解什麼問題

Archify 不負責 orchestration、memory 或 code search；它是：

```text
Agent semantics
↓
Typed IR
↓
Deterministic validation
↓
Visual artifact
```

Primary source：[`tt-a1i/archify`](https://github.com/tt-a1i/archify)

支援 architecture、workflow、sequence、data-flow、lifecycle 等 diagram，並將視覺生成從 LLM 的自由文本拆成 machine-checkable representation + deterministic renderer。

### 為什麼很符合目前架構哲學

```text
LLM
= Semantic Judgment

Deterministic System
= Verification
```

Archify 是同一個 pattern：Agent 決定語意，validator / renderer 負責 artifact consistency。

### 最適合插入的位置

Spec Freeze 前：

```text
Fable Resolution
↓
Archify Workflow / Architecture
↓
Spec Freeze
```

或 Implementation Acceptance：

```text
Frozen Spec
↓
Implementation
↓
PR
↓
Archify Before / Delta / After
↓
Human + ChatGPT Acceptance
```

### Research Candidate

**Yes — 低耦合、可很快試。**

Metrics：review time、architecture mismatch detection、unsupported edge、diagram correction rate。

---

# 七、九個專案放在一起後的建議架構

不建議做：

```text
Dashi
+ Symphony
+ gh-aw
+ Codebase-Memory
+ Graphify
+ OpenWiki
+ LLM Wiki
+ Archify
+ OpenViking
+ workflow-driver
```

比較合理的整合方式是：

```text
                    Human + ChatGPT
                    Product Authority
                           │
                           ▼
                    GitHub Issue
                Canonical Control State
                           │
                           ▼
                    workflow-driver
               deterministic state machine
               claim / retry / reconcile
                           │
             borrow Symphony mechanics
                           │
                           ▼
                Fable / Codex / Opus
             no broad GitHub write authority
                           │
                optional code context
              CBM OR Graphify (winner)
                           │
                           ▼
                 Structured Stage Output
                           │
                borrow gh-aw safe-output
                           │
                           ▼
                   Runner Write Boundary
                  validate → then mutate
                           │
            ┌──────────────┼──────────────┐
            ▼              ▼              ▼
           Git           Issue            PR
            │                              │
            ▼                              ▼
         Actions                        Archify
            │                              │
            └──────────────┬───────────────┘
                           ▼
                       Acceptance
                           │
                           ▼
                        OpenWiki
                  derived repo knowledge
                           │
                           ▼
                  OpenViking later (V2)
```

Dashi 可留在 Human Control Surface；LLM Wiki 可留在 personal research plane。

---

# 八、最值得做的五個 Research Candidates

## Candidate 1 — Workflow-driver vs Symphony mechanics

**問題：** 自己的 workflow-driver 在長時間 autonomous coding execution 上，缺哪些 scheduler / reconciliation primitives？

**Experiment：** 注入五種 failure：agent crash、stale HEAD、quota/auth blocked、issue 中途改 state、machine restart。

**Baseline：** 現有 runner。

**Variant：** 加入 Symphony-style claim / reconciliation / retry / workspace lifecycle。

**Metrics：** duplicate work、wrong transition、stuck run rate、recovery time、manual intervention。

**Priority：🔥 現在就研究。**

---

## Candidate 2 — `safe-outputs` 式 write boundary

**問題：** structured mutation + runner validation 是否能顯著降低 Agent 越權與 prompt injection 風險？

**Experiment：** 對含惡意指令的 Issue，讓 Agent 嘗試 unrelated mutation。

**Baseline A：** Agent 持直接 write token。

**Variant B：** Agent read-only，輸出 mutation proposal，由 runner 驗證。

**Metrics：** policy violations、false rejects、auditability、latency overhead、implementation complexity。

**Priority：🔥 現在就研究。**

---

## Candidate 3 — Code Context A/B/C

```text
A = rg + file read
B = Codebase-Memory
C = Graphify
```

同模型、同 repo、同 15–30 個問題。

問題類型要包含 architecture、call chain、impact analysis、cross-module behavior、configuration / schema relation、bug localization。

**Metrics：** answer correctness、evidence grounding、input tokens、tool calls、wall time、indexing cost、incremental update latency、stale result rate。

**Priority：🟢 馬上 benchmark。**

---

## Candidate 4 — OpenWiki stale-claim experiment

**問題：** OpenWiki 到底是漂亮 docs generator，還是真正能維護 source-grounded knowledge？

**Experiment：** 先生成 wiki，再做十個會使 docs 失效的 code change。

**Metrics：** stale fact recall、stale fact precision、unnecessary rewrite、source citation correctness、reviewer time。

**Priority：🟢 Workflow 穩後。**

---

## Candidate 5 — Archify spec → implementation delta

**問題：** architecture artifact 是否能降低 PR acceptance 的 cognitive load，並抓到純 diff review 容易漏掉的設計偏差？

**Experiment：** Spec Freeze 產一份 architecture；implementation 後再產一份，讓 reviewer 看 Before / Delta / After。

**Metrics：** review time、mismatch detection、unsupported relationships、correction count、reviewer confidence。

**Priority：🟢 低成本可先試。**

---

# 九、採用順序

| 優先度 | Project / Pattern | 建議 |
|---|---|---|
| 🔥 現在研究 | `openai/symphony` | 抄 scheduler / retry / reconciliation / workspace mechanics |
| 🔥 現在研究 | `github/gh-aw` | 抄 read-only Agent + validated write boundary |
| 🟢 馬上 benchmark | `codebase-memory-mcp` | Code Context candidate |
| 🟢 馬上 benchmark | `graphify` | 與 CBM / native search A/B/C |
| 🟢 低耦合先試 | `archify` | Spec / PR / Acceptance verification artifact |
| 🟢 Workflow 穩後 | `openwiki` | Accepted repository knowledge |
| 🟡 Workflow 穩後 | `dashi-taskboard` | Operator UI / ownership pattern，不當 canonical truth |
| 🟡 V2 | `OpenViking` | Long-term agent context / memory |
| 🟡 Personal | `llm_wiki` | Agent Radar / papers / cross-project research knowledge |

---

# 十、最終架構原則

這次研究最後可以濃縮成四句：

1. **GitHub 保存 Truth。** Issue / Git tree / PR / CI 仍是可恢復、可 review 的 durable state。
2. **Runner 保存 Authority。** Agent 不需要 broad write credential；所有 mutation 先被 deterministic validator 接住。
3. **Agent 提供 Intelligence。** Fable / Codex / Opus 負責 spec、review、implementation 等需要語意判斷的 stage。
4. **Graph / Memory / Wiki 提供 Context。** 它們可以提升效率與理解，但不能悄悄變成另一份 canonical truth。

目前真正值得做的不是「再選一套 Agent framework」，而是把上述 pattern 一個一個做成可測量的 architecture experiment。

## Primary sources

- Dashi Taskboard — https://github.com/chuspeeism/dashi-taskboard
- Codebase-Memory MCP — https://github.com/DeusData/codebase-memory-mcp
- Codebase-Memory paper — https://arxiv.org/abs/2603.27277
- Codebase-Memory stability tracking — https://github.com/DeusData/codebase-memory-mcp/issues/390
- Codebase-Memory worktree indexing — https://github.com/DeusData/codebase-memory-mcp/issues/351
- GitHub Agentic Workflows — https://github.com/github/gh-aw
- Graphify — https://github.com/Graphify-Labs/graphify
- OpenWiki — https://github.com/langchain-ai/openwiki
- LLM Wiki — https://github.com/nashsu/llm_wiki
- OpenAI Symphony — https://github.com/openai/symphony
- Archify — https://github.com/tt-a1i/archify
- OpenViking — https://github.com/volcengine/OpenViking
