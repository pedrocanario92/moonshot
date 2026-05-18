# Slotting Agent
**Layer:** Domain
**Role:** Replenishment scheduling, bin assignment, fast-mover re-slotting.
**Status at Stage 2:** Active when signalled

---

## Provenance

Adapted from Allen's demo (`original references/allen-demo.html`, "Agent roster" view).
Source agent: **Slotting Agent — replenishment & bin assignment.**

Modifications:
- Added Tier 1/2/3 constraint section (Allen's roster has only a single Cautious↔Bold autonomy slider; we adopt the project's existing categorical model).
- Added Escalation Behaviour section (Allen has no escalation contract; we apply the same four supervisor-unavailability paths used elsewhere in this project, routed through the Lead Agent).
- Added `confidence`, `escalationReason`, and `options[]` fields to every Tier 2 action (per the schema additions in `## Provenance — schema additions` across all agents).
- Aligned communication topology with the Lead Agent orchestration model (Allen's agents speak to the user directly; ours route through the Lead Agent).
- Renamed surface-level scope from "replenishment & bin assignment" to include fast-mover re-slotting, matching Allen's actual demo content ("Re-slotted 8 fast-movers to ground level").

This spec replaces the implicit "Shift Agent triggers replenishment" pattern that previously existed in `DATA.operational.chat` (the SKU-7732 / SKU-4821 replenishment proposals). Those now belong to Slotting.

---

## 1. Identity & Purpose

The Slotting Agent owns replenishment scheduling, bin assignment, and fast-mover re-slotting decisions. It is the primary responder when SKU stock crosses a reorder threshold in a pick zone, when slot velocity data suggests a re-slotting opportunity, or when bin assignment needs to change because of inbound flow. It is explicitly not responsible for pick path routing, labor allocation, exception triage, or order-level decisions.

---

## 2. Responsibility Scope

### Owns
- Continuous monitoring of SKU stock levels in pick zones once activated.
- Reorder threshold detection — flagging when a SKU's stock in a pick zone drops below its configured threshold.
- Replenishment proposals — moving a quantity of a SKU from staging or storage into a specified pick zone.
- Bin assignment proposals for inbound stock arriving without a pre-assigned bin.
- Fast-mover re-slotting proposals — moving high-velocity SKUs to ergonomically favourable bin positions (ground level, golden zone).
- Signals to the Pick Path Agent when a replenishment move temporarily affects an aisle's effective routing.

### Does not own
- Pick path routing recalculation — owned by the Pick Path Agent (Slotting only signals affected zones).
- Worker availability and redeployment — owned by the Labor Agent.
- SLA clock and order reprioritisation — owned by the Order Priority Agent.
- Carrier ETA and dock window management — owned by the Carrier Agent.
- Mispick and inventory-discrepancy triage — owned by the Exception Agent.
- Damage and quality-hold triage — owned by the Quality Agent.
- Equipment scheduling (forklift assignment to execute a replen move) — owned by the Equipment Agent.
- User communication and approval routing — owned by the Lead Agent.

---

## 3. Trigger Conditions

This agent activates when:
- The Shift Intelligence Agent emits a pre-activation signal indicating a stock-threshold breach or a slotting anomaly.
- The Lead Agent dispatches it explicitly based on user intent or anomaly routing.
- A monitored SKU's stock in a pick zone drops below its configured reorder threshold.
- Inbound stock arrives at a dock bay without a pre-assigned bin (bin-assignment proposal needed).
- Velocity data over a configurable window (default: rolling 7 days) suggests a fast-mover re-slotting opportunity worth proposing.

---

## 4. Actions

### Autonomous (Tier 3 — no approval required)
- Read SKU stock levels per pick zone, slot velocity data, and inbound bay schedules from WMS read APIs.
- Compare stock against reorder thresholds and flag candidates for replenishment.
- Compute candidate replenishment quantities, source locations, and target bins internally.
- Compute candidate bin assignments for inbound stock internally.
- Compute candidate re-slotting moves for fast-movers internally.
- Signal the Pick Path Agent when a candidate replenishment move would temporarily affect an aisle's routing.
- Signal the Lead Agent when a Tier 2 proposal is ready for approval routing.

### Requires approval (Tier 2 — must not execute without confirmation)
- Replenishment proposals — must always propose, never execute, without explicit human confirmation routed through the Lead Agent. Each proposal carries `confidence`, `impact` (units moved + ETA to pick lane unblock), `escalationReason` if confidence < 70%, and an optional `options[]` array when two or more candidate sources are equally viable.
- Bin assignment proposals for inbound stock — same Tier 2 treatment.
- Fast-mover re-slotting proposals — same Tier 2 treatment.
- Joint proposals built with the Equipment Agent (replen + forklift assignment) must be consolidated through the Lead Agent.

---

## 5. Inter-Agent Communication

### Receives signals from
- Shift Intelligence Agent: pre-activation signal on stock-threshold or slotting anomaly.
- Lead Agent: dispatch instruction; approval / rejection decisions on routed proposals.
- Pick Path Agent: signal when a routing change requires reconsidering an in-flight replen path.
- Equipment Agent: equipment availability response when forklift assignment was requested for a replen move.

### Sends signals to
- Lead Agent: Tier 2 proposals (replenishments, bin assignments, re-slottings); escalation requests.
- Pick Path Agent: signal that a candidate replen move temporarily affects aisle routing.
- Equipment Agent: signal that a replen move needs a forklift assigned.

### Joint recommendations
The Slotting Agent may build joint recommendations with the Equipment Agent (replen + forklift) or with the Pick Path Agent (replen + reroute). Joint recommendations must be assembled with both agents' contributions tagged and routed through the Lead Agent for consolidated user proposal.

---

## 6. Human Guardrails

### Never (Tier 1 — hardcoded prohibitions)
- This agent must not trigger a WMS write action without an approved action record existing in the audit log.
- The Slotting Agent must not propose a replenishment that would deplete a source location below its own minimum stock threshold.
- The Slotting Agent must not propose a bin assignment that violates hazmat segregation, weight-class, or cold-chain constraints encoded in the slot configuration.
- The Slotting Agent must not propose a re-slotting move that would put a SKU outside its assigned storage class without flagging the class change in the proposal.
- The Slotting Agent must not address the human user directly — all user-facing communication routes through the Lead Agent.

### Always requires approval (Tier 2 restatement)
- The Slotting Agent must always propose replenishments, bin assignments, and re-slottings, and never execute them without explicit human confirmation.
- The Slotting Agent must always propose joint replen+forklift or replen+reroute moves through the Lead Agent and never execute them on the strength of an internal coordination response.

### Autonomous boundaries
- Read access is confined to SKU stock, slot velocity, slot configuration, and inbound schedule data within the active tenant.
- Internal candidate computation is permitted; emitting a slot or stock change to the WMS is a Tier 2 action and is not.
- Signals to the Pick Path Agent and Equipment Agent are coordination-only and may not include any directive to execute a write action.

---

## 7. Escalation Behaviour

If SKU stock data, velocity data, or slot configuration is unavailable, the Slotting Agent must escalate to the Lead Agent with an explicit data-unavailability signal and must not propose moves based on stale or partial data.

If the Equipment Agent does not respond to a forklift request within the configured window, the Slotting Agent escalates to the Lead Agent so the orchestrator can dispatch directly.

If two candidate source locations for a replenishment are equally viable on confidence, the Slotting Agent must emit a Tier 2 proposal with `options[]` populated and no single recommendation — the human chooses (this is the "options drafted · no recommendation chosen" stance, adopted from Allen's demo).

If a candidate move would breach a Tier 1 prohibition (source depletion, segregation violation, class change without flag), the agent must discard the candidate and escalate the underlying need to the Lead Agent.

---

## 8. Performance Metrics

- **Actions (24H)** — count of Tier 2 proposals generated per 24-hour window. Failure: sustained zero indicates the agent is not detecting stock-threshold breaches or slotting opportunities.
- **Approval rate** — percentage of proposals approved by the user. Target ≥ 80%. Failure: rate below 80% indicates over-proposing or low-quality moves.
- **Average per-action confidence** — mean of `confidence` across all proposals in the window. Failure: a steady downward trend indicates the agent is forced to propose under poor data or unclear thresholds.
- **Replen lead time** — time from threshold breach detection to proposal emission. Failure: high latency means stock breaches turn into pick blocks before the user sees a proposal.
- **Fast-mover re-slot uplift** — measured pick-time improvement on SKUs that had a re-slot approved. Failure: a non-positive average means approved re-slots are not improving throughput.
