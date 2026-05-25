# Carnival UK Call Center OMS — Agent Strategy & Architecture Context

**Status**: Design exploration / Pre-build  
**Last updated**: May 2026  
**Scope**: Call center operator tooling for spa and experience booking management

---

## 1. Product Vision

Build a semi-agentic AI workspace for Carnival UK call center operators that helps them handle customer interactions faster, safer, and with greater operational confidence.

The agent is not a chatbot. It is an operational co-pilot: it interprets intent, stages actions for human approval, surfaces contextual opportunities at the right moment, and handles post-action work so operators can stay focused on the customer.

**The core mental shift:**

> Start from the customer interaction. Not from a booking list.

A booking is an object inside an interaction. The interface anchors on the customer — their channel, context, need, and history — and attaches booking operations to that anchor.

---

## 2. Problems Being Solved

### 2.1 No Interaction Context
Current booking-first layouts show records but don't answer: who is contacting us, through which channel, have they been verified, what do they need, are there unresolved topics from previous calls?

### 2.2 Permanent Booking List Creates Noise
During active modification or cancellation work, 23 unrelated bookings on screen increase cognitive load and the risk of acting on the wrong record.

### 2.3 Static Right Panel Wastes Decision Space
Showing only voyage metadata when the operator needs availability, refund impact, policy rules, or deal comparisons is a missed opportunity.

### 2.4 Chat as Display Only
A chat surface that only renders pre-defined cards doesn't allow operators to express intent naturally. Experienced operators should be able to type "move spa to the 12th" and have the agent interpret, validate, and stage the action.

### 2.5 No Proactive Intelligence
Operators don't have time to manually check whether a better deal exists, whether a loyalty discount applies, or whether a cancellation window is closing. The agent should surface these at the right moment — not through a script or continuous prompting, but as a single contextual hint when it's relevant.

### 2.6 Wrong-Booking Execution Risk
Without a clear active booking lock, an operator can confirm an action on the wrong booking. This is a costly operational error with customer trust and financial implications.

---

## 3. UI Architecture Decisions

### 3.1 Layout: Three-Zone Floating Card Model

```
┌─────────────────────────────────────────────────────────────────┐
│  LEFT PANEL (320px)  │  CENTER: AGENT WORKSPACE  │  RIGHT PANEL │
│  Floating card       │  Floating card            │  (320px)     │
│                      │                           │  Hidden by   │
│  Tab strip:          │  ┌─ Floating Call Bar ─┐  │  default.    │
│  On Going            │  │ Dark pill, above     │  │  Slides in   │
│  Bookings            │  │ chat. Phone only.    │  │  on booking  │
│  Call                │  └──────────────────────┘  │  card click. │
│                      │  ┌─ Agent Workspace ────┐  │              │
│                      │  │ Header + chat +      │  │  Focused     │
│                      │  │ input bar + chips    │  │  edit form   │
│                      │  └──────────────────────┘  │  only.       │
└─────────────────────────────────────────────────────────────────┘
```

All panels are floating cards: `border-radius: 12px`, `box-shadow`, page background `#F0EFF4`. Panels do not flush to viewport edges.

### 3.2 Left Panel — Three Tabs

| Tab | Content | When active |
|---|---|---|
| **On Going** | Customer identity, verification, loyalty, customer need, previous notes, pending topics, related bookings (max 5), quick action buttons, footer actions | Default when call is active |
| **Bookings** | Full operator booking queue with filter chips, search, pagination | When operator needs to browse or switch context |
| **Call** | Active call summary (mirrors timer), key points field, call history | When operator wants call record or history |

### 3.3 Floating Call Bar

Appears above the center chat card. Phone channel only. Dark pill (`#1C1C1E`). Contains: timer (counting up, monospace, green), Hold toggle, Mute toggle, Keypad, Contacts, End call (red, with confirmation). Animates in/out with call state. Pauses timer on Hold.

### 3.4 Right Panel — On-Demand Slide-In

Hidden by default. Slides in (`translateX` transition, 240ms) when a booking card in the chat is clicked. Shows a focused edit or cancel form for that booking only — no tabs, no extra context. Closes on completion or `✕`. Auto-opens on Blocked state (operator needs alternatives immediately).

### 3.5 Active Booking Lock

`SESSION.activeBookingId` is the single source of truth. All action cards, the center header pill, and quick action buttons derive from it. If an action card targets a different booking, an inline mismatch warning fires. Switching the active booking requires a deliberate UI action.

---

## 4. Agent Architecture

### 4.1 The Core Pattern

The agent mesh has four specialized agents coordinated by a Session Controller. The chat surface is the input/output layer. The real work happens in the mesh.

```
Operator Input (NL or chip)
        │
        ▼
┌──────────────────────────────────────────────────────────┐
│               SESSION CONTROLLER (Coordinator)           │
│  Holds session state. Routes to agents. Controls         │
│  timing of opportunity hints. Enforces booking lock.     │
│  Never executes OMS actions directly.                    │
└──────┬──────────────────┬──────────────────┬─────────────┘
       │                  │                  │
       ▼                  ▼                  ▼
┌────────────┐   ┌────────────────┐   ┌──────────────────┐
│  Intent    │   │  Action        │   │  Opportunity     │
│  Interpreter│   │  Staging Agent │   │  Scanner         │
│            │   │                │   │                  │
│  NL → typed│   │  Validates OMS │   │  Event-triggered │
│  intent +  │   │  rules, checks │   │  only. Fires on  │
│  confidence│   │  availability, │   │  modification    │
│  score.    │   │  calculates    │   │  flows. Returns  │
│  Disambig- │   │  price diff.   │   │  max 1 hint.     │
│  uates if  │   │  Builds diff   │   │  Never during    │
│  <80% conf.│   │  card for      │   │  cancel/lookup.  │
└──────┬─────┘   │  approval.     │   └──────────────────┘
       │         └───────┬────────┘
       │                 │
       ▼                 ▼
       Human Approval Checkpoint
       (Before/after diff — explicit confirm required)
                         │
                         ▼
                   OMS Execution
                         │
                         ▼
              ┌──────────────────┐
              │  Post-Action     │
              │  Agent           │
              │                  │
              │  Auto-drafts     │
              │  note. Populates │
              │  Call tab. Preps │
              │  audit trail.    │
              │  Operator        │
              │  reviews +       │
              │  one-click save. │
              └──────────────────┘
```

### 4.2 The Four Agents

#### Agent 1 — Intent Interpreter
**Job**: Turn operator NL input into a typed, structured intent with extracted parameters.  
**Input**: Raw text from the chat input bar.  
**Output**: `{ type: "modify", bookingId: "BK-78431", entity: "spa-line-item", fromDate: "11 Jun", toDate: "12 Jun", confidence: 0.92 }`  
**Constraint**: Must respond in under 1 second (live call). Lean prompt, no OMS calls.  
**Disambiguation rule**: If confidence < 0.80, ask exactly one clarifying question. Never ask two.  
**Fallback**: If NL fails or confidence is very low, surface the relevant structured action chip.

#### Agent 2 — Action Staging Agent
**Job**: Take structured intent, validate against OMS rules, check availability, calculate price diff or refund impact, build a before/after diff card for operator review.  
**Input**: Structured intent from Agent 1.  
**Output**: A staged action card (modify diff, cancel summary, or blocked explanation).  
**Latency budget**: 2–3 seconds acceptable (agent is "checking availability" — visible to operator).  
**Constraint**: Never executes. Always stages. Confirm button is disabled until operator reviews the diff.

#### Agent 3 — Opportunity Scanner
**Job**: Detect better deals or relevant options adjacent to what the customer is requesting.  
**Trigger**: Modification flows only. Fires when Action Staging Agent receives a date or time change request.  
**Output**: Maximum one hint per trigger event. Never a list. Structured as: `{ type: "discount-hint", message: "Thursday 11 Jun is 15% cheaper — same time slot is available.", action: "offer-alternative" }`  
**Delivery**: Ambient, non-blocking. Appears below the staged action card as a dismissible hint. Does not interrupt or delay the main action.  
**Fires on**: Date change, time slot change.  
**Does not fire on**: Cancellation flows, lookup flows, new booking flows, guest count changes.

#### Agent 4 — Post-Action Agent
**Job**: After OMS execution, auto-draft a note, populate the Call tab summary, and prepare the audit trail entry.  
**Input**: Full execution record from OMS confirmation.  
**Output**: Draft note (formatted for booking record), customer-facing summary (readable aloud by operator), audit event object.  
**Constraint**: Operator always reviews before saving. Agent never auto-saves to OMS. One-click save after review.  
**Latency budget**: No constraint. Fires after confirmation, not during.

### 4.3 Session Controller

The coordinator is the only component with full session state. It:
- Maintains `activeBookingId`, `channel`, `callState`, `currentFlow`, and `conversationHistory`
- Routes incoming input to the correct agent based on context
- Decides whether to surface an opportunity hint based on `currentFlow === "modify"`
- Enforces the one-action-card-at-a-time rule
- Handles the disambiguation loop (passes Agent 1's clarifying question to the UI, waits for operator response, re-routes)
- Never executes OMS actions

---

## 5. Agent Team vs. Single Agent — The Decision

### Why Agent Team for Production

| Dimension | Single Agent | Agent Team |
|---|---|---|
| **Latency control** | All tasks compete for the same response time | Intent Interpreter can be sub-1s; Action Staging runs async |
| **Context size** | One growing context window — degrades with scale | Each agent has a focused, minimal context |
| **Tuning** | Changing one capability risks breaking others | Each agent is independently tunable |
| **Parallelism** | Sequential by default | Opportunity Scanner can run async alongside Action Staging |
| **Failure handling** | One failure point | Coordinator catches individual agent failures and degrades gracefully |
| **Organizational scaling** | One team owns everything | Different teams can own different agents |

The specific constraint that forces the team model here is **latency asymmetry**: Intent Interpretation must be sub-second (operator is on a live call), Action Staging can take 2–3 seconds (checking availability is expected), and the Opportunity Scanner can be fully async (result surfaces when ready, doesn't block the main flow). A single agent cannot serve these three different latency budgets simultaneously.

### MVP Path: Start Single, Extract Gradually

For the prototype and early development, simulate the entire mesh as a single agent with the coordinator's routing logic baked into the response patterns. When you hit real constraints:
1. Extract the Intent Interpreter first (it's the most latency-sensitive)
2. Extract the Opportunity Scanner second (it's the most independently testable)
3. The Action Staging + Post-Action agents can remain together longer

---

## 6. Opportunity Scanner — Detailed Model

### Trigger Conditions (Modification Flows Only)

| Event | Check | Surface if |
|---|---|---|
| Operator requests date change | Check ±3 days for pricing delta | Saving > 10% or loyalty discount unlocks |
| Operator requests time slot change | Check adjacent slots for pricing | Different tier pricing applies |
| Guest count reduced | Check if bundle pricing breaks | Per-person cost increases post-change |

### Hint Design Rules

1. **Maximum one hint per trigger event** — never stack multiple hints
2. **Non-blocking** — hint appears below the action card, does not delay staging
3. **Dismissible** — one click removes it from the session (not re-surfaced)
4. **Two actions only**: "Mention to customer" (copies customer-facing line) / "Dismiss"
5. **Factual, not persuasive** — "Thursday 11 Jun is 15% cheaper" not "Great deal available!"
6. **No hint during cancellation** — wrong emotional context, damages trust

### What the Scanner Does Not Do

- Does not run continuously or on a timer
- Does not surface upsell suggestions unprompted
- Does not fire during cancellation, lookup, or new booking flows
- Does not surface more than one hint at a time
- Does not auto-apply any deal — operator always initiates

---

## 7. Human-in-the-Loop Principles

These are non-negotiable across all agent capabilities:

1. **No auto-execution** — every OMS write action requires explicit operator confirmation. Always.
2. **Staged diff before confirm** — every modification and cancellation shows a before/after state before the confirm button is enabled.
3. **Financial impact visible before confirm** — price difference, refund amount, and service charge changes must be rendered before the operator can confirm.
4. **One active booking at a time** — the system enforces an execution lock. Actions cannot target a booking other than `activeBookingId` without an explicit switch.
5. **Operator override is always available** — the operator can dismiss hints, collapse action cards, and type free-form commands at any time.
6. **Agent expresses uncertainty** — when confidence is below threshold, the agent asks rather than assumes. It never silently picks the most likely interpretation.
7. **Post-action review** — auto-drafted notes and summaries are always presented for review. Nothing is written to OMS or booking records without operator action.

---

## 8. What to Build vs. Skip

| Capability | Decision | Reason |
|---|---|---|
| NL command → staged action card | **Build — Core** | Saves 4–8 sec per action; serves experienced operators |
| Contextual discount hint (modification only) | **Build — Core** | Direct revenue recovery; scoped trigger reduces noise risk |
| Pre-call briefing (State 2 load) | **Build — Core** | Sets operator before first word; low complexity, high value |
| Auto-draft post-action note | **Build — Core** | Saves 1–2 min after-call; deterministic, low risk |
| Continuous background scanning | **Skip** | Latency, cost, alert fatigue — no upside over event triggers |
| Upsell suggestions during cancellation | **Skip** | Wrong context; damages customer trust and operator confidence |
| Agent auto-saving notes or records | **Skip** | Human review is non-negotiable; auto-save removes audit integrity |
| Proactive upsell outside modification | **Phase 3** | Requires customer sentiment model; too complex for Phase 1 |

---

## 9. Scorecard — Metrics Worth Tracking

| Metric | Target | Signal |
|---|---|---|
| Average Handling Time (AHT) | −20% vs. baseline | NL shortcuts + auto-notes remove manual steps |
| First Contact Resolution (FCR) | +12% vs. baseline | Pre-call briefing catches unresolved topics |
| Opportunity hint acceptance rate | >35% | Below 20% = hints too generic; above 50% = triggers too conservative |
| Note completion rate | >90% | From ~40–60% manual; auto-draft + one-click raises floor |
| Wrong-booking execution errors | 0 | Hard zero; active booking lock is the mechanism |
| Operator trust score (survey) | Track monthly | Drives adoption; adoption drives ROI |
| Intent interpretation accuracy | >85% correct staging on first attempt | Below this, NL adds friction instead of removing it |
| Time-to-staged-action (NL path) | <3 seconds | Above this, experienced operators revert to chips |

---

## 10. Key Design Decisions & Rationale

**Why the left panel gets three tabs instead of a permanent booking list:**  
The permanent list creates noise during active work. The three-tab model gives operators access to the full queue (Bookings tab) without surfacing it during a live modification. The On Going tab keeps the operator anchored to the customer context, not the record system.

**Why the call bar floats above the chat instead of living in the left panel:**  
Call controls need to be visible regardless of which left panel tab is active. An operator on the Bookings tab switching a queue item mid-call still needs the timer and hold button. Floating above the chat makes it persistent without consuming panel space.

**Why the right panel is hidden by default:**  
The edit form is only needed when the operator is actively modifying or reviewing a specific booking. Showing it always would narrow the chat area and surface irrelevant form fields during lookup and cancellation flows where they add no value.

**Why the Opportunity Scanner fires only on modification flows:**  
Discount hints during a cancellation call would feel tone-deaf. The customer is trying to leave; surfacing a deal is wrong timing and can damage trust. During a modification, the customer has already expressed willingness to engage — a deal on a nearby date is genuinely useful information.

**Why the agent team model instead of a single agent:**  
Latency asymmetry: Intent Interpretation must be sub-second on a live call. Action Staging needs 2–3 seconds to query OMS. The Opportunity Scanner can be fully async. A single agent cannot serve these three different latency budgets without degrading the most time-sensitive one.

---

## 11. Prototype Simulation Notes

In the prototype, the agent mesh is simulated as a single agent whose response patterns encode the coordinator's routing logic. Specifically:

- NL input triggers a brief "Interpreting…" state (300ms delay) before the staged action card renders
- The staged card shows a confidence indicator on the intent interpretation
- Modification flows with a date change trigger the discount hint component after the card renders
- State 2 (Call Active) renders a pre-call briefing block in the agent's first message
- State 8 (Completed) renders an auto-drafted note for operator review

These simulations demonstrate the agent team's behaviour without requiring a real multi-agent backend. The prototype is the design surface for validating the interaction model before committing to the infrastructure.
