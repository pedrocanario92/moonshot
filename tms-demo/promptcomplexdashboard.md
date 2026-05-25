# TM Operations Dashboard
## Guardian-first build prompt · TMS · Complex page

The TM Operations Dashboard is a command-centre view for a transportation manager starting their shift. It tries to show everything at once — live KPIs across the network, a map of active shipments, an exceptions feed, a carrier performance table, cost trend sparklines, and a quick-actions toolbar. The intent is good: one screen, full visibility. The problem is it violates almost every cognitive load principle the Guardian agents test for — too many elements above the fold, too many competing focal points, too many primary actions, and decorative data widgets that look useful but drive no decisions.

---

You are building a UI page for the Moonshot project (supply chain, Infios DS).

Before writing any HTML, you must run a Guardian review on the proposed design.

## The page
**Name:** TM Operations Dashboard
**Pillar:** TMS (Transportation Management)
**Purpose:** Give a transportation manager a single-screen view of network health at shift start — active shipments, exceptions, carrier performance, cost variance, and quick actions.

**Layout:** Full-width dashboard with 5 zones:
1. Top KPI bar — 6 live metrics displayed as tiles across the full width
2. Map panel (left, 60% width) — live map of active shipment routes with carrier pins
3. Exceptions feed (right, 40% width) — real-time list of delayed/at-risk shipments with inline Approve / Dismiss actions per row
4. Carrier performance table (full width, below map) — carrier name, on-time %, cost vs. contracted, active lanes, trend sparkline per row
5. Cost trend chart (full width, bottom) — 30-day rolling cost vs. budget line chart with annotations

**Toolbar (persistent, above KPI bar):** New Shipment · Approve All · Export · Filter · Sort · View Report · Configure

**KPI tiles (always visible above the fold):**
- Total active shipments: 142
- On-time %: 81%
- Delayed: 27
- Cost variance: +$14,200
- Carrier performance score: 74/100
- Network utilisation: 68%

**Exceptions feed (sample rows):**

| Shipment | Route | Issue | ETA slip | Actions |
|---|---|---|---|---|
| TMS-4798 | Dallas → LA | Carrier delay | +2 days | Approve reroute · Dismiss |
| TMS-4801 | Seattle → Portland | Weather hold | +1 day | Approve reroute · Dismiss |
| TMS-4815 | Memphis → Nashville | Capacity issue | +1 day | Approve reroute · Dismiss |
| TMS-4833 | Miami → Atlanta | Port congestion | +3 days | Approve reroute · Dismiss |
| TMS-4840 | Chicago → Detroit | Driver HOS | +1 day | Approve reroute · Dismiss |

**Carrier performance table (sample rows):**

| Carrier | On-time % | Cost vs. contracted | Active lanes | Trend |
|---|---|---|---|---|
| FedEx Freight | 88% | +2% | 12 | ↑ sparkline |
| XPO Logistics | 74% | +8% | 9 | ↓ sparkline |
| ABF Freight | 91% | -1% | 7 | → sparkline |
| Old Dominion | 83% | +4% | 11 | ↓ sparkline |
| UPS Freight | 78% | +6% | 8 | ↓ sparkline |

**Persistent decorative elements:**
- Each KPI tile has a delta indicator (▲/▼ vs. yesterday) and a mini sparkline
- The map has animated carrier pins that pulse when a shipment is delayed
- The cost chart has a "live" label that refreshes every 60 seconds with a subtle flash

**States:**
- Default: all 5 zones visible simultaneously above and below fold
- Exception selected: inline Approve / Dismiss appear alongside 3 other persistent actions in the toolbar
- No calm state — the dashboard always shows all data

---

## Step 1 — Derive the design spec
From the page definition above, produce a concise design spec:
- Interactive elements count above the fold
- Actions available at each state
- Persistent decorative or informational elements
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
# Guardian Audit — TM Operations Dashboard
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
