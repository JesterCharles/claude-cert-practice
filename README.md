# CCA-F Cert Practice Tracker

Team prep hub for the **Claude Certified Architect — Foundations** exam.

**Target sitting:** Thursday, May 29, 2026.

## 🧠 New: 210-question Quiz Mode (default)

Open `index.html` (or your deployed GitHub Pages URL) and the **Quiz mode** is the landing view:

- **30 questions per day**, 6 from each of the 5 exam domains. No question repeats in a 7-day cycle (210 unique Q's / 30 per day = exactly one full week).
- **Each question** is scenario-anchored (1–6 from the official guide), tagged with its task statement and difficulty, with explanations for the correct answer **and** every distractor.
- **Skill graph** above the quiz tracks per-domain mastery (correct/seen, color-coded). Domains scoring below 60% get a "target these next" callout.
- **Focused practice** — click "Practice this domain" on any skill bar to run a 10-Q quiz on that single domain instead of the mixed daily 30.
- **Per-teammate slots** — enter your name in the input at the top to keep your progress separate from teammates on the same browser.
- **Progress export/import** — buttons at the bottom save your progress as a JSON file. Re-import on a different device or after clearing cookies. This is how progress survives browser changes on a static-hosted site (see "Progress persistence" below).

## 🗓️ Daily plan mode (fallback)

If you'd rather follow a prescribed 18-day study schedule than self-direct via quizzes, switch to **Daily plan** view at the top. The plan runs May 12 → May 29 with a daily core (1 hr) + stretch (+30 min) split across the exam's five domains, linked to canonical resources.

- [00-start-here/study-plan.md](./00-start-here/study-plan.md) — the canonical study plan
- [00-start-here/resources.md](./00-start-here/resources.md) — tiered resource list

## 📚 Question bank

- [questions.md](./questions.md) — the canonical 210-question bank, browsable as markdown. Each question shows scenario / task statement / difficulty / 4 options / correct answer / explanations for all four options.
- [questions.js](./questions.js) — same content as a JS data structure (loaded by `index.html`).

## Progress persistence

This is a static site, so progress lives in your browser's `localStorage`. That means:

- Switching browsers or clearing cookies wipes progress.
- Two teammates on the same browser need different "Name" entries to keep stats separate.
- Use **Export progress (JSON)** periodically to save a backup. Use **Import** to restore on another device or after a clear.

If you want true cross-device sync, you'd need a backend, which is out of scope for a static GitHub Pages deployment.

## Conventions

- All dates are 2026.
- Domains: D1 Agentic Architecture (27%), D2 Tool Design & MCP (18%), D3 Claude Code (20%), D4 Prompt Engineering (20%), D5 Context & Reliability (15%).
- If you spot a wrong/sloppy question, the file to edit is `questions.md` (canonical); regenerate `questions.js` from it.
