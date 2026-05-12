# Project Moonshot — UX-Owned Agentic WMS

A two-part argument that **UX practitioners must own the AI capabilities being built around their discipline.**

1. **Move 1 — a real agentic system for a WMS.** Approval-first model: every agent proposes, no agent executes alone. Demonstrates what human-in-the-loop AI looks like in a production UX context — approval gates, escalation logic, role-scoped views, agent performance monitoring.
2. **Move 2 — extending the argument to the UX practice itself.** Guardian agents that run quality checks (Heuristics, Accessibility) before each homepage ships. Horizon agents that extend design intelligence into work that currently never gets done.

The single shipped deliverable is `moonshot presentation/moonshot.html` — a strategy doc + agent showcase + embedded prototype viewer in one file. The active working copy lives at `moonshot prototype/moonshot-prototype.html`.

---

## File map

```
moonshot-demo-repo/
├── README.md                         ← this file (start here)
├── AGENT-ARCHITECTURE.md             ← deep architecture document; source of truth for the agent system
├── WAY-OF-WORKING.md                 ← operating rules and ship discipline
├── ITERATION-LOG.md                  ← session-by-session history
├── tasks-afternoon-may-12.md         ← current task list (six prompts for the May 12 afternoon)
│
├── moonshot presentation/            ← SHIPPED, ZIP-AND-SEND artifact
│   ├── moonshot.html                  ↳ strategy doc + Agents page + Prototype tab
│   ├── moonshot-prototype.html        ↳ prototype iframed inside moonshot.html's Prototype tab
│   └── archive/                       ↳ previously-shipped artifacts (preserved for rollback)
│
├── moonshot prototype/               ← active working copy
│   └── moonshot-prototype.html        ↳ where the agentic system is being built
│
├── agents/                           ← agent specs (Markdown, one per agent — 14 total)
│   ├── lead-agent.md
│   ├── shift-intelligence-agent.md
│   ├── pick-path-agent.md
│   ├── labor-agent.md
│   ├── order-priority-agent.md
│   ├── carrier-agent.md
│   ├── exception-agent.md             ← scoped to transactional errors only (mispicks, short-picks, scan errors)
│   ├── slotting-agent.md              ← replen, bin assignment, fast-mover re-slotting
│   ├── equipment-agent.md             ← forklifts, AMRs, conveyors, charge cycles
│   ├── quality-agent.md               ← damage, quality holds, cycle counts, audit risk
│   ├── analytical-agents.md           ← BI · Innovation · Opportunity (shared spec, analysis mode only)
│   ├── meta/
│   │   └── warehouse-life-agent.md    ← scripted-causal sim engine (backstage; not in WMS topology)
│   └── guardian/
│       ├── heuristics-agent.md        ← NN10 + Hick + Miller + cognitive-load audit
│       └── accessibility-agent.md     ← WCAG 2.2 audit (AA load-bearing, AAA advisory)
│
├── original references/
│   ├── allen-demo.html                ← Allen Oleksak's WMS demo (lineage; springboard for this project)
│   └── initial claude prompt - wms agent cli prototype.md
│
├── prompt prototype/                 ← the one-shot prompt experiment (run in a separate fresh chat)
│   ├── README.md                      ↳ explains the two forms (A: control · B: real MVP)
│   ├── full prompt prototype/PROMPT.md           ↳ Form A — everything inlined
│   └── agent loaded prototype/PROMPT.md          ↳ Form B — agents folder + focused build prompt
│
├── task archive/                     ← completed task lists (historical)
│   └── MAY-12-PLAN.md                 ↳ archived end of May 12 morning
│
├── archive/                          ← earlier iterations (HTML, planning files, token deliveries)
└── inspiration/                      ← reference screenshots
```

---

## What to read next

In order:

1. **`AGENT-ARCHITECTURE.md`** — the deep architecture document. Source of truth for the agent system: design principles, agent layer decisions, approval and escalation logic, product surfaces, key metrics, risks, design system tokens.
2. **`WAY-OF-WORKING.md`** — operating rules. File-by-file purpose, how to extend each layer, ship discipline.
3. **`ITERATION-LOG.md`** — session-by-session history. Read end-to-end for the full project narrative through end of May 12 morning.
4. **`tasks-afternoon-may-12.md`** — current task list. Six prompts for the May 12 afternoon (design-tokens skill, alignment fix, accordion refactor, navbar update, verification pass, one-shot prompt experiment).

---

## The agents (summary)

Full details in `AGENT-ARCHITECTURE.md`. This table is the at-a-glance roster:

### WMS topology (in-fiction — agents that run inside the warehouse product)

| Layer | Agent | Role |
|---|---|---|
| Orchestration | **Lead Agent** | Sole human-facing agent. The single point of contact for the supervisor; enforces the approval gate. |
| Radar | **Shift Intelligence Agent** | Continuous monitor across all zones. Signal-only — never proposes. |
| Domain | **Pick Path** · **Labor** · **Order Priority** · **Carrier** · **Exception** · **Slotting** · **Equipment** · **Quality** | Eight specialists. Each owns a problem space; each emits Tier 2 proposals with `confidence`, `escalationReason`, `options[]`. |
| Analytical | **BI** · **Innovation** · **Opportunity** | Analysis mode only — read-only, no operational write actions. |

All Domain agents operate under a **3-tier constraint model**:

- **Tier 1** — never under any circumstances
- **Tier 2** — propose only; human confirms before execution
- **Tier 3** — autonomous (reads, internal calculations, routing)

### Meta (backstage — not in the in-fiction topology)

- **Warehouse Life Agent** — scripted-causal sim engine. Generates root events on the 08:00–09:30 timeline; causal rules emit derived events. Drives the prototype's "alive" feel. Never surfaces to the user; lives only in the working prototype, not the shipped product.

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

*As of end of May 12 morning.*

### The shipped artifact (`moonshot presentation/`)

- `moonshot.html` — strategy doc + Agents page + Prototype tab. The Agents page hosts an Apple-style segmented Timeline · System toggle in its Architecture section. Timeline view shows a zigzag conveyor of seven UX stations from prompt to delivered prototype. System view shows a polished CSS Grid hierarchy tree of the operational chain plus lateral Analytical, Guardian, and Meta layers.
- `moonshot-prototype.html` — the iframed prototype, freshly promoted from the working copy on May 12 morning. Renders the manager dashboard, Warehouse dashboard, dark terminal, KPI sparklines, and floating banner.
- The pair is zip-shippable. Don't share HTML files as bare attachments — Microsoft Defender flags standalone HTML with embedded scripts.

### The working prototype (`moonshot prototype/moonshot-prototype.html`)

Single-file vanilla JS state machine. Runs by double-clicking. Active surfaces:

- **Home dashboard** — 4-column manager layout: zones strip (capacity bars + throughput metrics + pulse-glow attention) · elevated stage card · soft-dark terminal panel · KPI strip with sparklines and semantic-polarity trend arrows.
- **Agent CLI** — full-bleed dark terminal panel. Single conversation thread across home + CLI surfaces.
- **Warehouse dashboard** — comprehensive operational view: hero KPIs, hourly throughput, wave progress, inventory health, ops tiles, status tiles.
- **Floor map · Shift log · Agent performance** — supporting surfaces unchanged from earlier sessions.
- **Sim engine** — 7 root events on 08:00–09:30 timeline, 30:1 compression. Two user-facing decisions; two autonomous resolutions through the action log.

### What's outstanding

The afternoon's six prompts are listed in `tasks-afternoon-may-12.md`:

1. Install the design-tokens skill at `.claude/skills/design-tokens.md` (auto-triggered on token deliveries), then run it for the May 12 token zip.
2. Fix the horizontal alignment of the Architecture section's System view.
3. Convert agent cards into row-synchronised accordions (collapsed default; full body in expanded state; whole row expands together).
4. Update the sticky in-page navbar (add Architecture and Signals links).
5. Full verification pass across all morning + afternoon work.
6. Run the one-shot agent-driven prototype build (**separate fresh chat** — uncontaminated session).

---

## Working style

- **Out of auto mode by default.** Ask clarifying questions for any ambiguous decision and get explicit go-ahead before editing. Answering clarifying questions is NOT the same as saying "start."
- **Direct, no-fluff updates.** Diagnose root causes, not symptoms.
- **Verify in the browser via the preview tool**, not by assertion.
- **Never claim a screenshot is fine if the screenshot tool timed out** — say "couldn't capture" and use DOM / geometry checks instead.
- **`moonshot presentation/` is the shipped artifact.** Treat it with the same caution as a production deploy. Never overwrite without explicit go-ahead. Archive the old file first.
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
- moonshot prototype/moonshot-prototype.html — the active working prototype. Single-file vanilla JS state machine, runs by double-clicking.
- moonshot presentation/moonshot.html — the shipped strategy doc + agent showcase + embedded prototype viewer.
- moonshot presentation/moonshot-prototype.html — the prototype iframed inside moonshot.html's Prototype tab.
- agents/ — 14 markdown specs (8 Domain + 1 Orchestration + 1 Radar + 1 Analytical-trio + 1 Meta + 2 Guardian).

Agent roster:
- WMS topology (in-fiction): Lead (orchestration), Shift Intelligence (radar), and eight Domain agents (Pick Path, Labor, Order Priority, Carrier, Exception, Slotting, Equipment, Quality), plus the BI / Innovation / Opportunity analytical layer.
- Meta (backstage): Warehouse Life Agent — scripted-causal sim engine.
- Guardian (process gate): Heuristics (NN10 + Hick + Miller + cognitive load) and Accessibility (WCAG 2.2). Run before every homepage deploy.
- Three shared agent patterns: per-action confidence (0–100), escalationReason rationale, options[] "no recommendation chosen" mode for uncertain proposals.
- Three UX design choices baked into the prototype: autonomous actions live only in the log (not on the home page), one inline focus card + queue chip (not parallel queues), categorical Tier 1/2/3 autonomy (not a continuous slider).

If there's a task list at the repo root matching tasks-*.md, that's the current working plan — read it next.

Working style the user expects:
- Out of auto mode by default. Ask clarifying questions for any ambiguous decision and get explicit go-ahead before editing. Answering clarifying questions is NOT the same as saying "start."
- Direct, no-fluff updates. Diagnose root causes, not symptoms.
- Verify in the browser via the preview tool, not by assertion.
- Never claim a screenshot is fine if the screenshot tool timed out — say "couldn't capture" and use DOM / geometry checks instead.
- moonshot presentation/ is the shipped artifact. Never overwrite without explicit go-ahead. Archive the old file first.
- Never share HTML files as bare attachments in Teams or Outlook — always zip first.

Start by reading the files. Confirm context. Then we go.
```
