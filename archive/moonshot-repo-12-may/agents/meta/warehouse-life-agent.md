# Warehouse Life Agent
**Layer:** Meta (backstage)
**Role:** Scripted-causal simulation engine — generates the events the WMS agents react to.
**Status at Stage 2:** Always active during demo / UX test runs; absent in shipped product

> **This is a meta agent.** It does not belong to the WMS topology. It does not propose, escalate, or address the user. It is backstage — it *is* the simulated warehouse. Its only purpose is to make the prototype's UI come alive so the UX can be exercised under operation.

---

## Provenance

Original to this project. No analogue in Allen's demo (`original references/allen-demo.html`) — Allen's demo is a static React snapshot. This agent's job is to make ours non-static so the "agents hide what they handle" UX argument can be tested against a warehouse that's actually happening.

---

## 1. Identity & Purpose

The Warehouse Life Agent owns the state of one simulated distribution center across one shift, compressed from 90 sim-minutes (08:00–09:30) into ~5 wall-minutes of demo time. It emits root events on a scripted timeline and emits derived events when state crosses declared thresholds, producing the cascade of signals that the WMS agents (Shift Intelligence, Pick Path, Labor, Order Priority, Carrier, Exception, Quality, Slotting, Equipment) react to.

It is explicitly not responsible for proposing user-facing actions, formatting UI, gating approvals, computing performance metrics, or anything that the in-fiction agents own.

---

## 2. Responsibility Scope

### Owns
- The `WAREHOUSE` state object: clock, zones, agent runtime state, inbound/outbound dock schedule, incidents, chat thread, action log.
- The scripted root event timeline (08:00–09:30 sim-time).
- The causal rule set that emits derived events when state thresholds are crossed.
- The tick loop that advances the sim clock, fires scheduled events, evaluates rules, and triggers re-render.
- The compression ratio between sim-time and wall-time (18:1 by default).
- The initial seed state — the "typical morning" the user lands in.

### Does not own
- Any user-facing action proposal — owned by the relevant WMS agent (Pick Path, Labor, Order Priority, Carrier, Exception, Quality, Slotting, Equipment).
- Approval gating — owned by the Lead Agent.
- Audit log entry creation for approved actions — owned by the Lead Agent.
- Render functions or DOM construction — owned by the prototype's UI layer.
- Performance metric calculation — owned by `DATA.agentPerformance` (system + agent KPIs are derived from `WAREHOUSE` state by the perf view, not by this agent).

---

## 3. Trigger Conditions

This agent activates when:
- The prototype loads (the tick loop begins at page-load).
- The user clicks "Restart shift" (the tick loop resets to the seed state and replays from 08:00).

It does not activate in response to user actions in any other way. User approvals/rejections write to `WAREHOUSE` but do not cause the sim to branch — the scripted timeline always proceeds.

---

## 4. Actions

### Autonomous (Tier 3 — no approval required, no human-facing surface)
- Advance `WAREHOUSE.clock` on each tick.
- Fire scheduled root events from the timeline at their declared `at:` time.
- Evaluate every causal rule against current `WAREHOUSE` state on each tick.
- Emit derived events when causal rules fire.
- Mutate `WAREHOUSE` state in response to event effects.
- Call the prototype's `renderAll()` after state changes.

### Requires approval (Tier 2)
- None. This agent never proposes actions to the user. All user-facing proposals come from the WMS agents that react to its events.

---

## 5. Inter-Agent Communication

### Receives signals from
- None during normal operation. The sim is not reactive to other agents.
- The user via "Restart shift" — resets state and replays from seed.

### Sends signals to
- The WMS agents implicitly, via state changes. When `WAREHOUSE` state crosses a threshold (e.g., inbound slip > 15 min), the corresponding causal rule causes the WMS agent (e.g., Pick Path) to emit a Tier 2 proposal into `WAREHOUSE.incidents` and `WAREHOUSE.chatThread`.

### Joint recommendations
Not applicable. This agent does not participate in user-facing recommendations.

---

## 6. Human Guardrails

### Never (Tier 1 — hardcoded prohibitions)
- This agent must not surface itself to the user. No UI element, no chat message, no map pin, no log entry may attribute content to "Warehouse Life Agent." All content surfaced to the user is attributed to a WMS agent.
- This agent must not be present in the shipped product. It exists only in `moonshot prototype/moonshot-prototype.html`. `moonshot presentation/wms-unified-figma-sidebar.html` must remain free of any sim engine.
- This agent must not modify content authored by WMS agents (chat text, proposal descriptions, escalation reasons). It only triggers their emission via causal rules.

### Always requires approval (Tier 2 restatement)
Not applicable.

### Autonomous boundaries
- The agent's reach is confined to mutating the `WAREHOUSE` state object and calling `renderAll()`. No network calls, no persistence, no off-page side effects.
- The tick loop must be cancellable (a single `clearInterval` reference) so the prototype can pause for testing.

---

## 7. Escalation Behaviour

If `WAREHOUSE` state becomes internally inconsistent (e.g., an event fires that references a zone not in the layout), the agent must log a console warning and skip the event rather than mutate state into a contradiction. There is no human user to escalate to — escalation here means visible failure mode for the developer running the prototype.

If the tick loop has not advanced for more than 2 wall-seconds (a `setInterval` stall), no recovery is attempted — the prototype will appear frozen and a reload is the expected remediation.

---

## 8. Performance Metrics

Not applicable in the usual sense — this agent is not measured against approval rate, escalation rate, or any user-facing metric.

**Internal developer-facing metrics** (for tuning the sim, not for the user):
- **Event density** — events fired per wall-minute. Target: ≤ 2 user-visible events per wall-minute. Failure: a higher rate means the homepage will read as Allen-style noise.
- **Calm-state coverage** — percentage of wall-time during which the stage's "calm" state is shown. Target: ≥ 60% of the 5-min run. Failure: a lower rate means the user never sees the "nothing demands you" state — defeating the project's UX argument.
- **Event-to-spec traceability** — every event kind emitted must trace to a Tier 2 action or trigger condition in a WMS agent spec. Target: 100%. Failure: untraceable events indicate the sim is doing something no agent owns.

---

## Root event script (authoritative)

The complete 08:00–09:30 timeline lives as a JS const in `moonshot-prototype.html` and is mirrored here for traceability. Each entry has the shape:

```
{ at: 'HH:MM', kind: '<event-kind>', target: '<zone-or-asset>', ...args }
```

Initial seed at 08:00:
- 94 active pickers across 8 zones
- 1,284 orders in flight, 0 at SLA risk
- Inbound bays 1–4 scheduled (08:30, 09:00, 09:15, 09:45)
- 2 standing low-priority incidents from overnight shift (auto-resolving)

Scripted root events (placeholder — to be authored in Phase 7):
```
{ at: '08:14', kind: 'inbound_late',    target: 'bay-4', delay: 22, source: 'carrier-DHL-882' }
{ at: '08:28', kind: 'pick_rate_dip',   zone: 'C', severity: 'mild' }
{ at: '08:42', kind: 'sla_pressure',    wave: 'wave-8842', risk: 0.7 }
{ at: '08:55', kind: 'equipment_fault', asset: 'FL-09', mode: 'battery_low' }
{ at: '09:10', kind: 'quality_hold',    sku: 'SKU-44210-B', cartons: 3 }
{ at: '09:22', kind: 'inventory_variance', sku: 'SKU-99217', units: 312 }
```

## Causal rules (authoritative)

Each rule names a watched-state condition and a derived-event emission. Rules live as JS predicate functions in `moonshot-prototype.html` and are mirrored here for traceability.

Initial rule set (placeholder — to be authored in Phase 5):
- **R1:** If `inbound.slip_min > 15` → emit `pick_path.replen_at_risk` for affected SKUs.
- **R2:** If `zone.pickRate < 0.7 * baseline` sustained for 10 sim-min → emit `labor.reassign` proposal.
- **R3:** If `wave.slaConsumed > 0.8` and `wave.laborDelta < required` → emit `joint.order_priority+labor` proposal.
- **R4:** If `equipment.fault.rootCauseConfidence < 0.5` → emit `equipment.escalation` with `options[]`, no recommendation.
- **R5:** If `quality.hold` triggered → emit `quality.review` proposal with confidence based on damage pattern match.
- **R6:** If `inventory.variance > 100 units` → emit `quality.investigation` proposal (high value at stake, audit risk flagged).

## Compression

- 1 wall-second = 18 sim-seconds.
- 90 sim-minutes (08:00–09:30) plays in 5 wall-minutes.
- Tick interval: 500ms wall, so each tick advances 9 sim-seconds.
- "Restart shift" resets `WAREHOUSE.clock` to 08:00 and re-seeds state.
