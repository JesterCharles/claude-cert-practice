# CCA-F daily study plan — May 12 to May 29, 2026

> **The recommended approach is Quiz mode in the [tracker](../index.html).** Open the tracker, take the daily 30-Q quiz (≤30 min, no enforced timer), and let the post-quiz "Review missed → resources" section tell you what to study. The skill graph builds itself as you go and highlights weak domains so you can target effort. This markdown file is the **backup plan** — if you'd rather follow a fixed daily schedule than self-direct via quizzes, follow what's below.

**Target exam date:** Thursday, May 29, 2026
**Daily pattern:** ~60-min quiz (30 Q at the real exam's 2-min/Q pace — timer is optional, not enforced) → targeted review of misses → optional stretch material
**Pace:** ~1 hr/day, ~30 hrs total over 18 days. One breathing day per week — no quiz pressure.

---

## 🎯 Practice mocks — test early, test often

The 30-Q daily quiz in the tracker is the primary practice. Outside that, these are the community mocks:

- **Quickest reps:** [Vercel mock exam][vercel] — free, no login (~40 Q)
- **Short set:** [claudecertifications.com][certs] — 25 questions
- **Smallest set, highest authority:** the 12 sample questions inside the [Exam Guide PDF][exam-pdf]

Use these for the May 26 timed diagnostic and any time you want variety beyond the tracker's bank.

> **Note:** Anthropic's Skilljar portal hosts cert registration and the free Academy courses — **but does not provide a separate practice exam**. The diagnostic uses the community mocks above.

---

## Week 1 — May 12–18 — Orientation + first pass through all 5 domains

**Goal:** touch every domain once with the quiz-then-review pattern. D1 gets two days (it's 27% of the exam).

| Date | Day | Domain | Core (30-60 min) | Stretch (+30 min) |
|------|-----|--------|------------------|-------------------|
| May 12 | Mon | Orientation | Read full [Exam Guide PDF][exam-pdf]; note gut weakest domain. (No quiz yet — calibrate.) | [Yondem §1 API Fundamentals][yondem-sec1] |
| May 13 | Tue | D1 Agentic | 30-Q quiz, then review misses. Read: [Agent loop][agent-loop] + [Stop reasons][stop-reasons] | [Yondem §8 Agentic Patterns][yondem-sec8] |
| May 14 | Wed | D1 Agentic | 30-Q quiz, then review misses. Read: [Subagents][subagents] + [engineering blog][eng-blog] | [Yondem §9 Customer Service / Production Workflow][yondem-sec9] |
| May 15 | Thu | D3 Claude Code | 30-Q quiz, then review misses. Read: [Claude Code memory (CLAUDE.md)][cc-memory] + [Yondem §10 Claude Code workflows][yondem-sec10] | Academy: [Claude Code 101][cc-101] |
| May 16 | Fri | D2 MCP | 30-Q quiz, then review misses. Read: [MCP in the SDK][mcp-sdk] + [MCP spec server section][mcp-spec] | [Yondem §2 Designing Tool Interfaces][yondem-sec2] |
| May 17 | Sat | D4 Prompt | 30-Q quiz, then review misses. Read: [Structured outputs][structured] + [Yondem §4 Structured Data Extraction][yondem-sec4] | [Yondem §6 System Prompt Engineering][yondem-sec6] |
| May 18 | Sun | 🌱 Breathing | **No required quiz today.** Skim your skill graph; spend 20 min on whatever's weakest. | — |

> **Checkpoint:** Each domain should be ≥60% in your skill graph by Sunday. If not, the breathing day is for fixing that — not loafing.

---

## Week 2 — May 19–25 — Domain depth + scenario integration

**Goal:** second-pass each domain with the harder material (hooks, advanced MCP, validation/retry, escalation).

| Date | Day | Domain | Core (30-60 min) | Stretch (+30 min) |
|------|-----|--------|------------------|-------------------|
| May 19 | Mon | D5 Context | 30-Q quiz, then review misses. Read: [Prompt caching][prompt-cache] + [Yondem §5 Conversation Context][yondem-sec5] | [Yondem §3 Error Handling][yondem-sec3] |
| May 20 | Tue | D1 Agentic | 30-Q quiz, then review misses. Read: [Hooks][hooks] — focus on programmatic enforcement vs prompt-based guidance | Academy: [Intro to Subagents][course-subagents] |
| May 21 | Wed | D3 Claude Code | 30-Q quiz, then review misses. Drill: slash commands, plan mode, `.claude/rules` with YAML frontmatter | Academy: [Intro to Agent Skills][course-skills] |
| May 22 | Thu | D4 Prompt | 30-Q quiz, then review misses. Drill: `tool_use` + JSON schemas, validation/retry loops, few-shot examples. Read: [Yondem §11 Iterative Refinement][yondem-sec11] | [Yondem §12 Batch / Cost / Latency][yondem-sec12] |
| May 23 | Fri | D2 MCP | 30-Q quiz, then review misses. Drill: MCP error responses (`isError`, `errorCategory`), tool description quality, tool distribution. Read: [Yondem §7 MCP][yondem-sec7] | Academy: [MCP Advanced Topics][course-mcp-adv] |
| May 24 | Sat | D5 Context | 30-Q quiz, then review misses. Drill: escalation criteria, error propagation, scratchpad files, provenance. Walk through [Larionov scenarios][larionov] | [Yondem §13 Quick Reference Cheat Sheet][yondem-cheat] |
| May 25 | Sun | 🌱 Breathing | **No required quiz today.** Re-skim your skill graph; spend 20 min on whatever's weakest. | — |

> **Checkpoint:** Can you whiteboard the architecture for each of the six exam scenarios from memory? Are your domain bars all at 70%+? If not, this is the week to fix it.

---

## Week 3 — May 26–29 — Diagnostic + targeted cleanup + exam

**Goal:** simulate exam conditions, fix what surfaces, walk in calm.

| Date | Day | Domain | Core (30-60 min, except May 26 which is 120 min) | Stretch |
|------|-----|--------|------------------------|----------|
| May 26 | Mon | Diagnostic | **Timed mock — 120 min, no notes.** [claudecertifications.com][certs] (25 fresh Q) + [Vercel mock][vercel] re-attempt under time pressure (some are repeats from week 2 — focus on missed ones) | Score honestly. Compare against your tracker skill graph. |
| May 27 | Tue | Targeted | Drill your weakest domain from May 26: take a focused 10-Q quiz on it (tracker → "Practice this domain"), then re-read the relevant primary doc | Re-read [engineering blog][eng-blog] if D1 was weakest, or [MCP in SDK][mcp-sdk] if D2 |
| May 28 | Wed | Final review | Skim [Exam Guide PDF][exam-pdf] once more + run through [Yondem §13 Cheat Sheet][yondem-cheat]. Light 15-Q quiz to confirm readiness. | Sleep early. Hydrate. Don't cram new material. |
| May 29 | Thu | Exam day | Light cheat-sheet glance in the morning only. **Don't cram.** | — |

---

## Adjustments if your week falls apart

- **Behind by 1 day:** absorb it into the next breathing day (Sun May 18 or Sun May 25).
- **Behind by 3+ days:** drop the stretch column entirely and protect the core. Skip Tier 4 resources.
- **Behind by a full week:** non-negotiables in priority order are the [Exam Guide PDF][exam-pdf] (May 12), the daily 30-Q quizzes in the tracker (Tue–Sat of weeks 1 and 2), and the May 26 diagnostic. Everything else compresses around those.

---

## Red-flag rule

If you score below **65%** on the May 26 diagnostic, given the 6-month cooldown on a fail, talk to whoever's coordinating before burning the attempt on May 29. A delayed sitting beats a failed one.

---

## Day-by-domain count (sanity)

- **D1 Agentic** (27%): 3 days (May 13, 14, 20)
- **D2 MCP** (18%): 2 days (May 16, 23)
- **D3 Claude Code** (20%): 2 days (May 15, 21)
- **D4 Prompt** (20%): 2 days (May 17, 22)
- **D5 Context** (15%): 2 days (May 19, 24)
- **Orientation / Breathing / Mock / Final / Exam:** 7 days

Daily quiz is always mixed (6 Q from each domain), so even on D1 day you're touching every domain.

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
[cc-101]: https://anthropic.skilljar.com/claude-code-101
