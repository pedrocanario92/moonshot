---
name: guardian-review
description: Run the Guardian UX audit suite (accessibility, heuristics, design-system, content, interaction, responsive) on a screen and produce a joint PASS/WARN/FAIL report. Use proactively before shipping or finalizing any UI work, and whenever the user asks to review, audit, QA, or sanity-check a screen, page, or component, or asks "is this ready to ship / good to go?".
when_to_use: Building, changing, reviewing, or shipping a UI/screen/page/component; pre-deploy UX/accessibility/design checks; "review this UI", "audit my page", "is this ready to ship", "check this before I push".
---

# Guardian UX Review

Orchestrate the six Guardian UX agents into one pre-ship review of a user interface, then hand a verdict report to the human. The guardians are **read-only process gates** — they audit and recommend; the human decides ship.

## When this skill applies

Activate when the user is finishing or evaluating UI work — e.g. "review this screen", "audit my page", "is this ready to ship?", "QA this component", or right after building/changing a UI and before declaring it done. If the user only wants one dimension (e.g. "just check accessibility"), run that single guardian instead of the full suite.

## Steps

1. **Identify the target.** A running URL, a file/component path, or a described screen. If unclear, ask once — and also ask which states (loading/empty/error), breakpoints, or personas matter. Don't guess the target.

2. **Prepare evidence.** If it's a web UI and a browser-preview tool is available, start/confirm a preview server so guardians can inspect the rendered DOM, computed styles, and behaviour. If source-only, proceed statically and label it so.

3. **Fan out — all six guardians in parallel.** In one message, dispatch six subagents via the Agent tool: `accessibility-guardian`, `heuristics-guardian`, `design-system-guardian`, `content-guardian`, `interaction-guardian`, `responsive-guardian`. Pass each the same target plus relevant states/breakpoints. (For a single-dimension request, dispatch only the matching guardian.)

4. **Assemble the joint report** — concatenate the sections with attribution, without altering any verdict. Add a banner:
   - Total **load-bearing FAILs** (the ship gate) · advisory FAILs · WARNs.
   - Per-guardian one-line roll-up.
   - **Overall recommendation: SHIP / SEND BACK / NEEDS HUMAN JUDGEMENT** — `SEND BACK` if any load-bearing FAIL exists.

5. **Persist + hand back.** Write the full report to `reports/guardian-audit-<target-slug>-<YYYY-MM-DD>.md` in the user's project (create `reports/` if needed). Present the banner and the report path.

## Hard rules

- **Never self-certify ship.** The report is a proposal to the human; the human approves ship.
- **Guardians never modify code.** If the user wants fixes applied, that is a separate, explicit step you take *after* showing the report — never silently, and never by the guardians.
- **Don't soften FAILs.** Report load-bearing FAILs plainly even under time pressure.
- **Single source of truth for criteria** lives in each guardian's own definition — don't re-derive or override their verdicts here.
