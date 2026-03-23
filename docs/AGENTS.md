# Agent Guide — Component Behavioral Contracts

This file is the operational manual for AI agents working with component
contracts. Read this before reading or writing any contract file.

---

## Your role

You extract behavioral intent from Figma components and encode it as a
structured contract. You are not implementing the component. You are not
describing its appearance. You are documenting what it *does* in response to
events, actions, and size changes.

---

## Connecting to the Figma MCP server

The Figma Dev Mode MCP server runs locally:

```
http://127.0.0.1:3845/mcp
```

### Session initialization (required before every tool call)

```http
POST /mcp
Content-Type: application/json
Accept: application/json, text/event-stream

{ "jsonrpc": "2.0", "id": 1, "method": "initialize",
  "params": { "protocolVersion": "2024-11-05",
               "capabilities": {},
               "clientInfo": { "name": "agent", "version": "1.0" } } }
```

Capture the `mcp-session-id` header from the response. Include it as
`mcp-session-id: <value>` on every subsequent request.

Then send the initialized notification:

```http
POST /mcp
mcp-session-id: <value>
Content-Type: application/json
Accept: application/json, text/event-stream

{ "jsonrpc": "2.0", "method": "notifications/initialized", "params": {} }
```

### Available tools

| Tool | When to use |
|---|---|
| `get_metadata` | First pass — discover all nodes, variants, and hierarchy for a page or frame |
| `get_design_context` | For each variant — extract rendered code and behavioral logic |
| `get_variable_defs` | Extract design tokens (colors, spacing) — use only for naming, not values |
| `get_screenshot` | Verify visual interpretation; do not extract pixel values from it |

---

## How to extract a component contract from Figma

### Step 1 — Discover pages

Call `get_metadata` with no `nodeId` or with page IDs (`0:1`, `0:2`, …) until
you find the component set. Look for frames containing `<symbol>` nodes — those
are component variants.

### Step 2 — Map the variant matrix

Read each variant name. Figma encodes the variant matrix as
`Property=Value, Property=Value` strings. List every unique property and its
possible values. These become your `props` and `states`.

```
State=Rest, Switch=True
State=Hovered, Switch=True
State=Disabled, Switch=False
→ props: checked (true/false)
→ states: rest, hovered, disabled
```

### Step 3 — Read the design context for each variant

Call `get_design_context` on each variant node. Read the generated code for:
- Conditional class names → state-driven visual changes (confirms a state exists)
- Prop names and types → confirms the prop API
- Position changes (left/right, top/bottom) → thumb/indicator movement on toggle
- `justify-end` vs default alignment → indicates value is communicated through position

### Step 4 — Infer missing states

Figma often omits states that have no visual difference from their neighbors.
Always document:
- `focused` — required for keyboard accessibility even if no Figma variant exists
- `pressed` — required for pointer fidelity even if no Figma variant exists

### Step 5 — Fill the contract

Work through each top-level key in order. Rules:

- Include every key even if its value is `{}` or `null`.
- Do not write dimensions, colors, font sizes, border radii, or shadow values.
- Do not write CSS class names or Tailwind utilities.
- Write behavior in plain English, present tense: "transitions to hovered state",
  "calls onChange with new value", "moves focus to next focusable element".
- `figma_property` on a prop maps to the Figma variant property name exactly.
- `figma_variant` on a state maps to the Figma variant label prefix exactly.

### Step 6 — Validate

Before saving, verify:
- Every Figma variant has a corresponding entry in `states`
- Every state has entry and exit conditions
- Every `emits` key has a `not_emitted_when` guard
- `constraints.required_combinations` includes any required accessibility attributes
- `scenarios` covers at minimum: happy path, keyboard path, disabled path,
  controlled-update path

---

## Key inference rules

| Observation in design context | Inference |
|---|---|
| Prop flips position of a child element | `state_machine.toggle_action` exists |
| Conditional class on hover | `state: hovered` exists with pointer triggers |
| Opacity/cursor change on a state | confirm state exists; do not write the opacity value |
| A variant is labeled `Disabled` | `constraints.conditional_behaviors` must block all triggers when disabled |
| No `Focused` variant in Figma | Add `focused` state anyway; note `figma_variant: null` |
| Component is a leaf (no `<slot>`) | `composition.slots: {}` |
| Component consumes no context | `composition.parent_requirements.context: none` |

---

## What to leave empty vs. what to omit

- Leave a key with `{}` or `[]` when it applies to the component concept but
  has no current definition (e.g., `responsive.behaviors: {}`).
- Never omit a top-level key entirely — omission signals the key was not
  considered, not that it is empty.
- `null` is acceptable for fields that are intentionally absent
  (e.g., `figma_variant: null` for a state with no Figma counterpart).

---

## Common mistakes

| Mistake | Fix |
|---|---|
| Writing `font-size: 92px` in a prop | Remove — visual detail, not behavioral |
| Writing `color: #ecedf0` anywhere | Remove — belongs in design tokens |
| Listing only Figma-defined states | Always add `focused` and `pressed` |
| Writing `emits: change` without a payload | Always include payload and suppression rules |
| Leaving `constraints` empty for a disabled component | Always add the disabled suppression rule |
| Copying Tailwind class names into descriptions | Translate to behavior: "thumb moves to the right" |
