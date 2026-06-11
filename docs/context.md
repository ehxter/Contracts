# Project Context

## What problem this solves

Figma files describe *how things look*. Code describes *how things work*. When an AI agent
translates a design to code it bridges a gap that has no formal specification — so it
guesses the behavior.

This project formalizes that gap. A **behavioral contract** sits between design and code and
captures everything that cannot be read from pixels: what the component does, which states
and variants it has, what events drive it, what it requires of its parent, and the rules
that bound it.

---

## Core principle: behavior, not appearance

A contract contains **zero visual details** — no colors, sizes, typography, spacing, radii,
shadows. Those live in design tokens and style files. The contract describes what happens,
not what it looks like when it happens.

This separation means a contract survives a visual redesign unchanged, can be read
independently of any styling system, and constrains an AI agent to consistent logic
regardless of the framework it targets.

---

## The three inputs

A contract is the merge of three sources:

1. **Behavioral prompt** (the user) — the primary authority on intent and behavior.
2. **Figma Console MCP** (`figma-console`, Desktop Bridge) — live structure and reference.
3. **Figma MCP** (official, URL-based) — structure and reference from a URL.

The behavioral prompt drives the others. Figma supplies structure (variants, props,
nesting) and a visual reference — never pixel values.

---

## Source-of-truth hierarchy

```
Behavioral prompt (intent)  +  Figma (structure/reference)
                  │  merge (prompt wins conflicts)
                  ▼
   components/<category>/<name>.json   ← source of truth for behavior
                  │  implementation (coding agent reads first)
                  ▼
   Component code + the codebase's own design tokens
```

The contract is upstream of code: code is wrong if it contradicts the contract. And the
**contract is authoritative over Figma** — where they differ, the contract reflects the
intended behavior and records the divergence in a `figma_note`.

---

## The consumer is an LLM

The contract is written to be read by a coding agent (Claude Code), not by a validator.
That shapes the design:

- **No machine validation.** An LLM tolerates and benefits from rich, slightly irregular
  JSON. The guideline is a hierarchical **template** plus a **section policy** plus a
  **skill** — not a JSON Schema.
- **Adaptive, not rigid.** A stable required core is always present; optional sections
  appear only when the component earns them. Lean components stay lean.
- **Prose is fine.** `description`, `notes`, and `scenarios` are plain present-tense
  English; precision matters more than a fixed grammar.

---

## What this is not

- Not documentation (contracts are not Storybook stories).
- Not test files (they can *generate* tests; they are not tests).
- Not a visual spec (that is Figma's job).
- Not an API spec or a style guide.

---

## Intended consumers

| Consumer | How they use it |
|---|---|
| AI coding agent | Reads the contract before implementing; bounds generated logic by `props`/`states`/`constraints`/`scenarios` |
| Human developer | Understands expected behavior before writing/reviewing code |
| The authoring agent | Produces/updates contracts via the `component-contract` skill |
| Another contract | Cross-references this component's `composition` when nesting it |
