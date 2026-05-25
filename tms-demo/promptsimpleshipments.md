# Shipments — Table + Details
## Guardian-first build prompt · TMS · Simple page

The Shipments page is a TMS operator view for monitoring active freight movements. It gives a dispatcher or logistics manager a real-time scan of all shipments in one table — carrier, route, mode, status, and ETA — and lets them drill into any shipment to see the full details and take action, specifically approving a route or pulling up the map view. The delayed row (TMS-4798, Dallas → LA, flagged with ⚠) is the key use case: the operator spots it in the table, selects it, sees the lane rate and ETA slip, and decides whether to approve the current route or escalate.

---

You are building a UI page for the Moonshot project (supply chain, Infios DS).

Before writing any HTML, you must run a Guardian review on the proposed design.

## The page
**Name:** Shipments — Table + Details
**Pillar:** TMS (Transportation Management)
**Purpose:** Allow a TMS user to monitor active shipments, identify delays, and approve or act on individual shipment routes from a single view.

**Layout:** Two-panel — a data table on the left, a details tile on the right that opens when a row is selected.

**Table columns:** Shipment ID · Route · Carrier · Mode · Status · ETA

**Detail tile fields (on row select):**
- Route section: Origin · Destination · Mode
- Carrier & Schedule section: Carrier · Pickup date · ETA · Lane rate
- Actions: "Approve route" (primary) · "View on map" (secondary)

**States:**
- Calm (no row selected): table visible, details tile shows empty state with "Select a shipment"
- Selection: details tile populates; one primary action visible
- No persistent decorative or informational elements above the table

**Dummy data:**

| ID | Route | Carrier | Mode | Status | ETA |
|---|---|---|---|---|---|
| TMS-4821 | Chicago → Atlanta | FedEx Freight | FTL | In Transit | May 26 |
| TMS-4798 | Dallas → Los Angeles | XPO Logistics | LTL | Delayed | May 28 ⚠ |
| TMS-4756 | Newark → Boston | ABF Freight | LTL | Delivered | May 22 ✓ |
| TMS-4801 | Seattle → Portland | Old Dominion | LTL | In Transit | May 25 |
| TMS-4815 | Memphis → Nashville | UPS Freight | FTL | Pending | May 27 |
| TMS-4829 | Denver → Kansas City | Estes Express | LTL | In Transit | May 26 |

**Selected row detail (TMS-4798 as example):**
Origin: Dallas, TX · Destination: Los Angeles, CA · Mode: LTL
Carrier: XPO Logistics · Pickup: May 22 · ETA: May 28 ⚠ · Lane rate: $1,870

---

## Step 1 — Derive the design spec
From the page definition above, produce a concise design spec:
- Interactive elements count above the fold
- Actions available at each state (calm, selection)
- Any persistent decorative or informational elements
- Anything ambiguous or missing that could affect the audit

## Step 2 — Run the Guardian review
Evaluate the spec against all 6 Guardian agents in sequence.

**Heuristics Agent** (fully specced — load-bearing FAILs block generation):
- H1. Nielsen #8 — Aesthetic & minimalist design (≤6 elements above fold)
- H2. Hick's Law — actionable choices at any state (≤3 primary actions)
- H3. Miller's 7±2 — distinct information chunks (≤7)
- H4. Cognitive Load — no extraneous elements
- H5. Information scent — calm state has one primary focus
- H6. Information scent — busy state has exactly one primary action
- H7. F-pattern alignment (advisory)
- H8. Banner blindness (advisory)
- H9. Recognition over recall (advisory)
- H10. Consistency (advisory)

**Accessibility Agent** (fully specced — Level AA FAILs block generation; AAA is advisory only and never surfaces as WARN when AA is met):
- A1. WCAG 1.4.3 Contrast — body text ≥4.5:1, large text ≥3:1. PASS if met; FAIL if not.
- A2. WCAG 2.1.1 Keyboard — all actions must be keyboard reachable
- A3. WCAG 2.2.1 Timing Adjustable — no auto-dismissing actions
- A4. WCAG 2.3.1 Three Flashes — no rapid animation
- A5. WCAG 2.4.3 Focus Order — logical tab sequence
- A6. WCAG 2.4.7 Focus Visible — focus indicators required
- A7. WCAG 2.5.8 Target Size — ≥24×24px minimum
- A8. WCAG 3.2.4 Consistent Identification
- A9. WCAG 3.3.5 Help (AAA advisory — surface only if escalation paths are missing)
- A10. WCAG 1.4.6 Contrast Enhanced (AAA advisory — verdict is N/A when AA contrast is met; only surface if A1 is borderline)

**Design System Agent** (stub):
- DS1. Color tokens — values reference Infios DS tokens
- DS2. Typography — sizes and weights use DS scale
- DS3. Spacing — padding and gap values match DS scale

**Content Agent** (stub):
- C1. Action labels — imperative verb + object pattern
- C2. Status labels — consistent capitalisation and tense
- C3. Empty states — instructional copy present

**Interaction Agent** (stub):
- I1. Row selection — clear selected state affordance
- I2. Panel open/close — consistent dismiss pattern
- I3. Action confirmation — irreversible actions gated

**Responsive Agent** (stub):
- R1. Layout degrades gracefully at 1024px
- R2. Table horizontal scroll before clipping
- R3. Touch targets maintain minimum size on mobile

## Step 3 — Emit the audit report

```
# Guardian Audit — Shipments (TMS)
Audit run: [timestamp]

## Heuristics Agent
H1. [criterion] — PASS | WARN | FAIL | N/A
    Rationale: [one sentence]
... (H1–H10)

Summary: [n] load-bearing FAILs · [n] WARNs · [n] advisory FAILs
Recommendation: SHIP | SEND BACK | NEEDS HUMAN JUDGEMENT

## Accessibility Agent
A1. [criterion] — PASS | WARN | FAIL | N/A
    Rationale: [one sentence]
... (A1–A10)

Summary: [n] AA FAILs · [n] WARNs
Recommendation: SHIP | SEND BACK | NEEDS HUMAN JUDGEMENT

## Design System Agent (stub)
DS1–DS3 verdicts...

## Content Agent (stub)
C1–C3 verdicts...

## Interaction Agent (stub)
I1–I3 verdicts...

## Responsive Agent (stub)
R1–R3 verdicts...

---
## Overall Guardian Decision
Load-bearing FAILs: [n]
Advisory FAILs + WARNs: [n]
Recommendation: APPROVED FOR DEVELOPMENT | NEEDS UX TEAM INVOLVEMENT | BLOCKED — SEND BACK

[If BLOCKED or NEEDS UX TEAM: list each issue with a one-line suggested fix]
```

## Step 4 — Wait for my decision
Do not generate any HTML until I explicitly approve. My options:
- **Approve** → generate the full HTML page using Infios DS tokens, vanilla HTML/CSS/JS, no build step, matching the moonshot-prototype.html design language, with the dummy data above included
- **Fix and re-run** → I describe changes; you revise the spec and re-run the Guardian review from Step 2
- **Escalate** → I'll take it to the UX team; do not generate HTML
