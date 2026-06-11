---
name: component-contract-builder
description: >-
  Build a component behavioral contract from a natural-language prompt plus a Figma
  MCP by assembling predefined building blocks. Self-contained and portable: it runs
  in an empty directory with no other project files, carries its own block library
  (see BLOCKS.md), and writes one JSON contract to the current working directory.
  Reads Figma via the Figma Console MCP (Desktop Bridge, preferred) or the official
  Figma MCP (URL-based, fallback). Use whenever someone wants to create, extract,
  recheck, or update a component behavioral contract without any repo scaffolding.
---

# Component Contract Builder — Portable Authoring Kit

You assemble a **behavioral contract**: the spec that sits between a Figma design and the
agent that will implement the component. It captures every event-, interaction-, and
resize-driven behavior — and **nothing visual**. The consumer is an LLM, so there is no
machine validator; your judgment plus the rules here are the enforcement.

This skill is **self-contained**. Everything you need is this file plus `BLOCKS.md` in the
same folder. It assumes **no** other files exist — no template, no sibling contracts, no
project config. The only runtime inputs are a user's prompt and a Figma MCP. The only output
is one JSON file written to the current working directory.

---

## 1. What you produce

One JSON object — a single contract for one component — that is:

- **Behavioral, not visual.** No colors, sizes, typography, spacing, radii, shadows,
  z-index, durations, or class names anywhere. Those live in Figma / design tokens.
- **Authoritative over Figma.** Where the prompt and the Figma demo disagree, the contract
  encodes the prompt's intent and records the divergence in a `figma_note`.
- **Assembled from blocks.** A stable required core is always present; every other block is
  included only when the component earns it. Lean by default.

---

## 2. Your two inputs

1. **The natural-language prompt (the user) — primary authority.** What the component is
   *for*, how it behaves, and what is required vs optional. The prompt wins every conflict.
2. **A Figma MCP — structure and visual reference only.** Supplies the variant matrix,
   property definitions, nesting, and a screenshot for anatomy. It is **never** a source of
   pixel values.

Use the prompt to decide **behavior**; use Figma to confirm **structure** (what variants,
props, and nested parts exist).

---

## 3. Figma protocol

Prefer the **Figma Console MCP** (Desktop Bridge). Fall back to the **official Figma MCP**
(URL-based). Use either to read structure and a confirmation screenshot — not pixels.

### Preferred — Figma Console MCP (Desktop Bridge)
1. Probe: `figma_get_status` with `probe: true`. If it is not connected, try
   `figma_reconnect`; if that still fails, tell the user to open the Desktop Bridge plugin
   **or** fall back to the official MCP.
2. Find the target: `figma_get_selection` (what the user is pointing at) is often the
   fastest start; `figma_list_open_files` / `figma_navigate` to switch files.
3. Pull structure: `figma_get_component_for_development` (deep tree: property definitions,
   variant matrix, nested instances, instance overrides, reactions). Capture
   `figma_capture_screenshot` to confirm the current anatomy. Variable tools return token
   **names** only, never resolved values.
4. **Large output:** `figma_get_component_for_development` can exceed the token limit and be
   saved to a file. When it does, read it in slices and extract only three things: the
   component-set **property definitions**, the **variant tree**, and each instance's
   **overrides**. Those give you props, states/variants, and nested composition.

### Fallback — official Figma MCP (URL-based)
1. Parse the URL: `figma.com/design/<fileKey>/<file>?node-id=<n-m>` → `fileKey`, and convert
   the node id `-` to `:` (e.g. `1-2` → `1:2`).
2. `get_metadata` (overview: node ids, types, names, sizes, variant symbols) →
   `get_design_context` (reference code + variant/property structure + reactions; read it for
   prop names/types, conditional rendering, and nesting — **not** pixels) →
   `get_screenshot` (download to inspect anatomy) → `get_variable_defs` (token names).

In both cases the screenshot confirms **layout and anatomy**; it is never used to read color,
size, or spacing.

---

## 4. The building-blocks model

A contract is assembled from **blocks** — one per contract section. Each block in `BLOCKS.md`
has a tier, a *when*, a JSON shape, and loose rules. Pick:

- **Required core** — always present.
- **Recommended** — include unless clearly not applicable.
- **Optional** — include only when its *when* applies.

**Lean by default.** Padding a simple component with blocks it does not need is as wrong as
omitting one it does. **One concept, one home:**

- `composition` is the only structural block (no separate "anatomy").
- `state_machine` holds *all* transitions and named operations (no separate
  "transitions"/"interactions").
- `triggers` holds *all* runtime interactions (no separate "interactivity").

Read the block's entry in `BLOCKS.md` before you fill it.

---

## 5. Workflow

1. **Gather** — read the prompt; resolve the Figma source; pick the MCP (Console if
   connected, else the official MCP).
2. **Map structure** — variant matrix + property definitions + nested components + instance
   overrides → these become `props`, `states`/`variants`, and `composition`.
3. **Screenshot** — confirm anatomy/layout. Never extract values.
4. **Merge** — fold in the prompt; resolve every conflict in the prompt's favor and record
   the divergence in a `figma_note`.
5. **Pick blocks** — start from the required core; add recommended/optional blocks from
   `BLOCKS.md` only as their *when* is earned.
6. **Assemble & fill** — write each chosen block, filling it from the prompt + Figma.
   Put real data inputs that are not Figma variant properties under `Extra Props`.
   Cross-reference a nested child's own contract **if one exists**; otherwise inline its key
   props and behaviors (in portable use there is usually no sibling contract to point to).
7. **Self-review** — run the checklist in §9.
8. **Write** — save one JSON file named `<kebab-name>.json` in the **current working
   directory** (or a path the user gives). Confirm it parses (`python3 -m json.tool <file>`).

When **rechecking** an existing contract: re-pull the Figma node, re-read the prompt, diff
both against the current contract, report what is missing or wrong, then edit.

---

## 6. Golden rules

- **No visual values, ever** — dimensions, color, typography, spacing, radii, shadows,
  opacity, z-index, animation durations/easing, breakpoint pixels, or CSS/class names. The
  one exception is a **token name** when it is behaviorally relevant — the name, never the
  value.
  - A *fixed vs. fluid* dimension may be a **behavioral** fact (e.g. "height is fixed, so it
    shows a window, not the whole list"). Capture the **consequence**, never the number.
- **Behavioral test** — could the sentence appear unchanged in a behavioral unit-test
  assertion? Keep "activating it opens the link"; drop "the track turns teal on hover".
- **Contract over Figma** — encode the prompt's behavior even when the Figma demo differs;
  record it in a `figma_note` and restate authority in `notes`.
- **Extra Props** — capture every real data input (sources, URLs/handlers, item
  collections, data series) that a Figma variant cannot express.
- **Cross-reference, don't duplicate** — if a child has its own contract, point to it;
  otherwise inline only its key props/behaviors.
- **JSON only**, one component per file.

---

## 7. Figma signal → block

| What you see in Figma | Where it goes |
|---|---|
| Variant property (e.g. a Size/State axis) | `props` (enum) and/or `states` / `variants` |
| Boolean component property | `props` (boolean) + a `constraints.conditional_behaviors` entry |
| Instance override / instance-swap | `composition.nested_components` (with `used_when`) |
| A nested instance with its own component set | a child contract — cross-reference it, or inline its key props/`behaviors` |
| Internal structural part (incl. chart axes, grid, tooltip) | `composition.nested_elements` |
| Reactions / prototype interactions | `triggers` (+ `state_machine` if transitions are non-trivial) |
| Bound variable (token) | token **name** only if behaviorally relevant; else omit |
| A data value that is not a variant property | `props` → `Extra Props` |
| Static / baked sample (chart bars, curves) | `figma_design_notes` + provisional `props` note; runtime interaction → `triggers` with a `figma_limitation` |

---

## 8. Block index

Full shape + rules for each live in `BLOCKS.md`. Tiers: **R** = required core ·
**Rec** = recommended · **O** = optional (include only when *when* applies).

| Block | Tier | When |
|---|---|---|
| `kind` | R | always (usually `"component"`) |
| `name` | R | always — matches the Figma component name |
| `version` | R | always — semantic version of the behavior |
| `figma` | R | always — `nodeId` + `URL` at minimum |
| `description` | R | always — behavioral, no appearance |
| `props` | R | always — every behavioral input; include `Extra Props` for data |
| `composition` | R | always — the single structural block |
| `constraints` | R | always — at least `conditional_behaviors` |
| `scenarios` | R | always as a key; fill with flows (may be `{}` for the most trivial) |
| `notes` | R | always — closing summary; restates authority over Figma |
| `intent` | Rec | product/feature components — concrete use cases |
| `responsive` | Rec | layout/behavior varies with size |
| `category` | O | the component belongs to a named family (e.g. a chart) |
| `figma_design_notes` | O | Figma is a static/poor mock — screenshots win |
| `modes` | O | an external/parent-driven mode changes behavior |
| `variants` | O | one component renders several major variants |
| `states` | O | distinct interactive UI states (+ entry/exit) |
| `state_machine` | O | it moves between states/variants — all transitions + named ops |
| `triggers` | O | it is interactive (pointer/keyboard/touch/resize/programmatic) |
| `emits` | O | it fires outward events to its consumer |
| `accessibility` | O | a11y matters **and** the prompt calls for it |

---

## 9. Self-review checklist

- [ ] All required-core blocks present; `props` has an `Extra Props` entry for data inputs.
- [ ] No visual values or class names anywhere.
- [ ] Every optional block present is justified by its *when*; none the behavior demands is
      missing.
- [ ] Conflicts with Figma are resolved in the prompt's favor and recorded in a `figma_note`.
- [ ] Nested children are cross-referenced (if they have a contract) or inlined, not
      duplicated.
- [ ] `notes` restates authority-over-Figma and that visuals are out of scope.
- [ ] One JSON file written to the working directory and it parses (`python3 -m json.tool`).
