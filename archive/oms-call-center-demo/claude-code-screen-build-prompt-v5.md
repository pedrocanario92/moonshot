# Claude Code Prompt: Carnival UK Call Center OMS Agent — Prototype v5

---

## Your Task

Build a high-fidelity, interactive, single-file HTML/CSS/JS prototype for the Carnival UK Call Center OMS Order Agent.

This prototype must demonstrate the **agent mesh behaviour** — not just the UI shell. The chat is alive: the agent interprets natural language, stages actions, surfaces contextual hints, and handles post-action work. All of this is simulated in the prototype using realistic scripted responses with appropriate timing.

Read this entire prompt before writing a single line of code.

---

## What Is New In This Version (vs v4)

v4 defined the visual structure. v5 adds the agent intelligence layer on top of it. Specifically:

1. **NL command → interpretation → staged action card** — when the operator types natural language, the agent shows a brief interpretation step before rendering the action card.
2. **Modification discount hint** — during modification flows only, after the action card renders, the agent checks for a better nearby deal and surfaces a single dismissible hint.
3. **Pre-call briefing** — in State 2 (Call Active), the agent's first message includes a structured briefing block with pending topics, loyalty notes, and any time-sensitive flags.
4. **Agent confidence indicator** — action cards show a small confidence badge showing how the agent interpreted the command.
5. **Post-action note draft** — in State 8 (Completed), the agent renders a draft note for operator review with a one-click save button.

Everything else (layout, panels, call bar, tab structure) is unchanged from v4. Build on top of it.

---

## Existing UI Problems (Do Not Replicate)

1. No active call indicator, timer, or call controls.
2. Permanent booking list always visible — noise during active work.
3. No customer interaction context.
4. Two action cards visible simultaneously — wrong-booking execution risk.
5. Static right panel — same content regardless of workflow.
6. No active booking lock.

---

## Visual Style (Unchanged From v4)

Floating cards on `--color-bg-page: #F0EFF4`. All panels: `border-radius: 12px`, `box-shadow: 0 2px 8px rgba(0,0,0,0.08), 0 0 1px rgba(0,0,0,0.10)`. 12px gap between panels and viewport edges.

---

## Layout (Unchanged From v4)

- Left panel: 320px fixed, floating card, three tabs (On Going / Bookings / Call)
- Floating call bar: dark pill above center chat, phone channel only
- Center agent workspace: fluid width, floating card
- Right panel: 320px, hidden by default, slides in on booking card click

---

## Agent Behaviour — How To Simulate It

The agent mesh is simulated as scripted responses with realistic timing. Use JavaScript `setTimeout` to sequence the steps. Every interaction that goes through the agent must follow the **Interpret → Stage → Approve** pattern.

### Pattern A — NL Command Flow

When the operator types free-form text and hits send:

```
Step 1 (immediate):   Operator message renders right-aligned in chat.
Step 2 (300ms later): Agent shows typing indicator — three animated dots.
Step 3 (800ms later): Agent shows interpretation confirmation:
                       "I understood: [action type] · [booking ID] · [change summary]"
                       Small confidence badge: "95% confident" in muted text.
Step 4 (400ms later): Action card renders inline in the chat stream.
Step 5 (600ms later): [Modification flows only] Discount hint renders below the card
                       if a better nearby date/time is available in the sample data.
```

### Pattern B — Chip / Quick Action Command

When the operator clicks a quick action button or chip:

```
Step 1 (immediate):   Operator command renders right-aligned as a chip bubble.
Step 2 (500ms later): Agent shows typing indicator.
Step 3 (600ms later): Action card renders directly — no interpretation step needed
                       (intent was explicit). Confidence badge shows "Chip command".
Step 4 (600ms later): [Modification flows only] Discount hint if applicable.
```

### The Typing Indicator

A small component — three animated dots in the agent bubble — that appears while the agent is "working". CSS keyframe pulse animation. Shows for the duration of the timeout before the next step renders.

### Confidence Badge

A small inline badge on the action card header:
- Green: `95–100%` — "High confidence"
- Amber: `75–94%` — "Review intent"
- Red: `<75%` — "Please confirm this is correct ↓"

For chip commands: gray badge — "Explicit command"

### Disambiguation (Sub-80% Confidence)

If the operator types something ambiguous (simulate this for one example in the prototype), the agent asks exactly one question instead of staging an action:

```
Agent: "I want to make sure I have this right — are you modifying the
        spa line item on BK-78431, or the experience on BK-78442?"
        [BK-78431 — Spa] [BK-78442 — Experience]   ← two inline buttons
```

Clicking either button resolves the ambiguity and continues to the staging step.

---

## Modification Discount Hint — Component Spec

This component renders below the staged modification action card. It fires **only** when:
- The active flow is `modify`
- The change includes a date or time slot

### Visual Design

```
┌─────────────────────────────────────────────────────────────┐
│ 💡  Heads up — Thursday 11 Jun is 15% cheaper               │
│     Same time slot (10:00) is available. Worth mentioning?  │
│                                                             │
│  [Mention to customer]          [Dismiss]                   │
└─────────────────────────────────────────────────────────────┘
```

Styling:
```css
background: #FFFBEB;           /* --color-warning-bg */
border: 1px solid #FDE68A;
border-left: 3px solid #B45309;
border-radius: 8px;
padding: 12px 16px;
font-size: 12px;
margin-top: 8px;
```

Behaviour:
- "Mention to customer" button: copies the text "I noticed Thursday 11 Jun has the same time slot available and it's 15% cheaper — would that work for you?" to clipboard. Button label changes to "✓ Copied" for 2 seconds, then resets.
- "Dismiss" button: removes the hint from the DOM. Does not re-appear for the rest of the session.
- The hint does NOT delay or block the modification action card. Both render; the hint is subordinate.

---

## Pre-Call Briefing Block — State 2 Agent Message

When State 2 loads (Call Active, customer identified), the agent's first message must include a structured briefing block before the action starters.

Agent message format:

```
Agent bubble:

"I found Grace Chen on Carnival Venezia sailing 10 Jun 2026.
BK-78431-mp1dzew3 is set as active. Here's what to know before you start:"

┌─────────────────────────────────────────────────────────────┐
│ PRE-CALL BRIEFING                                           │
│                                                             │
│ 🔵 Platinum member — qualifies for free date change         │
│    (no fee on rescheduling within 14 days of booking)       │
│                                                             │
│ ⚠  Open topic from last call: Refund query (03 May 2026)   │
│    Not yet resolved.                                        │
│                                                             │
│ ℹ  3 related bookings across 3 guests                      │
│    Ensure you confirm which booking before any action.      │
└─────────────────────────────────────────────────────────────┘

What would you like to do?
```

Briefing block styling:
```css
background: var(--color-primary-light);   /* #EDE9F7 */
border: 1px solid #C4B5E8;
border-radius: 8px;
padding: 14px 16px;
font-size: 12px;
margin: 8px 0;
```

Each briefing row: icon + bold label + description on a new line. Max 3 rows in the prototype.

---

## Post-Action Note Draft — State 8 Completed Card

After the completed action renders, the agent shows a note draft section at the bottom of the completed action card.

```
┌─────────────────────────────────────────────────────────────┐
│ ✓  BK-78440-mp1dzew3 has been cancelled.                   │
│    Confirmation ref: CNF-20260521-8821                      │
│    ✉ Confirmation email sent  ✓                            │
│    🗂 Audit event written     ✓                            │
│                                                             │
│ ─────────────────────────────────────────────────────────── │
│                                                             │
│ 📝  DRAFT NOTE — REVIEW BEFORE SAVING                      │
│                                                             │
│  ┌────────────────────────────────────────────────────────┐ │
│  │ 21 May 2026 · BK-78440-mp1dzew3 cancelled on          │ │
│  │ operator request. Guest: Scarlett White · White Group. │ │
│  │ Full refund processed: £77.96. Confirmation sent to    │ │
│  │ guest email. Operator: [auto-populated from session].  │ │
│  └────────────────────────────────────────────────────────┘ │
│                                                             │
│  [Edit note]    [Save to booking record ✓]                  │
│                                                             │
│ ─────────────────────────────────────────────────────────── │
│ CUSTOMER SUMMARY (read aloud)                               │
│ "Your booking BK-78440 has been cancelled and a full        │
│  refund of £77.96 has been processed. You'll receive a     │
│  confirmation email shortly."            [Copy ▸]          │
└─────────────────────────────────────────────────────────────┘
```

Behaviour:
- Draft note textarea is editable. Pre-populated with the scripted text.
- "Edit note" makes the textarea border visible and focuses it.
- "Save to booking record" button: shows a loading state (300ms), then changes to "✓ Saved" — green, non-interactive.
- "Copy ▸" on customer summary: copies the summary text. Label changes to "✓ Copied" for 2s.
- In State 8, the Call tab in the left panel is auto-selected and the call summary field shows a pre-populated summary matching the completed action.

---

## Left Panel — Three Tabs (Full Spec)

### Tab 1 — On Going (default when call active)

- Customer name, contact, verification badge, loyalty badge
- On Hold amber badge (when hold active)
- Customer need textarea (editable, 120 chars)
- Previous notes: 2 compact cards, View all link
- Pending topics: chips + Add topic
- Related bookings: max 5 cards, active booking highlighted with `⚡ Active`, others have `Set active` button
- Quick actions: 3 pre-populated buttons (Modify BK-78431 / Cancel BK-78440 / Look up BK-78442)
- Footer: Search / New booking / Add note / Escalate

### Tab 2 — Bookings

- Filter chips: All / Confirmed / On Hold / Cancelled / Spa / Exp
- Search input
- Booking list: 3 sample bookings + 5–7 synthetic rows (realistic Carnival UK names + amounts)
- Active booking has left border in `--color-primary`
- Clicking a row while an action is in progress: shows confirmation before switching active booking

### Tab 3 — Call

**When call active:**
- `● Active Call` status, customer name, duration (mirrors floating call bar timer)
- Summary field (editable): pre-populated in State 8 with the action summary
- `Add call note` button

**When no call:**
- `No active call` placeholder
- Call history list (5 entries from `CALL_HISTORY` sample data)

---

## Floating Call Bar (Unchanged From v4)

Dark pill (`#1C1C1E`). Phone channel only. Above center chat card.  
Contains: timer (green, monospace, `setInterval`), Hold toggle (`aria-pressed`), Mute toggle (`aria-pressed`), Keypad (DTMF modal), Contacts (dropdown), End call (red, confirmation).  
Hold: pauses timer, amber section background.  
End call: animates bar out, populates Call tab with session summary.

---

## Center Agent Workspace

### Header Bar

- Left: `⚡ Order Agent` + `● Connected to OMS · Carnival UK`
- Center: Active context pill — `⚡ BK-78431-MP1DZEW3 · GRACE CHEN` — updates on Set active
- Right: `New session` button

**Mismatch warning**: if action card targets a different booking than `activeBookingId`, render inline: `⚠ This action targets BK-XXXXX — not the active booking. Set it active first.`

### Chat Area Rules

- One action card open at a time. Second action collapses first to `↩ Resume [action]` bar.
- Booking cards in chat: `→ Click to edit` hover affordance, opens right panel slide-in.
- NL input: always follows Pattern A (Interpret → Stage).
- Chip input: always follows Pattern B (Stage directly).

### Input Bar

- Full-width, placeholder: `Ask the agent or type an action…`
- Send button
- Context-sensitive chips above bar (max 4)

**Sample NL commands to wire up (each triggers the full Interpret → Stage sequence):**

| Typed input | Interpreted as | Action card |
|---|---|---|
| "Move the spa to the 12th" | Modify BK-78431 · spa line item · date: 12 Jun | Modify card + discount hint |
| "Cancel Scarlett's booking" | Cancel BK-78440 · full booking | Cancel card |
| "What's in Grace's booking?" | Lookup BK-78431 | Lookup result card |
| "Add a note about the refund query" | Post-action note · custom text | Note draft card |

---

## Right Panel — On-Demand Slide-In (Unchanged From v4)

Hidden by default. Opens on booking card click. Floating card style. `✕` to close.  
**Modify form**: fields + before/after diff + availability indicator + staged confirm.  
**Cancel form**: read-only summary + refund breakdown + two-press confirm.  
**Blocked form**: reason + alternatives table + Escalate.

---

## Required Screen States

Dev toolbar at top. All 8 states accessible without typing.

### State 1 — Empty / New Interaction
- Call bar: hidden
- Left On Going: channel selector, empty fields
- Center: empty state — icon + title + 5 generic action starters
- Right: hidden

### State 2 — Active Call, Customer Identified ★ NEW BEHAVIOUR
- Call bar: visible, timer counting from 00:00
- Left On Going: Grace Chen, Verified ✓, Platinum, customer need pre-filled, 2 notes, "Refund query" topic, 3 related bookings (BK-78431 active), quick actions
- Center: **Pre-call briefing block** + agent greeting + 5 action starters
- Right: hidden

### State 3 — Booking Lookup
- Center: agent message + lookup result card for BK-78431 with `→ Click to edit` affordance
- Right: hidden (opens on click)

### State 4 — Modify Booking ★ NEW BEHAVIOUR
- Center: simulate NL input "Move the spa to the 12th" →
  - Operator message bubble
  - Typing indicator (300ms)
  - Interpretation message: "I understood: modify spa line item on BK-78431 · from 11 Jun → 12 Jun · 95% confident"
  - Modification action card (collapsed in chat) with confidence badge
  - **Discount hint**: "💡 Thursday 11 Jun is 15% cheaper — same time slot available"
- Right: slide-in open — modify form for BK-78431

### State 5 — Cancel Booking
- Center: collapsed cancel summary bar in chat
- Right: slide-in open — cancel form for BK-78440, refund shown, two-press confirm

### State 6 — Blocked Action
- Center: agent message "⚠ Sold out — 11 Jun 2026 · 14:00"
- Right: auto-opens — blocked form with alternatives table

### State 7 — New Booking (Email Channel)
- Call bar: hidden
- Left On Going: Grace Chen, no active booking
- Center: new booking card
- Right: hidden

### State 8 — Completed Action ★ NEW BEHAVIOUR
- Call bar: still visible (call ongoing)
- Left: tab stays on **On Going** (do not auto-switch). Call tab summary is silently pre-populated in the background — operator can navigate there manually to see it.
- Center: completed action card with: confirmation ref, email ✓, audit ✓, **draft note for review**, customer-facing summary with Copy button
- Right: hidden

---

## Tech Stack

- Single `.html` file, no build step, no framework
- Vanilla HTML5 + CSS3 + vanilla JS
- CSS custom properties
- Lucide icons via CDN: `https://unpkg.com/lucide@latest`
- No external CSS frameworks

---

## Sample Data

```js
const SESSION = {
  channel: "phone",
  customer: {
    name: "Grace Chen",
    phone: "+44 7700 900123",
    email: "grace.chen@email.com",
    loyalty: "Platinum",
    verified: true
  },
  activeBookingId: "BK-78431-mp1dzew3",
  customerNeed: "Modify spa time, cancel mug order",
  pendingTopics: ["Refund query"],
  previousNotes: [
    { date: "03 May 2026", text: "Called re: cabin upgrade. Declined upgrade, happy with Deck 11." },
    { date: "28 Apr 2026", text: "Booked spa package. Loyalty discount applied." }
  ]
};

const BOOKINGS = [
  {
    id: "BK-78431-mp1dzew3", ref: "c9fededa-d556-4b6c-a8d5-0bf115dd4f9c",
    status: "CONFIRMED", channel: "eCommerce", bookedOn: "06 May 2026",
    guest: "Grace Chen", loyalty: "Platinum", vessel: "Carnival Venezia",
    sailDate: "10 Jun 2026", port: "Southampton", cabin: "Cabin 5050", deck: "Lido Deck",
    lineItems: [{ type: "Spa", name: "Oversized Mug Package", date: "11 Jun 2026", time: "10:00", guests: 3, price: 22.99 }],
    services: 22.99, serviceCharge: 0.00, total: 22.99
  },
  {
    id: "BK-78442-mp1dzew3", status: "CONFIRMED", guest: "Natalie Harris",
    deck: "Deck 11", vessel: "Carnival Venezia", sailDate: "10 Jun 2026",
    cabin: "Cabin 5030", venue: "Deck 11 — Havana Bar",
    lineItems: [
      { type: "Spa", name: "Color Crush 20oz Mug — Cream", date: "11 Jun 2026", time: "10:00", guests: 1, price: 0.00 },
      { type: "Spa", name: "Oversized Mug Package", date: "12 Jun 2026", time: "10:00", guests: 3, price: 0.00 }
    ], total: 70.96
  },
  {
    id: "BK-78440-mp1dzew3", status: "CONFIRMED", guest: "Scarlett White · White Group",
    deck: "Deck 9", vessel: "Carnival Venezia", sailDate: "10 Jun 2026",
    lineItems: [{ type: "Spa", name: "Havana Bar Package", date: "10 Jun 2026", time: "14:00", guests: 2, price: 77.96 }],
    total: 77.96
  }
];

// Discount hint data — used in State 4 modification flow
const DISCOUNT_HINT = {
  currentDate: "12 Jun 2026",
  betterDate: "11 Jun 2026",
  saving: "15%",
  slot: "10:00",
  available: true,
  customerLine: "I noticed Thursday 11 Jun has the same time slot available and it's 15% cheaper — would that work for you?"
};

const CALL_HISTORY = [
  { date: "03 May 2026", customer: "Grace Chen", duration: "4:22", outcome: "Resolved" },
  { date: "14 Apr 2026", customer: "Natalie Harris", duration: "7:01", outcome: "Escalated" },
  { date: "02 Apr 2026", customer: "Priya Patel", duration: "2:45", outcome: "Resolved" },
  { date: "28 Mar 2026", customer: "Oliver Clarke", duration: "9:15", outcome: "Callback" },
  { date: "15 Mar 2026", customer: "Sarah Thompson", duration: "3:30", outcome: "Resolved" }
];

const BLOCKED_SLOT = {
  item: "Color Crush Massage", date: "11 Jun 2026", requestedTime: "14:00",
  alternatives: [
    { time: "09:00", available: true }, { time: "10:00", available: true },
    { time: "11:00", waitlist: true }, { time: "14:00", available: false }
  ]
};
```

---

## Design Tokens

```css
:root {
  --color-bg-page: #F0EFF4;
  --color-primary: #6B3FA0;
  --color-primary-light: #EDE9F7;
  --color-primary-dark: #4A2B72;
  --color-success: #1A7F4B;
  --color-success-bg: #ECFDF5;
  --color-warning: #B45309;
  --color-warning-bg: #FFFBEB;
  --color-danger: #C0392B;
  --color-danger-bg: #FEF2F0;
  --color-surface: #FFFFFF;
  --color-surface-2: #F8F8FA;
  --color-border: #E4E4E7;
  --color-text: #18181B;
  --color-text-muted: #71717A;
  --color-text-faint: #A1A1AA;
  --color-call-bar-bg: #1C1C1E;
  --color-call-timer: #34C759;
  --panel-left: 320px;
  --panel-right: 320px;
  --panel-radius: 12px;
  --panel-shadow: 0 2px 8px rgba(0,0,0,0.08), 0 0 1px rgba(0,0,0,0.10);
  --panel-gap: 12px;
  --font-base: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
  --font-mono: 'SF Mono', 'Fira Code', monospace;
}
```

---

## Component Checklist

### Layout & Structure
- [ ] `PageLayout` — bg page, flex row, panel gaps, all panels as floating cards
- [ ] `StateDevToolbar` — tab strip for 8 states, top of page
- [ ] `LeftPanel` — 320px card, tab strip, content switcher
- [ ] `AgentWorkspace` — fluid card, header pill, chat area, input bar, chips
- [ ] `RightPanel` — fixed, slide-in, floating card style, `✕` close, focus trap

### Left Panel Tabs
- [ ] `OnGoingTab` — full customer context, related bookings, quick actions, footer
- [ ] `BookingsTab` — filter chips, search, paginated list with active booking highlight
- [ ] `CallTab` — active call summary + history list

### Agent Behaviour Components
- [ ] `TypingIndicator` — animated three dots, shown during agent "processing"
- [ ] `InterpretationMessage` — "I understood: [intent] · [confidence badge]"
- [ ] `ConfidenceBadge` — green/amber/red/gray pill based on confidence level
- [ ] `DisambiguationCard` — one clarifying question + two inline booking buttons
- [ ] `PreCallBriefingBlock` — styled block in State 2 first message
- [ ] `DiscountHintCard` — amber bordered hint with Mention/Dismiss buttons, modification flows only
- [ ] `DraftNoteCard` — editable textarea + Save button in State 8 completed card

### Action Cards
- [ ] `ActionCard` — base container, booking ID header, one-open-at-a-time rule
- [ ] `LookupResultCard` — booking metadata, line items, Click-to-edit affordance
- [ ] `ModifyActionCard` — in-chat collapsed summary
- [ ] `CancelActionCard` — in-chat collapsed summary
- [ ] `CompletedActionCard` — ref, status rows, draft note, customer summary + copy

### Right Panel Forms
- [ ] `ModifyForm` — fields, before/after diff, availability, staged confirm
- [ ] `CancelForm` — read-only summary, refund, two-press confirm
- [ ] `BlockedForm` — reason, alternatives, escalate

### Call Controls
- [ ] `FloatingCallBar` — dark pill, timer setInterval, Hold/Mute toggles, Keypad modal, End call
- [ ] `RelatedBookingCard` — compact, active badge, Set active button
- [ ] `QuickActionButtons` — pre-populated from SESSION, fire commands into chat
- [ ] `ActiveBookingPill` — updates on Set active, mismatch warning

---

## Critical Guardrails

1. **One active booking** — `SESSION.activeBookingId` is single source of truth. All agent responses reference it.
2. **One action card at a time** — collapse existing before rendering new.
3. **Call bar: phone + active only** — not shown for email, chat, case, or back-office.
4. **Call timer** — starts State 2, pauses on Hold, resets on End call. Call tab mirrors timer.
5. **Confirm disabled until preview** — modify confirm is `disabled` until Preview is triggered.
6. **Two-press cancel** — first press changes label + style; second press within 4s executes.
7. **Mismatch warning** — action card targeting non-active booking shows inline warning.
8. **Discount hint: modification only** — never fires on cancel, lookup, new booking, or guest count change.
9. **Note never auto-saves** — "Save to booking record" requires explicit operator click.
10. **Interpretation always precedes staging on NL input** — no action card renders without the interpretation step first.
11. **Left panel tab never auto-switches** — no agent action, save, edit, confirmation, or state change may programmatically change the active left panel tab. Tab state is operator-controlled only. The Call tab summary is updated in the background when relevant, but the operator navigates there manually.

---

## Accessibility

- All inputs: `<label for>` + `aria-describedby`
- Status badges: `aria-label`, never color-only
- Hold / Mute: `aria-pressed`
- Right panel: focus trap, `Escape` closes with unsaved-change guard
- Active context pill: `aria-live="polite"`
- Call timer: `aria-live="off"`
- Confidence badge: `aria-label="Agent confidence: 95 percent"`
- Discount hint dismiss: `aria-label="Dismiss deal hint"`
- Typing indicator: `aria-label="Agent is processing"` + `role="status"`
- `:focus-visible` ring (2px `--color-primary`) on all interactive elements

---

## Output

Single file: `call-center-oms-agent.html`

Must work:
- All 8 states via dev toolbar
- NL input in State 4 triggers full Interpret → Stage → Hint sequence with correct timing
- Typing indicator visible during agent processing delays
- Confidence badge renders on action card
- Discount hint renders in State 4, dismisses correctly, copies to clipboard
- Pre-call briefing block renders in State 2
- Draft note renders in State 8, is editable, save button changes state
- Call tab auto-selects in State 8 with pre-populated summary
- Call bar timer functional, Hold/Mute states functional
- Left panel tab switching functional
- Right panel slide-in functional
- Set active updates header pill + quick actions
- Floating card visual style applied consistently
- No placeholder sections — full working prototype
