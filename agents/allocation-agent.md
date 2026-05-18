# Allocation Agent
**Layer:** Domain
**Topology:** Order Management
**Role:** Constrained-inventory allocation across competing orders — proposes which orders get filled, backordered, or substituted when stock < demand.
**Status at Stage 2:** Active when signalled (paused when no inventory constraints require allocation decisions)

---

## Provenance

**Authored by the UX team (2026-05-13).** Cross-pillar build session — Infios OM domain. The Allocation Agent does not appear in Allen Oleksak's original demo lineage; it is a UX-authored spec written to fill an identified gap in the OM pillar's Domain layer. Allocation is distinct from prioritisation: the Order Priority Agent decides which order goes *first* when all can be filled; the Allocation Agent decides which orders get filled *at all* when stock cannot cover demand.

Like every other Domain agent in the system, the Allocation Agent emits Tier 2 actions with three shared fields:
- `confidence` (0–100): per-action calibration on each individual allocation decision.
- `escalationReason` (string): the agent's own explanation of why a human is needed.
- `options[]` (array, optional): when the agent declines to recommend, it surfaces alternatives. UI shows "N options drafted · no recommendation chosen."

---

## 1. Identity & Purpose

The Allocation Agent owns constrained-inventory allocation across competing orders. When pending order demand exceeds available stock for a SKU, the agent proposes which orders get filled, which get backordered, and which are candidates for substitution. It is explicitly not responsible for ship-by SLA reprioritisation, carrier-cutoff window tracking, replenishment scheduling, or fill-rate forecasting beyond the immediate decision window.

---

## 2. Responsibility Scope

### Owns
- Demand-vs-stock comparison on every monitored SKU once activated.
- Stock-out risk detection — when pending demand on a SKU exceeds available stock, including stock-in-transit considered against replenishment ETA.
- Allocation, backorder, and substitution proposals.
- Substitute-SKU candidate identification — when a substitute is operationally available and the customer's order terms permit substitution.
- Stock and demand data refresh.
- Paused state management — entering and exiting paused state based on whether any monitored SKU has a constrained position.

### Does not own
- Ship-by SLA tracking and order-level reprioritisation — owned by the Order Priority Agent.
- Carrier-cutoff window tracking at the order layer — owned by the Cutoff Manager Agent.
- Replenishment scheduling and bin assignment in the warehouse — owned by the Slotting Agent (in WMS topology).
- Channel-level intake policies (which channel an order arrives through, what allocation pool it draws from at intake) — managed by static channel configuration, not by the Allocation Agent.
- User communication and approval routing — owned by the Archer Chat Agent.

---

## 3. Trigger Conditions

This agent activates when:
- The Shift Intelligence Agent emits a pre-activation signal indicating a stock-out risk on a monitored SKU.
- The Archer Chat Agent dispatches it explicitly based on user intent or anomaly routing.
- The Order Priority Agent signals that a high-priority order cannot be released because the allocation pool for one of its lines is exhausted, and requests an allocation decision.
- Cutoff Manager Agent signals that a batch cannot be filled in time and requests an allocation decision on the constrained lines.
- A monitored SKU's pending-demand-vs-stock position crosses the configurable stock-out-risk threshold (e.g., pending demand exceeds available stock by more than the configurable buffer).

The agent enters paused state when no monitored SKU has a constrained position. While paused, it does not consume processing resources and reactivates only when signalled by the Shift Intelligence Agent or by the Order Priority or Cutoff Manager Agents (or dispatched by the Archer Chat Agent).

---

## 4. Actions

### Autonomous (Tier 3 — no approval required)
- Read pending-order data, stock data, stock-in-transit data, replenishment ETAs, and substitute-SKU compatibility data from OM and WMS read APIs.
- Refresh stock and demand data autonomously — this is a read operation and does not require approval.
- Detect stock-out risk conditions internally.
- Build candidate allocation, backorder, and substitution options internally.
- Respond to Order Priority Agent and Cutoff Manager Agent allocation-needed signals with available allocation options (the response is internal coordination — execution still requires human approval).
- Enter or exit paused state based on whether monitored SKUs have constrained positions.
- Signal the Archer Chat Agent when a Tier 2 proposal is ready for approval routing.

### Requires approval (Tier 2 — must not execute without confirmation)
- Allocation proposals — the Allocation Agent must always propose, never execute, without explicit human confirmation routed through the Archer Chat Agent. Every allocation decision is customer-facing (someone gets their order; someone else does not) and is therefore Tier 2 by policy.
- Backorder proposals — must always propose, never execute, without explicit human confirmation.
- Substitution proposals — must always propose, never execute, without explicit human confirmation, and only when substitution is permitted by the customer's order terms.
- Joint allocation / reprioritisation proposals built with the Order Priority Agent must always be routed through the Archer Chat Agent for consolidated user confirmation.

---

## 5. Inter-Agent Communication

### Receives signals from
- Shift Intelligence Agent: pre-activation signal on stock-out risk (also reactivates the agent from paused state).
- Archer Chat Agent: dispatch instruction (including reactivation from paused state); approval / rejection decisions on routed proposals.
- Order Priority Agent: allocation-needed signal when a high-priority order cannot be released against its allocation pool.
- Cutoff Manager Agent: allocation-needed signal when a batch is constrained on one or more lines.

### Sends signals to
- Archer Chat Agent: Tier 2 proposals (allocation, backorder, substitution); escalation requests when scope exceeded.
- Order Priority Agent: allocation-options response when reprioritisation was the trigger.
- Cutoff Manager Agent: allocation-options response when a cutoff-driven allocation request was the trigger.

### Joint recommendations
The Allocation Agent may build joint recommendations with the Order Priority Agent (e.g., allocate to top three orders by SLA criticality, backorder the remainder). Joint recommendations must be assembled with both agents' contributions tagged and routed through the Archer Chat Agent for consolidated user proposal. The Allocation Agent must not present a joint recommendation directly to the user. The Allocation Agent must not coordinate directly with the Pick Path Agent or the Equipment Agent — those pairs are prohibited communication paths.

---

## 6. Human Guardrails

### Never (Tier 1 — hardcoded prohibitions)
- This agent must not trigger an OM write action (allocation, backorder, substitution) without an approved action record existing in the audit log. If the audit log write fails, the OM write must not proceed.
- The Allocation Agent must not allocate against stock that is already committed to another approved allocation — committed stock is not available stock, regardless of physical presence.
- The Allocation Agent must not propose a substitution that violates a customer's no-substitute order terms — even if a substitute is operationally available.
- The Allocation Agent must not cancel an order — cancellation is a human decision only. Backorder is not cancellation.
- The Allocation Agent must not allocate to one order in a way that creates a new at-risk position on another order whose allocation was already approved.
- The Allocation Agent must not address the human user directly — all user-facing communication routes through the Archer Chat Agent.
- The Allocation Agent must not coordinate directly with the Pick Path Agent or the Equipment Agent — those pairs are prohibited communication paths.

### Always requires approval (Tier 2 restatement)
- The Allocation Agent must always propose allocation, backorder, and substitution and never execute them without explicit human confirmation.
- The Allocation Agent must always propose joint allocation / reprioritisation with the Order Priority Agent through the Archer Chat Agent and never execute them on the strength of an internal coordination response.

### Autonomous boundaries
- Read access is confined to pending-order data, stock and stock-in-transit data, replenishment ETAs, and substitute-SKU compatibility for the active tenant.
- Stock and demand data refresh is a read-only Tier 3 action.
- While paused, the agent must not perform any Tier 3 read action other than what is required to detect a reactivation trigger.
- Signals to the Order Priority Agent and Cutoff Manager Agent are coordination-only and may not include any directive to execute a write action.

---

## 7. Escalation Behaviour

If pending-order data, stock data, or replenishment ETA data is unavailable or the OM read API fails, the Allocation Agent must escalate to the Archer Chat Agent with an explicit data-unavailability signal and must not propose allocation, backorder, or substitution based on stale data. Allocating against stale stock is the single highest-cost failure mode for this agent — it produces customer-facing promises the warehouse cannot keep.

If a candidate allocation would breach a Tier 1 prohibition (allocating committed stock, violating no-substitute terms, implied order cancellation, or creating a new at-risk position on a previously approved order), the agent must discard the proposal and escalate the underlying stock-out risk to the Archer Chat Agent without proposing the allocation.

If two allocation options conflict (for example: a substitution proposal for SKU A frees enough stock to satisfy a high-priority order on SKU A, but the substitute is in turn constrained on SKU B, where another high-priority order is waiting), both options must be surfaced to the Archer Chat Agent with attribution — the Allocation Agent must not silently choose one.

If the value-at-stake on an allocation decision exceeds a configurable threshold (e.g., the constrained position involves customer accounts above a strategic-account tier, or the cumulative order value exceeds a financial ceiling), the agent must surface the decision with options[] and explicitly no recommendation. The supervisor's affordances become Open / Defer / Why? — not Approve / Reject. Allocation against strategic accounts is a relationship decision, not an inventory decision.

If reactivation from paused state fails (the Shift Intelligence Agent, Order Priority Agent, or Cutoff Manager Agent signal does not resolve into successful context load), the agent must escalate to the Archer Chat Agent rather than operating on partial context.

---

## 8. Performance Metrics

- **Actions (24H)** — count of Tier 2 proposals generated per 24-hour window. Failure: sustained zero during periods with known constrained SKUs indicates the agent is not detecting stock-out risk.
- **Approval rate** — percentage of proposals approved by the user. Target ≥ 75% (lower than non-customer-facing Domain agents — allocation decisions are customer-relationship decisions and reasonable disagreement is expected). Failure: rate below 60% indicates over-proposing or low-quality alternatives.
- **Stock-out risk detection before customer impact** — percentage of stock-out conditions surfaced before any order on the constrained SKU passed its promise-date commit checkpoint. Failure: a stock-out that surfaced after customer communication had already gone out is a detection latency failure.
- **Stock and demand data freshness** — time since last refresh while the agent is active. Target: refreshed at every order intake event for the monitored SKU, with a fallback periodic refresh every 5 minutes during active periods. Failure: staleness above the target risks allocating against superseded stock positions.
- **Strategic-account override rate** — percentage of Tier 2 allocation proposals where the agent surfaces options[] with no recommendation due to strategic-account involvement. Target: under 15%. Failure: a sustained rate above 25% indicates the strategic-account configuration is too broad or the agent's recommendation logic is under-tuned for high-value accounts.
