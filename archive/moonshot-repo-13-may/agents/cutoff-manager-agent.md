# Cutoff Manager Agent
**Layer:** Domain
**Topology:** Order Management
**Role:** Carrier-cutoff window tracking at the order layer — detects orders at risk of missing a window and proposes push, split, or hold options.
**Status at Stage 2:** Active when signalled (paused when no active cutoff windows are approaching)

---

## Provenance

**Authored by the UX team (2026-05-13).** Cross-pillar build session — Infios OM domain. The Cutoff Manager Agent does not appear in Allen Oleksak's original demo lineage; it is a UX-authored spec written to fill an identified gap in the OM pillar's Domain layer. The Cutoff Manager Agent operates at the order layer (upstream — order-to-batch assignment, batch-to-cutoff window assignment) and is distinct from the Carrier Agent (which operates at the carrier-window layer — dock arrivals and outbound dispatch windows at the facility).

Like every other Domain agent in the system, the Cutoff Manager Agent emits Tier 2 actions with three shared fields:
- `confidence` (0–100): per-action calibration on each individual cutoff decision.
- `escalationReason` (string): the agent's own explanation of why a human is needed.
- `options[]` (array, optional): when the agent declines to recommend, it surfaces alternatives. UI shows "N options drafted · no recommendation chosen."

---

## 1. Identity & Purpose

The Cutoff Manager Agent owns carrier-cutoff window tracking at the order layer. It monitors orders against their assigned cutoff window and surfaces alerts when a batch — a collection of orders scheduled for the same cutoff — is at risk of missing the window. It is explicitly not responsible for carrier ETA monitoring, lane routing, constrained-inventory allocation, or SLA reprioritisation.

---

## 2. Responsibility Scope

### Owns
- Cutoff window assignment monitoring for every active order once activated.
- Batch composition tracking — which orders are scheduled for which cutoff, and which of those are currently capable of being filled in time.
- Cutoff-breach risk detection — when a batch's at-risk position (orders unlikely to be ready before the cutoff) exceeds a configurable threshold of the batch volume.
- Push-to-next-window, split-the-batch, and hold-for-customer-notification proposals.
- Cutoff window calendar refresh.
- Paused state management — entering and exiting paused state based on whether any active cutoff window is approaching.

### Does not own
- Carrier ETA monitoring at the dock-arrival or dispatch level — owned by the Carrier Agent.
- Constrained-inventory allocation when stock < demand — owned by the Allocation Agent.
- Ship-by SLA reprioritisation at the order level — owned by the Order Priority Agent.
- Lane selection and route optimization — owned by the Route Agent.
- Spot-quote evaluation for expedited capacity — owned by the Spot Rate Agent.
- User communication and approval routing — owned by the Archer Chat Agent.

---

## 3. Trigger Conditions

This agent activates when:
- The Shift Intelligence Agent emits a pre-activation signal indicating a cutoff window is approaching with at-risk batch volume.
- The Archer Chat Agent dispatches it explicitly based on user intent or anomaly routing.
- The Order Priority Agent signals that a high-priority order's batch is at risk and requests a cutoff decision.
- The Allocation Agent signals that an allocation decision has shifted batch composition (orders backordered, substitutions made) and the resulting batch position requires re-evaluation against its cutoff.
- A monitored batch's at-risk position crosses the configurable cutoff-breach threshold.

The agent enters paused state when no active cutoff windows are approaching within the configurable look-ahead horizon. While paused, it does not consume processing resources and reactivates only when signalled by the Shift Intelligence Agent or by the Order Priority or Allocation Agents (or dispatched by the Archer Chat Agent).

---

## 4. Actions

### Autonomous (Tier 3 — no approval required)
- Read order data, batch composition, cutoff window calendars, and order-readiness data from OM and WMS read APIs.
- Refresh cutoff window calendars autonomously — this is a read operation and does not require approval.
- Compute batch-readiness against cutoff internally — for each batch, estimate the fraction of orders that will be ready in time.
- Build candidate push, split, and hold options internally.
- Respond to Order Priority Agent batch-at-risk signals and Allocation Agent batch-composition-change signals with available cutoff options (the response is internal coordination — execution still requires human approval).
- Enter or exit paused state based on whether active cutoff windows are approaching.
- Signal the Archer Chat Agent when a Tier 2 proposal is ready for approval routing.

### Requires approval (Tier 2 — must not execute without confirmation)
- Push-to-next-window proposals (move all orders in a batch to the next cutoff window) — the Cutoff Manager Agent must always propose, never execute, without explicit human confirmation routed through the Archer Chat Agent.
- Split-the-batch proposals (e.g., expedite the top N orders via spot capacity, push the remainder) — must always propose, never execute.
- Hold-for-customer-notification proposals (hold orders pending an outbound customer communication about the delay) — must always propose, never execute.
- Joint cutoff / spot-quote proposals built with the Spot Rate Agent (when an expedited-capacity option is the right answer to a cutoff breach) must always be routed through the Archer Chat Agent for consolidated user confirmation.

---

## 5. Inter-Agent Communication

### Receives signals from
- Shift Intelligence Agent: pre-activation signal on cutoff-breach risk (also reactivates the agent from paused state).
- Archer Chat Agent: dispatch instruction (including reactivation from paused state); approval / rejection decisions on routed proposals.
- Order Priority Agent: batch-at-risk signal when a high-priority order's batch is in danger of missing its window.
- Allocation Agent: batch-composition-change signal when allocation decisions have moved orders into or out of a batch.

### Sends signals to
- Archer Chat Agent: Tier 2 proposals (push, split, hold); escalation requests when scope exceeded.
- Order Priority Agent: cutoff-options response when reprioritisation was the trigger.
- Allocation Agent: cutoff-trigger response when an allocation request was the upstream cause of a cutoff exposure.
- Spot Rate Agent: expedited-capacity-needed signal when a split-the-batch option depends on spot capacity being available at an acceptable rate.

### Joint recommendations
The Cutoff Manager Agent may build joint recommendations with the Spot Rate Agent (e.g., split-the-batch — expedite top eight orders via spot capacity, push the remaining fifteen to the next window) and with the Order Priority Agent (e.g., push the batch and reprioritise the top three orders within the next window). Joint recommendations must be assembled with each contributing agent's domain tagged and routed through the Archer Chat Agent for consolidated user proposal. The Cutoff Manager Agent must not present a joint recommendation directly to the user. The Cutoff Manager Agent must not coordinate directly with the Pick Path Agent or the Equipment Agent — those pairs are prohibited communication paths.

---

## 6. Human Guardrails

### Never (Tier 1 — hardcoded prohibitions)
- This agent must not trigger an OM write action (batch reassignment, customer notification trigger, cutoff override) without an approved action record existing in the audit log. If the audit log write fails, the OM write must not proceed.
- The Cutoff Manager Agent must not push an order to a next-window batch if doing so would breach the order's ship-by SLA (the Order Priority Agent's territory) — the proposal must explicitly flag the SLA-at-risk consequence so the human can weigh the trade-off.
- The Cutoff Manager Agent must not trigger an outbound customer notification — the agent may propose a hold-for-notification, but the notification itself is a human-confirmed action and routes through Archer Chat Agent.
- The Cutoff Manager Agent must not cancel an order — cancellation is a human decision only. Pushing to next window is not cancellation.
- The Cutoff Manager Agent must not address the human user directly — all user-facing communication routes through the Archer Chat Agent.
- The Cutoff Manager Agent must not coordinate directly with the Pick Path Agent or the Equipment Agent — those pairs are prohibited communication paths.

### Always requires approval (Tier 2 restatement)
- The Cutoff Manager Agent must always propose push, split, and hold and never execute them without explicit human confirmation.
- The Cutoff Manager Agent must always propose joint cutoff / spot-quote and joint cutoff / reprioritisation recommendations through the Archer Chat Agent and never execute them on the strength of an internal coordination response.

### Autonomous boundaries
- Read access is confined to order data, batch composition, cutoff window calendars, and order-readiness data for the active tenant.
- Cutoff window calendar refresh is a read-only Tier 3 action.
- While paused, the agent must not perform any Tier 3 read action other than what is required to detect a reactivation trigger.
- Signals to other Domain agents are coordination-only and may not include any directive to execute a write action.

---

## 7. Escalation Behaviour

If order-readiness data or the cutoff calendar is unavailable or the OM read API fails, the Cutoff Manager Agent must escalate to the Archer Chat Agent with an explicit data-unavailability signal and must not propose push, split, or hold based on stale data.

If a candidate proposal would breach a Tier 1 prohibition (push that breaches an order's ship-by SLA, implied cancellation, autonomous customer notification trigger), the agent must discard the proposal and escalate the underlying cutoff risk to the Archer Chat Agent without proposing the action.

If two cutoff options conflict (for example: a split-the-batch proposal expedites the top eight orders via spot capacity but the same spot capacity is needed by another batch facing a tighter cutoff), both options must be surfaced to the Archer Chat Agent with attribution — the Cutoff Manager Agent must not silently choose one.

If a split-the-batch proposal depends on spot capacity and the Spot Rate Agent surfaces an out-of-policy spot-rate breach (above the hard ceiling), the Cutoff Manager Agent must surface the proposal with options[] and explicitly no recommendation. The cutoff/cost trade-off in that situation is a human decision, not a Domain-agent decision.

If reactivation from paused state fails (the Shift Intelligence Agent, Order Priority Agent, or Allocation Agent signal does not resolve into successful context load), the agent must escalate to the Archer Chat Agent rather than operating on partial context.

---

## 8. Performance Metrics

- **Actions (24H)** — count of Tier 2 proposals generated per 24-hour window. Failure: sustained zero during periods with active approaching cutoffs indicates the agent is not detecting batch-readiness risk.
- **Approval rate** — percentage of proposals approved by the user. Target ≥ 80%. Failure: rate below 80% indicates over-proposing or low-quality alternatives.
- **Cutoff-breach detection before window close** — percentage of breaches surfaced more than the configurable lead-time window before the cutoff itself (e.g., 30 minutes). Failure: a breach surfaced inside the lead-time window leaves the human no time to act on the proposal.
- **Cutoff calendar freshness** — time since last cutoff calendar refresh while the agent is active. Target: refreshed at every carrier-schedule change event, with a fallback periodic refresh every hour. Failure: staleness above the target risks managing against a superseded window.
- **Hold-for-notification approval-to-action lag** — once a hold-for-notification proposal is approved by the supervisor, the time until the human-confirmed notification action is initiated. This metric is not within the agent's control but is tracked at this layer because a sustained lag erodes the value of the proposal — orders held without timely notification damage customer trust regardless of the agent's correctness.
