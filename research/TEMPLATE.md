# Agent Radar — YYYY-MM-DD

## 每日摘要

用幾個重點整理當天最重要的進展。聚焦「真正發生了什麼變化」以及「哪些值得持續追蹤」，不要為了湊篇幅重複沒有新進展的項目。

## 重要進展

### 1. 專案 / 論文 / Release 名稱

**類型：** Repository / Paper / Release / Engineering Post / Security Research / 其他  
**來源：** 優先放第一手來源連結。

#### 發生了什麼

精確描述這次的新變化。

#### 為什麼重要

說明它對 Agent Engineering 或 AI Security 的實際影響，不要只描述公告本身很有趣。

#### 技術機制

解釋讓這項進展具技術價值的 architecture、algorithm、runtime behavior、interface、benchmark 或 implementation detail。

#### 安全影響 / 攻擊面

說明這項進展是否新增、降低或改變安全風險。適用時至少考慮：

- Direct / Indirect Prompt Injection
- Jailbreak 或 instruction-boundary failure
- Tool Abuse / Confused Deputy
- Credential / Secret Exposure 與 Data Exfiltration
- RAG / Context / Memory Poisoning
- MCP、Tool、Skill、Plugin Supply-chain Risk
- Agent-to-Agent Trust 與 Delegated Authority
- Unsafe Code Execution / Sandbox Boundary
- Provenance、Output Validation、Least Privilege

必須明確區分：

- 已確認 vulnerability
- 有證據支持的風險
- 合理但尚未驗證的 threat-model inference

#### 與既有方法的差異

盡可能與最接近的 pattern、system、defense 或前一版設計做比較。

#### 證據與信心

說明判斷依據，並標出重要 uncertainty、缺少的 benchmark、尚未驗證 claim 或 vendor-reported result。

#### 可做的實驗

若值得實驗，提出具體 reproduction、benchmark、architecture comparison、安全評測或 implementation test。Adversarial testing 必須限制在可控、授權的環境內。

**研究候選：是 / 否**

---

## 既有專案的重要更新

只有專案真的有實質變更時才放進來。不要重複前幾天完全沒有新進展的項目。

## Security Radar

當天若有值得獨立追蹤的 AI / Agent Security 進展，就放在這一節。優先：security advisory、論文、可重現 evaluation、實際 implementation change，而不是泛泛的安全評論。

每個 Security 項目至少整理：

- Threat Model
- Attacker-controlled Input / Trust Boundary
- Preconditions
- Affected Component
- Impact
- Defense / Mitigation
- Remaining Bypass / Trade-off
- Evidence Quality

## Research Candidates

只列出真的值得做下一步實驗的候選。

### Candidate：名稱

- **問題：** 我們到底想知道什麼？
- **為什麼現在值得測：** 為什麼現在有研究價值？
- **建議實驗：** 應該跑什麼、建什麼或重現什麼？
- **比較 / Baseline：** 要跟什麼方案比較？
- **衡量指標：** Quality、Cost、Latency、Token Use、Reliability、Developer Effort、Attack Success Rate、Defense Success Rate、False Positive Rate、Policy Violation 或其他相關指標。

## Watchlist

選填。放還不足以升格為 Research Candidate，但如果後續有新 evidence 就值得再檢查的項目。

## 本日主要來源

列出本次報告實際依賴的主要第一手來源。這不是完整 web-search log，不需要把所有搜尋結果都塞進來。
