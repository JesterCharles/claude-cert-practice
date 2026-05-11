# CCA-F prep — resource list

8 resources, sorted by trust tier. **Tier 1** — authoritative, required reading. **Tier 2** — Anthropic primary sources, the bulk of your study time. **Tier 3** — community resources, verify against Tier 1/2 before trusting. **Tier 4** — optional breadth, skip first if time gets tight.

> **Heads-up on the "official practice exam":** Anthropic's Skilljar portal hosts cert registration and the free Academy courses but **does not provide a separate practice exam**. The May 26 timed diagnostic in the daily plan uses the community mocks below (Vercel + claudecertifications.com).

---

## Tier 1 — Authoritative (start here)

**1. Official CCA-F Exam Guide PDF** — official domain weights, six scenarios, 12 sample questions. This is the canonical authoritative resource.

- Canonical access path (Skilljar portal): https://anthropic.skilljar.com/claude-certified-architect-foundations-access-request
- Direct PDF (Skilljar-hosted, may require access): https://everpath-course-content.s3-accelerate.amazonaws.com/instructor%2F8lsy243ftffjjy1cx9lm3o2bw%2Fpublic%2F1773274827%2FClaude+Certified+Architect+%E2%80%93+Foundations+Certification+Exam+Guide.pdf

---

## Tier 2 — Anthropic primary sources

**2. Anthropic Academy courses** (Skilljar, free) — direct links to the four most exam-relevant:

- [Introduction to Model Context Protocol](https://anthropic.skilljar.com/introduction-to-model-context-protocol)
- [Model Context Protocol: Advanced Topics](https://anthropic.skilljar.com/model-context-protocol-advanced-topics)
- [Introduction to subagents](https://anthropic.skilljar.com/introduction-to-subagents)
- [Introduction to agent skills](https://anthropic.skilljar.com/introduction-to-agent-skills)
- Full course catalog: https://anthropic.skilljar.com/

**3. Anthropic docs — the eight pages that map to exam domains:**

- Agent loop: https://platform.claude.com/docs/en/agent-sdk/agent-loop
- Subagents: https://platform.claude.com/docs/en/agent-sdk/subagents
- Hooks: https://platform.claude.com/docs/en/agent-sdk/hooks
- MCP in the SDK: https://docs.claude.com/en/docs/agent-sdk/mcp
- Structured outputs: https://platform.claude.com/docs/en/build-with-claude/structured-outputs
- Prompt caching: https://platform.claude.com/docs/en/build-with-claude/prompt-caching
- Stop reasons: https://platform.claude.com/docs/en/build-with-claude/handling-stop-reasons
- Claude Code memory: https://code.claude.com/docs/en/memory

**4. MCP Specification** (server section + transports): https://modelcontextprotocol.io/specification/2025-06-18/

**5. Anthropic engineering blog** — "Building agents with the Claude Agent SDK": https://www.anthropic.com/engineering/building-agents-with-the-claude-agent-sdk

---

## Tier 3 — Best community resources (verify against primary)

**6. Community GitHub guides** — triangulate; trust where all three agree:

- [Yondem guide](https://github.com/daronyondem/claude-architect-exam-guide) — 13 sections of deep prose. The [exam-preparation-guide.md](https://github.com/daronyondem/claude-architect-exam-guide/blob/main/exam-preparation-guide.md) is the meat; sections are numbered §1–§13 and the daily plan deep-links to the matching section on each topic day.
- [Larionov](https://github.com/paullarionov/claude-certified-architect) — scenario walkthroughs (English + Russian), best for the 6 documented scenarios.
- [avidevelops](https://github.com/avidevelops/claude-architect-exam-prep) — domain notes and patterns.

**7. Vercel mock exam** — best free practice volume on the web (~40 questions): https://claude-certified-architect-mock-exam-cyberskill.vercel.app/

---

## Tier 4 — Optional breadth

**8. claudecertifications.com** — 25 practice questions, 12-week plan: https://claudecertifications.com/

---

## Non-negotiables if time gets tight

In priority order:

1. Exam Guide PDF (#1)
2. The 8 Anthropic doc pages (#3)
3. A timed Vercel + claudecertifications.com mock by May 26

Everything else is helpful but skippable.
