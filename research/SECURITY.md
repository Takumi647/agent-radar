# AI / Agent Security Research Scope

This document defines the security research track for Agent Radar.

The objective is to understand how AI systems fail under adversarial or untrusted input, how agentic systems expand the attack surface, and which defenses survive realistic evaluation. The emphasis is on defensive architecture, measurable security properties, and reproducible testing in controlled environments.

## 1. Prompt and context attacks

Research areas:

- direct prompt injection
- indirect prompt injection through web pages, documents, email, code comments, retrieved content, tool output, and other untrusted context
- jailbreaks and instruction-hierarchy failures
- system-prompt or hidden-context disclosure
- context confusion and instruction/data boundary failures
- multimodal injection when images, PDFs, audio, or UI content can influence the model

Key questions:

- Which input is attacker-controlled?
- Does the model distinguish instructions from data?
- Can untrusted content alter tool behavior or authorization decisions?
- Does a proposed detector actually reduce attack success, or merely catch known strings?

## 2. Tool-use and agent authority attacks

Research areas:

- confused-deputy problems
- excessive tool permissions
- unsafe tool composition
- credential or secret exposure
- data exfiltration through legitimate tools
- unauthorized side effects
- cross-repository / cross-tenant access
- delegated-agent privilege propagation
- stale or unreclaimed delegated credentials
- agent-to-agent message trust

Preferred defenses to study:

- least privilege
- scoped and short-lived credentials
- capability-based authorization
- read-only-by-default agents
- structured proposed mutations + deterministic validators
- explicit approval boundaries for high-impact actions
- auditability and revocation

## 3. MCP, skills, plugins, and supply-chain security

Research areas:

- malicious or compromised MCP servers
- tool-description poisoning
- unsafe remote resources
- malicious skills or plugins
- dependency and package compromise
- provenance and version drift
- remote skill updates that silently change behavior
- trust relationships across multiple MCP servers

Key questions:

- What code or context is dynamically loaded?
- How is origin/provenance represented?
- Can a previously approved tool or skill change after approval?
- Are digests, version pins, signatures, or policy checks meaningful in practice?

## 4. RAG, memory, and knowledge poisoning

Research areas:

- RAG poisoning
- retrieval hijacking
- malicious document ingestion
- context poisoning
- persistent memory poisoning
- cross-session contamination
- incorrect derived knowledge becoming durable truth
- stale knowledge and provenance loss

Preferred defenses to study:

- provenance attached to retrieved facts
- trust tiers for sources
- write gating for persistent memory
- separation of canonical truth from derived knowledge
- expiry / re-verification of stale claims
- suspicious-source isolation
- memory rollback and audit trails

## 5. Code execution and sandbox security

Research areas:

- unsafe generated code execution
- malicious repository content influencing coding agents
- command/tool injection
- sandbox and container boundaries
- filesystem and network isolation
- environment-secret exposure
- dependency-install risk
- privilege escalation or sandbox escape research when relevant to agent runtimes

The goal is to evaluate isolation and containment, not to provide operational intrusion playbooks.

## 6. Model-level security

Research areas:

- jailbreak robustness
- policy bypass
- sensitive-data leakage
- model extraction or inversion research when relevant
- training-data or memorization leakage
- hallucinated authorization or fabricated provenance

For model-security claims, separate vendor-reported results from independently reproduced evidence.

## 7. Defensive architecture patterns

Patterns worth tracking and benchmarking:

- instruction/data separation
- taint or provenance tracking for untrusted context
- least privilege
- capability-based security
- sandboxing and process isolation
- allowlisted egress
- scoped tool exposure
- deterministic policy enforcement outside the LLM
- output validation
- human approval for irreversible or high-impact actions
- immutable / append-only audit trails
- memory write gates
- source-grounded claims
- defense in depth rather than single prompt-based guardrails

Prompt-only defenses should generally be treated as one layer, not a security boundary.

## 8. Security evaluation methodology

A security claim is most useful when it defines:

1. **Threat model** — attacker capability and controlled inputs
2. **Target boundary** — what should remain protected
3. **Attack set** — representative adversarial cases
4. **Baseline** — system without the proposed defense
5. **Defense variant** — exact mitigation being evaluated
6. **Utility test** — whether normal tasks still work
7. **Reproducibility** — enough detail to verify safely in a controlled environment

Recommended metrics:

- attack success rate (ASR)
- defense success / block rate
- false-positive rate
- false-negative rate
- policy-violation rate
- secret / data leakage rate
- unauthorized tool-call rate
- task success / quality under defense
- latency overhead
- token / model cost overhead
- recovery time
- audit completeness
- developer / operational complexity

A defense that drives ASR down but destroys normal task utility should not be called successful without qualification.

## 9. Source priority

Prefer:

1. primary security advisories and incident reports
2. original research papers
3. official engineering / security publications
4. source code, commits, issues, and reproducible proof-of-concept tests in controlled environments
5. independent replication
6. high-quality secondary analysis

Useful ecosystems to monitor include major model providers, GitHub, MCP implementations, OWASP AI-security work, MITRE ATLAS, NIST AI-security guidance, academic security conferences, and open-source red-team / evaluation tooling.

## 10. How Security integrates with Agent Radar

Security should not become a completely separate news feed. Every important Agent Radar item should ask:

```text
New capability
    ↓
What new authority or untrusted input does it introduce?
    ↓
What security boundary is supposed to contain it?
    ↓
Can that boundary be tested?
```

Examples:

- Agent delegation → delegated authority, revocation, confused deputy
- MCP skills → remote-context provenance and supply-chain risk
- Agent memory → persistent poisoning and rollback
- Codebase RAG → malicious repository/document ingestion
- Autonomous coding → sandbox, credentials, and write authority
- Multi-agent systems → trust between agents and message provenance

The aim is to make security part of the architecture review rather than something added after deployment.
