# Claude Code Prompt: Call Center OMS Agent Screens

You are building a high-fidelity product prototype for a Carnival UK Call Center OMS Order Agent.

The product is a semi-agentic AI assistant used by call center operators while handling customer interactions by phone, email, chat, case, or back-office request. The goal is to help operators quickly understand the customer context, find or create bookings, modify bookings, cancel bookings, explain blocked actions, and execute OMS actions only after explicit confirmation.

## Product Direction

Do not build a generic chatbot page. Build an operational call center workspace.

Follow this implementation plan:

1. Build the three-zone layout: Interaction Panel, Agent Workspace, Dynamic Context Panel.
2. Create representative state data for customer, interaction, related bookings, notes, rules, and OMS actions.
3. Implement screen states for lookup, modification, cancellation, blocked action, new booking, and completed action.
4. Make the active customer and active execution booking impossible to miss.
5. Add functional controls for switching states, selecting related bookings, staging changes, previewing impact, confirming execution, and viewing completion.
6. Add accessibility details: labels, focus states, keyboard-friendly controls, semantic status text, and non-color-only feedback.
7. Polish the density, hierarchy, and operational clarity of the UI.

The primary mental model is:

```text
Interaction Panel + Agent Workspace + Dynamic Context Panel
```

The interface should start from the customer interaction, not from a permanent booking list.

## Core Layout

### Left Panel: Interaction Panel

Replace the permanent booking list with a customer/contact/session panel.

Include:

- Interaction source: phone call, email, chat, case, or back-office request.
- Customer identity.
- Contact details.
- Verification status.
- Current customer need.
- Previous notes.
- Pending topics.
- Related bookings.
- Search booking/customer action.
- Start new booking action.
- Add note action.
- Escalate action.

Important:

- Related bookings should be compact.
- Do not show a long unrelated booking list by default.
- A booking list can appear as a search result, menu, drawer, or queue mode when needed.

### Center: Agent Workspace

This is the main work area.

Include:

- Header showing active Order Agent, OMS connection status, and current active context.
- New session / reset session control.
- AI conversation area.
- Suggested action starters.
- Structured action cards for operational workflows.
- Input bar with command suggestions.

The center should support these workflows:

- Look up existing booking.
- Start a new booking with no booking ID.
- Modify a booking.
- Cancel a full booking.
- Cancel a spa line item.
- Cancel an experience line item.
- Explain why an action is blocked, such as sold-out day.
- Preview impact before confirmation.
- Execute after operator confirmation.

### Right Panel: Dynamic Context / Impact / Rules Panel

Do not make this a static booking details panel only.

It should change based on the selected workflow.

Examples:

- During lookup: customer profile, verification, related bookings, previous notes.
- During modification: current state, requested change, availability, price difference, policy checks.
- During cancellation: cancellation scope, refund amount, fees, email confirmation, audit note.
- During new booking: availability, basket, payment status, required fields.
- During blocked action: reason, policy rule, alternatives, escalation option.

## Required Screens Or States

Build the prototype with at least these states:

1. Empty/new interaction state.
2. Active call with identified customer and previous notes.
3. Existing booking lookup result.
4. Modify booking draft with before/after preview.
5. Cancel booking confirmation with refund/fee impact.
6. Blocked action state, such as requested day sold out.
7. New booking flow where no booking ID exists.
8. Completed action state with confirmation email, note update, and audit event.

## Agent Behavior Guidelines

The Order Agent should:

- Keep the active customer clear.
- Keep the active execution booking clear.
- Show when the operator is working across related bookings.
- Ask for missing information before staging an action.
- Check availability, pricing, refund impact, and business rules.
- Prepare structured drafts.
- Explain blockers and suggest alternatives.
- Require explicit operator confirmation before OMS execution.
- Show final result after OMS execution.
- Offer to send confirmation email.
- Offer or automatically draft a note.
- Write an audit trail after confirmed action.
- Provide a customer-facing summary after execution.

The Order Agent should not:

- Execute any OMS action without explicit confirmation.
- Mix booking context across unrelated records.
- Hide fees, refund impact, or policy restrictions.
- Present sold-out or unavailable options as valid.
- Force every workflow to start from a booking ID.

## UX Principles

- Interaction first: customer/channel/context before booking operations.
- One active execution context at a time.
- Chat for conversation, structured UI for action.
- Dynamic context over static details.
- Preview before commit.
- Explain why an action is blocked or fee-bearing.
- Reduce cognitive load during active calls.
- Keep common actions fast and visible.
- Make follow-up work easy after execution.

## Functionality Principles

The prototype should make these capabilities visible:

- Customer search.
- Booking search.
- Related booking selection.
- New booking start.
- Existing booking modification.
- Full and partial cancellation.
- Availability check.
- Refund and price difference preview.
- Policy and eligibility explanation.
- Confirmation email step.
- Notes update.
- Audit event.
- Escalation path.
- Agent performance signals.

## Accessibility Requirements

- All interactive controls must be keyboard accessible.
- Use visible focus states.
- Do not rely on color alone for status.
- Use accessible labels for forms and buttons.
- Use clear headings and logical reading order inside cards.
- Use readable contrast.
- Avoid truncating critical booking, payment, refund, or rule information.
- Ensure the design remains usable at desktop call center resolutions.
- Icons need accessible names unless decorative.
- Error, warning, success, and blocked states need text labels.

## Visual Direction

- Build a dense but calm operational interface.
- Avoid a marketing-style landing page.
- Avoid large decorative hero sections.
- Use practical panels, tabs, forms, status chips, menus, and action cards.
- Keep cards compact and purposeful.
- Use restrained color with clear semantic status treatment.
- Make the active customer and active booking visually unmistakable.

## Example Data

Use realistic Carnival UK sample data:

- Customer: Grace Chen
- Channel: Phone call
- Loyalty: Platinum
- Vessel: Carnival Venezia
- Port: Southampton
- Sail date: 10 Jun 2026
- Active booking: BK-78431-mp1dzew3
- Related booking: BK-78442-mp1dzew3
- Guest example: Scarlett White
- Example items: Spa treatment, Havana Bar experience, oversized mug package
- Example blocked state: requested spa slot is sold out

## Critical Product Guardrails

- A session represents one customer interaction.
- A session may reference multiple related bookings.
- Only one booking can be active for execution.
- Switching the active booking requires a clear UI moment.
- Every action card must show the affected booking.
- Financial impact must be visible before confirmation.
- The operator confirms before OMS execution.
- After execution, the system shows confirmation, email status, note status, and audit status.

## Output Goal

Create a polished, realistic, functional prototype screen set that demonstrates how a call center operator uses an AI Order Agent to handle Carnival UK booking workflows with speed, safety, and confidence.
