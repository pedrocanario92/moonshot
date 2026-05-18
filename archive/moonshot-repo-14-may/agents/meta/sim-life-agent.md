# Sim Life Agent
**Layer:** Meta (backstage)
**Role:** Scripted-causal simulation engine for the cross-pillar demo — generates events across Warehouse (WA), Transportation (TM), and Order Management (OM) with correlation logic for cross-pillar cascades.
**Status at Stage 2:** Always active during demo / UX test runs; absent in shipped product.

> **This is a meta agent.** It does not belong to the WMS / TMS / OMS topologies. It does not propose, escalate, or address the user. It is backstage — it *is* the simulated operation across three pillars. Its only purpose is to make the prototype's UI come alive across WA, TM, OM, and the Watchtower so the UX can be exercised under operation.

---

## Provenance

Original to this project. Began life under a prior name as a single-pillar (WA only) sim engine, modelled on the gap left by Allen's static React snapshot (`original references/allen-demo.html`). Renamed and broadened on May 13 2026 when the demo's scope expanded to Warehouse + Transportation + Order Management + Watchtower. The original spec is preserved in `archive/` for lineage (see the file dated PRE-RENAME).

---

## 1. Identity & Purpose

The Sim Life Agent owns the state of one simulated distribution center, one transportation hub, and one OMS region across one shift, compressed from 90 sim-minutes (08:00–09:30) into ≈3 wall-minutes of demo time at 30:1 compression. It emits root events on a scripted timeline for **each of the three pillars** plus a **cross-pillar correlation cascade** with a **branching decision point** (Director of Operations chooses Path 1 "Wait" or Path 2 "Deploy backup + re-allocate"). Both resolution waves are pre-scripted; the prototype renders whichever path the Director clicks.

It produces:
- A per-pillar stream of root events (WA, TM, OM), most of which stay inside their pillar.
- One primary cross-pillar cascade (Carrier #4471 MEM→CHI delay → OM ATP re-evaluation → human escalation → either Path 1 or Path 2 resolution).
- A handful of isolated single-pillar events that visibly do **not** cascade, so the system reads as realistic rather than magical.

It is explicitly not responsible for proposing user-facing actions, formatting UI, gating approvals, computing performance metrics, or anything that the in-fiction agents own.

---

## 2. Responsibility Scope

### Owns
- The `WAREHOUSE` state object: clock, zones, agent runtime state, inbound/outbound dock schedule, incidents, chat thread, action log.
- The `TM_DATA` state object: lanes, kpi, scenarios, action log, chat thread.
- The `OM_DATA` state object: channels, kpi, scenarios, action log, chat thread.
- The `CROSS_DATA` state object: cross-pillar chat thread and cascade state (`pending` | `awaiting-decision` | `path-1` | `path-2` | `resolved`).
- The scripted root event timelines (08:00–09:30 sim-time) for WA, TM, OM, and the cross-pillar cascade.
- The branching decision point: both Path 1 and Path 2 resolution waves are scripted; the agent fires whichever path the prototype's `directorChoice` selector points to.
- The causal rule set that emits derived events when state thresholds are crossed.
- The tick loop that advances the sim clock, fires scheduled events, evaluates rules, and triggers re-render.
- The compression ratio between sim-time and wall-time (30:1 by default).
- The initial seed state — the "typical morning" the user lands in across all three pillars.

### Does not own
- Any user-facing action proposal — owned by the relevant in-fiction agent (WA: Pick Path, Labor, Order Priority, Carrier, Exception, Quality, Slotting, Equipment; TM: Carrier, Spot Rate, Route; OM: Order Priority, Allocation, Cutoff Manager).
- Approval gating — owned by the Archer Chat Agent.
- Audit log entry creation for approved actions — owned by the Archer Chat Agent.
- Render functions or DOM construction — owned by the prototype's UI layer.
- Performance metric calculation — owned by `DATA.agentPerformance`.

---

## 3. Trigger Conditions

This agent activates when:
- The prototype loads (the tick loop begins at page-load).
- The user clicks "Restart shift" (the tick loop resets to the seed state and replays from 08:00).

It does not activate in response to user actions in any other way, with one exception: the **Director's choice of Path 1 or Path 2** on the cross-pillar decision card sets `directorChoice`, which gates which path's pre-scripted resolution events the agent emits. The agent does not branch the underlying scripted timeline — both paths are authored ahead of time; only the gating flag changes.

---

## 4. Actions

### Autonomous (Tier 3 — no approval required, no human-facing surface)
- Advance the sim clock on each tick.
- Fire scheduled root events from the WA / TM / OM / Cross timelines at their declared `atSec` time.
- Evaluate every causal rule against current state on each tick.
- Emit derived events when causal rules fire.
- Mutate `WAREHOUSE`, `TM_DATA`, `OM_DATA`, and `CROSS_DATA` state in response to event effects.
- Call the prototype's `renderAll()` after state changes that affect the currently visible view.

### Requires approval (Tier 2)
- None. This agent never proposes actions to the user. All user-facing proposals come from the in-fiction agents that react to its events.

---

## 5. Inter-Agent Communication

### Receives signals from
- None during normal operation. The sim is not reactive to other agents.
- The user via "Restart shift" — resets state and replays from seed.
- The user via the cross-pillar decision card — flips `directorChoice` between Path 1 and Path 2; the agent then gates which scripted resolution wave fires.

### Sends signals to
- The in-fiction agents implicitly, via state changes. When per-pillar state crosses a threshold (e.g., inbound slip > 15 min, ATP shortfall, spot quote spike), the corresponding causal rule causes the in-fiction agent to emit a Tier 2 proposal into the relevant chat thread / action log.

### Joint recommendations
Not applicable. This agent does not participate in user-facing recommendations.

---

## 6. Human Guardrails

### Never (Tier 1 — hardcoded prohibitions)
- This agent must not surface itself to the user. No UI element, no chat message, no map pin, no log entry may attribute content to "Sim Life Agent." All content surfaced to the user is attributed to an in-fiction agent.
- This agent must not be present in any shipped homepage or dashboard variant of the deliverable. It exists only in `moonshot-prototype.html`. Any future shipped surface (WA / TM / OM / Watchtower included) must remain free of sim engine code.
- This agent must not modify content authored by in-fiction agents (chat text, proposal descriptions, escalation reasons). It only triggers their emission via causal rules.

### Always requires approval (Tier 2 restatement)
Not applicable.

### Autonomous boundaries
- The agent's reach is confined to mutating the `WAREHOUSE` / `TM_DATA` / `OM_DATA` / `CROSS_DATA` state objects and calling `renderAll()`. No network calls, no persistence, no off-page side effects.
- The tick loop must be cancellable (a single `clearInterval` reference) so the prototype can pause for testing.

---

## 7. Escalation Behaviour

If any state object becomes internally inconsistent (e.g., an event fires that references a zone or lane not in the layout), the agent must log a console warning and skip the event rather than mutate state into a contradiction. There is no human user to escalate to — escalation here means visible failure mode for the developer running the prototype.

If the tick loop has not advanced for more than 2 wall-seconds (a `setInterval` stall), no recovery is attempted — the prototype will appear frozen and a reload is the expected remediation.

---

## 8. Performance Metrics

Not applicable in the usual sense — this agent is not measured against approval rate, escalation rate, or any user-facing metric.

**Internal developer-facing metrics** (for tuning the sim, not for the user):
- **Event density** — events fired per wall-minute. Target: ≤ 3 user-visible events per wall-minute (raised from the WA-only target of 2, to accommodate cross-pillar events). Failure: a higher rate means dashboards will read as Allen-style noise.
- **Calm-state coverage** — percentage of wall-time during which the visible pillar's "calm" state is shown. Target: ≥ 50% of the 3-min run *per pillar*. Failure: a lower rate means the user never sees the "nothing demands you" state — defeating the project's UX argument.
- **Event-to-spec traceability** — every event kind emitted must trace to a Tier 2 action or trigger condition in an in-fiction agent spec. Target: 100%.
- **Cross-pillar discipline** — at most one primary cascade per demo window. Isolated single-pillar events must outnumber cascade events.

---

## Root event scripts (authoritative)

Each pillar's timeline is mirrored as a JS const in `moonshot-prototype.html` for traceability. Each entry has the shape:

```
{ atSec: <sim-seconds-since-midnight>, kind: '<event-kind>', payload: { ... } }
```

Initial seed at 08:00:
- **WA**: 94 active pickers across 8 zones; 1,284 orders in flight; 0 at SLA risk; inbound bays scheduled (08:30, 09:00, 09:15, 09:45).
- **TM**: 847 active loads; 94% on-time; 3 open alerts on the seed action log.
- **OM**: 2,341 active orders; 96.2% fill rate; 2 pending decisions on the seed action log.

### WA (warehouse) events

| Sim time | Event | In-fiction agent | Outcome |
|---|---|---|---|
| 08:20 | Bin reassignment auto-correct — SKU-4821 ground-level | Slotting Agent | Autonomous log only |
| 08:30 | Pick rate dip in Pick Zone C — reassign picker Ana Ramirez | Labor Agent | Autonomous log only |
| 08:30 | Inbound DHL-882 at Bay 4 delayed 22 min → replen at risk for SKU-7732 | Pick Path Agent | Pending decision on WA home stage |
| 08:35 | Pick path autonomous reroute around D-4 congestion | Pick Path Agent | Autonomous log only |
| 08:50 | Conveyor C-04 unresponsive — pack-4 halted; root-cause confidence 42% | Equipment Agent | Escalation card with 3 options, no recommendation |
| 09:10 | Charge cycle complete — FL-12 returned to floor | Equipment Agent | Autonomous log only |
| 09:20 | Wave 8842 SLA pressure — resequenced waves 8843 before 8842 | Order Priority Agent | Autonomous log only |

Causal rules referenced: R1 (`inbound.slip_min > 15` → replen at risk), R4 (`equipment.rootCauseConfidence < 50` → options[] escalation). Originals in the PRE-RENAME archive snapshot.

### TM (transportation) events

| Sim time | Event | In-fiction agent | Outcome |
|---|---|---|---|
| 08:18 | Carrier ETA update — #2841 LAX→SEA on track, ETA 11:30 | Carrier Agent | Autonomous log only (isolated single-pillar) |
| 08:22 | Spot quote arrival — MEM→DEN at $4,200 (2.3× lane average) | Spot Rate Agent | Pending escalation card on TM home; flagged for Director |
| 08:35 | Carrier check-in — #5912 ORD→ATL departed on schedule | Carrier Agent | Autonomous log only (isolated single-pillar) |

### OM (order management) events

| Sim time | Event | In-fiction agent | Outcome |
|---|---|---|---|
| 08:31 | Order intake spike — 12 new orders SKU #WH-4419 | Order Priority Agent | Stock-out risk surfaces on OM home (pending decision) |
| 08:34 | Allocation request — 200 units SKU-1188 from DC-14 | Allocation Agent | Autonomous log only (isolated single-pillar) |
| 08:55 | Cutoff signal — 17:00 EOD approaching for Tier 1 accounts | Cutoff Manager Agent | Reminder message in OM Archer terminal |

### Cross-pillar cascade — Carrier #4471 MEM→CHI delay

This is the one primary cross-pillar cascade in the demo window. It runs from 08:43–09:10 (Path 2) or 08:43–11:15 (Path 1 — extends past sim end).

**Trigger (always fires):**

| Sim time | Event | In-fiction agent |
|---|---|---|
| 08:43 | Carrier #4471 MEM→CHI 2h late (arrival 11:00 vs 09:00) | TM Carrier Agent |

**OM pickup (always fires):**

| Sim time | Event | In-fiction agent |
|---|---|---|
| 08:44 | "12 orders depending on 09:00 inbound; re-evaluate available-to-promise" | OM Allocation Agent |
| 08:45 | "Re-rank pending order queue using new ATP" | OM Order Priority Agent |

**Decision point (always fires):**

| Sim time | Event | In-fiction agent |
|---|---|---|
| 08:47 | Escalate to Director of Operations with two paths | Archer Chat Agent |

The Director chooses between:
- **Path 1 — Wait.** Accept 2h slip. No extra spend. 12 orders ship late.
- **Path 2 — Deploy backup + re-allocate.** Engage Saia backup carrier ($1,800 spot). Re-allocate 4 SKUs from DC-19. All 12 ship on time.

**Path 2 resolution wave** (default if the Director makes no manual choice):

| Sim time | Event | In-fiction agent |
|---|---|---|
| 08:49 | "Saia backup engaged for parallel load on MEM→CHI" | TM Carrier Agent |
| 08:51 | "Re-allocate 4 SKUs from DC-19; update ATP" | OM Allocation Agent |
| 08:53 | "Reassign 4 receivers to outbound during 2h window" | WA Labor Agent |
| 08:54 | "Re-prioritise Zone B pick paths" | WA Slotting Agent |
| 09:10 | Resolution: TM "Saia delivery on track" · OM "10/12 within SLA" · WA "outbound throughput within target" · Watchtower amber → green | All three pillars |

**Path 1 resolution wave** (fires if the Director chooses Wait):

| Sim time | Event | In-fiction agent |
|---|---|---|
| 08:50 | "Hold inbound receiving; redirect receivers to outbound until 11:00" | WA Labor Agent |
| 08:53 | "Re-prioritise Zone B for outbound focus during 2h window" | WA Slotting Agent |
| 11:00 | "Carrier #4471 arrived; inbound receiving resumes" | TM Carrier Agent |
| 11:15 | Resolution: TM "delivery complete" · OM "12 orders shipped late" · WA "throughput resumed" · Watchtower amber → green | All three pillars |

**Note:** Path 1's 11:00 and 11:15 events extend past the 09:30 sim end. They are pre-scripted in the event table but only land in the action log / shift log scrub — not on a live home stage — since the tick loop stops at 09:30. The Director sees the cascade resolution via the shift log timeline rather than via a live amber-to-green transition during the demo window.

---

## Correlation logic

Most events stay in their pillar. The Sim Life Agent fires cross-pillar correlations **when they should, not when they could**. The table below names every scripted event and tags it `isolated` or `cascades`:

| Sim time | Event | Tag | Why |
|---|---|---|---|
| 08:18 | TM ETA update #2841 LAX→SEA on track | isolated | On-track ETAs don't propagate; no WA/OM dependency. |
| 08:20 | WA bin auto-correct SKU-4821 | isolated | Routine slotting; no downstream effect. |
| 08:22 | TM spot quote MEM→DEN $4,200 | isolated | Spot price decisions are TM-local until accepted; reject keeps it inside TM. |
| 08:30 | WA pick rate dip Zone C → Labor reassign | isolated | Cross-zone reassignment within WA. |
| 08:30 | WA inbound DHL-882 delay → replen at risk | isolated | Replen affects a single WA wave; no TM/OM signal. |
| 08:31 | OM order intake spike SKU #WH-4419 | isolated | Stock-out risk is OM-local until a re-allocation is approved. |
| 08:34 | OM allocation SKU-1188 from DC-14 | isolated | Routine ATP draw; no cross-pillar contention. |
| 08:35 | WA pick path reroute D-4 → D-6 | isolated | Floor-internal routing. |
| 08:35 | TM check-in #5912 ORD→ATL | isolated | On-schedule departure; no anomaly. |
| 08:43 | **TM Carrier #4471 MEM→CHI 2h late** | **cascades** | **Inbound block affects 12 OM orders' ATP → OM re-evaluates → human decision → resolution touches WA labor + slotting.** |
| 08:50 | WA Equipment C-04 unresponsive | isolated | Equipment fault affects pack-4 within WA; doesn't propagate to TM/OM in this demo window. |
| 08:55 | OM cutoff 17:00 reminder | isolated | Operational reminder; no cross-pillar effect. |

### Worked example — the 08:43 cascade

Step by step, what fires and why:

1. **TM Carrier Agent** detects #4471 will arrive 2h late. Emits delay event into `TM_DATA.actionLog` and `TM_DATA.chatThread`.
2. The state mutation also writes into `CROSS_DATA.chatThread` because the delay affects an inbound that 12 OM orders depend on — this is the correlation rule firing.
3. **OM Allocation Agent** reads the delayed inbound and re-evaluates available-to-promise. Pushes a message into `OM_DATA.chatThread` and `CROSS_DATA.chatThread`.
4. **OM Order Priority Agent** re-ranks the pending queue using the new ATP. Pushes a message.
5. **Archer Chat Agent** has a Tier-2 escalation rule: "cross-pillar impact + ≥10 orders affected + decision involves spend" → escalate to Director. Posts the two-path decision card.
6. Director clicks Path 1 or Path 2. `directorChoice` updates.
7. The Sim Life Agent's tick loop only fires `CROSS_EVENTS` with `path === directorChoice` (or `path === null`). The chosen wave plays.

### Closing principle

> Most events stay in their pillar. Cross-pillar correlations fire when they should, not when they could. The demo shows one primary cascade alongside multiple isolated events, so reviewers see the system handle real correlations without conflating "agentic" with "everything cascades."

---

## Compression

- 1 wall-second = 30 sim-seconds.
- 90 sim-minutes (08:00–09:30) plays in 3 wall-minutes.
- Tick interval: 500ms wall, so each tick advances 15 sim-sec.
- "Restart shift" resets all four state objects to seed, clears `_fired` flags on every event table, and resets `directorChoice` to 2.

---

## What it produces

Across a 3-minute demo run, the Sim Life Agent produces:

- A WA event stream of ~7 root events driving the floor map, shift log, and home stage.
- A TM event stream of ~3 root events driving the lane strip and TM home stage.
- An OM event stream of ~3 root events driving the channel strip and OM home stage.
- A cross-pillar cascade beginning 08:43 with a branching decision point at 08:47 and a resolution wave at 08:49–09:10 (Path 2) or 08:50–11:15 (Path 1).
- A unified Archer terminal log that, in cross-pillar context, shows every pillar's messages in chronological order, and in WA/TM/OM context shows the filtered per-pillar stream.

The agent itself is invisible. The viewer sees three pillars and a Watchtower that look alive; the Director sees a real cross-pillar decision land in front of them; nobody attributes any of it to "Sim Life Agent" because nothing is attributed to it.
