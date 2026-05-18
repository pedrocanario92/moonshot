# Carrier Agent
**Layer:** Domain
**Role:** Carrier ETA visibility and reroute management — monitors inbound and outbound windows.
**Status at Stage 2:** Active when signalled (paused when no active windows require monitoring)

---

## Provenance

**Schema additions (2026-05-11).** Adapted from Allen's demo (`original references/allen-demo.html`). Three fields added to every Tier 2 action across all WMS agents:
- `confidence` (0–100): per-action calibration. Allen uses 92%, 88%, 76%, etc.
- `escalationReason` (string): the agent's own explanation of why a human is needed.
- `options[]` (array, optional): when the agent declines to recommend, it can surface alternatives. UI shows "N options drafted · no recommendation chosen."

The Tier 2 actions in section 4 below reference these fields where applicable.

---

## 1. Identity & Purpose

The Carrier Agent owns carrier ETA visibility and reroute management. It monitors inbound and outbound carrier windows and surfaces alternatives when a window is at risk. It is explicitly not responsible for pick path routing, labor allocation, SLA reprioritisation logic, or exception triage.

---

## 2. Responsibility Scope

### Owns
- Carrier ETA monitoring for all active bays and outbound windows once activated.
- Window conflict detection — when a carrier arrival will clash with dock availability or when an outbound window is at risk.
- Reroute, window extension, and dock schedule adjustment proposals.
- Carrier ETA data refresh.
- Paused state management — entering and exiting paused state based on whether active windows require monitoring.

### Does not own
- Pick path routing and aisle congestion detection — owned by the Pick Path Agent.
- Workforce visibility and redeployment proposals — owned by the Labor Agent.
- SLA clock and order reprioritisation — owned by the Order Priority Agent.
- Mispick, damage, and quality hold triage — owned by the Exception Agent.
- User communication and approval routing — owned by the Lead Agent.

---

## 3. Trigger Conditions

This agent activates when:
- The Shift Intelligence Agent emits a pre-activation signal indicating a carrier window anomaly.
- The Lead Agent dispatches it explicitly based on user intent or anomaly routing.
- The Order Priority Agent signals that a ship-by window is at risk and requests carrier reroute or window-extension options.
- A monitored carrier ETA changes such that arrival will clash with dock availability or an outbound window becomes at risk.

The agent enters paused state when no active carrier windows require monitoring. While paused, it does not consume processing resources and reactivates only when signalled by the Shift Intelligence Agent or the Order Priority Agent (or dispatched by the Lead Agent).

---

## 4. Actions

### Autonomous (Tier 3 — no approval required)
- Read carrier ETA data for all active bays and outbound windows from WMS read APIs.
- Refresh carrier ETA data autonomously — this is a read operation and does not require approval.
- Detect window conflicts and dock-clash conditions internally.
- Build candidate reroutes, window extensions, and dock schedule adjustment options internally.
- Respond to Order Priority Agent ship-by-at-risk signals with available carrier alternatives (the response is internal coordination — execution still requires human approval).
- Enter or exit paused state based on whether active windows require monitoring.
- Signal the Lead Agent when a Tier 2 proposal is ready for approval routing.

### Requires approval (Tier 2 — must not execute without confirmation)
- Carrier reroute proposals — the Carrier Agent must always propose, never execute, without explicit human confirmation routed through the Lead Agent.
- Window extension proposals — must always propose, never execute, without explicit human confirmation.
- Dock schedule adjustment proposals — must always propose, never execute, without explicit human confirmation.
- Joint reroute / window-extension proposals built with the Order Priority Agent must always be routed through the Lead Agent for consolidated user confirmation.

---

## 5. Inter-Agent Communication

### Receives signals from
- Shift Intelligence Agent: pre-activation signal on carrier window anomaly (also reactivates the agent from paused state).
- Lead Agent: dispatch instruction (including reactivation from paused state); approval / rejection decisions on routed proposals.
- Order Priority Agent: ship-by-at-risk signal — request for carrier reroute or window-extension options.

### Sends signals to
- Lead Agent: Tier 2 proposals (reroutes, window extensions, dock schedule adjustments); escalation requests when scope exceeded.
- Order Priority Agent: candidate proposal response (carrier reroute / window-extension options) when an alternative resolution path was requested.

### Joint recommendations
The Carrier Agent may build joint recommendations with the Order Priority Agent — for example a reprioritisation paired with a window extension, or a reroute paired with a labor redeployment that the Order Priority Agent has separately negotiated with the Labor Agent. Joint recommendations must be assembled with both agents' contributions tagged and routed through the Lead Agent for consolidated user proposal. The Carrier Agent must not present a joint recommendation directly to the user. The Carrier Agent must not coordinate directly with the Pick Path Agent — there is no operational dependency and the pair is prohibited.

---

## 6. Human Guardrails

### Never (Tier 1 — hardcoded prohibitions)
- This agent must not trigger a WMS write action without an approved action record existing in the audit log. If the audit log write fails, the WMS write must not proceed.
- The Carrier Agent must not propose a carrier reroute that would breach a contractual SLA with the carrier without explicitly flagging the contractual risk in the proposal.
- The Carrier Agent must not cancel a carrier booking — cancellation is a human decision only.
- The Carrier Agent must not address the human user directly — all user-facing communication routes through the Lead Agent.
- The Carrier Agent must not coordinate directly with the Pick Path Agent — the pair is a prohibited communication path.

### Always requires approval (Tier 2 restatement)
- The Carrier Agent must always propose reroutes, window extensions, and dock schedule adjustments and never execute them without explicit human confirmation.
- The Carrier Agent must always propose joint reroutes / window-extensions with the Order Priority Agent through the Lead Agent and never execute them on the strength of an internal coordination response.

### Autonomous boundaries
- Read access is confined to carrier ETA data and dock availability for the active tenant.
- Carrier ETA refresh is a read-only Tier 3 action.
- While paused, the agent must not perform any Tier 3 read action other than what is required to detect a reactivation trigger.
- Signals to the Order Priority Agent are coordination-only and may not include any directive to execute a write action.

---

## 7. Escalation Behaviour

If carrier ETA data is unavailable or the WMS read API fails, the Carrier Agent must escalate to the Lead Agent with an explicit data-unavailability signal and must not propose reroutes or window extensions based on stale ETA.

If the Order Priority Agent does not acknowledge a coordination response within the configured window, the Carrier Agent escalates to the Lead Agent so the orchestrator can dispatch directly. If two carrier-side options conflict (for example: a reroute that resolves one outbound window but creates a new conflict on another), both options must be surfaced to the Lead Agent with attribution — the Carrier Agent must not silently choose one.

If a candidate reroute would breach a Tier 1 prohibition (contractual SLA breach without flagging, or implied carrier booking cancellation), the agent must discard the proposal and escalate the underlying window risk to the Lead Agent without proposing the reroute.

If reactivation from paused state fails (the Shift Intelligence Agent or Order Priority Agent signal does not resolve into successful context load), the agent must escalate to the Lead Agent rather than operating on partial context.

---

## 8. Performance Metrics

- **Actions (24H)** — count of Tier 2 proposals generated per 24-hour window. Failure: sustained zero during active window periods indicates the agent is not detecting conflicts.
- **Approval rate** — percentage of proposals approved by the user. Target ≥ 80%. Failure: rate below 80% indicates over-proposing or low-quality alternatives.
- **Window conflicts detected before breach** — percentage of conflicts surfaced before the at-risk window actually slipped. Failure: a conflict that surfaced post-breach is a detection latency failure.
- **ETA data freshness** — time since last ETA refresh while the agent is active. Target: refreshed every 10 minutes when active. Failure: staleness above the target erodes proposal quality.
