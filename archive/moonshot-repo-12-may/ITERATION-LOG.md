# Moonshot Prototype — Iteration Log
> Morning of May 11 2026 — sessions 0–4. Afternoon of May 11 2026 — session 5. Morning of May 12 2026 — Tasks 1–4 of MAY-12-PLAN. Early afternoon of May 12 2026 — Prompts 1–5 of `tasks-afternoon-may-12.md`. Mid-afternoon — architecture width, navbar centering + Infios-yellow propagation, topology rebuild, page-wide soft-topo pizzazz.

> A session-by-session record of how this prototype evolved. Written for colleagues who weren't in the room, so they can see what was decided, why, and where we are now.

---

## What this project is

See `README.md` for the project framing and `AGENT-ARCHITECTURE.md` for the deep design rationale. This log is the session-by-session record of how the prototype evolved.

---

## Folder restructure (Session 0)

Before any work, we separated the two artifacts:

| Folder | Contents |
|---|---|
| `moonshot presentation/` | The shipped pair — `moonshot.html` + `wms-unified-figma-sidebar.html`. Untouched, ship-ready. Zip these two when sharing. |
| `moonshot prototype/` | The working copy — `moonshot-prototype.html`. Edited freely without touching the finished demo. |
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
- **Active prototype file**: `moonshot prototype/moonshot-prototype.html`
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
4. `moonshot prototype/moonshot-prototype.html` — the active prototype.

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
├── moonshot presentation/             ← shipped, untouched
│   ├── moonshot.html
│   └── wms-unified-figma-sidebar.html
├── moonshot prototype/                ← active working copy
│   └── moonshot-prototype.html
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

**What**: audited `moonshot prototype/moonshot-prototype.html` against the four cleanup categories from the plan (unused CSS selectors, debug `console.*` calls, commented-out blocks longer than two lines, orphaned `DATA.*` properties superseded by `WAREHOUSE.*` equivalents). The reconnaissance pass turned up almost nothing: zero console calls, zero commented-out blocks of meaningful size, zero orphaned `DATA.*` reads. Two CSS selectors were flagged as candidates — `.home-shift` and `.sys-stage-pill` — and re-grepping showed the first was a genuinely empty rule (kept the class in HTML, dropped the empty CSS), while the second was load-bearing for the "Stage 2 baseline" pill on the System health panel and was left alone. Net change: `moonshot-prototype.html` shrank from 4,426 to 4,425 lines.

**Why**: the plan asked for a conservative pass before promoting the prototype to the shipped folder. The discovery — that the file is already lean — was useful information in itself, and the temptation to over-prune was resisted.

**Verified**: re-grepped both selectors against HTML + JS before deleting; opened the prototype in the browser preview and ran through the full 3-minute demo. No regressions.

### Task 2 — Promote the working prototype to the shipped artifact

**What**: three steps.

1. **Archived** the previously-shipped `moonshot presentation/wms-unified-figma-sidebar.html` (2,508 lines) to `moonshot presentation/archive/wms-unified-figma-sidebar.PRE-MAY-12.html`. Archive lives on disk in case rollback is needed.
2. **Copied** `moonshot prototype/moonshot-prototype.html` into `moonshot presentation/` and **renamed** it to `moonshot-prototype.html` (dropping the legacy `wms-unified-figma-sidebar` name — the figma sidebar variant is no longer relevant to the project).
3. **Updated** the iframe `src` in `moonshot presentation/moonshot.html` to point at the new filename. Confirmed no other references to the old filename remained.

**Why**: the presentation folder's prototype had been stale relative to the working copy for weeks. Promoting in a single swing keeps the shipped artifact aligned without introducing a rename ripple across the rest of the codebase.

**Verified**: opened the Prototype tab in moonshot.html; manager dashboard, Warehouse view, KPI sparklines, soft-dark terminal, and floating banner all rendered inside the iframe with no console errors.

### Task 3 — Audit, attribute, and re-shape The Agents page

The longest task by file diff. It started as "add the six new agents to the Agents page" and grew into a substantial reconciliation across cards, specs, structure, and copy. Broken into ordered sub-steps:

**3a — Agent folder reconciliation.** Walked `/agents/` (14 spec files) against `moonshot presentation/moonshot.html`. Found:
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

- **`moonshot prototype/moonshot-prototype.html`** — cleaner by one line. Otherwise unchanged from end of May 11. Still the source of truth for prototype iteration.
- **`moonshot presentation/moonshot-prototype.html`** — newly promoted artifact, byte-identical to the working prototype. Old `wms-unified-figma-sidebar.html` archived under `archive/`.
- **`moonshot presentation/moonshot.html`** — The Agents page substantially rebuilt. Six new agent cards added across Domain (Slotting, Equipment, Quality), Guardian (Heuristics, Accessibility upgraded), and a brand-new Meta layer (Warehouse Life). Kicker and lede updated to reflect six layers / twenty agents. "How the signals move" topology expanded with new edges and lateral-layer paragraphs. Architecture section rebuilt with a Timeline · System toggle — Timeline view holds the polished zigzag conveyor with the seven-station, two-row info-card layout; System view holds a polished CSS Grid hierarchy tree showing the operational spine and the three lateral layer boxes. Home page closing pull-quote relocated from the (now-deleted) howagents view. The standalone "How the agents work" page no longer exists in nav or DOM.
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

Reconciled the union into `moonshot presentation/moonshot.html`'s `:root` and mirrored byte-identically to **three** files — not two as the original brief stated. Discovery: the deck's Prototype tab iframe loads `moonshot presentation/moonshot-prototype.html` (the sibling copy promoted in the morning), not `moonshot prototype/moonshot-prototype.html`. All three `:root` blocks now sit at 9,769 chars byte-identical. Sweep replaced standalone `color:#CADF35` declarations with `var(--wa-nav-accent)` in both prototype copies (12 sites each) and the deck's `border-top:4px solid #CADF35` with `var(--accent)`. Multi-stop gradients using `#CADF35` as a middle stop alongside `#E4F26B` / `#B5C82F` were left intact — flagged as intentional off-grid in TOKENS.md.

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

**Fixes** (three additions to [moonshot presentation/moonshot.html](moonshot presentation/moonshot.html)):
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
2. **External Google Fonts CDN.** Both `moonshot.html` and `moonshot-prototype.html` `<link>` to `fonts.googleapis.com` for Noto Sans (+ JetBrains Mono on the deck). The `--font-display` token includes a system fallback stack, so offline machines render in the fallback without a broken layout — but strictly, the zip-shippability rule *"no `<link>` pointing outside the folder"* is violated.

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
