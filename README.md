# Agent Radar

A continuously updated research radar for **AI agent engineering and AI/Agent Security**.

Agent Radar tracks meaningful technical developments across agent runtimes, coding agents, multi-agent systems, context engineering, memory, tool use, evaluation, observability, orchestration, and security. The goal is not to collect every announcement—it is to identify developments worth understanding, reproducing, benchmarking, or applying.

## What this repository tracks

Primary research areas include:

- Agent runtimes and execution architectures
- Coding agents and software-engineering agents
- Multi-agent systems, delegation, fan-out, and escalation
- Context engineering and context management
- Agent memory and knowledge systems
- MCP, skills, tools, and capability interfaces
- Agent evaluation and benchmarking
- Agent observability and tracing
- Agent orchestration and workflow design
- **AI / Agent Security**
  - direct and indirect prompt injection
  - jailbreaks and instruction-boundary failures
  - tool abuse and confused-deputy problems
  - credential, secret, and data-exfiltration risks
  - RAG, context, and memory poisoning
  - MCP / tool / skill / plugin supply-chain security
  - agent-to-agent trust and delegated-authority boundaries
  - unsafe code execution, sandboxing, and isolation
  - provenance, output validation, least privilege, and capability-based security
  - security evaluation, red teaming, and defense benchmarking
- Important open-source agent frameworks and infrastructure

The security track is defensive and evaluation-focused. Attack research should clarify threat models, prerequisites, impact, and reproducible test methodology without turning the repository into an operational intrusion manual.

See [`research/SECURITY.md`](research/SECURITY.md) for the security taxonomy and evaluation principles.

## Research philosophy

Agent Radar prioritizes **technical signal over hype**.

A project is not considered important simply because it has many GitHub stars or appears frequently on social media. Research briefs prioritize primary sources, substantive implementation changes, reproducible evidence, and ideas that may affect how agents should actually be built.

For each notable development, the research should answer:

1. **What changed?**
2. **Why does it matter?**
3. **How does it work technically?**
4. **What is the security implication / attack surface?**
5. **How is it different from existing approaches?**
6. **Can it be reproduced, tested, or benchmarked?**
7. **Is it worth becoming a Research Candidate?**

Unchanged items should not be repeated merely to fill a daily report.

## Source priority

Preferred sources, roughly in order:

1. Official documentation, engineering posts, release notes, security advisories, and repositories
2. Original papers and technical reports
3. GitHub commits, pull requests, issues, ADRs, benchmarks, and source code
4. Reproducible security research and red-team evaluations
5. High-quality technical analysis
6. Community discussion only when it adds useful evidence or implementation experience

Where possible, briefs link directly to the original source rather than secondary summaries.

## Repository structure

```text
agent-radar/
├── README.md
└── research/
    ├── README.md
    ├── SECURITY.md
    ├── TEMPLATE.md
    └── YYYY-MM-DD.md
```

Daily reports live in `research/` and use the Asia/Taipei calendar date.

As the project grows, experiments, benchmarks, or topic-specific synthesis may be added when there is real content to justify them rather than creating empty structure in advance.

## Daily research briefs

<!-- DAILY_BRIEFS_START -->
- [2026-09-07](research/2026-09-07.md) — 動態 repository enclave、Agent action-commit 安全邊界、MCP filesystem canonicalization，以及 AI Security scanner baseline
- [2026-09-06](research/2026-09-06.md) — Repository delegation control plane、Skills over MCP、GPT-6 Astra、GitHub Agentic Workflows，以及九專案 AI Workflow 綜合研究
- [2026-09-06 AI Workflow Deep Dive](research/2026-09-06-ai-workflow-deep-dive.md) — Dashi、Codebase-Memory、gh-aw、Graphify、OpenWiki、LLM Wiki、Symphony、Archify、OpenViking 的 Agent Stack 深度比較
<!-- DAILY_BRIEFS_END -->

## Research Candidates

A **Research Candidate** is more than an interesting link. It is something that appears valuable enough to justify a concrete next step, such as:

- reproducing an experiment
- running a benchmark
- comparing two architectures
- testing cost / latency / quality trade-offs
- measuring attack and defense success rates
- inspecting an implementation in depth
- validating whether a claimed improvement survives realistic workloads

Candidates should include a proposed experiment or verification step rather than only saying that a project looks interesting.

## Automation

This repository is maintained with a scheduled research workflow. The workflow searches for new developments, compares them with recent briefs, writes a dated research report, and updates the reverse-chronological index above.

Automation is used for discovery and synthesis, not as a substitute for evidence. Claims should remain traceable to source material, and uncertain conclusions should be identified as such.

## Status

This repository is an evolving public research notebook. The taxonomy, evaluation criteria, and experiment structure may change as better patterns emerge.
