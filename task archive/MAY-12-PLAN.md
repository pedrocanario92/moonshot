# Today's plan — May 12 2026

5-hour working session. Extracted from the May 12 handoff block in `ITERATION-LOG.md` (the iteration log retains all May 11 history; this file is the active working plan for May 12).

**Budget at a glance.** Tasks 1–5 fit inside the 5-hour budget. Task 6 (the one-shot prompt experiment) is independent and runs in a separate fresh chat — not slotted in today.

---

## Read-in files (do this first, before starting any task)

1. `ITERATION-LOG.md` at the repo root — full history across all sessions through the afternoon of May 11 2026. Read end-to-end.
2. `WAY-OF-WORKING.md` at the same repo root — file map and project intent.

Then glance at:

- `moonshot prototype/moonshot-prototype.html` — the active working prototype. Being promoted to the shipped deliverable today.
- `moonshot presentation/moonshot.html` — the strategy doc + agent showcase + embedded prototype viewer. The ship-ready deliverable. Note structure and Agents page.
- `moonshot presentation/wms-unified-figma-sidebar.html` — the OLD prototype currently embedded in moonshot.html's Prototype tab.
- `agents/slotting-agent.md`, `agents/equipment-agent.md`, `agents/quality-agent.md` — three Domain agents adapted from Allen Oleksak's demo. Specced but not yet shown in moonshot.html.
- `agents/meta/warehouse-life-agent.md` — the sim engine. Gets its own card on the Agents page today.
- `agents/guardian/heuristics-agent.md`, `agents/guardian/accessibility-agent.md` — the two Guardian agents. Each gets its own card on the Agents page today (in addition to the existing Stage 3 Guardian section — confirm whether that section also needs updates).

---

## Time budget — 5 hours, sequenced

| Block | Window | Task |
|---|---|---|
| Context load | T+0 → T+15 (15 min) | Read-in files |
| Block 1 — Foundation | T+15 → T+45 (30 min) | **Task 1** — Dead-code cleanup of the prototype |
| Block 2 — Swap | T+45 → T+60 (15 min) | **Task 2** — Replace the embedded prototype in `moonshot presentation/` |
| Block 3 — Agent cards | T+60 → T+120 (60 min) | **Task 3** — Add six new agent cards to the Agents page |
| Block 4 — Marquee build | T+120 → T+275 (155 min, ~2h35m) | **Task 4** — Build the "How the agents work" page |
| Block 5 — Verification | T+275 → T+300 (25 min) | **Task 5** — Full verification pass |

**Why this order.** Cleanup first so the prototype that gets swapped into the presentation folder is the cleaner version. Swap second so the moonshot.html iframe is already pointing at the current prototype before we add agent cards or a new page around it. Agent cards before the marquee because they're shorter, lower-risk edits to moonshot.html — a warmup that lets us pattern-match the existing card/pill conventions before the bigger build. Marquee page in the longest block — it's the highest-impact task and needs the most uninterrupted time. Verification last as a sweep across everything.

**Sub-breakdown for the marquee task (Block 4, 155 min):**

| Sub-step | Minutes | What |
|---|---|---|
| 4a | 15 | Audit existing moonshot.html nav + tab structure; plan integration of the new "How the agents work" tab |
| 4b | 30 | Conveyor diagram SVG — geometry, both tracks, eight stations on the UX track as `<g>` groups, prompt + prototype endpoints |
| 4c | 20 | Stamps/badges (UX track) + red flags (blind track) |
| 4d | 25 | Hover-card content sourced from `/agents/` specs + interaction CSS (Tab + focus-visible) |
| 4e | 25 | Sequential reveal animation (CSS keyframes + IntersectionObserver, blind track flash 200ms, UX track stagger over 1.5s) |
| 4f | 15 | Receipts stat strip (six tiles) |
| 4g | 10 | Closing pull-quote |
| 4h | 15 | Headline + kicker + nav tab wiring + final polish |

---

## Working style the user expects

- **Out of auto mode by default.** Ask clarifying questions for any ambiguous decision and get explicit go-ahead before editing. Answering clarifying questions is NOT the same as saying "start."
- **Direct, no-fluff updates.** Diagnose root causes, not symptoms.
- **Verify in the browser via the preview tool**, not by assertion.
- **Never claim a screenshot is fine if the screenshot tool timed out** — say "couldn't capture" and use DOM / geometry checks instead.
- **Never overwrite `moonshot presentation/` files without explicit go-ahead.** That folder is the shipped artifact; treat it with the same caution as a production deploy.

---

## The tasks in detail

### 1. Dead-code cleanup of `moonshot prototype/moonshot-prototype.html`

Conservative pass to shrink the prototype before the rest of today's work — improves the shipped artifact, makes Form A of the one-shot prompt experiment (task 6) easier, and surfaces any legacy patterns we should retire. **In scope:**

- Remove CSS rules whose class selectors are not referenced anywhere in the HTML or JS. Likely candidates: Phase 3 single-focus stage classes superseded by the dashboard pivot, the `.l-alcove` family superseded by `renderKpiStrip`, original CLI-only layout classes superseded by the full-bleed CLI.
- Remove debug `console.log` calls and render-count instrumentation left over from the Session 5 flicker hunts.
- Remove commented-out HTML / CSS / JS blocks longer than two lines.
- Remove any `DATA.*` properties that have been fully replaced by `WAREHOUSE.*` equivalents and are not read anywhere.

**Out of scope** (do not do these in this pass — flag them for a separate future session if you find candidates):

- Renaming functions or files
- Restructuring view code
- Consolidating styles into new tokens
- Splitting the file into sections with comment dividers

**Verification:** open `moonshot prototype/moonshot-prototype.html` in the browser preview before and after. The prototype must render and behave identically — same views, same KPI sparklines, same modal, same banner, same sim timeline. Run through the 3-minute demo. Re-run the Heuristics + Accessibility Guardian audits — verdict should remain SHIP.

### 2. Replace the embedded prototype with the current working copy

Inside `moonshot presentation/`, the file `wms-unified-figma-sidebar.html` is what loads in moonshot.html's Prototype tab via an iframe `<src>`. That file is now several months stale relative to `moonshot prototype/moonshot-prototype.html`. The ask:

- Copy the working prototype into the presentation folder so the iframe picks it up. The simplest path is to overwrite `moonshot presentation/wms-unified-figma-sidebar.html` with the contents of `moonshot prototype/moonshot-prototype.html`. **Confirm with the user before clobbering** — that file is the previously-shipped artifact; offer a quick `git diff` view or a copy to `archive/`.
- After copying, verify: open `moonshot presentation/moonshot.html` in a browser, click the Prototype tab. The new manager dashboard, Warehouse dashboard, dark terminal, KPI sparklines, and floating banner should all render inside the iframe with no console errors and no broken paths.
- Confirm the iframe `src` in `moonshot presentation/moonshot.html` still resolves. If the filename needs to change (e.g., rename to `moonshot-prototype.html` and update the iframe `src`), that's a valid alternative — discuss with the user first.

### 3. Add the six new agents from the morning session to the moonshot.html Agents page

`moonshot presentation/moonshot.html`'s Agents page currently shows the original WMS topology (Lead, Shift Intelligence, Pick Path, Labor, Order Priority, Carrier, Exception, plus BI/Innovation/Opportunity). The May 11 morning session produced six new agents that exist as full specs in `/agents/` but have no cards yet in the presentation. They fall into two groups:

#### Group A — Three Domain agents adapted from Allen Oleksak's demo

Source: `original references/allen-demo.html`. **Credit Allen Oleksak by name on each of these three cards** (match whatever attribution convention the existing cards use — verify before writing copy; if there's no precedent, propose a small "Adapted from Allen Oleksak's demo" line that fits the existing tile aesthetic and confirm with the user).

- **Slotting Agent** — replenishment scheduling, bin assignment, fast-mover re-slotting (`agents/slotting-agent.md`)
- **Equipment Agent** — forklifts, AMRs, conveyors, charge cycles, equipment faults (`agents/equipment-agent.md`) — NEW domain, was not in the original topology
- **Quality Agent** — damage, quality holds, cycle counts (`agents/quality-agent.md`) — carved out of the old Exception agent's scope

Add three cards to the Domain experts section using the existing `.card` + `.pill--domain` pattern (see `wms-unified-figma-sidebar.html` and `moonshot.html` for the conventions). Match the existing tone and length — one-line description, key responsibilities, same visual layout as Pick Path / Labor / Order Priority. Source language from the agent spec markdown files; don't invent new copy.

Also confirm the **Exception Agent** card has been rescoped to "transactional errors only" — Quality now owns damage / holds / cycle counts. Update the existing Exception card description if it still mentions damage or holds.

#### Group B — Three new agents we built ourselves this morning

These are not in Allen's demo — they're original to this project. No external credit needed; they belong to the team.

**Meta agent (one):**

- **Warehouse Life Agent** — scripted-causal simulation engine; not in the WMS topology; drives the events the WMS agents react to (`agents/meta/warehouse-life-agent.md`).

Reverses the earlier "intentionally backstage" stance — give it its own card. Check `moonshot.html` for an existing non-domain pill class (something like `.pill--meta` or similar). If none exists, propose a new pill style that's visually distinct from `.pill--domain` and confirm before adding. Source copy from the spec.

**Guardian agents (two):**

- **Heuristics Agent** — Nielsen Norman + Hick + Miller + cognitive-load audit (`agents/guardian/heuristics-agent.md`).
- **Accessibility Agent** — WCAG 2.2 (AA load-bearing, AAA advisory) (`agents/guardian/accessibility-agent.md`).

These already appear in `moonshot.html`'s Stage 3 Guardian section. The new ask is to give each its own card on the Agents page too, in addition to whatever Stage 3 treatment exists. Use whatever pill convention `moonshot.html` already uses for Guardian/process-gate agents; if none, propose one and confirm. Source copy from the specs.

### 4. New page in `moonshot presentation/moonshot.html` — "How the agents work"

**This is the marquee task.** Audience: a senior UX executive presenting upward to C-level. **30-second comprehension or it fails.** Must feel like a B2C product-launch landing page — visual, kinetic, attention-grabbing, motion-anchored — but stay inside the existing moonshot.html visual language (brand-yellow `#CADF35`, soft-dark terminal accent `#1E2530`, universal tile shadow `0 2px 30px rgba(0,0,0,0.15)`, Noto Sans + Noto Sans Mono, 8px tile radius). Must NOT look bolted-on; must read as the natural climax of the deck.

The differentiating claim — **the page's headline** — is that a UX-owned, agent-driven build produces structurally better interfaces than a PM prompting an LLM blind. The main piece below dramatizes that by walking the viewer through what each agent does between "prompt arrives" and "prototype delivered," with UX-led agents distinctly highlighted.

#### Placement

**A new top-level page in the moonshot.html navigation, titled "How the agents work."** Sits alongside Home, The Agents, The Prototype, etc. Match the existing nav pattern (active/inactive states, typography, hover). Do NOT merge into any existing tab. This page does one job.

#### Page structure — four blocks, top to bottom

##### Block 1 — Headline + kicker

- **Headline** in Noto Sans, weight 700-800, 48-64px depending on viewport. Working draft: *"AI without UX is slop. We built the UX layer."* (Confirm with user before building.)
- **One-line kicker** beneath, 18-20px, neutral gray. Working draft: *"From prompt to prototype, here's what eleven agents and two audits did that prompting blind can't."* (Confirm before building.)

##### Block 2 — The conveyor diagram (the main piece)

A horizontal, full-width SVG composition. Time flows left → right. Two stacked tracks share the same starting point (a `prompt` document icon entering on the left) and finish line (a `prototype` document icon exiting on the right). The conveyor metaphor reinforces the WMS subject visually.

**Top track — "Prompting blind."** Sparse. The prompt slides straight to delivery with no intermediate stations. The prototype that exits has red flags stuck to it:

- ❌ `alert("Approved!")` popups
- ❌ Three parallel queues (cognitive load trap)
- ❌ Single Cautious↔Bold autonomy slider
- ❌ Low-contrast text
- ❌ "Handled by agents" noise feed

**Bottom track — "UX-owned agent system."** Full conveyor with eight stations. At each station the prototype document picks up a stamp/badge. By the finish line it's covered in gold stamps.

Stations (left to right):

1. **UX authors the agent specs** *(gold accent — UX-led)* → stamp: "Approval-first schema"
2. **Warehouse Life seeds the world** *(neutral)* → stamp: "Scripted-causal events"
3. **Domain agents propose** *(mixed accent — gold on UX-influenced patterns)* → stamp: "Per-action `confidence`"
4. **Lead Agent orchestrates** *(gold — UX-led)* → stamp: "Single human-facing surface"
5. **Heuristics audit** *(gold — UX-led)* → stamp: "Calm state · NN10"
6. **Accessibility audit** *(gold — UX-led)* → stamp: "WCAG 2.2 AA"
7. **Human-in-the-loop gate** *(gold — UX-led structural property)* → stamp: "No agent executes alone"
8. **Prototype delivered** — final document card with all gold stamps attached.

**Visual treatment:**

- **Gold-vs-neutral accent system** is the page's primary signal. UX-led stations get a brand-yellow (`#CADF35`) border, station icon, and stamp fill. Neutral stations get a muted gray. The accent itself is the answer to "which agents are UX-related?"
- **Sequential reveal animation** on first viewport entry. The blind track flashes through in ~200ms (deliberately jarring — the abruptness IS the message). The UX track reveals station-by-station over ~1.5s, with each stamp slapping onto the prototype document at its station before moving on. Final stamps animate slightly (subtle scale-pulse) when they land.
- **Hover / focus micro-interaction on each station** reveals a tooltip-card showing: agent name + one-line role + schema fragment (`confidence + escalationReason + options[]` where applicable) + one concrete pattern it produced in the live prototype. Keyboard-accessible (Tab + focus-visible). Source the descriptions from the agent spec files in `/agents/` — do NOT invent them.
- **Subtle ongoing motion** that signals "live" without being distracting: gentle gradient shimmer along the conveyor belt itself (3s ease-in-out infinite, brand-yellow on the UX track, muted gray on the blind track). Wrap all motion in `@media (prefers-reduced-motion: no-preference)`.
- **Labels and copy** on the stations should be 3-5 words each. The stamps are also 3-5 words. Long descriptions live in the hover-cards, not on the surface.

##### Block 3 — Receipts stat strip

Six oversized stat tiles, brand-yellow accents, designed for skim. Each tile is one big number + a one-line caption. Tiles are equal-width, full-row, wrap to two rows on narrow viewports.

- **14** · agent specs authored before a single line of UI
- **2** · Guardian audits — both SHIP
- **AA** · WCAG 2.2 on every measured contrast pair
- **0** · `alert()` placeholders — every Details / Why opens a real modal
- **30:1** · sim compression — 3-minute live human-in-the-loop demo
- **100%** · approval-first — no agent executes alone

##### Block 4 — Closing pull-quote

A single oversized pull-quote on a brand-yellow accent (e.g., a thick left border or a yellow underline beneath the text). Working draft for user to revise before building:

> *The future of UX isn't designers prompting AI. It's UX practitioners owning the agents themselves — defining the guardrails, writing the schemas, auditing the output. That's what UX leadership looks like in 2026.*

This is the line the executive repeats in the next meeting. Without it, the page ends on a stat strip.

#### Bar to clear — the 30-second test

A stranger lands on this page, scrolls once, and within 30 seconds can answer two questions:

1. What did this team's UX practice add that an LLM prompt alone couldn't?
2. Which specific agents did the UX-relevant work?

If they can't, the design isn't done.

#### Constraints

- Single self-contained file — inline SVG, inline CSS, no external assets, no build step. Same constraints as the rest of `moonshot.html`.
- Diagram is pure SVG. No raster images. If a screenshot of the live prototype is needed inside a hover-card or stat tile, base64-encode it inline so `moonshot.html` stays a single zip-shippable file per `WAY-OF-WORKING.md`.
- Motion respects `prefers-reduced-motion`. The sequential reveal must complete in ≤2s total even when fully animated.
- Must pass both Guardian audits — contrast on red/yellow accents must clear AA; sequential reveal must not trap focus or block keyboard nav; hover-cards must be reachable by keyboard (Tab + focus-visible).
- Must not break the existing moonshot.html aesthetic. This page is an escalation of the same visual language, not a different one.

### 5. Verification pass

After tasks 1–4 land:

- Open `moonshot prototype/moonshot-prototype.html` in a browser preview. Confirm it renders and behaves identically to before the dead-code cleanup. Run through the 3-minute demo. No console errors.
- Click the new "How the agents work" tab in `moonshot presentation/moonshot.html`. Confirm the conveyor diagram renders, the sequential-reveal animation fires once on viewport entry, hover-cards on each station show agent name + schema + concrete pattern, and the receipts stat strip + closing pull-quote render cleanly. Confirm `prefers-reduced-motion: reduce` disables motion. Time a cold reader from first paint to "I understand what UX added and which agents did it" — target ≤ 30 seconds.
- Open `moonshot presentation/moonshot.html` in a browser. Click through Home, The Agents, The Prototype.
- Confirm the three new Domain agents (Slotting, Equipment, Quality) render with the same `.card` + `.pill--domain` pattern as the original ones.
- Confirm **Allen Oleksak is credited by name** on each of the three Allen-adapted Domain cards.
- Confirm the **Warehouse Life Agent** card renders on the Agents page (meta pill).
- Confirm the **Heuristics Agent** + **Accessibility Agent** cards render on the Agents page (Guardian pill).
- Confirm the Prototype tab iframe loads the new working copy: manager dashboard renders, Warehouse sidebar button navigates to the warehouse dashboard, KPI sparklines visible, soft-dark terminal renders on right column.
- No console errors. No broken paths.
- The pair `moonshot.html` + `wms-unified-figma-sidebar.html` (or whatever filename ends up embedded) is still self-contained and zip-shippable per `WAY-OF-WORKING.md`.

### 6. Run the one-shot agent-driven prototype build — MVP of "do the agent specs actually drive good interfaces?"

**Not part of today's 5-hour budget.** Independent of tasks 1–5 and runs in a separate fresh chat. The folder `prompt prototype/` at the repo root contains two paired prompts that test whether the agent specs in `/agents/` can drive a fresh Claude session to build the prototype in one shot. The folder has its own README explaining the experiment, how to run each form, and how to save the outputs.

- **Form A** — `prompt prototype/full prompt prototype/PROMPT.md` — one mega-prompt with everything inlined. The control.
- **Form B** — `prompt prototype/agent loaded prototype/PROMPT.md` — agents folder + focused build prompt. The real MVP. Form B's deliverable includes a post-mortem identifying gaps in the agent specs; those gaps are the next round of work on `/agents/`.

Neither form will produce byte-identical output to `moonshot prototype/moonshot-prototype.html` (LLMs are stochastic, the prototype is ~4,700 lines). The test is **structural consistency** — same agents, same dashboard shape, same approval semantics, same visual language. After running both:

- Save outputs alongside each prompt as `output.html` (and `postmortem.md` for Form B).
- Run the Heuristics + Accessibility Guardian audits against each output.
- Report which form produced the more shippable artifact and what was missing from the other.
- Use Form B's post-mortem to plan a follow-up edit pass on the agent specs in `/agents/`.

---

## End-of-day exit criteria

Before declaring the 5-hour session done:

- `moonshot prototype/moonshot-prototype.html` — cleaner, renders identically, both Guardian audits still SHIP.
- `moonshot presentation/moonshot.html` — Agents page shows 14 cards total (8 original WMS topology + 3 Allen-adapted Domain + 1 Meta + 2 Guardian). Prototype tab loads the current prototype. A new "How the agents work" tab exists in the nav and renders the conveyor diagram + receipts strip + pull-quote.
- `moonshot presentation/wms-unified-figma-sidebar.html` (or its replacement) — matches the cleaned prototype.
- Both Guardian audits SHIP on the new page.
- No console errors anywhere.
- `moonshot presentation/` is still zip-shippable as a self-contained pair.

If we run over budget on task 4, **the conveyor diagram + stat strip + pull-quote are non-negotiable**; the sequential-reveal animation and hover-cards can ship as a follow-up if absolutely necessary. The page must function and tell its story even with motion disabled.
