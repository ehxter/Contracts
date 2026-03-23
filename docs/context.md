# Project Context

## What problem this solves

Figma files describe *how things look*. Code describes *how things work*.
When an AI agent translates a Figma design to code, it bridges a gap that has
no formal specification — it guesses the behavior.

This project formalizes that gap. A behavioral contract sits between design and
code, capturing everything that cannot be read from pixels:

- What states the component can be in
- What events cause state transitions
- What the component emits when the user interacts with it
- What accessibility behavior is required
- What the parent component is obligated to provide

---

## Core principle

**Behavior is not appearance.**

A component contract contains zero visual details: no colors, no font sizes, no
dimensions, no spacing values, no border radii. Those belong in design tokens
and style files. Contracts describe what happens, not what it looks like when it
happens.

This separation means:
- A contract survives a visual redesign unchanged
- A contract can be validated independently of any styling system
- An AI agent constrained by a contract produces consistent logic regardless of
  which styling framework it targets

---

## Source of truth hierarchy

```
Figma design file
        ↓  (behavioral extraction via MCP)
component/contract.json   ← source of truth for behavior
        ↓  (implementation)
Component.tsx + styles
        ↓  (validation)
Tests derived from scenarios and state_machine
```

The contract is upstream of code. Code is wrong if it contradicts the contract.
The contract is wrong if it contradicts Figma-defined states or props.

---

## Controlled vs. uncontrolled pattern

Contracts document both modes explicitly because they have different behavioral
rules. In controlled mode the parent owns state; in uncontrolled mode the
component owns it. The `onChange` event is only defined in terms of the
**user-initiated toggle** — never as a reaction to a programmatic prop change.

---

## Relationship to Figma MCP

The Figma Dev Mode MCP server (`http://127.0.0.1:3845/mcp`) provides:

- `get_metadata` — component set structure, variant names, node hierarchy
- `get_design_context` — rendered code showing conditional logic per variant

These are the two primary extraction tools. The variant matrix in Figma directly
maps to the `props` × `states` space in the contract. Every variant label
encodes both a prop value and a state value.

---

## What this is not

- Not a documentation tool (contracts are not Storybook stories)
- Not a test file (contracts generate tests; they are not tests themselves)
- Not a visual spec (that is Figma's job)
- Not an API spec (that is OpenAPI's job)
- Not a style guide (that is design tokens' job)

---

## Intended consumers

| Consumer | How they use it |
|---|---|
| AI code agent | Reads contract before implementing; uses state_machine and constraints to bound generated logic |
| Human developer | Reads contract to understand expected behavior before writing or reviewing code |
| Test generator | Reads scenarios and state_machine to produce interaction tests |
| Design system auditor | Reads emits and accessibility to verify compliance |
| Another component's contract | References this component's composition rules when nesting |
