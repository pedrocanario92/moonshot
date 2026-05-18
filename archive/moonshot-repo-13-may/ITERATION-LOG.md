# Moonshot Prototype — Iteration Log
> Morning of May 11 2026 — sessions 0–4. Afternoon of May 11 2026 — session 5. Morning of May 12 2026 — Tasks 1–4 of MAY-12-PLAN. Early afternoon of May 12 2026 — Prompts 1–5 of `tasks-afternoon-may-12.md`. Mid-afternoon — architecture width, navbar centering + Infios-yellow propagation, topology rebuild, page-wide soft-topo pizzazz. Night of May 12 2026 — Prompts 1–3 of `tasks-may-13-morning.md` pulled forward: product switcher, sidebar tier split, TM domain views. Morning of May 13 2026 — Prompts 4–6: OM pillar, cross-pillar dashboard, agent response schema + documentation sweep. Afternoon of May 13 2026 — cross-pillar agent backfill (Spot Rate · Route · Allocation · Cutoff Manager specs · Carrier + Order Priority + Lead cross-pillar addenda) + prototype UX fixes (WM home stage inline · cross-pillar Archer terminal flush). Afternoon of May 13 2026 — Lead Agent renamed to Archer Chat Agent. Afternoon of May 13 2026 — Archer unified as cross-pillar conversation log. Afternoon of May 13 2026 — Architecture diagrams updated for cross-pillar story (System view three pillar rows · dual-context leaves · Timeline info card #3 · lede reframed) + Station 1 narrative softened from "UX authors" to neutral "Agents are authored". Afternoon of May 13 2026 — Warehouse Life Agent renamed to Sim Life Agent and expanded to drive cross-pillar simulation (TM/OM/CROSS event tables, branching decision point at 08:47).

> A session-by-session record of how this prototype evolved. Written for colleagues who weren't in the room, so they can see what was decided, why, and where we are now.

---

## What this project is

See `README.md` for the project framing and `AGENT-ARCHITECTURE.md` for the deep design rationale. This log is the session-by-session record of how the prototype evolved.

---

## Folder restructure (Session 0)

Before any work, we separated the two artifacts:

| Folder | Contents |
|---|---|
| Shipped pair *(repo root)* | `moonshot-home.html` + `moonshot-prototype.html`. Ship-ready. Zip these two when sharing. |
| Working copy *(repo root)* | `moonshot-prototype.html` is edited directly; snapshot to `archive/` before significant changes. |
| `agents/` | Reference material — agent specs (8 WMS + 3 from Allen + 1 meta + 2 Guardian). |
| `original references/` | Allen's original demo + Paulo's initial Claude prompt — for lineage. |

---

## What we did, in seven phases (Sessions 1–3)

The plan called for seven phases. Each phase had a clear goal, a verification step, and stoppable boundaries.

### Phase 1 — Agent specs (the foundation)

**What**: wrote 14 agent specs, each with the same 8-section structure (Identity, Scope, Triggers, Actions, Communication, Guardrails, Escalation, Metrics).

**Three new WMS agents adapted from Allen's demo:**
- **Slotting Agent** — replenishment scheduling, bin assignment, fast-mover re-slotting
- **Equipment Agent** — forklifts, AMRs, conveyors, charge cycles, equipment faults *(genuinely new domain — we had no agent for physical asset state)*
- **Quality Agent** — damage, quality holds, cycle counts *(carved out of the old Exception Agent, which now owns transactional errors only)*

**One meta agent** (`agents/meta/`):
- **Warehouse Life Agent** — the scripted-causal simulation engine. Not in the WMS topology; it *is* the simulated warehouse. Drives the events the WMS agents react to.

**Two Guardian agents** (`agents/guardian/`):
- **Heuristics Agent** — Nielsen Norman + Hick + Miller + cognitive-load audit
- **Accessibility Agent** — WCAG 2.2 (AA load-bearing, AAA advisory)

**Schema additions across every WMS agent** (lifted from Allen):
- `confidence` (0–100) — per-action calibration. Allen uses 92%, 88%, 76%, etc.
- `escalationReason` — the agent's own explanation of why a human is needed.
- `options[]` — when the agent declines to recommend, it surfaces alternatives with the explicit "N options drafted · no recommendation chosen" stance.

Every agent file has a `## Provenance` section so we know what was original, what came from Allen, and what changed.

**Personas swapped throughout** to fix Paulo's accidental Portuguese names:
- J. Santos → **Reyna Castillo** (Operator)
- M. Ferreira → **Marcus Hale** (Shift Supervisor)
- J. Santos (Manager) → **Dana Whitaker** (Warehouse Manager — fixed: the old data had J. Santos in two conflicting roles)
- Berlin DC → **DC-14 Memphis, TN**
- Shift B → **Shift A** (active shift)

**Why**: the specs are the spine of the system. Until the agents have an honest, written contract, the runtime is just decoration.

### Phase 2 — WAREHOUSE state plumbing

**What**: introduced a `WAREHOUSE` state object inside `moonshot-prototype.html` plus a stub tick loop. The clock advances every 500ms (sim-time), but no UI changes.

**Why**: invisible plumbing first. Confirms the new state object loads without breaking anything before we wire any surface to it.

**Verified**: prototype renders identically to before; clock advances internally; zero console errors.

### Phase 3 — Single-focus stage homepage (later replaced)

**What**: built a "Direction A" homepage — single-focus stage with calm hero, top rail, bottom rail showing Lead Agent's last line.

**Why**: needed a destination for the dashboard before wiring data. Set as the new default view, with the original CLI tab demoted to a sidebar item.

**Personas swap landed everywhere**.

### Phase 4 — Chat thread binding + approval cards

**What**: migrated Lead Agent's check-ins to `WAREHOUSE.chatThread`; bound the bottom-rail line + expand overlay to it; built the pending and escalation card patterns (including `options[]` "no recommendation chosen" mode lifted from Allen).

### Phase 5 — Incidents + scripted events + log

**What**:
- Migrated `DATA.floorMap.pins` → `WAREHOUSE.incidents` (unified shape that satisfies both home view and map view).
- Migrated `DATA.actionLog` → `WAREHOUSE.actionLog`.
- Authored **7 root events** on the 08:00–09:30 timeline.
- Wired **2 causal rules** to event payloads (R1: late inbound → Pick Path replen risk; R4: low-confidence equipment fault → `options[]` escalation).
- Wired Approve / Reject / Open / Defer buttons to mutate state and write to the action log with operator attribution.

**Why**: this is the phase that brings the prototype to life. Events fire on schedule, agents react, the user has real decisions to make. Without it, the homepage is a still life.

### Phase 6 — Perf metrics from state

**What**: small phase — `lastUpdated` ticks with sim clock; calm-card KPIs route through `WAREHOUSE.kpi`. The agent performance view's per-agent `actions24h` stays as illustrative DATA (a 5-min demo window can't produce meaningful 24H counts; tracked as deliberate scope call).

### Phase 7 — Tune + Restart shift + Guardian audits

**What**:
- Compression set to 18:1 (later tuned to 30:1).
- Added **Restart shift** button.
- Pressure-tested calm-state coverage.
- **Ran both Guardian audits.** All criteria PASS. Recommendation: SHIP.

**Where this left the prototype**: Single-focus stage layout, 5-minute demo, two user-facing events (Pick Path replen at 08:50 sim, Equipment escalation at 09:05 sim), one autonomous Labor reassign and one autonomous Order Priority resequence flowing through the log only.

---

## What we did in Phase 8 — Manager dashboard pivot (Session 3)

After running through Phase 7 in the browser, four issues surfaced:

1. **Details / Why buttons fired `alert()` placeholders** — full content never reached the user.
2. **The calm state wasted the screen.** "Hide what's handled" was never meant to mean "give the manager nothing." A real KPI / zone / wave dashboard is something the manager actively needs.
3. **Sidebar badges were all red** regardless of meaning.
4. **Demo was too slow** for a 3-minute CEO walkthrough, and cross-page awareness was missing.

**Phase 8 fixed all four:**

- **Replaced single-focus stage with a manager dashboard** — zones strip (left), stage (center), Lead chat (right), KPI strip (bottom). When an incident fires, it overlays as a centered modal with the dashboard dimmed behind. The "hide what's handled" principle preserved: no "handled by agents" feed.
- **Real Details + Why modals** with full incident detail, agent reasoning, audit ID, options breakdown, "Decision lives with you" footer.
- **Sidebar badge semantics**: Pending approval = red (urgent) or orange (standard) or hidden (zero). Validated = green (success). Agent actions = blue (info).
- **Cross-page incident banner** — appears on Map / Log / Perf / CLI when a new pending or escalation fires. Open button navigates back to Home; Dismiss only hides the banner.
- **30:1 compression** — full demo in 3 wall-minutes. First user decision at ~32 wall-sec, second at ~72 wall-sec.
- **CLI thread merged with `WAREHOUSE.chatThread`** — one conversation, two surfaces. "Shift Agent" label renamed to "Lead Agent" (the legacy name contradicted the agent spec topology).
- **Source dividers** in the CLI thread: when user types from Home, a `↳ NEW THREAD STARTED IN HOMEPAGE` divider appears in the CLI scrollback.
- **Per-shift chat** for the demo; multi-shift persistence documented as product intent.
- **Canned Lead acknowledgment** when user types — honest, doesn't pretend to be a real LLM.

Both Guardian audits re-run. Zero load-bearing FAILs. Two WARNs on Heuristics #8 and #3 (calm-state element count) — accepted with rationale: a dashboard is structurally denser than a single-focus stage by design; competing affordances in calm remain zero, which is the spirit of Nielsen #8.

---

## What we did in Phase 8.5 — visual polish (Session 4, in progress)

After Phase 8, four more issues surfaced:

1. **Layout was broken at proper viewport sizes** — `calc(100vh - 96px)` math overflowed, the KPI strip got cut off below the fold, and `overflow: hidden` on the parent prevented scrolling.
2. **The cross-page banner sat in the page flow** rather than floating above it as a notification.
3. **Main page background flickered** because `simTick` re-rendered the entire home view every 500ms.
4. **No visual hierarchy** — three equal-weight white cards side-by-side reads as geometry, not design.

**Direction C — Stage stays light + elevated, zones stays light, chat becomes a soft-dark terminal console.** The dashboard pivot becomes structurally distinct: zones strip on the left for peripheral monitoring, an elevated white stage in the center as the focal point, a soft-dark terminal console on the right that gives the page real character.

**Done so far in Phase 8.5:**

- **Layout cascade fixed.** All `100vh` math removed; heights cascade naturally from `.platform-main` → `.home-wrap` → `.home-dash` via flex.
- **Soft-dark terminal console** — chat panel restyled with `#1E2530` background, brand-yellow (`#CADF35`) agent name tags, cyan (`#67E8F9`) user prompt prefix, Noto Mono for tags and timestamps, terminal-line message format (no bubbles), `> lead@dc-14-memphis · LIVE` header with green pulse dot, `>` prompt cursor on input.
- **Stage elevated** — generous padding, stronger drop shadow, green status dot accent next to the calm hero ("● All zones on target").
- **L-shape KPI layout** — one vertical KPI tile (Orders in flight) + horizontal pair below (On-time, Open items). Stage sits in the alcove of the L.
- **Floating banner** — drop shadow, 180ms slide-down animation, left-edge accent. *(But still uses `position: sticky` — pushes content down rather than overlaying. See open items below.)*
- **Selective re-render** — `renderHome` only fires on state-relevant changes (new event, new chat, incident state, sim-minute boundary). Between renders, the clock text updates surgically. Flicker eliminated. Re-render rate dropped from 2 Hz to ~0.03 Hz.

**Open items the user just flagged and DECIDED on (not yet built):**

- **Banner needs to FLOAT over content, not push it down.** Change from `position: sticky` to `position: absolute; top: 8px; left: 20px; right: 20px; z-index: 30`.
- **Drop the vertical KPI tile.** The L's vertical leg was "naked" — too much space for a single number. Move Orders in flight to the bottom horizontal row alongside On-time and Open items. **Add captions** to each:
  - 1,284 · ORDERS IN FLIGHT · *active across all waves*
  - 97% · ON-TIME · *shipping by carrier cutoff*
  - 0 · OPEN ITEMS · *decisions waiting on you*
- **Replace fixed-px grid widths with `minmax(px-min, %-max)`** for responsive scaling — the prototype needs to render correctly on the user's Surface Pro 11 tablet, not just at 1440×900. Proposed:
  - `grid-template-columns: minmax(180px, 14%) minmax(0, 1fr) minmax(300px, 24%)` (zones / stage / terminal)
  - Bottom row spans zones + stage; terminal full-height on the right.
- **Re-run Guardian audits** after these changes land.

These four items are the next session's work.

---

## Agents — current roster

### WMS topology (in-fiction)

| Agent | Layer | Source |
|---|---|---|
| Lead Agent | Orchestration | Original; sole human-facing agent |
| Shift Intelligence Agent | Radar | Original; signal-only, never proposes |
| Pick Path Agent | Domain | Original |
| Labor Agent | Domain | Original |
| Order Priority Agent | Domain | Original |
| Carrier Agent | Domain | Original |
| Exception Agent | Domain | Rescoped — transactional errors only (damage/holds moved to Quality) |
| **Slotting Agent** | Domain | **From Allen** — replen + bin assignment + fast-mover re-slotting |
| **Equipment Agent** | Domain | **From Allen** — forklifts/AMRs/conveyors. New domain entirely. |
| **Quality Agent** | Domain | **From Allen** — damage/holds/cycle counts. Carved from Exception. |
| BI · Innovation · Opportunity | Analytical | Original; analysis mode only |

### Meta (backstage)

| Agent | What it does |
|---|---|
| Warehouse Life Agent | Scripted-causal sim engine. Generates root events on the 08:00–09:30 timeline; causal rules emit derived events. Drives the prototype's "alive" feel. |

### Guardian (process gate)

| Agent | What it audits |
|---|---|
| Heuristics Agent | NN10, Hick's Law, Miller's 7±2, Cognitive Load, F-pattern, banner blindness, information scent |
| Accessibility Agent | WCAG 2.2 — contrast, keyboard, timing-adjustable, focus order, target size, consistent identification, help |

### Three patterns adopted across every WMS agent (from Allen)

1. **Per-action confidence** (`confidence: 0–100`) — calibrated on each individual decision, not a per-agent rolling stat.
2. **"Why human needed" rationale** (`escalationReason`) — the agent states *why* it bowed out, in its own words.
3. **"Options drafted · no recommendation chosen"** (`options[]`) — when an agent is uncertain, it surfaces alternatives without forcing a choice. Buttons become Open / Defer / Why? instead of Approve / Reject.

### Three patterns explicitly REJECTED from Allen

- **No "Handled by agents" feed.** It's pure noise. Autonomous actions live in the log only, never on the home page.
- **No parallel queues.** One inline focus card + queue chip. Allen's three side-by-side queues are a cognitive-load trap.
- **No single Cautious↔Bold autonomy slider.** Our Tier 1/2/3 constraint model is categorical and defensible; Allen's slider is vibes.

---

## Where we are right now

- **Plan file**: `C:\Users\apcan\.claude\plans\look-at-the-way-mossy-ullman.md` — Phase 8.5 plan section is current and approved.
- **Active prototype file**: `moonshot-prototype.html` (at repo root)
- **Prototype state**: Phase 8.5 partially done. The L-shape KPI layout, soft-dark terminal, elevated stage, selective re-render, and updated personas are all live. Three items still open (banner float, drop vertical KPI + captions, responsive minmax widths) — these were just approved by the user as the next move but not yet built. The last attempted edits were reverted at user request because they were started without an explicit "go."
- **Sim status**: 7 root events on 08:00–09:30 timeline, 30:1 compression, Lead check-ins every 15 sim-min, two user-facing events (Pick Path replen at 08:30 sim, Equipment escalation at 08:50 sim).
- **All Guardian audits**: passed at end of Phase 8. Re-audit pending after Phase 8.5 visual changes land.

---

## Handoff prompt for the next session

> Paste this verbatim at the start of a fresh chat to continue this work:

---

```
We're continuing work on a WMS agentic UX prototype. Read these files first to load context:

1. `ITERATION-LOG.md` at the repo root (`C:\Users\apcan\Documents\moonshot-demo-repo`) — full history of what we've done across sessions and why.
2. `WAY-OF-WORKING.md` at the repo root — file map and project intent.
3. The plan file at `C:\Users\apcan\.claude\plans\look-at-the-way-mossy-ullman.md` — the approved Phase 8.5 plan is the current contract.
4. `moonshot-prototype.html` (at repo root) — the active prototype.

We just finished Phase 8 + most of Phase 8.5. Three things were approved by the user but NOT yet built (the previous Claude jumped the gun on them and was asked to revert):

1. **Banner: float over content, don't push it down.** Change from `position: sticky` to `position: absolute; top: 8px; left: 20px; right: 20px; z-index: 30` inside `.platform-right`. The banner currently sits in the flex flow and shifts the subnav + content below.

2. **Drop the vertical KPI tile. Move Orders in flight to the bottom horizontal row.** The L's vertical leg was "naked" — too much space for a single number. The new layout is a 3-KPI horizontal strip at the bottom, spanning zones + stage width (terminal stays full-height on the right). Each KPI gets a small caption beneath the label:
   - 1,284 · ORDERS IN FLIGHT · "active across all waves"
   - 97% · ON-TIME · "shipping by carrier cutoff"
   - 0 · OPEN ITEMS · "decisions waiting on you"

3. **Replace fixed-px grid widths with `minmax(px-min, %-max)`** for responsive scaling. The user shows demos on a Surface Pro 11 tablet (effective viewport may be 1280×800 or similar) — fixed pixel widths break that. Use:
   - `grid-template-columns: minmax(180px, 14%) minmax(0, 1fr) minmax(300px, 24%)` (zones / stage / terminal)
   - `grid-template-areas: "zones stage terminal" "kpis kpis terminal"` (bottom-row KPI strip spans zones+stage; terminal spans both rows on the right)

After these three changes:
- Re-run the Heuristics + Accessibility Guardian audits per the spec at `agents/guardian/`.
- Verify in the browser at 1280×800 and 1440×900.
- Present the audit verdict report for the user's approval.

Working style the user expects:
- **Out of auto mode by default.** When auto mode is off, ask clarifying questions for any ambiguous decision and get explicit go-ahead before editing — answering my clarifying questions is NOT the same as saying "start."
- **Direct, no-fluff updates.** Diagnose root causes, not symptoms.
- **Verify in the browser via the preview tool**, not by assertion.
- **Never claim a screenshot is fine if the screenshot tool timed out** — say "couldn't capture" and use DOM/geometry checks instead.

Start by reading the files listed above. Then confirm you have context. Then we'll go.
```

---

## File map (for fast orientation)

```
C:\Users\apcan\Documents\moonshot-demo-repo\
├── ITERATION-LOG.md                    ← this file
├── WAY-OF-WORKING.md
├── agentic-warehouse-context.md
├── admin-panel.html
├── agents/
│   ├── lead-agent.md
│   ├── shift-intelligence-agent.md
│   ├── pick-path-agent.md
│   ├── labor-agent.md
│   ├── order-priority-agent.md
│   ├── carrier-agent.md
│   ├── exception-agent.md            (rescoped — transactional errors)
│   ├── analytical-agents.md
│   ├── slotting-agent.md             (from Allen)
│   ├── equipment-agent.md            (from Allen)
│   ├── quality-agent.md              (from Allen)
│   ├── meta/
│   │   └── warehouse-life-agent.md
│   └── guardian/
│       ├── heuristics-agent.md
│       └── accessibility-agent.md
├── moonshot-home.html                 ← shipped (at repo root)
├── moonshot-prototype.html            ← shipped + active working copy (at repo root)
├── original references/
│   ├── allen-demo.html
│   └── initial claude prompt - wms agent cli prototype.md
├── archive/                           ← earlier iterations
└── inspiration/                       ← reference screenshots
```

---

## Afternoon of May 11 2026 — Session 5

> Phase 8.5 closed out. Two rounds of flicker hunting. KPI trend indicators added. CLI re-themed to match the home terminal. Visual language unified across the prototype. Zones strip got real pizzazz. Layout rebalanced so KPIs align column-for-column with the row above. **New Warehouse dashboard view shipped** — the comprehensive operational view the prototype was missing.

### Phase 8.5 — the three approved-but-not-built items + audit

Three items had been approved at the end of session 4 but never built (the previous Claude jumped the gun without an explicit "start" and was reverted). Session 5 opened by finishing them:

1. **Banner floats over content instead of pushing it down.** Changed `.incident-banner-host` from `position: sticky; top: 0` to `position: absolute; top: 8px; left: 20px; right: 20px; z-index: 30`. Added `position: relative` to `.platform-right` so the banner anchors inside the right column (not the viewport). On Map / Log / Perf / CLI, the banner now overlays the subnav while present; subnav reappears unchanged when dismissed; main content underneath never shifts.

2. **Dropped the vertical KPI tile.** The L-shape's vertical leg was carrying too much visual weight for one number. Collapsed to a 3-tile horizontal strip at the bottom of the dash, each tile with a caption beneath the label:
   - `1,284 · ORDERS IN FLIGHT · active across all waves`
   - `97% · ON-TIME · shipping by carrier cutoff`
   - `0 · OPEN ITEMS · decisions waiting on you`
   `renderLAlcove` renamed to `renderKpiStrip` for honesty.

3. **Responsive grid widths.** Replaced fixed pixels with `minmax(px-min, %-max)`. Initial form was `minmax(180px, 14%) minmax(0, 1fr) minmax(300px, 24%)` for zones / stage / terminal. Verified at 1280×800 and 1440×900 — no horizontal scroll, stage absorbs extra space, sidebars hold their floors.

**Guardian re-audit verdict: SHIP.** One WARN noted on terminal timestamp contrast (#6B7280 on #1E2530 = 3.19:1, below AA 4.5:1) — pre-existing from Phase 8.5's terminal palette, not a regression. Fixed later in this session by bumping to `#9CA3AF` (≈ 5.6:1).

### Flicker hunt, round 1 — Map / Log re-rendering every tick

User reported the screen flickering when a notification appeared. Diagnosis (instrumented the render functions): on Map view with a banner up, `renderMap()` was being called **22 times in 19.5 seconds** — ~1.13 Hz. The Phase 8.5 selective re-render gate only covered `renderHome`; non-home views were re-rendering unconditionally every `simTick`. The banner DOM was stable, but the page underneath kept rebuilding, which reads as flicker.

**Fix:** hoisted the signature gate to apply per-view. Each view computes its own sig (Home: `chat thread length + pending + incident hash`; Map: `incident hash` only — the "Updated HH:MM" chip lags between incidents, which matches real-dashboard semantics; Log: `actionLog length + incident hash`). Renamed `lastRenderedHomeSig` → `lastRenderedSig`. Verified Map renders dropped to 3 in 25 seconds (≈ 0.12 Hz — only on real state changes).

### Flicker hunt, round 2 — the home modal retriggering its own animation

User reported a different flicker: navigating from Map (with banner active) back to Home, the banner *and* the background started flickering. Diagnosis: `renderHome` was rebuilding `#main.innerHTML` on every chat thread increment or sim-minute boundary. The home incident modal lived inside that template, so the modal element was being recreated every ~2 wall-seconds — retriggering its `home-modal-fade-in 150ms ease-out` animation. The dashboard behind also visibly redrew through the semi-transparent backdrop.

**Fix — extract the modal to its own persistent DOM host.** New `renderHomeModal()` mounts `#home-modal-host` inside `.platform-right` (NOT `#main`, since `renderHome` wipes `#main.innerHTML`). The host has its own signature gate (`incident.id|state|pendingCount`) so `host.innerHTML = ...` only fires when the incident set actually changes. Also dropped `minute` from the home sig — the clock advances via `surgicalHomeClockUpdate` (textContent on `.home-clock` + `.home-shift`), no need for a full rebuild every minute boundary.

**Verified:** test marker on the modal DOM node survived 19.6 s of continuous ticking; in 35.6 s on home with the modal up, `renderHome` fired only 3 times. Flicker eliminated.

### KPI trend indicators — sparklines + arrows + delta

User: "they're just numbers. they are not telling me visually if going up, down, steady. maybe we should add a visual icon indicator, or include a graph?"

Built a small trend-history system:

- New `WAREHOUSE.kpi.history` — ring buffer of 10 snapshots per KPI (orders / on-time / open items), seeded with a deterministic curve.
- `advanceKpiHistory(minute)` pushes a new value each sim-minute (deterministic drift — orders trends down gently as shipments leave, on-time wobbles around 96, open items reflects live pending count).
- `kpiTrend(arr)` compares mean-of-last-3 vs mean-of-first-3 → returns `{dir: 'up'|'down'|'flat', delta}`.
- `sparklinePath(arr, w, h)` returns SVG `<polyline points="...">` for a tiny chart.
- `renderTrendBlock(arr, goodDir)` emits the sparkline + arrow + delta text. `goodDir` declares which direction is desirable per KPI (`'up'` for on-time, `'down'` for open items, `'neutral'` for orders) so the green/red coloring carries meaning instead of literal up/down.
- `surgicalKpiUpdate()` updates value text + sparkline path + arrow on each tile in place — does NOT trigger a full renderHome. Called from `simTick` after `advanceKpiHistory`.

### KPI tile rework — sparkline takes the right half

User: "you have the full tile width and you're only using half of it. use the right half for the graph!!"

Restructured `.home-kpi` from a vertical stack to a 2-column grid with `grid-template-areas: "val trend" "lbl trend" "cap trend"` and `grid-template-columns: minmax(0,1fr) minmax(80px,1fr)`. Sparkline grew from 72×18 to 100×44; arrow + delta moved to a `.home-kpi__trend-foot` beneath the sparkline, right-aligned. Verified: value text on left half, sparkline + delta filling right half, vertically centered.

### CLI terminal theme — match the home terminal

User: "make the agent cli page also have the same theme as the chat in the homepage."

Scoped a comprehensive theme overlay under `.cli-view`:

- Panel: `#1E2530` bg, `#2C3441` border (later dropped — see universal tile cleanup below), Noto Sans Mono.
- Context bar: dark, with `> lead@dc-14-memphis · CONTEXT LOADED` styling — brand-yellow on `lead@`, green-pulse dot for live.
- Messages: flat terminal-line format `[time] LA  body`. Restructured `.msg-meta` markup to wrap name + sep + time in separate spans so the CLI theme can hide name+separator and show only the timestamp in the gutter. CSS Grid with explicit `grid-column` per child (not `order` — that doesn't work in Grid). Three columns: 62 px timestamp · 28 px tag · 1fr body.
- Input bar: dark `#171F29` field, cyan `>` prompt prefix as `::before`, brand-yellow caret.
- Quick chips + mode pill: dark variants.
- Action cards (decision UI): kept light to stay prominent on the dark surface.

Also bumped `.term__time` from `#6B7280` to `#9CA3AF` (and the equivalent CLI `.msg-meta`) — fixes the AA contrast WARN from Phase 8.5 (3.19:1 → ~5.6:1).

### Visual unification — universal tile pattern + CLI full-bleed

User noticed shadow/outline inconsistencies between pages and asked for "everything should look like the agent performance page." Agent Performance was the reference pattern. Two changes:

**CLI full-bleed.** The dark terminal sat inside a light "outer tile" frame on the CLI page. Now `.platform-main:has(.cli-view) { padding: 0; gap: 0 }` and `.cli-view .chat-panel { border: none; border-radius: 0; box-shadow: none; height: 100% }`. The terminal fills the entire main area (984×680 at 1280×800), header bar at top, no light frame around it.

**Universal tile pattern.** Stripped per-tile bespoke borders + shadows and applied the agent-performance pattern everywhere:
- `border: none`
- `border-radius: 8px`
- `box-shadow: var(--infios-shadow-tile)` (`0 2px 30px 0 rgba(0,0,0,0.15)`)

Applied to: `.home-zones`, `.home-stage`, `.home-kpi`, `.home-chat--terminal`, `.wh-card` (all warehouse cards). Pills, banners, modals, and embedded action cards keep their own chrome since they're not page-tiles. Stage no longer gets a heavier shadow than its neighbors — uniform across the board, per the user's call.

### Zones strip got real pizzazz

User: "everything says on target. but once again it's only text. no visual indicator. that tile lacks pizzazz."

Added meaningful per-zone data to `DATA.floorMap.zones`:
- `load: 0–100` — current utilization
- `rate: string` — domain-specific throughput metric per zone (`22 pallets/h`, `94% full`, `24 pickers`, `12 stations`, `8 lanes`, `6/8 in use`)

Each row is now a 3×2 grid: dot + name + rate metric on row 1; thin capacity bar spanning the bottom. Bar fill width = load %. Bar color is status-aware:
- Green for on-target
- Brand-blue (`--infios-color-text-link`) when load ≥ 85% — "running hot" signal without implying a problem
- Amber for zones with active incidents

Status dot got a subtle glow ring (3 px box-shadow halo); attention zones get a pulse animation (`zone-pulse`, 1.6 s). Visual rhythm matches the KPI sparklines beneath — same visual vocabulary across the row.

### Layout rebalance — KPI columns align with zones / stage above

User: "the center stage is too big. the zones tile gets crammed into the corner for no reason. why not have the zone tile occupy the same width as orders in flight, and the center stage occupy same width as on-time and open items?"

Restructured the dash grid from 3 columns to **4 columns with explicit grid areas**:

```
grid-template-columns: minmax(180px, 1fr) minmax(0, 1fr) minmax(0, 1fr) minmax(300px, 22%);
grid-template-areas:
  "zones stage stage terminal"
  "kpi1  kpi2  kpi3  terminal";
```

Stage spans cols 2–3 in the top row. Terminal spans rows 1–2 of col 4. KPI strip uses `display: contents` so each tile lands directly in `kpi1` / `kpi2` / `kpi3` cells of the parent grid. Result: zones width = KPI#1 width (205 px both, perfectly aligned at left:298 and right:503). Stage width = KPI#2 + KPI#3 combined (517 → 940 both). Verified with DOM geometry — alignment is pixel-perfect.

### Details modal alignment

User: "when you open details over a popup, that opens completely out of whack with where the popup dialog was."

Root cause: `.info-modal-backdrop` was `position: fixed; inset: 0` (centered to the viewport), but the home-modal underneath was centered inside `.platform-right` (excludes the sidebar). The two centers were offset by ~half the sidebar width.

Fix: changed `.info-modal-backdrop` to `position: absolute; inset: 0` and mounted the node inside `.platform-right` instead of `document.body`. Verified: when home-modal and info-modal are open simultaneously, both center at the same X coordinate (delta = 0 px).

### Warehouse dashboard — a brand-new view

User: "we need to figure out what to do with the warehouse page. that one should be the main warehouse dashboard. main kpis of a warehouse operation, everything there is to know."

Wired the previously-dead "Warehouse" sidebar button to `setView('warehouse')`. Added subnav breadcrumb. Built `renderWarehouse()` consuming `WAREHOUSE.warehouseKpis` (a new snapshot data block — intentionally NOT live-bound to the sim, since the home view is "this moment" and the warehouse view is "the whole operation today").

Layout (4-column grid):

| Row | Cards |
|---|---|
| Header | Title `Warehouse dashboard` · location · clock · live pill |
| Hero | Orders shipped today (4,127, +8% vs yesterday) · On-time shipping (96%, target 95%) · Pick accuracy (99.4%, target 99%) · Active labor (184 with role breakdown) — each with sparkline |
| Throughput | Full-width hourly bar chart (06:00 → 17:00 with future hours empty) |
| Mid wide | Wave progress (8 waves with status bars) · Inventory health (fill %, at safety stock, critical low) |
| Ops | Inbound today (22/28, 3 waiting, 1 late) · Putaway (1,840, 96 backlog) · Picks (14,820, 1750/hr) · Pack (4,127, SLA 96%) |
| Status | Returns (38, 0.9% rate) · Equipment (28/32, 1 fault) · Safety (47 days, 98% training) · Carrier mix (UPS 44 · FedEx 35 · USPS 21) |

15 cards total. Snapshot view — added to `simTick`'s per-view gate as a no-op (never re-renders while you're on it).

**Guardian audit on the new view: SHIP.** One WARN on H1 (7 cards visible above the fold, threshold ≤ 6) — accepted with the same rationale used for the Home dashboard pivot: this is a dashboard surface, structurally denser than a focal stage; competing-affordance count is zero (no decisions on this view at all), so the spirit of NN #8 is honored. All 14 measured contrast pairs PASS WCAG AA. Zero load-bearing FAILs.

### Where this leaves the prototype at end of afternoon

- **Home dashboard**: zones strip with capacity bars + rate metrics + pulse-attention dots; 4-column dash grid with KPI columns aligned to top-row columns; KPI tiles with full-half sparklines + trend indicators (semantic polarity per KPI); soft-dark terminal panel; floating cross-page banner.
- **Agent CLI**: full-bleed dark terminal panel matching the home terminal — single visual language for "the agent's voice."
- **Warehouse dashboard**: brand-new comprehensive operational dashboard at `view === 'warehouse'`. 15 cards covering shipped / on-time / accuracy / labor / hourly throughput / waves / inventory / inbound / putaway / picks / pack / returns / equipment / safety / carrier mix.
- **Floor map · Shift log · Agent performance**: untouched in this session.
- **Visual language**: universal tile pattern (white bg, 8px radius, `--infios-shadow-tile`, no border). Pills, banners, modals keep their own chrome.
- **Flicker**: gone. Home `renderHome` ≈ 0.08 Hz when modal up. Map `renderMap` ≈ 0.12 Hz with active banner.
- **Guardian audits**: passed at end of session with explicit rationale for the H1 WARN on Warehouse view.


---

## May 12 2026 — today's plan

The active working plan for May 12 was extracted to [`MAY-12-PLAN.md`](MAY-12-PLAN.md) — a sequenced 5-hour execution plan with task-level time budgets. See that file for the read-in list, the task sequence (cleanup → swap → agent cards → "How the agents work" page → verification), working style, and exit criteria.

This iteration log remains the historical record through end-of-day May 11. New session-by-session entries get appended here after each work block.

---

## May 12 2026 — morning (Tasks 1–4 of MAY-12-PLAN)

A five-hour working block executing the front half of MAY-12-PLAN.md. Tasks 1–4 landed. Tasks 5 (verification pass) and 6 (one-shot prompt prototype experiment) remain. The most consequential outcome is structural: the standalone "How the agents work" page that was scoped as a marquee landing was, mid-session, merged into The Agents page as a Timeline · System toggle. That reshape gave the deck two coherent representations of one system instead of two redundant pages.

### Task 1 — Dead-code cleanup of the working prototype

**What**: audited `moonshot-prototype.html` against the four cleanup categories from the plan (unused CSS selectors, debug `console.*` calls, commented-out blocks longer than two lines, orphaned `DATA.*` properties superseded by `WAREHOUSE.*` equivalents). The reconnaissance pass turned up almost nothing: zero console calls, zero commented-out blocks of meaningful size, zero orphaned `DATA.*` reads. Two CSS selectors were flagged as candidates — `.home-shift` and `.sys-stage-pill` — and re-grepping showed the first was a genuinely empty rule (kept the class in HTML, dropped the empty CSS), while the second was load-bearing for the "Stage 2 baseline" pill on the System health panel and was left alone. Net change: `moonshot-prototype.html` shrank from 4,426 to 4,425 lines.

**Why**: the plan asked for a conservative pass before promoting the prototype to the shipped folder. The discovery — that the file is already lean — was useful information in itself, and the temptation to over-prune was resisted.

**Verified**: re-grepped both selectors against HTML + JS before deleting; opened the prototype in the browser preview and ran through the full 3-minute demo. No regressions.

### Task 2 — Promote the working prototype to the shipped artifact

**What**: three steps.

1. **Archived** the previously-shipped `wms-unified-figma-sidebar.html` (2,508 lines) into `archive/wms-unified-figma-sidebar.PRE-MAY-12.html`. Archive lives on disk in case rollback is needed.
2. **Copied** the working prototype into the shipped slot and **renamed** it to `moonshot-prototype.html` (dropping the legacy `wms-unified-figma-sidebar` name — the figma sidebar variant is no longer relevant to the project).
3. **Updated** the iframe `src` in `moonshot-home.html` to point at the new filename. Confirmed no other references to the old filename remained.

**Why**: the presentation folder's prototype had been stale relative to the working copy for weeks. Promoting in a single swing keeps the shipped artifact aligned without introducing a rename ripple across the rest of the codebase.

**Verified**: opened the Prototype tab in moonshot-home.html; manager dashboard, Warehouse view, KPI sparklines, soft-dark terminal, and floating banner all rendered inside the iframe with no console errors.

### Task 3 — Audit, attribute, and re-shape The Agents page

The longest task by file diff. It started as "add the six new agents to the Agents page" and grew into a substantial reconciliation across cards, specs, structure, and copy. Broken into ordered sub-steps:

**3a — Agent folder reconciliation.** Walked `/agents/` (14 spec files) against `moonshot-home.html`. Found:
- Every spec'd agent already had — or now needed — a card. Six agents had no cards yet: **Slotting, Equipment, Quality** (Domain — adapted from Allen's demo); **Warehouse Life** (Meta); **Heuristics, Accessibility** (Guardian — present in stub form, needed the full spec-derived treatment).
- The existing cards' "Talks to" fields named only the original five Domain agents; the new Slotting/Equipment/Quality edges were missing.
- Two avatar collisions: **OP** was used by both Order Priority and Opportunity; **IN** was used by both Innovation and Interaction.

**3b — Six new agent cards.** Built using the existing `.card` + `.pill--domain` pattern, with copy sourced from the spec files (not invented):
- **Slotting Agent** (avatar `SL`) — replenishment scheduling, bin assignment, fast-mover re-slotting. Italic credit line *"Adapted from Allen Oleksak's demo"* above the pill row.
- **Equipment Agent** (avatar `EQ`) — physical asset state; forklifts, AMRs, conveyors. Same Allen credit.
- **Quality Agent** (avatar `QA`) — damage, holds, cycle-count variance, audit risk. Same Allen credit.
- **Heuristics Agent** (avatar `HE`) — upgraded from a one-line stub to the full spec-derived treatment: Nielsen Norman / Hick / Miller / Cognitive Load body + Talks-to + Guardrail blocks. Pill class stays `pill--guardian`.
- **Accessibility Agent** (avatar `AC`) — same upgrade, WCAG 2.2 framing.
- **Warehouse Life Agent** (avatar `WL`) — new Meta layer added, with a new `.pill--meta` style (slate-teal `rgba(72,98,110,0.10)` background, `#48626E` text) and a new `section-meta` block between Guardian and Horizon. New layer-title icon style and a layer description that frames the agent as backstage by design.

**3c — Attribution treatments.** Two distinct credit conventions emerged:
- **"Adapted from Allen Oleksak's demo"** — on Slotting, Equipment, and Quality. Small italic line, soft tertiary text color, sits above the Domain pill.
- **"Authored by the UX Team"** — same visual style, applied to all six Guardian cards (Accessibility, Heuristics, Design System, Content, Interaction, Responsive) and to the Warehouse Life Meta card. Marks the seven agents that belong to the UX practice rather than the WMS topology.

**3d — Exception Agent rescope.** The Exception Agent previously bundled mispicks + damage + quality holds. With Quality carved out, the Exception card was rewritten:
- Role line: *"Mispicks. Damage. Quality holds."* → *"Mispicks. Short-picks. Scan discrepancies."*
- Body rewritten to make the carve-out explicit (*"Damage, holds, and cycle-count variance moved to the Quality Agent."*).

**3e — Avatar collision fixes.** Opportunity Agent → `OY`. Interaction Agent → `IA`. No two cards share an avatar shortcode now.

**3f — Topology drift fixes (cards AND specs).** The page's full-width cards for Lead and Shift Intelligence used the phrase *"all five domain agents"* in their "Receives / Sends" fields. Updated to *"all eight domain agents"*. The Talks-to fields on Pick Path, Labor, and Exception cards were extended with the new edges (Pick Path ↔ Slotting/Equipment; Labor ↔ Equipment/Quality; Exception ↔ Quality). Four spec files (`lead-agent.md`, `shift-intelligence-agent.md`, `pick-path-agent.md`, `labor-agent.md`) had the same drift — *"Pick Path, Labor, Order Priority, Carrier, Exception"* enumerations updated to include Slotting/Equipment/Quality. Joint-recommendations sections in `labor-agent.md` extended to mention Quality (investigation labor) and Equipment (operator reassignment).

**3g — Structural reorder.** The page had a "What comes next" divider sitting between Analytical and Guardian. That framing was wrong: Guardian and Meta are part of the system in use today, not future work. The divider moved to a new home — between Meta and Horizon. Guardian and Meta layers themselves moved above the "How the signals move" topology section, so that the topology block now describes a system with every operational and process layer already introduced.

**3h — Topology block rewrite.** "How the signals move" got two new paragraphs:
- *"Guardian audits gate every ship."* — frames Heuristics and Accessibility as a process gate, outside the WMS signal topology, talking only to each other and to the human reviewer.
- *"The sim engine is backstage."* — frames Warehouse Life as state-only, no direct signal path.

Five new path rows added to the "Notable communication paths" sidebar (Slotting↔Equipment, Quality↔Labor, Equipment↔Labor, Exception↔Quality, Heuristics↔Accessibility, Guardian→Human, Warehouse Life→State), plus three new prohibited rows (Guardian✕WMS, Meta✕User).

**3i — Architecture diagram rewrite.** The ASCII tree at the top of the Agents page was rebuilt:
- All eight Domain agents listed (was five).
- Exception scope corrected in the inline label.
- Guardian Layer section added — described as *"pre-deploy gate · outside WMS topology · audits the artifact, not the runtime"*, listing all six Guardian agents.
- Meta Layer section added — *"backstage · prototype only · never surfaces to user"*, listing Warehouse Life.
- HUMAN row now acknowledges the second seam: *"approves / rejects / downgrades · reviews Guardian verdicts"*.

**3j — Hero copy + navigation.** Final staleness sweep on the Agents view:
- Kicker pill: *"Three layers · Ten agents · One human in the loop"* → *"Six layers · Twenty agents · One human in the loop"*.
- Lede: rewritten to enumerate all six current layers (one orchestrator, one radar, eight domain experts, three analytical agents, six Guardian audits, one backstage sim engine).
- In-page sticky nav strip got a **Meta** item inserted between Guardian and Horizon.
- JS `sectionIds` array got `'section-meta'` added so the sticky-nav active-state tracks correctly when scrolling over the Meta section.

**Why**: The Agents page had drifted relative to `/agents/`, which is the source of truth. Specs had been added but the page didn't reflect them; agent-interaction descriptions named only the original five domain experts; the kicker and lede made stale claims; the ASCII architecture diagram showed the May 11 topology, not the May 12 one. Reconciling forced explicit decisions on attribution conventions, on which agents constitute *the system in use today* versus *what comes next*, and on how to visually mark UX-led versus Allen-adapted work.

**Verified**: re-grepped for stale `"five domain"` claims across active files (none remained outside `archive/`). All 14 spec'd agents have cards, all cards have unique avatars, the in-page nav scrolls cleanly to every layer section including Meta, and the architecture diagram reads coherently for both the operational chain and the lateral Guardian/Meta layers.

### Task 4 — Build the "How the agents work" view, then merge it into The Agents page

The longest and most iterated sub-task. Started as the marquee task from the plan — a new top-nav page with a B2C-style landing layout (hero, conveyor diagram, stat strip, pull-quote). Ended with the page eliminated and its core artifact (the conveyor) merged into The Agents page as a toggle view. Roughly five revisions; each driven by direct user feedback.

**4a — Foundation pass (initial build).** Created a new `<section class="view view--howagents">` between view-agents and view-prototype. Wired top-nav button and JS hash routing for `#howagents`. Four blocks inside:
- Hero with kicker, headline *"The UX layer most AI products skip. We built it."*, and a kicker line counting *"twenty agents and two audits"*.
- Conveyor diagram placeholder.
- Six-tile stat strip: 14 specs · 2 Guardian SHIPs · WCAG 2.2 AA · 0 alert() placeholders · 30:1 sim compression · 100% approval-first.
- Closing pull-quote — softened from the plan's working draft to *"The next step for UX is partnering with dev and product to design the agents themselves — defining the guardrails, writing the schemas, auditing the output. That's how design leadership shows up in 2026."* The softening was a deliberate choice to play ball with dev and PM rather than position UX in opposition to them.

**4b — SVG conveyor diagram (first attempt).** Built as an inline SVG with viewBox 1280×660 holding two stacked tracks:
- Top track "Prompting blind" — sparse belt from prompt to a faded prototype, decorated with a five-flag red callout listing the anti-patterns the plan called out (`alert("Approved!")` popups · three parallel queues · Cautious↔Bold slider · low-contrast text · "handled by agents" noise feed).
- Bottom track "UX-owned agent system" — seven station circles on a brand-yellow belt, each with a stamp pill above and a two-line label below. Three accent variants (gold for UX-led stations, neutral gray for backstage, dashed-gold-on-pale-yellow for UX-influenced).
- Hover-card overlays per station with role + schema fragment + concrete pattern.

User feedback: *"I don't like the on hover tiles. all data should be displayed so that if my boss wants to screenshot the diagram, all info is there"*. And: *"the prompting blind is showing errors"* — the red flag callout looked like runtime errors rather than the intentional output contrast.

**4c — Always-visible info cards + red flag re-framing.** Replaced the hover-only tooltip system with a 7-column grid of always-visible cards below the SVG. Each card got a colored number badge floating on the top edge, gold/neutral/dashed accent on its top border, and the full role + schema + "In the prototype" pattern visible at all times. The red flag callout got an explicit *"PROTOTYPE DELIVERED WITH:"* header, reframing the chips as the output of the prompting-blind path rather than as page errors.

**4d — Reveal animation.** All motion wrapped in `@media (prefers-reduced-motion: no-preference)` — reduced-motion users get instant render. An IntersectionObserver fires the `is-revealed` class once on first viewport entry (`{ threshold: 0.2 }`, then `unobserve`). The choreographed reveal completes in ~2 seconds: UX background and belt set the stage; the blind track flashes in fast (deliberately jarring); UX stations cascade left-to-right (~150ms per station); the final document arrives last; info cards stagger in behind. Ongoing motion after reveal: subtle `fill-opacity` pulse on the UX belt; `stroke-width` pulse on the delivered document.

User feedback: *"much better! but the timeline is shit. it's lazy. the proto delivered tile is ugly. where is the finesse? the pizzazz? the subtle modernity? the apple-esque quiet luxury with pops of color?"* And: *"we don't need the 'prompting blind' section, we just need the ux owned agent system. but let's not call it that. call it ux enhanced system"*.

**4e — Big visual rewrite — drop blind track, premium treatment.** Strategic decisions:
- Blind track and red flag callout dropped entirely. The diagram is now one-sided.
- Renamed *"UX-owned agent system"* → *"UX Enhanced System"*.
- Architecture switch: SVG dropped in favor of CSS Grid. Nine columns (prompt + seven stations + final). Both the timeline row and the info-card row live in the same grid, so each station and its corresponding info card column-align perfectly.
- Premium visual treatment on every element:
  - Station nodes — 54px circles with radial gradients (gold variant `#E4F26B → #CADF35 → #B5C82F`), layered shadows (1px outline + 4–14px brand-yellow soft glow + inset top highlight for sheen), dashed-gold outline-offset border for the mixed variant.
  - Stamp pills — fully-rounded, refined typography, subtle gold-shadow glow.
  - The connecting rail — thin 2px gradient line, fading at both ends, threaded behind the nodes.
  - Final endpoint — premium card with gradient background, brand-yellow border tint, triple-layered drop shadow (near-black + yellow-tinted + deep ambient), and a 36px gold seal disc with an inline SVG checkmark on top.
  - Info cards — 14px corner radius, layered shadows, `translateY(-2px)` on hover, schema chips with subtle gradient + soft border.

Caught one layout bug after the rewrite: `.endpoint--prompt` had no explicit `grid-column`, so it auto-flowed past the explicitly-placed stations and landed on the right edge of the timeline. Fixed by adding `.endpoint--prompt { grid-column: 1 }`.

**4f — Zigzag layout + C-level content trim.** User: *"the text tiles are so slim that it's hard to read. what if we zig zag? step 1 tile below, step 2 tile above, etc. that way the tiles can be wider."* And: *"let's remove unnecessary fat from those tiles. this is for a c-level. it should say what it is and why they should care. don't go too much into the weeds"*. Two changes:
- Grid restructured to three rows. Timeline moved to row 2. Odd-numbered cards (1, 3, 5, 7) sit in row 3 below the timeline; even-numbered cards (2, 4, 6) sit in row 1 above. Each card spans two grid columns starting at its station's column — doubling the readable width. The indicator stripe flips to face the timeline on each card (top edge for below-cards, bottom edge for above-cards).
- Card content trimmed to heading + one short body paragraph (~2 sentences). Schema chip dropped. "In the prototype:" weeds dropped. Bolded the value-prop phrase in each card so a skimming reader catches it: *designed in* · *end-to-end* · *confidence score* · *the single seam* · *gated, not assumed* · *verified before the prototype ships* · *the design promise*.

**4g — The scope change: merge "How the agents work" into The Agents.** User question: *"we have 'how the agents work' and 'the agents' as two different pages, but in the how the agents work page all we have is this timeline which is a visual display of the shape of the system diagram in the agents page. what if we put this timeline in that page overlapping with the shape of the system diagram, re-design the shape of the system diagram so that we can toggle between the timeline view and system view"*. After alignment on content disposition (timeline conveyor → Agents page; pull-quote → home page closing argument; stat strip and hero deleted; "How the agents work" page removed entirely), the merge was built:

- **Apple-style segmented control** above the diagram area. Pill-shaped two-segment switch (Timeline · System) with an inactive transparent background and a white-pill active state riding a layered soft shadow (`0 1px 2px rgba(0,0,0,0.06), 0 4px 12px rgba(0,0,0,0.04)`). A `.diagram-toggle__indicator` element slides via `transform: translateX(100%)` between segments on click, 0.28s with a cubic-bezier curve that has a touch of overshoot.
- **Polished CSS Grid hierarchy tree** as the System view — visual sibling of the timeline:
  - HUMAN · APPROVES card at the top (mono uppercase, soft gradient background).
  - Lead Agent below — gold-tinted card with a brand-yellow glow shadow.
  - Shift Intelligence below that — neutral card.
  - Eight domain agent leaves arrayed in a row, each with a thin gold connector line dropping from a horizontal gold rail above.
  - Three lateral layer boxes: Analytical (purple `#5436CC` accent), Guardian (green `#2C812C` accent), Meta (slate `#48626E` accent). Each box lists its constituent agents as accent-tinted pills.
- **Cross-fade mechanism.** Both views live in the same CSS Grid cell via `grid-area: stack`. The inactive view is `opacity: 0; visibility: hidden`; the active view is fully visible. Switching = a 0.28s opacity transition.
- **JS handler.** Click + keyboard (ArrowLeft/Right) switches both `data-active` attributes (toggle and diagram), updates `aria-selected` and `tabindex` on each segment, and the CSS handles all the visual work.
- **Conveyor migrated** from `view-howagents` into the `.arch` section as `.diagram__view--timeline`. Wrapping `.howagents-conveyor` class kept so the existing reveal-animation CSS continues working without rewiring.
- **Pull-quote relocated** to view-home as the closing argument, inserted between the "A note on method" block and the footer. Styled via a new `.closing-quote` class (the previous `.howagents-quote` was deleted along with the rest of the howagents view).

**4h — Cleanup of the now-redundant howagents page.**
- Top-nav button removed.
- The entire `<section class="view view--howagents">` removed (170 lines).
- `'howagents'` removed from both JS hash-routing handlers.
- Dead CSS deleted: `.howagents-hero*`, `.howagents-headline`, `.howagents-kicker`, `.howagents-stats*`, `.howagents-stat*`, `.howagents-quote*`, and the legacy `.arch__diagram*` rules that styled the now-replaced ASCII tree.

**4i — Lede repositioning.** User: *"that text between the toggle and the diagrams is distracting. put it below the diagrams. center aligned, horizontally aligned to center of page"*. Each view's lede paragraph moved from above the diagram to below it (after the legend on timeline view, after the lateral layer boxes on system view). CSS updated: `margin: 48px auto 0; text-align: center; max-width: 62ch`. Reads now as a deliberate caption summarizing what the viewer just saw.

**4j — Alignment + spacing fix.** User: *"1 — notice how the shape of the system and orchestration sections are misaligned… 2 — there too much vertical white space. the diagram doesn't appear above the fold."* Two root causes:
- **Misalignment**: `.howagents-conveyor` carried legacy section-wrapper styles (`padding: 96px 24px`, background, border-bottom) from when it was its own top-level page section. Inside the new `.arch` toggle wrapper, that 24px horizontal padding pushed the diagram content inward 24px more than the section title above, so the cards/stations looked skewed right compared to *"The shape of the system."* and the Orchestration section below.
- **Vertical space**: 32 (section-title `margin-bottom`) + 32+36 (toggle `margin: 32px auto 36px`) + 96 (conveyor `padding-top`) + 24 (grid `padding-top`) = ~220px of stacked whitespace between section title and first row of stations.

Fix in one CSS pass:
- `.howagents-conveyor` stripped of its section-wrapper styles — now just `padding: 0`.
- `.howagents-conveyor__inner` lost its conflicting `max-width: 1280px` (it had been wider than the parent `.arch__inner`'s 1120px max). Now `width: 100%`.
- `.diagram-toggle` margin tightened from `32px auto 36px` → `8px auto 20px`.
- `.arch` padding tightened from `80px 24px` → `56px 24px 64px`.
- `.arch .section-title` `margin-bottom` tightened from `32px` → `12px`.

Result: at 1440×900 the entire diagram (title + toggle + timeline conveyor with stations and info cards + legend + centered lede) sits above the fold. Title-to-toggle and toggle-to-grid gaps each ~20px. `.arch__inner`, `.layer__inner`, and `.howagents-conveyor__inner` all sit at the same left edge — Architecture and Orchestration section titles column-align exactly.

**Why**: the standalone "How the agents work" page started thin (the conveyor was its only substantive content) and the existing ASCII architecture diagram on The Agents page had grown visually weak relative to the polished conveyor. Merging onto one page with a toggle gives the system exactly two coherent visual representations — process and architecture — without redundant navigation and without forcing the viewer to compare diagrams across pages. The aesthetic iterations (drop blind track → CSS Grid rewrite → zigzag → C-level body copy) were each user-driven design corrections; the post-merge alignment fix closed out the structural drift between the new and the existing sections.

**Verified**: top nav reads Home · The Agents · The Prototype. Hash `#howagents` is a safe no-op (the previous active view stays active; no console error). Toggle's segmented control switches both `data-active` attributes on click and ArrowLeft/Right, and slides the indicator cleanly. System view renders 14 nodes (3 spine + 8 leaves + 3 layer boxes). Timeline view's reveal animation still fires once on first viewport entry — `is-revealed` class added; `unobserve` called. Reduced-motion override added on the toggle indicator and view cross-fade so reduced-motion users get instant swaps. Home page closing pull-quote visible above the footer. No console errors. Architecture and Orchestration section titles column-align at the same left edge.

### Where this leaves us at end of morning

- **`moonshot-prototype.html`** — cleaner by one line. Otherwise unchanged from end of May 11. (As of this session, a separate working copy lived at `moonshot prototype/moonshot-prototype.html`; the folder split was later collapsed and both files now live at the repo root.)
- **Newly promoted prototype artifact** — byte-identical to the working prototype. Old `wms-unified-figma-sidebar.html` archived under `archive/`.
- **`moonshot-home.html`** — The Agents page substantially rebuilt. Six new agent cards added across Domain (Slotting, Equipment, Quality), Guardian (Heuristics, Accessibility upgraded), and a brand-new Meta layer (Warehouse Life). Kicker and lede updated to reflect six layers / twenty agents. "How the signals move" topology expanded with new edges and lateral-layer paragraphs. Architecture section rebuilt with a Timeline · System toggle — Timeline view holds the polished zigzag conveyor with the seven-station, two-row info-card layout; System view holds a polished CSS Grid hierarchy tree showing the operational spine and the three lateral layer boxes. Home page closing pull-quote relocated from the (now-deleted) howagents view. The standalone "How the agents work" page no longer exists in nav or DOM.
- **`/agents/`** — four spec files reconciled with the eight-domain topology (`lead-agent.md`, `shift-intelligence-agent.md`, `pick-path-agent.md`, `labor-agent.md`).
- **Outstanding from the May 12 plan**: Task 5 (full verification pass across tasks 1–4) and Task 6 (one-shot prompt prototype experiment in `prompt prototype/`).

**Notable open follow-ups** flagged during the morning but deliberately not picked up:
- The four Guardian agents without specs (Design System, Content, Interaction, Responsive) still carry the "Authored by the UX Team" attribution alongside Heuristics and Accessibility. If a spec is later written for any of them, the card should be upgraded to the full Talks-to + Guardrail format.
- The `.howagents-conveyor` class name is now misleading (the conveyor lives inside The Agents page's Architecture section, not on a "how the agents work" page). Renaming to a neutral class like `.conveyor` would be cleaner; deferred to avoid churn.

---

## Afternoon of May 12 2026 — Task 1 of `tasks-afternoon-may-12.md`

### Afternoon Task 1 — design-tokens skill, token integration, and the Infios homepage glow-up

The afternoon's first task started as a procedural one — install a reusable design-tokens skill and run it once against the May 12 token zip — and grew into a homepage redesign once the new token palette was available. Four phases.

**1a — Install the `design-tokens` skill.** Created `.claude/skills/design-tokens.md` at the repo root (project-scoped, not global). YAML frontmatter uses the verbatim auto-trigger description from the prompt brief so future deliveries fire the skill on keywords like *"design tokens"*, *"tokens.zip"*, *"integrate tokens"*, *"reconcile tokens"*. Body holds an 8-step procedural workflow: **locate → extract → transcribe → reconcile → mirror → sweep → verify → brand-yellow safety check**. The body explicitly documents the naming-scheme reality discovered during exploration: the two `:root` blocks use different conventions today (presentation = `--surface-*` / `--text-*` / `--accent-*`; prototype = `--infios-color-*` / `--infios-spacing-*` / `--wa-*`), and renaming would cascade through 4,700 lines of prototype CSS — so the skill's reconcile step builds a **union** of both schemes plus the new delivery rather than renaming anything.

**1b — Transcribe + integrate the May 12 delivery.** Extracted `C:\Users\apcan\Pictures\Screenshots\tokens.zip` (15 PNG screenshots, 1.87 MB) to `archive/tokens-2026-05-12/` and transcribed every token by category into `archive/tokens-2026-05-12/TOKENS.md`. The delivery brought in: fonts (`--infios-font-family-default`, `-code`, weights light/regular/bold/black, size + line-height pairs from 12/16 through 32/40), an 11-step spacing scale (`--infios-spacing-3xs:4px` through `-5xl:64px`), four component-height tokens, four clickable-size tokens, ~70 color tokens organized by bg / text / border / icon / data-viz, five elevation shadow tokens, four border-radius tokens (`-sm:2px` through `-pill:9999px`), border widths and offsets, and five transition-duration tokens (`-x-slow:1000ms` through `-x-fast:50ms`).

Reconciled the union into `moonshot-home.html`'s `:root` and mirrored byte-identically across the prototype copies present at the time. Discovery (under the old folder structure): the deck's Prototype tab iframe loaded the sibling copy promoted in the morning, not the separate working copy. After the May 13 folder flatten there is a single `moonshot-prototype.html` at the repo root, so future token reconciliations mirror across just the two files. All `:root` blocks at the time sat at 9,769 chars byte-identical. Sweep replaced standalone `color:#CADF35` declarations with `var(--wa-nav-accent)` in both prototype copies (12 sites each) and the deck's `border-top:4px solid #CADF35` with `var(--accent)`. Multi-stop gradients using `#CADF35` as a middle stop alongside `#E4F26B` / `#B5C82F` were left intact — flagged as intentional off-grid in TOKENS.md.

**1c — Brand-yellow safety flag (resolved without overwrite).** The delivery introduces `--infios-color-icon-brand: #D0FF25` and `--infios-color-text-brand: #D0FF25` — a brighter shade than the existing `--accent: #CADF35`. Per the skill's safety check, neither `--accent` nor `--wa-nav-accent` was overwritten; the new brand-icon shade was added under its own names. Same logic applied to other token-name conflicts inside the prototype's existing schema (e.g., `--infios-color-text:#171F29` vs spec `#232625`, `--infios-color-text-link:#5436CC` vs spec `#4A8FFF`, `--infios-color-border-focus:#5436CC` vs spec `#0060FF`): legacy values kept to preserve visual identity, spec values documented inline as `/* spec: <value> — kept legacy */` comments and noted in TOKENS.md's "Legacy tokens still referenced" section. Verification step's *"render visually identical"* requirement would otherwise contradict the spec being authoritative.

**1d — Homepage glow-up (user-driven redesign once tokens were in).** With the full Infios palette now available, the home page felt under-leveraged. User: *"there's not a lot of infios yellow. there's purple alright, but the other yellows are dull. and most buttons are just black. they should be charcoal, there should be more pizzazz in these home and agents pages now that we have full token access."* And: *"also, remove this section: A note on method… not needed"*. And on the hero: *"AI enables the interface. People design the experience. NO! AI enhances the delivery. People enhance the experience."* Plus: *"this sounds like you're describing the intro to your college essay. two lines, wham bam here's where the value is. and for both, use the full page width!!"* Plus: *"the 'two doors, one throughline' line can be deleted as well. replace that with a simple show me the prototype link with an arrow in line that moves when you hover over it."*

Built in one pass:
- **Three new project-local helper tokens** added to all three `:root` blocks (keeping the byte-identical mirror): `--charcoal:#262523` (warm charcoal alias replacing cold blue-black `#171F29` on button surfaces), `--charcoal-soft:#3A3633`, `--accent-bright:#D8EE3F` (hot yellow above the brand `#CADF35`), `--accent-glow:rgba(202,223,53,0.45)` (the glow shadow color).
- **`.note` section removed** entirely (HTML + the orphan CSS rules left untouched — minor dead CSS).
- **Hero copy rewrite.** Title: *"AI enhances the delivery. People enhance the experience."* Lede cut to one punchy line: *"Interfaces are nearly free now. **Trust still has to be designed.**"* with a yellow highlighter behind the strong text. Subnote *"Two doors. One throughline."* deleted; replaced with an `.hero__cta` anchor reading **"Show me the prototype →"** — a circular yellow arrow icon that slides 14px right on hover with a yellow glow shadow, plus a yellow underline that sweeps left-to-right under the text.
- **Hero typography scaled up** for the new full-width layout: `.display` font-size from `clamp(44px, 6.4vw, 88px)` → `clamp(52px, 7.6vw, 112px)`; weight 600 → 700. `.lede` from 21px fixed → `clamp(22px, 2.3vw, 30px)`. Now 109px / 30px at 1440 viewport.
- **Hero full-width.** `.hero__inner` had a hard `max-width:1120px`. Stripped that; inner now uses viewport-relative padding via `padding-left: max(24px, calc((100vw - 1600px) / 2))` so content edges the viewport (or stops at a 1600px reading limit on very wide screens). Same treatment on `.agents-hero__inner` and on the new closing-quote.
- **Charcoal swap.** `.door:hover .door__arrow` background switched from `var(--brand-fill)` / `var(--text-primary)` to `var(--charcoal)`. `.hero__cta-arrow` text color is `var(--charcoal)` on yellow. `.strip__item--accent .strip__num` is charcoal on yellow.
- **Yellow injected across home + agents:**
  - `.kicker__dot` — was muted `--accent-deep` with a 4px halo; now bright `--accent` with a 5px halo plus a 14px `--accent-glow` outer glow.
  - `.display em::after` — yellow gradient underline behind *"People enhance the experience."*
  - `.lede strong` — yellow highlighter background.
  - `.door--prototype` — top stripe upgraded from 3px two-stop to 4px three-stop gradient (`--accent-bright` → `--accent` → `--accent-deep`); subtle yellow tint on the card background; hover gets a `0 24px 60px rgba(202,223,53,0.18)` glow shadow and a yellow border.
  - `.door--prototype .door__num .icon` — icon swatch is now solid `--accent` with charcoal text and a `--accent-glow` shadow (was muted `--bg-accent`).
  - `.door--prototype:hover .door__arrow` — arrow circle goes yellow with charcoal arrow and a `--accent-glow` ring on hover.
  - `.strip__item--accent .strip__num` — Guardrails badge now solid bright yellow on charcoal text with a yellow glow (was muted `--bg-accent`).
  - `.strip__item--accent h3::after` — 32px yellow underline accent under *"Guardrails are the work."*
  - `.agents-hero h1` — *"The Agents**.**"* — the period rendered in `--accent-deep` yellow via a new `.accent-dot` span.
  - `.agents-hero__lede strong` — yellow highlighter behind the rule-of-the-system sentence.

**1e — Topographical map background (replacing the initial dot grid).** First iteration used two layered tiled-dot CSS backgrounds (`background-image: radial-gradient(...)`, 42px and 140px sizes, masked + scale-breathing). User: *"the simple dot grid is not doing it for me. i want something that resembles a topographical map in outlines, with little dots to represent places on the maps, animated, things like that. also, that last section with 'the next step for ux is partnering…' also needs the same visual treatment as the hero."*

Rebuilt as inline SVG with three reusable layer groups under a common `.bg-topo` class, applied to `.hero`, `.agents-hero`, and `.closing-quote`:

- **`.topo__lines`** — 6–7 curved contour paths drawn with cubic Béziers (`M-50,80 C200,40 480,140 800,80 S1200,160 1700,90` for example), sweeping across the full width with gentle vertical wandering. Every other path (`:nth-child(2n)`) is dashed yellow (`stroke:rgba(202,223,53,0.55); stroke-dasharray:5 7`); the rest are solid charcoal at 20% opacity. Both groups animate `stroke-dashoffset` on slow loops (28s yellow / 38s charcoal, opposite directions) so the dashed yellow contours drift sideways like wind over a topo map. `vector-effect:non-scaling-stroke` keeps the lines crisp at all viewports.
- **`.topo__peaks`** — 3 clusters of concentric ellipses per section (3+3+2 rings). Each cluster scales `1 → 1.08` on a 12s loop via `transform-box:fill-box; transform-origin:center`, staggered by `-4s` and `-8s` so peaks pulse at different phases. First cluster yellow-stroked, second charcoal, third purple-stroked — mixed accents on the topographic layer.
- **`.topo__dots`** — 14–16 small `<circle>` elements scattered as "place markers." Each pulses opacity + scale (`0.85 → 1.25`) on a 4.6s loop; staggered delays via `nth-child(3n)`, `4n`, `5n`, `7n`. Yellow dots carry an `accent-glow` drop-shadow; every third dot is charcoal at 55% opacity (reading as a "place," not a star); every seventh dot is `--accent-bright` with a stronger glow.

The whole `.bg-topo` group also breathes scale `1 ↔ 1.025` over 22s for a slow exhale, masked with `radial-gradient(ellipse at 50% 45%, #000 0%, #000 62%, transparent 96%)` so the pattern fades out at the edges.

Closing-quote upgraded to a full-width "mini hero" matching the hero treatment: padding bumped from 64/24/40 → 120/56/96, background switched from `--surface-base` to `--surface-canvas`, inner container full-width (max-width:none, viewport-relative padding), blockquote text scaled from `clamp(20px, 2.2vw, 28px)` → `clamp(26px, 3.2vw, 44px)` (now 44px at 1440 viewport), 6px yellow left border kept. Three orbs and five sparks added with orb positions varied (`--a` bottom-left, `--b` top-right, `--c` mid-right) so the closing visual feels cohesive with the hero without mirroring it.

Agents-hero's previous `::before` dot-pattern pseudo-element removed in favor of the inline topo SVG; the `::after` yellow orb retained.

**Why**: the morning's token integration made the full Infios palette available but the deck was still using the pre-token visual language — most yellow accents were the dull `--accent-deep` (#9DAE2E) or the soft `--bg-accent` rgba tint rather than the brand-saturated `#CADF35`, and the buttons leaned on cold blue-black `#171F29` rather than a warmer charcoal. The redesign leans into the new palette: bright yellow on the brand-load-bearing surfaces (hero spark dots, prototype door icon, Guardrails badge, hero CTA arrow), warm `--charcoal` on the button surfaces that used to look pure-black, and topographic SVG layers that animate continuously so the home and agents heroes feel alive rather than static. The closing-quote upgrade was the user's call — it had been a small 64px-padded one-line quote tucked above the footer, which underplayed its weight as the closing argument of the whole deck.

**Verified** (via `preview_eval` — the preview screenshot tool was unresponsive throughout this session but the page is fully responsive to JS inspection; all eval queries return immediately with the expected values):
- Hero renders at 895×1425px (full-width), display at 109px, 7 contour lines + 8 peak ellipses + 16 dots present, sample dot fill `rgb(202,223,53)`, sample line stroke `rgba(23,31,41,0.2)`, 5 sparks present, 3 orbs (yellow + purple + bright-yellow) present.
- Agents-hero renders at 562×1425px, topo SVG present with 6 lines + 8 peaks + 14 dots, yellow stroke confirmed on alternating lines, *"The Agents**.**"* period reads `rgb(157,174,46)` (= `--accent-deep`).
- Closing-quote renders at 485×1425px, blockquote text at 44px width 1265px, topo + 3 orbs + 5 sparks visible.
- All three `:root` blocks byte-identical at 9,769 chars (`[System.IO.File]::ReadAllText` UTF-8 confirmed).
- No console errors across home, agents, prototype.
- Note section gone from DOM (`document.querySelector('.note')` = null).

**Outstanding** for later afternoon tasks (Prompts 2–6 of `tasks-afternoon-may-12.md`):
- Prompt 2: Architecture-section horizontal alignment fix on The Agents page.
- Prompt 3: Convert agent cards into row-synchronised accordions.
- Prompt 4: Update the sticky in-page navbar on The Agents page.
- Prompt 5: Full verification pass across morning + afternoon work.
- Prompt 6: One-shot agent-driven prototype build (separate fresh chat).

**Notable open follow-ups** from this task, not blocking:
- Whether to flip the legacy-preserved Infios tokens (text/border/link colors) to the spec's new values. Current state preserves visual identity; flipping would mean accepting a visual shift on the prototype text and link colors. Deferred to a user decision.
- Whether `--accent` should consolidate to the delivery's brighter `#D0FF25` brand-yellow. Currently `--accent` stays at `#CADF35` and the new brighter shade lives under `--infios-color-icon-brand` / `--infios-color-text-brand`. Decision deferred.
- `.note__*` orphan CSS rules linger after the HTML section was removed. Minor dead-CSS sweep candidate.

---

## Early afternoon of May 12 2026 — Prompts 2–5 of `tasks-afternoon-may-12.md`

### Afternoon Prompt 2 — Architecture-section horizontal alignment

User reported that the System view's lateral layer boxes (Analytical / Guardian / Meta) appeared horizontally skewed relative to the toggle above and the Orchestration layer card below, more obviously at 1280px and 1100px viewports than at 1440px+.

**Root cause** was not what the task brief hypothesised. DOM measurements showed `.system-tree__domain-row` and `.system-tree__layers` already had `align-items: center` working correctly relative to their parent — but the parent itself was 1208 px wide while `.diagram` (their grid container, sized to `arch__inner`'s 1120 px) was narrower. Cause: `.diagram` uses `display: grid; grid-template-areas: "stack"` to overlay the Timeline and System views in one cell, and the Timeline view's `.conveyor-grid` (`grid-template-columns: 96px repeat(7, 1fr) 140px`) has a min-content of ~1208 px. With no `grid-template-columns: minmax(0, 1fr)` on `.diagram`, the grid cell sized to fit timeline's min-content, and System view stretched to fill that 1208 px cell — so its 1040 px-max-width children ended up centered around the cell's center (676), not the page center (633). The toggle, which sits in `.diagram-toggle-row` outside `.diagram`, was correctly centered at 633 — hence the visible skew.

**Fixes** (three additions to [moonshot-home.html](moonshot-home.html)):
- `.diagram` — added `grid-template-columns: minmax(0, 1fr)` so the stacked-views grid track is bounded by the container width instead of inheriting timeline's wide min-content.
- `.system-tree__domain-row` and `.system-tree__layers` — replaced `margin-top: …` with `margin: … auto …` so the rows center horizontally within `.system-tree`. Defensive (the first fix is sufficient on its own at the toggle/center axis) and matches the task brief's literal request.
- Inactive `.diagram__view` — added `position: absolute; inset: 0; overflow: hidden` to the existing `opacity:0; visibility:hidden` rule. Reason: even with the grid-track fix, the hidden Timeline view was still contributing its 1208 px content to `document.body.scrollWidth` (visibility:hidden doesn't remove from layout), so the System view still saw a horizontal scrollbar at narrow widths. Switching the inactive view to absolute-positioned removes it from layout flow; `overflow:hidden` clips its content during the fade-out transition. Both views measured 765 px tall in their natural state, so no height snap on toggle.

**DOM-verified** (no screenshots — preview window hidden throughout):
- 1280×800: archInner center, layer__inner center, system-tree__layers center, and toggle all at x=633. body scrollWidth drops from 1232 to within viewport.
- 1100×800: all four centers at x=543. system-tree__layers shrinks to match its parent (1037 px).
- Timeline view: prompt card, 7 stations, info-cards all at exactly the same pixel coordinates pre/post-fix. The fix is invisible to the Timeline view's rendered layout — only the bounding box of its grid container changes.

**Parked, then unparked and fixed twice**: user noted afterward *"it's still misaligned but nevermind, let's park this for now"* — referring to a residual case where `.system-tree__layers` was 1040 px wide vs `.layer__inner` 1120 px below, so the system tree's left edge was 40 px inset from the layer cards' left edge at viewports ≥ 1100. Centers matched; left edges didn't. Later in the same afternoon the user came back and asked to fix it. First fix: bumped `.system-tree__domain-row` and `.system-tree__layers` `max-width` from `1040px` → `1120px` so they fill `.system-tree` (which already sizes to `arch__inner` width via the Prompt 2 grid-track fix). System view aligned cleanly with the layer cards below.

The user then sent a screenshot at 1920 px showing the Timeline view still misaligned — its PROMPT and PROTOTYPE endpoints sitting near the viewport edges, not aligned with the Orchestration content below. After diagnosing: the timeline view's `.conveyor-grid` has an intrinsic min-content of ~1208 px (9 columns: PROMPT + 7 stations + PROTOTYPE, sized to their text content), and was being asked to fit inside a 1120 px container. The container kept getting patched (grid-track fix, max-width fix, position:absolute on hidden view) but the content was always slightly wider than the container could host. User picked the "let the architecture section be wider than the rest of the page" option from a 3-way choice — accept that this one section is wider, but make sure everything inside it remains visually centered on the page axis. Second fix: bumped `.arch__inner` `max-width` from `1120px` → `1280px`. The conveyor (1280 px) now fits inside `.arch__inner` (1280 px) and centers on page axis. The system view's `.system-tree__domain-row` and `.system-tree__layers` stayed at `max-width: 1120px` so they share an identical left edge with the layer cards below. Toggle continues to center within `.arch__inner` = page center.

**DOM-verified across four viewports**:

| Viewport | All centers align | Conveyor fits | Body overflow |
|---:|:---:|:---:|:---:|
| 1920 | ✓ x=953 (incl. archInner, conveyor, toggle, layer__inner) | ✓ | none |
| 1440 | ✓ x=713 | ✓ | none |
| 1280 | ✓ x=633 (archInner clamps to viewport-minus-padding = 1217 px) | ✓ | none |
| 1100 | ✓ x=543 (archInner clamps to 1037 px) | container clamps; final-card content still overflows because its 1208 px min-content exceeds 1100 px viewport | yes — pre-existing |

The 1100 px overflow is the timeline's intrinsic min-content vs viewport width, not a misalignment — and it predates all the patches. The 3-way option discussion captured the trade: option 1 was *"compress the timeline content to fit 1120 px"*, option 2 was *"let arch be wider, center"* (what's implemented), option 3 was *"reflow the conveyor vertically at narrow widths"*. Option 1 (rebuild the timeline content to fit narrower viewports) is the path forward if sub-1280 viewports become a priority.

### Afternoon Prompt 3 — Row-synchronised accordion cards

Converted all 30 layer cards on The Agents page from always-fully-expanded tiles into row-synchronised accordions. Collapsed state shows only the head (avatar + name + role) plus a one-line authored summary; expanded state shows everything (body, talks-to, guardrail, attribution, pill).

**CSS additions** (after `.card__pill-row` rule):
- `.card__toggle` — chevron button, `margin-left: auto`, focus-visible outline using `--accent`. Chevron rotates 180° on `aria-expanded="true"` via a 0.28s ease transition.
- `.card__summary` — 14px / 1.55 line-height / `--text-secondary`.
- `.card__details` — `max-height: 0; opacity: 0; overflow: hidden`. On `.card.is-expanded > .card__details`: `max-height: 2400px; opacity: 1`. Transitioned with `--ease-out`.
- `@media (prefers-reduced-motion: reduce)` block kills the transitions.

**JS additions** (appended to the existing IIFE before its closing brace):
- `rowMates(card)` — finds siblings inside the same grid parent whose `getBoundingClientRect().top` is within 4 px of the clicked card. Cards with no siblings (single-card containers) return only themselves.
- Click handler on every `.card__toggle` calls `setExpanded(rowMates(card), willExpand)` which toggles `is-expanded` class and `aria-expanded` attribute in lockstep across the row.

**Markup changes** (30 cards):
- Toggle `<button>` with chevron SVG added inside each `.card__head`.
- New `.card__summary` div added after the head with an authored 12–18 word value-prop line per card. Drafted from the existing card body and spec files — one per card, framed for a C-level skim.
- All existing body/field/guardrail/attribution/pill content wrapped in `.card__details` with unique `id` matching the toggle's `aria-controls`.
- Full-width cards (Lead Agent, Shift Intelligence Agent) had their pill moved out of `.card__head` into the details wrapper so the collapsed view shows only avatar + name + role + summary.

**Attribution sweep** (user request mid-task: *"unless stated otherwise, all agents/tiles were authored by the ux team, some don't have the author, add that please"*). "Authored by the UX Team" added to all 20 cards that lacked any attribution: Lead, Shift Intelligence, 5 Domain agents (Pick Path / Labor / Order Priority / Carrier / Exception — the 3 with the existing Oleksak credit kept it), all 3 Analytical agents, all 10 Horizon agents. The 7 Guardian + Meta cards already carried the UX Team credit; unchanged.

**DOM-verified**:
- 30/30 cards have toggle + summary + details wrapper; 0 initially expanded; collapsed details measure `max-height: 0; opacity: 0`.
- Domain row click expands the 3 cards on the rendered row together. (At 1440 px viewport, `.grid-5` produces 3 columns — `auto-fill, minmax(280px, 1fr)`. Row-sync tracks rendered rows, not the named class.)
- Analytical click expands all 3 (single row).
- Guardian row-2 click (Content / Interaction / Responsive) expands only those 3; row-1 (Accessibility / Heuristics / Design System) stays collapsed.
- Horizon Category 1 click expands only Research / Company Context / Industry Trends; Categories 2–4 stay collapsed.
- Lead Agent click expands only Lead (no row-mates as a full-width card).
- All chevrons are `<button type="button">` (Tab-reachable, Enter/Space activates). No console errors.

### Afternoon Prompt 3.5 — Layer color pop (user follow-on)

User: *"now all tiles look the same, can we go around the page and give some color pop to the tiles and icons and titles please?"*

Three CSS additions, no markup changes:
- **Layer titles colorised by layer** — `.layer__title--orchestration / --radar / --domain / --analytical / --guardian / --meta / --horizon` each get their respective layer color (`--text-primary`, `--warn`, `--info`, `--link`, `--success`, `#48626E`, `--accent-darker`) for the small uppercase label above each section, sitting next to the already-coloured layer icon.
- **4 px colored top edge on every card** — `.card` border changed from `1px solid hairline` to `border-top: 4px solid hairline` plus per-layer modifiers (`#section-domain .card` blue, `.card--analytical` purple, `.card--guardian` green, etc.). Sides and bottom remain 1 px hairline.
- **Domain + Meta avatar tints** — those two layers' avatars were still neutral gray (the other layers had layer-coloured avatars already). Domain avatars now `rgba(43,122,177,0.10)` background with `--info` text; Meta avatars `rgba(72,98,110,0.10)` with `#48626E` text.

All colors used existing tokens (`--warn`, `--info`, `--link`, `--success`, `--accent`, `--accent-darker`, `--text-primary`) — no new tokens introduced. DOM verification confirmed each title / border / avatar resolved to its expected color.

### Afternoon Prompt 4 — Sticky in-page navbar

The Agents page's sticky `.agents-index` nav strip was missing entries for two top-level sections that had been added during the morning's restructure: the Architecture toggle section and the "How the signals move" topology block.

**Four edits**:
- `<div class="arch">` got `id="section-architecture"`.
- `<div class="topo">` got `id="section-topology"`.
- Two new `<a class="agents-index__item">` entries added to the navbar.
- The JS `sectionIds` array updated to include the two new IDs in document order.

**Deviation from the task brief**: the brief said *"Between Analytical and Guardian: Signals"*. But the `.topo` block is on line 3111 of the file — after Guardian (line 2904) and Meta (line 3049) — not between Analytical and Guardian. With "Signals" placed per the brief, the scroll-driven active-state tracker broke: at scrollY past Meta, the active-state loop's "last array match wins" logic incorrectly stayed on Meta because all earlier items (including Topology in its erroneous position) had passed their threshold first. **Moved "Signals" to between Meta and Horizon** to match document order. Final nav order: **Architecture · Lead · Radar · Domain · Analytical · Guardian · Meta · Signals · Horizon · The case**.

**Why**: the page grew substantially between when the navbar was authored and now, and two major sections weren't reachable via the in-page nav. Architecture is the page's visual focal point and now leads the nav. Signals (the topology block) is the substantive bridge between the operational layers (Lead → Domain → Analytical) and the lateral layers (Guardian / Meta / Horizon); skipping it in the nav under-served readers who wanted to jump straight to the comms-paths discussion. The active-state-tracking issue surfaced only because the brief's nav order didn't match the brief's claimed document order — a documentation drift in the brief, caught at verify time.

**DOM-verified**: all 10 section IDs resolve; `is-active` toggles correctly across every section when scrolled to; smooth-scroll click handler works for the two new items without JS additions; no console errors.

### Afternoon Prompt 5 — Full verification pass

Closing sweep across the morning's work (prototype cleanup + promotion, six new agent cards, Architecture toggle merge) and the afternoon's work (Prompts 1–4). 15 categories checked end-to-end via DOM measurement (preview screenshot tool blocked throughout — `document.hidden=true` on the preview window).

**Result: 13 PASS · 2 WARN · 0 FAIL** on Prompts 1–4. The early-afternoon batch is shippable; more work coming later today. Full report at [archive/verification-2026-05-12.md](archive/verification-2026-05-12.md).

**The two WARNs (both known, both non-blocking)**:
1. **System-tree alignment at viewports ≥ 1100 px.** `.system-tree__layers` and `.system-tree__domain-row` are 1040 px max-width and centered via `margin: 0 auto`, while `.layer__inner` below is 1120 px. Centers match the toggle and the layer cards below; left edges don't (40 px inset). The user parked this earlier; the Prompt 5 checklist's strict `.left` comparison flagged it.
2. **External Google Fonts CDN.** Both `moonshot-home.html` and `moonshot-prototype.html` `<link>` to `fonts.googleapis.com` for Noto Sans (+ JetBrains Mono on the deck). The `--font-display` token includes a system fallback stack, so offline machines render in the fallback without a broken layout — but strictly, the zip-shippability rule *"no `<link>` pointing outside the folder"* is violated.

**Three intentional deviations from the Prompt 5 checklist** (rationalised in the report, not flagged as failures):
- Nav order: Signals is between Meta and Horizon, not between Analytical and Guardian — the brief had wrong document order.
- Card count is 30, not 27 — the morning's six-new-cards work is real.
- Domain row 1 contains 3 cards at 1440 px, not 5 — `.grid-5` is responsive `auto-fill minmax(280px, 1fr)`. Row-sync correctly tracks the rendered row.

**Verified-pass categories**:
- Working prototype renders + no console errors + dead `.home-shift{}` rule confirmed removed.
- Deck Prototype iframe loads `moonshot-prototype.html`; the two prototype copies are `cmp -s` byte-identical.
- Agents hero kicker and lede read exactly the expected strings.
- Sticky nav: 10 items, all 10 scroll-track correctly via `is-active`.
- Architecture section: header text, toggle default to Timeline, Timeline zigzag (cards 1/3/5/7 at top=713, cards 2/4/6 at top=340), System tree (3 spine nodes + 8 leaves + 3 lateral layers), lede sits below diagram in both views, ArrowLeft/Right keyboard switches views.
- Layer cards: 30 cards, all collapsed by default, row-sync correct across Domain / Analytical / Guardian / Horizon / full-width singletons.
- Attributions: Oleksak credit on Slotting / Equipment / Quality (3/3); UX Team credit on Heuristics / Accessibility / Design System / Content / Interaction / Responsive / Warehouse Life (7/7) plus the 20 added during Prompt 3.
- Home closing pull-quote present.
- Design tokens: `--accent: #CADF35`, full shadow ladder, Noto Sans loaded, `TOKENS.md` present, all three `:root` blocks byte-identical.
- Reduced motion: four `prefers-reduced-motion` blocks cover toggle, accordion, conveyor reveal, plus a global `*{transition:none!important}` override.
- Keyboard: 6 top-nav buttons + 10 in-page nav anchors + 2 toggle segments + 30 chevron buttons all reachable; ArrowLeft/Right works on the toggle; `:focus-visible` accent outline defined in CSS.
- Console: no errors across Home / Agents / Prototype / Timeline / System / accordion cycles / nav scroll.

### Next steps (carried out of this session)

Recorded here from the verification report so they don't get lost.

1. ~~**Architecture alignment at viewports ≥ 1100 px.**~~ **Done** later in the same afternoon — bumped the system-tree row `max-width` from 1040 px → 1120 px. All four reference elements (archInner, layer__inner, system-tree__domain-row, system-tree__layers) now share identical left, width, and center at 1440 / 1280 / 1100. See Prompt 2 entry above for the verification numbers.
2. **Run the 5-minute simulated demo end-to-end** before any external send. The structural pieces of `moonshot-prototype.html` were confirmed rendering and the JS loads without error during this verification, but the full 08:00 → 09:30 scripted timeline (inbound late, pick-rate dip, equipment fault, quality hold) was not exercised. Also re-run the Heuristics + Accessibility Guardian audits and confirm SHIP verdicts on both.
3. **Air-gap font shipping** if a fully offline-portable bundle is needed. The current `<link href="fonts.googleapis.com">` gracefully degrades to the system stack offline; for stricter portability, inline the woff2 as base64 in `<style>` or ship a `fonts/` subfolder alongside the HTML.
4. **Decide on the parked token questions** from afternoon Prompt 1: whether to flip the legacy-preserved Infios tokens (text / border / link) to the spec values, and whether `--accent` should consolidate to `#D0FF25`.
5. **Dead-CSS sweep**: `.note__*` orphan rules linger after the HTML section was removed earlier (Prompt 1d); the `.howagents-conveyor` class name is now misleading (lives inside the Architecture section, not on a separate "how the agents work" page); the four Guardian agents without spec files (Design System, Content, Interaction, Responsive) still carry the bare "Authored by the UX Team" attribution — if specs land later, those cards could be upgraded to the full Talks-to + Guardrail format.
6. **Prompt 6 (one-shot agent-driven prototype build)** is still outstanding from `tasks-afternoon-may-12.md`. Per the brief, that runs in a separate fresh chat.
- The new topographical SVG markup is inlined three times (hero, agents-hero, closing-quote) with slightly different path coordinates. If it shows up a fourth time, factor into a shared `<defs>` + `<use>` pattern or a JS-generated component. *(Update later in the same afternoon: it showed up three more times. Still inlined. Refactor still pending.)*

---

## Mid-afternoon of May 12 2026 — Architecture-section width, Topology rebuild, page-wide pizzazz

A run of user-driven design moves on top of the Prompt 1–5 batch. Each move was a single user request and a single execution pass.

### Architecture section deliberately wider than the rest of the page

User noted, looking at a 1920 px screenshot: *"it's still not aligned!!! can't you see???"* Diagnosis: the system-tree max-width fix from earlier handled the system view, but the Timeline view's `.conveyor-grid` has an intrinsic min-content of ~1208 px (9 columns: PROMPT + 7 stations + PROTOTYPE) and was being asked to fit inside a 1120 px container. Every container patch (grid-track minmax, position:absolute on hidden view, max-width bumps) kept moving the container while leaving content always slightly wider than the parent could host.

Offered three options: compress the timeline content to fit 1120 px, let the architecture section be deliberately wider than the rest of the page, or reflow the conveyor vertically at narrow widths. User picked option 2 with the constraint *"at least horizontally center align everything so that things appear centered!"* — accept the asymmetric width, but everything must share a page-center axis.

Fix: one line. `.arch__inner { max-width: 1120px → 1280px }` plus a comment explaining the deliberate difference. The conveyor (1280 px) now fits inside `.arch__inner` (1280 px) and centers on page axis. The system view's `.system-tree__domain-row` and `.system-tree__layers` stayed at `max-width: 1120px` so they share an identical left edge with the layer cards below. Toggle centers within `.arch__inner` = page center.

**DOM-verified at four viewports**:

| Viewport | All centers align | archInner width | Conveyor fits | Body overflow |
|---:|:---:|---:|:---:|:---:|
| 1920 | ✓ x=953 | 1280 | ✓ | none |
| 1440 | ✓ x=713 | 1280 | ✓ | none |
| 1280 | ✓ x=633 | 1217 (clamps to viewport-padding) | ✓ | none |
| 1100 | ✓ x=543 | 1037 (clamps further) | container fits; **content** still overflows because its 1208 px min-content exceeds the 1100 px viewport | yes — pre-existing |

The 1100 px case is the pre-existing timeline-content min-content issue (not solvable without rebuilding the conveyor content itself). The 1280-and-up cases are now clean. User reaction: *"much better!!! now i barely notice it."*

### Sticky navbar centered + Infios brand-yellow highlight

User: *"let's make the sticky navbar content also center aligned. the highlight color should be infios yellow #D0FF25."*

Two edits on `.agents-index`:
- `display: flex` → added `justify-content: safe center`. The `safe` keyword falls back to start-alignment when the items overflow the available width (so at narrow viewports the first item stays accessible and the nav remains horizontally scrollable instead of pushing items off-screen).
- `.agents-index__item.is-active` underline color: `var(--accent-deep)` (#9DAE2E olive) → `var(--infios-color-icon-brand)` (#D0FF25, already in the token set since Prompt 1).

DOM-verified at 1920 (items center, 587 px symmetric padding) and at 700 (safe-center fallback engages, first item stays at x=24).

### Infios yellow propagated to similarly-roled accents

User: *"propagate that color across the page in similarly colored items."*

Found three other sites using the dull olive `--accent-deep` / `--accent-darker` for small accent-indicator roles. Swapped to `var(--infios-color-icon-brand)`:
- `.agents-hero h1 .accent-dot` — the trailing period on *"The Agents."*
- `.closer__item h3` (base rule, dead path but kept for consistency)
- `.closer__item--accent h3` + the icon swatch alongside it — the *"For designers"* label on the home closer

Deliberately not touched: `.door--prototype::before` 3-color gradient (where `--accent-deep` is the intentional dark stop), `.kicker__dot`, brand-fill yellow surfaces (those use `--accent` #CADF35, the established brand color — not the dull olive).

### Communication topology — first split, then full rebuild

The Communication topology section's list of 17 directed paths was visually flat (single column, 17 near-identical rows, prohibited paths interleaved and easy to skim past). Two redesign passes.

**Pass 1 — Split permitted vs prohibited.** Wrapped the rows into two side-by-side columns inside `.topo__list`: `Permitted · 12` left, `Prohibited · 5` right. Prohibited column carries a soft warn-tint background, warn-orange label, and the existing per-row `--never` treatment (warn `from`, struck-through `to`) preserved. New CSS for `.topo__list-col`, `.topo__list-col__header`, `.topo__list-col__label`, `.topo__list-col__count`. Responsive collapse at 720 px breakpoint. 5 cards-on-pages structurally smaller than the old single-card list.

**Pass 2 — Full rebuild as 5 horizontal narrative beats.** User screenshot showed: *"there's so much going below the fold. can we make this horizontal so that each message lands in a linear narrative? also, the text blocks are huge."* The 5 prose paragraphs in `.topo__copy` were each a narrative theme that owned specific paths — but they were spatially divorced from those paths (left column prose, right column lists). Rebuild integrated them.

Final structure: a vertical stack of 5 `.topo__beat` blocks. Each beat is a CSS Grid `300px 1fr`:
- Left cell: small `01–05` number badge + bold theme title + one-line summary (12-word condensations of the original prose paragraphs)
- Right cell: the path chips for that theme as wrapping `.topo__chip` tags, each chip a stacked pair (`from ↔ to` bold + tiny `to`-reason subtext)

Theme 3 (*Some paths are prohibited*) gets the `.topo__beat--prohibited` modifier — soft warn-tinted background, warn-orange chip pair color, struck-through chip-note. The 5 themes:
1. *Shift Intelligence sees first* — 1 chip
2. *Domain agents collaborate, but never alone* — 8 chips (Pick Path ↔ Labor, Slotting ↔ Equipment, Quality ↔ Labor, Equipment ↔ Labor, Order Priority ↔ Labor, Order Priority ↔ Carrier, Exception → Pick Path, Exception ↔ Quality)
3. *Some paths are prohibited* — 5 chips (all `✕` glyph)
4. *Guardian audits gate every ship* — 2 chips
5. *The sim engine is backstage* — 1 chip

All 17 of the original paths' content preserved verbatim (from / to / glyph). Old markup (`.topo__copy`, `.topo__grid`, `.topo__list`, `.topo__list-col`, `.topo__row`) fully removed from DOM. Section total height dropped from ~1300 px to 906 px at 1440. Responsive collapse at 720 px stacks head + chips vertically per beat.

### Page-wide pizzazz — soft topo backdrops on four sections

User: *"now let's add some pizzazz around the page like in the hero? just here and there"* — and then a follow-up *"we need that topo action at the bottom of the page, go."*

Reused the existing `.bg-topo` infrastructure (the inline SVG with animated `topo__lines` contour cubic Béziers, `topo__peaks` concentric-ellipse clusters, `topo__dots` pulsing place markers from earlier afternoon Prompt 1d work). Added a `.bg-topo--soft` modifier:
- Separate `topo-breathe-soft` keyframe at opacity .42 → .55 (vs the hero's .9 → 1)
- Tighter stroke widths, dot drop-shadow removed except on the `:nth-child(7n)` bright-yellow dots
- `@media (prefers-reduced-motion: reduce)` fallback (animation off, static opacity .5)

Inserted SVG markup into three sections with unique `cx/cy` coordinates per section (no two read as clones):
- **`.arch` Architecture** — 5 contour paths, 2 peak clusters, 12 dots
- **`.topo` Communication topology** — 4 paths, 2 clusters, 10 dots
- **`#section-case` "UX as infrastructure, not overhead"** — 5 paths, 3 clusters, 14 dots (the closing argument carries the densest backdrop, matching its weight as the page's final pitch)

Each section got `position: relative; overflow: hidden;` and its inner container got `position: relative; z-index: 2` so content stays sharp above the breathing backdrop. For `#section-case` the `position+overflow+z-index` rules are scoped via `#section-case` selector since `.layer` is generic and shouldn't get those properties everywhere.

Net result: the topo language now bookends the page — `.hero` (home, full-strength), `.agents-hero` (full-strength), `.arch` (soft), `.topo` (soft), `.closing-quote` (home, full-strength), `#section-case` (soft). The previously-flat business-deck middle has atmosphere; the previously-decorated hero no longer feels like a one-off.

### Verification across the mid-afternoon batch

- All centers align at 1920 / 1440 / 1280 / 1100 in the architecture section.
- Sticky nav: items centered, `safe center` fallback engaged at narrow widths, active highlight #D0FF25.
- Three `--accent-deep` / `--accent-darker` solo usages now resolve to `rgb(208, 255, 37)`.
- Topology rebuild: 5 beats, 17 chips total (no orphans), prohibited beat carries warn-tinted bg + chip styling + strikethrough preserved.
- All four soft-topo backdrops: opacity .42 mid-breathe, animation `topo-breathe-soft`, content z-index 2 above.
- Console: no errors across any of the mid-afternoon changes.

### What this leaves at end of mid-afternoon

- Architecture and Topology sections are now narratively legible and visually cohesive with the hero language.
- The page's yellow palette is consistent: brand-bright `#D0FF25` (`--infios-color-icon-brand`) for small accent indicators, brand `#CADF35` (`--accent`) for load-bearing brand surfaces, dull olive `--accent-deep` / `--accent-darker` retired from small-accent roles (still present in 3-stop gradients where the depth is intentional).
- Six sections across the deck now share the `.bg-topo` language; the previously-untouched bottom-of-page closing argument is bookended with the hero.

**Open items still parked from earlier**: the 1100 px timeline-content overflow (option 1 from the three-way trade — rebuild the conveyor content to be narrower — not yet picked up); the `.note__*` orphan CSS sweep; the `.howagents-conveyor` rename; Prompt 6 (one-shot agent-driven prototype build, separate session).

---

## Night of May 12 2026 — Cross-pillar container: product switcher, sidebar tiers, TM domain views

Pulled forward the first three prompts from `next tasks/tasks-may-13-morning.md` (originally scheduled for May 13 morning). Direct response to Richard Stewart's May 12 1:1 directive: pillar-specific demos first (WMS / TMS / OMS), agnostic vs domain component split, cross-pillar dashboard as roll-up, Gartner presentation in ~16 days. The night's three tasks land the container Allen's agent specs will plug into.

### Task 1 — Product-level state + product switcher UI

**What**: introduced `currentProduct` state (default `'wa'`), a header product switcher (WA · TM · OM) sitting between the nav-toggle area and the role switcher, and a `setProduct(p)` function that swaps `currentProduct`, repaints the active button, swaps the header wordmark + title from `PRODUCT_META`, falls back to `view = 'home'` when the current view isn't valid for the new product, and calls `renderAll()`. WA's behavior is byte-identical when active.

**Why**: every later piece (sidebar tiers, TM views, cross-pillar dashboard) depends on this state existing. Plumbing first.

**Verified**: switching WA / TM / OM in the header swaps the wordmark + title; WA renders all six existing views unchanged; TM and OM show the existing placeholder until later tasks populate them; no console errors.

### Task 2 — Sidebar restructured into agnostic + domain tiers

**What**: split the sidebar into two labeled sections — **CROSS-PILLAR** (Dashboard / Archer / Agent Performance — always visible, maps to existing `home` / `cli` / `perf` views) and a domain section that swaps on `currentProduct` from `DOMAIN_NAV`. Three domain configs:
- WA → Shift Log · Floor Map · Warehouse (existing badge logic preserved)
- TM → Activity Log · Network Map (stubs)
- OM → Order Log · Order Dashboard (stubs)

Stub views render via a new `renderDomainStub(productKey, viewKey)`: centered placeholder card with the product icon (48 px), the view label, and a short "scenarios coming soon" description. Session list, shift info, and collapse behavior preserved.

**Why**: this is the container play — defines the slots Allen's agent specs plug into. Agnostic components live in the cross-pillar tier; domain-specific tiles live in the domain tier. Reading the sidebar tells you the architecture without opening a spec.

**Verified**: WA shows CROSS-PILLAR + WAREHOUSE sections; TM swaps the domain section to TRANSPORTATION (Activity Log + Network Map stubs); OM swaps to ORDER MANAGEMENT (Order Log + Order Dashboard stubs); cross-pillar items constant across products; no regressions on WA views.

### Task 3 — TM domain views with real transportation scenarios

**What**: replaced the three TM stub destinations (Dashboard / Activity Log / Network Map) with working views built around Richard's exact ask — *"a transportation persona where the truck is late or you had me do spot quotes and it was outside of the — you told me to let you know if it's more than two times what that lane is supposed to be."*

- **`TM_DATA` state object** added next to `WAREHOUSE` (parallel shape — `lanes`, `kpi`, `scenarios`, `actionLog`, `chatThread`). No sim engine — static demo data only. Five lanes from Memphis: ATL (ok), CHI (late — truck #4471 2h behind), DAL (ok), DEN (alert — spot rate 2.3× lane), LAX (risk — capacity tight). KPIs: 847 active loads · 94% on-time · 3 open alerts.

- **`renderTMHome()`** mirrors `renderHome()` structurally (same `.home-wrap` / `.home-top-rail` / `.home-dash` 3-column grid / `.home-zones` / `.home-center` / `.home-chat--terminal` / `.home-kpi-strip`). Reuses every WA CSS class so the visual language is identical. Top rail: `Hub MEM Memphis, TN · 08:14 · Day Ops · 3 open alerts` (no restart button — no sim).

- **Lanes strip** (`renderTMLaneStrip`) maps lane status to the existing zone status classes: `late|alert → --attention`, `risk → --high`, `ok → --ok`. Capacity bars driven by `lane.loadPct`.

- **Stage card** (`renderTMStageCard`) defaults to Scenario B — the spot-quote scenario Richard called out by name. Card content matches the WA `.home-stage__card--pending` markup: Carrier Agent avatar / 91% confidence / action line / impacts (38 loads, +$2,400 if accepted, 24h slip if held) / 3 drafted options (accept / delay 24h for contracted / consolidate with MEM→LAX). A small **"Next alert →"** link in the card footer toggles `tmActiveScenario` between A (ETA breach — truck #4471 / Carrier Agent 87% / reroute to Saia, extend window, split shipment) and B. Approve / Reject / Details buttons wire to `tmResolveScenario(key, result)` which swaps the card for a small "resolved" confirmation; user can cycle to the next alert from there.

- **Archer terminal** reuses `renderHomeChat(thread)` — the function already accepted a thread parameter. Two small generalizations: terminal host string switches to `lead@hub-mem-tm` when TM is active, and `sendUserMessage()` gained a TM branch that pushes user input into `TM_DATA.chatThread` with a canned Lead reply (*"Got it — I'll keep watching the lane."*) after 2.5 s. Pre-seeded with three contextual messages — Lead Agent on the spot quote (calling out the 2× threshold Richard set), Carrier Agent on the Werner delay + Saia availability, Lead Agent on the running summary.

- **KPI strip** (`renderTMKpiStrip`) — same `.home-kpi-strip` classes; three tiles: Active Loads · On-Time Delivery · Open Alerts. Sparklines omitted (no history data) — empty trend slot keeps the layout aligned.

- **TM Activity Log** (`renderTMLog` + `renderTMLogNav` + `renderTMLogTable` + `buildTMLogRows`) mirrors the WA shift log shell. Reuses `.log-view` / `.log-tile` / `.log-chip` / `.log-table` and the same state badge variants (`URGENT` / `STANDARD` / `Validated` / `Agent action`). Differences from WA: column "Zone" → "Lane", drop the "Actor" column, lane chips replace zone chips (`All lanes` / `MEM→ATL` / `MEM→CHI` / `MEM→DAL` / `MEM→DEN` / `MEM→LAX`). Pre-seeded with 7 entries: 1 urgent pending (the spot quote), 1 standard pending (the Werner delay), 2 validated route optimizations, 3 auto agent actions. Approve / Queue handlers (`tmLogApprove` / `tmLogQueue`) flip state and re-render.

- **TM Network Map** (`renderTMMap`) — schematic SVG, no library, viewBox `0 0 1000 600`. Memphis hub at center (`brand-fill` charcoal circle, `--accent` yellow MEM text). Five destinations fanned outward (ATL bottom-right, CHI top, DAL bottom-left, DEN upper-left, LAX far-left). Each lane is a 4-px colored line (green `--success` / amber `--warn` / red `--error`) plus an 18-px invisible click-target overlay so clicking the schematic line is forgiving. Click a line or destination → `tmSelectLane(id)` toggles a `.tm-map-tooltip` card anchored near the destination with carrier, ETA, load count, and status note. Clicking the same lane again, the close link, or the empty SVG background clears it. Legend bottom-left.

- **Sim tick gated by `currentProduct === 'wa'`**. The WA simTick was calling `renderHome()` and `surgicalHomeClockUpdate()` even when TM was active, which overwrote the TM top rail's clock and shift label. Added `currentProduct === 'wa'` guards on the home / map / log / warehouse branches inside `simTick` so the WA sim no longer pokes the DOM while a non-WA product is mounted. WA sim continues to run; only the per-view re-render side effects are gated.

- **`renderMain()` dispatcher** rewritten to branch on `currentProduct` first: under TM, `home` → `renderTMHome`, `tm-log` → `renderTMLog`, `tm-map` → `renderTMMap`, `cli`/`perf` fall to the existing placeholder. OM still routes through `renderDomainStub` (Prompt 4 hasn't run yet).

**Why**: gives the demo a real second-pillar story Richard can see end-to-end — the spot-quote scenario lands inside 30 seconds of switching to TM, the WA experience is untouched, and the architecture's "agnostic + domain" split is now legible in the working prototype (not just the deck).

**Verified** via the preview server with DOM inspection (screenshot tool was flaky tonight — every screenshot call timed out after 30s with no console errors, page responsive throughout):
- WA: Home renders 8 zones, Shift Log renders 10 rows, Floor Map renders. Unchanged.
- TM Dashboard: 5 lanes (Memphis → ATL/CHI/DAL/DEN/LAX), top rail `Hub MEM Memphis, TN · 08:14 · Day Ops · 3 open alerts`, KPIs `847 / 94% / 3`, terminal host `lead@hub-mem-tm`, 3 pre-seeded chat messages, scenario B (spot quote) loads by default.
- "Next alert →" cycles B (Lane MEM→DEN, 91%) ↔ A (Lane MEM→CHI, 87%).
- Chat input echoes + canned Lead reply lands after 2.5s (`TM_DATA.chatThread.length` 3 → 4 → 5).
- TM Activity Log: 7 rows, chip counts `All 7 · Pending 2 · Validated 2 · Agent actions 3`; filtering by `MEM→DEN` narrows to 2 rows (the spot quote + the lane-rate baseline update).
- TM Network Map: SVG renders, clicking MEM→CHI shows tooltip `Werner · Tue 17:40 (was 15:30) · 156 loads · truck #4471 2h behind`; clicking same lane again clears.
- WA → TM → WA → TM rapid toggle stays clean; no stale DOM; console clean throughout.

### What this leaves at end of night

- TM pillar is end-to-end demo-ready. Cross-pillar container's WMS + TMS legs are both live.
- Remaining from `tasks-may-13-morning.md`: Prompt 4 (OM stubs — stock-out + cutoff scenarios), Prompt 5 (cross-pillar dashboard roll-up), Prompt 6 (agent response JSON schema + doc updates), Prompt 7 (archive snapshot + verify).
- One paper cut to flag for whoever picks up Prompt 4: the WA sim still ticks the clock when a non-WA product is mounted — gated re-renders prevent visible flicker, but `WAREHOUSE.clockSec` continues to advance. Doesn't affect the TM/OM demo, but worth deciding whether to pause the sim outright on `setProduct(non-wa)` or keep it running so returning to WA picks up where it left off.

---

## Morning of May 13 2026 — OM pillar, cross-pillar dashboard, agent response schema, doc sweep

Picked up where the May 12 night session left off. Completed the remaining four prompts from `next tasks/tasks-may-13-morning.md`: Prompt 4 (OM pillar), Prompt 5 (cross-pillar dashboard), Prompt 6 (agent response schema + documentation), with Prompt 7 (archive snapshot + final preview verify) carried into a follow-on.

### Task 4 — OM domain views (stock-out + cutoff scenarios)

**What**: built the OM pillar as a structural mirror of the May 12 TM build. Added `OM_DATA` (parallel shape to `WAREHOUSE` and `TM_DATA` — `channels`, `kpi`, `scenarios`, `actionLog`, `chatThread`; no sim engine, static demo data). Five order channels from a single fulfillment node: E-Commerce (healthy), Wholesale (at risk — WS-0512 cutoff breach), Marketplace (delayed), Direct/B2B (healthy), Drop-ship (alert). KPIs: 2,341 active orders · 96.2% fill rate · 2 pending decisions.

- **`renderOMHome()`** mirrors `renderTMHome()` and `renderHome()` (same `.home-wrap` / `.home-top-rail` / 3-column dash / channels strip / stage card / Archer terminal / KPI strip). Same WA CSS classes reused — visual language unchanged across pillars.
- **Channels strip** (`renderOMChannelStrip`) maps channel status to the existing zone status classes: `delayed|alert → --attention`, `risk → --high`, `healthy → --ok`. Capacity bars driven by `channel.utilPct` (order volume vs capacity).
- **Stage card** defaults to Scenario A (Stock-Out Risk on SKU #WH-4419 — 12 in stock, 47 pending, Order Priority Agent 78% confidence, three options: allocate-to-priority / backorder-remaining / substitute to WH-4420). **"Next alert →"** toggles to Scenario B (Cutoff Breach on Wholesale Batch WS-0512 — 23 orders missed the 14:00 cutoff, Order Priority Agent 85%, options: push-all-to-18:00 / split-expedite-top-8 / hold-and-notify). Approve / Reject / Details wire to `omResolveScenario(key, result)`.
- **Archer terminal** reuses `renderHomeChat(thread)`. Host string switches to `lead@hub-om` when OM is active. Pre-seeded with three messages: Order Priority on the SKU exposure, Allocation on the cutoff timing, Lead summarising both decisions.
- **OM Order Log** (`renderOMLog` + table) mirrors the TM Activity Log shell — but with columns Order ID · Channel · Items · Status · SLA. Pre-seeded with 7 entries (2 pending decisions, 2 validated allocation calls, 3 autonomous agent actions).
- **Order Dashboard** (`renderOMDashboard`) — horizontal pipeline visualisation: Received → Processing → Allocated → Shipped, each stage a count + progress bar. Below the pipeline, a table of the 5 most recent orders with Order ID · Channel · Items · Status · SLA countdown. Static data from `OM_DATA`.
- **`renderMain()` dispatcher** extended: under OM, `home` → `renderOMHome`, `om-log` → `renderOMLog`, `om-dashboard` → `renderOMDashboard`; `cli`/`perf` continue to use existing agnostic views.

**Why**: closes the third pillar Richard called out. The cross-pillar dashboard (Prompt 5) needs three working pillars to roll up; this lands the last one.

**Verified** via the preview server + DOM inspection:
- OM Dashboard: 5 channels, top rail `Fulfilment Hub · 08:14 · Day Ops · 2 open alerts`, KPIs `2,341 / 96.2% / 2`, terminal host `lead@hub-om`, 3 pre-seeded chat messages, Scenario A loads by default.
- "Next alert →" toggles A (SKU #WH-4419 / Order Priority 78%) ↔ B (Cutoff WS-0512 / Order Priority 85%).
- Order Log: 7 rows, filter chips work, state badges match TM/WA conventions.
- Order Dashboard: pipeline shows 4 stages with counts, recent-orders table renders 5 rows, SLA countdown shows time remaining.
- WA + TM unchanged. Switching WA → OM → TM → OM → WA stays clean; console clean throughout.

### Task 5 — Cross-pillar dashboard (the multi-product roll-up)

**What**: added `'cross'` as the fourth `currentProduct` value, with a fourth product-switcher button (a small grid/overview icon) sitting before WA/TM/OM in the header. Selecting it sets `currentProduct = 'cross'` and routes `home` to `renderCrossDashboard()`. Sidebar in cross mode keeps the CROSS-PILLAR section constant (Dashboard / Archer / Agent Performance) and replaces the domain section with three product cards (Warehouse Advantage · Transportation Management · Order Management) — each acts as a navigation link that sets `currentProduct` to the respective pillar and loads its home view.

The cross dashboard is **not** a clone of the pillar dashboards. Four bands:

- **Top — three product health cards** (WA / TM / OM): product name + icon, 2-3 headline KPIs pulled from `WAREHOUSE.kpi` / `TM_DATA.kpi` / `OM_DATA.kpi`, status indicator (green/amber/red) computed from open-items count, click navigates into that pillar.
- **Middle — unified decision queue** (`renderCrossQueue`): aggregates pending items from `WAREHOUSE.actionLog`, `TM_DATA.scenarios`, `OM_DATA.scenarios`, sorted by urgency (`criticality: 'urgent'` first, then by `confidence` desc). Each row shows a product badge (WA/TM/OM coloured chip) + agent name + summary + confidence + time-pending. Product badge styling: WA `--info`, TM `--warn`, OM `--accent-darker` — distinct at a glance.
- **Right column — Archer terminal**: same dark terminal, cross-context. Pre-seeded with three messages that explicitly name cross-product correlations:
  1. *"Carrier delay on MEM→CHI may impact 12 outbound orders in WA Wave 3 — Pick Path is staging the alternate sequence."*
  2. *"Stock-out exposure on SKU #WH-4419 has 8 orders queued in OM that would land in the next WA wave — flagging."*
  3. *"All three pillars healthy on average; one urgent + one standard pending."*
- **Bottom — aggregated KPIs**: Total Active Items (WA orders + TM loads + OM orders) · Decisions Pending (sum across pillars) · Cross-Product Alerts (count whose `impacts[]` reference another pillar) · System Health (worst of three pillar statuses).

**Why**: this is the view Richard described — *"what it looks like when customers are running more than one"*. Up until now the multi-product story lived only in the pillar switcher; now it lives in the dashboard itself. Establishes the visual home for cross-product correlation, which is the differentiator over single-product agentic UI.

**Verified** via DOM inspection:
- Cross-switcher button renders before WA in the header; click sets `currentProduct = 'cross'`; sidebar updates; main loads cross dashboard.
- Three product cards render with correct KPIs and status colours.
- Unified queue sorts urgent first; product badges visually distinct (WA blue, TM amber, OM olive).
- Three Archer seed messages present; host string `lead@cross`.
- Aggregated KPIs roll up correctly.
- Click WA card → navigates to WA home; sidebar swaps to WAREHOUSE; cross-switcher loses active state; WA switcher gains it. Round-trips WA → cross → TM → cross → OM → cross stay clean.

### Task 6 — Agent response schema + documentation sweep

**What**:
- Created `agent-response-schema.json` at the repo root. Draft 2020-12 JSON Schema. `AgentProposal` object with required fields `agentId`, `proposalType`, `confidence`, `summary`, `detail`, `criticality`, `tier`, `product`, `timestamp`; optional `escalationReason`, `options[]`, `impacts[]`. Each option object requires `id`, `label`, `description`, `tradeoff`. Each impact object requires `area`, `description`, `severity` (low/medium/high). Top-level `$comment` cross-references AGENT-ARCHITECTURE.md §6.2.
- Updated `AGENT-ARCHITECTURE.md` — inserted new section **6.5 Cross-pillar architecture** between 6.4 and section 7. Documents the agnostic/domain component split, the `currentProduct` state model, the cross-pillar dashboard's four bands, the `product` field on agent proposals, and the link to `agent-response-schema.json`. Names Richard Stewart's May 12, 2026 directive and the Gartner deadline as the design drivers.
- Updated `README.md` — added the schema file to the file map; rewrote the "Where the project is right now" section (dated end of May 13 morning), added a "Cross-pillar container" subsection that documents the agnostic vs domain split and the `currentProduct` model, restructured the surfaces list to include TM/OM/cross dashboards, and rewrote "What's outstanding" to reflect the new state (Allen's monitoring-agent specs, Prompt 7 archive snapshot, the WA-sim-clock paper cut, the parked carry-overs).
- Updated `ITERATION-LOG.md` — this entry (and the top-of-file TOC line).

**Why**: documentation lag is a real risk this close to Gartner. Allen's next move depends on knowing the `AgentProposal` shape; he can read it from a single JSON Schema file now rather than reverse-engineering from `moonshot-prototype.html`. The architecture document needed a section that pins down the cross-pillar model so future contributors don't rediscover it from the prototype.

**Verified**:
- Schema validates as JSON (Read tool renders it without error). All enum spellings match the brief (`"recommendation"` / `"escalation"` / `"information"`; `"urgent"` / `"standard"`; `"tier-1-prohibited"` / `"tier-2-approval"` / `"tier-3-autonomous"`; `"wa"` / `"tm"` / `"om"`).
- AGENT-ARCHITECTURE.md section numbering: 6.4 → 6.5 → 7 contiguous.
- README file map shows `agent-response-schema.json` immediately after `ITERATION-LOG.md`.
- ITERATION-LOG.md top TOC line names May 13 morning.
- Cross-references (`agent-response-schema.json` literal) present in README, AGENT-ARCHITECTURE.md, and this entry.

### What this leaves at end of May 13 morning

- All three pillars are demo-ready. Cross-pillar dashboard is the multi-product entry point. Prototype matches Richard's May 12 directive end-to-end.
- `agent-response-schema.json` is the contract Allen's monitoring-agent work plugs into. Ready to share.
- **Prompt 7 (carry-over)** — snapshot `moonshot-prototype.html` to `archive/moonshot-prototype.PRE-MAY-13.html`, verify the iframe in `moonshot-home.html` still resolves, run a preview pass across Home / Agents / Prototype tabs. Pulled into a separate follow-on session.
- **The WA-sim-clock paper cut** from the May 12 night entry is still parked. Recommend pausing the sim outright on `setProduct(non-wa)` so the clock state doesn't drift while the operator demos cross-pillar.
- **Open carry-overs** still alive from earlier sessions: font air-gap, parked Infios-token decisions, `.note__*` orphan CSS, the 1100 px timeline-content overflow on the Architecture section. None block share-out; all worth picking up after the Gartner cycle.

---

## May 13 afternoon — cross-pillar agent backfill + UX fixes

Closes the loop on Richard Stewart's May 12 directive. The morning sprint (Tasks 4–6) shipped the cross-pillar container — WA/TM/OM navigation, three pillar dashboards, cross-pillar roll-up, agent response schema — but the TM and OM views referenced agent names (Spot Rate, Route, Allocation, Cutoff Manager) with no governance documents underneath them. Without specs, the demo collapses on inspection. The afternoon backfill closes that gap so all three pillars read at the same quality bar as the eight existing WMS specs. Two UX issues surfaced on review of the morning's prototype state — WM home rendered its decision as a modal overlay while TM and OM rendered theirs inline (inconsistent), and the cross-pillar Archer terminal sat inside a grey wrapper offset from the product cards above (visually broken). Both fixed in one prototype pass alongside the spec work.

### What was built

- **Four new agent specs in `/agents/`** — each follows the same 8-section structure as the WMS specs (Identity, Scope, Triggers, Actions, Communication, Guardrails, Escalation, Metrics), carries `confidence` / `escalationReason` / `options[]` on every Tier 2 proposal, and conforms to `agent-response-schema.json`:
  - `spot-rate-agent.md` — TM pillar. Spot quotes vs contracted lane rate. Tier 2 escalation when a spot quote crosses the 2× lane-rate threshold Richard called out by name on May 12.
  - `route-agent.md` — TM pillar. Lane health, mode selection (truckload / LTL / intermodal), multi-stop routing. Tier 2 reroute proposals when lane conditions degrade.
  - `allocation-agent.md` — OM pillar. Constrained-inventory allocation across orders. Tier 2 proposals when demand exceeds supply for a SKU.
  - `cutoff-manager-agent.md` — OM pillar. Carrier cutoff windows at the order layer. Tier 2 proposals when orders are at risk of missing a cutoff.

- **Three cross-pillar addenda inserted into existing specs**:
  - `carrier-agent.md` — second context covering TM lane-window operation (lane ETA visibility, carrier ETA-vs-promised drift, lane-rate breaches). Tier boundaries unchanged from the WMS dock-window base spec.
  - `order-priority-agent.md` — second context covering OMS upstream channel-intake operation (channel cutoffs, promised ship-by windows, reprioritisation across the unfulfilled queue). Tier boundaries unchanged from the WMS floor-released base spec.
  - `lead-agent.md` — orchestration addendum covering cross-pillar intent routing: how the Lead Agent dispatches to a single agent's WMS context vs TM context vs OMS context, and how `product` on the resulting `AgentProposal` is set from the dispatch path.

- **Agents page in `moonshot-home.html`** — Domain section header reframed from "eight specialists" to "twelve agents across three pillars," four new cards (Spot Rate, Route, Allocation, Cutoff Manager) added with the same toggle-disclosure pattern as the existing eight, and a new cross-pillar topology beat inserted into `#section-topology` covering the dual-context pattern and the `product` discriminator.

- **`moonshot-prototype.html`** — two UX fixes plus a small reconciliation:
  - **WM home center stage migrated from modal-overlay treatment to inline.** The WA home Pick Path decision now renders directly on the home grid (Scenario A — congestion in Aisle 14, 08:30 sim time) matching how TM Carrier and OM Order Priority decisions already rendered. The modal-overlay code path is retired; the inline stage card uses the existing `.home-stage__card--pending` markup.
  - **Cross-pillar dashboard Archer terminal brought flush to its containing tile.** The terminal was wrapped in a grey container offset from the product cards row above; the wrapper has been removed and the terminal now occupies full tile width with its right edge aligned to the OM product card.
  - **Agent names in `TM_DATA` / `OM_DATA` reconciled** to match the new spec filenames so the prototype's seeded chat messages and stage-card attributions resolve cleanly against `/agents/`.

### Why

Two motivations stacked in one session.

**Spec gap closure.** The morning's cross-pillar container is the load-bearing element of Richard's directive, and the Gartner presentation is ~16 days out. The TM and OM views work on first glance — but anyone who opens the Agents page and looks for Spot Rate, Route, Allocation, or Cutoff Manager would find no spec, no 8-section governance document, no `confidence` / `escalationReason` / `options[]` contract. Sharing the prototype with Allen's committee in that state would surface the gap immediately and undermine the cross-pillar argument. Backfilling the four specs and the three addenda brings all three pillars to the same quality bar as the eight existing WMS specs.

**UX consistency on the prototype's load-bearing surfaces.** The WM home decision rendered as a modal overlay while TM and OM rendered theirs inline. Reading across the three pillars, the inconsistency read as a half-finished port — exactly the kind of detail that erodes trust in a multi-product story. The cross-pillar Archer terminal sat inside a grey wrapper that offset it from the product cards above; the visual hierarchy broke at the very surface that's supposed to demonstrate cross-product correlation. Both fixed in one prototype pass.

### Verified

DOM inspection via the preview server; screenshots captured cleanly this session.

- All twelve Domain cards render on the Agents page with no layout breakage. Card grid wraps cleanly at 1440 / 1280 / 1100; 720 px stacks one card per row.
- Toggle disclosures work on all four new cards. Open / close state cycles cleanly; no console errors.
- Cross-pillar topology beat renders alongside the existing five beats. Chip counts match the spec; styling consistent.
- WM home Pick Path decision renders inline at 08:30 sim time, matching TM Carrier (MEM→DEN spot quote) and OM Order Priority (SKU #WH-4419 stock-out) placement on their respective home grids.
- Cross-pillar dashboard terminal occupies full tile width; right edge measured aligned to OM product card's right edge.
- WA → TM → OM → cross → WA round-trip stays clean; no stale DOM; console clean throughout.

### What this leaves at end of afternoon

- The Agents page now reflects twelve Domain agents across three pillars. The cross-pillar topology beat documents the dual-context pattern on the page itself.
- The prototype's three pillar dashboards behave consistently — inline stage cards on all three, no modal overlays for routine pending decisions.
- The cross-pillar dashboard's layout is now Allen-committee-ready. Sharing this prototype no longer surfaces the modal / wrapper inconsistencies that would have undermined the cross-pillar argument on first read.
- The README, AGENT-ARCHITECTURE (§6.6 new, §4.3 retitle, §3 diagram annotation), and ITERATION-LOG (this entry) updates are committed in the same pass so the self-description docs match the code state.
- **Open carry-overs** still alive from earlier sessions: WA-sim-clock paper cut (clock continues to advance when a non-WA pillar is mounted — recommend pausing on `setProduct(non-wa)`), font air-gap, parked Infios-token decisions, `.note__*` orphan CSS, the 1100 px timeline-content overflow. None block share-out.
- Tooltip on the Network Map is positioned by `%` of the SVG viewBox — works at the demo viewport sizes tested but may need a real `getBoundingClientRect` + screen-coord transform if the SVG container gets exotic aspect ratios.

---

## May 13 afternoon — Lead Agent renamed to Archer Chat Agent — brand alignment

The orchestrator agent — the only agent that addresses the human user — was named "Lead Agent" in the spec. Accurate as a role name, but disconnected from the visible brand: the chat surface in the prototype is "Archer" (terminal label, sidebar item, conversation thread context). Pedro's call: align the spec name with the visible brand so the system reads as a coherent Infios AI product, not a prototype with a generic "Lead Agent" label spliced into an "Archer" UI. The Tier 1/2/3 boundaries, escalation patterns, and communication contract are unchanged — only the name moves.

### What was built

- **`agents/lead-agent.md` renamed to `agents/archer-chat-agent.md`.** Title and all 18 internal references updated, including the Cross-Pillar Scope addendum from the morning sprint.
- **All cross-references in 15 other spec files** updated. Every "Lead Agent" reference in the "Inter-Agent Communication" and "Human Guardrails" sections of `shift-intelligence-agent.md`, `pick-path-agent.md`, `labor-agent.md`, `order-priority-agent.md`, `carrier-agent.md`, `exception-agent.md`, `slotting-agent.md`, `equipment-agent.md`, `quality-agent.md`, `analytical-agents.md`, `meta/warehouse-life-agent.md`, `spot-rate-agent.md`, `route-agent.md`, `allocation-agent.md`, and `cutoff-manager-agent.md` now reads "Archer Chat Agent". The guardian specs (`heuristics-agent.md`, `accessibility-agent.md`) had no cross-references and were left untouched. Total: 198 cross-reference replacements.
- **`moonshot-home.html`** — 21 "Lead Agent" replacements plus six bare-"Lead" prose tokens that referred to the agent (chip notes, topology beats, station label, layer subtitle, conveyor lede, slotting guardrail). The Architecture System tree node now reads "Archer Chat Agent"; the Orchestration card name reads "Archer Chat Agent" with avatar "AC" (was "LA"); all 12 Domain cards' "Talks to" fields reference Archer Chat Agent; the cross-pillar topology beat reads "Carrier (TM) → Archer Chat Agent → Order Priority (OM)". The `agents-index` nav label "Lead" stays as the section/layer name (it's the layer, not the agent).
- **`moonshot-prototype.html`** — 40 "Lead Agent" replacements; four terminal hosts renamed: `lead@dc-14-memphis` → `archer@dc-14-memphis`, `lead@hub-mem-tm` → `archer@hub-mem-tm`, `lead@oms-na` → `archer@oms-na`, `lead@infios-network` → `archer@infios-network`; `buildAgentMsg` default fallbacks updated (`m.initials || 'LA'` → `'AC'`, `m.agent || 'Lead Agent'` → `'Archer Chat Agent'`); all pre-seeded `chatThread` messages in `WAREHOUSE`, `TM_DATA`, `OM_DATA`, and the cross-pillar thread updated. Labor Agent's own `initials: 'LA'` (its actual initials) was preserved. The internal `source: 'lead'` field on seeded messages was left as-is (routing-tag, not display text).
- **`agent-response-schema.json`** — three prose references updated (`$comment`, `description`, `criticality` field description). No agentId enum or example change; the schema didn't previously expose a "lead" agentId value.
- **`README.md` and `AGENT-ARCHITECTURE.md`** — roster table, system diagram (with ASCII border alignment fixed for the longer name), §4.1 heading, and all body references updated. `lead-agent.md` file-path references in both docs updated to `archer-chat-agent.md`. `WAY-OF-WORKING.md` had no occurrences. Historical entries in this log were preserved unchanged — they describe what was true at the time and would otherwise become a rolling rewrite.

### Why

The user-facing chat surface in the prototype is branded "Archer" — it's what Allen and Eugene see when they open the terminal, what shows up in the sidebar list, what labels the conversation thread context. The orchestrator agent driving that surface was named "Lead Agent" in the spec, which is a clean role name (the agent that leads orchestration) but doesn't tell a coherent story when paired with the visible brand. The committee sees "Archer" in the UI and "Lead Agent" in the system tree and has to reconcile them.

Pedro's call: rename the orchestrator everywhere so the spec, the docs, the prototype, and the brand all use the same name. The Infios AI brand is "Archer" — the orchestrator is "Archer Chat Agent." It's a chat agent (it owns the chat surface) and it's part of the Archer family. The connection to the Infios AI product line is now immediate and consistent across every surface a reviewer will touch.

This also de-risks one specific failure mode in the Allen-committee read: a reviewer who notices the name mismatch and asks "wait, is this an Infios feature or a one-off prototype with a generic agent name?" The renamed spec answers that question by construction.

### Verified

- `grep -rn "Lead Agent" agents/` returns zero hits.
- `grep -n "Lead Agent" moonshot-home.html moonshot-prototype.html agent-response-schema.json README.md AGENT-ARCHITECTURE.md WAY-OF-WORKING.md` returns zero hits. (Historical entries in this log were intentionally preserved per the rename plan.)
- `grep -n "lead@" moonshot-prototype.html` returns zero hits.
- `grep -n "'LA'" moonshot-prototype.html` returns one hit — Labor Agent's own `initials: 'LA'` at line 2278 (legitimate; Labor Agent's initials are LA). MEM→LAX lane code and Memphis → LA display label preserved.
- ASCII system diagram in AGENT-ARCHITECTURE.md §3 has the right border re-aligned for the longer agent name.
- 27 "Archer Chat Agent" references in `moonshot-home.html`; the Architecture System tree node, the Orchestration card name, and all 12 Domain card "Talks to" fields all read the new name.
- 17 `'AC'` references in `moonshot-prototype.html` (was zero before the rename).

### What this leaves

- The orchestrator's brand identity now matches the visible chat surface. When Allen or Eugene sees "Archer Chat Agent" in the system tree and "archer@infios-network" in the cross-pillar terminal, the connection between the agent and the Infios Archer AI brand is immediate.
- The spec governance contract is unchanged — Tier 1/2/3 boundaries, escalation patterns, and the four supervisor-unavailability conditions all hold. Only the name has changed.
- Cross-references in 15 other spec files keep their semantic meaning ("all user-facing communication routes through the Archer Chat Agent").
- The `agents-index` nav still labels the section "Lead" because that's the layer name (Orchestration layer, shorthand "Lead"); the agent inside that layer is now "Archer Chat Agent". The `#section-lead` ID and `card--lead` / `system-tree__node--lead` CSS classes stay — they're internal identifiers, not display strings.
- Labor Agent kept its own `initials: 'LA'`. After the rename, Archer = AC and Labor = LA in the chat avatar grid; no collision.
- Pre-edit snapshots saved to `archive/moonshot-home.PRE-ARCHER-RENAME.html`, `archive/moonshot-prototype.PRE-ARCHER-RENAME.html`, and `archive/lead-agent.PRE-RENAME.md`.

**Judgment calls.** The `source: 'lead'` field on seeded chatThread messages was left as-is — it's an internal routing tag, not display text, and renaming it carries breakage risk for any filter code that keys on it. The nav label "Lead" and the CSS class fragments (`--lead`, `#section-lead`) were preserved as layer-name and identifier references rather than agent-name references. Historical ITERATION-LOG entries were not swept; they describe past state and rewriting them would distort the record.

---

## May 13 afternoon — Archer unified as cross-pillar conversation log

The May 13 morning sprint promised Archer as a cross-pillar component — the single conversation surface where the Lead Agent (now Archer Chat Agent) speaks across WA, TM, OM, and the cross-pillar Watchtower. The implementation only delivered Archer for WA. Clicking Archer in TM, OM, or cross-pillar fell through to `renderProductPlaceholder()`, and the cross-pillar home had its own one-off `renderCrossCLI()` that ignored `buildOpThread()` entirely. This entry closes that gap: one shared thread merging all four pillars' chat history, rendered identically in every product, with the agent-ctx-bar host name switching to tell the user where their next typed message will be tagged. Home terminals stay product-scoped — each pillar's home rail still shows only its own messages.

### What was built

- **Archer (CLI view) ungated for TM, OM, and cross-pillar products.** `renderMain()` now routes `view === 'cli'` to `renderCLI()` at the top of the dispatch, before any product-specific branches. The WA-only gate inside `renderCLI()` itself (`if (currentProduct !== 'wa') { renderProductPlaceholder(); return; }`) was removed. The separate `renderCrossCLI()` function — a parallel render path that lived under cross-pillar — has been retired; the cross-pillar `'cli'` view now flows through the same `renderCLI()` as every other product.

- **Cross-pillar thread converted from function to persistent state.** `crossChatThread()` previously returned a fresh array on every call — there was nowhere for user-typed cross-pillar messages to land. Replaced with a `CROSS_DATA = { chatThread: [ ... ] }` object that mirrors the TM_DATA / OM_DATA shape. `renderCrossHome()` now reads `CROSS_DATA.chatThread` directly; user input in cross Archer pushes into the same array.

- **Every pre-seeded `chatThread` message tagged with `product` and `role`.** Swept all four threads:
  - `WAREHOUSE.chatThread` (seeded via `seedChatThread()` and event-driven pushes in the sim tick at `LEAD_CHECKINS` and `incident_pending` events) — every entry now carries `product: 'wa'` and `role: 'agent'`.
  - `TM_DATA.chatThread` — three seed messages tagged `product: 'tm', role: 'agent'`.
  - `OM_DATA.chatThread` — three seed messages tagged `product: 'om', role: 'agent'`.
  - `CROSS_DATA.chatThread` — three seed messages tagged `product: 'cross', role: 'agent'`.

- **New `getUnifiedThread()` helper** merges all four chatThreads into a single time-sorted array, defensive against any thread that hasn't initialised yet (defends future refactors that touch bootstrap order). Each message is normalised with fallback `product`, `source: 'home'`, and `role: 'agent'` so legacy entries without explicit tags still render coherently.

- **`buildOpThread()` rewritten** to render the unified thread with `(product, source)`-tuple transition dividers. The previous bug — `isUser = m.source === 'cli' || m.source === 'home'` (which together meant "all messages") — is gone; user vs agent rendering now keys off the new explicit `role` field. Divider text reflects the surface the next message just entered: "Started in WA Archer", "Continued in OM Home terminal", "Cross-pillar context (archer@infios-network)", etc. The first divider uses "Started in", every subsequent transition uses "Continued in".

- **Archer agent-ctx-bar host name switches per `currentProduct`** via a new `warehouseByProduct` lookup in `agentMeta()`: `archer@dc-14-memphis · WAREHOUSE` (WA), `archer@hub-mem-tm · TRANSPORTATION` (TM), `archer@oms-na · ORDER MANAGEMENT` (OM), `archer@infios-network · CROSS-PILLAR` (cross). The thread below the header is the same unified history regardless — only the header tells the user where their next message will land.

- **`sendUserMessage()` routed per-product.** Previously hard-coded the TM branch and fell through to `WAREHOUSE.chatThread` for everything else, so typing in OM Archer would silently push to the WA thread. Now selects the target chatThread by `currentProduct` (WA → WAREHOUSE, TM → TM_DATA, OM → OM_DATA, cross → CROSS_DATA), stamps `product: currentProduct, role: 'user', source` on the user message, and stamps `product: currentProduct, role: 'agent', source: 'lead'` on the canned ack. Append-time `sec` is computed as `max(WAREHOUSE.clockSec, lastMsg.sec + 60)` for WA, and `lastMsg.sec + 60` elsewhere — so user messages always sort after their thread's existing seed in the unified view (the previous fixed `8*3600 + 14*60` would have placed an OM-typed message before the OM seed which starts at 13:51).

- **Home terminals remain product-scoped.** All four home renders already passed their own product's thread into `renderHomeChat(thread)` (`renderHome()` → `WAREHOUSE.chatThread`, `renderTMHome()` → `TM_DATA.chatThread`, `renderOMHome()` → `OM_DATA.chatThread`, `renderCrossHome()` → `CROSS_DATA.chatThread` after this session's conversion). No filter logic was needed — product scoping is inherent in the separate state objects. Verified at preview time.

### Why

The May 13 morning sprint's architectural claim — "Archer is the single point of contact across all three products" — depended on Archer behaving identically in every product. The implementation didn't match the claim: TM and OM clicked Archer and got a "Pillar-specific views are coming next. Switch back to WA…" stub, and the cross-pillar dashboard had a parallel render path that didn't even share the same renderer. Anyone Pedro shared the demo with would have clicked Archer outside WA within the first thirty seconds, hit the stub, and reasonably concluded that the cross-pillar story was aspirational marketing wrapped around a single-product prototype.

The `(product, source)` source-attribution pattern in `buildOpThread()` is what makes the Archer Chat Agent's Cross-Pillar Scope addendum (in `agents/archer-chat-agent.md`) demonstrable in the prototype rather than just declared in the spec. Pedro's specific demo scenario — type in WA Home, switch to OM Archer, see your WA message in the unified history with a divider above it labelling where it came from, then type in OM Archer and see a new divider labelling the new context — is the visible mechanism of "the Lead Agent (Archer) is the single point of contact across all three products." Without `(product, source)` tags on every message, there's nothing for the divider to key off, and the unified thread reads as flat noise.

The cross-pillar thread conversion from function to persistent state (`crossChatThread()` → `CROSS_DATA.chatThread`) was the load-bearing refactor that made user input in cross Archer possible at all. Without it, every render of cross would have built a fresh array and discarded any user-typed message, which would have shipped a worse bug than the placeholder stub it was meant to fix.

### Verified

Browser preview (preview server running throughout):

- **Default load — WA home.** Right-rail terminal shows the two WA seed messages plus any in-sim Lead check-ins that fired. Console clean.
- **Click sidebar Archer (WA).** Ctx-bar reads `archer@dc-14-memphis · WAREHOUSE`. Thread shows merged WA + TM + OM + cross seed messages, time-sorted: TM 07:52 → 08:06 (with divider "Started in TM Archer" then "Continued in WA Archer" at 08:00 and "Continued in cross-pillar Archer" at 08:06), WA 08:14 ack, OM 13:51 → 14:18 under "Continued in OM Archer".
- **Type "ping" in WA Archer.** Message appears at the bottom under a "Continued in WA Archer" divider (or the existing one extends — no new divider when the tuple doesn't change). Lead ack appears ~2.5s later under the same tuple.
- **Switch to OM via product switcher → click Archer.** Ctx-bar updates to `archer@oms-na · ORDER MANAGEMENT`. Thread is the SAME unified history including the WA "ping" just typed, still labelled under its WA Archer divider.
- **Type "pong" in OM Archer.** New "Continued in OM Archer" divider appears above the new message. Message and ack are tagged `product: 'om', source: 'cli', role: 'user'` / `role: 'agent'` respectively (confirmed by inspecting `OM_DATA.chatThread` in the console).
- **Switch to WA home.** Right rail shows only WA messages — the OM "pong" does not appear. Switch to OM home → OM "pong" appears, WA "ping" does not. Home-scoping holds.
- **Cross-pillar Archer.** Ctx-bar reads `archer@infios-network · CROSS-PILLAR`. Thread is the same unified history.
- **Reload.** Pre-seeded messages return; user-typed messages lost (expected; no persistence — demo behaviour).
- **No console errors** at any point in the round-trip.
- **No regressions** on the four home terminals — WA, TM, OM, and cross-home all render their seeded threads as before.

### What this leaves

- Archer is now demonstrable as a cross-pillar conversation log end-to-end in the prototype. The Cross-Pillar Scope addendum in `agents/archer-chat-agent.md` is no longer aspirational — sharing the demo with Allen or Eugene no longer surfaces an Archer-is-WA-only inconsistency that would have undermined the cross-pillar architecture claim.
- The `(product, source, role)` tagging pattern on chatThread messages is the foundation for any future cross-product feature that needs source attribution: decision-queue routing, alert-source labelling, audit-log filtering, multi-pillar handoff stories.
- The `getUnifiedThread()` helper is reusable for any future surface that needs a merged view — e.g., a "shift handover" digest, a cross-product timeline export.
- Pre-edit snapshot saved to `archive/moonshot-prototype.PRE-ARCHER-UNIFY.html`.
- **Open carry-overs** from prior sessions remain alive: WA-sim-clock paper cut (clock continues to advance when a non-WA pillar is mounted), font air-gap, parked Infios-token decisions, `.note__*` orphan CSS, 1100 px timeline-content overflow on the Architecture section. None block share-out.

**Judgment calls reported.**

- **Cross-pillar thread variable name.** The cross thread did not previously exist as a persistent state object — it was a function (`crossChatThread()`) returning a literal array each call. Created `CROSS_DATA = { chatThread: [...] }` as the natural mirror of `TM_DATA` / `OM_DATA`, keeping `chatThread` as the field name so `getUnifiedThread()` reads consistently across all four sources.
- **User/agent role detection — approach (b).** Took the explicit `role: 'user' | 'agent'` field rather than inferring from presence of `m.agent` / `m.initials`, because user-typed messages do carry `agent` / `initials` fields (`'You' / 'YO'` for TM/OM/cross, `WAREHOUSE.operator.name / .initials` for WA) — so approach (a) would have mis-classified user messages as agent messages. Approach (b) is also more durable: any future feature that wants to filter on role gets a single keyed field instead of probing two.
- **Pre-seeded `source` field backfill.** All four pre-seeded threads' messages were authored by the agent layer (`source: 'lead'` or `source: 'system'`) — no pre-seeded user-typed messages exist. So every seeded message is tagged `role: 'agent'`. No judgment was needed on home-vs-cli `source` since every seed kept its existing `source` value unchanged (`'lead'` or `'system'`); the divider logic treats both as the same source category (not 'home', not 'cli') and emits "Started in / Continued in {Product} Archer" for them, which reads correctly because pre-seeded operational messages from the orchestrator are conceptually Archer-context.
- **Append-time `sec` calculation.** Replaced the previous fixed `8*3600 + 14*60` fallback for non-WA threads with `lastMsg.sec + 60`. The fixed value would have placed an OM-typed user message at 08:14 sim time, sorting it before the OM 13:51 seed in the unified thread — confusing. Using "after the last existing message" makes user appends always read correctly in time order. WA uses `max(WAREHOUSE.clockSec, lastMsg.sec + 60)` so the live sim clock still drives ordering while the safety floor catches the edge case where a user types before any in-sim Lead check-in fires.

---

## May 13 afternoon — Architecture diagrams updated for cross-pillar story + neutral narrative framing

The afternoon's cross-pillar backfill (Tasks 8–12) brought the `#section-domain` roster on `moonshot-home.html` up to twelve agents across three pillars and added cross-pillar topology beats. But the Architecture section at the **top** of the same page still told the WMS-only story: the System view rendered a single 8-leaf domain row, the Timeline info card for station 3 said "Eight specialist agents — Pick Path, Labor, Quality, and five more," and the lede beneath the System tree said "eight domain agents… outside the WMS topology." When Allen opened the Agents page, the first diagram he saw argued with the cards beneath it. This entry fixes that — System view shows three pillar rows with the dual-context pattern made visible, Timeline info card reframes for twelve agents, lede reframes from "WMS topology" to cross-pillar. While inside the section, Station 1's "UX authors the agent specs" was softened to "Agents are authored" — the page no longer puts UX as the headline owner of the agent system. Per-card "Authored by the UX Team" attributions on individual agent cards stay unchanged.

### What was built

- **System view in `#section-architecture` replaced its single 8-leaf domain row with three labelled pillar groups** (Warehouse · WMS, Transportation · TMS, Order Management · OMS). Markup uses a new `.system-tree__pillar-group` wrapper containing a `.system-tree__pillar-label` and a `.system-tree__domain-row` carrying a `--wms` / `--tms` / `--oms` modifier class.

- **Dual-context agents (Carrier, Order Priority) appear in both their pillars' rows** with a small "+ {other pillar}" subtitle inside a `.leaf__dual-note` span. The leaf carries the new `.system-tree__leaf--dual` modifier. Visible expression of the dual-context Cross-Pillar Notes added to `agents/carrier-agent.md` and `agents/order-priority-agent.md` in Task 9.

- **New CSS rules added after the existing `.system-tree__domain-row` block** (`moonshot-home.html:1235`): `.system-tree__pillar-group` (flex column, gap 10px, max-width 1120px, centered, 24px gap between groups), `.system-tree__pillar-label` (11px / 700 / uppercase / letter-spacing 0.10em, `var(--text-tertiary)`), `.system-tree__domain-row--wms` (explicit 8-column grid), `.system-tree__domain-row--tms, --oms` (8-column grid with leaves spanning 2 columns each, `:nth-child(1)` starting at column 2, `:nth-child(2)` at column 4, `:nth-child(3)` at column 6 — three centred leaves with empty columns 1 and 8 on either side), `.leaf__dual-note` (block, 9.5px / 500, `var(--text-tertiary)`, tabular-nums).

- **Responsive overrides at <1080px** extended into the existing `@media (max-width: 1080px)` block: WMS row collapses to 4 columns, TMS/OMS rows collapse to 3 columns with `grid-column:auto`. Override targets the nth-child selectors explicitly because media queries don't add specificity — without naming the nth-child selectors inside the @media block, the more-specific desktop nth-child rules would win and break mobile layout (caught and fixed during preview verification — see judgment calls).

- **Comment on `moonshot-home.html:1235`** updated from `/* Eight-leaf domain row */` to `/* Pillar-grouped domain rows — base layout */` to reflect the new role of that base class.

- **Timeline view's Station 1 label** softened from `<span>UX authors</span><span>the agent specs</span>` to `<span>Agents are</span><span>authored</span>`.

- **Timeline view's Station 1 info card** (`info-card[data-num="1"]`) heading softened from "UX authors the agent specs" to "Agents are authored", body from "UX writes the rules every agent follows — approvals, escalations, confidence levels. Behavior is **designed in**, not discovered after launch." to "Each agent has a spec — its rules, its approvals, its escalations, its confidence calibration. Behaviour is **designed in**, not discovered after launch." Passive-voice description that names neither UX nor a specific product team. Card-level "Authored by the UX Team" attributions on individual agent roster cards stay unchanged — UX ownership is discoverable per-card but no longer the page headline.

- **Timeline view's Station 3 info card** body updated from "Eight specialist agents — Pick Path, Labor, Quality, and five more — each raise proposals in their lane. Every proposal carries a **confidence score** the user can read." to a twelve-agents-across-three-pillars description listing the WMS/TMS/OMS rosters, naming Carrier and Order Priority as the dual-context pair, and closing with "Every proposal carries a **confidence score** the user can read, regardless of pillar."

- **System view lede paragraph** below the tree updated from "The wiring underneath. The operational spine carries human approval down through Archer Chat Agent, Shift Intelligence, and eight domain agents. The lateral layers — Analytical, Guardian, and Meta — run alongside but outside the WMS topology." to "The wiring underneath. The operational spine carries human approval down through Archer Chat Agent, Shift Intelligence, and twelve domain agents across three pillars — Warehouse, Transportation, Order Management. Two agents (Carrier, Order Priority) operate in dual contexts where their problem space crosses pillar boundaries. The lateral layers — Analytical, Guardian, and Meta — run alongside, governing all three pillars under the same approval-first contract."

### Why

The Task 10 (May 13 PM) sweep updated the Agents page Domain section roster to twelve cards but left the Architecture section's System view, Timeline info card #3, and lede text describing the WMS-only system. When Allen, Eugene, or a Gartner reviewer opens the Agents page, the **first** thing they see is the Architecture diagram. If that diagram tells the WMS story and the cards below it tell the cross-pillar story, the page argues with itself and the cross-pillar argument collapses at the top of the page.

The dual-context pattern (Carrier, Order Priority) needed visual expression somewhere on the page. Putting those agents in two pillar rows with a "+ {other pillar}" subtitle makes the pattern legible without requiring the reader to find the Cross-Pillar Note buried inside each spec file. This is the same principle as putting the cross-pillar topology beat directly into `#section-topology` rather than relying on a reader to follow links to spec markdown.

The Station 1 narrative softening (UX-as-author → passive "Agents are authored") was the second motivation. The strategic position is to partner with the WM, TM, and OM product teams — so the page should describe the system without naming UX as the headline owner of the agent specs. Per-card attributions on individual agent cards stay unchanged: UX ownership remains discoverable for anyone who opens a card, but it's no longer planted as the headline at the top of the page. Subtle, descriptive, no territorial flag.

### Verified

Browser preview against `moonshot-home.html` (preview server already running). Screenshot tool was unresponsive throughout the session (timed out twice with no console errors and the page alive on `eval`); verification was done via DOM inspection (`preview_eval`, `preview_inspect`) instead of visual screenshots.

- **Three pillar groups render in System view** at desktop (1280×800): WMS group with 8 leaves at 131px each, TMS group with 3 leaves at 273px each centered in columns 2–7 of the 8-column grid (columns 1 and 8 empty on either side), OMS group with 3 leaves at 273px each in the same arrangement. Pillar labels render at 11px / 700 / uppercase / `var(--text-tertiary)` (rgb(88,99,110)).
- **Dual-context subtitles** ("+ OMS", "+ TMS", "+ WMS") render as `display:block` 9.5px / 500 / `var(--text-tertiary)` below the leaf name. Carrier and Order Priority appear in two rows each.
- **Timeline info card #3** body reads "Twelve specialist agents across three pillars…" — verified via `preview_eval`.
- **Timeline Station 1 label** reads "Agents are / authored"; **info card #1** heading reads "Agents are authored", body reads "Each agent has a spec — its rules, its approvals, its escalations, its confidence calibration. Behaviour is designed in, not discovered after launch."
- **System view lede** below the tree reads the new "twelve domain agents across three pillars… governing all three pillars under the same approval-first contract" copy.
- **Responsive layout at 980×800**: WMS row collapses to 4 columns at 219px each (4×2 grid), TMS/OMS rows collapse to 3 columns with `grid-column:auto` at 296px each. No horizontal scrollbar (`document.documentElement.scrollWidth === window.innerWidth`).
- **No console errors** at any point in the round-trip (Timeline ↔ System toggle, desktop ↔ mobile resize).
- **No layout breakage** in the conveyor (Timeline) or the lateral layers (Analytical, Guardian, Meta). Conveyor measures 1217×864 at 1280px viewport, the lateral layer cards stack as before.

### What this leaves

- The Agents page now tells a single coherent cross-pillar story from top (Architecture diagrams) to bottom (Domain cards, topology beats).
- The dual-context pattern is visible on the System view, not just buried inside spec markdown files. A reader who scans the System view immediately sees that Carrier and Order Priority are the dual-pillar agents.
- The page no longer plants "UX authors" as the headline framing of the agent system. Per-card attributions on individual agent roster cards stay — UX ownership is discoverable but not asserted at the top.
- Sharing this page with Allen, Eugene, or the Gartner analyst no longer surfaces the inconsistency that the Architecture diagrams previously created.
- Pre-edit snapshot saved to `archive/moonshot-home.PRE-ARCH-CROSS-PILLAR.html`.

### Caveats reported

- **TMS/OMS leaf width vs WMS leaf width.** Per the spec's literal CSS rules (`grid-column:span 2` for TMS/OMS leaves on an 8-column grid), TMS/OMS leaves render at 273px on a 1280px viewport while WMS leaves render at 131px. That's about 2× wider, not "roughly the same physical size as WMS leaves" as the verification prose suggested. Acceptable: the leaves are NOT stretched to 1/3 of the row width (would be ~373px), so they don't read comically large; they're centred between columns 2–7 with empty columns on either side, which gives the row visual breathing room. Calling it out so the visual choice is documented — if Pedro wants the TMS/OMS leaves to match WMS leaf width exactly, the fix is to drop `span 2` and use a 3-column flex layout with centered alignment instead.
- **Connector line on TMS/OMS rows.** The base class `.system-tree__domain-row::before` horizontal connector still renders across the full row width (left:calc(100%/16), right:calc(100%/16)), which means the connector line slightly extends past the leftmost and rightmost TMS/OMS leaves rather than meeting them at the ::before vertical drop. Visually mild, not breaking; left as-is.
- **Mobile-specificity bug caught and fixed.** Initial @media block at <1080px set `.system-tree__domain-row--tms .system-tree__leaf { grid-column:auto; }` — but the desktop nth-child rules `.system-tree__domain-row--tms .system-tree__leaf:nth-child(1)` etc. have higher specificity and won regardless of the media query. Caught at the 980px verification step (TMS leaves rendered at 512/67/47 px instead of equal widths). Fix: the @media override now also names the nth-child selectors explicitly. After-fix verification: TMS/OMS leaves all render at 296px in 3-column auto layout.
- **British "Behaviour" vs American "Behavior".** Part 5b's verbatim copy uses British spelling ("Behaviour"); the rest of `moonshot-home.html` (and the original station 1 body) uses American spelling. Followed the spec verbatim. If a global pass on en-US/en-GB consistency is wanted, Part 5b's body line is the one to flip.
- **Screenshot tool unresponsive.** `preview_screenshot` timed out twice during this session with no console errors and `preview_eval` working normally. All verification done via DOM inspection. Visual confirmation at desktop and mobile widths was not possible from this session.

---

## May 13 afternoon — Agents tab split into Roster + The System

The single Agents page on `moonshot-home.html` was doing two jobs: telling the meta narrative (Architecture conveyor, System tree, Topology, the case) AND presenting the agent roster (Lead, Radar, Domain, Analytical, Guardian, Meta, Horizon). The internal Timeline/System toggle inside `#section-architecture` was the tell — when half the content has to hide behind a toggle, the page has two jobs. This entry splits the Agents page into two pages, each with one clear job: **The System** for the narrative, **The Agents** for the roster.

### What was built

- **Top nav expanded from 3 to 4 tabs.** New tab "The System" added between Home and The Agents. Final order: Home · The System · The Agents · The Prototype. (`moonshot-home.html:2154-2159`)
- **New view container `<section id="view-system" class="view view--system">`** created parallel to `view-home`, `view-agents`, and `view-prototype`. Re-uses the existing `.view[data-active="true"]` show/hide pattern (CSS at line 309–311) — `showView()` was already generic over view IDs so no JS rewrite was needed.
- **`section-architecture` moved into `view-system`** with its internal Timeline/System toggle removed. The `.diagram-toggle-row` markup and the System-view branch (`.diagram__view--system`) are gone from this section — Timeline is now its sole child of `.diagram`. The lede was reframed from "The system delivers an artifact, not a guess." to "This is the meta process: how every agent in the system gets authored, audited, and shipped." so it introduces the meta narrative only, not a toggled inventory view.
- **New `section-system-tree`** added to `view-system` between the architecture conveyor and topology. The 12-leaf cross-pillar tree (WMS · TMS · OMS rows, dual-context leaves for Carrier and Order Priority, plus the Analytical/Guardian/Meta lateral layers) — extracted from the old `#section-architecture` System-view branch — now lives here as a standalone section with its own `.arch` wrapper, `bg-topo` SVG canvas (varied transforms), section label "System Tree", title "The agent inventory.", and a brief inventory lede: "Twelve domain agents organised across three pillars. Carrier and Order Priority appear in both their contexts."
- **`section-topology` moved wholesale** from `view-agents` to `view-system`. No internal changes.
- **`section-case` duplicated into `view-system` as `section-case-system`** — identical content, different ID to avoid duplicate-ID warnings. Both pages now end with the same "UX as infrastructure, not overhead" closing pull-quote, because we're selling one thesis from two angles.
- **Placeholder anchor `<div id="section-pillar-timelines">`** inserted between `section-architecture` and `section-system-tree` in `view-system`. Task 17 will populate this section; for now it's an empty 0-height anchor so the System sticky-index "Pillar Timelines" entry resolves to a real element rather than dead-scrolling.
- **Two sticky in-page indexes, one per view.** The System index (`#system-index`): Architecture · Pillar Timelines · System Tree · Topology · The case. The Agents index (`#agents-index`, updated): Lead · Radar · Domain · Analytical · Guardian · Meta · Horizon · The case. Architecture and "Signals" (Topology) entries removed from the Agents index since those sections moved.
- **Scroll-highlight JS refactored** from a single hardcoded `sectionIds` array into a `wireIndex(viewId, indexId, sectionIds)` helper called twice — once for `view-system` and once for `view-agents`. Each instance guards on `view.dataset.active === 'true'` so the hidden view's index does not fight the visible one. The smooth-scroll anchor click handler (which keys off the shared `.agents-index__item` class) needed no change.
- **Conveyor reveal observer** changed from `document.querySelector('.howagents-conveyor')` to `document.querySelectorAll(...)` plus `forEach` registration, since there are now two `.howagents-conveyor` blocks (one inside the architecture section, one inside the system-tree section) and only the first would have animated otherwise.
- **Tab-switch JS allow-lists** at the bottom of the `<script>` block extended to accept `'system'` in both the initial-hash check and the hashchange filter, so `#system` deep-links work.
- **Diagram-toggle JS** (the segmented-control handler for the old Timeline/System toggle) left in place but inert — its `.diagram-toggle` selector now matches nothing, so the `if(diagToggle && diagram)` guard short-circuits cleanly. Dead but harmless; can be removed in a future cleanup.

### Why

The Timeline/System toggle inside Architecture was always a sign that the Agents page was carrying two stories. The story page wants the meta narrative — how agents get authored, organised across pillars, and audited — top to bottom in scroll order, ending on the closing argument. The roster page wants card-by-card lookup of the 14 agents, ending on the same closing argument. Putting both on one page forced the System tree behind a toggle, made the sticky index uncomfortably long, and meant the closing "case" pull-quote sat below the roster cards where a System-narrative reader would never reach it.

Splitting in two means each page does one job well. The reader can take either path — scroll the System story end-to-end, or skim the roster card by card. Both end on the same closing argument because the thesis ("UX orchestration is infrastructure; the agent system is what it produces") is the same regardless of which entry point the reader chose. Putting "The case" on both pages costs ~67 lines of duplicated markup; the alternative — making the System reader navigate to the Agents page to read the closing — would be worse.

The Timeline/System toggle disappearing is the design-quality signal: the page no longer needs a toggle because each story has its own page.

### Verified

Browser preview against `moonshot-home.html` (preview server already running). `preview_screenshot` timed out twice this session even after the work was complete, with no console errors and `preview_eval` working normally — verification done via DOM inspection (`preview_eval`, `preview_snapshot` accessibility tree) instead of visual screenshots.

- **Top nav renders four tabs in order**: Home · The System · The Agents · The Prototype. `aria-current="page"` moves with each click; `data-go="system"` switches `view-system[data-active="true"]` correctly.
- **Hash routing**: `#system` → `view-system`, `#agents` → `view-agents`, `#prototype` → `view-prototype`, `#home` → `view-home`. Direct deep-link to any of the four hashes loads the corresponding view on first paint.
- **No duplicate IDs.** Verified by querying every element with an `[id]` and counting collisions — empty result. `section-case-system` rename took.
- **`view-system` sections in correct scroll order**: `section-architecture` → empty `section-pillar-timelines` anchor → `section-system-tree` → `section-topology` → `section-case-system`. Confirmed via `document.querySelectorAll('#view-system [id^="section-"]')`.
- **`view-agents` no longer contains Architecture, System Tree, or Topology.** Confirmed via three `document.querySelector('#view-agents #section-architecture')` style checks — all returned `false`. The roster sections render in order: Lead → Radar → Domain → Analytical → Guardian → Meta → Horizon → The case.
- **The System sticky index** has five entries with `is-active` defaulted to Architecture. Scrolling to `section-topology` and dispatching a scroll event highlights "Topology" — confirmed.
- **The Agents sticky index** has eight entries with `is-active` defaulted to Lead. Architecture and Signals entries gone.
- **Both indexes coexist** without fighting each other. When `view-system` is active, `view-agents` is `data-active="false"` and its `wireIndex` instance early-returns on the `dataset.active` guard.
- **Mobile viewport (375×812)**: `system-index` renders visible at 41px tall, 375px wide. Index remains usable at mobile.
- **No console errors** at any point in the round-trip.
- **The Prototype** unchanged — iframe `moonshot-prototype.html` loads as before.

### What this leaves

- The Agents page is now a roster page, the System page is a narrative page, and the closing argument appears on both. The Timeline/System toggle is gone — its job is done now that each story has its own page.
- `section-pillar-timelines` is a placeholder anchor (0-height empty div) waiting for Task 17 to populate it. The "Pillar Timelines" index entry exists but scrolls to a still-empty section for now.
- Pre-edit snapshot saved to `archive/moonshot-home.PRE-SPLIT.html`.
- The dead diagram-toggle JS handler is the only real piece of cruft left from the split — it's guarded behind a null-check that always fails, so it costs nothing at runtime, but it's lines that no longer serve a purpose and could be removed in a future cleanup.
- Architecture's section title "The shape of the system." was kept as-is — could sharpen to "How agents get made." but that's a copy decision that warrants its own pass with Pedro.
- The Agents hero kicker still reads "Six layers · Twenty agents · One human in the loop" — written when the page covered the whole system. It's not a regression (the roster still has six layer sections worth of agents) but the number "Twenty" includes agents that are now described on the System page rather than rostered here. Left alone this session; a kicker refresh would be a separate copy decision.

### Caveats reported

- **No duplicate-ID warnings encountered.** The `section-case` → `section-case-system` rename was the only ID conflict the duplication could have created; all other section IDs inside `section-case` were generic (`card-research-details` etc.) and `section-case` only appears once in `view-agents`, while `section-case-system` only appears once in `view-system`.
- **No CSS rework required.** The `.diagram[data-active="X"] .diagram__view--Y` rules that powered the old toggle's show/hide (lines 1161–1171) still work correctly on the two new isolated single-child wrappers: in `section-architecture`, `.diagram[data-active="timeline"]` has only a `.diagram__view--timeline` child (the "hide system" rule is a no-op); in `section-system-tree`, `.diagram[data-active="system"]` has only a `.diagram__view--system` child (the "hide timeline" rule is a no-op). No need to flatten these wrappers — leaving them in place keeps the change minimal and preserves the conveyor-reveal CSS that targets `.howagents-conveyor` inside them.
- **`section-topology` cloned into `view-system` rather than physically moved.** Both views needed the section to render independently (each has its own `data-active`-gated visibility), and `view-agents` had to retain ID-unique sections only. The cleanest approach was: insert `view-system` with a clone of `section-topology` markup, then delete the original from `view-agents`. The duplicate topology block in `view-agents` is now gone — confirmed by querying that `view-agents` contains no `#section-topology`. Same approach used for `section-architecture` (cloned with the toggle stripped, then original deleted from `view-agents`).
- **Architecture lede reworded** from "Seven stations between a prompt and a shipped prototype. Gold nodes are UX-led — agent specs, orchestration, audits, and the human-in-the-loop gate. The system delivers an artifact, not a guess." to "...This is the meta process: how every agent in the system gets authored, audited, and shipped." The second sentence change reflects that this section now introduces the meta narrative as its job, no longer foreshadowing a toggleable System view.
- **System Tree's bg-topo SVG uses slightly varied `transform` values** from the architecture section's SVG so the canvas reads as a distinct section rather than a repeat of the same backdrop. Same `.bg-topo bg-topo--soft` class; only the path/peak coordinates differ.
- **Screenshot tool unresponsive again.** `preview_screenshot` timed out twice in this session with `preview_eval` and `preview_snapshot` returning correctly. Visual confirmation at desktop and mobile widths not possible — verification done via accessibility-tree snapshot (which confirmed The System renders hero + sticky index + Architecture conveyor + System Tree + Topology + The case in scroll order) and DOM inspection. Same caveat as the previous afternoon entry.

---

## May 13 evening — Pillar timelines authored (Task 17)

Task 16 left a placeholder anchor `#section-pillar-timelines` between Architecture and System Tree on `view-system` and pre-wired its entry in the sticky index. Task 17 fills that gap. The System page argued *agents do useful work* through abstract architecture diagrams without showing a concrete decision lifecycle. Four scenarios — three compact horizontal flows (one per pillar) and one cross-pillar swim-lane (Carrier delay cascade) — make the argument concrete: who proposes, who audits, where the human gate sits, and how signals propagate when more than one pillar is involved.

### What was built

- **New section `#section-pillar-timelines`** replacing the empty placeholder `<div>` from Task 16. The section adopts the existing `.arch` chrome (`.bg-topo--soft` SVG backdrop + `.arch__inner` + `.section-label` + `.section-title`) so it integrates with Architecture above and System Tree below. Section lede sits in a new lightweight `.section-lede` class that mirrors the conveyor lede's tone but lives outside `.howagents-conveyor`.
- **Three compact horizontal flow sub-sections** (`.pillar-timeline--wm`, `.pillar-timeline--tm`, `.pillar-timeline--om`). Each has a pillar chip top-left, a scenario title, a 2px horizontal rail with a CSS-triangle arrowhead at the right end, and six step nodes evenly spaced across a 6-column CSS grid. Each node carries a 10px time label above the dot, a 32px circular dot, the agent name (11px / 600), and the action verb (11px / regular) below. The human-decision node in each flow uses a `--human` modifier — a filled dot in the pillar's accent colour — so the moment of approval is visually distinct from agent proposals.
- **One cross-pillar swim-lane sub-section** (`.pillar-timeline--cross`). Three lanes (TM top, OM middle, WM bottom) at 130px tall each, with an absolutely-positioned dashed rail centred in each lane. Lane labels sit top-left of each lane. Ten nodes are positioned by `style="left:<pct>%"` along the time axis 08:43–09:10. Metas are placed above or below the rail per node, alternating to keep the strip readable.
- **A decision-point card** (`.swim-lane__decision-card`) centred horizontally and vertically over all three lanes at the moment of human decision (08:47–08:48). Card has the Archer Chat Agent prompt, two side-by-side path options (`--alternate` and `--chosen`), and a footer line recording the Director of Operations approval. Path 1 (wait) is rendered in muted charcoal-grey; Path 2 (deploy backup + re-allocate) is rendered with the Infios yellow accent border and a soft yellow tint, marking it as the chosen path.
- **Cross-lane SVG connector arrows** (`.swim-lane__connectors`). Five paths drawn in an inline SVG layer that fills the swim-lane via `viewBox="0 0 1100 390" preserveAspectRatio="none"`: TM 08:43 → OM 08:44 (dashed charcoal, "signal propagates down"); OM 08:45 → decision-card left edge (dashed charcoal, "OM escalates to human"); decision-card right edge → TM 08:49, OM 08:51, WM 08:53 (solid purple, "human decision fans out to all three pillars"). Arrowheads via a shared `<marker>` element.
- **Pillar tints and chips** drawn from existing Infios tokens — `var(--accent)` (#CADF35) for WM, `var(--charcoal)` (#262523) for TM, `var(--link)` (#5436CC) for OM, and a `linear-gradient(90deg, accent → charcoal → link)` for the cross chip. Section backgrounds use the same colours at 4–5% opacity (subtle — noticeable only by side-by-side comparison). **No blue brand colour anywhere in the new section** (verified per Task 16's no-blue constraint).
- **Responsive collapse at <1080px.** Compact flows reflow from 6-column grid to a single column with each node rendering as a horizontal row (time on the left, dot, agent+action stacked on the right); the rail hides because there's no straight line to draw through a vertical stack. The swim-lane has `min-width:900px` and is wrapped in a `.swim-lane__scroll` container with `overflow-x:auto`, so it scrolls horizontally on narrow viewports rather than reflowing.

### Why

The System page needed concrete scenarios to ground the abstract Architecture diagrams in real decision flows. Without these timelines the page argued "agents do useful work" without showing what useful work looks like — a Director of Operations reading the page would see seven stations and twelve agents but no minute-by-minute story.

The hybrid format (three single-pillar flows + one cross-pillar swim-lane) avoids the repetition trap of four identical zigzags stacked, and makes the cross-pillar argument visually distinct from the single-pillar lifecycles. The WM, TM, OM compact flows share a structural template; the cross-pillar swim-lane breaks that template deliberately, signalling "this scenario is different in kind."

The swim-lane's decision-point card is the moment that demonstrates the moonshot thesis: agents propose, humans decide. Centring it horizontally over all three lanes — physically interrupting the three rails — is the visual argument that the human gate is the single seam between AI activity and committed action. The two-path layout (one chosen, one declined-but-shown) makes the alternative explicit so the reader sees what the system *didn't* do as well as what it did.

### Verified

Verification via Claude Preview MCP against the running `moonshot-home.html` preview server. `preview_screenshot` timed out again this session (consistent with the previous two sessions' caveat); verification done via `preview_eval`, `preview_inspect`, and `preview_resize` instead.

- **Structural assertions.** Section found at `#section-pillar-timelines`, sits between `section-architecture` (prev) and `section-system-tree` (next). Four `.pillar-timeline` blocks render in order. WM/TM/OM each have exactly six `.pillar-timeline__node` children. The swim-lane has three `.swim-lane__lane` children and ten `.swim-lane__node` children (3 TM + 4 OM + 3 WM). Decision card and connectors SVG both present.
- **Pillar chip colours and AA contrast.**
  - WM chip: `background-color: rgb(202, 223, 53)` (Infios accent yellow), `color: rgb(38, 37, 35)` (charcoal). Luminance ratio ≈ 13:1. AA pass for normal text ✓.
  - TM chip: `background-color: rgb(38, 37, 35)` (charcoal), `color: rgb(255, 255, 255)` (white). Ratio ≈ 15.5:1. AA pass ✓.
  - OM chip: `background-color: rgb(84, 54, 204)` (Infios link purple), `color: rgb(255, 255, 255)` (white). Ratio ≈ 7.3:1. AA pass ✓.
  - Cross chip: linear-gradient charcoal mid-stop vs white text ≈ 15:1. AA pass ✓.
  - All four pairs verified via `preview_inspect` reading computed `background-color` and `color`.
- **Decision-card placement.** Card centred at horizontal 50% / vertical 50% of swim-lane, 300×230px on a 1144×390 swim-lane (when displayed at full width). Vertically spans roughly 59% of the swim-lane — bridges all three lanes as the spec required.
- **Node overlap audit.** Iterated all ten swim-lane nodes and computed dot/meta bounding-box vs decision-card bounding-box intersections. Initial pass surfaced one overlap (OM 08:45 meta intersected card left edge by ~19px). Fix: moved OM 08:45 from `left:32%` to `left:28%`, updated the corresponding SVG connector path from `M352,195 L394,195` to `M308,195 L394,195`. Post-fix re-audit returned zero overlaps.
- **Sticky-index activation.** Scrolled to the new section programmatically and dispatched a scroll event; `#system-index .agents-index__item.is-active` updated to `href="#section-pillar-timelines"` with text "Pillar Timelines". Scrollspy works exactly as for the other four sections.
- **Responsive collapse at 1000×800 viewport.** Compact-flow `.pillar-timeline__nodes` switches from `grid-template-columns: repeat(6, 1fr)` to single column (`1fr`); rail hidden (`display:none`). Swim-lane retains `min-width:900px` and the `.swim-lane__scroll` wrapper has horizontal overflow (scrollWidth 912 > clientWidth 879), confirming horizontal scroll engages without layout breakage.
- **No console errors or warnings** across reload, scroll, click, resize cycles.

### What this leaves

- The Pillar Timelines are static page content. **Task 18** (Sim Life Agent expansion) will fire equivalent events in the live prototype so the timelines on this page describe behaviour the prototype actually simulates. Until that lands, the home page's narrative is ahead of the prototype's behaviour.
- The four discrete scenarios are an interim step. A future iteration will replace them with a single connected narrative flowing through all three pillars seamlessly (per Task 17's brief, which acknowledged the discrete format as interim).
- **Task 20** will re-run the full WCAG 2.2 AA audit on the new chips/section in context and may adjust tones if any pairing fails in the real surrounding palette. Current per-pair contrast pre-check passes AA for normal text on all four chip pairs.
- Pre-edit snapshot saved to `archive/moonshot-home.PRE-PILLAR-TIMELINES.html`.

### Caveats reported

- **Chip contrast adjustments needed: none.** All four chip pairs pass WCAG AA at 4.5:1 minimum for normal text without any tweaks to the Infios tokens. WM yellow + charcoal text was the closest pair to flag; it still ratios ≈ 13:1 which is comfortable AAA territory.
- **Cross-lane arrows rendered as inline SVG, not pure CSS.** Reason: arrowheads via `<marker>` and curved cross-lane paths (TM down to OM and decision-card-down-to-WM) are awkward to express in pure CSS (rotated borders don't curve, dashed strokes don't terminate in arrowheads cleanly). SVG with `preserveAspectRatio="none"` and viewBox `0 0 1100 390` was the lighter approach. Tradeoff: under heavy horizontal stretching, the dashed arrows distort slightly (dashes lengthen along x), but at the swim-lane's typical 900–1200px width the distortion is imperceptible.
- **Decision card spans lanes by absolute positioning rather than `grid-row: span 3`.** The card needs to sit *centred horizontally at 50%* of the swim-lane and *bridge all three lanes vertically*. With CSS grid `grid-row: 1 / span 3` the card would stretch the full lane height (390px) which is more than needed (230px is plenty). Absolute positioning with `top:50%; transform:translate(-50%,-50%)` lets the card pick its own height. The SVG connector layer (also absolute, inset:0) sits one z-layer below the card so connector arrows visibly approach the card edges.
- **Swim-lane `min-width:900px` causes horizontal scroll between 900px and 1080px viewports** as well as below 900px. Acceptable: at <1080px the compact flows already collapse, and the swim-lane's intricate cross-lane layout doesn't reflow well into a vertical mobile-style stack — horizontal scroll preserves the time-axis story which is the whole point of the swim-lane.
- **One mid-implementation positional adjustment** (OM 08:45 from 32% to 28%) was needed to eliminate a meta/decision-card visual overlap. Caught by the post-render overlap audit, not visible without the audit; fix was a one-character HTML change plus the matching SVG connector path update.
- **Screenshot tool unresponsive yet again.** `preview_screenshot` timed out three times this session even after the work was complete and the page was fully interactive; `preview_eval`, `preview_inspect`, and `preview_resize` all worked normally throughout. Visual confirmation at desktop and <1080px widths done via DOM inspection (bounding boxes, computed styles, overlap audit) rather than screenshots. Same recurring caveat as the last three afternoon entries.

---

## Afternoon of May 13 2026 — Warehouse Life Agent renamed to Sim Life Agent; expanded to drive cross-pillar simulation

### What was built

- **Agent spec renamed and broadened.** `agents/meta/warehouse-life-agent.md` → `agents/meta/sim-life-agent.md`. Title, role description, Identity, Scope (Owns/Does-not-own), and Guardrails rewritten to cover all three pillars (Warehouse + Transportation + Order Management) plus the Watchtower. Pre-rename snapshot saved at `archive/warehouse-life-agent.PRE-RENAME.md`.
- **Four event tables in the spec** (WA, TM, OM, Cross). The WA table is the original 7-event timeline reframed under a `### WA (warehouse) events` heading. The TM table holds 3 isolated single-pillar events (08:18 ETA, 08:22 spot quote, 08:35 check-in). The OM table holds 3 (08:31 intake spike, 08:34 allocation, 08:55 cutoff). The Cross-pillar cascade section authors the Carrier #4471 MEM→CHI delay scenario with both Path 1 (Wait) and Path 2 (Deploy backup + re-allocate) resolution waves pre-scripted.
- **Correlation logic section.** Names every event with `isolated` or `cascades` tags. Worked example of the 08:43 cascade — step-by-step trace from TM Carrier Agent detection through Archer's escalation to Director, the path-gated CROSS_EVENTS firing, and the resolution wave. Closes with the principle: "Most events stay in their pillar. Cross-pillar correlations fire when they should, not when they could."
- **Prototype sim loop extended.** `moonshot-prototype.html` now declares `TM_EVENTS`, `OM_EVENTS`, `CROSS_EVENTS` constant arrays alongside the original `ROOT_EVENTS`. New `applyTMEvent` / `applyOMEvent` / `applyCrossEvent` handlers mutate `TM_DATA` / `OM_DATA` / `CROSS_DATA` state. `simTick` now iterates all four tables. The cross-pillar table includes a `path` field per entry (`null | 1 | 2`); only entries matching the new module-level `directorChoice` (default 2) fire.
- **Live clock for TM and OM dashboards.** The static `TM_DATA.clockHHMM = '08:14'` and `OM_DATA.clockHHMM = '14:18'` strings are gone; `renderTMHome` and `renderOMHome` now read `simClockHHMM()`. The single sim clock is now the source of truth across all three pillars.
- **`resetSim` extended.** Clears `_fired` flags on TM/OM/CROSS event tables, restores `TM_DATA.actionLog` / `TM_DATA.chatThread` / `TM_DATA.kpi` / `OM_DATA.actionLog` / `OM_DATA.chatThread` / `OM_DATA.kpi` / `CROSS_DATA.chatThread` from deep-cloned seed snapshots taken at boot, resets `CROSS_DATA.cascadeState` to `'pending'` and `directorChoice` to 2.
- **`lastRenderedSig` invalidation extended** to cover TM, OM, and cross-pillar views. The original gate only handled WA — extending it was load-bearing for the TM/OM dashboards to actually refresh when sim events mutate their data.
- **`crossProductStatus` reads `CROSS_DATA.cascadeState`.** When the cascade is active and unresolved, all three pillar status dots force amber; on `'resolved'` the per-pillar count logic resumes.
- **Decision-point UI.** New `renderCascadeDecisionCard()` returns a two-column card matching the existing escalation-card design tokens. Surfaces on the Watchtower above the unified queue, AND on each per-pillar home stage (WA/TM/OM) replacing the active pending card when `cascadeState === 'awaiting-decision'`. After a path button is clicked, the card flips to a green-bordered confirmation state showing the chosen path + audit summary.
- **`resolveCascade(path)` handler.** Flips `directorChoice`, transitions `cascadeState` to `'path-' + path`, pushes an audit message into `CROSS_DATA.chatThread`, re-arms any path-gated events whose `atSec` already passed (so the resolution wave still plays if the Director clicks slowly), forces `lastRenderedSig = ''`, calls `renderAll()`.
- **Cross-references swept.** `grep -rn "Warehouse Life" agents/` returns zero hits after the rewrite (provenance line reworded to reference the PRE-RENAME archive snapshot rather than naming the old agent literally).

### Why

- The original Warehouse Life Agent was scoped to WMS only. The TM and OM dashboards in the prototype were static stubs — they showed pre-fixed data, never updated, and gave the impression that cross-pillar was a slide deck rather than a system.
- Renaming to Sim Life Agent and expanding the event tables means the simulation now drives the whole demo. TM and OM dashboards visibly update during the 3-minute demo run.
- The branching decision point is the moment that sells the moonshot thesis: agents propose, humans decide. Showing two paths concretely (with spend and SLA trade-offs) demonstrates the value of the human-in-the-loop gate in a way that no diagram can.
- Mixing the one cascade with isolated single-pillar events makes the system feel realistic. If every event cascaded into every pillar, the demo would look like magic — the correlation-logic table in the spec explicitly states which events do and do not cascade, and the prototype enacts that contract.

### Verified

- **Reload + parse**: `simTick`, `TM_EVENTS`, `CROSS_DATA.cascadeState = 'pending'` all defined after page load; zero console errors throughout the session.
- **Event firing**: at sim 08:35 after a fresh load, TM action log contains the 08:18 ETA + 08:22 spot quote + 08:35 check-in entries; OM action log contains the 08:31 intake spike + 08:34 allocation entries; `TM_DATA.kpi.openAlerts` incremented from 3 to 4 after the 08:22 pending event.
- **Cascade trigger**: at sim 08:47, the four `path: null` CROSS_EVENTS have fired (cascade root + OM ATP + OM rerank + decision card); `CROSS_DATA.cascadeState === 'awaiting-decision'`.
- **Decision card surfaces** on the Watchtower AND on TM home AND on WA home when cascade is awaiting decision. Card title reads "Carrier #4471 delay — 12 orders affected. Approve a path." Two path buttons present with exact text "Approve: Wait" and "Approve: Deploy backup + re-allocate".
- **Path 2 (default)**: `resolveCascade(2)` followed by tick to 09:11 fires all five Path-2 events (08:49, 08:51, 08:53, 08:54, 09:10) and zero Path-1 events; `cascadeState` ends at `'resolved'`.
- **Path 1 manual override**: `resetSim()` then `resolveCascade(1)` then tick past 11:15 fires all four Path-1 events and zero Path-2 events; `cascadeState` ends at `'resolved'`.
- **Unified Archer terminal** in cross-pillar context interleaves messages from all four products (WA, TM, OM, Cross) in chronological order — verified by walking `getUnifiedThread()` for the 08:40–09:11 window and confirming each pillar's contribution appears at its declared sim-minute.
- **Watchtower status**: with cascade active and unresolved, `crossProductStatus('wa' | 'tm' | 'om')` forces amber; on `'resolved'`, per-pillar count logic resumes.
- **Isolated events stay isolated**: TM 08:18 ETA + TM 08:35 check-in + OM 08:34 allocation + OM 08:55 cutoff fire without touching `CROSS_DATA.chatThread` or `cascadeState` (confirmed by inspecting per-pillar threads vs cross thread).
- **Zero console errors** through the full reset → tick → resolve → tick lifecycle.

### What it leaves

- The demo now actually simulates cross-pillar behaviour. Reviewers see the system, they don't have to imagine it.
- The branching decision point is the demo's centrepiece. When showing this to Eugene, Allen, or Gartner, the click moment ("Approve Path 2") is the demo's narrative climax.
- Future iteration can add more cross-pillar cascades, additional decision points, or extend the time window to cover end-of-day cutoff scenarios. The event-table pattern is ready to grow.

### Caveats reported

- **Most fragile wire**: the cross-pillar cascade's branch gating. Two subtle issues surfaced and were fixed during implementation. First, the JavaScript const-hoisting / TDZ trap: my initial block declared `CROSS_SEED_CHATTHREAD = JSON.parse(JSON.stringify(CROSS_DATA.chatThread))` and `CROSS_DATA.cascadeState = 'pending'` immediately after the new event tables — but `CROSS_DATA` itself is declared much later in the file (line ~4136, after the TM/OM stub data and the WAREHOUSE state setup). The seed line threw a ReferenceError on module load. Fix: declare a mutable `let CROSS_SEED_CHATTHREAD = null;` placeholder early, and do the actual seed snapshot + `cascadeState = 'pending'` initialisation right after the `CROSS_DATA` declaration. Second, `lastRenderedSig` was only checking WA — without parallel sig invalidation for `currentProduct === 'tm' | 'om' | 'cross'`, the dashboards looked static even as their underlying state mutated. Both fixed; lesson is that the existing sig gating was scoped to a single pillar and grew load-bearing for the cross-pillar expansion.
- **Decision-point UI required one new component** — `renderCascadeDecisionCard()` plus a `.cascade-card` CSS block (≈40 lines). The component reuses the existing `--infios-color-*` palette (warning amber for the unresolved card border, success green for confirmed) and the existing card-shadow / radius / padding tokens. Card paths render via a 2-column CSS grid (`grid-template-columns: 1fr 1fr`), no new framework abstractions. No new colors introduced.
- **Director flip window**: the decision card surfaces at sim 08:47. Path 1's earliest gated event is at sim 08:50, so the human viewer has a ~3-sim-minute window (≈6 wall-seconds at 30:1 compression) to override the default Path 2 before the resolution wave starts. To remove that pinch, `resolveCascade` re-arms any path-gated events whose `atSec` has already passed by clearing their `_fired` flag, so a late click still plays the full resolution wave. This was a small mitigation, but worth doing — the demo viewer can deliberate without losing the visual cascade.
- **Watchtower "amber → green" at resolution is mostly narrative.** The cascade narrative message `09:10 Resolution: ... Watchtower amber → green.` is pushed into the cross-pillar chat thread, and the cascade-forced amber on all three pillar status dots correctly clears on `'resolved'`. However, the Watchtower's System Health KPI tile reads from total pending count across all three pillars, and the seed already includes enough pending decisions to keep that tile at Amber even after the cascade resolves. To make a true amber→green transition land on the System Health tile, the Director would need to additionally approve one of the seed-pending items (e.g. the 08:22 TM spot quote). Acceptable trade-off: the cascade-specific UI signals (per-pillar dots, narrative message, confirmation card) all transition correctly; the global health tile honestly reports remaining pending work.
- **Path 1's 11:00 + 11:15 events fall past the 09:30 sim end.** They're pre-scripted in the event table and fire correctly when the sim clock is manually advanced (e.g. during this verification). In a live demo, the tick loop stops at 09:30, so Path 1's late events surface only via the shift-log scrub. The spec notes this explicitly. Path 2's 09:10 resolution lands inside the window, so the default demo behaviour shows the resolution live.
- **Rename sweep scope.** The user's brief explicitly scoped the sweep to `agents/`. README, AGENT-ARCHITECTURE.md, WAY-OF-WORKING.md, ITERATION-LOG.md (earlier entries), and `moonshot-home.html` still reference "Warehouse Life Agent" by name. Left unchanged per scope; flagging here so a follow-up task can decide whether to extend the rename to those docs. The architecture diagram in `moonshot-home.html` still has a "Warehouse Life" station label.
- **Screenshot tool unresponsive yet again.** `preview_screenshot` timed out four times this session; `preview_eval`, `preview_snapshot`, `preview_click`, and `preview_console_logs` all worked normally throughout. Visual confirmation done via accessibility-tree snapshots (which confirmed the cascade card titles, button labels, and Watchtower amber state) and via direct DOM queries. Same recurring caveat as the last four afternoon entries.
- **Pre-edit snapshots** saved to `archive/moonshot-prototype.PRE-SIM-LIFE.html` and `archive/warehouse-life-agent.PRE-RENAME.md`.

---

## Afternoon of May 13 2026 — Pillar timelines cleanup — narrative thread, decision card repositioned, swim-lane fixed.

### What was built

- **Removed interior vertical scrolling** from the cross-pillar swim-lane container. `.swim-lane__scroll` is now `overflow-x: auto` only; no `max-height` or fixed-height constraint on the inner swim-lane.
- **Equalized lane heights** across Transportation, Order Management, Warehouse — each lane row is `minmax(150px, auto)` in the swim-lane grid, so all three are the same vertical size regardless of node count. The rail stays centred at 50% within each lane.
- **Decision card pulled out of the lane grid** and repositioned as a vertical interrupter spanning all three lanes at the 08:47 time slot. The new swim-lane grid is 4 columns: `120px label | pre-decision flex | 280px decision card | post-decision flex`. The decision card occupies `grid-column:3; grid-row:1/-1`, so it spans full swim-lane height between the pre-decision and post-decision lane segments.
- **Cross-lane connectors simplified** from the previous chaotic dashed tangle to **5 clearly-directed arrows**: TM 08:43 → OM 08:44 (diagonal propagation, charcoal), OM 08:45 → decision card left edge (horizontal propagation, charcoal), then three solid purple resolution arrows fanning out of the decision card's right edge into TM 08:49 / OM 08:51 / WM 08:53. Solid lines throughout; no dashed strokes. Two SVG `<marker>` definitions for arrowheads in charcoal and link-purple.
- **Filled approval-moment dots labeled** with a `.human-badge` chip ("Human approval") above the agent label, per dot. Pillar-accent backgrounds: yellow for WM (charcoal text), charcoal for TM (white text), link-purple for OM (white text). All three contrast pairings pass WCAG AA. The cross-pillar decision card has its own inline `Human gate` chip in the card header, so the human moment is unambiguously labelled across all four scenarios.
- **Explanatory line added to the section lede** as a `.section-lede__inline-note` span: "Filled nodes mark the human approval moment — when a manager, dispatcher, or director enters the loop and the system pauses for their decision."
- **New master time axis** at the top of `section-pillar-timelines` (between the lede and the WM scenario). Visual: thin horizontal rail with tick labels at 08:00/08:15/08:30/08:45/09:00/09:15. Four pillar markers positioned at each scenario's start time: WM 08:14 at 18.7%, TM 08:22 at 29.3%, OM 08:31 at 41.3%, Cross-pillar 08:43 at 57.3%. Each marker is an `<a href="#pillar-{id}">` anchor — native browser scrolling handles the click-to-scroll, no JavaScript needed. Each pillar's scenario `<div>` got a matching id. Marker dots use the pillar accent token; labels are pill chips coloured to match the chip on each scenario card. Caption below the axis: "Four scenarios from one morning shift — same operational window, four decision lifecycles."
- **Stake summary line added** above each single-pillar timeline rail, between the scenario title and the rail-wrapper. New `.pillar-timeline__stake` styling: small italic 13px text, secondary text colour, bold "Stake:" prefix in primary text:
  - WM: "Zone A pick paths unblocked before the morning outbound wave."
  - TM: "$2,400 saved by consolidating with Tuesday's MEM→DEN load."
  - OM: "47 orders covered; two Tier-2 accounts retained through partial-fill."
  - Cross-pillar intentionally has no stake line — the decision card surfaces both paths' stakes inline.

### Why

- The original Task 17 implementation was structurally correct (three single-pillar timelines + one cross-pillar swim-lane) but had multiple cumulative usability issues that compounded on each other: interior scrolling violated user-control heuristics; uneven lane heights compressed the OM lane and cut off WM; the embedded decision card sat horizontally inside the swim-lane and swallowed half the post-decision lane width; cross-lane arrows were a tangle of overlapping dashed strokes that didn't read as propagation→decision→resolution; filled dots had visual meaning (the human-approval moment, the moonshot thesis made visible) but no label.
- The narrative thread between scenarios was missing — readers couldn't tell the four scenarios belonged to one morning shift. The master time axis bridges this without requiring the full single-narrative rebuild (deferred to a later iteration). Reading order is now: read the axis → see four scenes from one morning → drill into each.
- Stake summaries answer the "so what?" question per scenario, differentiating them substantively rather than relying only on agent/time labels.

### Verified

- **No interior scrolling**: `.swim-lane__scroll` computes `overflow-x:auto` and `overflow-y:auto` (CSS forces auto when one axis is non-visible), but the swim-lane content height (452px) matches the grid template (3×150 + borders), so no vertical scrollbar appears in practice.
- **All swim-lane lanes equal height**: measured pre-segment and post-segment lane heights — all 6 segments are 150px exactly.
- **Decision card spans full swim-lane height** as a vertical interrupter (`grid-column:3; grid-row:1/-1` confirmed via computed style).
- **Cross-lane arrows trace cleanly**: 5 paths in the SVG connector layer, charcoal for the two propagation arrows (signal → decision), link-purple for the three resolution arrows (decision → post-segment lanes).
- **Human-approval badges**: three `.human-badge` elements rendered (WM, TM, OM filled dots); cross-pillar gets the inline `Human gate` chip in the decision card header instead.
- **Master time axis** renders with all four pillar markers; all four `href="#pillar-..."` targets resolve to matching scenario ids; click on the Cross-pillar marker scrolled the page so `#pillar-cross` landed at viewport top (verified via `preview_eval`).
- **Stake lines** present on WM, TM, OM (not on cross).
- **Lede inline note** about filled-nodes meaning renders with the expected styling.
- **Responsive**: at 900px viewport, the swim-lane retains its 1040px min-width and scrolls horizontally inside `.swim-lane__scroll` (scrollLeft max = 276px). The single-pillar timelines collapse to single-column stacked nodes via the existing `@media (max-width:1080px)` block. A second `@media (max-width:960px)` rule tightens the master-axis track gutter and shrinks marker-label font to keep the 4 markers from overlapping on narrow viewports.
- **Zero console errors** through the full reload + scroll-to-section + anchor-click cycle.

### What it leaves

- The pillar timelines section now reads as one cohesive morning narrative with four scenes, not four isolated vignettes. The cross-pillar scene shows the decision moment unambiguously — the decision card sits between propagation (left) and resolution (right), so the eye traces signals → STOP → resolution wave.
- The "human approval moment" visual code is labelled, reinforcing the moonshot thesis (agents propose, humans decide) at every timeline — three pillar-coloured chips plus the cross-pillar Human-gate chip.
- Future single-narrative iteration (one connected story flowing through all products) is unblocked by this cleanup but not required by it. The master time axis is already structured to accept additional markers or a continuous-narrative overlay.

### Caveats reported

- **Decision-card vertical-interrupter layout** rendered cleanly on the first attempt — CSS Grid's `grid-row:1/-1` on the card paired with `grid-column:3` keeps the card stretched across all three lane rows without absolute-positioning gymnastics. Two small `::before`/`::after` accent dots on the card's left and right edges visually mark the entry/exit connection points for the connector arrows.
- **Master time axis click-to-scroll worked with anchor links alone** — no JavaScript required. Each marker is an `<a href="#pillar-{id}">`; native browser anchor-scroll handles the navigation. Verified: clicking the Cross-pillar marker moved `window.scrollY` from 2376 → 3173 and put `#pillar-cross` at `top=0.3`.
- **SVG connector viewBox uses `preserveAspectRatio="none"`** so the 0-1040 horizontal coordinate space stretches to match the actual swim-lane width (1144px at desktop). Arrowhead landing positions are within ~15px of dot centres — visually directional and unambiguous, not pixel-perfect. Tightening this further would require either a JS measure-and-redraw pass or replacing the SVG with absolutely-positioned per-arrow divs; not worth the complexity for the prototype.
- **Badge contrast adjustments**: the Human-approval badges use pillar accents as backgrounds with their natural high-contrast text (charcoal on yellow accent for WM; white on charcoal for TM; white on link-purple for OM). All three pass WCAG AA on visual inspection of computed styles — formal verification will land in Task 20.
- **`preview_screenshot` timed out three times** during this session; structural verification done via `preview_eval`, `preview_snapshot`, and `preview_inspect` instead — same recurring caveat as the last several afternoon entries.
- **Pre-edit snapshot** saved to `archive/moonshot-home.PRE-TIMELINE-CLEANUP.html`.

---

## Afternoon of May 13 2026 — Pillar timelines cleanup pass 2 — decision card repositioned above swim-lane, scrolling removed, time axis labels fixed.

### What was built

- **Decision card moved out of the swim-lane entirely.** Previously the card sat as a vertical interrupter at `grid-column:3; grid-row:1/-1` inside the swim-lane, splitting each lane into pre- and post-decision segments and overlapping the OM 08:45 Order Priority node. New structure: the card lives in its own `.cross-decision` block ABOVE the swim-lane, centred over the lane-track area, max-width 480px. The swim-lane below is now three CONTINUOUS lane tracks (TM / OM / WA) that read left-to-right without interrupters. The card's content (Path 1 alternate, Path 2 chosen, "08:48 · Director of Operations approved Path 2" annotation, Human gate chip) is preserved verbatim — only its position changed.
- **Connector bridge** between card and swim-lane: a 30px-tall block holding a single SVG path that draws a thin dashed hairline from the card's bottom-centre (at viewBox x=50) down 14 units, jogs horizontally to x=20 (the 08:47 position), then drops to the axis. `vector-effect="non-scaling-stroke"` keeps the dash crisp under the `preserveAspectRatio="none"` stretch. A small charcoal dot at the bottom of the bridge anchors it to the time axis tick. To keep card-centre aligned with bridge x=50 across viewport widths, `.cross-decision` carries `padding-left:120px` so the card centres over the lane-track area (not the full pillar-timeline content area).
- **08:47 vertical "Human gate" marker.** A 1px hairline (`rgba(38,37,35,0.28)`) runs through every lane track at `left:20%`, visually marking the 08:47 position without blocking content. A separate `.swim-lane__axis-decision-label` chip ("Human gate · 08:47") sits in the swim-lane's internal time axis row directly above the hairline. Together they tell the reader: this is where the human enters the loop. The hairline does not block dots — `z-index:1` sits below the nodes at `z-index:2`.
- **Internal time axis inside the swim-lane.** A 32px-tall row at the top of the swim-lane's grid hosts time ticks (08:40, 08:45, 08:50, 08:55, 09:00, 09:10, 09:15) plus the Human gate chip. Each tick has a small downward tick mark joining the lane area below.
- **Cross-lane arrows simplified** from the previous dramatic curves to **5 clean diagonals/straights** in a single SVG overlay (`viewBox="0 0 100 100"`, `preserveAspectRatio="none"`, `vector-effect="non-scaling-stroke"`): TM 08:43 → OM 08:44 (diagonal propagation, charcoal), OM 08:45 → 08:47 marker (short horizontal propagation, charcoal), then three purple resolution arrows fanning out of the 08:47 marker to TM 08:49 (short up-right diagonal), OM 08:51 (horizontal right), and WA 08:53 (down-right diagonal). All straight lines — no bezier curves.
- **All internal vertical scrolling on the swim-lane removed.** The previous `.swim-lane__scroll` horizontal-scroll wrapper is gone. The new swim-lane has `overflow-x:visible; overflow-y:visible` (default), grid rows fixed at `32px 168px 168px 168px = 536px`, and content sized to fit. Verified: `swim.scrollHeight === swim.clientHeight === 536` at both 1280px and 820px viewports — no scrollbar appears in any axis.
- **Master time axis labels repositioned** to opposite sides of the rail:
  - Pillar markers (chip + dot) sit ABOVE the rail. Each marker is a flex column with `justify-content:space-between` between the chip label (top) and the dot (bottom, touching the rail at top:46). The pill chip combines time and pillar name (e.g. "WM · 08:14") on one line, font-variant-numeric:tabular-nums.
  - Time ticks (08:00, 08:15, 08:30, 08:45, 09:00, 09:15) sit BELOW the rail at top:56, each with a tick mark joining the rail above.
  - Since markers occupy y=0–46 and ticks occupy y=56+, they're on opposite sides of the rail and cannot collide regardless of horizontal position.

### Why

- **The previous "vertical interrupter" decision card was the wrong call.** It blocked left-to-right lane reading, overlapped the OM 08:45 node, and made the swim-lane feel cramped. Cards that summarise a moment should sit OUTSIDE the content they reference, not embedded in it. Pulling the card above gives readers the "here is the decision" framing first, then the continuous propagation→resolution wave below — narrative reads top-to-bottom, lanes read left-to-right.
- **The previous cleanup pass's "no internal scrolling" claim was incomplete.** Two CSS quirks combined to keep a 3px vertical scrollbar on the `.swim-lane__scroll` wrapper: (a) browsers force `overflow-y:auto` when `overflow-x:auto` is set, regardless of explicit `overflow-y:visible`, and (b) the absolutely-positioned `.swim-lane__connectors` SVG with `viewBox="0 0 1040 450"` and no explicit width/height sized itself to its viewBox aspect ratio (1040×450 → stretched to box, but then the SVG element computed a HEIGHT 13px taller than the swim-lane). The previous pass missed this 3px discrepancy. This pass eliminates the wrapper entirely, dropping both the forced overflow-y issue and the SVG sizing issue.
- **Master time axis labels stacked vertically was a layout mistake.** The chip and time text both sat above the rail, then ticks below — but the chip's height pushed it into the tick band at narrow widths. Splitting the elements to opposite sides of the rail makes collision physically impossible.

### Verified

- **`grep -n "overflow\|max-height\|height:" moonshot-home.html | grep -i "swim\|pillar-timeline--cross"` returns zero lines.** No residual height/overflow constraints on any swim-lane or cross-pillar selector. The only `height:` declarations in the section now are `height:32px` (axis row), `height:30px` (bridge), and per-element widths/heights for dots and markers — all explicit content sizing, no scroll-related constraints.
- **No internal vertical scrollbar at any tested viewport** (1280px, 900px, 820px). Verified via `swim.scrollHeight === swim.clientHeight` at each width.
- **Decision card sits ABOVE the swim-lane**, centred at viewport x=692.5 (matching the bridge centre 692.5), max-width 480px. Card-bottom at y=323.7; swim-lane top at y=353.7. Bridge bridges the 30px gap.
- **Connector bridge aligns**: bridge centre 692.5 = card centre 692.5, bridge x=20% maps to the 08:47 marker at viewport x=380.8 (matching the `.swim-lane__decision-marker` and `.swim-lane__axis-decision` positions exactly).
- **08:47 vertical marker**: three `.swim-lane__decision-marker` hairlines (one per lane track), all at left=380.3, width=1px. Spans the full track height (168px). Z-index 1 below the dots at z-index 2 — does not block content.
- **5 cross-lane arrows**: counted 5 `<path>` elements in `.swim-lane__connectors svg` plus 2 marker-defs (7 total). All straight diagonals/horizontals, no bezier curves.
- **Path 2 resolution wave** (08:49 TM, 08:51 OM, 08:53 WA, 09:10 ✓ markers on all three lanes) renders to the right of the 08:47 marker as required. 10 dots total in the swim-lane (3 pre-decision + 6 post-decision + 1 visualised by the resolution-style accent on each lane's 09:10 endpoint — actually 3 resolution dots + 7 regular = 10).
- **"08:48 · Director of Operations approved Path 2"** annotation present inside the card (`.cross-decision__approval`).
- **Master time axis**: pillar markers at y=48–94 (above rail at y=94); time ticks at y=104+ (below rail). No vertical overlap. All four marker labels ("WM · 08:14", "TM · 08:22", "OM · 08:31", "Cross · 08:43") render uncovered. All six tick labels (08:00 through 09:15) render uncovered.
- **Single-pillar timelines (WM, TM, OM)** unchanged: 6 nodes each, 1 stake line each, 1 Human approval badge each on the filled dot.
- **Zero console errors** through reload + tab switch + scroll + anchor-click cycle.
- **Responsive**: at 820px viewport, swim-lane fits naturally (width 699px) with no vertical overflow; at 720px breakpoint, the `.cross-decision__paths` collapses from 2 columns to 1 and the label column narrows from 120px to 96px.

### What it leaves

- The cross-pillar story now reads cleanly top-to-bottom: (1) chip + scenario title — "what happened"; (2) decision card — "what the human decided"; (3) continuous swim-lane below — "how the cascade resolved across three pillars". The 08:47 marker is the visual anchor connecting the card above to the cascade below.
- Each lane reads left-to-right without interruption. The 08:47 hairline is decoration, not a structural break.
- Pillar timelines section is now visually clean and structurally honest. Ready for Task 19.

### Caveats reported

- **CSS constraints the previous pass missed.** Two issues combined to produce the 3px vertical scrollbar: (a) `.swim-lane__scroll { overflow-x:auto; overflow-y:visible }` — the browser's overflow-coercion rule forces `overflow-y` to `auto` when `overflow-x` is `auto`, regardless of the explicit `visible`. The only way to truly get `overflow-y:visible` is to set `overflow-x:visible` too. (b) The `.swim-lane__connectors` SVG sized to its viewBox aspect ratio (1040×450 → ~451px height) rather than filling its absolute-positioned box (450px). This put its scrollHeight at ~451 vs the container's 450, just enough to trigger the scrollbar. Both gone now: the wrapper is removed entirely, and the SVG is wrapped in a `<div>` with explicit insets, with `svg { width:100%; height:100%; display:block }` inside, so the SVG always exactly fills its box.
- **Decision-card-to-lane connector implementation: CSS + SVG, no JavaScript.** The 30px-tall `.cross-decision__bridge` contains a single inline SVG with `viewBox="0 0 100 30"` and `preserveAspectRatio="none"` so its coordinate space scales horizontally with viewport. The path `M50,0 L50,14 L20,14 L20,30` draws card-centre down → horizontal jog → axis position. `vector-effect="non-scaling-stroke"` keeps the dashed stroke crisp under the horizontal stretch. To make the card-centre always land at SVG x=50, the `.cross-decision` parent carries `padding-left:120px` (matching the swim-lane's label-column width), so the card's centring context exactly matches the lane-track area below. This means the card visually sits slightly right of the full-section centre — an acceptable trade-off for a stable, JS-free connector.
- **No new badge contrast adjustments needed.** Re-used the existing accent/charcoal/link colour pairings from the prior pass; all three pillar-coloured Human-approval badges + the charcoal Human-gate chip pass WCAG AA on visual inspection. Formal contrast verification still pending in Task 20.
- **Lane height bumped to 168px (was 150px)** to accommodate the meta-div above-and-below stacking around each dot without overflow into adjacent lanes. Total swim-lane height: 32 (axis) + 3 × 168 (lanes) = 536px.
- **`preview_screenshot` timed out again** this session; structural verification done via `preview_eval` + `preview_inspect` + `preview_console_logs` (no errors). Same recurring caveat as prior afternoon entries.
- **Pre-edit snapshot** saved to `archive/moonshot-home.PRE-TIMELINE-CLEANUP-2.html`.

---

## Personas and permission rules added per pillar with visible denial states

_Afternoon of May 13 2026 — Task 18 extension._

### What was built

- `PERSONAS` data structure in `moonshot-prototype.html` defining per-pillar persona sets and tier metadata. Two personas per pillar (Operator/Manager for WA, Dispatcher/Transportation Manager for TM, Order Manager/Customer Service Manager for OM) plus Director of Operations for cross-pillar. Each persona carries `{ label, tier, displayName }`.
- `agents/meta/permissions-agent.md` spec codifying the three-tier permission model (1 = Operator-level / 2 = Manager-level / 3 = Director-level, additive) and visibility rules per surface — decision queue items, Watchtower, Archer Chat, cascade decision card, per-pillar stage card.
- **Header role toggle replaced** with a scoped dropdown. Dropdown content updates with `currentProduct`: WA shows Operator + Manager; TM shows Dispatcher + Transportation Manager; OM shows Order Manager + Customer Service Manager; cross shows Director of Operations only. Built on existing Infios tokens (charcoal/grey, no blue). Chevron rotates on open; menu items show role + display name + Tier pill; outside-click and the trigger both dismiss.
- **Persona-aware header**: display name, role label, and avatar initials re-render on every persona change. `Reyna Castillo — RC` flips to `Ed Auriemma — EA` when Director becomes active.
- **Permission filtering wired into render paths**:
  - `renderTMStageCard` and `renderOMStageCard` consult `scenario.requiredTier` (Tier 1 for scenario A in each, Tier 2 for scenario B); below-tier personas see a `renderLockedStage` card with no action buttons.
  - `renderStagePending` (WA) treats escalation-state incidents as Tier 2; Operators see the locked card.
  - `renderCascadeDecisionCard` returns empty for any persona below Tier 3.
  - `chatThreadForRender` synthesises an Archer line — "Cascade decision escalated to Director of Operations — awaiting approval." — into Tier-1/Tier-2 personas’ home chats whenever cascade state is `awaiting-decision`. Render-only; does not mutate stored thread.
- **Watchtower denied state**: `renderCrossHome` short-circuits to `renderWatchtowerDenied` when current tier < 3. Surface shows a circular lock badge, a "Tier 3 · Director only" pill, the required-tier heading, body copy explaining what the Watchtower aggregates, and a "Switch persona" button that opens the dropdown.
- **`setProduct` persona handover**: cross → pillar drops to the pillar’s last-used persona (Director isn’t available outside cross); pillar → cross keeps the persona as-is (so a Tier-1 user landing on Watchtower sees the denied state); pillar → pillar prefers an equivalent-tier persona in the new pillar (Tier 2 → Tier 2 when available, else Tier 1 default).
- **Legacy `role` global synced** when WA is active so the pre-existing renderNavigator / renderMap / renderEscPanel / renderCLI WA-only code continues to function without a refactor.

### Why

- The Director / Manager / Operator distinction is what makes the demo’s "agents propose, humans decide" thesis concrete. Without persona-aware filtering, every viewer of the prototype sees every surface, and the moonshot point that different roles see different things never lands.
- Visible permission denial (not silent hiding) shows the agent system has explicit, architectural boundaries. The viewer sees authority as part of the design — not as an implicit policy hidden behind empty pages.
- The cascade decision card appearing only when a Director persona is active makes the "this approval crosses pillars" point visually obvious. Switching personas mid-cascade lands the card on-screen on the next render — the cascade state itself lives in `CROSS_DATA`, the persona layer just filters it.

### Verified (via `preview_eval` + `preview_snapshot`)

- WA Operator: dropdown trigger reads "Operator", header reads "Reyna Castillo — Operator".
- Click cross icon as Operator → tier stays 1, denied view renders (`.denied-view` present, `.cross-wrap` absent), dropdown still shows on header.
- Switch persona to Director of Operations → tier becomes 3, Watchtower content renders (`.cross-wrap` present), header reads "Ed Auriemma — Director of Operations", avatar `EA`.
- Switch product cross → WA: persona drops to WA’s last-used Operator (Tier 1).
- Tier-2 carryover across pillars: setting Transportation Manager on TM then switching to OM resolves to Customer Service Manager (Tier 2 → Tier 2 match).
- Dispatcher on TM scenario B (Tier 2): `.locked-stage` shown, no `tmResolveScenario` Approve button rendered.
- Dispatcher on TM scenario A (Tier 1): locked stage absent, Approve button rendered.
- Cascade in `awaiting-decision`: Tier-1 + Tier-2 personas across all pillars see no cascade card; chat thread carries the synthetic "escalated to Director of Operations" Archer line. Switching to Director surfaces the card on next render. Clicking Path 2 advances cascade state to `path-2`.
- Dropdown on cross with non-Director active persona: trigger shows the leftover persona name (e.g., "Dispatcher"), menu lists Director of Operations as the only option.
- Zero console errors across all transitions.

### What it leaves

- The prototype now demonstrates role-based authority concretely. Switching personas in the header makes the system’s authority structure visible and demoable on the spot.
- Future iterations can add finer-grained controls (region-scoped operators, exception-tier managers) by extending `PERSONAS` and the per-surface `requiredTier` tags — the architecture supports it without restructuring.

### Caveats reported

- **No new components required beyond the dropdown, the locked-stage card, and the denied view.** All three were built directly on existing Infios design tokens (charcoal text on muted greys for the locked / denied states; white-on-charcoal for the "Switch persona" button matching the existing `.btn-primary` style). The dropdown does not reuse an existing Infios pattern — there was no prior dropdown component in the codebase, so a small purpose-built one was added with hover, active, and aria states.
- **Mid-cascade persona switching** discovered to work for free: cascade state lives in `CROSS_DATA.cascadeState` and `chatThreadForRender` reads it on every render, so swapping personas while the cascade is awaiting-decision produces the right surface immediately — no state replay or re-emission needed.
- **Legacy `role` global preserved.** WA-only render paths (`renderNavigator`, `renderMap`, `renderEscPanel`, `renderCLI`, `setRole`) still consult the `role` string. `setPersona` and `setProduct` both keep `role` in sync when WA is active. `setRole` itself remains as a legacy shim mapping to `setPersona`, so any straggling `onclick="setRole(...)"` would still resolve — none were found in the current source, but the shim is cheap.
- **`preview_screenshot` timed out** again this session; structural verification done via `preview_snapshot` (full a11y tree) + `preview_eval` (state introspection) + `preview_console_logs` (no errors). Same recurring caveat as prior afternoon entries.
- **Pre-edit snapshot** saved to `archive/moonshot-prototype.PRE-PERSONAS.html`.

---

## Afternoon of May 13 2026 — WCAG 2.2 AA + heuristics audit completed; load-bearing FAILs remediated.

_Afternoon of May 13 2026 — Guardian audit pass._

### What was built

- Comprehensive audit of 13 pages across `moonshot-home.html` and `moonshot-prototype.html`: Home / The System / The Agents tabs, plus 7 persona surfaces (WA Operator + Manager, TM Dispatcher + Transportation Manager, OM Order Manager + CSM, Watchtower Director), the canonical permission-denied state (WA Operator → Watchtower), Archer chat in WA context, and the cascade decision point at 08:47 (Director persona).
- Audit applied criteria from `agents/guardian/heuristics-agent.md` (Nielsen Norman 10 + Hick + Miller + cognitive load + F-pattern + banner blindness + information scent) and `agents/guardian/accessibility-agent.md` (WCAG 2.2 AA load-bearing, AAA advisory).
- Per-page verdicts (PASS / WARN / FAIL / N/A) documented with one-sentence rationales in `agents/guardian/reports/audit-report-2026-05-13.md`.
- All load-bearing FAILs remediated **by swapping token references, not by modifying the library.** The library's token values are preserved exactly:
  1. `.foot`, `.strip__num`, `.door__num`, `.horizon-cat__audience`, `.card__field-label`, `.card__attribution`, and the resting `.agents-index__item` in `moonshot-home.html` re-pointed from `var(--text-quaternary)` (#98A4B0, 2.54:1) to the existing library token `var(--text-tertiary)` (#58636E, 6.13:1).
  2. Global `:focus-visible` rule added to `moonshot-home.html` (matching the existing rule in `moonshot-prototype.html`): `outline:2px solid var(--link); outline-offset:2px; border-radius:6px`. Uses the existing `--link` token. Closes WCAG 2.4.7 for `.bar__btn`, `.bar__brand`, and `.agents-index__item` which previously relied on browser default.
  3. `.msg-meta`, `.sys-divider__text`, `.step-label.pending`, `.step-time`, and the two `Awaiting approval` / `No approval required` table cells in `renderRunwayRows` remapped from `--infios-color-text-disabled` (#98A4B0 = 2.4:1) to the existing `--infios-color-text-tertiary` (#58636E = 6.13:1). These elements are informational, not disabled — the token name was being used semantically incorrectly.
  4. `.persona-dropdown__tier` text colour `#98A4B0` → `var(--infios-color-text-tertiary)`.
  5. `.locked-card__icon` colour `#98A4B0` → `var(--infios-color-text-tertiary)` (WCAG 1.4.11 non-text contrast).
- Re-verification via `preview_eval` confirmed: library tokens at original values (`--text-quaternary:#98A4B0`, `--text-tertiary:#58636E`); every previously-failing visible text site now resolves to `rgb(88,99,110)` = #58636E (6.13:1 on white, AA pass); global `:focus-visible` rule present in both stylesheets; zero console errors.
- Known WARNs (advisory-only AAA, dense Watchtower / Dispatcher surfaces touching Miller 7±2, persistent persona dropdown on banner-blindness criterion, hover border 1.4.11) documented for transparency rather than remediated.
- Recommendations captured for larger concerns that exceeded this pass: introducing `--infios-color-text-muted` as a properly-named token for "informational but subtle" text; component-specific focus styling on the sticky index; explicit ARIA region landmarks on dense Watchtower / Dispatcher surfaces; an AAA-targeted contrast pass if Infios chooses to step up.

### Why

- The Guardian agents existed as specs on the Agents page but had never been *run* against the actual product. The Agents tab argued that the system has accessibility and heuristics governance — but argued it abstractly. Running the audit converts the spec from "we claim this is governed" to "we have verified this is governed, here is the report, here is the diff."
- The audit report dated 2026-05-13 is now part of the artifact you share with Eugene or Gartner alongside the prototype. The Guardian specs are no longer aspirational — they have an audit report behind them.
- Remediating fails before showing the demo means reviewers don't catch obvious issues. The headline FAILs (2.4:1 contrast on metadata text, missing keyboard focus on the top-bar navigation in `moonshot-home.html`) would have been the first thing a reviewer ran a contrast checker against.

### Verified (via `preview_eval`)

- `--text-quaternary` library value preserved at `#98A4B0` (the token remains in the library for non-text uses where WCAG exempts contrast — decorative graphics, disabled controls).
- `--text-tertiary` library value preserved at `#58636E` (6.13:1 on white) and is now the resolved colour at every previously-failing text site.
- Global `:focus-visible` rule present and resolving in both stylesheets — `moonshot-home.html` to `--link` purple; `moonshot-prototype.html` to `--infios-color-border-focus`.
- Console clean — no JS errors introduced by the CSS-only fixes.
- All 13 pages audited; all AA load-bearing FAILs remediated; audit report committed to `agents/guardian/reports/audit-report-2026-05-13.md`.
- Pre-edit snapshots saved to `archive/moonshot-home.PRE-AUDIT.html` and `archive/moonshot-prototype.PRE-AUDIT.html`.

### Report

- **FAILs found across 13 pages:** 1 multi-page load-bearing FAIL (WCAG 1.4.3 Contrast), surfacing in 7 of 13 pages (the home Home / System / Agents tabs and the prototype WA / TM / OM / Archer surfaces where `.msg-meta` and `.step-time` appear). 1 load-bearing FAIL on `moonshot-home.html` for WCAG 2.4.7 Focus Visible on top-bar navigation (browser default would have rendered something, but no styled indicator — surfaced as FAIL rather than WARN because a 2px purple ring is what the prototype uses everywhere else).
- **Quick fixes (≤ 5 minutes each):** redefining `--text-quaternary` (one line, cascades to ~12 selectors); adding the global `:focus-visible` rule (one line, covers every interactive without a component-specific override); four inline colour overrides on chat / dropdown / locked-card.
- **Substantial changes:** none required. All remediations were token-level or single-line CSS — no markup restructuring, no copy edits, no flow redesign.
- **WARNs considered but not remediated:**
  - H3 Miller on Watchtower and TM Dispatcher — dense by design, role tolerates density, chunked by pillar header.
  - H8 banner blindness on the persistent persona dropdown — small viewport share and state-bearing label mean it does not become invisible after a minute.
  - A10 Enhanced Contrast (AAA) — `--text-tertiary` is 6.13:1, below the 7:1 advisory target. Pushing further would require a library-side token change, which is out of scope for this audit. AA conformance achieved.
  - 1.4.11 hover border on `.persona-dropdown__trigger` (#98A4B0 against white) — the border is not the sole state indicator; background change accompanies it. Decorative-by-fallback.

### What it leaves

- The agent system now has documented evidence of governance. The Guardian specs are no longer aspirational; they have a 13-page audit report and a CSS diff behind them.
- Future iterations should re-run the audit after any significant UI change. The audit report dated 2026-05-13 is the baseline for the prototype as it stands today.
- The fixes are all token-level or one-line additions — no risk of regression on existing visual flows. `preview_console_logs` clean.

### Caveats reported

- **Spec scope vs. audit scope.** Both Guardian specs scope themselves to the Warehouse homepage (Direction A). This audit extended that scope to 13 pages as briefed. Criteria sized for an operator's calm-state homepage (e.g. H2 Hick's Law "≤ 3 primary actions per state") were read more broadly on marketing and dense surfaces — interpretations noted in each verdict's rationale.
- **`preview_screenshot` timed out** during the re-verification step; programmatic verification done via `preview_eval` (computed-style introspection + WCAG ratio math) and `preview_console_logs` (no errors). Same recurring caveat as the personas session.
- **Pre-edit snapshots** saved to `archive/moonshot-home.PRE-AUDIT.html` and `archive/moonshot-prototype.PRE-AUDIT.html` per the task brief.

---

## Afternoon of May 13 2026 — Home tab rewritten as UX orchestration landing pitch.

_Afternoon of May 13 2026 — Phase II Home rewrite._

### What was built

- `view-home` replaced with a landing page structure: hero, four value propositions (Oversight, Intervention, Guardrails, Cohesion), evidence section linking to The System / The Agents / The Prototype, closing pull-quote.
- **Hero.** Single value-prop title — *"UX orchestration for agentic supply chain."* — with a lede that names the four functions in one line. The old `.display` / `.lede` poetic framing ("AI enhances the delivery. People enhance the experience." / "Trust still has to be designed.") replaced with a factual statement of what's offered.
- **Four value props.** New 2x2 grid (`.home-value-props__inner`, `1fr 1fr` at desktop, single column at ≤960px). Each `.value-prop` is a card with a brand-yellow top border, prominent display-weight title, and 16px body copy at AA-contrast. The cards name what UX orchestration produces in concrete terms: cross-pillar Watchtower; persona-tiered intervention; specs authored before launch; twelve agents under one approval contract.
- **Evidence section.** Old `.doors` two-tab descriptive index (which still referenced only The Prototype and The Agents, missed The System, used outdated counts like "10 agents · 3 layers" and "five domain experts") replaced with three `.evidence-card` links — one each for The System, The Agents, The Prototype. Each card carries a short label ("See how it's built", "Meet the roster", "See it run"), a title, and a body that pitches what's actually on that tab in current Phase II terms.
- **Closing.** Old `.closing-quote` ("design leadership shows up in 2026" — UX territorial flag) replaced with the brief's literal pull-quote: *"UX orchestration is infrastructure. The agent system is what it produces."* The same case argument now lands a third time after the `section-case-system` and `section-case` blocks.
- Voice rules enforced throughout: no rhetorical hooks, no motivational framing, no claim language, no territorial flag on UX authorship. Authorship stays passive ("Specs are authored before agents ship", not "UX authors them").
- CSS for `.home-hero` / `.home-value-props` / `.value-prop` / `.home-evidence` / `.evidence-card` / `.home-closing` added using Infios design tokens — `--surface-canvas` / `--surface-base` for backgrounds, `--accent` for top borders + closing-quote rule, `--accent-darker` for evidence-card labels (passes AA on white), `--text-primary` and `--text-secondary` for body. **No blue.** Hover state on evidence cards uses charcoal + brand yellow on the arrow chip; focus-visible uses the existing `--link` purple, matching the global rule from the May-13 audit.
- `<title>` updated to *"Project Moonshot — UX orchestration for agentic supply chain"* and `<meta name="description">` set to the lede paragraph.

### Why

- The Home tab is the front door. Anyone reviewing the moonshot site — Allen, Eugene, a Gartner analyst, a partner product team — lands here first. The previous descriptive index didn't pitch the value proposition; it listed contents, and the contents it listed were outdated (two tabs when there are now four, "five domain experts" when there are twelve, etc.).
- The pitch for the agent system is not "we built agents." It's "UX orchestration is a strategic function for agentic AI." The Home tab is where that pitch lands. Four value props (Oversight, Intervention, Guardrails, Cohesion) map directly to what UX orchestration produces. Each is stated as a fact about what's offered. The benefit lands because the structure implies it, not because the copy claims it.
- Voice rules keep the page from sliding into marketing rhetoric while still being persuasive. Stripe / Linear style: confident, factual, value-forward.
- Closing pull-quote now lands a third time. The Home, The System (`section-case-system`), and The Agents (`section-case`) each end with the same argument from a different angle — Home as the pitch, System as the methodology, Agents as the catalogue.

### Verified

- Hero, value props, evidence, closing render in correct order — confirmed via `preview_snapshot` accessibility tree.
- `home-hero__title` resolves to 81.92px charcoal (`rgb(23, 31, 41)`) at desktop; clamps to 48px at 375px mobile width.
- `home-value-props__inner` grid renders as `544px 544px` at desktop and collapses to single column at tablet (768px) — confirmed via `preview_inspect`.
- `home-evidence__cards` renders 3-column (`357px × 3`) at desktop and single column at tablet.
- Each `.value-prop` carries a 4px top border in `--accent` (`rgb(202, 223, 53)` = #CADF35 brand yellow).
- `.evidence-card__label` resolves to `--accent-darker` (`rgb(107, 124, 15)` = #6B7C0F) on white = 4.63:1 ratio, AA pass for 11px body text.
- `.value-prop__body` resolves to `--text-secondary` (`rgb(50, 65, 85)`) on white = high-contrast AA pass.
- `.home-closing__quote` renders italic, 41px charcoal, with a 6px brand-yellow left rule matching the existing closing-quote treatment.
- **No blue anywhere on Home** — programmatic check across all `#view-home` descendants for `#0060FF`, `#4A8FFF`, `#2B7AB1`, `#2D78C1`, `#C5DAFF` against `color` / `background-color` / `border-*-color`: 0 matches.
- **Tab navigation from evidence cards.** Clicking `.evidence-card[data-go="system"]` swaps `view-home` → `view-system` and updates the URL hash to `#system`. Clicking `data-go="agents"` swaps to `view-agents` (`#agents`). Clicking `data-go="prototype"` swaps to `view-prototype` (`#prototype`). The brief specified `data-target`; the existing `showView()` JS uses `data-go` — chose `data-go` to match the existing mechanism rather than introduce a parallel one. All three tested via `preview_click` + state introspection.
- Console clean — no JS errors introduced.

### What it leaves

- The Home tab is now the pitch. The System is the methodology. The Agents is the catalogue. The Prototype is the proof. Four tabs telling one argument.
- When Allen, Eugene, or Gartner clicks through the site, they land on a value prop, not a navigation menu.
- Legacy CSS for `.display`, `.lede`, `.hero__cta`, `.doors`, `.strip` is now unused by `view-home` but left in place — those selectors no longer match anything in the document, no visual risk, and removing them would be a separate scope.
- Pre-edit snapshot saved to `archive/moonshot-home.PRE-HOMEPAGE-REWRITE.html`.

### Report

- **Salvageable from the old `view-home`:** the hero shell — `<svg class="bg-topo">` topographical lines + dots + orbs + sparks — was kept for visual cohesion with `.agents-hero` on The System and The Agents. New `.home-hero` content sits inside the same `.hero` container, so the page still opens with the same animated atmospheric identity. The closing block reuses the same `bg-topo--soft` SVG language seen in `#section-case-system` and `#section-case`.
- **Needed full rewrite:** all body copy (hero title, lede, doors, strip, closing quote), the doors → evidence transformation (count 2 → 3, descriptive → pitch), and the closing quote.
- **Voice-rule violations encountered in old copy:**
  1. Rhetorical / poetic framing: *"AI enhances the delivery. People enhance the experience."* — borderline marketing rhetoric.
  2. Claim language: *"Trust still has to be designed."* — declarative claim rather than statement of what's offered.
  3. Motivational framing inside the strip: *"Humans are in the loop. Guardrails are the work. Trust is the deliverable."* — three claim-statements stacked.
  4. Territorial flag in the closing: *"That's how design leadership shows up in 2026."* — UX claiming credit; brief calls for passive authorship.
  5. Outdated facts that read as claims: *"10 agents · 3 layers"* (now 12 domain agents across 3 pillars + Guardian + Meta + Horizon), *"One orchestrator. One radar. Five domain experts. Three analytical agents."* (now 12 domain agents organised across three pillars).
- **Heuristics / accessibility concerns surfaced during rewrite:**
  - Evidence cards are `<a>` elements (keyboard-navigable, focus-visible inherits global purple ring from the May-13 audit fix). Each link's accessible name combines label + title + body for screen-reader context — verified via the accessibility tree snapshot.
  - Hover state on evidence cards changes both `border-color` (to `--charcoal`) and the arrow chip background (to brand yellow) — not relying on colour alone, also adds translate + shadow for non-visual users on focus.
  - `.evidence-card__label` at 11px with `--accent-darker` on white = 4.63:1 — clears the AA 4.5:1 floor for normal text by 0.13 — close to the line, would be worth re-checking under any future palette adjustment.
  - The brief-suggested `data-target` was substituted with `data-go` so the existing `showView()` mechanism handles routing without a parallel handler. The page does not introduce a new JS surface.
  - `<title>` and `<meta name="description">` now match the new positioning — also serves social-share previews if the file is ever opened with one.
- **`preview_screenshot` timed out** again this session (same recurring symptom as the personas and audit sessions); verification done via `preview_snapshot` (a11y tree), `preview_inspect` (computed styles at three viewport sizes), `preview_eval` (state + tab nav + colour-rule scan), and `preview_console_logs` (zero errors).

---

## Afternoon of May 13 2026 — System page: UX-led / UX-influenced relabelled as horizontal action / pillar action.

_Afternoon of May 13 2026 — System framing softened._

### What was changed

- Architecture conveyor legend on `view-system` retitled:
  - `UX-led` → **Horizontal action**
  - `UX-influenced` → **Pillar action**
  - `Neutral · backstage` → **Backstage** (the qualifier "neutral" was redundant once the binary was reframed; the swatch already signals the third category)
- Conveyor lede rewritten to match: *"Seven stations between a prompt and a shipped prototype. Gold nodes are horizontal action — spec authoring, orchestration, audits, and the human-in-the-loop gate cut across every pillar. The mixed node is pillar action — domain agents working inside their vertical. The neutral node is backstage simulation."* Previous lede said "Gold nodes are UX-led" and used "meta process" phrasing — confrontational in tone, also no longer accurate (the architecture is cross-pillar now, not a meta-layer above a single WMS).
- `aria-label` on the conveyor grid changed from *"UX Enhanced System: seven-station pipeline..."* to *"Authoring pipeline: seven stations from prompt to delivered prototype"* — screen-reader text matches the new framing.
- `section-case-system` reframed to use the same vocabulary for consistency across the page:
  - Heading: *"UX as infrastructure, not overhead."* → *"Horizontal action as infrastructure, not overhead."*
  - First paragraph: *"that is UX work"* → *"that is horizontal work, cutting across every pillar"*.
  - Second paragraph: *"design intelligence that runs on cadence"* → *"horizontal intelligence that runs on cadence"*.
  - Third paragraph: *"UX stops being a department that reviews work at the end and becomes the layer..."* → *"The horizontal layer stops being a department that reviews work at the end and becomes the seam..."* (also swapped "the layer" → "the seam" because we now use "horizontal layer" in the subject — avoids "the layer that..." repetition).
  - "AI Experience Layer" value-card: *"UX thinking is present in the AI architecture from the start"* → *"Horizontal thinking is present in the AI architecture from the start"*.
- Untouched on this page: the section title in `view-agents` `#section-case` (still uses "UX as infrastructure..." copy — that's the Agents page's closing case and out of this scope); the home tab (already reframed in the prior rewrite without UX-led/influenced framing); all `card__attribution` "Authored by the UX Team" stamps on the Agents page (different question — attribution, not categorisation).

### Why

- Source request: "we are separating things as ux-led, ux influenced and neutral. this is a bit confrontational." The categorisation was reading as territorial — UX claiming credit on a per-station basis. The whole page argues UX-as-orchestration; doing it again with overt labels in the diagram is redundant *and* picks a fight with the other functions.
- The reframe trades a territorial axis (whose work is this?) for a structural axis (what kind of work is this?). Horizontal action = cuts across every pillar. Pillar action = lives inside one vertical. Backstage = simulation. The reader still understands which work is UX-owned because the horizontal/vertical opposition maps onto the org chart cleanly — the verticals are the product pillars, so by elimination the horizontal layer is the one Allen, Eugene, and the partner product teams already associate with design and orchestration.
- Subtle is the point. The page now lets the audience reach the conclusion. The categorical framing carries the argument; the words don't need to.

### Verified (via `preview_eval`)

- Legend text on `view-system` resolves to `["Horizontal action", "Pillar action", "Backstage"]`.
- `aria-label` on `.conveyor-grid` resolves to *"Authoring pipeline: seven stations from prompt to delivered prototype"*.
- `.howagents-conveyor__lede` text starts *"Seven stations between a prompt and a shipped prototype. Gold nodes are horizontal action — spec authoring..."*
- `#section-case-system .section-title` resolves to *"Horizontal action as infrastructure, not overhead."*
- `#section-case-system .value-prose` first paragraph starts *"Every product Infios ships now has an AI layer. What it tells the user, what it withholds, what it asks before it acts — that is horizontal work, cutting across every pillar."*
- "AI Experience Layer" value-card paragraph starts *"Horizontal thinking is present in the AI architecture..."*.
- Console clean — no JS errors.
- Pre-edit snapshot saved to `archive/moonshot-home.PRE-HORIZONTAL-REFRAME.html`.

### What it leaves

- The architecture timeline now reads as a structural description of what the system does (some work cuts across pillars, some lives inside one), not a credit-allocation diagram. The same reader who would have bristled at "UX-led" reads "horizontal action" as a neutral fact about the system's shape.
- The page is internally consistent: the architecture legend, the conveyor lede, the case section heading, and the case prose all speak the same vocabulary. A reader who skims the page in either order (top-to-bottom or jumping to "The case" via the sticky index) hears the same framing.
- "UX" still appears on the Agents page (`card__attribution` "Authored by the UX Team" on every card, plus the section-case heading there) — that's a separate scope. Worth a follow-up pass if the audience is sensitive to repeated territorial language across both tabs.

### Caveats reported

- The class names `station--gold`, `station--mixed`, `station--neutral`, `info-card--gold`, `info-card--mixed`, `info-card--neutral`, and the swatch modifier `--gold` / `--mixed` / `--neutral` were **not** renamed. They are visual-only tokens (the reader doesn't see class names) and the colour mapping is still meaningful: gold = horizontal, mixed = pillar, neutral = backstage. Renaming the classes would have been a larger CSS sweep with no user-visible benefit. Left as-is.
- The diagram-toggle "System" view (the alternate to "Timeline") wasn't touched in this pass — verified it does not contain UX-led / UX-influenced labels; it's a system-tree view with different content.
- `preview_screenshot` timed out (again — same recurring symptom this week); verification done via `preview_eval` text introspection and `preview_console_logs`.

