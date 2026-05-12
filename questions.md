# CCA-F Practice Question Bank

**Total: 210 questions** (42 per domain × 5 domains).

Generated from the official CCA-F Exam Guide. Each question is scenario-anchored, mapped to a task statement, and includes per-option explanations.

This file is the canonical source. The interactive quiz in the tracker pulls from `questions.js` which is generated from this content.

## Table of Contents

- [Domain 1: Agentic Architecture & Orchestration](#domain-1-agentic-architecture-and-orchestration)
- [Domain 2: Tool Design & MCP Integration](#domain-2-tool-design-and-mcp-integration)
- [Domain 3: Claude Code Configuration & Workflows](#domain-3-claude-code-configuration-and-workflows)
- [Domain 4: Prompt Engineering & Structured Output](#domain-4-prompt-engineering-and-structured-output)
- [Domain 5: Context Management & Reliability](#domain-5-context-management-and-reliability)

---

## Domain 1: Agentic Architecture & Orchestration


42 scenario-based questions covering Task Statements 1.1–1.7.

---

### Q1 | Scenario 1 | TS 1.1 | difficulty: easy

**Stem:** A new engineer on your support agent team is implementing the agentic loop for the returns workflow. They write code that sends a request to Claude, calls any tools that appear in the response, then immediately sends a fresh follow-up message asking "are you done yet?" before deciding whether to loop again. In dev testing the agent often answers "yes" prematurely and ends mid-refund. Which change should you recommend?

A) Replace the natural-language "are you done?" check with a control-flow check on `stop_reason`: continue the loop while `stop_reason == "tool_use"` and terminate when it is `"end_turn"`.
B) Lower the temperature so the "are you done?" check is more deterministic and gives reliable yes/no answers.
C) Cap the loop at 5 iterations so the agent cannot terminate prematurely before all refund steps run.
D) Have the agent emit a sentinel string like `<<COMPLETE>>` in its final assistant message and grep for that string to decide when to exit the loop.

**Correct:** A

**Explanation:**
- **A (correct):** The agentic loop terminates correctly when you inspect `stop_reason`: `"tool_use"` means Claude wants another tool executed (loop again), `"end_turn"` means the model is finished. This is a deterministic control signal and the canonical pattern.
- **B:** Temperature changes do not turn a natural-language completion probe into a reliable termination signal — it remains probabilistic.
- **C:** Iteration caps are a safety net, not a termination mechanism; using them as the primary stopping rule risks both premature termination and runaway loops.
- **D:** Parsing assistant text for sentinel strings is exactly the anti-pattern the agentic loop is designed to avoid — `stop_reason` already provides this signal.

---

### Q2 | Scenario 3 | TS 1.1 | difficulty: medium

**Stem:** Your research coordinator agent calls `web_search`, executes it, and produces a draft report. Logs show that after receiving the `web_search` tool result the agent often produces a final answer without ever calling the `document_analysis` or `synthesis` subagent tools, even when the search results clearly require follow-up. On inspection, your loop code is appending tool results into a separate scratchpad string that is passed as a new user turn rather than as a `tool_result` content block in the prior assistant turn's conversation history. What is the most direct fix?

A) Append tool results as proper `tool_result` blocks back into the conversation history so the model can reason about them on the next iteration.
B) Increase `max_tokens` so Claude has room to plan more tool calls after seeing the search results.
C) Add a system prompt instruction telling Claude it must always call `document_analysis` after `web_search`.
D) Replace the coordinator with a fixed pipeline that always invokes `web_search` then `document_analysis` then `synthesis` in order.

**Correct:** A

**Explanation:**
- **A (correct):** Tool results must be appended to conversation history (as `tool_result` blocks tied to the prior `tool_use` IDs) so the next iteration's reasoning is grounded in them. A side-channel scratchpad breaks the loop's continuity and the model effectively ignores those results.
- **B:** Token budget is not the issue; the model isn't seeing the results in the form it expects.
- **C:** A prompt rule can't compensate for a broken loop transport — the model isn't reasoning over the results at all.
- **D:** Hard-coding the sequence sidesteps model-driven decision-making and discards adaptability for cases where analysis is unnecessary.

---

### Q3 | Scenario 4 | TS 1.1 | difficulty: medium

**Stem:** You are building a developer-productivity agent using the Claude Agent SDK with built-in tools (Read, Grep, Glob, Bash). The engineer using it reports that the agent sometimes "freezes": Claude returns a response with `tool_use` blocks, but no further tool execution occurs and the agent never produces a final answer. You verify the SDK is wired up but suspect your control flow. Which loop logic is most consistent with the symptom?

A) The loop terminates whenever the assistant turn contains any text content, rather than continuing while `stop_reason == "tool_use"`.
B) The loop uses an exponential backoff between iterations that is currently set too high.
C) The loop is not seeding the conversation with a system prompt, so the model declines to act.
D) The loop is sending tool results as a separate API call rather than as a `tool_result` block.

**Correct:** A

**Explanation:**
- **A (correct):** A common anti-pattern is treating the presence of assistant text as a completion signal. When Claude emits both reasoning text and `tool_use` blocks, exiting on the text check skips executing the tools and leaves the agent "frozen" with no further iterations.
- **B:** Backoff delays slow the loop but do not cause it to halt without completing.
- **C:** Missing a system prompt would change quality, not produce a hung tool-use state.
- **D:** That would cause the model to ignore results, not cause the loop to stop firing tools.

---

### Q4 | Scenario 1 | TS 1.1 | difficulty: hard

**Stem:** During a billing dispute conversation, your support agent calls `lookup_order`, the tool errors out (the order ID was malformed), and your code currently sees the error and terminates the loop, returning a generic apology to the customer. Product wants the agent to recover automatically when an MCP tool returns an error, e.g., by asking for clarification or trying a different lookup strategy. Which loop-level change best supports this?

A) Append the tool error as a `tool_result` with `is_error=true` to conversation history and continue the loop; let Claude decide the next action based on the error content.
B) Catch the error in code, retry the same `lookup_order` call up to 3 times, then escalate if it still fails.
C) Hard-code a fallback that on any `lookup_order` failure immediately calls `escalate_to_human`.
D) Wrap every MCP tool in a try/except that suppresses errors and returns a fake "no results found" string to the model.

**Correct:** A

**Explanation:**
- **A (correct):** Returning the actual error as a `tool_result` (flagged as an error) lets the model reason about *why* the call failed and pick a recovery — clarifying with the user, trying a different parameter, or escalating. This preserves model-driven decision-making.
- **B:** Blind retries don't help when the input itself is malformed and short-circuit useful reasoning.
- **C:** Hard-coding escalation removes the agent's ability to recover with a simple clarification and degrades resolution rate.
- **D:** Masking errors as "no results" hides information the model needs to make a good next-step decision and can produce incorrect behavior downstream.

---

### Q5 | Scenario 3 | TS 1.1 | difficulty: medium

**Stem:** Your research agent's coordinator loop currently terminates after a fixed 6 iterations regardless of what the model is doing. Reviewers complain that long, multi-source research tasks produce truncated reports because the loop exits while Claude is still calling subagent tools. Your engineer proposes raising the cap to 30. What is a better approach?

A) Remove the iteration count as the primary termination mechanism and rely on `stop_reason == "end_turn"`; keep a high cap purely as a runaway-guard safety net.
B) Raise the cap to 30 and add a system-prompt reminder that the agent should "finish all research before answering."
C) Replace the cap with a wall-clock timeout of 5 minutes per session.
D) Have the agent emit a structured "I'm done" JSON object that the loop parses to decide when to stop.

**Correct:** A

**Explanation:**
- **A (correct):** `stop_reason` is the canonical termination signal; iteration limits exist as a runaway safeguard, not as the deciding factor. Raising the cap just postpones the same problem.
- **B:** Prompt reminders are probabilistic and don't fix a control-flow anti-pattern.
- **C:** A timeout is also a safeguard, not the right primary signal — it can truncate just as arbitrarily.
- **D:** Parsing model-emitted JSON for completion is the same anti-pattern as parsing natural-language signals; `stop_reason` already provides this deterministically.

---

### Q6 | Scenario 4 | TS 1.1 | difficulty: easy

**Stem:** A developer asks "what does `stop_reason == "end_turn"` mean for my agentic loop in the SDK?" You want to give the simplest correct answer.

A) The model has finished its turn and is not requesting another tool call, so the loop should terminate.
B) The model has used up its token budget and the loop should retry with a smaller prompt.
C) The model is requesting that you call a tool, so the loop should iterate again.
D) The model encountered an error and the loop should restart from the initial user message.

**Correct:** A

**Explanation:**
- **A (correct):** `end_turn` signals that the model has nothing further to do this turn — no tool calls requested — so the loop's exit condition is met.
- **B:** Token-budget exhaustion produces a different stop reason (`max_tokens`).
- **C:** That description fits `tool_use`, not `end_turn`.
- **D:** Errors are not represented by `end_turn`; they surface as exceptions or tool_result errors.

---

### Q7 | Scenario 3 | TS 1.2 | difficulty: easy

**Stem:** Your research system has four specialized agents (web_search, document_analysis, synthesis, report_generation). Currently each subagent can directly call any other subagent when it thinks more information is needed, producing tangled call graphs that are hard to debug and inconsistent in error handling. Which architectural change addresses this?

A) Adopt a hub-and-spoke pattern in which a coordinator agent owns all inter-subagent communication, error handling, and information routing.
B) Have each subagent emit a structured log entry before any direct call so the call graph can be reconstructed later.
C) Keep peer-to-peer delegation but add a shared in-memory message bus so all calls flow through one channel.
D) Allow direct calls but require each subagent to also notify the coordinator via a webhook for observability.

**Correct:** A

**Explanation:**
- **A (correct):** Hub-and-spoke with a coordinator owning routing is the canonical multi-agent pattern — it centralizes observability, error handling, and information flow control.
- **B:** Better logging helps diagnose problems but doesn't fix the underlying coupling.
- **C:** A message bus still permits peer-to-peer logic; the issue is who decides routing.
- **D:** Webhook notifications add observability but leave routing decentralized and inconsistent.

---

### Q8 | Scenario 3 | TS 1.2 | difficulty: medium

**Stem:** Your coordinator delegates a sub-question to a `document_analysis` subagent. The subagent responds asking for clarification about which corpus to search — clearly missing context the coordinator already established earlier in the conversation. You verify the coordinator's prompt to the subagent was "Analyze the documents and report findings." What's the underlying issue?

A) Subagents operate with isolated context — they do not inherit the coordinator's conversation history — so necessary context must be passed explicitly in the subagent's prompt.
B) The coordinator's conversation history was too long and got truncated before reaching the subagent.
C) The subagent's system prompt is missing a default behavior for the "no corpus specified" case.
D) The Anthropic SDK requires you to call a `share_context()` API to copy parent state to children.

**Correct:** A

**Explanation:**
- **A (correct):** Subagents do not automatically inherit the parent's context or memory. The coordinator must explicitly include relevant findings, constraints, and references in the subagent's prompt.
- **B:** Truncation isn't the issue — the history was never shared in the first place.
- **C:** A better default still requires the right input; the structural problem is missing context, not weak defaults.
- **D:** There is no such required API; context passing is the caller's responsibility via the prompt.

---

### Q9 | Scenario 3 | TS 1.2 | difficulty: hard

**Stem:** Your coordinator decomposes the query "Summarize recent advances in retrieval-augmented generation" into three subtasks: "find a paper about dense retrievers," "find a paper about sparse retrievers," and "find a paper about hybrid retrieval." Reports come back cited but reviewers note glaring omissions: re-ranking, long-context alternatives, retrieval-free approaches. What is the failure mode and the best mitigation?

A) The coordinator decomposed the topic too narrowly. Mitigation: have the coordinator first generate a broader scoping outline, then delegate per-subtopic, and iterate with a coverage-evaluation step that re-delegates to fill gaps.
B) The subagents were given too much freedom. Mitigation: tighten each subagent's prompt with stricter inclusion/exclusion rules.
C) The synthesis subagent compressed too aggressively. Mitigation: increase its `max_tokens` and require longer outputs.
D) The web_search tool returned stale results. Mitigation: switch to a different MCP search server and add a date filter.

**Correct:** A

**Explanation:**
- **A (correct):** Overly narrow task decomposition by the coordinator leads to incomplete coverage of broad research topics. The fix is broader initial scoping plus an iterative refinement loop where the coordinator inspects the synthesis for gaps and re-delegates with targeted queries.
- **B:** Tightening prompts doubles down on a too-narrow plan rather than fixing the plan.
- **C:** Output length isn't why entire subtopics are missing.
- **D:** The search tool isn't returning bad data — it was never asked about the missing subtopics.

---

### Q10 | Scenario 1 | TS 1.2 | difficulty: medium

**Stem:** Your support team is considering moving to a multi-agent design where a coordinator triages each customer message and delegates to specialists (returns, billing, account). Every customer query currently runs through the full pipeline, even simple "where is my order?" requests, doubling latency for trivial cases. What design change best addresses this?

A) Have the coordinator analyze each query first and dynamically select only the subagents needed — invoking just the order-lookup subagent for status questions and the full pipeline only when the query warrants it.
B) Run all subagents in parallel for every query and have the coordinator pick whichever response looks best.
C) Cache previous answers per customer for 24 hours so common questions skip the pipeline entirely.
D) Pre-compute responses for the top 50 questions at startup and short-circuit the coordinator on a string match.

**Correct:** A

**Explanation:**
- **A (correct):** A core coordinator responsibility is dynamic subagent selection based on query complexity — not always routing through the full pipeline. This minimizes wasted work and latency for simple cases.
- **B:** Parallel-everything wastes resources and the coordinator still has to evaluate all outputs.
- **C:** Caching helps repeated queries but doesn't address single-query routing inefficiency.
- **D:** A static FAQ short-circuit bypasses the agent entirely and is brittle to phrasing variation.

---

### Q11 | Scenario 3 | TS 1.2 | difficulty: medium

**Stem:** Two of your research subagents — `web_search` and `document_analysis` — repeatedly retrieve overlapping content (the same arxiv papers and review articles), bloating the synthesis context and wasting tokens. Both are given essentially identical sub-queries. Which coordinator change most directly reduces duplication?

A) Partition the research scope across subagents by assigning distinct subtopics or source types (e.g., web_search handles news and blog posts; document_analysis handles uploaded PDFs and papers).
B) Add a deduplication step after synthesis that drops paragraphs with high cosine similarity.
C) Reduce each subagent's `max_tokens` so they return less content.
D) Have each subagent emit a "topics covered" list and discard duplicates client-side.

**Correct:** A

**Explanation:**
- **A (correct):** Partitioning research scope across subagents by subtopic or source type is the recommended way to minimize duplication at the source.
- **B:** Downstream deduplication is reactive and still incurs the cost of running the duplicated work.
- **C:** Smaller outputs don't prevent overlap; they just truncate it.
- **D:** Client-side deduping after the fact is similar to (B) — the overlap is already paid for.

---

### Q12 | Scenario 3 | TS 1.2 | difficulty: easy

**Stem:** A teammate proposes that the report_generation subagent should call `web_search` directly when it notices a missing citation, instead of asking the coordinator to do it. From an observability and reliability standpoint, why is this discouraged?

A) Routing all inter-subagent communication through the coordinator preserves a single point for logging, consistent error handling, and controlled information flow.
B) The `web_search` MCP tool can only be registered on one agent at a time, so report_generation cannot call it.
C) Subagents in the SDK are not permitted to call MCP tools — only the coordinator can.
D) Direct calls would force the report_generation subagent to share its memory with web_search, which is a security risk.

**Correct:** A

**Explanation:**
- **A (correct):** Hub-and-spoke routes all inter-subagent calls through the coordinator so observability, error handling, and information flow stay consistent and debuggable.
- **B:** MCP tools can be registered on multiple agents; this isn't a technical block.
- **C:** Subagents absolutely can call MCP tools; the question is architectural, not capability-based.
- **D:** Subagents have isolated context and don't share memory regardless; this isn't the reason.

---

### Q13 | Scenario 3 | TS 1.3 | difficulty: easy

**Stem:** Your coordinator agent in the Claude Agent SDK is configured but cannot seem to spawn any subagents — calls that should delegate just produce text describing what it would do. You verify subagents are defined and reachable. What is the most likely configuration mistake?

A) The coordinator's `allowedTools` list does not include `"Task"`, which is required to spawn subagents.
B) The coordinator is missing a `system_prompt` field, so it can't issue Task tool calls.
C) The subagents are missing `description` fields, so they aren't discoverable.
D) The coordinator is missing an `api_key` override, so subagent invocations silently fail.

**Correct:** A

**Explanation:**
- **A (correct):** The `Task` tool is the spawn mechanism for subagents; if it isn't in the coordinator's `allowedTools`, the coordinator can't invoke any subagent.
- **B:** A missing system prompt affects quality but doesn't block tool invocation.
- **C:** Descriptions help the model pick the right subagent but aren't a hard prerequisite for the Task tool to function.
- **D:** Subagents inherit credentials from the SDK process; there's no per-coordinator key override that gates spawning.

---

### Q14 | Scenario 3 | TS 1.3 | difficulty: medium

**Stem:** Your coordinator delegates a synthesis task to the synthesis subagent. The synthesis subagent produces a polished narrative, but every citation reads "according to a source," with no document names, URLs, or page numbers. You confirm that the web_search and document_analysis subagents returned proper attribution to the coordinator. What's wrong?

A) The coordinator concatenated all prior findings into the synthesis subagent's prompt as a single prose blob, losing the structured separation between content and source metadata.
B) The synthesis subagent's `max_tokens` is too low to include citations.
C) The Anthropic SDK strips URLs from subagent prompts by default for safety.
D) The web_search MCP tool only returns content, not URLs; the coordinator must call a separate `get_url` tool.

**Correct:** A

**Explanation:**
- **A (correct):** Use a structured data format when passing context between agents so source metadata (URLs, document names, page numbers) stays attached to the content it came from. Flattening to prose destroys attribution.
- **B:** A token budget doesn't selectively erase citations; it would truncate the whole output.
- **C:** No such default stripping exists.
- **D:** Web search MCP tools typically return URLs alongside content; the coordinator just needs to preserve them in handoff.

---

### Q15 | Scenario 3 | TS 1.3 | difficulty: medium

**Stem:** Your research coordinator currently fans out subtasks sequentially: it emits a Task call to web_search, waits for the result, emits a Task call to document_analysis, waits, then emits a Task call to a second web_search for a related subtopic. Total wall time is high. The three subtasks are independent of each other. What is the most direct improvement?

A) Emit multiple Task tool calls in a single coordinator response so the SDK spawns the parallelizable subagents concurrently.
B) Reduce each subagent's `max_tokens` to make individual invocations faster.
C) Pre-warm the subagent processes at coordinator startup.
D) Replace the three subagent calls with a single call to a larger model that does all three in one shot.

**Correct:** A

**Explanation:**
- **A (correct):** Parallel subagent spawning is achieved by emitting multiple Task tool calls in a single coordinator response — the SDK can dispatch them concurrently rather than serially across turns.
- **B:** Smaller outputs may help marginally per call but don't change the sequential dependency pattern.
- **C:** Process pre-warming isn't the bottleneck; the bottleneck is that each Task call happens in its own turn.
- **D:** Collapsing into one big call loses the specialization benefit of the multi-agent design.

---

### Q16 | Scenario 4 | TS 1.3 | difficulty: hard

**Stem:** Your engineering productivity coordinator passes findings to a `refactor_planner` subagent. Right now its prompt looks like: "Step 1: open `auth/login.py`. Step 2: rename function `do_login`. Step 3: update callers." When the codebase has been refactored such that callers are now in `api/auth_routes.py`, the subagent stubbornly tries to follow the original step list and fails. How should the coordinator's delegation be redesigned?

A) Specify research goals and quality criteria for the subagent (e.g., "Plan a safe rename of `do_login`, including locating and updating all call sites; ensure tests still pass") rather than literal step-by-step procedures.
B) Add an explicit "if step fails, retry once" instruction so the subagent recovers from broken steps.
C) Increase the subagent's `max_tokens` so it can fit the new file structure into its plan.
D) Have the coordinator re-do the planning every iteration and shorten the subagent's prompt to a single sentence.

**Correct:** A

**Explanation:**
- **A (correct):** Coordinator prompts should specify goals and quality criteria rather than step-by-step procedures. This lets the subagent adapt its plan when the environment differs from what the coordinator expected.
- **B:** A retry instruction doesn't fix a brittle, prescriptive plan; it just repeats the wrong steps.
- **C:** Token budget isn't why a step-list-following subagent struggles when files moved.
- **D:** Stripping context too far makes the subagent less effective, not more adaptable.

---

### Q17 | Scenario 3 | TS 1.3 | difficulty: medium

**Stem:** You want to explore two alternative analysis approaches from the same baseline document_analysis output: one that emphasizes quantitative results and one that emphasizes qualitative themes. Both should start from an identical shared analysis context. Which mechanism is designed for this?

A) Use `fork_session` to create independent branches from the shared analysis baseline so each branch can explore a divergent approach without polluting the other.
B) Run two synthesis subagents in parallel from the coordinator and average their outputs.
C) Send the analysis output to one subagent and ask it to produce both perspectives in a single response.
D) Resume the same session twice with `--resume` and a different system prompt each time.

**Correct:** A

**Explanation:**
- **A (correct):** `fork_session` is built precisely for forking from a shared baseline to explore divergent approaches in isolation.
- **B:** Two independent subagents don't share a baseline; the divergence is uncontrolled.
- **C:** A single response forced to do both is more likely to blur them, not cleanly explore each direction.
- **D:** `--resume` continues the *same* trajectory; it isn't the right tool for parallel exploration from one shared point.

---

### Q18 | Scenario 4 | TS 1.3 | difficulty: easy

**Stem:** When defining a subagent in `AgentDefinition`, which combination of fields most directly controls its behavior and capabilities?

A) A `description` of its purpose, a `system_prompt` defining its role and instructions, and tool restrictions specifying which tools it may call.
B) Only the `description` field; everything else is inferred from the parent agent.
C) Only the `tools` field; the model decides the system prompt from the tool list.
D) A `temperature` setting and a `max_iterations` count; tool access is implicit.

**Correct:** A

**Explanation:**
- **A (correct):** AgentDefinition typically specifies description (what it's for), system prompt (how it behaves), and tool restrictions (what it can do).
- **B:** Description alone doesn't define behavior; subagents need explicit prompts and tool grants.
- **C:** System prompts aren't inferred from tools; they're authored.
- **D:** Temperature and iteration caps aren't the primary configuration knobs for subagent definition.

---

### Q19 | Scenario 1 | TS 1.4 | difficulty: easy

**Stem:** Compliance requires that `process_refund` never fire unless `get_customer` has returned a verified customer ID in the same session. A new analyst suggests "Just put it in the system prompt." Why is this insufficient for this requirement?

A) Prompt-based instructions are probabilistic — there is always a non-zero failure rate — which is unacceptable when errors have direct financial consequences.
B) System prompts are limited to 2,000 tokens and the verification rule alone would exceed it.
C) The Agent SDK ignores compliance language in system prompts unless it appears in a developer prompt.
D) System prompts are cached and stale rules would never reach production.

**Correct:** A

**Explanation:**
- **A (correct):** Prompt instructions have a non-zero violation rate. For deterministic compliance (especially financial), programmatic enforcement (hooks or prerequisite gates) is required.
- **B:** No such fixed prompt-size limit applies here.
- **C:** The SDK doesn't discriminate against compliance rules in system prompts.
- **D:** Caching doesn't make rules "not reach production"; that's not a real concern.

---

### Q20 | Scenario 1 | TS 1.4 | difficulty: medium

**Stem:** A customer message contains three concerns in one breath: "I want to return order #123, I was double-charged on my last invoice, and please update my shipping address." Your agent currently handles whichever concern was mentioned last and ignores the others. How should the workflow be restructured?

A) Decompose the message into distinct items, investigate each in parallel using shared customer context, then synthesize a unified resolution covering all three.
B) Reply asking the customer to send three separate messages, one per concern.
C) Always handle billing first, then returns, then account updates, regardless of what's in the message.
D) Pick the most urgent concern via a sentiment model and skip the others.

**Correct:** A

**Explanation:**
- **A (correct):** Multi-concern customer requests should be decomposed into distinct items, investigated in parallel using shared context (e.g., the verified customer), and synthesized into one coherent response.
- **B:** Offloading the work to the customer harms experience and resolution rate.
- **C:** A fixed concern order is arbitrary and doesn't ensure all concerns are addressed.
- **D:** Dropping concerns is the original problem; sentiment-based picking just dresses it up.

---

### Q21 | Scenario 1 | TS 1.4 | difficulty: hard

**Stem:** Your agent must escalate to a human when fraud is suspected. Currently it calls `escalate_to_human` with the message "Customer asked about a refund, may be fraud." Human agents complain the handoff is useless — they lack the conversation transcript and have to re-investigate from scratch. Which improvement most directly addresses this?

A) Have the agent compile a structured handoff summary including verified customer ID, root cause analysis, exact refund amount, recent order activity, and a recommended action, then pass it as the escalate_to_human payload.
B) Attach the full raw conversation transcript to `escalate_to_human` so the human has everything.
C) Add a system-prompt instruction telling the agent to be more detailed when escalating.
D) Run a sentiment-analysis tool on the customer's last three messages and include the score with the escalation.

**Correct:** A

**Explanation:**
- **A (correct):** Structured handoff summaries (customer ID, root cause, amount, recommended action) give the human agent — who lacks access to the transcript — the exact information needed to resume work efficiently.
- **B:** Dumping the raw transcript shifts the burden of synthesis onto the human and is the opposite of an actionable handoff.
- **C:** "Be more detailed" is vague and probabilistic, not structured.
- **D:** A sentiment score is a small signal and doesn't substitute for structured operational facts.

---

### Q22 | Scenario 1 | TS 1.4 | difficulty: medium

**Stem:** You want to enforce that the agent never calls `process_refund` for amounts greater than $500 without human approval. Which mechanism gives you deterministic compliance?

A) A programmatic prerequisite gate that inspects the `process_refund` call before it executes, blocks any amount over $500, and routes the case to `escalate_to_human`.
B) A few-shot example in the system prompt demonstrating "if amount > 500, escalate."
C) A line in the system prompt: "Never refund more than $500 without escalating."
D) A post-hoc audit job that flags policy violations the next day for review.

**Correct:** A

**Explanation:**
- **A (correct):** Programmatic prerequisite/interception gates produce deterministic compliance — they catch violations before the tool fires.
- **B:** Few-shot examples improve, but don't guarantee, model behavior.
- **C:** Same — system-prompt rules are probabilistic.
- **D:** Audits detect violations after the fact; they don't prevent them.

---

### Q23 | Scenario 1 | TS 1.4 | difficulty: medium

**Stem:** Your team is debating where to encode the "verify customer before refund" rule. Two proposals are on the table: (1) a system-prompt instruction; (2) a programmatic prerequisite that blocks `process_refund` until `get_customer` has returned a verified ID. The CFO needs to certify the control for SOX compliance. Which proposal is suitable, and why?

A) Proposal 2, because programmatic enforcement provides a deterministic guarantee suitable for control certification; prompt-based rules have a non-zero violation rate.
B) Proposal 1, because system prompts are visible in audit logs and can be diffed over time.
C) Either one, since modern frontier models follow system prompts essentially 100% of the time.
D) Proposal 1 plus a temperature of 0, which together make the rule deterministic.

**Correct:** A

**Explanation:**
- **A (correct):** SOX-style controls require deterministic guarantees. Programmatic prerequisites give that; prompt instructions don't.
- **B:** Auditability of prompts is irrelevant to whether the control reliably prevents the violation.
- **C:** Even strong models violate prompt rules occasionally; that's not zero-risk.
- **D:** Temperature 0 reduces variance but doesn't eliminate prompt non-compliance.

---

### Q24 | Scenario 1 | TS 1.4 | difficulty: easy

**Stem:** A teammate proposes encoding the order of `get_customer` → `lookup_order` → `process_refund` purely with a clear system prompt. What is the central tradeoff they're accepting?

A) Prompt-based ordering is flexible but probabilistic; there will be cases where the model skips or reorders steps, which may be acceptable for low-stakes flows but not for financial ones.
B) Prompt-based ordering is deterministic but slow because the model reads the prompt before every tool call.
C) Prompt-based ordering only works with `temperature=0`.
D) Prompt-based ordering is forbidden by the Agent SDK for refund tools.

**Correct:** A

**Explanation:**
- **A (correct):** The tradeoff is flexibility vs. determinism. Prompts are flexible and easy to iterate but provide no guarantee — acceptable for low-stakes ordering, insufficient for financial controls.
- **B:** Reading the system prompt is not the latency bottleneck.
- **C:** Temperature 0 doesn't make prompts deterministic.
- **D:** The SDK doesn't ban prompt-based ordering; this is a design judgment, not a policy.

---

### Q25 | Scenario 1 | TS 1.5 | difficulty: medium

**Stem:** Your support agent receives heterogeneous timestamps from MCP tools: `get_customer` returns ISO 8601 strings, `lookup_order` returns Unix epoch integers, and a legacy `billing_history` tool returns formatted dates like "Mar 14, 2025." The agent regularly miscompares them ("is this charge before the last refund?") and produces wrong answers. Which Agent SDK mechanism best fixes this without retraining the agent?

A) A PostToolUse hook that normalizes every timestamp coming back from these tools into a single canonical format (e.g., ISO 8601 UTC) before the model processes the tool result.
B) A system-prompt section explaining the three timestamp formats and asking the model to convert as needed.
C) A new MCP tool `convert_time` the agent can call after every other tool call.
D) Replacing the legacy `billing_history` tool with one that returns ISO 8601, and accepting the inconsistency for the other two.

**Correct:** A

**Explanation:**
- **A (correct):** PostToolUse hooks intercept tool results and let you normalize heterogeneous formats deterministically before the model ever sees them — exactly the use case for this hook pattern.
- **B:** Asking the model to convert is probabilistic and adds reasoning burden.
- **C:** A "call convert_time after every tool" pattern adds latency and relies on the model remembering to do it.
- **D:** Partial normalization still leaves two formats inconsistent.

---

### Q26 | Scenario 1 | TS 1.5 | difficulty: medium

**Stem:** Compliance wants `process_refund` calls above $500 to be blocked at runtime and rerouted to `escalate_to_human`. Engineering wants a solution that doesn't depend on prompt phrasing. Which Agent SDK pattern fits?

A) A tool call interception hook that inspects outgoing `process_refund` calls, blocks the call if `amount > 500`, and triggers the `escalate_to_human` workflow instead.
B) A PostToolUse hook that checks the refund result and reverses it if the amount exceeded $500.
C) A system prompt with a hard rule: "Never call `process_refund` with amount > 500."
D) A daily report comparing refund amounts to policy thresholds.

**Correct:** A

**Explanation:**
- **A (correct):** Tool call interception hooks evaluate outgoing tool calls before they execute, allowing deterministic policy enforcement and redirection to alternative workflows.
- **B:** PostToolUse fires after execution — the refund would already be processed; "reversing" adds risk and complexity.
- **C:** Prompt rules are probabilistic.
- **D:** Reports are detective controls, not preventive.

---

### Q27 | Scenario 3 | TS 1.5 | difficulty: hard

**Stem:** Two different web search MCP servers feed your research agent. One returns results with field `published_at` in ISO 8601; the other returns `pubdate` as a Unix timestamp. The agent's "most recent first" sorting frequently mis-orders results across the two sources. You want a fix that is invisible to the model and to downstream subagents. Which approach is best?

A) Register a PostToolUse hook on both web search tools that maps each result to a normalized schema (`published_at: ISO 8601`) before returning the data to the agent.
B) Append a system-prompt rule telling the agent how each source labels and formats dates.
C) Have the synthesis subagent re-sort the merged results after both subagent responses arrive.
D) Switch every downstream subagent to ignore dates and just use insertion order.

**Correct:** A

**Explanation:**
- **A (correct):** A PostToolUse hook is the right mechanism for transparent, deterministic normalization across heterogeneous tool outputs. Once normalized, the model and downstream subagents work on a consistent schema.
- **B:** Telling the model about formats relies on it correctly parsing every time — error-prone.
- **C:** Pushing the fix into synthesis duplicates work and is hard to keep consistent.
- **D:** Discarding date information defeats "most recent first."

---

### Q28 | Scenario 1 | TS 1.5 | difficulty: easy

**Stem:** Which statement best captures when to use hooks versus prompt instructions for compliance rules in the Agent SDK?

A) Use hooks when business rules require guaranteed compliance; use prompts when probabilistic best-effort guidance is acceptable.
B) Use hooks for read-only operations and prompts for write operations.
C) Use prompts for any rule that involves money; use hooks for anything involving customer PII.
D) Prefer prompts in production and hooks only during local development.

**Correct:** A

**Explanation:**
- **A (correct):** Hooks give deterministic guarantees; prompts give probabilistic guidance. Choose based on how strict the compliance requirement is.
- **B:** Read/write isn't the dividing line.
- **C:** Topic-by-topic rules aren't the principle; determinism vs. probability is.
- **D:** Production vs. dev isn't the criterion either.

---

### Q29 | Scenario 1 | TS 1.5 | difficulty: medium

**Stem:** Engineering implements a PostToolUse hook that pretty-prints `lookup_order` results into Markdown tables before the model sees them. After deployment, the agent starts confidently quoting nonexistent fields like "Status emoji" that the hook introduced for human readability. What's the lesson?

A) PostToolUse hooks must preserve the semantic content the model needs to reason; adding fabricated decorative content can be ingested by the model as if it were real data.
B) PostToolUse hooks should never modify text content at all, only metadata.
C) The model should be retrained to ignore decorative content.
D) Replace the hook with a PreToolUse hook so the decoration happens before the call.

**Correct:** A

**Explanation:**
- **A (correct):** Hooks shape what the model sees; any "decorative" addition becomes part of the data the model reasons over. Normalization should be faithful, not creative.
- **B:** A blanket "no text mods" rule is too strong — normalization is legitimate.
- **C:** Retraining isn't accessible and ignores the design problem.
- **D:** PreToolUse runs before the call, not on the result, so it can't change what the model reads from the result.

---

### Q30 | Scenario 3 | TS 1.5 | difficulty: easy

**Stem:** Which is the canonical Agent SDK pattern for transforming tool results before the model processes them?

A) A PostToolUse hook that intercepts tool results and modifies them before they're added to conversation history.
B) A wrapper around `litellm.completion()` that rewrites assistant messages.
C) A system-prompt template variable that pre-formats outputs.
D) A retry decorator on each MCP server that re-executes calls with different parameters.

**Correct:** A

**Explanation:**
- **A (correct):** PostToolUse is exactly the hook pattern for intercepting and transforming tool results.
- **B:** Rewriting assistant messages is the wrong layer — that's the model's output, not tool results.
- **C:** Template variables can shape the prompt, not the dynamic tool results.
- **D:** Retry decorators address transient failures, not transformation.

---

### Q31 | Scenario 5 | TS 1.6 | difficulty: medium

**Stem:** Your CI agent reviews PRs that consistently touch 10–20 files. You currently send the entire diff in one pass and get inconsistent feedback (deep on some files, superficial on others, cross-file issues sometimes missed). Which decomposition strategy fits this predictable, multi-aspect workload best?

A) Prompt chaining: a per-file local-analysis pass followed by a separate cross-file integration pass.
B) Dynamic adaptive decomposition that generates new subtasks based on intermediate findings each review.
C) A single agent loop with a higher `max_tokens` limit so attention is spread evenly.
D) Round-robin sampling: review a random subset of files per PR and trust statistical coverage.

**Correct:** A

**Explanation:**
- **A (correct):** PR reviews are predictable and multi-aspect — a classic case for prompt chaining: per-file passes for consistent local depth, plus a cross-file integration pass for system-level concerns.
- **B:** Dynamic decomposition shines for open-ended investigation; PR reviews don't need that flexibility and pay for it in unpredictability.
- **C:** Larger context windows don't fix attention dilution; the literature here is clear.
- **D:** Sampling deliberately misses files — unacceptable for code review.

---

### Q32 | Scenario 4 | TS 1.6 | difficulty: medium

**Stem:** A product manager hands you a vague request: "add comprehensive tests to our legacy billing service." There's no spec for what "comprehensive" means and you don't know which modules are highest risk. Which decomposition strategy fits?

A) Dynamic adaptive decomposition: first map the codebase structure, identify high-impact areas, then generate a prioritized plan that adapts as dependencies are discovered.
B) Fixed prompt chaining: write unit tests, then integration tests, then end-to-end tests, in that order, for every file.
C) Spawn one subagent per file with identical prompts asking each to "write comprehensive tests."
D) Pick a random 10% sample of files and write tests for them.

**Correct:** A

**Explanation:**
- **A (correct):** Open-ended tasks need adaptive decomposition: explore structure, prioritize by impact, and let the plan evolve as you learn about dependencies and risk.
- **B:** A rigid fixed pipeline ignores what's actually risky and important in this codebase.
- **C:** Uniform per-file fan-out wastes effort and misses the high-impact-first principle.
- **D:** Random sampling doesn't satisfy "comprehensive" by any reasonable measure.

---

### Q33 | Scenario 3 | TS 1.6 | difficulty: hard

**Stem:** Your research agent is asked "What are the most promising near-term applications of quantum error correction?" — an open-ended question whose right subtasks depend on what the literature actually contains. Currently you use a fixed 4-step pipeline (web_search → document_analysis → synthesis → report). Reports keep missing important threads (e.g., LDPC codes, biases toward bosonic codes you weren't expecting). What's the better pattern and why?

A) Switch to dynamic adaptive decomposition: after each round, have the coordinator inspect findings, identify gaps, and generate the next round of subtasks (e.g., spawn a new search on LDPC codes discovered mid-research).
B) Keep the fixed pipeline but run it three times in parallel and merge.
C) Keep the fixed pipeline but enlarge each subagent's `max_tokens` to capture more content per step.
D) Replace the pipeline with a single very large model call that does everything at once.

**Correct:** A

**Explanation:**
- **A (correct):** Open-ended investigation benefits from generating subtasks based on what's discovered at each step, exactly what dynamic adaptive decomposition delivers.
- **B:** Running the same fixed pipeline three times doesn't surface threads it can't currently surface.
- **C:** Bigger outputs from each fixed step don't cover topics the plan never targeted.
- **D:** Collapsing into one shot loses the iterative discovery dynamic that this question requires.

---

### Q34 | Scenario 5 | TS 1.6 | difficulty: easy

**Stem:** When is fixed sequential prompt chaining most appropriate?

A) When the workflow is predictable and multi-aspect, so the steps can be enumerated up front (e.g., per-file analysis then cross-file integration in a PR review).
B) When the next step depends entirely on what the previous step discovers and cannot be planned in advance.
C) When you want to maximize creativity from the model by leaving the steps undefined.
D) When you have only one tool available and don't need to decompose.

**Correct:** A

**Explanation:**
- **A (correct):** Prompt chaining suits predictable workflows with enumerable steps; PR review's local-then-integration pattern is the canonical example.
- **B:** That describes when dynamic decomposition is preferable, not chaining.
- **C:** Chaining is structured, not creativity-maximizing.
- **D:** Single-tool work doesn't need decomposition at all.

---

### Q35 | Scenario 4 | TS 1.6 | difficulty: medium

**Stem:** Your developer-productivity agent reviews changes across a 600-file microservice. You see that one-shot reviews of the diff produce shallow comments for files touched lightly and deep comments for the file the model "fixated" on. Which decomposition adjustment best addresses attention dilution without expanding scope?

A) Split the review into per-file local-analysis passes (one focused call per modified file) followed by one cross-file integration pass.
B) Increase the temperature so the model attends more evenly.
C) Ask the model to "be thorough on every file" in the system prompt.
D) Combine all files into one giant blob and send it as a single user message with line numbers.

**Correct:** A

**Explanation:**
- **A (correct):** Per-file focused passes ensure consistent depth, and a separate integration pass catches cross-file issues — directly mitigating attention dilution.
- **B:** Temperature changes don't fix attention quality.
- **C:** Prompt cheerleading is probabilistic.
- **D:** Combining into one giant blob is the original problem stated differently.

---

### Q36 | Scenario 5 | TS 1.6 | difficulty: medium

**Stem:** Your CI reviewer's per-file pass is solid, but reviewers complain it misses bugs caused by interactions between files — e.g., a renamed function in one file that another file still calls. How should you extend the existing per-file chain?

A) Add a dedicated cross-file integration pass after the per-file passes, with the explicit goal of examining inter-file data flow, call sites, and shared contracts.
B) Drop the per-file passes and do everything in a single combined pass.
C) Have each per-file pass also re-read every other modified file for cross-references.
D) Switch to a larger model and rely on it to "just notice" cross-file issues.

**Correct:** A

**Explanation:**
- **A (correct):** Per-file passes plus a dedicated cross-file integration pass is the classic decomposition for code review: local depth + system-level checks.
- **B:** Reverting to a single pass reintroduces attention dilution.
- **C:** Each file re-reading all others duplicates work and recreates the diluted-attention problem at the file level.
- **D:** Hoping a bigger model notices is not a structural fix.

---

### Q37 | Scenario 2 | TS 1.7 | difficulty: easy

**Stem:** You spent an hour with Claude Code planning a tricky migration for your `auth-service`. You named the session `auth-migration-plan`. The next morning, you want to continue right where you left off. What is the most direct way to do that?

A) Run Claude Code with `--resume auth-migration-plan` to continue the named session.
B) Start a new session and paste the prior transcript at the top as a user message.
C) Open a forked session from the original with `fork_session` and continue there.
D) Use the global `/restart` command, which restores the last session automatically.

**Correct:** A

**Explanation:**
- **A (correct):** `--resume <session-name>` is the canonical way to continue a specific prior conversation by name.
- **B:** Pasting transcripts loses tool result fidelity and is brittle.
- **C:** `fork_session` is for branching into divergent exploration, not for picking up the same trajectory.
- **D:** There is no such global restart command that reliably attaches to a named session.

---

### Q38 | Scenario 4 | TS 1.7 | difficulty: medium

**Stem:** You resumed `--resume legacy-explore` after a teammate refactored `payments/processor.py`. The agent's first few responses cite obsolete function names that no longer exist. What should you do?

A) Inform the resumed session about the specific file changes (which files were modified and how) so it can re-analyze those areas instead of trusting stale tool results.
B) Force the agent to forget everything and re-explore the entire codebase from scratch.
C) Increase the agent's temperature so it explores more broadly.
D) Ignore the issue; the agent will eventually figure out the changes on its own.

**Correct:** A

**Explanation:**
- **A (correct):** When resuming after code changes, the right move is to inform the agent about the specific changes so it can do targeted re-analysis rather than relying on stale tool results.
- **B:** Full re-exploration wastes time when only a few files changed; targeted re-analysis is cheaper.
- **C:** Temperature doesn't make stale data fresh.
- **D:** Without a nudge, the agent has no signal that its memory is stale.

---

### Q39 | Scenario 2 | TS 1.7 | difficulty: hard

**Stem:** A week ago you ran a deep session that produced a detailed plan for refactoring the billing module. Since then, the team has merged 47 PRs touching billing, including significant restructuring. You want to continue the refactor work today. Which approach is most reliable?

A) Start a fresh session with a structured summary of the prior plan injected as context; the prior session's tool results are stale and likely misleading.
B) Resume the original session with `--resume` and assume the agent will figure out what's changed.
C) Fork the original session with `fork_session` and continue in the fork.
D) Resume the original session and ask "what's changed since last week?" expecting the agent to know.

**Correct:** A

**Explanation:**
- **A (correct):** When prior tool results are stale (especially after extensive code changes), it's more reliable to start fresh with a structured summary injected as context rather than resume a session whose memory has drifted out of sync with reality.
- **B:** Resuming hopes the agent self-corrects; it has no way to know which of its cached observations are now wrong.
- **C:** Forking still carries the stale baseline forward.
- **D:** The agent cannot magically know what changed; this just produces confabulated answers.

---

### Q40 | Scenario 4 | TS 1.7 | difficulty: medium

**Stem:** Two engineers want to evaluate competing refactor strategies for the same legacy module: one inverts dependencies via interfaces, the other extracts a service. Both should start from the same shared codebase analysis already done in session `legacy-explore`. Which mechanism is best?

A) Use `fork_session` to create two independent branches from `legacy-explore`, one per strategy, so each evolves without contaminating the other.
B) Use `--resume legacy-explore` from two terminals simultaneously and let them race.
C) Start two new sessions from scratch and re-do the analysis in each.
D) Have one session evaluate both strategies sequentially.

**Correct:** A

**Explanation:**
- **A (correct):** `fork_session` is purpose-built for parallel exploration branches from a shared analysis baseline — exactly this comparison-of-strategies use case.
- **B:** Two terminals resuming the same session creates a race and conflicting state.
- **C:** Re-doing the analysis discards reusable shared baseline work.
- **D:** Sequential exploration in a single session contaminates each strategy with reasoning from the other.

---

### Q41 | Scenario 2 | TS 1.7 | difficulty: medium

**Stem:** A teammate asks: "When should I `--resume` vs. start fresh with a summary?" What's the right rule of thumb?

A) Resume when most of the prior context is still valid; start fresh with a structured summary when prior tool results are likely stale.
B) Always resume — it's cheaper than rebuilding context.
C) Always start fresh — old sessions accumulate too much noise.
D) Resume only when the codebase hasn't changed in the last 24 hours.

**Correct:** A

**Explanation:**
- **A (correct):** The right axis is staleness of prior tool results. If the world hasn't moved much, resume; if it has, start fresh with a curated summary to avoid being misled by cached observations.
- **B:** "Always resume" risks operating on stale state.
- **C:** "Always start fresh" wastes accumulated useful context when it's still valid.
- **D:** A blanket 24-hour rule is arbitrary; the question is whether the relevant files changed.

---

### Q42 | Scenario 4 | TS 1.7 | difficulty: hard

**Stem:** You forked a session with `fork_session` to evaluate two test strategies (mocking-heavy vs. integration-heavy). Halfway through the mocking-heavy branch you realize the analysis baseline both forks inherited assumed an older version of the `db_client` library that has since shipped a breaking change. What is the right corrective action?

A) Inject a structured summary of the breaking change into the active fork (and the other one) and instruct the agent to re-analyze only the affected areas; do not assume the fork will discover the change on its own.
B) Abandon both forks and start completely from scratch with no shared analysis.
C) Continue both forks unchanged; the divergent strategies will surface the issue eventually.
D) Use `--resume` on the parent session to "refresh" both forks.

**Correct:** A

**Explanation:**
- **A (correct):** Forks inherit the parent's baseline. When that baseline is invalidated by an external change, the correct action is to inject a targeted summary of the change so the agent can re-analyze affected areas — rather than starting over or hoping the divergent reasoning catches it.
- **B:** Discarding all shared analysis throws away useful work; targeted updates are cheaper.
- **C:** Hoping divergent strategies surface a known issue is unreliable.
- **D:** `--resume` doesn't propagate updates into existing forks; that isn't what the command does.


## Domain 2: Tool Design & MCP Integration


42 scenario-based multiple-choice questions covering Task Statements 2.1–2.5.

---

### Q1 | Scenario 3 | TS 2.1 | difficulty: medium

**Stem:** In your multi-agent research system, two MCP tools exist: `analyze_content` ("Analyzes content and returns insights") and `analyze_document` ("Analyzes document content and returns findings"). Production traces show the web-search subagent calls `analyze_document` on raw HTML snippets in ~30% of cases when it should call `analyze_content`, and downstream the synthesis subagent occasionally hits `analyze_content` on PDF text where `analyze_document` would have produced better-structured output. What is the most effective fix?

A) Add a system-prompt rule stating "use analyze_document for documents and analyze_content for web content."
B) Rename and rewrite the tools — e.g., `extract_web_results` (for HTML/search snippets) and `extract_pdf_insights` (for structured documents) — with descriptions that spell out input format, expected output, and when each is correct versus its sibling.
C) Wrap both in a single new `analyze_anything` tool that dispatches internally based on input MIME type.
D) Use `tool_choice: "any"` to force the agent to choose between the two on each turn.

**Correct:** B

**Explanation:**
- **B (correct):** The root cause is near-identical descriptions of overlapping tools. Renaming for purpose-specificity and writing differentiated descriptions (inputs, outputs, "use this when…") is the canonical fix for misrouting between similar tools.
- **A:** Adds prompt rules but leaves the ambiguous tool descriptions in place; tool descriptions are the primary signal the model uses for selection.
- **C:** Collapsing into a generic dispatcher hides intent and forces internal heuristics that the model can't reason about during planning.
- **D:** `tool_choice: "any"` forces *a* tool call but doesn't disambiguate *which* of the two confusable tools to pick.

---

### Q2 | Scenario 1 | TS 2.1 | difficulty: easy

**Stem:** You are writing the description for the `process_refund` MCP tool in the support agent. Which description is most likely to result in reliable, correct selection by Claude?

A) "Process a refund."
B) "Issues a refund for an order. Input: `order_id` (string, must be a verified order returned by `lookup_order`) and `amount_cents` (integer ≤ original charge). Returns refund confirmation with `refund_id`. Use only after `get_customer` has verified the caller; do not use for partial-credit gift cards (use `issue_gift_card_credit` instead)."
C) "Refunds money to customers as needed."
D) "Calls the refund API."

**Correct:** B

**Explanation:**
- **B (correct):** Good tool descriptions include input format, expected outputs, prerequisites, edge cases, and boundaries versus similar tools — all of which this description supplies.
- **A:** Minimal description; the model has no information about inputs, outputs, or when to use it versus alternatives.
- **C:** Vague action statement with no contract.
- **D:** Describes implementation rather than purpose; provides no guidance to the model about when or how to call it.

---

### Q3 | Scenario 4 | TS 2.1 | difficulty: medium

**Stem:** Your developer-productivity agent has a single MCP tool `analyze_document` used by engineers for three distinct needs: pulling specific data points out of a spec, summarizing a long design doc, and verifying that a claim in code comments matches the source RFC. Engineers complain that the same tool returns wildly different output shapes depending on how Claude phrases the call, and downstream code that consumes the output is brittle. What is the recommended redesign?

A) Keep `analyze_document` but add a `mode` enum parameter (`extract`, `summarize`, `verify`) and document each mode in the description.
B) Split `analyze_document` into three purpose-specific tools — `extract_data_points`, `summarize_content`, `verify_claim_against_source` — each with a defined input/output contract.
C) Leave the tool as-is and instruct engineers via CLAUDE.md to phrase requests consistently.
D) Add few-shot examples in the system prompt that show each of the three use cases.

**Correct:** B

**Explanation:**
- **B (correct):** Splitting a generic tool into purpose-specific tools with defined I/O contracts is the prescribed pattern: it gives the model unambiguous selection signals and stabilizes output shape per tool.
- **A:** A `mode` enum still leaves output ambiguous and ties three contracts to one tool, which is the source of the brittleness.
- **C:** Pushes complexity onto users and depends on humans being consistent — not a structural fix.
- **D:** Few-shot examples may help selection but don't address the unstable output contract.

---

### Q4 | Scenario 1 | TS 2.1 | difficulty: medium

**Stem:** A new system prompt for the support agent includes the line: "When customers describe an *issue*, always search for related *issues* in our knowledge base." Soon afterward you notice the agent invoking a `search_github_issues` MCP tool (intended for engineering bug triage) on routine refund tickets. What is the most likely cause and best remediation?

A) The model is hallucinating tool names; rename `search_github_issues` to `search_kb_articles`.
B) The system prompt's repeated use of the keyword "issues" is steering tool selection toward `search_github_issues`. Rephrase the prompt (e.g., "search the knowledge base for related articles") and/or tighten the tool description to scope it to engineering bug reports only.
C) Lower the temperature so the model is less likely to pick the wrong tool.
D) Remove `search_github_issues` entirely from the support agent.

**Correct:** B

**Explanation:**
- **B (correct):** Keyword-sensitive system-prompt wording can override well-written tool descriptions. Reviewing the prompt for words that overlap with tool names and adjusting either the wording or the tool's description is the targeted fix.
- **A:** Hallucination of a non-existent tool isn't the issue — the tool exists and the prompt is steering toward it.
- **C:** Temperature won't reliably overcome an explicit lexical cue in the prompt.
- **D:** Removing the tool may be appropriate if it isn't needed, but the diagnostic insight (prompt wording is steering selection) is what the task statement is testing.

---

### Q5 | Scenario 3 | TS 2.1 | difficulty: hard

**Stem:** Your synthesis subagent has three tools whose descriptions read: `cite_source` ("Cites a source for a claim"), `attach_reference` ("Attaches a reference to a claim"), and `link_citation` ("Links a citation to a claim"). The subagent produces reports where ~20% of claims have the wrong tool used or no tool used at all. Before refactoring, what is the most useful first step?

A) Add a global system-prompt instruction listing all three tools and when to use each.
B) Audit the three tools for functional overlap, define which one (if any) is needed for which path, then either consolidate (e.g., a single `add_citation` tool) or differentiate descriptions to spell out the unique contract of each.
C) Force `tool_choice` to `cite_source` on every turn.
D) Replace all three with a non-tool prompt instruction telling the model to output Markdown footnotes inline.

**Correct:** B

**Explanation:**
- **B (correct):** When three tools have near-identical descriptions, the first design step is to determine whether they're functionally distinct. Either consolidate to one well-described tool or differentiate purposes — both fixes start from this audit.
- **A:** Adds prompt noise on top of already overlapping tool descriptions; doesn't fix the structural ambiguity.
- **C:** Mechanically forces one tool but doesn't address whether the others should exist.
- **D:** Replacing structured citation tooling with free-text footnotes loses validation and downstream linking — a regression, not a fix.

---

### Q6 | Scenario 4 | TS 2.1 | difficulty: easy

**Stem:** An engineer adds a new MCP tool to the developer-productivity agent and writes the description as: "Searches code." After deployment, the agent rarely picks this tool, instead falling back to the built-in `Grep`. What is the most likely cause?

A) Built-in tools always take priority over MCP tools regardless of description quality.
B) The new tool's description is too minimal to differentiate it from `Grep`; without input format, output description, and a "use this instead of Grep when…" boundary, Claude has no reason to prefer it.
C) MCP tools are only discovered on the second turn of a conversation.
D) The tool name needs to start with an uppercase letter to be considered.

**Correct:** B

**Explanation:**
- **B (correct):** Tool descriptions are the primary mechanism the model uses for selection. A minimal description like "Searches code" gives no advantage over the rich built-in `Grep` semantics, so Claude defaults to what it knows.
- **A:** Built-in tools have no inherent priority — description quality drives selection.
- **C:** All connected MCP tools are available from the start of the session.
- **D:** Tool naming conventions don't impose case requirements that affect selection.

---

### Q7 | Scenario 1 | TS 2.1 | difficulty: medium

**Stem:** Your support agent has both `escalate_to_human` ("Escalates the conversation to a human agent") and a newly added `transfer_to_specialist` ("Transfers the conversation to a specialist agent"). Production logs show both tools being used interchangeably, including escalations on tickets that should have gone to a billing specialist (and vice-versa). Which intervention best addresses the root cause?

A) Add a parameter `reason: string` to both tools and ask the model to fill it in carefully.
B) Rewrite the descriptions to spell out exact scope: e.g., `escalate_to_human` "for cases the agent cannot resolve and that require a generalist human (refund disputes >$500, abuse, threats)" and `transfer_to_specialist` "for routing to a named specialist queue (`billing`, `tech_support`, `loyalty`); not for general escalation."
C) Lower the model temperature on this agent.
D) Combine both into a single `route_ticket` tool with a destination parameter.

**Correct:** B

**Explanation:**
- **B (correct):** When two tools serve different purposes but have overlapping descriptions, the right move is to rewrite descriptions that explicitly differentiate scope and include "use this versus…" boundary language.
- **A:** A reason parameter doesn't help the model decide which tool to call.
- **C:** Temperature isn't the cause; ambiguous descriptions are.
- **D:** Combining can work but it conflates two genuinely different operations (human handoff vs queue routing). Differentiation is preferred when the operations are distinct.

---

### Q8 | Scenario 3 | TS 2.1 | difficulty: medium

**Stem:** A senior engineer reviews your research-system tool definitions and asks: "What signals does the LLM actually use to decide which tool to call?" Which is the most accurate answer?

A) Primarily the order in which tools were registered with the SDK.
B) Primarily the tool descriptions (and parameter schemas) plus contextual cues in the system prompt and conversation; tool names and JSON-schema field descriptions matter, descriptions matter most.
C) Primarily a learned lookup table inside the model that maps user phrasings to tool names.
D) Primarily the assistant's most recent prior turn.

**Correct:** B

**Explanation:**
- **B (correct):** Tool descriptions are the primary mechanism, supported by parameter schemas, names, and the surrounding prompt and conversation context.
- **A:** Registration order has no special selection effect.
- **C:** There is no internal user-phrasing-to-tool table; selection is driven by the in-context tool spec.
- **D:** Recent turns provide context but are not the primary selection driver.

---

### Q9 | Scenario 4 | TS 2.1 | difficulty: hard

**Stem:** You added an MCP tool `fetch_url` ("Fetches the contents of any URL") to support engineers reading docs. Over time, you observe Claude using it to call internal admin endpoints based on guesses, including endpoints that mutate state. Stricter URL scoping is needed. Which redesign best aligns with the principle of constraining tool surface through interface design?

A) Replace `fetch_url` with `load_document` that takes a `document_id` from a known catalog or a URL that the tool itself validates against an allowlist; document the constraint in the description.
B) Keep `fetch_url` and instruct Claude in CLAUDE.md to "never call internal endpoints."
C) Keep `fetch_url` but rate-limit calls to 1 per minute.
D) Move `fetch_url` to a separate MCP server.

**Correct:** A

**Explanation:**
- **A (correct):** Replacing a generic tool with a constrained, purpose-specific alternative (allowlisted documents, validated URLs) is the recommended pattern. The constraints become part of the interface contract rather than relying on the model to follow instructions.
- **B:** Pure prompt instruction is probabilistic; doesn't prevent abuse.
- **C:** Rate limiting reduces volume but not the underlying class of misuse.
- **D:** Moving to another server doesn't change what the tool can do.

---

### Q10 | Scenario 1 | TS 2.2 | difficulty: easy

**Stem:** The `lookup_order` MCP tool currently returns the same shape on every failure: `{ "isError": true, "content": [{ "type": "text", "text": "Operation failed" }] }`. After a multi-day investigation you realize the agent retries indefinitely on permanent failures (e.g., order doesn't exist) and gives up too quickly on transient failures (e.g., upstream 503). What change to the error response is most effective?

A) Stop returning errors entirely and let timeouts speak for themselves.
B) Return structured error metadata including `errorCategory` (`transient`/`validation`/`business`/`permission`), an `isRetryable` boolean, and a human-readable description.
C) Always set `isRetryable: true` so the agent can decide.
D) Increase the agent's retry budget globally.

**Correct:** B

**Explanation:**
- **B (correct):** Uniform "Operation failed" responses give the agent no basis for recovery. Structured metadata with category and retryability is the documented fix.
- **A:** Removing errors makes the situation strictly worse.
- **C:** Lying about retryability causes wasted retries on permanent failures.
- **D:** A bigger retry budget doesn't distinguish transient from permanent failures.

---

### Q11 | Scenario 1 | TS 2.2 | difficulty: medium

**Stem:** A customer requests a refund for a 60-day-old purchase, but your refund policy is 30 days. The `process_refund` MCP tool refuses internally and returns: `{ "isError": true, "content": [{ "type": "text", "text": "Refund failed" }] }`. The agent often retries the call several times, then escalates with a confusing message. What is the best fix to the tool's error response?

A) Return `{ isError: true, errorCategory: "business", isRetryable: false, message: "Order is outside the 30-day refund window (purchased 60 days ago). Eligible for store credit instead." }`.
B) Return `{ isError: true, errorCategory: "transient", isRetryable: true, message: "Refund failed, please retry." }`.
C) Return success with `refund_id: null` so the agent moves on.
D) Throw a non-MCP exception so the SDK halts the session.

**Correct:** A

**Explanation:**
- **A (correct):** Business-rule failures should be marked with `errorCategory: "business"`, `isRetryable: false`, and a customer-friendly explanation that lets the agent communicate accurately and offer alternatives.
- **B:** Mislabels a policy violation as transient, causing wasted retries.
- **C:** Faking success hides a policy denial from the agent and the customer.
- **D:** Crashing the session loses context and degrades UX.

---

### Q12 | Scenario 3 | TS 2.2 | difficulty: medium

**Stem:** Your web-search subagent encounters intermittent 503s from the upstream search API. Currently it propagates every failure up to the coordinator, which restarts the entire research plan. What is the recommended pattern?

A) Have the subagent retry transient errors locally with backoff and only escalate to the coordinator after it has exhausted local recovery — propagating partial results and a description of what was attempted.
B) Have the coordinator catch all errors and immediately restart all subagents.
C) Have the subagent fail fast on the first 503 to avoid hiding problems.
D) Have the subagent swallow all 503s and return empty results so the coordinator never sees the failure.

**Correct:** A

**Explanation:**
- **A (correct):** Local recovery for transient failures within the subagent (with structured propagation of unresolved failures, partial results, and attempts) is the documented multi-agent error pattern.
- **B:** Restarting the whole plan wastes work and amplifies the impact of transient failures.
- **C:** Failing fast on transient errors discards information the subagent could have recovered from.
- **D:** Hiding errors prevents the coordinator from making informed decisions and produces inaccurate final reports.

---

### Q13 | Scenario 1 | TS 2.2 | difficulty: hard

**Stem:** Your `get_customer` tool currently returns `{ isError: true, message: "Not found" }` when a phone number lookup yields no matching customer. Postmortems show the agent treats "not found" the same as "service unavailable" and triggers a retry loop. Which redesign is correct?

A) Return `{ isError: true, errorCategory: "transient", isRetryable: true }` so the agent always retries.
B) Distinguish "valid empty result" from "access failure": return a successful response with `customers: []` when the lookup completed successfully but matched nothing, reserving `isError: true` for actual access failures (timeouts, auth, upstream errors) with `errorCategory` and `isRetryable` set appropriately.
C) Always return `isError: true` and let the agent figure out the meaning from the message text.
D) Return `isError: false` with a `success_probability` field.

**Correct:** B

**Explanation:**
- **B (correct):** "No matches" is a successful query, not a failure. The MCP error pattern distinguishes access failures (which need retry decisions) from valid empty results.
- **A:** Treating empty results as transient guarantees an infinite retry loop.
- **C:** Forces the agent to parse free-text strings to infer semantics.
- **D:** Fabricating a probability field is not part of the error contract and is undefined for retry logic.

---

### Q14 | Scenario 1 | TS 2.2 | difficulty: medium

**Stem:** Your support agent has a tool `process_refund` that fails with HTTP 401 when the API token has expired. Currently the tool returns `{ isError: true, message: "Unauthorized" }`. The agent has been observed retrying many times and eventually escalating. Which error response category and metadata best fit this case?

A) `errorCategory: "permission"`, `isRetryable: false`, plus a message telling the agent that the token needs to be rotated by an operator.
B) `errorCategory: "transient"`, `isRetryable: true` — auth issues usually resolve themselves.
C) `errorCategory: "validation"`, `isRetryable: false` — the input was invalid.
D) `errorCategory: "business"`, `isRetryable: false` — a policy violation.

**Correct:** A

**Explanation:**
- **A (correct):** Expired credentials are a permission failure; retries by the agent won't resolve it. Marking `isRetryable: false` and surfacing operator-actionable language prevents wasted attempts.
- **B:** Auth errors don't self-heal; the agent retrying is pure waste.
- **C:** The user's input was fine — the agent's credential was rejected.
- **D:** A policy violation would imply the operation is forbidden by business rules, not by authentication state.

---

### Q15 | Scenario 3 | TS 2.2 | difficulty: easy

**Stem:** Which of the following best describes the purpose of the MCP `isError` flag in a tool response?

A) It tells the MCP server to retry the tool automatically.
B) It communicates to the agent that the tool call failed so the agent can read the error metadata and decide how to recover, instead of treating the response as a successful result.
C) It is a UI-only flag that has no effect on agent behavior.
D) It causes the agent's session to terminate.

**Correct:** B

**Explanation:**
- **B (correct):** `isError` is the standard MCP pattern for surfacing failures to the agent so it can make recovery decisions.
- **A:** The MCP server does not auto-retry based on `isError`.
- **C:** The flag is meaningful to the agent's planning, not just UI.
- **D:** It does not terminate the session; it informs the agent's next action.

---

### Q16 | Scenario 1 | TS 2.2 | difficulty: medium

**Stem:** You receive a bug report that the support agent tells frustrated customers things like "I encountered an unexpected error, please try again" when in fact the refund was denied because the order belongs to a different account. The tool's current error payload is `{ isError: true, message: "Cannot process" }`. What is the most effective fix?

A) Tell the agent in the system prompt to always offer specific reasons for refund denials.
B) Change the tool to return a structured business error: `{ isError: true, errorCategory: "business", isRetryable: false, message: "Refund denied: order belongs to a different account. Verify the caller's identity or escalate." }` — giving the agent the concrete reason and a next step it can communicate.
C) Add a retry with exponential backoff inside the tool.
D) Have the tool return success and a `denied: true` field instead of using `isError`.

**Correct:** B

**Explanation:**
- **B (correct):** Customer-friendly explanations on business-rule failures, plus `isRetryable: false`, let the agent communicate accurately and choose an appropriate next step.
- **A:** Without the underlying reason, the agent can't generate accurate text.
- **C:** Retrying a policy-denied refund won't change the outcome.
- **D:** Repurposing success responses to signal failure breaks the MCP contract the agent expects.

---

### Q17 | Scenario 3 | TS 2.2 | difficulty: hard

**Stem:** A subagent in your research system fetched 12 pages from one source but failed to fetch the 13th because of a 429 rate limit, after retrying locally per backoff policy. What is the correct way to propagate this to the coordinator?

A) Fail the entire subagent call with `isError: true` and no payload, so the coordinator restarts the task from scratch.
B) Return a successful result containing the 12 fetched pages along with structured metadata about the failed page: source URL, `errorCategory: "transient"`, `isRetryable: true`, and a note about local retry attempts already made — letting the coordinator decide whether to defer, route around, or retry later.
C) Drop the failed page silently and return only the 12 successes.
D) Return a hard error and rely on the coordinator to query logs to find the partial results.

**Correct:** B

**Explanation:**
- **B (correct):** Subagents should propagate partial results plus structured metadata about what failed and what was attempted, so the coordinator can make an informed decision.
- **A:** Discards 12 successful fetches and forces wasted re-work.
- **C:** Hides failure from the coordinator and may produce a citation gap without explanation.
- **D:** Logs aren't part of the agent's reasoning context — structured metadata in the response is.

---

### Q18 | Scenario 1 | TS 2.3 | difficulty: easy

**Stem:** Your support agent currently has 18 MCP tools available (refunds, account, billing, shipping, returns, gift cards, loyalty points, ticketing, escalations, etc.). Logs show frequent miscalls between similarly named tools and a drop in first-contact resolution since the surface grew. Which intervention is most aligned with the underlying principle?

A) Add more detailed descriptions to all 18 tools.
B) Reduce the agent's tool set to the 4–5 it actually needs for the most common ticket types, routing other needs through a dispatcher or specialized subagents.
C) Switch to a larger model with more parameters.
D) Force `tool_choice: "any"` so the agent always calls a tool.

**Correct:** B

**Explanation:**
- **B (correct):** Tool selection reliability degrades as the surface grows (e.g., 18 tools vs 4–5). The principle is to scope each agent's tool set to its role.
- **A:** Better descriptions help, but the core decision-complexity problem remains with 18 options.
- **C:** Model size doesn't solve attention/selection issues caused by tool surface.
- **D:** Forcing a tool doesn't fix which tool is selected.

---

### Q19 | Scenario 3 | TS 2.3 | difficulty: medium

**Stem:** Your synthesis subagent — whose job is to combine findings into a draft report — has access to `web_search`, `fetch_url`, `extract_text`, and `cite_source`. You notice it occasionally calls `web_search` mid-synthesis, producing duplicate or off-topic citations. What is the recommended fix?

A) Restrict the synthesis subagent's tools to those needed for synthesis (e.g., `cite_source` and possibly a scoped `verify_fact`), routing any new search needs back through the coordinator to a search subagent.
B) Tell the synthesis subagent in its system prompt not to use `web_search`.
C) Lower the temperature on the synthesis subagent.
D) Rename `web_search` to `do_not_use_search`.

**Correct:** A

**Explanation:**
- **A (correct):** Agents with tools outside their specialization tend to misuse them. Scoping tool access to the subagent's role — with carefully chosen cross-role tools (e.g., a constrained `verify_fact`) — is the documented pattern.
- **B:** Probabilistic prompt instructions don't reliably prevent tool calls when the tool is available.
- **C:** Temperature doesn't change tool availability.
- **D:** Renaming to deter use is brittle and unprofessional.

---

### Q20 | Scenario 3 | TS 2.3 | difficulty: medium

**Stem:** During synthesis, the subagent occasionally needs to confirm a single statistic (e.g., a population figure) without launching a full research delegation. Which design best balances scoped tool access with high-frequency cross-role needs?

A) Give the synthesis subagent the full `web_search` tool.
B) Provide a scoped `verify_fact` tool that takes a single claim and returns a verification result, while still routing complex multi-source research back through the coordinator.
C) Have the synthesis subagent stop and ask the user to verify facts manually.
D) Pre-fetch all possible facts before synthesis begins so no verification is ever needed.

**Correct:** B

**Explanation:**
- **B (correct):** A scoped cross-role tool for a high-frequency narrow need is the prescribed pattern: it avoids handing over a powerful general tool while keeping complex cases routed through the coordinator.
- **A:** Full search re-opens the misuse risk this design avoids.
- **C:** Pushes the cost to the user and breaks the autonomous flow.
- **D:** Pre-fetching all possible facts is infeasible for open-ended research.

---

### Q21 | Scenario 1 | TS 2.3 | difficulty: easy

**Stem:** During an extraction pipeline turn, the support agent needs to ALWAYS call exactly one tool first (specifically `get_customer`) before any other tool. Which Anthropic-supported mechanism most directly enforces this on the first turn?

A) `tool_choice: { "type": "tool", "name": "get_customer" }` to force the model to call that specific tool on the first turn, then process subsequent steps in follow-up turns.
B) `tool_choice: "any"` with a strong system-prompt hint.
C) A custom regex filter on the model output that rejects non-`get_customer` responses.
D) Setting `temperature: 0`.

**Correct:** A

**Explanation:**
- **A (correct):** Forced tool selection via `tool_choice` with a specific tool name is the supported way to guarantee that exact tool is called on a turn.
- **B:** `"any"` forces *a* tool call but not which one.
- **C:** Output filtering doesn't change what the model does.
- **D:** Lower temperature makes selection more deterministic but doesn't guarantee a specific tool.

---

### Q22 | Scenario 4 | TS 2.3 | difficulty: medium

**Stem:** You're building a metadata-enrichment pipeline where the agent should always run `extract_metadata` first, then optionally call zero or more enrichment tools (e.g., `tag_topics`, `detect_language`). What is the cleanest implementation?

A) Set `tool_choice` to force `extract_metadata` on the first turn, then let subsequent turns proceed with `tool_choice: "auto"` so the model decides which enrichment tools (if any) to call.
B) Concatenate `extract_metadata` and all enrichment tools into one giant tool.
C) Force `tool_choice: "any"` on every turn so the agent keeps calling tools.
D) Hard-code the enrichment sequence in the prompt and trust the model to follow it.

**Correct:** A

**Explanation:**
- **A (correct):** Forced tool selection guarantees the first step; auto on subsequent turns gives the model latitude to choose enrichment based on context. This is the canonical pattern.
- **B:** A mega-tool hides intent and prevents the model from skipping unneeded enrichments.
- **C:** Always forcing a tool can cause unnecessary calls when no enrichment is needed.
- **D:** Pure prompt-based sequencing is probabilistic, not guaranteed.

---

### Q23 | Scenario 1 | TS 2.3 | difficulty: medium

**Stem:** You add a feature where the agent must call a tool (any tool — not just respond conversationally) when the user message contains a refund or billing keyword. Without this, the agent sometimes responds with a chat-only message. Which `tool_choice` setting most directly addresses this?

A) `tool_choice: "auto"` (the default).
B) `tool_choice: "any"` — guarantees the model calls at least one tool rather than returning conversational text.
C) `tool_choice: { "type": "tool", "name": "process_refund" }` for every turn.
D) Disable all tools and rely on the system prompt.

**Correct:** B

**Explanation:**
- **B (correct):** `tool_choice: "any"` ensures the model invokes some tool rather than producing free-text, which is exactly the behavior described.
- **A:** Auto lets the model decide and so leaves room for chat-only responses.
- **C:** Forcing a specific tool may pick the wrong one for billing-only messages.
- **D:** Removes the capability entirely.

---

### Q24 | Scenario 3 | TS 2.3 | difficulty: hard

**Stem:** The coordinator in your research system currently has access to every subagent's tools so it can "help out" if a subagent is slow. Over time you observe the coordinator skipping delegation and calling subagent tools directly, sometimes inconsistently. Which redesign best aligns with the multi-agent tool distribution principles?

A) Keep all tools on the coordinator and add prompt rules telling it to delegate.
B) Strip the coordinator down to a small delegation/orchestration toolset (e.g., `dispatch_to_subagent`, `aggregate_results`), giving each subagent its own scoped toolset; the coordinator should not have direct access to subagent-specific tools.
C) Duplicate every subagent's tools onto the coordinator and add a tag on each tool.
D) Have the coordinator and every subagent share the same flat list of tools.

**Correct:** B

**Explanation:**
- **B (correct):** Coordinators delegate; tool distribution should keep each agent's surface scoped to its role. Stripping the coordinator down to delegation tools prevents the "help out" anti-pattern.
- **A:** Prompt instructions are not enforceable; structural fix is needed.
- **C:** Duplication makes the issue worse.
- **D:** A shared flat list is the original problem.

---

### Q25 | Scenario 4 | TS 2.3 | difficulty: easy

**Stem:** Which statement best captures the principle behind scoping an agent's tool set?

A) More tools always lead to better behavior because the agent has more options.
B) Restricting an agent's tools to those relevant to its role increases selection reliability and reduces cross-specialization misuse; cross-role needs are handled by limited, well-scoped tools or by coordinator routing.
C) Tool sets should be expanded over time until selection accuracy reaches 100%.
D) Each agent should always have access to every tool registered in the project.

**Correct:** B

**Explanation:**
- **B (correct):** This is the principle stated directly in the task statement.
- **A:** Tool surface growth degrades selection reliability.
- **C:** Selection accuracy is degraded, not improved, by surface growth.
- **D:** Shared global access is the anti-pattern.

---

### Q26 | Scenario 1 | TS 2.3 | difficulty: hard

**Stem:** The support agent often replies with chatty acknowledgements ("Got it, I'll look that up for you!") instead of immediately calling `get_customer`, even when it has a phone number to look up. You want to ensure that on intake turns it always calls `get_customer` first, but on later turns you want the model to choose freely. Which configuration is best?

A) Set `tool_choice` to force `get_customer` on the intake turn (e.g., when only a phone or email has been provided), then switch back to `tool_choice: "auto"` for subsequent turns.
B) Always set `tool_choice: "any"` so the agent never chats.
C) Set `tool_choice` to force `get_customer` on every turn for the entire conversation.
D) Disable `tool_choice` and add a strict CLAUDE.md rule.

**Correct:** A

**Explanation:**
- **A (correct):** Forced tool selection on the first turn followed by auto on later turns is the canonical pattern for "always start with X, then plan freely."
- **B:** Always forcing a tool can cause inappropriate tool calls when none is needed.
- **C:** Forcing `get_customer` every turn would be incorrect once verification is done.
- **D:** Prompt-only solutions are probabilistic.

---

### Q27 | Scenario 4 | TS 2.4 | difficulty: easy

**Stem:** Your team wants a shared MCP server (an internal `repo-search` service) available to every developer using Claude Code on the project. Personal experimental servers (a developer's local LLM trace viewer) should not be shared. Which scoping pattern is correct?

A) Configure `repo-search` in the project's `.mcp.json` (checked into the repo) and configure personal/experimental servers in the user-scoped `~/.claude.json`.
B) Configure both in `.mcp.json` for consistency.
C) Configure both in `~/.claude.json` so each developer is in control.
D) Configure `repo-search` in a one-off shell script that each developer runs at startup.

**Correct:** A

**Explanation:**
- **A (correct):** Project-level `.mcp.json` is for shared team tooling; user-level `~/.claude.json` is for personal/experimental servers. This is the recommended scoping pattern.
- **B:** Putting personal servers in the shared file forces them on the whole team.
- **C:** Keeps shared tools out of source control and breaks discoverability.
- **D:** A startup script doesn't integrate with Claude Code's MCP discovery model.

---

### Q28 | Scenario 1 | TS 2.4 | difficulty: medium

**Stem:** Your `.mcp.json` references a GitHub API server that requires a token. You want to avoid committing the token to the repository. What is the recommended pattern?

A) Hard-code the token in `.mcp.json` and add the file to `.gitignore`.
B) Use environment variable expansion in `.mcp.json` (e.g., `${GITHUB_TOKEN}`), keep the file committed, and have each developer/CI environment set the variable locally.
C) Store the token in a `secrets.json` file alongside `.mcp.json`.
D) Encode the token in base64 in `.mcp.json` so it isn't readable at a glance.

**Correct:** B

**Explanation:**
- **B (correct):** Environment variable expansion (`${GITHUB_TOKEN}`) in `.mcp.json` is the documented pattern for keeping secrets out of source control while still sharing the configuration.
- **A:** Putting `.mcp.json` itself in `.gitignore` removes the benefit of shared team configuration.
- **C:** A `secrets.json` file is still committed by default and introduces a parallel config that the MCP runtime doesn't read.
- **D:** Base64 is encoding, not encryption — anyone with repo access can decode it.

---

### Q29 | Scenario 4 | TS 2.4 | difficulty: medium

**Stem:** An engineer adds a new MCP tool, `code_intel.search`, that has richer semantics than the built-in `Grep` (it understands the codebase's symbol graph). After deployment Claude still prefers `Grep` for most code searches. Which intervention is most likely to flip this behavior?

A) Rewrite the MCP tool's description to spell out its capabilities in detail — symbol-graph awareness, the kinds of queries it supports best, outputs it returns, and when it should be preferred over `Grep`.
B) Remove the built-in `Grep` from the agent's tool set.
C) Force `tool_choice` to `code_intel.search` on every turn.
D) Rename the MCP tool to `Grep2`.

**Correct:** A

**Explanation:**
- **A (correct):** When agents prefer built-ins over more capable MCP tools, the documented fix is to enhance the MCP tool's description so the model can recognize when to prefer it.
- **B:** Removing `Grep` may hurt useful cases (plain content search) and isn't the targeted fix.
- **C:** Forcing the tool on every turn ignores cases where `Grep` is the right choice.
- **D:** Naming tricks don't substitute for description quality.

---

### Q30 | Scenario 3 | TS 2.4 | difficulty: medium

**Stem:** Your research system frequently spends turns calling tools like `list_files`, `read_file`, and `head_table` just to discover what data is available before doing real work. Which MCP feature is best suited to reduce these exploratory calls?

A) MCP resources — exposing a catalog of available content (issue summaries, doc hierarchies, database schemas) to give the agent visibility into available data without requiring exploratory tool calls.
B) MCP prompts — adding system prompt fragments to remind the agent to plan.
C) More aggressive caching inside individual tools.
D) Disabling exploratory tools entirely so the agent can't waste turns.

**Correct:** A

**Explanation:**
- **A (correct):** MCP resources are the mechanism for exposing content catalogs to the agent up-front, which is the documented way to reduce exploratory tool calls.
- **B:** Prompts don't substitute for actual catalogs of available content.
- **C:** Caching helps repeat calls but doesn't surface availability up-front.
- **D:** Disabling discovery makes the agent more likely to flounder.

---

### Q31 | Scenario 1 | TS 2.4 | difficulty: medium

**Stem:** Your team is integrating Jira into the support agent. A junior engineer proposes writing a custom MCP server from scratch to wrap Jira's REST API. What is the better default choice, and why?

A) Always write a custom MCP server because it gives the most control.
B) Prefer an existing, well-maintained community MCP server for Jira when one exists; reserve custom servers for team-specific workflows that aren't covered by available servers.
C) Skip MCP entirely and have the agent paste Jira links to the user.
D) Build a custom server, then later replace it with the community one.

**Correct:** B

**Explanation:**
- **B (correct):** Choosing existing community MCP servers for standard integrations (like Jira) saves implementation cost and ongoing maintenance; custom servers are for team-specific needs.
- **A:** "Always custom" ignores available, vetted tooling.
- **C:** Doesn't solve the integration problem.
- **D:** Building then discarding is wasteful.

---

### Q32 | Scenario 4 | TS 2.4 | difficulty: easy

**Stem:** When does Claude Code discover the tools exposed by a configured MCP server?

A) Only when the agent invokes a previous tool from the same server.
B) Tools from all configured MCP servers are discovered at connection/startup time and are then available simultaneously to the agent for the rest of the session.
C) On demand each turn, by re-handshaking with every server.
D) Only after the user runs an explicit `/refresh-tools` command.

**Correct:** B

**Explanation:**
- **B (correct):** All configured MCP servers' tools are discovered at connection time and remain available simultaneously.
- **A:** No such "first-use unlock" model exists.
- **C:** Per-turn rediscovery is not how MCP works.
- **D:** No such required command exists.

---

### Q33 | Scenario 3 | TS 2.4 | difficulty: hard

**Stem:** A new MCP server is added to your research system. It exposes both tools and resources. The team wants the agent to be able to: (a) see a catalog of available documents up-front, (b) call a `fetch_document` tool when it wants the full content. Which division of responsibilities is correct?

A) Put both the catalog and the document content into a single mega-tool `get_everything`.
B) Expose the catalog (document IDs, titles, summaries) as MCP resources and keep `fetch_document` as a tool that takes a document ID; the agent uses the resource catalog to choose IDs without making exploratory tool calls.
C) Expose the document content as MCP resources and call the catalog through a tool.
D) Skip resources entirely and instruct the agent to use `fetch_document` exhaustively to enumerate the catalog.

**Correct:** B

**Explanation:**
- **B (correct):** Resources are for catalogs/content visibility; tools are for action. Exposing the catalog as resources reduces exploratory tool calls and gives the agent up-front visibility into what's available.
- **A:** Conflates two distinct responsibilities into one opaque tool.
- **C:** Inverts the resource/tool intent and reintroduces exploration overhead.
- **D:** Exhaustive enumeration is exactly the pattern resources are designed to eliminate.

---

### Q34 | Scenario 4 | TS 2.4 | difficulty: hard

**Stem:** A developer commits `.mcp.json` containing `"GITHUB_TOKEN": "${GITHUB_TOKEN}"`, but on CI the agent fails to authenticate. Investigation shows the CI environment doesn't export `GITHUB_TOKEN`. Which fix best preserves the security and team-sharing properties?

A) Hard-code the actual token into `.mcp.json` so CI works.
B) Add `GITHUB_TOKEN` as a masked CI secret and expose it as an environment variable to the Claude Code job, leaving the `${GITHUB_TOKEN}` expansion in `.mcp.json` untouched.
C) Remove `.mcp.json` from version control.
D) Move the GitHub MCP server to `~/.claude.json` on every CI runner.

**Correct:** B

**Explanation:**
- **B (correct):** Environment variable expansion is the correct pattern; the fix is to make sure the CI environment provides the variable via its secrets mechanism.
- **A:** Commits a secret — exactly what the pattern is designed to avoid.
- **C:** Removes team-shared tooling configuration.
- **D:** User-level configuration on ephemeral CI runners defeats both shared and personal scoping.

---

### Q35 | Scenario 4 | TS 2.5 | difficulty: easy

**Stem:** A developer asks Claude to "find every caller of `enqueueJob` across the repo." Which built-in tool is the most appropriate first step?

A) `Glob` — to enumerate every `.ts` file and read them all.
B) `Grep` — to search file contents for the string `enqueueJob` across the codebase.
C) `Read` on the first file in the project and follow links manually.
D) `Bash` to execute the function and inspect the runtime stack.

**Correct:** B

**Explanation:**
- **B (correct):** `Grep` searches file contents for patterns — exactly the operation needed to find callers of a function across a codebase.
- **A:** `Glob` matches paths by pattern, not content.
- **C:** Manual reading is impractical and inefficient.
- **D:** Runtime execution isn't a code-search technique and won't enumerate static callers.

---

### Q36 | Scenario 4 | TS 2.5 | difficulty: easy

**Stem:** Claude needs to find every Playwright test file in a large monorepo. Which built-in tool best matches the task?

A) `Read` on each top-level directory.
B) `Glob` with a pattern like `**/*.test.tsx` (or whatever the project's test-file convention is) to enumerate matching paths.
C) `Grep` for the literal string `playwright`.
D) `Bash` to print a directory tree.

**Correct:** B

**Explanation:**
- **B (correct):** `Glob` is designed for file-path pattern matching — the canonical use case for finding files by name/extension.
- **A:** Inefficient and won't directly produce a path list.
- **C:** Grep searches contents and will miss files that don't mention "playwright" explicitly.
- **D:** A tree dump still needs filtering and isn't the targeted tool.

---

### Q37 | Scenario 4 | TS 2.5 | difficulty: medium

**Stem:** Claude tried to make a small change with `Edit`, but the call failed because the anchor text it chose (`return result`) appears in multiple places in the file. What is the recommended fallback?

A) Try `Edit` again with the same anchor in case it was a transient error.
B) Use `Read` to load the full file, construct the new contents, and then `Write` to replace the file — using `Read` + `Write` as a reliable fallback when `Edit` cannot find unique anchor text.
C) Use `Bash` to invoke `sed -i` instead.
D) Truncate the file with `Write` and reconstruct it from memory.

**Correct:** B

**Explanation:**
- **B (correct):** The documented fallback for non-unique `Edit` anchors is `Read` + `Write`: load the file, modify in memory, write the full new contents.
- **A:** The anchor is structurally non-unique; retrying does not change that.
- **C:** Shelling out to `sed` is unreliable for structured edits and bypasses the agent's safer file abstractions.
- **D:** Writing from memory without `Read` risks data loss — you may not have the full file content.

---

### Q38 | Scenario 4 | TS 2.5 | difficulty: medium

**Stem:** An engineer asks Claude to "understand how the new billing module works." Which approach best applies incremental codebase exploration?

A) Read every file in the billing module from top to bottom and summarize.
B) Start with `Grep` (or `Glob`) to find likely entry points (e.g., exported functions, route handlers, `index.ts`), then `Read` those, follow imports, and trace flows on demand — building understanding incrementally rather than reading everything upfront.
C) Run the module in `Bash` and observe stdout.
D) Ask the engineer to summarize the module first, then read only the parts they mention.

**Correct:** B

**Explanation:**
- **B (correct):** Incremental exploration — Grep/Glob for entry points, Read to follow imports, trace flows on demand — is the recommended pattern for understanding unfamiliar code.
- **A:** Reading everything wastes context and dilutes attention.
- **C:** Runtime behavior doesn't reveal structure or design intent.
- **D:** Pushes the work back on the engineer and may miss critical pieces.

---

### Q39 | Scenario 4 | TS 2.5 | difficulty: medium

**Stem:** A developer asks Claude to "trace where `formatAmount` is used." The function is re-exported through several wrapper modules under different names (e.g., `formatMoney`, `displayPrice`). Which strategy best matches the prescribed technique?

A) Grep for `formatAmount` once and stop there.
B) First identify all exported names that ultimately refer to `formatAmount` by reading the wrapper modules' index/exports, then `Grep` for each of those names across the codebase to find all real call sites.
C) Read every test file and infer usage from the tests.
D) Replace `formatAmount` with `throw new Error()` and run the test suite to find call sites.

**Correct:** B

**Explanation:**
- **B (correct):** Tracing function usage across wrappers requires first enumerating exported names, then searching for each name — exactly the documented technique.
- **A:** Stops too early and misses re-exports under other names.
- **C:** Test files are a partial signal and likely incomplete.
- **D:** Destructive and slow; not an exploration technique.

---

### Q40 | Scenario 2 | TS 2.5 | difficulty: easy

**Stem:** Which built-in tool is best for a small, targeted change such as replacing a single function body where the surrounding code provides unique context?

A) `Edit`, using a uniquely identifying anchor string around the change.
B) `Write` — always replace the whole file to be safe.
C) `Bash` with `sed`.
D) `Grep` followed by manually constructed `Write`.

**Correct:** A

**Explanation:**
- **A (correct):** `Edit` is the right tool for targeted modifications when there's unique anchor text identifying the change location.
- **B:** Whole-file rewrites are riskier and slower for small targeted changes.
- **C:** `sed` is fragile for structured edits and bypasses safer abstractions.
- **D:** Unnecessary when `Edit` will succeed.

---

### Q41 | Scenario 4 | TS 2.5 | difficulty: hard

**Stem:** Claude needs to add a new field to a TypeScript interface that appears in ~40 files. Some files use the interface as `User`, others import it as a renamed alias `Account`. Which sequence is the best fit for the built-in tools?

A) Use `Glob` to find every `.ts` file, then run `Edit` blindly on each.
B) Use `Grep` to find the interface declaration and all import sites; build the set of local aliases per file; then for each file, use `Read` to load it, modify in memory accordingly, and `Write` to persist — preferring `Edit` only where there's a clearly unique anchor.
C) Use `Bash` to run a global `sed` replacement and trust it.
D) Use `Write` on the interface declaration file only and assume TypeScript will propagate.

**Correct:** B

**Explanation:**
- **B (correct):** This sequence — Grep to enumerate sites, Read to inspect local aliases, Edit or Read+Write per file based on anchor uniqueness — is the prescribed pattern for codebase-wide refactoring with built-in tools.
- **A:** Blind edits across 40 files will fail when anchors aren't unique.
- **C:** Global `sed` ignores file-local context (aliasing, renames) and risks silent breakage.
- **D:** Adding a field to the declaration alone doesn't update call sites that destructure or check the type.

---

### Q42 | Scenario 4 | TS 2.5 | difficulty: hard

**Stem:** Claude is asked to locate every place in a large codebase where a specific error message is thrown so a logging tag can be added. The error string is `"User session has expired"`. Which workflow is most efficient and reliable?

A) Use `Glob` to list `.ts` files, then ask the user which to inspect.
B) Use `Grep` to search file contents for the literal error string across the codebase, then `Read` each matching file at the matched location to confirm context, and apply `Edit` with a unique surrounding anchor (or fall back to `Read` + `Write` when anchors aren't unique).
C) Use `Bash` to run the application and trigger the error in production to find the file from the stack trace.
D) Use `Glob` for files named `*errors*.ts` and assume the message lives there.

**Correct:** B

**Explanation:**
- **B (correct):** Grep finds content matches; Read confirms each site's context; Edit applies a targeted change (or Read+Write when anchors aren't unique). This is the documented chain for content-anchored refactors.
- **A:** Defers the work back to the user and doesn't use the right tool.
- **C:** Running production to trigger an error is unsafe and finds at most one stack at a time.
- **D:** Assumes a naming convention that may not hold.


## Domain 3: Claude Code Configuration & Workflows


42 scenario-based multiple-choice questions covering Task Statements 3.1–3.6.

---

### Q1 | Scenario 2 | TS 3.1 | difficulty: easy

**Stem:** A new engineer joins your backend team and starts using Claude Code in the repository. They report that Claude keeps suggesting `console.log` for debugging even though the team strictly uses the structured `logger` module. You inspect your own setup and find the relevant rule in `~/.claude/CLAUDE.md` and your sessions follow it correctly. Where should the rule actually live so the new teammate inherits it automatically?

A) Add the rule to the project's root `CLAUDE.md` (or `.claude/CLAUDE.md`) and commit it to version control.
B) Have the new teammate copy your `~/.claude/CLAUDE.md` into their own home directory.
C) Move the rule into your personal `~/.claude/commands/` directory as a slash command.
D) Add the rule to a `.gitignore`-tracked file so it's available locally per developer.

**Correct:** A

**Explanation:**
- **A (correct):** Project-level CLAUDE.md (committed to the repo) is the mechanism for sharing instructions with the whole team. User-level `~/.claude/CLAUDE.md` applies only to the local user.
- **B:** Manual copy-paste is fragile, won't update with the project, and defeats version control as the source of truth.
- **C:** Slash commands are on-demand invocations, not always-on standards; this rule needs to apply automatically.
- **D:** `.gitignore` excludes files from version control, which is the opposite of what is needed to share team conventions.

---

### Q2 | Scenario 2 | TS 3.1 | difficulty: medium

**Stem:** Your monorepo contains five packages: `web/`, `api/`, `mobile/`, `infra/`, and `docs/`. Each package has its own coding conventions (React patterns, FastAPI patterns, React Native, Terraform, Markdown style). The root `CLAUDE.md` has grown to 900 lines as engineers piled in every package's rules. Engineers complain that Claude loads tons of irrelevant guidance when working in any one package. What is the most effective restructuring?

A) Place a focused `CLAUDE.md` inside each package's directory containing only that package's conventions, leaving the root file with cross-cutting rules.
B) Keep one root `CLAUDE.md` and add a prologue instructing Claude to "only read the section relevant to your current task."
C) Move all package rules into `~/.claude/CLAUDE.md` so they're available globally without bloating the project file.
D) Delete the root `CLAUDE.md` and ask each engineer to paste relevant rules into the chat at the start of every session.

**Correct:** A

**Explanation:**
- **A (correct):** Directory-level CLAUDE.md files load when working in that subdirectory, scoping context naturally. The root file then carries only repo-wide conventions.
- **B:** Prose self-pruning is unreliable; Claude still loads the whole file into context, wasting tokens and risking misapplication.
- **C:** User-level config isn't shared with teammates and shouldn't carry project conventions.
- **D:** Manual pasting is error-prone and undermines having version-controlled conventions at all.

---

### Q3 | Scenario 4 | TS 3.1 | difficulty: medium

**Stem:** Your team's `CLAUDE.md` is 1,200 lines covering testing patterns, API conventions, deployment runbooks, observability standards, and security review checklists. Sessions are slow to start and engineers report Claude sometimes applies the wrong rules in the wrong contexts. Which refactor best preserves all the guidance while making it more maintainable?

A) Split into topic-specific files under `.claude/rules/` (e.g., `testing.md`, `api-conventions.md`, `deployment.md`) and reference them from a slim CLAUDE.md.
B) Move every section to `~/.claude/CLAUDE.md` so it's not loaded from the repo.
C) Compress the file by removing examples and shortening sentences so it fits under 400 lines.
D) Concatenate everything into a single long string in the system prompt at runtime.

**Correct:** A

**Explanation:**
- **A (correct):** `.claude/rules/` with topic-specific files keeps content modular, easier to maintain, and avoids a monolithic file. CLAUDE.md can pull them in via `@import` when needed.
- **B:** User-level config isn't shared with teammates and doesn't reduce session bloat for them.
- **C:** Removing examples often degrades quality; the underlying organizational problem remains.
- **D:** This bypasses the configuration system entirely and is not how Claude Code is designed to be used.

---

### Q4 | Scenario 2 | TS 3.1 | difficulty: medium

**Stem:** Engineers across multiple sessions are getting inconsistent behavior: some sessions follow the new "always include type hints" rule and others don't. You suspect different memory files are being loaded in different sessions. Which built-in mechanism most directly diagnoses what's being loaded?

A) Run the `/memory` command in each session to see exactly which memory files are loaded.
B) Delete the user-level `~/.claude/CLAUDE.md` and rely solely on the project file.
C) Reinstall Claude Code on every developer's machine.
D) Manually diff every engineer's working directory CLAUDE.md against the repo.

**Correct:** A

**Explanation:**
- **A (correct):** `/memory` is purpose-built to surface which memory files are currently loaded, making it the right diagnostic tool for this exact problem.
- **B:** A blind delete may remove legitimate personal config without proving the diagnosis.
- **C:** Reinstalling doesn't reveal which files are loaded, and is heavy-handed.
- **D:** Manual diffing is slow and still doesn't show which files Claude actually loaded for a given session.

---

### Q5 | Scenario 2 | TS 3.1 | difficulty: medium

**Stem:** Your repository has several packages, each with distinct standards files: `standards/python.md`, `standards/typescript.md`, `standards/terraform.md`. You want each package's `CLAUDE.md` to pull in only the standards that are relevant. Which approach best matches the design intent of CLAUDE.md modularity?

A) Use `@import` references inside each package's CLAUDE.md to selectively include the standards files relevant to that package.
B) Copy the contents of relevant standards files directly into each package's CLAUDE.md.
C) Symlink every standards file into every package directory.
D) Place every standards file in `~/.claude/CLAUDE.md` so they are universally available.

**Correct:** A

**Explanation:**
- **A (correct):** `@import` is the supported mechanism to compose CLAUDE.md from external files, keeping each package's config focused while reusing a single source of truth.
- **B:** Duplicating content causes drift and defeats the purpose of having shared standards files.
- **C:** Symlinking dumps full files into every context, not selective inclusion.
- **D:** User-level config is per-developer and not shared; standards belong in the repo.

---

### Q6 | Scenario 4 | TS 3.1 | difficulty: hard

**Stem:** You see this layout in your repo: a root `CLAUDE.md`, a `services/payments/CLAUDE.md`, and a developer-only `~/.claude/CLAUDE.md` that contains "Always run `make test` before suggesting code changes." During a code review, the new engineer pushes a commit that breaks tests, and Claude never ran them. The senior engineer's local Claude always runs them. What is the most likely root cause and remedy?

A) The "always run tests" rule lives only in user-level config, so it doesn't apply to teammates; move it into the project's root CLAUDE.md so all engineers inherit it.
B) The directory-level CLAUDE.md for `services/payments` overrides the user-level rule; remove the directory file to restore the global rule.
C) Claude Code does not support running tests; switch to a CI-only review model.
D) The root CLAUDE.md needs to be renamed `.claude/CLAUDE.md` to take effect.

**Correct:** A

**Explanation:**
- **A (correct):** User-level rules apply only to that user. To make a rule team-wide, it must live in a project-level configuration file committed to the repo.
- **B:** Directory-level files scope further but do not silently nullify user-level rules for other users — the new engineer simply never had that rule.
- **C:** Claude Code can run shell commands; the issue is configuration scope, not capability.
- **D:** Either filename works for project-level config; this is not the cause.

---

### Q7 | Scenario 4 | TS 3.1 | difficulty: hard

**Stem:** Your team adopts a convention: each microservice owns its CLAUDE.md and pulls shared standards via `@import ../../standards/<file>.md`. A new service is created by copying an existing service, including its `CLAUDE.md`. Engineers working in the new service report Claude applies REST conventions even though this service is gRPC. What is the most likely cause?

A) The copied CLAUDE.md still imports the REST standards file; update the `@import` references to point at the gRPC standards file.
B) `@import` only resolves files inside `.claude/`; the path is broken so no rules are loaded at all.
C) Directory-level CLAUDE.md files are not honored beneath the repo root; move the rules to the root.
D) `@import` has been deprecated; inline all content directly.

**Correct:** A

**Explanation:**
- **A (correct):** The most likely cause is the inherited import still references REST conventions. Each service's CLAUDE.md should import only the standards relevant to its domain.
- **B:** `@import` supports paths to external files, including relative paths outside `.claude/`.
- **C:** Directory-level CLAUDE.md files are supported and load when working in that directory.
- **D:** `@import` is a supported mechanism for modular CLAUDE.md composition.

---

### Q8 | Scenario 2 | TS 3.2 | difficulty: easy

**Stem:** You build a `/ship` slash command that runs your team's release workflow (changelog, version bump, tag). You want every engineer to pick it up automatically when they pull main. Where should the command file live?

A) `.claude/commands/ship.md` (project-scoped, committed to version control).
B) `~/.claude/commands/ship.md` (user-scoped).
C) `~/.bashrc` as a shell alias.
D) Inline inside CLAUDE.md as a code block.

**Correct:** A

**Explanation:**
- **A (correct):** Project-scoped commands live in `.claude/commands/` and are shared via version control, exactly the case for a team-wide release workflow.
- **B:** User-scoped commands live only on one machine and aren't shared with the team.
- **C:** Shell aliases don't integrate with Claude Code's slash command system.
- **D:** CLAUDE.md is for always-on instructions, not on-demand slash commands.

---

### Q9 | Scenario 2 | TS 3.2 | difficulty: medium

**Stem:** You author a `codebase-explorer` skill that traces dependency graphs and produces 4,000+ lines of intermediate output. When developers invoke it during a normal coding session, the main conversation gets flooded with this output and Claude later loses track of the original task. Which frontmatter setting best addresses this?

A) Set `context: fork` on the skill so it runs in an isolated sub-agent context and only its summary returns to the main session.
B) Set `allowed-tools` to only `Read` so the skill produces less output.
C) Add `argument-hint` so the user is forced to narrow the scope.
D) Move the skill to `~/.claude/skills/` to avoid affecting teammates.

**Correct:** A

**Explanation:**
- **A (correct):** `context: fork` runs the skill in a sub-agent so verbose internal output stays out of the main conversation; only the summary is returned. This is the canonical use case.
- **B:** Restricting tools doesn't reduce the volume of analysis output and may break the skill.
- **C:** Argument prompting doesn't address conversational pollution.
- **D:** Personal placement doesn't change the context pollution problem; it just hides the skill from the team.

---

### Q10 | Scenario 4 | TS 3.2 | difficulty: medium

**Stem:** A teammate ships a `format-code` skill that's helpful but invokes shell commands you don't trust on your machine. You want a personal variant of the skill that uses a different formatter and doesn't run shell commands — without affecting teammates who depend on the original. What's the recommended approach?

A) Create a personal skill in `~/.claude/skills/` with a different name and a more restrictive `allowed-tools` frontmatter; leave the project skill untouched.
B) Edit the project skill in `.claude/skills/` to remove the shell tool, then commit your change.
C) Comment out the project skill in `CLAUDE.md` so it stops loading.
D) Delete the `.claude/skills/` directory locally to disable it.

**Correct:** A

**Explanation:**
- **A (correct):** Personal skill variants in `~/.claude/skills/` with a distinct name let you customize behavior locally without affecting teammates' workflows.
- **B:** Committing a personal preference forces it on the whole team and removes a tool others rely on.
- **C:** CLAUDE.md doesn't gate skill loading; you can't disable a skill that way.
- **D:** Local deletion is fragile (it returns on every pull) and doesn't provide your alternative behavior.

---

### Q11 | Scenario 2 | TS 3.2 | difficulty: medium

**Stem:** Your `/generate-component` slash command works well when developers type `/generate-component Button primary`, but when they just type `/generate-component`, Claude either hallucinates inputs or asks long clarifying questions. Which frontmatter improvement most directly fixes this?

A) Add an `argument-hint` so the developer is prompted for required parameters (e.g., component name, variant) when they invoke without arguments.
B) Add `context: fork` so the prompts don't pollute the main conversation.
C) Add `allowed-tools` to restrict the command to `Write` only.
D) Convert the command to a CLAUDE.md rule that always loads.

**Correct:** A

**Explanation:**
- **A (correct):** `argument-hint` is designed precisely to prompt for required parameters at invocation when they are missing.
- **B:** `context: fork` controls context isolation, not input collection.
- **C:** Restricting tools doesn't address missing inputs.
- **D:** Always-on rules can't replace an on-demand command that takes arguments.

---

### Q12 | Scenario 4 | TS 3.2 | difficulty: medium

**Stem:** You're authoring a skill that should be able to write configuration files but must never modify source code or run shell commands. Which frontmatter setting most directly enforces this?

A) Set `allowed-tools` in the skill's SKILL.md to a narrow list (e.g., `Write` for config paths only), excluding `Bash` and broader file editing tools.
B) Add a sentence in the skill body telling Claude not to use Bash.
C) Set `context: fork` to isolate the skill's effects.
D) Place the skill in `~/.claude/skills/` to limit blast radius.

**Correct:** A

**Explanation:**
- **A (correct):** `allowed-tools` in skill frontmatter restricts tool access during skill execution. This is the enforcement mechanism, not a request in prose.
- **B:** Prose instructions are advisory; tool restriction must be at the configuration layer.
- **C:** `context: fork` controls conversation isolation, not tool permissions.
- **D:** Location affects who runs the skill, not what tools it can use.

---

### Q13 | Scenario 2 | TS 3.2 | difficulty: hard

**Stem:** Your team has two patterns that overlap: a "test conventions" rule applied universally (always-on) and a "run-coverage-report" task workflow (on-demand). A new engineer proposes putting both into `.claude/skills/`. What's the most accurate guidance?

A) Keep "test conventions" in CLAUDE.md as always-loaded universal standards; keep "run-coverage-report" as a skill for on-demand invocation — they serve different roles.
B) Move both into `.claude/skills/` because skills are simply newer than CLAUDE.md.
C) Move both into CLAUDE.md because CLAUDE.md supersedes skills.
D) Move both into `~/.claude/commands/` so each engineer can opt in.

**Correct:** A

**Explanation:**
- **A (correct):** CLAUDE.md is for always-loaded universal standards; skills are for on-demand task-specific workflows. The right tool depends on whether the content should always apply or be invoked deliberately.
- **B:** Skills aren't a strict replacement for always-on standards.
- **C:** CLAUDE.md is not designed to host invocation workflows.
- **D:** User-scoped commands prevent team-wide consistency for standards.

---

### Q14 | Scenario 4 | TS 3.2 | difficulty: hard

**Stem:** You build a "brainstorm-alternatives" skill that intentionally generates many speculative options before narrowing. In testing, the speculative output sticks in the conversation and biases subsequent unrelated questions. What is the cleanest configuration fix?

A) Add `context: fork` to the skill so brainstorming happens in a sub-agent context and only the chosen alternatives return to the main session.
B) Lower the model's temperature in CLAUDE.md so it produces fewer alternatives.
C) Add a system rule asking Claude to "forget" the brainstormed alternatives after each invocation.
D) Have developers manually restart their session after running the skill.

**Correct:** A

**Explanation:**
- **A (correct):** Exploratory or brainstorming skills are the canonical use case for `context: fork` — keep the noisy intermediate work out of the main conversation.
- **B:** Temperature is not a CLAUDE.md control and doesn't solve context pollution.
- **C:** Asking the model to "forget" doesn't actually purge context.
- **D:** Manual session restarts are disruptive and bypass the configuration system designed for this.

---

### Q15 | Scenario 4 | TS 3.3 | difficulty: easy

**Stem:** Your codebase has test files scattered throughout the repo: `src/**/*.test.tsx`, `apps/*/tests/*.spec.ts`, etc. You want a "testing conventions" rule that loads only when editing test files, regardless of where they live. Which approach matches the design?

A) Create a `.claude/rules/testing.md` with YAML frontmatter `paths: ["**/*.test.tsx", "**/*.spec.ts"]` so the rule activates only on matching files.
B) Put the testing conventions in the root CLAUDE.md so they always load.
C) Create a `CLAUDE.md` inside every directory that happens to contain test files.
D) Add the testing conventions as comments in each test file.

**Correct:** A

**Explanation:**
- **A (correct):** Path-scoped rules with glob patterns load only when editing matching files, which is ideal for cross-cutting conventions like test files spread throughout the codebase.
- **B:** Always-on loading wastes context when editing non-test files.
- **C:** Maintaining many directory CLAUDE.md files is brittle and misses files that move.
- **D:** Comments aren't a configuration mechanism and won't be applied consistently.

---

### Q16 | Scenario 4 | TS 3.3 | difficulty: medium

**Stem:** Your platform team wants a Terraform-specific rule set to apply when engineers edit any file under `terraform/`. They want minimal context bloat when editing other code. Which configuration is best?

A) Add a `.claude/rules/terraform.md` with frontmatter `paths: ["terraform/**/*"]` so the rule activates only when editing Terraform files.
B) Append the Terraform rules to the root CLAUDE.md.
C) Place a CLAUDE.md inside `terraform/` and another inside every Terraform module subdirectory.
D) Ask engineers to copy/paste the Terraform rules at the start of every Terraform-editing session.

**Correct:** A

**Explanation:**
- **A (correct):** YAML `paths` glob scoping in `.claude/rules/` loads the rule only for matching files, minimizing context usage elsewhere.
- **B:** Loading Terraform rules for every session wastes context when working in unrelated areas.
- **C:** Possible but more cumbersome and doesn't scale as cleanly as a single path-scoped rule.
- **D:** Manual pasting is error-prone and undermines configuration.

---

### Q17 | Scenario 2 | TS 3.3 | difficulty: medium

**Stem:** A team has frontend tests in `apps/web/tests/`, backend tests in `services/*/tests/`, and integration tests in `e2e/`. They want one rule file to apply across all of these. Which choice best fits the design of path-scoped rules?

A) Create one `.claude/rules/test-conventions.md` with `paths: ["apps/web/tests/**", "services/*/tests/**", "e2e/**"]` (or a broader `**/tests/**` pattern).
B) Create three separate directory-level CLAUDE.md files, one per location, with identical content.
C) Append the conventions to each test file as a doc comment.
D) Add an always-on rule in the root CLAUDE.md.

**Correct:** A

**Explanation:**
- **A (correct):** Glob-pattern rules in `.claude/rules/` are designed precisely for conventions that span multiple directories without duplication.
- **B:** Duplicating identical rule content across directories invites drift.
- **C:** Doc comments aren't reliably enforced and aren't a configuration mechanism.
- **D:** Always-on loading wastes context for non-test work.

---

### Q18 | Scenario 4 | TS 3.3 | difficulty: medium

**Stem:** You add a path-scoped rule with frontmatter `paths: ["src/legacy/**"]` describing brittle patterns to avoid. An engineer reports that when they edit `src/modern/auth.ts`, Claude still references the legacy rules. They share their session and you see the rule indeed showed up. What's the most likely cause?

A) Another always-on file (root CLAUDE.md or a non-path-scoped rule) references the legacy patterns and is loading them unconditionally.
B) Path scoping requires absolute paths and the file is relative.
C) `.claude/rules/` files cannot use frontmatter — that's an Anthropic Code feature, not Claude Code.
D) Path-scoped rules always load no matter what; the `paths` field is informational.

**Correct:** A

**Explanation:**
- **A (correct):** Path-scoped rules only load on matching files; if rules are appearing for non-matching paths, another always-loaded file (CLAUDE.md or non-scoped rule) is likely the source.
- **B:** Glob patterns are repo-relative; absolute paths aren't required.
- **C:** YAML frontmatter is supported in `.claude/rules/` files.
- **D:** Path scoping is functional, not informational; rules with `paths` load conditionally.

---

### Q19 | Scenario 2 | TS 3.3 | difficulty: medium

**Stem:** Your team uses a subdirectory `services/payments/` and a directory-level `services/payments/CLAUDE.md` for that service. Now they also want a rule that activates whenever any test file is edited (anywhere in the repo). Which configuration covers both intents correctly?

A) Keep the `services/payments/CLAUDE.md` for service-specific conventions, and add a separate `.claude/rules/test-conventions.md` with `paths: ["**/*.test.*", "**/*.spec.*"]` for cross-cutting test rules.
B) Move all rules into `services/payments/CLAUDE.md`, including the test rules.
C) Put everything in the root CLAUDE.md.
D) Duplicate the test rules into every directory's CLAUDE.md.

**Correct:** A

**Explanation:**
- **A (correct):** Use directory-level CLAUDE.md for directory-scoped conventions and path-scoped rules for conventions that span across directories — different problems, different mechanisms.
- **B:** Service-level CLAUDE.md won't apply when editing test files outside that service.
- **C:** Always-on loading bloats the context for unrelated work.
- **D:** Duplication leads to drift and is exactly what path-scoped rules avoid.

---

### Q20 | Scenario 5 | TS 3.3 | difficulty: hard

**Stem:** A CI-invoked Claude Code review on a PR that modifies only Terraform files keeps applying Python style commentary. You inspect the repo and find `.claude/rules/python-style.md` has no frontmatter at all. What's the most likely cause and fix?

A) Without `paths` frontmatter, the rule is loaded universally; add `paths: ["**/*.py"]` to scope it to Python files so it stops loading on Terraform-only PRs.
B) `.claude/rules/` files always load regardless of frontmatter; delete the file.
C) CI ignores `.claude/rules/` entirely; the bug must be elsewhere.
D) The rule fires because CI passes `--all-rules` by default; remove that flag.

**Correct:** A

**Explanation:**
- **A (correct):** Without `paths` scoping, a rule file is treated as always-on. Adding a `paths` glob limits activation to matching files.
- **B:** Rules with no `paths` field still load — they aren't ignored, they're universal.
- **C:** CI does honor `.claude/rules/`; the misconfiguration is the cause.
- **D:** There's no `--all-rules` flag that drives this behavior; the field-level scoping is what's missing.

---

### Q21 | Scenario 4 | TS 3.3 | difficulty: hard

**Stem:** Your team wants conventions for React components (`*.tsx`, excluding tests) and a separate set for React tests (`*.test.tsx`). Engineers find that when they edit a `Button.test.tsx`, both sets of rules fire and contradict each other. Which configuration adjustment best resolves the conflict?

A) Split into two `.claude/rules/` files: one with `paths: ["**/*.tsx", "!**/*.test.tsx"]` (or equivalent exclusion) for components, and one with `paths: ["**/*.test.tsx"]` for tests.
B) Merge both rule sets into the root CLAUDE.md.
C) Put both rules in one file and let Claude figure out which applies.
D) Move the component rules into `~/.claude/CLAUDE.md` so they don't conflict with the project file.

**Correct:** A

**Explanation:**
- **A (correct):** Glob patterns can exclude test files from the component rule, so each rule activates only for its intended file type and the contradictions disappear.
- **B:** Loading both rules always-on does not resolve the conflict.
- **C:** Mixing contradicting rules in one file is exactly what causes the inconsistency.
- **D:** User-level config isn't shared and doesn't address path-scope conflicts.

---

### Q22 | Scenario 2 | TS 3.4 | difficulty: easy

**Stem:** A developer asks Claude Code to "add a single null-check to the `getUserName` function — it's blowing up when the user has no profile." The stack trace is clear and the file is small. Which mode is most appropriate?

A) Direct execution — the change is small, well-scoped, and the diagnosis is clear.
B) Plan mode, because every code change benefits from a written plan first.
C) Plan mode, then ask Claude to spawn three subagents to debate the fix.
D) Refuse the change and ask the developer to write a design doc.

**Correct:** A

**Explanation:**
- **A (correct):** Direct execution is appropriate for simple, well-scoped changes with a clear diagnosis (a single-file bug fix is the canonical example).
- **B:** Plan mode adds overhead for trivial changes; it's designed for complex multi-file work.
- **C:** Spawning subagents for a one-line fix is needless overhead.
- **D:** Design docs aren't warranted for a single null check.

---

### Q23 | Scenario 2 | TS 3.4 | difficulty: medium

**Stem:** A senior engineer asks Claude Code to "migrate our codebase from the deprecated `auth-v1` library to `auth-v2`. There are about 45 files using the old API, the new API has slightly different return shapes, and we need to choose between a shim layer or a deeper refactor." Which approach best fits this work?

A) Use plan mode to investigate, weigh shim vs. deeper refactor, and produce a written plan before editing any code.
B) Use direct execution and start with the most-used file first.
C) Spawn 45 parallel subagents, one per file, with direct execution.
D) Refuse the task because it touches too many files.

**Correct:** A

**Explanation:**
- **A (correct):** Plan mode is designed for tasks with architectural choices, multi-file scope, and multiple valid approaches — exactly this migration's profile.
- **B:** Direct execution risks committing to an approach prematurely and producing inconsistent files.
- **C:** Parallel uncoordinated edits will produce inconsistent design choices and merge headaches.
- **D:** The task is well within Claude Code's capability — it just needs planning first.

---

### Q24 | Scenario 4 | TS 3.4 | difficulty: medium

**Stem:** During a multi-phase task ("research the codebase, then design, then implement"), the discovery phase produces 50+ file reads and 100+ search results. By the implementation phase, Claude is making mistakes and forgetting earlier decisions. Which technique most directly addresses the symptom?

A) Use the Explore subagent for the discovery phase so its verbose output stays out of the main conversation; only summaries come back.
B) Switch to a model with a larger context window so all the discovery output fits.
C) Tell Claude to "be more careful" in the implementation phase.
D) Restart the session before the implementation phase, starting from scratch.

**Correct:** A

**Explanation:**
- **A (correct):** The Explore subagent isolates verbose discovery output and returns concise summaries, preserving main-conversation context for later phases.
- **B:** Larger windows alleviate hard limits but don't fix attention dilution; the original problem returns at scale.
- **C:** Prompt scolding is not a reliable cure for context exhaustion.
- **D:** Starting from scratch loses all the discovery work and risks repeating it.

---

### Q25 | Scenario 2 | TS 3.4 | difficulty: medium

**Stem:** You're planning a microservice split: extract `billing` out of the monolith, define new service boundaries, choose a message-bus pattern, and migrate database tables. Which combination of modes best matches typical workflow guidance?

A) Use plan mode for investigation and design decisions, then switch to direct execution to implement the agreed-upon plan.
B) Use plan mode for everything end-to-end.
C) Use direct execution from the start to iterate quickly.
D) Use plan mode after each commit to retroactively explain choices.

**Correct:** A

**Explanation:**
- **A (correct):** Plan-then-execute is the recommended pattern: plan mode for architectural investigation, direct execution to carry out the planned approach.
- **B:** Continuing in plan mode after the design is settled wastes effort.
- **C:** Skipping planning on architectural work invites costly rework.
- **D:** Plan mode is a forward-looking tool, not a post-hoc commentary.

---

### Q26 | Scenario 4 | TS 3.4 | difficulty: medium

**Stem:** A developer says, "Add a date-range validation conditional to `parseFilter` — if `startDate > endDate`, throw a `ValidationError`." There is a clear function, a clear contract, and a clear error type. Which mode is most appropriate?

A) Direct execution — the change is small, the contract is clear, and there's only one reasonable approach.
B) Plan mode, because validation logic should always be planned.
C) Spawn an Explore subagent to map out every caller of `parseFilter` before editing.
D) Open a design doc and circulate it for review before changing code.

**Correct:** A

**Explanation:**
- **A (correct):** Direct execution is the right choice when scope is small, the diagnosis is precise, and there's a clear single change to make.
- **B:** Plan mode is for complex, multi-approach, or architectural work, not a one-line conditional.
- **C:** Mapping all callers is overkill for adding a guard inside the function itself.
- **D:** Design review for a single conditional is excessive.

---

### Q27 | Scenario 2 | TS 3.4 | difficulty: hard

**Stem:** Your team is choosing between two integration approaches for a new payments processor: A) a thin synchronous wrapper that requires no new infrastructure, or B) an event-driven queue-based design that requires a new message broker and reshaped DB tables. Both are technically viable. Which Claude Code workflow gives the best decision support before code is changed?

A) Use plan mode to compare both approaches: cost, complexity, blast radius, rollback story; then commit to one and execute.
B) Use direct execution to scaffold both options in parallel and pick whichever compiles first.
C) Use direct execution on approach A; if it doesn't work, revert and try B.
D) Skip planning since plan mode cannot evaluate infrastructure trade-offs.

**Correct:** A

**Explanation:**
- **A (correct):** Plan mode is designed precisely for tasks with multiple valid approaches and architectural implications — laying out trade-offs before committing prevents costly rework.
- **B:** "Whichever compiles first" is not a sound design heuristic and produces throwaway work.
- **C:** Build-and-revert is wasteful when the decision is fundamentally about design fit.
- **D:** Plan mode is well-suited to weighing architectural and infrastructure trade-offs.

---

### Q28 | Scenario 4 | TS 3.4 | difficulty: hard

**Stem:** A complex refactor was scoped in plan mode, producing a clear sequence of file changes. The engineer then asks Claude to "execute the plan." Two hours in, Claude has burned tons of context re-reading and re-exploring files mentioned in the plan, and is now slow and forgetful. What technique would have most helped?

A) During execution, use the Explore subagent for any deep-dive re-reads so that verbose discovery output stays out of the main session.
B) Keep all re-reads in the main conversation so Claude has full visibility.
C) Switch back to plan mode for execution to keep the plan in view.
D) Disable file reading during execution.

**Correct:** A

**Explanation:**
- **A (correct):** The Explore subagent is meant to isolate verbose discovery and return summaries, preserving main-conversation context — useful during implementation as well as initial discovery.
- **B:** Keeping every re-read in main context is exactly what caused the exhaustion.
- **C:** Plan mode is for planning; switching back doesn't address the discovery-output volume.
- **D:** Disabling file reads cripples execution.

---

### Q29 | Scenario 6 | TS 3.5 | difficulty: easy

**Stem:** You ask Claude to "normalize names so they're consistent." Across runs, you get "First Last," "LAST, FIRST," and "First M Last" in different outputs. Which iterative refinement technique most directly fixes this?

A) Provide 2–3 concrete input/output examples showing exactly how a name should be transformed.
B) Add more synonyms for "normalize" in the prompt.
C) Increase the temperature so Claude is more creative.
D) Ask Claude to invent its own examples first.

**Correct:** A

**Explanation:**
- **A (correct):** Concrete input/output examples are the most effective way to disambiguate transformations when prose is interpreted inconsistently.
- **B:** Adding synonyms doesn't resolve the underlying ambiguity of "consistent."
- **C:** Higher temperature increases variability, not consistency.
- **D:** Self-invented examples may anchor on the wrong interpretation; you want to anchor with yours.

---

### Q30 | Scenario 2 | TS 3.5 | difficulty: easy

**Stem:** A developer is unfamiliar with cache invalidation patterns and wants Claude to implement a cache for an API. Which iterative pattern best surfaces design considerations before code is written?

A) Use the interview pattern — have Claude ask the developer questions (TTL strategy, eviction, stale-while-revalidate) before implementing.
B) Have Claude pick defaults silently and refactor later if needed.
C) Ask Claude to produce three independent cache implementations and choose one at the end.
D) Skip discussion and have Claude implement immediately based on the function signature.

**Correct:** A

**Explanation:**
- **A (correct):** The interview pattern surfaces considerations the developer may not have anticipated, which is especially valuable in unfamiliar domains like cache invalidation.
- **B:** Silent defaults often miss the actual requirements and lead to costly rework.
- **C:** Producing three full implementations is wasteful when the interview pattern can converge faster.
- **D:** Skipping the conversation increases the chance of building the wrong thing.

---

### Q31 | Scenario 6 | TS 3.5 | difficulty: medium

**Stem:** You're writing a parser and want Claude to handle edge cases reliably. Which approach matches the "test-driven iteration" technique?

A) Write a test suite covering expected behavior, edge cases, and performance constraints first; then share failures with Claude and iterate until they pass.
B) Ask Claude to write the parser first, then write tests after to verify it.
C) Ask Claude to invent both the tests and the parser at once and not look at the results.
D) Skip tests and rely on Claude's training data to handle edge cases.

**Correct:** A

**Explanation:**
- **A (correct):** Test-driven iteration involves writing tests first and using failures as concrete feedback to guide Claude's progressive improvement.
- **B:** Writing tests after the fact misses the iteration loop and can lead to tests that match buggy behavior.
- **C:** Inventing both at once removes the human-in-the-loop verification.
- **D:** Hoping for correct edge-case handling without tests is exactly what causes the inconsistency.

---

### Q32 | Scenario 6 | TS 3.5 | difficulty: medium

**Stem:** A migration script fails on `null` values in three independent columns. You can fix all three at once or one at a time. Each null case is independent (fixing one doesn't change another), but you're worried about context bloat. What is the best iteration strategy?

A) Fix the three independent issues sequentially in separate messages, since the fixes don't interact.
B) Always send all issues in one big message regardless of independence.
C) Fix one and hope the model generalizes the rest without being told.
D) Ask Claude to rewrite the migration from scratch.

**Correct:** A

**Explanation:**
- **A (correct):** When fixes are independent, sequential iteration is cleaner. Bundling everything into one message is reserved for interacting problems.
- **B:** Bundling independent issues unnecessarily increases the chance of cross-interference.
- **C:** Hoping for generalization is unreliable for distinct edge cases.
- **D:** A full rewrite is excessive when the issues are localized.

---

### Q33 | Scenario 2 | TS 3.5 | difficulty: medium

**Stem:** You're fixing a feature where three problems interact: a state-machine bug, an event-naming inconsistency, and a missing reducer case. Each fix changes how the others should behave. What's the best iteration approach?

A) Provide all three issues in a single detailed message so Claude can plan an integrated fix accounting for interactions.
B) Fix them strictly one at a time; the others can wait.
C) Fix them in three parallel sessions and merge later.
D) Ask Claude to ignore the interactions and fix each in isolation.

**Correct:** A

**Explanation:**
- **A (correct):** When issues interact, a single detailed message lets Claude reason about the interactions and produce a coherent fix.
- **B:** Sequential fixes for interacting problems lead to oscillation as each fix breaks the next.
- **C:** Parallel sessions can't coordinate interactions and produce conflicting fixes.
- **D:** Ignoring interactions guarantees regressions.

---

### Q34 | Scenario 6 | TS 3.5 | difficulty: medium

**Stem:** A data-extraction script returns inconsistent date formats: "2025-05-11", "May 11, 2025", and "11/5/25" across runs from the same input. Which technique most directly stabilizes the output?

A) Provide 2–3 concrete examples showing input → "2025-05-11" (or your preferred format) so the transformation is unambiguous.
B) Ask Claude to "be consistent" more emphatically.
C) Switch models.
D) Add a longer rationale section before the answer.

**Correct:** A

**Explanation:**
- **A (correct):** Concrete input/output examples are the single most effective way to communicate exact transformations when prose is interpreted inconsistently.
- **B:** Emphasis doesn't disambiguate "consistent."
- **C:** Model swaps don't fix the underlying ambiguity.
- **D:** Rationale doesn't pin down format.

---

### Q35 | Scenario 4 | TS 3.5 | difficulty: hard

**Stem:** You're asking Claude to implement a complex retry policy for a distributed system. The team has strong views on backoff strategy, jitter, and idempotency keys, but each engineer's preferences differ. What workflow best surfaces these constraints before code is committed?

A) Use the interview pattern: have Claude ask the developer specific questions (max retries, backoff curve, jitter source, idempotency strategy, failure budget) before implementing.
B) Have Claude pick reasonable defaults and merge the PR; iterate via code review.
C) Ask Claude to implement three different retry policies in parallel and let the team vote.
D) Skip discussion and let production behavior reveal the right policy.

**Correct:** A

**Explanation:**
- **A (correct):** The interview pattern surfaces design considerations and choices ahead of implementation, especially valuable for opinionated cross-cutting concerns like retry policies.
- **B:** "Implement first, debate later" is exactly the rework that the interview pattern prevents.
- **C:** Three parallel implementations are wasteful when targeted questions would converge faster.
- **D:** Production should not be the design forum for retry behavior.

---

### Q36 | Scenario 5 | TS 3.6 | difficulty: easy

**Stem:** Your CI job runs Claude Code as part of a GitHub Actions workflow. The job hangs indefinitely waiting for input. What CLI flag should you use to run Claude Code non-interactively?

A) `-p` (or `--print`) to run in non-interactive mode suitable for automated pipelines.
B) `--quiet` to suppress logs.
C) `--ci` to enable a CI mode flag.
D) `--background` to detach the process.

**Correct:** A

**Explanation:**
- **A (correct):** The `-p` / `--print` flag is the documented way to run Claude Code non-interactively in pipelines, preventing waits on stdin.
- **B:** Suppressing logs doesn't change the interaction model.
- **C:** There is no canonical `--ci` flag described for this purpose.
- **D:** Backgrounding doesn't prevent stdin waits.

---

### Q37 | Scenario 5 | TS 3.6 | difficulty: medium

**Stem:** Your CI pipeline needs Claude Code to emit findings that a downstream script will parse and post as inline PR comments. Free-form text is causing parser errors. Which flags most directly produce machine-parseable output?

A) Use `--output-format json` together with `--json-schema` to enforce a schema-validated structured response.
B) Use `--markdown` to standardize on markdown output.
C) Parse the natural-language output with regex.
D) Use `-p` alone and assume Claude returns JSON when asked nicely.

**Correct:** A

**Explanation:**
- **A (correct):** `--output-format json` plus `--json-schema` produces structured, schema-validated output suitable for automated parsing.
- **B:** Markdown is still natural language and fragile to parse reliably.
- **C:** Regex on free-form output is brittle and the reason the team is struggling.
- **D:** Asking nicely for JSON without schema enforcement still yields format drift.

---

### Q38 | Scenario 5 | TS 3.6 | difficulty: medium

**Stem:** Your CI reviewer comments on every push, repeating the same findings on issues a developer hasn't fixed yet. Reviewers and authors are drowning in duplicate comments. What's the most effective fix?

A) Include prior review findings in the next run's context and instruct Claude to report only new or still-unaddressed issues.
B) Hide comments after 24 hours.
C) Disable Claude reviews on subsequent commits.
D) Have Claude regenerate all comments but mark old ones as "duplicate."

**Correct:** A

**Explanation:**
- **A (correct):** Feeding prior findings into context and asking Claude to suppress duplicates is the recommended pattern for re-reviews after new commits.
- **B:** Hiding comments doesn't reduce noise on the next run; the bug repeats.
- **C:** Disabling reviews loses the value entirely.
- **D:** Producing the same comments and tagging them as duplicates doesn't reduce noise; the channel still floods.

---

### Q39 | Scenario 5 | TS 3.6 | difficulty: medium

**Stem:** Your CI-generated tests often duplicate scenarios already covered by the existing test suite, bloating PRs. What's the most direct mitigation?

A) Provide the existing test files in the prompt context so Claude can avoid duplicating scenarios already covered.
B) Tell Claude to "not repeat tests" in the system prompt.
C) Lower the model's temperature.
D) Delete the existing test suite before generation.

**Correct:** A

**Explanation:**
- **A (correct):** Without visibility into the existing test suite, Claude can't know what's already covered. Including the existing tests in context lets it complement rather than duplicate.
- **B:** Prose instructions without visibility into existing tests can't reliably prevent duplication.
- **C:** Temperature changes don't supply information that isn't in the context.
- **D:** Deleting tests destroys coverage and is the opposite of what's needed.

---

### Q40 | Scenario 5 | TS 3.6 | difficulty: medium

**Stem:** Your team observes that the CI-invoked Claude Code job produces lower-quality test suggestions than the same engineer gets locally. Local sessions automatically pick up project context from CLAUDE.md; the CI session seems not to. What change most directly improves CI quality without changing the invocation flags?

A) Ensure the CI working directory is the repo root (or the relevant package directory) so the CI-invoked Claude Code picks up the project's CLAUDE.md context (testing standards, fixtures, criteria).
B) Hard-code testing standards into the GitHub Actions YAML inline.
C) Ask the developer to attach their personal `~/.claude/CLAUDE.md` to the CI environment.
D) Tell the CI job to ignore CLAUDE.md and rely on Claude's training data.

**Correct:** A

**Explanation:**
- **A (correct):** CLAUDE.md is the documented mechanism for providing project context (testing standards, fixture conventions, review criteria) to CI-invoked Claude Code. Running from the right directory ensures it loads.
- **B:** Inline YAML duplicates and drifts; CLAUDE.md is the single source of truth.
- **C:** User-level config is personal and shouldn't drive CI behavior.
- **D:** Ignoring CLAUDE.md throws away the team's accumulated guidance — the opposite of what you want.

---

### Q41 | Scenario 5 | TS 3.6 | difficulty: hard

**Stem:** Your CI pipeline first uses Claude Code to generate a feature and then, in the same session, asks Claude to review its own changes. The reviews are unusually mild and miss issues that human reviewers catch later. What's the most likely cause and recommended remedy?

A) Self-review in the same session shares context with code generation, biasing Claude toward defending its choices; run the review in a separate, independent Claude Code invocation with no generation context.
B) Self-review failures are inevitable; abandon Claude reviews entirely.
C) Raise the model's temperature for the review step to encourage more criticism.
D) Use one large session but ask Claude twice to "please be more critical."

**Correct:** A

**Explanation:**
- **A (correct):** A Claude session that generated code is less effective at reviewing it because the prior reasoning anchors the review. An independent review instance avoids that bias.
- **B:** The problem is solvable with separation; abandoning reviews is overkill.
- **C:** Temperature is not a substitute for context isolation.
- **D:** Asking the same session to be more critical doesn't remove the bias from shared context.

---

### Q42 | Scenario 5 | TS 3.6 | difficulty: hard

**Stem:** Your CI runs Claude Code with `-p` and `--output-format json`, but downstream tooling occasionally crashes on malformed JSON (extra prose, missing fields). What single configuration adjustment most directly hardens the contract between Claude and your tooling?

A) Add `--json-schema` referencing a strict schema with required fields and types, so output is validated against the contract instead of relying on Claude to remember to emit JSON correctly.
B) Wrap downstream parsing in a try/except and discard malformed records silently.
C) Move from `-p` to interactive mode in CI.
D) Switch the output format to markdown and parse it with regex.

**Correct:** A

**Explanation:**
- **A (correct):** `--json-schema` enforces structural validity. It's the documented way to guarantee machine-parseable output beyond just "format: json."
- **B:** Silently discarding records hides bugs and loses signal.
- **C:** Interactive mode would hang the pipeline.
- **D:** Markdown is less parseable than JSON and is exactly the kind of brittle channel you're trying to avoid.


## Domain 4: Prompt Engineering & Structured Output


42 scenario-based questions covering Task Statements 4.1–4.6.

---

### Q1 | Scenario 5 | TS 4.1 | difficulty: medium

**Stem:** Your CI PR review system uses the instruction "Be conservative and only report high-confidence issues" but developers complain that they still see noisy comments about local variable naming and minor stylistic preferences in pull requests. Dismissal rates on style findings have reached 87%, and developers are starting to ignore the bot's bug reports as well. Which prompt change most directly addresses the noise?

A) Replace the confidence-based instruction with explicit categorical criteria that name which issue classes to report (security, data corruption, concurrency bugs) and which to skip (naming, local-scope style, formatting).
B) Lower the bot's "report threshold" by raising the confidence floor from 0.7 to 0.95 so only the highest-confidence findings reach the PR.
C) Add a global instruction "Only mention an issue if you are extremely certain it is a real bug, otherwise stay silent."
D) Re-rank findings by predicted severity and only post the top 3 issues per PR regardless of category.

**Correct:** A

**Explanation:**
- **A (correct):** Explicit categorical criteria — naming which issues to report and which to skip — give the model concrete decision boundaries, while general "be conservative" instructions have been shown not to reliably suppress unwanted categories.
- **B:** Confidence-based filtering doesn't change the underlying judgment the model makes; the model can still be very confident about style issues you don't want flagged.
- **C:** This is the same vague self-restraint pattern that already failed; explicit category lists outperform generic certainty instructions.
- **D:** Top-N filtering loses real bugs when many findings come in, and still doesn't suppress the noisy categories at the source.

---

### Q2 | Scenario 5 | TS 4.1 | difficulty: easy

**Stem:** A code-review prompt currently reads: "Flag comments that look inaccurate." Reviewers find the bot is flagging stale TODOs, comments referencing renamed variables, and even speculative wording. You want it to focus only on comments whose stated behavior contradicts what the code actually does. Which revision best embodies the principle of explicit criteria?

A) "Be more careful when flagging comments — only flag the ones that are clearly wrong."
B) "Flag a comment only when its claimed behavior contradicts the actual behavior of the surrounding code (e.g., comment says 'returns null on error' but code throws)."
C) "Use your best judgment about which comments are misleading."
D) "Only flag a comment if you are at least 90% confident it is inaccurate."

**Correct:** B

**Explanation:**
- **B (correct):** This replaces a vague directive with a precise behavioral criterion plus an example, exactly matching the "explicit criteria over vague instructions" principle from TS 4.1.
- **A:** "Clearly wrong" is the same class of vague instruction the original prompt used.
- **C:** "Best judgment" leaves the criterion entirely undefined.
- **D:** Confidence thresholds don't constrain what counts as inaccurate; the model may be 90% sure that a stylistic comment is "wrong."

---

### Q3 | Scenario 6 | TS 4.1 | difficulty: medium

**Stem:** Your invoice-extraction pipeline currently uses a single severity field with the instructions: "Set severity to high, medium, or low based on importance." Downstream automation routes "high" findings to manual review. Audit shows that classifications are inconsistent: identical missing-tax-ID issues get classified high by some runs and low by others, and routing volume is wildly variable. What change improves classification consistency the most?

A) Increase temperature to encourage exploration of severity values, then majority-vote across three runs.
B) Replace the severity instruction with explicit criteria for each level, including concrete examples of fields/conditions that map to each severity.
C) Add the line "Be consistent across runs" to the prompt and re-run.
D) Drop the severity field entirely and let downstream code infer severity from which fields are missing.

**Correct:** B

**Explanation:**
- **B (correct):** TS 4.1 emphasizes defining explicit severity criteria with concrete examples for each level — this gives the model a deterministic mapping rather than asking it to guess what "importance" means.
- **A:** Higher temperature increases variance, the exact opposite of what's needed; ensembling treats the symptom, not the cause.
- **C:** Vague compliance instructions like "be consistent" do not produce consistency.
- **D:** Stripping the field shifts the judgment problem into downstream code without giving it the context the model has.

---

### Q4 | Scenario 5 | TS 4.1 | difficulty: medium

**Stem:** Developer trust in your CI bot has cratered. Logs show three issue categories the bot reports: SQL injection (5% false-positive rate, low volume), null-pointer risks (12% FPR, medium volume), and "potential code smell" (78% FPR, high volume). Dismissal rates on the high-quality SQL-injection findings have started rising too, even though those reports are usually correct. What is the best immediate intervention?

A) Keep all three categories but add a banner explaining that "code smell" findings are advisory only.
B) Temporarily disable the "potential code smell" category in production while iterating on a better prompt for it; keep SQL injection and null-pointer findings live.
C) Apply a global confidence threshold of 0.9 across all three categories.
D) Replace the prompt with a single instruction to "find anything that might be wrong" and post all findings under one combined category.

**Correct:** B

**Explanation:**
- **B (correct):** TS 4.1 calls out temporarily disabling high-FPR categories to restore developer trust in the accurate categories while you iterate; trust is contagious across the bot's output as a whole.
- **A:** A disclaimer doesn't stop the noise from drowning out the good signals.
- **C:** Confidence thresholds don't fix the underlying criteria problem and may also suppress the good categories.
- **D:** Combining categories actively destroys the precise category-level controls you need.

---

### Q5 | Scenario 6 | TS 4.1 | difficulty: hard

**Stem:** A regulated-document extraction pipeline must distinguish between "data conflict" (two parts of the document contradict each other) and "data missing" (no value present). Reviewers find the model often reports "data conflict" when a field is simply absent, because the prompt says: "Report any inconsistency you find." Which prompt revision best embodies TS 4.1 guidance?

A) Add: "Be cautious about reporting conflicts unless you are sure."
B) Define explicit criteria: "Report 'data_conflict' only when two distinct stated values for the same field appear in the document. Report 'data_missing' when no value appears. Do not treat absence as conflict."
C) Increase the model's temperature so it considers alternative explanations before classifying.
D) Move the conflict/missing distinction into a downstream rule engine and remove it from the prompt entirely.

**Correct:** B

**Explanation:**
- **B (correct):** Explicit, mutually exclusive categorical criteria with definitions are exactly the TS 4.1 prescription — they replace a vague "inconsistency" directive with operationally distinct conditions.
- **A:** "Be cautious" is the same class of generic instruction that consistently underperforms explicit criteria.
- **C:** Temperature changes increase variance and don't disambiguate categories.
- **D:** Outsourcing the distinction loses the model's ability to look at surrounding context to discriminate, and adds complexity for an issue prompting can solve.

---

### Q6 | Scenario 5 | TS 4.1 | difficulty: easy

**Stem:** Which of the following review-criteria statements is the strongest example of "explicit, specific criteria" rather than "vague instructions"?

A) "Report issues only when you have high confidence they are real bugs."
B) "Report only the issues that an experienced senior engineer would care about."
C) "Report issues only in these categories: SQL injection, missing authorization checks, unhandled exceptions in request handlers. Do not report variable naming, comment style, or local refactoring suggestions."
D) "Use your best judgment about what is worth surfacing."

**Correct:** C

**Explanation:**
- **C (correct):** C names exact categories to include and exclude, the canonical TS 4.1 "explicit categorical criteria" pattern.
- **A:** Confidence-based filters don't constrain category boundaries.
- **B:** "Senior engineer would care" is undefined and leaves all judgment to the model.
- **D:** "Best judgment" is the prototypical vague instruction.

---

### Q7 | Scenario 5 | TS 4.1 | difficulty: medium

**Stem:** Your team adds a new "performance regression" review category to the CI bot. After a week, the category has a 65% dismissal rate and developers report that most performance findings are speculative (e.g., "this loop might be slow on large inputs"). Meanwhile, the bot's high-quality "security" findings still have a 6% dismissal rate. What should you do first?

A) Disable the performance-regression category until you can rewrite its prompt with explicit, concrete criteria (e.g., O(n^2) on inputs you can show exceed 1000 elements).
B) Push the performance findings as "informational only" instead of full review comments.
C) Add an LLM-as-judge that rates each performance finding's value and silently filters low scores.
D) Train developers to be more receptive to performance feedback.

**Correct:** A

**Explanation:**
- **A (correct):** Disabling high-FPR categories temporarily, while iterating on better criteria, is the TS 4.1 recommendation to protect trust in the rest of the system.
- **B:** Demoting to "informational" still pollutes the PR stream and doesn't fix the underlying criteria problem.
- **C:** Adds machinery on top of a prompt that should be fixed directly; introduces opaque filtering and added latency.
- **D:** Shifts blame to developers rather than addressing the false-positive rate, which is the legitimate cause of dismissals.

---

### Q8 | Scenario 6 | TS 4.2 | difficulty: medium

**Stem:** Your extraction pipeline processes purchase orders. Most are formal POs with explicit quantity and unit columns, but ~15% are informal e-mails where users write things like "2 boxes of #45, around 30 lbs total" or "a couple of pallets, give or take." Detailed natural-language instructions to "normalize informal measurements" have produced inconsistent results: sometimes the model fabricates exact numbers, sometimes it leaves fields null. What technique most effectively improves consistency?

A) Add 2-4 few-shot examples showing exactly how informal phrasings should map to the structured output (e.g., "a couple of pallets" → quantity_estimate: 2, confidence: "low", original_phrase: "a couple of pallets").
B) Increase temperature to encourage creative interpretation of informal language.
C) Reject any document that contains informal phrasing and require human review.
D) Add a long paragraph in the prompt explaining typical informal measurement idioms.

**Correct:** A

**Explanation:**
- **A (correct):** TS 4.2 explicitly calls out few-shot examples as the most effective technique when instructions alone produce inconsistent extraction, especially for varied document structures and informal phrasings.
- **B:** More temperature increases hallucination, the opposite of what's needed.
- **C:** Rejecting 15% of documents abandons the use case rather than solving it.
- **D:** Long natural-language descriptions of edge cases is exactly the approach that already failed; demonstrations beat descriptions for ambiguous-case handling.

---

### Q9 | Scenario 5 | TS 4.2 | difficulty: medium

**Stem:** Your CI bot's review output format is inconsistent: sometimes it gives a one-line gripe, sometimes a five-paragraph essay, occasionally just a code snippet without context. You want every finding to have location, issue description, severity, and suggested fix. You've already written detailed prose instructions describing this format but compliance is only ~70%. What's the best next step?

A) Add 3 few-shot examples showing the exact desired output structure with concrete (location, issue, severity, suggested fix) fields filled in.
B) Add the line "It is critical that you follow the format above" three times to the system prompt.
C) Move from a single LLM call to two calls — one to generate findings and one to reformat them.
D) Switch to a smaller model so output stays shorter and more uniform.

**Correct:** A

**Explanation:**
- **A (correct):** TS 4.2 highlights that few-shot examples demonstrating the desired output format (location, issue, severity, suggested fix) are the most effective way to achieve consistency when descriptive instructions alone produce ~70% compliance.
- **B:** Repetition of vague urgency doesn't substantively improve compliance.
- **C:** Two calls add cost and complexity; demonstrations alone usually solve format issues.
- **D:** Smaller models follow formatting instructions worse, not better, and "shorter" is not the same as "structured."

---

### Q10 | Scenario 6 | TS 4.2 | difficulty: hard

**Stem:** A research paper extractor must capture citations whether they appear as inline footnote markers ("[12]"), parenthetical author–year ("Smith, 2021"), or compiled bibliographies at the end. Single-format examples in the prompt don't generalize: the model handles one format and ignores the others. What few-shot strategy generalizes best?

A) One detailed example per format (inline footnote, parenthetical, bibliography), each showing how the same underlying citation should map to the same structured output.
B) Twenty examples of the most common format (parenthetical) only, since most papers use that style.
C) A single composite example that includes all three citation formats in one synthetic document.
D) No examples — write a longer natural-language description covering each format.

**Correct:** A

**Explanation:**
- **A (correct):** TS 4.2 calls out using a few targeted examples that demonstrate correct handling of varied document structures (inline citations vs. bibliographies); covering each structural variant with one clear example enables generalization across the patterns.
- **B:** Twenty examples of one format teach the model only that format and hurt generalization to the others.
- **C:** A single synthetic blob makes the boundaries between formats less clear and is harder for the model to map structurally.
- **D:** Descriptions were already insufficient — demonstrations of each variant outperform descriptions.

---

### Q11 | Scenario 5 | TS 4.2 | difficulty: medium

**Stem:** Your test-generation step tends to omit edge-case tests for newly added branches; when developers manually inspect, they find the model considered the branch "obvious enough not to test." You want the model to generalize judgment about when branch-level coverage gaps justify a new test. Which approach best supports generalization rather than memorization?

A) Provide 3 few-shot examples that include the reasoning ("this branch handles a null path that production data shows occurs in ~3% of inputs, so it warrants a unit test") for why each example chose to add or skip a test.
B) Provide a single negative example showing what not to do.
C) Hard-code a rule that every if/else branch must have a test.
D) Provide 50 examples that exhaustively enumerate every branch type the model might encounter.

**Correct:** A

**Explanation:**
- **A (correct):** Few-shot examples that show reasoning for why one action was chosen over plausible alternatives — exactly the TS 4.2 skill — enable the model to generalize judgment to novel branches rather than matching specific surface features.
- **B:** A single negative example doesn't establish the positive criterion you want.
- **C:** Hard rules ignore that not every trivial branch needs a test; produces noise and over-coverage.
- **D:** Massive enumerated example sets encourage pattern matching to specific surface features rather than generalization.

---

### Q12 | Scenario 6 | TS 4.2 | difficulty: easy

**Stem:** Which is the strongest motivation for using few-shot examples instead of detailed prose instructions when extracting fields from varied invoice layouts?

A) Few-shot examples are required to use any structured output features.
B) Few-shot examples are the most reliable way to demonstrate ambiguous-case handling and consistent output formatting when prose instructions alone produce inconsistent results.
C) Few-shot examples reduce token usage compared to prose instructions.
D) Few-shot examples bypass the need to define a JSON schema.

**Correct:** B

**Explanation:**
- **B (correct):** This is the core TS 4.2 finding: few-shot examples are the most effective lever for consistent, well-formatted output for ambiguous or varied inputs.
- **A:** Few-shot is independent of structured-output features; tool use works with or without examples.
- **C:** Few-shot examples typically add tokens, not subtract them.
- **D:** Few-shot and JSON schemas are complementary, not substitutes.

---

### Q13 | Scenario 5 | TS 4.2 | difficulty: medium

**Stem:** Your CI bot frequently flags common patterns that your codebase has standardized on (e.g., a custom logger wrapper) as if they were bugs. Developers complain these patterns are explicitly approved in CLAUDE.md, but the bot misses the context. You want it to recognize legitimate patterns while still catching genuine misuse. What few-shot approach helps most?

A) Add 3 examples each pairing an "acceptable pattern in this codebase" snippet with a "looks similar but is genuinely buggy" snippet, and show the correct flag/no-flag decision for both.
B) Add a single sentence: "Do not flag custom logger usage."
C) Increase the model's reasoning depth so it spends more time on each finding.
D) Remove all reasoning-related output and just dump tool call names.

**Correct:** A

**Explanation:**
- **A (correct):** TS 4.2 calls out using few-shot examples that distinguish acceptable code patterns from genuine issues to reduce false positives while enabling generalization beyond a single pattern.
- **B:** A one-off exclusion doesn't generalize to the next custom pattern and degrades over time.
- **C:** Deeper reasoning without examples doesn't change what counts as a "pattern" — the model still lacks codebase context.
- **D:** Removing reasoning output hides the problem and makes debugging harder; it doesn't improve judgment.

---

### Q14 | Scenario 6 | TS 4.2 | difficulty: medium

**Stem:** A clinical-document extractor must produce a "patient_age" field. Required fields can't be empty per your tool schema. You observe that for documents that say "elderly male" or "age not recorded," the model fabricates ages like "70" or "65" to satisfy the schema. You don't want to remove the schema constraint yet. What's the most effective short-term fix?

A) Add few-shot examples showing the correct extraction for documents that lack a patient age, explicitly demonstrating that the model should return a special sentinel ("unknown") or null where allowed, instead of inventing a value.
B) Increase temperature so the model is more creative about which ages it fabricates.
C) Add prose instructions saying "do not hallucinate."
D) Remove the patient_age field from the schema entirely.

**Correct:** A

**Explanation:**
- **A (correct):** TS 4.2 calls out adding few-shot examples for documents with varied formats and missing required fields to prevent fabrication, demonstrating the desired "missing" or null behavior.
- **B:** More temperature increases hallucination.
- **C:** "Don't hallucinate" is a generic instruction the model already nominally follows; demonstrated correct behavior is far more effective.
- **D:** Removing the field is overkill; the issue is fabrication, not field existence.

---

### Q15 | Scenario 6 | TS 4.3 | difficulty: easy

**Stem:** You need guaranteed schema-compliant JSON output from Claude when extracting structured fields from invoices. Currently you ask the model to "respond with JSON that matches this schema" in the system prompt, and ~3% of responses have malformed JSON (trailing commas, missing brackets) which break your parser. What's the most reliable mitigation?

A) Define an extraction tool with the JSON schema as its input parameters and parse the structured data from the `tool_use` response.
B) Add a final-pass regex to clean up trailing commas before parsing.
C) Switch to JSON5 parsing on the client side.
D) Add the phrase "do not include trailing commas" to the prompt.

**Correct:** A

**Explanation:**
- **A (correct):** TS 4.3: tool use with a JSON schema is the most reliable approach for guaranteed schema-compliant structured output and eliminates JSON syntax errors entirely.
- **B:** Regex cleanup is brittle and won't catch all malformations.
- **C:** Loosening the parser hides the underlying problem; the upstream model still produces inconsistent output.
- **D:** Prompt-only instructions still leave residual syntax errors; tool use eliminates that whole error class.

---

### Q16 | Scenario 6 | TS 4.3 | difficulty: medium

**Stem:** Your extraction service handles three document types (invoice, purchase_order, receipt). The document type is not always known up-front. You want Claude to call one of three extraction tools — `extract_invoice`, `extract_purchase_order`, `extract_receipt` — but the model sometimes returns conversational text instead of calling a tool when the document is ambiguous. What tool_choice setting most directly fixes this?

A) `tool_choice: "auto"` so the model can decide whether a tool is appropriate.
B) `tool_choice: "any"` so the model must call one of the available tools.
C) `tool_choice: {"type": "tool", "name": "extract_invoice"}` to always run the invoice extractor.
D) Omit `tool_choice` entirely so the model defaults to text output.

**Correct:** B

**Explanation:**
- **B (correct):** TS 4.3 explicitly recommends `tool_choice: "any"` when multiple extraction schemas exist and the document type is unknown — it guarantees structured output while letting the model pick the right tool.
- **A:** "auto" allows the model to return free text, which is exactly the failure mode.
- **C:** Forcing the invoice tool will misclassify receipts and POs.
- **D:** Default behavior is essentially "auto" and produces the same problem.

---

### Q17 | Scenario 6 | TS 4.3 | difficulty: medium

**Stem:** Your pipeline always extracts metadata (vendor, document date, currency) before running enrichment subsequently. You want to guarantee that the metadata-extraction tool runs every time, regardless of model judgment. What is the right configuration?

A) Set `tool_choice: "any"` and hope the model picks `extract_metadata`.
B) Set `tool_choice: {"type": "tool", "name": "extract_metadata"}` to force the specific tool.
C) Mention "always extract metadata first" in the system prompt with `tool_choice: "auto"`.
D) Provide only the `extract_metadata` tool, but allow the model to return text instead.

**Correct:** B

**Explanation:**
- **B (correct):** TS 4.3 explicitly covers forcing a specific tool via `tool_choice: {"type": "tool", "name": "extract_metadata"}` to ensure a particular extraction runs before enrichment steps.
- **A:** "any" allows any tool, not the specific one you need.
- **C:** Prompt-based ordering is probabilistic, not deterministic.
- **D:** With "auto" implicit and no forced tool, the model may still produce text.

---

### Q18 | Scenario 6 | TS 4.3 | difficulty: hard

**Stem:** Your schema requires `tax_id` for every extracted vendor. Documents from small vendors often omit a tax ID. The model satisfies the required field by inventing plausible-looking tax IDs. Which schema design change most directly addresses fabrication while preserving structure?

A) Add a regex constraint requiring the tax ID to match a specific format.
B) Make `tax_id` an optional/nullable field so the model can legitimately return null when no value is present in the source document.
C) Remove `tax_id` from the schema entirely.
D) Add a prompt instruction "Never hallucinate values."

**Correct:** B

**Explanation:**
- **B (correct):** TS 4.3 covers designing schema fields as optional/nullable when source documents may not contain the information, preventing the model from fabricating values to satisfy required fields.
- **A:** Format constraints reduce the form of hallucination but don't prevent it — the model can fabricate values that pass a regex.
- **C:** Removing the field loses information when it is present.
- **D:** Prompt-only instructions don't reliably prevent fabrication when the schema forces a required value.

---

### Q19 | Scenario 6 | TS 4.3 | difficulty: medium

**Stem:** Your tooling categorizes documents into a closed set of types (invoice, receipt, contract). Real-world data includes the occasional unusual document (a credit memo, a shipping manifest) that doesn't fit any known category. The schema forces an enum classification and you see the model crowbar these documents into the closest known type. Which schema adjustment best handles extensibility without losing precision?

A) Switch the field from enum to a free-text string field.
B) Add an "other" enum value plus an `other_detail` string field that captures the actual category name when "other" is selected.
C) Add 47 new enum values to cover every observed edge case.
D) Remove the enum entirely and let downstream code classify on raw text.

**Correct:** B

**Explanation:**
- **B (correct):** TS 4.3 covers the "other" + detail-string pattern as the canonical extensible categorization design — it preserves precision for known categories and captures the unknown without forcing misclassification.
- **A:** Free-text loses the consistency benefit of enums for the known categories.
- **C:** Enumerating every edge case is brittle and never finishes.
- **D:** Pushing classification downstream forfeits the model's context-awareness.

---

### Q20 | Scenario 6 | TS 4.3 | difficulty: easy

**Stem:** A strict JSON schema enforced via tool use guarantees that the model's output:

A) Is free from JSON syntax errors and matches the field types, but may still contain semantic errors like line items that don't sum to the stated total.
B) Is completely free from any factual or semantic errors.
C) Cannot omit required fields, ever.
D) Will match the source document's information exactly.

**Correct:** A

**Explanation:**
- **A (correct):** TS 4.3 is explicit: tool-use schemas eliminate JSON syntax errors but do not prevent semantic errors (values in wrong fields, totals that don't sum).
- **B:** Schemas don't guarantee factual correctness; only structural correctness.
- **C:** The model can still set required-but-nullable fields incorrectly and may produce empty values in some configurations.
- **D:** Schema compliance is independent of source-document fidelity.

---

### Q21 | Scenario 6 | TS 4.3 | difficulty: medium

**Stem:** Invoice dates appear in many formats: "01/02/2024," "Feb 1 2024," "2024-02-01," and sometimes "received Q1 2024." Your downstream system requires ISO 8601 dates. Schema alone doesn't guarantee normalization — the model often passes the source string through verbatim. What's the strongest combination of techniques?

A) Define a strict schema with the date field typed as ISO 8601 string and include normalization rules and a few-shot example mapping varied source formats to ISO 8601 in the prompt.
B) Skip the prompt-level rules; rely entirely on the JSON schema to coerce the date.
C) Accept raw strings in the schema and add a separate post-processing service that calls Claude again to normalize.
D) Increase temperature so the model varies its date format outputs.

**Correct:** A

**Explanation:**
- **A (correct):** TS 4.3 calls out including format normalization rules in the prompt alongside strict output schemas to handle inconsistent source formatting; schemas alone don't enforce normalization semantics.
- **B:** Schemas don't perform format coercion; they validate shape.
- **C:** Adds latency and cost for a transformation that can happen inline.
- **D:** Higher temperature creates more variance, the opposite of what's needed.

---

### Q22 | Scenario 6 | TS 4.4 | difficulty: medium

**Stem:** Your extraction tool's first pass on a document produces output that fails validation: line items sum to $1,420.00 but the `stated_total` field is $1,520.00. The source document contains both line items and a footer total. What is the most effective retry strategy?

A) Re-send the original request unchanged and hope for a different result.
B) Append a follow-up message that includes the original document, the failed extraction, and the specific validation error (line-item sum vs. stated_total mismatch), asking the model to reconcile.
C) Lower the temperature to zero and re-run.
D) Reject the document as un-extractable and route it to a human queue.

**Correct:** B

**Explanation:**
- **B (correct):** TS 4.4 describes the retry-with-error-feedback pattern: include the original document, the failed extraction, and specific validation errors so the model can self-correct.
- **A:** Identical inputs typically reproduce identical errors.
- **C:** Temperature tweaks don't fix a semantic reasoning error.
- **D:** Premature escalation; many of these are correctable with a single feedback retry.

---

### Q23 | Scenario 6 | TS 4.4 | difficulty: hard

**Stem:** Your extractor needs a "purchase_order_number" from each invoice. For 7% of invoices, the PO number was sent in a separate cover email that isn't part of the document passed to the model. Validation fails because the field is empty. Retries with the same input keep producing empty results. What's the right diagnosis?

A) The retry will eventually succeed if you try enough times because the model occasionally guesses correctly.
B) The retry will be ineffective because the required information simply isn't in the source document; you need to provide the cover email or accept null.
C) The retry needs a stricter schema enforcement to force the model to produce a value.
D) The retry needs more aggressive prompting to "find" the missing data.

**Correct:** B

**Explanation:**
- **B (correct):** TS 4.4 explicitly distinguishes when retries succeed (format/structural errors) from when they don't (information is simply absent from the source). When data isn't present, retries cannot recover it without changing the input.
- **A:** Retries don't conjure missing data; relying on lucky guesses is a fabrication risk.
- **C:** Stricter schema enforcement just pushes the model toward fabrication.
- **D:** "Find harder" cannot recover information that isn't there.

---

### Q24 | Scenario 5 | TS 4.4 | difficulty: medium

**Stem:** Your CI bot's findings have a 40% dismissal rate but you don't know which finding categories are driving it. You'd like to systematically analyze which code constructs (e.g., particular API patterns, certain idioms) tend to produce false positives. What output design supports this analysis best?

A) Add a `detected_pattern` field to each structured finding that captures which code construct or pattern triggered the finding, so dismissal data can be aggregated by pattern.
B) Add a free-text "rationale" field and ask developers to read it before dismissing.
C) Send all findings to a separate LLM-as-judge that decides whether to surface them.
D) Reduce findings to a single severity number and skip categorization.

**Correct:** A

**Explanation:**
- **A (correct):** TS 4.4 calls out adding `detected_pattern` fields to enable systematic analysis of dismissal patterns — once you can group dismissals by pattern, you can target prompt fixes precisely.
- **B:** Free-text rationale is not aggregable across thousands of findings.
- **C:** Adds an additional opaque filter rather than instrumenting the existing flow.
- **D:** Reducing to a number destroys the very categorical information needed to analyze patterns.

---

### Q25 | Scenario 6 | TS 4.4 | difficulty: medium

**Stem:** Your extractor returns a `stated_total` field for invoices. A common failure mode is misreading the cents place (e.g., $1,234.56 → $1,234.65). You want self-correction during extraction. Which validation design helps the model catch its own mistakes?

A) Extract both `calculated_total` (summed from line items) and `stated_total` (the document's footer), so a downstream check can flag discrepancies — and surface them back to the model for self-correction.
B) Extract only `stated_total` and rely on the schema to enforce correctness.
C) Drop the total field entirely.
D) Hardcode `stated_total` to always equal the sum of line items.

**Correct:** A

**Explanation:**
- **A (correct):** TS 4.4's self-correction validation flow exactly recommends extracting `calculated_total` alongside `stated_total` so that a discrepancy can be flagged and surfaced for correction.
- **B:** A single field provides nothing to validate against.
- **C:** Eliminates a critical business field.
- **D:** Silently rewriting the stated total destroys the document's truth and hides real discrepancies.

---

### Q26 | Scenario 6 | TS 4.4 | difficulty: hard

**Stem:** A vendor invoice lists two different "Bill To" addresses on the same page — one in the header, one in the footer — due to a template error. Your extractor consistently picks one without surfacing the conflict, and downstream addressing is sometimes wrong. What schema enhancement best surfaces this kind of issue for human review?

A) Have the schema produce only the first-found "Bill To" address with no metadata.
B) Add a `conflict_detected` boolean (and optionally a `conflicting_values` list) to the schema so the model can mark documents whose source data is internally inconsistent.
C) Increase temperature so the model picks differently each time and majority-vote.
D) Add a confidence score to the address field.

**Correct:** B

**Explanation:**
- **B (correct):** TS 4.4 calls out adding "conflict_detected" booleans (and supporting fields) for inconsistent source data, which routes ambiguous cases to human review instead of silently choosing.
- **A:** Silent picks are the failure mode you're trying to fix.
- **C:** Variance-based methods don't reliably surface the conflict and add cost.
- **D:** Confidence on the chosen value doesn't reveal that there was a conflict in the first place.

---

### Q27 | Scenario 6 | TS 4.4 | difficulty: medium

**Stem:** Which of the following error types is generally NOT addressed by tool-use schema validation but IS addressable by retry-with-error-feedback?

A) Trailing commas in JSON output.
B) Misnamed fields that don't exist in the schema.
C) Line items that sum to $980 when the stated total is $1,000.
D) Field type mismatches (string where integer was required).

**Correct:** C

**Explanation:**
- **C (correct):** TS 4.4 distinguishes semantic validation errors (values don't sum, wrong field placement) from schema syntax errors — tool use eliminates the syntax errors but not the semantic ones, which retry-with-error-feedback can address.
- **A:** Syntax errors are eliminated by tool use.
- **B:** Schema-defined field names prevent this class of error.
- **D:** Schema type enforcement prevents type mismatches at the syntactic level.

---

### Q28 | Scenario 5 | TS 4.4 | difficulty: easy

**Stem:** Your team wants to understand why developers dismiss specific findings so you can improve the prompt. Which design choice is most aligned with TS 4.4 best practice?

A) Capture only the count of dismissals per repo.
B) Add a `detected_pattern` field per finding, log it alongside dismissals, and aggregate to identify high-FPR patterns.
C) Survey developers manually each quarter.
D) Have the model dismiss its own findings before posting.

**Correct:** B

**Explanation:**
- **B (correct):** Structured detected_pattern logging gives you the data needed to do systematic prompt fixes, the exact TS 4.4 skill.
- **A:** Aggregate counts don't tell you which categories or patterns to fix.
- **C:** Manual surveys are slow and miss patterns at scale.
- **D:** Self-dismissal is opaque and removes signal from the feedback loop.

---

### Q29 | Scenario 5 | TS 4.5 | difficulty: easy

**Stem:** Your team currently runs CI PR reviews using the synchronous API and pays full rates. Someone suggests moving PR reviews to the Message Batches API for 50% cost savings. What is the most important reason to be cautious?

A) The batch API doesn't support the same models.
B) The batch API has up to a 24-hour processing window and no guaranteed latency SLA, which is incompatible with blocking pre-merge checks.
C) The batch API forces you to bundle thousands of requests at once.
D) The batch API can't return structured output.

**Correct:** B

**Explanation:**
- **B (correct):** TS 4.5 explicitly states batch is inappropriate for blocking workflows (pre-merge checks) because the up-to-24-hour window and lack of latency SLA conflict with the workflow's need to gate merges.
- **A:** Model availability isn't the main concern raised by TS 4.5.
- **C:** Batches can be any size; minimum bundling isn't the issue.
- **D:** Structured output works in batch.

---

### Q30 | Scenario 5 | TS 4.5 | difficulty: medium

**Stem:** Your CI system runs two workloads: (1) blocking pre-merge code review on every PR (must return in <5 minutes); and (2) a nightly test-generation pass over recently merged code (results consumed the next morning). How should you allocate APIs?

A) Use the batch API for both to maximize cost savings.
B) Use the synchronous API for the pre-merge review and the batch API for the nightly test generation.
C) Use the synchronous API for both to keep them simple.
D) Use the batch API for the pre-merge review and the synchronous API for the nightly job.

**Correct:** B

**Explanation:**
- **B (correct):** TS 4.5 prescribes matching API approach to latency: synchronous for blocking pre-merge checks, batch for overnight/weekly analyses where 24-hour processing is acceptable.
- **A:** Batch can't meet a 5-minute SLA.
- **C:** Pays full cost on a workload with no real-time requirement.
- **D:** Has the priorities backward.

---

### Q31 | Scenario 6 | TS 4.5 | difficulty: hard

**Stem:** Your service guarantees a 30-hour SLA on document extractions. You'd like to use the batch API (up to 24-hour processing window) to save 50%. What batch submission cadence guarantees the 30-hour SLA?

A) Submit every 24 hours, since the API guarantees results within 24 hours.
B) Submit every 6 hours, since the SLA is half a day.
C) Submit every 4 hours so that any document accepted into the system has at most 30 hours (4 hours wait + 24 hours processing + 2 hours buffer) before completion.
D) Submit every 30 hours, matching the SLA exactly.

**Correct:** C

**Explanation:**
- **C (correct):** TS 4.5 covers calculating batch submission frequency from SLA constraints: a 4-hour cadence keeps total worst-case latency (wait + 24h processing + buffer) within the 30-hour SLA.
- **A:** 24-hour cadence pushes worst case to ~48 hours.
- **B:** 6-hour cadence + 24-hour processing = up to 30 hours but leaves zero buffer for delays/failures.
- **D:** A 30-hour cadence vastly exceeds the SLA.

---

### Q32 | Scenario 6 | TS 4.5 | difficulty: medium

**Stem:** A batch run finishes and 6 out of 10,000 documents failed because they exceeded the model's context limit. The remaining 9,994 succeeded. What's the right way to handle the failures?

A) Resubmit the entire batch from scratch.
B) Resubmit only the 6 failed documents (identified by their `custom_id` values), chunking the oversized documents into smaller pieces before resubmission.
C) Mark all 10,000 results as invalid and start a manual review.
D) Lower the model size for all 10,000 and resubmit.

**Correct:** B

**Explanation:**
- **B (correct):** TS 4.5 covers using `custom_id` to identify failed documents and resubmitting only those with appropriate modifications (e.g., chunking when the failure was a context overflow).
- **A:** Wastes 99.94% of completed work.
- **C:** Invalidates good results unnecessarily.
- **D:** Smaller models often have smaller context windows; the change doesn't address the root cause.

---

### Q33 | Scenario 6 | TS 4.5 | difficulty: medium

**Stem:** You plan to extract metadata from 200,000 invoices using the batch API. A pilot suggests your current prompt has a ~30% first-pass failure rate (mostly validation errors). What's the most cost-effective preparation step?

A) Submit the full batch immediately; address failures by automatic retry.
B) Iterate on prompt refinement against a sample (e.g., 500 invoices) until the first-pass success rate is high, then run the full batch.
C) Switch to the synchronous API to get faster feedback.
D) Skip validation and accept the 30% failure rate.

**Correct:** B

**Explanation:**
- **B (correct):** TS 4.5 calls out prompt refinement on a sample before batch-processing large volumes to maximize first-pass success rates and avoid expensive iterative resubmission across 200K items.
- **A:** Resubmitting 60,000 failed documents wastes 50% savings and pipeline time.
- **C:** Drops the cost-savings benefit entirely; refinement is independent of API choice.
- **D:** A 30% failure rate is unacceptable for production extraction.

---

### Q34 | Scenario 5 | TS 4.5 | difficulty: medium

**Stem:** Your weekly security audit re-scans your entire codebase to look for newly disclosed vulnerability patterns. The job runs Sunday night and results are reviewed Monday morning. You also want to use multi-turn tool calling (the model calls a tool, gets the result back, and chooses a next step). Which constraint is most important to consider?

A) Batch API offers cost savings but does not support multi-turn tool calling within a single request.
B) Batch API supports only short prompts.
C) Batch API requires a different authentication flow that isn't available in your CI runner.
D) Batch API doesn't accept JSON schemas.

**Correct:** A

**Explanation:**
- **A (correct):** TS 4.5 highlights that the batch API doesn't support multi-turn tool calling within a single request (it can't execute tools mid-request and return results); for tool loops you need an alternative approach.
- **B:** Batch supports normal-size prompts.
- **C:** Authentication is the same.
- **D:** JSON schemas / tool use work fine in batch — but tool-result handling within a single batch call doesn't.

---

### Q35 | Scenario 6 | TS 4.5 | difficulty: easy

**Stem:** Why does the Message Batches API include a `custom_id` on each request?

A) It's required for billing reconciliation.
B) It correlates request/response pairs so you can map results back to your source items after asynchronous processing.
C) It's a security token to prevent replay attacks.
D) It controls request priority within a batch.

**Correct:** B

**Explanation:**
- **B (correct):** TS 4.5 covers `custom_id` as the mechanism for correlating batch request/response pairs since results arrive asynchronously.
- **A:** Billing is independent of custom_id.
- **C:** Not a security mechanism.
- **D:** Batches don't expose request-level priority through custom_id.

---

### Q36 | Scenario 2 | TS 4.6 | difficulty: medium

**Stem:** A developer asks Claude Code to generate a moderately complex module, then immediately asks the same session to "review the code you just wrote and find any bugs." The review pass returns "looks good!" but a code reviewer later finds a logical error. What's the most likely reason the self-review missed it?

A) The model's review context retained the reasoning it used during generation, biasing it toward defending its own choices rather than questioning them.
B) The model lacked the right MCP tools to detect the bug.
C) The model's temperature was too low.
D) The Bash tool was not available, so it couldn't run the code.

**Correct:** A

**Explanation:**
- **A (correct):** TS 4.6 highlights this exact self-review limitation: a model retains reasoning context from generation in the same session and is less likely to question its own decisions.
- **B:** Tool availability isn't the underlying limitation here.
- **C:** Temperature doesn't address self-review bias.
- **D:** Bash availability doesn't fix the reasoning-context retention problem.

---

### Q37 | Scenario 5 | TS 4.6 | difficulty: medium

**Stem:** Your CI system generates implementation code in one Claude session and you want a meaningful second-opinion review. Which architecture provides the most effective scrutiny?

A) Ask the generator to "now critically review what you just wrote" in the same session.
B) Add an "extended thinking" mode to the generator so it spends longer on its initial output.
C) Spin up a second, independent Claude instance with no prior reasoning context to review the generated code as if it were unfamiliar.
D) Run the generator at a higher temperature to introduce more variation.

**Correct:** C

**Explanation:**
- **C (correct):** TS 4.6 specifies using a second independent instance (without prior reasoning context) for substantive review; this catches subtle issues that self-review misses.
- **A:** Self-review in the same session inherits all of the generator's biases.
- **B:** Extended thinking improves generation but doesn't escape the model's own reasoning context.
- **D:** Higher temperature doesn't change the reviewer's perspective bias.

---

### Q38 | Scenario 5 | TS 4.6 | difficulty: hard

**Stem:** A PR touches 22 files spanning the auth, billing, and notifications modules. A single-pass review of all 22 files yields contradictory recommendations, misses obvious local bugs in some files, and produces shallow feedback on others. What multi-pass restructuring best addresses this?

A) Run a single review pass but raise the model size.
B) Run one focused per-file pass for local issues across all 22 files, plus a separate integration-focused pass examining cross-file data flow (e.g., how auth tokens propagate to billing and notifications).
C) Require developers to submit separate PRs per module.
D) Run the review three times and take majority vote.

**Correct:** B

**Explanation:**
- **B (correct):** TS 4.6's multi-pass review prescription: per-file passes for local issues plus a separate cross-file integration pass — exactly addressing attention dilution and contradictory findings.
- **A:** Bigger models don't fix attention quality issues across many files.
- **C:** Shifts burden to developers rather than fixing the review system.
- **D:** Majority vote suppresses real intermittent findings without resolving the root attention problem.

---

### Q39 | Scenario 5 | TS 4.6 | difficulty: medium

**Stem:** You want the model to self-report confidence on each finding so downstream routing can decide which findings to surface unconditionally vs. send to human triage. Which verification design supports calibrated routing?

A) Make the model emit confidence as part of its structured finding (e.g., "confidence": "high"|"medium"|"low") and route based on the value; periodically audit calibration.
B) Use only the model's binary surfaced/not-surfaced choice; ignore confidence.
C) Sample 1% of findings for human review and trust the rest.
D) Always escalate every finding to human triage.

**Correct:** A

**Explanation:**
- **A (correct):** TS 4.6 calls out verification passes where the model self-reports confidence alongside each finding to enable calibrated review routing — and audits to validate the calibration over time.
- **B:** Throws away the calibration signal you'd use for routing.
- **C:** Random sampling can't drive selective routing.
- **D:** Negates the value of automation.

---

### Q40 | Scenario 3 | TS 4.6 | difficulty: medium

**Stem:** In a multi-agent research system, a synthesis subagent generates a draft report from web-search and document-analysis findings. You'd like a second pass to check for factual errors and dropped citations. Which design aligns with TS 4.6 principles?

A) Have the synthesis subagent re-read its own draft and self-critique in the same session.
B) Spin up an independent reviewer subagent that takes only the draft plus the source materials (without the synthesis agent's reasoning context) and flags factual or citation errors.
C) Have the coordinator paraphrase the draft and accept it.
D) Append the instruction "be sure this is correct" to the synthesis prompt.

**Correct:** B

**Explanation:**
- **B (correct):** Independent review by another agent without the original reasoning context is more effective at catching subtle issues than self-review — TS 4.6's central insight.
- **A:** Same session = same reasoning context = same blind spots.
- **C:** Paraphrasing doesn't add scrutiny.
- **D:** Generic exhortations don't substitute for an independent reviewer.

---

### Q41 | Scenario 5 | TS 4.6 | difficulty: easy

**Stem:** Why is independent review by a second Claude instance generally more effective than asking the same instance to self-critique?

A) The second instance always uses a larger model.
B) The independent instance has no prior reasoning context, so it's more likely to question assumptions and catch subtle issues the generator already justified to itself.
C) Independent instances are cheaper to run.
D) Independent instances have access to more MCP tools.

**Correct:** B

**Explanation:**
- **B (correct):** TS 4.6 directly states that independent review instances without prior reasoning context are more effective at catching subtle issues than self-review.
- **A:** Model size isn't the mechanism.
- **C:** Cost isn't relevant to review quality.
- **D:** Tool availability is independent of the review architecture.

---

### Q42 | Scenario 5 | TS 4.6 | difficulty: hard

**Stem:** Your team observes that file-by-file review passes catch local bugs effectively but miss bugs whose root cause spans multiple files (e.g., one file emits a stale token format and another file silently accepts it). Adding more files to each pass causes attention dilution. What architecture best resolves both problems?

A) Run all reviews as one giant single pass and ignore the dilution complaints.
B) Use a multi-pass design: per-file passes for local issues, plus a separate dedicated integration pass that gets only the cross-file interfaces / data flow surfaces and reviews them together.
C) Switch reviewers to a different model family entirely and re-run as a single pass.
D) Eliminate cross-file review and rely on developers to catch integration issues manually.

**Correct:** B

**Explanation:**
- **B (correct):** TS 4.6 prescribes splitting large reviews into per-file local passes plus a separate integration-focused pass examining cross-file data flow — directly solving local depth and cross-file coverage without overloading any single pass.
- **A:** Reintroduces the original attention dilution.
- **C:** Model swapping doesn't change the structural problem.
- **D:** Pushes responsibility back to developers, defeating the automation's purpose.


## Domain 5: Context Management & Reliability


42 scenario-based multiple-choice questions covering Task Statements 5.1–5.6.

---

### Q1 | Scenario 1 | TS 5.1 | difficulty: medium

**Stem:** Your support agent handles long sessions where customers describe multiple issues. Logs show that after roughly 15 turns, the agent begins quoting incorrect refund amounts — typically the agent restates a "summary" like "the customer is owed several hundred dollars" when the actual stated amount was $327.43. The summarization step that compresses earlier turns is dropping precise numerical values. What is the most effective remediation?

A) Replace the in-conversation summarization step with a windowed truncation that keeps only the last 10 turns verbatim.
B) Maintain a structured "case facts" block (order IDs, amounts, dates, customer-stated expectations) outside the summarized history and inject it into every prompt.
C) Increase the temperature on the summarization call so the model produces more detail.
D) Ask the model at every turn to repeat the refund amount aloud in its response so it remains in conversational memory.

**Correct:** B

**Explanation:**
- **B (correct):** Transactional facts (amounts, dates, order numbers) must be extracted into a persistent structured block that survives summarization, because progressive summarization predictably degrades exact numerical values into vague phrases.
- **A:** Truncation discards the earliest turns where the issue was originally stated, often losing the very facts you need.
- **C:** Temperature does not control faithfulness to numerical detail; it tends to make summaries less accurate, not more.
- **D:** Relies on probabilistic recall in the conversation, which is exactly the failure mode being observed.

---

### Q2 | Scenario 1 | TS 5.1 | difficulty: easy

**Stem:** Each call to `lookup_order` returns 47 fields per order (shipping carrier, warehouse codes, internal SKU mappings, tax jurisdictions, etc.), but your refund workflow only needs order ID, total amount, items, order date, and current status. Over a long support session the conversation context fills with these verbose tool outputs and Claude begins missing relevant earlier details. What is the most appropriate fix?

A) Cache the full payloads in an external store and have the model query them on demand via a new tool.
B) Trim the tool output at the integration layer to only the five refund-relevant fields before it accumulates in context.
C) Move `lookup_order` results to a system prompt section so they don't count toward the conversation context.
D) Tell Claude in the system prompt to ignore irrelevant fields when reasoning over tool results.

**Correct:** B

**Explanation:**
- **B (correct):** Trimming verbose tool outputs to only the fields that matter prevents disproportionate token consumption and avoids burying earlier salient information.
- **A:** Adds complexity without solving the per-call context bloat; each future lookup still has the same problem.
- **C:** System prompt content still counts toward the context window; this misunderstands how context budgeting works.
- **D:** Asking the model to "ignore" content doesn't remove its impact on attention and token budget.

---

### Q3 | Scenario 3 | TS 5.1 | difficulty: medium

**Stem:** A research subagent returns a 6,000-word document analysis to the synthesis subagent. The synthesis agent has limited context budget and must combine outputs from four such subagents. Reports show that important findings from the middle of each subagent's output are frequently omitted from the final synthesis. What is the best redesign?

A) Have each subagent return structured key findings (claim, source, relevance score) plus a short executive summary at the top, instead of long prose with reasoning chains.
B) Increase the synthesis agent's max tokens so it can absorb all four full reports.
C) Have the synthesis agent re-read each subagent's full output twice before writing the final report.
D) Concatenate the four reports and pass them to the synthesis agent in a single very long user message.

**Correct:** A

**Explanation:**
- **A (correct):** When a downstream agent has limited context, upstream agents should return compact structured data (key claims, citations, scores) so the synthesis agent receives high-signal input rather than verbose prose.
- **B:** Output token limits aren't the problem; the issue is the input context being filled with low-density prose.
- **C:** Re-reading does not address the lost-in-the-middle effect and wastes context.
- **D:** Pure concatenation amplifies positional bias and makes middle-of-input findings even more likely to be dropped.

---

### Q4 | Scenario 3 | TS 5.1 | difficulty: hard

**Stem:** Your research coordinator aggregates findings from eight subagents into a single prompt that is roughly 80K tokens long. Evaluations show that findings from subagents #3 through #6 are systematically underrepresented in final reports, while subagents #1, #2, #7, and #8 are well covered. What change most directly addresses this pattern?

A) Randomize subagent order on each run so no single subagent is consistently in the middle.
B) Place a "key findings summary" at the very top of the aggregated input and use explicit section headers for each subagent's detailed output below.
C) Reduce each subagent's output to under 4K tokens so the aggregated input is shorter.
D) Have the coordinator ask follow-up questions to each subagent if its findings are missing from the draft report.

**Correct:** B

**Explanation:**
- **B (correct):** The lost-in-the-middle effect causes models to under-attend to content in the middle of long inputs. Surfacing key findings at the start and labeling each section explicitly mitigates this positional bias.
- **A:** Hides the systematic problem rather than fixing it; subagents in the middle will still suffer.
- **C:** Shortening helps only if it changes positioning; it doesn't address the structural issue when many subagents are still concatenated.
- **D:** Reactive patching after the fact — slow, expensive, and doesn't fix the underlying attention pattern.

---

### Q5 | Scenario 1 | TS 5.1 | difficulty: medium

**Stem:** A customer opens a single session containing three separate issues: a damaged-on-arrival item from order #A123 ($89), a billing dispute on order #B456 ($240), and a missing accessory from order #C789 ($35). Mid-conversation, the agent confuses the refund amounts and offers $89 on the billing dispute. What context management pattern best prevents this?

A) Have the agent process only the first issue per session, then start a new session for subsequent issues.
B) Maintain a separate structured issue log (order ID, amount, status per issue) that the agent updates and references on each turn.
C) Ask the customer to restate the three amounts each time the agent responds.
D) Use a single unified summary blob updated each turn that says "customer has multiple refund requests."

**Correct:** B

**Explanation:**
- **B (correct):** Multi-issue sessions require structured persistence of issue data (IDs, amounts, statuses) in a separate context layer so the agent can disambiguate without relying on conversational recall.
- **A:** Hurts customer experience and reduces first-contact resolution — the goal is to handle all three.
- **C:** Shifts cognitive burden to the customer; unreliable.
- **D:** Exactly the progressive summarization anti-pattern: collapses distinct facts into vague language.

---

### Q6 | Scenario 1 | TS 5.1 | difficulty: easy

**Stem:** A new engineer asks why your support agent always passes the entire conversation history (not just the latest user message) in each API request to Claude. Which justification is correct?

A) Each request would otherwise be billed at the cold-start rate and cost significantly more.
B) Claude has no server-side memory of prior turns; the full conversation must be passed to maintain conversational coherence and reference earlier facts.
C) Passing only the latest message would cause Claude to leak data from other customers' sessions.
D) The Anthropic API caches conversations internally only after eight turns, so early turns require manual replay.

**Correct:** B

**Explanation:**
- **B (correct):** Models are stateless across API calls — the application is responsible for sending the conversation history each turn to preserve continuity.
- **A:** Mischaracterizes pricing; there is no "cold-start" billing concept tied to history.
- **C:** Conflates per-session state with cross-session data leakage; the latter is not how the API works.
- **D:** Invented behavior; there is no such internal caching threshold tied to history retention.

---

### Q7 | Scenario 3 | TS 5.1 | difficulty: medium

**Stem:** Subagents in your research system return findings without metadata. The synthesis agent then produces statements like "GDP grew 3.2% last year" without indicating whether "last year" refers to 2023 or 2024 data, or whether the figure comes from the IMF or the country's own statistics office. What requirement on subagent outputs best fixes this?

A) Require subagents to include structured metadata (dates, source location, methodological context) in every finding to support accurate downstream synthesis.
B) Instruct the synthesis agent to assume the most recent year when dates are ambiguous.
C) Have the synthesis agent perform its own web search to verify each statistic.
D) Tell subagents to write in a formal academic tone so ambiguity is reduced.

**Correct:** A

**Explanation:**
- **A (correct):** Structured metadata (dates, sources, methodology) accompanying each finding is what enables a downstream synthesis agent to interpret and combine results accurately.
- **B:** Hallucinates certainty in place of preserving the original information.
- **C:** Wasteful duplication and introduces fresh interpretation errors; the upstream agent already had the source in hand.
- **D:** Style does not encode the missing facts (date, source).

---

### Q8 | Scenario 1 | TS 5.2 | difficulty: medium

**Stem:** A customer opens a chat with "I'm done with this — connect me to a human now." Your agent currently responds by asking clarifying questions about the underlying issue first, then offers to resolve it. The customer typically becomes more frustrated. What policy change is correct?

A) Continue investigating because the underlying issue may be quickly resolvable and avoid an escalation.
B) Honor the explicit human-agent request immediately by calling `escalate_to_human` without first attempting investigation.
C) Run sentiment analysis on the customer's message; only escalate if anger score exceeds a threshold.
D) Ask the customer to confirm whether they really want a human before escalating.

**Correct:** B

**Explanation:**
- **B (correct):** Explicit customer demand for a human is one of the canonical escalation triggers; the agent should escalate immediately without prior investigation.
- **A:** Ignores the explicit request — exactly the failure pattern being reported.
- **C:** Sentiment-based gating is an unreliable proxy and adds friction to a clear request.
- **D:** Re-asks what the customer already explicitly stated; degrades trust.

---

### Q9 | Scenario 1 | TS 5.2 | difficulty: medium

**Stem:** A customer messages: "This is incredibly frustrating, my package never arrived and I want a refund." Your agent escalates to a human immediately because the customer expressed frustration. The escalation queue is now overflowing with cases the agent could easily resolve via `process_refund`. What is the right behavior?

A) Acknowledge the frustration and offer to process the refund directly; escalate only if the customer reiterates a preference for a human.
B) Always escalate any message containing emotional language to avoid risk.
C) Reply only with neutral text and ignore the emotional content entirely.
D) Send a survey to verify frustration level before deciding whether to escalate.

**Correct:** A

**Explanation:**
- **A (correct):** Frustration alone is not a valid escalation trigger; the right pattern is to acknowledge sentiment while offering resolution within capability, escalating only if the customer reiterates.
- **B:** Sentiment-based escalation is unreliable and floods the human queue with resolvable cases.
- **C:** Failing to acknowledge frustration damages customer experience.
- **D:** Adds delay to a straightforward refund and uses a poor proxy for actual escalation need.

---

### Q10 | Scenario 1 | TS 5.2 | difficulty: hard

**Stem:** A customer requests a price match against a competitor's website. Your written policy covers price adjustments only when the same item is later discounted on your own site; it is silent on competitor matching. The agent currently guesses a 10% goodwill credit. Logs show inconsistent outcomes across similar requests. What is the correct behavior?

A) Have the agent invent a reasonable interpretation case-by-case using its own judgment.
B) Have the agent escalate to a human because the policy is ambiguous on the customer's specific request.
C) Have the agent always refuse the request because the policy doesn't authorize it.
D) Have the agent ask the customer to email the policy team and close the case.

**Correct:** B

**Explanation:**
- **B (correct):** Policy ambiguity or silence on a specific request is a canonical escalation trigger — the agent should hand off to a human authorized to make the call, not improvise.
- **A:** Improvising creates the very inconsistency the logs show.
- **C:** Blanket refusal isn't supported by policy either; it's still the agent inventing an answer.
- **D:** Closes the case without resolution; harms first-contact resolution and customer experience.

---

### Q11 | Scenario 1 | TS 5.2 | difficulty: easy

**Stem:** Your agent calls `get_customer` with the name "John Smith" and receives three matching customer records. Currently the agent picks the most recently active one. Misidentifications happen roughly 5% of the time. What is the appropriate change?

A) Pick the account with the largest order history because it is most likely the active customer.
B) Always pick the most recently created account as a tie-breaker.
C) Have the agent ask the customer for an additional identifier (email, ZIP, order number) to disambiguate.
D) Have the agent guess and apologize after the fact if wrong.

**Correct:** C

**Explanation:**
- **C (correct):** Multiple matches require clarification by asking for additional identifiers, not heuristic selection — exactly the pattern described in the task statement.
- **A & B:** Both are heuristics that produce confident-but-wrong selections, which is the failure mode being observed.
- **D:** Apologizing after a refund went to the wrong account is unacceptable and undermines the agent's value.

---

### Q12 | Scenario 1 | TS 5.2 | difficulty: medium

**Stem:** A new system prompt instruction reads: "Escalate any case where you feel less than 80% confident." After deployment, escalation rates rise to 40%, well above the 8% target, and the human queue is swamped with simple cases. What is the most accurate diagnosis?

A) The confidence threshold is too low; raising it to 95% will fix the volume.
B) Self-reported model confidence scores are unreliable proxies for actual case complexity; escalation should be based on explicit criteria with few-shot examples.
C) The model is correctly escalating because the cases are genuinely complex.
D) Confidence scores need to be averaged over multiple runs to be reliable.

**Correct:** B

**Explanation:**
- **B (correct):** Self-reported confidence is a poor proxy; reliable escalation criteria should be explicit (human request, policy ambiguity, lack of progress) with few-shot examples in the system prompt.
- **A:** Tuning the threshold ignores that the signal itself is unreliable.
- **C:** Doesn't match the observation — most escalated cases were simple.
- **D:** Averaging doesn't fix a fundamentally miscalibrated signal.

---

### Q13 | Scenario 1 | TS 5.2 | difficulty: medium

**Stem:** You want to improve your agent's escalation discipline. Which approach best teaches the agent when to escalate versus resolve autonomously?

A) Add explicit escalation criteria to the system prompt with multiple few-shot examples covering escalate-vs-resolve decisions.
B) Add a single sentence: "Use good judgment about when to escalate."
C) Remove the `escalate_to_human` tool when escalation rates are too high.
D) Have a second model classify each transcript after the fact and override the agent's choice.

**Correct:** A

**Explanation:**
- **A (correct):** Explicit criteria plus few-shot examples covering boundary cases is the documented pattern for teaching escalation decisions.
- **B:** Vague guidance produces highly variable behavior.
- **C:** Removing the tool means cases that genuinely require humans cannot be escalated.
- **D:** Post-hoc override doesn't help the customer in the active session.

---

### Q14 | Scenario 1 | TS 5.2 | difficulty: hard

**Stem:** Across 2,000 sessions, 30% of escalations occur on cases where the agent successfully called all three required tools and the customer never explicitly asked for a human. Reviewing transcripts, the agent escalates because tool calls "felt slow" or it had "doubts." Which intervention most directly addresses this overuse?

A) Add a system-prompt section enumerating the three valid escalation triggers (explicit human request, policy gap, no meaningful progress) with negative examples ruling out hesitation-based escalation.
B) Reduce the latency of tool calls so the agent feels less doubt.
C) Lower the temperature so the agent is more decisive.
D) Switch to a smaller model that escalates less often.

**Correct:** A

**Explanation:**
- **A (correct):** Codifying valid escalation triggers with negative examples (not "doubt," not "slowness") teaches the model the right boundaries.
- **B:** Latency isn't the real cause; the agent's escalation rule is wrong.
- **C:** Temperature changes don't fix the criteria the agent is applying.
- **D:** Model swap doesn't address ill-defined escalation rules.

---

### Q15 | Scenario 3 | TS 5.3 | difficulty: medium

**Stem:** Your web search subagent times out reaching one of three search APIs. Currently it returns the string "search unavailable" to the coordinator, which terminates the workflow. You want the coordinator to recover intelligently. What should the subagent return instead?

A) An empty result list so the coordinator continues without that source.
B) A structured error object including failure type (timeout), the attempted query, any partial results from the other two APIs, and suggested alternative approaches.
C) The raw stack trace from the failing API call.
D) Nothing — silently swallow the error and rely on retries.

**Correct:** B

**Explanation:**
- **B (correct):** Structured error context (type, attempted operation, partial results, alternatives) enables coordinator-level recovery decisions; generic strings hide essential information.
- **A:** Disguises an access failure as a valid empty result — a documented anti-pattern.
- **C:** Stack traces aren't a coordinator-friendly schema and lack the structured fields needed.
- **D:** Silent suppression is the other documented anti-pattern; the coordinator loses the ability to retry or reroute.

---

### Q16 | Scenario 3 | TS 5.3 | difficulty: medium

**Stem:** A document analysis subagent searches an internal corpus for "Q3 2024 revenue for division X" and finds no matching document. Separately, when the same corpus is unreachable due to a network outage, the subagent also returns "no results." The coordinator treats both cases identically and abandons that line of research. What is the right fix?

A) Distinguish access failures (e.g., timeouts) from valid empty results in error reporting so the coordinator can retry only the former.
B) Have the coordinator retry every "no results" response three times to be safe.
C) Always return partial guesses when no results are found, so the coordinator never stalls.
D) Combine both into a single "unknown" status to keep the schema simple.

**Correct:** A

**Explanation:**
- **A (correct):** Distinguishing access failures from valid empty results lets the coordinator choose retry vs. proceed appropriately.
- **B:** Wastes resources retrying genuine empty results and doesn't address the misclassification.
- **C:** Fabricates data — much worse than an empty result.
- **D:** Loses the very distinction the coordinator needs.

---

### Q17 | Scenario 3 | TS 5.3 | difficulty: easy

**Stem:** A research subagent encounters a transient 503 from a single source while two other sources succeed. It propagates a hard error to the coordinator, which aborts the entire research workflow. What is the recommended subagent behavior?

A) Implement local recovery (retry the transient 503 a few times) and only propagate an error to the coordinator if recovery fails — including what was attempted and any partial results.
B) Always propagate every error immediately because the coordinator is the only place errors should be handled.
C) Suppress all 5xx errors silently to keep the workflow moving.
D) Abort the workflow itself from within the subagent to save coordinator cycles.

**Correct:** A

**Explanation:**
- **A (correct):** Subagents should handle transient failures locally and only escalate unresolvable ones with structured context (what was tried, partial results).
- **B:** Causes brittle workflows that fail on every transient blip.
- **C:** Silent suppression hides information the coordinator needs.
- **D:** Subagents shouldn't terminate the larger workflow unilaterally.

---

### Q18 | Scenario 3 | TS 5.3 | difficulty: medium

**Stem:** A synthesis subagent receives findings from five document analysis subagents. Two reported access failures for their assigned topics, and three returned full findings. Currently the synthesis report makes no mention of the gaps and reads as if the picture is complete. What output structure should the synthesis subagent adopt?

A) Drop the failed-source topics entirely from the report so it reads cleanly.
B) Insert a generic disclaimer at the top: "some sources unavailable."
C) Add coverage annotations indicating which findings are well-supported versus which topic areas have gaps due to unavailable sources, with specifics.
D) Have the synthesis agent guess plausible content for the failed-source topics so coverage looks complete.

**Correct:** C

**Explanation:**
- **C (correct):** Coverage annotations distinguishing supported findings from gap areas (with specifics) preserve provenance and let downstream consumers act on the gaps.
- **A:** Dropping coverage gaps hides risk and may produce misleading reports.
- **B:** Generic disclaimers don't tell the reader which topics are affected.
- **D:** Fabrication — the worst option.

---

### Q19 | Scenario 3 | TS 5.3 | difficulty: hard

**Stem:** Your coordinator dispatches eight subagents in parallel. One subagent encounters a permanent authorization error on its data source after exhausting local retries. The other seven succeed. You want the coordinator to make a fully informed decision (e.g., reroute the failed topic to a different source). What should the failing subagent's final message contain?

A) An empty success response so the coordinator can move on.
B) A structured error with failure type (auth-denied), the attempted query, retries already performed, partial results gathered before the failure, and suggested alternative sources.
C) A natural-language apology describing the error in prose.
D) A retry signal asking the coordinator to immediately re-run the same subagent on the same source.

**Correct:** B

**Explanation:**
- **B (correct):** Coordinator recovery requires structured failure-type, attempt, partial-result, and alternative-approach fields.
- **A:** Disguises failure as success — the coordinator cannot reroute.
- **C:** Prose is hard for the coordinator to parse reliably.
- **D:** Asks the coordinator to repeat what already failed permanently.

---

### Q20 | Scenario 3 | TS 5.3 | difficulty: medium

**Stem:** A coordinator currently terminates the entire research workflow when any single subagent returns an error. As a result, even reports needing data from one of six sources fail when an unrelated source is down. What is the principle being violated?

A) Coordinators should retry subagent errors at least ten times before failing.
B) Coordinators should never propagate errors back to users under any circumstances.
C) Terminating entire workflows on a single subagent failure is an anti-pattern; the coordinator should use structured error context to make a localized recovery decision (continue with partial results, reroute, or fail).
D) Subagents should never report errors to the coordinator in the first place.

**Correct:** C

**Explanation:**
- **C (correct):** Whole-workflow termination on isolated failures is a documented anti-pattern; structured errors enable localized recovery decisions.
- **A:** Aggressive blind retry isn't the principle and doesn't help on permanent failures.
- **B:** False — users sometimes need to know about gaps.
- **D:** The opposite of the correct pattern; suppression is the other anti-pattern.

---

### Q21 | Scenario 3 | TS 5.3 | difficulty: easy

**Stem:** Which of the following is a true anti-pattern when a search subagent encounters a backend timeout?

A) Returning an empty result list and labeling the call successful.
B) Returning a structured error with the attempted query and partial results.
C) Retrying once locally before propagating an error.
D) Including suggested alternative sources in the error response.

**Correct:** A

**Explanation:**
- **A (correct):** Silently suppressing errors by returning empty success is the canonical anti-pattern; it hides failures from the coordinator.
- **B, C, D:** All are recommended practices for resilient subagent error handling.

---

### Q22 | Scenario 4 | TS 5.4 | difficulty: medium

**Stem:** An engineer using your Agent SDK is exploring a 600K-line legacy codebase in a single long session. After about 90 minutes of investigation, the assistant begins answering questions like "where is `UserRepository` instantiated?" with answers that reference "typical patterns" rather than the specific classes it discovered earlier. What is the most effective mitigation?

A) Have the agent maintain a scratchpad file with key findings (file paths, class names, call sites) and reference it for subsequent questions.
B) Restart the session each time the agent gives a vague answer.
C) Increase the model temperature so answers are more concrete.
D) Disable Grep so the agent cannot search broadly.

**Correct:** A

**Explanation:**
- **A (correct):** Scratchpad files persist key findings across context degradation and counteract the model's drift toward generic "typical patterns" language.
- **B:** Loses all accumulated state without addressing the underlying degradation.
- **C:** Higher temperature increases variability and often makes hallucination worse, not better.
- **D:** Removes a useful capability without solving the context degradation issue.

---

### Q23 | Scenario 4 | TS 5.4 | difficulty: medium

**Stem:** During a deep exploration of an unfamiliar payment module, the main agent is about to dispatch a sub-investigation: "find all places where refunds are issued and trace upstream dependencies." Why is delegating this to a subagent preferable to having the main agent perform the search inline?

A) Subagents run on different hardware so they are faster than the main agent.
B) Subagents isolate verbose exploration output (long file dumps, traces) from the main agent, which preserves the main agent's context for high-level coordination.
C) Subagents have access to network APIs the main agent does not.
D) Subagents reduce token cost to zero because their input is not billed.

**Correct:** B

**Explanation:**
- **B (correct):** Subagent delegation isolates verbose discovery output so the coordinating agent's context remains focused on high-level synthesis.
- **A:** Hardware isn't the rationale; context isolation is.
- **C:** Capability is unchanged unless explicitly granted; this isn't the reason.
- **D:** Subagent input is still billed; the benefit is context, not cost.

---

### Q24 | Scenario 2 | TS 5.4 | difficulty: easy

**Stem:** During a long Claude Code session refactoring a monorepo, the context window fills with verbose discovery output (file listings, grep results, long compile errors). The model starts re-reading files it already explored and giving inconsistent answers. Which Claude Code tool is intended to address this?

A) The `/init` command to regenerate `CLAUDE.md`.
B) The `/compact` command to reduce context usage by compressing prior turns.
C) Switching out of plan mode mid-session.
D) Disabling the Bash tool to reduce tool calls.

**Correct:** B

**Explanation:**
- **B (correct):** `/compact` is the documented mechanism for reducing context usage during extended sessions when context fills with verbose discovery output.
- **A:** `/init` initializes project documentation; it doesn't reduce active session context.
- **C:** Plan mode is unrelated to context compaction.
- **D:** Removes a useful tool without addressing the context bloat.

---

### Q25 | Scenario 4 | TS 5.4 | difficulty: medium

**Stem:** A coordinator agent spawns three exploration subagents in sequence. Between phase 1 and phase 2 of the investigation, the coordinator wants the phase-2 subagents to benefit from what was discovered in phase 1. What is the best technique?

A) Pass each phase-2 subagent the full raw transcript of phase 1 as its initial context.
B) Summarize the key findings of phase 1 into a structured note and inject that summary into each phase-2 subagent's initial context.
C) Skip the summary and let phase-2 subagents re-discover everything from scratch to be safe.
D) Save phase 1 outputs to disk but don't reference them in phase 2.

**Correct:** B

**Explanation:**
- **B (correct):** Summarizing phase-1 findings into structured notes and injecting them into phase-2 prompts both saves context and ensures continuity.
- **A:** Floods subagent context with low-density material.
- **C:** Wastes time and risks inconsistent rediscovery.
- **D:** Wastes the prior work by not surfacing it.

---

### Q26 | Scenario 4 | TS 5.4 | difficulty: hard

**Stem:** Your long-running exploration system needs to survive process crashes. A crash mid-investigation currently loses all progress and requires restarting from scratch. What design enables clean crash recovery?

A) Run all exploration in-memory with no checkpointing; rely on quick re-runs.
B) Have each agent export structured state (scratchpads, findings, partial outputs) to a known location and maintain a manifest the coordinator loads on resume and injects into each agent's prompt.
C) Periodically take screenshots of the terminal output so users can manually resume.
D) Increase RAM so the OS does not OOM-kill the process.

**Correct:** B

**Explanation:**
- **B (correct):** Structured agent state exports plus a coordinator-loaded manifest is the documented crash-recovery design.
- **A:** Doesn't survive crashes by design.
- **C:** Screenshots aren't machine-readable state.
- **D:** Doesn't help with non-OOM crashes (power loss, process kill, etc.).

---

### Q27 | Scenario 4 | TS 5.4 | difficulty: medium

**Stem:** A senior engineer asks the assistant: "Map all 17 services in this repo and identify cross-service contracts." The main agent attempts this inline and within an hour starts contradicting itself about which services exist. What is the best decomposition?

A) Spawn a subagent per service to investigate locally, have each export a structured findings file, and have the main agent read the manifests to coordinate the cross-service synthesis.
B) Increase the main agent's context window and try again in one pass.
C) Have the engineer narrow scope to two services.
D) Run the same prompt three times and merge answers.

**Correct:** A

**Explanation:**
- **A (correct):** Subagent-per-unit-of-work with structured exports plus coordinator synthesis is the canonical pattern for large codebase exploration.
- **B:** Larger contexts don't solve attention dilution.
- **C:** Shifts burden to the user without solving the design issue.
- **D:** Repetition doesn't address the underlying coordination problem.

---

### Q28 | Scenario 4 | TS 5.4 | difficulty: easy

**Stem:** True or false-style: Which of the following is a documented sign of context degradation in extended Claude exploration sessions?

A) The agent's responses become shorter than the user's questions.
B) The agent begins referencing "typical patterns" or "common implementations" rather than specific classes and files it discovered earlier.
C) The agent occasionally pauses for several seconds before answering.
D) The agent stops using the Read tool entirely and only uses Grep.

**Correct:** B

**Explanation:**
- **B (correct):** Drift to generic "typical patterns" language instead of specific discovered references is the documented degradation signal.
- **A:** Response length isn't a reliable marker.
- **C:** Latency is unrelated to context quality.
- **D:** Tool selection patterns aren't the documented signal.

---

### Q29 | Scenario 6 | TS 5.5 | difficulty: medium

**Stem:** Your extraction pipeline reports 97% accuracy aggregated across all document types. The team plans to auto-approve every high-confidence extraction. Before doing so, a careful reviewer points out that aggregate accuracy may hide poor performance on specific document types. What evaluation step should you add?

A) Re-run the same overall benchmark with a larger sample to tighten the 97% estimate.
B) Analyze accuracy by document type and by field, verifying consistent performance across all segments before reducing human review.
C) Trust the 97% number because aggregate measures are designed to capture overall performance.
D) Switch to a different model and compare aggregate numbers.

**Correct:** B

**Explanation:**
- **B (correct):** Aggregate accuracy can mask poor per-segment performance; per-document-type and per-field analysis is the documented prerequisite for automating high-confidence extractions.
- **A:** Tightens precision on a metric that already obscures the problem.
- **C:** Exactly the assumption the reviewer is warning against.
- **D:** Doesn't fix the segment-blind evaluation.

---

### Q30 | Scenario 6 | TS 5.5 | difficulty: medium

**Stem:** You have automated 80% of extractions because they were flagged "high confidence" by the model. Six months later, a new document template enters production and the system's accuracy on those documents quietly drops, but the aggregate confidence numbers look fine. What sampling strategy would have caught this earlier?

A) Stratified random sampling of high-confidence extractions for ongoing error rate measurement and novel pattern detection.
B) Reviewing only the lowest-confidence extractions because those are most likely to be wrong.
C) Reviewing every 1,000th extraction in chronological order.
D) Re-running each extraction twice and reviewing only the disagreements.

**Correct:** A

**Explanation:**
- **A (correct):** Stratified random sampling of the automated bucket is precisely what detects novel error patterns and accuracy regressions over time.
- **B:** Only sampling low-confidence misses regressions in the automated bucket.
- **C:** Chronological sampling doesn't stratify by document type or field.
- **D:** Two-run agreement doesn't catch systematic novel-template errors.

---

### Q31 | Scenario 6 | TS 5.5 | difficulty: hard

**Stem:** You want field-level confidence scores from the model so you can route low-confidence fields to humans. The model already produces a vague "overall confidence: high/medium/low" string. What additional step is required before using these scores to set review thresholds?

A) Trust the categorical labels as-is.
B) Have the model output a per-field numeric confidence and calibrate review thresholds using a labeled validation set.
C) Tune confidence outputs by giving the model fewer few-shot examples.
D) Replace the model with a non-LLM rules engine.

**Correct:** B

**Explanation:**
- **B (correct):** Per-field numeric confidences calibrated against labeled validation data are the documented pattern for setting reliable review thresholds.
- **A:** Uncalibrated categorical labels are not reliable routing signals.
- **C:** Few-shot count doesn't calibrate confidence semantics.
- **D:** Rules engines have their own brittleness and don't address calibration.

---

### Q32 | Scenario 6 | TS 5.5 | difficulty: easy

**Stem:** You have limited human reviewer capacity. Which extractions should be prioritized for human review?

A) Random selections regardless of model output.
B) Extractions with low model confidence or sourced from ambiguous/contradictory documents.
C) Extractions on the highest-volume document types only.
D) The most recent extractions, regardless of confidence.

**Correct:** B

**Explanation:**
- **B (correct):** Routing low-confidence or source-ambiguity cases to humans concentrates review where it adds the most value.
- **A:** Wastes reviewer capacity on likely-correct cases.
- **C:** Volume alone is not a quality signal.
- **D:** Recency is not a quality signal.

---

### Q33 | Scenario 6 | TS 5.5 | difficulty: medium

**Stem:** After three months, your team wants to reduce human review on "Form 1099" extractions from 100% to 0% because aggregate accuracy is 98%. A reviewer pushes back. What is the most defensible next step before reducing review?

A) Reduce immediately because 98% exceeds your 95% target.
B) Verify performance specifically on Form 1099, broken down by field (e.g., payer EIN vs. recipient address), and confirm each field meets the target before reducing review on that document type.
C) Reduce review on Form 1099 to 50% as a compromise without further analysis.
D) Replace human review with an additional LLM pass.

**Correct:** B

**Explanation:**
- **B (correct):** Field- and document-type-level validation is required before reducing review; otherwise an aggregate average can hide a weak field like "payer EIN."
- **A:** Acts on aggregate numbers that may mask field-level weakness.
- **C:** Arbitrary midpoint without evidence.
- **D:** LLM self-review without calibration provides little additional safety.

---

### Q34 | Scenario 6 | TS 5.5 | difficulty: medium

**Stem:** Your QA team plans to sample 1% of automated extractions for human review monthly. They propose to sample uniformly at random from all extractions. Why is stratified sampling preferred?

A) Stratified sampling is faster to implement than uniform sampling.
B) Stratified sampling ensures coverage across document types and confidence bins, increasing the chance of detecting novel error patterns concentrated in specific strata.
C) Stratified sampling reduces the total review burden by more than half.
D) Uniform sampling is statistically invalid for this purpose.

**Correct:** B

**Explanation:**
- **B (correct):** Stratification ensures each segment is represented and that novel error patterns confined to one stratum are detected.
- **A:** Often comparable in effort; not the reason.
- **C:** Same sample size; not a cost claim.
- **D:** Overstates the issue; uniform is valid but less efficient at detecting segment-specific drift.

---

### Q35 | Scenario 6 | TS 5.5 | difficulty: medium

**Stem:** A new document type appears in production. Your extraction system has never seen it but reports high confidence on the fields. Aggregate accuracy is unaffected because the new type is only 0.3% of volume. What design choice surfaces this risk soonest?

A) Increase aggregate sample sizes.
B) Stratified sampling that ensures the new document type is reviewed at a higher rate (or at minimum at all) and that high-confidence extractions in that stratum are checked for novel error patterns.
C) Auto-approve all high-confidence extractions to free up reviewer time.
D) Wait until customer complaints arrive.

**Correct:** B

**Explanation:**
- **B (correct):** Stratified sampling that explicitly covers low-volume strata catches issues that aggregate metrics miss.
- **A:** Larger overall samples still under-represent rare strata.
- **C:** Hides the problem.
- **D:** Reactive, not proactive.

---

### Q36 | Scenario 3 | TS 5.6 | difficulty: medium

**Stem:** Your research subagents return prose findings like: "The market grew 14% last year." The synthesis agent compresses this into a final report claiming "the market grew 14% last year" — with no link to which source said this. Customers later cannot verify claims. What change preserves provenance?

A) Require subagents to output structured claim-source mappings (source URL, document name, relevant excerpt) that the synthesis agent must preserve and merge into its output.
B) Increase the model size so it remembers sources implicitly.
C) Have the synthesis agent insert citations of its own choosing where it thinks they apply.
D) Drop the offending statistics from the report.

**Correct:** A

**Explanation:**
- **A (correct):** Structured claim-source mappings preserved through synthesis are the documented mechanism for maintaining provenance.
- **B:** Larger models still drop source attribution during summarization.
- **C:** Inventing citations is worse than missing ones.
- **D:** Sacrifices the report's utility instead of fixing the design.

---

### Q37 | Scenario 3 | TS 5.6 | difficulty: hard

**Stem:** Two reputable sources give different figures for the same metric: Source A reports 1.2 million units, Source B reports 1.05 million units. Both are credible. The synthesis agent currently picks one figure and writes it as fact. What is the recommended handling?

A) Average the two values and report the mean as the canonical figure.
B) Annotate the conflict with source attribution (e.g., "Source A reports 1.2M; Source B reports 1.05M") and let the reader (or coordinator) decide how to reconcile, rather than arbitrarily selecting one.
C) Pick the most recent source by default and discard the other.
D) Drop the metric entirely to avoid disagreement.

**Correct:** B

**Explanation:**
- **B (correct):** When credible sources disagree, the synthesis agent should annotate the conflict with attribution rather than arbitrarily select a value.
- **A:** Averaging fabricates a third number not reported by either source.
- **C:** Recency may not be the right tie-breaker without context; the user should choose.
- **D:** Suppresses useful information.

---

### Q38 | Scenario 3 | TS 5.6 | difficulty: medium

**Stem:** A subagent reports "unemployment was 4.2%" and another subagent reports "unemployment was 3.7%." The synthesis agent flags this as a contradiction. Investigation shows the first figure is from 2022 and the second is from Q1 2024. The conflict is illusory — it is a temporal difference. What requirement on subagent outputs would have avoided this?

A) Require subagents to include publication and data-collection dates in their structured outputs so temporal differences are not misread as contradictions.
B) Require subagents to choose only the most recent data point and discard older sources.
C) Increase the synthesis model's temperature to encourage nuance.
D) Have the synthesis agent assume any disagreement is temporal.

**Correct:** A

**Explanation:**
- **A (correct):** Publication and collection dates in structured outputs let downstream agents correctly interpret temporal differences vs. genuine contradictions.
- **B:** Discards useful historical context.
- **C:** Temperature doesn't address missing metadata.
- **D:** Wrong default assumption — many disagreements are genuine.

---

### Q39 | Scenario 3 | TS 5.6 | difficulty: medium

**Stem:** Your final research report covers a contested topic. The synthesis agent currently flattens all findings into a single uniform prose narrative. Reviewers complain they cannot tell which claims are well-established versus contested. What structural change is recommended?

A) Structure the report with explicit sections distinguishing well-established findings from contested ones, preserving original source characterizations and methodological context.
B) Bold every contested claim while leaving established ones unformatted.
C) Remove all contested claims from the report.
D) Mark the entire report as "draft" to indicate uncertainty.

**Correct:** A

**Explanation:**
- **A (correct):** Explicit sections separating established from contested findings, with preserved source context, is the recommended structure.
- **B:** Formatting hints aren't a substitute for explicit sectioning and source attribution.
- **C:** Erases relevant information.
- **D:** Lumps everything together rather than distinguishing.

---

### Q40 | Scenario 6 | TS 5.6 | difficulty: hard

**Stem:** A document analysis subagent finds two conflicting values for "total liabilities" within the same set of source documents (one figure from an audited 10-K, another from a press release). Currently it picks one and forwards it. What is the recommended behavior?

A) Complete the analysis output with both conflicting values explicitly annotated (source, document type, date), letting the coordinator decide how to reconcile before passing to synthesis.
B) Discard the lower of the two values to be conservative.
C) Discard the press release value because press releases are less authoritative in general.
D) Average the two values and emit the average.

**Correct:** A

**Explanation:**
- **A (correct):** Subagents should surface conflicts with full annotation so the coordinator (which has broader context) can decide on reconciliation policy.
- **B & C:** Arbitrary local heuristics that may not match the coordinator's policy.
- **D:** Fabricates a non-existent figure.

---

### Q41 | Scenario 3 | TS 5.6 | difficulty: medium

**Stem:** Your synthesis output currently converts everything — quarterly financial data, breaking news, and technical performance benchmarks — into uniform prose paragraphs. Readers find the financial sections hard to scan and the technical sections feel narrative rather than precise. What recommendation applies?

A) Render different content types appropriately: financial data as tables, news as prose, technical findings as structured lists — rather than forcing a uniform format.
B) Render everything as a single long table to maximize density.
C) Render everything as bullet points to maximize scannability.
D) Apply a single uniform Markdown style and let readers reformat manually.

**Correct:** A

**Explanation:**
- **A (correct):** Matching the rendering format to content type (tables for financials, prose for news, structured lists for technical) preserves the natural shape of each finding.
- **B, C:** Single-format approaches lose the affordances appropriate to each content type.
- **D:** Pushes the work to readers without solving the design problem.

---

### Q42 | Scenario 6 | TS 5.6 | difficulty: easy

**Stem:** Your extraction system processes multi-source filings and synthesizes summary fields. Auditors require that each summary field can be traced back to which source document supplied it. The current pipeline summarizes findings without preserving claim-source mappings. What is the right fix?

A) Have the extraction subagents output structured claim-source mappings (source document, page or section, excerpt) that the downstream synthesis step preserves alongside each output field.
B) Add a free-text "sources used" paragraph at the bottom of each report.
C) Tell auditors that LLM-based extraction inherently cannot preserve provenance.
D) Re-run the extraction whenever audit questions are asked.

**Correct:** A

**Explanation:**
- **A (correct):** Structured claim-source mappings preserved through synthesis are the documented mechanism for auditable provenance.
- **B:** A trailing paragraph doesn't tie individual fields to sources.
- **C:** False — well-designed pipelines preserve provenance.
- **D:** Wasteful and still doesn't preserve the original mappings systematically.


