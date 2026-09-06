# Research Briefs

This directory contains dated Agent Radar research briefs.

## File naming

Use the Asia/Taipei calendar date:

```text
YYYY-MM-DD.md
```

Example:

```text
2026-09-06.md
```

## What belongs in a daily brief

A daily brief should contain only developments with enough technical substance to justify inclusion. Prefer primary sources and explicitly distinguish confirmed facts from interpretation.

Each notable item should explain:

- what changed
- why it matters
- the technical mechanism
- security implications / attack surface when relevant
- how it differs from existing approaches
- what evidence supports the assessment
- whether it suggests an experiment, replication, benchmark, or security evaluation

Do not repeat an unchanged item from a previous brief unless there is a meaningful new development. If the day is quiet, a short brief explaining what was checked and why nothing qualified is better than filler.

## AI / Agent Security

Security is a first-class research track, not an appendix. Use [`SECURITY.md`](./SECURITY.md) as the research taxonomy.

Relevant areas include prompt injection, jailbreaks, tool abuse, credential and data exfiltration, RAG/context/memory poisoning, MCP/tool/skill supply-chain risk, agent-to-agent trust, sandboxing, delegated authority, provenance, output validation, least privilege, capability-based security, and security evaluation.

When covering attacks, focus on threat models, prerequisites, impact, controlled reproduction, and defenses. Do not turn briefs into operational instructions for unauthorized intrusion.

## Research Candidates

Add a Research Candidate only when there is a concrete next action. Good candidates include a proposed benchmark, reproduction, architecture comparison, implementation inspection, measurable experiment, or security evaluation.

Security candidates should define relevant metrics such as attack success rate, defense success rate, false-positive rate, policy violations, task-quality degradation, latency, token cost, and operational complexity.

## Source handling

Link to original sources whenever possible:

- official documentation, engineering posts, or security advisories
- original papers
- source repositories
- specific releases, commits, pull requests, issues, benchmark results, or reproducible security evaluations

Secondary discussion may provide context, but it should not replace primary evidence for technical claims.

## Indexing

After adding a dated brief, update the `Daily research briefs` section in the root `README.md` in reverse chronological order.
