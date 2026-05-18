# Agent Architecture
### Design Decisions & Rationale
**Product:** WMS AI Agent CLI — Warehouse Advantage
**Archetype:** High-volume e-commerce (primary)
**Status:** Prototype phase — Stage 2 (approve everything)

The source-of-truth design document for the agent system. For project intent, file map, and quick orientation see `README.md`. For operating rules see `WAY-OF-WORKING.md`. For session history see `ITERATION-LOG.md`.

---

## 1. The Problem We Are Solving

The core problem in a warehouse shift is **coordination latency** — the gap between when a bottleneck forms and when the right person has enough visibility to act on it.

Two failure modes drive this:

- **Visibility of the problem** — nobody sees it forming until it is already blocking operations. The dashboard updates every 5 minutes. A pick lane can block in 3.
- **Visibility of the solution** — when someone does see the problem, they do not know what levers are available to fix it in real time.

The labor manager is the most exposed: they need to quickly resolve shift issues in an unstable planning environment, with labor as the primary fix lever — but without real-time visibility into who is available, where, and at what cost to other tasks.

**What we are building is not a chatbot.** It is a proactive signal layer on top of WMS data — one that monitors continuously, detects anomalies before they become blockers, and surfaces specific, actionable recommendations to the right person at the right moment.

---

## 2. Core Design Principles

These principles were established through interrogation and govern every architectural decision below.

| Principle | Statement |
|---|---|
| **Chat is always dominant** | The chat panel is never smaller than 70% of the content area. Secondary surfaces overlay or appear alongside — they never replace. |
| **Human in the loop — always at Stage 2** | Every consequential action is proposed by an agent and confirmed by a human before execution. No action executes silently. |
| **Context isolation per tenant** | One agent brain, N isolated instances. Each tenant's operational data never bleeds into another session. |
| **Explicit over implicit** | Every agent action is tagged with criticality, attributed to an actor, timestamped, and logged. Nothing is assumed or inferred by the user. |
| **Fail loudly, never silently** | A context load failure must surface as an explicit error — not a spinner that resolves into a wrong-context agent. |
| **Accountability is structural** | Human-validated actions and auto-executed actions are visually distinct at every level — badge, background, actor field, icon. They cannot be confused. |
| **Calibrated uncertainty is first-class** | When an agent isn't sure, it says so via `confidence`, `escalationReason`, and `options[]`. Uncertainty becomes a UI state, not a hidden fact. |

---

## 3. Architecture Overview

The system has three distinct product surfaces, an agent layer (operational + analytical + meta + guardian), and a shared data foundation.

```
┌─────────────────────────────────────────────────────────────────────┐
│ PRODUCT SURFACES                                                    │
│                                                                     │
│  CLI Chat Interface      Action Log           Agent Performance     │
│  Operator · Supervisor   Operator · Supervisor  Warehouse · Ops mgr │
│                                                                     │
├─────────────────────────────────────────────────────────────────────┤
│ CONTEXT + SESSION LAYER                                             │
│ Tenant · role · archetype · operational data · health check         │
├─────────────────────────────────────────────────────────────────────┤
│ OPERATIONAL AGENT LAYER (in-fiction — runs inside the WMS)          │
│                                                                     │
│  Lead Agent (orchestrator)                                          │
│  └── Shift Intelligence Agent (radar — always on)                  │
│       ├── Pick Path Agent      (congestion, aisle routing)          │
│       ├── Labor Agent          (workforce flex, redeployment)       │
│       ├── Order Priority Agent (SLA risk, ship-by clock)            │
│       ├── Carrier Agent        (ETA, reroute, window management)    │
│       ├── Exception Agent      (mispicks, short-picks, scan errors) │
│       ├── Slotting Agent       (replen, bin assignment, re-slotting)│
│       ├── Equipment Agent      (forklifts, AMRs, conveyors, faults) │
│       └── Quality Agent        (damage, holds, cycle-count variance)│
│                                                                     │
│  Analytical Layer (analysis mode only)                              │
│       ├── BI Agent             (patterns, what happened)            │
│       ├── Innovation Agent     (challenges assumptions)             │
│       └── Opportunity Agent    (hidden signal, correlation)         │
│                                                                     │
├─────────────────────────────────────────────────────────────────────┤
│ APPROVAL + ESCALATION LAYER                                         │
│ Urgent (escalates) · Standard (queues) · Auto-executed (logs only)  │
├─────────────────────────────────────────────────────────────────────┤
│ SHARED AUDIT + SESSION STATE                                        │
│ All actions · all switches · all approvals · 100% completeness      │
├─────────────────────────────────────────────────────────────────────┤
│ WMS INTEGRATION                                                     │
│ Read APIs (all agents) · Write APIs (approved actions only)         │
└─────────────────────────────────────────────────────────────────────┘

         ┌── META LAYER (backstage — prototype only) ──┐
         │  Warehouse Life Agent                       │
         │  Scripted-causal sim engine                 │
         │  Emits events; agents react                 │
         │  Never surfaces to user                     │
         └─────────────────────────────────────────────┘

         ┌── GUARDIAN LAYER (process gate — pre-deploy) ┐
         │  Heuristics Agent     (NN10 + cognitive load)│
         │  Accessibility Agent  (WCAG 2.2 AA)          │
         │  + Design System / Content / Interaction /   │
         │    Responsive — cards present, specs deferred│
         │  Outside WMS topology. Audits the artifact.  │
         └──────────────────────────────────────────────┘
```

---

## 4. Agent Layer Decisions

### 4.1 The Lead Agent is an orchestrator — not a domain expert

**Decision:** The Lead Agent routes intent, manages conversation state, dispatches to sub-agents, and enforces approval gates. It does not monitor the warehouse or propose actions.

**Why:** Separating orchestration from domain knowledge keeps each layer replaceable. If a domain agent needs to be retrained or swapped, the orchestration logic is unaffected.

**What was rejected:** A single monolithic agent that both monitors and acts. Rejected because it creates an undebuggable system — when it fails, there is no layer boundary to diagnose against.

---

### 4.2 The Shift Intelligence Agent is the radar, not the responder

**Decision:** A dedicated Shift Intelligence Agent monitors all zones continuously, detects anomalies, and signals the Lead Agent. It does not propose actions or receive approvals. It feeds everything else.

**Why:** In high-volume e-commerce, the dominant problem is detection latency — bottlenecks form faster than the 5-minute dashboard refresh. The Shift Intelligence Agent closes this gap by watching continuously and pushing signals rather than waiting to be queried.

**Value it creates:** Transforms the system from reactive (operator notices problem) to proactive (agent detects anomaly before it blocks operations).

**Critical dependency:** Its health directly affects all domain agents. If the Shift Intelligence Agent misses an anomaly, the domain agents never activate. This is why it has a distinct card on the Agent Performance page — positioned first, spanning full width, with its own metric set.

**Metrics that matter for this agent (different from domain agents):**
- Anomalies detected 24H
- Signal-to-noise ratio (flagged anomalies that led to real actions)
- Average detection-to-alert latency
- Estimated missed anomaly rate

---

### 4.3 High-volume e-commerce agent roster — eight domain agents

**Decision (as of May 12):** The domain agent roster is **Pick Path · Labor · Order Priority · Carrier · Exception · Slotting · Equipment · Quality** — eight agents. Three of them (Slotting, Equipment, Quality) were originally deferred but became spec'd and active on May 11.

| Agent | Status | Provenance |
|---|---|---|
| Pick Path | Active | Original high-volume roster |
| Labor | Active | Original high-volume roster |
| Order Priority | Active | Merger of Order + SLA Clock — owns the full SLA lifecycle |
| Carrier | Active | Original high-volume roster |
| Exception | **Rescoped (May 11)** | Originally bundled damage + quality holds + cycle counts. Now scoped to transactional errors only — mispicks, short-picks, scan / putaway errors. |
| Slotting | **Promoted from deferred (May 11)** | Adapted from Allen Oleksak's demo. Owns replen scheduling, bin assignment, fast-mover re-slotting. |
| Equipment | **Promoted from deferred (May 11)** | Adapted from Allen Oleksak's demo. Physical asset state — forklifts, AMRs, conveyors, charge cycles, equipment faults. **NEW domain not in the original roster.** |
| Quality | **Promoted from deferred (May 11)** | Adapted from Allen Oleksak's demo. Carved out of the old Exception agent's scope — owns damage triage, quality holds, cycle-count variance, audit risk. |

**What changed and why:**

- The Equipment Agent fills a real gap. Without it, equipment incidents (conveyor halt, forklift battery low, AMR stall) had no agent owner — they fell through to Exception or got attributed to "the system." Now they have a home.
- The Quality carve-out separates two fundamentally different decision patterns: quality reasoning (audit risk, claim filing, supplier liability) is not the same as transactional-error reasoning (substitute, quarantine, re-pick). The Quality Agent owns the former; Exception owns the latter.
- The Slotting Agent's "no recommendation chosen" pattern (when two source locations are equally viable) was specifically adopted from Allen Oleksak's demo and is one of the load-bearing patterns of the `options[]` schema field.

**Still deferred:**

- **Dock Agent** — secondary in pure high-volume e-commerce where the dominant flow is outbound. Becomes primary in B2B.

---

### 4.4 Analytical layer — three agents, one data context

**Decision:** Three analytical agents (BI, Innovation, Opportunity) share the same data context but apply different reasoning lenses. They operate in analysis mode only — separate from the operational agent layer.

**Why three distinct agents instead of one:**

Each agent represents a fundamentally different cognitive mode:
- **BI Agent:** explains what happened — backward-looking, pattern recognition.
- **Innovation Agent:** challenges assumptions — lateral, asks "what if this is a symptom?"
- **Opportunity Agent:** finds hidden signal — correlational, surfaces non-obvious connections.

A single analytical agent would produce responses that average these modes into something less useful than any one of them.

**The agent switching model:**

Users switch between analytical agents mid-conversation. At switch time, they choose:

- **Summary handoff** — new agent loads with a compressed summary of the prior conversation as starting context. User sees and can edit the summary before confirming.
- **Clean slate** — new agent loads with the data only, no conversation history.

**Why the summary must be editable:** A blind handoff where the user cannot see what was compressed is a trust problem. The Innovation Agent might reason from a summary that lost the key nuance. Showing and allowing edits before switching makes the handoff transparent and correctable.

---

### 4.5 Meta layer — the sim engine that exercises the UX

**Decision:** A dedicated meta agent — the **Warehouse Life Agent** — owns the simulated warehouse state for the working prototype. It generates root events on a scripted 08:00–09:30 timeline and emits derived events when state crosses declared thresholds. The WMS agents react to its events.

**Why a meta agent at all:** The prototype needs to feel alive — the user needs to see the system handle a real-feeling stream of events, not a static screenshot. The Warehouse Life Agent supplies the world. Without it, you couldn't test whether the UX of the agentic system actually works under operation.

**Why it's "backstage":** A Tier 1 prohibition. The Warehouse Life Agent **must not surface to the user**. No chat message, no map pin, no log entry is attributed to "Warehouse Life Agent." All content surfaced is attributed to the WMS agent that reacted. The user perceives a coherent operational world; they don't perceive the sim engine generating it.

**Where it lives:** Only in `moonshot prototype/moonshot-prototype.html`. Must NEVER be present in `moonshot presentation/moonshot-prototype.html` (the shipped iframe). The spec for the sim engine lives at `agents/meta/warehouse-life-agent.md`.

**Compression:** 1 wall-second = 18 sim-seconds. 90 sim-minutes (08:00–09:30) plays in 5 wall-minutes. Tick interval: 500ms wall, each tick advances 9 sim-seconds. "Restart shift" resets `WAREHOUSE.clock` to 08:00 and re-seeds state.

---

### 4.6 Guardian layer — process gate, not in the WMS topology

**Decision:** A set of **Guardian agents** audit the prototype itself — its usability, accessibility, and design quality — before each Warehouse-homepage deploy. They are NOT part of the in-fiction WMS topology; they are a process gate during prototype development.

**Why a process gate, not a runtime agent:** The Guardian agents don't run inside the WMS product. They run when the human designer is about to ship a change to the homepage. Their job is to surface usability and accessibility violations BEFORE the human approves ship. They produce a verdict report per criterion (PASS / WARN / FAIL / N/A) with a one-sentence rationale per finding. Any FAIL on a load-bearing criterion blocks ship until addressed.

**Spec'd agents (live as of May 11 morning):**

| Agent | Domain | Spec location |
|---|---|---|
| **Heuristics Agent** | Nielsen Norman 10 heuristics + Hick's Law + Miller's 7±2 + cognitive-load theory + F-pattern + banner blindness + information scent | `agents/guardian/heuristics-agent.md` |
| **Accessibility Agent** | WCAG 2.2 — AA load-bearing, AAA advisory. Includes the sim-clock criterion (2.2.1 Timing Adjustable) unique to this prototype. | `agents/guardian/accessibility-agent.md` |

**Cards present on the Agents page but no full specs yet (deferred):**

- Design System Agent — DS compliance, token adherence (design-token enforcement is currently governed by the `design-tokens` skill in `.claude/skills/` rather than a runtime agent spec).
- Content Agent — label clarity, microcopy quality, empty states.
- Interaction Agent — pattern consistency, gesture conflicts, state coverage.
- Responsive Agent — layout across breakpoints, density appropriateness.

**Joint reports:** Heuristics and Accessibility may produce a joint Guardian audit report (one document covering both audits). The two agents do not negotiate — they each contribute their findings and the report concatenates them with attribution.

**Constraint they share with all agents:** Guardian agents must not modify the homepage. They audit and emit verdicts only. The human approves ship; the Guardian's audit is a proposal to the human, not a self-certification.

---

## 5. Context Injection Layer

### 5.1 Multi-tenant isolation

**Decision:** One base agent brain, N isolated instances per tenant. Each instance is loaded with:

- Operation data (shift schedules, SKU catalog, layout, SLAs)
- Historical patterns (that warehouse's throughput, error rates)
- Permission scope (what that user role can see and do)
- Operation archetype (high-volume / B2B / 3PL)

**Why this matters:** Without strict isolation, an agent reasoning about Berlin DC could be contaminated with data from Lagos. In a WMS context, this produces confident, authoritative wrong answers — which is worse than no answer.

**The critical failure mode:** Silent context load failure. If the context injection fails quietly, the agent activates and reasons from an empty or partial context with full confidence. This must fail loudly — a health check at session start with an explicit error state in the UI.

### 5.2 Operation archetype field

**Decision:** The context injection layer includes an `operation_archetype` field: `high-volume | b2b | 3pl`.

**Why:** This single field changes which agents the Shift Intelligence Agent activates first when it detects an anomaly. In high-volume, the primary signals are pick rate and labor flex. In B2B, they are dock scheduling and order accuracy. In 3PL, they are cross-client SLA conflict and labor allocation across tenants. The domain agents exist in all three archetypes — what changes is their priority weighting.

---

## 6. Approval + Escalation Layer

### 6.1 Approval-first — Stage 2 as the starting point

**Decision:** Every agent-proposed action requires human approval before execution. No action auto-executes at launch.

**Why:** Building trust before building automation. Operators and supervisors need to develop confidence in agent recommendations before those recommendations execute without confirmation.

**The automation maturity ladder:**

| Stage | Mode | Human role |
|---|---|---|
| 1 · Inform | Agent surfaces data, proposes nothing | Decides everything |
| 2 · Approve ← current | Agent proposes, human confirms | Confirms or rejects |
| 3 · Act + notify | Agent acts within policy bounds, human can rollback | Monitors, can rollback |
| 4 · Full automation | Agent acts, exception-only alerts | Exception handling only |

**The architecture must not make Stage 4 impossible.** Even though Stage 2 is the starting point, the data model, audit log, and approval gate design must accommodate future promotion of specific action types to Stage 3 or 4 without architectural surgery.

---

### 6.2 The shared schema across every Tier 2 action

**Decision:** Every Tier 2 action emitted by any agent carries three fields, adapted from Allen Oleksak's demo:

| Field | Type | Purpose |
|---|---|---|
| `confidence` | 0–100 | Per-action calibration — how sure THIS proposal is, not a rolling per-agent stat. Allen's demo uses values like 92%, 88%, 76%. |
| `escalationReason` | string | The agent's own explanation of why a human is needed. Free text. Lives next to the decision. |
| `options[]` | optional array | When the agent declines to recommend, it surfaces alternatives. The UI shows "N options drafted · no recommendation chosen." |

**Why these three:**

- Confidence forces the agent to commit to a calibration on each proposal. Operators learn to read it the same way they'd read a confidence interval — *"this one says 92% so I trust it; this one says 42% so I want to look closer."*
- escalationReason makes the *why* visible at the moment of decision. Without it, escalation is a black box and the operator has to guess what tripped it.
- options[] honors uncertainty as a first-class state. When the agent isn't sure, it doesn't fake confidence — it puts the choice in the human's hands explicitly. The UI affordances change from Approve/Reject to Open/Defer/Why.

---

### 6.3 Action criticality classification

**Decision:** Every proposed action is tagged `urgent` or `standard` at generation time by the Lead Agent. This tag drives the entire escalation path.

**Why at generation time:** If the system waits until a supervisor is unavailable to decide whether an action is urgent, the escalation decision becomes reactive. Tagging at generation time means the escalation path is pre-determined and the UI can communicate it immediately.

**The criticality tag is always the first visual element** in every action card — before description, before impact, before buttons. This is a layout constraint, not a styling choice. Its position communicates urgency before the user reads anything.

---

### 6.4 Supervisor unavailability — four conditions, not one

**Decision:** "Supervisor unavailable" is not a binary state. The system handles four distinct conditions with different escalation paths:

| Condition | Path |
|---|---|
| Not logged in | Escalate immediately to next role up |
| No response (session active) | Push notification → 2-minute window → escalate |
| In approval (handling another item) | Queue behind current item → max 3 min → escalate |
| Shift boundary (shift ended) | Escalate to incoming supervisor |

**Why this matters:** Treating all four as the same "unavailable" state and escalating immediately creates noise — supervisors receive escalations for items their colleague was about to handle. The conditions require different response times and different escalation targets.

**The operator experience during escalation:** The operator has zero action buttons during escalation. The escalation panel is read-only — status, timeline, context. No false affordances. All agency belongs to the manager. This asymmetry must be visually obvious without a label explaining it.

**Three manager resolution paths:**

- Approve → action executes, WMS write triggered, both panels update
- Reject → action cancelled, operator notified
- Downgrade to standard → moves to queue, resurfaces at next supervisor login

---

## 7. Product Surfaces

### 7.1 CLI Chat Interface

**Owner:** Operator + Supervisor
**Purpose:** Act on shift in real time

**Two modes in one interface:**

**Operational mode** — shift is live, time pressure, exceptions need fast resolution. Agent: Shift Intelligence routed through Lead. Controls: quick-action chips (Shift status, Pick exceptions, Dock arrivals). Approval cards inline in the chat thread.

**Analysis mode** — reflective, data-driven, slower pace, agent switching expected. Agents: BI, Innovation, Opportunity. Controls: agent switcher dropdown below input bar. Handoff preview modal on agent switch.

**Why two modes in one interface, not two separate tools:** Operators and supervisors move between operational response and analytical reflection within the same shift. A separate tool for analysis creates context-switching cost and risks becoming an unused shadow tool.

**Mode selector location:** Bottom-left of the input zone — as close as possible to the point of action without interrupting the chat thread.

---

### 7.2 Action Log — Live Tray + Shift Log Page

**Owner:** Operator + Supervisor
**Purpose:** Visibility of what agents have done, are doing, and are waiting on

**Two entry points, one data model:**

**Live action tray (right drawer):** Real-time, current shift only. Shows pending items (act now), queued items, last 3 validated actions, last 2 auto-executions. Badge count on sidebar icon. Opens without leaving the current view.

**Shift log page:** Post-shift audit. Full history, filterable by state and zone, exportable (supervisor only). Actor attribution on validated rows.

**Why two entry points instead of one:** Real-time action and post-shift audit have opposing design requirements. Real-time needs to be minimal, fast, and interruptive. Audit needs to be dense, filterable, and navigable. One surface that tries to serve both produces a screen that does neither well.

**The accountability distinction is non-negotiable:** Validated (human-approved) and auto-executed actions must be visually distinct at every level — badge label, row background color, actor cell content, and icon. A supervisor reviewing the log must never confuse a human-validated action with an auto-executed one. The accountability owner is different. The recovery path is different.

---

### 7.3 Agent Performance Page

**Owner:** Warehouse manager + Operations manager only
**Purpose:** Evaluate agent health over time, tune agent behaviour

**Access:** Via session navigator "Agent performance →" link (manager role only) and sidebar icon (permission-gated).

**Why not accessible from the CLI:** The CLI is for acting. The Agent Performance Page is for evaluating. Mixing operational and evaluation surfaces creates cognitive overload for operators and risks managers tuning agents during a live shift when they should be monitoring.

**Two sections:**

**System health panel** — four metrics with threshold-driven colors:

- Context load latency (target < 5s)
- Silent context failures (target 0)
- Routing accuracy (target ≥ 90%)
- Audit completeness (target 100%)

**Agent cards grid** — one card per domain agent, scannable, all visible at once. Each card shows: actions 24H, approval rate with threshold status, autonomy %, escalation rate, rejection pattern indicator, and the Cautious → Bold slider with projected approvals-per-shift.

**The Cautious → Bold slider:** Controls how aggressively the agent proposes actions. Moving toward Bold lowers the confidence threshold — the agent surfaces more recommendations. Moving toward Cautious raises it — fewer recommendations, higher certainty.

The slider has a **projected approvals-per-shift warning** that updates in real time as the manager drags it. This makes the downstream cost of increased autonomy visible before committing — a Bold agent generates more pending approvals, which increases supervisor cognitive load.

**The rejection pattern indicator:** Not just rejection rate — whether rejections are randomly distributed (normal variance) or clustered around a specific action type or time window (tuning signal). A clustered pattern means the agent is confidently wrong in a specific scenario. The warehouse manager has domain knowledge to act on this signal directly.

**Shift Intelligence Agent card** — distinct format, separate section above the domain agent grid. Spans full width. Different metrics: anomalies detected, signal-to-noise ratio, detection latency, missed anomaly rate. Positioned first because its health directly affects all domain agents.

---

### 7.4 Session Navigator

**Owner:** All roles (role-scoped content)
**Purpose:** Shift session management — start new shift, access recent shifts, navigate to shift log

**Pattern:** Left-side drawer, overlays content, does not push it. Width: 260px from right edge of the sidebar.

**Mutual exclusion:** Session navigator and action tray cannot both be open simultaneously. One global `activeDrawer` state variable governs this.

**Role-scoped content:**

| Section | Operator | Supervisor | Manager |
|---|---|---|---|
| Start new shift | ✅ | ✅ | ✅ |
| Current shift | ✅ with pick rate | ✅ with approvals | ✅ |
| Recent shifts | 3 shifts, pick rate metric | 3 shifts, approvals + escalations | 5 shifts |
| Agent performance link | ❌ | ❌ | ✅ |
| View all shifts | ❌ | ❌ | ✅ |

**Start new shift requires confirmation.** A confirmation modal with a pending approvals warning must be shown before clearing any active session. Reason: clearing an active shift conversation during a live operation is a high-cost mistake. One confirmation step is worth the friction.

**Recent shift navigation target:** Shift log page filtered to the selected shift. Not conversation replay — that requires persistent storage per shift per tenant which is a significant backend commitment deferred to a later stage.

---

## 8. Key Metrics — Stage 2 Baseline

These are the questions the Agent Performance Page must be able to answer. They are also the acceptance criteria for the agent layer being production-ready.

| Metric | Target | What failure means |
|---|---|---|
| Approval rate | ≥ 80% | Below 60% = intent routing broken |
| Context load latency | < 5s | Above 10s = operators bypass the tool entirely |
| Silent context failures | 0 | Any non-zero value = critical defect |
| Agent turns to resolve exception | ≤ 2 | More = intent routing too shallow |
| Approved actions logged | 100% | Non-negotiable for audit and accountability |
| Role-correct routing | ≥ 90% | Operator vs supervisor getting appropriate response depth |
| Rejection pattern | Distributed | Clustered = agent tuning required |
| Escalation rate | < 3% | Above 5% = approval gate too slow for action frequency |

---

## 9. Risks — Build for Now, Not Later

| Risk | Severity | Mitigation |
|---|---|---|
| Silent context load failure | Critical | Explicit health check at session start. Fail loudly. Never activate agent on partial context. |
| Context bleed between tenants | Critical | Hard isolation at the context injection layer. Separate data scopes per tenant instance. |
| Supervisor absent — urgent action blocks operation | High | Four-condition escalation model. Urgent always escalates up hierarchy. Standard queues and resurfaces. |
| Agent reasoning from stale WMS data | Medium | Cache invalidation strategy at WMS integration layer. Agents read live where possible. |
| High autonomy + high approval rate masking clustered rejection | Medium | Rejection pattern indicator on agent cards. Combined signal (low approval rate AND clustered) triggers warning banner. |
| Sub-agent proliferation without routing updates | Low | New agents cannot be added without updating Lead Agent routing logic. Intent black holes otherwise. |
| Supervisor cognitive overload from Bold agent settings | Low | Projected approvals-per-shift warning on Cautious → Bold slider. Cost is visible before commitment. |
| Sim engine leakage into shipped product | Medium | Warehouse Life Agent is a Tier 1 prohibition outside `moonshot prototype/`. Verified at every prototype-to-presentation promotion. |
| Guardian audits skipped under deadline pressure | Medium | Both Guardian agents run before every Warehouse-homepage deploy. Load-bearing FAIL blocks ship. Verified manually until automated. |

---

## 10. Prototype Build Plan

Original seven-prompt build sequence (May 11 morning, Sessions 1–4):

| Prompt | Builds | Status |
|---|---|---|
| Prompt 1 | Dual-mode CLI shell — operational + analysis, approval card, handoff modal | ✅ Complete |
| Prompt 2 | Action log — live tray drawer + shift log page | ✅ Complete |
| Prompt 3 | Escalation flow — four conditions, split view, operator + manager views | ✅ Complete (superseded by Prompt 5) |
| Prompt 4 | Full CLI extension — escalation layered onto Prompt 1 | ✅ Complete (superseded by Prompt 5) |
| Prompt 5 | Unified prototype — consolidates Prompts 1–4 with corrections | ✅ Current base |
| Prompt 6 | Session navigator drawer — additive layer on Prompt 5 | ✅ Complete |
| Prompt 7 | Agent performance page — additive layer on Prompts 5+6 | ✅ Complete |

**Subsequent expansion (May 11 afternoon + May 12 morning):**

- **Session 5 (May 11 PM)** — flicker fixes; KPI trend indicators with semantic polarity; CLI terminal theme unification; brand-new Warehouse dashboard.
- **May 12 morning, Tasks 1–4** — dead-code cleanup; prototype promoted to the shipped folder (renamed `moonshot-prototype.html`); six new agent cards added (Slotting / Equipment / Quality / Warehouse Life / Heuristics-upgraded / Accessibility-upgraded); attribution conventions established; structural reorder of The Agents page; topology section rewritten; Architecture diagram rebuilt; "How the agents work" page built then merged into The Agents page as a Timeline · System toggle; pull-quote relocated to home page.

**Corrections applied across the build sequence:**

- Approve button text → white `#FFFFFF` (was yellow `#CADF35` — incorrect)
- Agent switcher → `<select>` dropdown (was pills — reverted for scalability)
- Mode selector → bottom-left of input zone (was in subnav — moved to match built output)
- Exception Agent scope → transactional errors only (was bundled with damage / holds / cycle counts; carved out to the new Quality Agent on May 11)

---

## 11. Open Threads

These decisions were raised but deferred. They are ready to be picked up.

| Thread | What was decided | What is still open |
|---|---|---|
| B2B agent roster | Dock and Slotting become primary; Quality and Equipment carry over from high-volume | Roster not fully defined — deferred until high-volume is validated |
| 3PL agent roster | Labor and Order Priority serve cross-client SLA conflict | Roster not defined — hybrid operation scenario unresolved |
| Conversation replay | Deferred — shift log only for now | Requires persistent storage per shift per tenant |
| Stage 3 automation | Architecture supports it — not activated | Rule set for which action types can auto-execute not defined |
| Rejection pattern indicator visual | Agreed it is needed | Exact visual design not specified in prompts |
| Supervisor response window | Recommended 2 minutes | Not confirmed — requires operational validation |
| Availability detection mechanism | Recommended session heartbeat + duty roster | Not confirmed — depends on WMS session model |
| Design System Guardian agent | Cards exist on Agents page; token enforcement currently sits in the `design-tokens` skill | Full agent spec at `agents/guardian/design-system-agent.md` deferred |
| Content / Interaction / Responsive Guardian agents | Cards exist on Agents page; cards currently use "Authored by the UX Team" attribution as a placeholder | Full specs deferred — same pattern as Design System |
| Horizon layer specs | Ten cards on the Agents page; no operational specs in `/agents/` | Spec them with the same 8-section structure when the layer becomes active |

---

## 12. Design System Reference

All surfaces use the Infios Design System. Tokens are governed by the **`design-tokens` skill** at `.claude/skills/design-tokens.md` (auto-triggered on token deliveries or token-integration requests). The skill extracts the latest delivery, transcribes tokens into a dated reference at `archive/tokens-<YYYY-MM-DD>/TOKENS.md`, reconciles them with the existing `:root` blocks in `moonshot presentation/moonshot.html` and `moonshot prototype/moonshot-prototype.html`, and sweeps both files for hardcoded values to replace with `var(--token-name)`.

**Brand-yellow `#CADF35` is load-bearing** — the skill will stop and flag the discrepancy to the user rather than overwriting it if a new token spec proposes a change.

**Current key tokens** (as of May 12 morning — definitive values live in the `:root` blocks of `moonshot.html` and `moonshot-prototype.html`):

```
Shell:    header #262523 · sidebar #324154 · accent #CADF35
Brand:    fill #171F29 · text #171F29
Surface:  bg #FFFFFF · alt #F4F4F4 · canvas #F4F4F4
Text:     primary #171F29 · secondary #324155 · tertiary #58636E · quaternary #98A4B0
Link:     #5436CC
Border:   hairline #E5E8EC · subtle #D5DCE2 · strong #98A4B0
Success:  text #2C812C · bg #E6FFE6
Warning:  text #B64C00 · bg #FFFBE6
Error:    text #D0413A · bg #FFF0F0
Info:     text #2B7AB1 · bg #E6F7FF
Radius:   sm 8px · md 12px · lg 16px · xl 20px · pill 9999px
Shadow:   xs / sm / md / lg / xl tiers (see :root for values)
          legacy "tile" shadow: 0 2px 30px 0 rgba(0,0,0,0.15)
Font:     display 'Noto Sans' · text 'Noto Sans' · mono ui-monospace
Grid:     8pt (4 · 8 · 12 · 16 · 20 · 24 · 32px)
```

When a new token delivery lands, this section is updated by the skill in lockstep with the `:root` blocks. The canonical values are the ones in the HTML — this section is human-readable mirror.
