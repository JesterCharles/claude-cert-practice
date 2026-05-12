# CCA-F daily study plan — May 12 to May 29, 2026

> **There's also a quiz-driven alternative.** Open the [tracker](../index.html) and switch to **Quiz mode** for a self-directed approach: 30 fresh questions per day pulled from a 210-question bank, with a skill graph that highlights weak domains so you can target practice instead of following a fixed plan. This document is the **prescribed-plan fallback** for anyone who'd rather follow a fixed schedule.

**Target exam date:** Thursday, May 29, 2026
**Pace:** 1 hr/day core, +30 min stretch when available (~30 hrs total over 18 days)

**How to use:** do the core column every day, no exceptions. Add the stretch when you have time. Saturday and Sunday are the natural catch-up days if a weekday slips. Every resource is linked inline below — click through to read the canonical source as you go.

---

## 🎯 Practice Exam — test early, test often

The dedicated practice slots in this plan are May 24, 25, and 26 — but you don't have to wait. Pick up a 10–20 question set any time you finish your core early. Each rep tightens the diagnostic and reduces exam-day variance.

- **Quickest reps:** [Vercel mock exam][vercel] — free, no login (~40 Q)
- **Short set:** [claudecertifications.com][certs] — 25 questions
- **Smallest set, highest authority:** the 12 sample questions inside the [Exam Guide PDF][exam-pdf]

A reasonable rhythm: one or two short rep sessions in week 1, light Vercel reps mixed in through week 2, then the formal timed diagnostic on May 26 combining both Vercel and claudecertifications.com.

> **Note:** Anthropic's Skilljar portal hosts cert registration and the free Academy courses — **but does not provide a separate practice exam**. The diagnostic uses the community mocks above.

---

## Week 1 — May 12–18 — Foundations + heavy domains (~10 hr)

**Goal:** get the underlying tech solid before touching practice questions.

| Date | Day | Core (1 hr) | Stretch (+30 min) | Domain |
|------|-----|-------------|-------------------|--------|
| May 12 | Mon | Read full [Exam Guide PDF][exam-pdf]; note gut weakest domain | [Yondem §1 API Fundamentals and Output Control][yondem-sec1] | Overview |
| May 13 | Tue | Anthropic docs: [Agent loop][agent-loop] + [Stop reasons][stop-reasons] | [Yondem §8 Agentic Patterns and Task Decomposition][yondem-sec8] | D1 Agentic |
| May 14 | Wed | Anthropic docs: [Subagents][subagents] + [engineering blog post][eng-blog] | [Yondem §9 Customer Service and Production Workflow Design][yondem-sec9] | D1 Agentic |
| May 15 | Thu | Anthropic docs: [Claude Code memory (CLAUDE.md hierarchy)][cc-memory] | [Yondem §10 Claude Code and Claude Agent SDK Workflows][yondem-sec10] | D3 Claude Code |
| May 16 | Fri | Anthropic docs: [Hooks][hooks] | [Yondem §6 System Prompt Engineering and Conversational Behavior][yondem-sec6] | D1 Agentic |
| May 17 | Sat | Academy course: [Intro to Subagents][course-subagents] | Academy course: [Intro to Agent Skills][course-skills] | D1 + D3 |
| May 18 | Sun | Re-read weakest area from week | [Yondem §11 Iterative Refinement][yondem-sec11] + [§12 Batch / Cost / Latency][yondem-sec12] | — |

> **Checkpoint:** Can you explain the 5-step loop, name every `stop_reason` value, and describe what a subagent inherits — without notes? If no, slow down.

---

## Week 2 — May 19–25 — Remaining domains + scenarios (~10 hr)

**Goal:** cover MCP, prompt engineering, context management; start scenario thinking.

| Date | Day | Core (1 hr) | Stretch (+30 min) | Domain |
|------|-----|-------------|-------------------|--------|
| May 19 | Mon | Anthropic docs: [MCP in the SDK][mcp-sdk] + [MCP spec server section][mcp-spec] | Academy course: [Intro to MCP][course-mcp] | D2 MCP |
| May 20 | Tue | Anthropic docs: [Structured outputs][structured] | Academy course: [MCP Advanced Topics][course-mcp-adv] | D4 Prompt |
| May 21 | Wed | Anthropic docs: [Prompt caching][prompt-cache] | [Yondem §5 Conversation Context Management][yondem-sec5] | D5 Context |
| May 22 | Thu | [Larionov GitHub][larionov]: scenarios 1–3 walkthroughs | [Yondem §2 Designing Tool Interfaces for LLM Agents][yondem-sec2] | Scenarios |
| May 23 | Fri | [Larionov GitHub][larionov]: scenarios 4–6 walkthroughs | [Yondem §3 Error Handling in Agent Tools][yondem-sec3] | Scenarios |
| May 24 | Sat | [Vercel mock][vercel]: 20 questions, untimed, read explanations | [Yondem §4 Structured Data Extraction and Validation][yondem-sec4] | All domains |
| May 25 | Sun | [Vercel mock][vercel]: 20 more questions | [Yondem §7 Model Context Protocol (MCP)][yondem-sec7] | All domains |

> **Checkpoint:** Can you whiteboard the architecture for each of the six scenarios from memory? If a scenario feels fuzzy, re-read its primary-doc references before Week 3.

---

## Week 3 — May 26–29 — Diagnostic + targeted cleanup (~8 hr)

**Goal:** take a full timed mock, fix what it surfaces, walk in calm.

| Date | Day | Core (1 hr) | Stretch (+30 min) | Domain |
|------|-----|-------------|-------------------|--------|
| May 26 | Mon | Timed mock — 120 min, no notes, sit it like the real exam: [claudecertifications.com][certs] (25 fresh Q) + [Vercel mock][vercel] re-attempt under time pressure (some Q's are repeats from week 2 — focus on missed ones) | Score honestly, identify weakest domain | Diagnostic |
| May 27 | Tue | Review every wrong answer; map each to a domain + primary doc; re-read those pages | Re-read [engineering blog][eng-blog] if D1 was weakest | Targeted |
| May 28 | Wed | Final pass on weakest domain's primary docs; skim [Exam Guide PDF][exam-pdf] again | [Yondem §13 Quick Reference Cheat Sheet][yondem-cheat] review | Targeted |
| May 29 | Thu — Exam day | Light cheat-sheet review in the morning only. Don't cram. | — | — |

---

## Adjustments if your week falls apart

- **Behind by 1 day:** absorb it into the following Saturday/Sunday catch-up slot.
- **Behind by 3+ days:** drop the stretch column entirely and protect the core. Skip Tier 4 resources.
- **Behind by a full week:** the non-negotiables in priority order are the [Exam Guide PDF][exam-pdf] (May 12), the 8 Anthropic doc pages (Tue–Wed of weeks 1 and 2), and the diagnostic on May 26. Everything else compresses around those.

---

## Red-flag rule

If you score below **65%** on the May 26 diagnostic, given the 6-month cooldown on a fail, talk to whoever's coordinating before burning the attempt on May 29. A delayed sitting beats a failed one.

---

<!-- Link references -->
[exam-pdf]: https://everpath-course-content.s3-accelerate.amazonaws.com/instructor%2F8lsy243ftffjjy1cx9lm3o2bw%2Fpublic%2F1773274827%2FClaude+Certified+Architect+%E2%80%93+Foundations+Certification+Exam+Guide.pdf
[agent-loop]: https://platform.claude.com/docs/en/agent-sdk/agent-loop
[subagents]: https://platform.claude.com/docs/en/agent-sdk/subagents
[hooks]: https://platform.claude.com/docs/en/agent-sdk/hooks
[mcp-sdk]: https://docs.claude.com/en/docs/agent-sdk/mcp
[structured]: https://platform.claude.com/docs/en/build-with-claude/structured-outputs
[prompt-cache]: https://platform.claude.com/docs/en/build-with-claude/prompt-caching
[stop-reasons]: https://platform.claude.com/docs/en/build-with-claude/handling-stop-reasons
[cc-memory]: https://code.claude.com/docs/en/memory
[mcp-spec]: https://modelcontextprotocol.io/specification/2025-06-18/
[eng-blog]: https://www.anthropic.com/engineering/building-agents-with-the-claude-agent-sdk
[yondem-sec1]: https://github.com/daronyondem/claude-architect-exam-guide/blob/main/exam-preparation-guide.md#1-api-fundamentals-and-output-control
[yondem-sec2]: https://github.com/daronyondem/claude-architect-exam-guide/blob/main/exam-preparation-guide.md#2-designing-tool-interfaces-for-llm-agents
[yondem-sec3]: https://github.com/daronyondem/claude-architect-exam-guide/blob/main/exam-preparation-guide.md#3-error-handling-in-agent-tools
[yondem-sec4]: https://github.com/daronyondem/claude-architect-exam-guide/blob/main/exam-preparation-guide.md#4-structured-data-extraction-and-validation
[yondem-sec5]: https://github.com/daronyondem/claude-architect-exam-guide/blob/main/exam-preparation-guide.md#5-conversation-context-management
[yondem-sec6]: https://github.com/daronyondem/claude-architect-exam-guide/blob/main/exam-preparation-guide.md#6-system-prompt-engineering-and-conversational-behavior
[yondem-sec7]: https://github.com/daronyondem/claude-architect-exam-guide/blob/main/exam-preparation-guide.md#7-model-context-protocol-mcp
[yondem-sec8]: https://github.com/daronyondem/claude-architect-exam-guide/blob/main/exam-preparation-guide.md#8-agentic-patterns-and-task-decomposition
[yondem-sec9]: https://github.com/daronyondem/claude-architect-exam-guide/blob/main/exam-preparation-guide.md#9-customer-service-and-production-workflow-design
[yondem-sec10]: https://github.com/daronyondem/claude-architect-exam-guide/blob/main/exam-preparation-guide.md#10-claude-code-and-claude-agent-sdk-workflows
[yondem-sec11]: https://github.com/daronyondem/claude-architect-exam-guide/blob/main/exam-preparation-guide.md#11-iterative-refinement-testing-and-evaluation
[yondem-sec12]: https://github.com/daronyondem/claude-architect-exam-guide/blob/main/exam-preparation-guide.md#12-batch-processing-cost-and-latency
[yondem-cheat]: https://github.com/daronyondem/claude-architect-exam-guide/blob/main/exam-preparation-guide.md#13-quick-reference-cheat-sheet
[larionov]: https://github.com/paullarionov/claude-certified-architect
[vercel]: https://claude-certified-architect-mock-exam-cyberskill.vercel.app/
[certs]: https://claudecertifications.com/
[course-subagents]: https://anthropic.skilljar.com/introduction-to-subagents
[course-skills]: https://anthropic.skilljar.com/introduction-to-agent-skills
[course-mcp]: https://anthropic.skilljar.com/introduction-to-model-context-protocol
[course-mcp-adv]: https://anthropic.skilljar.com/model-context-protocol-advanced-topics
