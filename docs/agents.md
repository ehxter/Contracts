# Agent Guide — The Figma → Contract Pipeline

This is the operational manual for the AI agent that **authors** component contracts. It is
the in-depth companion to the skill at `.claude/skills/component-contract/SKILL.md` — load
the skill for the rule book; read this for the tooling protocol and worked workflow.

> The contract is the source of truth for behavior. Figma is a visual/structural reference.
> The behavioral prompt is the primary authority on intent. Where they disagree, the
> **contract wins** and records the divergence.

---

## The three inputs

| Input | Role | Authority |
|---|---|---|
| **Behavioral prompt** (the user) | Intent, behavior, what is required vs optional | **Primary** — wins conflicts |
| **Figma Console MCP** (`figma-console`) | Live structure: property defs, variant matrix, instance overrides, reactions, screenshots | Structure & visual reference |
| **Figma MCP** (official, URL-based) | Node structure/reference from a URL, no desktop plugin | Structure & visual reference |

Figma is **never** the source of pixel values (color, size, type, spacing, radius, shadow).
Those live in Figma/design tokens and stay out of the contract.

---

## Connecting to Figma

### Option A — Figma Console MCP (Desktop Bridge, preferred when connected)

A WebSocket bridge to the Figma desktop app via the Desktop Bridge plugin. It exposes the
live plugin runtime, so it sees component property definitions, instance overrides, and
reactions that a plain render does not.

1. Probe the connection first:
   - `figma_get_status` with `probe: true` → confirms the connected file and that the plugin
     is responsive. If it reports "No connection / Desktop Bridge may need to be opened,"
     try `figma_reconnect`; if that fails, tell the user to open the Desktop Bridge plugin
     (Plugins → Development → Figma Desktop Bridge) **or** fall back to Option B.
2. Find the target:
   - `figma_get_selection` — read what the user has selected (often the fastest start).
   - `figma_list_open_files` / `figma_navigate` — switch among connected files.
3. Pull the component:
   - `figma_get_component_for_development` (nodeId) → deep tree (componentPropertyDefinitions,
     nested components, instance `componentProperties`, reactions, boundVariables).
   - `figma_capture_screenshot` (nodeId) → verify the current state (use after any edit).
   - variable tools (`figma_get_variables` / `figma_get_token_values`) → token **names**
     only, never raw values.

**Large output:** `figma_get_component_for_development` can exceed the token limit and be
written to a file instead. When that happens, read the file in slices and extract just:
the component-set **property definitions**, the **variant tree**, and each instance's
**`componentProperties`** (overrides). Those three give you props, states/variants, and
nested composition.

### Option B — Figma MCP (official, URL-based; no desktop plugin)

Reads a node directly from a Figma URL. Use for shared links, headless work, when the
Desktop Bridge isn't connected, or to cross-check Option A.

Parse the URL: `figma.com/design/<fileKey>/<file>?node-id=<n-m>` → `fileKey`, and `node-id`
with `-` → `:` (e.g. `1-2` → `1:2`).

1. `get_metadata` (fileKey, nodeId) → structure overview: node ids, types, names, sizes.
   For a component set you'll see the variant symbols (e.g. `Size=Large`, `Size=Small`).
2. `get_design_context` (fileKey, nodeId) → reference code + variant/property structure +
   bound design tokens + reactions. Read it for prop names/types, conditional rendering,
   and nesting — **not** for pixel values.
3. `get_screenshot` (fileKey, nodeId) → visual confirmation (returns a short-lived URL;
   download to inspect).
4. `get_variable_defs` (fileKey, nodeId) → token names.

---

## Reconciliation rules

1. **Prompt → behavior. Figma → structure.** The prompt says what the component *does* and
   what is required vs optional; Figma confirms what exists (variants, props, nesting).
2. **Contract is authoritative over Figma.** Encode the prompt's behavior even when the
   Figma demo state differs (e.g. an image shown by default in Figma but optional per the
   prompt). Record the divergence in a `figma_note` and restate authority in `notes`.
3. **No pixels.** Strip all visual values; keep behavioral signal only.
4. **Static mocks (charts).** Screenshots win for look/proportion; code/measurements are
   approximate; counts/shapes/labels are data-driven. Capture these caveats in
   `figma_design_notes` and mark provisional prop sets with a `_note`.
5. **Cross-check** the two MCPs when both are available.

---

## Extract workflow (worked)

1. **Inputs** — read the prompt; resolve the Figma source; pick the MCP (console if
   connected, else URL-based).
2. **Structure** — variant matrix + component property definitions + nested components +
   instance overrides → `props`, `states`/`variants`, `composition.nested_components`.
3. **Screenshot** — confirm anatomy/layout; never extract values.
4. **Merge** — fold in the prompt; resolve conflicts in the prompt's favor.
5. **Choose sections** — start from the required core; add optional sections only as the
   policy in `docs/requirements.md` earns them.
6. **Fill** — copy `templates/contract.template.json`, strip the underscore-annotation keys,
   keep what you need.
7. **Authority & data** — add `figma_note` for divergences; put non-Figma data inputs under
   `Extra Props`.
8. **Cross-reference** shared/nested components instead of duplicating them.
9. **Write** `components/<category>/<name>.json`.
10. **Self-review** against the checklist in the skill / `docs/requirements.md`, then
    `python3 -m json.tool <file>` to confirm it parses.

### Rechecking / updating an existing contract
Re-pull the Figma node and re-read the prompt, diff both against the current contract,
**report what is missing or wrong**, then edit (identify gaps → fix → keep the contract
authoritative over the Figma demo state).

---

## Mapping Figma signals to contract sections

| What you see in Figma | Where it goes |
|---|---|
| Component-set variant property (`Size`, `State`) | `props` (enum) and/or `states`/`variants` |
| Boolean component property (`Show Action`) | `props` (boolean) + a `constraints.conditional_behaviors` entry |
| Instance override / instance-swap | `composition.nested_components` (with `used_when`) |
| A nested instance that has its own component set | a separate child contract; cross-reference it |
| A nested instance / internal structural part (incl. chart axes, grid, tooltip) | `composition.nested_components` / `composition.nested_elements` |
| Reactions / prototype interactions | `triggers` (+ `state_machine` if transitions are non-trivial) |
| Bound variable (token) | token **name** only if behaviorally relevant; otherwise omit |
| A data value that is not a variant property | `props` → `Extra Props` |
| Static/baked sample (chart bars, curves) | `figma_design_notes` + provisional `props._note`; runtime interactions → `triggers` with a `figma_limitation` |

---

## How a coding agent consumes the contract (downstream)

The implementing agent (e.g. Claude Code) reads `components/<category>/<name>.json` **before
writing code**, then:
- treats populated sections as requirements and absent optional sections as "not applicable
  to this component";
- implements behavior from `props`/`states`/`triggers`/`constraints`/`scenarios`;
- pulls all visual values from the target codebase's design tokens (never from the
  contract, which has none);
- respects `composition` (slots, parent requirements) and follows cross-referenced child
  contracts.
