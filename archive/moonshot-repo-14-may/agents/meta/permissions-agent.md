# Permissions Agent
**Layer:** Meta (backstage)
**Role:** Governs visibility — filters which proposals, queues, and surfaces each persona can access. Does not approve, escalate, or decide; only filters.
**Status at Stage 2:** Always active during demo / UX test runs; absent in shipped product.

> **This is a meta agent.** It does not belong to the WMS / TMS / OMS topologies. It does not address the user. It is backstage — it *is* the policy layer that decides what each persona sees. Its only purpose is to make the prototype's authority model visible so the UX argument ("agents propose, humans decide, different humans see different decisions") lands.

---

## Provenance

Original to this project. Codified on 2026-05-13 alongside Task 18's per-pillar persona expansion. Until that day the prototype shipped a single Operator/Manager toggle scoped to Warehouse Advantage only. The expansion to three pillars + Watchtower required an explicit, named policy layer rather than ad-hoc conditionals scattered across render functions.

---

## 1. Identity & Purpose

The Permissions Agent owns one piece of state — the current persona — and exposes one decision: *can this persona see this surface?* It is consulted by every render path that produces a Tier-2 or Tier-3 surface.

It produces:
- A tier number (1, 2, or 3) for the active persona.
- A boolean visibility verdict for any (persona, surface) pair.
- A "locked" rendering hint for surfaces a lower-tier persona is allowed to see but not act on.

It is explicitly not responsible for: emitting events, scoring confidence, escalating decisions, writing audit entries, or rendering surfaces itself. Render functions still own their DOM; they ask the Permissions Agent what to render.

---

## 2. Tier definitions

Tiers are **additive** — a higher tier sees everything lower tiers see, plus more.

### Tier 1 — Operator-level
- **Personas:** Operator (WA), Dispatcher (TM), Order Manager (OM).
- **Sees:** tactical proposals scoped to their assigned zone/region/channel; in-the-moment status; signals that need a hand on the floor.
- **Cannot see:** approval requests over threshold, pillar-level rollups outside their slice, Watchtower, cross-pillar cascade decisions.
- **Cannot do:** approve anything tagged Tier-2 or Tier-3.

### Tier 2 — Manager-level
- **Personas:** Manager (WA), Transportation Manager (TM), Customer Service Manager (OM).
- **Sees:** everything Tier 1 sees, **plus** approval requests over threshold, pillar-level exceptions, pillar-level rollups, and the slice of cross-pillar decisions that lands in their pillar.
- **Cannot see:** the full Watchtower view, the cross-pillar cascade decision card (that lives with the Director).
- **Can do:** approve Tier-1 and Tier-2 surfaces in their own pillar.

### Tier 3 — Director-level
- **Personas:** Director of Operations (cross-pillar only).
- **Sees:** everything across all pillars. Exclusive access to Watchtower and the cross-pillar cascade decision card.
- **Cannot see:** N/A — Tier 3 sees everything.
- **Can do:** approve coordinated responses that touch more than one pillar.

The Director persona is **only available in the cross-pillar context**. The header dropdown does not surface Director when the active product is WA, TM, or OM.

---

## 3. Visibility rules per surface

### Decision queue items
Each pending item carries a `requiredTier` (1 or 2). An item is visible-and-actionable to persona P iff `P.tier >= item.requiredTier`. An item with `requiredTier > P.tier` either renders in a **locked state** (showing agent name, confidence, timestamp; masking the action with "Tier N approval required — switch to a Manager persona to view") or is omitted entirely from the per-pillar queue depending on context.

In the cross-pillar unified queue (Watchtower), all items are visible to the Director — Watchtower itself is Tier 3 only, so no further filtering applies.

### Watchtower view
Tier 3 only. If a Tier-1 or Tier-2 persona navigates to Watchtower (via the cross-pillar icon in the product switcher), render the **permission-denied state** described in §4 — not the actual Watchtower content.

### Archer Chat
Same underlying thread for all tiers; surfaces differ:
- Tier 1 sees tactical guidance and status messages.
- Tier 2 sees everything Tier 1 sees, plus approval prompts.
- Tier 3 sees everything, plus coordinated cross-pillar prompts (e.g., the cascade decision invitation).
When a cross-pillar cascade is in `awaiting-decision` state, Tier-1 and Tier-2 personas see a synthetic Archer line — "Cascade decision escalated to Director of Operations — awaiting approval." — instead of the decision card.

### Cascade decision card
Tier 3 only. The `renderCascadeDecisionCard` function returns an empty string for any persona below Tier 3. If a Director persona becomes active mid-cascade, the card appears in their view on the next render — no state replay required, because cascade state lives in `CROSS_DATA`.

### Stage card (per-pillar pending decision)
Each scenario / incident carries a `requiredTier` (defaults to 2 — the demo's pending decisions are pillar-level approvals, not floor-level tactical work). A Tier-1 persona sees the agent's framing and impacts but no action buttons; instead, a "Manager approval required" line. Tier 2+ sees the full action set.

---

## 4. Visible permission denial

Lower-tier personas landing on a higher-tier surface see an **explicit denial state**, not a silent empty page:

- **Lock icon** rendered with Infios stroke iconography (no new colours).
- **Heading:** what tier is required, e.g., "Cross-pillar view requires Director of Operations tier."
- **Body:** one sentence on what the surface aggregates and why this tier is needed.
- **Affordance:** a "Switch persona" button that opens the header dropdown (or points the user to it).
- **Visual:** muted greys, charcoal text, no blue — the surface reads as *intentionally closed*, not broken.

The principle: **authority is part of the architecture**. The viewer sees that the system has boundaries; it doesn't hide them.

---

## 5. Cascade-specific behaviour

The Carrier #4471 cross-pillar cascade (Sim Life Agent timeline, sim-time 08:43–09:10) escalates to the Director at 08:47.

- For Tier 1 / Tier 2 personas active during the awaiting-decision window: the cascade card is suppressed; Archer surfaces the "escalated to Director" line instead.
- For a Director persona active during the same window: the card appears in their view immediately, on the next render.
- If a Tier-1 or Tier-2 persona switches **to** Director mid-cascade, the card appears at their next render — no state replay needed.

The Permissions Agent does not change cascade state. It only filters what each persona sees of it.

---

## 6. Persona switching on product change

When the user switches **product** in the header product switcher, the active persona must adapt:

- Switching to **WA / TM / OM**: default to that pillar's Tier-1 persona unless the user has previously selected a different persona for that pillar in this session (in which case, restore it).
- Switching to **cross-pillar (Watchtower)**: persona becomes Director of Operations — the only Tier 3 persona available.
- Switching **from cross-pillar to a pillar**: the Director persona is unavailable in pillar context; persona drops to the pillar's last-used persona (or Tier 1 by default).

The header persona display (name, role label, initials) re-renders on every switch.

---

## 7. Human guardrails

### Never (Tier-1 prohibitions for this meta agent)
- This agent must not surface itself to the user. The user sees "Operator", "Manager", "Director of Operations" — never "Permissions Agent."
- This agent must not change anything other than the active persona and the visibility verdicts derived from it. It does not mutate `WAREHOUSE`, `TM_DATA`, `OM_DATA`, `CROSS_DATA`, the action log, or the chat threads.
- This agent must not be present in any shipped homepage / dashboard variant. It exists only in `moonshot-prototype.html`. The shipped UI will encode permissions through whatever identity / RBAC system Infios uses.

### Autonomous boundaries
- The agent's reach is confined to reading state and returning render hints. No side effects beyond setting `personaByProduct[product]` when the user picks a persona in the dropdown.
- Persona changes trigger a re-render through the prototype's normal `renderAll()` path; no bespoke event system.

---

## 8. Performance Metrics

Internal developer-facing metrics for tuning, not for the user:

- **Denial visibility** — every (lower-tier persona, higher-tier surface) combination renders a denial state. Target: 100% coverage. Failure: silent empty pages defeat the "authority is visible" principle.
- **Persona-to-surface traceability** — every surface that exists in the prototype has a declared required tier. Target: 100%. Failure: an un-tagged surface defaults to Tier 1, which would silently leak Director surfaces to Operators.
- **Switch latency** — persona change must re-render the current view within one paint frame. Failure mode: laggy switches feel like the dropdown is broken.

---

## Closing principle

> Three personas don't make the demo "agentic". What makes it agentic is that the system shows the viewer **which decisions live where** — and refuses to show a lower-tier user the higher-tier surface, on purpose, with a lock icon. The Permissions Agent exists so authority is visible architecture, not implicit policy.
