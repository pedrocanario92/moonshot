# Project Moonshot — UX-Owned Agentic WMS

A two-part argument that **UX practitioners must own the AI capabilities being built around their discipline.**

1. **Move 1 — a real agentic system for a WMS.** Approval-first model: every agent proposes, no agent executes alone. Demonstrates what human-in-the-loop AI looks like in a production UX context — approval gates, escalation logic, role-scoped views, agent performance monitoring.
2. **Move 2 — extending the argument to the UX practice itself.** Guardian agents that run quality checks (Heuristics, Accessibility) before each homepage ships. Horizon agents that extend design intelligence into work that currently never gets done.

The shipped deliverable is the pair `moonshot-home.html` + `moonshot-prototype.html`, both at the repo root. `moonshot-home.html` is the strategy doc + agent showcase, with the prototype embedded via an iframe pointing at `moonshot-prototype.html`. Both files are edited directly; before significant changes, snapshot the current version into `archive/` first as a rollback.

---

## File map

```
moonshot-demo-repo/
├── README.md                         ← this file (start here)
├── AGENT-ARCHITECTURE.md             ← deep architecture document; source of truth for the agent system
├── WAY-OF-WORKING.md                 ← operating rules and ship discipline
├── ITERATION-LOG.md                  ← session-by-session history
├── agent-response-schema.json        ← machine-readable AgentProposal contract (JSON Schema) — see AGENT-ARCHITECTURE.md §6.2 and §6.5
│
├── moonshot-home.html                ← SHIPPED — strategy doc + Agents page + Prototype tab
├── moonshot-prototype.html           ← SHIPPED — prototype iframed inside moonshot-home.html's Prototype tab
│                                       (both files live at root; the iframe uses a relative src)
│
├── agents/                           ← agent specs (Markdown, one per agent — 18 total)
│   ├── archer-chat-agent.md                   ← cross-pillar orchestration addendum added May 13 PM
│   ├── shift-intelligence-agent.md
│   ├── pick-path-agent.md
│   ├── labor-agent.md
│   ├── order-priority-agent.md        ← dual-context (WMS + OMS) addendum added May 13 PM
│   ├── carrier-agent.md               ← dual-context (WMS + TMS) addendum added May 13 PM
│   ├── exception-agent.md             ← scoped to transactional errors only (mispicks, short-picks, scan errors)
│   ├── slotting-agent.md              ← replen, bin assignment, fast-mover re-slotting
│   ├── equipment-agent.md             ← forklifts, AMRs, conveyors, charge cycles
│   ├── quality-agent.md               ← damage, quality holds, cycle counts, audit risk
│   ├── spot-rate-agent.md             ← TMS: spot quotes vs contracted lane rate (added May 13 PM)
│   ├── route-agent.md                 ← TMS: lane health, mode selection, multi-stop routing (added May 13 PM)
│   ├── allocation-agent.md            ← OMS: constrained-inventory allocation (added May 13 PM)
│   ├── cutoff-manager-agent.md        ← OMS: carrier cutoff windows at order layer (added May 13 PM)
│   ├── analytical-agents.md           ← BI · Innovation · Opportunity (shared spec, analysis mode only)
│   ├── meta/
│   │   └── warehouse-life-agent.md    ← scripted-causal sim engine (backstage; not in WMS topology)
│   └── guardian/
│       ├── heuristics-agent.md        ← NN10 + Hick + Miller + cognitive-load audit
│       └── accessibility-agent.md     ← WCAG 2.2 audit (AA load-bearing, AAA advisory)
│
├── next tasks/                       ← current task lists and trackers
│   ├── tasks-may-13-morning.md       ↳ seven prompts for the May 13 morning (cross-pillar build)
│   └── moonshot-cross-pillar-tracker.html  ↳ progress tracker for the May 13 prompts
│
├── task archive/                     ← completed task lists (historical)
└── archive/                          ← earlier iterations, planning files, token deliveries, and rollback snapshots
```

---

## What to read next

In order:

1. **`AGENT-ARCHITECTURE.md`** — the deep architecture document. Source of truth for the agent system: design principles, agent layer decisions, approval and escalation logic, product surfaces, key metrics, risks, design system tokens.
2. **`WAY-OF-WORKING.md`** — operating rules. File-by-file purpose, how to extend each layer, ship discipline.
3. **`ITERATION-LOG.md`** — session-by-session history. Read end-to-end for the full project narrative through end of May 12 morning.
4. **`next tasks/tasks-may-13-morning.md`** — current task list. Seven prompts for the May 13 morning (cross-pillar architecture: product switcher, sidebar restructure, TM + OM stub views, cross-pillar dashboard, agent response schema, archive-and-verify).

---

## The agents (summary)

Full details in `AGENT-ARCHITECTURE.md`. This table is the at-a-glance roster:

### Agent roster (in-fiction — agents that run inside the products)

Twelve Domain agents across three pillars (WMS / TMS / OMS) as of May 13 afternoon. Two of those agents — Carrier and Order Priority — operate in dual contexts across pillars; their pillar column lists both. See `AGENT-ARCHITECTURE.md` §6.6 for the cross-pillar treatment.

| Layer | Agent | Pillar(s) | Role |
|---|---|---|---|
| Orchestration | **Archer Chat Agent** | All | Sole human-facing agent. Single point of contact across WMS / TMS / OMS. |
| Radar | **Shift Intelligence Agent** | WMS | Continuous monitor across all zones. Signal-only — never proposes. |
| Domain | **Pick Path** | WMS | Throughput. Aisle routing. Congestion. |
| Domain | **Labor** | WMS | Workforce flex. The primary fix lever. |
| Domain | **Order Priority** | WMS + OMS | SLA clock. Ship-by window. Reprioritisation across both pillars. |
| Domain | **Carrier** | WMS + TMS | Carrier ETA visibility and window management across both pillars. |
| Domain | **Exception** | WMS | Mispicks, short-picks, scan errors. |
| Domain | **Slotting** | WMS | Replen, bin assignment, fast-mover re-slotting. |
| Domain | **Equipment** | WMS | Forklifts, AMRs, conveyors, charge cycles. |
| Domain | **Quality** | WMS | Damage, quality holds, cycle counts, audit risk. |
| Domain | **Spot Rate** | TMS | Spot quotes vs contracted lane rate. |
| Domain | **Route** | TMS | Lane health, mode selection, multi-stop routing. |
| Domain | **Allocation** | OMS | Constrained-inventory allocation. |
| Domain | **Cutoff Manager** | OMS | Carrier cutoff windows at the order layer. |
| Analytical | **BI · Innovation · Opportunity** | All | Read-only analysis modes. |

All Domain agents operate under a **3-tier constraint model**:

- **Tier 1** — never under any circumstances
- **Tier 2** — propose only; human confirms before execution
- **Tier 3** — autonomous (reads, internal calculations, routing)

### Meta (backstage — not in the in-fiction topology)

- **Warehouse Life Agent** — scripted-causal sim engine. Generates root events on the 08:00–09:30 timeline; causal rules emit derived events. Drives the prototype's "alive" feel. Never surfaces to the user; lives only in `moonshot-prototype.html` and must remain absent from any shipped homepage variant.

### Guardian (process gate — audits the prototype before it ships)

- **Heuristics Agent** — Nielsen Norman 10 + Hick's Law + Miller's 7±2 + cognitive load + F-pattern + banner blindness + information scent.
- **Accessibility Agent** — WCAG 2.2 (AA load-bearing, AAA advisory).

Both run before every Warehouse-homepage deploy. They produce a verdict report per criterion (PASS / WARN / FAIL / N/A) with rationale. Only the human approves ship.

Four more Guardian agents have cards on the Agents page but no full specs yet: **Design System · Content · Interaction · Responsive**. Specs are deferred to future sessions.

### Three shared patterns across every WMS agent

1. **Per-action `confidence` (0–100)** — calibrated on each individual decision, not a per-agent rolling stat. The supervisor sees how sure the agent is about *this specific proposal*.
2. **`escalationReason`** — when an agent declines to recommend, it states *why* a human is needed, in its own words. The rationale lives next to the decision.
3. **`options[]` with "no recommendation chosen"** — when the agent is uncertain, it surfaces alternatives without forcing a choice. The supervisor's affordances become Open / Defer / Why? instead of Approve / Reject. Calibrated uncertainty is honored as a first-class state.

### Three UX design choices baked into the prototype

1. **Autonomous actions live only in the log, never on the home page.** Surfacing every agent action on the home page creates noise without decision value. The home page is reserved for things that need the supervisor's attention.
2. **One inline focus card, plus a queue chip.** Side-by-side parallel queues split attention and inflate cognitive load. A single focused decision at a time, with clear visibility into what's queued, respects Miller's 7±2 and Nielsen's minimalist-design heuristic.
3. **Categorical Tier 1 / 2 / 3 autonomy, not a continuous slider.** Each tier maps to an auditable policy ("never execute alone," "propose then confirm," "autonomous read-only"). Categorical boundaries are easier to govern, test, and explain to operators than a continuous "more cautious ↔ more bold" dial.

---

## Where the project is right now

*As of end of May 13 afternoon.*

### The deliverable pair (at repo root)

- `moonshot-home.html` — strategy doc + Agents page + Prototype tab. The Agents page hosts an Apple-style segmented Timeline · System toggle in its Architecture section. Timeline view shows a zigzag conveyor of seven UX stations from prompt to delivered prototype. System view shows a polished CSS Grid hierarchy tree of the operational chain plus lateral Analytical, Guardian, and Meta layers.
- `moonshot-prototype.html` — iframed into `moonshot-home.html`'s Prototype tab via relative `src`. Now a **multi-product container**: header product switcher (WA · TM · OM · cross), sidebar split into agnostic + domain tiers, three pillar dashboards, and a cross-pillar roll-up.
- The pair is zip-shippable. Don't share HTML files as bare attachments — Microsoft Defender flags standalone HTML with embedded scripts.

### The cross-pillar container

Driven by Richard Stewart's May 12 directive (pillar-specific demos first, agnostic vs domain components, cross-pillar roll-up, Gartner deadline in ~16 days). State governed by a single `currentProduct` variable: `'wa' | 'tm' | 'om' | 'cross'`. Header product switcher mutates it via `setProduct(p)`; `renderMain()` branches on `currentProduct` first, then on `view`.

- **Agnostic components** (shared across all products): Dashboard slot · Archer chat · Agent Performance · Activity Log pattern · Decision Queue.
- **Domain components** (swap with `currentProduct`):
  - **WA** — Floor Map, Shift Log, Warehouse view, zones strip (sim-engine-driven).
  - **TM** — Network Map, Activity Log (lanes), TM dashboard with lanes strip and carrier scenarios (static, sourced from `TM_DATA`).
  - **OM** — Order Dashboard pipeline, Order Log, channels strip and stock-out / cutoff scenarios (static, sourced from `OM_DATA`).
- **Cross-pillar dashboard** (`currentProduct === 'cross'`) — three product health cards, unified decision queue with product badges, Archer terminal seeded with cross-product correlations, aggregated KPIs.

The machine-readable contract for agent responses lives at `agent-response-schema.json` (repo root). It formalises section 6.2's three shared patterns (`confidence`, `escalationReason`, `options[]`) and adds `criticality`, `tier`, `product`, `impacts[]` — the cross-product correlation shape Allen Oleksak's monitoring-agent specs will plug into.

**Twelve Domain agents across three pillars are now specced in `/agents/`.** Four new specs landed May 13 afternoon: `spot-rate-agent.md` and `route-agent.md` (TMS), `allocation-agent.md` and `cutoff-manager-agent.md` (OMS). `carrier-agent.md` and `order-priority-agent.md` gained cross-pillar addenda documenting dual-context operation (Carrier across WMS dock-window + TM lane-window; Order Priority across WMS floor-released SLA + OMS upstream channel-intake). `archer-chat-agent.md` gained a cross-pillar orchestration addendum. See `AGENT-ARCHITECTURE.md` §6.6 for the architecture treatment.

### The prototype itself

Single-file vanilla JS state machine. Runs by double-clicking `moonshot-prototype.html`. Active surfaces:

- **WA Home dashboard** — 4-column manager layout: zones strip (capacity bars + throughput metrics + pulse-glow attention) · elevated stage card (now rendered inline on the home grid; the modal-overlay treatment was retired May 13 PM so WA matches TM and OM behaviourally) · soft-dark terminal panel · KPI strip with sparklines and semantic-polarity trend arrows.
- **TM Home dashboard** — same 4-column layout, parallel shape: lanes strip · transportation decision card (Werner late or 2.3× spot-quote) · TM-context Archer terminal · TM KPIs (Active Loads · On-Time Delivery · Open Alerts).
- **OM Home dashboard** — same 4-column layout: channels strip · order management decision card (stock-out SKU or wholesale cutoff breach) · OM-context Archer terminal · OM KPIs.
- **Cross-pillar dashboard** — three product health cards · unified decision queue · cross-context Archer terminal (brought flush to its containing tile May 13 PM; right edge now aligned with the product cards row above) · aggregated KPIs.
- **Agent CLI** — full-bleed dark terminal panel. Single conversation thread per pillar.
- **Warehouse dashboard** — comprehensive WA operational view: hero KPIs, hourly throughput, wave progress, inventory health, ops tiles, status tiles.
- **Floor map · Shift log · Agent performance** — WA supporting surfaces unchanged from earlier sessions; agent performance is agnostic and visible in all pillars.
- **TM Activity Log + Network Map · OM Order Log + Order Dashboard** — domain surfaces for each non-WA pillar.
- **Sim engine** — 7 root events on 08:00–09:30 timeline, 30:1 compression. Two user-facing decisions; two autonomous resolutions through the action log. Sim ticks are gated by `currentProduct === 'wa'` so non-WA pillars are not disturbed.

### What's outstanding

- **Allen's monitoring-agent specs** plug into `agent-response-schema.json` — pending Allen's work.
- **Prompt 7** from `next tasks/tasks-may-13-morning.md`: snapshot the prior `moonshot-prototype.html` into `archive/moonshot-prototype.PRE-MAY-13.html` and run the cross-product verification pass on `moonshot-home.html`. Carry-over from this morning.
- **WA sim clock** continues to advance while a non-WA pillar is mounted (re-renders are gated; only `WAREHOUSE.clockSec` keeps ticking). Decide whether to pause the sim on `setProduct(non-wa)` or keep it running.
- **Font air-gap** — `<link>` to `fonts.googleapis.com` still in both root files. System fallback works offline; strict portability would inline the woff2 or ship a `fonts/` subfolder.
- **Parked token decisions** from May 12 afternoon Prompt 1 (legacy Infios text/border/link tokens, whether to consolidate `--accent` to `#D0FF25`).
- **`.note__*` orphan CSS sweep** from the morning of May 12.

---

## Working style

- **Out of auto mode by default.** Ask clarifying questions for any ambiguous decision and get explicit go-ahead before editing. Answering clarifying questions is NOT the same as saying "start."
- **Direct, no-fluff updates.** Diagnose root causes, not symptoms.
- **Verify in the browser via the preview tool**, not by assertion.
- **Never claim a screenshot is fine if the screenshot tool timed out** — say "couldn't capture" and use DOM / geometry checks instead.
- **`moonshot-home.html` and `moonshot-prototype.html` are the shipped artifacts.** Treat them with the same caution as a production deploy. Before significant edits, snapshot the current version into `archive/` first as a rollback (e.g., `archive/moonshot-prototype.PRE-<YYYY-MM-DD>.html`).
- **Never share HTML files as bare attachments in Teams or Outlook.** Microsoft Defender flags standalone HTML files with embedded scripts. Always zip first.

---

## Brief a new Claude Code session

Copy everything inside the code fence below (start at `You're joining`, end at `Then we go.`) and paste it as the first message in a fresh Claude Code chat to bring the model up to speed.

```
You're joining a UX project mid-flight. The UX team is building an AI-augmented UX process — a set of human-in-the-loop agentic patterns and tooling that can act as an auxiliary to any product. We're demonstrating the approach end-to-end on top of a Warehouse Management System automation demo originally built by Allen Oleksak; that demo is the lineage and the springboard. From there we've extended into a full agentic UX system with approval-first orchestration, role-scoped views, agent performance monitoring, and a Guardian layer that audits the prototype itself for usability and accessibility before it ships.

Read these files first to load context:

1. README.md at the repo root — file map, agent roster summary, current project state, working style.
2. AGENT-ARCHITECTURE.md at the repo root — the deep architecture document. Source of truth for the agent system.
3. WAY-OF-WORKING.md at the same repo root — operating rules and ship discipline.
4. ITERATION-LOG.md at the same repo root — full session-by-session history through end of May 12 morning.

Then glance at:
- moonshot-home.html — the shipped strategy doc + agent showcase + embedded prototype viewer (at repo root).
- moonshot-prototype.html — the prototype iframed inside moonshot-home.html's Prototype tab (also at repo root). Single-file vanilla JS state machine, runs by double-clicking.
- agents/ — 18 markdown specs (12 Domain + 1 Orchestration + 1 Radar + 1 Analytical-trio + 1 Meta + 2 Guardian).

Agent roster:
- In-fiction (twelve Domain agents across three pillars — WMS, TMS, OMS): Lead (orchestration, all pillars), Shift Intelligence (radar, WMS). WMS Domain: Pick Path, Labor, Order Priority, Carrier, Exception, Slotting, Equipment, Quality. TMS Domain: Spot Rate, Route. OMS Domain: Allocation, Cutoff Manager. Carrier and Order Priority are dual-context (Carrier: WMS + TMS; Order Priority: WMS + OMS) — same Tier 1/2/3 boundaries, different operational substrate. Plus the BI / Innovation / Opportunity analytical layer.
- The cross-pillar dashboard is the entry point when a customer runs more than one Infios product; it rolls up product health, the unified decision queue, and cross-product correlations from a single Archer terminal.
- Meta (backstage): Warehouse Life Agent — scripted-causal sim engine.
- Guardian (process gate): Heuristics (NN10 + Hick + Miller + cognitive load) and Accessibility (WCAG 2.2). Run before every homepage deploy.
- Three shared agent patterns: per-action confidence (0–100), escalationReason rationale, options[] "no recommendation chosen" mode for uncertain proposals.
- Three UX design choices baked into the prototype: autonomous actions live only in the log (not on the home page), one inline focus card + queue chip (not parallel queues), categorical Tier 1/2/3 autonomy (not a continuous slider).

If there's a task list inside `next tasks/` matching tasks-*.md, that's the current working plan — read it next.

Working style the user expects:
- Out of auto mode by default. Ask clarifying questions for any ambiguous decision and get explicit go-ahead before editing. Answering clarifying questions is NOT the same as saying "start."
- Direct, no-fluff updates. Diagnose root causes, not symptoms.
- Verify in the browser via the preview tool, not by assertion.
- Never claim a screenshot is fine if the screenshot tool timed out — say "couldn't capture" and use DOM / geometry checks instead.
- moonshot-home.html and moonshot-prototype.html (both at repo root) are the shipped artifacts. Before significant edits, snapshot the current version into archive/ first as a rollback.
- Never share HTML files as bare attachments in Teams or Outlook — always zip first.

Start by reading the files. Confirm context. Then we go.
```
