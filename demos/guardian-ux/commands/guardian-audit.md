---
description: Run all six Guardian UX audits (accessibility, heuristics, design-system, content, interaction, responsive) against a screen and produce one joint PASS/WARN/FAIL report before ship.
argument-hint: "[url | file | screen description] (optional — will ask if omitted)"
---

Run a full **Guardian UX audit** on the target below and produce a single consolidated report.

**Target:** $ARGUMENTS

If the target above is empty or ambiguous, ask the user **once** what to audit — a running URL, a file/component path, or a described screen — and which states/personas matter. Do not guess.

## Procedure

1. **Prepare evidence.** If the target is a web UI and a browser-preview tool is available, start/confirm a preview server so the guardians can inspect the rendered DOM. If it's source-only, note that the audit will be static.

2. **Fan out — run all six guardians in parallel.** In a single message, dispatch six subagents via the Agent tool, one each:
   - `accessibility-guardian`
   - `heuristics-guardian`
   - `design-system-guardian`
   - `content-guardian`
   - `interaction-guardian`
   - `responsive-guardian`

   Give every guardian the **same target** (URL/file/description), the relevant states/breakpoints/personas, and whether rendered evidence is available. Each returns its own section in the standard verdict format.

3. **Assemble the joint report.** Concatenate the six reports with attribution (do not edit their verdicts — guardians do not negotiate). Add a top banner:
   - Total **load-bearing FAILs** across all guardians (this is the ship gate).
   - Total advisory FAILs and WARNs.
   - A per-guardian one-line roll-up.
   - **Overall recommendation: SHIP / SEND BACK / NEEDS HUMAN JUDGEMENT** — `SEND BACK` if any load-bearing FAIL exists anywhere.

4. **Write it to disk.** Save to `reports/guardian-audit-<target-slug>-<YYYY-MM-DD>.md` in the user's project (create `reports/` if absent). Use today's date if known; otherwise ask or label `undated`.

5. **Hand back to the human.** Present the banner + the path to the full report. **Never declare the screen shipped or approved** — the report is a proposal; ship is the human's decision. If there are load-bearing FAILs, list them plainly and offer to fix them (in a separate, explicit step — the guardians themselves never modify code).
